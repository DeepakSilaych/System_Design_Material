# Designing a Vending Machine

## Question
Design a Vending Machine

## Requirements
1. The vending machine should support multiple products with different prices and quantities.
1. The machine should accept coins and notes of different denominations.
1. The machine should dispense the selected product and return change if necessary.
1. The machine should keep track of the available products and their quantities.
1. The machine should handle multiple transactions concurrently and ensure data consistency.
1. The machine should provide an interface for restocking products and collecting money.
1. The machine should handle exceptional scenarios, such as insufficient funds or out-of-stock products.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: Product, Coin, Note, Inventory, VendingMachineState, IdleState, ReadyState,
DispenseState, VendingMachine, VendingMachineDemo.

### Design Details
1. The **Product** class represents a product in the vending machine, with properties such as name and price.
2. The **Coin** and **Note** enums represent the different denominations of coins and notes accepted by the vending machine.
3. The **Inventory** class manages the available products and their quantities in the vending machine. It uses a concurrent hash map to ensure thread safety.
4. The **VendingMachineState** interface defines the behavior of the vending machine in different states, such as idle, ready, and dispense.
5. The **IdleState**, **ReadyState**, and **DispenseState** classes implement the VendingMachineState interface and define the specific behaviors for each state.
6. The **VendingMachine** class is the main class that represents the vending machine. It follows the Singleton pattern to ensure only one instance of the vending machine exists.
7. The VendingMachine class maintains the current state, selected product, total payment, and provides methods for state transitions and payment handling.
8. The **VendingMachineDemo** class demonstrates the usage of the vending machine by adding products to the inventory, selecting products, inserting coins and notes, dispensing products, and returning change.

### Implementation (Python)
#### coin.py

```python
from enum import Enum

class Coin(Enum):
    PENNY = 1
    NICKEL = 5
    DIME = 10
    QUARTER = 25

    def get_value(self) -> int:
        return self.value
```

#### inventory.py

```python
from typing import Dict, Optional
from item import Item

class Inventory:
    def __init__(self):
        self.item_map: Dict[str, Item] = {}
        self.stock_map: Dict[str, int] = {}

    def add_item(self, code: str, item: Item, quantity: int) -> None:
        self.item_map[code] = item
        self.stock_map[code] = quantity

    def get_item(self, code: str) -> Optional[Item]:
        return self.item_map.get(code)

    def is_available(self, code: str) -> bool:
        return self.stock_map.get(code, 0) > 0

    def reduce_stock(self, code: str) -> None:
        self.stock_map[code] = self.stock_map[code] - 1
```

#### item.py

```python
class Item:
    def __init__(self, code: str, name: str, price: int):
        self.code = code
        self.name = name
        self.price = price

    def get_name(self) -> str:
        return self.name

    def get_price(self) -> int:
        return self.price
```

#### states.py

```python
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING
from coin import Coin

if TYPE_CHECKING:
    from .vending_machine import VendingMachine


class VendingMachineState(ABC):
    """Abstract base class for all vending machine states."""
    
    def __init__(self, machine: 'VendingMachine'):
        self.machine = machine
    
    @abstractmethod
    def insert_coin(self, coin: Coin):
        """Handle coin insertion."""
        pass
    
    @abstractmethod
    def select_item(self, code: str):
        """Handle item selection."""
        pass
    
    @abstractmethod
    def dispense(self):
        """Handle dispense request."""
        pass
    
    @abstractmethod
    def refund(self):
        """Handle refund request."""
        pass


class IdleState(VendingMachineState):
    """State when machine is waiting for user interaction."""
    
    def insert_coin(self, coin: Coin):
        print("Please select an item before inserting money.")
    
    def select_item(self, code: str):
        if not self.machine.get_inventory().is_available(code):
            print("Item not available.")
            return
        
        self.machine.set_selected_item_code(code)
        # Import here to avoid circular dependency
        from states import ItemSelectedState
        self.machine.set_state(ItemSelectedState(self.machine))
        print(f"Item selected: {code}")
    
    def dispense(self):
        print("No item selected.")
    
    def refund(self):
        print("No money to refund.")


class ItemSelectedState(VendingMachineState):
    """State when an item has been selected but insufficient money inserted."""
    
    def insert_coin(self, coin: Coin):
        self.machine.add_balance(coin.get_value())
        print(f"Coin inserted: {coin.get_value()}¢ ({coin.name})")
        
        selected_item = self.machine.get_selected_item()
        if selected_item and self.machine.get_balance() >= selected_item.get_price():
            print("Sufficient money received.")
            # Import here to avoid circular dependency
            from states import HasMoneyState
            self.machine.set_state(HasMoneyState(self.machine))
    
    def select_item(self, code: str):
        print("Item already selected. Please insert money or request refund to select a different item.")
    
    def dispense(self):
        print("Please insert sufficient money.")
    
    def refund(self):
        self.machine.refund_balance()
        self.machine.reset()
        # Import here to avoid circular dependency
        from states import IdleState
        self.machine.set_state(IdleState(self.machine))


class HasMoneyState(VendingMachineState):
    """State when sufficient money has been inserted."""
    
    def insert_coin(self, coin: Coin):
        # Allow more coins (will be returned as change)
        self.machine.add_balance(coin.get_value())
        print(f"Additional coin inserted: {coin.get_value()}¢ ({coin.name}) - will be returned as change.")
    
    def select_item(self, code: str):
        print("Item already selected. Please dispense or request refund to select a different item.")
    
    def dispense(self):
        # Import here to avoid circular dependency
        from states import DispensingState
        self.machine.set_state(DispensingState(self.machine))
        self.machine.dispense_item()
    
    def refund(self):
        self.machine.refund_balance()
        self.machine.reset()
        # Import here to avoid circular dependency
        from states import IdleState
        self.machine.set_state(IdleState(self.machine))


class DispensingState(VendingMachineState):
    """State when item is being dispensed - blocks all user input."""
    
    def insert_coin(self, coin: Coin):
        print("Currently dispensing. Please wait.")
    
    def select_item(self, code: str):
        print("Currently dispensing. Please wait.")
    
    def dispense(self):
        # Already triggered by HasMoneyState
        print("Dispensing in progress...")
    
    def refund(self):
        print("Dispensing in progress. Refund not allowed.")
```

#### vending_machine.py

```python
from inventory import Inventory
from coin import Coin
from item import Item
from states import VendingMachineState, IdleState

class VendingMachine:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(VendingMachine, cls).__new__(cls)
            cls._instance._initialized = False
        return cls._instance

    def __init__(self):
        if not hasattr(self, '_initialized') or not self._initialized:
            self.inventory = Inventory()
            self.current_state = IdleState(self)
            self.balance = 0
            self.selected_item_code = None
            self._initialized = True

    @classmethod
    def get_instance(cls):
        return cls()

    def insert_coin(self, coin: Coin) -> None:
        self.current_state.insert_coin(coin)

    def add_item(self, code: str, name: str, price: int, quantity: int) -> Item:
        item = Item(code, name, price)
        self.inventory.add_item(code, item, quantity)
        return item

    def select_item(self, code: str) -> None:
        self.current_state.select_item(code)

    def dispense(self) -> None:
        self.current_state.dispense()

    def dispense_item(self) -> None:
        item = self.inventory.get_item(self.selected_item_code)
        if self.balance >= item.get_price():
            self.inventory.reduce_stock(self.selected_item_code)
            self.balance -= item.get_price()
            print(f"Dispensed: {item.get_name()}")
            if self.balance > 0:
                print(f"Returning change: {self.balance}")
        self.reset()
        self.set_state(IdleState(self))

    def refund_balance(self) -> None:
        print(f"Refunding: {self.balance}")
        self.balance = 0

    def reset(self) -> None:
        self.selected_item_code = None
        self.balance = 0

    def add_balance(self, value: int) -> None:
        self.balance += value

    def get_selected_item(self) -> Item:
        return self.inventory.get_item(self.selected_item_code)

    def set_selected_item_code(self, code: str) -> None:
        self.selected_item_code = code

    def set_state(self, state: VendingMachineState) -> None:
        self.current_state = state

    # Getters for states and inventory
    def get_inventory(self) -> Inventory:
        return self.inventory

    def get_balance(self) -> int:
        return self.balance
```

#### vending_machine_demo.py

```python
from vending_machine import VendingMachine
from coin import Coin

class VendingMachineDemo:
    @staticmethod
    def main():
        vending_machine = VendingMachine.get_instance()

        # Add products to the inventory
        vending_machine.add_item("A1", "Coke", 25, 3)
        vending_machine.add_item("A2", "Pepsi", 25, 2)
        vending_machine.add_item("B1", "Water", 10, 5)

        # Select a product
        print("\n--- Step 1: Select an item ---")
        vending_machine.select_item("A1")

        # Insert coins
        print("\n--- Step 2: Insert coins ---")
        vending_machine.insert_coin(Coin.DIME)  # 10
        vending_machine.insert_coin(Coin.DIME)  # 10
        vending_machine.insert_coin(Coin.NICKEL)  # 5

        # Dispense the product
        print("\n--- Step 3: Dispense item ---")
        vending_machine.dispense()  # Should dispense Coke

        # Select another item
        print("\n--- Step 4: Select another item ---")
        vending_machine.select_item("B1")

        # Insert more amount
        print("\n--- Step 5: Insert more than needed ---")
        vending_machine.insert_coin(Coin.QUARTER)  # 25

        # Try to dispense the product
        print("\n--- Step 6: Dispense and return change ---")
        vending_machine.dispense()


if __name__ == "__main__":
    VendingMachineDemo.main()
```