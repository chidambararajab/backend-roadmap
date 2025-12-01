# Module 6: Generics

## 6.1 Generic Basics

Generics enable the creation of type-safe classes, interfaces, and methods that work with different data types while providing compile-time type safety.

### Generic Classes

```java
*// Basic generic class*
public class Box<T> {
    private T content;
    
    public Box(T content) {
        this.content = content;
    }
    
    public T getContent() {
        return content;
    }
    
    public void setContent(T content) {
        this.content = content;
    }
}

*// Usage*
Box<String> stringBox = new Box<>("Hello Generics");
String content = stringBox.getContent();  *// No casting needed*

Box<Integer> intBox = new Box<>(42);
Integer number = intBox.getContent();
```

**Implementation Details:**

- **Type Parameter Syntax**: Class name followed by type parameter(s) in angle brackets
- **Type Safety**: Compiler enforces type constraints
- **Backward Compatibility**: Compiled to use Object and casts (type erasure)
- **Generic Field**: Field type uses the type parameter

**Multiple Type Parameters:**

```java
*// Generic class with multiple type parameters*
public class Pair<K, V> {
    private K key;
    private V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    *// Getters and setters*
    public K getKey() { return key; }
    public V getValue() { return value; }
    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
}

*// Usage*
Pair<String, Integer> entry = new Pair<>("Age", 30);
String key = entry.getKey();
Integer value = entry.getValue();
```

**Deep Dive Tip:** Behind the scenes, the Java compiler replaces all generic type parameters with their bounds or Object if unbounded. This process is called "type erasure." As a result, the compiled bytecode contains no information about generic types - they exist only at compile-time. This design decision was made for backward compatibility with pre-generic Java code.

### Generic Methods

```java
*// Generic method in non-generic class*
public class Utilities {
    public static <T> T getFirst(List<T> list) {
        if (list == null || list.isEmpty()) {
            return null;
        }
        return list.get(0);
    }
    
    *// Generic method with multiple type parameters*
    public static <K, V> Map<K, V> zipToMap(List<K> keys, List<V> values) {
        Map<K, V> map = new HashMap<>();
        int size = Math.min(keys.size(), values.size());
        
        for (int i = 0; i < size; i++) {
            map.put(keys.get(i), values.get(i));
        }
        
        return map;
    }
}

*// Usage*
String first = Utilities.<String>getFirst(Arrays.asList("a", "b", "c"));
*// Type inference allows omitting explicit type*
String first2 = Utilities.getFirst(Arrays.asList("a", "b", "c"));

Map<String, Integer> map = Utilities.zipToMap(
    Arrays.asList("A", "B", "C"),
    Arrays.asList(1, 2, 3)
);
```

**Implementation Details:**

- **Method-Level Type Parameters**: Declared before return type
- **Type Inference**: Java can often infer the type parameter from context
- **Scope**: Type parameters are only in scope for that method
- **Static Methods**: Can be generic even in non-generic classes

**Generic Constructors:**

```java
public class DataContainer {
    private Object data;
    
    *// Generic constructor*
    public <T> DataContainer(T data) {
        this.data = data;
    }
    
    @SuppressWarnings("unchecked")
    public <T> T getData() {
        return (T) data;
    }
}

*// Usage*
DataContainer container = new DataContainer("Hello");
String data = container.<String>getData();
*// Or with type inference*
String data2 = container.getData();  *// Warning: unchecked cast*
```

**Interviewer Insight:** Generic methods are particularly powerful because they allow for type safety even in non-generic classes. They're commonly used for utility methods that need to operate on different types. When reviewing code, look for opportunities to convert methods with manual casting to generic methods for better type safety.

### Type Parameter Conventions

Java has established conventions for naming type parameters:

- **E** - Element (used extensively by the Java Collections Framework)
- **K** - Key
- **V** - Value
- **N** - Number
- **T** - Type
- **S, U, V, etc.** - Additional types
- **R** - Return type

```java
*// Examples of conventional type parameter naming*
public class Container<T> { */* ... */* }
public interface List<E> { */* ... */* }
public interface Map<K, V> { */* ... */* }

public class Utilities {
    *// T is a general type*
    public static <T> void swap(T[] array, int i, int j) {*/* ... */*}
    
    *// R is a return type, different from T*
    public static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {*/* ... */*}
    
    *// E is an element type*
    public static <E extends Comparable<E>> E max(Collection<E> collection) {*/* ... */*}
    
    *// K for key, V for value*
    public static <K, V> Map<K, V> createMap(K key, V value) {*/* ... */*}
}
```

**Deep Dive Tip:** These naming conventions are not enforced by the compiler but are widely followed in the Java ecosystem. Following these conventions makes your code more readable and intuitive for other Java developers. The Java Collections Framework is a great example of consistent type parameter naming, with List<E>, Set<E>, Map<K,V>, etc.

## 6.2 Advanced Generics

Advanced generics cover more complex usage patterns including bounded type parameters, wildcards, and generic inheritance.

### Bounded Type Parameters

```java
*// Upper bound - T must be Number or a subclass of Number*
public class NumericBox<T extends Number> {
    private T number;
    
    public NumericBox(T number) {
        this.number = number;
    }
    
    public double square() {
        return number.doubleValue() * number.doubleValue();
    }
    
    public T getNumber() {
        return number;
    }
}

*// Usage*
NumericBox<Integer> intBox = new NumericBox<>(5);
double squared = intBox.square();  *// 25.0// This would not compile// NumericBox<String> stringBox = new NumericBox<>("Hello");  // Error*
```

**Multiple Bounds:**

```java
*// T must implement both Comparable and Serializable*
public class SortableBox<T extends Comparable<T> & Serializable> {
    private T content;
    
    *// Methods...*
    public int compareTo(SortableBox<T> other) {
        return this.content.compareTo(other.content);
    }
    
    public void saveToFile(String filename) throws IOException {
        *// Can serialize because T is Serializable*
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename))) {
            out.writeObject(content);
        }
    }
}

*// Usage*
SortableBox<String> box = new SortableBox<>();  *// Valid, String is both Comparable and Serializable// SortableBox<StringBuilder> box2 = new SortableBox<>();  // Error: StringBuilder isn't Serializable*
```

**Deep Dive Tip:** When using multiple bounds, the first bound can be a class or an interface, but all subsequent bounds must be interfaces. This is because Java doesn't support multiple inheritance for classes. If a class bound is used, it must be listed first. For example: `<T extends AbstractList & Serializable & Cloneable>` is valid, but `<T extends Serializable & AbstractList>` would cause a compile error.

**Interviewer Insight:** Bounded type parameters are a powerful way to express requirements on generic types and enable more functionality within generic classes. When designing a generic API, consider what methods you need to call on the type parameters and add appropriate bounds to ensure those operations are available.

### Wildcards

Wildcards allow for more flexible generic code by representing unknown types.

### Unbounded Wildcards

```java
*// Method that works with any type of list*
public static void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

*// Usage*
List<String> strings = Arrays.asList("A", "B", "C");
List<Integer> integers = Arrays.asList(1, 2, 3);
printList(strings);
printList(integers);
```

**Limitations:**

- Cannot add elements to a collection with unbounded wildcard (except null)
- Can only read elements as Object

### Upper Bounded Wildcards

```java
*// Method that works with list of Number or any subclass*
public static double sumOfList(List<? extends Number> list) {
    double sum = 0.0;
    for (Number number : list) {
        sum += number.doubleValue();
    }
    return sum;
}

*// Usage*
List<Integer> integers = Arrays.asList(1, 2, 3);
List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
double sum1 = sumOfList(integers);  *// 6.0*
double sum2 = sumOfList(doubles);   *// 6.6*
```

**Limitations:**

- Cannot add elements to the collection (except null)
- Can read elements as the upper bound type

### Lower Bounded Wildcards

```java
*// Method that can add Integers to a list of Integer or any superclass*
public static void addNumbers(List<? super Integer> list) {
    for (int i = 1; i <= 5; i++) {
        list.add(i);  *// Valid*
    }
}

*// Usage*
List<Number> numbers = new ArrayList<>();
List<Object> objects = new ArrayList<>();
addNumbers(numbers);  *// Valid: Integer is a subclass of Number*
addNumbers(objects);  *// Valid: Integer is a subclass of Object*
```

**Limitations:**

- Can add elements of the lower bound type or its subclasses
- Can only read elements as Object

### PECS Principle (Producer Extends, Consumer Super)

```java
*// Producer - we get values from it (extends)*
public static <T> void copy(List<? extends T> source, List<? super T> dest) {
    for (T item : source) {
        dest.add(item);
    }
}

*// Usage*
List<Integer> integers = Arrays.asList(1, 2, 3);
List<Number> numbers = new ArrayList<>();
copy(integers, numbers);  *// Copying from a "producer" to a "consumer"*
```

**Deep Dive Tip:** The PECS principle (also known as the Get and Put Principle) is a guideline for using wildcards effectively:

- Use `? extends T` when you only need to "get" values from a structure (producer)
- Use `? super T` when you only need to "put" values into a structure (consumer)
- Use specific type `T` when you need to both get and put with exact type

This principle helps in designing flexible APIs that work with a wider range of types while maintaining type safety.

**Interviewer Insight:** Understanding when to use which type of wildcard is a common challenge. A good rule of thumb: if your method only reads from a data structure, use "extends"; if it only writes to a data structure, use "super"; if it does both, avoid wildcards and use specific types.

### Type Erasure

Type erasure is the process by which the Java compiler replaces generic type information with its bounds or Object if unbounded, to maintain backward compatibility.

```java
*// Before type erasure*
public class Box<T> {
    private T content;
    
    public T getContent() {
        return content;
    }
    
    public void setContent(T content) {
        this.content = content;
    }
}

*// After type erasure (conceptual equivalent)*
public class Box {
    private Object content;
    
    public Object getContent() {
        return content;
    }
    
    public void setContent(Object content) {
        this.content = content;
    }
}
```

**Bridge Methods:**

```java
*public class StringBox extends Box<String> {
    @Override
    public String getContent() {
        return "String: " + super.getContent();
    }
}
// After type erasure, compiler generates a bridge method:/*
public Object getContent() {  // Bridge method
    return getContent();  // Calls the String version
}
*/*
```

**Implications of Type Erasure:**

1. Cannot overload methods with different generic type parameters but same erasure
2. Cannot create arrays of generic types
3. Cannot use instanceof with generic types
4. Static fields are shared among all instances regardless of type parameter
5. Exceptions and catch blocks cannot use generic type parameters

**Deep Dive Tip:** Bridge methods are synthetic methods generated by the compiler to ensure proper method overriding in the presence of generics. When a subclass overrides a generic method with a more specific return type, the compiler creates a bridge method with the erased signature that delegates to the actual implementation. This allows for proper polymorphism despite type erasure.

### Generic Inheritance

```java
*// Generic class hierarchy*
interface Repository<T> {
    T findById(long id);
    void save(T entity);
}

*// Extending with the same type parameter*
class UserRepository implements Repository<User> {
    @Override
    public User findById(long id) {
        *// Implementation*
        return new User(id);
    }
    
    @Override
    public void save(User entity) {
        *// Implementation*
    }
}

*// Extending with a different type parameter*
class GenericRepository<E> implements Repository<E> {
    @Override
    public E findById(long id) {
        *// Implementation*
        return null;
    }
    
    @Override
    public void save(E entity) {
        *// Implementation*
    }
}

*// Usage*
Repository<User> userRepo = new UserRepository();
Repository<Product> productRepo = new GenericRepository<>();
```

**Type Parameter Inheritance:**

```java
class Base<T> {
    protected T data;
    
    public T getData() {
        return data;
    }
}

*// Inheriting the type parameter*
class Derived1<T> extends Base<T> {
    public void setData(T newData) {
        data = newData;
    }
}

*// Fixing the type parameter*
class StringBase extends Base<String> {
    public int getDataLength() {
        return data.length();
    }
}

*// Adding a new type parameter*
class Derived2<T, U> extends Base<T> {
    private U additionalData;
    
    public U getAdditionalData() {
        return additionalData;
    }
}
```

**Interviewer Insight:** When designing generic class hierarchies, you have three main options for handling type parameters: propagate them (pass them along), fix them (specify a concrete type), or add new ones. The choice depends on what flexibility you want to provide to subclasses and clients of your API.

### Generic Restrictions

Java generics have several important restrictions that developers need to be aware of:

### No Primitive Types

```java
*// Not allowed// List<int> numbers = new ArrayList<>();  // Error// Workaround: use wrapper classes*
List<Integer> numbers = new ArrayList<>();

*// Java 10+ local variable type inference can reduce verbosity*
var numbers = new ArrayList<Integer>();
```

### No Static Fields of Type Parameter Type

```java
public class Container<T> {
    *// Not allowed// private static T defaultValue;  // Error*
    
    *// Workaround: use static generic methods*
    private static Object defaultValue;
    
    @SuppressWarnings("unchecked")
    public static <E> E getDefaultValue() {
        return (E) defaultValue;
    }
    
    public static <E> void setDefaultValue(E value) {
        defaultValue = value;
    }
}
```

### No instanceof with Generics

```java
public <T> boolean checkType(Object obj, Class<T> clazz) {
    *// Not allowed// if (obj instanceof T) { ... }  // Error*
    
    *// Workaround: use Class.isInstance*
    return clazz.isInstance(obj);
}

*// Usage*
boolean isString = checkType("test", String.class);
```

### No Arrays of Generic Types

```java
*// Not allowed// T[] array = new T[10];  // Error// Workaround 1: Object array with cast*
@SuppressWarnings("unchecked")
public <T> T[] createArray(int size) {
    *// Unchecked cast, but unavoidable*
    return (T[]) new Object[size];
}

*// Workaround 2: Pass Class object and use Array.newInstance*
public <T> T[] createArray(Class<T> clazz, int size) {
    *// Safe but requires explicit class token*
    @SuppressWarnings("unchecked")
    T[] array = (T[]) Array.newInstance(clazz, size);
    return array;
}

*// Usage*
String[] strings = createArray(String.class, 10);
```

### No Instantiation of Type Parameters

```java
public <T> T createInstance() {
    *// Not allowed// return new T();  // Error*
    
    *// Workaround 1: Pass Class object with no-arg constructor*
    public <T> T createInstance(Class<T> clazz) throws Exception {
        return clazz.getDeclaredConstructor().newInstance();
    }
    
    *// Workaround 2: Factory pattern*
    public <T> T createInstance(Supplier<T> factory) {
        return factory.get();
    }
}

*// Usage*
String s = createInstance(String.class);
List<String> list = createInstance(ArrayList::new);
```

**Deep Dive Tip:** Most of these restrictions stem from type erasure. Since generic type information is not available at runtime, operations that require runtime type information (like instanceof or new T()) cannot work with generic type parameters. The workarounds typically involve passing explicit Class objects or other runtime type tokens.

**Interviewer Insight:** Understanding these limitations is crucial for designing effective generic APIs. In practice, many generic libraries use techniques like type tokens (passing Class<T> objects), factories (Supplier<T>), or other patterns to work around these restrictions. Recognizing these patterns in existing code is important for understanding how complex generic libraries work.

## 6.3 Type Inference

Type inference is the ability of the Java compiler to determine generic types from context, reducing verbosity and improving readability.

### Diamond Operator (Java 7+)

```java
*// Pre-Java 7*
Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();

*// Java 7+ with diamond operator*
Map<String, List<Integer>> map = new HashMap<>();

*// Works with complex nested generics*
List<Map<String, Set<Integer>>> complex = new ArrayList<>();
```

**Implementation Details:**

- Compiler infers the type arguments from the left-hand side declaration
- Applies to constructor calls in variable declarations, assignments, and method arguments
- Cannot be used with anonymous inner classes in Java 7 (added in Java 9)

**Java 9+ Anonymous Class Improvement:**

```java
*// Pre-Java 9*
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};

*// Java 9+*
Comparator<String> comp = new Comparator<>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};
```

**Interviewer Insight:** The diamond operator was one of the most welcomed features in Java 7, significantly reducing the verbosity of generic code. However, be aware that it only works when the compiler can infer the type from context. In rare cases where type inference fails, you may need to provide explicit type arguments.

### Improved Type Inference (Java 8+)

Java 8 significantly enhanced type inference in several contexts:

### Method Invocation Context

```java
*// Java 7 required explicit type parameters*
List<String> list = Collections.<String>emptyList();

*// Java 8+ can infer from assignment context*
List<String> list = Collections.emptyList();
```

### Chained Method Calls

```java
*// Pre-Java 8 (might require explicit type parameters)*
<String, Integer>doSomething().andThenSomethingElse();

*// Java 8+ improved inference in method chains*
doSomething().andThenSomethingElse();  *// Types inferred from context*
```

### Lambda Expressions

```java
*// Method that takes a function*
public <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
    List<R> result = new ArrayList<>();
    for (T item : list) {
        result.add(mapper.apply(item));
    }
    return result;
}

*// Java 8+ type inference with lambdas*
List<Integer> lengths = map(Arrays.asList("a", "bb", "ccc"), s -> s.length());
*// Compiler infers: T is String, R is Integer*
```

**Deep Dive Tip:** Java 8's type inference improvements were driven by the introduction of lambda expressions and method references, which required better contextual type inference to be practical. The enhanced algorithm looks at the "target type" (the expected type based on context) and uses that information to infer generic type parameters.

### Local Variable Type Inference

### var Keyword (Java 10+)

```java
*// Pre-Java 10*
List<Map<String, List<Integer>>> complex = new ArrayList<>();

*// Java 10+ with var*
var complex = new ArrayList<Map<String, List<Integer>>>();
```

**Implementation Details:**

- Compiler infers the type from the initializer expression
- Variable must be initialized when declared
- Only applies to local variables, not fields, method parameters, or return types
- The inferred type is fixed at compile-time (not dynamic typing)

### var with Generics

```java
*// Without var*
Map<String, List<Integer>> map = new HashMap<>();
for (Map.Entry<String, List<Integer>> entry : map.entrySet()) {
    *// ...*
}

*// With var*
var map = new HashMap<String, List<Integer>>();
for (var entry : map.entrySet()) {
    *// ...*
}
```

### var in Loops

```java
*// Traditional for loop*
for (int i = 0; i < 10; i++) {
    *// ...*
}

*// With var*
for (var i = 0; i < 10; i++) {
    *// ...*
}

*// Enhanced for loop*
List<String> names = List.of("Alice", "Bob", "Charlie");
for (var name : names) {
    *// ...*
}
```

### var in try-with-resources

```java
*// Pre-Java 10*
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    *// ...*
}

*// Java 10+*
try (var reader = new BufferedReader(new FileReader("file.txt"))) {
    *// ...*
}
```

### var in Lambda Parameters (Java 11+)

```java
*// Pre-Java 11*
Function<String, Integer> f = (String s) -> s.length();

*// Java 11+*
Function<String, Integer> f = (var s) -> s.length();

*// With multiple parameters*
BiFunction<String, String, Integer> sum = (var s1, var s2) -> s1.length() + s2.length();
```

**Usage Guidelines:**

- **Good for:**
    - Complex generic types
    - Variables with obvious types from initialization
    - When the type name is repeated in constructor
    - Local variables with limited scope
- **Avoid for:**
    - Variables without initializers
    - Initialized with null
    - When the inferred type is not clear from context
    - When explicit type improves readability

**Deep Dive Tip:** The `var` keyword is not a dynamic typing feature but a way to let the compiler infer the static type. The variable still has a fixed, static type determined at compile-time. Unlike similar features in scripting languages, `var` doesn't change Java's fundamental static typing nature.

**Interviewer Insight:** When discussing `var` in interviews, emphasize that it's about reducing boilerplate while maintaining type safety, not about dynamic typing. A good rule of thumb is to use `var` when it improves readability and clarity, but stick with explicit types when the inferred type isn't immediately obvious from the context. Over-using `var` can make code harder to understand, especially in code bases with complex types.

---

## ✈️ Happy Coding!

---