# Classes and Objects in Java

## 1. What is a Class?

A **class** is a blueprint, template, or recipe for creating objects.

It defines:

* **What an object contains** → data/state
* **What an object can do** → behavior/methods

A class itself is **not an object**. It is a template that can be used to create multiple objects with the same structure and behavior, while each object maintains its own independent state.

### Key Characteristics of a Class

A class:

* Groups related **data and behavior** together.
* Defines **attributes/fields** that represent the state of an object.
* Defines **methods** that represent the behavior or actions an object can perform.
* Can have a **constructor** used to initialize objects.

### Example: Class as a Blueprint

```java
public class Car {

    // Attributes / State
    private String brand;
    private String model;
    private int speed;

    // Constructor
    public Car(String brand, String model) {
        this.brand = brand;
        this.model = model;
        this.speed = 0;
    }

    // Behavior
    public void accelerate(int increment) {
        speed += increment;
    }

    // Behavior
    public void displayStatus() {
        System.out.println(
            brand + " " + model + " is running at " + speed + " km/h."
        );
    }
}
```

Here, `Car` is a **class**.

It defines:

| Component         | Purpose                          |
| ----------------- | -------------------------------- |
| `brand`           | Stores the car's brand           |
| `model`           | Stores the car's model           |
| `speed`           | Stores the current speed         |
| `Car()`           | Initializes a new car            |
| `accelerate()`    | Changes the car's speed          |
| `displayStatus()` | Displays the car's current state |

The class defines the **structure and behavior**, but no specific car exists until we create an object.

---

## 2. What is an Object?

An **object is an instance of a class**.

It is the actual entity created from the class that we can:

* Store data in
* Access its state through methods
* Invoke its behavior

If a class is a **blueprint**, an object is the **actual thing created using that blueprint**.

### Example

```java
Car corolla = new Car("Toyota", "Corolla");
Car mustang = new Car("Ford", "Mustang");
```

Here:

* `Car` → class
* `corolla` → object/reference variable
* `mustang` → object/reference variable
* `new Car(...)` → creates objects

Both objects are created from the same `Car` class, but they have **independent state**.

---

## 3. Creating Objects

Objects are commonly created using the `new` keyword.

### Syntax

```java
ClassName referenceVariable = new ClassName(arguments);
```

Example:

```java
Car corolla = new Car("Toyota", "Corolla");
```

Conceptually:

```text
Car class
   |
   |----> corolla object
   |
   |----> mustang object
```

Both objects have the same structure defined by `Car`, but their data can be different.

---

## 4. Objects Have Independent State

Consider:

```java
Car corolla = new Car("Toyota", "Corolla");
Car mustang = new Car("Ford", "Mustang");

corolla.accelerate(20);
mustang.accelerate(40);
```

The `speed` of each object is independent.

```text
corolla
brand = Toyota
model = Corolla
speed = 20

mustang
brand = Ford
model = Mustang
speed = 40
```

Changing the state of one object does **not** change the state of the other.

This is an important characteristic of objects created from the same class.

---

## 5. Complete Example

### `Car.java`

```java
public class Car {

    private String brand;
    private String model;
    private int speed;

    public Car(String brand, String model) {
        this.brand = brand;
        this.model = model;
        this.speed = 0;
    }

    public void accelerate(int increment) {
        speed += increment;
    }

    public void displayStatus() {
        System.out.println(
            brand + " " + model + " is running at " + speed + " km/h."
        );
    }
}
```

### `Main.java`

```java
public class Main {

    public static void main(String[] args) {

        // Creating objects of the Car class
        Car corolla = new Car("Toyota", "Corolla");
        Car mustang = new Car("Ford", "Mustang");

        // Calling methods on objects
        corolla.accelerate(20);
        mustang.accelerate(40);

        // Displaying the state of each object
        corolla.displayStatus();

        System.out.println("-----------------");

        mustang.displayStatus();
    }
}
```

### Output

```text
Toyota Corolla is running at 20 km/h.
-----------------
Ford Mustang is running at 40 km/h.
```

---

## 6. Class vs Object

| Class                                | Object                              |
| ------------------------------------ | ----------------------------------- |
| Blueprint/template                   | Instance created from the blueprint |
| Defines structure and behavior       | Represents an actual entity         |
| Does not represent a specific entity | Represents a specific entity        |
| Used to create objects               | Created from a class                |
| Example: `Car`                       | Example: `corolla`                  |
| Describes what an object should have | Contains actual state/data          |

### Simple Analogy

Think of a **class as a house blueprint**.

```text
             Class
        ┌───────────────┐
        │ House Blueprint│
        │ rooms          │
        │ doors          │
        │ windows        │
        └───────────────┘
             ↓
       ┌─────┴─────┐
       ↓           ↓
    Object 1     Object 2
    House A      House B
```

The blueprint is the same, but the actual houses can have different colors, furniture, owners, etc.

Similarly:

```java
class Car
```

is the blueprint, while:

```java
Car corolla = new Car("Toyota", "Corolla");
Car mustang = new Car("Ford", "Mustang");
```

creates two independent objects.

---

## 7. Key Interview Points

> **Class = Blueprint**
> **Object = Instance of a class**

Remember these points:

1. A class defines the **state and behavior** of objects.
2. An object is an **instance of a class**.
3. Multiple objects can be created from one class.
4. Objects created from the same class have the same structure but can have **different state**.
5. Objects are commonly created using the `new` keyword.
6. Methods define the behavior that objects can perform.
7. Each object generally has its own instance fields/state.

### One-Line Interview Answer

**A class is a blueprint that defines the state and behavior of objects, while an object is an actual instance of that class with its own state.**

