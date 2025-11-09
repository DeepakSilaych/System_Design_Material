# Designing an Online Auction System

## Question
Design an Online Auction System

## Requirements
1. The online auction system should allow users to register and log in to their accounts.
2. Users should be able to create new auction listings with details such as item name, description, starting price, and auction duration.
3. Users should be able to browse and search for auction listings based on various criteria (e.g., item name, category, price range).
4. Users should be able to place bids on active auction listings.
5. The system should automatically update the current highest bid and notify the bidders accordingly.
6. The auction should end when the specified duration is reached, and the highest bidder should be declared the winner.
7. The system should handle concurrent access to auction listings and ensure data consistency.
8. The system should be extensible to accommodate future enhancements and new features.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: User, AuctionStatus, AuctionListing, Bid, AuctionSystem, AuctionSystemDemo.

### Design Details
1. The **User** class represents a user in the online auction system, with properties such as id, username, and email.
2. The **AuctionStatus** enum defines the possible states of an auction listing, such as active and closed.
3. The **AuctionListing** class represents an auction listing in the system, with properties like id, item name, description, starting price, duration, seller, current highest bid, and a list of bids.
4. The **Bid** class represents a bid placed by a user on an auction listing, with properties such as id, bidder, amount, and timestamp.
5. The **AuctionSystem** class is the core of the online auction system and follows the Singleton pattern to ensure a single instance of the auction system.
6. The AuctionSystem class uses concurrent data structures (ConcurrentHashMap and CopyOnWriteArrayList) to handle concurrent access to auction listings and ensure thread safety.
7. The AuctionSystem class provides methods for registering users, creating auction listings, searching auction listings, and placing bids.
8. The **AuctionSystemDemo** class serves as the entry point of the application and demonstrates the usage of the online auction system.

### Implementation (Python)
#### auction.py

```python
from typing import List, Set, Optional
from bid import Bid
from auction_state import AuctionState
from datetime import datetime
from decimal import Decimal
import uuid
import threading
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User
    from auction_observer import AuctionObserver

class Auction:
    def __init__(self, item_name: str, description: str, starting_price: Decimal, end_time: datetime):
        self.id = str(uuid.uuid4())
        self.item_name = item_name
        self.description = description
        self.starting_price = starting_price
        self.end_time = end_time
        self.bids: List[Bid] = []
        self.observers: Set[AuctionObserver] = set()
        self.state = AuctionState.ACTIVE
        self.winning_bid: Optional[Bid] = None
        self._lock = threading.RLock()

    def place_bid(self, bidder: 'User', amount: Decimal):
        with self._lock:
            if self.state != AuctionState.ACTIVE:
                raise Exception("Auction is not active.")
            if datetime.now() > self.end_time:
                self.end_auction()
                raise Exception("Auction has already ended.")

            highest_bid = self.get_highest_bid()
            current_max_amount = self.starting_price if highest_bid is None else highest_bid.get_amount()

            if amount <= current_max_amount:
                raise ValueError("Bid must be higher than the current highest bid.")

            previous_highest_bidder = highest_bid.get_bidder() if highest_bid is not None else None

            new_bid = Bid(bidder, amount)
            self.bids.append(new_bid)
            self.add_observer(bidder)

            print(f"SUCCESS: {bidder.get_name()} placed a bid of ${amount:.2f} on '{self.item_name}'.")

            if previous_highest_bidder is not None and previous_highest_bidder != bidder:
                self.notify_observer(previous_highest_bidder, 
                    f"You have been outbid on '{self.item_name}'! The new highest bid is ${amount:.2f}.")

    def end_auction(self):
        with self._lock:
            if self.state != AuctionState.ACTIVE:
                return

            self.state = AuctionState.CLOSED
            self.winning_bid = self.get_highest_bid()

            if self.winning_bid is not None:
                end_message = f"Auction for '{self.item_name}' has ended. Winner is {self.winning_bid.get_bidder().get_name()} with a bid of ${self.winning_bid.get_amount():.2f}!"
            else:
                end_message = f"Auction for '{self.item_name}' has ended. There were no bids."

            print(f"\n{end_message.upper()}")
            self.notify_all_observers(end_message)

    def get_highest_bid(self) -> Optional[Bid]:
        if not self.bids:
            return None
        return max(self.bids)

    def is_active(self) -> bool:
        return self.state == AuctionState.ACTIVE

    def add_observer(self, observer: 'AuctionObserver'):
        self.observers.add(observer)

    def notify_all_observers(self, message: str):
        for observer in self.observers:
            observer.on_update(self, message)

    def notify_observer(self, observer: 'AuctionObserver', message: str):
        observer.on_update(self, message)

    def get_id(self) -> str:
        return self.id

    def get_item_name(self) -> str:
        return self.item_name

    def get_bid_history(self) -> List[Bid]:
        return self.bids.copy()

    def get_state(self) -> AuctionState:
        return self.state

    def get_winning_bid(self) -> Optional[Bid]:
        return self.winning_bid
```

#### auction_observer.py

```python
from abc import ABC, abstractmethod
from auction import Auction

class AuctionObserver(ABC):
    @abstractmethod
    def on_update(self, auction: 'Auction', message: str):
        pass
```

#### auction_service.py

```python
from user import User
from auction import Auction
from typing import Dict, List
from decimal import Decimal
from datetime import datetime
import threading
from concurrent.futures import ThreadPoolExecutor
import time

class AuctionService:
    _instance = None
    _lock = threading.Lock()

    def __init__(self):
        if AuctionService._instance is not None:
            raise Exception("This class is a singleton!")
        self.users: Dict[str, User] = {}
        self.auctions: Dict[str, Auction] = {}
        self.scheduler = ThreadPoolExecutor(max_workers=10)
        self._shutdown = False

    @staticmethod
    def get_instance():
        if AuctionService._instance is None:
            with AuctionService._lock:
                if AuctionService._instance is None:
                    AuctionService._instance = AuctionService()
        return AuctionService._instance

    def create_user(self, name: str) -> User:
        user = User(name)
        self.users[user.get_id()] = user
        return user

    def get_user(self, user_id: str) -> User:
        return self.users[user_id]

    def create_auction(self, item_name: str, description: str, starting_price: Decimal, end_time: datetime) -> Auction:
        auction = Auction(item_name, description, starting_price, end_time)
        self.auctions[auction.get_id()] = auction

        delay = (end_time - datetime.now()).total_seconds()
        if delay > 0:
            self.scheduler.submit(self._scheduled_end_auction, auction.get_id(), delay)

        print(f"New auction created for '{item_name}' (ID: {auction.get_id()}), ending at {end_time}.")
        return auction

    def _scheduled_end_auction(self, auction_id: str, delay: float):
        time.sleep(delay)
        if not self._shutdown:
            self.end_auction(auction_id)

    def view_active_auctions(self) -> List[Auction]:
        return [auction for auction in self.auctions.values() if auction.is_active()]

    def place_bid(self, auction_id: str, bidder_id: str, amount: Decimal):
        auction = self.get_auction(auction_id)
        auction.place_bid(self.users[bidder_id], amount)

    def end_auction(self, auction_id: str):
        auction = self.get_auction(auction_id)
        auction.end_auction()

    def get_auction(self, auction_id: str) -> Auction:
        auction = self.auctions.get(auction_id)
        if auction is None:
            raise KeyError(f"Auction with ID {auction_id} not found.")
        return auction

    def shutdown(self):
        self._shutdown = True
        self.scheduler.shutdown(wait=True)
```

#### auction_state.py

```python
from enum import Enum

class AuctionState(Enum):
    PENDING = "PENDING"
    ACTIVE = "ACTIVE"
    CLOSED = "CLOSED"
```

#### auction_system_demo.py

```python
from auction_service import AuctionService
from user import User
from auction import Auction
from bid import Bid
from typing import List
from decimal import Decimal
from datetime import datetime, timedelta
import time

class AuctionSystemDemo:
    @staticmethod
    def main():
        auction_service = AuctionService.get_instance()

        alice = auction_service.create_user("Alice")
        bob = auction_service.create_user("Bob")
        carol = auction_service.create_user("Carol")

        print("=============================================")
        print("        Online Auction System Demo           ")
        print("=============================================")

        end_time = datetime.now() + timedelta(seconds=10)
        laptop_auction = auction_service.create_auction(
            "Vintage Laptop",
            "A rare 1990s laptop, in working condition.",
            Decimal("100.00"),
            end_time
        )
        print()

        try:
            auction_service.place_bid(laptop_auction.get_id(), alice.get_id(), Decimal("110.00"))
            time.sleep(0.5)

            auction_service.place_bid(laptop_auction.get_id(), bob.get_id(), Decimal("120.00"))
            time.sleep(0.5)

            auction_service.place_bid(laptop_auction.get_id(), carol.get_id(), Decimal("125.00"))
            time.sleep(0.5)

            auction_service.place_bid(laptop_auction.get_id(), alice.get_id(), Decimal("150.00"))

            print("\n--- Waiting for auction to end automatically... ---")
            time.sleep(2)
        except Exception as e:
            print(f"An error occurred during bidding: {e}")

        print("\n--- Post-Auction Information ---")
        ended_auction = auction_service.get_auction(laptop_auction.get_id())

        if ended_auction.get_winning_bid() is not None:
            print(f"Final Winner: {ended_auction.get_winning_bid().get_bidder().get_name()}")
            print(f"Winning Price: ${ended_auction.get_winning_bid().get_amount():.2f}")
        else:
            print("The auction ended with no winner.")

        print("\nFull Bid History:")
        for bid in ended_auction.get_bid_history():
            print(bid)

        print("\n--- Attempting to bid on an ended auction ---")
        try:
            auction_service.place_bid(laptop_auction.get_id(), bob.get_id(), Decimal("200.00"))
        except Exception as e:
            print(f"CAUGHT EXPECTED ERROR: {e}")

        auction_service.shutdown()


if __name__ == "__main__":
    AuctionSystemDemo.main()
```

#### bid.py

```python
from datetime import datetime
from decimal import Decimal
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class Bid:
    def __init__(self, bidder: 'User', amount: Decimal):
        self.bidder = bidder
        self.amount = amount
        self.timestamp = datetime.now()

    def get_bidder(self) -> 'User':
        return self.bidder

    def get_amount(self) -> Decimal:
        return self.amount

    def get_timestamp(self) -> datetime:
        return self.timestamp

    def __lt__(self, other: 'Bid') -> bool:
        if self.amount != other.amount:
            return self.amount < other.amount
        return self.timestamp > other.timestamp

    def __eq__(self, other: 'Bid') -> bool:
        return self.amount == other.amount and self.timestamp == other.timestamp

    def __le__(self, other: 'Bid') -> bool:
        return self < other or self == other

    def __gt__(self, other: 'Bid') -> bool:
        return not self <= other

    def __ge__(self, other: 'Bid') -> bool:
        return not self < other

    def __ne__(self, other: 'Bid') -> bool:
        return not self == other

    def __str__(self) -> str:
        return f"Bidder: {self.bidder.get_name()}, Amount: {self.amount:.2f}, Time: {self.timestamp}"
```

#### user.py

```python
import uuid
from auction_observer import AuctionObserver
from auction import Auction

class User(AuctionObserver):
    def __init__(self, name: str):
        self.id = str(uuid.uuid4())
        self.name = name

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name

    def on_update(self, auction: 'Auction', message: str):
        print(f"--- Notification for {self.name} ---")
        print(f"Auction: {auction.get_item_name()}")
        print(f"Message: {message}")
        print("---------------------------\n")
```