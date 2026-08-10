# Overview

- Java is a statically typed, object-oriented, general-purpose programming language

- Java source code is compiled into bytecode, which runs on a Java Virtual Machine (JVM)

- The main pieces are:

    JDK (Java Development Kit): Compiler, debugger, standard tools, and runtime needed to develop Java

    JVM (Java Virtual Machine): Executes Java bytecode and manages runtime behavior

    JRE (Java Runtime Environment): Historical name for a JVM plus runtime libraries. Modern JDK distributions provide the runtime

- Java's "write once, run anywhere" idea comes from compiling the same bytecode for any compatible JVM

- Java manages memory with garbage collection, but resources such as files, sockets, and database connections still need explicit closing

- Java releases every six months. Long-Term Support releases are common production baselines. Check the project's build file before using newer language features

# Compilation and Basic Program

    ```java
    public class Main {
        public static void main(String[] args) {
            System.out.println("Hello World");
        }
    }
    ```

- File name and public class name must match: `Main.java`

- Compile and run:

    ```bash
    javac Main.java
    java Main
    ```

- `javac` converts source into `Main.class` bytecode. `java` starts the JVM and calls `main`

- `public`: Accessible to the JVM and other code

- `static`: Belongs to the class, so no `Main` object is required

- `void`: Returns no value

- `String[] args`: Command-line arguments

- Statements normally end with `;`. Blocks are surrounded by `{}`

- Java is case-sensitive

# Variables and Types

- A variable declaration contains a type, name, and optional value

    ```java
    int age = 30;
    String name = "Alice";
    final double TAX_RATE = 0.07;
    var message = "Inferred local String";
    ```

- `final` prevents a variable from being reassigned. For an object, the reference cannot change but the object may still be mutable

- `var` allows local type inference. Java remains statically typed, and `var` cannot be used for fields, parameters, or a declaration without an initializer

- Eight primitive types:

    `byte`: 8-bit signed integer

    `short`: 16-bit signed integer

    `int`: 32-bit signed integer

    `long`: 64-bit signed integer, often written `10L`

    `float`: 32-bit floating point, often written `1.5F`

    `double`: 64-bit floating point

    `char`: 16-bit UTF-16 code unit

    `boolean`: `true` or `false`

- Reference types include objects, arrays, interfaces, enums, records, and `String`

- A reference may be `null`. Calling an instance member through `null` throws `NullPointerException`

- Wrapper classes provide object forms of primitives: `Integer`, `Long`, `Double`, `Boolean`, `Character`, and others

- Autoboxing converts between primitives and wrappers automatically, but a null wrapper cannot be unboxed

- Widening conversions such as `int` to `long` are automatic. Narrowing conversions require an explicit cast and may lose data

    ```java
    double price = 19.99;
    int whole = (int) price; // 19
    ```

- Use `BigDecimal` for exact decimal money calculations instead of binary floating-point values

# Operators and Control Flow

- Arithmetic: `+`, `-`, `*`, `/`, `%`

- Comparison: `==`, `!=`, `<`, `<=`, `>`, `>=`

- Boolean: `&&`, `||`, `!`

- Assignment: `=`, `+=`, `-=`, `*=`, `/=`

- Increment/decrement: `++`, `--`

- `&&` and `||` short-circuit, meaning the right expression is skipped when the result is already known

    ```java
    if (age >= 18) {
        System.out.println("Adult");
    } else {
        System.out.println("Minor");
    }
    ```

    ```java
    String result = score >= 60 ? "pass" : "fail";
    ```

    ```java
    switch (status) {
        case "OPEN" -> System.out.println("Working");
        case "CLOSED" -> System.out.println("Finished");
        default -> System.out.println("Unknown");
    }
    ```

- Loops:

    ```java
    for (int i = 0; i < 5; i++) {
        System.out.println(i);
    }

    for (String item : items) {
        System.out.println(item);
    }

    while (condition) {
        doWork();
    }
    ```

- `break` exits a loop or classic switch. `continue` skips to the next loop iteration

# Methods

- A method declares access, optional modifiers, return type, name, parameters, and body

    ```java
    public static int add(int a, int b) {
        return a + b;
    }
    ```

- Java passes every argument by value. For objects, the copied value is the reference, so a method can mutate the referenced object but cannot replace the caller's variable

- Method overloading means multiple methods share a name but have different parameter lists

- A method signature uses its name and parameter types, not its return type

- Varargs collect remaining arguments into an array and must be the final parameter

    ```java
    static int sum(int... numbers) {
        int total = 0;
        for (int number : numbers) total += number;
        return total;
    }
    ```

# Classes and Objects

    ```java
    public class BankAccount {
        private final String id;
        private double balance;

        public BankAccount(String id, double openingBalance) {
            this.id = id;
            this.balance = openingBalance;
        }

        public void deposit(double amount) {
            if (amount <= 0) {
                throw new IllegalArgumentException("Amount must be positive");
            }
            balance += amount;
        }

        public double getBalance() {
            return balance;
        }
    }
    ```

- A class defines state through fields and behavior through methods

- An object is an instance created with `new`

- A constructor initializes an object and has the same name as the class with no return type

- `this` refers to the current instance. `this(...)` calls another constructor in the same class

- `static` members belong to the class and are shared across instances

- Access modifiers:

    `public`: Accessible everywhere

    `protected`: Same package and subclasses

    No modifier: Package-private

    `private`: Only inside the declaring top-level class

- Encapsulation keeps fields private and exposes behavior which protects class rules

# Object-Oriented Principles

- Encapsulation: Keep an object's internal state controlled by its methods

- Abstraction: Expose the behavior a caller needs while hiding implementation details

- Inheritance: Create a specialized class from another class using `extends`

- Polymorphism: Use a parent/interface reference while runtime behavior comes from the concrete object

    ```java
    interface Shape {
        double area();
    }

    final class Circle implements Shape {
        private final double radius;

        Circle(double radius) {
            this.radius = radius;
        }

        @Override
        public double area() {
            return Math.PI * radius * radius;
        }
    }
    ```

- Java classes use single inheritance but may implement multiple interfaces

- Prefer composition when one object uses another without being a true subtype

- `abstract` classes may contain state, constructors, implemented methods, and abstract methods

- Interfaces define a contract and may contain abstract, default, static, and private helper methods

- `@Override` asks the compiler to verify that a method actually overrides a parent declaration

- `final class` cannot be extended. A `final` method cannot be overridden

# Object Methods and Equality

- Every class ultimately inherits from `Object`

- Important methods include `toString`, `equals`, `hashCode`, and `getClass`

- `==` compares primitive values but compares object identity for references

- `.equals()` compares logical equality when the class implements it

- Equal objects must have equal hash codes. Override `equals` and `hashCode` together

    ```java
    @Override
    public boolean equals(Object other) {
        if (this == other) return true;
        if (!(other instanceof User user)) return false;
        return id == user.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
    ```

- Hash-based collections rely on stable equality and hash codes. Avoid changing fields used in a key's hash while it is stored in a map or set

# Strings and Arrays

- `String` is immutable. Operations return new strings

    ```java
    String full = first + " " + last;
    boolean sameText = full.equals("Alice Jones");
    ```

- Use `StringBuilder` for repeated concatenation in a loop

- Useful String methods include `length`, `isBlank`, `substring`, `contains`, `startsWith`, `replace`, `split`, `trim`, `strip`, and `formatted`

- Arrays have fixed length and one element type

    ```java
    int[] numbers = {1, 2, 3};
    int first = numbers[0];
    int size = numbers.length;
    ```

- Invalid indexes throw `ArrayIndexOutOfBoundsException`

- `Arrays` provides helpers such as `sort`, `binarySearch`, `copyOf`, `equals`, and `toString`

# Collections

- `List<E>`: Ordered collection allowing duplicates

- `Set<E>`: Unique elements

- `Map<K, V>`: Key-value associations

- `Queue<E>` / `Deque<E>`: Ordered processing and double-ended operations

    ```java
    List<String> names = new ArrayList<>();
    names.add("Alice");

    Set<String> tags = new HashSet<>();
    tags.add("java");

    Map<Long, User> users = new HashMap<>();
    users.put(1L, new User(1L, "Alice"));
    User user = users.get(1L);
    ```

- Common implementations:

    `ArrayList`: General-purpose resizable list with fast indexed access

    `LinkedList`: Linked list implementing List and Deque; rarely faster for normal application lists

    `HashSet` / `HashMap`: Average constant-time lookup without sorted order

    `LinkedHashSet` / `LinkedHashMap`: Preserve insertion/access order

    `TreeSet` / `TreeMap`: Sorted tree-based collections

    `ArrayDeque`: Efficient stack or queue; usually preferred over `Stack`

- Immutable factories include `List.of`, `Set.of`, and `Map.of`

- `Collections` contains algorithms and wrappers such as `sort`, `reverse`, `shuffle`, `unmodifiableList`, and `synchronizedList`

# Generics

- Generics provide compile-time type safety without manually casting values

    ```java
    public class Box<T> {
        private T value;

        public T get() {
            return value;
        }

        public void set(T value) {
            this.value = value;
        }
    }
    ```

- Generic methods declare parameters before the return type: `static <T> T first(List<T> values)`

- Bounds limit accepted types: `<T extends Comparable<T>>`

- Wildcard rules:

    `List<?>`: List of one unknown type; values can be safely read as Object

    `List<? extends Number>`: Produces Numbers but does not safely accept a Number

    `List<? super Integer>`: Accepts Integers but reads only as Object

- PECS means Producer Extends, Consumer Super

- Generics use type erasure. Most generic type information is unavailable at runtime, and primitive types cannot be generic arguments

# Exceptions and Resource Management

- Exceptions interrupt normal execution and travel up the call stack until caught

- Checked exceptions must be caught or declared. Unchecked exceptions extend `RuntimeException`

    ```java
    try {
        processFile(path);
    } catch (IOException exception) {
        logger.error("Could not process file", exception);
    } finally {
        cleanup();
    }
    ```

- `throw` creates an exception outcome. `throws` declares possible checked exceptions

- Catch specific exceptions. Do not ignore an exception or use broad `catch (Exception)` without a deliberate boundary

- Try-with-resources closes every `AutoCloseable` resource in reverse order

    ```java
    try (BufferedReader reader = Files.newBufferedReader(path)) {
        return reader.readLine();
    }
    ```

- Preserve the original exception as a cause when translating it: `throw new DataException("Read failed", exception)`

# Functional Java and Streams

- A functional interface has one abstract method. Examples include `Predicate`, `Function`, `Consumer`, `Supplier`, and `Runnable`

    ```java
    Predicate<String> isLong = value -> value.length() > 10;
    Function<String, Integer> length = String::length;
    ```

- A Stream processes a sequence through a pipeline. It does not store data

    ```java
    List<String> activeNames = users.stream()
        .filter(User::isActive)
        .map(User::getName)
        .sorted()
        .toList();
    ```

- Intermediate operations such as `filter`, `map`, `flatMap`, `distinct`, and `sorted` are lazy

- Terminal operations include `toList`, `collect`, `reduce`, `count`, `forEach`, `findFirst`, `anyMatch`, and `allMatch`

- Streams are single-use. Avoid side effects inside stream operations

- Parallel streams are not automatically faster. They require enough CPU-bound work, safe operations, and careful measurement

- `Optional<T>` represents a result which may be absent. It is useful as a return type, not normally as an entity field or method parameter

# Records, Enums, and Sealed Types

- An enum defines a fixed set of named instances and may contain fields and methods

    ```java
    enum Status {
        PENDING, ACTIVE, CLOSED
    }
    ```

- A record is a concise data carrier with generated accessors, constructor, equality, hash code, and string representation

    ```java
    public record UserDto(long id, String name) {}
    ```

- Record components are final references, but referenced objects may still be mutable

- A sealed hierarchy restricts which types may extend or implement it

    ```java
    sealed interface Result permits Success, Failure {}
    record Success(String value) implements Result {}
    record Failure(String message) implements Result {}
    ```

- Pattern matching can safely combine a type check with a bound variable and can make exhaustive switches over sealed hierarchies possible

# Dates and Time

- Use the `java.time` API

- `Instant`: A moment on the UTC timeline

- `LocalDate`: Date without time or zone

- `LocalTime`: Time without date or zone

- `LocalDateTime`: Date and time without zone

- `ZonedDateTime`: Date and time in a named time zone

- `Duration`: Time-based amount

- `Period`: Date-based amount

- `DateTimeFormatter`: Parsing and formatting

    ```java
    Instant now = Instant.now();
    LocalDate due = LocalDate.parse("2026-08-10");
    ZonedDateTime local = now.atZone(ZoneId.of("America/New_York"));
    ```

- Store machine timestamps as Instant/UTC and apply a user's time zone when presenting them

# Files and I/O

- `Path` represents a filesystem path. `Files` performs common operations

    ```java
    Path path = Path.of("data", "users.txt");
    String text = Files.readString(path);
    Files.writeString(path, "Alice\n");
    ```

- Byte streams (`InputStream`, `OutputStream`) handle binary data. Character readers/writers handle text

- Specify a character encoding such as UTF-8 when an API does not already default appropriately

- Buffered I/O reduces expensive low-level operations

- Serialization formats such as JSON should use maintained libraries and explicit data models. Avoid native Java object deserialization for untrusted data

# Concurrency

- A thread is an independently scheduled execution path sharing process memory

- Race conditions occur when correctness depends on unpredictable timing

- Prefer high-level concurrency tools over manually creating and coordinating threads

    ```java
    try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
        Future<String> result = executor.submit(() -> fetchData());
        System.out.println(result.get());
    }
    ```

- `synchronized` protects a critical section with an intrinsic lock

- `Lock`, atomic classes, concurrent collections, semaphores, latches, and executors solve different coordination problems

- `volatile` provides visibility and ordering for a field but does not make compound actions such as increment atomic

- `CompletableFuture` composes asynchronous stages

- Virtual threads make blocking I/O concurrency cheaper. They do not make CPU work faster and do not remove the need to protect shared state or limit scarce downstream resources

- Favor immutable values, message passing, bounded concurrency, and thread-safe data structures

# Packages, Modules, and Build Tools

- A package organizes types and provides an access boundary

    ```java
    package com.example.users;

    import java.util.List;
    ```

- Package names usually use a reversed domain and lowercase words

- The module system uses `module-info.java` to declare required modules, exported packages, services, and reflection access

- Maven and Gradle manage dependencies, compilation, tests, plugins, and packaged artifacts

- Standard Maven structure:

    ```text
    src/main/java
    src/main/resources
    src/test/java
    src/test/resources
    pom.xml
    ```

- Pin dependency versions through the build tool and use its wrapper (`mvnw` or `gradlew`) for repeatable builds

# Testing and Good Practices

- JUnit is the standard unit-testing framework. Mockito and similar tools create test doubles when a real dependency is unsuitable

    ```java
    @Test
    void depositAddsToBalance() {
        BankAccount account = new BankAccount("1", 10.0);
        account.deposit(5.0);
        assertEquals(15.0, account.getBalance());
    }
    ```

- Unit tests should be fast and isolated. Integration tests verify databases, HTTP, serialization, and framework wiring

- Program to interfaces at important boundaries and inject dependencies instead of creating them deep inside business logic

- Prefer small cohesive classes, immutable data, explicit validation, and meaningful domain names

- Use `equals` for object value comparison, try-with-resources for closeable resources, and parameterized APIs for SQL

- Log through a logging framework instead of `System.out` in production applications

- Measure before tuning garbage collection, streams, collections, or concurrency

- Useful commands depend on the build tool:

    ```bash
    ./mvnw test
    ./mvnw package
    ./gradlew test
    ./gradlew build
    ```
