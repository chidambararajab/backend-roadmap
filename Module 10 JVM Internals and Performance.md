# Module 10: JVM Internals and Performance

## 10.1 JVM Architecture

The Java Virtual Machine (JVM) is the foundation of the Java platform, providing an execution environment that abstracts away the underlying hardware and operating system details.

### ClassLoader Subsystem

The ClassLoader subsystem is responsible for loading class files into the JVM memory.

```java
*// Example: Using custom ClassLoader*
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            *// Load class file as byte array*
            byte[] classData = loadClassData(name);
            
            *// Define the class*
            return defineClass(name, classData, 0, classData.length);
        } catch (IOException e) {
            throw new ClassNotFoundException("Could not load class " + name, e);
        }
    }
    
    private byte[] loadClassData(String name) throws IOException {
        *// Convert class name to file path*
        String path = name.replace('.', '/') + ".class";
        Path classPath = Paths.get("custom-classes", path);
        return Files.readAllBytes(classPath);
    }
    
    public static void main(String[] args) throws Exception {
        CustomClassLoader loader = new CustomClassLoader();
        Class<?> clazz = loader.loadClass("com.example.MyClass");
        Object instance = clazz.getDeclaredConstructor().newInstance();
        System.out.println("Created instance: " + instance);
    }
}
```

**Loading Phase:**

- Reads .class file
- Verifies format of class file
- Creates internal representation in method area
- Creates a Class object in the heap

**Linking Phase:**

- **Verify**: Ensures bytecode correctness and JVM safety
- **Prepare**: Allocates memory for static fields, sets default values
- **Resolve**: Replaces symbolic references with direct references (optional)

**Initialization Phase:**

- Executes static initializers and initializes static fields
- Happens on first active use of the class

**ClassLoader Hierarchy:**

- **Bootstrap ClassLoader**: Loads core Java classes from rt.jar (written in native code)
- **Extension/Platform ClassLoader**: Loads classes from extension directories
- **Application ClassLoader**: Loads classes from application classpath
- **Custom ClassLoaders**: User-defined classloaders for specialized loading

**ClassLoader Delegation:**

- Parent-first delegation model
- Child delegates to parent before attempting to load
- Ensures core classes cannot be overridden

> **Deep Dive Tip:** The ClassLoader delegation model uses a parent-first approach for security and consistency. When loading a class, a ClassLoader first delegates to its parent. Only if the parent fails to load the class does the child attempt to load it. This ensures, for example, that core classes like `java.lang.String` are always loaded by the Bootstrap ClassLoader, preventing malicious code from substituting these fundamental classes. However, there are cases where you might want to override this behavior, such as OSGi frameworks which use a more complex delegation model to support modular applications.
> 

> **Interviewer Insight:** When discussing ClassLoaders in interviews, emphasize their role in Java's security architecture and dynamic class loading capabilities. A strong candidate should understand how ClassLoader isolation enables features like application servers (where each application has its own ClassLoader), hot deployment, and dynamic loading of plugins. Also mention potential pitfalls like ClassLoader leaks, which can occur when a reference to a loaded class (or object of that class) prevents the ClassLoader itself from being garbage collected, potentially leading to OutOfMemoryError in long-running applications.
> 

### Runtime Data Areas

The JVM defines various runtime data areas that are used during execution of a program.

### Method Area/Metaspace

```java
*// Method area stores class metadata*
Class<?> clazz = String.class;
System.out.println("Class name: " + clazz.getName());
System.out.println("Super class: " + clazz.getSuperclass().getName());
System.out.println("Methods: " + Arrays.toString(clazz.getDeclaredMethods()));
System.out.println("Fields: " + Arrays.toString(clazz.getDeclaredFields()));
```

- Contains class metadata, method code, constant pool
- Shared among all threads
- Called PermGen in older JVMs (before Java 8)
- Called Metaspace in Java 8+ (no fixed size, can expand to native memory)

### Heap Memory

```java
*// Everything allocated on heap*
List<byte[]> allocations = new ArrayList<>();
for (int i = 0; i < 100; i++) {
    *// Allocate 1MB array*
    allocations.add(new byte[1024 * 1024]);
    System.out.println("Allocated " + (i + 1) + " MB");
}
```

- Primary runtime data area where objects are allocated
- Shared among all threads
- Managed by garbage collector
- Divided into generations (young, old)
- Size controlled by -Xmx and -Xms flags

### Stack Memory

```java
*// Example: Recursive method demonstrating stack growth*
public static void recursiveMethod(int depth) {
    *// Local variables are allocated on the stack*
    int localVar = depth;
    
    *// Base case*
    if (depth > 0) {
        *// Each recursive call adds a new frame to the stack*
        recursiveMethod(depth - 1);
    } else {
        *// Print stack trace to show stack depth*
        new Exception("Stack trace at depth 0").printStackTrace();
    }
    
    *// This variable will be allocated for each stack frame*
    byte[] buffer = new byte[1024]; *// 1KB local array*
}
```

- Thread-specific
- Contains frames for method calls
- Each frame contains local variables, operand stack, and frame data
- LIFO (Last-In-First-Out) structure
- Fixed size per thread (default varies by JVM)
- Size controlled by -Xss flag

### PC Registers

- Thread-specific
- Holds the address of the current instruction
- If executing native method, value is undefined

### Native Method Stacks

- Thread-specific
- Used for executing native methods
- Similar to Java stack but for native code
- Size controlled by -Xoss flag (rarely used)

> **Deep Dive Tip:** Understanding the distinction between heap and stack memory is crucial for optimizing Java applications. Stack memory is faster to allocate and deallocate (simple pointer manipulation) but limited in size and lifetime (method scope). Heap memory is more flexible but requires garbage collection. For performance-critical code, consider whether objects can be kept on the stack instead of the heap. For example, using primitive types instead of wrapper classes, or using value types (records in modern Java) for small immutable objects. In future Java versions, the Valhalla project aims to bring true "value types" that can be allocated on the stack or inline in other objects.
> 

> **Interviewer Insight:** When discussing JVM memory areas, a strong candidate should understand the implications for concurrency and performance. For example, they should know that the heap is shared, requiring synchronization for thread safety, while each thread has its own stack, making local variables inherently thread-safe. They should also understand how different areas impact performance—e.g., thread creation is expensive partly because each thread needs its own stack memory, while heavy use of the metaspace (lots of classes/methods) affects start-up time and memory footprint. A good follow-up question might be about tuning strategies for applications with many threads versus applications with many classes.
> 

### Execution Engine

The Execution Engine executes the bytecode loaded by the ClassLoader subsystem.

**Interpreter:**

- Reads and executes bytecode instructions one by one
- Simple but slower execution
- No optimization during execution

**JIT Compiler:**

- Compiles frequently executed bytecode to native code
- Improves performance of hot code paths
- C1 compiler (client): fast compilation, basic optimizations
- C2 compiler (server): slower compilation, advanced optimizations
- Tiered compilation: starts with interpretation, then C1, then C2

```java
*// Example of a method that benefits from JIT compilation*
public static long sumRange(int n) {
    long sum = 0;
    for (int i = 1; i <= n; i++) {
        sum += i;
    }
    return sum;
}

*// Benchmark showing JIT warmup*
public static void main(String[] args) {
    *// First call - likely interpreted*
    long start1 = System.nanoTime();
    long result1 = sumRange(1_000_000);
    long time1 = System.nanoTime() - start1;
    System.out.println("First call: " + time1 + " ns, result: " + result1);
    
    *// Warm up JIT*
    for (int i = 0; i < 10000; i++) {
        sumRange(1000);
    }
    
    *// After JIT compilation - much faster*
    long start2 = System.nanoTime();
    long result2 = sumRange(1_000_000);
    long time2 = System.nanoTime() - start2;
    System.out.println("After warmup: " + time2 + " ns, result: " + result2);
    System.out.println("Speedup: " + (double)time1/time2 + "x");
}
```

**JIT Optimizations:**

- Method inlining (replacing method calls with body)
- Loop unrolling (reducing loop overhead)
- Escape analysis (stack allocation of non-escaping objects)
- Dead code elimination (removing unreachable code)
- Lock elision (removing unnecessary synchronization)
- Intrinsics (special handling for common methods)

**Garbage Collector:**

- Automatic memory management
- Different algorithms with different trade-offs
- Reclaims memory from unreachable objects
- Prevents memory leaks (in most cases)
- Configurable based on application needs

**Security Manager:**

- Enforces security policies (deprecated in newer Java versions)
- Controls access to sensitive operations
- Replaced by more fine-grained mechanisms in modern Java

> **Deep Dive Tip:** One of the most powerful JIT optimizations is escape analysis, which determines whether an object "escapes" its allocation context (e.g., is returned from a method or stored in a field). If an object doesn't escape and is only used locally, the JIT can perform scalar replacement—allocating the object's fields on the stack instead of the heap, eliminating GC overhead. This is why small, short-lived objects are often more efficient than expected in Java, despite the general perception that object allocation is expensive.
> 

> **Interviewer Insight:** When discussing the Execution Engine, emphasize the adaptive nature of modern JVMs. Rather than requiring upfront configuration, the JVM monitors execution patterns and adapts its compilation and optimization strategies accordingly. A good candidate should understand JIT warmup implications—production systems may need a warmup period to reach peak performance, and microbenchmarks must account for JIT behavior to avoid misleading results. They should also be able to explain how to use JVM flags like `-XX:+PrintCompilation` to observe JIT behavior and `-XX:CompileThreshold` to tune compilation thresholds for specific workloads.
> 

### Native Interface

The Java Native Interface (JNI) allows Java code to interact with native code written in languages like C and C++.

```java
*// Java code with native method declaration*
public class NativeExample {
    *// Load the native library*
    static {
        System.loadLibrary("nativelib");
    }
    
    *// Declare native method*
    public native String getNativeMessage();
    
    public static void main(String[] args) {
        NativeExample example = new NativeExample();
        System.out.println(example.getNativeMessage());
    }
}
```

C

```java
*// C implementation (nativelib.c)*
#include <jni.h>
#include "NativeExample.h"  *// Generated with javac -h .*

JNIEXPORT jstring JNICALL Java_NativeExample_getNativeMessage
  (JNIEnv *env, jobject obj) {
    return (*env)->NewStringUTF(env, "Hello from native code!");
}
```

**Key Concepts:**

- **JNI Functions**: Interface for manipulating Java objects from native code
- **Native Method Registration**: Connecting Java method declarations to native implementations
- **JNI Data Types**: Mapping between Java and native types
- **Native Memory Management**: Responsibility for freeing native resources

**JNI Use Cases:**

- Access to platform-specific features
- Performance-critical code sections
- Integration with existing native libraries
- Hardware access not available through Java APIs

**JNI Drawbacks:**

- Platform-dependent code
- Complexity and maintenance overhead
- Potential for memory leaks and crashes
- No automatic memory management
- Loss of Java's security guarantees

> **Deep Dive Tip:** The JNI interface involves considerable overhead for crossing the Java/native boundary. Each call requires marshalling arguments, transitioning from Java to native execution environment, and potentially copying data between Java heap and native memory. For performance-critical code, it's best to design your JNI calls to minimize boundary crossings—use fewer, larger operations rather than many small ones. For example, instead of calling a native method in a loop to process individual elements, pass an entire array in a single call and process it all at once in native code.
> 

> **Interviewer Insight:** When discussing JNI, a strong candidate should acknowledge both its utility and its dangers. They should know that while JNI provides essential capabilities for certain use cases, it's a last resort due to its complexity and risk. Modern alternatives to consider before using JNI include:
> 
> 1. Pure Java solutions (which have improved greatly in recent versions)
> 2. The Foreign Function & Memory API (Java 17+), which provides safer native interop
> 3. Cross-platform libraries that abstract native functionality
> 4. Process-based integration using standard I/O or IPC mechanisms
> 
> They should also understand that JNI code requires rigorous testing across all target platforms, proper error handling, and careful memory management to avoid destabilizing the JVM.
> 

## 10.2 Memory Management

Understanding JVM memory management is essential for diagnosing issues and optimizing performance in Java applications.

### Heap Structure

The Java heap is the runtime data area where objects are allocated. It's divided into generations based on the observation that most objects die young.

```java
*// Monitoring heap usage*
Runtime runtime = Runtime.getRuntime();

*// Initial memory statistics*
System.out.println("Initial Memory Statistics:");
System.out.println("Max Memory: " + runtime.maxMemory() / 1024 / 1024 + " MB");
System.out.println("Total Memory: " + runtime.totalMemory() / 1024 / 1024 + " MB");
System.out.println("Free Memory: " + runtime.freeMemory() / 1024 / 1024 + " MB");
System.out.println("Used Memory: " + (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024 + " MB");

*// Allocate memory to demonstrate heap usage*
List<byte[]> allocations = new ArrayList<>();
for (int i = 0; i < 10; i++) {
    allocations.add(new byte[1024 * 1024]); *// 1MB*
}

*// Updated memory statistics*
System.out.println("\nAfter Allocations:");
System.out.println("Total Memory: " + runtime.totalMemory() / 1024 / 1024 + " MB");
System.out.println("Free Memory: " + runtime.freeMemory() / 1024 / 1024 + " MB");
System.out.println("Used Memory: " + (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024 + " MB");

*// Force garbage collection*
System.gc();

*// Statistics after GC*
System.out.println("\nAfter GC:");
System.out.println("Total Memory: " + runtime.totalMemory() / 1024 / 1024 + " MB");
System.out.println("Free Memory: " + runtime.freeMemory() / 1024 / 1024 + " MB");
System.out.println("Used Memory: " + (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024 + " MB");
```

### Young Generation

The Young Generation is where new objects are allocated and is divided into three spaces:

**Eden Space:**

- Initial allocation area for most objects
- Fills up quickly, triggering minor GC
- Surviving objects move to survivor spaces

**Survivor Spaces (S0 and S1):**

- Two equal-sized spaces, only one active at a time
- Objects that survive Eden collection move here
- Objects ping-pong between S0 and S1 on each collection
- Objects that survive multiple collections get promoted to Old Generation

**Age Threshold:**

- Counter tracking how many collections an object has survived
- Default threshold varies by JVM (typically 15)
- Objects exceeding threshold are promoted to Old Generation
- Adaptive threshold based on survivor space utilization

### Old Generation

**Tenured Space:**

- Holds long-lived objects
- Objects promoted from Young Generation
- Collected during major GC (Full GC)
- Typically much larger than Young Generation

**Promotion Rules:**

- Survivor age threshold exceeded
- Survivor space overflow (premature promotion)
- Humongous objects (direct allocation to Old Gen in some collectors)

### Metaspace (Java 8+)

- Replaced PermGen from earlier Java versions
- Stores class metadata
- Allocated in native memory (not heap)
- Can grow dynamically (no fixed size)
- Size controlled by -XX:MetaspaceSize and -XX:MaxMetaspaceSize

> **Deep Dive Tip:** The generational heap design is based on the "weak generational hypothesis" which states that most objects die young, and those that survive for a while tend to live for a long time. This empirical observation holds true for most applications and allows the JVM to optimize garbage collection by focusing more frequent collections on the Young Generation (where they're most effective) while collecting the Old Generation less frequently. The key performance implication is that applications should minimize the number of objects that survive long enough to be promoted to the Old Generation but aren't actually needed long-term, as these "mid-life crisis" objects cause unnecessary promotion overhead and more frequent Full GC events.
> 

> **Interviewer Insight:** When discussing heap structure, a strong candidate should understand the practical implications for application design. They should know that:
> 
> 1. Minor GCs (Young Generation) are typically fast with minimal application pauses
> 2. Major GCs (Old Generation) are slower and can cause noticeable pauses
> 3. Objects that bounce between generations (survive in Young Gen but die before becoming truly old) can cause performance issues
> 4. Right-sizing the generations for the application's memory usage pattern is important
> 
> Ask them about strategies to minimize GC overhead, such as object pooling for frequently created/discarded objects, avoiding unnecessary object allocation in hot code paths, and ensuring short-lived temporary objects don't accidentally get promoted to the Old Generation.
> 

### Non-Heap Memory

Besides the heap, the JVM uses several other memory areas for its operation.

### Stack Memory

```java
*// Demonstrating stack memory usage with recursion*
public static void recursiveMethod(int depth) {
    *// Each call creates a new stack frame with these variables*
    int localVar1 = depth;
    double localVar2 = depth * 1.5;
    boolean localVar3 = depth % 2 == 0;
    
    *// Local object reference (reference on stack, object on heap)*
    StringBuilder builder = new StringBuilder("Depth: " + depth);
    
    *// Local array (reference on stack, array data on heap)*
    int[] localArray = new int[depth > 0 ? depth : 1];
    
    *// Base case*
    if (depth <= 0) {
        System.out.println("Reached base case, stack depth: " + builder.toString());
        return;
    }
    
    *// Add to stack depth with recursive call*
    recursiveMethod(depth - 1);
    
    *// This code executes after the recursive call returns*
    System.out.println("Returned to depth: " + depth);
}
```

**Stack Frame Components:**

- **Local Variables**: Method parameters and locally declared variables
- **Operand Stack**: Working area for bytecode instructions
- **Frame Data**: Constant pool reference, exception table, etc.

**Stack Memory Characteristics:**

- Each thread has its own stack
- Fixed size per thread (default depends on JVM and OS)
- LIFO (Last-In-First-Out) structure
- Automatically managed (no GC needed)
- Fast allocation and deallocation
- Limited size (StackOverflowError if exceeded)

### PC Registers

- One per thread
- Contains address of current JVM instruction
- Small, fixed size memory area
- Used by JVM to keep track of execution

### Native Memory

```java
*// Example: Direct ByteBuffer using native memory*
ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024 * 1024 * 100); *// 100MB*
System.out.println("Direct buffer capacity: " + directBuffer.capacity() / 1024 / 1024 + " MB");

*// Fill buffer with data*
for (int i = 0; i < directBuffer.capacity(); i++) {
    directBuffer.put(i, (byte) (i % 256));
}

*// Read from buffer*
byte value = directBuffer.get(1000);
System.out.println("Value at position 1000: " + value);

*// Direct buffers are not counted in heap usage*
Runtime runtime = Runtime.getRuntime();
System.out.println("Used heap: " + (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024 + " MB");
```

**Native Memory Components:**

- **Direct ByteBuffers**: Memory outside the heap for I/O operations
- **JNI Code**: Memory used by native methods
- **JVM Code**: Internal JVM structures and code cache
- **Thread Stacks**: Native memory for thread stacks
- **Metaspace**: Class metadata (Java 8+)

**Key Characteristics:**

- Not managed by garbage collector
- Not limited by heap size settings
- Can lead to OutOfMemoryError with native memory leak
- Requires explicit management in some cases
- Size limited by process address space and physical memory

### Code Cache

```java
*// JVM flags to monitor code cache// -XX:+PrintCodeCache// -XX:+PrintCodeCacheOnCompilation// -XX:ReservedCodeCacheSize=256m// Method likely to be JIT compiled*
public static long benchmarkMethod(int iterations) {
    long sum = 0;
    for (int i = 0; i < iterations; i++) {
        sum += i;
    }
    return sum;
}

*// Run method many times to trigger JIT compilation*
public static void main(String[] args) {
    for (int i = 0; i < 10000; i++) {
        benchmarkMethod(1000);
    }
    System.out.println("Final result: " + benchmarkMethod(1000));
}
```

**Code Cache Purpose:**

- Stores compiled native code from JIT compiler
- Improves performance by avoiding recompilation
- Limited size (can cause performance degradation if full)
- Size controlled by -XX:ReservedCodeCacheSize and -XX:InitialCodeCacheSize

> **Deep Dive Tip:** Managing native memory can be challenging because it's outside the JVM's automatic management. Direct ByteBuffers, while useful for high-performance I/O, can cause issues if not properly managed. They are garbage collected, but only when the ByteBuffer object itself becomes unreachable. In memory-constrained environments, it's possible to run out of native memory while still having available heap space. To mitigate this, consider:
> 
> 1. Explicitly calling `cleaner.clean()` (pre-Java 9) or using `Cleaner` (Java 9+) to release native memory early
> 2. Using memory-mapped files instead of in-memory buffers for very large data
> 3. Monitoring native memory usage with tools like NMT (Native Memory Tracking)

> **Interviewer Insight:** When discussing non-heap memory, a strong candidate should understand the implications for application stability and performance. They should know that monitoring only heap usage is insufficient—native memory issues can cause OutOfMemoryErrors even when heap metrics look healthy. Ask them about strategies for diagnosing native memory leaks (e.g., using NMT, tracking DirectByteBuffer allocations) and how they would design applications to efficiently use both heap and native memory. They should also understand the trade-offs involved in tuning thread stack size: too small leads to StackOverflowError, while too large limits the maximum number of threads and wastes memory.
> 

### Memory Allocation

The JVM uses sophisticated techniques to make object allocation efficient.

### TLAB (Thread Local Allocation Buffers)

```java
*// JVM flags for TLAB control// -XX:+UseTLAB (enabled by default)// -XX:+PrintTLAB// -XX:TLABSize=1m// Code that benefits from TLAB (many small allocations)*
public static void allocateMany() {
    for (int i = 0; i < 1_000_000; i++) {
        *// Small objects allocated in TLAB without synchronization*
        Object obj = new Object();
        *// Do something with obj to prevent optimization*
        if (obj.hashCode() % 1000000 == 0) {
            System.out.println("Rare event");
        }
    }
}
```

**TLAB Characteristics:**

- Small region of Eden space reserved for each thread
- Enables allocation without synchronization
- Improves performance in multi-threaded applications
- When TLAB fills up, thread gets a new one
- Size adaptively adjusted based on allocation rate

### PLAB (Promotion Local Allocation Buffers)

- Similar to TLAB but for survivor spaces and old generation
- Used during garbage collection for object promotion
- Reduces synchronization overhead during collection
- Size controlled by -XX:OldPLABSize and related flags

### Bump-the-Pointer Allocation

```java
*// Conceptual representation of bump-the-pointer allocation*
class ConceptualHeap {
    private byte[] memory = new byte[1000000]; *// Simplified heap*
    private int allocationPointer = 0;         *// Current allocation position*
    
    public Object allocate(int size) {
        *// Check if enough space available*
        if (allocationPointer + size > memory.length) {
            *// Trigger garbage collection or expand heap*
            System.out.println("Not enough space");
            return null;
        }
        
        *// Allocate by simply incrementing the pointer*
        int objectStart = allocationPointer;
        allocationPointer += size;
        
        return new HeapObject(objectStart, size);
    }
    
    *// Simplified representation of an object in our heap*
    class HeapObject {
        private final int address;
        private final int size;
        
        public HeapObject(int address, int size) {
            this.address = address;
            this.size = size;
        }
        
        @Override
        public String toString() {
            return "Object@" + address + " (size: " + size + ")";
        }
    }
}
```

**Bump-the-Pointer Mechanism:**

- Simplest allocation strategy
- Maintains a pointer to the next free memory location
- Allocation simply advances the pointer
- Very fast for contiguous free memory
- Used in Young Generation and TLABs

### Free List Allocation

- Used in fragmented memory regions (typically Old Generation)
- Maintains lists of free memory blocks
- Allocation finds suitable block in free list
- Slower than bump-the-pointer but handles fragmentation
- Various policies for selecting blocks (first-fit, best-fit, etc.)

> **Deep Dive Tip:** The combination of TLABs and bump-the-pointer allocation makes object creation in Java surprisingly fast in the common case. For small objects, allocation can be as simple as a few CPU instructions to check space and increment a pointer, with no synchronization required. This efficiency is why many Java performance guidelines that suggest avoiding object creation (based on older JVMs) are now outdated. Modern JVMs are highly optimized for creating and collecting short-lived objects. The key is ensuring those objects don't survive minor collections unnecessarily, as promotion to the Old Generation is much more expensive than initial allocation.
> 

> **Interviewer Insight:** When discussing memory allocation, a strong candidate should understand the performance implications for different application patterns. They should know that:
> 
> 1. Allocation of small, short-lived objects is very fast due to TLABs and bump-the-pointer
> 2. Thread-local allocation patterns (where objects are created and used within one thread) are more efficient than shared allocation patterns
> 3. Large object allocation (especially those that bypass Eden) can be significantly more expensive
> 4. High allocation rates in multiple threads can cause TLAB thrashing and increased GC overhead
> 
> Ask them about strategies for managing allocation patterns, such as object pooling for large or expensive-to-create objects, avoiding unnecessary allocations in hot code paths, and designing data structures to minimize garbage creation during operations.
> 

### Reference Types

Java provides different reference types to give programmers control over the lifecycle of objects with respect to garbage collection.

```java
import java.lang.ref.*;
import java.util.Map;
import java.util.WeakHashMap;
import java.util.concurrent.ConcurrentHashMap;

public class ReferenceTypesExample {
    public static void main(String[] args) throws InterruptedException {
        *// Strong references - standard object references*
        strongReferences();
        
        *// Weak references - collected when only weakly reachable*
        weakReferences();
        
        *// Soft references - collected when memory is tight*
        softReferences();
        
        *// Phantom references - for cleanup actions*
        phantomReferences();
        
        *// WeakHashMap example*
        weakHashMapExample();
    }
    
    static void strongReferences() {
        *// Strong reference - standard object reference*
        Object strongRef = new Object();
        System.out.println("Strong reference created: " + strongRef);
        
        *// Object remains in memory as long as strongRef exists*
        strongRef = null; *// Now eligible for GC*
        System.gc();
        System.out.println("After strongRef = null and GC");
    }
    
    static void weakReferences() throws InterruptedException {
        *// Create object with only a weak reference*
        Object referent = new Object();
        WeakReference<Object> weakRef = new WeakReference<>(referent);
        
        System.out.println("Weak reference created, object: " + weakRef.get());
        
        *// Remove strong reference*
        referent = null;
        
        *// Force garbage collection*
        System.gc();
        Thread.sleep(100); *// Give GC a chance to run*
        
        *// Check if object still exists*
        Object obj = weakRef.get();
        System.out.println("After GC, weak reference " + 
                         (obj == null ? "was cleared" : "still exists"));
    }
    
    static void softReferences() throws InterruptedException {
        *// Create object with a soft reference*
        Object referent = new Object();
        SoftReference<Object> softRef = new SoftReference<>(referent);
        
        System.out.println("Soft reference created, object: " + softRef.get());
        
        *// Remove strong reference*
        referent = null;
        
        *// Force garbage collection*
        System.gc();
        Thread.sleep(100); *// Give GC a chance to run*
        
        *// Check if object still exists - likely yes, as memory isn't tight*
        Object obj = softRef.get();
        System.out.println("After GC, soft reference " + 
                         (obj == null ? "was cleared" : "still exists"));
        
        *// To force collection of soft references, we'd need to create memory pressure// This is simplified and may not work in all environments*
        try {
            System.out.println("Creating memory pressure...");
            byte[][] memory = new byte[1000][];
            for (int i = 0; i < 1000; i++) {
                memory[i] = new byte[1024 * 1024]; *// 1MB each*
            }
        } catch (OutOfMemoryError e) {
            System.out.println("Out of memory error triggered");
        }
        
        *// Check soft reference again*
        obj = softRef.get();
        System.out.println("After memory pressure, soft reference " + 
                         (obj == null ? "was cleared" : "still exists"));
    }
    
    static void phantomReferences() throws InterruptedException {
        *// Create reference queue for notification*
        ReferenceQueue<Object> refQueue = new ReferenceQueue<>();
        
        *// Create object with a phantom reference*
        Object referent = new Object() {
            @Override
            protected void finalize() {
                System.out.println("Finalize called on referent");
            }
        };
        PhantomReference<Object> phantomRef = new PhantomReference<>(referent, refQueue);
        
        System.out.println("Phantom reference created");
        System.out.println("Phantom get() always returns null: " + phantomRef.get());
        
        *// Remove strong reference*
        referent = null;
        
        *// Force garbage collection*
        System.gc();
        Thread.sleep(100); *// Give GC a chance to run*
        
        *// Check if reference has been queued*
        Reference<?> ref = refQueue.poll();
        if (ref != null) {
            System.out.println("Phantom reference queued after GC");
            *// Perform cleanup actions here*
            
            *// Clear the reference*
            ref.clear();
        } else {
            System.out.println("Phantom reference not queued yet");
        }
    }
    
    static void weakHashMapExample() throws InterruptedException {
        *// Create weak hash map*
        WeakHashMap<Key, String> weakMap = new WeakHashMap<>();
        
        *// Add entries with both strong and weak keys*
        Key strongKey = new Key(1);
        Key weakKey = new Key(2);
        
        weakMap.put(strongKey, "Strong key value");
        weakMap.put(weakKey, "Weak key value");
        
        System.out.println("Initial map: " + weakMap);
        
        *// Remove reference to weak key*
        weakKey = null;
        
        *// Force garbage collection*
        System.gc();
        Thread.sleep(100); *// Give GC a chance to run*
        
        *// Check map contents*
        System.out.println("After GC: " + weakMap);
    }
    
    *// Custom key class for WeakHashMap example*
    static class Key {
        private final int id;
        
        public Key(int id) {
            this.id = id;
        }
        
        @Override
        public String toString() {
            return "Key(" + id + ")";
        }
    }
}
```

### Strong References

- Default reference type (e.g., `Object obj = new Object()`)
- Objects with strong references are never eligible for garbage collection
- Only collected when no strong references exist

### Weak References

- Wrapped with `WeakReference<T>` class
- Collected during next GC cycle if only weakly reachable
- Used for objects that can be recreated if needed
- Common in caches to avoid memory leaks

### Soft References

- Wrapped with `SoftReference<T>` class
- Collected only when memory is low
- JVM tries to keep soft references as long as possible
- Ideal for memory-sensitive caches
- Behavior can be tuned with `XX:SoftRefLRUPolicyMSPerMB`

### Phantom References

- Wrapped with `PhantomReference<T>` class
- Always returns null when dereferenced
- Used with a ReferenceQueue for post-finalization cleanup
- Most flexible but complex reference type
- Used for advanced memory management and resource cleanup

### Reference Queues

- Notifies when a reference's referent is collected
- References are enqueued after their referents are collected
- Enables processing after object becomes unreachable
- Essential for phantom references, optional for weak/soft

> **Deep Dive Tip:** Reference types are particularly useful for implementing caches and resource management systems. For example, WeakHashMap (which uses weak references for keys) is ideal for caches where entries should be removed when the key is no longer used elsewhere. SoftReference is better for memory-sensitive caches that should release entries only under memory pressure. PhantomReference, the most complex type, enables clean-up actions to occur after an object's finalizer has run, making it useful for managing non-memory resources like file handles or off-heap memory that needs special cleanup.
> 

> **Interviewer Insight:** When discussing reference types, a strong candidate should understand their practical applications and trade-offs. They should know:
> 
> 1. Weak references are appropriate for "use it or lose it" caches where entries are cheap to recreate
> 2. Soft references are better for performance caches where recreation is expensive but eventually possible
> 3. Phantom references are for advanced resource management beyond what finalizers can handle
> 4. Reference queues enable proactive management of the reference lifecycle
> 
> Ask them about real-world applications, such as implementing a three-level cache (memory, disk, remote) with appropriate reference types, or handling native resources with phantom references to prevent resource leaks. They should also understand the limitations—for instance, that the JVM makes no guarantees about exactly when soft references will be collected, making them unsuitable for critical resource management.
> 

## 10.3 Garbage Collection

Garbage collection is a key feature of the JVM that automates memory management, preventing memory leaks and making Java development safer and more productive.

### GC Fundamentals

```java
*// Demonstrating garbage collection basics*
public class GCDemo {
    public static void main(String[] args) {
        *// Enable GC logging// -Xlog:gc (Java 9+)// -XX:+PrintGC (Java 8)*
        
        System.out.println("Creating objects...");
        
        *// Create objects that will become garbage*
        for (int i = 0; i < 10; i++) {
            createGarbage(1000);
            
            *// Suggest GC run (no guarantee)*
            System.gc();
            
            *// Sleep to allow GC to happen*
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        System.out.println("Demo completed");
    }
    
    private static void createGarbage(int count) {
        for (int i = 0; i < count; i++) {
            *// Create objects that immediately become garbage*
            Object obj = new Object();
            
            *// Create linked objects to demonstrate reachability*
            LinkedList list = new LinkedList();
            for (int j = 0; j < 10; j++) {
                list.add(new byte[1024]); *// 1KB*
            }
        }
        System.out.println("Created " + count + " garbage objects");
    }
    
    *// Simple linked list for demonstration*
    static class LinkedList {
        private Node head;
        
        void add(Object data) {
            if (head == null) {
                head = new Node(data);
            } else {
                Node current = head;
                while (current.next != null) {
                    current = current.next;
                }
                current.next = new Node(data);
            }
        }
        
        static class Node {
            Object data;
            Node next;
            
            Node(Object data) {
                this.data = data;
            }
        }
    }
}
```

**Automatic Memory Management:**

- Eliminates explicit memory deallocation
- Prevents memory leaks (in most cases)
- Eliminates dangling pointer bugs
- Developer focuses on object creation, not destruction

**GC Roots:**

- Starting points for determining object reachability
- Include:
    - Local variables in active stack frames
    - Static fields of loaded classes
    - JNI references
    - Thread objects
    - Synchronization locks
    - JVM internal references

**Reachability Analysis:**

- Objects reachable from GC roots are "live"
- Unreachable objects are garbage
- Reachability is transitive (object A references B, B references C; if A is live, B and C are live)
- Cyclical references don't prevent collection if unreachable from GC roots

**Mark and Sweep:**

- Basic algorithm underlying most GC implementations
- **Mark**: Traverse object graph from roots, marking reachable objects
- **Sweep**: Collect unmarked (unreachable) objects
- Variations include mark-sweep-compact, mark-copy, etc.

**Generational Hypothesis:**

- Most objects die young
- Few objects live for a long time
- Basis for generational garbage collection
- Enables optimization of collection strategy

> **Deep Dive Tip:** The efficiency of garbage collection largely depends on the validity of the generational hypothesis for a given application. When this hypothesis holds (most objects are short-lived), generational GC is very efficient. However, applications that create many long-lived objects or have unpredictable lifetime patterns may experience performance issues. Understanding the memory usage patterns of your application is crucial for optimal GC tuning. Tools like JVisualVM or GC logs can help identify whether your application follows the expected patterns or needs special tuning to accommodate atypical object lifetime distributions.
> 

> **Interviewer Insight:** When discussing GC fundamentals, a strong candidate should understand both the benefits and limitations of automatic memory management. They should recognize that while GC eliminates many memory management bugs, it introduces non-deterministic pauses and overhead. Ask them about strategies for writing "GC-friendly" code, such as avoiding unnecessary object creation in tight loops, considering object pooling for expensive resources, and structuring data to minimize cross-generational references. They should also understand common GC-related performance issues, like allocation pressure (too many objects being created), memory leaks (objects remaining reachable unintentionally), and premature promotion (short-lived objects being promoted to old generation).
> 

### GC Algorithms

The JVM offers several garbage collection algorithms, each with different trade-offs between throughput, latency, and pause times.

```java
*// JVM flags for different collectors// Serial GC:// -XX:+UseSerialGC// Parallel GC:// -XX:+UseParallelGC// -XX:ParallelGCThreads=4// G1GC:// -XX:+UseG1GC// -XX:MaxGCPauseMillis=200// -XX:G1HeapRegionSize=4m// ZGC:// -XX:+UseZGC// -XX:ZCollectionInterval=5// Monitoring GC with jstat// jstat -gc <pid> 1000// jstat -gcutil <pid> 1000*
```

### Serial GC

- Single-threaded collection
- Stop-the-world pauses
- Simple and efficient for small heaps
- Good for client applications with limited resources
- Flag: `XX:+UseSerialGC`

### Parallel GC

- Multi-threaded collection
- Stop-the-world pauses but faster than Serial
- Focuses on throughput (total application performance)
- Default in many JVM versions
- Good for batch processing
- Flags: `XX:+UseParallelGC`, `XX:ParallelGCThreads=N`

### G1GC (Garbage First)

```java
*// G1GC monitoring example*
public class G1GCDemo {
    private static final int MB = 1024 * 1024;
    private static List<byte[]> memoryObjects = new ArrayList<>();
    
    public static void main(String[] args) throws InterruptedException {
        *// Set up G1GC with flags:// -XX:+UseG1GC// -XX:MaxGCPauseMillis=200// -Xlog:gc*=debug (Java 9+)// -XX:+PrintGCDetails -XX:+PrintGCTimeStamps (Java 8)*
        
        System.out.println("Starting G1GC demo...");
        
        *// Allocate memory in chunks, forcing collections*
        for (int i = 0; i < 10; i++) {
            System.out.println("Allocation round " + (i + 1));
            
            *// Allocate several 1MB arrays*
            for (int j = 0; j < 10; j++) {
                memoryObjects.add(new byte[MB]); *// 1MB*
            }
            
            *// Release some objects to demonstrate mixed collections*
            if (i % 3 == 0 && !memoryObjects.isEmpty()) {
                int removeCount = memoryObjects.size() / 3;
                for (int k = 0; k < removeCount; k++) {
                    memoryObjects.remove(0);
                }
                System.out.println("Released " + removeCount + " objects");
            }
            
            *// Sleep to allow GC to run and logs to be printed*
            Thread.sleep(500);
        }
        
        System.out.println("G1GC demo completed");
    }
}
```

**G1GC Key Features:**

- Region-based collector (heap divided into equal-sized regions)
- Predictable pause times (targets pause goals)
- Concurrent marking to minimize pauses
- Collects regions with most garbage first (hence the name)
- Good for large heaps (>4GB) with pause time requirements
- Default in Java 9+
- Flags: `XX:+UseG1GC`, `XX:MaxGCPauseMillis=N`

### ZGC (Java 11+)

- Scalable low-latency collector
- Concurrent processing (sub-millisecond pauses)
- Uses colored pointers and load barriers
- Good for applications requiring consistent response times
- Handles very large heaps (terabytes)
- Flag: `XX:+UseZGC`

### Shenandoah GC

- Ultra-low pause times
- Concurrent evacuation (moving objects while app runs)
- Uses Brooks pointers for concurrent updates
- Similar goals to ZGC with different implementation
- Flag: `XX:+UseShenandoahGC`

### Epsilon GC

- No-op collector (doesn't collect garbage)
- Allocates memory until heap is exhausted
- Useful for performance testing and short-lived applications
- Flag: `XX:+UseEpsilonGC`

> **Deep Dive Tip:** The choice of garbage collector significantly impacts application behavior, especially under load. G1GC, the default in modern Java, targets pause time goals but may sacrifice some throughput compared to Parallel GC. It works by dividing the heap into regions and prioritizing collection of regions with the most garbage. This approach allows it to collect only parts of the old generation in each cycle, reducing pause times. One key optimization in G1GC is the "Remembered Sets" that track references between regions, allowing it to collect regions independently without scanning the entire heap. Understanding these mechanisms can help in diagnosing and resolving GC-related performance issues.
> 

> **Interviewer Insight:** When discussing GC algorithms, a strong candidate should understand which collector is appropriate for different application types. They should know that:
> 
> 1. Throughput-oriented applications (batch processing, scientific computing) might prefer Parallel GC
> 2. Latency-sensitive applications (web services, trading systems) might prefer G1GC or ZGC
> 3. Applications with large heaps benefit from region-based collectors like G1GC
> 4. Applications with extreme latency requirements might need ZGC or Shenandoah
> 
> Ask them about how they would choose and tune a collector for specific scenarios, such as a high-frequency trading application with strict latency requirements, or a big data processing job that needs to maximize throughput. They should also understand the monitoring and tuning process, including interpreting GC logs and adjusting parameters based on observed behavior.
> 

### GC Tuning

Proper garbage collection tuning can significantly improve application performance and stability.

```java
*// Common GC tuning flags// Heap sizing// -Xms<size>  (initial heap size)// -Xmx<size>  (maximum heap size)// -XX:NewSize=<size>  (young generation size)// -XX:MaxNewSize=<size>  (maximum young generation size)// -XX:NewRatio=<n>  (ratio of old to young generation size)// -XX:SurvivorRatio=<n>  (ratio of eden to survivor space size)// G1GC specific// -XX:MaxGCPauseMillis=<n>  (target pause time)// -XX:G1HeapRegionSize=<size>  (region size, power of 2, 1-32MB)// -XX:InitiatingHeapOccupancyPercent=<n>  (start concurrent marking at n% occupancy)// ZGC specific// -XX:ZCollectionInterval=<seconds>  (time between GC cycles)// -XX:ZAllocationSpikeTolerance=<n>  (allocation spike tolerance)// GC logging// Java 9+:// -Xlog:gc*=info:file=gc.log:time,uptime,level,tags// Java 8:// -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:gc.log*
```

### Heap Sizing

```java
*// Example monitoring heap usage for tuning*
public class HeapMonitoring {
    public static void main(String[] args) {
        *// Set with:// -Xms256m -Xmx1g -XX:NewRatio=2*
        
        *// Monitor heap at various points*
        printHeapStats("At startup");
        
        *// Allocate memory to see how heap behaves*
        List<byte[]> allocations = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            allocations.add(new byte[10 * 1024 * 1024]); *// 10MB*
            printHeapStats("After allocation " + (i + 1));
        }
        
        *// Clear some allocations*
        for (int i = 0; i < 5; i++) {
            allocations.remove(0);
        }
        System.gc();
        printHeapStats("After freeing memory and GC");
    }
    
    private static void printHeapStats(String stage) {
        Runtime rt = Runtime.getRuntime();
        long totalMem = rt.totalMemory() / (1024 * 1024);
        long freeMem = rt.freeMemory() / (1024 * 1024);
        long usedMem = totalMem - freeMem;
        long maxMem = rt.maxMemory() / (1024 * 1024);
        
        System.out.printf("%s: Total=%dMB, Used=%dMB, Free=%dMB, Max=%dMB%n", 
                        stage, totalMem, usedMem, freeMem, maxMem);
    }
}
```

**Key Heap Sizing Principles:**

- Set `Xms` and `Xmx` to the same value for consistent performance
- Size based on available system memory and application needs
- Reserve memory for OS and other processes
- Consider generation sizing based on application allocation patterns
- Monitor actual usage to refine settings

### Generation Sizing

- Young generation size affects minor GC frequency and duration
- Old generation size affects full GC frequency and duration
- Survivor space ratio affects promotion rate
- Optimal settings depend on object lifetime distribution
- Flags: `XX:NewRatio`, `XX:SurvivorRatio`, `XX:TargetSurvivorRatio`

### Pause Time Goals

- G1GC allows specifying target pause times
- Lower targets reduce throughput but improve responsiveness
- Realistic goals depend on heap size and hardware
- Flag: `XX:MaxGCPauseMillis=<n>`

### GC Logging

```java
*// Java 9+ GC logging configuration// -Xlog:gc*=info:file=gc.log:time,uptime,level,tags// Example GC log entry (G1GC)// [2.218s][info][gc] GC(2) Pause Young (Normal) (G1 Evacuation Pause) 242M->27M(512M) 8.238ms// Java 8 GC logging configuration// -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:gc.log// Example GC log entry (Parallel GC, Java 8)// 2023-06-15T10:15:30.123+0000: 2.218: [GC (Allocation Failure) [PSYoungGen: 242M->27M(256M)] 242M->27M(512M), 0.0082380 secs]*
```

**GC Log Analysis:**

- Monitor collection frequency and duration
- Look for patterns of increasing pause times
- Check promotion rates and old generation utilization
- Identify memory leaks through growing old generation
- Tools: GCViewer, GCEasy, jClarity, IBM GCMV

### GC Overhead Limits

- JVM has safeguards against excessive GC
- Throws OutOfMemoryError if spending too much time in GC
- Can be tuned or disabled
- Flags: `XX:GCTimeLimit=<n>`, `XX:GCHeapFreeLimit=<n>`

> **Deep Dive Tip:** One of the most effective GC tuning approaches is to right-size the young generation based on your application's allocation rate and object lifetime distribution. If most objects die young, a larger young generation reduces minor GC frequency and promotion rate. However, this comes at the cost of potentially longer minor GC pauses. For applications with many medium-lived objects, consider tuning survivor spaces to allow objects to age properly before promotion, possibly by adjusting `-XX:SurvivorRatio` and `-XX:TargetSurvivorRatio`. The goal is to ensure objects that will die soon don't get promoted to the old generation, where their collection becomes more expensive.
> 

> **Interviewer Insight:** When discussing GC tuning, a strong candidate should emphasize the importance of a methodical, data-driven approach rather than arbitrary parameter adjustments. They should outline a tuning process like:
> 
> 1. Establish performance goals (throughput, latency, footprint)
> 2. Choose an appropriate collector based on these goals
> 3. Enable detailed GC logging in a test environment
> 4. Analyze logs to identify specific issues (e.g., frequent full GCs, long pause times)
> 5. Make targeted adjustments based on observed behavior
> 6. Measure the impact of changes
> 7. Iterate until goals are met
> 
> Ask them about common pitfalls in GC tuning, such as over-tuning (making too many changes at once), focusing on the wrong metrics, or using settings from one application for a different application without considering the specific characteristics of each. They should also understand the trade-offs involved, such as throughput vs. latency or memory usage vs. GC frequency.
> 

### Memory Profiling

Identifying and resolving memory issues requires effective profiling techniques and tools.

```java
*// JVM flags for memory profiling// -XX:+HeapDumpOnOutOfMemoryError// -XX:HeapDumpPath=<path>// -XX:+PrintClassHistogram// -XX:+PrintGCDetails// -XX:+PrintGCDateStamps// -XX:+PrintTenuringDistribution// Command-line tools// jmap -dump:format=b,file=heap.hprof <pid>// jmap -histo <pid>// jcmd <pid> GC.heap_dump <file-path>// jstat -gcutil <pid> 1000*
```

### Heap Dumps

```java
*// Programmatic heap dump generation*
import com.sun.management.HotSpotDiagnosticMXBean;

public class HeapDumpGenerator {
    public static void dumpHeap(String filePath) {
        try {
            MBeanServer server = ManagementFactory.getPlatformMBeanServer();
            HotSpotDiagnosticMXBean bean = ManagementFactory.newPlatformMXBeanProxy(
                server, "com.sun.management:type=HotSpotDiagnostic", HotSpotDiagnosticMXBean.class);
            bean.dumpHeap(filePath, true);
            System.out.println("Heap dump written to " + filePath);
        } catch (Exception e) {
            System.err.println("Error creating heap dump: " + e);
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) {
        *// Create some objects for the heap dump*
        List<Object> objects = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            objects.add(new String("Object " + i));
        }
        
        *// Generate heap dump*
        dumpHeap("heap_dump.hprof");
        
        *// You can analyze the dump with tools like:// - Eclipse Memory Analyzer (MAT)// - VisualVM// - YourKit// - JProfiler*
    }
}
```

**Heap Dump Analysis:**

- Capture point-in-time snapshot of heap
- Analyze with specialized tools (MAT, VisualVM, YourKit)
- Look for memory leaks, excessive object retention
- Identify dominators (objects keeping large graphs alive)
- Analyze object histograms and reference chains

### GC Logs Analysis

```java
*// Analyzing GC logs to identify issues// Example log entries:// Minor GC (G1GC)// [2023-06-15T10:15:30.123+0000: 2.218s][info][gc] GC(2) Pause Young (Normal) (G1 Evacuation Pause) 242M->27M(512M) 8.238ms// What to look for:// - Frequency of collections// - Duration of pauses// - Memory reclaimed vs. total// - Trends over time// - Promotion rates// - Full GC occurrences*
```

**Key GC Log Metrics:**

- Collection frequency and triggers
- Pause times and trends
- Memory reclaimed per collection
- Promotion rates
- Old generation occupancy
- Concurrent cycle details (G1GC, ZGC)

### Memory Leak Detection

```java
*// Common memory leak patterns*
public class MemoryLeakExamples {
    *// 1. Static fields holding references*
    private static final List<Object> staticList = new ArrayList<>();
    
    *// 2. Unclosed resources*
    public void resourceLeak() throws IOException {
        InputStream is = new FileInputStream("file.txt");
        byte[] buffer = new byte[1024];
        is.read(buffer);
        *// Missing is.close() - resource leak*
    }
    
    *// 3. Improper equals/hashCode implementation*
    static class LeakyKey {
        private int id;
        private byte[] data = new byte[1024]; *// 1KB*
        
        public LeakyKey(int id) {
            this.id = id;
        }
        
        *// Bad equals - uses only id*
        @Override
        public boolean equals(Object obj) {
            if (obj instanceof LeakyKey) {
                return ((LeakyKey) obj).id == this.id;
            }
            return false;
        }
        
        *// Doesn't override hashCode - violates equals/hashCode contract// This can cause HashMap to retain duplicate objects*
    }
    
    *// 4. Inner class references*
    public Object createLeakyObject() {
        *// Non-static inner class holds implicit reference to outer instance*
        class InnerClass {
            private byte[] data = new byte[1024 * 1024]; *// 1MB*
        }
        
        return new InnerClass();
    }
    
    *// 5. Thread-local leaks*
    private static ThreadLocal<byte[]> threadLocal = 
        new ThreadLocal<byte[]>() {
            @Override
            protected byte[] initialValue() {
                return new byte[1024 * 1024]; *// 1MB*
            }
        };
    
    *// In thread pools, ThreadLocal values can be leaked// if not removed when thread is returned to pool*
    public void threadLocalLeak() {
        *// Use thread-local*
        byte[] data = threadLocal.get();
        
        *// Missing threadLocal.remove() - can cause leak in thread pools*
    }
}
```

**Common Memory Leak Patterns:**

- Static collections growing unbounded
- Long-lived objects holding references to short-lived objects
- Unclosed resources (streams, connections)
- Improper caching without eviction
- Thread-local variables in thread pools
- Classloader leaks
- Inappropriate use of finalizers and cleaners

### Object Allocation Profiling

- Track where objects are allocated
- Identify "allocation hotspots"
- Focus optimization on high-allocation areas
- Tools: Java Flight Recorder, YourKit, JProfiler

### Live Object Analysis

- Examine currently live objects
- Determine object age distribution
- Identify unexpectedly retained objects
- Analyze reference chains keeping objects alive
- Tools: jmap -histo, Eclipse MAT, VisualVM

> **Deep Dive Tip:** When analyzing heap dumps, focus on "dominators"—objects that keep large portions of the heap alive through their references. The dominator tree shows the objects that, if removed, would free the most memory. Another powerful technique is examining "retained sets"—all objects that would be collected if a particular object were removed. This helps identify unexpected retention patterns where seemingly small objects are keeping large object graphs alive. For GC log analysis, pay particular attention to the ratio of time spent in different GC phases and the effectiveness of each collection (how much memory was reclaimed relative to pause time).
> 

> **Interviewer Insight:** When discussing memory profiling, a strong candidate should demonstrate a systematic approach to identifying and resolving memory issues. They should be able to differentiate between different types of memory problems:
> 
> 1. Memory leaks (growing memory usage that never stabilizes)
> 2. High allocation rates (causing frequent GC)
> 3. Inefficient memory usage (using more memory than necessary)
> 4. Excessive promotion (short-lived objects living longer than intended)
> 
> Ask them about their experience with memory profiling tools and techniques they've used to resolve specific memory issues. A good candidate will emphasize the importance of understanding normal application behavior before jumping to conclusions about memory problems, and will have a toolkit of techniques for different scenarios—from quick analysis with jmap to in-depth heap dump analysis with specialized tools. They should also understand the trade-offs involved in fixing memory issues, such as the potential performance impact of reducing memory usage through more complex data structures.
> 

## 10.4 Performance Optimization

Optimizing JVM performance involves understanding and tuning various aspects of the runtime environment, from JVM flags to application-specific optimizations.

### JVM Tuning Flags

```java
*// Key JVM tuning categories// 1. Memory management// -Xms<size>  (initial heap size)// -Xmx<size>  (maximum heap size)// -XX:MaxMetaspaceSize=<size>  (metaspace limit)// -XX:MaxDirectMemorySize=<size>  (direct memory limit)// 2. Garbage collection// -XX:+Use<GC>  (select GC algorithm)// -XX:NewRatio=<n>  (ratio of old to young generation)// -XX:SurvivorRatio=<n>  (ratio of eden to survivor space)// -XX:MaxGCPauseMillis=<n>  (target pause time for G1GC)// 3. JIT compilation// -XX:CompileThreshold=<n>  (invocations before compilation)// -XX:+TieredCompilation  (enable tiered compilation)// -XX:CICompilerCount=<n>  (number of compiler threads)// -XX:+PrintCompilation  (log compilation activity)// 4. Thread management// -XX:ThreadStackSize=<size>  (thread stack size)// -XX:+UseBiasedLocking  (enable biased locking)// -XX:BiasedLockingStartupDelay=<n>  (delay biased locking at startup)// 5. Diagnostic/monitoring// -XX:+HeapDumpOnOutOfMemoryError  (create heap dump on OOM)// -XX:+PrintGCDetails  (detailed GC logging)// -XX:+FlightRecorder  (enable JFR for profiling)*
```

### Heap Configuration

```java
*// Example of heap configuration*
public class HeapConfigDemo {
    private static final int MB = 1024 * 1024;
    
    public static void main(String[] args) {
        *// Run with:// -Xms512m -Xmx1g -XX:+PrintGCDetails*
        
        *// Print heap configuration*
        printHeapConfig();
        
        *// Allocate memory to demonstrate heap behavior*
        List<byte[]> allocations = new ArrayList<>();
        try {
            for (int i = 0; i < 2000; i++) {
                allocations.add(new byte[MB]); *// 1MB*
                if (i % 100 == 0) {
                    System.out.println("Allocated " + i + " MB");
                    printHeapConfig();
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("Out of memory: " + e.getMessage());
        }
    }
    
    private static void printHeapConfig() {
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory() / MB;
        long totalMemory = runtime.totalMemory() / MB;
        long freeMemory = runtime.freeMemory() / MB;
        long usedMemory = totalMemory - freeMemory;
        
        System.out.println("\nHeap Configuration:");
        System.out.println("Max Memory (Xmx): " + maxMemory + " MB");
        System.out.println("Current Allocated (totalMemory): " + totalMemory + " MB");
        System.out.println("Currently Used: " + usedMemory + " MB");
        System.out.println("Currently Free: " + freeMemory + " MB");
    }
}
```

**Key Heap Settings:**

- `Xms` and `Xmx`: Initial and maximum heap size
- `XX:NewRatio`: Ratio of old to young generation
- `XX:MaxMetaspaceSize`: Limit for class metadata
- `XX:MaxDirectMemorySize`: Limit for direct buffers

### GC Selection

```java
*// Example showing different GC behaviors*
public class GCSelectionDemo {
    public static void main(String[] args) throws Exception {
        *// Run with different collectors:// -XX:+UseSerialGC// -XX:+UseParallelGC// -XX:+UseG1GC// -XX:+UseZGC*
        
        *// Enable GC logging// Java 9+: -Xlog:gc*// Java 8: -XX:+PrintGCDetails*
        
        System.out.println("Starting GC test...");
        List<Object> objects = new ArrayList<>();
        
        *// Create a sawtooth memory pattern to trigger GC*
        for (int iteration = 0; iteration < 5; iteration++) {
            System.out.println("\nIteration " + (iteration + 1));
            
            *// Allocate objects*
            for (int i = 0; i < 100; i++) {
                objects.add(new byte[1024 * 1024]); *// 1MB*
                Thread.sleep(10); *// Small delay to spread allocations*
            }
            
            *// Clear some objects to allow collection*
            for (int i = 0; i < 80; i++) {
                objects.remove(0);
            }
            
            *// Force GC to see collector behavior*
            System.gc();
            Thread.sleep(500); *// Give GC time to run*
        }
        
        System.out.println("Test completed");
    }
}
```

**GC Selection Criteria:**

- **Throughput**: Parallel GC for batch processing
- **Latency**: G1GC, ZGC for responsive applications
- **Memory Footprint**: Adjust heap size and generation ratios
- **Application Type**: Web services, batch, desktop, etc.

### JIT Compilation

```java
*// Example demonstrating JIT compilation behavior*
public class JITDemo {
    public static void main(String[] args) {
        *// Run with:// -XX:+PrintCompilation// -XX:CompileThreshold=1000*
        
        System.out.println("Starting JIT compilation demo...");
        
        *// Warm up - method will be compiled after threshold*
        for (int i = 0; i < 10000; i++) {
            computeIntensive(i);
        }
        
        *// Measure performance before and after compilation*
        long start = System.nanoTime();
        long sum = 0;
        for (int i = 0; i < 1000; i++) {
            sum += computeIntensive(i);
        }
        long end = System.nanoTime();
        
        System.out.println("Sum: " + sum);
        System.out.println("Time: " + (end - start) / 1_000_000.0 + " ms");
    }
    
    private static long computeIntensive(int value) {
        long result = 0;
        for (int i = 0; i < 1000; i++) {
            result += (value * i) % 256;
        }
        return result;
    }
}
```

**JIT Configuration:**

- `XX:+TieredCompilation`: Enable/disable tiered compilation
- `XX:CompileThreshold`: Invocations before compilation
- `XX:MaxInlineSize`: Maximum bytecode size for inlining
- `XX:FreqInlineSize`: Size limit for frequently executed methods

### Thread Stack Size

```java
*// Example demonstrating thread stack size impact*
public class ThreadStackDemo {
    private static int recursionDepth = 0;
    
    public static void main(String[] args) {
        *// Run with different stack sizes:// Default: -XX:ThreadStackSize=1024 (1MB)// Smaller: -XX:ThreadStackSize=256 (256KB)// Larger: -XX:ThreadStackSize=2048 (2MB)*
        
        System.out.println("Starting thread stack size demo...");
        
        try {
            *// Call recursive method to fill stack*
            recursiveMethod();
        } catch (StackOverflowError e) {
            System.out.println("Stack overflow at depth: " + recursionDepth);
        }
    }
    
    private static void recursiveMethod() {
        *// Track recursion depth*
        recursionDepth++;
        
        *// Local variables consume stack space*
        byte[] localArray = new byte[1024]; *// 1KB local array*
        double localDouble1 = Math.random();
        double localDouble2 = Math.random();
        
        *// Every 1000 calls, print progress*
        if (recursionDepth % 1000 == 0) {
            System.out.println("Recursion depth: " + recursionDepth);
        }
        
        *// Recurse deeper*
        recursiveMethod();
    }
}
```

**Thread Stack Settings:**

- `XX:ThreadStackSize`: Thread stack size
- `Xss`: Alternative syntax for thread stack size
- Size affects maximum thread count and recursion depth
- Default varies by JVM version and platform

> **Deep Dive Tip:** JVM flags can interact in complex ways, and optimal settings depend on both application characteristics and hardware. For example, the ideal heap size and GC settings for a server with 4 cores and 16GB RAM will differ significantly from those for a server with 32 cores and 128GB RAM. Additionally, some flags have different default values or behaviors across JVM versions and platforms. When tuning production systems, it's important to baseline performance before changes, make adjustments incrementally, and validate each change with realistic load testing. Tools like JMH (Java Microbenchmark Harness) can help evaluate the impact of different JVM settings on specific performance aspects.
> 

> **Interviewer Insight:** When discussing JVM tuning flags, a strong candidate should emphasize that effective tuning requires understanding the specific characteristics and requirements of the application. They should be skeptical of "magic" configurations that claim to work for all applications. Ask them about their approach to JVM tuning, looking for a methodical process like:
> 
> 1. Establishing a baseline with default settings
> 2. Identifying specific performance issues or goals
> 3. Making targeted adjustments based on application behavior
> 4. Measuring the impact of each change
> 5. Documenting and monitoring the configuration over time
> 
> They should also understand the risks of over-tuning, such as brittle configurations that break with application changes or JVM updates, and the importance of working within operational constraints like available memory and deployment processes.
> 

### Profiling Tools

Profiling tools help identify performance bottlenecks and optimization opportunities in Java applications.

```java
*// Common Java profiling tools:// - JVisualVM (bundled with JDK through Java 8)// - Java Mission Control (JMC) and Flight Recorder (JFR)// - YourKit Java Profiler (commercial)// - JProfiler (commercial)// - Async-profiler (open source)// - Eclipse Memory Analyzer Tool (MAT) for heap analysis*
```

### JProfiler

```java
*// Example of code to profile with JProfiler*
public class ProfilerDemo {
    public static void main(String[] args) throws Exception {
        *// To profile with JProfiler:// 1. Install JProfiler// 2. Add JProfiler agent to JVM://    -agentpath:/path/to/libjprofilerti.so=port=8849// 3. Connect JProfiler UI to the agent*
        
        System.out.println("Starting profiler demo...");
        
        *// CPU-intensive operation*
        System.out.println("Running CPU-intensive task...");
        long cpuResult = cpuIntensiveTask();
        System.out.println("CPU task result: " + cpuResult);
        
        *// Memory-intensive operation*
        System.out.println("Running memory-intensive task...");
        memoryIntensiveTask();
        
        *// I/O-intensive operation*
        System.out.println("Running I/O-intensive task...");
        ioIntensiveTask();
        
        System.out.println("Demo completed");
    }
    
    private static long cpuIntensiveTask() {
        long result = 0;
        for (int i = 0; i < 10_000_000; i++) {
            result += (i * i) % 1000;
        }
        return result;
    }
    
    private static void memoryIntensiveTask() {
        List<byte[]> arrays = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            arrays.add(new byte[1024 * 1024]); *// 1MB*
            
            *// Remove some to avoid OOM*
            if (i % 100 == 0) {
                for (int j = 0; j < 50; j++) {
                    if (!arrays.isEmpty()) {
                        arrays.remove(0);
                    }
                }
            }
        }
    }
    
    private static void ioIntensiveTask() throws Exception {
        for (int i = 0; i < 10; i++) {
            *// Create a temporary file*
            File tempFile = File.createTempFile("profiler-demo", ".tmp");
            tempFile.deleteOnExit();
            
            *// Write data*
            try (FileOutputStream fos = new FileOutputStream(tempFile)) {
                byte[] data = new byte[1024 * 1024]; *// 1MB*
                new Random().nextBytes(data);
                fos.write(data);
            }
            
            *// Read data*
            try (FileInputStream fis = new FileInputStream(tempFile)) {
                byte[] buffer = new byte[8192];
                while (fis.read(buffer) != -1) {
                    *// Process data*
                }
            }
        }
    }
}
```

**JProfiler Features:**

- CPU profiling (sampling and instrumentation)
- Memory profiling and leak detection
- Thread analysis and deadlock detection
- I/O profiling
- Database and JPA monitoring
- JVM telemetry

### Java Mission Control and Flight Recorder

```java
*// Using Java Flight Recorder (JFR)*
public class JFRDemo {
    public static void main(String[] args) throws Exception {
        *// Enable JFR recording with:// Java 11+: -XX:StartFlightRecording=duration=60s,filename=recording.jfr// Java 8: -XX:+UnlockCommercialFeatures -XX:+FlightRecorder //         -XX:StartFlightRecording=duration=60s,filename=recording.jfr*
        
        *// Or programmatically:*
        if (FlightRecorder.isAvailable()) {
            System.out.println("Starting JFR recording...");
            
            *// Configure recording*
            Map<String, String> settings = new HashMap<>();
            settings.put("name", "Profiling Demo");
            settings.put("settings.profile", "profile");
            
            *// Start recording*
            Recording recording = new Recording(Configuration.getConfiguration("profile"));
            recording.start();
            
            *// Run tasks to profile*
            System.out.println("Running tasks to profile...");
            runTasks();
            
            *// Stop and save recording*
            recording.stop();
            recording.dump(Paths.get("jfr-demo-recording.jfr"));
            System.out.println("Recording saved to jfr-demo-recording.jfr");
            
            *// Can be analyzed with Java Mission Control*
        } else {
            System.out.println("Flight Recorder not available");
        }
    }
    
    private static void runTasks() throws Exception {
        *// Similar tasks as in the JProfiler example// CPU, memory, and I/O intensive operations// ...*
    }
}
```

**JFR/JMC Features:**

- Low-overhead continuous monitoring
- Detailed event recording
- CPU profiling and hot methods
- Memory allocation tracking
- GC analysis
- Thread contention and locks
- I/O and network activity

### VisualVM

```java
*// No special setup needed for basic VisualVM profiling// Start VisualVM and connect to running application// jvisualvm or visualvm command// Optionally, install plugins for enhanced functionality// For more detailed profiling, use VisualVM startup options:// -Dcom.sun.management.jmxremote// -Dcom.sun.management.jmxremote.port=9010// -Dcom.sun.management.jmxremote.authenticate=false// -Dcom.sun.management.jmxremote.ssl=false*
```

**VisualVM Features:**

- CPU and memory profiling
- Thread monitoring
- Heap dumps and analysis
- Visual GC monitoring
- MBean browser
- Plugin-based extension

### Async-profiler

```java
*// Using async-profiler// 1. Download async-profiler from https://github.com/jvm-profiling-tools/async-profiler// 2. Attach to running process://    ./profiler.sh -d 30 -f profile.html <pid>// 3. Or start with agent://    java -agentpath:/path/to/libasyncProfiler.so=start,file=profile.html ...// Key features:// - Low overhead CPU profiling// - Allocation profiling// - Wall-clock profiling// - Native code support// - Flamegraph output*
```

**Deep Dive Tip:** Profiling tools use different techniques with various trade-offs. Sampling profilers periodically check what code is executing, providing lower overhead but potentially missing short-lived methods. Instrumentation profilers modify bytecode to add measurement code, capturing every method call but with higher overhead. For production environments, consider async-profiler or JFR, which use efficient sampling techniques with minimal performance impact. For development environments, more intrusive tools like JProfiler or YourKit can provide more detailed insights. Understanding these differences helps choose the right tool for each situation.

**Interviewer Insight:** When discussing profiling tools, a strong candidate should demonstrate familiarity with multiple tools and their appropriate use cases. They should understand the trade-offs between different profiling techniques and the impact of profiling on application performance. Ask them about their approach to performance investigation, looking for a methodical process that starts with high-level monitoring to identify general issues, then progressively focuses on specific areas with appropriate profiling techniques. They should also understand how to interpret profiling results correctly, avoiding common pitfalls like focusing solely on "hot" methods without considering their context or misinterpreting allocation statistics.

### JMH (Java Microbenchmark Harness)

JMH is a specialized tool for accurately measuring code performance at the method level, accounting for JVM optimizations and warm-up effects.

```java
*// JMH benchmark example// Add JMH dependencies to your project:// org.openjdk.jmh:jmh-core:1.36// org.openjdk.jmh:jmh-generator-annprocess:1.36*

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
@State(Scope.Thread)
public class ListBenchmark {
    private static final int SIZE = 1000;
    
    private List<Integer> arrayList;
    private List<Integer> linkedList;
    
    @Setup
    public void setup() {
        arrayList = new ArrayList<>();
        linkedList = new LinkedList<>();
        
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }
    }
    
    @Benchmark
    public int arrayListGet() {
        int sum = 0;
        for (int i = 0; i < SIZE; i++) {
            sum += arrayList.get(i);
        }
        return sum;
    }
    
    @Benchmark
    public int linkedListGet() {
        int sum = 0;
        for (int i = 0; i < SIZE; i++) {
            sum += linkedList.get(i);
        }
        return sum;
    }
    
    @Benchmark
    public int arrayListIteration() {
        int sum = 0;
        for (int value : arrayList) {
            sum += value;
        }
        return sum;
    }
    
    @Benchmark
    public int linkedListIteration() {
        int sum = 0;
        for (int value : linkedList) {
            sum += value;
        }
        return sum;
    }
    
    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
            .include(ListBenchmark.class.getSimpleName())
            .build();
        
        new Runner(opt).run();
    }
}
```

**Key JMH Annotations:**

- **@Benchmark**: Marks methods to benchmark
- **@BenchmarkMode**: Measurement mode (throughput, average time, etc.)
- **@OutputTimeUnit**: Time unit for results
- **@Warmup**: JVM warm-up configuration
- **@Measurement**: Benchmark execution configuration
- **@Fork**: Number of JVM forks for testing
- **@State**: Scope of state objects (thread, group, benchmark)
- **@Setup/@TearDown**: Initialization and cleanup

**Common JMH Pitfalls:**

- **Dead Code Elimination**: JVM might optimize away code that doesn't produce used results
- **Constant Folding**: Compile-time evaluation of constant expressions
- **Loop Unrolling**: Compiler optimization that changes loop structure
- **Inlining**: Method calls replaced with method body
- **Unrealistic Inputs**: Using non-representative test data

**Avoiding Pitfalls:**

- Return results from benchmarks to prevent dead code elimination
- Use @State objects for mutable state
- Include Blackhole.consume() for unused values
- Use realistic, randomized inputs
- Properly configure warm-up to ensure JIT compilation

> **Deep Dive Tip:** JMH uses a sophisticated approach to prevent common benchmarking pitfalls. For example, it runs benchmarks in separate JVM instances (forks) to avoid cross-benchmark interference, carefully manages state objects to prevent unrealistic caching effects, and provides mechanisms like Blackhole.consume() to prevent dead code elimination without introducing artificial dependencies. Understanding these mechanisms is crucial for writing accurate benchmarks. For the most reliable results, also consider environment factors like CPU power management, system load, and memory layout (cache alignment), which can significantly impact performance measurements.
> 

> **Interviewer Insight:** When discussing JMH, a strong candidate should emphasize that microbenchmarking is both valuable and dangerous—it provides precise measurements of isolated code paths but can lead to misleading conclusions if not done carefully. They should understand the challenges of benchmarking in a JIT-compiled, garbage-collected environment and how JMH addresses these challenges. Ask them about their experience with JMH or similar tools, looking for awareness of common pitfalls and a healthy skepticism about benchmark results. They should view microbenchmarking as one tool in a broader performance optimization approach, recognizing that real-world performance depends on system interactions that might not be captured in isolated benchmarks.
> 

### Performance Best Practices

Applying proven performance best practices can significantly improve Java application performance.

### Object Pooling

```java
*// Example of object pooling for expensive objects*
public class ObjectPoolExample {
    *// Simple object pool implementation*
    static class ExpensiveObjectPool {
        private final int maxPoolSize;
        private final Queue<ExpensiveObject> pool;
        
        public ExpensiveObjectPool(int maxPoolSize) {
            this.maxPoolSize = maxPoolSize;
            this.pool = new ConcurrentLinkedQueue<>();
            
            *// Pre-populate pool*
            for (int i = 0; i < maxPoolSize / 2; i++) {
                pool.offer(new ExpensiveObject());
            }
        }
        
        public ExpensiveObject borrow() {
            ExpensiveObject object = pool.poll();
            if (object == null) {
                *// Pool is empty, create new object*
                object = new ExpensiveObject();
            }
            return object;
        }
        
        public void returnToPool(ExpensiveObject object) {
            *// Reset object state if necessary*
            object.reset();
            
            *// Only return to pool if not full*
            if (pool.size() < maxPoolSize) {
                pool.offer(object);
            }
        }
    }
    
    *// Expensive object to pool*
    static class ExpensiveObject {
        private final byte[] data;
        
        public ExpensiveObject() {
            *// Simulate expensive initialization*
            data = new byte[1024 * 1024]; *// 1MB*
            for (int i = 0; i < data.length; i++) {
                data[i] = (byte) i;
            }
            System.out.println("Created expensive object");
        }
        
        public void doWork() {
            *// Simulate work with the object*
            int sum = 0;
            for (byte b : data) {
                sum += b;
            }
            System.out.println("Work completed, checksum: " + sum);
        }
        
        public void reset() {
            *// Reset state for reuse// In this example, no reset needed*
        }
    }
    
    public static void main(String[] args) {
        *// Create pool*
        ExpensiveObjectPool pool = new ExpensiveObjectPool(10);
        
        *// Simulate multiple threads using pooled objects*
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                ExpensiveObject obj = pool.borrow();
                try {
                    obj.doWork();
                    Thread.sleep(100); *// Simulate using the object*
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    pool.returnToPool(obj);
                }
            }).start();
        }
    }
}
```

**Object Pooling Use Cases:**

- Connection pools (database, HTTP)
- Thread pools
- Buffer pools
- Objects with expensive initialization
- Frequently created/destroyed objects

**Object Pooling Trade-offs:**

- Reduces garbage collection pressure
- Improves performance for expensive object creation
- Increases code complexity
- May cause memory leaks if not managed properly
- Not beneficial for lightweight objects

### String Deduplication

```java
*// String deduplication (JVM feature)// Enable with:// -XX:+UseStringDeduplication (G1GC only)// Example demonstrating string duplication issue*
public class StringDeduplicationDemo {
    public static void main(String[] args) {
        *// Create many duplicate strings*
        List<String> strings = new ArrayList<>(1_000_000);
        
        for (int i = 0; i < 1_000_000; i++) {
            *// These strings have the same content but are different objects*
            strings.add(new String("Hello, World!"));
        }
        
        *// Without deduplication, these consume ~24MB (24 bytes per String instance)// With deduplication enabled, the JVM will eventually deduplicate them*
        
        *// Force GC to trigger deduplication*
        System.gc();
        
        *// Wait to allow deduplication to complete*
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        *// Check memory usage*
        Runtime rt = Runtime.getRuntime();
        long usedMemory = rt.totalMemory() - rt.freeMemory();
        System.out.println("Used memory: " + (usedMemory / 1024 / 1024) + " MB");
        
        *// Keep strings alive*
        System.out.println("String sample: " + strings.get(500_000));
    }
}
```

**String Deduplication:**

- Available with G1GC
- Deduplicates strings with identical content
- Runs as a background process during GC
- Most effective for applications with many duplicate strings
- Enabled with `XX:+UseStringDeduplication`

### Lazy Initialization

```java
*// Lazy initialization strategies*
public class LazyInitializationExample {
    *// Strategy 1: Basic lazy initialization*
    private byte[] largeArray;
    
    public byte[] getLargeArray() {
        if (largeArray == null) {
            largeArray = new byte[10 * 1024 * 1024]; *// 10MB*
            initializeArray(largeArray);
        }
        return largeArray;
    }
    
    *// Strategy 2: Thread-safe lazy initialization with double-checked locking*
    private volatile byte[] threadSafeArray;
    
    public byte[] getThreadSafeArray() {
        if (threadSafeArray == null) {
            synchronized (this) {
                if (threadSafeArray == null) {
                    threadSafeArray = new byte[10 * 1024 * 1024]; *// 10MB*
                    initializeArray(threadSafeArray);
                }
            }
        }
        return threadSafeArray;
    }
    
    *// Strategy 3: Holder class idiom for static fields*
    public static class LargeDataHolder {
        *// This class is loaded only when getInstance() is called*
        private static final byte[] INSTANCE = createInstance();
        
        private static byte[] createInstance() {
            byte[] instance = new byte[10 * 1024 * 1024]; *// 10MB// Initialize instance*
            for (int i = 0; i < instance.length; i++) {
                instance[i] = (byte) (i % 256);
            }
            return instance;
        }
    }
    
    public static byte[] getStaticLargeArray() {
        return LargeDataHolder.INSTANCE;
    }
    
    *// Helper method*
    private void initializeArray(byte[] array) {
        for (int i = 0; i < array.length; i++) {
            array[i] = (byte) (i % 256);
        }
    }
    
    public static void main(String[] args) {
        LazyInitializationExample example = new LazyInitializationExample();
        
        *// First access initializes*
        long start = System.currentTimeMillis();
        byte[] array1 = example.getLargeArray();
        System.out.println("First access time: " + (System.currentTimeMillis() - start) + " ms");
        
        *// Second access returns cached instance*
        start = System.currentTimeMillis();
        byte[] array2 = example.getLargeArray();
        System.out.println("Second access time: " + (System.currentTimeMillis() - start) + " ms");
        
        *// Test thread-safe version*
        start = System.currentTimeMillis();
        byte[] array3 = example.getThreadSafeArray();
        System.out.println("Thread-safe first access time: " + (System.currentTimeMillis() - start) + " ms");
        
        *// Test static holder pattern*
        start = System.currentTimeMillis();
        byte[] array4 = getStaticLargeArray();
        System.out.println("Static holder first access time: " + (System.currentTimeMillis() - start) + " ms");
    }
}
```

**Lazy Initialization Strategies:**

- Defer initialization until first use
- Reduces startup time and memory usage
- Different techniques for thread safety
- Double-checked locking for instance fields
- Holder class idiom for static fields

### Caching Strategies

```java
*// Different caching strategies*
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.WeakHashMap;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

public class CachingStrategies {
    *// Strategy 1: Simple in-memory cache with ConcurrentHashMap*
    static class SimpleCache<K, V> {
        private final ConcurrentHashMap<K, V> cache = new ConcurrentHashMap<>();
        
        public V get(K key, java.util.function.Function<K, V> computeFunction) {
            *// computeIfAbsent atomically computes and adds value if key not present*
            return cache.computeIfAbsent(key, computeFunction);
        }
        
        public void put(K key, V value) {
            cache.put(key, value);
        }
        
        public void invalidate(K key) {
            cache.remove(key);
        }
        
        public void clear() {
            cache.clear();
        }
        
        public int size() {
            return cache.size();
        }
    }
    
    *// Strategy 2: LRU Cache using LinkedHashMap*
    static class LRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int maxSize;
        
        public LRUCache(int maxSize) {
            *// Initial capacity, load factor, access order (true for LRU behavior)*
            super(16, 0.75f, true);
            this.maxSize = maxSize;
        }
        
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > maxSize;
        }
    }
    
    *// Strategy 3: Time-based expiration cache*
    static class ExpiringCache<K, V> {
        private final ConcurrentHashMap<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
        private final long expirationMs;
        
        public ExpiringCache(long duration, TimeUnit unit) {
            this.expirationMs = unit.toMillis(duration);
        }
        
        public V get(K key) {
            CacheEntry<V> entry = cache.get(key);
            if (entry == null) {
                return null;
            }
            
            *// Check if entry is expired*
            if (System.currentTimeMillis() - entry.timestamp > expirationMs) {
                cache.remove(key);
                return null;
            }
            
            return entry.value;
        }
        
        public void put(K key, V value) {
            cache.put(key, new CacheEntry<>(value, System.currentTimeMillis()));
        }
        
        public void invalidate(K key) {
            cache.remove(key);
        }
        
        public void clear() {
            cache.clear();
        }
        
        *// Helper class to track entry timestamps*
        private static class CacheEntry<V> {
            final V value;
            final long timestamp;
            
        CacheEntry(V value, long timestamp) {
            this.value = value;
            this.timestamp = timestamp;
        }
    }
}

// Strategy 4: Weak reference cache
static class WeakCache<K, V> {
    private final WeakHashMap<K, V> cache = new WeakHashMap<>();
    
    public synchronized V get(K key) {
        return cache.get(key);
    }
    
    public synchronized void put(K key, V value) {
        cache.put(key, value);
    }
    
    public synchronized int size() {
        return cache.size();
    }
}

// Usage example
public static void main(String[] args) throws InterruptedException {
    // Test SimpleCache
    SimpleCache<String, byte[]> simpleCache = new SimpleCache<>();
    System.out.println("Simple cache test:");
    testCache(key -> {
        System.out.println("Computing value for " + key);
        return new byte[1024 * 1024]; // 1MB
    }, simpleCache);
    
    // Test LRU cache
    System.out.println("\nLRU cache test:");
    final LRUCache<String, byte[]> lruCache = new LRUCache<>(3);
    // Since LRUCache extends LinkedHashMap, we need a wrapper
    for (int i = 0; i < 5; i++) {
        String key = "key" + i;
        System.out.println("Adding " + key);
        lruCache.put(key, new byte[1024 * 1024]); // 1MB
        System.out.println("Cache size: " + lruCache.size());
    }
    
    // Test expiring cache
    System.out.println("\nExpiring cache test:");
    ExpiringCache<String, String> expiringCache = new ExpiringCache<>(2, TimeUnit.SECONDS);
    expiringCache.put("key1", "value1");
    System.out.println("Initial value: " + expiringCache.get("key1"));
    Thread.sleep(1000);
    System.out.println("After 1 second: " + expiringCache.get("key1"));
    Thread.sleep(1500);
    System.out.println("After 2.5 seconds: " + expiringCache.get("key1"));
    
    // Test weak cache
    System.out.println("\nWeak cache test:");
    WeakCache<String, byte[]> weakCache = new WeakCache<>();
    
    // Regular string literals are interned and strongly reachable
    weakCache.put(new String("key1"), new byte[1024 * 1024]);
    weakCache.put(new String("key2"), new byte[1024 * 1024]);
    
    System.out.println("Initial size: " + weakCache.size());
    System.gc();
    Thread.sleep(100);
    System.out.println("Size after GC: " + weakCache.size());
}

private static <K> void testCache(java.util.function.Function<K, byte[]> computeFunc, 
                                 SimpleCache<K, byte[]> cache) {
    K key = (K) "testKey";
    
    // First access computes the value
    long start = System.nanoTime();
    byte[] value1 = cache.get(key, computeFunc);
    long firstAccess = System.nanoTime() - start;
    
    // Second access should be from cache
    start = System.nanoTime();
    byte[] value2 = cache.get(key, computeFunc);
    long secondAccess = System.nanoTime() - start;
    
    System.out.println("First access time: " + firstAccess / 1_000_000.0 + " ms");
    System.out.println("Second access time: " + secondAccess / 1_000_000.0 + " ms");
    System.out.println("Speed improvement: " + (firstAccess / (double)secondAccess) + "x");
}
}
```

**Caching Strategies:**

- **Simple Cache**: Basic key-value storage with ConcurrentHashMap
- **LRU Cache**: Evicts least recently used entries when capacity exceeded
- **Expiring Cache**: Time-based entry expiration
- **Weak Cache**: Entries removed when keys are garbage collected
- **Multi-level Cache**: Combines different caching strategies

**Caching Considerations:**

- Cache hit ratio and efficiency
- Memory consumption
- Concurrency requirements
- Eviction policies
- Cache invalidation strategies
- Consistency with source data

### Batch Processing

```java
// Example of batch processing for improved performance
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class BatchProcessingExample {
    // Simulate database connection and operations
    public static void main(String[] args) throws Exception {
        // Setup sample data
        List<User> users = generateUsers(10000);

        // Compare individual vs. batch inserts
        System.out.println("Individual inserts:");
        long individualTime = timeIndividualInserts(users);

        System.out.println("\\nBatch inserts:");
        long batchTime = timeBatchInserts(users);

        System.out.println("\\nPerformance comparison:");
        System.out.println("Individual: " + individualTime + " ms");
        System.out.println("Batch: " + batchTime + " ms");
        System.out.println("Speedup: " + (double)individualTime / batchTime + "x");

        // Batch reading example
        System.out.println("\\nBatch reading:");
        demonstrateBatchReading();
    }

    // Example of individual database operations
    private static long timeIndividualInserts(List<User> users) throws Exception {
        try (Connection conn = getConnection()) {
            long start = System.currentTimeMillis();

            // Insert each user with separate statement
            String sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {

                int count = 0;
                for (User user : users) {
                    stmt.setString(1, user.name);
                    stmt.setString(2, user.email);
                    stmt.setInt(3, user.age);

                    // Execute each insert individually
                    stmt.executeUpdate();

                    if (++count % 1000 == 0) {
                        System.out.println("Inserted " + count + " users individually");
                    }
                }
            }

            long duration = System.currentTimeMillis() - start;
            System.out.println("Individual insert time: " + duration + " ms");
            return duration;
        }
    }

    // Example of batch database operations
    private static long timeBatchInserts(List<User> users) throws Exception {
        try (Connection conn = getConnection()) {
            // Disable auto-commit for better performance
            conn.setAutoCommit(false);

            long start = System.currentTimeMillis();

            // Use same prepared statement for batch
            String sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {

                int count = 0;
                int batchSize = 1000;

                for (User user : users) {
                    stmt.setString(1, user.name);
                    stmt.setString(2, user.email);
                    stmt.setInt(3, user.age);

                    // Add to batch instead of executing
                    stmt.addBatch();
                    count++;

                    // Execute batch when batch size reached
                    if (count % batchSize == 0) {
                        stmt.executeBatch();
                        conn.commit();
                        System.out.println("Inserted batch of " + batchSize + " users");
                    }
                }

                // Execute remaining batch
                if (count % batchSize != 0) {
                    stmt.executeBatch();
                    conn.commit();
                }
            }

            // Restore auto-commit
            conn.setAutoCommit(true);

            long duration = System.currentTimeMillis() - start;
            System.out.println("Batch insert time: " + duration + " ms");
            return duration;
        }
    }

    // Example of batch reading with fetch size
    private static void demonstrateBatchReading() throws Exception {
        try (Connection conn = getConnection()) {
            // Set fetch size for batch retrieval
            try (Statement stmt = conn.createStatement()) {
                // Fetch size controls how many rows are retrieved at once from the database
                stmt.setFetchSize(1000);

                long start = System.currentTimeMillis();

                // Execute query
                try (ResultSet rs = stmt.executeQuery("SELECT * FROM users")) {
                    int count = 0;
                    while (rs.next()) {
                        // Process row
                        String name = rs.getString("name");
                        String email = rs.getString("email");
                        int age = rs.getInt("age");

                        // Just count rows for this example
                        count++;
                    }
                    System.out.println("Read " + count + " users with batch fetch");
                }

                long duration = System.currentTimeMillis() - start;
                System.out.println("Batch read time: " + duration + " ms");
            }
        }
    }

    // Helper methods for the example

    private static Connection getConnection() throws SQLException {
        // In a real application, this would be a real database connection
        // For this example, we'll simulate database operations
        return new SimulatedConnection();
    }

    private static List<User> generateUsers(int count) {
        List<User> users = new ArrayList<>(count);
        Random random = new Random();

        for (int i = 0; i < count; i++) {
            String name = "User" + i;
            String email = "user" + i + "@example.com";
            int age = 18 + random.nextInt(50);
            users.add(new User(name, email, age));
        }

        return users;
    }

    // Simple user class for the example
    static class User {
        String name;
        String email;
        int age;

        User(String name, String email, int age) {
            this.name = name;
            this.email = email;
            this.age = age;
        }
    }

    // Simulated database classes for the example
    // In a real application, you would use actual JDBC classes

    static class SimulatedConnection implements Connection {
        private boolean autoCommit = true;

        @Override
        public PreparedStatement prepareStatement(String sql) {
            return new SimulatedPreparedStatement();
        }

        @Override
        public Statement createStatement() {
            return new SimulatedStatement();
        }

        @Override
        public void setAutoCommit(boolean autoCommit) {
            this.autoCommit = autoCommit;
        }

        @Override
        public void commit() {
            // Simulate commit operation
            try {
                // Add slight delay to simulate network/disk I/O
                Thread.sleep(5);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        @Override
        public void close() {
            // Simulate closing the connection
        }

        // Other Connection methods omitted for brevity
        // This is just a simulation for the example

        public boolean getAutoCommit() { return autoCommit; }
        public void clearWarnings() {}
        public void rollback() {}
        public void setReadOnly(boolean readOnly) {}
        public boolean isReadOnly() { return false; }
        public void setCatalog(String catalog) {}
        public String getCatalog() { return null; }
        public void setTransactionIsolation(int level) {}
        public int getTransactionIsolation() { return 0; }
        public SQLWarning getWarnings() { return null; }
        public boolean isClosed() { return false; }
        public void setHoldability(int holdability) {}
        public int getHoldability() { return 0; }
        public Savepoint setSavepoint() { return null; }
        public Savepoint setSavepoint(String name) { return null; }
        public void rollback(Savepoint savepoint) {}
        public void releaseSavepoint(Savepoint savepoint) {}
        public Statement createStatement(int resultSetType, int resultSetConcurrency) { return null; }
        public Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) { return null; }
        public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) { return null; }
        public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) { return null; }
        public PreparedStatement prepareStatement(String sql, int autoGeneratedKeys) { return null; }
        public PreparedStatement prepareStatement(String sql, int[] columnIndexes) { return null; }
        public PreparedStatement prepareStatement(String sql, String[] columnNames) { return null; }
        public CallableStatement prepareCall(String sql) { return null; }
        public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency) { return null; }
        public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) { return null; }
        public String nativeSQL(String sql) { return null; }
        public Map<String, Class<?>> getTypeMap() { return null; }
        public void setTypeMap(Map<String, Class<?>> map) {}
        public <T> T unwrap(Class<T> iface) { return null; }
        public boolean isWrapperFor(Class<?> iface) { return false; }
        public Clob createClob() { return null; }
        public Blob createBlob() { return null; }
        public NClob createNClob() { return null; }
        public SQLXML createSQLXML() { return null; }
        public boolean isValid(int timeout) { return true; }
        public void setClientInfo(String name, String value) {}
        public void setClientInfo(Properties properties) {}
        public String getClientInfo(String name) { return null; }
        public Properties getClientInfo() { return null; }
        public Array createArrayOf(String typeName, Object[] elements) { return null; }
        public Struct createStruct(String typeName, Object[] attributes) { return null; }
        public void setSchema(String schema) {}
        public String getSchema() { return null; }
        public void abort(Executor executor) {}
        public void setNetworkTimeout(Executor executor, int milliseconds) {}
        public int getNetworkTimeout() { return 0; }
    }

    static class SimulatedPreparedStatement implements PreparedStatement {
        private final List<Object[]> batchParams = new ArrayList<>();

        @Override
        public void setString(int parameterIndex, String x) {
            // Simulate parameter setting
        }

        @Override
        public void setInt(int parameterIndex, int x) {
            // Simulate parameter setting
        }

        @Override
        public int executeUpdate() {
            // Simulate individual statement execution
            try {
                // Add delay to simulate database operation
                Thread.sleep(5);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return 1;
        }

        @Override
        public void addBatch() {
            // Simulate adding to batch
            batchParams.add(new Object[3]);
        }

        @Override
        public int[] executeBatch() {
            // Simulate batch execution
            // Batch is much more efficient per statement
            int size = batchParams.size();
            int[] result = new int[size];

            try {
                // Add delay that's much less than individual statements would take
                Thread.sleep(size / 100 + 10);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }

            for (int i = 0; i < size; i++) {
                result[i] = 1; // 1 row affected per statement
            }

            batchParams.clear();
            return result;
        }

        @Override
        public void close() {
            // Simulate closing the statement
        }

        // Other PreparedStatement methods omitted for brevity
        // This is just a simulation for the example

        public ResultSet executeQuery() { return null; }
        public void setNull(int parameterIndex, int sqlType) {}
        public void setBoolean(int parameterIndex, boolean x) {}
        public void setByte(int parameterIndex, byte x) {}
        public void setShort(int parameterIndex, short x) {}
        public void setLong(int parameterIndex, long x) {}
        public void setFloat(int parameterIndex, float x) {}
        public void setDouble(int parameterIndex, double x) {}
        public void setBigDecimal(int parameterIndex, BigDecimal x) {}
        public void setBytes(int parameterIndex, byte[] x) {}
        public void setDate(int parameterIndex, Date x) {}
        public void setTime(int parameterIndex, Time x) {}
        public void setTimestamp(int parameterIndex, Timestamp x) {}
        public void setAsciiStream(int parameterIndex, InputStream x, int length) {}
        public void setUnicodeStream(int parameterIndex, InputStream x, int length) {}
        public void setBinaryStream(int parameterIndex, InputStream x, int length) {}
        public void clearParameters() {}
        public void setObject(int parameterIndex, Object x, int targetSqlType) {}
        public void setObject(int parameterIndex, Object x) {}
        public boolean execute() { return false; }
        public void addBatch(String sql) {}
        public void clearBatch() {}
        public ResultSetMetaData getMetaData() { return null; }
        public void setRef(int parameterIndex, Ref x) {}
        public void setBlob(int parameterIndex, Blob x) {}
        public void setClob(int parameterIndex, Clob x) {}
        public void setArray(int parameterIndex, Array x) {}
        public ResultSet getGeneratedKeys() { return null; }
        public int executeUpdate(String sql) { return 0; }
        public boolean execute(String sql) { return false; }
        public void setURL(int parameterIndex, URL x) {}
        public ParameterMetaData getParameterMetaData() { return null; }
        public void setRowId(int parameterIndex, RowId x) {}
        public void setNString(int parameterIndex, String value) {}
        public void setNCharacterStream(int parameterIndex, Reader value, long length) {}
        public void setNClob(int parameterIndex, NClob value) {}
        public void setClob(int parameterIndex, Reader reader, long length) {}
        public void setBlob(int parameterIndex, InputStream inputStream, long length) {}
        public void setNClob(int parameterIndex, Reader reader, long length) {}
        public void setSQLXML(int parameterIndex, SQLXML xmlObject) {}
        public void setObject(int parameterIndex, Object x, int targetSqlType, int scaleOrLength) {}
        public void setAsciiStream(int parameterIndex, InputStream x, long length) {}
        public void setBinaryStream(int parameterIndex, InputStream x, long length) {}
        public void setCharacterStream(int parameterIndex, Reader reader, long length) {}
        public void setAsciiStream(int parameterIndex, InputStream x) {}
        public void setBinaryStream(int parameterIndex, InputStream x) {}
        public void setCharacterStream(int parameterIndex, Reader reader) {}
        public void setNCharacterStream(int parameterIndex, Reader value) {}
        public void setClob(int parameterIndex, Reader reader) {}
        public void setBlob(int parameterIndex, InputStream inputStream) {}
        public void setNClob(int parameterIndex, Reader reader) {}
        public void setDate(int parameterIndex, Date x, Calendar cal) {}
        public void setTime(int parameterIndex, Time x, Calendar cal) {}
        public void setTimestamp(int parameterIndex, Timestamp x, Calendar cal) {}
        public void setNull(int parameterIndex, int sqlType, String typeName) {}
        public void setCharacterStream(int parameterIndex, Reader reader, int length) {}
        public void setObject(int parameterIndex, Object x, SQLType targetSqlType, int scaleOrLength) {}
        public void setObject(int parameterIndex, Object x, SQLType targetSqlType) {}
        public long executeLargeUpdate() { return 0; }
        public ResultSet executeQuery(String sql) { return null; }
        public int getMaxFieldSize() { return 0; }
        public void setMaxFieldSize(int max) {}
        public int getMaxRows() { return 0; }
        public void setMaxRows(int max) {}
        public void setEscapeProcessing(boolean enable) {}
        public int getQueryTimeout() { return 0; }
        public void setQueryTimeout(int seconds) {}
        public void cancel() {}
        public SQLWarning getWarnings() { return null; }
        public void clearWarnings() {}
        public void setCursorName(String name) {}
        public int[] executeBatch(String... sql) { return null; }
        public void setFetchDirection(int direction) {}
        public int getFetchDirection() { return 0; }
        public void setFetchSize(int rows) {}
        public int getFetchSize() { return 0; }
        public int getResultSetConcurrency() { return 0; }
        public int getResultSetType() { return 0; }
        public Connection getConnection() { return null; }
        public boolean getMoreResults(int current) { return false; }
        public boolean getMoreResults() { return false; }
        public int getResultSetHoldability() { return 0; }
        public int executeUpdate(String sql, int autoGeneratedKeys) { return 0; }
        public int executeUpdate(String sql, int[] columnIndexes) { return 0; }
        public int executeUpdate(String sql, String[] columnNames) { return 0; }
        public boolean execute(String sql, int autoGeneratedKeys) { return false; }
        public boolean execute(String sql, int[] columnIndexes) { return false; }
        public boolean execute(String sql, String[] columnNames) { return false; }
        public void closeOnCompletion() {}
        public boolean isCloseOnCompletion() { return false; }
        public <T> T unwrap(Class<T> iface) { return null; }
        public boolean isWrapperFor(Class<?> iface) { return false; }
        public long getLargeUpdateCount() { return 0; }
        public void setLargeMaxRows(long max) {}
        public long getLargeMaxRows() { return 0; }
        public long[] executeLargeBatch() { return null; }
        public long executeLargeUpdate(String sql) { return 0; }
        public long executeLargeUpdate(String sql, int autoGeneratedKeys) { return 0; }
        public long executeLargeUpdate(String sql, int[] columnIndexes) { return 0; }
        public long executeLargeUpdate(String sql, String[] columnNames) { return 0; }
    }

    static class SimulatedStatement implements Statement {
        private int fetchSize = 0;

        @Override
        public void setFetchSize(int rows) {
            this.fetchSize = rows;
        }

        @Override
        public ResultSet executeQuery(String sql) {
            // Simulate query execution and return result set
            return new SimulatedResultSet(10000, fetchSize);
        }

        @Override
        public void close() {
            // Simulate closing the statement
        }

        // Other Statement methods omitted for brevity
        // This is just a simulation for the example

        public int executeUpdate(String sql) { return 0; }
        public int getMaxFieldSize() { return 0; }
        public void setMaxFieldSize(int max) {}
        public int getMaxRows() { return 0; }
        public void setMaxRows(int max) {}
        public void setEscapeProcessing(boolean enable) {}
        public int getQueryTimeout() { return 0; }
        public void setQueryTimeout(int seconds) {}
        public void cancel() {}
        public SQLWarning getWarnings() { return null; }
        public void clearWarnings() {}
        public void setCursorName(String name) {}
        public boolean execute(String sql) { return false; }
        public ResultSet getResultSet() { return null; }
        public int getUpdateCount() { return 0; }
        public boolean getMoreResults() { return false; }
        public void setFetchDirection(int direction) {}
        public int getFetchDirection() { return 0; }
        public int getFetchSize() { return fetchSize; }
        public int getResultSetConcurrency() { return 0; }
        public int getResultSetType() { return 0; }
        public void addBatch(String sql) {}
        public void clearBatch() {}
        public int[] executeBatch() { return null; }
        public Connection getConnection() { return null; }
        public boolean getMoreResults(int current) { return false; }
        public ResultSet getGeneratedKeys() { return null; }
        public int executeUpdate(String sql, int autoGeneratedKeys) { return 0; }
        public int executeUpdate(String sql, int[] columnIndexes) { return 0; }
        public int executeUpdate(String sql, String[] columnNames) { return 0; }
        public boolean execute(String sql, int autoGeneratedKeys) { return false; }
        public boolean execute(String sql, int[] columnIndexes) { return false; }
        public boolean execute(String sql, String[] columnNames) { return false; }
        public int getResultSetHoldability() { return 0; }
        public boolean isClosed() { return false; }
        public void setPoolable(boolean poolable) {}
        public boolean isPoolable() { return false; }
        public void closeOnCompletion() {}
        public boolean isCloseOnCompletion() { return false; }
        public <T> T unwrap(Class<T> iface) { return null; }
        public boolean isWrapperFor(Class<?> iface) { return false; }
        public long getLargeUpdateCount() { return 0; }
        public void setLargeMaxRows(long max) {}
        public long getLargeMaxRows() { return 0; }
        public long[] executeLargeBatch() { return null; }
        public long executeLargeUpdate(String sql) { return 0; }
        public long executeLargeUpdate(String sql, int autoGeneratedKeys) { return 0; }
        public long executeLargeUpdate(String sql, int[] columnIndexes) { return 0; }
        public long executeLargeUpdate(String sql, String[] columnNames) { return 0; }
        public String enquoteLiteral(String val) { return null; }
        public String enquoteIdentifier(String identifier, boolean alwaysQuote) { return null; }
        public boolean isSimpleIdentifier(String identifier) { return false; }
        public String enquoteNCharLiteral(String val) { return null; }
    }

    static class SimulatedResultSet implements ResultSet {
        private final int totalRows;
        private final int fetchSize;
        private int currentRow = -1;

        public SimulatedResultSet(int totalRows, int fetchSize) {
            this.totalRows = totalRows;
            this.fetchSize = fetchSize > 0 ? fetchSize : 100; // Default fetch size
        }

        @Override
        public boolean next() {
            currentRow++;

            if (currentRow < totalRows) {
                // Simulate network delay for fetching batches
                if (fetchSize > 0 && currentRow % fetchSize == 0) {
                    try {
                        // Longer delay for smaller fetch sizes
                        Thread.sleep(100 / (fetchSize / 100));
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
                return true;
            }

            return false;
        }

        @Override
        public String getString(String columnLabel) {
            // Simulate getting string value
            return "value" + currentRow;
        }

        @Override
        public int getInt(String columnLabel) {
            // Simulate getting int value
            return currentRow % 100;
        }

        @Override
        public void close() {
            // Simulate closing the result set
        }

        // Other ResultSet methods omitted for brevity
        // This is just a simulation for the example

        public boolean wasNull() { return false; }
        public String getString(int columnIndex) { return null; }
        public boolean getBoolean(int columnIndex) { return false; }
        public byte getByte(int columnIndex) { return 0; }
        public short getShort(int columnIndex) { return 0; }
        public int getInt(int columnIndex) { return 0; }
        public long getLong(int columnIndex) { return 0; }
        public float getFloat(int columnIndex) { return 0; }
        public double getDouble(int columnIndex) { return 0; }
        public BigDecimal getBigDecimal(int columnIndex, int scale) { return null; }
        public byte[] getBytes(int columnIndex) { return null; }
        public Date getDate(int columnIndex) { return null; }
        public Time getTime(int columnIndex) { return null; }
        public Timestamp getTimestamp(int columnIndex) { return null; }
        public InputStream getAsciiStream(int columnIndex) { return null; }
        public InputStream getUnicodeStream(int columnIndex) { return null; }
        public InputStream getBinaryStream(int columnIndex) { return null; }
        public boolean getBoolean(String columnLabel) { return false; }
        public byte getByte(String columnLabel) { return 0; }
        public short getShort(String columnLabel) { return 0; }
        public long getLong(String columnLabel) { return 0; }
        public float getFloat(String columnLabel) { return 0; }
        public double getDouble(String columnLabel) { return 0; }
        public BigDecimal getBigDecimal(String columnLabel, int scale) { return null; }
        public byte[] getBytes(String columnLabel) { return null; }
        public Date getDate(String columnLabel) { return null; }
        public Time getTime(String columnLabel) { return null; }
        public Timestamp getTimestamp(String columnLabel) { return null; }
        public InputStream getAsciiStream(String columnLabel) { return null; }
        public InputStream getUnicodeStream(String columnLabel) { return null; }
        public InputStream getBinaryStream(String columnLabel) { return null; }
        public SQLWarning getWarnings() { return null; }
        public void clearWarnings() {}
        public String getCursorName() { return null; }
        public ResultSetMetaData getMetaData() { return null; }
        public Object getObject(int columnIndex) { return null; }
        public Object getObject(String columnLabel) { return null; }
        public int findColumn(String columnLabel) { return 0; }
        public Reader getCharacterStream(int columnIndex) { return null; }
        public Reader getCharacterStream(String columnLabel) { return null; }
        public BigDecimal getBigDecimal(int columnIndex) { return null; }
        public BigDecimal getBigDecimal(String columnLabel) { return null; }
        public boolean isBeforeFirst() { return false; }
        public boolean isAfterLast() { return false; }
        public boolean isFirst() { return false; }
        public boolean isLast() { return false; }
        public void beforeFirst() {}
        public void afterLast() {}
        public boolean first() { return false; }
        public boolean last() { return false; }
        public int getRow() { return 0; }
        public boolean absolute(int row) { return false; }
        public boolean relative(int rows) { return false; }
        public boolean previous() { return false; }
        public void setFetchDirection(int direction) {}
        public int getFetchDirection() { return 0; }
        public void setFetchSize(int rows) {}
        public int getFetchSize() { return 0; }
        public int getType() { return 0; }
        public int getConcurrency() { return 0; }
        public boolean rowUpdated() { return false; }
        public boolean rowInserted() { return false; }
        public boolean rowDeleted() { return false; }
        public void updateNull(int columnIndex) {}
        public void updateBoolean(int columnIndex, boolean x) {}
        public void updateByte(int columnIndex, byte x) {}
        public void updateShort(int columnIndex, short x) {}
        public void updateInt(int columnIndex, int x) {}
        public void updateLong(int columnIndex, long x) {}
        public void updateFloat(int columnIndex, float x) {}
        public void updateDouble(int columnIndex, double x) {}
        public void updateBigDecimal(int columnIndex, BigDecimal x) {}
        public void updateString(int columnIndex, String x) {}
        public void updateBytes(int columnIndex, byte[] x) {}
        public void updateDate(int columnIndex, Date x) {}
        public void updateTime(int columnIndex, Time x) {}
        public void updateTimestamp(int columnIndex, Timestamp x) {}
        public void updateAsciiStream(int columnIndex, InputStream x, int length) {}
        public void updateBinaryStream(int columnIndex, InputStream x, int length) {}
        public void updateCharacterStream(int columnIndex, Reader x, int length) {}
        public void updateObject(int columnIndex, Object x, int scaleOrLength) {}
        public void updateObject(int columnIndex, Object x) {}
        public void updateNull(String columnLabel) {}
        public void updateBoolean(String columnLabel, boolean x) {}
        public void updateByte(String columnLabel, byte x) {}
        public void updateShort(String columnLabel, short x) {}
        public void updateInt(String columnLabel, int x) {}
        public void updateLong(String columnLabel, long x) {}
        public void updateFloat(String columnLabel, float x) {}
        public void updateDouble(String columnLabel, double x) {}
        public void updateBigDecimal(String columnLabel, BigDecimal x) {}
        public void updateString(String columnLabel, String x) {}
        public void updateBytes(String columnLabel, byte[] x) {}
        public void updateDate(String columnLabel, Date x) {}
        public void updateTime(String columnLabel, Time x) {}
        public void updateTimestamp(String columnLabel, Timestamp x) {}
        public void updateAsciiStream(String columnLabel, InputStream x, int length) {}
        public void updateBinaryStream(String columnLabel, InputStream x, int length) {}
        public void updateCharacterStream(String columnLabel, Reader reader, int length) {}
        public void updateObject(String columnLabel, Object x, int scaleOrLength) {}
        public void updateObject(String columnLabel, Object x) {}
        public void insertRow() {}
        public void updateRow() {}
        public void deleteRow() {}
        public void refreshRow() {}
        public void cancelRowUpdates() {}
        public void moveToInsertRow() {}
        public void moveToCurrentRow() {}
        public Statement getStatement() { return null; }
        public Object getObject(int columnIndex, Map<String, Class<?>> map) { return null; }
        public Ref getRef(int columnIndex) { return null; }
        public Blob getBlob(int columnIndex) { return null; }
        public Clob getClob(int columnIndex) { return null; }
        public Array getArray(int columnIndex) { return null; }
        public Object getObject(String columnLabel, Map<String, Class<?>> map) { return null; }
        public Ref getRef(String columnLabel) { return null; }
        public Blob getBlob(String columnLabel) { return null; }
        public Clob getClob(String columnLabel) { return null; }
        public Array getArray(String columnLabel) { return null; }
        public Date getDate(int columnIndex, Calendar cal) { return null; }
        public Date getDate(String columnLabel, Calendar cal) { return null; }
        public Time getTime(int columnIndex, Calendar cal) { return null; }
        public Time getTime(String columnLabel, Calendar cal) { return null; }
        public Timestamp getTimestamp(int columnIndex, Calendar cal) { return null; }
        public Timestamp getTimestamp(String columnLabel, Calendar cal) { return null; }
        public URL getURL(int columnIndex) { return null; }
        public URL getURL(String columnLabel) { return null; }
        public void updateRef(int columnIndex, Ref x) {}
        public void updateRef(String columnLabel, Ref x) {}
        public void updateBlob(int columnIndex, Blob x) {}
        public void updateBlob(String columnLabel, Blob x) {}
        public void updateClob(int columnIndex, Clob x) {}
        public void updateClob(String columnLabel, Clob x) {}
        public void updateArray(int columnIndex, Array x) {}
        public void updateArray(String columnLabel, Array x) {}
        public RowId getRowId(int columnIndex) { return null; }
        public RowId getRowId(String columnLabel) { return null; }
        public void updateRowId(int columnIndex, RowId x) {}
        public void updateRowId(String columnLabel, RowId x) {}
        public int getHoldability() { return 0; }
        public boolean isClosed() { return false; }
        public void updateNString(int columnIndex, String nString) {}
        public void updateNString(String columnLabel, String nString) {}
        public void updateNClob(int columnIndex, NClob nClob) {}
        public void updateNClob(String columnLabel, NClob nClob) {}
        public NClob getNClob(int columnIndex) { return null; }
        public NClob getNClob(String columnLabel) { return null; }
        public SQLXML getSQLXML(int columnIndex) { return null; }
        public SQLXML getSQLXML(String columnLabel) { return null; }
        public void updateSQLXML(int columnIndex, SQLXML xmlObject) {}
        public void updateSQLXML(String columnLabel, SQLXML xmlObject) {}
        public String getNString(int columnIndex) { return null; }
        public String getNString(String columnLabel) { return null; }
        public Reader getNCharacterStream(int columnIndex) { return null; }
        public Reader getNCharacterStream(String columnLabel) { return null; }
        public void updateNCharacterStream(int columnIndex, Reader x, long length) {}
        public void updateNCharacterStream(String columnLabel, Reader reader, long length) {}
        public void updateAsciiStream(int columnIndex, InputStream x, long length) {}
        public void updateBinaryStream(int columnIndex, InputStream x, long length) {}
        public void updateCharacterStream(int columnIndex, Reader x, long length) {}
        public void updateAsciiStream(String columnLabel, InputStream x, long length) {}
        public void updateBinaryStream(String columnLabel, InputStream x, long length) {}
        public void updateCharacterStream(String columnLabel, Reader reader, long length) {}
        public void updateBlob(int columnIndex, InputStream inputStream, long length) {}
        public void updateBlob(String columnLabel, InputStream inputStream, long length) {}
        public void updateClob(int columnIndex, Reader reader, long length) {}
        public void updateClob(String columnLabel, Reader reader, long length) {}
        public void updateNClob(int columnIndex, Reader reader, long length) {}
        public void updateNClob(String columnLabel, Reader reader, long length) {}
        public void updateNCharacterStream(int columnIndex, Reader x) {}
        public void updateNCharacterStream(String columnLabel, Reader reader) {}
        public void updateAsciiStream(int columnIndex, InputStream x) {}
        public void updateBinaryStream(int columnIndex, InputStream x) {}
        public void updateCharacterStream(int columnIndex, Reader x) {}
        public void updateAsciiStream(String columnLabel, InputStream x) {}
        public void updateBinaryStream(String columnLabel, InputStream x) {}
        public void updateCharacterStream(String columnLabel, Reader reader) {}
        public void updateBlob(int columnIndex, InputStream inputStream) {}
        public void updateBlob(String columnLabel, InputStream inputStream) {}
        public void updateClob(int columnIndex, Reader reader) {}
        public void updateClob(String columnLabel, Reader reader) {}
        public void updateNClob(int columnIndex, Reader reader) {}
        public void updateNClob(String columnLabel, Reader reader) {}
        public <T> T getObject(int columnIndex, Class<T> type) { return null; }
        public <T> T getObject(String columnLabel, Class<T> type) { return null; }
        public <T> T unwrap(Class<T> iface) { return null; }
        public boolean isWrapperFor(Class<?> iface) { return false; }
    }
}
```

**Batch Processing Benefits:**

- Reduces network/IO overhead
- Amortizes operation costs across multiple items
- Improves throughput for bulk operations
- Reduces transaction overhead
- Efficient resource utilization

**Application Areas:**

- Database operations (inserts, updates)
- File operations (read/write)
- Network operations (HTTP, messaging)
- Processing large datasets
- ETL and data pipelines

> **Deep Dive Tip:** When implementing batch processing, consider the optimal batch size based on the operation's characteristics. For database operations, batch sizes between 100-1000 records typically offer a good balance between throughput and latency. For memory-intensive operations, consider the available memory and garbage collection patterns. Too large batch sizes can cause GC pressure, while too small batches reduce efficiency. Additionally, be mindful of error handling—one failed item in a batch might cause the entire batch to fail, requiring appropriate retry mechanisms or partial batch processing capabilities.
> 

> **Interviewer Insight:** When discussing performance best practices, a strong candidate should demonstrate an understanding of when and how to apply different optimization techniques. They should recognize that each technique involves trade-offs and might be appropriate in some contexts but harmful in others. Ask them how they approach performance optimization in real-world applications, looking for a systematic approach that:
> 
> 1. Starts with measurement and profiling to identify actual bottlenecks
> 2. Applies targeted optimizations based on empirical data
> 3. Validates improvements with benchmarks
> 4. Balances performance gains against code complexity and maintainability
> 
> A good candidate should also be able to discuss how different techniques interact—for example, how object pooling might reduce garbage collection pressure, or how caching can complement batch processing for frequently accessed data.
> 

## 10.5 Common Performance Issues

Understanding common performance issues and their resolution is essential for maintaining high-performing Java applications.

### Memory Leaks

Memory leaks in Java occur when objects remain referenced even though they're no longer needed, preventing garbage collection.

```java
*// Common memory leak patterns*
public class MemoryLeakExamples {
    *// 1. Static collections*
    private static final List<Object> staticCollection = new ArrayList<>();
    
    public void leakThroughStaticCollection() {
        *// This data is never removed, leading to a memory leak*
        for (int i = 0; i < 1000; i++) {
            staticCollection.add(new byte[1024 * 1024]); *// 1MB object*
        }
    }
    
    *// 2. Forgotten listeners*
    public void leakThroughListeners() {
        DataProcessor processor = new DataProcessor();
        EventSource eventSource = new EventSource();
        
        *// Add listener but never remove it*
        eventSource.addListener(processor);
        
        *// Process and discard, but listener reference remains*
        processor.processData();
        processor = null; *// Processor should be garbage collected, but can't be*
    }
    
    *// 3. Improper cache implementation*
    private final Map<String, byte[]> unboundedCache = new HashMap<>();
    
    public void leakThroughCache() {
        for (int i = 0; i < 1000; i++) {
            String key = "data" + i;
            *// Cache keeps growing without bounds*
            unboundedCache.put(key, new byte[1024 * 1024]); *// 1MB*
        }
    }
    
    *// 4. Unclosed resources*
    public void leakThroughResources() {
        try {
            *// Open a resource*
            FileInputStream fis = new FileInputStream("data.txt");
            byte[] buffer = new byte[1024];
            fis.read(buffer);
            
            *// Forgot to close the resource - file handle leak// Missing: fis.close()*
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    *// 5. Inner class references*
    public Object leakThroughInnerClass() {
        *// Non-static inner class holds implicit reference to outer instance*
        class DataHolder {
            private byte[] data = new byte[1024 * 1024]; *// 1MB*
        }
        
        return new DataHolder();
    }
    
    *// Supporting classes for examples*
    static class EventSource {
        private final List<DataProcessor> listeners = new ArrayList<>();
        
        public void addListener(DataProcessor processor) {
            listeners.add(processor);
        }
        
        public void removeListener(DataProcessor processor) {
            listeners.remove(processor);
        }
    }
    
    static class DataProcessor {
        private byte[] buffer = new byte[1024 * 1024]; *// 1MB*
        
        public void processData() {
            *// Process data and done - but might still be held by listeners*
        }
    }
    
    *// 6. Thread-local leaks*
    private static ThreadLocal<byte[]> threadLocalBuffer = 
        new ThreadLocal<byte[]>() {
            @Override
            protected byte[] initialValue() {
                return new byte[1024 * 1024]; *// 1MB*
            }
        };
    
    public void leakThroughThreadLocal() {
        *// Use thread-local*
        byte[] buffer = threadLocalBuffer.get();
        
        *// Process data with buffer*
        
        *// Missing: threadLocalBuffer.remove();// In thread pools, this leads to leaks*
    }
    
    *// 7. Classloader leaks*
    public void leakThroughClassLoader() {
        try {
            *// Create custom classloader*
            URL[] urls = new URL[] { new URL("file:///path/to/classes/") };
            URLClassLoader classLoader = new URLClassLoader(urls);
            
            *// Load class with custom loader*
            Class<?> clazz = classLoader.loadClass("com.example.SomeClass");
            Object instance = clazz.getDeclaredConstructor().newInstance();
            
            *// Register in static map (oops!)*
            registerInStaticMap(clazz.getName(), instance);
            
            *// Missing: classLoader.close();// The static reference prevents the classloader and all its classes from being GC'd*
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    private static final Map<String, Object> staticRegistry = new HashMap<>();
    
    private static void registerInStaticMap(String key, Object instance) {
        staticRegistry.put(key, instance);
    }
    
    *// 8. Object finalization issues*
    public void leakThroughFinalization() {
        for (int i = 0; i < 100000; i++) {
            *// Objects with finalizers are processed specially by GC// and can cause memory pressure*
            new ObjectWithFinalizer();
        }
    }
    
    static class ObjectWithFinalizer {
        private byte[] data = new byte[1024]; *// 1KB*
        
        @Override
        protected void finalize() throws Throwable {
            *// Finalizer delays garbage collection// and can even resurrect objects*
            System.out.println("Finalizing object");
            super.finalize();
        }
    }
}
```

**Common Memory Leak Patterns:**

1. **Static Collections**: Collections stored in static fields that grow unbounded
2. **Forgotten Listeners**: Event listeners registered but never unregistered
3. **Improper Caching**: Caches without size limits or eviction policies
4. **Unclosed Resources**: File handles, connections not properly closed
5. **Inner Class References**: Non-static inner classes holding references to outer instances
6. **Thread-Local Variables**: Not removed when threads are recycled in thread pools
7. **Classloader Leaks**: References preventing classloader unloading
8. **Object Finalization**: Objects with finalizers causing memory pressure

**Detection Techniques:**

- Heap dumps analysis (Eclipse MAT, VisualVM)
- Memory usage trending (JConsole, JMX)
- Profilers with allocation tracking
- GC log analysis for increasing tenured space

**Prevention Strategies:**

- Use weak references for caches and listeners
- Implement proper resource management (try-with-resources)
- Prefer static inner classes when outer instance not needed
- Set size limits for collections and caches
- Use thread-local with caution, always remove when done

> **Deep Dive Tip:** One particularly challenging type of memory leak is the classloader leak, which can occur in container environments like application servers. When a class from a deployed application is referenced by a singleton or static field in the container, the entire classloader and all classes it loaded cannot be garbage collected, even when the application is undeployed. This can lead to PermGen/Metaspace exhaustion over time. To detect such leaks, look for classes that should have been unloaded in heap dumps, often identifiable by unique classloader references.
> 

> **Interviewer Insight:** When discussing memory leaks, a strong candidate should understand that Java's garbage collection doesn't eliminate all memory leaks—it only handles unreferenced objects. They should demonstrate awareness of how to detect memory leaks through a combination of monitoring, profiling, and heap analysis. Ask them how they would diagnose a suspected memory leak in a production application, looking for a methodical approach that:
> 
> 1. Confirms the symptom (growing memory usage that doesn't stabilize)
> 2. Collects diagnostic information (heap dumps, GC logs)
> 3. Analyzes the data to identify leaking objects and their retention paths
> 4. Implements targeted fixes rather than general memory optimizations
> 
> They should also understand the trade-offs in leak prevention techniques, such as the performance impact of using WeakHashMap vs. regular HashMap for caching.
> 

### Thread Issues

Thread-related issues can significantly impact application performance and stability.

### Deadlocks

```java
*// Example of deadlock*
public class DeadlockExample {
    private static final Object LOCK_1 = new Object();
    private static final Object LOCK_2 = new Object();
    
    public static void main(String[] args) {
        *// Thread 1: Acquires LOCK_1, then tries to acquire LOCK_2*
        Thread thread1 = new Thread(() -> {
            System.out.println("Thread 1: Attempting to acquire LOCK_1");
            synchronized (LOCK_1) {
                System.out.println("Thread 1: Acquired LOCK_1");
                
                *// Simulate some work*
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                
                System.out.println("Thread 1: Attempting to acquire LOCK_2");
                synchronized (LOCK_2) {
                    System.out.println("Thread 1: Acquired LOCK_2");
                    *// Critical section with both locks*
                }
            }
        });
        
        *// Thread 2: Acquires LOCK_2, then tries to acquire LOCK_1*
        Thread thread2 = new Thread(() -> {
            System.out.println("Thread 2: Attempting to acquire LOCK_2");
            synchronized (LOCK_2) {
                System.out.println("Thread 2: Acquired LOCK_2");
                
                *// Simulate some work*
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                
                System.out.println("Thread 2: Attempting to acquire LOCK_1");
                synchronized (LOCK_1) {
                    System.out.println("Thread 2: Acquired LOCK_1");
                    *// Critical section with both locks*
                }
            }
        });
        
        *// Start both threads*
        thread1.start();
        thread2.start();
        
        *// Wait for threads to finish (they won't due to deadlock)*
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Program completed");
    }
    
    *// Deadlock prevention - establish lock ordering*
    public static void preventDeadlock() {
        *// Always acquire locks in the same order*
        Thread thread1 = new Thread(() -> {
            synchronized (LOCK_1) {
                synchronized (LOCK_2) {
                    *// Critical section with both locks*
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            synchronized (LOCK_1) { *// Same order as thread1*
                synchronized (LOCK_2) {
                    *// Critical section with both locks*
                }
            }
        });
        
        thread1.start();
        thread2.start();
    }
    
    *// Deadlock detection*
    public static void detectDeadlock() {
        *// Get all threads*
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        long[] threadIds = threadMXBean.findDeadlockedThreads();
        
        if (threadIds != null) {
            System.out.println("Deadlock detected!");
            
            ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(threadIds, true, true);
            for (ThreadInfo threadInfo : threadInfos) {
                System.out.println(threadInfo.getThreadName() + " is deadlocked");
                System.out.println("Waiting for lock: " + threadInfo.getLockName());
                System.out.println("Lock held by: " + threadInfo.getLockOwnerName());
                System.out.println("Stack trace:");
                for (StackTraceElement element : threadInfo.getStackTrace()) {
                    System.out.println("\t" + element);
                }
            }
        } else {
            System.out.println("No deadlock detected");
        }
    }
}
```

**Deadlock Prevention:**

- Establish a consistent lock ordering
- Use lock timeouts (tryLock with timeout)
- Avoid nested locks when possible
- Use higher-level concurrency utilities (java.util.concurrent)
- Acquire multiple locks atomically

**Deadlock Detection:**

- Use JConsole or VisualVM to identify deadlocked threads
- ThreadMXBean.findDeadlockedThreads() for programmatic detection
- Thread dumps and stack trace analysis
- Automated monitoring with deadlock detection

### Thread Leaks

```java
*// Example of thread leak*
public class ThreadLeakExample {
    public static void main(String[] args) {
        *// Create a leaky thread pool*
        LeakyExecutor executor = new LeakyExecutor();
        
        *// Submit many tasks*
        for (int i = 0; i < 1000; i++) {
            final int taskId = i;
            executor.submitTask(() -> {
                try {
                    System.out.println("Executing task " + taskId);
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        *// In a real app, thread leak would continue*
        System.out.println("Main thread completed - check thread count");
    }
    
    *// Leaky executor that creates threads but doesn't reuse them*
    static class LeakyExecutor {
        private final List<Thread> threads = new ArrayList<>();
        
        public void submitTask(Runnable task) {
            *// Create new thread for each task - no reuse!*
            Thread thread = new Thread(task);
            threads.add(thread); *// Store reference preventing GC*
            thread.start();
        }
    }
    
    *// Proper implementation using thread pool*
    static class ProperExecutor {
        private final ExecutorService executor = Executors.newFixedThreadPool(10);
        
        public void submitTask(Runnable task) {
            executor.submit(task);
        }
        
        public void shutdown() {
            executor.shutdown();
        }
    }
    
    *// Thread leak through timers*
    static void leakThroughTimer() {
        Timer timer = new Timer();
        
        *// Schedule recurring task - timer creates thread that doesn't terminate*
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                System.out.println("Timer task executed");
            }
        }, 1000, 1000);
        
        *// Missing: timer.cancel() - thread continues running*
    }
    
    *// Thread leak in thread-local cleanup*
    static ThreadLocal<byte[]> hugeThreadLocal = new ThreadLocal<byte[]>() {
        @Override
        protected byte[] initialValue() {
            return new byte[1024 * 1024 * 10]; *// 10MB*
        }
    };
    
    static void leakThroughThreadLocal() {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                *// Use thread-local*
                byte[] data = hugeThreadLocal.get();
                
                *// Process data*
                
                *// Missing: hugeThreadLocal.remove()// Each thread in pool now has 10MB in thread-local*
            });
        }
    }
}
```

**Thread Leak Causes:**

- Creating threads without bounds or pools
- Uncancelled Timer/ScheduledExecutorService tasks
- Non-daemon threads not terminated
- Thread pools not shut down properly
- Threads blocked indefinitely

**Prevention Strategies:**

- Use thread pools with appropriate sizing
- Properly shut down ExecutorService instances
- Use daemon threads for background tasks
- Implement proper cancellation handling
- Monitor thread creation and termination

### Thread Contention

```java
*// Example of thread contention*
public class ThreadContentionExample {
    private static final int THREAD_COUNT = 20;
    private static final int ITERATION_COUNT = 100000;
    
    public static void main(String[] args) throws InterruptedException {
        *// Compare different synchronization approaches*
        demonstrateHighContention();
        demonstrateLowContention();
        demonstrateNoContention();
    }
    
    *// High contention: All threads compete for the same lock*
    private static void demonstrateHighContention() throws InterruptedException {
        System.out.println("\nHigh Contention Test");
        
        *// Single shared counter with synchronized method*
        Counter sharedCounter = new Counter();
        
        *// Start multiple threads*
        Thread[] threads = new Thread[THREAD_COUNT];
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < ITERATION_COUNT; j++) {
                    sharedCounter.increment();
                }
            });
            threads[i].start();
        }
        
        *// Wait for all threads to complete*
        for (Thread thread : threads) {
            thread.join();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("High contention duration: " + duration + " ms");
        System.out.println("Final count: " + sharedCounter.getCount());
    }
    
    *// Low contention: Each thread has its own counter, only aggregate at end*
    private static void demonstrateLowContention() throws InterruptedException {
        System.out.println("\nLow Contention Test");
        
        *// Final shared counter for aggregation*
        Counter sharedCounter = new Counter();
        
        *// Start multiple threads*
        Thread[] threads = new Thread[THREAD_COUNT];
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(() -> {
                *// Thread-local counter*
                Counter localCounter = new Counter();
                
                for (int j = 0; j < ITERATION_COUNT; j++) {
                    localCounter.incrementNonSynchronized();
                }
                
                *// Only synchronize once at the end*
                sharedCounter.add(localCounter.getCount());
            });
            threads[i].start();
        }
        
        *// Wait for all threads to complete*
        for (Thread thread : threads) {
            thread.join();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Low contention duration: " + duration + " ms");
        System.out.println("Final count: " + sharedCounter.getCount());
    }
    
    *// No contention: Using AtomicLong instead of synchronized methods*
    private static void demonstrateNoContention() throws InterruptedException {
        System.out.println("\nNo Contention Test (AtomicLong)");
        
        *// Atomic counter*
        AtomicCounter atomicCounter = new AtomicCounter();
        
        *// Start multiple threads*
        Thread[] threads = new Thread[THREAD_COUNT];
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < ITERATION_COUNT; j++) {
                    atomicCounter.increment();
                }
            });
            threads[i].start();
        }
        
        *// Wait for all threads to complete*
        for (Thread thread : threads) {
            thread.join();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("No contention duration: " + duration + " ms");
        System.out.println("Final count: " + atomicCounter.getCount());
    }
    
    *// Synchronized counter implementation*
    static class Counter {
        private long count = 0;
        
        *// Synchronized method - high contention*
        public synchronized void increment() {
            count++;
        }
        
        *// Non-synchronized method - for thread-local counters*
        public void incrementNonSynchronized() {
            count++;
        }
        
        *// Synchronized method for final aggregation*
        public synchronized void add(long value) {
            count += value;
        }
        
        public long getCount() {
            return count;
        }
    }
    
    *// Atomic counter implementation*
    static class AtomicCounter {
        private AtomicLong count = new AtomicLong(0);
        
        public void increment() {
            count.incrementAndGet();
        }
        
        public long getCount() {
            return count.get();
        }
    }
}
```

**Contention Reduction Strategies:**

1. **Minimize Lock Scope**: Reduce synchronized block size
2. **Lock Splitting**: Use separate locks for independent operations
3. **Lock Striping**: Divide single lock into multiple locks (e.g., ConcurrentHashMap)
4. **Thread-Local Processing**: Process locally, synchronize only for aggregation
5. **Atomic Variables**: Use java.util.concurrent.atomic for simple operations
6. **Concurrent Collections**: Use specialized thread-safe collections
7. **Non-Blocking Algorithms**: Implement lock-free data structures

> **Deep Dive Tip:** Thread contention can be difficult to diagnose because its impact varies with load and timing. Under light load, contention may not be noticeable, but as concurrency increases, performance can degrade non-linearly. A key technique for identifying contention is profiling with thread state analysis—look for threads spending significant time in BLOCKED or WAITING states. Tools like async-profiler can generate flame graphs that highlight blocking patterns, making it easier to identify which locks are causing the most contention.
> 

> **Interviewer Insight:** When discussing thread issues, a strong candidate should demonstrate a deep understanding of concurrency hazards and mitigation strategies. They should recognize that high-level concurrency abstractions (like ExecutorService, concurrent collections, and atomic variables) are generally preferable to manual thread management and low-level synchronization. Ask them about their approach to designing thread-safe applications, looking for principles like:
> 
> 1. Immutability where possible (immutable objects are inherently thread-safe)
> 2. Encapsulation of mutable state within thread-safe classes
> 3. Minimizing synchronization scope to reduce contention
> 4. Favoring composition over inheritance for thread-safe classes
> 5. Using appropriate abstractions for different concurrency patterns
> 
> They should also be able to explain how they would diagnose and resolve thread issues in production environments, demonstrating familiarity with tools like thread dumps, deadlock detection, and profilers.
> 

### Excessive Garbage Collection

Excessive garbage collection can cause application pauses and performance degradation.

```java
*// Example demonstrating excessive GC*
public class ExcessiveGCExample {
    public static void main(String[] args) {
        *// Enable GC logging// Java 9+: -Xlog:gc*=info// Java 8: -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps*
        
        *// Example 1: Temporary object allocation in loop*
        demonstrateTemporaryObjectsInLoop();
        
        *// Example 2: Unnecessary boxing/unboxing*
        demonstrateBoxingUnboxing();
        
        *// Example 3: String concatenation in loops*
        demonstrateStringConcatenation();
        
        *// Example 4: Collection resizing*
        demonstrateCollectionResizing();
    }
    
    *// Problem: Creates many short-lived objects*
    private static void demonstrateTemporaryObjectsInLoop() {
        System.out.println("\nTemporary Objects in Loop:");
        
        long startTime = System.currentTimeMillis();
        long sum = 0;
        
        *// Inefficient: Creates many temporary objects*
        for (int i = 0; i < 10_000_000; i++) {
            *// Each iteration creates a new Integer object*
            sum += new Integer(i).intValue();
        }
        
        long inefficientTime = System.currentTimeMillis() - startTime;
        System.out.println("Inefficient way: " + inefficientTime + " ms, Sum: " + sum);
        
        *// Reset and try efficient way*
        startTime = System.currentTimeMillis();
        sum = 0;
        
        *// Efficient: Avoids temporary objects*
        for (int i = 0; i < 10_000_000; i++) {
            *// Direct primitive operations, no objects*
            sum += i;
        }
        
        long efficientTime = System.currentTimeMillis() - startTime;
        System.out.println("Efficient way: " + efficientTime + " ms, Sum: " + sum);
        System.out.println("Improvement: " + (inefficientTime / (double)efficientTime) + "x");
    }
    
    *// Problem: Unnecessary boxing and unboxing*
    private static void demonstrateBoxingUnboxing() {
        System.out.println("\nBoxing and Unboxing:");
        
        long startTime = System.currentTimeMillis();
        Long sum = 0L;
        
        *// Inefficient: Implicit boxing/unboxing on every iteration*
        for (long i = 0; i < 10_000_000; i++) {
            sum += i; *// Boxing i, unboxing sum, boxing result*
        }
        
        long inefficientTime = System.currentTimeMillis() - startTime;
        System.out.println("With boxing/unboxing: " + inefficientTime + " ms, Sum: " + sum);
        
        *// Reset and try primitive version*
        startTime = System.currentTimeMillis();
        long primitiveSum = 0L;
        
        *// Efficient: Direct primitive operations*
        for (long i = 0; i < 10_000_000; i++) {
            primitiveSum += i;
        }
        
        long efficientTime = System.currentTimeMillis() - startTime;
        System.out.println("With primitives: " + efficientTime + " ms, Sum: " + primitiveSum);
        System.out.println("Improvement: " + (inefficientTime / (double)efficientTime) + "x");
    }
    
    *// Problem: String concatenation in loops*
    private static void demonstrateStringConcatenation() {
        System.out.println("\nString Concatenation:");
        
        *// Inefficient: Creates many temporary strings*
        long startTime = System.currentTimeMillis();
        String result = "";
        
        for (int i = 0; i < 100_000; i++) {
            *// Each concatenation creates a new String object*
            result += i;
        }
        
        long inefficientTime = System.currentTimeMillis() - startTime;
        System.out.println("With string concatenation: " + inefficientTime + " ms, Length: " + result.length());
        
        *// Reset and try StringBuilder*
        startTime = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        
        for (int i = 0; i < 100_000; i++) {
            *// Append modifies the existing StringBuilder*
            sb.append(i);
        }
        
        result = sb.toString();
        long efficientTime = System.currentTimeMillis() - startTime;
        System.out.println("With StringBuilder: " + efficientTime + " ms, Length: " + result.length());
        System.out.println("Improvement: " + (inefficientTime / (double)efficientTime) + "x");
    }
    
    *// Problem: Collection resizing*
    private static void demonstrateCollectionResizing() {
        System.out.println("\nCollection Resizing:");
        
        *// Inefficient: ArrayList with default initial capacity*
        long startTime = System.currentTimeMillis();
        List<Integer> list = new ArrayList<>();
        
        for (int i = 0; i < 10_000_000; i++) {
            *// Will resize multiple times as it grows*
            list.add(i);
        }
        
        long inefficientTime = System.currentTimeMillis() - startTime;
        System.out.println("Default ArrayList: " + inefficientTime + " ms, Size: " + list.size());
        
        *// Reset and try with pre-sized ArrayList*
        startTime = System.currentTimeMillis();
        list = new ArrayList<>(10_000_000); *// Pre-allocate capacity*
        
        for (int i = 0; i < 10_000_000; i++) {
            *// No resizing needed*
            list.add(i);
        }
        
        long efficientTime = System.currentTimeMillis() - startTime;
        System.out.println("Pre-sized ArrayList: " + efficientTime + " ms, Size: " + list.size());
        System.out.println("Improvement: " + (inefficientTime / (double)efficientTime) + "x");
    }
}
```

**Common Causes of Excessive GC:**

1. **High Allocation Rate**: Creating too many objects too quickly
2. **Short-lived Temporary Objects**: Especially in tight loops
3. **Autoboxing/Unboxing**: Implicit conversion between primitives and wrappers
4. **String Concatenation**: Using + operator in loops
5. **Collection Resizing**: Not pre-sizing collections appropriately
6. **Excessive Logging**: Especially with complex string formatting
7. **Inefficient Data Structures**: Using structures that create unnecessary objects

**Detection Methods:**

- GC log analysis (frequency and duration of collections)
- Allocation profiling to find hotspots
- Monitoring GC overhead percentage
- Observing application pause patterns

**Mitigation Strategies:**

1. **Reduce Allocation Rate**: Minimize object creation in hot paths
2. **Object Pooling**: Reuse expensive objects
3. **Use Primitives**: Avoid autoboxing where possible
4. **StringBuilder**: Use for string concatenation in loops
5. **Properly Size Collections**: Pre-allocate with expected capacity
6. **Reduce Logging Overhead**: Use isLoggable() checks
7. **Tune Generation Sizes**: Adjust young/old generation balance

> **Deep Dive Tip:** One subtle cause of excessive GC is "premature promotion"—short-lived objects being promoted to the old generation due to survivor space overflow or high allocation rates. This causes more frequent and expensive full GC cycles. To detect this pattern, look for high promotion rates in GC logs and high allocation rates in profiler data. Potential solutions include increasing survivor space size, adjusting tenuring threshold, or reducing allocation rate in critical sections. Some GC implementations like G1GC allow tuning the "MaxGCPauseMillis" target to balance throughput against pause times based on application needs.
> 

> **Interviewer Insight:** When discussing excessive GC, a strong candidate should emphasize the importance of measurement and targeted optimization. They should recognize that GC tuning alone rarely solves performance issues—reducing unnecessary allocations is often more effective. Ask them about their approach to diagnosing GC issues, looking for a methodology that:
> 
> 1. Starts with enabling appropriate GC logging
> 2. Analyzes logs to identify patterns (frequency, duration, promotion rates)
> 3. Uses profiling to identify allocation hotspots
> 4. Makes targeted code changes to reduce allocations
> 5. Tunes GC parameters based on observed behavior
> 
> A good candidate should also understand the trade-offs involved in different optimization strategies. For example, object pooling reduces GC pressure but increases code complexity and can cause memory leaks if not implemented carefully.
> 

### Slow I/O Operations

I/O operations are often the bottleneck in Java applications, affecting overall performance.

```java
*// Examples of inefficient and efficient I/O operations*
import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;

public class IOPerformanceExample {
    private static final int BUFFER_SIZE = 8192;
    private static final String TEST_FILE = "test_data.txt";
    private static final int FILE_SIZE_MB = 10;
    
    public static void main(String[] args) throws IOException {
        *// Create test file if it doesn't exist*
        createTestFile(TEST_FILE, FILE_SIZE_MB);
        
        *// Compare different file reading methods*
        compareFileReading();
        
        *// Compare different file writing methods*
        compareFileWriting();
        
        *// Demonstrate common I/O mistakes*
        demonstrateIOPitfalls();
    }
    
    *// Create a test file of specified size*
    private static void createTestFile(String fileName, int sizeMB) throws IOException {
        File file = new File(fileName);
        
        if (file.exists() && file.length() >= sizeMB * 1024 * 1024) {
            System.out.println("Test file already exists with sufficient size");
            return;
        }
        
        System.out.println("Creating test file: " + fileName + " (" + sizeMB + " MB)");
        
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
            *// Create data to write*
            char[] data = new char[1024];
            for (int i = 0; i < data.length; i++) {
                data[i] = (char) ('A' + (i % 26));
            }
            
            *// Write data to file*
            int kilobytesWritten = 0;
            int target = sizeMB * 1024; *// Convert MB to KB*
            
            while (kilobytesWritten < target) {
                writer.write(data);
                kilobytesWritten++;
                
                if (kilobytesWritten % 1024 == 0) {
                    System.out.println("Wrote " + (kilobytesWritten / 1024) + " MB");
                }
            }
        }
        
        System.out.println("Test file created: " + file.length() / (1024 * 1024) + " MB");
    }
    
    *// Compare different file reading methods*
    private static void compareFileReading() throws IOException {
        System.out.println("\n=== File Reading Performance Comparison ===");
        
        *// Method 1: FileInputStream without buffering*
        long startTime = System.currentTimeMillis();
        long bytesRead = readWithFileInputStream(TEST_FILE);
        long fileInputStreamTime = System.currentTimeMillis() - startTime;
        System.out.println("FileInputStream: " + fileInputStreamTime + " ms, Read: " + formatBytes(bytesRead));
        
        *// Method 2: BufferedInputStream*
        startTime = System.currentTimeMillis();
        bytesRead = readWithBufferedInputStream(TEST_FILE);
        long bufferedInputTime = System.currentTimeMillis() - startTime;
        System.out.println("BufferedInputStream: " + bufferedInputTime + " ms, Read: " + formatBytes(bytesRead));
        
        *// Method 3: FileChannel with ByteBuffer*
        startTime = System.currentTimeMillis();
        bytesRead = readWithFileChannel(TEST_FILE);
        long fileChannelTime = System.currentTimeMillis() - startTime;
        System.out.println("FileChannel: " + fileChannelTime + " ms, Read: " + formatBytes(bytesRead));
        
        *// Method 4: Files.readAllBytes*
        startTime = System.currentTimeMillis();
        bytesRead = readWithFilesAPI(TEST_FILE);
        long filesAPITime = System.currentTimeMillis() - startTime;
        System.out.println("Files.readAllBytes: " + filesAPITime + " ms, Read: " + formatBytes(bytesRead));
        
        *// Method 5: BufferedReader (line by line)*
        startTime = System.currentTimeMillis();
        bytesRead = readWithBufferedReader(TEST_FILE);
        long bufferedReaderTime = System.currentTimeMillis() - startTime;
        System.out.println("BufferedReader: " + bufferedReaderTime + " ms, Read: " + formatBytes(bytesRead));
    }
    
    *// Method 1: FileInputStream without buffering*
    private static long readWithFileInputStream(String fileName) throws IOException {
        long total = 0;
        try (FileInputStream fis = new FileInputStream(fileName)) {
            int b;
            *// Read one byte at a time - very inefficient*
            while ((b = fis.read()) != -1) {
                total++;
            }
        }
        return total;
    }
    
    *// Method 2: BufferedInputStream*
    private static long readWithBufferedInputStream(String fileName) throws IOException {
        long total = 0;
        byte[] buffer = new byte[BUFFER_SIZE];
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(fileName))) {
            int bytesRead;
            while ((bytesRead = bis.read(buffer)) != -1) {
                total += bytesRead;
            }
        }
        return total;
    }
    
    *// Method 3: FileChannel with ByteBuffer*
    private static long readWithFileChannel(String fileName) throws IOException {
        long total = 0;
        try (FileChannel channel = FileChannel.open(Paths.get(fileName), StandardOpenOption.READ)) {
            ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
            int bytesRead;
            while ((bytesRead = channel.read(buffer)) != -1) {
                total += bytesRead;
                buffer.clear();
            }
        }
        return total;
    }
    
    *// Method 4: Files.readAllBytes*
    private static long readWithFilesAPI(String fileName) throws IOException {
        byte[] allBytes = Files.readAllBytes(Paths.get(fileName));
        return allBytes.length;
    }
    
    *// Method 5: BufferedReader (line by line)*
    private static long readWithBufferedReader(String fileName) throws IOException {
        long total = 0;
        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = reader.readLine()) != null) {
                total += line.length();
            }
        }
        return total;
    }
    
    *// Compare different file writing methods*
    private static void compareFileWriting() throws IOException {
        System.out.println("\n=== File Writing Performance Comparison ===");
        
        *// Prepare data to write*
        byte[] data = new byte[1024 * 1024]; *// 1MB*
        for (int i = 0; i < data.length; i++) {
            data[i] = (byte) (i % 256);
        }
        
        *// Method 1: FileOutputStream without buffering*
        String file1 = "test_write_1.dat";
        long startTime = System.currentTimeMillis();
        long bytesWritten = writeWithFileOutputStream(file1, data);
        long fileOutputTime = System.currentTimeMillis() - startTime;
        System.out.println("FileOutputStream: " + fileOutputTime + " ms, Wrote: " + formatBytes(bytesWritten));
        
        *// Method 2: BufferedOutputStream*
        String file2 = "test_write_2.dat";
        startTime = System.currentTimeMillis();
        bytesWritten = writeWithBufferedOutputStream(file2, data);
        long bufferedOutputTime = System.currentTimeMillis() - startTime;
        System.out.println("BufferedOutputStream: " + bufferedOutputTime + " ms, Wrote: " + formatBytes(bytesWritten));
        
        *// Method 3: FileChannel with ByteBuffer*
        String file3 = "test_write_3.dat";
        startTime = System.currentTimeMillis();
        bytesWritten = writeWithFileChannel(file3, data);
        long fileChannelTime = System.currentTimeMillis() - startTime;
        System.out.println("FileChannel: " + fileChannelTime + " ms, Wrote: " + formatBytes(bytesWritten));
        
        *// Method 4: Files.write*
        String file4 = "test_write_4.dat";
        startTime = System.currentTimeMillis();
        bytesWritten = writeWithFilesAPI(file4, data);
        long filesAPITime = System.currentTimeMillis() - startTime;
        System.out.println("Files.write: " + filesAPITime + " ms, Wrote: " + formatBytes(bytesWritten));
        
        *// Clean up test files*
        new File(file1).delete();
        new File(file2).delete();
        new File(file3).delete();
        new File(file4).delete();
    }
    
    *// Method 1: FileOutputStream without buffering*
    private static long writeWithFileOutputStream(String fileName, byte[] data) throws IOException {
        long total = 0;
        try (FileOutputStream fos = new FileOutputStream(fileName)) {
            *// Write one byte at a time - very inefficient*
            for (int i = 0; i < data.length; i++) {
                fos.write(data[i]);
                total++;
            }
        }
        return total;
    }
    
    *// Method 2: BufferedOutputStream*
    private static long writeWithBufferedOutputStream(String fileName, byte[] data) throws IOException {
        long total = 0;
        try (BufferedOutputStream bos = new BufferedOutputStream(
                new FileOutputStream(fileName), BUFFER_SIZE)) {
            
            *// Write in chunks*
            int chunkSize = 8192;
            for (int i = 0; i < data.length; i += chunkSize) {
                int length = Math.min(chunkSize, data.length - i);
                bos.write(data, i, length);
                total += length;
            }
        }
        return total;
    }
    
    *// Method 3: FileChannel with ByteBuffer*
    private static long writeWithFileChannel(String fileName, byte[] data) throws IOException {
        long total = 0;
        try (FileChannel channel = FileChannel.open(Paths.get(fileName),
                StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {
            
            ByteBuffer buffer = ByteBuffer.wrap(data);
            while (buffer.hasRemaining()) {
                int bytesWritten = channel.write(buffer);
                total += bytesWritten;
            }
        }
        return total;
    }
    
    *// Method 4: Files.write*
    private static long writeWithFilesAPI(String fileName, byte[] data) throws IOException {
        Files.write(Paths.get(fileName), data);
        return data.length;
    }
    
    *// Demonstrate common I/O pitfalls*
    private static void demonstrateIOPitfalls() throws IOException {
        System.out.println("\n=== Common I/O Pitfalls ===");
        
        *// Pitfall 1: Not closing resources (shown with try-with-resources for correct version)*
        System.out.println("\nPitfall 1: Not closing resources");
        long startTime = System.currentTimeMillis();
        
        try (FileInputStream fis = new FileInputStream(TEST_FILE);
             BufferedInputStream bis = new BufferedInputStream(fis)) {
            *// Properly closed with try-with-resources*
            byte[] buffer = new byte[8192];
            int bytesRead;
            long total = 0;
            while ((bytesRead = bis.read(buffer)) != -1) {
                total += bytesRead;
            }
            System.out.println("Read " + formatBytes(total) + " with proper resource management");
        }
        
        System.out.println("Time with proper closing: " + (System.currentTimeMillis() - startTime) + " ms");
        
        *// Pitfall 2: Small buffer sizes*
        System.out.println("\nPitfall 2: Small buffer sizes");
        
        *// Test with different buffer sizes*
        testBufferSize(TEST_FILE, 1);       *// Tiny buffer*
        testBufferSize(TEST_FILE, 1024);    *// 1KB buffer*
        testBufferSize(TEST_FILE, 8192);    *// 8KB buffer*
        testBufferSize(TEST_FILE, 32768);   *// 32KB buffer*
        
        *// Pitfall 3: Excessive flushing*
        System.out.println("\nPitfall 3: Excessive flushing");
        
        String flushFile = "flush_test.txt";
        
        *// With excessive flushing*
        startTime = System.currentTimeMillis();
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(flushFile))) {
            for (int i = 0; i < 100000; i++) {
                writer.write("Line " + i + "\n");
                writer.flush(); *// Excessive flushing*
            }
        }
        long flushTime = System.currentTimeMillis() - startTime;
        System.out.println("Time with excessive flushing: " + flushTime + " ms");
        
        *// Without excessive flushing*
        startTime = System.currentTimeMillis();
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(flushFile))) {
            for (int i = 0; i < 100000; i++) {
                writer.write("Line " + i + "\n");
                *// No flush here - let buffering work*
            }
            *// Flush happens automatically on close*
        }
        long noFlushTime = System.currentTimeMillis() - startTime;
        System.out.println("Time without excessive flushing: " + noFlushTime + " ms");
        System.out.println("Improvement: " + (flushTime / (double)noFlushTime) + "x");
        
        new File(flushFile).delete();
        
        *// Pitfall 4: Line-by-line processing for non-text files*
        System.out.println("\nPitfall 4: Inappropriate I/O method for file type");
        
        *// Create binary test file*
        String binaryFile = "binary_test.dat";
        try (FileOutputStream fos = new FileOutputStream(binaryFile)) {
            byte[] binaryData = new byte[1024 * 1024]; *// 1MB of binary data*
            for (int i = 0; i < binaryData.length; i++) {
                binaryData[i] = (byte) i;
            }
            fos.write(binaryData);
        }
        
        *// Reading binary with character streams - BAD*
        startTime = System.currentTimeMillis();
        try {
            try (BufferedReader reader = new BufferedReader(new FileReader(binaryFile))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    *// Process binary data as text - will corrupt data*
                }
            }
        } catch (Exception e) {
            System.out.println("Error reading binary as text: " + e.getMessage());
        }
        long wrongMethodTime = System.currentTimeMillis() - startTime;
        
        *// Reading binary with byte streams - GOOD*
        startTime = System.currentTimeMillis();
        try (InputStream is = new BufferedInputStream(new FileInputStream(binaryFile))) {
            byte[] buffer = new byte[8192];
            while (is.read(buffer) != -1) {
                *// Process binary data correctly*
            }
        }
        long rightMethodTime = System.currentTimeMillis() - startTime;
        
        System.out.println("Time with inappropriate method: " + wrongMethodTime + " ms");
        System.out.println("Time with appropriate method: " + rightMethodTime + " ms");
        
        new File(binaryFile).delete();
    }
    
    *// Test the impact of buffer size on performance*
    private static void testBufferSize(String fileName, int bufferSize) throws IOException {
        long startTime = System.currentTimeMillis();
        long total = 0;
        
        try (BufferedInputStream bis = new BufferedInputStream(
                new FileInputStream(fileName), bufferSize)) {
            
            byte[] buffer = new byte[bufferSize];
            int bytesRead;
            while ((bytesRead = bis.read(buffer)) != -1) {
                total += bytesRead;
            }
        }
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Buffer size " + bufferSize + " bytes: " + duration + 
                          " ms, Read: " + formatBytes(total));
    }
    
    *// Format bytes as KB, MB, etc.*
    private static String formatBytes(long bytes) {
        if (bytes < 1024) {
            return bytes + " bytes";
        } else if (bytes < 1024 * 1024) {
            return String.format("%.2f KB", bytes / 1024.0);
        } else if (bytes < 1024 * 1024 * 1024) {
            return String.format("%.2f MB", bytes / (1024.0 * 1024.0));
        } else {
            return String.format("%.2f GB", bytes / (1024.0 * 1024.0 * 1024.0));
        }
    }
}
```

**Common I/O Performance Issues:**

1. **Unbuffered I/O**: Using FileInputStream/FileOutputStream directly
2. **Small Buffer Sizes**: Insufficient buffer size for the workload
3. **Byte-by-Byte Processing**: Reading/writing one byte at a time
4. **Excessive Flushing**: Calling flush() too frequently
5. **Resource Leaks**: Not closing streams properly
6. **Inappropriate I/O Method**: Using character streams for binary data
7. **Random Access in Sequential Files**: Seeking in files that should be read sequentially
8. **Inefficient Text Processing**: Using readLine() for large files

**Optimization Strategies:**

1. **Use Buffered Streams**: BufferedInputStream/BufferedOutputStream
2. **Appropriate Buffer Sizes**: 8KB-64KB depending on workload
3. **NIO Channels and Buffers**: For high-performance I/O
4. **Memory-Mapped Files**: For very large files
5. **Proper Resource Management**: try-with-resources
6. **Batch Processing**: Process data in chunks
7. **Asynchronous I/O**: For non-blocking operations
8. **Parallel I/O**: Multiple threads for independent I/O operations

> **Deep Dive Tip:** For highly optimized file I/O, consider using direct ByteBuffers with FileChannel. Direct buffers are allocated outside the Java heap in native memory, potentially improving I/O performance by reducing copying between JVM and native memory. However, direct buffers have higher allocation/deallocation costs, so they're most beneficial for long-lived buffers used for multiple operations. For maximum performance with large files, memory-mapped files (FileChannel.map) can be extremely efficient, as they leverage the operating system's virtual memory system to map file contents directly into memory.
> 

> **Interviewer Insight:** When discussing I/O performance, a strong candidate should understand the layered nature of I/O operations—from application code through JVM, operating system, filesystem, to hardware. They should recognize that I/O optimization requires understanding bottlenecks at each layer. Ask them about strategies for efficient I/O in different scenarios, such as:
> 
> 1. Reading/writing large files
> 2. Processing streams of data
> 3. Random access to structured files
> 4. Handling many small files
> 
> A good candidate will emphasize the importance of measurement rather than premature optimization, recognizing that I/O performance can vary significantly based on hardware, operating system, and workload characteristics. They should also understand modern approaches like asynchronous I/O and reactive streams for scalable I/O handling in high-concurrency environments.
> 

### Database Performance Issues

Database interactions are often a major bottleneck in enterprise applications.

```java
// Example demonstrating database performance issues
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class DatabasePerformanceExample {
    // For demonstration, we'll use H2 in-memory database
    private static final String DB_URL = "jdbc:h2:mem:testdb";
    private static final String DB_USER = "sa";
    private static final String DB_PASSWORD = "";
    
    public static void main(String[] args) throws Exception {
        // Initialize database
        initializeDatabase();
        
        // Demonstrate common database performance issues
        
        // 1. Inefficient queries
        demonstrateIneffientQueries();
        
        // 2. N+1 query problem
        demonstrateNPlusOneQuery();
        
        // 3. Connection management issues
        demonstrateConnectionManagement();
        
        // 4. Batch processing
        demonstrateBatchProcessing();
        
        // 5. Statement vs PreparedStatement
        demonstratePreparedStatements();
        
        // 6. Result set fetching
        demonstrateResultSetFetching();
    }
    
    // Initialize the database with test data
    private static void initializeDatabase() throws SQLException {
        try (Connection conn = getConnection()) {
            // Create tables
            try (Statement stmt = conn.createStatement()) {
                // Create departments table
                stmt.execute("CREATE TABLE departments (" +
                           "id INT PRIMARY KEY, " +
                           "name VARCHAR(50))");
                
                // Create employees table
                stmt.execute("CREATE TABLE employees (" +
                       "id INT PRIMARY KEY, " +
                       "name VARCHAR(100), " +
                       "department_id INT, " +
                       "salary DECIMAL(10, 2), " +
                       "hire_date DATE, " +
                       "FOREIGN KEY (department_id) REFERENCES departments(id))");
            
            // Insert sample departments
            stmt.execute("INSERT INTO departments VALUES (1, 'Engineering')");
            stmt.execute("INSERT INTO departments VALUES (2, 'Marketing')");
            stmt.execute("INSERT INTO departments VALUES (3, 'Finance')");
            stmt.execute("INSERT INTO departments VALUES (4, 'Human Resources')");
            stmt.execute("INSERT INTO departments VALUES (5, 'Sales')");
            
            // Insert sample employees
            for (int i = 1; i <= 1000; i++) {
                int deptId = 1 + (i % 5); // Distribute across 5 departments
                double salary = 50000 + (i % 10) * 5000;
                stmt.execute("INSERT INTO employees VALUES (" + i + ", 'Employee " + i + 
                           "', " + deptId + ", " + salary + ", CURRENT_DATE - " + (i % 3650) + ")");
            }
            
            // Create index on department_id
            stmt.execute("CREATE INDEX idx_dept_id ON employees(department_id)");
        }
        
        System.out.println("Database initialized with sample data");
    }
}

// Demonstrate inefficient queries and their optimized alternatives
private static void demonstrateIneffientQueries() throws SQLException {
    System.out.println("\n=== Inefficient vs. Optimized Queries ===");
    
    try (Connection conn = getConnection()) {
        // Example 1: SELECT * vs. specific columns
        System.out.println("\nExample 1: SELECT * vs. specific columns");
        
        // Inefficient: SELECT *
        long startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM employees WHERE department_id = 1")) {
            
            int count = 0;
            while (rs.next()) {
                // Only using name but retrieving all columns
                String name = rs.getString("name");
                count++;
            }
            System.out.println("Retrieved " + count + " employees with SELECT *");
        }
        long inefficientTime = System.nanoTime() - startTime;
        
        // Optimized: SELECT specific columns
        startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT name FROM employees WHERE department_id = 1")) {
            
            int count = 0;
            while (rs.next()) {
                // Only retrieving what we need
                String name = rs.getString("name");
                count++;
            }
            System.out.println("Retrieved " + count + " employees with SELECT name");
        }
        long optimizedTime = System.nanoTime() - startTime;
        
        System.out.println("SELECT * time: " + TimeUnit.NANOSECONDS.toMillis(inefficientTime) + " ms");
        System.out.println("SELECT name time: " + TimeUnit.NANOSECONDS.toMillis(optimizedTime) + " ms");
        System.out.println("Improvement: " + String.format("%.2f", (double)inefficientTime/optimizedTime) + "x");
        
        // Example 2: Inefficient JOINs vs. optimized
        System.out.println("\nExample 2: Inefficient vs. optimized JOINs");
        
        // Inefficient: Using WHERE instead of JOIN
        startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT e.name, d.name " +
                 "FROM employees e, departments d " +
                 "WHERE e.department_id = d.id")) {
            
            int count = 0;
            while (rs.next()) {
                count++;
            }
            System.out.println("Retrieved " + count + " employee-department pairs with implicit join");
        }
        inefficientTime = System.nanoTime() - startTime;
        
        // Optimized: Using explicit JOIN
        startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT e.name, d.name " +
                 "FROM employees e " +
                 "JOIN departments d ON e.department_id = d.id")) {
            
            int count = 0;
            while (rs.next()) {
                count++;
            }
            System.out.println("Retrieved " + count + " employee-department pairs with explicit join");
        }
        optimizedTime = System.nanoTime() - startTime;
        
        System.out.println("Implicit join time: " + TimeUnit.NANOSECONDS.toMillis(inefficientTime) + " ms");
        System.out.println("Explicit join time: " + TimeUnit.NANOSECONDS.toMillis(optimizedTime) + " ms");
        System.out.println("Improvement: " + String.format("%.2f", (double)inefficientTime/optimizedTime) + "x");
        
        // Example 3: Inefficient filter vs. using index
        System.out.println("\nExample 3: Non-indexed vs. indexed column search");
        
        // Inefficient: Function on column prevents index use
        startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT id, name FROM employees WHERE UPPER(name) LIKE 'EMPLOYEE 5%'")) {
            
            int count = 0;
            while (rs.next()) {
                count++;
            }
            System.out.println("Retrieved " + count + " employees with function on column");
        }
        inefficientTime = System.nanoTime() - startTime;
        
        // Optimized: Allow index use
        startTime = System.nanoTime();
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT id, name FROM employees WHERE name LIKE 'Employee 5%'")) {
            
            int count = 0;
            while (rs.next()) {
                count++;
            }
            System.out.println("Retrieved " + count + " employees without function on column");
        }
        optimizedTime = System.nanoTime() - startTime;
        
        System.out.println("Function on column time: " + TimeUnit.NANOSECONDS.toMillis(inefficientTime) + " ms");
        System.out.println("Without function time: " + TimeUnit.NANOSECONDS.toMillis(optimizedTime) + " ms");
        System.out.println("Improvement: " + String.format("%.2f", (double)inefficientTime/optimizedTime) + "x");
    }
}

// Demonstrate the N+1 query problem and solution
private static void demonstrateNPlusOneQuery() throws SQLException {
    System.out.println("\n=== N+1 Query Problem ===");
    
    // Problem: For each department, we fetch employees in a separate query
    long startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        List<Department> departments = new ArrayList<>();
        
        // First query: Get all departments
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT id, name FROM departments")) {
            
            while (rs.next()) {
                Department dept = new Department();
                dept.id = rs.getInt("id");
                dept.name = rs.getString("name");
                departments.add(dept);
            }
        }
        
        // N additional queries: For each department, get its employees
        for (Department dept : departments) {
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery(
                     "SELECT id, name FROM employees WHERE department_id = " + dept.id)) {
                
                while (rs.next()) {
                    Employee emp = new Employee();
                    emp.id = rs.getInt("id");
                    emp.name = rs.getString("name");
                    dept.employees.add(emp);
                }
            }
        }
        
        System.out.println("N+1 query approach: " + departments.size() + 
                         " departments with " + getTotalEmployees(departments) + " total employees");
    }
    
    long nPlusOneTime = System.nanoTime() - startTime;
    
    // Solution: Use a JOIN and group the results in application code
    startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        List<Department> departments = new ArrayList<>();
        
        // Single query with JOIN
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(
                 "SELECT d.id as dept_id, d.name as dept_name, " +
                 "e.id as emp_id, e.name as emp_name " +
                 "FROM departments d " +
                 "LEFT JOIN employees e ON d.id = e.department_id " +
                 "ORDER BY d.id")) {
            
            Department currentDept = null;
            int currentDeptId = -1;
            
            while (rs.next()) {
                int deptId = rs.getInt("dept_id");
                
                // If we've moved to a new department
                if (deptId != currentDeptId) {
                    currentDept = new Department();
                    currentDept.id = deptId;
                    currentDept.name = rs.getString("dept_name");
                    departments.add(currentDept);
                    currentDeptId = deptId;
                }
                
                // Add employee to current department if exists
                if (!rs.wasNull()) {
                    int empId = rs.getInt("emp_id");
                    if (!rs.wasNull()) {
                        Employee emp = new Employee();
                        emp.id = empId;
                        emp.name = rs.getString("emp_name");
                        currentDept.employees.add(emp);
                    }
                }
            }
        }
        
        System.out.println("Single query approach: " + departments.size() + 
                         " departments with " + getTotalEmployees(departments) + " total employees");
    }
    
    long singleQueryTime = System.nanoTime() - startTime;
    
    System.out.println("N+1 query time: " + TimeUnit.NANOSECONDS.toMillis(nPlusOneTime) + " ms");
    System.out.println("Single query time: " + TimeUnit.NANOSECONDS.toMillis(singleQueryTime) + " ms");
    System.out.println("Improvement: " + String.format("%.2f", (double)nPlusOneTime/singleQueryTime) + "x");
}

// Demonstrate connection management issues
private static void demonstrateConnectionManagement() throws SQLException {
    System.out.println("\n=== Connection Management ===");
    
    // Problem: Opening and closing connections for each operation
    System.out.println("\nProblem: Individual connections for each operation");
    long startTime = System.nanoTime();
    
    for (int i = 0; i < 20; i++) {
        // Each operation gets its own connection
        Connection conn = getConnection();
        try {
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM employees")) {
                
                if (rs.next()) {
                    int count = rs.getInt(1);
                    if (i == 0) {
                        System.out.println("Employee count: " + count);
                    }
                }
            }
        } finally {
            conn.close(); // Close connection after each operation
        }
    }
    
    long inefficientTime = System.nanoTime() - startTime;
    
    // Solution: Reuse connection for multiple operations
    System.out.println("\nSolution: Reuse connection for multiple operations");
    startTime = System.nanoTime();
    
    Connection conn = getConnection();
    try {
        for (int i = 0; i < 20; i++) {
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM employees")) {
                
                if (rs.next()) {
                    int count = rs.getInt(1);
                    if (i == 0) {
                        System.out.println("Employee count: " + count);
                    }
                }
            }
        }
    } finally {
        conn.close(); // Close once at the end
    }
    
    long efficientTime = System.nanoTime() - startTime;
    
    System.out.println("Individual connections time: " + TimeUnit.NANOSECONDS.toMillis(inefficientTime) + " ms");
    System.out.println("Reused connection time: " + TimeUnit.NANOSECONDS.toMillis(efficientTime) + " ms");
    System.out.println("Improvement: " + String.format("%.2f", (double)inefficientTime/efficientTime) + "x");
    
    // Note about connection pooling
    System.out.println("\nConnection pooling would provide even greater benefits in production environments");
}

// Demonstrate batch processing for better performance
private static void demonstrateBatchProcessing() throws SQLException {
    System.out.println("\n=== Batch Processing ===");
    
    // First, create a test table
    try (Connection conn = getConnection()) {
        try (Statement stmt = conn.createStatement()) {
            stmt.execute("CREATE TABLE batch_test (id INT, name VARCHAR(100))");
        }
    }
    
    // Problem: Individual inserts
    System.out.println("\nProblem: Individual INSERT statements");
    long startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        try (Statement stmt = conn.createStatement()) {
            for (int i = 1; i <= 1000; i++) {
                stmt.executeUpdate("INSERT INTO batch_test VALUES (" + i + ", 'Test " + i + "')");
            }
        }
    }
    
    long individualTime = System.nanoTime() - startTime;
    
    // Solution: Batch inserts
    System.out.println("\nSolution: Batched INSERT statements");
    
    // First clear the table
    try (Connection conn = getConnection()) {
        try (Statement stmt = conn.createStatement()) {
            stmt.execute("DELETE FROM batch_test");
        }
    }
    
    startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        // Disable auto-commit for batch operations
        conn.setAutoCommit(false);
        
        try (Statement stmt = conn.createStatement()) {
            for (int i = 1; i <= 1000; i++) {
                stmt.addBatch("INSERT INTO batch_test VALUES (" + i + ", 'Test " + i + "')");
                
                if (i % 100 == 0) {
                    stmt.executeBatch();
                }
            }
            
            // Execute any remaining statements
            stmt.executeBatch();
            
            // Commit the transaction
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
    
    long batchTime = System.nanoTime() - startTime;
    
    System.out.println("Individual inserts time: " + TimeUnit.NANOSECONDS.toMillis(individualTime) + " ms");
    System.out.println("Batch inserts time: " + TimeUnit.NANOSECONDS.toMillis(batchTime) + " ms");
    System.out.println("Improvement: " + String.format("%.2f", (double)individualTime/batchTime) + "x");
}

// Demonstrate PreparedStatement vs. Statement
private static void demonstratePreparedStatements() throws SQLException {
    System.out.println("\n=== PreparedStatement vs. Statement ===");
    
    // Problem: Using Statement with string concatenation
    System.out.println("\nProblem: Using Statement with string concatenation");
    long startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        // Execute the same query with different parameters many times
        for (int i = 1; i <= 5; i++) {
            for (int j = 0; j < 100; j++) {
                try (Statement stmt = conn.createStatement();
                     ResultSet rs = stmt.executeQuery(
                         "SELECT COUNT(*) FROM employees WHERE department_id = " + i)) {
                    
                    if (rs.next()) {
                        int count = rs.getInt(1);
                        if (j == 0) {
                            System.out.println("Department " + i + " has " + count + " employees");
                        }
                    }
                }
            }
        }
    }
    
    long statementTime = System.nanoTime() - startTime;
    
    // Solution: Using PreparedStatement
    System.out.println("\nSolution: Using PreparedStatement");
    startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        // Prepare the statement once
        try (PreparedStatement pstmt = conn.prepareStatement(
                 "SELECT COUNT(*) FROM employees WHERE department_id = ?")) {
            
            // Execute with different parameters
            for (int i = 1; i <= 5; i++) {
                pstmt.setInt(1, i);
                
                for (int j = 0; j < 100; j++) {
                    try (ResultSet rs = pstmt.executeQuery()) {
                        if (rs.next()) {
                            int count = rs.getInt(1);
                            if (j == 0) {
                                System.out.println("Department " + i + " has " + count + " employees");
                            }
                        }
                    }
                }
            }
        }
    }
    
    long preparedTime = System.nanoTime() - startTime;
    
    System.out.println("Statement time: " + TimeUnit.NANOSECONDS.toMillis(statementTime) + " ms");
    System.out.println("PreparedStatement time: " + TimeUnit.NANOSECONDS.toMillis(preparedTime) + " ms");
    System.out.println("Improvement: " + String.format("%.2f", (double)statementTime/preparedTime) + "x");
}

// Demonstrate ResultSet fetching options
private static void demonstrateResultSetFetching() throws SQLException {
    System.out.println("\n=== ResultSet Fetching ===");
    
    // Problem: Default fetch size
    System.out.println("\nProblem: Default fetch size");
    long startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        try (Statement stmt = conn.createStatement()) {
            // Using default fetch size
            try (ResultSet rs = stmt.executeQuery("SELECT * FROM employees")) {
                int count = 0;
                while (rs.next()) {
                    count++;
                }
                System.out.println("Retrieved " + count + " employees with default fetch size");
            }
        }
    }
    
    long defaultTime = System.nanoTime() - startTime;
    
    // Solution: Optimized fetch size
    System.out.println("\nSolution: Optimized fetch size");
    startTime = System.nanoTime();
    
    try (Connection conn = getConnection()) {
        try (Statement stmt = conn.createStatement()) {
            // Set an optimized fetch size
            stmt.setFetchSize(100);
            
            try (ResultSet rs = stmt.executeQuery("SELECT * FROM employees")) {
                int count = 0;
                while (rs.next()) {
                    count++;
                }
                System.out.println("Retrieved " + count + " employees with optimized fetch size");
            }
        }
    }
    
    long optimizedTime = System.nanoTime() - startTime;
    
    System.out.println("Default fetch size time: " + TimeUnit.NANOSECONDS.toMillis(defaultTime) + " ms");
    System.out.println("Optimized fetch size time: " + TimeUnit.NANOSECONDS.toMillis(optimizedTime) + " ms");
    System.out.println("Improvement: " + String.format("%.2f", (double)defaultTime/optimizedTime) + "x");
}

// Helper method to get a database connection
private static Connection getConnection() throws SQLException {
    return DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
}

// Helper method to count total employees across departments
private static int getTotalEmployees(List<Department> departments) {
    int total = 0;
    for (Department dept : departments) {
        total += dept.employees.size();
    }
    return total;
}

// Simple Department class for demonstrations
static class Department {
    int id;
    String name;
    List<Employee> employees = new ArrayList<>();
}

// Simple Employee class for demonstrations
static class Employee {
    int id;
    String name;
}
}
```

**Common Database Performance Issues:**

1. **Inefficient Queries**: Selecting unnecessary columns, improper joins
2. **N+1 Query Problem**: Executing a query for each result of a previous query
3. **Connection Management**: Creating/closing connections frequently
4. **Lack of Batch Processing**: Individual statements instead of batching
5. **Statement vs. PreparedStatement**: Not using prepared statements for repeated queries
6. **Inefficient ResultSet Handling**: Default fetch size, not scrolling efficiently
7. **Missing Indexes**: Not indexing frequently queried or joined columns
8. **Improper Transaction Management**: Transaction scope too small or too large

**Optimization Strategies:**

1. **Query Optimization**: Select only needed columns, use proper joins
2. **Join Fetching**: Use joins instead of multiple queries
3. **Connection Pooling**: Reuse database connections
4. **Batch Processing**: Execute multiple operations in a batch
5. **Prepared Statements**: Use for repeated queries with different parameters
6. **Optimized Fetch Size**: Adjust based on result set size
7. **Proper Indexing**: Index columns used in WHERE, JOIN, and ORDER BY
8. **Transaction Management**: Right-size transactions for operation type

> **Deep Dive Tip:** Database performance optimization requires understanding both the database engine and the JDBC API. For example, the fetch size parameter controls how many rows the JDBC driver retrieves at once from the database. A small fetch size means more network roundtrips but less memory usage, while a large fetch size reduces roundtrips but increases memory consumption. The optimal value depends on row size, network latency, and available memory. Similarly, connection pooling configurations should consider both the application's concurrency needs and the database's connection limits—too small a pool causes queuing, while too large a pool wastes resources and can overload the database.
> 

> **Interviewer Insight:** When discussing database performance issues, a strong candidate should demonstrate knowledge of both database concepts and Java-specific optimizations. They should understand the interplay between application design and database design—for example, how entity relationships in the domain model influence query patterns and index design. Ask them about their approach to diagnosing and resolving database performance issues, looking for a methodology that:
> 
> 1. Identifies problematic queries through monitoring and profiling
> 2. Analyzes execution plans to understand query performance
> 3. Considers both schema-level optimizations (indexes, denormalization) and code-level optimizations (query rewriting, caching)
> 4. Balances performance against maintainability and data integrity
> 
> They should also understand modern approaches like database sharding, read replicas, and CQRS (Command Query Responsibility Segregation) for scaling database performance in high-load applications.
> 

## Summary of JVM Performance Optimization

Optimizing Java application performance requires understanding and addressing issues at multiple levels:

### Memory Management

- Right-size the heap and its generations
- Minimize unnecessary object creation
- Use appropriate collection types and sizes
- Consider object pooling for expensive resources
- Address memory leaks through proper resource management
- Monitor and tune garbage collection

### Concurrency

- Choose appropriate synchronization mechanisms
- Minimize lock contention through finer-grained locking
- Use thread pools and executors rather than raw threads
- Prevent deadlocks through consistent lock ordering
- Leverage thread-local processing where appropriate
- Consider newer concurrency models (e.g., CompletableFuture, virtual threads)

### I/O and Network

- Use buffered streams and appropriate buffer sizes
- Leverage NIO for high-performance I/O
- Batch operations to reduce overhead
- Close resources properly using try-with-resources
- Consider asynchronous I/O for non-blocking operations
- Optimize serialization/deserialization

### Database Access

- Write efficient queries that use indexes
- Use connection pooling
- Leverage prepared statements
- Batch database operations
- Set appropriate fetch sizes
- Consider caching strategies
- Right-size transactions

### Profiling and Monitoring

- Use JVM profiling tools to identify bottlenecks
- Monitor memory usage and garbage collection
- Use JMH for microbenchmarking
- Implement application-level metrics
- Establish performance baselines and test regularly

> **Deep Dive Tip:** Performance optimization is both an art and a science. While tools and metrics provide data about what's happening, interpreting that data requires understanding the underlying systems and making appropriate trade-offs. For example, optimizing for throughput often increases latency variance, while optimizing for consistent latency may reduce overall throughput. Similarly, memory optimizations that reduce garbage collection pauses might increase CPU usage. The key is to optimize for the metrics that matter most for your specific application type and user experience requirements.
> 

> **Interviewer Insight:** When discussing performance optimization, a strong candidate should emphasize the importance of a systematic, data-driven approach rather than premature or speculative optimization. They should recognize that performance is a feature that competes with other features for development resources, and optimizations should be prioritized based on their impact on user experience and business metrics. Ask them about their process for addressing performance issues, looking for an approach that:
> 
> 1. Establishes clear, measurable performance goals
> 2. Identifies bottlenecks through profiling and monitoring
> 3. Makes targeted improvements based on data
> 4. Measures the impact of changes
> 5. Iterates until goals are met
> 
> A good candidate will also understand the importance of designing for performance from the start—making architectural choices that enable scaling and optimization—while avoiding premature micro-optimizations that complicate code without proven benefits.
> 

---

## ✈️ Happy Coding!

---