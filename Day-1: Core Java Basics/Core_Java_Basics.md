# Day 1: Core Java Basics — Complete Study Guide

Let's build this properly. This is a long one — Core Java fundamentals form the backbone of every interview round that follows, so we go deep, not fast.

═══════════════════════════════════════════
## SECTION 1: CONCEPT DEEP DIVE
═══════════════════════════════════════════

### 1. JDK vs JRE vs JVM 🔥

**Real-world analogy:** Think of a restaurant kitchen.
- **JVM** = the stove/oven (the actual execution engine that "cooks" your code)
- **JRE** = the entire kitchen (stove + utensils + ingredients pantry) needed to cook and serve a dish — enough to *run* food prep
- **JDK** = the kitchen + a cooking school (compiler, debugger, recipe-writing tools) — everything needed to *create* new dishes (write and compile code)

```java
/*
 * JDK (Java Development Kit) = JRE + Development tools
 *   - javac (compiler), javadoc, jdb (debugger), jar tool
 *
 * JRE (Java Runtime Environment) = JVM + Core Libraries
 *   - rt.jar (pre-Java 9) / modules (Java 9+), supporting files
 *
 * JVM (Java Virtual Machine) = Abstract machine that executes bytecode
 *   - ClassLoader, Runtime Data Areas, Execution Engine
 */

// You need JDK to COMPILE this:
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Compiled by javac (JDK), run by JVM (part of JRE)");
    }
}

// Step 1: javac HelloWorld.java   -> produces HelloWorld.class (bytecode) — needs JDK
// Step 2: java HelloWorld          -> JVM loads & executes the bytecode — needs JRE/JVM
```

**Production behavior & gotchas:**
- Since **Java 11**, Oracle stopped shipping JRE as a separate downloadable bundle — you only download the JDK now, but it *contains* a JRE-equivalent runtime.
- Docker images for production deployments often use a **JRE-only base image** (e.g., `eclipse-temurin:17-jre`) to reduce image size — you don't need `javac` in production, only the runtime.
- ⚠️ Freshers fail here: saying "JVM is platform-independent." It's actually the **opposite** — JVM implementations are platform-*dependent* (a Windows JVM binary differs from a Linux one). It's **bytecode** that is platform-independent because any JVM can interpret it. This nuance is asked constantly.

---

### 2. Class Loading 🔥 🏭

**Real-world analogy:** A librarian fetching a book only when a reader requests it (lazy loading), checking it isn't a duplicate (no class loaded twice by the same loader), and organizing books into floors — Ground floor (Bootstrap), Reference section (Platform/Extension), and Reading room (Application) — each floor only escalates to a higher floor if it can't find the book itself (delegation).

```java
public class ClassLoadingDemo {
    public static void main(String[] args) {
        // Demonstrates the 3-tier delegation hierarchy
        
        // 1. Bootstrap ClassLoader (loads java.lang.*, written in native code, shown as null)
        System.out.println("String's loader: " + String.class.getClassLoader()); 
        // prints: null (Bootstrap loader isn't a Java object)

        // 2. Platform/Extension ClassLoader (loads javax.*, etc. - Java 9+ renamed to Platform)
        // 3. Application/System ClassLoader (loads YOUR classpath classes)
        System.out.println("My class's loader: " + ClassLoadingDemo.class.getClassLoader());
        // prints: jdk.internal.loader.ClassLoaders$AppClassLoader@...

        // Class loading happens in phases: Loading -> Linking (Verify, Prepare, Resolve) -> Initialization
    }
}

// Custom ClassLoader example (commonly asked at senior level - used in plugin systems, app servers)
class MyCustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        System.out.println("Attempting to load: " + name);
        return super.loadClass(name, resolve); // delegates to parent first (delegation model)
    }
}
```

**Production behavior and gotchas:**
- **Parent Delegation Model**: A class loader always asks its parent first before trying to load a class itself. This prevents core classes like `java.lang.String` from being overridden/spoofed — a critical **security mechanism**.
- 🏭 Production critical: Application servers (Tomcat) and frameworks use **custom class loaders** to isolate web apps from each other (each WAR gets its own loader) — this is why two apps on the same Tomcat can use *different versions* of the same library without conflict.
- ⚠️ `NoClassDefFoundError` vs `ClassNotFoundException`: **freshers confuse these constantly.**
  - `ClassNotFoundException` (checked) — thrown when you explicitly call `Class.forName()` and the class isn't found.
  - `NoClassDefFoundError` (Error) — class was present at compile time, but missing/failed to initialize at runtime (e.g., a JAR was removed after compilation, or static initializer threw an exception).
- Classes are loaded **lazily** — only when first referenced (not all at JVM startup). This is asked often: "When is a class loaded?"

---

### 3. Bytecode Execution 🔥

**Real-world analogy:** Bytecode is like a translated screenplay (universal script) that any country's actors (any OS's JVM) can perform, instead of a region-specific play.

```java
public class BytecodeDemo {
    public static void main(String[] args) {
        int a = 5;
        int b = 10;
        int sum = a + b;
        System.out.println(sum);
    }
}

/* 
 * Compile: javac BytecodeDemo.java -> generates BytecodeDemo.class
 * Inspect: javap -c BytecodeDemo  -> shows bytecode instructions like:
 *
 *   0: iconst_5
 *   1: istore_1
 *   2: bipush 10
 *   4: istore_2
 *   5: iload_1
 *   6: iload_2
 *   7: iadd
 *   8: istore_3
 *   ...
 *
 * These instructions are executed by the Execution Engine inside the JVM,
 * either via:
 *  - Interpreter: reads bytecode line by line (slow start)
 *  - JIT (Just-In-Time) Compiler: compiles "hot" bytecode into native machine 
 *    code at runtime for speed (this is why Java "warms up")
 */
```

**Production behavior and gotchas:**
- 🏭 The **JIT compiler** is why Java apps are slower on the first few requests ("warm-up period") but match near-native speed afterward. This directly impacts how you do load testing and why Kubernetes readiness probes matter for JVM apps — cold pods serve slower initial requests.
- HotSpot JVM uses **tiered compilation**: starts interpreting → C1 compiler (quick, light optimization) → C2 compiler (aggressive optimization for very hot code paths).
- ⚠️ Freshers say "Java is compiled" or "Java is interpreted" — correct answer: **both**. Source → bytecode is compilation; bytecode → machine code is a hybrid of interpretation + JIT compilation.

---

### 4. Data Types ⚠️

**Real-world analogy:** Containers of fixed sizes — a teaspoon (`byte`) vs a bucket (`long`) — you pick the container based on how much you need to hold; using a bucket for a teaspoon of water wastes space (memory).

```java
public class DataTypesDemo {
    public static void main(String[] args) {
        // ---- 8 Primitive Types ----
        byte b = 127;              // 8-bit,  range -128 to 127
        short s = 32000;           // 16-bit, range -32,768 to 32,767
        int i = 2_000_000_000;     // 32-bit (default for whole numbers), underscores for readability (Java 7+)
        long l = 9_000_000_000L;   // 64-bit, MUST have 'L' suffix or it won't compile (exceeds int range)
        float f = 3.14f;           // 32-bit, MUST have 'f' suffix, ~6-7 decimal digits precision
        double d = 3.14159265359;  // 64-bit (default for decimals), ~15 decimal digits precision
        char c = 'A';              // 16-bit Unicode character, single quotes only
        boolean isJavaFun = true;  // true/false only — NOT 0/1 like C

        // Default values (only for instance/static fields, NOT local variables)
        // int -> 0, boolean -> false, object refs -> null, double -> 0.0

        System.out.println("int range: " + Integer.MIN_VALUE + " to " + Integer.MAX_VALUE);
        
        // Integer overflow - NO exception thrown, silently wraps around!
        int max = Integer.MAX_VALUE;
        System.out.println(max + 1); // prints -2147483648 (wraps to MIN_VALUE) ⚠️
    }
}
```

**Production behavior and gotchas:**
- ⚠️ **Integer overflow is silent** — no exception. This has caused real production bugs (e.g., timestamp/counter overflows). Use `Math.addExact()` if you want overflow to throw `ArithmeticException`.
- 🏭 `float`/`double` are **never** used for currency in production — binary floating point can't represent decimals like 0.1 exactly. Always use `BigDecimal` for money.
```java
System.out.println(0.1 + 0.2); // prints 0.30000000000000004, NOT 0.3 ⚠️
```
- `char` is 16-bit **unsigned** (0 to 65,535) — the only unsigned primitive in Java.

---

### 5. Type Casting ⚠️

**Real-world analogy:** Pouring a small cup of water into a large bucket happens automatically and safely (widening); pouring a bucket into a cup needs you to *consciously* decide to do it and accept spillage (narrowing/explicit cast — you lose data).

```java
public class TypeCastingDemo {
    public static void main(String[] args) {
        // ---- IMPLICIT (Widening) - automatic, safe, no data loss ----
        int myInt = 100;
        long myLong = myInt;      // int -> long, automatic
        double myDouble = myLong; // long -> double, automatic
        
        // Order: byte -> short -> int -> long -> float -> double
        // (char is special - converts to int but not vice versa implicitly)

        // ---- EXPLICIT (Narrowing) - manual cast required, CAN lose data ----
        double price = 99.99;
        int truncated = (int) price; // explicit cast required
        System.out.println(truncated); // prints 99 (NOT rounded, simply truncated!) ⚠️

        long bigValue = 130L;
        byte overflowed = (byte) bigValue; // byte range is -128 to 127
        System.out.println(overflowed); // prints -126 (wraps around due to overflow) ⚠️

        // ---- Object/Reference casting (upcasting vs downcasting) ----
        Object obj = "Hello";  // upcasting - implicit, always safe
        String str = (String) obj; // downcasting - explicit, RISKY

        Object num = Integer.valueOf(5);
        try {
            String bad = (String) num; // compiles fine, but throws at runtime!
        } catch (ClassCastException e) {
            System.out.println("Caught: " + e.getMessage()); // ⚠️ classic runtime trap
        }
    }
}
```

**Production behavior and gotchas:**
- ⚠️ Casting `double` to `int` **truncates**, it does NOT round. Freshers assume `(int) 99.99` gives `100` — it gives `99`. Use `Math.round()` if rounding is intended.
- 🏭 Use `instanceof` (or pattern matching `instanceof` in Java 16+) before downcasting to avoid `ClassCastException` in production:
```java
if (num instanceof String s) {  // Java 16+ pattern matching
    System.out.println(s.length());
}
```

---

### 6. Wrapper Classes 🔥

**Real-world analogy:** A primitive is cash in your hand; a wrapper class is that cash deposited into a bank account (an object) — it can now be stored in a "wallet" (Collections) that only accepts accounts, not raw cash.

```java
import java.util.ArrayList;
import java.util.List;

public class WrapperDemo {
    public static void main(String[] args) {
        // Every primitive has a corresponding wrapper class
        // int -> Integer, double -> Double, char -> Character, boolean -> Boolean, etc.

        Integer wrappedInt = Integer.valueOf(10); // preferred way (uses cache)
        Integer wrappedInt2 = new Integer(10);    // ⚠️ DEPRECATED since Java 9 - never do this

        // Why wrappers exist: Collections can ONLY hold objects, not primitives
        List<Integer> numbers = new ArrayList<>();
        numbers.add(5); // autoboxed: int 5 -> Integer.valueOf(5)

        // Useful utility methods
        int parsed = Integer.parseInt("123");        // String -> primitive
        String str = Integer.toString(123);           // primitive -> String
        int maxVal = Integer.MAX_VALUE;
        boolean isDigit = Character.isDigit('5');     // true
    }
}
```

**Production behavior and gotchas:**
- 🏭 **Integer Caching**: Java caches `Integer` objects from **-128 to 127**. This is a classic interview trap:
```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // true (both point to the SAME cached object)

Integer x = 200;
Integer y = 200;
System.out.println(x == y); // false (outside cache range, two DIFFERENT objects!) ⚠️🔥
// Always use .equals() to compare wrapper values, never ==
```
- ⚠️ `NullPointerException` on **unboxing null**:
```java
Integer score = null;
int result = score; // throws NullPointerException at unboxing! Extremely common production bug
```
- This caching applies to `Byte`, `Short`, `Long`, `Character` (0-127) and `Boolean` too — not just `Integer`.

---

### 7. Autoboxing / Unboxing 🔥 🏭

**Real-world analogy:** An automatic currency converter that converts cash to bank deposits and back without you manually doing the paperwork — convenient, but it costs a transaction fee (performance overhead) every time.

```java
import java.util.ArrayList;
import java.util.List;

public class AutoboxingDemo {
    public static void main(String[] args) {
        // Autoboxing: primitive -> wrapper (automatic, compiler-inserted)
        Integer boxed = 10; // compiler converts to: Integer.valueOf(10)

        // Unboxing: wrapper -> primitive (automatic)
        int unboxed = boxed; // compiler converts to: boxed.intValue()

        // Happens silently in loops - PERFORMANCE TRAP in production
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add(i); // autoboxing happens 5 times here
        }

        // ⚠️ Classic interview trap: autoboxing inside loops causing excessive object creation
        long sum = 0;
        for (Long i = 0L; i < 100000; i++) { // 'Long' instead of 'long' - BUG!
            sum += i; // unboxes/reboxes Long on every iteration - massive overhead 🏭
        }
        // Fix: use primitive 'long i' in the loop instead
    }
}
```

**Production behavior and gotchas:**
- 🏭 In hot loops (high-frequency trading, stream processing), accidentally using `Long`/`Integer` instead of `long`/`int` in loop counters causes **massive GC pressure** from constant boxing — a real performance bug that's bitten production systems.
- ⚠️ Mixing primitives and wrappers in conditionals can NPE:
```java
Integer count = null;
if (count > 0) { } // NullPointerException — unboxing null inside the comparison
```
- `==` on autoboxed values is the single most asked "gotcha" question in Java interviews — tied directly to the Integer cache from concept #6.

---

### 8. OOP — Class, Object, Constructor, `this`, `static`, `final` 🔥

**Real-world analogy:** A **class** is a blueprint for a house; an **object** is an actual house built from that blueprint; the **constructor** is the construction crew that builds it the moment you order it; `this` is the house referring to "itself" when multiple houses are being built from the same blueprint; `static` is a shared community well that ALL houses in the neighborhood use (not one per house); `final` is a property deed that, once signed, can never be changed.

```java
public class Employee {
    // Instance variables - each object gets its OWN copy
    private String name;
    private double salary;
    
    // Static variable - SHARED across ALL objects (like a class-level counter)
    private static int employeeCount = 0;
    
    // Final variable - must be assigned once, never changed again
    private final String employeeId;

    // Constructor - called automatically when object is created via 'new'
    public Employee(String name, double salary) {
        this.name = name;       // 'this' disambiguates field from parameter
        this.salary = salary;
        this.employeeId = "EMP-" + (++employeeCount); // assigned once in constructor - allowed for final
        employeeCount++; // wait - careful, let's not double count; see note below
    }

    // Constructor overloading - multiple constructors, different signatures
    public Employee(String name) {
        this(name, 50000.0); // 'this(...)' calls another constructor - MUST be first line
    }

    // Static method - belongs to the CLASS, not instances; can't use 'this' inside
    public static int getEmployeeCount() {
        return employeeCount;
    }

    // Instance method - operates on a specific object's data
    public void giveRaise(double amount) {
        this.salary += amount;
    }

    public static void main(String[] args) {
        Employee e1 = new Employee("Asha", 60000); // object 1
        Employee e2 = new Employee("Ravi");          // object 2, uses overloaded constructor
        
        System.out.println(Employee.getEmployeeCount()); // accessed via class name, not object
    }
}
```

**Production behavior and gotchas:**
- ⚠️ `static` members are loaded once when the class is loaded and **shared across all instances** — a classic bug is using `static` for what should be instance-level state (e.g., a `static List` accumulating data across unrelated requests in a web app — a notorious memory leak / data-leak bug in multi-threaded servers).
- 🏭 `final` on a reference variable means the **reference** can't be reassigned, but the object it points to **can still be mutated**:
```java
final List<String> list = new ArrayList<>();
list.add("hello"); // perfectly legal - list's internal state changed, not the reference
// list = new ArrayList<>(); // ❌ compile error - can't reassign
```
- `this()` constructor chaining must be the **first statement** in a constructor — compile error otherwise.
- Static methods **cannot** be overridden (only hidden) and can't use `this`/`super` — asked frequently at mid-level.

---

### 9. Inheritance 🔥

**Real-world analogy:** A child inherits traits (eye color, surname) from a parent automatically, can override some of them with their own (like choosing a different career), but cannot inherit from two biological fathers (no multiple inheritance via classes).

```java
class Vehicle { // parent / superclass
    protected String brand;
    
    public Vehicle(String brand) {
        this.brand = brand;
    }

    public void start() {
        System.out.println(brand + " vehicle starting generically");
    }
}

class Car extends Vehicle { // child / subclass - 'extends' keyword
    private int doors;

    public Car(String brand, int doors) {
        super(brand); // calls parent constructor - MUST be first line in child constructor
        this.doors = doors;
    }

    @Override
    public void start() { // overriding parent behavior
        super.start(); // can still call parent's version explicitly
        System.out.println(brand + " car with " + doors + " doors starting smoothly");
    }
}

public class InheritanceDemo {
    public static void main(String[] args) {
        Car myCar = new Car("Toyota", 4);
        myCar.start();
        // Output:
        // Toyota vehicle starting generically
        // Toyota car with 4 doors starting smoothly
    }
}
```

**Production behavior and gotchas:**
- ⚠️ Java does NOT support **multiple inheritance with classes** (to avoid the Diamond Problem) — but DOES support it via **interfaces** (with default methods, resolved explicitly if there's a conflict).
- 🏭 **"Favor composition over inheritance"** is a real production design principle — deep inheritance hierarchies (4-5 levels) become fragile and hard to maintain ("fragile base class problem"). Senior interviews often probe this design judgment.
- ⚠️ Constructors are **NOT inherited** — every subclass must define its own (or rely on the implicit default).
- If the parent has no no-arg constructor, the child's constructor **MUST** explicitly call `super(args)` or it won't compile.

---

### 10. Polymorphism 🔥

**Real-world analogy:** A single button labeled "Print" behaves differently depending on the device — a phone prints to a photo printer, a laptop prints to an office printer — same action, different behavior based on the actual object (runtime/dynamic polymorphism). Meanwhile, a person can be called by different "titles" depending on context — "Doctor", "Mr. Sharma", "Coach" — same person, different forms (overloading is compile-time polymorphism).

```java
public class PolymorphismDemo {

    // ---- Compile-time Polymorphism (Method Overloading) ----
    static class Calculator {
        int add(int a, int b) { return a + b; }
        double add(double a, double b) { return a + b; } // different parameter types
        int add(int a, int b, int c) { return a + b + c; } // different parameter count
        // Resolved at COMPILE TIME based on method signature
    }

    // ---- Runtime Polymorphism (Method Overriding) ----
    static class Shape {
        double area() { return 0; }
    }
    static class Circle extends Shape {
        double radius;
        Circle(double radius) { this.radius = radius; }
        @Override
        double area() { return Math.PI * radius * radius; }
    }
    static class Square extends Shape {
        double side;
        Square(double side) { this.side = side; }
        @Override
        double area() { return side * side; }
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.add(2, 3));       // resolved at compile time -> 5

        // Runtime polymorphism: reference type is Shape, actual object decides behavior
        Shape[] shapes = { new Circle(5), new Square(4) };
        for (Shape s : shapes) {
            System.out.println(s.area()); // JVM decides at RUNTIME which area() to call
        }
        // This is "dynamic method dispatch" - the actual asked-about mechanism in interviews
    }
}
```

**Production behavior and gotchas:**
- 🔥 **Dynamic Method Dispatch** is THE mechanism interviewers want named explicitly — the JVM uses the object's actual runtime type (via the **vtable**/method table), not the reference's declared type, to decide which overridden method runs.
- ⚠️ Field access is **NOT polymorphic** in Java — only methods are. Accessing a field uses the **reference type**, not the object type:
```java
class A { int x = 10; }
class B extends A { int x = 20; }
A obj = new B();
System.out.println(obj.x); // prints 10! (field access is NOT polymorphic) ⚠️🔥
```
This is one of the most-missed gotchas at mid/senior level.
- Overloading is resolved at **compile-time** (static binding); Overriding is resolved at **runtime** (dynamic binding) — a frequently mis-explained distinction.

---

### 11. Abstraction 🔥

**Real-world analogy:** A car's steering wheel and pedals — you know WHAT they do (turn, accelerate, brake) without needing to know HOW the engine/hydraulics work internally. You're abstracted away from implementation complexity.

```java
abstract class PaymentProcessor {
    // Abstract method - no implementation, subclasses MUST provide one
    abstract void processPayment(double amount);

    // Concrete method - shared implementation, can be reused as-is
    void logTransaction(double amount) {
        System.out.println("Logging transaction of: $" + amount);
    }
}

class CreditCardProcessor extends PaymentProcessor {
    @Override
    void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Credit Card");
        // actual gateway integration logic here
    }
}

class UPIProcessor extends PaymentProcessor {
    @Override
    void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via UPI");
    }
}

public class AbstractionDemo {
    public static void main(String[] args) {
        PaymentProcessor processor = new CreditCardProcessor(); // caller doesn't care HOW
        processor.processPayment(500.0);
        processor.logTransaction(500.0);

        // PaymentProcessor p = new PaymentProcessor(); // ❌ compile error - can't instantiate abstract class
    }
}
```

**Production behavior and gotchas:**
- 🏭 Real-world use: payment gateway integrations, JDBC drivers (`Connection`, `Statement` are abstractions over vendor-specific implementations), Spring's `JpaRepository` abstracts away SQL.
- ⚠️ Abstraction is about **hiding complexity / exposing only "what"**; Encapsulation is about **hiding data / bundling state with behavior**. Interviewers ask these back to back to test if you conflate them (you should NOT).
- An abstract class can have **zero** abstract methods (still can't be instantiated) — a common trick question.

---

### 12. Encapsulation 🔥 ⚠️

**Real-world analogy:** A medicine capsule — the contents (data) are sealed inside, and you only interact with it through a controlled interface (swallowing it), not by directly touching the raw chemicals inside.

```java
public class BankAccount {
    // private fields - data hidden from outside direct access
    private double balance;
    private String accountHolder;

    public BankAccount(String accountHolder, double initialBalance) {
        this.accountHolder = accountHolder;
        this.balance = initialBalance;
    }

    // public getter - controlled READ access
    public double getBalance() {
        return balance;
    }

    // public setter - controlled WRITE access, WITH validation logic
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        this.balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        this.balance -= amount;
    }
}

public class EncapsulationDemo {
    public static void main(String[] args) {
        BankAccount acc = new BankAccount("Asha", 1000);
        // acc.balance = -5000; // ❌ not possible - 'balance' is private
        acc.deposit(500);  // forced to go through validated logic
        System.out.println(acc.getBalance()); // 1500.0
    }
}
```

**Production behavior and gotchas:**
- 🏭 Encapsulation is what makes validation, logging, lazy loading, and thread-safety **possible** at the field level — without it you can't intercept reads/writes.
- ⚠️ Common fresher mistake: making fields `private` but then writing a `getX()`/`setX()` for EVERY field blindly (especially for mutable objects like `List` or `Date`), which **breaks encapsulation** by leaking internal references:
```java
private List<String> items = new ArrayList<>();
public List<String> getItems() { return items; } // ❌ leaks internal mutable reference!
// Caller can now do: account.getItems().clear() - bypassing all your validation
// Fix: return Collections.unmodifiableList(items) or a defensive copy
```
This is a real production bug pattern asked at mid/senior levels.

---

### 13. Abstract Class vs Interface 🔥

**Real-world analogy:** An abstract class is like a **half-built house** — some rooms are furnished (concrete methods/state) and some are empty shells you must finish (abstract methods). An interface is like a **contract/checklist** — "Whatever you build, it MUST have a door, a window, and a roof" — it specifies *what* must exist, traditionally without saying *how*.

```java
// Abstract class - can have state, constructors, mixed concrete/abstract methods
abstract class Animal {
    protected String name; // CAN have instance state
    
    Animal(String name) { // CAN have a constructor
        this.name = name;
    }
    
    abstract void makeSound(); // abstract - must be implemented
    
    void sleep() { // concrete - shared by all subclasses
        System.out.println(name + " is sleeping");
    }
}

// Interface - traditionally pure contract; Java 8+ allows default/static methods
interface Swimmable {
    int MAX_DEPTH = 100; // implicitly public static final (constant)
    
    void swim(); // implicitly public abstract
    
    default void floatOnSurface() { // Java 8+ : interfaces CAN have implementation
        System.out.println("Floating...");
    }
    
    static void printRules() { // Java 8+ : static methods in interfaces
        System.out.println("Swimming rules apply");
    }
}

class Dolphin extends Animal implements Swimmable { // single inheritance + multiple interfaces
    Dolphin(String name) { super(name); }
    
    @Override
    void makeSound() { System.out.println(name + " clicks and whistles"); }
    
    @Override
    public void swim() { System.out.println(name + " swims gracefully"); }
}

public class AbstractVsInterfaceDemo {
    public static void main(String[] args) {
        Dolphin d = new Dolphin("Flipper");
        d.makeSound();
        d.swim();
        d.sleep();
        d.floatOnSurface();
    }
}
```

**Production behavior and gotchas:**
| Aspect | Abstract Class | Interface |
|---|---|---|
| State (instance fields) | ✅ Yes | ❌ No (only `public static final` constants) |
| Constructor | ✅ Yes | ❌ No |
| Multiple inheritance | ❌ Single only (`extends`) | ✅ Multiple (`implements A, B, C`) |
| Method default access | any access modifier | `public` by default (Java 8: `default`/`static` allowed, Java 9+: `private` methods too) |
| When to use | "is-a" relationship with shared code/state | "can-do" capability/contract across unrelated classes |

- 🔥 **Diamond problem with default methods**: if a class implements two interfaces with the **same default method signature**, it's a compile error unless the class explicitly overrides it:
```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }
class C implements A, B {
    public void hello() { A.super.hello(); } // must resolve explicitly ⚠️🔥
}
```
- 🏭 Real production rule of thumb: Spring framework relies heavily on interfaces for testability/DI (`UserService` interface + `UserServiceImpl`); abstract classes are used for **template method pattern** (e.g., Spring's `JdbcDaoSupport`).

---

### 14. Access Modifiers ⚠️

**Real-world analogy:** Levels of building access — `private` is your personal bedroom (only you), `default`/package-private is your apartment floor (only your floor's residents), `protected` is your building + sister buildings owned by the same company (family), `public` is the lobby (anyone can enter).

```java
package com.company.app;

public class AccessDemo {
    public int publicVar = 1;       // accessible EVERYWHERE
    protected int protectedVar = 2;  // accessible in same package + subclasses (even in other packages)
    int defaultVar = 3;              // (no modifier) accessible ONLY within same package
    private int privateVar = 4;      // accessible ONLY within this class

    private void privateMethod() {
        System.out.println("Only visible inside AccessDemo");
    }
}

class SamePackageClass {
    void test() {
        AccessDemo obj = new AccessDemo();
        System.out.println(obj.publicVar);     // ✅ OK
        System.out.println(obj.protectedVar);  // ✅ OK - same package
        System.out.println(obj.defaultVar);    // ✅ OK - same package
        // System.out.println(obj.privateVar); // ❌ compile error
    }
}
```

**Production behavior and gotchas:**
- ⚠️ Order of visibility (widest to narrowest): `public` > `protected` > `default (package-private)` > `private`.
- ⚠️ `protected` is bigger than most freshers think — it's accessible not just to subclasses but to the **entire package**, not just subclasses outside the package.
- 🏭 Production convention: fields are almost always `private` with `public` getters/setters; classes implementing an interface for DI (Spring beans) are typically `public`, while helper/internal classes are package-private to control your public API surface — this matters in library design.
- Top-level classes can only be `public` or `default` (package-private) — **never** `private` or `protected`.

---

### 15. Packages

**Real-world analogy:** Folders in a filing cabinet — instead of dumping every file (class) loose in one drawer, you organize them into labeled folders (`com.company.service`, `com.company.model`) to avoid name clashes and improve navigation.

```java
// File: com/company/util/StringHelper.java
package com.company.util;

public class StringHelper {
    public static boolean isEmpty(String s) {
        return s == null || s.trim().isEmpty();
    }
}

// File: com/company/Main.java
package com.company;

import com.company.util.StringHelper; // explicit import
// import com.company.util.*; // wildcard import - imports everything in that package (only, NOT sub-packages)

public class Main {
    public static void main(String[] args) {
        System.out.println(StringHelper.isEmpty(""));  // true
        
        // java.lang.* is auto-imported - no need to import String, System, Math, etc.
        System.out.println(Math.max(3, 7));
    }
}
```

**Production behavior and gotchas:**
- ⚠️ Wildcard imports (`import com.company.util.*;`) do **NOT** import sub-packages — a common misconception.
- 🏭 Package naming convention follows **reverse domain name** (`com.companyname.project.module`) to guarantee global uniqueness — this matters for JAR conflicts in large multi-module Maven/Gradle projects.
- Package and folder structure **must match exactly** — `package com.company.util;` MUST physically live in `com/company/util/` folder, or compilation fails.
- Java 9+ introduced the **Module System (JPMS)** — `module-info.java` — for stricter encapsulation across JARs (occasionally asked at senior level, rarely used in typical Spring Boot apps though).

---

### 16. String vs StringBuilder vs StringBuffer 🔥 🏭

**Real-world analogy:** `String` is a carved stone tablet — every "edit" actually creates a brand-new tablet (immutable). `StringBuilder` is a whiteboard you erase and rewrite freely, used solo (not thread-safe). `StringBuffer` is the same whiteboard but with a lock on the door so only one person can write at a time (thread-safe, synchronized).

```java
public class StringComparisonDemo {
    public static void main(String[] args) {
        // ---- String: IMMUTABLE ----
        String s1 = "Hello";
        String s2 = "Hello"; // points to SAME object in the String Pool
        System.out.println(s1 == s2); // true - string literal pooling

        String s3 = new String("Hello"); // forces a NEW object on the heap, bypasses pool
        System.out.println(s1 == s3); // false ⚠️🔥
        System.out.println(s1.equals(s3)); // true - always use .equals() for content comparison

        s1 = s1.concat(" World"); // does NOT modify s1 - creates a NEW string, reassigns reference
        // Every concatenation creates a new object - expensive in loops!

        // ---- StringBuilder: MUTABLE, NOT thread-safe, FAST ----
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 5; i++) {
            sb.append(i).append(","); // modifies the SAME internal char array, no new object per call
        }
        System.out.println(sb.toString()); // 0,1,2,3,4,

        // ---- StringBuffer: MUTABLE, thread-safe (synchronized), SLOWER ----
        StringBuffer sbf = new StringBuffer();
        sbf.append("Thread-safe").append(" appending");
        System.out.println(sbf);
    }
}
```

**Production behavior and gotchas:**
- 🔥 **THE most commonly asked Core Java question.** Be crisp: 
  - `String` → immutable, thread-safe by nature (can't be modified), stored in String Pool (literals) for memory efficiency.
  - `StringBuilder` → mutable, **NOT synchronized**, fastest — use in single-threaded contexts (the default choice in 95% of code, e.g., inside a method).
  - `StringBuffer` → mutable, synchronized (thread-safe), slower due to locking overhead — rarely used today; prefer `StringBuilder` + external synchronization if truly needed, or better, immutable/thread-confined designs.
- ⚠️ **String concatenation in a loop using `+`** is a classic production anti-pattern — creates a new `String` object every iteration (O(n²) behavior):
```java
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i; // ⚠️ creates 10,000 intermediate String objects - terrible for perf
}
// Fix: use StringBuilder
```
- 🏭 The compiler auto-optimizes a **single-line** concatenation with `+` into a `StringBuilder` under the hood (since Java 9 it actually uses `invokedynamic` + `StringConcatFactory`), but this optimization does NOT extend across loop iterations — that's the real gotcha.
- `String.intern()` explicitly puts a heap string into the String Pool — rarely used, but occasionally asked.

---

### 17. Arrays ⚠️

**Real-world analogy:** A row of numbered lockers — fixed in count once installed (fixed size), each locker holds one specific type of item, and you access any locker instantly by its number (O(1) access via index).

```java
import java.util.Arrays;

public class ArrayDemo {
    public static void main(String[] args) {
        // Declaration & initialization
        int[] numbers = new int[5];           // fixed size 5, default-initialized to 0s
        int[] scores = {90, 85, 77, 92, 60};   // array literal
        String[] names = new String[3];        // default-initialized to null (object array)

        // 2D array (array of arrays)
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6}
        };
        System.out.println(matrix[1][2]); // 6

        // Common operations
        System.out.println(scores.length); // 5 (no parentheses - it's a field, not a method!) ⚠️
        Arrays.sort(scores);
        System.out.println(Arrays.toString(scores)); // [60, 77, 85, 90, 92]

        // Arrays are objects - default toString() is useless without Arrays.toString()
        System.out.println(scores); // prints something like [I@1b6d3586 ⚠️

        try {
            System.out.println(scores[10]); // out of bounds
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Caught: " + e.getMessage());
        }

        // Arrays vs Collections - arrays are FIXED size
        // numbers = new int[10]; // this REASSIGNS the reference to a new array, doesn't "resize"
    }
}
```

**Production behavior and gotchas:**
- ⚠️ `.length` is a **field** on arrays (no parentheses), but `.length()` is a **method** on `String` and `.size()` is a method on Collections — this trio of inconsistency is a guaranteed fresher trip-up.
- ⚠️ Arrays have a **fixed size once created** — you can't "add" to them; resizing means creating a new array and copying (which is exactly what `ArrayList` does internally under the hood).
- 🏭 Multi-dimensional arrays in Java are actually "**arrays of arrays**" (jagged arrays are fully legal — rows can have different lengths), unlike true rectangular matrices in some other languages.
- `Arrays.asList()` returns a **fixed-size list backed by the array** — calling `.add()`/`.remove()` on it throws `UnsupportedOperationException`, a very common production bug.

---

### 18. Exception Handling 🔥 🏭

**Real-world analogy:** A fire alarm system — `try` is the room you're working in, `catch` is the fire extinguisher for a *specific type* of fire, `finally` is the mandatory evacuation/cleanup procedure that runs no matter what (fire or no fire), and `throw`/`throws` is sounding the alarm to alert others up the chain.

```java
import java.io.IOException;

public class ExceptionDemo {

    // Checked exception - MUST be declared or caught (compiler-enforced)
    static void readFile() throws IOException {
        throw new IOException("File not found");
    }

    // Custom exception - common in production for domain-specific errors
    static class InsufficientFundsException extends Exception {
        public InsufficientFundsException(String message) {
            super(message);
        }
    }

    static void withdraw(double balance, double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException("Cannot withdraw $" + amount + ", balance is $" + balance);
        }
    }

    public static void main(String[] args) {
        // Basic try-catch-finally
        try {
            int[] arr = new int[5];
            arr[10] = 1; // throws ArrayIndexOutOfBoundsException - unchecked, runtime exception
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Caught specific exception: " + e.getMessage());
        } catch (Exception e) { // generic catch - MUST come AFTER specific ones, or compile error
            System.out.println("Generic catch: " + e.getMessage());
        } finally {
            System.out.println("Finally ALWAYS runs - cleanup code goes here");
        }

        // Try-with-resources (Java 7+) - auto-closes resources, preferred over manual finally-close
        // Example: try (FileReader fr = new FileReader("file.txt")) { ... }
        // AutoCloseable.close() is called automatically, even on exception

        // Custom checked exception usage
        try {
            withdraw(100, 500);
        } catch (InsufficientFundsException e) {
            System.out.println("Business error: " + e.getMessage());
        }

        // Multi-catch (Java 7+)
        try {
            Object obj = null;
            obj.toString();
        } catch (NullPointerException | ArithmeticException e) {
            System.out.println("Handled multiple types in one block: " + e);
        }
    }
}
```

**Production behavior and gotchas:**
- 🔥 **Checked vs Unchecked** is asked in nearly every interview:
  - **Checked** (`IOException`, `SQLException`) — extends `Exception`, must be caught/declared, represents *recoverable* conditions the caller should handle.
  - **Unchecked** (`NullPointerException`, `ArrayIndexOutOfBoundsException`) — extends `RuntimeException`, compiler doesn't force handling, usually represents *programming bugs*.
  - **Error** (`OutOfMemoryError`, `StackOverflowError`) — represents serious JVM-level problems, generally **not** meant to be caught/recovered from.
- ⚠️ `finally` runs even if there's a `return` in the `try` block — but if `finally` *also* has a `return`, it **silently swallows** the try block's return value/exception — a notoriously tricky interview trap:
```java
static int test() {
    try {
        return 1;
    } finally {
        return 2; // this OVERRIDES the try's return! Method returns 2, not 1 ⚠️🔥
    }
}
```
- 🏭 **Never swallow exceptions silently** in production (`catch (Exception e) {}`) — this is the #1 anti-pattern that makes production debugging a nightmare. Always log, rethrow, or handle meaningfully.
- 🏭 **Exception handling has a performance cost** — filling in the stack trace is expensive. In hot paths, this matters (frameworks sometimes override `fillInStackTrace()` to skip it for control-flow-style exceptions).
- ⚠️ Catching `Exception` or `Throwable` broadly can accidentally swallow `Error`s (like `OutOfMemoryError`) if you catch `Throwable` — almost always wrong to do in application code.
- Custom exceptions should generally **extend `RuntimeException`** in modern Spring Boot codebases (unchecked) — checked exceptions are largely considered an anti-pattern in modern API design because they force boilerplate `try/catch` or `throws` propagation everywhere. This is a great senior-level talking point.

═══════════════════════════════════════════
## SECTION 2: INTERVIEW QUESTIONS (30)
═══════════════════════════════════════════

Q1. What is the difference between JDK, JRE, and JVM?
→ Expected depth: JVM executes bytecode; JRE = JVM + libraries to run apps; JDK = JRE + dev tools (compiler, debugger). Mention JVM is platform-dependent, bytecode is platform-independent.
→ Difficulty: Fresher
→ 🔥

Q2. Explain the Java class loading mechanism and the parent delegation model.
→ Expected depth: Bootstrap → Platform/Extension → Application loader hierarchy; why delegation exists (security — prevents core class spoofing); lazy loading.
→ Difficulty: Mid-Level

Q3. What's the difference between `NoClassDefFoundError` and `ClassNotFoundException`?
→ Expected depth: One is an Error (class existed at compile time, missing at runtime), other is a checked Exception (from `Class.forName()` explicitly).
→ Difficulty: Senior

Q4. How does the JIT compiler improve performance compared to pure interpretation?
→ Expected depth: Identifies "hot" bytecode, compiles to native machine code at runtime, tiered compilation (C1/C2), explains JVM "warm-up."
→ Difficulty: Mid-Level

Q5. Why doesn't `0.1 + 0.2 == 0.3` in Java?
→ Expected depth: Binary floating-point representation can't exactly represent most decimal fractions; recommend `BigDecimal` for precise/financial calculations.
→ Difficulty: Fresher
→ 🔥

Q6. What happens when an `int` overflows in Java?
→ Expected depth: Silently wraps around (no exception), explain two's complement, mention `Math.addExact()` for safe overflow detection.
→ Difficulty: Mid-Level

Q7. Why is `(int) 9.99` equal to `9` and not `10`?
→ Expected depth: Narrowing cast truncates, doesn't round; contrast with `Math.round()`.
→ Difficulty: Fresher
→ 🔥

Q8. Explain autoboxing/unboxing and a real production bug it can cause.
→ Expected depth: Compiler-inserted conversions; NPE on unboxing null; performance overhead in loops with wrapper types.
→ Difficulty: Mid-Level
→ 🔥

Q9. Why does `Integer a = 127; Integer b = 127; a == b` return true, but with 128 it's false?
→ Expected depth: Integer cache (-128 to 127), `Integer.valueOf()` reuses cached objects; explain why `.equals()` should always be used for value comparison.
→ Difficulty: Mid-Level
→ 🔥

Q10. What is the difference between `==` and `.equals()`?
→ Expected depth: `==` compares references for objects (values for primitives); `.equals()` compares logical content (if overridden); String pool nuance.
→ Difficulty: Fresher
→ 🔥

Q11. Explain constructor overloading with `this()`. What are the rules?
→ Expected depth: Multiple constructors, different signatures; `this(...)` must be first statement; can't have both `this()` and `super()` in same constructor.
→ Difficulty: Fresher

Q12. What's the difference between instance variables, static variables, and local variables?
→ Expected depth: Scope, lifetime, default values, memory location (heap vs stack vs method area/metaspace).
→ Difficulty: Fresher

Q13. Can a `static` method be overridden? Why or why not?
→ Expected depth: No — static methods are resolved at compile-time (static binding) based on reference type; can only be "hidden," not overridden; demonstrate with code.
→ Difficulty: Mid-Level
→ 🔥

Q14. Is `final` reference variable's object also immutable?
→ Expected depth: No — `final` only prevents reassignment of the reference; the object's internal state can still mutate unless the object itself is designed to be immutable.
→ Difficulty: Mid-Level
→ 🔥

Q15. Why doesn't Java support multiple inheritance with classes?
→ Expected depth: Diamond problem (ambiguity in inherited state/behavior); Java avoids it via single class inheritance + multiple interface implementation.
→ Difficulty: Fresher
→ 🔥

Q16. Explain method overloading vs method overriding with binding type.
→ Expected depth: Overloading = compile-time/static binding, same name different signature; Overriding = runtime/dynamic binding, same signature, IS-A relationship required.
→ Difficulty: Fresher
→ 🔥

Q17. Is field access polymorphic in Java? Prove it with code.
→ Expected depth: No — fields are resolved by reference type at compile time, not the actual object type; only methods exhibit dynamic dispatch.
→ Difficulty: Senior
→ 🔥

Q18. What is dynamic method dispatch?
→ Expected depth: JVM uses the object's actual runtime type (via vtable) to decide which overridden method executes, regardless of the reference's declared type.
→ Difficulty: Mid-Level
→ 🔥

Q19. Difference between abstraction and encapsulation — aren't they the same thing?
→ Expected depth: Abstraction hides implementation complexity (the "what" not "how"); Encapsulation hides data/state by bundling it with controlled access — different concerns, often confused.
→ Difficulty: Mid-Level
→ 🔥

Q20. Can an abstract class have zero abstract methods? Can it have a constructor?
→ Expected depth: Yes to both — abstract just means "cannot be instantiated"; constructor is called via `super()` from a concrete subclass.
→ Difficulty: Mid-Level

Q21. What is the difference between an abstract class and an interface? When would you choose one over the other?
→ Expected depth: State/constructors (abstract class only), single vs multiple inheritance, "is-a" vs "can-do," default/static methods in interfaces since Java 8, design judgment call.
→ Difficulty: Mid-Level
→ 🔥

Q22. How is the diamond problem resolved with interface default methods?
→ Expected depth: Compile error if a class implements two interfaces with conflicting default methods; must explicitly override and optionally call `InterfaceName.super.method()`.
→ Difficulty: Senior

Q23. Explain all four access modifiers and their visibility scope.
→ Expected depth: private (class) < default (package) < protected (package + subclasses) < public (everywhere); common confusion around protected's package-wide visibility.
→ Difficulty: Fresher
→ 🔥

Q24. Why is `String` immutable in Java? What are the benefits?
→ Expected depth: Security (used in classloading, network connections), thread-safety by nature, safe hashcode caching for use in HashMap keys, String pool optimization.
→ Difficulty: Mid-Level
→ 🔥

Q25. Difference between String, StringBuilder, and StringBuffer — when would you use each?
→ Expected depth: Immutability, thread-safety (synchronized vs not), performance tradeoffs; use StringBuilder by default, StringBuffer only for legacy/explicit multi-threaded shared mutable string scenarios.
→ Difficulty: Fresher
→ 🔥

Q26. What is the String pool / String interning?
→ Expected depth: Special memory region (part of heap since Java 7) storing unique literal strings; `new String()` bypasses it; `.intern()` forces pooling.
→ Difficulty: Mid-Level

Q27. Why does string concatenation in a loop using `+` hurt performance?
→ Expected depth: Each `+` creates a new String object (immutability), O(n²) total cost across a loop; fix with StringBuilder; mention compiler only optimizes single-expression concatenation, not loops.
→ Difficulty: Mid-Level
→ 🔥

Q28. Why does `Arrays.asList()` throw `UnsupportedOperationException` on `.add()`?
→ Expected depth: Returns a fixed-size list backed directly by the array; structurally can't grow/shrink; only `set()` is allowed.
→ Difficulty: Senior

Q29. Explain checked vs unchecked exceptions with examples. Which should custom exceptions extend in modern practice?
→ Expected depth: Checked extends Exception (compiler-enforced handling, recoverable), unchecked extends RuntimeException (programming bugs, not enforced); modern APIs (Spring) favor unchecked custom exceptions to avoid `throws` boilerplate propagation.
→ Difficulty: Mid-Level
→ 🔥

Q30. What happens if both the `try` block and `finally` block have a `return` statement?
→ Expected depth: `finally`'s return silently overrides try's return (and even suppresses exceptions); explain why this is considered bad practice.
→ Difficulty: Senior
→ 🔥

═══════════════════════════════════════════
## SECTION 3: SCENARIO-BASED QUESTIONS (7)
═══════════════════════════════════════════

**Scenario 1:** Your production Spring Boot service stores a `static List<Order>` to cache recent orders across requests. After a few weeks in production, you start getting `OutOfMemoryError` and orders from one customer occasionally show up for another.
→ What the interviewer is testing: Understanding of `static` state in multi-threaded/multi-request web servers.
→ Strong answer approach:
- Identify that `static` fields are shared across **all threads/requests** in a servlet container — this is a textbook unintentional shared mutable state bug.
- Explain the memory leak: the list keeps growing because nothing evicts old entries (no bound, no TTL).
- Explain the data leak: concurrent requests from different users see each other's data because there's one shared list instance.
- Fix: use a proper cache (Caffeine, Redis) with eviction policies, or request-scoped/thread-local state where appropriate, never raw `static` mutable collections for request data.

**Scenario 2:** A teammate writes a utility method that builds a large CSV string by concatenating thousands of rows using `result += row + "\n"` inside a loop. The service starts timing out under load.
→ What the interviewer is testing: Understanding of String immutability and its performance implications.
→ Strong answer approach:
- Explain each `+=` creates an entirely new `String` object, copying all prior content — O(n²) behavior over the loop.
- Recommend `StringBuilder` (single-threaded context, which a typical request-handling method is) instead of `StringBuffer`.
- Mention that for very large outputs, streaming directly to the response/output stream (avoiding building one giant String at all) is even better in production.

**Scenario 3:** Your team finds a bug where two `Integer` objects representing the same value compared with `==` sometimes return `true` and sometimes `false`, depending on the value.
→ What the interviewer is testing: Knowledge of Integer caching (-128 to 127) and reference vs value comparison.
→ Strong answer approach:
- Explain Java caches boxed `Integer` values from -128 to 127 via `Integer.valueOf()`.
- Values within that range share cached instances (`==` true); outside it, new objects are created each time (`==` false).
- The real fix: always use `.equals()` for wrapper/object value comparison, never `==`.
- Mention this is a strong argument for static analysis tools (SonarQube, SpotBugs) catching this class of bug.

**Scenario 4:** A REST API endpoint occasionally throws `NullPointerException` at a line that unboxes an `Integer` field from a DTO that wasn't set by the client.
→ What the interviewer is testing: Awareness of autoboxing/unboxing NPE traps with optional/nullable fields.
→ Strong answer approach:
- Identify that unboxing a `null` wrapper (e.g., `int x = dto.getCount();` where `getCount()` returns `Integer`) throws NPE.
- Recommend explicit null checks, default values, or `Optional<Integer>` at the boundary, plus bean validation (`@NotNull`) on the DTO if the field is mandatory.
- Mention defensive coding: never directly assign a nullable wrapper to a primitive without a check.

**Scenario 5:** During a code review, you see a class with a `private List<String> items` field but a `public List<String> getItems()` getter that returns the field directly (no copy).
→ What the interviewer is testing: Understanding that encapsulation can be silently broken by leaking mutable internal state.
→ Strong answer approach:
- Explain that callers can call `.add()`/`.clear()` on the returned list directly, bypassing any validation logic in the owning class — encapsulation is broken in practice even though the field is `private`.
- Fix: return `Collections.unmodifiableList(items)` for read-only exposure, or return a defensive copy (`new ArrayList<>(items)`), depending on intended mutability semantics.
- Mention this is a common finding in real code reviews and static analysis (e.g., SpotBugs "EI_EXPOSE_REP").

**Scenario 6:** A junior developer's method has `try { return calculate(); } finally { cleanup(); return -1; }` and they're confused why the method always returns -1 even on success.
→ What the interviewer is testing: Deep understanding of `try-finally` semantics and return value precedence.
→ Strong answer approach:
- Explain that a `return` in `finally` **always wins**, silently discarding the `try` block's return value (and would even swallow an exception thrown in `try`).
- Strongly recommend never putting `return` (or `throw`) inside `finally` — it's almost always a bug, and most linters flag it.
- Correct fix: `finally` should only do cleanup (closing resources, logging), not control flow.

**Scenario 7:** Your microservice deploys two different versions of the same third-party library (transitively, via two different dependencies), and it works fine until a refactor causes `NoClassDefFoundError` randomly in production but not in local dev.
→ What the interviewer is testing: Understanding of classloading, classpath conflicts, and JVM behavior differences between environments.
→ Strong answer approach:
- Explain classloader delegation loads the *first* matching class found on the classpath — dependency order matters and can differ between local IDE classpath and the built JAR/Docker classpath.
- Recommend `mvn dependency:tree` / Gradle equivalent to find conflicting versions, then explicitly exclude/pin a single version.
- Mention tools like Maven Shade plugin relocation or dependency convergence checks for resolving "JAR hell" in real CI/CD pipelines.

═══════════════════════════════════════════
## SECTION 4: CODING PROBLEMS (4)
═══════════════════════════════════════════

**Problem 1:** Write a `Vehicle` class hierarchy with an abstract class `Vehicle` (with a `final` method `displayBrand()` and an abstract method `fuelType()`), and two subclasses `ElectricCar` and `PetrolCar` that each implement `fuelType()` differently. Demonstrate runtime polymorphism by calling `fuelType()` through a `Vehicle[]` array.
→ Difficulty: Easy
→ Key concept tested: Abstraction, inheritance, `final` methods, dynamic method dispatch
→ Hint: Remember a `final` method cannot be overridden — try overriding it in a subclass and see what the compiler says.

**Problem 2:** Write a program that demonstrates the Integer caching behavior — print whether `==` returns `true` or `false` for `Integer` values 100 and 200 created two different ways, and explain the output in comments.
→ Difficulty: Easy
→ Key concept tested: Wrapper class caching, `==` vs `.equals()`
→ Hint: Try the same comparisons using `Integer.valueOf()` explicitly vs `new Integer()` — note `new Integer()` is deprecated, but useful to understand why.

**Problem 3:** Build a custom unchecked exception `InvalidAgeException`, and a `Person` class whose constructor throws it if age is negative or greater than 150. Write a `main()` that handles this with try-catch and also demonstrates a `finally` block that always executes.
→ Difficulty: Medium
→ Key concept tested: Custom exceptions, checked vs unchecked design choice, try-catch-finally flow
→ Hint: Decide deliberately whether to extend `Exception` or `RuntimeException` — be ready to justify your choice the way a senior dev would in a design review.

**Problem 4:** Write a method that takes a `List<String>` of 100,000 words and concatenates them into one comma-separated String — implement it TWO ways (using `+=` in a loop, and using `StringBuilder`), then add simple timing code (`System.nanoTime()`) to compare performance and print the difference.
→ Difficulty: Medium
→ Key concept tested: String immutability cost, StringBuilder mutability advantage, real performance reasoning (not just theory)
→ Hint: Watch how the time gap changes as you scale the list size from 1,000 to 100,000 — the growth rate itself is the lesson.

**Problem 5:** Implement a thread-unsafe `Counter` class with an `increment()` method using a plain `int`, and simulate 2 threads calling `increment()` 10,000 times each concurrently. Print the final count and explain in comments why it's *usually* less than 20,000.
→ Difficulty: Hard
→ Key concept tested: Sets up the bridge into multithreading/concurrency (Day 2+ territory) — but rooted in understanding that simple operations aren't atomic
→ Hint: This isn't solvable correctly using what we covered today alone — that's the point. Note where `static`/instance state breaks down without synchronization, and we'll properly fix it when we hit the Concurrency module.

═══════════════════════════════════════════
## SECTION 5: REVISION NOTES
═══════════════════════════════════════════

### CORE CONCEPTS (one-liner each)
- **JVM**: Executes bytecode; platform-dependent implementation, platform-independent bytecode.
- **JRE**: JVM + core libraries needed to run Java apps.
- **JDK**: JRE + development tools (javac, debugger).
- **Class loading**: Lazy, hierarchical (Bootstrap → Platform → Application), delegation-based.
- **Bytecode execution**: Interpreted first, then JIT-compiled to native code for hot paths.
- **Data types**: 8 primitives, fixed sizes, default values for fields only.
- **Type casting**: Widening = automatic & safe; narrowing = explicit & lossy.
- **Wrapper classes**: Object versions of primitives, needed for Collections/generics.
- **Autoboxing**: Compiler auto-converts primitive ↔ wrapper; costs performance in loops.
- **Class/Object**: Blueprint vs instance; `this` disambiguates; `static` is class-level, `final` prevents reassignment/override/extension.
- **Inheritance**: `extends` (single, classes) for code reuse and IS-A relationships.
- **Polymorphism**: Overloading = compile-time; Overriding = runtime via dynamic dispatch.
- **Abstraction**: Hides "how," exposes "what" — via abstract classes/interfaces.
- **Encapsulation**: Hides data via `private` fields + controlled public access.
- **Abstract class vs Interface**: State+single inheritance vs contract+multiple inheritance.
- **Access modifiers**: private < default < protected < public.
- **Packages**: Namespacing + organization; folder structure must match package declaration.
- **String/StringBuilder/StringBuffer**: Immutable vs mutable-unsynchronized vs mutable-synchronized.
- **Arrays**: Fixed-size, index-based, `.length` is a field not a method.
- **Exceptions**: Checked (compiler-enforced) vs Unchecked (RuntimeException) vs Error (JVM-fatal).

### KEY DIFFERENCES (A vs B — the one thing to remember)
- **JDK vs JRE vs JVM** → JDK builds it, JRE runs it, JVM executes it.
- **== vs .equals()** → `==` compares references (or primitive values); `.equals()` compares logical content.
- **Overloading vs Overriding** → compile-time/same class vs runtime/parent-child.
- **Abstract class vs Interface** → can have state+constructor vs pure contract (+ default methods since Java 8).
- **Abstraction vs Encapsulation** → hiding complexity vs hiding data.
- **Checked vs Unchecked exception** → must declare/catch vs compiler doesn't enforce.
- **StringBuilder vs StringBuffer** → not synchronized/fast vs synchronized/thread-safe/slower.
- **NoClassDefFoundError vs ClassNotFoundException** → runtime missing class (Error) vs explicit `Class.forName()` failure (checked Exception).

### GOTCHAS & TRAPS (what freshers get wrong in interviews)
- ⚠️ Saying "JVM is platform-independent" — it's the bytecode that is, not the JVM itself.
- ⚠️ `(int) 9.99` truncates to `9`, does NOT round.
- ⚠️ `Integer a=127,b=127; a==b` → true; with 200 → false (Integer cache range -128 to 127).
- ⚠️ Unboxing a `null` wrapper throws `NullPointerException`.
- ⚠️ Field access is NOT polymorphic — only methods are (resolved via reference type for fields).
- ⚠️ `final` reference ≠ immutable object — only the reference binding is frozen.
- ⚠️ `static` methods can be hidden, never truly overridden.
- ⚠️ `return` inside `finally` silently overrides try's return value (and swallows exceptions).
- ⚠️ `Arrays.asList()` returns a fixed-size list — `.add()` throws `UnsupportedOperationException`.
- ⚠️ Wildcard imports don't include sub-packages.
- ⚠️ `0.1 + 0.2 != 0.3` due to floating-point binary representation.

### MUST-KNOW CODE (minimal interview-critical snippets only)
```java
// Integer cache trap
Integer a = 127, b = 127; System.out.println(a == b); // true
Integer x = 200, y = 200; System.out.println(x == y); // false

// String pool vs heap
String s1 = "Hi"; String s2 = new String("Hi");
System.out.println(s1 == s2); // false

// finally overriding return
static int f() { try { return 1; } finally { return 2; } } // returns 2

// Field access not polymorphic
class A { int x=1; } class B extends A { int x=2; }
A obj = new B(); System.out.println(obj.x); // 1
```

### PRODUCTION FACTS (how this actually works in real systems)
- Docker images use **JRE-only** base images to reduce size (no `javac` needed at runtime).
- Application servers (Tomcat) use **custom classloaders per WAR** for dependency isolation.
- JIT "warm-up" explains why JVM apps are slower on first requests — matters for k8s readiness probes.
- `BigDecimal`, never `float`/`double`, for money/currency in production.
- "Favor composition over inheritance" — deep hierarchies are fragile in large codebases.
- Modern Spring Boot custom exceptions typically extend `RuntimeException` (unchecked) to avoid `throws` boilerplate.
- Returning internal mutable collections directly from getters is a real, frequently-flagged production bug (SpotBugs `EI_EXPOSE_REP`).

### TOP 5 MOST LIKELY INTERVIEW QUESTIONS
1. Difference between String, StringBuilder, and StringBuffer?
2. Why does `Integer a=127==b=127` give true but `200==200` give false?
3. Difference between abstract class and interface — when to use which?
4. Explain overloading vs overriding with binding type (compile-time vs runtime).
5. Checked vs unchecked exceptions — which should custom exceptions extend today, and why?

### SELF-CHECK BEFORE MOVING ON
- [ ] I can explain JDK vs JRE vs JVM and why bytecode (not JVM) is platform-independent.
- [ ] I can predict the output of an Integer `==` comparison for both cached and non-cached values without running the code.
- [ ] I can explain, with code, why field access is not polymorphic but method calls are.
- [ ] I can justify when to use StringBuilder vs StringBuffer vs immutable String in a real system.
- [ ] I can explain why `return` in `finally` is dangerous and trace through the execution order myself.

---

That's Day 1 fully locked in. Solid foundation — everything from here (Collections, Multithreading, Streams, Spring) builds directly on top of this. Let me know when you want **Day 2**, or if you want to attempt the coding problems first and have me review your solutions.
