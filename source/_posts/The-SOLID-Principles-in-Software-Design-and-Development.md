---
title: The SOLID Principles in Software Design and Development
date: 2024-10-27 21:56:31
categories:
- Theory
tags:
- Theory
---

- [Introduction](#introduction)
- [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
  - [Definition](#definition)
  - [Broken Example](#broken-example)
  - [Correct Example](#correct-example)
- [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
  - [Definition](#definition-1)
  - [Broken Example](#broken-example-1)
  - [Correct Example](#correct-example-1)
- [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
  - [Definition](#definition-2)
  - [Broken Example](#broken-example-2)
  - [Correct Example](#correct-example-2)
- [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
  - [Definition](#definition-3)
  - [Broken Example](#broken-example-3)
  - [Correct Example](#correct-example-3)
- [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  - [Definition](#definition-4)
  - [Broken Example](#broken-example-4)
  - [Explanation](#explanation)
  - [Why High-Level Module Depends on Low-Level Module](#why-high-level-module-depends-on-low-level-module)
  - [Correct Example](#correct-example-4)
  - [Explanation of Correct Example](#explanation-of-correct-example)
- [Conclusion](#conclusion)
- [References:](#references)

---

<a name="introduction"></a>
## Introduction

The SOLID principles are a set of five design principles intended to make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (also known as Uncle Bob) and are widely accepted in the software development community. The acronym SOLID stands for:

- **S**ingle Responsibility Principle (SRP)
- **O**pen/Closed Principle (OCP)
- **L**iskov Substitution Principle (LSP)
- **I**nterface Segregation Principle (ISP)
- **D**ependency Inversion Principle (DIP)

![SOLID Principles](images/The-SOLID-Principles-in-Software-Design-and-Development/SOLID_Principles.png)
This article will explain each principle, provide examples of how they can be broken, and show the correct implementation to fix them.

---

<a name="single-responsibility-principle-srp"></a>
## Single Responsibility Principle (SRP)

### Definition
The Single Responsibility Principle states that a class should have only one reason to change, meaning it should have only one job or responsibility.

### Broken Example
```java
class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public void save() {
        // Code to save employee data to the database
        System.out.println("Employee saved to database: " + name);
    }

    public void calculateSalary() {
        // Code to calculate salary
        System.out.println("Salary calculated: " + salary);
    }
}
```

### Correct Example
```java
class Employee {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public void calculateSalary() {
        // Code to calculate salary
        System.out.println("Salary calculated: " + salary);
    }
}

class EmployeeRepository {
    public void save(Employee employee) {
        // Code to save employee data to the database
        System.out.println("Employee saved to database: " + employee.getName());
    }
}
```

---

<a name="openclosed-principle-ocp"></a>
## Open/Closed Principle (OCP)

### Definition
The Open/Closed Principle states that software entities (classes, modules, functions, etc.) should be open for extension but closed for modification.

### Broken Example
```java
class Shape {
    String type;

    public Shape(String type) {
        this.type = type;
    }
}

class AreaCalculator {
    public double calculateArea(Shape shape) {
        if (shape.type.equals("circle")) {
            // Calculate area for circle
            return Math.PI * Math.pow(5, 2); // Assuming radius is 5
        } else if (shape.type.equals("rectangle")) {
            // Calculate area for rectangle
            return 10 * 5; // Assuming width is 10 and height is 5
        }
        return 0;
    }
}
```

### Correct Example
```java
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * Math.pow(radius, 2);
    }
}

class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
}
```

---

<a name="liskov-substitution-principle-lsp"></a>
## Liskov Substitution Principle (LSP)

### Definition
The Liskov Substitution Principle states that objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.

### Broken Example
```java
class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Ostriches cannot fly");
    }
}
```

### Correct Example
```java
class Bird {
    public void eat() {
        System.out.println("Eating...");
    }
}

class FlyingBird extends Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

class Ostrich extends Bird {
    // Ostriches do not fly, so no fly method here
}
```

---

<a name="interface-segregation-principle-isp"></a>
## Interface Segregation Principle (ISP)

### Definition
The Interface Segregation Principle states that clients should not be forced to depend on interfaces they do not use.

### Broken Example
```java
interface Worker {
    void work();
    void eat();
}

class Robot implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working...");
    }

    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots do not eat");
    }
}
```

### Correct Example
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Robot implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working...");
    }
}

class Human implements Workable, Eatable {
    @Override
    public void work() {
        System.out.println("Human working...");
    }

    @Override
    public void eat() {
        System.out.println("Human eating...");
    }
}
```

---

<a name="dependency-inversion-principle-dip"></a>
## Dependency Inversion Principle (DIP)

### Definition
The Dependency Inversion Principle states that high-level modules should not depend on low-level modules. Both should depend on abstractions. Additionally, abstractions should not depend on details. Details should depend on abstractions.

### Broken Example
```java
class LightBulb {
    public void turnOn() {
        System.out.println("LightBulb turned on");
    }

    public void turnOff() {
        System.out.println("LightBulb turned off");
    }
}

class Switch {
    private LightBulb bulb;

    public Switch() {
        this.bulb = new LightBulb();
    }

    public void operate() {
        if (isOn) {
            bulb.turnOff();
            isOn = false;
        } else {
            bulb.turnOn();
            isOn = true;
        }
    }

    private boolean isOn = false;
}
```

### Explanation

In the broken example, the `Switch` class directly depends on the `LightBulb` class. This creates a tight coupling between the `Switch` (high-level module) and the `LightBulb` (low-level module). Here’s why this violates the Dependency Inversion Principle:

1. **Direct Dependency**: The `Switch` class directly creates an instance of `LightBulb` within its constructor. This means that the `Switch` class is tightly coupled to the `LightBulb` class. If you want to change the type of device that the `Switch` controls (e.g., a fan instead of a light bulb), you would need to modify the `Switch` class itself.

2. **Lack of Abstraction**: There is no abstraction layer between the `Switch` and the `LightBulb`. The `Switch` class is directly aware of the implementation details of the `LightBulb` class. This makes the `Switch` class less flexible and harder to extend or modify.

3. **Rigidity**: The `Switch` class is rigid because it can only operate on a `LightBulb`. If you want to add support for other devices (e.g., a `Fan`), you would need to create a new `Switch` class or modify the existing one, which is not ideal.

### Why High-Level Module Depends on Low-Level Module

- **Direct Instantiation**: The `Switch` class directly instantiates the `LightBulb` class within its constructor. This means that the `Switch` class is directly dependent on the `LightBulb` class.
- **No Abstraction Layer**: There is no interface or abstract class that both the `Switch` and `LightBulb` classes implement. This lack of abstraction means that the `Switch` class is tightly coupled to the specific implementation of the `LightBulb` class.

### Correct Example

To adhere to the Dependency Inversion Principle, we introduce an abstraction (an interface) that both the `Switch` and the `LightBulb` classes depend on.

```java
interface Switchable {
    void turnOn();
    void turnOff();
}

class LightBulb implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("LightBulb turned on");
    }

    @Override
    public void turnOff() {
        System.out.println("LightBulb turned off");
    }
}

class Switch {
    private Switchable device;

    public Switch(Switchable device) {
        this.device = device;
    }

    public void operate() {
        if (isOn) {
            device.turnOff();
            isOn = false;
        } else {
            device.turnOn();
            isOn = true;
        }
    }

    private boolean isOn = false;
}
```

### Explanation of Correct Example

1. **Abstraction Layer**: The `Switchable` interface acts as an abstraction layer. Both the `Switch` and the `LightBulb` classes depend on this abstraction.
2. **Dependency Injection**: The `Switch` class now takes a `Switchable` object as a parameter in its constructor. This allows the `Switch` class to operate on any device that implements the `Switchable` interface, not just a `LightBulb`.
3. **Flexibility**: This design makes the `Switch` class flexible and easier to extend. You can now create other classes that implement the `Switchable` interface (e.g., `Fan`) and pass them to the `Switch` class without modifying the `Switch` class itself.

By adhering to the Dependency Inversion Principle, we ensure that high-level modules (like `Switch`) do not depend on low-level modules (like `LightBulb`). Instead, both depend on abstractions (like the `Switchable` interface), making the system more modular, flexible, and easier to maintain.

---

<a name="conclusion"></a>
## Conclusion

The SOLID principles are fundamental to writing clean, maintainable, and scalable software. By adhering to these principles, developers can create code that is easier to understand, extend, and refactor. Each principle addresses a specific aspect of software design, and understanding how to apply them correctly can significantly improve the quality of your code.

## References:
[https://blog.bytebytego.com/p/mastering-design-principles-solid](https://blog.bytebytego.com/p/mastering-design-principles-solid)