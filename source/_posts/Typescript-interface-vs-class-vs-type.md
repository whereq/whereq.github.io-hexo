---
title: Typescript interface vs class vs type
date: 2024-12-09 10:23:53
categories:
- Typescript
tags:
- Typescript
---

In TypeScript, `interface`, `class`, and `type` are all used to define the shape of objects, but they serve different purposes and have different use cases. Let's delve into each of them in detail, including real-world scenarios with sample code, and discuss their pros and cons.

### 1. **Interface**

An `interface` is a way to define a contract for the shape of an object. It can describe the structure of an object, including properties and methods, but it cannot contain implementation details.

#### **Sample Code:**

```typescript
// Defining an interface
interface User {
    id: number;
    name: string;
    email: string;
    isActive: boolean;
    getDetails(): string;
}

// Implementing the interface in a class
class RegisteredUser implements User {
    id: number;
    name: string;
    email: string;
    isActive: boolean;

    constructor(id: number, name: string, email: string, isActive: boolean) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.isActive = isActive;
    }

    getDetails(): string {
        return `ID: ${this.id}, Name: ${this.name}, Email: ${this.email}, Active: ${this.isActive}`;
    }
}

// Using the class
const user = new RegisteredUser(1, "John Doe", "john@example.com", true);
console.log(user.getDetails()); // Output: ID: 1, Name: John Doe, Email: john@example.com, Active: true
```

#### **Pros:**
- **Contract Definition:** Interfaces are great for defining contracts that classes or objects must adhere to.
- **Extensibility:** You can extend interfaces using the `extends` keyword, which allows for composition of types.
- **Readability:** Interfaces are often more readable and easier to understand than type aliases, especially for complex object shapes.

#### **Cons:**
- **No Implementation:** Interfaces cannot contain implementation details, so they are limited to defining the shape of objects.
- **Cannot Be Used for Primitive Types:** Interfaces cannot be used to define types for primitive values (e.g., `string`, `number`).

### 2. **Class**

A `class` in TypeScript is a blueprint for creating objects. It can define both the structure and the behavior of objects. Classes can implement interfaces and can also be extended.

#### **Sample Code:**

```typescript
// Defining a class
class Product {
    id: number;
    name: string;
    price: number;

    constructor(id: number, name: string, price: number) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    getDiscountedPrice(discount: number): number {
        return this.price * (1 - discount);
    }
}

// Extending a class
class Electronics extends Product {
    warrantyPeriod: number;

    constructor(id: number, name: string, price: number, warrantyPeriod: number) {
        super(id, name, price);
        this.warrantyPeriod = warrantyPeriod;
    }

    getWarrantyDetails(): string {
        return `Warranty Period: ${this.warrantyPeriod} years`;
    }
}

// Using the class
const laptop = new Electronics(1, "Laptop", 1000, 2);
console.log(laptop.getDiscountedPrice(0.1)); // Output: 900
console.log(laptop.getWarrantyDetails()); // Output: Warranty Period: 2 years
```

#### **Pros:**
- **Implementation:** Classes can contain both properties and methods, allowing for full implementation of behavior.
- **Inheritance:** Classes can be extended, allowing for code reuse and hierarchical relationships.
- **Encapsulation:** Classes support encapsulation through access modifiers like `private`, `protected`, and `public`.

#### **Cons:**
- **Complexity:** Classes can become complex, especially when dealing with inheritance and polymorphism.
- **Performance:** Classes can introduce some overhead compared to plain objects, especially in scenarios where you don't need the full OOP features.

### 3. **Type**

A `type` in TypeScript is a way to create a new name for a type. It can be used to alias existing types, combine types, or create complex types. Unlike interfaces, types can represent primitive types, union types, and intersection types.

#### **Sample Code:**

```typescript
// Defining a type alias
type Point = {
    x: number;
    y: number;
};

// Using the type alias
const origin: Point = { x: 0, y: 0 };

// Defining a union type
type Status = "active" | "inactive" | "pending";

// Using the union type
let userStatus: Status = "active";

// Defining an intersection type
type User = {
    id: number;
    name: string;
};

type Admin = {
    permissions: string[];
};

type AdminUser = User & Admin;

// Using the intersection type
const admin: AdminUser = {
    id: 1,
    name: "Admin",
    permissions: ["read", "write"],
};
```

#### **Pros:**
- **Flexibility:** Types are very flexible and can be used to create complex types, including unions, intersections, and primitive types.
- **Alias Existing Types:** Types can be used to alias existing types, making code more readable and maintainable.
- **No Declaration Merging:** Unlike interfaces, types do not support declaration merging, which can be a pro in scenarios where you want to avoid accidental type extensions.

#### **Cons:**
- **No Extensibility:** Types cannot be extended or merged like interfaces. If you need to extend a type, you need to create a new type that includes the original type.
- **Less Readable for Complex Objects:** For complex object shapes, interfaces are often more readable and easier to understand than types.

### **Summary**

- **Interface:** Best for defining contracts for object shapes, especially when you want to enforce a structure across multiple classes or objects.
- **Class:** Best for defining both the structure and behavior of objects, especially when you need inheritance, encapsulation, and polymorphism.
- **Type:** Best for creating complex types, including unions, intersections, and aliases for existing types.

Each of these constructs has its own strengths and weaknesses, and the choice between them depends on the specific requirements of your project.

### **Comparison: `interface`, `type`, and `class`**

| **Feature**                     | **interface**                                                                                 | **type**                                                                                        | **class**                                                                                       |
|----------------------------------|----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| **Object shapes**                | ✅ Yes                                                                                       | ✅ Yes                                                                                        | ✅ Yes                                                                                        |
| **Unions/Intersections**         | ❌ Limited                                                                                   | ✅ Fully supported                                                                           | ❌ Limited                                                                                   |
| **Extending existing types**     | ✅ Can extend multiple interfaces                                                             | ✅ Supports unions and intersections for combining types                                      | ✅ Can extend other classes and implement interfaces                                          |
| **Declaration merging**          | ✅ Yes (interfaces with the same name merge automatically)                                   | ❌ No                                                                                         | ❌ No                                                                                         |
| **Runtime presence**             | ❌ Does not exist in runtime                                                                  | ❌ Does not exist in runtime                                                                  | ✅ Yes (exists in runtime as JavaScript classes)                                              |
| **Instantiable (e.g., via `new`)**| ❌ No                                                                                       | ❌ No                                                                                         | ✅ Yes                                                                                        |
| **Implementation**               | ❌ Cannot contain implementation details                                                      | ❌ Cannot contain implementation details                                                      | ✅ Can contain implementation details (methods, properties, etc.)                             |
| **Inheritance**                  | ❌ No                                                                                       | ❌ No                                                                                         | ✅ Supports inheritance using `extends`                                                        |
| **Encapsulation**                | ❌ No                                                                                       | ❌ No                                                                                         | ✅ Supports encapsulation through access modifiers (`private`, `protected`, `public`)          |
| **Polymorphism**                 | ❌ No                                                                                       | ❌ No                                                                                         | ✅ Supports polymorphism through method overriding and interface implementation               |
| **Performance**                  | ✅ Lightweight (only type checking)                                                          | ✅ Lightweight (only type checking)                                                          | ❌ Slightly heavier due to runtime presence and additional features (inheritance, encapsulation) |
| **Readability**                  | ✅ Often more readable for complex object shapes                                              | ❌ Can become less readable for complex object shapes                                         | ✅ Often more readable for complex object shapes with behavior                                |
| **Use Case**                     | Best for defining contracts for object shapes, especially when enforcing structure across multiple classes/objects | Best for creating complex types, including unions, intersections, and aliases for existing types | Best for defining both the structure and behavior of objects, especially when you need inheritance, encapsulation, and polymorphism |

---
