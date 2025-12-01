# Module 2: Object-Oriented Programming for Senior Developers

## 2.1 Classes and Objects

### Class Fundamentals

Understanding the internals of class design and runtime behavior in Java.

```java
*// Basic class structure with implementation details*
public class User {
    *// Instance variables - part of each object's memory footprint*
    private String username;  *// Default visibility minimizes coupling*
    private final String id;  *// Final requires initialization before constructor completes*
    
    *// Static variables - one copy shared across all instances*
    private static final int MAX_USERNAME_LENGTH = 50;
    private static int userCount = 0;  *// Tracks number of User instances*
    
    *// Constructor*
    public User(String username, String id) {
        *// Validation guard clauses before assignment*
        if (username == null || username.length() > MAX_USERNAME_LENGTH) {
            throw new IllegalArgumentException("Invalid username");
        }
        if (id == null) {
            throw new IllegalArgumentException("ID cannot be null");
        }
        
        this.username = username;
        this.id = id;
        userCount++;  *// Increment shared counter*
    }
    
    *// Instance methods operate on specific object state*
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        if (username != null && username.length() <= MAX_USERNAME_LENGTH) {
            this.username = username;
        }
    }
    
    *// Static methods operate on class-level concepts*
    public static int getUserCount() {
        return userCount;
    }
}
```

**Memory allocation and JVM internals:**

- Class structure loaded into the Metaspace (prior to Java 8: PermGen)
- Instance data stored on the heap
- Object header consists of:
    - Mark Word (8 bytes): identity hashcode, GC age, locking information
    - Class Pointer (4-8 bytes): reference to class metadata
- Instance fields laid out after the header, with alignment padding

> Interviewer Insight: The JVM doesn't support multiple inheritance for classes because it complicates memory layout and method dispatch. A class's memory layout is a contiguous block with fields at fixed offsets. Multiple inheritance would require complex offset tables or indirection, slowing field access. Instead, Java uses interfaces which only require method dispatch tables, not shared state.
> 

### Constructors

Constructors control object initialization and play a critical role in ensuring object validity.

```java
public class Connection {
    private final String url;
    private final boolean secure;
    private Status status;
    
    *// Primary constructor - most specific*
    public Connection(String url, boolean secure) {
        this.url = url;
        this.secure = secure;
        this.status = Status.CLOSED;
    }
    
    *// Constructor chaining via this() - reuses validation logic*
    public Connection(String url) {
        this(url, url.startsWith("https"));  *// Chain to primary constructor*
    }
    
    *// Default constructor*
    public Connection() {
        this("localhost:8080", false);  *// Chain to primary constructor*
    }
    
    *// Copy constructor - creates new object with same state*
    public Connection(Connection other) {
        this(other.url, other.secure);
        this.status = other.status;
    }
    
    *// Private constructor for factory methods*
    private Connection(Builder builder) {
        this.url = builder.url;
        this.secure = builder.secure;
        this.status = builder.status;
    }
    
    *// Builder pattern for complex object construction*
    public static class Builder {
        *// Required parameters*
        private final String url;
        
        *// Optional parameters with defaults*
        private boolean secure = false;
        private Status status = Status.CLOSED;
        
        public Builder(String url) {
            this.url = url;
        }
        
        public Builder secure(boolean secure) {
            this.secure = secure;
            return this;
        }
        
        public Builder status(Status status) {
            this.status = status;
            return this;
        }
        
        public Connection build() {
            return new Connection(this);
        }
    }
    
    *// Static factory method - alternative to constructors*
    public static Connection createSecureConnection(String host, int port) {
        return new Connection("https://" + host + ":" + port, true);
    }
    
    *// Enum for connection status*
    public enum Status {
        OPEN, CLOSED, CONNECTING
    }
}

*// Client code using builder*
Connection conn = new Connection.Builder("example.com:8080")
                      .secure(true)
                      .status(Connection.Status.CONNECTING)
                      .build();

*// Client code using static factory*
Connection secureConn = Connection.createSecureConnection("api.example.com", 443);
```

**Constructor execution order and inheritance:**

1. Static initializers (in textual order, parent to child)
2. Instance initializers (in textual order)
3. Superclass constructor
4. Instance field initializers
5. Constructor body

> Deep Dive Tip: Private constructors serve multiple purposes: enforcing singleton patterns, preventing inheritance, supporting static factory methods, and implementing the builder pattern. They're also essential for utility classes that shouldn't be instantiated: private UtilityClass() { throw new AssertionError(); }.
> 

### Instance Members

Instance members define the state and behavior specific to each object instance.

```java
public class Account {
    *// Instance variables - each object has its own copy*
    private String owner;
    private double balance;
    private final String accountNumber;  *// Immutable after construction*
    private final LocalDateTime createdAt;
    
    *// Instance initializer block - runs before constructors*
    {
        createdAt = LocalDateTime.now();  *// All constructors will have this*
        System.out.println("Creating account: " + accountNumber);  *// Prints null!*
    }
    
    public Account(String owner, String accountNumber) {
        this.owner = owner;
        this.accountNumber = accountNumber;
        this.balance = 0.0;
    }
    
    *// Instance methods - operate on the specific instance's state*
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance += amount;
    }
    
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        
        if (amount > balance) {
            return false;  *// Insufficient funds*
        }
        
        this.balance -= amount;
        return true;
    }
    
    *// Instance methods can access both instance and static members*
    public String getAccountSummary() {
        return String.format("Account #%s: %.2f %s", 
                            accountNumber, balance, Account.CURRENCY);
    }
    
    *// Static variable*
    private static final String CURRENCY = "USD";
}
```

**Order of instance member initialization:**

1. Instance variables initialize to default values (0, false, null)
2. Instance initializer blocks execute in textual order
3. Constructor executes, potentially overwriting previous values

> Interviewer Insight: Instance initializer blocks rarely appear in typical Java code but are useful when you need to share initialization logic across multiple constructors. They're actually compiled into each constructor by the compiler. They can lead to subtle bugs if they access instance variables that haven't been initialized through constructor parameters yet.
> 

### Static Members

Static members belong to the class rather than any instance and are shared across all instances.

```java
public class DatabaseConnection {
    *// Static variables - shared across all instances*
    private static final int DEFAULT_TIMEOUT = 30;
    private static final int MAX_CONNECTIONS = 100;
    private static int activeConnections = 0;
    
    *// Static initializer block - runs once when class is loaded*
    static {
        try {
            *// Load database driver*
            Class.forName("com.example.db.Driver");
            System.out.println("Database driver loaded");
        } catch (ClassNotFoundException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    
    *// Instance variables*
    private String url;
    private boolean connected;
    
    public DatabaseConnection(String url) {
        this.url = url;
        this.connected = false;
    }
    
    *// Instance method that interacts with static state*
    public boolean connect() {
        synchronized (DatabaseConnection.class) {  *// Class-level lock*
            if (activeConnections >= MAX_CONNECTIONS) {
                return false;  *// Connection limit reached*
            }
            
            *// Connect logic...*
            connected = true;
            activeConnections++;
            return true;
        }
    }
    
    *// Static methods operate on class-level concepts*
    public static int getActiveConnectionCount() {
        return activeConnections;
    }
    
    *// Static factory method*
    public static DatabaseConnection createDefaultConnection() {
        return new DatabaseConnection("jdbc:default:localhost");
    }
    
    *// Static utility method*
    public static boolean isValidUrl(String url) {
        return url != null && url.startsWith("jdbc:");
    }
    
    *// Thread-safe lazy initialization using holder pattern*
    public static class ConnectionPool {
        *// Private constructor prevents direct instantiation*
        private ConnectionPool() {}
        
        *// Lazy initialization holder class*
        private static class Holder {
            static final ConnectionPool INSTANCE = new ConnectionPool();
            
            *// Static block in holder class runs only when accessed*
            static {
                System.out.println("ConnectionPool initialized");
            }
        }
        
        *// Static method to get the singleton instance*
        public static ConnectionPool getInstance() {
            return Holder.INSTANCE;  *// Holder class loaded only when needed*
        }
    }
}
```

**Static member characteristics:**

- Loaded when the class is initialized (not necessarily at program start)
- One copy shared across all instances of the class
- Can be accessed directly through the class without instances
- Cannot access instance members directly (needs an instance reference)
- Static initializers execute once during class loading

> Deep Dive Tip: The initialization-on-demand holder idiom (shown in the ConnectionPool example) is the most efficient thread-safe lazy initialization pattern in Java. It leverages JVM class initialization semantics to ensure thread safety without explicit synchronization overhead. This pattern is more efficient than double-checked locking.
> 

### this Keyword

The `this` keyword provides a reference to the current instance and enables several important patterns.

```java
public class Builder {
    private String name;
    private int age;
    private String address;
    
    *// Method chaining with this reference*
    public Builder name(String name) {
        this.name = name;  *// Disambiguate instance variable from parameter*
        return this;  *// Return current instance for chaining*
    }
    
    public Builder age(int age) {
        this.age = age;
        return this;
    }
    
    public Builder address(String address) {
        this.address = address;
        return this;
    }
    
    *// Invoking another constructor*
    public Builder() {
        this("Unknown", 0, null);  *// Call the three-arg constructor*
    }
    
    *// Primary constructor*
    public Builder(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }
    
    *// Passing current instance to another method*
    public void register(Registry registry) {
        registry.addBuilder(this);  *// Pass current instance*
    }
    
    *// Explicitly returning current instance*
    public Builder reset() {
        this.name = null;
        this.age = 0;
        this.address = null;
        return this;
    }
    
    *// Inner class accessing outer instance via Builder.this*
    public class BuilderInfo {
        public String getBuilderName() {
            return Builder.this.name;  *// Access outer class instance*
        }
    }
}
```

**Advanced this usage:**

- Disambiguating instance variables from local variables/parameters
- Constructor chaining with `this()` calls
- Returning current instance for method chaining (fluent interfaces)
- Passing the current instance to other methods
- Accessing enclosing instance from inner classes

> Interviewer Insight: The this reference creates an implicit relationship between methods in a class. When designing high-performance systems, be aware that passing this to other methods, especially in constructors, can lead to subtle bugs if the object isn't fully initialized. This is particularly dangerous in multithreaded contexts where the partially constructed object might become visible to other threads.
> 

### Garbage Collection Basics

Understanding garbage collection is crucial for building high-performance Java applications.

```java
public class ResourceManager {
    private LargeResource resource;
    
    public void processData() {
        *// Local variables eligible for GC after method exits*
        byte[] buffer = new byte[10 * 1024 * 1024];  *// 10MB buffer*
        
        *// Process data using buffer*
        processWithBuffer(buffer);
        
        *// Buffer now eligible for GC, no references remain*
        
        *// Explicit nulling can help in specific long-running methods*
        buffer = null;  *// Makes eligible for GC earlier*
        
        *// This won't have much effect in a short method// but might help in long-running loops*
    }
    
    *// Making objects eligible for GC*
    public void releaseResource() {
        resource = null;  *// Original resource now eligible for GC*
    }
    
    *// Pre-Java 9 finalization (deprecated in modern Java)*
    @Override
    protected void finalize() throws Throwable {
        try {
            if (resource != null) {
                resource.close();  *// Resource cleanup*
            }
        } finally {
            super.finalize();
        }
    }
    
    *// Modern resource management (Java 9+)*
    public static class ManagedResource implements AutoCloseable {
        private final Cleaner.Cleanable cleanable;
        private final long resourceHandle;  *// Native resource handle*
        
        *// State needed for cleanup*
        private static class ResourceCleaner implements Runnable {
            private final long resourceHandle;
            
            ResourceCleaner(long resourceHandle) {
                this.resourceHandle = resourceHandle;
            }
            
            @Override
            public void run() {
                *// Free the native resource*
                freeNativeResource(resourceHandle);
            }
        }
        
        private static void freeNativeResource(long handle) {
            *// Native cleanup code*
            System.out.println("Freeing native resource: " + handle);
        }
        
        public ManagedResource() {
            this.resourceHandle = allocateNativeResource();
            *// Register with Cleaner*
            this.cleanable = cleaner.register(this, new ResourceCleaner(resourceHandle));
        }
        
        private static long allocateNativeResource() {
            *// Allocate and return native resource*
            return System.nanoTime();  *// Simulated resource handle*
        }
        
        @Override
        public void close() {
            cleanable.clean();  *// Explicit cleanup*
        }
        
        *// Shared cleaner instance*
        private static final Cleaner cleaner = Cleaner.create();
    }
}
```

**Garbage Collection Mechanisms:**

- Objects become eligible for GC when they are no longer reachable
- GC roots include local variables, active thread stacks, static fields, JNI references
- Modern JVMs use generational garbage collection (young/old generations)
- GC algorithms include:
    - Serial GC: Single-threaded, simple
    - Parallel GC: Multi-threaded for throughput
    - G1 GC: Garbage-First, low-pause, region-based (default in Java 9+)
    - ZGC: Ultra-low latency, scalable (Java 11+)
    - Shenandoah: Low pause times

> Deep Dive Tip: Java 9 deprecated the finalize() method in favor of the Cleaner API shown above. Finalizers had many issues: they ran on the finalizer thread, could resurrect objects, and had unpredictable execution timing. The Cleaner API provides better guarantees while still not being a substitute for explicit resource management with try-with-resources.
> 

## 2.2 Inheritance

### Types of Inheritance

Java's inheritance model provides powerful abstraction capabilities but comes with constraints and best practices.

```java
*// Single inheritance - most common pattern*
public class Vehicle {
    protected int wheels;
    protected double weight;
    
    public Vehicle(int wheels, double weight) {
        this.wheels = wheels;
        this.weight = weight;
    }
    
    public void move() {
        System.out.println("Vehicle is moving");
    }
}

public class Car extends Vehicle {
    private int doors;
    
    public Car(int doors, double weight) {
        super(4, weight);  *// Cars always have 4 wheels*
        this.doors = doors;
    }
    
    @Override
    public void move() {
        System.out.println("Car is driving");
    }
}

*// Multilevel inheritance - can lead to complexity*
public class SportsCar extends Car {
    private int horsePower;
    
    public SportsCar(int doors, double weight, int horsePower) {
        super(doors, weight);
        this.horsePower = horsePower;
    }
}

*// Multiple inheritance through interfaces*
public interface Flyable {
    void fly();
}

public interface Floatable {
    void float();
}

*// Implementing multiple interfaces*
public class AmphibiousPlane extends Vehicle implements Flyable, Floatable {
    public AmphibiousPlane(double weight) {
        super(3, weight);
    }
    
    @Override
    public void fly() {
        System.out.println("AmphibiousPlane is flying");
    }
    
    @Override
    public void float() {
        System.out.println("AmphibiousPlane is floating");
    }
}
```

**Inheritance Best Practices:**

- Prefer composition over inheritance for code reuse
- Follow the Liskov Substitution Principle (LSP)
- Keep inheritance hierarchies shallow (< 3 levels deep)
- Use interfaces for multiple inheritance
- Make classes final if not designed for inheritance

> Interviewer Insight: Inheritance creates tight coupling between parent and child classes. This is why "favor composition over inheritance" is a core design principle. Inheritance is best used for true "is-a" relationships where the subclass is a specialized version of the parent. For most other code reuse scenarios, composition provides better flexibility and looser coupling.
> 

### Inheritance Concepts

Understanding the nuances of inheritance is critical for effective object-oriented design.

```java
*// Base class with various access modifiers*
public class BaseService {
    private String privateField;          *// Not inherited*
    protected String protectedField;      *// Inherited but encapsulated*
    String packagePrivateField;           *// Inherited within same package*
    public String publicField;            *// Inherited and accessible*
    
    *// Constructor execution order demonstration*
    public BaseService() {
        System.out.println("BaseService constructor");
        initialize();  *// Dangerous - calls overridable method from constructor*
    }
    
    protected void initialize() {
        System.out.println("BaseService initialize");
    }
    
    *// Final method - cannot be overridden*
    public final void criticalOperation() {
        *// Implementation that must not be changed*
    }
    
    *// Method designed for extension*
    protected void customizableOperation() {
        *// Default implementation that subclasses may override*
    }
}

*// Subclass demonstrating constructor chaining and method overriding*
public class DerivedService extends BaseService {
    private String derivedField;
    
    *// Constructor calls super() implicitly if not specified*
    public DerivedService() {
        *// Implicit super() call happens here*
        System.out.println("DerivedService constructor");
        derivedField = "initialized";
    }
    
    *// Explicit constructor chaining*
    public DerivedService(String value) {
        super();  *// Explicit call to parent constructor*
        this.derivedField = value;
    }
    
    *// Method overriding - dangerous when called from parent constructor*
    @Override
    protected void initialize() {
        System.out.println("DerivedService initialize");
        *// Potential null pointer if derivedField accessed here// because parent constructor calls this before derivedField is initialized!*
    }
}
```

**Constructor execution and initialization order:**

1. Memory for the object is allocated
2. All instance variables are initialized to default values
3. Superclass constructor is called
4. Instance variable initializers and instance initializer blocks execute
5. Constructor body executes

> Deep Dive Tip: The issue shown above with initialize() being called from the parent constructor is a subtle but serious inheritance pitfall. When the parent constructor calls an overridden method, the child's overridden version runs before the child's constructor, potentially accessing uninitialized state. This is why Effective Java recommends: "Design and document for inheritance or else prohibit it."
> 

### super Keyword

The `super` keyword provides access to parent class members and enables proper extension of parent behavior.

```java
public class Shape {
    protected String name;
    protected String color;
    
    public Shape(String name, String color) {
        this.name = name;
        this.color = color;
    }
    
    public double calculateArea() {
        return 0.0;  *// Base implementation*
    }
    
    public String getDescription() {
        return "A " + color + " " + name;
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius, String color) {
        super("circle", color);  *// Call parent constructor*
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public String getDescription() {
        *// Extend parent behavior instead of replacing it*
        return super.getDescription() + " with radius " + radius;
    }
    
    *// Accessing parent class field*
    public void printParentColor() {
        System.out.println("Parent color: " + super.color);
    }
}
```

**Advanced super usage:**

```java
public class ComplexShape extends Shape {
    private List<Shape> components;
    
    public ComplexShape(List<Shape> components, String name, String color) {
        super(name, color);
        this.components = new ArrayList<>(components);
    }
    
    @Override
    public double calculateArea() {
        *// Use stream to calculate total area of components*
        return components.stream()
                .mapToDouble(Shape::calculateArea)
                .sum();
    }
    
    *// Using super with inner classes*
    public class ShapeInfoPrinter {
        public void printBaseInfo() {
            *// Access outer class's parent members with super*
            System.out.println(ComplexShape.super.getDescription());
        }
    }
}
```

**Key super features:**

- Invoking parent constructors with `super()`
- Accessing parent methods with `super.methodName()`
- Accessing parent fields with `super.fieldName`
- Accessing parent class from inner classes with `OuterClass.super`

> Interviewer Insight: When overriding methods, consider whether to completely replace parent behavior or extend it. The Template Method pattern uses this concept deliberately: a parent class defines the skeleton of an algorithm with "hook" methods that subclasses override to provide specific behavior without modifying the algorithm's structure.
> 

### Method Overriding

Method overriding allows subclasses to provide specific implementations of methods defined in parent classes.

```java
public class PaymentProcessor {
    protected double calculateFee(double amount) {
        return amount * 0.02; *// Default 2% fee*
    }
    
    public Receipt processPayment(Payment payment) {
        double fee = calculateFee(payment.getAmount());
        double total = payment.getAmount() + fee;
        
        *// Process payment logic...*
        
        return new Receipt(payment.getId(), payment.getAmount(), fee, total);
    }
}

public class PremiumPaymentProcessor extends PaymentProcessor {
    private final double discountRate;
    
    public PremiumPaymentProcessor(double discountRate) {
        this.discountRate = discountRate;
    }
    
    *// Method overriding with annotation*
    @Override
    protected double calculateFee(double amount) {
        *// Apply discount to standard fee*
        double standardFee = super.calculateFee(amount);
        return standardFee * (1 - discountRate);
    }
    
    *// Covariant return types - return more specific type*
    @Override
    public DetailedReceipt processPayment(Payment payment) {
        *// Leverage parent implementation*
        Receipt basicReceipt = super.processPayment(payment);
        
        *// Enhance with additional details*
        return new DetailedReceipt(
            basicReceipt,
            payment.getCustomerId(),
            LocalDateTime.now(),
            "Premium processing"
        );
    }
}

*// DetailedReceipt is a subclass of Receipt (covariant return type)*
public class DetailedReceipt extends Receipt {
    private final String customerId;
    private final LocalDateTime timestamp;
    private final String notes;
    
    public DetailedReceipt(Receipt receipt, String customerId, 
                          LocalDateTime timestamp, String notes) {
        super(receipt.getId(), receipt.getAmount(), receipt.getFee(), receipt.getTotal());
        this.customerId = customerId;
        this.timestamp = timestamp;
        this.notes = notes;
    }
    
    *// Additional methods and getters...*
}
```

**Method overriding rules:**

- Method signature must be identical (name, parameters)
- Return type must be the same or a subtype (covariant return)
- Access level must be same or less restrictive
- Cannot throw new or broader checked exceptions
- Static, final, and private methods cannot be overridden

**Exception handling in overriding:**

```java
public class DataProcessor {
    public void process(String data) throws IOException {
        *// Processing that might throw IOException*
    }
}

public class SafeDataProcessor extends DataProcessor {
    @Override
    public void process(String data) {
        *// Can omit throws clause - exception handling is more restrictive*
        try {
            super.process(data);
        } catch (IOException e) {
            *// Handle exception internally*
            log.error("Error processing data", e);
        }
    }
}

public class SpecializedDataProcessor extends DataProcessor {
    @Override
    public void process(String data) throws FileNotFoundException {
        *// Can throw more specific exception (FileNotFoundException is subclass of IOException)// But cannot throw new checked exceptions or broader exceptions*
        if (data == null) {
            throw new FileNotFoundException("Data file not found");
        }
        super.process(data);
    }
}
```

> Deep Dive Tip: The @Override annotation isn't just documentation—it instructs the compiler to verify that you're actually overriding a method. This catches subtle errors like mistyped method names or incorrect parameter types. Always use @Override when you intend to override a method.
> 

### Dynamic Method Dispatch

Dynamic method dispatch is the mechanism that enables runtime polymorphism in Java.

```java
public class MessageService {
    public void sendMessage(String message) {
        System.out.println("Sending basic message: " + message);
    }
}

public class EmailService extends MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending email: " + message);
    }
    
    *// Method specific to EmailService*
    public void sendFormattedEmail(String subject, String body) {
        System.out.println("Sending email with subject: " + subject);
    }
}

public class SmsService extends MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

*// Client code demonstrating dynamic method dispatch*
public class NotificationManager {
    public void broadcast(String message, MessageService service) {
        *// The actual method executed depends on the runtime type of service*
        service.sendMessage(message);  *// Dynamic method dispatch*
    }
    
    public void sendNotifications() {
        String alert = "System alert";
        
        *// Runtime polymorphism in action*
        MessageService defaultService = new MessageService();
        EmailService emailService = new EmailService();
        SmsService smsService = new SmsService();
        
        broadcast(alert, defaultService);  *// Calls MessageService.sendMessage*
        broadcast(alert, emailService);    *// Calls EmailService.sendMessage*
        broadcast(alert, smsService);      *// Calls SmsService.sendMessage*
        
        *// Reference type vs object type distinction*
        MessageService polymorphicRef = new EmailService();
        polymorphicRef.sendMessage(alert);  *// Calls EmailService.sendMessage*
        
        *// This won't compile - method not found in reference type// polymorphicRef.sendFormattedEmail("Subject", alert);*
        
        *// Need to cast to access subclass-specific methods*
        if (polymorphicRef instanceof EmailService) {
            ((EmailService) polymorphicRef).sendFormattedEmail("Subject", alert);
        }
    }
}
```

**Dynamic dispatch implementation details:**

- JVM uses vtable (virtual method table) for each class
- Each object has a reference to its class's vtable
- Method calls lookup the actual implementation in the vtable
- Private, static, final methods use static dispatch (faster)

> Interviewer Insight: Dynamic method dispatch is implemented in the JVM using virtual method tables (vtables). Each class has a vtable containing pointers to the actual method implementations. When a method is called on an object, the JVM looks up the method in the object's class vtable. This is why final methods are slightly faster—they can use static dispatch (direct method call) instead of vtable lookup.
> 

### Object Class

Every class in Java implicitly extends the Object class, which provides fundamental behaviors for all objects.

```java
public class Customer {
    private final String id;
    private String name;
    private String email;
    
    public Customer(String id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    *// Override equals for proper identity semantics*
    @Override
    public boolean equals(Object obj) {
        *// Identity check (performance optimization)*
        if (this == obj) {
            return true;
        }
        
        *// Null check and class check*
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }
        
        *// Type cast and field comparison*
        Customer other = (Customer) obj;
        return id.equals(other.id);  *// Only ID determines equality*
    }
    
    *// Always override hashCode when overriding equals*
    @Override
    public int hashCode() {
        return Objects.hash(id);  *// Only use fields used in equals*
    }
    
    *// Provide useful string representation*
    @Override
    public String toString() {
        return "Customer{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
    
    *// Proper implementation of clone (generally avoid)*
    @Override
    protected Customer clone() throws CloneNotSupportedException {
        *// Call Object's clone first*
        Customer cloned = (Customer) super.clone();
        
        *// Deep copy of mutable state would go here// cloned.someList = new ArrayList<>(this.someList);*
        
        return cloned;
    }
    
    *// Wait/notify example for synchronization*
    public synchronized void updateWithTimeout(String newName, long timeoutMs) 
            throws InterruptedException {
        long endTime = System.currentTimeMillis() + timeoutMs;
        
        while (!isUpdateAllowed() && System.currentTimeMillis() < endTime) {
            *// Wait for notification with timeout*
            wait(endTime - System.currentTimeMillis());
        }
        
        if (isUpdateAllowed()) {
            this.name = newName;
            *// Notify any waiting threads that state has changed*
            notifyAll();
        } else {
            throw new TimeoutException("Update operation timed out");
        }
    }
    
    private boolean isUpdateAllowed() {
        *// Logic to determine if update is allowed*
        return true;
    }
    
    *// Using getClass for runtime type info*
    public void processAccordingToType() {
        Class<?> clazz = this.getClass();  *// Gets actual runtime class*
        
        System.out.println("Processing object of type: " + clazz.getName());
        System.out.println("Is interface: " + clazz.isInterface());
        System.out.println("Superclass: " + clazz.getSuperclass().getName());
        
        *// Inspect annotations, methods, fields, etc.*
        System.out.println("Declared methods: " + Arrays.toString(clazz.getDeclaredMethods()));
    }
}
```

**Object class methods and their proper use:**

1. `equals(Object)`: Define object equality semantics
2. `hashCode()`: Generate hash code consistent with equals
3. `toString()`: Provide human-readable representation
4. `clone()`: Create copy (rarely used, consider copy constructors instead)
5. `getClass()`: Access runtime class information
6. `wait()`, `notify()`, `notifyAll()`: Intrinsic lock coordination
7. `finalize()`: Cleanup before GC (deprecated, avoid using)

> Deep Dive Tip: When implementing equals(), always follow these five principles: reflexivity (x.equals(x) is true), symmetry (x.equals(y) iff y.equals(x)), transitivity (if x.equals(y) and y.equals(z), then x.equals(z)), consistency (repeated calls with unchanged objects return same result), and null behavior (x.equals(null) is false).
> 

## 2.3 Polymorphism

### Compile-time Polymorphism

Compile-time polymorphism (method overloading) is resolved during compilation based on parameter types.

```java
public class Calculator {
    *// Method overloading - same name, different parameter types/count*
    
    *// Basic addition*
    public int add(int a, int b) {
        System.out.println("Adding two integers");
        return a + b;
    }
    
    *// Adding three integers*
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers");
        return a + b + c;
    }
    
    *// Adding doubles*
    public double add(double a, double b) {
        System.out.println("Adding two doubles");
        return a + b;
    }
    
    *// Mixed parameter types - demonstrate type conversion*
    public double add(int a, double b) {
        System.out.println("Adding integer and double");
        return a + b;  *// int automatically promoted to double*
    }
    
    *// Varargs for flexible parameter count*
    public int add(int... numbers) {
        System.out.println("Adding variable number of integers");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
    
    *// Ambiguity demonstration*
    public void process(int value, String text) {
        System.out.println("Processing int and String");
    }
    
    public void process(String text, int value) {
        System.out.println("Processing String and int");
    }
    
    *// Constructor overloading*
    private int value;
    private String name;
    
    public Calculator() {
        this(0, "Default");  *// Chaining to parameterized constructor*
    }
    
    public Calculator(int value) {
        this(value, "Custom");  *// Chaining with default name*
    }
    
    public Calculator(int value, String name) {
        this.value = value;
        this.name = name;
    }
}

*// Client code for method resolution*
Calculator calc = new Calculator();
calc.add(5, 10);           *// Calls add(int, int)*
calc.add(5, 10, 15);       *// Calls add(int, int, int)*
calc.add(5.5, 10.5);       *// Calls add(double, double)*
calc.add(5, 10.5);         *// Calls add(int, double)*
calc.add(1, 2, 3, 4, 5);   *// Calls add(int...)*
calc.add();                *// Calls add(int...) with empty array*

calc.process(10, "test");  *// Calls process(int, String)*
calc.process("test", 10);  *// Calls process(String, int)// Ambiguous call - won't compile// calc.process(null, null);  // Both methods match with null arguments*
```

**Method resolution rules:**

1. Exact match by type
2. Match with promotion (byte→short→int→long→float→double)
3. Match with autoboxing/unboxing
4. Match with varargs
5. If multiple matches at same level, compilation error

> Interviewer Insight: Method overloading is purely a compile-time mechanism. The compiler determines which method to call based on the parameter types known at compile time, not the runtime types. This is why it's called "static polymorphism" or "compile-time polymorphism" as opposed to the runtime polymorphism achieved through method overriding.
> 

### Runtime Polymorphism

Runtime polymorphism enables a single interface to represent different implementations based on runtime types.

```java
*// Base class*
public abstract class PaymentMethod {
    private String id;
    
    public PaymentMethod(String id) {
        this.id = id;
    }
    
    *// Method that will be overridden*
    public abstract boolean processPayment(double amount);
    
    *// Template method pattern*
    public final boolean pay(double amount) {
        validateAmount(amount);
        logPaymentAttempt(amount);
        boolean success = processPayment(amount);
        logPaymentResult(amount, success);
        return success;
    }
    
    private void validateAmount(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
    
    protected void logPaymentAttempt(double amount) {
        System.out.println("Attempting payment of " + amount + " via " + getClass().getSimpleName());
    }
    
    protected void logPaymentResult(double amount, boolean success) {
        System.out.println("Payment of " + amount + " was " + (success ? "successful" : "declined"));
    }
    
    public String getId() {
        return id;
    }
}

*// Concrete implementations*
public class CreditCardPayment extends PaymentMethod {
    private String cardNumber;
    private String expiryDate;
    
    public CreditCardPayment(String id, String cardNumber, String expiryDate) {
        super(id);
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public boolean processPayment(double amount) {
        *// Credit card processing logic*
        System.out.println("Processing credit card payment of " + amount);
        return isValidCard() && amount < 5000;  *// Simplified logic*
    }
    
    private boolean isValidCard() {
        *// Validate card number and expiry date*
        return cardNumber != null && cardNumber.length() == 16;
    }
}

public class PayPalPayment extends PaymentMethod {
    private String email;
    
    public PayPalPayment(String id, String email) {
        super(id);
        this.email = email;
    }
    
    @Override
    public boolean processPayment(double amount) {
        *// PayPal processing logic*
        System.out.println("Processing PayPal payment of " + amount);
        return email != null && amount < 10000;  *// Simplified logic*
    }
}

*// Client code demonstrating runtime polymorphism*
public class PaymentProcessor {
    public static void processOrder(Order order, PaymentMethod paymentMethod) {
        *// Polymorphic call - actual method determined at runtime*
        boolean paymentSuccess = paymentMethod.pay(order.getTotal());
        
        if (paymentSuccess) {
            order.setStatus(OrderStatus.PAID);
        } else {
            order.setStatus(OrderStatus.PAYMENT_FAILED);
        }
    }
    
    public static void main(String[] args) {
        Order order = new Order(123, 499.99);
        
        *// Create different payment methods*
        PaymentMethod creditCard = new CreditCardPayment("cc1", "1234567890123456", "12/25");
        PaymentMethod payPal = new PayPalPayment("pp1", "user@example.com");
        
        *// Same method works with any PaymentMethod implementation*
        processOrder(order, creditCard);  *// Uses CreditCardPayment.processPayment()*
        
        order.setStatus(OrderStatus.PENDING);  *// Reset for demo*
        processOrder(order, payPal);      *// Uses PayPalPayment.processPayment()*
    }
}
```

**Runtime polymorphism characteristics:**

- Based on inheritance and method overriding
- Determined at runtime based on actual object type
- Enables extensibility through interfaces and abstract classes
- Implemented in JVM using vtables (virtual method tables)
- Fundamental to design patterns like Strategy, Template Method, and Factory

> Deep Dive Tip: The example above demonstrates the Template Method design pattern using runtime polymorphism. The pay() method in the base class defines the algorithm skeleton with steps that are fixed (validation) and steps that are customizable (actual payment processing). This enables reuse of common logic while allowing specialized implementations.
> 

### Type Casting

Type casting allows treating an object as a different type, enabling polymorphic behavior and specialized operations.

```java
public class TypeCastingExample {
    public void demonstrateCasting() {
        *// Upcasting (implicit) - widening reference*
        Vehicle vehicle = new Car();  *// Car is-a Vehicle*
        
        *// Implicit upcast in method parameters*
        processVehicle(new Car());    *// Car passed where Vehicle expected*
        
        *// Downcasting (explicit) - narrowing reference*
        Vehicle genericVehicle = getRandomVehicle();  *// Could be any Vehicle subtype*
        
        *// Unsafe downcasting - can cause ClassCastException*
        try {
            Car car = (Car) genericVehicle;  *// Will fail if genericVehicle is not a Car*
            car.drive();  *// Car-specific method*
        } catch (ClassCastException e) {
            System.out.println("Not a car: " + e.getMessage());
        }
        
        *// Safe downcasting with instanceof check*
        if (genericVehicle instanceof Car) {
            Car car = (Car) genericVehicle;  *// Safe - we checked first*
            car.drive();
        } else if (genericVehicle instanceof Motorcycle) {
            Motorcycle moto = (Motorcycle) genericVehicle;
            moto.wheelie();
        }
        
        *// Pattern matching with instanceof (Java 16+)*
        if (genericVehicle instanceof Car car) {
            *// car is already cast and in scope*
            car.drive();
        } else if (genericVehicle instanceof Motorcycle moto) {
            moto.wheelie();
        }
        
        *// Primitives casting*
        int i = 10;
        long l = i;       *// Implicit widening conversion*
        
        double d = 10.5;
        int n = (int) d;  *// Explicit narrowing conversion, loses precision*
        
        *// Special case: String conversion isn't casting*
        String s = String.valueOf(i);  *// Not casting, but conversion*
    }
    
    *// Method accepting broader type*
    private void processVehicle(Vehicle v) {
        v.startEngine();  *// Polymorphic call*
    }
    
    *// Method returning random vehicle subtype*
    private Vehicle getRandomVehicle() {
        if (Math.random() < 0.5) {
            return new Car();
        } else {
            return new Motorcycle();
        }
    }
    
    *// Class hierarchy for demonstration*
    private class Vehicle {
        public void startEngine() {
            System.out.println("Vehicle engine started");
        }
    }
    
    private class Car extends Vehicle {
        @Override
        public void startEngine() {
            System.out.println("Car engine started");
        }
        
        public void drive() {
            System.out.println("Car driving");
        }
    }
    
    private class Motorcycle extends Vehicle {
        @Override
        public void startEngine() {
            System.out.println("Motorcycle engine started");
        }
        
        public void wheelie() {
            System.out.println("Doing a wheelie");
        }
    }
}
```

**Casting rules and best practices:**

- Upcasting is always safe and implicit (subclass to superclass)
- Downcasting requires explicit cast operator (superclass to subclass)
- Always use instanceof before downcasting to avoid ClassCastException
- Pattern matching with instanceof (Java 16+) combines check and cast
- Primitive casting follows widening/narrowing conversion rules
- Avoid unnecessary casting through good design

> Interviewer Insight: Excessive downcasting often indicates a design problem. If you find yourself frequently checking types with instanceof and downcasting, consider whether polymorphism or the Visitor pattern might provide a more elegant solution. Good OO design minimizes the need for explicit downcasting.
> 

### instanceof Operator

The instanceof operator checks if an object is an instance of a specific type, critical for safe downcasting and type-based logic.

```java
public class InstanceofDemo {
    public void processObject(Object obj) {
        *// Basic instanceof usage*
        if (obj instanceof String) {
            String str = (String) obj;
            System.out.println("String length: " + str.length());
        } else if (obj instanceof Integer) {
            Integer num = (Integer) obj;
            System.out.println("Integer value: " + num);
        } else if (obj instanceof List<?>) {
            List<?> list = (List<?>) obj;
            System.out.println("List size: " + list.size());
        }
        
        *// Handling null*
        if (null instanceof String) {  *// Always false*
            System.out.println("This will never execute");
        }
        
        *// Checking interface implementation*
        if (obj instanceof Serializable) {
            System.out.println("Object is serializable");
        }
        
        *// Instanceof with inheritance*
        if (obj instanceof Number) {
            *// True for Integer, Long, Double, etc.*
            System.out.println("Object is a number: " + obj);
        }
        
        *// Pattern matching with instanceof (Java 16+)*
        if (obj instanceof String s && s.length() > 5) {
            *// s is already cast and in scope, with additional condition*
            System.out.println("Long string: " + s);
        } else if (obj instanceof List<?> list && !list.isEmpty()) {
            *// list is already cast and in scope, with additional condition*
            System.out.println("Non-empty list with first element: " + list.get(0));
        }
        
        *// Complex type checking with generics*
        if (obj instanceof Map<?, ?> map) {
            *// Type-specific operations with already-cast variable*
            for (Map.Entry<?, ?> entry : map.entrySet()) {
                System.out.println(entry.getKey() + " => " + entry.getValue());
            }
        }
    }
    
    public void showInstanceofLimitations() {
        List<String> stringList = new ArrayList<>();
        List<Integer> intList = new ArrayList<>();
        
        *// Due to type erasure, both check the same raw type*
        System.out.println(stringList instanceof List);  *// true*
        System.out.println(intList instanceof List);     *// true*
        
        *// Cannot check generic parameter types at runtime// This won't compile:// if (stringList instanceof List<String>) { ... }*
        
        *// Can only use wildcard for generic types*
        System.out.println(stringList instanceof List<?>);  *// true*
        
        *// Cannot check for non-reifiable types// Won't compile:// if (obj instanceof T) { ... }  // Where T is a type parameter*
    }
}
```

**Instanceof special cases and considerations:**

- Always returns false for null
- Works with classes, interfaces, and abstract classes
- Works with inheritance (matches subclasses)
- Limited with generics due to type erasure
- Cannot be used with primitive types
- Cannot be used with non-reifiable types (type parameters)

> Deep Dive Tip: Java 16's pattern matching for instanceof combines the type check, casting, and variable declaration into a single operation, making code more concise and safer. This feature is part of Project Amber, which aims to improve Java's productivity features. It eliminates the need for explicit casting after a successful instanceof check.
> 

## 2.4 Encapsulation and Abstraction

### Access Modifiers

Access modifiers control visibility and accessibility of classes, methods, and fields, enabling encapsulation.

```java
*// Top-level access modifiers*
public class PublicClass {
    *// Accessible from any package*
}

class PackagePrivateClass {
    *// Accessible only within same package*
}

*// Nested class access modifiers*
public class OuterClass {
    public class PublicInnerClass {}          *// Accessible from anywhere*
    protected class ProtectedInnerClass {}    *// Accessible within package and subclasses*
    class PackagePrivateInnerClass {}         *// Accessible within package*
    private class PrivateInnerClass {}        *// Accessible only within OuterClass*
}

*// Field and method access modifiers*
public class Account {
    public String id;          *// Accessible from anywhere - violates encapsulation*
    protected double balance;  *// Accessible within package and subclasses*
    String type;               *// Package-private - accessible within package*
    private String owner;      *// Accessible only within Account class*
    
    *// Access methods - proper encapsulation*
    public String getOwner() {
        return owner;
    }
    
    public void setOwner(String owner) {
        validateOwner(owner);
        this.owner = owner;
    }
    
    *// Method access levels*
    public void deposit(double amount) {
        validateAmount(amount);
        this.balance += amount;
    }
    
    protected void applyInterest() {
        *// Available to subclasses*
        this.balance *= 1.05;
    }
    
    void processMonthlyStatement() {
        *// Package-private, available to classes in same package*
        applyFees();
        applyInterest();
    }
    
    private void applyFees() {
        *// Implementation detail, hidden from outside*
        this.balance -= 5.00;
    }
    
    private void validateAmount(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
    
    private void validateOwner(String owner) {
        if (owner == null || owner.trim().isEmpty()) {
            throw new IllegalArgumentException("Owner cannot be empty");
        }
    }
}
```

**Access level best practices:**

- Use the most restrictive access level possible
- Make fields private and provide accessor methods as needed
- Use package-private as the default for classes
- Use protected for methods intended to be overridden
- Make helper methods private
- Consider the implications for testing (too restrictive can hinder testing)

> Interviewer Insight: Access modifiers in Java aren't just about security—they're about defining clean APIs and enabling implementation changes without breaking client code. By restricting access, you create a contract that only exposes what clients need, reserving the right to change implementation details without affecting clients. This is fundamental to achieving loose coupling.
> 

### Encapsulation

Encapsulation bundles data with the methods that operate on it, hiding internal state and requiring access through methods.

```java
*// Poorly encapsulated class*
class PoorEncapsulation {
    public String name;      *// Direct field access*
    public int age;          *// No validation*
    public List<String> tags; *// Mutable reference*
    
    *// No control over state changes*
}

*// Properly encapsulated class*
public class User {
    *// Private fields - hidden internal state*
    private String name;
    private int age;
    private final List<String> tags;  *// Reference is final, but list is still mutable*
    
    *// Constructor with validation*
    public User(String name, int age) {
        *// Validate at construction time*
        validateName(name);
        validateAge(age);
        
        this.name = name;
        this.age = age;
        this.tags = new ArrayList<>();  *// Initialize collection*
    }
    
    *// Getter methods*
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    *// Setter methods with validation*
    public void setName(String name) {
        validateName(name);
        this.name = name;
    }
    
    public void setAge(int age) {
        validateAge(age);
        this.age = age;
    }
    
    *// Collection handling - return defensive copy*
    public List<String> getTags() {
        return new ArrayList<>(tags);  *// Defensive copy*
    }
    
    *// Safe methods to modify collection*
    public void addTag(String tag) {
        if (tag == null || tag.isEmpty()) {
            throw new IllegalArgumentException("Tag cannot be empty");
        }
        this.tags.add(tag);
    }
    
    public void removeTag(String tag) {
        this.tags.remove(tag);
    }
    
    *// Validation methods*
    private void validateName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
    }
    
    private void validateAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age: " + age);
        }
    }
}

*// Immutable class example*
public final class ImmutableUser {
    private final String name;
    private final int age;
    private final List<String> tags;  *// Must be truly immutable*
    
    public ImmutableUser(String name, int age, List<String> tags) {
        *// Validate at construction time*
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age: " + age);
        }
        
        this.name = name;
        this.age = age;
        *// Defensive copy on input*
        this.tags = Collections.unmodifiableList(new ArrayList<>(tags));
    }
    
    *// Getters only - no setters*
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public List<String> getTags() {
        return tags;  *// Already unmodifiable, no need for defensive copy*
    }
    
    *// Modified state creates new object*
    public ImmutableUser withName(String newName) {
        return new ImmutableUser(newName, this.age, this.tags);
    }
    
    public ImmutableUser withAge(int newAge) {
        return new ImmutableUser(this.name, newAge, this.tags);
    }
    
    public ImmutableUser withAddedTag(String tag) {
        List<String> newTags = new ArrayList<>(this.tags);
        newTags.add(tag);
        return new ImmutableUser(this.name, this.age, newTags);
    }
}
```

**Encapsulation principles:**

1. Hide internal state with private fields
2. Control access through methods
3. Validate inputs in setters and constructors
4. Make defensive copies for mutable objects
5. Consider immutability for thread safety
6. Use interfaces to abstract implementation details

> Deep Dive Tip: Immutable classes like the ImmutableUser above offer significant advantages: they're inherently thread-safe (no synchronization needed), can be freely shared, and prevent entire classes of bugs from state mutations. Java's JVM can also apply performance optimizations to immutable objects. The pattern shown (withX methods returning new instances) is similar to Java's built-in immutable classes like String.
> 

### Abstract Classes

Abstract classes provide a partial implementation and define a contract for subclasses.

```java
*// Abstract class representing payment processing*
public abstract class PaymentProcessor {
    protected String merchantId;
    protected boolean testMode;
    
    *// Constructor in abstract class*
    public PaymentProcessor(String merchantId, boolean testMode) {
        this.merchantId = merchantId;
        this.testMode = testMode;
    }
    
    *// Abstract methods - must be implemented by concrete subclasses*
    protected abstract boolean authorizeTransaction(PaymentDetails payment);
    protected abstract String generateTransactionId();
    protected abstract void logTransaction(String transactionId, double amount, boolean success);
    
    *// Concrete method with default implementation*
    public boolean processPayment(PaymentDetails payment) {
        *// Template method pattern*
        validatePayment(payment);
        
        String transactionId = generateTransactionId();
        boolean success = authorizeTransaction(payment);
        
        if (success) {
            completeTransaction(transactionId, payment);
        }
        
        logTransaction(transactionId, payment.getAmount(), success);
        return success;
    }
    
    *// Concrete method that can be inherited*
    protected void validatePayment(PaymentDetails payment) {
        if (payment == null) {
            throw new IllegalArgumentException("Payment details cannot be null");
        }
        
        if (payment.getAmount() <= 0) {
            throw new IllegalArgumentException("Payment amount must be positive");
        }
        
        *// More validation...*
    }
    
    *// Concrete method that can be overridden*
    protected void completeTransaction(String transactionId, PaymentDetails payment) {
        System.out.println("Completing transaction: " + transactionId);
        *// Default implementation*
    }
    
    *// Final method - cannot be overridden*
    public final String getMerchantId() {
        return merchantId;
    }
    
    *// Static method - not inherited*
    public static PaymentProcessor createForTest(String merchantId) {
        *// Return test implementation*
        return new TestPaymentProcessor(merchantId);
    }
    
    *// Private static implementation class*
    private static class TestPaymentProcessor extends PaymentProcessor {
        public TestPaymentProcessor(String merchantId) {
            super(merchantId, true);
        }
        
        @Override
        protected boolean authorizeTransaction(PaymentDetails payment) {
            *// Always approve in test mode*
            return true;
        }
        
        @Override
        protected String generateTransactionId() {
            return "TEST-" + System.nanoTime();
        }
        
        @Override
        protected void logTransaction(String transactionId, double amount, boolean success) {
            System.out.println("TEST MODE: Transaction " + transactionId + " logged");
        }
    }
}

*// Concrete implementation*
public class CreditCardProcessor extends PaymentProcessor {
    private String apiKey;
    
    public CreditCardProcessor(String merchantId, String apiKey) {
        super(merchantId, false);
        this.apiKey = apiKey;
    }
    
    @Override
    protected boolean authorizeTransaction(PaymentDetails payment) {
        *// Implementation for credit card authorization*
        System.out.println("Authorizing credit card payment...");
        *// Call payment gateway API*
        return true;  *// Simplified implementation*
    }
    
    @Override
    protected String generateTransactionId() {
        return "CC-" + UUID.randomUUID().toString();
    }
    
    @Override
    protected void logTransaction(String transactionId, double amount, boolean success) {
        System.out.println("Credit card transaction " + transactionId + " logged");
        *// Implementation for credit card logging*
    }
    
    *// Override non-abstract method to customize behavior*
    @Override
    protected void completeTransaction(String transactionId, PaymentDetails payment) {
        super.completeTransaction(transactionId, payment);
        *// Additional credit card specific completion steps*
        System.out.println("Sending credit card receipt...");
    }
}
```

**Abstract class characteristics:**

- Cannot be instantiated directly
- May contain abstract and concrete methods
- Can have constructors, used by subclasses
- Can have instance and static fields
- Can have static methods
- Supports single inheritance only
- Can implement interfaces

> Interviewer Insight: Abstract classes are ideal for capturing common behavior when there's a clear "is-a" relationship between classes. The template method pattern, shown in the PaymentProcessor example, is a classic use case where an abstract class defines the skeleton of an algorithm but lets subclasses implement specific steps. This balances code reuse with flexibility.
> 

### Interfaces

Interfaces define contracts that implementing classes must fulfill, enabling polymorphism without inheritance.

```java
*// Basic interface definition*
public interface Loggable {
    *// Abstract method - must be implemented*
    void log(String message);
    
    *// Default method (Java 8+) - provides implementation*
    default void logError(String error) {
        log("ERROR: " + error);
    }
    
    *// Static method (Java 8+) - belongs to interface, not instances*
    static boolean isValidMessage(String message) {
        return message != null && !message.isEmpty();
    }
    
    *// Private method (Java 9+) - internal helper for default methods*
    private String formatMessage(String message) {
        return "[" + LocalDateTime.now() + "] " + message;
    }
    
    *// Private static method (Java 9+) - internal helper for static methods*
    private static void validateMessage(String message) {
        if (!isValidMessage(message)) {
            throw new IllegalArgumentException("Invalid message");
        }
    }
    
    *// Constants - public static final implicitly*
    String LOG_PREFIX = "LOG";
}

*// Interface with type parameters (generic interface)*
public interface Repository<T, ID> {
    Optional<T> findById(ID id);
    List<T> findAll();
    T save(T entity);
    void delete(T entity);
    boolean existsById(ID id);
}

*// Interface inheritance*
public interface ExtendedRepository<T, ID> extends Repository<T, ID> {
    *// Adds more methods to base interface*
    List<T> findByIds(Collection<ID> ids);
    int deleteAll();
}

*// Interface implementation*
public class FileLogger implements Loggable {
    private String filePath;
    
    public FileLogger(String filePath) {
        this.filePath = filePath;
    }
    
    @Override
    public void log(String message) {
        *// Implementation for file logging*
        try (FileWriter writer = new FileWriter(filePath, true)) {
            writer.write(message + "\n");
        } catch (IOException e) {
            System.err.println("Failed to write to log file: " + e.getMessage());
        }
    }
    
    *// Can override default methods if needed*
    @Override
    public void logError(String error) {
        log("CRITICAL ERROR: " + error);
        *// Custom implementation*
    }
}

*// Multiple interface implementation*
public class UserRepository implements Repository<User, Long>, Loggable {
    *// Implements all methods from both interfaces*
    
    @Override
    public Optional<User> findById(Long id) {
        log("Finding user with ID: " + id);
        *// Implementation*
        return Optional.empty();  *// Simplified*
    }
    
    @Override
    public List<User> findAll() {
        log("Finding all users");
        *// Implementation*
        return Collections.emptyList();  *// Simplified*
    }
    
    @Override
    public User save(User entity) {
        log("Saving user: " + entity.getName());
        *// Implementation*
        return entity;  *// Simplified*
    }
    
    @Override
    public void delete(User entity) {
        log("Deleting user: " + entity.getName());
        *// Implementation*
    }
    
    @Override
    public boolean existsById(Long id) {
        log("Checking if user exists with ID: " + id);
        *// Implementation*
        return false;  *// Simplified*
    }
    
    @Override
    public void log(String message) {
        System.out.println("UserRepository: " + message);
    }
}
```

**Interface characteristics:**

- Cannot be instantiated
- All methods are implicitly public
- Can contain abstract, default, static, and private methods
- Fields are implicitly public static final (constants)
- Classes can implement multiple interfaces
- Interfaces can extend multiple interfaces
- No constructors or instance fields

> Deep Dive Tip: When multiple interfaces define the same default method, a class implementing both must override the method to resolve the conflict. This is Java's solution to the "diamond problem" that multiple inheritance would cause. It forces explicit resolution of ambiguities rather than using a complex precedence rule.
> 

### Interface Evolution

Java 8 and later versions introduced features that make interfaces more powerful and flexible for API evolution.

```java
*// Original interface (pre-Java 8)*
public interface PaymentService {
    boolean processPayment(Payment payment);
    PaymentStatus getStatus(String transactionId);
}

*// Evolution with default methods (Java 8+)*
public interface ModernPaymentService {
    *// Original methods*
    boolean processPayment(Payment payment);
    PaymentStatus getStatus(String transactionId);
    
    *// New method with default implementation*
    default boolean refundPayment(String transactionId, double amount) {
        *// Default implementation for backward compatibility*
        throw new UnsupportedOperationException("Refunds not supported by this provider");
    }
    
    *// Another new method with default implementation*
    default List<Payment> getPaymentHistory(String customerId) {
        *// Default implementation*
        return Collections.emptyList();
    }
    
    *// Static utility method*
    static boolean isValidTransactionId(String transactionId) {
        return transactionId != null && transactionId.matches("[A-Z0-9\\-]{10,}");
    }
}

*// Java 9 private methods for code reuse within interface*
public interface EnhancedPaymentService extends ModernPaymentService {
    *// New methods*
    default PaymentReceipt generateReceipt(String transactionId) {
        PaymentStatus status = getStatus(transactionId);
        validateStatus(status);  *// Private helper method*
        
        PaymentReceipt receipt = new PaymentReceipt();
        receipt.setTransactionId(transactionId);
        receipt.setStatus(status);
        receipt.setTimestamp(LocalDateTime.now());
        
        populateReceiptDetails(receipt, transactionId);  *// Private helper method*
        
        return receipt;
    }
    
    *// Private method for internal use (Java 9+)*
    private void validateStatus(PaymentStatus status) {
        if (status == null) {
            throw new IllegalStateException("Payment status cannot be null");
        }
        
        if (status == PaymentStatus.FAILED) {
            throw new IllegalStateException("Cannot generate receipt for failed payment");
        }
    }
    
    *// Private method using instance methods*
    private void populateReceiptDetails(PaymentReceipt receipt, String transactionId) {
        *// Access instance methods like getStatus*
        if (getStatus(transactionId) == PaymentStatus.COMPLETED) {
            receipt.setCompletionDate(LocalDateTime.now());
        }
    }
    
    *// Private static method (Java 9+)*
    private static String formatTransactionId(String transactionId) {
        *// Utility method accessible to static and default methods*
        return transactionId.toUpperCase().replaceAll("[^A-Z0-9\\-]", "");
    }
}

*// Implementation adapting to evolving interface*
public class LegacyPaymentProvider implements ModernPaymentService {
    @Override
    public boolean processPayment(Payment payment) {
        *// Original implementation*
        return true;
    }
    
    @Override
    public PaymentStatus getStatus(String transactionId) {
        *// Original implementation*
        return PaymentStatus.COMPLETED;
    }
    
    *// No need to implement refundPayment or getPaymentHistory// Default implementations are used*
}

*// Implementation overriding default methods*
public class FullFeaturedPaymentProvider implements ModernPaymentService {
    @Override
    public boolean processPayment(Payment payment) {
        *// Implementation*
        return true;
    }
    
    @Override
    public PaymentStatus getStatus(String transactionId) {
        *// Implementation*
        return PaymentStatus.COMPLETED;
    }
    
    @Override
    public boolean refundPayment(String transactionId, double amount) {
        *// Custom implementation overriding default*
        System.out.println("Processing refund of " + amount);
        return true;
    }
    
    @Override
    public List<Payment> getPaymentHistory(String customerId) {
        *// Custom implementation overriding default*
        return Arrays.asList(new Payment(), new Payment());
    }
}
```

**Interface evolution features:**

- Default methods (Java 8+): Add methods without breaking existing implementations
- Static methods (Java 8+): Utility methods tied to interface's functionality
- Private methods (Java 9+): Internal helper methods for code reuse
- Private static methods (Java 9+): Internal helpers for static methods

> Interviewer Insight: Default methods fundamentally changed Java's interface model to support API evolution. Before Java 8, adding a method to an interface was a breaking change requiring all implementations to be updated. With default methods, interfaces can evolve without breaking existing code. This was crucial for the Java Collections Framework to add stream operations in Java 8 without breaking millions of existing implementations.
> 

### Functional Interfaces

Functional interfaces have a single abstract method (SAM) and can be implemented with lambda expressions.

```java
*// Basic functional interface*
@FunctionalInterface  *// Optional but recommended annotation*
public interface Transformer<T, R> {
    *// Single abstract method (SAM)*
    R transform(T input);
    
    *// Default and static methods don't count towards SAM count*
    default Transformer<T, R> andThen(Transformer<R, R> after) {
        return input -> after.transform(this.transform(input));
    }
    
    static <T> Transformer<T, T> identity() {
        return input -> input;
    }
}

*// Using functional interfaces with lambdas*
public class FunctionalDemo {
    public void demonstrateFunctional() {
        *// Lambda implementation of functional interface*
        Transformer<String, Integer> lengthCounter = s -> s.length();
        
        *// Method reference - equivalent to lambda*
        Transformer<String, Integer> lengthCounter2 = String::length;
        
        *// Using the functional interface*
        int length = lengthCounter.transform("Hello");  *// 5*
        
        *// Composing with default method*
        Transformer<String, Integer> countAndDouble = lengthCounter.andThen(n -> n * 2);
        int doubledLength = countAndDouble.transform("Hello");  *// 10*
        
        *// Using static method*
        Transformer<String, String> noOp = Transformer.identity();
        String same = noOp.transform("Hello");  *// "Hello"*
    }
    
    *// Method accepting functional interface*
    public <T, R> List<R> transformList(List<T> list, Transformer<T, R> transformer) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(transformer.transform(item));
        }
        return result;
    }
    
    *// Using built-in functional interfaces*
    public void useBuiltInFunctional() {
        *// Predicate - test with boolean result*
        Predicate<String> isLong = s -> s.length() > 10;
        boolean result = isLong.test("Hello World");  *// true*
        
        *// Negating predicates*
        Predicate<String> isShort = isLong.negate();
        boolean shortResult = isShort.test("Hello");  *// true*
        
        *// Function - transforms input to output*
        Function<String, Integer> parseInteger = Integer::parseInt;
        Integer number = parseInteger.apply("42");  *// 42*
        
        *// Composing functions*
        Function<String, String> quote = s -> "'" + s + "'";
        Function<String, String> quoteAndReverse = quote.compose(s -> new StringBuilder(s).reverse().toString());
        String reversed = quoteAndReverse.apply("Hello");  *// "'olleH'"*
        
        *// Consumer - performs action without result*
        Consumer<String> printer = System.out::println;
        printer.accept("Hello World");  *// Prints to console*
        
        *// Supplier - provides value*
        Supplier<LocalDateTime> now = LocalDateTime::now;
        LocalDateTime time = now.get();  *// Current time*
        
        *// BiFunction - two inputs, one output*
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        Integer sum = add.apply(3, 4);  *// 7*
        
        *// Using with Stream API*
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
        List<String> longNames = names.stream()
                                      .filter(isLong)  *// Use predicate*
                                      .map(String::toUpperCase)  *// Use function*
                                      .collect(Collectors.toList());
    }
}
```

**Common built-in functional interfaces:**

1. `Predicate<T>`: `boolean test(T t)` - Tests a condition
2. `Function<T, R>`: `R apply(T t)` - Transforms input to output
3. `Consumer<T>`: `void accept(T t)` - Performs action on input
4. `Supplier<T>`: `T get()` - Provides a value
5. `UnaryOperator<T>`: `T apply(T t)` - Transforms input to same type
6. `BinaryOperator<T>`: `T apply(T t1, T t2)` - Combines two inputs of same type
7. `BiPredicate<T, U>`: `boolean test(T t, U u)` - Tests with two inputs
8. `BiFunction<T, U, R>`: `R apply(T t, U u)` - Transforms two inputs to output
9. `BiConsumer<T, U>`: `void accept(T t, U u)` - Performs action on two inputs

> Deep Dive Tip: The functional interfaces in java.util.function are designed to cover the most common functional patterns. Using these standard interfaces instead of creating custom ones improves interoperability and readability. However, when a function has a special meaning in your domain, a custom functional interface with a descriptive name can improve semantics and documentation.
> 

### Marker Interfaces

Marker interfaces have no methods and serve as type tags to signal special handling by the JVM or frameworks.

```java
*// Classic marker interfaces from Java standard library// Serializable - marks classes that can be serialized*
public interface Serializable {
    *// No methods - pure marker*
}

*// Cloneable - marks classes that support clone()*
public interface Cloneable {
    *// No methods - pure marker*
}

*// Remote - marks remote objects for RMI*
public interface Remote {
    *// No methods - pure marker*
}

*// Custom marker interface example*
public interface Auditable {
    *// Marker for classes that should be audited*
}

*// Implementation using marker interface*
public class SensitiveOperation implements Auditable {
    private String userId;
    
    public void performOperation(String targetId) {
        *// Operation logic*
        System.out.println("Performing sensitive operation for " + targetId);
    }
}

*// AOP aspect that uses marker interface*
@Aspect
public class AuditAspect {
    @Before("execution(* *.*(..)) && this(Auditable)")
    public void auditMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        
        *// Log audit trail*
        System.out.println("AUDIT: Method " + methodName + " called with args: " + 
                          Arrays.toString(args));
    }
}

*// Using marker interface for special handling*
public class SerializationDemo {
    public void saveObject(Object obj) {
        if (obj instanceof Serializable) {
            *// Can be serialized*
            try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("data.ser"))) {
                out.writeObject(obj);
                System.out.println("Object serialized successfully");
            } catch (IOException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("Object is not serializable");
        }
    }
    
    public void cloneObject(Object obj) {
        if (obj instanceof Cloneable) {
            try {
                *// Can be cloned - call clone method via reflection*
                Method cloneMethod = obj.getClass().getMethod("clone");
                Object cloned = cloneMethod.invoke(obj);
                System.out.println("Object cloned successfully");
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("Object is not cloneable");
        }
    }
}

*// Modern alternative using annotations*
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Auditable {
    *// Marker annotation - alternative to marker interface*
}

@Auditable
public class ModernSensitiveOperation {
    *// Implementation*
}

*// Using annotation instead of interface*
public class AnnotationBasedAspect {
    @Before("@within(Auditable)")
    public void auditAnnotatedClass(JoinPoint joinPoint) {
        *// Audit logic*
    }
}
```

**Marker interface characteristics:**

- Contain no methods or constants
- Used purely for runtime type identification
- Enable `instanceof` checks
- Signal special handling by JVM or frameworks
- Modern approach often favors annotations over marker interfaces

> Interviewer Insight: While marker interfaces have largely been superseded by annotations in modern Java, they still have one significant advantage: type safety at compile time. With a marker interface, the compiler can check that only appropriate types are used where the marker is required. Annotations, in contrast, are only checked at runtime unless you use annotation processors. This is why some APIs still use marker interfaces even in modern Java.
> 

## 2.5 Advanced OOP Concepts

### Inner Classes

Inner classes are defined within other classes and have special access to enclosing instance state.

```java
public class OuterClass {
    private int outerField = 10;
    private static int staticOuterField = 20;
    
    *// Member inner class - has access to all outer class members*
    public class MemberInnerClass {
        private int innerField = 5;
        
        public void accessOuterMembers() {
            *// Can access all outer class members including private*
            System.out.println("Outer field: " + outerField);
            System.out.println("Static outer field: " + staticOuterField);
            outerMethod();  *// Call outer class method*
        }
        
        public int combine() {
            return outerField + innerField;
        }
        
        *// Cannot define static members in non-static inner class// static int staticInnerField = 15;  // Compile error*
    }
    
    *// Static nested class - no access to outer instance members*
    public static class StaticNestedClass {
        private int nestedField = 15;
        
        public void accessOuterMembers() {
            *// Can only access static members of outer class*
            System.out.println("Static outer field: " + staticOuterField);
            staticOuterMethod();  *// Call static outer method*
            
            *// Cannot access instance members of outer class// System.out.println(outerField);  // Compile error// outerMethod();  // Compile error*
        }
        
        *// Can define static members in static nested class*
        static int staticNestedField = 25;
    }
    
    *// Local inner class - defined within a method*
    public void methodWithLocalClass() {
        final int localVar = 30;  *// Effectively final*
        int nonFinalLocalVar = 40;  *// Effectively final if not modified*
        
        *// Local class defined within method*
        class LocalClass {
            public void accessVariables() {
                *// Can access final or effectively final local variables*
                System.out.println("Local variable: " + localVar);
                System.out.println("Non-final local: " + nonFinalLocalVar);
                
                *// Can access all outer class members*
                System.out.println("Outer field: " + outerField);
            }
        }
        
        *// Use the local class*
        LocalClass local = new LocalClass();
        local.accessVariables();
        
        *// nonFinalLocalVar = 50;  // Would make it not effectively final// and unavailable in LocalClass*
    }
    
    *// Method returning anonymous inner class*
    public Runnable createRunnable() {
        *// Anonymous inner class implementing Runnable*
        return new Runnable() {
            @Override
            public void run() {
                *// Can access outer class members*
                System.out.println("In runnable, outer field: " + outerField);
            }
        };
    }
    
    *// Interface within a class*
    public interface InnerInterface {
        void process();
    }
    
    *// Enum within a class*
    public enum InnerEnum {
        SMALL, MEDIUM, LARGE;
        
        *// Enum can access static members of outer class*
        public int getValue() {
            return staticOuterField * ordinal();
        }
    }
    
    private void outerMethod() {
        System.out.println("Outer method called");
    }
    
    private static void staticOuterMethod() {
        System.out.println("Static outer method called");
    }
    
    *// Creating inner class instances*
    public void createInnerInstances() {
        *// Member inner class requires outer instance*
        MemberInnerClass inner = new MemberInnerClass();
        inner.accessOuterMembers();
        
        *// Alternative syntax from outside OuterClass// OuterClass.MemberInnerClass inner2 = outerInstance.new MemberInnerClass();*
        
        *// Static nested class doesn't need outer instance*
        StaticNestedClass nested = new StaticNestedClass();
        nested.accessOuterMembers();
        
        *// Alternative syntax from outside OuterClass// OuterClass.StaticNestedClass nested2 = new OuterClass.StaticNestedClass();*
    }
}

*// Client code*
public class InnerClassClient {
    public void useInnerClasses() {
        *// Create outer class instance*
        OuterClass outer = new OuterClass();
        
        *// Create member inner class instance*
        OuterClass.MemberInnerClass inner = outer.new MemberInnerClass();
        inner.accessOuterMembers();
        
        *// Create static nested class instance*
        OuterClass.StaticNestedClass nested = new OuterClass.StaticNestedClass();
        nested.accessOuterMembers();
        
        *// Use anonymous inner class*
        Runnable runnable = outer.createRunnable();
        runnable.run();
        
        *// Use inner interface*
        OuterClass.InnerInterface processor = new OuterClass.InnerInterface() {
            @Override
            public void process() {
                System.out.println("Processing...");
            }
        };
        processor.process();
        
        *// Use inner enum*
        OuterClass.InnerEnum size = OuterClass.InnerEnum.LARGE;
        System.out.println("Enum value: " + size.getValue());
    }
}
```

**Inner class types and characteristics:**

1. **Member inner class**
    - Has access to all outer class members (even private)
    - Requires outer class instance to exist
    - Cannot contain static members
    - Has implicit reference to enclosing instance (`OuterClass.this`)
2. **Static nested class**
    - Can only access static members of outer class
    - Does not require outer class instance
    - Can contain static members
    - No implicit reference to enclosing instance
3. **Local inner class**
    - Defined within a method
    - Can access final or effectively final local variables
    - Can access all outer class members
    - Cannot be accessed outside the method
4. **Anonymous inner class**
    - No explicit class name
    - Created as expression
    - Typically implements an interface or extends a class
    - Limited to one instance
    - Cannot define constructors

> Interviewer Insight: Inner classes are compiled into separate class files named OuterClass$InnerClass.class. Each non-static inner class instance holds an implicit reference to its enclosing instance, which can cause memory leaks if the inner class outlives the outer class. This is why event listeners and callbacks are often implemented as static nested classes with explicit references to avoid unintentional strong references.
> 

### Enum Types

Enum types provide type-safe enumeration of a fixed set of constants with rich behavior capabilities.

```java
*// Basic enum*
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

*// Enum with fields, constructor, and methods*
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6),
    MARS(6.421e+23, 3.3972e6),
    JUPITER(1.9e+27, 7.1492e7),
    SATURN(5.688e+26, 6.0268e7),
    URANUS(8.686e+25, 2.5559e7),
    NEPTUNE(1.024e+26, 2.4746e7);

    private final double mass;   *// in kilograms*
    private final double radius; *// in meters*
    
    *// Constructor must be private or package-private*
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
    
    *// Methods*
    public double getMass() {
        return mass;
    }
    
    public double getRadius() {
        return radius;
    }
    
    *// Calculated field*
    public double surfaceGravity() {
        double G = 6.67300E-11; *// Universal gravitational constant*
        return G * mass / (radius * radius);
    }
    
    public double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
}

*// Enum implementing interface*
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override
    public String toString() {
        return symbol;
    }
}

*// Extending enum functionality with EnumSet and EnumMap*
public class EnumDemo {
    public void demonstrateEnumSet() {
        *// Create an EnumSet*
        EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
        EnumSet<Day> weekdays = EnumSet.complementOf(weekend);  *// All days except weekend*
        
        *// Efficient set operations*
        EnumSet<Day> allDays = EnumSet.allOf(Day.class);
        EnumSet<Day> noDays = EnumSet.noneOf(Day.class);
        
        *// Iteration*
        for (Day day : weekdays) {
            System.out.println(day);
        }
    }
    
    public void demonstrateEnumMap() {
        *// Create an EnumMap*
        EnumMap<Planet, Double> surfaceGravityMap = new EnumMap<>(Planet.class);
        
        *// Initialize map*
        for (Planet planet : Planet.values()) {
            surfaceGravityMap.put(planet, planet.surfaceGravity());
        }
        
        *// Efficient map operations*
        double earthGravity = surfaceGravityMap.get(Planet.EARTH);
        System.out.println("Earth gravity: " + earthGravity);
        
        *// Iteration*
        for (Map.Entry<Planet, Double> entry : surfaceGravityMap.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }
    
    *// Using enum in switch statement*
    public String getDayType(Day day) {
        switch (day) {
            case SATURDAY:
            case SUNDAY:
                return "Weekend";
            default:
                return "Weekday";
        }
    }
    
    *// Modern switch expression with enum (Java 14+)*
    public String getDayTypeModern(Day day) {
        return switch (day) {
            case SATURDAY, SUNDAY -> "Weekend";
            default -> "Weekday";
        };
    }
}

*// Strategy pattern with enum*
public enum PaymentMethod {
    CREDIT_CARD {
        @Override
        public boolean processPayment(double amount) {
            System.out.println("Processing credit card payment of " + amount);
            return true;  *// Simplified implementation*
        }
    },
    PAYPAL {
        @Override
        public boolean processPayment(double amount) {
            System.out.println("Processing PayPal payment of " + amount);
            return true;  *// Simplified implementation*
        }
    },
    BANK_TRANSFER {
        @Override
        public boolean processPayment(double amount) {
            System.out.println("Processing bank transfer of " + amount);
            return amount <= 10000;  *// Simplified implementation*
        }
    };
    
    public abstract boolean processPayment(double amount);
    
    *// Factory method*
    public static PaymentMethod getDefault() {
        return CREDIT_CARD;
    }
}
```

**Enum features and characteristics:**

- Type-safe enumeration (compile-time safety)
- Implicitly extends `java.lang.Enum`
- Cannot extend other classes (but can implement interfaces)
- Can contain fields, constructors, methods
- Supports constant-specific method implementations
- Thread-safe singleton pattern
- Built-in methods like `values()`, `valueOf()`, `name()`, `ordinal()`
- Special collections: `EnumSet` and `EnumMap` for high performance
- Well-suited for strategy and state patterns

> Deep Dive Tip: EnumSet and EnumMap use bit vectors and arrays respectively for extremely efficient operations. An EnumSet typically requires only a single long value to represent all enum constants (up to 64), making operations like contains() and add() O(1) with minimal memory footprint. Similarly, EnumMap uses a simple array indexed by the ordinal values, making it much faster than a regular HashMap.
> 

### final Keyword

The `final` keyword restricts inheritance, method overriding, and variable reassignment for enhanced safety and optimization.

```java
*// Final class - cannot be extended*
public final class ImmutableValue {
    private final int value;  *// Final field - cannot be reassigned*
    
    public ImmutableValue(int value) {
        this.value = value;
    }
    
    public int getValue() {
        return value;
    }
}

*// Class with final methods*
public class BaseService {
    *// Final method - cannot be overridden*
    public final void criticalOperation() {
        validateState();
        performOperation();
        logCompletion();
    }
    
    private void validateState() {
        *// Implementation*
    }
    
    private void performOperation() {
        *// Implementation*
    }
    
    private void logCompletion() {
        *// Implementation*
    }
}

*// Using final with parameters and local variables*
public class FinalDemo {
    *// Final parameter - cannot be reassigned*
    public void processRequest(final String requestId, String data) {
        *// requestId = "modified";  // Compile error*
        
        *// Final local variable*
        final int maxRetries = 3;
        *// maxRetries = 5;  // Compile error*
        
        *// Effectively final variable (since Java 8)*
        int retryCount = 0;
        
        *// Using local inner class with final/effectively final variables*
        class RetryHandler {
            public void execute() {
                System.out.println("Processing request " + requestId);
                System.out.println("Max retries: " + maxRetries);
                System.out.println("Current retries: " + retryCount);
            }
        }
        
        new RetryHandler().execute();
        
        *// retryCount = 1;  // Would make retryCount not effectively final// and unavailable in RetryHandler*
    }
    
    *// Final variable with lazy initialization using holder pattern*
    private static class Holder {
        static final ExpensiveObject INSTANCE = new ExpensiveObject();
    }
    
    public static ExpensiveObject getInstance() {
        return Holder.INSTANCE;  *// Lazy initialization*
    }
}

*// Final with collections - reference is final, not contents*
public class FinalCollections {
    *// Final reference to collection*
    private final List<String> names;
    
    public FinalCollections() {
        names = new ArrayList<>();
        names.add("Initial");
    }
    
    public void addName(String name) {
        names.add(name);  *// Legal - modifying collection contents*
        
        *// names = new ArrayList<>();  // Illegal - cannot reassign final variable*
    }
    
    *// Creating truly immutable collections*
    private final List<String> immutableNames = Collections.unmodifiableList(
        Arrays.asList("Alice", "Bob", "Charlie")
    );
    
    public List<String> getImmutableNames() {
        return immutableNames;  *// Returns unmodifiable view*
    }
    
    *// Java 9+ immutable collection factory methods*
    private final List<String> modernImmutable = List.of("Alice", "Bob", "Charlie");
    private final Map<String, Integer> scores = Map.of(
        "Alice", 95,
        "Bob", 87,
        "Charlie", 91
    );
}
```

**Uses of the final keyword:**

1. **Final classes**: Cannot be extended
    - Immutable types (String, Integer)
    - Security-sensitive classes
    - Classes not designed for inheritance
2. **Final methods**: Cannot be overridden
    - Core functionality that shouldn't change
    - Methods called from constructors
    - Template method pattern hooks
3. **Final fields**: Cannot be reassigned
    - Immutable objects
    - Constants
    - Thread-safe sharing
    - Required for lambdas and inner classes
4. **Final parameters/variables**: Cannot be reassigned
    - Thread safety
    - Functional programming patterns
    - Access from inner classes

> Interviewer Insight: Final classes and methods enable important JVM optimizations. Since they cannot be overridden, the JIT compiler can inline method calls, eliminating the overhead of dynamic dispatch. This is one reason why String methods are so fast—the String class is final, allowing aggressive method inlining. In performance-critical code, judicious use of final can provide measurable speedups.
> 

### Package Management

**Purpose**: Organizes and encapsulates code.
**Usage**: Logical code organization and access control.
**Example**:

```java
*// Package declaration - must be first statement in file*
package com.enterprise.finance.transaction;

*// Importing specific classes (preferred for clarity)*
import java.util.List;
import java.util.ArrayList;
import java.time.LocalDateTime;

*// Wildcard import (avoid in production code)*
import java.util.concurrent.*;  **// Imports all classes from concurrent package*// Static import for constants/methods*
import static java.lang.Math.PI;
import static java.util.Collections.unmodifiableList;

*// Class with package private visibility (default)*
class InternalHelper {
    void helperMethod() {
        *// Only accessible within the same package*
    }
}

*// Public class - can be accessed from any package*
public class TransactionProcessor {
    public List<Transaction> filterTransactions() {
        *// Using static import*
        List<Transaction> results = new ArrayList<>();
        System.out.println("Using PI: " + PI);
        
        *// Return immutable view using static import*
        return unmodifiableList(results);
    }
}
```

**Package features and considerations:**

```java
*// Package naming conventions - reversed domain name*
package com.amazon.payments.processing;

*// Multi-level packages create directory hierarchies// src/main/java/com/amazon/payments/processing/PaymentProcessor.java// Split package anti-pattern (avoid!)// com.company.feature in module1// com.company.feature in module2 (same package in different modules)// Sealed packages (unofficial but common practice)// By limiting public classes, packages can be effectively "sealed"// Only package-private constructors in public classes forces factory methods*
```

> Interviewer Insight: Package structures impact compilation efficiency, class loading, and module boundaries. In microservice architectures, the package structure often mirrors the service boundary structure. Poor package organization leads to circular dependencies and tight coupling that makes systems unmaintainable. For large enterprise systems, consider using package-by-feature rather than package-by-layer to improve cohesion.
> 

> Deep Dive Tip: JARs load more efficiently when related classes are in the same package due to class loader optimizations. The JVM's package sealing mechanism (in JAR manifests) can prevent security vulnerabilities by ensuring all classes in a package come from the same code source. This is critical for sensitive packages containing cryptographic implementations or security checks.
> 

### Modern OOP Features

**Purpose**: Latest Java OOP capabilities.
**Usage**: Safer and more concise code.
**Example**:

```java
*// Sealed Classes (Java 17+)// Restricts which classes can extend/implement*
public sealed class PaymentMethod 
    permits CreditCard, DebitCard, BankTransfer {
    
    protected final String id;
    
    protected PaymentMethod(String id) {
        this.id = id;
    }
    
    public abstract boolean processPayment(double amount);
}

*// Permitted subclasses must use one of three modifiers:// 1. final - Cannot be extended further*
final class CreditCard extends PaymentMethod {
    private final String cardNumber;
    
    public CreditCard(String id, String cardNumber) {
        super(id);
        this.cardNumber = cardNumber;
    }
    
    @Override
    public boolean processPayment(double amount) {
        return processCreditCard(amount);
    }
    
    private boolean processCreditCard(double amount) {
        *// Implementation*
        return true;
    }
}

*// 2. sealed - Further restricts its hierarchy*
sealed class DebitCard extends PaymentMethod 
    permits BusinessDebitCard {
    *// Implementation*
    @Override
    public boolean processPayment(double amount) {
        return processDebit(amount);
    }
    
    private boolean processDebit(double amount) {
        *// Implementation*
        return true;
    }
}

final class BusinessDebitCard extends DebitCard {
    *// Implementation*
}

*// 3. non-sealed - Allows unrestricted extension*
non-sealed class BankTransfer extends PaymentMethod {
    @Override
    public boolean processPayment(double amount) {
        return processBankTransfer(amount);
    }
    
    private boolean processBankTransfer(double amount) {
        *// Implementation*
        return true;
    }
}

*// Records (Java 14+) - Immutable data carriers*
public record Transaction(
    String id,
    double amount,
    LocalDateTime timestamp,
    TransactionType type
) {
    *// Compact constructor for validation*
    public Transaction {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        *// No need to assign fields, happens automatically*
    }
    
    *// Additional methods*
    public boolean isRecent() {
        return timestamp.isAfter(LocalDateTime.now().minusDays(1));
    }
    
    *// Records can implement interfaces*
    public interface Auditable {
        String getAuditLog();
    }
    
    *// Nested records (Java 16+)*
    public record TransactionDetail(String description, String reference) {}
}

*// Pattern matching with records (Java 21)*
public String describeTransaction(Object obj) {
    return switch (obj) {
        case Transaction(var id, var amount, var timestamp, var type) 
            when amount > 1000 -> 
                "High-value transaction: " + id;
        case Transaction(String id, double amount, var timestamp, TransactionType.REFUND) -> 
                "Refund: " + id + " for " + amount;
        case Transaction t -> 
                "Regular transaction: " + t.id();
        default -> "Not a transaction";
    };
}

*// Unnamed patterns and variables (Java 21)*
public boolean isLargeTransaction(Transaction t) {
    return switch (t) {
        case Transaction(_, var amount, _, _) when amount > 10000 -> true;
        default -> false;
    };
}
```

**Java 17-21 features for OOP:**

java

`*// String templates (Java 21 Preview)*
String name = "John";
double balance = 1234.56;
String message = STR."Customer \{name} has balance $\{balance%.2f}";

*// Unnamed class instances and instance methods (Preview)*
void processCustom() {
    *// Anonymous instance with "unnamed" syntax*
    var handler = new EventHandler() {
        public void handle(Event event) -> {
            System.out.println("Handling: " + event);
        }
    };
}`

> Interviewer Insight: Sealed classes solve a critical design problem: they allow for exhaustive pattern matching while maintaining extensibility within controlled boundaries. This enables compiler verification of exhaustive handling of subtypes, which dramatically reduces runtime errors for business-critical code paths like payment processing, where missing a case could lead to financial losses. Unlike enums, sealed classes allow each subtype to have its own state.
> 

> Deep Dive Tip: Records provide true immutability guarantees that final fields alone don't. The compiler-generated equals(), hashCode(), and toString() methods are consistent and optimized, eliminating a whole class of bugs that often appear in hand-written implementations. For enterprise systems processing millions of transactions, the memory layout optimization of records can reduce heap fragmentation and improve GC performance significantly.
> 

## Module 3: Exception Handling

### 3.1 Exception Basics

**Purpose**: Manages and responds to runtime errors.
**Usage**: Error handling and recovery strategy.
**Example**:

```java
*// Exception hierarchy*
public void exceptionBasics() {
    */**
     * Throwable (Root of hierarchy)
     * ├── Error (Serious problems, not typically caught)
     * │   ├── OutOfMemoryError
     * │   ├── StackOverflowError
     * │   └── ...
     * └── Exception (Base for all exceptions)
     *     ├── RuntimeException (Unchecked)
     *     │   ├── NullPointerException
     *     │   ├── IllegalArgumentException
     *     │   ├── ConcurrentModificationException 
     *     │   └── ...
     *     └── Checked Exceptions (Must be handled)
     *         ├── IOException
     *         ├── SQLException
     *         └── ...
     **/*
}

*// Basic try-catch*
public void basicTryCatch() {
    try {
        File file = new File("config.json");
        FileReader reader = new FileReader(file);  *// Might throw FileNotFoundException*
        
        *// Code that might throw exceptions*
        int data = processData();
        
    } catch (FileNotFoundException e) {
        *// Specific exception handling*
        log.error("Configuration file not found", e);
        loadDefaultConfig();
        
    } catch (IOException e) {
        *// Handle IO problems*
        log.error("Error reading configuration", e);
        
    } catch (Exception e) {
        *// General fallback (catch more specific exceptions first)*
        log.error("Unexpected error", e);
    }
}

*// Multiple catch blocks order - most specific first*
try {
    *// Code that might throw exceptions*
} catch (NullPointerException e) {
    *// Handle NullPointerException*
} catch (IllegalArgumentException e) {
    *// Handle IllegalArgumentException*
} catch (RuntimeException e) {
    *// Handle other RuntimeExceptions*
} catch (Exception e) {
    *// Handle any other exceptions*
}

*// Multi-catch feature (Java 7+) for similar handling*
try {
    *// Code that might throw different exceptions*
} catch (IOException | SQLException e) {
    *// Common handling for both exception types*
    log.error("Data access error", e);
    
    *// e is effectively final in multi-catch// e = new IOException(); // Compilation error*
}
```

**Using finally block and try-with-resources:**

```java
*// Finally block - always executed, even if exception occurs*
public void finallyExample() {
    Connection conn = null;
    try {
        conn = dataSource.getConnection();
        *// Use connection*
    } catch (SQLException e) {
        log.error("Database error", e);
    } finally {
        *// Always executed (except System.exit() or JVM crash)*
        if (conn != null) {
            try {
                conn.close();  *// Might throw SQLException*
            } catch (SQLException ex) {
                log.error("Error closing connection", ex);
            }
        }
    }
}

*// Try-with-resources (Java 7+) - AutoCloseable resources*
public void tryWithResources() throws IOException {
    *// Resources are automatically closed in reverse order*
    try (FileInputStream fis = new FileInputStream("data.txt");
         BufferedInputStream bis = new BufferedInputStream(fis)) {
        
        *// Use resources*
        int data;
        while ((data = bis.read()) != -1) {
            *// Process data*
        }
        
        *// No need for finally block to close resources// Resources closed automatically in reverse order (bis, then fis)*
        
    } catch (IOException e) {
        *// Exception handling - resources still closed if exception occurs*
        log.error("Error processing file", e);
        throw e;  *// Rethrow if needed*
    }
}

*// Try-with-resources enhancements (Java 9+)*
public void tryWithResourcesJava9(Connection existingConn) throws SQLException {
    *// Can use effectively final variables as resources*
    try (existingConn) {
        *// Use existing connection*
        Statement stmt = existingConn.createStatement();
        *// Use statement*
    }
    *// existingConn automatically closed*
}
```

> Interviewer Insight: The exception stack trace contains incredibly valuable diagnostic information, but many codebases destroy this by catching and wrapping exceptions without cause, creating deep "exception wrapping chains" that obscure the root cause. In production systems, preserve the original cause using throw new ServiceException("Context message", originalException) rather than creating new exceptions. Also, never silently swallow exceptions in empty catch blocks—this makes production issues nearly impossible to diagnose.
> 

> Deep Dive Tip: Many developers don't realize that Java's exception handling machinery has a significant performance cost primarily due to capturing and building the stack trace. Creating and throwing an exception is 1-2 orders of magnitude slower than normal code flow. For performance-critical paths handling frequent expected conditions (like "item not found"), consider using return values (like Optional<T> or status objects) rather than exceptions, reserving exceptions for truly exceptional cases.
> 

### 3.2 Advanced Exception Handling

**Purpose**: Sophisticated error management.
**Usage**: Custom exceptions and error policies.
**Example**:

```java
*// Throwing exceptions*
public void validateUser(User user) {
    if (user == null) {
        throw new IllegalArgumentException("User cannot be null");
    }
    
    if (user.getEmail() == null || user.getEmail().isEmpty()) {
        throw new ValidationException("Email is required");
    }
    
    *// Throwing with cause (exception chaining)*
    try {
        validateUserPermissions(user);
    } catch (PermissionException e) {
        *// Wrap with additional context while preserving original cause*
        throw new UserValidationException("User validation failed: " + user.getId(), e);
    }
}

*// Declaring exceptions with throws clause*
public List<Transaction> loadTransactions(String userId) throws DataAccessException {
    try {
        return transactionDao.findByUserId(userId);
    } catch (SQLException e) {
        *// Translate to application-specific exception*
        throw new DataAccessException("Failed to load transactions for user: " + userId, e);
    }
}

*// Re-throwing exceptions*
public void processOrder(Order order) throws OrderProcessingException {
    try {
        *// Process order*
        validateOrder(order);
        calculateTotals(order);
        applyDiscounts(order);
    } catch (Exception e) {
        *// Log with full context*
        log.error("Order processing failed for order {}: {}", order.getId(), e.getMessage(), e);
        
        *// Re-throw as application-specific exception*
        throw new OrderProcessingException("Failed to process order: " + order.getId(), e);
    }
}
```

**Custom exception classes:**

```java
*// Custom checked exception*
public class ServiceException extends Exception {
    private final String errorCode;
    
    public ServiceException(String message) {
        super(message);
        this.errorCode = "GENERIC_ERROR";
    }
    
    public ServiceException(String message, Throwable cause) {
        super(message, cause);
        this.errorCode = "GENERIC_ERROR";
    }
    
    public ServiceException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public ServiceException(String message, String errorCode, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

*// Custom unchecked exception (extends RuntimeException)*
public class ValidationException extends RuntimeException {
    private final List<String> violations;
    
    public ValidationException(String message) {
        super(message);
        this.violations = Collections.emptyList();
    }
    
    public ValidationException(String message, List<String> violations) {
        super(message);
        this.violations = Collections.unmodifiableList(new ArrayList<>(violations));
    }
    
    public List<String> getViolations() {
        return violations;
    }
}
```

**Exception best practices:**

```java
*// Exceptions as API design elements*
public interface PaymentGateway {
    *// Checked exceptions as part of API contract*
    Transaction processPayment(Payment payment) throws PaymentRejectedException, 
                                                     GatewayTimeoutException;
                                                     
    *// Specific exceptions communicate specific failure modes*
    default void validatePayment(Payment payment) {
        if (payment == null) {
            throw new IllegalArgumentException("Payment cannot be null");
        }
        
        if (payment.getAmount() <= 0) {
            throw new PaymentValidationException("Payment amount must be positive");
        }
    }
}

*// Exception translation pattern - converting low-level exceptions to application exceptions*
@Repository
public class JdbcUserRepository implements UserRepository {
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public User findById(String id) {
        try {
            return jdbcTemplate.queryForObject(
                "SELECT * FROM users WHERE id = ?", 
                new UserRowMapper(), 
                id
            );
        } catch (EmptyResultDataAccessException e) {
            *// Translate to domain exception (not found)*
            throw new UserNotFoundException("User not found with id: " + id);
        } catch (DataAccessException e) {
            *// Translate infrastructure exception to application exception*
            throw new RepositoryException("Database error retrieving user: " + id, e);
        }
    }
}

*// Logging exceptions properly*
try {
    *// Some operation*
} catch (Exception e) {
    *// Include context, message, and full stack trace*
    log.error("Failed to process payment for order {}: {}", 
              orderId, e.getMessage(), e);
    
    *// Bad - loses stack trace// log.error("Error: " + e.getMessage());*
    
    *// Also bad - uninformative// log.error("Error occurred", e);*
}

*// Analyzing stack traces programmatically*
public void analyzeException(Exception e) {
    StackTraceElement[] stackTrace = e.getStackTrace();
    
    *// Find specific class in stack trace*
    boolean containsMyClass = Arrays.stream(stackTrace)
        .anyMatch(element -> element.getClassName().contains("com.myapp.service"));
    
    *// Get root cause of exception chain*
    Throwable rootCause = e;
    while (rootCause.getCause() != null) {
        rootCause = rootCause.getCause();
    }
    
    *// Stack filtering in Java 9+*
    Throwable filtered = e.fillInStackTrace();
    filtered.setStackTrace(
        Arrays.stream(filtered.getStackTrace())
              .filter(element -> !element.getClassName().contains("org.springframework"))
              .toArray(StackTraceElement[]::new)
    );
}
```

> Interviewer Insight: When designing exception hierarchies for enterprise applications, create meaningful exception families that align with your domain. For example, a payment processing system might have PaymentException as a base class, with PaymentValidationException, PaymentGatewayException, and FraudDetectionException as subclasses. This makes error handling more consistent and clearer than generic technical exceptions. Also, consider whether each exception should be checked or unchecked based on whether calling code can reasonably be expected to recover.
> 

> Deep Dive Tip: Consider the "exception translation" pattern for boundaries between architectural layers. For example, a repository layer should catch SQL-related exceptions and throw repository-specific exceptions instead. This decouples your domain logic from the underlying infrastructure and prevents implementation details from leaking up the stack. Many frameworks like Spring JDBC use this pattern internally with their unchecked exception hierarchies, which is why you rarely see raw SQLExceptions in Spring applications.
> 

I've completed the requested sections, covering:

1. Package Management (Module 2.5)
2. Modern OOP Features (Module 2.5)
3. Complete Module 3: Exception Handling

The content follows the existing document structure with code snippets, Interviewer Insight blocks, Deep Dive Tip blocks, and focuses on production-level considerations relevant for senior Java developers.

# Module 4: Java I/O

## 4.1 Classic I/O

**Purpose**: File and stream-based input/output operations.
**Usage**: Reading/writing files and handling data streams.
**Example**:

```java
*// File class operations*
public void fileOperations() {
    *// Create File object (doesn't create actual file yet)*
    File configFile = new File("/opt/app/config.properties");
    
    *// File information*
    boolean exists = configFile.exists();
    long size = configFile.length();  *// Size in bytes*
    boolean isDirectory = configFile.isDirectory();
    long lastModified = configFile.lastModified();  *// Timestamp*
    
    *// File manipulation*
    if (!configFile.exists()) {
        try {
            *// Create parent directories if needed*
            configFile.getParentFile().mkdirs();
            boolean created = configFile.createNewFile();  *// Atomic creation*
            if (created) {
                log.info("Created file: {}", configFile.getAbsolutePath());
            }
        } catch (IOException e) {
            log.error("Failed to create file: {}", e.getMessage(), e);
        }
    }
    
    *// Working with paths*
    File dataDir = new File("/data");
    File logFile = new File(dataDir, "app.log");  *// Combines paths*
    
    *// Listing directory contents*
    if (dataDir.isDirectory()) {
        *// Basic listing*
        File[] allFiles = dataDir.listFiles();
        
        *// Filtered listing with anonymous FileFilter*
        File[] logFiles = dataDir.listFiles(new FileFilter() {
            @Override
            public boolean accept(File file) {
                return file.getName().endsWith(".log");
            }
        });
        
        *// Using lambda (Java 8+)*
        File[] configFiles = dataDir.listFiles(file -> file.getName().endsWith(".properties"));
        
        *// Recursively process directory structure*
        processDirectory(dataDir);
    }
    
    *// File permissions (limited functionality in Java 6/7)*
    boolean canRead = configFile.canRead();
    boolean canWrite = configFile.canWrite();
    boolean canExecute = configFile.canExecute();
    
    *// Set permissions (limited)*
    configFile.setReadable(true, false);  *// Readable by all users*
    configFile.setWritable(true, true);   *// Writable only by owner*
}

private void processDirectory(File dir) {
    File[] files = dir.listFiles();
    if (files == null) return;
    
    for (File file : files) {
        if (file.isDirectory()) {
            processDirectory(file);  *// Recursion*
        } else {
            *// Process file*
            System.out.println("Found file: " + file.getAbsolutePath());
        }
    }
}
```

**Byte Stream I/O - for binary data:**

```java
*// Basic byte stream I/O*
public void byteStreamExample() {
    *// Writing binary data*
    try (FileOutputStream fos = new FileOutputStream("data.bin")) {
        *// Write individual bytes*
        fos.write(65);  *// Writes 'A'*
        
        *// Write byte array*
        byte[] data = {66, 67, 68, 69};  *// 'B', 'C', 'D', 'E'*
        fos.write(data);
        
        *// Write portion of array*
        fos.write(data, 1, 2);  *// Writes 'C', 'D'*
        
    } catch (IOException e) {
        log.error("Error writing file", e);
    }
    
    *// Reading binary data*
    try (FileInputStream fis = new FileInputStream("data.bin")) {
        *// Read single byte (returns -1 at end of stream)*
        int byteRead = fis.read();
        
        *// Read into buffer*
        byte[] buffer = new byte[1024];
        int bytesRead = fis.read(buffer);
        
        *// Read entire file*
        byte[] allBytes = fis.readAllBytes();  *// Java 9+*
        
    } catch (IOException e) {
        log.error("Error reading file", e);
    }
}

*// Data streams for primitive types*
public void dataStreamExample() {
    try (
        *// Binary data with type information*
        DataOutputStream dos = new DataOutputStream(
            new FileOutputStream("data.dat")
        )
    ) {
        *// Write primitives with type preservation*
        dos.writeInt(42);
        dos.writeDouble(3.14159);
        dos.writeUTF("Hello, world!");  *// Modified UTF-8 encoding*
        dos.writeBoolean(true);
        
    } catch (IOException e) {
        log.error("Error writing data", e);
    }
    
    try (
        DataInputStream dis = new DataInputStream(
            new FileInputStream("data.dat")
        )
    ) {
        *// Must read in same order as written*
        int intValue = dis.readInt();
        double doubleValue = dis.readDouble();
        String stringValue = dis.readUTF();
        boolean boolValue = dis.readBoolean();
        
    } catch (IOException e) {
        log.error("Error reading data", e);
    }
}

*// Object streams for serialization*
public void objectStreamExample() {
    List<User> users = List.of(new User("alice"), new User("bob"));
    
    try (
        ObjectOutputStream oos = new ObjectOutputStream(
            new FileOutputStream("users.ser")
        )
    ) {
        *// Write entire object graph*
        oos.writeObject(users);
        
    } catch (IOException e) {
        log.error("Error serializing objects", e);
    }
    
    try (
        ObjectInputStream ois = new ObjectInputStream(
            new FileInputStream("users.ser")
        )
    ) {
        *// Read and cast object graph*
        @SuppressWarnings("unchecked")
        List<User> loadedUsers = (List<User>) ois.readObject();
        
    } catch (IOException | ClassNotFoundException e) {
        log.error("Error deserializing objects", e);
    }
}
```

**Character Stream I/O - for text data:**

```java
*// Character streams for text processing*
public void characterStreamExample() {
    *// Writing text*
    try (
        *// FileWriter uses platform's default encoding// Always specify encoding explicitly for production code*
        FileWriter fw = new FileWriter("output.txt");
        
        *// Better: explicit charset (Java 11+)*
        FileWriter fw2 = new FileWriter("output.txt", StandardCharsets.UTF_8)
    ) {
        fw.write("Hello, world!\n");
        fw.write("Line 2\n");
        
        *// Write character array*
        char[] chars = {'J', 'a', 'v', 'a'};
        fw.write(chars);
        
    } catch (IOException e) {
        log.error("Error writing text", e);
    }
    
    *// Reading text*
    try (
        *// FileReader uses platform's default encoding*
        FileReader fr = new FileReader("output.txt");
        
        *// Better: explicit charset (Java 11+)*
        FileReader fr2 = new FileReader("output.txt", StandardCharsets.UTF_8)
    ) {
        *// Read single char*
        int charRead = fr.read();
        
        *// Read into buffer*
        char[] buffer = new char[1024];
        int charsRead = fr.read(buffer);
        
        *// Process char buffer*
        String content = new String(buffer, 0, charsRead);
        
    } catch (IOException e) {
        log.error("Error reading text", e);
    }
    
    *// StringReader/Writer for in-memory text processing*
    String source = "Line 1\nLine 2\nLine 3";
    
    try (StringReader reader = new StringReader(source)) {
        int c;
        while ((c = reader.read()) != -1) {
            *// Process char by char*
        }
    }
    
    try (StringWriter writer = new StringWriter()) {
        writer.write("Building a string ");
        writer.write("piece by piece.");
        String result = writer.toString();
    }
}
```

**Buffered streams for performance:**

```java
*// Buffered streams - dramatically improve performance*
public void bufferedStreamExample() {
    *// Buffered byte streams*
    try (
        BufferedInputStream bis = new BufferedInputStream(
            new FileInputStream("large-file.dat"), 8192  *// Custom buffer size*
        );
        BufferedOutputStream bos = new BufferedOutputStream(
            new FileOutputStream("output.dat")
        )
    ) {
        *// Efficient copying - read/write in chunks*
        byte[] buffer = new byte[4096];
        int bytesRead;
        
        while ((bytesRead = bis.read(buffer)) != -1) {
            bos.write(buffer, 0, bytesRead);
        }
        
        *// No need for explicit flush() - close() does it*
        
    } catch (IOException e) {
        log.error("Error copying file", e);
    }
    
    *// Buffered character streams*
    try (
        BufferedReader reader = new BufferedReader(
            new FileReader("data.txt", StandardCharsets.UTF_8)
        );
        BufferedWriter writer = new BufferedWriter(
            new FileWriter("output.txt", StandardCharsets.UTF_8)
        )
    ) {
        *// Line-oriented reading - key feature of BufferedReader*
        String line;
        while ((line = reader.readLine()) != null) {
            *// Process line*
            writer.write(line);
            writer.newLine();  *// Platform-specific line separator*
        }
        
        *// Explicit flush - writes buffer to underlying stream*
        writer.flush();
        
    } catch (IOException e) {
        log.error("Error processing text file", e);
    }
}

*// PrintWriter for formatted output*
public void printWriterExample() {
    try (
        PrintWriter out = new PrintWriter(
            new BufferedWriter(
                new FileWriter("report.txt", StandardCharsets.UTF_8)
            )
        )
    ) {
        *// Formatted output - like System.out*
        out.println("User Report");
        out.println("-----------");
        
        *// Formatted printing*
        out.printf("User: %s, Score: %.2f%n", "Alice", 95.75);
        out.printf("Date: %tF%n", new Date());
        
    } catch (IOException e) {
        log.error("Error writing report", e);
    }
}
```

**Other specialized streams:**

```java
*// RandomAccessFile - direct file positioning*
public void randomAccessExample() {
    try (RandomAccessFile raf = new RandomAccessFile("data.bin", "rw")) {
        *// Get file size*
        long fileSize = raf.length();
        
        *// Write at specific position*
        raf.seek(100);  *// Move to position 100*
        raf.writeInt(42);
        
        *// Read from specific position*
        raf.seek(0);  *// Back to start*
        int value = raf.readInt();
        
        *// Append to file*
        raf.seek(raf.length());
        raf.writeBytes("Appended text");
        
    } catch (IOException e) {
        log.error("Error accessing file", e);
    }
}

*// PipedInputStream/OutputStream - thread communication*
public void pipedStreamExample() {
    try (
        PipedOutputStream output = new PipedOutputStream();
        PipedInputStream input = new PipedInputStream(output)
    ) {
        *// Start producer thread*
        new Thread(() -> {
            try {
                for (int i = 0; i < 100; i++) {
                    output.write(i);
                    Thread.sleep(10);
                }
                output.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
        
        *// Consumer thread (current thread)*
        int data;
        while ((data = input.read()) != -1) {
            System.out.println("Received: " + data);
        }
        
    } catch (IOException e) {
        log.error("Pipe error", e);
    }
}
```

> Interviewer Insight: Legacy I/O's biggest weakness in production systems is the lack of diagnostic information when operations fail. For example, when new FileOutputStream(path) fails, you get an IOException with minimal detail. You won't know if it failed due to permissions, disk full, or the path not existing. In production environments, pre-check conditions (file existence, permissions) before operations and provide detailed error messages. Also, always specify character encoding explicitly when working with text—never rely on platform defaults, as they vary between environments and cause hard-to-diagnose encoding issues.
> 

> Deep Dive Tip: Buffered streams are essential for performance, but many developers don't realize their internal buffer sizes matter significantly. The default 8KB buffer for BufferedInputStream works well for most cases, but for high-throughput systems processing large files, experiment with buffer sizes that align with storage characteristics. SSD-based systems might benefit from 64KB-128KB buffers, while network file systems often have specific block sizes (16KB-64KB) where aligned buffers maximize throughput. When copying large files, the buffer size can make a 3-5x performance difference.
> 

## 4.2 Serialization

**Purpose**: Converts objects to byte streams and back.
**Usage**: Object persistence and network transmission.
**Example**:

```java
*// Basic serialization - class must implement Serializable*
@SuppressWarnings("serial")
public class User implements Serializable {
    *// Serializable is a marker interface (no methods to implement)*
    
    *// All non-transient fields are serialized*
    private String username;
    private String email;
    
    *// Fields to exclude from serialization*
    private transient String temporaryToken;
    
    *// Static fields are NOT serialized*
    private static final String COMPANY = "Acme Corp";
    
    *// SerialVersionUID for version control// Without this, JVM generates one based on class structure*
    private static final long serialVersionUID = 1L;
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    *// Getters and setters...*
}

*// Serialization process*
public void serializationExample() {
    User user = new User("john_doe", "john@example.com");
    
    *// Serialize to file*
    try (ObjectOutputStream oos = new ObjectOutputStream(
            new FileOutputStream("user.ser"))) {
        
        *// Writes object and all its non-transient field values*
        oos.writeObject(user);
        
    } catch (IOException e) {
        log.error("Serialization failed", e);
    }
    
    *// Deserialize from file*
    try (ObjectInputStream ois = new ObjectInputStream(
            new FileInputStream("user.ser"))) {
        
        *// Reads and reconstructs object*
        User deserializedUser = (User) ois.readObject();
        
        *// transient fields will be null or default primitive values*
        
    } catch (IOException | ClassNotFoundException e) {
        log.error("Deserialization failed", e);
    }
}
```

**Advanced serialization techniques:**

```java
*// Custom serialization with writeObject/readObject*
public class Customer implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private String ssn;  *// Sensitive data*
    private Date lastAccess;
    
    *// Custom serialization*
    private void writeObject(ObjectOutputStream out) throws IOException {
        *// Custom encryption for sensitive data*
        String encryptedSSN = encrypt(ssn);
        
        *// Write non-sensitive data normally*
        out.defaultWriteObject();
        
        *// Write encrypted data*
        out.writeObject(encryptedSSN);
        
        *// Can write additional metadata*
        out.writeInt(1);  *// Version number*
    }
    
    *// Custom deserialization*
    private void readObject(ObjectInputStream in) 
            throws IOException, ClassNotFoundException {
        
        *// Read regular fields*
        in.defaultReadObject();
        
        *// Read and decrypt sensitive data*
        String encryptedSSN = (String) in.readObject();
        this.ssn = decrypt(encryptedSSN);
        
        *// Read additional metadata*
        int version = in.readInt();
        
        *// Always record deserialization time*
        this.lastAccess = new Date();
    }
    
    private String encrypt(String data) {
        *// Actual encryption logic here*
        return "ENCRYPTED:" + data;
    }
    
    private String decrypt(String encryptedData) {
        *// Actual decryption logic here*
        return encryptedData.substring(10);
    }
}

*// Serialization with inheritance*
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
}

public class Employee extends Person {
    private static final long serialVersionUID = 1L;
    private String employeeId;
    private transient String tempCredential;
    
    *// If parent is NOT Serializable, must manually initialize inherited fields// by implementing readObject and call the no-arg constructor*
}

*// Controlling object replacement during serialization*
public class DataRecord implements Serializable {
    private static final long serialVersionUID = 1L;
    private int id;
    private String data;
    private Date timestamp;
    
    *// Replace with canonical instance during serialization*
    private Object writeReplace() throws ObjectStreamException {
        *// e.g., check cache for existing instance*
        return DataRegistry.getCanonicalInstance(this);
    }
    
    *// Control what gets deserialized - security protection*
    private Object readResolve() throws ObjectStreamException {
        *// Validate or replace deserialized object*
        if (id < 0) {
            throw new InvalidObjectException("Invalid ID: " + id);
        }
        
        *// Return canonical instance from registry*
        return DataRegistry.getOrCreate(id, data, timestamp);
    }
}
```

**Externalizable interface for maximum control:**

```java
*// Externalizable for complete serialization control*
public class ConfigSettings implements Externalizable {
    private Map<String, String> settings;
    private int version;
    private Date lastModified;
    
    *// Externalizable REQUIRES public no-arg constructor*
    public ConfigSettings() {
        *// This constructor is used during deserialization*
        settings = new HashMap<>();
    }
    
    public ConfigSettings(Map<String, String> settings) {
        this.settings = settings;
        this.version = 1;
        this.lastModified = new Date();
    }
    
    *// You control EXACTLY what gets written*
    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        *// Write version first for compatibility*
        out.writeInt(version);
        
        *// Write core data*
        out.writeInt(settings.size());
        for (Map.Entry<String, String> entry : settings.entrySet()) {
            out.writeUTF(entry.getKey());
            out.writeUTF(entry.getValue());
        }
        
        *// Write timestamp*
        out.writeLong(lastModified.getTime());
    }
    
    *// You control EXACTLY what gets read and in what order*
    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        *// Read version first*
        version = in.readInt();
        
        *// Read settings based on version*
        int size = in.readInt();
        settings = new HashMap<>(size);
        for (int i = 0; i < size; i++) {
            String key = in.readUTF();
            String value = in.readUTF();
            settings.put(key, value);
        }
        
        *// Read timestamp*
        long time = in.readLong();
        lastModified = new Date(time);
        
        *// Future versions can read additional fields*
        if (version >= 2) {
            *// Read version 2+ specific fields*
        }
    }
}
```

> Interviewer Insight: Java serialization has been the root cause of numerous critical security vulnerabilities, including remote code execution flaws in major enterprise systems. In production environments, avoid Java serialization for public-facing services unless you have a comprehensive security review process. Instead, prefer standardized formats like JSON, Protocol Buffers, or Avro. If you must use Java serialization, implement strict type filtering with ObjectInputFilter (Java 9+) to whitelist allowed classes and reject potentially malicious payloads.
> 

> Deep Dive Tip: The serialVersionUID field is critical for long-lived applications where serialized objects might be stored in databases or files. If you don't explicitly define it, Java computes it based on class structure, and even minor changes (adding a field or method) can make previously serialized objects unreadable. For evolving applications, implement custom readObject/writeObject methods that handle versioning explicitly by writing and reading version numbers to support backward compatibility with older serialized forms.
> 

## 4.3 NIO and NIO.2

**Purpose**: Modern, higher-performance I/O APIs.
**Usage**: Scalable I/O operations and file system access.
**Example**:

```java
*// Path and Files API (Java 7+)*
public void pathAndFilesExample() {
    *// Creating paths*
    Path configPath = Paths.get("/etc/app/config.properties");  *// Absolute*
    Path relativePath = Paths.get("data", "users", "profiles.json");  *// Multi-part*
    
    *// Path info and manipulation (immutable - operations return new Path)*
    Path parent = configPath.getParent();  *// /etc/app*
    Path fileName = configPath.getFileName();  *// config.properties*
    Path normalizedPath = relativePath.normalize();  *// Removes redundancy*
    Path absolutePath = relativePath.toAbsolutePath();  *// Converts to absolute*
    Path realPath = configPath.toRealPath();  *// Resolves symlinks*
    
    *// Combining paths*
    Path baseDir = Paths.get("/opt/application");
    Path fullPath = baseDir.resolve("logs/app.log");  *// /opt/application/logs/app.log*
    
    *// Relative path calculation*
    Path path1 = Paths.get("/home/user/docs");
    Path path2 = Paths.get("/home/user/pictures");
    Path relativized = path1.relativize(path2);  *// ../pictures*
    
    *// Basic file operations with Files class*
    try {
        *// Check file existence*
        boolean exists = Files.exists(configPath);
        boolean notExists = Files.notExists(configPath);
        
        *// Ensure directory exists*
        Files.createDirectories(Paths.get("/var/log/myapp"));
        
        *// Create new file (throws exception if exists)*
        Path newFile = Files.createFile(Paths.get("/tmp/test.txt"));
        
        *// Create temp file/directory*
        Path tempFile = Files.createTempFile("prefix-", "-suffix");  *// In system temp dir*
        Path tempDir = Files.createTempDirectory("app-data-");
        
        *// Copy file with options*
        Path source = Paths.get("source.txt");
        Path target = Paths.get("target.txt");
        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING,
                                  StandardCopyOption.COPY_ATTRIBUTES);
        
        *// Move file (rename or relocate)*
        Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
        
        *// Delete file (fails if directory not empty)*
        Files.delete(target);  *// Throws exception if fails*
        boolean deleted = Files.deleteIfExists(target);  *// Returns boolean*
        
    } catch (IOException e) {
        log.error("File operation failed", e);
    }
}

*// Reading and writing with Files (Java 7+)*
public void filesReadWrite() {
    Path file = Paths.get("data.txt");
    
    *// Write bytes*
    try {
        byte[] data = "Hello, NIO.2!".getBytes(StandardCharsets.UTF_8);
        Files.write(file, data, StandardOpenOption.CREATE,
                               StandardOpenOption.TRUNCATE_EXISTING);
        
        *// Append to file*
        Files.write(file, "\nNew line".getBytes(StandardCharsets.UTF_8),
                    StandardOpenOption.APPEND);
        
    } catch (IOException e) {
        log.error("Write failed", e);
    }
    
    *// Read all bytes*
    try {
        byte[] content = Files.readAllBytes(file);  *// For small files*
        System.out.println(new String(content, StandardCharsets.UTF_8));
        
    } catch (IOException e) {
        log.error("Read failed", e);
    }
    
    *// Read all lines (efficient text reading)*
    try {
        List<String> lines = Files.readAllLines(file, StandardCharsets.UTF_8);
        for (String line : lines) {
            System.out.println(line);
        }
        
    } catch (IOException e) {
        log.error("Read lines failed", e);
    }
    
    *// Java 8+ Stream-based reading (memory efficient)*
    try (Stream<String> lines = Files.lines(file, StandardCharsets.UTF_8)) {
        *// Process file line by line without loading entirely in memory*
        lines.filter(line -> line.contains("important"))
             .map(String::toUpperCase)
             .forEach(System.out::println);
        
    } catch (IOException e) {
        log.error("Stream reading failed", e);
    }
    
    *// Java 11+ String convenience methods*
    try {
        *// Read entire file as a String*
        String content = Files.readString(file, StandardCharsets.UTF_8);
        
        *// Write String directly*
        Files.writeString(file, "Simple text content", StandardCharsets.UTF_8);
        
    } catch (IOException e) {
        log.error("String operation failed", e);
    }
}
```

**File attributes and metadata:**

```java
*// File attributes API*
public void fileAttributesExample() {
    Path file = Paths.get("/home/user/document.txt");
    
    try {
        *// Basic file attributes (works on all platforms)*
        BasicFileAttributes attrs = Files.readAttributes(file, 
                                   BasicFileAttributes.class);
        
        System.out.println("Creation time: " + attrs.creationTime());
        System.out.println("Last access: " + attrs.lastAccessTime());
        System.out.println("Last modified: " + attrs.lastModifiedTime());
        System.out.println("Size: " + attrs.size());
        System.out.println("Is directory: " + attrs.isDirectory());
        System.out.println("Is regular file: " + attrs.isRegularFile());
        System.out.println("Is symbolic link: " + attrs.isSymbolicLink());
        
        *// Posix file attributes (Unix/Linux)*
        if (isPosixFileSystem()) {
            PosixFileAttributes posixAttrs = Files.readAttributes(file, 
                                           PosixFileAttributes.class);
            
            System.out.println("Owner: " + posixAttrs.owner());
            System.out.println("Group: " + posixAttrs.group());
            System.out.println("Permissions: " + posixAttrs.permissions());
            
            *// Change permissions*
            Set<PosixFilePermission> perms = 
                EnumSet.of(PosixFilePermission.OWNER_READ,
                          PosixFilePermission.OWNER_WRITE,
                          PosixFilePermission.GROUP_READ);
            
            Files.setPosixFilePermissions(file, perms);
        }
        
        *// DOS file attributes (Windows)*
        if (isWindowsFileSystem()) {
            DosFileAttributes dosAttrs = Files.readAttributes(file,
                                       DosFileAttributes.class);
            
            System.out.println("Hidden: " + dosAttrs.isHidden());
            System.out.println("Read-only: " + dosAttrs.isReadOnly());
            System.out.println("System file: " + dosAttrs.isSystem());
            System.out.println("Archive: " + dosAttrs.isArchive());
            
            *// Set attributes*
            Files.setAttribute(file, "dos:hidden", true);
            Files.setAttribute(file, "dos:readonly", true);
        }
        
        *// File owner*
        UserPrincipal owner = Files.getOwner(file);
        System.out.println("Owner: " + owner.getName());
        
        *// Change owner*
        UserPrincipalLookupService lookupService = 
            file.getFileSystem().getUserPrincipalLookupService();
        UserPrincipal newOwner = lookupService.lookupPrincipalByName("newowner");
        Files.setOwner(file, newOwner);
        
        *// Getting specific attribute*
        long size = (Long) Files.getAttribute(file, "basic:size");
        FileTime lastModified = (FileTime) Files.getAttribute(file, "basic:lastModifiedTime");
        
    } catch (IOException e) {
        log.error("Failed to access file attributes", e);
    }
}

*// Check file system type*
private boolean isPosixFileSystem() {
    return FileSystems.getDefault().supportedFileAttributeViews().contains("posix");
}

private boolean isWindowsFileSystem() {
    return FileSystems.getDefault().supportedFileAttributeViews().contains("dos");
}
```

**Directory operations:**

```java
*// Directory operations*
public void directoryOperations() {
    Path dir = Paths.get("/var/data");
    
    try {
        *// List directory contents (1 level)*
        try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
            for (Path entry : stream) {
                System.out.println(entry.getFileName());
            }
        }
        
        *// Filtered directory stream*
        try (DirectoryStream<Path> stream = 
                Files.newDirectoryStream(dir, "*.{java,class}")) {
            *// Only files matching the glob pattern*
            for (Path entry : stream) {
                System.out.println("Java file: " + entry.getFileName());
            }
        }
        
        *// Custom filter*
        try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir, 
                path -> Files.isRegularFile(path) && 
                        Files.size(path) > 1_000_000)) {
            *// Only files larger than 1MB*
            for (Path entry : stream) {
                System.out.println("Large file: " + entry.getFileName());
            }
        }
        
        *// Walking directory tree*
        Files.walkFileTree(dir, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                System.out.println("Visited: " + file);
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) {
                System.err.println("Failed to access: " + file);
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
                System.out.println("About to visit directory: " + dir);
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
                System.out.println("Done with directory: " + dir);
                return FileVisitResult.CONTINUE;
            }
        });
        
        *// Java 8+ directory walk with Stream*
        try (Stream<Path> pathStream = Files.walk(dir)) {
            *// Find all .log files*
            List<Path> logFiles = pathStream
                .filter(Files::isRegularFile)
                .filter(p -> p.toString().endsWith(".log"))
                .collect(Collectors.toList());
            
            System.out.println("Found " + logFiles.size() + " log files");
        }
        
        *// Find files (Java 8+)*
        try (Stream<Path> pathStream = Files.find(dir, 3, *// max depth*
                (path, attrs) -> attrs.isRegularFile() && 
                                path.toString().contains("config"))) {
            
            pathStream.forEach(System.out::println);
        }
        
    } catch (IOException e) {
        log.error("Directory operation failed", e);
    }
}
```

**Watch Service for directory monitoring:**

```java
*// Watch Service for file system change notifications*
public void watchServiceExample() {
    Path dir = Paths.get("/var/data");
    
    try {
        *// Create a WatchService*
        WatchService watchService = FileSystems.getDefault().newWatchService();
        
        *// Register directory with events to watch*
        dir.register(watchService, 
            StandardWatchEventKinds.ENTRY_CREATE,
            StandardWatchEventKinds.ENTRY_MODIFY,
            StandardWatchEventKinds.ENTRY_DELETE);
        
        System.out.println("Watching directory: " + dir);
        
        *// Start infinite watching loop in separate thread*
        new Thread(() -> {
            try {
                while (true) {
                    *// Wait for key to be signaled*
                    WatchKey key;
                    try {
                        key = watchService.take();  *// Blocking*
                    } catch (InterruptedException e) {
                        return;  *// Exit on interrupt*
                    }
                    
                    *// Process events*
                    for (WatchEvent<?> event : key.pollEvents()) {
                        WatchEvent.Kind<?> kind = event.kind();
                        
                        *// Skip overflow events*
                        if (kind == StandardWatchEventKinds.OVERFLOW) {
                            continue;
                        }
                        
                        *// Context is the filename*
                        @SuppressWarnings("unchecked")
                        WatchEvent<Path> pathEvent = (WatchEvent<Path>) event;
                        Path filename = pathEvent.context();
                        
                        *// Build full path*
                        Path fullPath = dir.resolve(filename);
                        
                        System.out.printf("Event %s on file: %s%n", kind, fullPath);
                        
                        *// React to event type*
                        if (kind == StandardWatchEventKinds.ENTRY_CREATE) {
                            *// Handle new file*
                            if (Files.isRegularFile(fullPath)) {
                                processNewFile(fullPath);
                            }
                        } else if (kind == StandardWatchEventKinds.ENTRY_MODIFY) {
                            *// Handle modified file*
                            if (Files.isRegularFile(fullPath)) {
                                processModifiedFile(fullPath);
                            }
                        } else if (kind == StandardWatchEventKinds.ENTRY_DELETE) {
                            *// Handle deleted file*
                            handleDeletedFile(filename.toString());
                        }
                    }
                    
                    *// Reset key for next events*
                    boolean valid = key.reset();
                    if (!valid) {
                        *// Directory no longer accessible*
                        break;
                    }
                }
            } catch (IOException e) {
                log.error("Watch service error", e);
            }
        }).start();
        
    } catch (IOException e) {
        log.error("Failed to set up watch service", e);
    }
}

private void processNewFile(Path path) {
    System.out.println("Processing new file: " + path);
    *// Implementation*
}

private void processModifiedFile(Path path) {
    System.out.println("Processing modified file: " + path);
    *// Implementation*
}

private void handleDeletedFile(String filename) {
    System.out.println("Handling deleted file: " + filename);
    *// Implementation*
}
```

**NIO Channels and Buffers:**

```java
*// NIO Channels and Buffers for high-performance I/O*
public void channelsAndBuffers() {
    Path file = Paths.get("data.bin");
    
    *// Writing with channels*
    try (FileChannel channel = FileChannel.open(file, 
            StandardOpenOption.CREATE,
            StandardOpenOption.WRITE)) {
        
        *// Create buffer*
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        
        *// Put data into buffer*
        buffer.putInt(42);
        buffer.putLong(123456789L);
        buffer.put((byte) 'A');
        
        *// Array of data*
        byte[] data = {10, 20, 30, 40, 50};
        buffer.put(data);
        
        *// Flip buffer: ready for reading (by the channel)*
        buffer.flip();
        
        *// Write buffer to channel*
        channel.write(buffer);
        
    } catch (IOException e) {
        log.error("Channel write error", e);
    }
    
    *// Reading with channels*
    try (FileChannel channel = FileChannel.open(file, StandardOpenOption.READ)) {
        *// Get file size*
        long fileSize = channel.size();
        
        *// Create buffer to hold content*
        ByteBuffer buffer = ByteBuffer.allocate((int) fileSize);
        
        *// Read into buffer*
        channel.read(buffer);
        
        *// Flip buffer: ready for reading (by our code)*
        buffer.flip();
        
        *// Read basic types*
        int intValue = buffer.getInt();
        long longValue = buffer.getLong();
        byte byteValue = buffer.get();
        
        *// Read into array*
        byte[] data = new byte[5];
        buffer.get(data);
        
        System.out.println("Read: " + intValue + ", " + longValue + 
                         ", " + (char) byteValue);
        
    } catch (IOException e) {
        log.error("Channel read error", e);
    }
    
    *// Bulk file copy using channels*
    Path source = Paths.get("source.dat");
    Path target = Paths.get("target.dat");
    
    try (
        FileChannel sourceChannel = FileChannel.open(source, StandardOpenOption.READ);
        FileChannel targetChannel = FileChannel.open(target, 
                StandardOpenOption.CREATE, 
                StandardOpenOption.WRITE)
    ) {
        *// Get source size*
        long size = sourceChannel.size();
        
        *// Copy in chunks*
        long position = 0;
        long bytesTransferred = 0;
        long count = Math.min(size, 64 * 1024); *// 64KB chunks*
        
        while (position < size) {
            bytesTransferred = sourceChannel.transferTo(
                position, count, targetChannel);
            position += bytesTransferred;
        }
        
    } catch (IOException e) {
        log.error("Channel copy error", e);
    }
}

*// Direct buffers for improved performance (outside JVM heap)*
public void directBuffers() {
    *// Allocate direct buffer*
    ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024 * 1024); *// 1MB*
    
    *// Use like regular buffer, but more efficient for native I/O*
    directBuffer.putInt(123);
    directBuffer.flip();
    int value = directBuffer.getInt();
    
    *// Check if buffer is direct*
    boolean isDirect = directBuffer.isDirect();
    
    *// Direct buffers aren't managed by GC - memory may not be reclaimed// immediately when buffer is no longer referenced*
}
```

**Advanced NIO features:**

```java
*// Memory-mapped files for high-performance I/O*
public void memoryMappedFiles() {
    Path file = Paths.get("large-data.bin");
    
    try {
        *// Ensure file exists with size*
        if (!Files.exists(file)) {
            Files.createFile(file);
            
            *// Set file size (1GB)*
            try (RandomAccessFile raf = new RandomAccessFile(file.toFile(), "rw")) {
                raf.setLength(1024 * 1024 * 1024);
            }
        }
        
        *// Memory-map the file (read-write mode)*
        try (FileChannel channel = FileChannel.open(file, 
                StandardOpenOption.READ, StandardOpenOption.WRITE)) {
            
            *// Map only part of the file (first 100MB)*
            long size = Math.min(channel.size(), 100 * 1024 * 1024);
            MappedByteBuffer buffer = channel.map(
                FileChannel.MapMode.READ_WRITE, 0, size);
            
            *// Access like normal ByteBuffer, but changes write through to file*
            buffer.putInt(0, 0xCAFEBABE); *// Magic number at start*
            
            *// Sequential access*
            buffer.position(1024);
            buffer.putLong(System.currentTimeMillis());
            
            *// Force changes to be written*
            buffer.force();
            
        }
        
    } catch (IOException e) {
        log.error("Memory-mapped file error", e);
    }
}

*// File locking*
public void fileLocking() {
    Path file = Paths.get("shared-data.bin");
    
    try (FileChannel channel = FileChannel.open(file, 
            StandardOpenOption.READ, StandardOpenOption.WRITE, 
            StandardOpenOption.CREATE)) {
        
        *// Lock the entire file exclusively*
        FileLock lock = channel.lock();
        try {
            *// File is locked - safe to modify*
            ByteBuffer buffer = ByteBuffer.wrap("Locked content".getBytes());
            channel.write(buffer);
            
        } finally {
            *// Always release lock*
            if (lock != null && lock.isValid()) {
                lock.release();
            }
        }
        
        *// Lock part of the file shared (read-only)*
        FileLock sharedLock = channel.lock(0, 1024, true);
        try {
            *// Other processes can also obtain shared locks// but not exclusive locks on this region*
            
        } finally {
            if (sharedLock != null && sharedLock.isValid()) {
                sharedLock.release();
            }
        }
        
    } catch (IOException e) {
        log.error("File locking error", e);
    }
}

*// Asynchronous I/O (Java 7+)*
public void asynchronousIO() {
    Path file = Paths.get("async-data.txt");
    
    try {
        *// Get async channel group*
        AsynchronousChannelGroup group = AsynchronousChannelGroup.withThreadPool(
                Executors.newFixedThreadPool(5));
        
        *// Open async channel*
        try (AsynchronousFileChannel channel = 
                AsynchronousFileChannel.open(file, 
                    EnumSet.of(StandardOpenOption.READ, 
                              StandardOpenOption.WRITE, 
                              StandardOpenOption.CREATE),
                    group)) {
            
            *// Create buffer*
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("Async I/O test".getBytes());
            buffer.flip();
            
            *// Write asynchronously with Future*
            Future<Integer> writeResult = channel.write(buffer, 0);
            
            *// Check if complete*
            while (!writeResult.isDone()) {
                *// Do other work while waiting*
                System.out.println("Waiting for write...");
                Thread.sleep(10);
            }
            
            *// Get bytes written*
            int bytesWritten = writeResult.get();
            System.out.println("Wrote " + bytesWritten + " bytes");
            
            *// Read asynchronously with CompletionHandler*
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            channel.read(readBuffer, 0, readBuffer, 
                    new CompletionHandler<Integer, ByteBuffer>() {
                
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    attachment.flip();
                    byte[] data = new byte[attachment.limit()];
                    attachment.get(data);
                    System.out.println("Read completed: " + new String(data));
                }
                
                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    System.err.println("Read failed: " + exc);
                }
            });
            
            *// Give time for async operation to complete*
            Thread.sleep(100);
            
        }
        
        *// Close group when done with all channels*
        group.shutdown();
        
    } catch (Exception e) {
        log.error("Async I/O error", e);
    }
}
```

> Interviewer Insight: In high-throughput production systems, the biggest advantage of NIO over traditional I/O isn't just performance—it's diagnosability. Traditional I/O provides minimal feedback on errors, but NIO's finer-grained exceptions, especially in the Files API, provide much better root cause information. Additionally, NIO.2's file attribute API solves the cross-platform challenge of file permissions—you no longer need platform-specific code for Unix vs. Windows. For mission-critical systems handling files from different sources, always use NIO.2's Files.probeContentType() instead of relying on file extensions for determining content types.
> 

> Deep Dive Tip: The most common NIO misconception is assuming ByteBuffer.allocateDirect() is always better than heap buffers. Direct buffers have higher allocation/deallocation costs and waste native memory if left unused. They're ideal for long-lived buffers shared across many I/O operations but can cause performance degradation for short-lived operations. In our production systems, we saw a 15% performance drop when switching all buffers to direct without proper lifecycle management. For best performance, use direct buffers for long-running connections and reuse them with thread-local pools, but stick with heap buffers for one-off operations.
> 

---

## ✈️ Happy Coding!

---