# Creational Design Patterns

Creational design patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.

## Summary

Creational design patterns focus on object creation mechanisms, helping to make systems more flexible and reusable. These patterns abstract the instantiation process and help manage object creation complexity.

### Overview of Creational Patterns

1. **Factory Method** - Defers object creation to subclasses, allowing them to decide which class to instantiate
2. **Abstract Factory** - Provides an interface for creating families of related objects without specifying their concrete classes
3. **Builder** - Separates the construction of complex objects from their representation, allowing step-by-step construction
4. **Prototype** - Creates new objects by cloning existing instances, useful when object creation is expensive
5. **Singleton** - Ensures a class has only one instance and provides global access to it

## Bookmarks

- [1. Factory Method Pattern](#1-factory-method-pattern)
  - [What is it?](#1-what-is-the-factory-method-pattern)
  - [Why use it?](#2-why-use-factory-method)
  - [When to use](#3-when-to-use-factory-method)
  - [Structure](#structure-of-factory-method)
  - [Advantages/Disadvantages](#4-advantages-and-disadvantages-of-factory-method)
  - [Python Tip](#5-python-tip-use-classmethod-for-simple-cases)
- [2. Abstract Factory Pattern](#2-abstract-factory-pattern)
  - [2.1 What is it?](#21-what-is-the-abstract-factory-pattern)
  - [2.2 Why use](#22-why-use-abstract-factory)
  - [2.3 When to use](#23-when-to-use-abstract-factory)
  - [2.4 Real-Life Example](#24-real-life-example)
  - [2.5 Structure](#25-structure-of-abstract-factory)
  - [2.6 Python Example](#26-python-example-cross-platform-ui-family)
  - [2.7 AF vs Factory Method](#27-abstract-factory-vs-factory-method--whats-the-difference)
  - [2.8 Advantages/Disadvantages](#28-advantages-and-disadvantages-of-abstract-factory)
- [3. Builder Pattern](#3-builder-pattern)
  - [3.1 What is it?](#31-what-is-the-builder-pattern)
  - [3.2 Why use](#32-why-use-builder)
  - [3.3 When to use](#33-when-to-use-builder)
  - [3.4 Structure](#34-structure-of-builder)
  - [3.5 Python Example](#35-python-example-building-a-computer)
  - [3.6 Builder vs Abstract Factory](#36-builder-vs-abstract-factory)
  - [3.7 Advantages/Disadvantages](#37-advantages-and-disadvantages-of-builder)
- [4. Prototype Pattern](#4-prototype-pattern)
  - [4.1 What is it?](#41-what-is-the-prototype-pattern)
  - [4.2 Why use](#42-why-use-prototype)
  - [4.3 When to use](#43-when-to-use-prototype)
  - [4.4 Structure](#44-structure-of-prototype)
  - [4.5 Python Example](#45-python-example-cloning-with-registry)
  - [4.6 Shallow vs Deep](#46-shallow-vs-deep-copy-in-python)
  - [4.7 Advantages/Disadvantages](#47-advantages-and-disadvantages-of-prototype)
- [5. Singleton Pattern](#5-singleton-pattern)
  - [5.1 What is it?](#51-what-is-the-singleton-pattern)
  - [5.2 When to use](#52-when-and-when-not-to-use-singleton)
  - [5.3 Implementations](#53-python-implementations)
  - [5.4 Thread Safety](#54-thread-safety-and-multiprocessing)
  - [5.5 Advantages/Disadvantages](#55-advantages-and-disadvantages-of-singleton)
  - [5.6 Alternatives](#56-alternatives-to-singleton)

---

## 1. Factory Method Pattern

See detailed documentation: [4.1_Factory_Method_Pattern.md](./4.1_Factory_Method_Pattern.md)

- **Refactoring Guru – Factory Method**: [https://refactoring.guru/design-patterns/factory-method](https://refactoring.guru/design-patterns/factory-method)
- **Factory Method in Python (Hindi)**: [https://www.youtube.com/watch?v=dMK4TbG29fk](https://www.youtube.com/watch?v=dMK4TbG29fk)

**Key Concept**: Define an interface for creating an object, but let subclasses decide which class to instantiate.

---

## 2. Abstract Factory Pattern

See detailed documentation: [4.2_Abstract_Factory_Pattern.md](./4.2_Abstract_Factory_Pattern.md)

- **Refactoring Guru – Abstract Factory**: [https://sourcemaking.com/design_patterns/abstract_factory](https://sourcemaking.com/design_patterns/abstract_factory)
- **Abstract Factory in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

**Key Concept**: Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

---

## 3. Builder Pattern

See detailed documentation: [4.3_Builder_Pattern.md](./4.3_Builder_Pattern.md)

- **Refactoring Guru – Builder Pattern**: [https://refactoring.guru/design-patterns/builder](https://refactoring.guru/design-patterns/builder)
- **Builder Pattern in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

**Key Concept**: Separate the construction of a complex object from its representation so that the same construction process can create different representations.

---

## 4. Prototype Pattern

See detailed documentation: [4.4_Prototype_Pattern.md](./4.4_Prototype_Pattern.md)

- **Refactoring Guru – Prototype Pattern**: [https://refactoring.guru/design-patterns/prototype](https://refactoring.guru/design-patterns/prototype)
- **Prototype Pattern in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

**Key Concept**: Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

---

## 5. Singleton Pattern

See detailed documentation: [4.5_Singleton_Pattern.md](./4.5_Singleton_Pattern.md)

- **Refactoring Guru – Singleton Pattern**: [https://refactoring.guru/design-patterns/singleton](https://refactoring.guru/design-patterns/singleton)
- **Youtube - Singleton Pattern**: https://www.youtube.com/watch?v=tSZn4wkBIu8

**Key Concept**: Ensure a class has only one instance and provide a global point of access to it.

---

## Pattern Comparison

| Pattern              | Use Case                    | Complexity | Flexibility |
| -------------------- | --------------------------- | ---------- | ----------- |
| **Factory Method**   | Single product creation     | Low        | Medium      |
| **Abstract Factory** | Family of related products  | High       | High        |
| **Builder**          | Complex object construction | Medium     | High        |
| **Prototype**        | Expensive object creation   | Low        | Medium      |
| **Singleton**        | Single instance requirement | Low        | Low         |
