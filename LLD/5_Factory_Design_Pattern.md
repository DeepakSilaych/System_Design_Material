# Factory Method Design Pattern Interview Questions

- **Refactoring Guru – Factory Method**: [https://refactoring.guru/design-patterns/factory-method](https://refactoring.guru/design-patterns/factory-method)
- **Factory Method in Python (Hindi)**: [https://www.youtube.com/watch?v=dMK4TbG29fk](https://www.youtube.com/watch?v=dMK4TbG29fk)

---

## Revision Questions

### 1. What is the Factory Method Pattern?
> **Define an interface for creating an object, but let subclasses decide which class to instantiate.**

The **Factory Method** allows a class to **defer instantiation** to its **subclasses**.

It’s a **creational pattern** that promotes:
- **Loose coupling**
- **Open/Closed Principle**
- **Extensibility**

---

### 2. Why use Factory Method?

| Benefit | Explanation |
|-------|-------------|
| **Avoid tight coupling** | Client doesn’t need to know concrete classes |
| **Open for extension** | Add new product types without changing client |
| **Encapsulate object creation** | Creation logic in one place |
| **Better testability** | Mock factories easily |
| **Supports polymorphism** | Return different objects via same interface |

---

### 3. When to use Factory Method?

Use it when:
- A class **cannot anticipate** the type of objects it needs to create
- You want to **localize** object creation logic
- You want **subclasses** to decide what to instantiate
- You’re building a **framework** or **library**

#### Real-Life Example:
A **UI framework** supports:
- Windows buttons
- macOS buttons
- Linux buttons

Each platform creates its own button via a **factory method**.

---

### 4. Structure of Factory Method

```text
Client
   |
Creator (abstract)
   ├── factory_method() → returns Product
   │
   ├── ConcreteCreatorA → overrides factory_method() → returns ConcreteProductA
   └── ConcreteCreatorB → overrides factory_method() → returns ConcreteProductB

Product (interface)
   ├── ConcreteProductA
   └── ConcreteProductB
```

---

### 5. Python Example: Cross-Platform UI

#### Bad Way (Without Factory)
```python
class Button:
    def render(self):
        pass

class WindowsButton(Button):
    def render(self):
        return "Rendering Windows button"

class MacButton(Button):
    def render(self):
        return "Rendering macOS button"

# Client is tightly coupled
def create_button(os_type):
    if os_type == "windows":
        return WindowsButton()
    elif os_type == "mac":
        return MacButton()
    else:
        raise ValueError("Unknown OS")
```

Problem: Client must know **all concrete classes** → hard to extend.

---

#### Good Way (With Factory Method)

```python
from abc import ABC, abstractmethod

# Product Interface
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

# Concrete Products
class WindowsButton(Button):
    def render(self) -> str:
        return "Rendering button in Windows style"

class MacButton(Button):
    def render(self) -> str:
        return "Rendering button in macOS style"

# Creator (Factory) Interface
class Dialog(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass

    def render(self):
        button = self.create_button()  # Factory method
        return f"Dialog renders: {button.render()}"

# Concrete Creators
class WindowsDialog(Dialog):
    def create_button(self) -> Button:
        return WindowsButton()

class MacDialog(Dialog):
    def create_button(self) -> Button:
        return MacButton()
```

#### Usage:
```python
def main():
    os_type = "windows"  # Could come from config

    if os_type == "windows":
        dialog = WindowsDialog()
    elif os_type == "mac":
        dialog = MacDialog()
    else:
        raise ValueError("Unknown OS")

    print(dialog.render())

main()
# Output: Dialog renders: Rendering button in Windows style
```

---

### 6. Factory Method vs Simple Factory – What’s the difference?

| Feature | Simple Factory | Factory Method |
|-------|----------------|----------------|
| **Pattern Type** | Not a GoF pattern | GoF Creational Pattern |
| **Extensibility** | Add new products → modify factory | Add new creator subclass |
| **Uses inheritance?** | No | Yes |
| **Example** | `ButtonFactory.create(type)` | `WindowsDialog.create_button()` |

> **Simple Factory** = static method  
> **Factory Method** = virtual method in hierarchy

---

### 7. Can Factory Method be used with Dependency Injection?

**Yes!** Combine with config or DI container.

```python
class Application:
    def __init__(self, dialog_factory):
        self.dialog_factory = dialog_factory

    def run(self):
        dialog = self.dialog_factory()
        print(dialog.render())

# Inject factory
app = Application(WindowsDialog)
app.run()
```

---

### 8. Real-World Analogy

| Component | Real Life |
|---------|----------|
| **Creator** | A **car factory manager** |
| **Concrete Creator** | Toyota vs Ford plant |
| **Factory Method** | `build_car()` |
| **Product** | Car |
| **Concrete Product** | Camry vs Mustang |

Each plant decides **what car** to build.

---

### 9. Common Interview Question: "Refactor using Factory Method"

#### Before (Tight coupling):
```python
class Logger:
    def log(self, msg):
        print(msg)

class FileLogger(Logger):
    def log(self, msg):
        print(f"[FILE] {msg}")

class DatabaseLogger(Logger):
    def log(self, msg):
        print(f"[DB] {msg}")

# Client knows all types
def get_logger(type):
    if type == "file":
        return FileLogger()
    elif type == "db":
        return DatabaseLogger()
```

#### After (Factory Method):
```python
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message: str):
        pass

class FileLogger(Logger):
    def log(self, message: str):
        print(f"[FILE] {message}")

class DatabaseLogger(Logger):
    def log(self, message: str):
        print(f"[DB] {message}")

# Creator
class Application(ABC):
    @abstractmethod
    def create_logger(self) -> Logger:
        pass

    def run(self):
        logger = self.create_logger()
        logger.log("App started")

# Concrete Creators
class WebApp(Application):
    def create_logger(self) -> Logger:
        return FileLogger()

class AdminApp(Application):
    def create_logger(self) -> Logger:
        return DatabaseLogger()

# Usage
app = WebApp()
app.run()  # [FILE] App started
```

---

### 10. Advantages of Factory Method

| Advantage | Description |
|----------|-------------|
| **Decentralized creation** | Each subclass controls its product |
| **Extensible** | Add new types via new subclasses |
| **Supports OCP** | No need to modify existing code |
| **Framework-friendly** | Great for libraries/SDKs |

---

### 11. Disadvantages

| Drawback | Explanation |
|---------|-------------|
| **Class explosion** | One creator per product type |
| **Complexity** | Overkill for simple cases |
| **Inheritance required** | Forces class hierarchy |

---

### 12. Python Tip: Use `@classmethod` for Simple Cases

```python
class Button:
    @classmethod
    def create(cls, os_type):
        if os_type == "windows":
            return WindowsButton()
        elif os_type == "mac":
            return MacButton()
```

But this is **Simple Factory**, not **Factory Method**.

---

### 13. UML Diagram (Text Version)

```text
+----------------+          +--------------------+
|     Client     |          |     Creator        |
+----------------+          +--------------------+
|                |          | + factory_method() |
|                |          +--------------------+
|                |                   /_\
|                |                    |
|                |        +-----------------------+
|                |        |     ConcreteCreator   |
|                |        +-----------------------+
|                |        | + factory_method()    |
|                |        +-----------------------+
|                |
|                |        +--------------------+
|                +------->|      Product       |
|                         +--------------------+
|                         | + operation()      |
|                         +--------------------+
|                                  /_\
|                                   |
|                  +---------------------------------+
|                  |                |                |
|         +----------------+ +----------------+ +----------------+
|         | ConcreteProdA  | | ConcreteProdB  | | ConcreteProdC  |
|         +----------------+ +----------------+ +----------------+
```

---

### 14. Bonus: Combine with Abstract Factory?

Yes! Use **Factory Method** inside **Abstract Factory** for complex object families.

---

**Summary**:  
**Factory Method = Let subclasses decide what to create.**  
Use it when:
- You want **extensibility**
- Creation logic **varies by context**
- You're building **frameworks**

In Python, prefer **ABC + `create_*` method** for clarity.  
Avoid overusing — for 1–2 types, a simple function may suffice.
