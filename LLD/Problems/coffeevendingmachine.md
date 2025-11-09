# Designing a Coffee Vending Machine

## Question
Design a Coffee Vending Machine

## Requirements
1. The coffee vending machine should support different types of coffee, such as espresso, cappuccino, and latte.
2. Each type of coffee should have a specific price and recipe (ingredients and their quantities).
3. The machine should have a menu to display the available coffee options and their prices.
4. Users should be able to select a coffee type and make a payment.
5. The machine should dispense the selected coffee and provide change if necessary.
6. The machine should track the inventory of ingredients and notify when they are running low.
7. The machine should handle multiple user requests concurrently and ensure thread safety.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: Coffee, Ingredient, Payment, CoffeeMachine, CoffeeMachine,
CoffeeVendingMachine.

### Design Details
1. The **Coffee** class represents a coffee type with its name, price, and recipe (ingredients and their quantities).
2. The **Ingredient** class represents an ingredient used in making coffee, with its name and quantity. It provides a synchronized method to update the quantity.
3. The **Payment** class represents a payment made by a user, with the amount paid.
4. The **CoffeeMachine** class is the main class that manages the coffee vending machine. It follows the Singleton pattern to ensure a single instance of the machine.
5. The **CoffeeMachine** class initializes the coffee menu and ingredients in its constructor. It provides methods to display the menu, select a coffee, dispense coffee, and update ingredient quantities.
6. The hasEnoughIngredients method checks if there are sufficient ingredients to make a selected coffee, while the updateIngredients method updates the ingredient quantities after dispensing a coffee.
7. The **CoffeeVendingMachine** class is the entry point of the application and demonstrates the usage of the coffee vending machine. It creates an instance of the machine, displays the menu, and simulates concurrent user requests using an ExecutorService.

### Implementation (Python)
#### coffee.py

```python
from abc import ABC, abstractmethod
from typing import Dict
from enums import Ingredient

class Coffee(ABC):
    def __init__(self):
        self.coffee_type = "Unknown Coffee"
    
    def get_coffee_type(self) -> str:
        return self.coffee_type
    
    # The Template Method
    def prepare(self):
        print(f"\nPreparing your {self.get_coffee_type()}...")
        self._grind_beans()
        self._brew()
        self.add_condiments()  # The "hook" for base coffee types
        self._pour_into_cup()
        print(f"{self.get_coffee_type()} is ready!")
    
    # Common steps
    def _grind_beans(self):
        print("- Grinding fresh coffee beans.")
    
    def _brew(self):
        print("- Brewing coffee with hot water.")
    
    def _pour_into_cup(self):
        print("- Pouring into a cup.")
    
    # Abstract step to be implemented by subclasses
    @abstractmethod
    def add_condiments(self):
        pass
    
    @abstractmethod
    def get_price(self) -> int:
        pass
    
    @abstractmethod
    def get_recipe(self) -> Dict[Ingredient, int]:
        pass

class Espresso(Coffee):
    def __init__(self):
        super().__init__()
        self.coffee_type = "Espresso"
    
    def add_condiments(self):
        pass  # No extra condiments for espresso
    
    def get_price(self) -> int:
        return 150
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        return {Ingredient.COFFEE_BEANS: 7, Ingredient.WATER: 30}

class Cappuccino(Coffee):
    def __init__(self):
        super().__init__()
        self.coffee_type = "Cappuccino"
    
    def add_condiments(self):
        print("- Adding steamed milk and foam.")
    
    def get_price(self) -> int:
        return 250
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        return {Ingredient.COFFEE_BEANS: 7, Ingredient.WATER: 30, Ingredient.MILK: 100}

class Latte(Coffee):
    def __init__(self):
        super().__init__()
        self.coffee_type = "Latte"
    
    def add_condiments(self):
        print("- Adding steamed milk.")
    
    def get_price(self) -> int:
        return 220
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        return {Ingredient.COFFEE_BEANS: 7, Ingredient.WATER: 30, Ingredient.MILK: 150}
```

#### coffee_factory.py

```python
from enums import CoffeeType
from coffee import Coffee, Espresso, Latte, Cappuccino

class CoffeeFactory:
    @staticmethod
    def create_coffee(coffee_type: CoffeeType) -> Coffee:
        if coffee_type == CoffeeType.ESPRESSO:
            return Espresso()
        elif coffee_type == CoffeeType.LATTE:
            return Latte()
        elif coffee_type == CoffeeType.CAPPUCCINO:
            return Cappuccino()
        else:
            raise ValueError(f"Unsupported coffee type: {coffee_type}")
```

#### coffee_vending_machine.py

```python
from typing import List
from enums import CoffeeType, ToppingType
from coffee import Coffee
from coffee_factory import CoffeeFactory
from decorators import ExtraSugarDecorator, CaramelSyrupDecorator
from states import VendingMachineState, ReadyState
import threading

class CoffeeVendingMachine:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self._state = ReadyState()
            self._selected_coffee: Coffee = None
            self._money_inserted = 0
            self._initialized = True
    
    @classmethod
    def get_instance(cls):
        return cls()
    
    def select_coffee(self, coffee_type: CoffeeType, toppings: List[ToppingType]):
        # 1. Create the base coffee using the factory
        coffee = CoffeeFactory.create_coffee(coffee_type)
        
        # 2. Wrap it with decorators
        for topping in toppings:
            if topping == ToppingType.EXTRA_SUGAR:
                coffee = ExtraSugarDecorator(coffee)
            elif topping == ToppingType.CARAMEL_SYRUP:
                coffee = CaramelSyrupDecorator(coffee)
        
        # Let the state handle the rest
        self._state.select_coffee(self, coffee)
    
    def insert_money(self, amount: int):
        self._state.insert_money(self, amount)
    
    def dispense_coffee(self):
        self._state.dispense_coffee(self)
    
    def cancel(self):
        self._state.cancel(self)
    
    # Getters and Setters used by State objects
    def set_state(self, state: VendingMachineState):
        self._state = state
    
    def get_state(self) -> VendingMachineState:
        return self._state
    
    def set_selected_coffee(self, coffee: Coffee):
        self._selected_coffee = coffee
    
    def get_selected_coffee(self) -> Coffee:
        return self._selected_coffee
    
    def set_money_inserted(self, amount: int):
        self._money_inserted = amount
    
    def get_money_inserted(self) -> int:
        return self._money_inserted
    
    def reset(self):
        self._selected_coffee = None
        self._money_inserted = 0
```

#### coffee_vending_machine_demo.py

```python
from coffee_vending_machine import CoffeeVendingMachine
from inventory import Inventory
from enums import CoffeeType, ToppingType, Ingredient

class CoffeeVendingMachineDemo:
    @staticmethod
    def main():
        machine = CoffeeVendingMachine.get_instance()
        inventory = Inventory.get_instance()
        
        # Initial setup: Refill inventory
        print("=== Initializing Vending Machine ===")
        inventory.add_stock(Ingredient.COFFEE_BEANS, 50)
        inventory.add_stock(Ingredient.WATER, 500)
        inventory.add_stock(Ingredient.MILK, 200)
        inventory.add_stock(Ingredient.SUGAR, 100)
        inventory.add_stock(Ingredient.CARAMEL_SYRUP, 50)
        inventory.print_inventory()
        
        # Scenario 1: Successful Purchase of a Latte
        print("\n--- SCENARIO 1: Buy a Latte (Success) ---")
        machine.select_coffee(CoffeeType.LATTE, [])
        machine.insert_money(200)
        machine.insert_money(50)  # Total 250, price is 220
        machine.dispense_coffee()
        inventory.print_inventory()
        
        # Scenario 2: Purchase with Insufficient Funds & Cancellation
        print("\n--- SCENARIO 2: Buy Espresso (Insufficient Funds & Cancel) ---")
        machine.select_coffee(CoffeeType.ESPRESSO, [])
        machine.insert_money(100)  # Price is 150
        machine.dispense_coffee()  # Should fail
        machine.cancel()  # Should refund 100
        inventory.print_inventory()  # Should be unchanged
        
        # Scenario 3: Attempt to Buy with Insufficient Ingredients
        print("\n--- SCENARIO 3: Buy Cappuccino (Out of Milk) ---")
        inventory.print_inventory()
        machine.select_coffee(CoffeeType.CAPPUCCINO, [ToppingType.CARAMEL_SYRUP, ToppingType.EXTRA_SUGAR])
        machine.insert_money(300)
        machine.dispense_coffee()  # Should fail and refund
        inventory.print_inventory()
        
        # Refill and final test
        print("\n--- REFILLING AND FINAL TEST ---")
        inventory.add_stock(Ingredient.MILK, 200)
        inventory.print_inventory()
        machine.select_coffee(CoffeeType.LATTE, [ToppingType.CARAMEL_SYRUP])
        machine.insert_money(250)
        machine.dispense_coffee()
        inventory.print_inventory()

if __name__ == "__main__":
    CoffeeVendingMachineDemo.main()
```

#### decorators.py

```python
from coffee import Coffee
from typing import Dict
from enums import Ingredient

class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        super().__init__()
        self.decorated_coffee = coffee
    
    def get_price(self) -> int:
        return self.decorated_coffee.get_price()
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        return self.decorated_coffee.get_recipe()
    
    def add_condiments(self):
        self.decorated_coffee.add_condiments()
    
    def prepare(self):
        self.decorated_coffee.prepare()

class CaramelSyrupDecorator(CoffeeDecorator):
    COST = 30
    RECIPE_ADDITION = {Ingredient.CARAMEL_SYRUP: 10}
    
    def __init__(self, coffee: Coffee):
        super().__init__(coffee)
    
    def get_coffee_type(self) -> str:
        return self.decorated_coffee.get_coffee_type() + ", Caramel Syrup"
    
    def get_price(self) -> int:
        return self.decorated_coffee.get_price() + self.COST
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        new_recipe = self.decorated_coffee.get_recipe().copy()
        for ingredient, qty in self.RECIPE_ADDITION.items():
            new_recipe[ingredient] = new_recipe.get(ingredient, 0) + qty
        return new_recipe
    
    def prepare(self):
        # First, prepare the underlying coffee
        super().prepare()
        # Then, add the specific step for this decorator
        print("- Drizzling Caramel Syrup on top.")

class ExtraSugarDecorator(CoffeeDecorator):
    COST = 10
    RECIPE_ADDITION = {Ingredient.SUGAR: 1}
    
    def __init__(self, coffee: Coffee):
        super().__init__(coffee)
    
    def get_coffee_type(self) -> str:
        return self.decorated_coffee.get_coffee_type() + ", Extra Sugar"
    
    def get_price(self) -> int:
        return self.decorated_coffee.get_price() + self.COST
    
    def get_recipe(self) -> Dict[Ingredient, int]:
        new_recipe = self.decorated_coffee.get_recipe().copy()
        for ingredient, qty in self.RECIPE_ADDITION.items():
            new_recipe[ingredient] = new_recipe.get(ingredient, 0) + qty
        return new_recipe
    
    def prepare(self):
        super().prepare()
        print("- Stirring in Extra Sugar.")
```

#### enums.py

```python
from enum import Enum

class CoffeeType(Enum):
    ESPRESSO = "ESPRESSO"
    LATTE = "LATTE"
    CAPPUCCINO = "CAPPUCCINO"

class Ingredient(Enum):
    COFFEE_BEANS = "COFFEE_BEANS"
    MILK = "MILK"
    SUGAR = "SUGAR"
    WATER = "WATER"
    CARAMEL_SYRUP = "CARAMEL_SYRUP"

class ToppingType(Enum):
    EXTRA_SUGAR = "EXTRA_SUGAR"
    CARAMEL_SYRUP = "CARAMEL_SYRUP"
```

#### inventory.py

```python
from typing import Dict
from enums import Ingredient
import threading

class Inventory:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self._stock: Dict[Ingredient, int] = {}
            self._lock = threading.Lock()
            self._initialized = True
    
    @classmethod
    def get_instance(cls):
        return cls()
    
    def add_stock(self, ingredient: Ingredient, quantity: int):
        self._stock[ingredient] = self._stock.get(ingredient, 0) + quantity
    
    def has_ingredients(self, recipe: Dict[Ingredient, int]) -> bool:
        return all(self._stock.get(ingredient, 0) >= quantity 
                  for ingredient, quantity in recipe.items())
    
    def deduct_ingredients(self, recipe: Dict[Ingredient, int]):
        with self._lock:
            if not self.has_ingredients(recipe):
                print("Not enough ingredients to make coffee.")
                return
            
            for ingredient, quantity in recipe.items():
                self._stock[ingredient] -= quantity
    
    def print_inventory(self):
        print("--- Current Inventory ---")
        for ingredient, quantity in self._stock.items():
            print(f"{ingredient.value}: {quantity}")
        print("-------------------------")
```

#### states.py

```python
from abc import ABC, abstractmethod
from inventory import Inventory
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from coffee_vending_machine import CoffeeVendingMachine
    from coffee import Coffee

class VendingMachineState(ABC):
    @abstractmethod
    def select_coffee(self, machine: 'CoffeeVendingMachine', coffee: 'Coffee'):
        pass
    
    @abstractmethod
    def insert_money(self, machine: 'CoffeeVendingMachine', amount: int):
        pass
    
    @abstractmethod
    def dispense_coffee(self, machine: 'CoffeeVendingMachine'):
        pass
    
    @abstractmethod
    def cancel(self, machine: 'CoffeeVendingMachine'):
        pass

class ReadyState(VendingMachineState):
    def select_coffee(self, machine: 'CoffeeVendingMachine', coffee: 'Coffee'):
        machine.set_selected_coffee(coffee)
        machine.set_state(SelectingState())
        print(f"{coffee.get_coffee_type()} selected. Price: {coffee.get_price()}")
    
    def insert_money(self, machine: 'CoffeeVendingMachine', amount: int):
        print("Please select a coffee first.")
    
    def dispense_coffee(self, machine: 'CoffeeVendingMachine'):
        print("Please select and pay first.")
    
    def cancel(self, machine: 'CoffeeVendingMachine'):
        print("Nothing to cancel.")

class SelectingState(VendingMachineState):
    def select_coffee(self, machine: 'CoffeeVendingMachine', coffee: 'Coffee'):
        print("Already selected. Please pay or cancel.")
    
    def insert_money(self, machine: 'CoffeeVendingMachine', amount: int):
        machine.set_money_inserted(machine.get_money_inserted() + amount)
        print(f"Inserted {amount}. Total: {machine.get_money_inserted()}")
        if machine.get_money_inserted() >= machine.get_selected_coffee().get_price():
            machine.set_state(PaidState())
    
    def dispense_coffee(self, machine: 'CoffeeVendingMachine'):
        print("Please insert enough money first.")
    
    def cancel(self, machine: 'CoffeeVendingMachine'):
        print(f"Transaction cancelled. Refunding {machine.get_money_inserted()}")
        machine.reset()
        machine.set_state(ReadyState())

class PaidState(VendingMachineState):
    def select_coffee(self, machine: 'CoffeeVendingMachine', coffee: 'Coffee'):
        print("Already paid. Please dispense or cancel.")
    
    def insert_money(self, machine: 'CoffeeVendingMachine', amount: int):
        machine.set_money_inserted(machine.get_money_inserted() + amount)
        print(f"Additional {amount} inserted. Total: {machine.get_money_inserted()}")
    
    def dispense_coffee(self, machine: 'CoffeeVendingMachine'):
        inventory = Inventory.get_instance()
        coffee = machine.get_selected_coffee()
        
        if not inventory.has_ingredients(coffee.get_recipe()):
            print("Sorry, we are out of ingredients. Refunding your money.")
            print(f"Refunding {machine.get_money_inserted()}")
            machine.reset()
            machine.set_state(OutOfIngredientState())
            return
        
        # Deduct ingredients and prepare coffee
        inventory.deduct_ingredients(coffee.get_recipe())
        coffee.prepare()
        
        # Calculate change
        change = machine.get_money_inserted() - coffee.get_price()
        if change > 0:
            print(f"Here's your change: {change}")
        
        machine.reset()
        machine.set_state(ReadyState())
    
    def cancel(self, machine: 'CoffeeVendingMachine'):
        print(f"Transaction cancelled. Refunding {machine.get_money_inserted()}")
        machine.reset()
        machine.set_state(ReadyState())


class OutOfIngredientState(VendingMachineState):
    def select_coffee(self, machine: 'CoffeeVendingMachine', coffee: 'Coffee'):
        print("Sorry, we are sold out.")
    
    def insert_money(self, machine: 'CoffeeVendingMachine', amount: int):
        print("Sorry, we are sold out. Money refunded.")
    
    def dispense_coffee(self, machine: 'CoffeeVendingMachine'):
        print("Sorry, we are sold out.")
    
    def cancel(self, machine: 'CoffeeVendingMachine'):
        print(f"Refunding {machine.get_money_inserted()}")
        machine.reset()
        machine.set_state(ReadyState())
```