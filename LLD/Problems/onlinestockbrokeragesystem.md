# Designing an Online Stock Brokerage System

## Question
Design an Online Stock Brokerage System

## Requirements
1. The online stock brokerage system should allow users to create and manage their trading accounts.
2. Users should be able to buy and sell stocks, as well as view their portfolio and transaction history.
3. The system should provide real-time stock quotes and market data to users.
4. The system should handle order placement, execution, and settlement processes.
5. The system should enforce various business rules and validations, such as checking account balances and stock availability.
6. The system should handle concurrent user requests and ensure data consistency and integrity.
7. The system should be scalable and able to handle a large number of users and transactions.
8. The system should be secure and protect sensitive user information.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: User, Account, Stock, Order, BuyOrder, SellOrder, OrderStatus, Portfolio,
StockBroker, InsufficientFundsException, InsufficientStockException, StockBrokerageSystem.

### Design Details
1. The **User** class represents a user of the stock brokerage system, with properties such as user ID, name, and email.
2. The **Account** class represents a user's trading account, with properties like account ID, associated user, and balance. It provides methods for depositing and withdrawing funds.
3. The **Stock** class represents a stock that can be traded, with properties such as symbol, name, and price. It provides a method for updating the stock price.
4. The **Order** class is an abstract base class representing an order placed by a user. It contains common properties such as order ID, associated account, stock, quantity, price, and order status. The execute() method is declared as abstract, to be implemented by concrete order classes.
5. The **BuyOrder** and **SellOrder** classes are concrete implementations of the Order class, representing buy and sell orders respectively. They provide the implementation for the execute() method specific to each order type.
6. The **OrderStatus** enum represents the possible statuses of an order, such as PENDING, EXECUTED, or REJECTED.
7. The **Portfolio** class represents a user's portfolio, which holds the stocks owned by the user. It provides methods for adding and removing stocks from the portfolio.
8. The **StockBroker** class is the central component of the stock brokerage system. It follows the Singleton pattern to ensure a single instance of the stock broker. It manages user accounts, stocks, and order processing. It provides methods for creating accounts, adding stocks, placing orders, and processing orders.
9. The **InsufficientFundsException** and **InsufficientStockException** classes are custom exceptions used to handle insufficient funds and insufficient stock scenarios respectively.
10. The **StockBrokerageSystem** class serves as the entry point of the application and demonstrates the usage of the stock brokerage system.

### Implementation (Python)
#### account.py

```python
import threading
from typing import Dict
from exceptions import InsufficientFundsException, InsufficientStockException

class Account:
    def __init__(self, initial_cash: float):
        self.balance = initial_cash
        self.portfolio: Dict[str, int] = {}  # Stock symbol -> quantity
        self.lock = threading.Lock()

    def debit(self, amount: float) -> None:
        with self.lock:
            if self.balance < amount:
                raise InsufficientFundsException(f"Insufficient funds to debit {amount}")
            self.balance -= amount

    def credit(self, amount: float) -> None:
        with self.lock:
            self.balance += amount

    def add_stock(self, symbol: str, quantity: int) -> None:
        with self.lock:
            self.portfolio[symbol] = self.portfolio.get(symbol, 0) + quantity

    def remove_stock(self, symbol: str, quantity: int) -> None:
        with self.lock:
            current_quantity = self.portfolio.get(symbol, 0)
            if current_quantity < quantity:
                raise InsufficientStockException(f"Not enough {symbol} stock to sell.")
            self.portfolio[symbol] = current_quantity - quantity

    def get_balance(self) -> float:
        return self.balance

    def get_portfolio(self) -> Dict[str, int]:
        return self.portfolio.copy()

    def get_stock_quantity(self, symbol: str) -> int:
        return self.portfolio.get(symbol, 0)
```

#### enums.py

```python
from enum import Enum

class OrderType(Enum):
    MARKET = "MARKET"
    LIMIT = "LIMIT"

class TransactionType(Enum):
    BUY = "BUY"
    SELL = "SELL"

class OrderStatus(Enum):
    OPEN = "OPEN"
    PARTIALLY_FILLED = "PARTIALLY_FILLED"
    FILLED = "FILLED"
    CANCELLED = "CANCELLED"
    FAILED = "FAILED"
```

#### exceptions.py

```python
class InsufficientFundsException(Exception):
    def __init__(self, message: str):
        super().__init__(message)

class InsufficientStockException(Exception):
    def __init__(self, message: str):
        super().__init__(message)
```

#### execution_strategy.py

```python
from abc import ABC, abstractmethod
from enums import TransactionType
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from order import Order

class ExecutionStrategy(ABC):
    @abstractmethod
    def can_execute(self, order: 'Order', market_price: float) -> bool:
        pass

class LimitOrderStrategy(ExecutionStrategy):
    def __init__(self, transaction_type: TransactionType):
        self.type = transaction_type

    def can_execute(self, order: 'Order', market_price: float) -> bool:
        if self.type == TransactionType.BUY:
            # Buy if market price is less than or equal to limit price
            return market_price <= order.get_price()
        else:  # SELL
            # Sell if market price is greater than or equal to limit price
            return market_price >= order.get_price()

class MarketOrderStrategy(ExecutionStrategy):
    def can_execute(self, order: 'Order', market_price: float) -> bool:
        return True  # Market orders can always execute
```

#### order.py

```python
from enums import OrderType, OrderStatus
from stock import Stock
from execution_strategy import ExecutionStrategy
from order_states import OrderState, OpenState
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class Order:
    def __init__(self, order_id: str, user: 'User', stock: Stock, order_type: OrderType, 
                 quantity: int, price: float, execution_strategy: ExecutionStrategy, owner: 'User'):
        self.order_id = order_id
        self.user = user
        self.stock = stock
        self.type = order_type
        self.quantity = quantity
        self.price = price  # Limit price for Limit orders
        self.execution_strategy = execution_strategy
        self.owner = owner
        self.current_state = OpenState()  # Initial state
        self.status = OrderStatus.OPEN

    def cancel(self) -> None:
        self.current_state.cancel(self)

    def get_order_id(self) -> str:
        return self.order_id

    def get_user(self) -> 'User':
        return self.user

    def get_stock(self) -> Stock:
        return self.stock

    def get_type(self) -> OrderType:
        return self.type

    def get_quantity(self) -> int:
        return self.quantity

    def get_price(self) -> float:
        return self.price

    def get_status(self) -> OrderStatus:
        return self.status

    def get_execution_strategy(self) -> ExecutionStrategy:
        return self.execution_strategy

    def set_state(self, state: OrderState) -> None:
        self.current_state = state

    def set_status(self, status: OrderStatus) -> None:
        self.status = status
        self._notify_owner()

    def _notify_owner(self) -> None:
        if self.owner:
            self.owner.order_status_update(self)
```

#### order_builder.py

```python
from enums import OrderType, TransactionType
from user import User
from stock import Stock
from order import Order
from execution_strategy import MarketOrderStrategy, LimitOrderStrategy
from typing import Optional
import uuid

class OrderBuilder:
    def __init__(self):
        self.user: Optional[User] = None
        self.stock: Optional[Stock] = None
        self.type: Optional[OrderType] = None
        self.transaction_type: Optional[TransactionType] = None
        self.quantity: int = 0
        self.price: float = 0.0

    def for_user(self, user: User) -> 'OrderBuilder':
        self.user = user
        return self

    def with_stock(self, stock: Stock) -> 'OrderBuilder':
        self.stock = stock
        return self

    def buy(self, quantity: int) -> 'OrderBuilder':
        self.transaction_type = TransactionType.BUY
        self.quantity = quantity
        return self

    def sell(self, quantity: int) -> 'OrderBuilder':
        self.transaction_type = TransactionType.SELL
        self.quantity = quantity
        return self

    def at_market_price(self) -> 'OrderBuilder':
        self.type = OrderType.MARKET
        self.price = 0  # Not needed for market order
        return self

    def with_limit(self, limit_price: float) -> 'OrderBuilder':
        self.type = OrderType.LIMIT
        self.price = limit_price
        return self

    def build(self) -> Order:
        execution_strategy = MarketOrderStrategy() if self.type == OrderType.MARKET else LimitOrderStrategy(self.transaction_type)
        return Order(
            str(uuid.uuid4()),
            self.user,
            self.stock,
            self.type,
            self.quantity,
            self.price,
            execution_strategy,
            self.user
        )
```

#### order_command.py

```python
from abc import ABC, abstractmethod
from account import Account
from order import Order
from enums import OrderType
from exceptions import InsufficientFundsException, InsufficientStockException
from stock_exchange import StockExchange

class OrderCommand(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass

class BuyStockCommand(OrderCommand):
    def __init__(self, account: Account, order: Order):
        self.account = account
        self.order = order
        self.stock_exchange = StockExchange.get_instance()

    def execute(self) -> None:
        # For market order, we can't pre-check funds perfectly.
        # For limit order, we can pre-authorize the amount.
        estimated_cost = self.order.get_quantity() * self.order.get_price()
        if self.order.get_type() == OrderType.LIMIT and self.account.get_balance() < estimated_cost:
            raise InsufficientFundsException("Not enough cash to place limit buy order.")
        print(f"Placing BUY order {self.order.get_order_id()} for {self.order.get_quantity()} shares of {self.order.get_stock().get_symbol()}.")
        self.stock_exchange.place_buy_order(self.order)

class SellStockCommand(OrderCommand):
    def __init__(self, account: Account, order: Order):
        self.account = account
        self.order = order
        self.stock_exchange = StockExchange.get_instance()

    def execute(self) -> None:
        if self.account.get_stock_quantity(self.order.get_stock().get_symbol()) < self.order.get_quantity():
            raise InsufficientStockException("Not enough stock to place sell order.")
        print(f"Placing SELL order {self.order.get_order_id()} for {self.order.get_quantity()} shares of {self.order.get_stock().get_symbol()}.")
        self.stock_exchange.place_sell_order(self.order)
```

#### order_states.py

```python
from abc import ABC, abstractmethod
from enums import OrderStatus
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from order import Order

class OrderState(ABC):
    @abstractmethod
    def handle(self, order: 'Order') -> None:
        pass

    @abstractmethod
    def cancel(self, order: 'Order') -> None:
        pass

class OpenState(OrderState):
    def handle(self, order: 'Order') -> None:
        print("Order is open and waiting for execution.")

    def cancel(self, order: 'Order') -> None:
        order.set_status(OrderStatus.CANCELLED)
        order.set_state(CancelledState())
        print(f"Order {order.get_order_id()} has been cancelled.")

class FilledState(OrderState):
    def handle(self, order: 'Order') -> None:
        print("Order is already filled.")

    def cancel(self, order: 'Order') -> None:
        print("Cannot cancel a filled order.")

class CancelledState(OrderState):
    def handle(self, order: 'Order') -> None:
        print("Order is cancelled.")

    def cancel(self, order: 'Order') -> None:
        print("Order is already cancelled.")
```

#### stock.py

```python
from typing import List
from stock_observer import StockObserver

class Stock:
    def __init__(self, symbol: str, initial_price: float):
        self.symbol = symbol
        self.price = initial_price
        self.observers: List[StockObserver] = []

    def get_symbol(self) -> str:
        return self.symbol

    def get_price(self) -> float:
        return self.price

    def set_price(self, new_price: float) -> None:
        if self.price != new_price:
            self.price = new_price
            self._notify_observers()

    def add_observer(self, observer: StockObserver) -> None:
        self.observers.append(observer)

    def remove_observer(self, observer: StockObserver) -> None:
        if observer in self.observers:
            self.observers.remove(observer)

    def _notify_observers(self) -> None:
        for observer in self.observers:
            observer.update(self)
```

#### stock_brokerage_system.py

```python
from typing import Dict, Optional
from user import User
from stock import Stock
from order import Order
from order_command import BuyStockCommand, SellStockCommand
import threading

class StockBrokerageSystem:
    _instance: Optional['StockBrokerageSystem'] = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if hasattr(self, 'initialized'):
            return
        self.users: Dict[str, User] = {}
        self.stocks: Dict[str, Stock] = {}
        self.initialized = True

    @classmethod
    def get_instance(cls) -> 'StockBrokerageSystem':
        return cls()

    def register_user(self, name: str, initial_amount: float) -> User:
        user = User(name, initial_amount)
        self.users[user.get_user_id()] = user
        return user

    def add_stock(self, symbol: str, initial_price: float) -> Stock:
        stock = Stock(symbol, initial_price)
        self.stocks[stock.get_symbol()] = stock
        return stock

    def place_buy_order(self, order: Order) -> None:
        user = order.get_user()
        command = BuyStockCommand(user.get_account(), order)
        command.execute()

    def place_sell_order(self, order: Order) -> None:
        user = order.get_user()
        command = SellStockCommand(user.get_account(), order)
        command.execute()

    def cancel_order(self, order: Order) -> None:
        order.cancel()
```

#### stock_brokerage_system_demo.py

```python
from user import User
from stock_brokerage_system import StockBrokerageSystem
import time
from order_builder import OrderBuilder

def print_account_status(user: User) -> None:
    print(f"Member: {user.get_name()}, Cash: ${user.get_account().get_balance():.2f}, Portfolio: {user.get_account().get_portfolio()}")

class StockBrokerageSystemDemo:
    def run():
        # System Setup
        system = StockBrokerageSystem.get_instance()

        # Create Stocks
        apple = system.add_stock("AAPL", 150.00)
        google = system.add_stock("GOOG", 2800.00)

        # Create Members (Users)
        alice = system.register_user("Alice", 20000.00)
        bob = system.register_user("Bob", 25000.00)

        # Bob already owns some Apple stock
        bob.get_account().add_stock("AAPL", 50)

        # Members subscribe to stock notifications (Observer Pattern)
        apple.add_observer(alice)
        google.add_observer(alice)
        apple.add_observer(bob)

        print("--- Initial State ---")
        print_account_status(alice)
        print_account_status(bob)

        print("\n--- Trading Simulation Starts ---\n")

        # SCENARIO 1: Limit Order Match
        print("--- SCENARIO 1: Alice places a limit buy, Bob places a limit sell that matches ---")

        # Alice wants to buy 10 shares of AAPL if the price is $150.50 or less
        alice_buy_order = OrderBuilder() \
            .for_user(alice) \
            .buy(10) \
            .with_stock(apple) \
            .with_limit(150.50) \
            .build()
        system.place_buy_order(alice_buy_order)

        # Bob wants to sell 20 of his shares if the price is $150.50 or more
        bob_sell_order = OrderBuilder() \
            .for_user(bob) \
            .sell(20) \
            .with_stock(apple) \
            .with_limit(150.50) \
            .build()
        system.place_sell_order(bob_sell_order)

        # The exchange will automatically match and execute this trade.
        # Let's check the status after the trade.
        time.sleep(0.1)  # Give time for notifications to print
        print("\n--- Account Status After Trade 1 ---")
        print_account_status(alice)
        print_account_status(bob)

        # SCENARIO 2: Price Update triggers notifications
        print("\n--- SCENARIO 2: Market price of GOOG changes ---")
        google.set_price(2850.00)  # Alice will get a notification

        # SCENARIO 3: Order Cancellation (State Pattern)
        print("\n--- SCENARIO 3: Alice places an order and then cancels it ---")
        alice_cancel_order = OrderBuilder() \
            .for_user(alice) \
            .buy(5) \
            .with_stock(google) \
            .with_limit(2700.00) \
            .build()  # Price is too low, so it won't execute immediately
        system.place_buy_order(alice_cancel_order)

        print(f"Order status before cancellation: {alice_cancel_order.get_status().value}")
        system.cancel_order(alice_cancel_order)
        print(f"Order status after cancellation attempt: {alice_cancel_order.get_status().value}")

        # Now try to cancel an already filled order
        print("\n--- Trying to cancel an already FILLED order (State Pattern) ---")
        print(f"Bob's sell order status: {bob_sell_order.get_status().value}")
        system.cancel_order(bob_sell_order)  # This should fail
        print(f"Bob's sell order status after cancel attempt: {bob_sell_order.get_status().value}")

if __name__ == "__main__":
    StockBrokerageSystemDemo.run()
```

#### stock_exchange.py

```python
from typing import Dict, List, Optional
from order import Order
from stock import Stock
from enums import OrderType, OrderStatus
from order_states import FilledState
import threading
from collections import defaultdict

class StockExchange:
    _instance: Optional['StockExchange'] = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if hasattr(self, 'initialized'):
            return
        self.buy_orders: Dict[str, List[Order]] = defaultdict(list)
        self.sell_orders: Dict[str, List[Order]] = defaultdict(list)
        self.match_lock = threading.Lock()
        self.initialized = True

    @classmethod
    def get_instance(cls) -> 'StockExchange':
        return cls()

    def place_buy_order(self, order: Order) -> None:
        self.buy_orders[order.get_stock().get_symbol()].append(order)
        self._match_orders(order.get_stock())

    def place_sell_order(self, order: Order) -> None:
        self.sell_orders[order.get_stock().get_symbol()].append(order)
        self._match_orders(order.get_stock())

    def _match_orders(self, stock: Stock) -> None:
        with self.match_lock:  # Critical section to prevent race conditions during matching
            buys = self.buy_orders.get(stock.get_symbol(), [])
            sells = self.sell_orders.get(stock.get_symbol(), [])

            if not buys or not sells:
                return

            match_found = True
            while match_found:
                match_found = False
                best_buy = self._find_best_buy(buys)
                best_sell = self._find_best_sell(sells)

                if best_buy and best_sell:
                    buy_price = stock.get_price() if best_buy.get_type() == OrderType.MARKET else best_buy.get_price()
                    sell_price = stock.get_price() if best_sell.get_type() == OrderType.MARKET else best_sell.get_price()

                    if buy_price >= sell_price:
                        self._execute_trade(best_buy, best_sell, sell_price)  # Trade at the seller's asking price
                        match_found = True

    def _execute_trade(self, buy_order: Order, sell_order: Order, trade_price: float) -> None:
        print(f"--- Executing Trade for {buy_order.get_stock().get_symbol()} at ${trade_price:.2f} ---")

        buyer = buy_order.get_user()
        seller = sell_order.get_user()

        trade_quantity = min(buy_order.get_quantity(), sell_order.get_quantity())
        total_cost = trade_quantity * trade_price

        # Perform transaction
        buyer.get_account().debit(total_cost)
        buyer.get_account().add_stock(buy_order.get_stock().get_symbol(), trade_quantity)

        seller.get_account().credit(total_cost)
        seller.get_account().remove_stock(sell_order.get_stock().get_symbol(), trade_quantity)

        # Update orders
        self._update_order_status(buy_order, trade_quantity)
        self._update_order_status(sell_order, trade_quantity)

        # Update stock's market price to last traded price
        buy_order.get_stock().set_price(trade_price)

        print("--- Trade Complete ---")

    def _update_order_status(self, order: Order, quantity_traded: int) -> None:
        # This is a simplified update logic. A real system would handle partial fills.
        order.set_status(OrderStatus.FILLED)
        order.set_state(FilledState())
        stock_symbol = order.get_stock().get_symbol()
        
        # Remove from books
        if order in self.buy_orders[stock_symbol]:
            self.buy_orders[stock_symbol].remove(order)
        if order in self.sell_orders[stock_symbol]:
            self.sell_orders[stock_symbol].remove(order)

    def _find_best_buy(self, buys: List[Order]) -> Optional[Order]:
        open_orders = [o for o in buys if o.get_status() == OrderStatus.OPEN]
        if not open_orders:
            return None
        return max(open_orders, key=lambda o: o.get_price())  # Highest limit price is best

    def _find_best_sell(self, sells: List[Order]) -> Optional[Order]:
        open_orders = [o for o in sells if o.get_status() == OrderStatus.OPEN]
        if not open_orders:
            return None
        return min(open_orders, key=lambda o: o.get_price())  # Lowest limit price is best
```

#### stock_observer.py

```python
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from stock import Stock

class StockObserver(ABC):
    @abstractmethod
    def update(self, stock: 'Stock') -> None:
        pass
```

#### user.py

```python
import uuid
from account import Account
from stock import Stock
from stock_observer import StockObserver
from order import Order

class User(StockObserver):
    def __init__(self, name: str, initial_cash: float):
        self.user_id = str(uuid.uuid4())
        self.name = name
        self.account = Account(initial_cash)

    def get_user_id(self) -> str:
        return self.user_id

    def get_name(self) -> str:
        return self.name

    def get_account(self) -> Account:
        return self.account

    def update(self, stock: Stock) -> None:
        print(f"[Notification for {self.name}] Stock {stock.get_symbol()} price updated to: ${stock.get_price():.2f}")

    def order_status_update(self, order: 'Order') -> None:
        print(f"[Order Notification for {self.name}] Order {order.get_order_id()} for {order.get_stock().get_symbol()} is now {order.get_status().value}.")
```