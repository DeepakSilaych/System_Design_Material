# Behavioral Design Patterns

Behavioral design patterns focus on the interactions and communication between objects, defining how they collaborate and distribute responsibilities.

**One Shot Video**: [Behavioral Design Patterns in One Shot](https://www.youtube.com/watch?v=DBDnUkTobaE)

## Summary

Behavioral design patterns focus on the communication and interaction between objects, defining how they collaborate to achieve specific tasks. These patterns help in managing complex control flows and responsibilities among objects.

### Overview of Behavioral Patterns

1. **Chain of Responsibility** - Allows a request to pass through a chain of handlers, where each handler decides either to process the request or pass it to the next handler
2. **Command** - Encapsulates a request as an object, allowing for parameterization of clients with different requests
3. **Interpreter** - Defines a grammatical representation for a language and provides an interpreter to deal with this grammar
4. **Mediator** - Defines an object that encapsulates how a set of objects interact, promoting loose coupling
5. **Memento** - Captures and externalizes an object's internal state so that it can be restored later
6. **Observer** - Establishes a one-to-many dependency between objects so that when one object changes state, all its dependents are notified
7. **State** - Allows an object to alter its behavior when its internal state changes
8. **Strategy** - Defines a family of algorithms, encapsulates each one, and makes them interchangeable
9. **Template Method** - Defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps
10. **Visitor** - Represents an operation to be performed on elements of an object structure

---

## 1. Chain of Responsibility Pattern

- **GeeksforGeeks**: [Chain of Responsibility Design Pattern](https://www.geeksforgeeks.org/system-design/chain-responsibility-design-pattern/)
- **Chain of Responsibility Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=LXVKB6deQMo)

> **Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.**

---

## 2. Command Pattern

- **GeeksforGeeks**: [Command Design Pattern](https://www.geeksforgeeks.org/system-design/command-pattern/)
- **Command Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=cnQZsN0jxEY)

> **Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.**

---

## 3. Interpreter Pattern

- **GeeksforGeeks**: [Interpreter Design Pattern](https://www.geeksforgeeks.org/system-design/interpreter-design-pattern/)
- **Interpreter Pattern in Python (Hindi)**: Coming soon on the [System Design Playlist](https://www.youtube.com/playlist?list=PLQEaRBV9gAFvzp6XhcNFpk1WdOcyVo9qT)

> **Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.**

---

## 4. Mediator Pattern

- **GeeksforGeeks**: [Mediator Design Pattern](https://www.geeksforgeeks.org/system-design/mediator-design-pattern/)
- **Mediator Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=3lGIICzgyQQ)

> **Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.**

---

## 5. Memento Pattern

- **GeeksforGeeks**: [Memento Design Pattern](https://www.geeksforgeeks.org/system-design/memento-design-pattern/)
- **Memento Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=p8-ile_nWnY)

> **Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.**

---

## 6. Observer Pattern

- **GeeksforGeeks**: [Observer Design Pattern](https://www.geeksforgeeks.org/system-design/observer-pattern-set-1-introduction/)
- **Observer Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=Jpmp4GY8r3Q)

> **Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.**

---

## 7. State Pattern

- **GeeksforGeeks**: [State Design Pattern](https://www.geeksforgeeks.org/system-design/state-design-pattern/)
- **State Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=bJPmvie_p4w)

> **Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.**

---

## 8. Strategy Pattern

- **GeeksforGeeks**: [Strategy Pattern](https://www.geeksforgeeks.org/system-design/strategy-pattern-set-1/)
- **Strategy Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=PpKvPrl_gRg)

> **Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.**

---

## 9. Template Method Pattern

- **GeeksforGeeks**: [Template Method Design Pattern](https://www.geeksforgeeks.org/system-design/template-method-design-pattern/)
- **Template Method Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=8-vE_bmEt18)

> **Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.**

---

## 10. Visitor Pattern

- **GeeksforGeeks**: [Visitor Design Pattern](https://www.geeksforgeeks.org/system-design/visitor-design-pattern/)
- **Visitor Pattern in Python (Hindi)**: [YouTube](https://www.youtube.com/watch?v=DnmsxnlCyl0)

> **Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.**
