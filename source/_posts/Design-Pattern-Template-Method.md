---
title: Design Pattern - Template Method
date: 2024-11-04 23:06:27
categories:
- Design Pattern
- Behavioral 
tags:
- Design Pattern
- Behavioral 
---

- [Introduction](#introduction)
- [Template Method Design Pattern Basics](#template-method-design-pattern-basics)
  - [2.1 What is the Template Method Pattern?](#21-what-is-the-template-method-pattern)
  - [2.2 Key Components](#22-key-components)
- [Design Diagrams](#design-diagrams)
  - [3.1 Component Diagram](#31-component-diagram)
  - [3.2 Class Diagram](#32-class-diagram)
  - [Explanation:](#explanation)
- [4. Implementation Sample](#4-implementation-sample)
  - [4.1 Abstract Class](#41-abstract-class)
  - [4.2 Concrete Classes](#42-concrete-classes)
  - [4.3 Client Code](#43-client-code)
  - [Output](#output)
- [5. Implementation Sample II](#5-implementation-sample-ii)
  - [5.1 Abstract Class](#51-abstract-class)
  - [5.2 Concrete Classes](#52-concrete-classes)
  - [5.3 Client Code](#53-client-code)
  - [Output:](#output-1)
- [6. Scenarios](#6-scenarios)
  - [5.1 Scenario 1: Document Processing](#51-scenario-1-document-processing)
    - [Abstract Class](#abstract-class)
    - [Concrete Classes](#concrete-classes)
    - [Client Code](#client-code)
  - [Output](#output-2)
  - [5.2 Scenario 2: Game Development](#52-scenario-2-game-development)
    - [Abstract Class](#abstract-class-1)
    - [Concrete Classes](#concrete-classes-1)
    - [Client Code](#client-code-1)
  - [Output](#output-3)
  - [5.3 Scenario 3: ETL Data Pipeline](#53-scenario-3-etl-data-pipeline)
- [7. PROS and CONS](#7-pros-and-cons)
  - [PROS:](#pros)
  - [CONS:](#cons)
- [8. Conclusion](#8-conclusion)
- [References](#references)

<a name="introduction"></a>
## Introduction

The Template Method design pattern is a behavioral design pattern that defines the skeleton of an algorithm in a method, deferring some steps to subclasses. It lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure. This pattern is particularly useful when you want to allow customization of parts of an algorithm while keeping the overall structure intact.

This article will deep dive into the Template Method design pattern, including its key components, design diagrams, sample code in Java, and real production scenarios.

<a name="template-method-design-pattern-basics"></a>
## Template Method Design Pattern Basics

<a name="21-what-is-the-template-method-pattern"></a>
### 2.1 What is the Template Method Pattern?

The Template Method pattern is a behavioral design pattern that defines the skeleton of an algorithm in a method, deferring some steps to subclasses. It lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

<a name="22-key-components"></a>
### 2.2 Key Components

- **Abstract Class**: Defines the template method and abstract methods that represent the steps of the algorithm.
- **Concrete Class**: Extends the abstract class and implements the abstract methods to provide specific behavior.

<a name="design-diagrams"></a>
## Design Diagrams

<a name="31-component-diagram"></a>
### 3.1 Component Diagram


```
[Template Method Pattern]
+----------------------------+
|   AbstractClass            |
|  -----------------------   |
|  + templateMethod()        |
+----------------------------+
           |
           +---------------+
           |               |
   +----------------+   +----------------+
   | ConcreteClassA |   | ConcreteClassB |
   +----------------+   +----------------+
```


```
+-------------------+       +-------------------+       +-------------------+
| Client            |       | Abstract Class    |       | Concrete Class    |
+-------------------+       +-------------------+       +-------------------+
        |                            |                           |
        | (1) Use Template Method    |                           |
        |--------------------------->|                           |
        |                            | (2) Call Abstract Methods |
        |                            |-------------------------->|
        |                            |                           |
        | (3) Execute Algorithm      |                           |
        |<---------------------------|                           |
        |                            |                           |
+-------------------+       +-------------------+       +-------------------+
| Client            |       | Abstract Class    |       | Concrete Class    |
+-------------------+       +-------------------+       +-------------------+
```

<a name="32-class-diagram"></a>
### 3.2 Class Diagram

```
+-------------------+       +-------------------+       +-------------------+
| AbstractClass     |       | ConcreteClassA    |       | ConcreteClassB    |
+-------------------+       +-------------------+       +-------------------+
| +templateMethod() |<------| +step1()          |       | +step1()          |
| +step1()          |       | +step2()          |       | +step2()          |
| +step2()          |       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+
```

### Explanation:
- **AbstractClass**: Defines the `templateMethod()` and other concrete or abstract methods.
- **ConcreteClassA/B**: Implements the specific steps defined as abstract in the base class.

---

<a name="4-implementation-sample"></a>
## 4. Implementation Sample

<a name="41-abstract-class"></a>
### 4.1 Abstract Class

The abstract class defines the template method and the abstract methods that represent the steps of the algorithm.

```java
abstract class AbstractClass {

    // Template method defines the skeleton of an algorithm
    public final void templateMethod() {
        step1();
        step2();
        step3();
    }

    // Abstract methods to be implemented by subclasses
    protected abstract void step1();
    protected abstract void step2();

    // Hook method with default implementation
    protected void step3() {
        System.out.println("Default implementation of step3");
    }
}
```

<a name="42-concrete-classes"></a>
### 4.2 Concrete Classes

Concrete classes extend the abstract class and implement the abstract methods to provide specific behavior.

```java
class ConcreteClassA extends AbstractClass {

    @Override
    protected void step1() {
        System.out.println("ConcreteClassA: Step 1");
    }

    @Override
    protected void step2() {
        System.out.println("ConcreteClassA: Step 2");
    }
}

class ConcreteClassB extends AbstractClass {

    @Override
    protected void step1() {
        System.out.println("ConcreteClassB: Step 1");
    }

    @Override
    protected void step2() {
        System.out.println("ConcreteClassB: Step 2");
    }

    @Override
    protected void step3() {
        System.out.println("ConcreteClassB: Custom implementation of step3");
    }
}
```

<a name="43-client-code"></a>
### 4.3 Client Code

The client code uses the template method to execute the algorithm.

```java
public class Client {

    public static void main(String[] args) {
        AbstractClass classA = new ConcreteClassA();
        classA.templateMethod();

        System.out.println();

        AbstractClass classB = new ConcreteClassB();
        classB.templateMethod();
    }
}
```

### Output

```
ConcreteClassA: Step 1
ConcreteClassA: Step 2
Default implementation of step3

ConcreteClassB: Step 1
ConcreteClassB: Step 2
ConcreteClassB: Custom implementation of step3
```

<a name="5-implementation-sample-II"></a>
## 5. Implementation Sample II

<a name="51-abstract-class"></a>
### 5.1 Abstract Class

```java
public abstract class DataProcessor {
    // Template method defining the algorithm
    public final void process() {
        readData();
        processData();
        writeData();
    }

    // Steps to be implemented by subclasses
    protected abstract void readData();
    protected abstract void processData();

    // Concrete method
    protected void writeData() {
        System.out.println("Writing data to output...");
    }
}
```

<a name="52-concrete-classes"></a>
### 5.2 Concrete Classes

```java
public class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from a CSV file...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing CSV data...");
    }
}

public class JSONDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from a JSON file...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing JSON data...");
    }
}
```

<a name="53-client-code"></a>
### 5.3 Client Code


```java
public class Main {
    public static void main(String[] args) {
        DataProcessor csvProcessor = new CSVDataProcessor();
        csvProcessor.process();

        System.out.println();

        DataProcessor jsonProcessor = new JSONDataProcessor();
        jsonProcessor.process();
    }
}
```

### Output:
```
Reading data from a CSV file...
Processing CSV data...
Writing data to output...

Reading data from a JSON file...
Processing JSON data...
Writing data to output...
```

<a name="6-scenarios"></a>
## 6. Scenarios

<a name="51-scenario-1-document-processing"></a>
### 5.1 Scenario 1: Document Processing

In a document processing application, different types of documents (e.g., PDF, Word, Excel) may require different processing steps. The Template Method pattern can be used to define the common processing steps while allowing subclasses to implement document-specific steps.

#### Abstract Class

```java
abstract class DocumentProcessor {

    // Template method defines the skeleton of the document processing algorithm
    public final void processDocument() {
        openDocument();
        extractContent();
        saveDocument();
    }

    // Abstract methods to be implemented by subclasses
    protected abstract void openDocument();
    protected abstract void extractContent();

    // Hook method with default implementation
    protected void saveDocument() {
        System.out.println("Saving document to default location");
    }
}
```

#### Concrete Classes

```java
class PdfDocumentProcessor extends DocumentProcessor {

    @Override
    protected void openDocument() {
        System.out.println("Opening PDF document");
    }

    @Override
    protected void extractContent() {
        System.out.println("Extracting text and images from PDF");
    }
}

class WordDocumentProcessor extends DocumentProcessor {

    @Override
    protected void openDocument() {
        System.out.println("Opening Word document");
    }

    @Override
    protected void extractContent() {
        System.out.println("Extracting text and tables from Word");
    }

    @Override
    protected void saveDocument() {
        System.out.println("Saving Word document to custom location");
    }
}
```

#### Client Code

```java
public class DocumentProcessingClient {

    public static void main(String[] args) {
        DocumentProcessor pdfProcessor = new PdfDocumentProcessor();
        pdfProcessor.processDocument();

        System.out.println();

        DocumentProcessor wordProcessor = new WordDocumentProcessor();
        wordProcessor.processDocument();
    }
}
```

### Output

```
Opening PDF document
Extracting text and images from PDF
Saving document to default location

Opening Word document
Extracting text and tables from Word
Saving Word document to custom location
```

<a name="52-scenario-2-game-development"></a>
### 5.2 Scenario 2: Game Development

In game development, different types of games (e.g., RPG, Strategy, Puzzle) may have common game loop steps (e.g., initialize, update, render) but require different implementations for each step. The Template Method pattern can be used to define the common game loop while allowing subclasses to implement game-specific steps.

#### Abstract Class

```java
abstract class Game {

    // Template method defines the skeleton of the game loop
    public final void runGame() {
        initialize();
        while (!isGameOver()) {
            update();
            render();
        }
        cleanup();
    }

    // Abstract methods to be implemented by subclasses
    protected abstract void initialize();
    protected abstract void update();
    protected abstract void render();

    // Hook method with default implementation
    protected boolean isGameOver() {
        return false;
    }

    // Hook method with default implementation
    protected void cleanup() {
        System.out.println("Cleaning up game resources");
    }
}
```

#### Concrete Classes

```java
class RPGGame extends Game {

    @Override
    protected void initialize() {
        System.out.println("Initializing RPG game");
    }

    @Override
    protected void update() {
        System.out.println("Updating RPG game state");
    }

    @Override
    protected void render() {
        System.out.println("Rendering RPG game graphics");
    }

    @Override
    protected boolean isGameOver() {
        // Custom game over condition for RPG
        return false;
    }
}

class PuzzleGame extends Game {

    @Override
    protected void initialize() {
        System.out.println("Initializing Puzzle game");
    }

    @Override
    protected void update() {
        System.out.println("Updating Puzzle game state");
    }

    @Override
    protected void render() {
        System.out.println("Rendering Puzzle game graphics");
    }

    @Override
    protected boolean isGameOver() {
        // Custom game over condition for Puzzle
        return false;
    }

    @Override
    protected void cleanup() {
        System.out.println("Cleaning up Puzzle game resources");
    }
}
```

#### Client Code

```java
public class GameClient {

    public static void main(String[] args) {
        Game rpgGame = new RPGGame();
        rpgGame.runGame();

        System.out.println();

        Game puzzleGame = new PuzzleGame();
        puzzleGame.runGame();
    }
}
```

### Output

```
Initializing RPG game
Updating RPG game state
Rendering RPG game graphics
Updating RPG game state
Rendering RPG game graphics
...
Cleaning up game resources

Initializing Puzzle game
Updating Puzzle game state
Rendering Puzzle game graphics
Updating Puzzle game state
Rendering Puzzle game graphics
...
Cleaning up Puzzle game resources
```

<a name="53-scenario-3-etl-data-pipeline"></a>
### 5.3 Scenario 3: ETL Data Pipeline
In a production ETL (Extract, Transform, Load) data pipeline, different types of data sources (CSV, JSON, XML, databases) need to be processed uniformly but with some custom parsing or transformation logic. Using the Template Method pattern, you can create a base `ETLProcessor` class with a template for the steps (`extract()`, `transform()`, `load()`), and concrete classes implement the details for different data formats.


```java
// Abstract Class
public abstract class ETLProcessor {
    public final void runETL() {
        extract();
        transform();
        load();
    }

    protected abstract void extract();
    protected abstract void transform();

    protected void load() {
        System.out.println("Loading data into the target system...");
    }
}

// Concrete Classes
public class CSVETLProcessor extends ETLProcessor {
    @Override
    protected void extract() {
        System.out.println("Extracting data from CSV...");
    }

    @Override
    protected void transform() {
        System.out.println("Transforming CSV data...");
    }
}

// Concrete Classes
public class DatabaseETLProcessor extends ETLProcessor {
    @Override
    protected void extract() {
        System.out.println("Extracting data from Database...");
    }

    @Override
    protected void transform() {
        System.out.println("Transforming database records...");
    }
}
```

>>**Consistency**: Ensures a consistent ETL workflow across data sources.
>>**Customization**: Allows specific extraction and transformation logic for each data source while reusing the common `load()` step.

---

<a name="7-pros-cons"></a>
## 7. PROS and CONS

### PROS:
- **Code Reusability**: Common parts of the algorithm are implemented in the base class.
- **Enforces Structure**: Ensures that subclasses follow the defined structure.
- **Customizability**: Subclasses can implement specific steps as needed.

### CONS:
- **Rigid Structure**: Changes to the base template can affect all subclasses.
- **Overuse**: Overusing the pattern can lead to an overly complex class hierarchy.

---

<a name="8-conclusion"></a>
## 8. Conclusion

The Template Method design pattern is a powerful tool for defining the skeleton of an algorithm while allowing subclasses to provide specific implementations for certain steps. This pattern promotes code reuse and flexibility, enabling you to create extensible and maintainable software.

By understanding the key components, design diagrams, sample code, and real production scenarios provided in this article, you can effectively apply the Template Method pattern in your Java projects.

<a name="references"></a>
## References

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
- [Template Method Pattern - Wikipedia](https://en.wikipedia.org/wiki/Template_method_pattern)

---

This article provides a comprehensive guide to the Template Method design pattern, including detailed explanations, design diagrams, sample code in Java, and real production scenarios. By understanding and applying this pattern, developers can create flexible and maintainable software systems.