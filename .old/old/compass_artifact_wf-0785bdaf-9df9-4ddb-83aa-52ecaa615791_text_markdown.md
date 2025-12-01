# Complete Programming Cheat Sheet: Java, Spring Boot 3.0, MySQL, Docker & More

This comprehensive cheat sheet covers essential programming concepts and practical syntax for modern Java development. Each section includes copy-paste ready examples and simple explanations.

## Java Core Concepts

### Object-Oriented Programming fundamentals

**Encapsulation** wraps data and methods together with access control:

```java
public class Employee {
    private String name;    // Private fields
    private int salary;

    public String getName() { return name; }
    public void setSalary(int salary) {
        if(salary > 0) this.salary = salary;
    }
}
```

**Inheritance** allows classes to inherit properties from parent classes:

```java
class Animal {
    protected String name;
    void eat() { System.out.println("Animal is eating"); }
}

class Dog extends Animal {
    void bark() { System.out.println("Dog is barking"); }

    @Override
    void eat() { System.out.println("Dog is eating bones"); }
}
```

**Polymorphism** enables multiple implementations of the same interface:

```java
// Method Overloading (Compile-time)
class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
}

// Method Overriding (Runtime)
Shape[] shapes = {new Circle(), new Rectangle()};
for(Shape shape : shapes) {
    shape.draw(); // Calls appropriate overridden method
}
```

### Collections framework essentials

```java
// ArrayList - Resizable array, best for random access
List<String> arrayList = new ArrayList<>();
arrayList.add("Java");
arrayList.add(1, "Python");           // Insert at index
arrayList.set(0, "Kotlin");           // Replace

// HashMap - Key-value pairs, O(1) access
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("Java", 25);
int age = hashMap.getOrDefault("Python", 0);

// HashSet - No duplicates, fast lookup
Set<String> hashSet = new HashSet<>();
hashSet.add("Apple");
hashSet.add("Apple");    // Duplicate ignored

// TreeSet - Sorted set
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(5); treeSet.add(1); treeSet.add(3);
System.out.println(treeSet); // [1, 3, 5]
```

### Generics and type safety

```java
// Generic class
public class Box<T> {
    private T content;
    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

// Wildcards for flexibility
List<? extends Number> numbers = new ArrayList<Integer>();  // Read-only
List<? super Integer> integers = new ArrayList<Number>();   // Write-capable

// Bounded type parameters
public class NumberContainer<T extends Number> {
    private T value;
    public double getDoubleValue() {
        return value.doubleValue(); // Can call Number methods
    }
}
```

### Streams API for data processing

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// Transform and filter data
List<String> result = words.stream()
    .filter(s -> s.length() > 4)        // Filter elements
    .map(String::toUpperCase)           // Transform elements
    .distinct()                         // Remove duplicates
    .sorted()                          // Sort elements
    .collect(Collectors.toList());      // Collect results

// Common terminal operations
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);
numbers.stream().forEach(System.out::println);

// Grouping and collecting
Map<Integer, List<String>> wordsByLength = words.stream()
    .collect(Collectors.groupingBy(String::length));
```

### Exception handling patterns

```java
// Basic exception handling
try {
    int result = riskyOperation();
} catch (SpecificException e) {
    System.out.println("Specific error: " + e.getMessage());
} catch (Exception e) {
    System.out.println("General error: " + e.getMessage());
} finally {
    cleanup(); // Always executed
}

// Try-with-resources (automatic resource management)
try (FileInputStream fis = new FileInputStream("file.txt");
     BufferedReader br = new BufferedReader(new InputStreamReader(fis))) {
    return br.readLine();
} catch (IOException e) {
    System.out.println("File operation failed: " + e.getMessage());
}

// Custom exceptions
public class CustomBusinessException extends Exception {
    public CustomBusinessException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Lambda expressions and functional interfaces

```java
// Basic lambda syntax
Runnable r = () -> System.out.println("Hello World");
Consumer<String> printer = s -> System.out.println(s);
Function<String, Integer> getLength = String::length;
Predicate<String> startsWithA = name -> name.startsWith("A");

// Method references
words.sort(String::compareToIgnoreCase);          // Static method
words.forEach(System.out::println);               // Instance method
words.stream().map(String::toUpperCase).collect(Collectors.toList());

// Functional interface usage
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::length)
    .forEach(System.out::println);
```

## Spring Boot 3.0 Complete Guide

### Critical changes in Spring Boot 3.0

**System Requirements:**

- Java 17 minimum (tested up to Java 21)
- Jakarta EE 9+ (all `javax.*` → `jakarta.*`)
- Native image support with GraalVM
- Spring Framework 6.0+ required

**Breaking Changes:**

```java
// Old (Java EE)
import javax.persistence.Entity;
import javax.servlet.http.HttpServletRequest;

// New (Jakarta EE)
import jakarta.persistence.Entity;
import jakarta.servlet.http.HttpServletRequest;
```

### Essential annotations for development

```java
// Stereotype annotations
@Service
public class UserService {
    private final UserRepository userRepository;

    // Constructor injection (no @Autowired needed for single constructor)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByActiveTrue();

    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
}

@RestController
@RequestMapping("/api/v1/users")
public class UserRestController {

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse("User not found", ex.getMessage());
    }
}
```

### Configuration properties and application setup

```yaml
# application.yml - Modern Spring Boot 3.0 configuration
server:
  port: 8080
  ssl:
    bundle: "web-server" # New SSL Bundles feature

spring:
  application:
    name: my-spring-app

  # Profile activation (new syntax)
  config:
    activate:
      on-profile: dev

  # DataSource with Jakarta EE
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER:user}
    password: ${DB_PASSWORD:password}
    hikari:
      maximum-pool-size: 20
      connection-timeout: 30000

  # JPA Configuration
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
    show-sql: false

# New observability features
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
```

### Spring Security 6 modern configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/products/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/products/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))
            );
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Dependency injection best practices

```java
// Constructor injection (recommended)
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository, PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }
}

// Qualifier for multiple implementations
@Service
public class NotificationService {
    private final MessageService emailService;
    private final MessageService smsService;

    public NotificationService(
            @Qualifier("emailMessageService") MessageService emailService,
            @Qualifier("smsMessageService") MessageService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}
```

### Microservices patterns with Spring Cloud

```java
// API Gateway with Spring Cloud Gateway
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r.path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(2)
                    .circuitBreaker(config -> config.setName("user-service-cb"))
                )
                .uri("lb://user-service")
            )
            .build();
    }
}

// Circuit Breaker with Resilience4j
@Service
public class ExternalApiService {

    @CircuitBreaker(name = "payment-service", fallbackMethod = "fallbackPayment")
    @TimeLimiter(name = "payment-service")
    @Retry(name = "payment-service")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> paymentClient.processPayment(request));
    }

    public CompletableFuture<PaymentResponse> fallbackPayment(PaymentRequest request, Exception ex) {
        return CompletableFuture.completedFuture(
            PaymentResponse.builder()
                .status("PENDING")
                .message("Payment will be processed later")
                .build()
        );
    }
}
```

### Native compilation with GraalVM

```bash
# Build native executable
./mvnw clean package -Pnative

# Native Docker image
./mvnw spring-boot:build-image -Pnative

# Run native executable (2-10x faster startup)
./target/my-app
```

## JPA/Hibernate Database Mastery

### Entity annotations and relationships

```java
@Entity
@Table(name = "users", schema = "public")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
    @SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
    private Long id;

    @Column(name = "username", length = 50, nullable = false, unique = true)
    private String username;

    @Version
    private Long version;          // Optimistic locking

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    @Transient
    private String temporaryField; // Excluded from persistence
}
```

### Relationship mappings made simple

```java
// One-to-Many (Department has many Employees)
@Entity
public class Department {
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Employee> employees = new ArrayList<>();
}

@Entity
public class Employee {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id", nullable = false)
    private Department department;
}

// Many-to-Many (Students enroll in Courses)
@Entity
public class Student {
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}
```

### Spring Data JPA query methods

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Query method naming (automatically implemented)
    List<User> findByUsernameContaining(String username);
    List<User> findByEmailAndStatusNot(String email, UserStatus status);
    List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // Custom JPQL queries
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);

    // Native SQL queries
    @Query(value = "SELECT * FROM users WHERE username LIKE %:name%", nativeQuery = true)
    List<User> findByUsernameNative(@Param("name") String name);

    // Modifying queries (UPDATE/DELETE)
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateUserStatus(@Param("id") Long id, @Param("status") UserStatus status);

    // Pagination and sorting
    Page<User> findByStatus(UserStatus status, Pageable pageable);
}
```

### Performance optimization techniques

```java
// Entity graphs for efficient fetching
@NamedEntityGraph(
    name = "User.profile",
    attributeNodes = @NamedAttributeNode("profile")
)
@Entity
public class User { /* ... */ }

// Usage in repository
@EntityGraph(value = "User.profile", type = EntityGraph.EntityGraphType.FETCH)
Optional<User> findById(Long id);

// Read-only queries
@Query("SELECT u FROM User u WHERE u.status = :status")
@QueryHints(@QueryHint(name = QueryHints.HINT_READONLY, value = "true"))
List<User> findReadOnlyUsers(@Param("status") UserStatus status);

// Batch processing
@Service
@Transactional
public class BatchService {
    @PersistenceContext
    private EntityManager entityManager;

    public void batchInsert(List<User> users) {
        int batchSize = 20;
        for (int i = 0; i < users.size(); i++) {
            entityManager.persist(users.get(i));
            if (i % batchSize == 0) {
                entityManager.flush();
                entityManager.clear();
            }
        }
    }
}
```

### JPA configuration for performance

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        jdbc:
          batch_size: 20 # Batch multiple operations
          fetch_size: 50 # Fetch size for result sets
        order_inserts: true # Order inserts for better batching
        order_updates: true # Order updates for better batching
        cache:
          use_second_level_cache: true
          use_query_cache: true
        generate_statistics: true # Monitor performance
```

## MySQL Database Essentials

### Basic commands for daily operations

```sql
-- Database operations
CREATE DATABASE IF NOT EXISTS myapp;
USE myapp;
SHOW DATABASES;

-- Table creation with proper data types
CREATE TABLE users (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB;

-- Data manipulation
INSERT INTO users (username, email) VALUES
('john_doe', 'john@example.com'),
('jane_smith', 'jane@example.com');

SELECT id, username, email FROM users WHERE username LIKE '%john%';
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;
DELETE FROM users WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

### Indexing strategies for performance

```sql
-- Primary key (automatically indexed)
ALTER TABLE products ADD PRIMARY KEY (product_id);

-- Unique index for business constraints
CREATE UNIQUE INDEX idx_product_code ON products (product_code);

-- Regular index for frequent WHERE clauses
CREATE INDEX idx_category ON products (category);

-- Composite index for multiple columns
CREATE INDEX idx_category_price ON products (category, price);

-- Covering index (includes all query columns)
CREATE INDEX idx_covering ON products (category, price, name);

-- Show indexes
SHOW INDEX FROM products;

-- Drop unused indexes
DROP INDEX idx_unused ON products;
```

### Query optimization fundamentals

```sql
-- ✅ Good: Use specific columns
SELECT id, name, price FROM products WHERE category = 'electronics';

-- ❌ Bad: SELECT * with function on indexed column
SELECT * FROM products WHERE UPPER(category) = 'ELECTRONICS';

-- ✅ Good: Sargable queries (Search ARGument ABLE)
SELECT * FROM orders WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01';

-- ✅ Prefer JOINs over subqueries
SELECT DISTINCT c.name
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date > '2023-01-01';

-- ✅ Use EXISTS instead of IN for subqueries
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.total > 1000
);
```

### Performance tuning with EXPLAIN

```sql
-- Analyze query execution plan
EXPLAIN SELECT * FROM products WHERE price > 100;

-- Detailed execution plan (MySQL 8.0+)
EXPLAIN FORMAT=TREE SELECT p.name, s.name
FROM products p JOIN suppliers s ON p.supplier_id = s.supplier_id;

-- Actual execution analysis with timing
EXPLAIN ANALYZE SELECT * FROM products WHERE category = 'electronics';
```

**Key EXPLAIN columns to understand:**

- **type**: Join type (`const` > `eq_ref` > `ref` > `range` > `index` > `ALL`)
- **key**: Which index is used (NULL means no index)
- **rows**: Estimated rows examined
- **Extra**: Additional info (`Using index` is good, `Using filesort` may indicate optimization needed)

### MySQL configuration optimization

```ini
# my.cnf - Essential performance settings
[mysqld]
# InnoDB Buffer Pool (50-75% of available RAM)
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 8

# Log files (25% of buffer pool size)
innodb_log_file_size = 1G
innodb_log_buffer_size = 64M

# Connection settings
max_connections = 500
thread_cache_size = 128
connect_timeout = 5
wait_timeout = 300

# Per-connection buffers (keep small)
sort_buffer_size = 256K
join_buffer_size = 256K
read_buffer_size = 128K

# Table settings
table_open_cache = 4000
table_definition_cache = 2000
```

### SQL performance best practices

**Always do:**

- Use specific column names instead of `SELECT *`
- Add indexes on `WHERE`, `JOIN`, and `ORDER BY` columns
- Use `LIMIT` for large result sets
- Prefer JOINs over subqueries when possible
- Use appropriate data types (don't over-allocate)

**Avoid:**

- Functions on indexed columns in WHERE clauses
- `SELECT *` without proper indexing
- Large `OFFSET` values in pagination
- Unnecessary `DISTINCT` operations
- Correlated subqueries when JOINs work better

## Docker Containerization Guide

### Essential commands for daily development

```bash
# Build and run containers
docker build -t myapp:latest .
docker run -p 8080:8080 -d --name spring-app myapp:latest

# Container management
docker ps                    # Running containers
docker ps -a                # All containers
docker stop spring-app      # Stop container
docker restart spring-app   # Restart container
docker logs -f spring-app   # Follow logs
docker exec -it spring-app bash  # Access container shell

# Image management
docker images               # List images
docker pull openjdk:17-jre-alpine
docker rmi myapp:latest     # Remove image
docker system prune         # Clean up unused resources
```

### Dockerfile optimization for Spring Boot

```dockerfile
# Multi-stage build for optimal size
FROM maven:3.8.6-eclipse-temurin-17-alpine AS builder
WORKDIR /build
COPY . .
RUN mvn clean package -DskipTests

# Production stage
FROM eclipse-temurin:17-jre-alpine
RUN addgroup --system spring && adduser -S spring -G spring
USER spring:spring
WORKDIR /app

# Copy JAR from build stage
COPY --from=builder /build/target/*.jar app.jar

# JVM optimization for containers
ENV JAVA_OPTS="-XX:MaxRAMPercentage=80.0 -XX:+UseContainerSupport"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Docker Compose for full-stack development

```yaml
version: "3.8"

services:
  app:
    build: .
    container_name: spring-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/myapp
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
    depends_on:
      - db
      - redis
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: redis-cache
    ports:
      - "6379:6379"
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose commands

```bash
# Start all services
docker-compose up -d

# Build and start (rebuild images)
docker-compose up --build

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs -f app

# Scale services
docker-compose up -d --scale app=3

# Execute commands in service
docker-compose exec app bash
```

### Production deployment best practices

```bash
# Build with specific version
docker build -t myapp:1.0.0 .

# Run with resource limits
docker run -d \
  --name myapp-prod \
  --memory=512m \
  --cpus=0.5 \
  --restart=unless-stopped \
  --health-cmd="curl -f http://localhost:8080/actuator/health || exit 1" \
  --health-interval=30s \
  -p 8080:8080 \
  myapp:1.0.0

# Monitor container health
docker stats myapp-prod
docker inspect --format='{{.State.Health.Status}}' myapp-prod
```

### Docker security essentials

```dockerfile
# Use official, minimal base images
FROM eclipse-temurin:17-jre-alpine

# Don't run as root
RUN addgroup --system spring && adduser -S spring -G spring
USER spring:spring

# Set proper file permissions
COPY --chown=spring:spring target/*.jar app.jar

# Add health checks
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --spider -q http://localhost:8080/actuator/health || exit 1
```

**.dockerignore for faster builds:**

```
target/
!target/*.jar
.git
.idea
*.log
README.md
Dockerfile*
docker-compose*
.dockerignore
```

## Performance Optimization Quick Reference

### Java Collections Choosing Guide

| Use Case                      | Best Choice | Why                              |
| ----------------------------- | ----------- | -------------------------------- |
| Random access, frequent reads | ArrayList   | O(1) indexed access              |
| Frequent insertions/deletions | LinkedList  | O(1) add/remove at ends          |
| Fast lookup, no duplicates    | HashSet     | O(1) average access              |
| Sorted set, no duplicates     | TreeSet     | O(log n) operations, sorted      |
| Key-value fast lookup         | HashMap     | O(1) average access              |
| Sorted map by keys            | TreeMap     | O(log n) operations, sorted keys |

### Database Performance Checklist

**Query Optimization:**

- [ ] Use specific columns instead of `SELECT *`
- [ ] Add indexes on `WHERE`, `JOIN`, `ORDER BY` columns
- [ ] Use `LIMIT` for large result sets
- [ ] Prefer JOINs over subqueries when combining data
- [ ] Use `EXISTS` instead of `IN` for existence checks
- [ ] Avoid functions on indexed columns in WHERE clauses

**Configuration Optimization:**

- [ ] Set `innodb_buffer_pool_size` to 50-75% of RAM
- [ ] Keep per-connection buffers small (`sort_buffer_size`, `join_buffer_size`)
- [ ] Enable slow query log for queries > 2 seconds
- [ ] Monitor `SHOW STATUS` for buffer hit rates
- [ ] Use `ANALYZE TABLE` regularly for updated statistics

### Spring Boot Performance Tips

**Application Startup:**

- Use `@Lazy` annotation for non-critical beans
- Configure `spring.main.lazy-initialization=true` for development
- Use `@ConditionalOnProperty` instead of `@Profile` for native compilation
- Enable native image compilation for 2-10x faster startup

**Database Access:**

- Use `FetchType.LAZY` by default for associations
- Configure `hibernate.jdbc.batch_size=20` for batch operations
- Use `@EntityGraph` to control fetching strategy per query
- Add `@QueryHints(HINT_READONLY)` for read-only queries

### Docker Performance Best Practices

**Image Optimization:**

- Use multi-stage builds to minimize final image size
- Order Dockerfile layers from least to most frequently changed
- Use `.dockerignore` to exclude unnecessary files from build context
- Use specific base image versions, not `latest`

**Container Configuration:**

- Set memory limits: `docker run --memory=512m`
- Use health checks for proper orchestration
- Configure JVM for containers: `-XX:MaxRAMPercentage=80.0 -XX:+UseContainerSupport`
- Use non-root users for security

This comprehensive cheat sheet provides practical, copy-paste ready examples for modern Java development with Spring Boot 3.0, optimized database operations, and containerized deployment. Keep it handy for quick reference during development and use the performance tips to optimize your applications.
