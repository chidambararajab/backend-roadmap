# Module 9: Java 9-21+ Features

## 9.1 Java 9 Features

Java 9, released in September 2017, introduced several significant features and enhancements, with the module system being the most transformative change.

### Module System (JPMS)

The Java Platform Module System (JPMS) introduced a higher level of aggregation above packages, allowing developers to create more maintainable and secure applications.

```java
*// module-info.java in the root of your module*
module com.example.myapp {
    *// Dependencies (other modules this module requires)*
    requires java.base;        *// Implicitly required by all modules*
    requires java.logging;     *// Standard module dependency*
    requires transitive java.sql;  *// Makes java.sql available to modules requiring this module*
    
    *// Exported packages (public API of this module)*
    exports com.example.myapp.api;
    exports com.example.myapp.model to com.example.client;  *// Qualified export*
    
    *// Opening packages for reflection (for frameworks)*
    opens com.example.myapp.entity;
    opens com.example.myapp.config to com.example.framework;  *// Qualified opens*
    
    *// Service consumption and provision*
    uses com.example.service.MyService;  *// Service interface we consume*
    provides com.example.service.MyService with com.example.myapp.impl.MyServiceImpl;  *// Service we implement*
}
```

**Key Module Directives:**

- **requires**: Dependencies on other modules
- **exports**: Packages accessible to other modules
- **opens**: Packages open for reflection at runtime
- **provides...with**: Service provider implementation
- **uses**: Service dependency (interface)

**Module Types:**

- **Named Modules**: Explicit module with module-info.java
- **Automatic Modules**: JAR files on module path without module-info.java
- **Unnamed Modules**: Classes on classpath (for backward compatibility)

**Module Architecture Example:**

```java
Application
├── app.core (module)
│   ├── module-info.java
│   └── com.app.core.*
├── app.ui (module)
│   ├── module-info.java
│   └── com.app.ui.*
└── app.service (module)
    ├── module-info.java
    └── com.app.service.*
```

> **Deep Dive Tip:** Understanding module resolution is crucial for troubleshooting module-related issues. The module system follows a more restrictive resolution process than the classpath, enforcing explicit dependencies. When the JVM starts, it builds a module graph by resolving dependencies. This graph must be:
> 
> 1. **Complete**: All required modules must be present
> 2. **Acyclic**: No circular dependencies
> 3. **Readable**: A module can only read another module if it requires it (directly or indirectly)
> 
> When faced with "module not found" errors, check both the module declaration and the module path to ensure all dependencies are properly specified and available.
> 

> **Interviewer Insight:** When discussing JPMS, emphasize that it addresses longstanding issues with JAR hell and classpath limitations by providing explicit boundaries and dependencies. While adoption has been gradual due to migration challenges, modularization offers significant benefits for large applications: clearer architecture, better encapsulation, reliable configuration, and reduced startup time through custom runtime images. For legacy applications, focus on the incremental migration approach: first ensure compatibility with the module path (fixing illegal reflective access), then introduce modules at the edges, gradually working inward.
> 

### JShell (REPL)

Java 9 introduced an interactive shell for evaluating Java code snippets without the need for full program compilation.

```java
*# Starting JShell*
$ jshell

*# Basic expressions and variable declarations*
jshell> 2 + 2
$1 ==> 4

jshell> String greeting = "Hello, JShell!"
greeting ==> "Hello, JShell!"

jshell> greeting.toUpperCase()
$3 ==> "HELLO, JSHELL!"

*# Method declarations*
jshell> int factorial(int n) {
   ...>     if (n <= 1) return 1;
   ...>     return n * factorial(n - 1);
   ...> }
|  created method factorial(int)

jshell> factorial(5)
$5 ==> 120

*# Import statements*
jshell> import java.util.stream.*

jshell> Stream.of(1, 2, 3, 4, 5).map(n -> n * n).collect(Collectors.toList())
$7 ==> [1, 4, 9, 16, 25]

*# Class declarations*
jshell> class Person {
   ...>     private String name;
   ...>     private int age;
   ...>     
   ...>     public Person(String name, int age) {
   ...>         this.name = name;
   ...>         this.age = age;
   ...>     }
   ...>     
   ...>     public String toString() {
   ...>         return name + " (" + age + ")";
   ...>     }
   ...> }
|  created class Person

jshell> new Person("John", 30)
$9 ==> John (30)

*# JShell commands*
jshell> /list
   1 : 2 + 2
   2 : String greeting = "Hello, JShell!"
   3 : greeting.toUpperCase()
   4 : int factorial(int n) {
           if (n <= 1) return 1;
           return n * factorial(n - 1);
       }
   5 : factorial(5)
   6 : import java.util.stream.*
   7 : Stream.of(1, 2, 3, 4, 5).map(n -> n * n).collect(Collectors.toList())
   8 : class Person {
           private String name;
           private int age;
           
           public Person(String name, int age) {
               this.name = name;
               this.age = age;
           }
           
           public String toString() {
               return name + " (" + age + ")";
           }
       }
   9 : new Person("John", 30)

jshell> /vars
|    String greeting = "Hello, JShell!"
|    int $5 = 120
|    List<Integer> $7 = [1, 4, 9, 16, 25]
|    Person $9 = John (30)

jshell> /exit
```

**Key Features:**

- **Immediate Evaluation**: Execute Java code without compilation
- **Command History**: Access and reuse previous inputs
- **Tab Completion**: Autocomplete code and imports
- **Snippet Management**: View, save, and reload code snippets
- **Error Handling**: Helpful feedback for syntax errors

**Common Commands:**

- **/list**: Show all snippets
- **/vars**: Show all variables
- **/methods**: Show all methods
- **/imports**: Show all imports
- **/edit**: Edit snippets in external editor
- **/save file**: Save snippets to a file
- **/open file**: Load snippets from a file
- **/exit**: Exit JShell

> **Interviewer Insight:** JShell transforms the Java development experience by enabling rapid prototyping and exploration, which previously required external tools or excessive boilerplate. When discussing JShell in interviews, emphasize its utility for:
> 
> 1. Learning and experimentation (especially for API exploration)
> 2. Teaching Java fundamentals
> 3. Rapid prototyping of algorithms or API usage
> 4. Debugging complex expressions
> 5. Live demonstrations in presentations
> 
> While JShell isn't meant to replace IDEs for full application development, it complements them by providing a lightweight environment for testing isolated code fragments.
> 

### Language Enhancements

Java 9 introduced several smaller language improvements that made code more concise and expressive.

### Private Interface Methods

```java
*// Before Java 9, shared code in interfaces required duplication*
interface LegacyCalculator {
    default double calculateArea(double length, double width) {
        *// Code duplication in multiple default methods*
        if (length <= 0 || width <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
        return length * width;
    }
    
    default double calculateVolume(double length, double width, double height) {
        *// Duplicate validation code*
        if (length <= 0 || width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
        return length * width * height;
    }
}

*// With Java 9 private methods in interfaces*
interface ModernCalculator {
    *// Private helper method*
    private void validatePositive(double... dimensions) {
        for (double d : dimensions) {
            if (d <= 0) {
                throw new IllegalArgumentException("Dimensions must be positive");
            }
        }
    }
    
    *// Private static helper*
    private static double multiply(double... values) {
        double result = 1.0;
        for (double value : values) {
            result *= value;
        }
        return result;
    }
    
    *// Default methods using private helpers*
    default double calculateArea(double length, double width) {
        validatePositive(length, width);
        return multiply(length, width);
    }
    
    default double calculateVolume(double length, double width, double height) {
        validatePositive(length, width, height);
        return multiply(length, width, height);
    }
}
```

### Try-with-Resources Improvements

```java
*// Before Java 9*
void processFileOld() throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
    try (BufferedReader r = reader) {  *// Redeclaration required// Process file*
    }
}

*// With Java 9 (effectively final variables can be used directly)*
void processFileNew() throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
    try (reader) {  *// No redeclaration needed// Process file*
    }
    *// reader is closed here*
}
```

### Diamond Operator with Anonymous Classes

```java
*// Before Java 9*
Map<String, List<String>> mapOld = new HashMap<String, List<String>>() {
    *// Anonymous class with explicit type arguments*
    @Override
    public String toString() {
        return "Custom Map";
    }
};

*// With Java 9*
Map<String, List<String>> mapNew = new HashMap<>() {
    *// Diamond operator with anonymous class*
    @Override
    public String toString() {
        return "Custom Map";
    }
};
```

### Other Language Enhancements

- **@SafeVarargs on private methods**: Previously limited to final/static methods
- **Underscore no longer a valid identifier**: Preparing for future language features
- **StackWalking API**: Efficient access to stack traces with filtering capabilities

> **Interviewer Insight:** When discussing Java 9's language enhancements, highlight how they address practical pain points in everyday coding. Private interface methods filled a gap in interface default methods by enabling code reuse without exposing implementation details. The try-with-resources improvement eliminates boilerplate when working with existing resources. These changes demonstrate Java's evolution toward more concise, expressive code while maintaining backward compatibility. While individually small, together they significantly improve developer productivity and code maintainability.
> 

### Collection Factory Methods

Java 9 introduced convenient factory methods for creating small, immutable collections.

```java
*// Before Java 9 - verbose collection creation*
List<String> listOld = Collections.unmodifiableList(Arrays.asList("a", "b", "c"));
Set<String> setOld = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
Map<String, Integer> mapOld = Collections.unmodifiableMap(new HashMap<String, Integer>() {{
    put("a", 1);
    put("b", 2);
    put("c", 3);
}});

*// Java 9 factory methods - concise and immutable*
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of(
    "a", 1,
    "b", 2,
    "c", 3
);

*// For more than 10 map entries*
Map<String, Integer> largeMap = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2),
    Map.entry("c", 3),
    Map.entry("d", 4)
    *// ... more entries*
);

*// Attempting to modify the collections throws UnsupportedOperationException*
try {
    list.add("d");  *// Throws exception*
} catch (UnsupportedOperationException e) {
    System.out.println("Cannot modify immutable collection");
}

*// Null elements are not allowed*
try {
    List<String> withNull = List.of("a", null, "c");  *// Throws NullPointerException*
} catch (NullPointerException e) {
    System.out.println("Null elements not allowed");
}
```

**Key Features:**

- **Concise Creation**: One-line collection creation
- **Immutability**: Collections are unmodifiable
- **Null Rejection**: Null elements not allowed
- **Optimized Implementation**: Specialized implementations for small collections
- **Available For**: List, Set, Map, and Map.Entry

> **Deep Dive Tip:** The factory methods use specialized implementations optimized for small, immutable collections. For example, `List.of()` with 0-10 elements uses specialized classes `List0` through `List10` that are more memory-efficient than `ArrayList`. Similarly, `Set.of()` with 0-10 elements uses optimized implementations. Beyond performance, these immutable collections also provide stronger guarantees than `Collections.unmodifiableXXX()` wrappers—they ensure immutability at the implementation level rather than just wrapping a potentially mutable collection.
> 

> **Interviewer Insight:** When discussing collection factory methods, emphasize their role in promoting immutability as a design principle. Immutable collections are thread-safe, hashcode-cacheable, and prevent accidental modification—all valuable properties for robust software. The factory methods also eliminate common boilerplate, especially for test data and configuration. A good practice is to use them by default for small collections that don't need to be modified, reserving builders and traditional constructors for collections that require mutability or custom initialization logic.
> 

### Stream API Enhancements

Java 9 extended the Stream API with several useful methods for more expressive data processing.

```java
*// takeWhile() - takes elements while predicate is true*
List<Integer> numbers = Arrays.asList(2, 4, 6, 8, 9, 10, 12);
List<Integer> evenOnly = numbers.stream()
    .takeWhile(n -> n % 2 == 0)  *// Takes elements until the first odd number*
    .collect(Collectors.toList());
System.out.println(evenOnly);  *// [2, 4, 6, 8]// dropWhile() - drops elements while predicate is true*
List<String> lines = Arrays.asList("# Comment", "# Another comment", "First line", "Second line");
List<String> contentLines = lines.stream()
    .dropWhile(line -> line.startsWith("#"))  *// Drops comments at the beginning*
    .collect(Collectors.toList());
System.out.println(contentLines);  *// [First line, Second line]// ofNullable() - creates a stream with 0 or 1 elements*
String nullableValue = getSomeNullableValue();
Stream<String> stream = Stream.ofNullable(nullableValue);  *// Empty if null, single-element if non-null// Simplified null handling with flatMap*
List<String> values = getValuesWithNulls();
List<String> nonNullValues = values.stream()
    .flatMap(Stream::ofNullable)  *// Filters out nulls*
    .collect(Collectors.toList());

*// iterate() with predicate - finite streams with a termination condition// Generate even numbers < 100*
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n < 100, n -> n + 2);
System.out.println(evenNumbers.collect(Collectors.toList()));
*// [0, 2, 4, ..., 98]// Legacy iterate (infinite, requires limit)*
Stream<Integer> infiniteEvens = Stream.iterate(0, n -> n + 2).limit(50);
```

**Key Additions:**

- **takeWhile()**: Takes elements until predicate is false
- **dropWhile()**: Skips elements until predicate is false
- **ofNullable()**: Creates stream with 0-1 elements, handling nulls
- **iterate() with predicate**: Creates finite streams with termination condition

> **Deep Dive Tip:** The `takeWhile()` and `dropWhile()` operations differ from `filter()` in an important way: they operate on contiguous elements from the beginning of the stream. Once the predicate becomes false for an element, `takeWhile()` stops completely (even if later elements would match), and `dropWhile()` includes all remaining elements (even if they would not match). This makes them particularly useful for ordered data with recognizable transitions, such as log files with headers, sorted data with thresholds, or time-series data with phase changes.
> 

> **Interviewer Insight:** When discussing Stream API enhancements, highlight how they fill specific gaps in common data processing patterns. `takeWhile()` and `dropWhile()` are especially valuable for handling files and streams with distinct sections, such as CSV files with headers or configuration files with comments. The `ofNullable()` method, while simple, eliminates a common pattern of conditional stream creation for nullable elements. These additions show Java's commitment to making the Stream API more comprehensive based on real-world usage patterns, without sacrificing its elegant functional design.
> 

### Optional Enhancements

Java 9 extended the Optional class with methods to make chaining and defaulting more expressive.

```java
*// ifPresentOrElse() - handles both present and empty cases*
Optional<String> optional = Optional.of("Value");
optional.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);  *// Prints: Found: Value*

Optional<String> empty = Optional.empty();
empty.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);  *// Prints: Not found// or() - provides an alternative Optional if empty*
Optional<String> result = empty.or(() -> Optional.of("Default"));
System.out.println(result.get());  *// Default// Alternative chain - first non-empty result*
Optional<String> first = getUserInput();  *// Might be empty*
Optional<String> fallback = first.or(() -> getDefaultValue());  *// Try fallback if first is empty// stream() - converts Optional to Stream (0 or 1 elements)*
List<Optional<String>> optionals = Arrays.asList(
    Optional.empty(),
    Optional.of("one"),
    Optional.empty(),
    Optional.of("two")
);

*// Filter out empty Optionals and extract values*
List<String> filteredValues = optionals.stream()
    .flatMap(Optional::stream)  *// Empty Optionals become empty streams*
    .collect(Collectors.toList());
System.out.println(filteredValues);  *// [one, two]*
```

**Key Additions:**

- **ifPresentOrElse()**: Handles both present and empty cases
- **or()**: Returns alternative Optional if this one is empty
- **stream()**: Converts to Stream of 0-1 elements

> **Deep Dive Tip:** The `stream()` method is particularly powerful in combination with `flatMap()` for working with collections of Optionals, as shown in the example. This pattern simplifies many operations that previously required more verbose solutions. Another useful pattern is chaining multiple `or()` calls to create a prioritized sequence of alternatives, similar to a fallback chain:
> 
> 
> ```java
> Optional<User> user = findUserById(id)
>     .or(() -> findUserByEmail(email))
>     .or(() -> findUserByPhone(phone));
> ```
> 
> This creates a concise, readable way to express "try this, then that, then that" logic.
> 

> **Interviewer Insight:** When discussing Optional enhancements, emphasize how they promote more functional programming patterns in Java by reducing conditional logic and encouraging declarative approaches. The new methods align Optional more closely with the Stream API's design philosophy, enabling more fluent, expressive code for handling potentially absent values. This evolution makes Optional not just a container for nullability but a full-fledged monadic type that can be composed and transformed in powerful ways, similar to concepts from functional languages.
> 

### Process API Updates

Java 9 enhanced the Process API to provide better information about and control over system processes.

```java
*// Get current process information*
ProcessHandle current = ProcessHandle.current();
System.out.println("Current PID: " + current.pid());
System.out.println("Info: " + current.info());

*// Process information*
ProcessHandle.Info info = current.info();
info.command().ifPresent(cmd -> System.out.println("Command: " + cmd));
info.startInstant().ifPresent(start -> System.out.println("Start time: " + start));
info.totalCpuDuration().ifPresent(cpu -> System.out.println("CPU time: " + cpu));
info.user().ifPresent(user -> System.out.println("User: " + user));

*// List all processes*
ProcessHandle.allProcesses()
    .filter(ph -> ph.info().command().isPresent())
    .sorted(Comparator.comparing(ProcessHandle::pid))
    .limit(5)  *// Just show a few*
    .forEach(ph -> System.out.println(ph.pid() + " : " + 
                                     ph.info().command().orElse("Unknown")));

*// Process tree*
ProcessHandle.current().children()
    .forEach(child -> System.out.println("Child process: " + child.pid()));

*// Process destruction*
long pid = 12345;  *// Replace with actual PID*
Optional<ProcessHandle> processHandle = ProcessHandle.of(pid);
processHandle.ifPresent(handle -> {
    boolean terminated = handle.destroy();
    if (!terminated) {
        *// Force destroy if normal destroy doesn't work*
        boolean forceTerminated = handle.destroyForcibly();
        System.out.println("Force terminated: " + forceTerminated);
    }
});

*// Wait for process to exit*
processHandle.ifPresent(handle -> {
    handle.onExit().thenAccept(ph -> 
        System.out.println("Process " + ph.pid() + " has exited with " + 
                          ph.info().totalCpuDuration().orElse(Duration.ZERO)));
});
```

**Key Features:**

- **ProcessHandle**: Interface representing native process
- **Process Information**: Command, arguments, start time, user, etc.
- **Process Hierarchy**: Access to parent, children, descendants
- **Process Management**: Destroy processes, check if alive
- **Event Handling**: Notifications for process exit

> **Interviewer Insight:** When discussing the Process API updates, emphasize their importance for system management, monitoring, and self-healing applications. Prior to Java 9, interacting with system processes often required platform-specific code or third-party libraries. The enhanced API provides a standardized, cross-platform approach for tasks like:
> 
> 1. Building process monitoring tools
> 2. Managing child processes in server applications
> 3. Implementing graceful shutdown procedures
> 4. Creating admin consoles with process information
> 5. Developing self-healing applications that can restart failed components
> 
> This capability is particularly valuable in containerized and microservice environments where process management is an essential operational concern.
> 

### Other Features

Java 9 included several other notable improvements that enhanced specific areas of the platform.

### Reactive Streams API

```java
*// Java 9 introduced the Flow API for reactive programming*
import java.util.concurrent.Flow;
import java.util.concurrent.SubmissionPublisher;
import java.util.concurrent.Flow.Subscriber;
import java.util.concurrent.Flow.Subscription;

*// Simple subscriber implementation*
class SimpleSubscriber<T> implements Subscriber<T> {
    private Subscription subscription;
    private final String name;
    
    public SimpleSubscriber(String name) {
        this.name = name;
    }
    
    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);  *// Request first item*
        System.out.println(name + ": Subscribed");
    }
    
    @Override
    public void onNext(T item) {
        System.out.println(name + ": Received " + item);
        subscription.request(1);  *// Request next item*
    }
    
    @Override
    public void onError(Throwable error) {
        System.out.println(name + ": Error - " + error.getMessage());
    }
    
    @Override
    public void onComplete() {
        System.out.println(name + ": Completed");
    }
}

*// Using the reactive streams API*
try (SubmissionPublisher<Integer> publisher = new SubmissionPublisher<>()) {
    *// Subscribe*
    publisher.subscribe(new SimpleSubscriber<>("Subscriber 1"));
    publisher.subscribe(new SimpleSubscriber<>("Subscriber 2"));
    
    *// Publish items*
    System.out.println("Publishing items...");
    for (int i = 0; i < 5; i++) {
        publisher.submit(i);
    }
    
    *// Allow time for processing before publisher is closed*
    Thread.sleep(500);
}
```

### Stack-Walking API

```java
*// Efficient stack trace access and filtering*
StackWalker walker = StackWalker.getInstance(StackWalker.Option.RETAIN_CLASS_REFERENCE);

*// Get calling class*
Class<?> callerClass = walker.getCallerClass();
System.out.println("Called by: " + callerClass.getName());

*// Walk and filter stack frames*
walker.walk(frames -> 
    frames.filter(frame -> frame.getClassName().startsWith("com.example"))
          .map(frame -> frame.getClassName() + "." + frame.getMethodName() + " (line " + frame.getLineNumber() + ")")
          .limit(10)
          .collect(Collectors.toList())
);

*// Getting the full stack trace more efficiently*
List<StackWalker.StackFrame> stackTrace = walker.walk(frames -> frames.collect(Collectors.toList()));
```

### Multi-Resolution Images

```java
*// Support for resolution-variant images*
import java.awt.Image;
import java.awt.image.BaseMultiResolutionImage;
import java.awt.image.MultiResolutionImage;
import java.util.List;

*// Create images with different resolutions*
Image baseImage = loadImage("icon-16x16.png");
Image highResImage = loadImage("icon-32x32.png");
Image ultraHDImage = loadImage("icon-64x64.png");

*// Create multi-resolution image*
MultiResolutionImage multiResImage = new BaseMultiResolutionImage(
    baseImage, highResImage, ultraHDImage);

*// Get all variants*
List<Image> variants = multiResImage.getResolutionVariants();

*// Get resolution-specific version (based on DPI, etc.)*
Image bestVariant = multiResImage.getResolutionVariant(32, 32);
```

### Other Enhancements

- **CompletableFuture Improvements**: New methods like `timeout()`, `delayedExecutor()`, `completeOnTimeout()`
- **Platform Logging API**: Common logging framework for JDK components
- **HTML5 Javadoc**: Modern documentation with search functionality
- **UTF-8 Property Files**: Better support for internationalization
- **Deprecated API Warnings**: Enhanced annotations with forRemoval flag
- **Enhanced Method Handles**: Better performance and capabilities
- **Improved doclet API**: Better documentation tool support
- **Compact Strings**: More memory-efficient internal string representation

> **Deep Dive Tip:** The Reactive Streams API (java.util.concurrent.Flow) standardizes the core interfaces for asynchronous stream processing with non-blocking backpressure. While the JDK provides basic infrastructure classes like SubmissionPublisher, full reactive programming typically involves third-party libraries like Project Reactor, RxJava, or Akka Streams, which build on these interfaces. The value of the Flow API isn't in its direct use but in establishing a common language for reactive libraries to interoperate.
> 

> **Interviewer Insight:** When discussing Java 9's diverse feature set, emphasize how it addressed different aspects of modern application development: modularization for large codebases, reactive programming for responsive applications, improved process management for containerized environments, and various optimizations for performance and resource usage. This demonstrates Java's evolution from a simple object-oriented language into a comprehensive platform for diverse application styles and deployment models. Understanding the full spectrum of these features—not just the headline module system—shows a deeper appreciation for Java's adaptability to changing development paradigms.
> 

## 9.2 Java 10-11 Features

Java 10 (March 2018) and Java 11 (September 2018, LTS) introduced several important features that continued Java's evolution toward more concise, powerful syntax and improved APIs.

### Local Variable Type Inference (var)

Java 10 introduced the `var` keyword for local variable type inference, reducing verbosity while maintaining strong typing.

```java
*// Before Java 10 - explicit types*
ArrayList<String> list = new ArrayList<>();
Map<String, List<Integer>> map = new HashMap<>();
String text = Files.readString(Path.of("file.txt"));

*// Java 10+ with var - inferred types*
var list = new ArrayList<String>();  *// ArrayList<String>*
var map = new HashMap<String, List<Integer>>();  *// HashMap<String, List<Integer>>*
var text = Files.readString(Path.of("file.txt"));  *// String// Works with loops*
for (var entry : map.entrySet()) {
    var key = entry.getKey();    *// String*
    var values = entry.getValue();  *// List<Integer>// ...*
}

*// Works with try-with-resources*
try (var reader = new BufferedReader(new FileReader("data.txt"))) {
    var line = reader.readLine();  *// String// ...*
}

*// Lambda with explicit parameter types (Java 11+)*
var processor = (String s) -> s.toUpperCase();
```

**Key Points:**

- **Type Inference**: Compiler determines type from initialization
- **Scope**: Only for local variables with initializers
- **Strong Typing**: Still statically typed, not dynamic
- **Not For**: Fields, method parameters, return types
- **Limitations**: Cannot use with diamond operator without type
- **Lambda Parameters**: Supported from Java 11 with explicit type

**var Restrictions:**

```java
*// Not allowed: var without initializer*
var uninitialized;  *// Error: cannot use 'var' without initialization// Not allowed: null initializer*
var nothing = null;  *// Error: cannot infer type for var with null initializer// Not allowed: array initializers (use explicit array type)*
var array = { 1, 2, 3 };  *// Error: array initializer needs explicit target type// Not allowed: lambda without explicit parameter types (before Java 11)*
var lambda = x -> x.length();  *// Error: lambda requires explicit target type// Not recommended: inferred type may be too specific*
var obj = "hello";  *// Type is String, not CharSequence*
var list = List.of(1, 2, 3);  *// Type is ImmutableCollections$ListN, not List<Integer>*
```

> **Deep Dive Tip:** The `var` keyword uses local type inference, not global type inference like some other languages. This means inference is based only on the right-hand side of the assignment, not on how the variable is used. This localized approach maintains Java's design principle of making code easy to read from top to bottom. It also means that the inferred type might be more specific than you intend (e.g., inferring ArrayList instead of List), so be mindful when using `var` with interfaces and superclasses.
> 

> **Interviewer Insight:** When discussing `var`, emphasize that it's about reducing boilerplate without sacrificing Java's strong typing. Unlike dynamic typing in languages like JavaScript, `var` is fully resolved at compile-time with no runtime overhead. Good usage practices include:
> 
> 1. Using `var` when the type is obvious from context or initialization
> 2. Avoiding `var` when the inferred type would be surprising or too specific
> 3. Keeping meaningful variable names that indicate the type
> 4. Using it mostly for local variables with complex generic types
> 
> A balanced approach treats `var` as a readability tool, not a way to avoid thinking about types.
> 

### HTTP Client (Standard)

Java 11 standardized the HTTP Client API (introduced as incubator in Java 9), providing a modern replacement for HttpURLConnection.

```java
*// Synchronous HTTP request*
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Content-Type", "application/json")
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println("Status: " + response.statusCode());
System.out.println("Body: " + response.body());

*// Asynchronous HTTP request*
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println)
    .join();  *// Wait for completion// POST request with body*
HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/submit"))
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"John\",\"age\":30}"))
    .header("Content-Type", "application/json")
    .build();

HttpResponse<String> postResponse = client.send(postRequest, HttpResponse.BodyHandlers.ofString());

*// Custom client configuration*
HttpClient customClient = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)  *// Prefer HTTP/2*
    .followRedirects(HttpClient.Redirect.NORMAL)  *// Follow redirects*
    .connectTimeout(Duration.ofSeconds(10))  *// Connection timeout*
    .proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 8080)))  *// Proxy*
    .authenticator(Authenticator.getDefault())  *// Authentication*
    .build();

*// Different response handlers*
HttpResponse<byte[]> binaryResponse = client.send(request, HttpResponse.BodyHandlers.ofByteArray());
HttpResponse<Path> fileResponse = client.send(request, HttpResponse.BodyHandlers.ofFile(Path.of("response.json")));
HttpResponse<Stream<String>> streamResponse = client.send(request, HttpResponse.BodyHandlers.ofLines());

*// WebSocket support*
CompletableFuture<WebSocket> webSocket = client.newWebSocketBuilder()
    .buildAsync(URI.create("wss://ws.example.com"), new WebSocket.Listener() {
        @Override
        public void onOpen(WebSocket webSocket) {
            System.out.println("WebSocket opened");
            webSocket.request(1);
        }
        
        @Override
        public CompletionStage<?> onText(WebSocket webSocket, CharSequence data, boolean last) {
            System.out.println("Received: " + data);
            webSocket.request(1);
            return CompletableFuture.completedFuture(null);
        }
        
        @Override
        public CompletionStage<?> onClose(WebSocket webSocket, int statusCode, String reason) {
            System.out.println("WebSocket closed: " + reason);
            return CompletableFuture.completedFuture(null);
        }
    });
```

**Key Features:**

- **Modern API**: Fluent builder-style API
- **Synchronous & Asynchronous**: Support for both modes
- **HTTP/2**: Built-in support for HTTP/2 protocol
- **WebSocket**: Native WebSocket client
- **Various Body Handlers**: String, byte array, file, stream
- **Authentication**: Built-in support for authentication
- **Timeouts**: Connection and read timeout support

> **Deep Dive Tip:** The HTTP Client uses Java's CompletableFuture for asynchronous operations, making it easy to integrate with existing asynchronous code. Under the hood, it maintains connection pooling and uses non-blocking I/O, which makes it more efficient than the older HttpURLConnection. For high-throughput scenarios, you can reuse the same HttpClient instance across multiple requests to benefit from connection pooling and HTTP/2 multiplexing.
> 

> **Interviewer Insight:** When discussing the HTTP Client API, emphasize how it addresses the limitations of HttpURLConnection while providing modern features expected in contemporary web applications. Key advantages include:
> 
> 1. Support for modern protocols (HTTP/2, WebSockets)
> 2. First-class asynchronous support with CompletableFuture
> 3. Immutability and thread safety by design
> 4. Streaming capabilities for large responses
> 5. Sensible defaults with flexible configuration
> 
> These improvements make it suitable for both simple scripts and high-performance microservices, reducing the need for external HTTP client libraries in many cases.
> 

### Nest-Based Access Control

Java 11 introduced a formal concept of nested classes at the JVM level, enabling more intuitive access between related classes.

```java
*// Nested classes with private members*
public class Outer {
    private String outerPrivate = "Outer private field";
    
    public void accessInnerPrivate() {
        Inner inner = new Inner();
        *// Direct access to Inner's private field (nest-based access)*
        System.out.println(inner.innerPrivate);
    }
    
    public class Inner {
        private String innerPrivate = "Inner private field";
        
        public void accessOuterPrivate() {
            *// Direct access to Outer's private field*
            System.out.println(outerPrivate);
        }
    }
}

*// Checking nest relationships programmatically*
Class<?> outerClass = Outer.class;
Class<?> innerClass = Outer.Inner.class;

*// Get host class (the top-level class of the nest)*
Class<?> host = innerClass.getNestHost();
System.out.println("Nest host: " + host.getName());  *// Outer// Check if classes are nestmates*
boolean areNestmates = outerClass.isNestmateOf(innerClass);
System.out.println("Are nestmates: " + areNestmates);  *// true// Get all nest members*
Class<?>[] members = outerClass.getNestMembers();
for (Class<?> member : members) {
    System.out.println("Nest member: " + member.getName());
}
```

**Key Concepts:**

- **Nest Host**: The top-level class that contains nested classes
- **Nest Members**: All classes declared within the nest host
- **Nestmates**: Classes belonging to the same nest
- **Access Control**: Nestmates can access each other's private members directly
- **Reflection API**: Methods to check nest relationships

> **Deep Dive Tip:** Nest-based access control formalizes what was already happening at the source level. Before Java 11, the compiler would generate synthetic accessor methods to allow nested classes to access each other's private members, which added runtime overhead and cluttered the bytecode. With nest-based access, the JVM recognizes the special relationship between nestmates, enabling direct access without these synthetic methods. This improves performance and provides a cleaner reflection API for analyzing class relationships.
> 

> **Interviewer Insight:** When discussing nest-based access control, emphasize that it's primarily an implementation improvement rather than a source-level feature. It doesn't change how you write nested classes in Java, but it makes the compiled code more efficient and the reflection API more accurate. This feature demonstrates Java's commitment to improving the JVM's alignment with the language's design principles and optimizing existing patterns rather than just adding new syntax. It's a good example of how Java evolves both its language features and its runtime platform in tandem.
> 

### String Methods (Java 11)

Java 11 introduced several convenience methods to the String class for common operations.

```java
*// isBlank() - checks if string is empty or contains only whitespace*
String blank = "   \t \n ";
System.out.println(blank.isBlank());  *// true*

String notBlank = "   Hello   ";
System.out.println(notBlank.isBlank());  *// false// lines() - splits string into a stream of lines*
String multiline = "Line 1\nLine 2\nLine 3";
multiline.lines().forEach(System.out::println);
*// Prints:// Line 1// Line 2// Line 3// Count lines*
long lineCount = multiline.lines().count();
System.out.println("Line count: " + lineCount);  *// 3// strip(), stripLeading(), stripTrailing() - Unicode-aware trim*
String text = "  Hello, World!  ";
System.out.println("Original: '" + text + "'");
System.out.println("strip: '" + text.strip() + "'");            *// 'Hello, World!'*
System.out.println("stripLeading: '" + text.stripLeading() + "'");  *// 'Hello, World!  '*
System.out.println("stripTrailing: '" + text.stripTrailing() + "'");  *// '  Hello, World!'// Difference between strip() and trim()*
String unicodeWhitespace = "\u2005Hello\u2005";  *// Unicode whitespace*
System.out.println("trim: '" + unicodeWhitespace.trim() + "'");   *// '\u2005Hello\u2005' (unchanged)*
System.out.println("strip: '" + unicodeWhitespace.strip() + "'");  *// 'Hello'// repeat() - repeats string n times*
String repeated = "Java ".repeat(3);
System.out.println(repeated);  *// Java Java Java // Empty string for zero repetitions*
String empty = "Java".repeat(0);
System.out.println("Empty: '" + empty + "'");  *// Empty: ''*
```

**Key Methods:**

- **isBlank()**: Checks if string is empty or whitespace-only
- **lines()**: Splits string into a stream of lines
- **strip()/stripLeading()/stripTrailing()**: Unicode-aware trimming
- **repeat(n)**: Creates a string by repeating the original n times

> **Deep Dive Tip:** The `strip()` methods differ from the older `trim()` method in how they handle whitespace. While `trim()` only removes characters with code points ≤ U+0020 (space), the `strip()` methods use `Character.isWhitespace()` to identify whitespace, correctly handling all Unicode whitespace characters. This makes `strip()` more appropriate for international text processing where various Unicode whitespace characters might be present.
> 

> **Interviewer Insight:** When discussing the new String methods, highlight that they address common pain points in string handling:
> 
> 1. `isBlank()` simplifies input validation by handling empty and whitespace-only strings
> 2. `lines()` provides a modern, stream-based alternative to `String.split("\n")`
> 3. `strip()` fixes the long-standing limitations of `trim()` with Unicode
> 4. `repeat()` eliminates the need for manual loops or StringBuilder for string repetition
> 
> These methods reduce boilerplate and improve code readability for common operations, making them good examples of Java's incremental evolution to smooth rough edges in the API.
> 

### Files Methods (Java 11)

Java 11 added convenient methods to the Files class for reading and writing strings.

```java
*// Reading a file as a string*
Path path = Path.of("sample.txt");
String content = Files.readString(path);
System.out.println("File content: " + content);

*// Reading with specific charset*
String contentUtf8 = Files.readString(path, StandardCharsets.UTF_8);

*// Writing a string to a file*
String data = "Hello, Java 11!";
Files.writeString(path, data);

*// Writing with specific charset and options*
Files.writeString(
    path,
    data,
    StandardCharsets.UTF_8,
    StandardOpenOption.CREATE,
    StandardOpenOption.TRUNCATE_EXISTING
);

*// Reading all lines into a List*
List<String> lines = Files.readAllLines(path);

*// Working with streams of lines*
try (Stream<String> lineStream = Files.lines(path)) {
    long lineCount = lineStream.count();
    System.out.println("Line count: " + lineCount);
}

*// Reading/writing binary data*
byte[] binaryData = Files.readAllBytes(path);
Files.write(path, binaryData);
```

**Key Methods:**

- **readString(Path)**: Reads entire file into a string
- **writeString(Path, String)**: Writes string to a file
- **readAllLines(Path)**: Reads all lines into a List
- **lines(Path)**: Stream of lines from a file
- **readAllBytes(Path)/write(Path, byte[])**: Binary file operations

> **Deep Dive Tip:** While these methods are convenient for small to medium-sized files, they load the entire file content into memory at once. For very large files, streaming approaches like `Files.lines()` or using `BufferedReader` with `Files.newBufferedReader()` are more memory-efficient as they process the file incrementally. The new methods are best used when the complete content is needed at once and fits comfortably in memory.
> 

> **Interviewer Insight:** When discussing the Files methods, emphasize how they simplify common I/O patterns with clean, concise code. Prior to Java 11, reading a file as a string required multiple lines with try-catch blocks, readers, and buffers. The new methods reduce this to a single line, making code more readable and less error-prone. This is part of Java's ongoing effort to reduce boilerplate for everyday tasks while maintaining performance and flexibility for advanced scenarios.
> 

### Collection Enhancements

Both Java 10 and 11 introduced subtle but useful improvements to the Collections API.

```java
*// Java 10: copyOf methods for unmodifiable copies*
List<String> original = new ArrayList<>(List.of("a", "b", "c"));
original.add("d");

*// Create unmodifiable copy*
List<String> copy = List.copyOf(original);
*// copy.add("e");  // Throws UnsupportedOperationException// Also available for Set and Map*
Set<String> setCopy = Set.copyOf(original);
Map<String, Integer> mapCopy = Map.copyOf(Map.of("a", 1, "b", 2));

*// Java 10: toUnmodifiableList/Set/Map collectors*
List<String> unmodifiableList = original.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toUnmodifiableList());

Set<String> unmodifiableSet = original.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toUnmodifiableSet());

Map<String, Integer> unmodifiableMap = original.stream()
    .collect(Collectors.toUnmodifiableMap(
        s -> s,
        String::length
    ));

*// Java 11: Collection.toArray(IntFunction)*
String[] array = original.toArray(String[]::new);
*// More concise than: original.toArray(new String[0])*
```

**Key Enhancements:**

- **List/Set/Map.copyOf()**: Creates unmodifiable copies
- **Collectors.toUnmodifiableXxx()**: Collects to unmodifiable collections
- **Collection.toArray(IntFunction)**: Simplifies array creation

> **Deep Dive Tip:** The `copyOf()` methods differ from `Collections.unmodifiableXxx()` in an important way: they create a completely new collection, breaking the connection to the original. With `Collections.unmodifiableXxx()`, changes to the original collection are visible through the unmodifiable view, which can lead to unexpected behavior. The `copyOf()` methods guarantee true immutability by creating defensive copies.
> 

> **Interviewer Insight:** When discussing collection enhancements, highlight the broader trend toward immutability in Java's collections API. From Java 9's collection factory methods to Java 10's copyOf methods and Java 11's simplified toArray, each release has added tools that make immutable collections more convenient to create and use. This shift reflects a growing recognition of immutability's benefits for concurrent programming, functional styles, and defensive programming. A modern Java developer should default to immutable collections where possible, using mutable collections only when necessary for performance or specific algorithms.
> 

### Lambda Parameter Var

Java 11 extended local variable type inference (`var`) to lambda expression parameters, enabling annotations on lambda parameters.

```java
*// Before Java 11 - implicit or explicit typing*
Predicate<String> implicitP1 = s -> s.length() > 5;
Predicate<String> explicitP1 = (String s) -> s.length() > 5;

*// Java 11 - var in lambda parameters*
Predicate<String> varP1 = (var s) -> s.length() > 5;

*// Multiple parameters*
BiFunction<String, String, Integer> bf = (var s1, var s2) -> s1.length() + s2.length();

*// Annotations on lambda parameters (main benefit of var for lambdas)*
import java.util.Objects;
import javax.annotation.Nonnull;

Predicate<String> notNull = (@Nonnull var s) -> s.length() > 0;

Consumer<String> consumer = (@Nonnull var s) -> {
    Objects.requireNonNull(s);
    System.out.println(s.toUpperCase());
};

*// Type inference still works*
var list = List.of("a", "b", "c");
var mapped = list.stream()
    .map((var s) -> s.toUpperCase())
    .collect(Collectors.toList());
```

**Key Points:**

- **Syntax**: Must use `var` for all parameters or none
- **Type Inference**: Works the same as with explicit types
- **Main Benefit**: Enables annotations on lambda parameters
- **Restrictions**: Cannot mix with explicit or implicit typing

> **Deep Dive Tip:** The primary motivation for `var` in lambda parameters was not conciseness (implicit parameters were already supported) but the ability to add annotations to parameters without using explicit types. Before Java 11, adding annotations required fully specifying the type, which could be verbose for complex generic types. With `var`, you can add annotations while still benefiting from type inference.
> 

> **Interviewer Insight:** When discussing `var` in lambda parameters, emphasize that it fills a specific gap in the language rather than providing a major new capability. It's most valuable when you need to add annotations to lambda parameters, particularly for tools that rely on annotations for static analysis, validation, or dependency injection. For simple lambdas without annotations, implicit parameters (without `var`) remain the most concise option. This feature demonstrates Java's attention to supporting ecosystem tools and frameworks that leverage annotations for extended functionality.
> 

### Other Java 10-11 Features

Java 10 and 11 introduced several other improvements to different areas of the platform.

### Garbage Collection Improvements

```java
*// Java 10: Application Class-Data Sharing// Command line: java -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.lst MyApp// Command line: java -XX:+UseAppCDS -XX:SharedClassListFile=classes.lst -XX:SharedArchiveFile=app.jsa MyApp// Java 10: Parallel Full GC for G1// Command line: java -XX:+UseG1GC MyApp// Java 11: ZGC - low latency GC (experimental)// Command line: java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC MyApp// Java 11: Epsilon GC - no-op GC for testing// Command line: java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC MyApp*
```

### Flight Recorder (Now Free)

```java
*// Java 11: Flight Recorder (previously commercial feature)// Command line: java -XX:StartFlightRecording=duration=60s,filename=recording.jfr MyApp// Programmatic access*
jdk.jfr.Recording recording = new jdk.jfr.Recording();
recording.start();
*// ... application code ...*
recording.stop();
recording.dump(Path.of("recording.jfr"));
```

### Dynamic Class-File Constants

```java
*// Java 11: Enhanced constant pool for JVM classes// Primarily for JVM implementers and language designers// Enables more efficient implementation of dynamic languages on the JVM*
```

### Removal of Java EE and CORBA Modules

```java
*// Java 11 removed several Java EE and CORBA modules// Affected packages:// - javax.activation// - javax.xml.bind.*// - javax.xml.ws.*// - javax.jws.*// - javax.transaction// - java.corba// - java.xml.ws.annotation// Migration: Add explicit dependencies to build file// Maven example:/*
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>
*/*
```

### Unicode 10 Support (Java 11)

```java
*// Java 11: Support for Unicode 10*
String emoji = "😀";  *// Unicode 6.0*
String newEmoji = "🥰";  *// Unicode 10.0, supported in Java 11+*
```

> **Deep Dive Tip:** The removal of Java EE modules in Java 11 was a significant change that affected many applications, especially those using JAXB for XML processing or JAX-WS for web services. This change was part of Java's modularization strategy, moving components that weren't core to the platform into separate libraries. Applications using these APIs need to add explicit dependencies on the standalone versions, which are now maintained as separate projects.
> 

> **Interviewer Insight:** When discussing Java 10-11 features, highlight how they reflect Java's dual focus on developer productivity and runtime performance. Features like `var` and new string/file methods improve developer experience, while enhancements to garbage collectors and class loading improve runtime characteristics. The removal of Java EE modules demonstrates Oracle's strategic focus on a leaner, more modular core platform with clear boundaries between standard and enterprise features. Understanding these broader trends helps contextualize individual features within Java's evolution strategy.
> 

## 9.3 Java 12-16 Features

Java 12 through Java 16 introduced several innovative language features that continued to modernize Java's syntax and programming model.

### Switch Expressions

Java 12 introduced switch expressions (preview in 12-13, standard in 14), making switch more expressive and safer.

```java
*// Traditional switch statement (verbose, error-prone)*
String oldResult;
Day day = Day.WEDNESDAY;

switch (day) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        oldResult = "Weekday";
        break;
    case SATURDAY:
    case SUNDAY:
        oldResult = "Weekend";
        break;
    default:
        oldResult = "Unknown";
        break;
}

*// Java 12-13 (preview): switch expression with arrow syntax*
String result = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Unknown";
};

*// Java 14+ (standard): switch expression with arrow or block syntax*
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    case THURSDAY, SATURDAY -> 8;
    case WEDNESDAY -> 9;
    default -> {
        String dayName = day.toString();
        int letters = dayName.length();
        yield letters;  *// Use yield to return from a block*
    }
};

*// Handling all enum cases without default (Java 14+)*
String result2 = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
};  *// No default needed if all enum cases are covered// Multiple case labels (Java 14+)*
String result3 = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
};
```

**Key Features:**

- **Expression Form**: Returns a value directly
- **Arrow Syntax**: Concise single-expression cases
- **Multiple Case Labels**: Group cases without fall-through
- **Exhaustiveness Checking**: Compiler ensures all cases covered
- **yield Statement**: Return values from case blocks

> **Deep Dive Tip:** Switch expressions enforce exhaustiveness, requiring that all possible input values are handled. For enum types, this means covering all enum constants or providing a default case. This compile-time check prevents a common source of bugs in traditional switch statements where missing cases silently fail. The exhaustiveness check also helps future-proof code—if new enum constants are added, the compiler will flag switch expressions that don't handle them.
> 

> **Interviewer Insight:** When discussing switch expressions, emphasize how they address three key problems with traditional switch statements:
> 
> 1. **Safety**: No accidental fall-through between cases
> 2. **Expressiveness**: Directly yields a value, reducing boilerplate
> 3. **Exhaustiveness**: Compiler ensures all cases are handled
> 
> These improvements make switch expressions a clear example of Java's evolution toward safer, more expressive syntax while maintaining backward compatibility. Modern Java code should prefer switch expressions over statements for most use cases, especially when assigning a result based on a switch.
> 

### Text Blocks (Java 15)

Java 15 standardized text blocks, which provide a clean way to include multi-line string literals in code.

```java
*// Before text blocks - escaped newlines and quotes*
String jsonOld = "{\n" +
                "    \"name\": \"John Doe\",\n" +
                "    \"age\": 30,\n" +
                "    \"address\": {\n" +
                "        \"street\": \"123 Main St\",\n" +
                "        \"city\": \"Anytown\"\n" +
                "    }\n" +
                "}";

*// With text blocks (Java 15+)*
String json = """
        {
            "name": "John Doe",
            "age": 30,
            "address": {
                "street": "123 Main St",
                "city": "Anytown"
            }
        }
        """;

*// HTML example*
String html = """
        <html>
            <body>
                <h1>Hello, World!</h1>
                <p>This is a paragraph with <strong>bold text</strong>.</p>
            </body>
        </html>
        """;

*// SQL example*
String sql = """
        SELECT e.employee_id, e.first_name, e.last_name, d.department_name
        FROM employees e
        JOIN departments d ON e.department_id = d.department_id
        WHERE e.hire_date > '2020-01-01'
        ORDER BY e.last_name, e.first_name
        """;

*// Controlling indentation*
String indented = """
        This is a text block
        that will preserve my indentation
            This line has extra indentation
        Back to original indentation
        """;

*// Escaping end delimiter*
String withTripleQuotes = """
        This text includes "quotes" and even
        a triple-quote sequence: \"""
        without ending the text block.
        """;

*// Escaping line break (Java 15+)*
String singleLine = """
        This is actually \
        a single line of text \
        without line breaks.
        """;

*// String operations with text blocks*
String withExpression = """
        Hello, %s!
        Today is %s.
        """.formatted("John", LocalDate.now());

String upperCase = """
        This will be
        UPPERCASE
        """.toUpperCase();

String trimmed = """
        
        This has blank lines
        above and below
        
        """.strip();
```

**Key Features:**

- **Multi-line Strings**: Preserves line breaks and formatting
- **No Escaping Needed**: For most quotes and newlines
- **Indentation Control**: Automatically removes common indentation
- **Line Continuation**: Escape end-of-line with backslash
- **Triple Quotes**: Delimited by three double-quote characters

> **Deep Dive Tip:** Text blocks handle indentation intelligently. The compiler determines the "effective indentation" by finding the whitespace common to all non-blank lines, then removes this from each line. This allows you to indent the text block in your source code to match surrounding code while controlling the actual indentation in the resulting string. The closing delimiter's position determines the baseline for this calculation, so you can control the indentation by positioning the closing `"""`.
> 

> **Interviewer Insight:** When discussing text blocks, emphasize that they're not just a convenience feature but address real readability and maintainability issues with multi-line strings. Code that embeds formats like JSON, HTML, SQL, or XML becomes significantly more readable and less error-prone. Text blocks also eliminate the "escaping hell" that often leads to bugs in complex string literals. They represent Java's pragmatic approach to language evolution—addressing common pain points without fundamentally changing the language model.
> 

### Pattern Matching for instanceof (Java 16)

Java 16 standardized pattern matching for instanceof, combining type checking and casting into a single operation.

```java
*// Before pattern matching*
Object obj = getObject();
if (obj instanceof String) {
    String s = (String) obj;  *// Explicit cast needed*
    if (s.length() > 5) {
        System.out.println(s.toUpperCase());
    }
}

*// With pattern matching (Java 16+)*
if (obj instanceof String s) {  *// Pattern variable 's' introduced// 's' is already cast to String and available here*
    if (s.length() > 5) {
        System.out.println(s.toUpperCase());
    }
} *// 's' goes out of scope here// Pattern variables are only in scope when the instanceof test is true*
if (obj instanceof String s && s.length() > 5) {  *// 's' in scope for the right side of &&*
    System.out.println(s.toUpperCase());
}

*// Pattern variables are effectively final*
if (obj instanceof String s) {
    *// s = "something else";  // Error: pattern variables are effectively final*
    System.out.println(s);
}

*// Nested conditions*
if (obj instanceof String s && s.length() > 10) {
    System.out.println("Long string: " + s);
} else if (obj instanceof Number n) {
    System.out.println("Number value: " + n.doubleValue());
} else if (obj instanceof List<?> list && !list.isEmpty()) {
    System.out.println("First element: " + list.get(0));
}

*// Null handling*
obj = null;
if (obj instanceof String s) {  *// Evaluates to false for null// This block is never executed for null*
    System.out.println(s);
}
```

**Key Features:**

- **Pattern Variables**: Introduced when instanceof is true
- **Scope Control**: Variables only in scope when pattern matches
- **Flow Analysis**: Compiler tracks definite assignment
- **Final Variables**: Pattern variables are effectively final
- **Null Safety**: Evaluates to false for null values

> **Deep Dive Tip:** Pattern matching for instanceof is implemented using flow-sensitive type analysis in the compiler. This allows the compiler to track when a pattern variable is definitely assigned based on the control flow of the program. The same mechanism enables pattern variables to be in scope only when the instanceof check is true, including in complex boolean expressions with short-circuit evaluation. This sophisticated analysis ensures type safety while making the feature convenient to use in real-world code.
> 

> **Interviewer Insight:** When discussing pattern matching for instanceof, emphasize that it addresses a common source of boilerplate and potential errors in Java code. The traditional pattern of "instanceof check followed by cast" is not only verbose but error-prone—the cast might be forgotten or mistyped. Pattern matching makes code both more concise and safer by automating this common pattern. This feature is part of a larger trend in Java toward pattern matching, which will expand to switch expressions and other contexts in future releases, bringing more expressiveness to the language without sacrificing type safety.
> 

### Records (Java 14)

Java 14 introduced records (preview in 14-15, standard in 16), a concise way to declare immutable data classes.

```java
*// Before records - verbose data class*
class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PersonOld that = (PersonOld) o;
        return age == that.age && Objects.equals(name, that.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "Person[name=" + name + ", age=" + age + "]";
    }
}

*// With records (Java 16+)*
record Person(String name, int age) {
    *// That's it! Automatically gets:// - Constructor// - Accessor methods (name(), age())// - equals(), hashCode(), toString()*
}

*// Using a record*
Person person = new Person("John", 30);
String name = person.name();  *// Accessor method (not getName())*
int age = person.age();
System.out.println(person);  *// Nice toString(): Person[name=John, age=30]// Records with validation - compact constructor*
record ValidatedPerson(String name, int age) {
    *// Compact constructor for validation*
    public ValidatedPerson {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        *// No explicit assignment needed - implicit*
    }
}

*// Regular constructor (can call the canonical constructor)*
record PersonWithFormatter(String name, int age, DateTimeFormatter formatter) {
    public PersonWithFormatter(String name, int age) {
        this(name, age, DateTimeFormatter.ISO_LOCAL_DATE);
    }
}

*// Instance methods in records*
record Rectangle(double width, double height) {
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
}

*// Static members in records*
record MathValue(double value) {
    private static final double PI = 3.14159;
    
    public static MathValue of(double value) {
        return new MathValue(value);
    }
    
    public MathValue add(MathValue other) {
        return new MathValue(this.value + other.value);
    }
}

*// Local records (Java 16+)*
void processData(List<String> data) {
    *// Local record for temporary data structure*
    record Entry(int index, String value, boolean valid) {}
    
    List<Entry> entries = new ArrayList<>();
    for (int i = 0; i < data.size(); i++) {
        String value = data.get(i);
        boolean valid = value != null && !value.isBlank();
        entries.add(new Entry(i, value, valid));
    }
    
    *// Process entries...*
}
```

**Key Features:**

- **Concise Syntax**: Declare state, accessors, and methods in one place
- **Immutability**: All fields are final by default
- **Automatic Methods**: Constructor, accessors, equals/hashCode/toString
- **Compact Constructors**: Validate without repeating parameters
- **Custom Methods**: Add behavior to data classes
- **Local Records**: Define records within methods for local use

**Record Components:**

- Define the state of the record
- Generate accessor methods named after the components
- Are final and cannot be modified after construction
- Cannot be shadowed by fields with same name
- Are directly accessible in methods of the record

> **Deep Dive Tip:** Records are a form of product type (a type defined by the product of its components), which is a fundamental concept in type theory and functional programming. However, records in Java are more than just product types—they're full classes with methods, constructors, and inheritance from Object. What makes them special is their built-in behavior based on state, not identity. This state-based equality makes records suitable for immutable data transfer, domain modeling, and pattern matching. Records are intentionally restricted (no extending other classes, no mutable state) to reinforce their nature as transparent carriers of immutable data.
> 

> **Interviewer Insight:** When discussing records, emphasize how they address a common source of boilerplate in Java applications—data classes that primarily hold state. Records not only reduce the code needed to define these classes but also enforce good practices like immutability and value semantics. They're particularly valuable for:
> 
> 1. Data transfer objects (DTOs)
> 2. Message objects in event-driven systems
> 3. Result holders for complex operations
> 4. Grouped parameters (replacing "parameter objects")
> 5. Tuples in functional-style code
> 
> Records represent Java's evolution toward more declarative programming, allowing developers to express what data they need rather than how to implement standard behaviors.
> 

### Sealed Classes (Preview)

Java 15 and 16 introduced sealed classes and interfaces as a preview feature, providing more control over inheritance.

```java
*// Sealed class with permitted subclasses*
public sealed class Shape permits Circle, Rectangle, Triangle {
    *// Common shape methods...*
    public abstract double area();
}

*// Permitted subclasses must explicitly extend and be final, sealed, or non-sealed*
public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

*// Another permitted subclass, also final*
public final class Rectangle extends Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

*// Another permitted subclass*
public final class Triangle extends Shape {
    private final double base;
    private final double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double area() {
        return 0.5 * base * height;
    }
}

*// Non-sealed allows unrestricted extension*
public sealed class Vehicle permits Car, Truck, Bus, Motorcycle {
    *// Vehicle methods...*
}

public non-sealed class Car extends Vehicle {
    *// Anyone can extend Car*
}

public final class Truck extends Vehicle {
    *// No further extension possible*
}

*// Further sealing the hierarchy*
public sealed class Bus extends Vehicle permits SchoolBus, CityBus {
    *// Only specified buses allowed*
}

public final class SchoolBus extends Bus {
    *// No further extension*
}

public final class CityBus extends Bus {
    *// No further extension*
}

public final class Motorcycle extends Vehicle {
    *// No further extension*
}

*// Sealed interfaces*
public sealed interface PaymentMethod permits CreditCard, DebitCard, Cash, CryptoCurrency {
    *// Payment method behavior*
}

public final class CreditCard implements PaymentMethod {
    *// Implementation*
}

public final class DebitCard implements PaymentMethod {
    *// Implementation*
}

public final class Cash implements PaymentMethod {
    *// Implementation*
}

public non-sealed interface CryptoCurrency extends PaymentMethod {
    *// Open for extension*
}
```

**Key Concepts:**

- **Sealed Classes/Interfaces**: Restrict which classes can extend/implement them
- **Permits Clause**: Explicitly lists allowed subclasses
- **Subclass Modifiers**: Subclasses must be final, sealed, or non-sealed
- **Compile-Time Checking**: Inheritance hierarchy enforced at compile time
- **Package Restrictions**: Permitted subclasses must be in same package or module

> **Deep Dive Tip:** Sealed classes provide a middle ground between open inheritance (public classes) and no inheritance (final classes). They enable a form of algebraic data types in Java, where a supertype can enumerate all possible subtypes. This is particularly powerful when combined with pattern matching, as the compiler can perform exhaustiveness checking. The "sealed" concept also facilitates optimizations in the JVM, as the runtime knows the complete set of possible subtypes, enabling more efficient method dispatch and inline caching.
> 

> **Interviewer Insight:** When discussing sealed classes, emphasize how they support better domain modeling by allowing developers to define closed hierarchies of related types. This is valuable for:
> 
> 1. Domain models where all subtypes are known (e.g., payment methods, account types)
> 2. API design where third-party extensions should be controlled
> 3. Pattern matching scenarios where exhaustiveness is important
> 4. Framework development where extension points should be explicit
> 
> Sealed classes represent Java's move toward more expressive type systems that can encode more domain knowledge directly in the code, making programs safer and more self-documenting.
> 

### Other Features

Java 12 through 16 introduced several other features that improved various aspects of the platform.

### Helpful NullPointerExceptions (Java 14)

```java
*// Before Java 14// Exception message: NullPointerException// person.getAddress().getStreet().toUpperCase();// With Java 14+// Exception message: "Cannot invoke "String.toUpperCase()" because // the return value of "Address.getStreet()" is null"*
```

### Hidden Classes (Java 15)

```java
*// Java 15: API for defining hidden classes// Used primarily by frameworks that generate classes dynamically// Example (for framework developers):*
byte[] classBytes = generateClassBytes();  *// Generate bytecode*
Lookup lookup = MethodHandles.lookup();
Class<?> hiddenClass = lookup.defineHiddenClass(classBytes, true, ClassOption.NESTMATE)
    .lookupClass();
*// Hidden class is not discoverable via reflection*
```

### Foreign Memory Access API (Java 14-16 Incubator)

```java
*// Access off-heap memory safely (incubator API)*
import jdk.incubator.foreign.*;

*// Allocate off-heap memory*
try (ResourceScope scope = ResourceScope.newConfinedScope()) {
    MemorySegment segment = MemorySegment.allocateNative(100, scope);
    
    *// Access memory safely*
    MemoryAddress address = segment.address();
    VarHandle intHandle = MemoryHandles.varHandle(int.class, ByteOrder.nativeOrder());
    
    *// Write to memory*
    intHandle.set(segment, 0L, 42);
    
    *// Read from memory*
    int value = (int) intHandle.get(segment, 0L);
    System.out.println("Value: " + value);
}  *// Memory automatically freed when scope is closed*
```

### Vector API (Incubator)

```java
*// SIMD (Single Instruction, Multiple Data) operations (incubator)*
import jdk.incubator.vector.*;

*// Create vectors*
VectorSpecies<Float> species = FloatVector.SPECIES_256;  *// 256-bit vector (8 floats)*
float[] a = new float[1024];
float[] b = new float[1024];
float[] c = new float[1024];
*// Initialize arrays...// Process vectors in parallel*
for (int i = 0; i < a.length; i += species.length()) {
    FloatVector va = FloatVector.fromArray(species, a, i);
    FloatVector vb = FloatVector.fromArray(species, b, i);
    FloatVector vc = va.mul(vb);  *// Multiply vectors*
    vc.intoArray(c, i);  *// Store result*
}
```

### Unix-Domain Socket Channels (Java 16)

```java
*// Unix domain sockets for local inter-process communication*
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.file.Path;
import java.net.UnixDomainSocketAddress;

*// Server*
UnixDomainSocketAddress address = UnixDomainSocketAddress.of(
    Path.of("/tmp/server.socket"));
ServerSocketChannel server = ServerSocketChannel.open(StandardProtocolFamily.UNIX);
server.bind(address);
SocketChannel channel = server.accept();
*// Use channel for communication...// Client*
SocketChannel client = SocketChannel.open(StandardProtocolFamily.UNIX);
client.connect(address);
*// Use client for communication...*
```

> **Deep Dive Tip:** The Foreign Memory Access API (now the Foreign Function & Memory API) represents a significant evolution in Java's memory management model. It provides safe, controlled access to memory outside the Java heap without using JNI or Unsafe. This is crucial for applications that need to interact with native libraries, process large datasets without garbage collection pressure, or implement memory-mapped file access. The API uses resource scopes to ensure memory safety through structured access patterns, preventing common issues like memory leaks and dangling pointers.
> 

> **Interviewer Insight:** When discussing these features, highlight that Java 12-16 introduced not just language improvements but also significant infrastructure changes. Features like helpful NPEs and sealed classes improve developer experience, while the Foreign Memory API and Vector API address performance for specialized use cases. The breadth of these changes demonstrates Java's balanced approach to evolution—enhancing both the language itself and its underlying platform capabilities. A modern Java developer needs to understand both aspects to leverage the full power of the platform, especially for performance-critical or system-level applications.
> 

## 9.4 Java 17 Features (LTS)

Java 17, released in September 2021, is a Long-Term Support (LTS) release with several important features finalized from previous preview versions.

### Sealed Classes (Final)

Java 17 finalized the sealed classes feature, which was previously in preview in Java 15 and 16.

```java
*// Sealed hierarchies with pattern matching*
public sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
}

public final class Circle implements Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public final class Rectangle implements Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

public final class Triangle implements Shape {
    private final double base;
    private final double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double area() {
        return 0.5 * base * height;
    }
}

*// Using sealed classes with pattern matching for instanceof*
void printArea(Shape shape) {
    if (shape instanceof Circle c) {
        System.out.println("Circle area: " + c.area());
    } else if (shape instanceof Rectangle r) {
        System.out.println("Rectangle area: " + r.area());
    } else if (shape instanceof Triangle t) {
        System.out.println("Triangle area: " + t.area());
    }
    *// No need for else clause - compiler knows these are all possibilities*
}

*// Using with enhanced switch (pattern matching for switch in preview)*
String getShapeType(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius";
        case Rectangle r -> "Rectangle";
        case Triangle t -> "Triangle";
        *// No default needed - compiler knows this covers all cases*
    };
}

*// Reflection API support*
Class<?> clazz = Shape.class;
boolean isSealed = clazz.isSealed();  *// true*
Class<?>[] permittedSubclasses = clazz.getPermittedSubclasses();
for (Class<?> subclass : permittedSubclasses) {
    System.out.println("Permitted: " + subclass.getName());
}
```

**Design Principles:**

- **Inheritance Control**: Define exactly which classes can extend/implement
- **Domain Modeling**: Express closed sets of related types
- **Exhaustiveness**: Enable compiler to verify all cases are handled
- **Reflection Support**: Inspect sealed nature and permitted subclasses

> **Deep Dive Tip:** Sealed classes are more than just a restrictive inheritance mechanism—they're designed to work in concert with pattern matching to enable exhaustiveness checking. When a sealed hierarchy is matched in a switch expression or if-else chain, the compiler can verify that all possible subtypes are handled. This synergy between sealed classes and pattern matching is a foundational part of Java's strategy for supporting algebraic data types and functional programming patterns within an object-oriented language.
> 

> **Interviewer Insight:** When discussing sealed classes in Java 17, emphasize that their finalization represents a major milestone in Java's pattern matching journey. They enable safer, more maintainable code by:
> 
> 1. Making inheritance explicit and documented in the code itself
> 2. Preventing unexpected extensions that could break assumptions
> 3. Enabling exhaustiveness checks in pattern matching contexts
> 4. Providing a foundation for future pattern matching enhancements
> 
> Sealed classes are particularly valuable in API design, where controlling extension points is crucial for ensuring correctness and enabling future evolution. They represent Java's pragmatic approach to incorporating functional programming concepts without abandoning its object-oriented foundation.
> 

### Pattern Matching for Switch (Preview)

Java 17 introduced pattern matching for switch as a preview feature, extending switch expressions to work with patterns.

```java
*// Pattern matching for switch (preview in Java 17)*
Object obj = getValue();

*// Type patterns*
String formatted = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case Long l -> String.format("long %d", l);
    case Double d -> String.format("double %f", d);
    case String s -> String.format("String %s", s);
    case null -> "null";
    default -> obj.toString();
};

*// Guarded patterns (case ... when ...)*
String description = switch (obj) {
    case Integer i when i < 0 -> "negative integer";
    case Integer i when i > 0 -> "positive integer";
    case Integer i -> "zero";
    case String s when s.length() > 5 -> "long string";
    case String s -> "short string";
    default -> "something else";
};

*// Combining with sealed classes*
Shape shape = getShape();
double area = switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t -> 0.5 * t.base() * t.height();
};  *// No default needed with sealed classes// Null handling*
String result = switch (obj) {
    case null -> "It's null";
    case String s -> "String: " + s;
    default -> "Not a string";
};

*// Multiple patterns with same action*
int category = switch (obj) {
    case Integer i, Long l, Short s, Byte b -> 1;  *// All integer types*
    case Float f, Double d -> 2;  *// All floating point types*
    case Boolean b -> 3;  *// Boolean*
    case Character c -> 4;  *// Character*
    default -> -1;  *// Other types*
};
```

**Key Features:**

- **Type Patterns**: Match on the type of the value
- **Guarded Patterns**: Add conditions to case labels
- **Null Handling**: Explicit case for null values
- **Exhaustiveness**: Compiler checks for completeness
- **Combined Patterns**: Multiple patterns with same result

> **Deep Dive Tip:** Pattern matching for switch builds on two previous features: switch expressions (Java 14) and pattern matching for instanceof (Java 16). It extends the type pattern syntax from instanceof to switch contexts, allowing for more concise and expressive handling of multi-type logic. Under the hood, this is implemented with dynamic dispatch (for final classes) and/or class checks with additional casts, optimized by the JVM. The pattern variables introduced in case labels are scoped to the associated code block and are effectively final.
> 

> **Interviewer Insight:** When discussing pattern matching for switch, emphasize how it addresses a common source of verbosity and error in Java code: handling values of different types with type-specific logic. Before this feature, developers would use long chains of instanceof checks with casts, or implement the visitor pattern for more complex cases. Pattern matching for switch makes this code more concise, readable, and safer by combining type checking, casting, and conditional logic in a single construct. This feature represents Java's ongoing evolution toward more expressive pattern matching capabilities, following the lead of functional languages while maintaining Java's strong typing.
> 

### Enhanced Pseudo-Random Number Generators

Java 17 introduced a new API for random number generation with improved design and capabilities.

```java
*// Legacy approach*
Random random = new Random();
int value = random.nextInt(100);  *// 0-99// New API with multiple algorithms// Default algorithm (same as Random)*
RandomGenerator generator = RandomGenerator.getDefault();
int value1 = generator.nextInt(100);  *// 0-99// Specific algorithm (L32X64MixRandom - better quality)*
RandomGenerator xoroshiro = RandomGeneratorFactory.of("L32X64MixRandom")
    .create(42);  *// Seed for reproducibility*
int value2 = xoroshiro.nextInt(100);  *// 0-99// Get all available algorithms*
RandomGeneratorFactory.all()
    .map(factory -> factory.name())
    .sorted()
    .forEach(System.out::println);

*// Stream generation*
IntStream randomInts = generator.ints(10, 1, 101);  *// 10 ints from 1-100*
randomInts.forEach(System.out::println);

*// Using a splittable generator for parallel streams*
SplittableRandomGenerator splittable = RandomGeneratorFactory.of("SplittableMT")
    .create();

*// Split the generator for parallel usage*
SplittableRandomGenerator split1 = splittable.split();
SplittableRandomGenerator split2 = splittable.split();

*// Different threads can use different splits independently// Thread 1*
IntStream stream1 = split1.ints(1000);
*// Thread 2*
IntStream stream2 = split2.ints(1000);
```

**Key Features:**

- **Multiple Algorithms**: Choose algorithm based on needs
- **Common Interface**: Unified API for all generators
- **Factory Pattern**: Create generators by name
- **Stream Generation**: Create streams of random values
- **Splittable Generators**: Efficient parallel generation

**Algorithm Selection:**

- **L32X64MixRandom**: Fast, good quality (default)
- **Xoroshiro128PlusPlus**: Excellent quality, very fast
- **Xoshiro256PlusPlus**: Highest quality
- **LXMGenerator**: Cryptographically strong
- **SplittableRandom**: For parallel streams

> **Deep Dive Tip:** The new RandomGenerator API separates the generator interface from its implementation, allowing different algorithms to be selected based on specific needs. This is important because random number generation involves trade-offs between speed, quality, and period (how long before the sequence repeats). For example, LXMGenerator provides cryptographic-quality randomness but is slower, while Xoroshiro128PlusPlus offers excellent statistical quality with high performance. The SplittableRandom variants are specifically designed for parallel applications, allowing a generator to be split into independent streams that can be used concurrently without coordination.
> 

> **Interviewer Insight:** When discussing the enhanced PRNGs, emphasize that this API modernization addresses several limitations of the legacy Random class:
> 
> 1. It provides access to modern, high-quality algorithms developed in the last decade
> 2. It separates interface from implementation, enabling algorithm selection
> 3. It standardizes stream generation across all implementations
> 4. It supports parallel applications through splittable generators
> 
> This demonstrates Java's commitment to updating foundational APIs with modern designs and capabilities. For applications where randomness quality matters (simulations, gaming, statistical sampling), these improvements can be significant.
> 

### Strong Encapsulation of JDK Internals

Java 17 strengthened the encapsulation of internal JDK APIs, limiting access to sensitive implementation details.

```java
*// Before Java 16, could access internal JDK classes// Using reflection to access sun.misc.Unsafe*
try {
    Field f = Unsafe.class.getDeclaredField("theUnsafe");
    f.setAccessible(true);  *// This would succeed in earlier versions*
    Unsafe unsafe = (Unsafe) f.get(null);
    *// Use unsafe...*
} catch (Exception e) {
    e.printStackTrace();
}

*// In Java 17, this fails by default// Must use command-line flag to allow it:// --add-opens java.base/jdk.internal.misc=ALL-UNNAMED// Proper alternative: use standardized APIs// For memory operations: java.nio.ByteBuffer*
ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
buffer.putInt(0, 42);
int value = buffer.getInt(0);

*// For reflection operations: setAccessible() still works on application classes// but not on internal JDK classes by default*
Field field = MyClass.class.getDeclaredField("privateField");
field.setAccessible(true);  *// This still works for your own classes*
```

**Migration Strategies:**

1. **Use public APIs**: Replace internal API usage with standard alternatives
2. **Command-line flags**: For legacy code, use `-add-opens` or `-add-exports`
3. **Third-party libraries**: Use maintained libraries that handle compatibility
4. **Upgrade dependencies**: Update libraries that rely on internal APIs

> **Deep Dive Tip:** The strong encapsulation is implemented through the module system. Each JDK module explicitly controls which packages it exports and opens. Internal packages that aren't exported cannot be accessed via reflection unless explicitly opened with command-line flags. This protection applies even to unnamed modules (code on the classpath). The primary motivation is security and maintainability—internal APIs can change between releases without warning, and some provide access to sensitive operations that could compromise security.
> 

> **Interviewer Insight:** When discussing strong encapsulation, emphasize that it represents Java's commitment to long-term maintainability and security, even when it requires breaking changes. Applications depending on internal JDK classes were always living on borrowed time, as these APIs were never guaranteed to remain stable. The proper approach is to:
> 
> 1. Audit your codebase for uses of `sun.*`, `jdk.internal.*`, and other internal packages
> 2. Identify proper public API alternatives for each use case
> 3. For cases where no alternative exists, use the `-add-opens` flag as a temporary solution
> 4. Engage with the OpenJDK community to request public APIs for missing functionality
> 
> This strengthening of encapsulation boundaries improves Java's long-term evolution by allowing internal implementations to change without breaking dependent code.
> 

### Other Features

Java 17 included several other important features and improvements.

### Context-Specific Deserialization Filters

```java
*// Setting up a deserialization filter*
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "java.base/*;!java.lang.Process");

*// Apply filter to a specific ObjectInputStream*
try (ObjectInputStream ois = new ObjectInputStream(inputStream)) {
    ois.setObjectInputFilter(filter);
    Object obj = ois.readObject();
    *// Process obj safely...*
}

*// Context-specific filter factory (Java 17 feature)*
ObjectInputFilter.Config.setSerialFilterFactory(new ObjectInputFilter.FilterFactory() {
    @Override
    public ObjectInputFilter apply(FilterInfo filterInfo) {
        Class<?> serialClass = filterInfo.serialClass();
        if (serialClass != null && serialClass.getName().startsWith("com.example")) {
            *// Custom filter for our own classes*
            return info -> ObjectInputFilter.Status.ALLOWED;
        } else {
            *// Default filter for other classes*
            return ObjectInputFilter.Config.getSerialFilter();
        }
    }
});
```

### Removal of RMI Activation

```java
*// RMI Activation (removed in Java 17)// This code no longer works:/**
ActivationGroupDesc groupDesc = new ActivationGroupDesc(props, null);
ActivationGroupID groupID = ActivationGroup.getSystem().registerGroup(groupDesc);
ActivationDesc desc = new ActivationDesc(groupID, "MyService", "file:myapp.jar", null);
MyService service = (MyService) Activatable.register(desc);
**/// Alternative: Use regular RMI without activation*
MyService service = new MyServiceImpl();
MyService stub = (MyService) UnicastRemoteObject.exportObject(service, 0);
Registry registry = LocateRegistry.createRegistry(1099);
registry.bind("MyService", stub);
```

### Deprecate Security Manager

```java
*// Security Manager is deprecated for removal// Don't use in new code:*
System.setSecurityManager(new SecurityManager());  *// Deprecated// Better alternatives:// 1. JVM sandbox tools (like jshell --execution=untrusted)// 2. Container isolation (Docker, etc.)// 3. Module system for code boundaries// 4. Process isolation for untrusted code*
```

### Foreign Function & Memory API (Incubator)

```java
*// Access native memory and call native functions (incubator)*
import jdk.incubator.foreign.*;

*// Allocate native memory*
try (ResourceScope scope = ResourceScope.newConfinedScope()) {
    MemorySegment segment = MemorySegment.allocateNative(100, scope);
    
    *// Write to memory*
    MemoryAccess.setInt(segment, 0, 42);
    
    *// Read from memory*
    int value = MemoryAccess.getInt(segment, 0);
    System.out.println("Value: " + value);
    
    *// Call native function (C library)*
    LibraryLookup stdlib = LibraryLookup.ofDefault();
    MethodHandle strlen = CLinker.getInstance().downcallHandle(
        stdlib.lookup("strlen").get(),
        MethodType.methodType(long.class, MemoryAddress.class),
        FunctionDescriptor.of(CLinker.C_LONG, CLinker.C_POINTER)
    );
    
    try (ResourceScope scope2 = ResourceScope.newConfinedScope()) {
        *// Create a C string*
        byte[] bytes = "Hello, World!".getBytes(StandardCharsets.UTF_8);
        MemorySegment cString = MemorySegment.allocateNative(bytes.length + 1, scope2);
        MemoryAccess.copyFromArray(bytes, 0, cString, 0, bytes.length);
        
        *// Call strlen*
        long length = (long) strlen.invoke(cString.address());
        System.out.println("String length: " + length);  *// 13*
    }
}
```

### Always-Strict Floating-Point

```java
*// Java 17 uses strict floating-point semantics by default// No need for strictfp modifier anymore// Old code:*
strictfp class MathClass {
    public double calculate(double a, double b) {
        return (a * b) / (a + b);
    }
}

*// Java 17+ equivalent (implicit strictfp):*
class MathClass {
    public double calculate(double a, double b) {
        return (a * b) / (a + b);
    }
}
```

> **Deep Dive Tip:** The Foreign Function & Memory API (incubator in Java 17) represents a significant evolution in Java's native interoperability. Unlike JNI, which requires writing and compiling C code, this API allows direct interaction with native libraries from pure Java code. It provides memory safety through managed lifetimes with ResourceScope, type safety through explicit memory layouts, and efficient performance through direct memory access. This API is particularly valuable for applications that need to interact with hardware, implement memory-mapped files, or call into native libraries without the complexity of JNI.
> 

> **Interviewer Insight:** When discussing Java 17's feature set, emphasize its role as an LTS release that solidifies several important evolutionary threads in the platform:
> 
> 1. **Pattern Matching**: Sealed classes finalized, switch patterns previewed
> 2. **Memory Management**: Foreign Memory API incubating
> 3. **Security**: Strong encapsulation enforced, serialization filters improved
> 4. **Legacy Cleanup**: RMI Activation and Security Manager deprecated
> 
> These changes reflect Java's balanced approach to evolution: introducing new capabilities while maintaining backward compatibility where possible, but also being willing to remove problematic legacy features when necessary. As an LTS release, Java 17 represents a stable foundation for applications that will be maintained for years, with enough new features to justify migration.
> 

## 9.5 Java 18-21+ Features

Java versions 18 through 21 continued Java's evolution with several significant features, building on the foundations established in earlier releases.

### Java 18 Features

Released in March 2022, Java 18 introduced several improvements and new incubating features.

### UTF-8 by Default

```java
*// Prior to Java 18 - platform default encoding varied// Windows might use windows-1252, macOS might use UTF-8, etc.*
byte[] bytes = "Hello, 世界".getBytes();  *// Uses platform default encoding// Java 18+ - UTF-8 is the default*
byte[] bytesUtf8 = "Hello, 世界".getBytes();  *// Always UTF-8*
String str = new String(bytesUtf8);  *// Always decoded as UTF-8// Explicitly specifying another encoding still works*
byte[] bytesLatin1 = "Hello, World".getBytes(StandardCharsets.ISO_8859_1);
String strLatin1 = new String(bytesLatin1, StandardCharsets.ISO_8859_1);

*// Check the default charset*
System.out.println(Charset.defaultCharset());  *// UTF-8*
```

**Key Points:**

- All platforms now use UTF-8 by default
- Improves cross-platform consistency
- Affects file I/O, network communication, etc.
- Can be overridden with system property if needed

### Simple Web Server

```java
*// Running the built-in web server from command line:// $ jwebserver -p 8000 -d /path/to/directory// Programmatic usage*
import com.sun.net.httpserver.*;

*// Create and start a simple HTTP file server*
var server = SimpleFileServer.createFileServer(
    new InetSocketAddress(8000),
    Path.of("/path/to/serve"),
    SimpleFileServer.OutputLevel.VERBOSE
);
server.start();

*// Create a handler with authentication*
var handler = SimpleFileServer.createFileHandler(Path.of("/path/to/serve"));
var authenticator = SimpleFileServer.createBasicAuthenticator("realm", Map.of(
    "user", "password".toCharArray()
));

var secureServer = HttpServer.create(new InetSocketAddress(8000), 0);
var context = secureServer.createContext("/", handler);
context.setAuthenticator(authenticator);
secureServer.start();
```

**Key Features:**

- Built-in HTTP server for serving static files
- Minimal setup required
- Useful for development, testing, and demos
- Configurable port, directory, and logging level
- Support for basic authentication

### Code Snippets in JavaDoc

```java
*/***
 * Example method to demonstrate code snippets in JavaDoc.
 * 
 ** {@snippet :*
 *   // This is a code snippet with syntax highlighting
 **   var list = new ArrayList<String>();*
 *   list.add("Example");
 **   for (var item : list) {* *       System.out.println(item);
 **   } * }* * 
 * You can also highlight specific parts:
 ** {@snippet :*
 *   var result = obj.process(input); // @highlight
 ** }* * 
 * Or reference external files:
 ** {@snippet file="Example.java" region="example-region"} */*
public void exampleMethod() {
    *// Implementation*
}
```

**Key Features:**

- Better code example formatting in documentation
- Syntax highlighting support
- Highlight specific lines or regions
- Reference external source files
- Language-appropriate escaping

### Internet Address Resolution SPI

```java
*// Service provider interface for custom hostname resolution*
import java.net.spi.InetAddressResolver;
import java.net.spi.InetAddressResolverProvider;

*// Custom resolver provider implementation*
public class CustomResolverProvider extends InetAddressResolverProvider {
    @Override
    public InetAddressResolver get(Configuration configuration) {
        return new CustomResolver(configuration);
    }
    
    @Override
    public String name() {
        return "CustomResolver";
    }
}

*// Custom resolver implementation*
class CustomResolver implements InetAddressResolver {
    private final Configuration config;
    
    CustomResolver(Configuration config) {
        this.config = config;
    }
    
    @Override
    public Stream<InetAddress> lookupByName(String host, LookupPolicy policy) {
        *// Custom hostname resolution logic// ...*
    }
    
    @Override
    public String lookupByAddress(byte[] addr) {
        *// Custom reverse lookup logic// ...*
    }
}

*// Using the system resolver (default behavior)*
InetAddress address = InetAddress.getByName("example.com");
```

**Key Points:**

- Enables custom hostname resolution strategies
- Useful for specialized network environments
- Service provider interface pattern for extensibility
- Can implement DNS-over-HTTPS, custom DNS servers, etc.

### Java 19 Features

Released in September 2022, Java 19 continued the development of several incubating features and introduced new preview features.

### Virtual Threads (Preview)

```java
*// Virtual threads preview (finalized in Java 21)// Creating a virtual thread*
Thread vt = Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread");
});

*// Using the builder pattern*
Thread vt2 = Thread.ofVirtual()
    .name("worker-thread")
    .start(() -> {
        System.out.println("Named virtual thread");
    });

*// Virtual thread per task executor*
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    *// Submit many tasks*
    for (int i = 0; i < 10_000; i++) {
        int taskId = i;
        executor.submit(() -> {
            *// This creates 10,000 virtual threads*
            System.out.println("Task " + taskId + " in " + Thread.currentThread());
            return taskId;
        });
    }
} *// Executor shuts down automatically*
```

**Key Features:**

- Lightweight threads (few kilobytes vs megabytes)
- Enables millions of concurrent threads
- Managed by JVM, not OS
- Simplifies concurrency model
- Ideal for I/O-bound applications

### Structured Concurrency (Incubator)

```java
*// Structured concurrency incubator*
import jdk.incubator.concurrent.StructuredTaskScope;

*// Example: fetch user and order details concurrently*
class UserOrderInfo {
    User user;
    Order order;
    UserOrderInfo(User user, Order order) {
        this.user = user;
        this.order = order;
    }
}

UserOrderInfo fetchUserAndOrder(int userId, int orderId) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        *// Fork both tasks*
        Future<User> userFuture = scope.fork(() -> fetchUser(userId));
        Future<Order> orderFuture = scope.fork(() -> fetchOrder(orderId));
        
        *// Wait for both to complete (or either to fail)*
        scope.join();
        
        *// Handle any exceptions*
        scope.throwIfFailed(e -> new RuntimeException("Failed to fetch data", e));
        
        *// Both tasks completed successfully*
        return new UserOrderInfo(userFuture.resultNow(), orderFuture.resultNow());
    }
}

*// First-result scope example*
String findFirstAvailableMirror(List<String> mirrors) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
        *// Fork a task for each mirror*
        for (String mirror : mirrors) {
            scope.fork(() -> checkMirrorAvailability(mirror));
        }
        
        *// Wait for first successful result*
        scope.join();
        
        *// Get the first successful result*
        return scope.result();
    }
}
```

**Key Concepts:**

- Task scopes with defined lifetimes
- Parent-child relationship between tasks
- Structured teardown of subtasks
- Error handling strategies
- First-result or all-results patterns

### Record Patterns (Preview)

```java
*// Record patterns preview*
record Point(int x, int y) {}
record Rectangle(Point topLeft, Point bottomRight) {}

*// Using record patterns to destructure records*
void printRectangle(Object obj) {
    if (obj instanceof Rectangle(Point(int x1, int y1), Point(int x2, int y2))) {
        int width = x2 - x1;
        int height = y2 - y1;
        System.out.println("Rectangle with width " + width + " and height " + height);
    }
}

*// With switch expressions (requires pattern matching for switch)*
String describe(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == y -> "Diagonal point at " + x;
        case Point(int x, int y) -> "Point at " + x + ", " + y;
        case Rectangle(Point(int x1, int y1), Point(int x2, int y2)) ->
            "Rectangle from " + x1 + "," + y1 + " to " + x2 + "," + y2;
        default -> "Unknown shape";
    };
}
```

**Key Features:**

- Nested pattern matching for records
- Destructuring record components
- Seamless integration with pattern matching
- Type safety through patterns

### Java 20 Features

Released in March 2023, Java 20 continued to refine several preview features.

### Scoped Values (Incubator)

```java
*// Scoped values incubator (alternative to ThreadLocal)*
import jdk.incubator.concurrent.ScopedValue;

*// Define a scoped value*
final ScopedValue<String> TRANSACTION_ID = ScopedValue.newInstance();

*// Use the scoped value within a scope*
void processRequest() {
    ScopedValue.where(TRANSACTION_ID, generateTransactionId())
        .run(() -> {
            processFirstPart();
            processSecondPart();
        });
}

void processFirstPart() {
    *// Access the scoped value*
    String txId = TRANSACTION_ID.get();
    System.out.println("Processing in transaction: " + txId);
}

void processSecondPart() {
    *// Same value is visible here*
    String txId = TRANSACTION_ID.get();
    System.out.println("Continuing transaction: " + txId);
}

*// Values are inherited by child virtual threads*
void processWithSubtasks() {
    ScopedValue.where(TRANSACTION_ID, "tx-12345")
        .run(() -> {
            *// Spawn multiple tasks that inherit the scoped value*
            try (var scope = new StructuredTaskScope<Void>()) {
                scope.fork(() -> {
                    *// Same TRANSACTION_ID is visible here*
                    System.out.println("Subtask 1: " + TRANSACTION_ID.get());
                    return null;
                });
                
                scope.fork(() -> {
                    *// And here*
                    System.out.println("Subtask 2: " + TRANSACTION_ID.get());
                    return null;
                });
                
                scope.join();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
}
```

**Key Features:**

- Thread-local values with structured lifecycles
- Immutable (can't be changed after setting)
- Automatically inherited by child virtual threads
- More efficient than ThreadLocal with virtual threads
- Seamless integration with structured concurrency

### Foreign Function & Memory API (2nd Preview)

```java
*// Foreign Function & Memory API (2nd preview)*
import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;

*// Access native memory*
try (Arena arena = Arena.ofConfined()) {
    *// Allocate memory*
    MemorySegment segment = arena.allocate(100);
    
    *// Write values to memory*
    segment.set(ValueLayout.JAVA_INT, 0, 42);
    
    *// Read values from memory*
    int value = segment.get(ValueLayout.JAVA_INT, 0);
    System.out.println("Value: " + value);
    
    *// Call C standard library function*
    Linker linker = Linker.nativeLinker();
    SymbolLookup stdlib = linker.defaultLookup();
    
    *// Find the strlen function*
    MethodHandle strlen = linker.downcallHandle(
        stdlib.lookup("strlen").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS)
    );
    
    *// Create a C string*
    MemorySegment cString = arena.allocateUtf8String("Hello, World!");
    
    *// Call strlen*
    try {
        long length = (long) strlen.invoke(cString);
        System.out.println("String length: " + length);
    } catch (Throwable e) {
        throw new RuntimeException(e);
    }
}
```

**Key Improvements:**

- Simplified memory management with Arena
- Enhanced layout system
- Improved native function calling
- Better support for complex data structures
- Safer memory access patterns

### Java 21 Features (LTS)

Released in September 2023, Java 21 is a Long-Term Support (LTS) release that finalized several important features from previous preview versions.

### Virtual Threads (Final)

```java
*// Virtual threads (finalized in Java 21)// Creating virtual threads*
Thread vt = Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread");
});

*// Thread-per-task executor (recommended pattern)*
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    *// Submit many tasks without concern for thread count*
    List<Future<String>> futures = new ArrayList<>();
    
    for (int i = 0; i < 10_000; i++) {
        int id = i;
        futures.add(executor.submit(() -> {
            *// Simulate blocking I/O*
            Thread.sleep(100);
            return "Result from task " + id;
        }));
    }
    
    *// Get results*
    for (Future<String> future : futures) {
        System.out.println(future.get());
    }
}

*// Blocking operations with virtual threads*
void processRequests() {
    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        for (Request request : getRequests()) {
            executor.submit(() -> {
                *// These blocking operations are now efficient with virtual threads*
                Response response = sendHttpRequest(request);  *// Blocks*
                saveToDatabase(response);  *// Blocks*
                sendNotification(response);  *// Blocks*
                return null;
            });
        }
    }
}
```

**Key Features:**

- Lightweight threads managed by JVM
- Enables millions of concurrent threads
- Automatically yields during blocking operations
- Simplifies concurrent programming model
- Thread-per-request model becomes practical
- No API changes for thread code

### Pattern Matching for Switch (Final)

```java
*// Pattern matching for switch (finalized in Java 21)// Basic type patterns*
String formatted = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case Long l -> String.format("long %d", l);
    case Double d -> String.format("double %f", d);
    case String s -> String.format("String %s", s);
    case null -> "null";
    default -> obj.toString();
};

*// Guarded patterns*
String describe = switch (obj) {
    case String s when s.length() > 10 -> "Long string";
    case String s -> "Short string";
    case Integer i when i > 0 -> "Positive number";
    case Integer i when i < 0 -> "Negative number";
    case Integer i -> "Zero";
    default -> "Something else";
};

*// Record patterns*
record Point(int x, int y) {}
record Rectangle(Point topLeft, Point bottomRight) {}
record Circle(Point center, int radius) {}

String shapeDescription = switch (shape) {
    case Rectangle(Point(var x1, var y1), Point(var x2, var y2)) ->
        String.format("Rectangle from (%d,%d) to (%d,%d)", x1, y1, x2, y2);
    case Circle(Point(var x, var y), var r) ->
        String.format("Circle at (%d,%d) with radius %d", x, y, r);
    case null -> "No shape";
    default -> "Unknown shape";
};
```

**Key Features:**

- Type patterns for instanceof-like matching
- Record patterns for destructuring
- Guarded patterns with when conditions
- Null handling with explicit case null
- Exhaustiveness checking for sealed types

### Record Patterns (Final)

```java
*// Record patterns (finalized in Java 21)// Basic record destructuring*
record Person(String name, int age) {}
record Employee(Person person, String department) {}

*// Pattern matching with nested records*
void processPerson(Object obj) {
    if (obj instanceof Person(String name, int age)) {
        System.out.println("Person: " + name + ", " + age + " years old");
    }
}

void processEmployee(Object obj) {
    if (obj instanceof Employee(Person(String name, int age), String dept)) {
        System.out.println(name + " (" + age + ") works in " + dept);
    }
}

*// Combined with switch pattern matching*
String getEmployeeInfo(Object obj) {
    return switch (obj) {
        case Employee(Person(String name, int age), String dept) when age > 50 ->
            name + " is a senior employee in " + dept;
        case Employee(Person(String name, int age), String dept) ->
            name + " works in " + dept;
        case Person(String name, int age) ->
            name + " is not an employee";
        default -> "Not a person";
    };
}
```

**Key Features:**

- Destructure records into components
- Nested pattern matching for complex structures
- Type checking and binding in a single step
- Works with both instanceof and switch
- Enhances record usability

### Sequenced Collections

```java
*// Sequenced collections framework// Lists now implement SequencedCollection*
List<String> list = new ArrayList<>(List.of("first", "middle", "last"));

*// New methods from SequencedCollection*
String first = list.getFirst();  *// "first"*
String last = list.getLast();    *// "last"*

list.addFirst("new first");
list.addLast("new last");

*// Reverse view*
List<String> reversed = list.reversed();
System.out.println(reversed);  *// [new last, last, middle, first, new first]// Sets now implement SequencedSet*
NavigableSet<Integer> set = new TreeSet<>(List.of(1, 2, 3, 4, 5));
Integer firstElement = set.getFirst();  *// 1*
Integer lastElement = set.getLast();    *// 5// Maps now implement SequencedMap*
NavigableMap<String, Integer> map = new TreeMap<>();
map.put("a", 1);
map.put("b", 2);
map.put("c", 3);

*// First/last entries*
Map.Entry<String, Integer> firstEntry = map.firstEntry();  *// a=1*
Map.Entry<String, Integer> lastEntry = map.lastEntry();    *// c=3// Put at beginning/end*
map.putFirst("z", 26);  *// Now z is first despite lexicographical order*
map.putLast("y", 25);   *// Now y is last despite lexicographical order// Reverse view*
Map<String, Integer> reversedMap = map.reversed();
```

**Key Interfaces:**

- **SequencedCollection<E>**: Collection with first/last element access
- **SequencedSet<E>**: Set with defined iteration order
- **SequencedMap<K,V>**: Map with defined entry order

**New Methods:**

- **getFirst()/getLast()**: Access first/last elements
- **addFirst()/addLast()**: Add at beginning/end
- **removeFirst()/removeLast()**: Remove from beginning/end
- **reversed()**: Get reverse-ordered view

> **Deep Dive Tip:** The Sequenced Collections API formalizes the concept of collections with a defined encounter order. Before Java 21, methods like `getFirst()` and `getLast()` were scattered across different collection classes with inconsistent naming. This API unifies these operations under a common hierarchy, making it easier to work with ordered collections in a uniform way. The reversed() method is particularly powerful, as it provides a view of the collection in reverse order without copying the elements, making it very efficient for operations that need bidirectional access.
> 

### String Templates (Preview)

```java
*// String templates preview// Basic string interpolation*
String name = "John";
int age = 30;
String message = STR."Hello, \{name}! You are \{age} years old.";
System.out.println(message);  *// Hello, John! You are 30 years old.// Expressions in templates*
int a = 10;
int b = 20;
String calculation = STR."\{a} + \{b} = \{a + b}";
System.out.println(calculation);  *// 10 + 20 = 30// Multi-line templates*
String html = STR."""
    <html>
        <body>
            <h1>\{name}'s Profile</h1>
            <p>Age: \{age}</p>
            <p>Email: \{name.toLowerCase()}@example.com</p>
        </body>
    </html>
    """;

*// Using the FMT processor for formatting*
import java.time.LocalDate;

LocalDate today = LocalDate.now();
double price = 123.456;
String formatted = FMT."""
    Date: \{today}
    Price: \{price}
    """;
*// Uses DateTimeFormatter and NumberFormat for locale-aware formatting// Custom template processor*
StringTemplate st = STR."\{name} is \{age} years old.";
String json = JSON.process(st);  *// Hypothetical JSON processor// Would produce: {"name":"John","age":30}*
```

**Key Features:**

- String interpolation with expressions
- Multiple template processors (STR, FMT, etc.)
- Multi-line text block support
- Type-safe template processing
- Extensible processor API

### Unnamed Patterns and Variables

```java
*// Unnamed patterns in record patterns*
record Point(int x, int y) {}
record Rectangle(Point topLeft, Point bottomRight) {}

*// Using unnamed patterns (underscore) for components we don't need*
if (obj instanceof Rectangle(Point(int x1, int y1), Point(_, _))) {
    *// We only care about the top-left corner*
    System.out.println("Rectangle starts at (" + x1 + "," + y1 + ")");
}

*// Unnamed variables for unused method parameters*
class Logger {
    public void log(String message, int _ */* unused level */*) {
        System.out.println(message);
    }
}

*// Unnamed variables in lambda expressions*
list.forEach(item -> {
    *// Only use the item, don't need the index*
    System.out.println(item);
});

*// Try-with-resources where resource is not used*
try (var _ = acquireResource()) {
    *// We only care about the resource being closed, not using it*
    doSomething();
}
```

**Key Features:**

- Use underscore (`_`) for unused variables or patterns
- Makes intent clearer in code
- Avoids compiler warnings for unused variables
- Works in various contexts (patterns, parameters, etc.)

> **Interviewer Insight:** When discussing Java 21 features, emphasize that this LTS release brings several key transformative features to production readiness. Virtual threads fundamentally change Java's concurrency model, making it far more efficient for I/O-bound applications without requiring a complete rewrite. Pattern matching for switch and records brings modern pattern matching capabilities that simplify working with complex data structures. These features collectively modernize Java's programming model while maintaining backward compatibility, highlighting Java's balanced approach to evolution—providing modern language features while respecting its enterprise roots and stability requirements.
> 

## Summary of Java 9-21+ Evolution

The evolution of Java from version 9 to 21+ demonstrates several key trends:

### 1. Functional and Declarative Programming

- **Java 9**: Enhanced Stream API, improved Optional
- **Java 10+**: Collection factory methods, immutable collectors
- **Java 16+**: Records for immutable data
- **Java 21+**: Pattern matching in switch and records

### 2. Language Expressiveness

- **Java 10**: Local variable type inference (`var`)
- **Java 14-17**: Switch expressions, text blocks
- **Java 16-21**: Pattern matching, records, sealed classes
- **Java 21**: String templates (preview)

### 3. Performance and Resource Efficiency

- **Java 9**: Compact Strings, optimized collections
- **Java 10-14**: Garbage collector improvements (G1, ZGC, Shenandoah)
- **Java 19-21**: Virtual threads for scalable concurrency
- **Java 14-21**: Foreign Memory API for efficient native memory access

### 4. Modularity and Encapsulation

- **Java 9**: Module System (JPMS)
- **Java 16-17**: Strong encapsulation of JDK internals
- **Java 15-17**: Sealed classes for controlled inheritance
- **Java 17+**: Progressive removal of internal/deprecated APIs

### 5. Developer Productivity

- **Java 9**: JShell for interactive development
- **Java 10-11**: `var` for reduced verbosity
- **Java 11**: Simplified HTTP client, file methods
- **Java 17-21**: Structured concurrency, virtual threads, pattern matching

> **Deep Dive Tip:** Java's evolution strategy involves an innovative approach using preview features and incubator modules. This allows language designers to gather feedback from the community before finalizing features, resulting in more polished and practical APIs. Major features like records, switch expressions, and virtual threads all went through multiple preview cycles before being standardized, with significant improvements based on real-world usage. This approach balances innovation with Java's traditional emphasis on stability and backward compatibility.
> 

> **Interviewer Insight:** When discussing Java's evolution across versions 9-21, emphasize that it represents a significant modernization of the language and platform while maintaining backward compatibility. The changes address key pain points in Java development (verbosity, concurrency complexity, boilerplate code) and introduce modern programming paradigms without abandoning Java's core strengths. Understanding this broader evolution helps developers make informed decisions about when to adopt new features and how to evolve their coding style as the language advances. A skilled Java developer today embraces these new features selectively, combining them with traditional Java patterns where appropriate, rather than treating them as separate paradigms.
> 

This completes our in-depth exploration of Java 9-21+ Features. These versions collectively represent a substantial evolution of the Java platform, bringing modern language features, improved performance, and enhanced developer productivity while maintaining the stability and backward compatibility that Java is known for.

---

## ✈️ Happy Coding!

---