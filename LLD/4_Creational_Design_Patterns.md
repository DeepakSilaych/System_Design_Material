# Creational Design Patterns

Creational design patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.

## 1. Factory Method Pattern

- **Refactoring Guru – Factory Method**: [https://refactoring.guru/design-patterns/factory-method](https://refactoring.guru/design-patterns/factory-method)
- **Factory Method in Python (Hindi)**: [https://www.youtube.com/watch?v=dMK4TbG29fk](https://www.youtube.com/watch?v=dMK4TbG29fk)

#### 1. What is the Factory Method Pattern?

> **Define an interface for creating an object, but let subclasses decide which class to instantiate.**

The **Factory Method** allows a class to **defer instantiation** to its **subclasses**.

It's a **creational pattern** that promotes:

- **Loose coupling**
- **Open/Closed Principle**
- **Extensibility**

#### 2. Why use Factory Method?

| Benefit                         | Explanation                                   |
| ------------------------------- | --------------------------------------------- |
| **Avoid tight coupling**        | Client doesn't need to know concrete classes  |
| **Open for extension**          | Add new product types without changing client |
| **Encapsulate object creation** | Creation logic in one place                   |
| **Better testability**          | Mock factories easily                         |
| **Supports polymorphism**       | Return different objects via same interface   |

#### 3. When to use Factory Method?

Use it when:

- A class **cannot anticipate** the type of objects it needs to create
- You want to **localize** object creation logic
- You want **subclasses** to decide what to instantiate
- You're building a **framework** or **library**

#### Real-Life Example:

A **UI framework** supports:

- Windows buttons
- macOS buttons
- Linux buttons

Each platform creates its own button via a **factory method**.

#### Structure of Factory Method

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

#### 4. Advantages and Disadvantages of Factory Method

| Advantage                  | Description                        |
| -------------------------- | ---------------------------------- |
| **Decentralized creation** | Each subclass controls its product |
| **Extensible**             | Add new types via new subclasses   |
| **Supports OCP**           | No need to modify existing code    |
| **Framework-friendly**     | Great for libraries/SDKs           |

| Drawback                 | Explanation                  |
| ------------------------ | ---------------------------- |
| **Class explosion**      | One creator per product type |
| **Complexity**           | Overkill for simple cases    |
| **Inheritance required** | Forces class hierarchy       |

#### 5. Python Tip: Use `@classmethod` for Simple Cases

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

## 2. Abstract Factory Pattern

- **Refactoring Guru – Abstract Factory**: [https://sourcemaking.com/design_patterns/abstract_factory](https://sourcemaking.com/design_patterns/abstract_factory)
- **Abstract Factory in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

#### 2.1. What is the Abstract Factory Pattern?

> **Provide an interface for creating families of related or dependent objects without specifying their concrete classes.**

The **Abstract Factory** pattern provides a way to create families of related objects without specifying their concrete classes.

It's a **creational pattern** that promotes:

- **Loose coupling**
- **Open/Closed Principle**
- **Extensibility**

#### 2.2. Why use Abstract Factory?

| Benefit                         | Explanation                                   |
| ------------------------------- | --------------------------------------------- |
| **Avoid tight coupling**        | Client doesn't need to know concrete classes  |
| **Open for extension**          | Add new product types without changing client |
| **Encapsulate object creation** | Creation logic in one place                   |
| **Better testability**          | Mock factories easily                         |
| **Supports polymorphism**       | Return different objects via same interface   |

#### 2.3. When to use Abstract Factory?

Use it when:

- You need to create families of related objects
- You want to **localize** object creation logic
- You want **subclasses** to decide what to instantiate
- You're building a **framework** or **library**

#### 2.4. Real-Life Example:

A **UI framework** supports:

- Windows buttons
- macOS buttons
- Linux buttons

Each platform creates its own button via a **abstract factory**.

#### 2.5. Structure of Abstract Factory

```text
Client
   |
Creator (abstract)
   ├── create_button() → returns Button
   │
   ├── ConcreteCreatorA → overrides create_button() → returns WindowsButton
   └── ConcreteCreatorB → overrides create_button() → returns MacButton

Button (interface)
   ├── ConcreteButtonA
   └── ConcreteButtonB
```

#### 2.6. Python Example: Cross-Platform UI Family

```python
from abc import ABC, abstractmethod

# Product Interfaces
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

class Checkbox(ABC):
    @abstractmethod
    def check(self) -> str:
        pass

# Concrete Products - Windows
class WindowsButton(Button):
    def render(self) -> str:
        return "Rendering Windows button"

class WindowsCheckbox(Checkbox):
    def check(self) -> str:
        return "Windows checkbox checked"

# Concrete Products - Mac
class MacButton(Button):
    def render(self) -> str:
        return "Rendering macOS button"

class MacCheckbox(Checkbox):
    def check(self) -> str:
        return "macOS checkbox checked"

# Abstract Factory
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

# Concrete Factories
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()

    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()

    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()

# Client
class Application:
    def __init__(self, factory: GUIFactory):
        self.button = factory.create_button()
        self.checkbox = factory.create_checkbox()

    def render(self):
        return f"{self.button.render()} | {self.checkbox.check()}"

# Usage
factory = WindowsFactory()  # or MacFactory()
app = Application(factory)
print(app.render())
```

#### 2.7. Abstract Factory vs Factory Method – What's the difference?

| Aspect           | Abstract Factory                 | Factory Method                              |
| ---------------- | -------------------------------- | ------------------------------------------- |
| **Goal**         | Create related product families  | Defer single product creation to subclasses |
| **Returns**      | Multiple related products        | One product                                 |
| **Pattern Type** | Object composition of factories  | Inheritance-based factory method            |
| **Example**      | `GUIFactory → Button + Checkbox` | `Dialog.create_button()`                    |

#### 2.8. Advantages and Disadvantages of Abstract Factory

| Advantage                  | Description                        |
| -------------------------- | ---------------------------------- |
| **Decentralized creation** | Each subclass controls its product |
| **Extensible**             | Add new types via new subclasses   |
| **Supports OCP**           | No need to modify existing code    |
| **Framework-friendly**     | Great for libraries/SDKs           |

| Drawback                 | Explanation                  |
| ------------------------ | ---------------------------- |
| **Class explosion**      | One creator per product type |
| **Complexity**           | Overkill for simple cases    |
| **Inheritance required** | Forces class hierarchy       |

## 3. Builder Pattern

- **Refactoring Guru – Builder Pattern**: [https://refactoring.guru/design-patterns/builder](https://refactoring.guru/design-patterns/builder)
- **Builder Pattern in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

#### 3.1. What is the Builder Pattern?

> **Separate the construction of a complex object from its representation so that the same construction process can create different representations.**

The **Builder** pattern focuses on constructing complex objects step-by-step. It lets you vary the internal representation while keeping construction logic consistent.

#### 3.2. Why use Builder?

| Benefit                            | Explanation                                                    |
| ---------------------------------- | -------------------------------------------------------------- |
| **Avoid telescoping constructors** | Replace many optional params with step-by-step fluent building |
| **Readable creation**              | Chain methods describe intent clearly                          |
| **Immutable end product**          | Build then finalize object                                     |
| **Different representations**      | Same steps can produce different variants                      |
| **Isolation of construction**      | Keep creation separate from business logic                     |

#### 3.3. When to use Builder?

- Object has many optional parameters or complex setup
- Need different representations of the same object
- Want to hide complex construction details
- Prefer fluent, readable configuration

#### 3.4. Structure of Builder

```text
Director (optional)
   └── Builder (interface)
         ├── ConcreteBuilderA
         └── ConcreteBuilderB
                └── builds → Product
```

#### 3.5. Python Example: Building a Computer

```python
from abc import ABC, abstractmethod

class Computer:
    def __init__(self):
        self.cpu = None
        self.ram_gb = None
        self.storage_gb = None
        self.gpu = None

    def __repr__(self):
        return f"Computer(cpu={self.cpu}, ram={self.ram_gb}GB, storage={self.storage_gb}GB, gpu={self.gpu})"

class Builder(ABC):
    @abstractmethod
    def set_cpu(self, model: str): ...
    @abstractmethod
    def set_ram(self, gb: int): ...
    @abstractmethod
    def set_storage(self, gb: int): ...
    @abstractmethod
    def set_gpu(self, model: str): ...
    @abstractmethod
    def build(self) -> Computer: ...

class GamingPCBuilder(Builder):
    def __init__(self):
        self._pc = Computer()
    def set_cpu(self, model: str):
        self._pc.cpu = model; return self
    def set_ram(self, gb: int):
        self._pc.ram_gb = gb; return self
    def set_storage(self, gb: int):
        self._pc.storage_gb = gb; return self
    def set_gpu(self, model: str):
        self._pc.gpu = model; return self
    def build(self) -> Computer:
        return self._pc

class OfficePCBuilder(Builder):
    def __init__(self):
        self._pc = Computer()
    def set_cpu(self, model: str):
        self._pc.cpu = model; return self
    def set_ram(self, gb: int):
        self._pc.ram_gb = gb; return self
    def set_storage(self, gb: int):
        self._pc.storage_gb = gb; return self
    def set_gpu(self, model: str):
        self._pc.gpu = model; return self
    def build(self) -> Computer:
        return self._pc

# Director (optional)
class PCDirector:
    def __init__(self, builder: Builder):
        self.builder = builder
    def build_basic(self):
        return (self.builder
                .set_cpu("i5")
                .set_ram(16)
                .set_storage(512)
                .set_gpu("Integrated")
                .build())

# Usage
gaming = (GamingPCBuilder()
          .set_cpu("Ryzen 7")
          .set_ram(32)
          .set_storage(2000)
          .set_gpu("RTX 4070")
          .build())

office = PCDirector(OfficePCBuilder()).build_basic()
```

#### 3.6. Builder vs Abstract Factory

| Aspect         | Builder                   | Abstract Factory             |
| -------------- | ------------------------- | ---------------------------- |
| **Focus**      | Step-by-step construction | Families of related products |
| **Output**     | 1 complex object          | Multiple related objects     |
| **Director**   | Optional                  | Not applicable               |
| **Fluent API** | Common                    | Rare                         |

#### 3.7. Advantages and Disadvantages of Builder

| Advantage         | Description                             |
| ----------------- | --------------------------------------- |
| **Clarity**       | Fluent steps make construction readable |
| **Flexibility**   | Different builders for variants         |
| **Encapsulation** | Hides complex assembly                  |

| Drawback         | Explanation                                  |
| ---------------- | -------------------------------------------- |
| **More classes** | Builders/Director add types                  |
| **Overhead**     | Simple objects don't need it                 |
| **Validation**   | Must ensure object completeness before build |

## 4. Prototype Pattern

- **Refactoring Guru – Prototype Pattern**: [https://refactoring.guru/design-patterns/prototype](https://refactoring.guru/design-patterns/prototype)
- **Prototype Pattern in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

#### 4.1. What is the Prototype Pattern?

> **Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.**

The **Prototype** pattern clones existing objects instead of creating from scratch. Useful when object creation is expensive or complex.

#### 4.2. Why use Prototype?

| Benefit                      | Explanation                               |
| ---------------------------- | ----------------------------------------- |
| **Performance**              | Clone instead of reconstructing           |
| **Decouple from classes**    | Create via interface, not concrete class  |
| **Runtime configuration**    | Register and clone prototypes dynamically |
| **Avoid subclass explosion** | Configure via state rather than types     |

#### 4.3. When to use Prototype?

- Object creation is costly or involves complex graph
- Need to create many similar objects
- Want to avoid factories for dynamic types
- Need copies (snapshots) with modifications

#### 4.4. Structure of Prototype

```text
Prototype (interface) ── clone()
   ├── ConcretePrototypeA
   └── ConcretePrototypeB

PrototypeRegistry (optional) ── register(key, proto), clone(key)
```

#### 4.5. Python Example: Cloning with Registry

```python
import copy

class Document:
    def __init__(self, title, pages, metadata=None):
        self.title = title
        self.pages = pages  # list of strings
        self.metadata = metadata or {}
    def clone(self, deep: bool = True):
        return copy.deepcopy(self) if deep else copy.copy(self)
    def __repr__(self):
        return f"Document(title={self.title}, pages={len(self.pages)}, meta={self.metadata})"

class PrototypeRegistry:
    def __init__(self):
        self._prototypes = {}
    def register(self, key: str, prototype: object):
        self._prototypes[key] = prototype
    def clone(self, key: str, deep: bool = True, **attrs):
        if key not in self._prototypes:
            raise KeyError(f"No prototype for key: {key}")
        obj = self._prototypes[key].clone(deep=deep)
        for k, v in attrs.items():
            setattr(obj, k, v)
        return obj

# Usage
registry = PrototypeRegistry()
base = Document("Report", ["p1", "p2"], {"author": "Alice"})
registry.register("report", base)

copy1 = registry.clone("report", title="Q1 Report")         # deep clone
copy2 = registry.clone("report", deep=False, title="Draft") # shallow clone
```

#### 4.6. Shallow vs Deep Copy in Python

| Aspect         | Shallow (`copy.copy`) | Deep (`copy.deepcopy`)          |
| -------------- | --------------------- | ------------------------------- |
| Nested objects | Shared references     | Fully cloned graph              |
| Performance    | Faster                | Slower                          |
| Safety         | Risky with mutables   | Safer, but watch custom objects |

#### 4.7. Advantages and Disadvantages of Prototype

| Advantage               | Description                        |
| ----------------------- | ---------------------------------- |
| **Fast creation**       | Skip repeated setup                |
| **Runtime flexibility** | Register/clone without new classes |
| **Decoupling**          | Client unaware of concrete classes |

| Drawback             | Explanation                                  |
| -------------------- | -------------------------------------------- |
| **Clone complexity** | Cycles, external resources, descriptors      |
| **Copy semantics**   | Must choose shallow vs deep carefully        |
| **Hidden coupling**  | Prototypes can carry unintended shared state |

## 5. Singleton Pattern

- **Refactoring Guru – Singleton Pattern**: [https://refactoring.guru/design-patterns/singleton](https://refactoring.guru/design-patterns/singleton)
- **Youtube - Singleton Pattern**: https://www.youtube.com/watch?v=tSZn4wkBIu8

#### 5.1. What is the Singleton Pattern?

> **Ensure a class has only one instance and provide a global point of access to it.**

Singleton restricts instantiation to a single object. Use sparingly; it's often considered an anti-pattern in large systems due to hidden global state.

#### 5.2. When (and when not) to use Singleton?

- Use for: configuration, logging sinks, device managers, caches
- Avoid when: testability matters, you need clear dependency injection, or concurrency/isolation is critical

#### 5.3. Python Implementations

1. Module-level (Pythonic, simplest)

```python
# settings.py
DB_URL = "postgres://..."
```

Importing module variables acts like a singleton.

2. **new** override

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            with cls._lock:
                if not cls._instance:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

3. Decorator

```python
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Cache:
    pass
```

4. Borg/Monostate (shared state)

```python
class Borg:
    _state = {}
    def __init__(self):
        self.__dict__ = self._state
```

#### 5.4. Thread Safety and Multiprocessing

- Use locks around first instantiation in multi-threaded contexts
- Across processes, OS-level singletons don't hold; use external coordination (files, sockets, DB)

#### 5.5. Advantages and Disadvantages of Singleton

| Advantage                | Description                   |
| ------------------------ | ----------------------------- |
| **Single access point**  | Centralize shared resource    |
| **Controlled lifecycle** | Lazily initialize when needed |

| Drawback                 | Explanation                       |
| ------------------------ | --------------------------------- |
| **Global state**         | Hinders testability and isolation |
| **Hidden dependencies**  | Harder to reason about            |
| **Concurrency pitfalls** | Race conditions without care      |

#### 5.6. Alternatives to Singleton

- Prefer dependency injection (pass instances explicitly)
- Use application context/container
- For constants, prefer module-level variables
