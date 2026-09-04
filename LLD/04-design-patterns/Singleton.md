# Singleton Design Pattern

> **Category:** Creational Design Pattern
> **Purpose:** Ensure that a class has only one instance and provide a global point of access to that instance.

---

## 1. What is Singleton Pattern?

In software development, there are situations where we need a class to have **only one object throughout the application**.

Examples include:

* Thread pools
* Caches
* Loggers
* Configuration managers
* Database connection pools

Creating multiple objects of these classes can result in:

* Incorrect application behavior
* Unnecessary resource consumption
* Inconsistent state or results

The **Singleton Design Pattern** addresses this problem.

### Definition

The **Singleton Pattern** is a **creational design pattern** that:

1. Ensures that a class has **only one instance**.
2. Provides a **global point of access** to that instance.

### Two Core Requirements

#### 1. Single Instance

Regardless of how many times different parts of the application request the object, the **same instance** should be returned.

#### 2. Global Access

Any component should be able to access the instance without having to pass it through constructors or method parameters.

---

## Real-World Analogy

Consider a **print spooler** in an operating system.

There should be one central print spooler responsible for managing print jobs.

Applications don't create their own print spoolers. Instead, they submit their jobs to the existing spooler.

If every application created its own spooler, print jobs could conflict or become incorrectly ordered.

The single spooler provides centralized coordination.

---

## Common Use Cases

Singleton can be useful for managing:

### Shared Resources

* Database connection pools
* Thread pools
* Caches
* Configuration settings

### System-Wide Operations

* Logging
* Print spoolers
* File managers

### Shared State

* Application state
* User session state

### Specific Examples

**Logger**

A logging system can use one shared logging object so that messages are consistently handled through the same logging mechanism.

**Database Connection Pool**

A Singleton can ensure that the application uses one shared connection pool instead of creating multiple pools.

**Cache**

A shared in-memory cache can provide a single place for storing and retrieving cached data.

**Thread Pool**

A shared thread pool prevents different parts of an application from creating unnecessary worker threads.

**File System**

A Singleton can represent the application's file-system manager and provide a unified interface for file operations.

---

# 2. Class Structure

The basic Singleton structure has three important parts:

```text
                ┌───────────────────────┐
                │      Singleton        │
                ├───────────────────────┤
                │ - instance            │
                ├───────────────────────┤
                │ - Singleton()         │
                │ + getInstance()       │
                └───────────┬───────────┘
                            │
                            │ returns
                            ▼
                     Single Instance
```

### Important Components

#### 1. `instance`

A field stores the **one and only Singleton object**.

#### 2. Private Constructor

The constructor is private or otherwise restricted so that external classes cannot directly create objects.

```java
private Singleton() {
}
```

This prevents:

```java
Singleton s = new Singleton(); // Not allowed
```

#### 3. `getInstance()`

A class-level method provides access to the Singleton instance.

```java
Singleton.getInstance();
```

---

## Why Not Use a Global Variable?

A global variable can provide similar accessibility, but a Singleton gives more control over object creation.

A Singleton can control:

* When the object is created
* Whether initialization is lazy
* Thread safety during creation
* Whether only one instance can exist

---

# 3. How Singleton Works

The basic workflow is:

```text
Client
  │
  │ getInstance()
  ▼
Does instance exist?
  │
  ├── No ──► Create instance
  │              │
  │              ▼
  │        Store instance
  │
  └── Yes
         │
         ▼
   Return instance
```

### Step 1: First Request

The client calls:

```java
Singleton.getInstance();
```

The method checks whether an instance already exists.

### Step 2: Instance Creation

If no instance exists, the Singleton creates one using its private constructor.

### Step 3: Return Instance

The newly created instance is returned to the caller.

### Step 4: Subsequent Requests

When another component calls `getInstance()`, the existing instance is returned.

No new object is created.

```text
First call:

Client A ──► getInstance() ──► Create Object ──► Instance


Later:

Client B ──► getInstance() ──┐
                             ├──► Same Instance
Client C ──► getInstance() ──┘
```

Therefore:

```java
Singleton a = Singleton.getInstance();
Singleton b = Singleton.getInstance();

System.out.println(a == b);
```

Output:

```text
true
```

---

# 4. Singleton Implementations

The main challenge when implementing Singleton is **thread safety**.

Consider two threads:

```text
Thread 1 ─────► getInstance()
                      │
                 instance == null
                      │
Thread 2 ─────► getInstance()
                      │
                 instance == null
```

If both threads check the instance before either creates it, they could potentially create two objects.

Therefore, the implementation must be designed carefully.

---

## 4.1 Lazy Initialization

In **lazy initialization**, the Singleton object is created only when it is first requested.

### Concept

```java
if (instance == null) {
    instance = new Singleton();
}

return instance;
```

### Advantages

* Instance is created only when needed.
* Avoids unnecessary initialization.

### Problem

This approach is **not thread-safe**.

If multiple threads call `getInstance()` at the same time while `instance` is `null`, more than one object can potentially be created.

```text
Thread 1                 Thread 2
   │                         │
   ▼                         ▼
instance == null       instance == null
   │                         │
   ▼                         ▼
create object          create object
   │                         │
   └──────────┬──────────────┘
              ▼
       Multiple instances
```

---

# 4.2 Thread-Safe Singleton

One solution is to synchronize access to the method or critical section.

Conceptually:

```java
synchronized getInstance()
```

Synchronization ensures that only one thread can execute the protected section at a time.

### How It Works

1. A thread requests the instance.
2. It acquires the lock.
3. It checks whether the instance exists.
4. If required, it creates the instance.
5. The lock is released.
6. Other threads can continue.

This guarantees that only one instance is created.

### Problem

The approach has a performance cost.

Every call to `getInstance()` needs to acquire the lock, even after the Singleton has already been created.

Once the instance exists, synchronization is generally unnecessary.

---

# 4.3 Double-Checked Locking

**Double-checked locking** improves the synchronized implementation.

The idea is to check the instance twice:

```text
             getInstance()
                  │
                  ▼
          instance == null?
             /          \
           No            Yes
           │              │
           ▼              ▼
      Return instance    Lock
                           │
                           ▼
                    Check again
                           │
                    instance == null?
                       /        \
                     No          Yes
                     │            │
                     ▼            ▼
                Return        Create
                              instance
```

The first check avoids synchronization once the instance already exists.

If the first check indicates that the instance is missing, synchronization is acquired and the condition is checked again.

### Why Check Twice?

Multiple threads may pass the first check before one of them creates the object.

The second check ensures that another thread has not already created the instance while the current thread was waiting for the lock.

### Advantage

* Synchronization is primarily needed during initial creation.
* Subsequent accesses avoid the locking overhead.

### Disadvantage

* More complicated to implement correctly.

---

# 4.4 Eager Initialization

With **eager initialization**, the Singleton object is created when the class/module is initialized rather than when it is first requested.

Conceptually:

```java
private static final Singleton INSTANCE = new Singleton();
```

The instance already exists before clients request it.

### Why Is It Thread-Safe?

Class/static initialization happens once, and the runtime guarantees safe initialization.

Therefore, explicit synchronization is not required for the creation of the instance.

### Advantages

* Simple
* Thread-safe
* No synchronization required

### Disadvantage

The object is created even if the application never actually needs it.

This can waste resources when initialization is expensive.

---

# 5. Java-Specific Singleton Implementations

Java provides several well-known approaches for implementing Singleton.

---

## 5.1 Bill Pugh Singleton / Initialization-on-Demand Holder

This approach uses a **static inner class**.

```java
class Singleton {

    private Singleton() {
    }

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### How It Works

The `Holder` class is not initialized when `Singleton` itself is loaded.

It is initialized only when `getInstance()` accesses it.

```text
Singleton loaded
      │
      ▼
Holder NOT loaded
      │
      │ getInstance()
      ▼
Holder loaded
      │
      ▼
INSTANCE created
```

Java class initialization is thread-safe, so explicit synchronization is not required.

### Advantages

* Lazy initialization
* Thread-safe
* No explicit synchronization
* Good performance

This provides a strong balance between **lazy loading, thread safety, and performance**.

---

# 5.2 Enum Singleton

For Java, an **enum Singleton** is one of the simplest and safest approaches.

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // logic
    }
}
```

Usage:

```java
Singleton.INSTANCE.doSomething();
```

### Why Is Enum Singleton Special?

The JVM provides several useful guarantees:

#### 1. Thread-Safe Initialization

Enum constants are initialized once when the enum class is initialized.

#### 2. Serialization Safety

Serialization and deserialization preserve the same enum instance.

#### 3. Reflection Safety

The JVM prevents normal reflective construction of enum instances.

#### 4. Single Instance Guarantee

The Singleton property is enforced at the JVM level.

### Limitation

An enum cannot extend another class because every enum already extends:

```java
java.lang.Enum
```

Therefore, an enum Singleton cannot be used when the Singleton needs to extend another class.

---

# 5.3 Static Block Initialization

Another approach is to create the Singleton inside a static initializer block.

```java
class Singleton {

    private static Singleton instance;

    static {
        try {
            instance = new Singleton();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

The static block executes during class initialization.

### Advantage

It provides an opportunity to handle exceptions during object creation.

### Disadvantage

Like eager initialization, the object is created during class initialization rather than lazily.

This may be undesirable when initialization is expensive.

---

# 6. Practical Example: In-Memory Cache Manager

Consider an application where multiple components need to cache expensive data.

For example:

* HTTP handlers
* Database layers
* Background jobs

They may all need access to:

* User profiles
* Configuration
* Query results

You want all components to use **one shared cache**.

Without a Singleton, different components might maintain separate cache objects.

```text
Component A ──► Cache A
Component B ──► Cache B
Component C ──► Cache C
```

This can result in:

* Duplicate data
* Wasted memory
* Stale values
* Different components having different cache states

With Singleton:

```text
Component A ──┐
Component B ──┼──► CacheManager ──► One shared cache
Component C ──┘
```

Every component accesses the same `CacheManager`.

### Benefits

* One shared cache
* No duplicate cached data
* Updates are visible to other components
* Centralized cache management
* Thread-safe access can be handled internally
* TTL/expiration logic can be centralized
* Cache references don't need to be passed everywhere

---

# 7. Pros and Cons

## Pros

### 1. Guarantees a Single Instance

The pattern ensures that only one instance of the class is available.

### 2. Global Access

The instance can be accessed from different parts of the application.

### 3. Resource Efficiency

Only one object is created, which can be useful for resource-heavy components.

### 4. Shared State

All components can work with the same shared state.

### 5. Lazy Initialization

Some implementations allow the instance to be created only when it is first needed.

---

## Cons

### 1. Global State

Singleton introduces global state, which can make an application harder to reason about.

### 2. Tight Coupling

Classes using the Singleton directly can become tightly coupled to that specific implementation.

### 3. Testing Difficulties

Global state can make unit testing harder because tests may share the same Singleton state.

### 4. Thread-Safety Concerns

Incorrect implementations can create multiple instances in concurrent environments.

### 5. Single Responsibility Principle

Singleton can be considered problematic from an SRP perspective because the class may have both:

* Its actual business responsibility
* The responsibility of controlling its own lifecycle/instance creation

---

# 8. Singleton vs Dependency Injection

Singleton should be used carefully.

A common alternative is **Dependency Injection (DI)**.

Instead of having a class directly access:

```java
Singleton.getInstance();
```

the dependency can be provided to it:

```java
class Service {

    private final CacheManager cache;

    Service(CacheManager cache) {
        this.cache = cache;
    }
}
```

### Why Prefer DI?

Dependency Injection can improve:

* Loose coupling
* Testability
* Maintainability
* Flexibility

For example, a test can provide a mock/fake implementation instead of depending on a global Singleton.

---

# 9. Quick Comparison

| Implementation         | Lazy | Thread-Safe | Complexity | Performance |
| ---------------------- | ---- | ----------- | ---------- | ----------- |
| Lazy Initialization    | ✅    | ❌           | Low        | Good        |
| Synchronized Method    | ✅    | ✅           | Low        | Lower       |
| Double-Checked Locking | ✅    | ✅           | Medium     | Good        |
| Eager Initialization   | ❌    | ✅           | Low        | Good        |
| Bill Pugh Holder       | ✅    | ✅           | Low        | Excellent   |
| Enum Singleton         | ❌    | ✅           | Very Low   | Excellent   |

---

# 10. Interview Takeaways

### What is Singleton?

> A creational design pattern that ensures a class has only one instance and provides a global point of access to it.

### Why make the constructor private?

To prevent other classes from directly creating instances using `new`.

### What is the main challenge with Singleton?

**Thread safety**, especially when lazy initialization is used.

### What is double-checked locking?

A technique that checks whether the instance exists before acquiring a lock and checks again after acquiring the lock to avoid unnecessary synchronization.

### What is Bill Pugh Singleton?

A Java implementation that uses a static inner holder class to achieve lazy initialization and thread safety without explicit synchronization.

### What is the recommended Java Singleton approach?

An **enum Singleton** is a simple and robust approach, particularly because Java provides strong guarantees around enum initialization and serialization.

### What is a major drawback of Singleton?

It introduces **global state**, which can increase coupling and make testing more difficult.

---

# 11. Key Points to Remember

```text
Singleton
   │
   ├── Creational Design Pattern
   │
   ├── Only ONE instance
   │
   ├── Global access
   │
   ├── Private constructor
   │
   ├── getInstance()
   │
   ├── Thread safety is important
   │
   ├── Lazy / Eager initialization
   │
   ├── Double-Checked Locking
   │
   ├── Bill Pugh Holder
   │
   └── Enum Singleton
```

> **Important:** Singleton is not automatically a good design simply because only one object is needed. Consider whether global state and tight coupling are actually appropriate. Dependency Injection is often a better alternative when testability and loose coupling are important.
