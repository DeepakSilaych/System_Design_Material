# Behavioral Design Patterns

Behavioral design patterns are concerned with algorithms and the assignment of responsibilities between objects.

---

## Strategy Pattern

- **Refactoring Guru – Strategy Pattern**: [https://refactoring.guru/design-patterns/strategy](https://refactoring.guru/design-patterns/strategy)

- **Strategy Pattern in Python (Hindi)**: [https://www.youtube.com/watch?v=PpKvPrl_gRg](https://www.youtube.com/watch?v=PpKvPrl_gRg)

----

### 1. What is the Strategy Design Pattern?
> **Define a family of algorithms, encapsulate each one, and make them interchangeable.**

The **Strategy pattern** lets you:
- Define a set of algorithms (behaviors)
- Put each in its own class
- Switch between them at **runtime**

It promotes **"favor composition over inheritance"** and enables **flexible, reusable code**.

---

### 2. Why use the Strategy Pattern?
| Benefit | Explanation |
|-------|-------------|
| **Open/Closed Principle** | Add new strategies without changing existing code |
| **Replace Inheritance** | Avoid deep inheritance trees |
| **Runtime Flexibility** | Change behavior dynamically |
| **Testability** | Easy to mock or test individual strategies |
| **Cleaner Code** | Logic is separated into focused classes |

---

### 3. When should you use Strategy Pattern?

Use it when:
- You have **multiple ways** to perform a task
- You want to **switch behavior at runtime**
- You want to **avoid conditional statements** (if/else chains)
- Algorithms vary **independently** from the client

#### Real-Life Example:
A **navigation app** can route via:
- Car
- Bike
- Walking
- Public Transit

Each is a **strategy** — user picks one at runtime.

---

### 4. Structure of Strategy Pattern

```text
Context
   └── Strategy (interface)
         ├── ConcreteStrategyA
         ├── ConcreteStrategyB
         └── ConcreteStrategyC
```

- **Context**: Uses a strategy
- **Strategy**: Interface for all algorithms
- **Concrete Strategies**: Implement the algorithm

---

### 5. Python Example: Payment Processing

#### Bad Way (Without Strategy)
```python
class Order:
    def __init__(self, total):
        self.total = total

    def pay(self, payment_type):
        if payment_type == "credit":
            print(f"Paying {self.total} using Credit Card.")
        elif payment_type == "paypal":
            print(f"Paying {self.total} using PayPal.")
        elif payment_type == "crypto":
            print(f"Paying {self.total} using Bitcoin.")
        else:
            raise ValueError("Unknown payment method")
```

Problem: New payment → modify `pay()` → **violates OCP**

---

#### Good Way (With Strategy Pattern)

```python
from abc import ABC, abstractmethod

# Strategy Interface
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float):
        pass

# Concrete Strategies
class CreditCardPayment(PaymentStrategy):
    def pay(self, amount: float):
        print(f"Paying ${amount:.2f} using Credit Card.")

class PayPalPayment(PaymentStrategy):
    def pay(self, amount: float):
        print(f"Paying ${amount:.2f} using PayPal.")

class BitcoinPayment(PaymentStrategy):
    def pay(self, amount: float):
        print(f"Paying ${amount:.2f} using Bitcoin.")

# Context
class Order:
    def __init__(self, total: float):
        self.total = total
        self.payment_strategy = None

    def set_payment_strategy(self, strategy: PaymentStrategy):
        self.payment_strategy = strategy

    def pay(self):
        if not self.payment_strategy:
            raise ValueError("Payment strategy not set!")
        self.payment_strategy.pay(self.total)
```

#### Usage:
```python
order = Order(100.0)

# Switch strategies at runtime
order.set_payment_strategy(CreditCardPayment())
order.pay()  # Paying $100.00 using Credit Card.

order.set_payment_strategy(PayPalPayment())
order.pay()  # Paying $100.00 using PayPal.

order.set_payment_strategy(BitcoinPayment())
order.pay()  # Paying $100.00 using Bitcoin.
```

---

### 6. Strategy vs State Pattern – What's the difference?

| Feature | Strategy | State |
|-------|----------|-------|
| **Purpose** | Change **algorithm** | Change **object behavior** based on internal state |
| **When to change** | Client decides | Object changes itself |
| **Example** | Sorting: Quick vs Merge | Traffic Light: Red → Green → Yellow |

---

### 7. Can Strategy be used with Dependency Injection?

**Yes!** It's perfect for DI.

```python
class ShoppingCart:
    def __init__(self, payment_strategy: PaymentStrategy):
        self.payment_strategy = payment_strategy
        self.items = []

    def checkout(self, total):
        self.payment_strategy.pay(total)
```

Now inject strategy via constructor:
```python
cart = ShoppingCart(PayPalPayment())
cart.checkout(99.99)
```

---

### 8. Real-World Analogy

| Component | Real Life |
|---------|----------|
| **Context** | A **chef** |
| **Strategy** | Cooking style: Grill, Bake, Fry |
| **Concrete Strategy** | `GrillStrategy`, `BakeStrategy` |
| **Switching** | Chef picks method based on dish |

---

### 9. Common Interview Question: "Refactor this code using Strategy"

#### Before (if-else hell):
```python
def compress_file(file, method):
    if method == "zip":
        print("Compressing with ZIP")
    elif method == "rar":
        print("Compressing with RAR")
    elif method == "7z":
        print("Compressing with 7Z")
```

#### After (Strategy):
```python
from abc import ABC, abstractmethod

class CompressionStrategy(ABC):
    @abstractmethod
    def compress(self, file):
        pass

class ZipCompression(CompressionStrategy):
    def compress(self, file):
        print(f"Compressing {file} using ZIP")

class RarCompression(CompressionStrategy):
    def compress(self, file):
        print(f"Compressing {file} using RAR")

class Compressor:
    def __init__(self, strategy: CompressionStrategy):
        self.strategy = strategy

    def compress(self, file):
        self.strategy.compress(file)

# Usage
compressor = Compressor(ZipCompression())
compressor.compress("data.txt")
```

---

### 10. Advantages of Strategy Pattern

| Advantage | Description |
|----------|-------------|
| **Eliminates conditionals** | No `if-elif` chains |
| **Reusable algorithms** | Use same strategy in multiple contexts |
| **Easy to test** | Test strategies in isolation |
| **Runtime switching** | Change behavior without restarting |

---

### 11. Disadvantages

| Drawback | Explanation |
|---------|-------------|
| **More classes** | Can increase class count |
| **Client must know strategies** | Sometimes exposes implementation |
| **Overhead for simple cases** | Not worth it for 1–2 algorithms |

---

### 12. Strategy in Python: Idiomatic Way (Using Functions)

Python allows **first-class functions**, so you can simplify:

```python
def pay_credit(amount):
    print(f"Paying ${amount} via Credit Card")

def pay_paypal(amount):
    print(f"Paying ${amount} via PayPal")

class Order:
    def __init__(self, total):
        self.total = total
        self.pay_func = None

    def set_payment(self, func):
        self.pay_func = func

    def pay(self):
        self.pay_func(self.total)

# Usage
order = Order(50)
order.set_payment(pay_paypal)
order.pay()
```

**Best for simple cases** — no need for full classes.

---

### 13. UML Diagram (Text Version)

```text
+----------------+          +--------------------+
|    Context     |<>------->|     Strategy       |
+----------------+ 1     1  +--------------------+
| - strategy     |          | + execute()        |
+----------------+          +--------------------+
| + execute()    |                /\_/\
+----------------+                 / \
                                   |
             +---------------------+--------------------+
             |                     |                    |
    +----------------+    +----------------+    +----------------+
    | ConcreteStratA |    | ConcreteStratB |    | ConcreteStratC |
    +----------------+    +----------------+    +----------------+
    | + execute()    |    | + execute()    |    | + execute()    |
    +----------------+    +----------------+    +----------------+
```

---

**Summary**:  
**Strategy Pattern = Plug-and-play algorithms.**  
Use it when you want **behavior to be configurable**, **extensible**, and **clean**.  
In Python, combine with **functions or ABCs** depending on complexity.

