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