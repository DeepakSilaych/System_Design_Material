# SOLID Principles

# Resources

- **SOLID Principles with Real-life Examples**: [https://www.geeksforgeeks.org/system-design/solid-principle-in-programming-understand-with-real-life-examples/](https://www.geeksforgeeks.org/system-design/solid-principle-in-programming-understand-with-real-life-examples/)

### Videos

#### Hindi
- **Part 1**: [https://www.youtube.com/watch?v=UsNl8kcU4UA](https://www.youtube.com/watch?v=UsNl8kcU4UA)
- **Part 2**: [https://www.youtube.com/watch?v=hU9koy6A2I0](https://www.youtube.com/watch?v=hU9koy6A2I0)

#### English
- **SOLID Principles Explained**: [https://www.youtube.com/watch?v=pTB30aXS77U](https://www.youtube.com/watch?v=pTB30aXS77U)

---
<br><br>

# Revision Questions

### 1. What is SOLID? Why is it important?
**SOLID** is an acronym for five fundamental principles of object-oriented design introduced by Robert C. Martin (Uncle Bob). These principles help developers create software that is:
- Easy to maintain
- Easy to extend
- Flexible and scalable
- Less prone to bugs when changes are made

The five principles are:
1. **S** – Single Responsibility Principle (SRP)
2. **O** – Open/Closed Principle (OCP)
3. **L** – Liskov Substitution Principle (LSP)
4. **I** – Interface Segregation Principle (ISP)
5. **D** – Dependency Inversion Principle (DIP)

Following SOLID makes code **cleaner**, **more readable**, and **resilient to change**.

---

### 2. What is the Single Responsibility Principle (SRP)?
> **"A class should have only one reason to change."**

This means a class should do **only one job** or have **only one responsibility**.

#### Real-Life Example:
Think of a **Swiss Army Knife** – it has many tools, but if one breaks, you replace the whole thing.  
Better: Use **separate tools** (knife, screwdriver, scissors) – each does one job well.

#### Bad Example (Violation):
```cpp
class Employee {
public:
    void calculatePay() { /* ... */ }
    void saveToDatabase() { /* ... */ }
    void generateReport() { /* ... */ }
};
```
This class has **3 responsibilities** → hard to maintain.

#### Good Example (SRP):
```cpp
class Employee { /* only data & pay logic */ };

class EmployeeRepository {
public:
    void save(const Employee& e);
};

class ReportGenerator {
public:
    void generate(const Employee& e);
};
```

---

### 3. What is the Open/Closed Principle (OCP)?
> **"Software entities should be open for extension but closed for modification."**

You should be able to **add new functionality** without changing existing code.

#### Real-Life Example:
Think of a **remote control**. You can add new buttons (extend), but you don’t open the remote to rewire it (modify).

#### Bad Example (Violation):
```cpp
class AreaCalculator {
public:
    double calculateArea(void* shape) {
        if (/* is Circle */) { /* ... */ }
        if (/* is Square */) { /* ... */ }
        // Need to modify this class for every new shape
    }
};
```

#### Good Example (OCP):
```cpp
class Shape {
public:
    virtual double calculateArea() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
public:
    double calculateArea() const override { /* ... */ }
};

class Square : public Shape {
public:
    double calculateArea() const override { /* ... */ }
};

// Calculator doesn't change when new shapes are added
class AreaCalculator {
public:
    double calculate(const Shape& shape) const {
        return shape.calculateArea();
    }
};
```

---

### 4. What is the Liskov Substitution Principle (LSP)?
> **"Subtypes must be substitutable for their base types."**

If class `B` inherits from `A`, you should be able to use `B` wherever `A` is expected — **without breaking the program**.

#### Real-Life Example:
A **Penguin** is a bird, but it **cannot fly**. If your `Bird` class has a `fly()` method, replacing a `Sparrow` with a `Penguin` will fail.

#### Bad Example (Violation):
```cpp
class Bird {
public:
    virtual void fly() { /* ... */ }
};

class Penguin : public Bird {
public:
    void fly() override { throw std::runtime_error("Can't fly!"); }
};
```

#### Good Example (LSP):
```cpp
class Flyable {
public:
    virtual void fly() = 0;
    virtual ~Flyable() = default;
};

class Sparrow : public Flyable {
public:
    void fly() override { /* ... */ }
};

class Ostrich { }; // Doesn't implement Flyable
```

---

### 5. What is the Interface Segregation Principle (ISP)?
> **"Clients should not be forced to depend on interfaces they do not use."**

Don’t create **fat interfaces**. Split large interfaces into smaller, specific ones.

#### Real-Life Example:
A **printer** shouldn’t be forced to implement `fax()` if it only prints.

#### Bad Example (Violation):
```cpp
class Worker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
};

class Robot : public Worker {
public:
    void work() override { /* ... */ }
    void eat() override { }  // Robots don’t eat!
    void sleep() override { }
};
```

#### Good Example (ISP):
```cpp
class Workable {
public:
    virtual void work() = 0;
    virtual ~Workable() = default;
};

class Eatable {
public:
    virtual void eat() = 0;
    virtual ~Eatable() = default;
};

class Human : public Workable, public Eatable {
public:
    void work() override { /* ... */ }
    void eat() override { /* ... */ }
};

class Robot : public Workable {
public:
    void work() override { /* ... */ }
}; // Only what it needs
```

---

### 6. What is the Dependency Inversion Principle (DIP)?
> **"High-level modules should not depend on low-level modules. Both should depend on abstractions."**

Depend on **interfaces**, not concrete classes.

#### Real-Life Example:
A **light bulb** doesn’t care if the switch is from Philips or Havells — as long as it follows the standard interface.

#### Bad Example (Violation):
```cpp
class Keyboard { /* ... */ };

class Windows {
private:
    Keyboard keyboard;  // Tight coupling
public:
    Windows() = default;
};
```

#### Good Example (DIP):
```cpp
class IKeyboard {
public:
    virtual void type() = 0;
    virtual ~IKeyboard() = default;
};

class MechanicalKeyboard : public IKeyboard {
public:
    void type() override { /* ... */ }
};

class Windows {
private:
    std::unique_ptr<IKeyboard> keyboard;
public:
    Windows(std::unique_ptr<IKeyboard> k) : keyboard(std::move(k)) {}
    void typeText() { keyboard->type(); }
};
```

---

### 7. How do SOLID principles relate to clean code?
| Principle | Clean Code Benefit |
|---------|------------------|
| **SRP** | Each class has clear purpose → easier to read |
| **OCP** | Add features without touching old code |
| **LSP** | Predictable behavior in inheritance |
| **ISP** | No unused methods → less confusion |
| **DIP** | Loose coupling → easier testing & reuse |

---

### 8. Can you give a real-world analogy for all SOLID principles?

| Principle | Real-World Analogy |
|---------|------------------|
| **S** | A chef only cooks — doesn’t wash dishes or serve |
| **O** | LEGO blocks — add new pieces without breaking old structure |
| **L** | Electric plugs — any device works if it fits the socket |
| **I** | Restaurant menu — order only what you want |
| **D** | USB devices — laptop doesn’t care about brand, just interface |

---

### 9. What happens if you violate SOLID principles?
- **Code becomes rigid** – hard to change
- **Fragile** – changes break unrelated parts
- **Immobile** – hard to reuse in other projects
- **High maintenance cost**
- **Spaghetti code over time**

---

### 10. Are SOLID principles only for OOP?
No. While designed for **object-oriented programming**, the ideas apply to:
- Functional programming (pure functions → SRP)
- Microservices (single responsibility services)
- API design (interface segregation)

---

### 11. How to remember SOLID easily?
```
S – Single Job
O – Open to Extend, Closed to Modify
L – Like Parent, Child Should Behave
I – Interfaces Should Be Small & Specific
D – Depend on Interface, Not Implementation
```

Or use the **mnemonic**:  
**"SOLID = Strong, Organized, Logical, Intelligent Design"**

---

### 12. Can you show a before/after refactoring using SOLID?

#### Before (All violations):
```cpp
class Invoice {
public:
    double calculateTotal() { /* ... */ }
    void printInvoice() { /* ... */ }
    void saveToFile() { /* ... */ }
    void sendEmail() { /* ... */ }
};
```

#### After (SOLID applied):
```cpp
class Invoice {
public:
    double calculateTotal() const;
};

class IPrintable {
public:
    virtual void print(const Invoice&) = 0;
    virtual ~IPrintable() = default;
};

class ISavable {
public:
    virtual void save(const Invoice&) = 0;
    virtual ~ISavable() = default;
};

class IEmailable {
public:
    virtual void sendEmail(const Invoice&) = 0;
    virtual ~IEmailable() = default;
};

class InvoicePrinter : public IPrintable {
public:
    void print(const Invoice&) override;
};

class InvoiceRepository : public ISavable {
public:
    void save(const Invoice&) override;
};

class EmailService : public IEmailable {
public:
    void sendEmail(const Invoice&) override;
};
```

Now each class has **one job**, is **extendable**, and **replaceable**.

---

**Summary**:  
**SOLID is not a rule — it’s a guideline** to write **maintainable, scalable, and testable** code. Master it to become a better software engineer.
