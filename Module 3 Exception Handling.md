# Module 3: Exception Handling

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

---

## ✈️ Happy Coding!

---