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

One-shot : [https://www.youtube.com/watch?v=OuNOyFg942M](https://www.youtube.com/watch?v=OuNOyFg942M)


---

## 1. Factory Method Pattern

- **GeeksforGeeks**: [Factory Method Design Pattern](https://www.geeksforgeeks.org/system-design/factory-method-for-designing-pattern/)
- **Refactoring Guru**: [Factory Method](https://refactoring.guru/design-patterns/factory-method)
- **Factory Method in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=dMK4TbG29fk)

> **Define an interface for creating an object, but let subclasses decide which class to instantiate.**

---

## 2. Abstract Factory Pattern

- **GeeksforGeeks**: [Abstract Factory Pattern](https://www.geeksforgeeks.org/system-design/abstract-factory-pattern/)
- **Refactoring Guru**: [Abstract Factory](https://sourcemaking.com/design_patterns/abstract_factory)
- **Abstract Factory in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=PpKvPrl_gRg)

> **Provide an interface for creating families of related or dependent objects without specifying their concrete classes.**

---

## 3. Builder Pattern

- **GeeksforGeeks**: [Builder Design Pattern](https://www.geeksforgeeks.org/system-design/builder-design-pattern/)
- **Refactoring Guru**: [Builder Pattern](https://refactoring.guru/design-patterns/builder)
- **Builder Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=PpKvPrl_gRg)

> **Separate the construction of a complex object from its representation so that the same construction process can create different representations.**

---

## 4. Prototype Pattern

- **GeeksforGeeks**: [Prototype Design Pattern](https://www.geeksforgeeks.org/system-design/prototype-design-pattern/)
- **Refactoring Guru**: [Prototype Pattern](https://refactoring.guru/design-patterns/prototype)
- **Prototype Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=PpKvPrl_gRg)

> **Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.**

---

## 5. Singleton Pattern

- **GeeksforGeeks**: [Singleton Design Pattern](https://www.geeksforgeeks.org/system-design/singleton-design-pattern-introduction/)
- **Refactoring Guru**: [Singleton Pattern](https://refactoring.guru/design-patterns/singleton)
- **Youtube - Singleton Pattern**: [YouTube](https://www.youtube.com/watch?v=tSZn4wkBIu8)

> **Ensure a class has only one instance and provide a global point of access to it.**
