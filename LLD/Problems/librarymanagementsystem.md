# Designing a Library Management System

## Question
Design a Library Management System

## Requirements
1. The library management system should allow librarians to manage books, members, and borrowing activities.
2. The system should support adding, updating, and removing books from the library catalog.
3. Each book should have details such as title, author, ISBN, publication year, and availability status.
4. The system should allow members to borrow and return books.
5. Each member should have details such as name, member ID, contact information, and borrowing history.
6. The system should enforce borrowing rules, such as a maximum number of books that can be borrowed at a time and loan duration.
7. The system should handle concurrent access to the library catalog and member records.
8. The system should be extensible to accommodate future enhancements and new features.

## Solution
### Design overview
The design treats each physical copy as the unit of circulation (not the title). State transitions on
copies are guarded; holds and notifications are implemented using the Observer pattern.

- Factory builds items (books, magazines) behind a single API.
- Strategy provides pluggable search.
- Circulation is tracked by a `TransactionService` and copy-level state machine.

### Core model
- `LibraryItem` (abstract) with `Book`, `Magazine` (+ `ItemFactory`)
- `BookCopy` (Available → CheckedOut → OnHold)
- `Member` (observer for holds)
- `LibraryManagementSystem` (facade for catalog, members, circulation)
- `TransactionService` (active loans)
- `SearchStrategy` variants

### Key flows
1. Add item(s) and create physical copies
2. Checkout → create loan and transition to `CheckedOut`
3. Return → transition to `Available` or `OnHold` if waiters exist, then notify observers
4. Holds → members subscribe as observers and are notified on availability

### Design Details
1. The **Book** class represents a book in the library catalog, with properties such as ISBN, title, author, publication year, and availability status.
2. The **Member** class represents a library member, with properties like member ID, name, contact information, and a list of borrowed books.
3. The **LibraryManager** class is the core of the library management system and follows the Singleton pattern to ensure a single instance of the library manager.
4. The LibraryManager class uses concurrent data structures (ConcurrentHashMap) to handle concurrent access to the library catalog and member records.
5. The LibraryManager class provides methods for adding and removing books, registering and unregistering members, borrowing and returning books, and searching for books based on keywords.
6. The **LibraryManagementSystemDemo** class serves as the entry point of the application and demonstrates the usage of the library management system.

### Implementation (Python)
#### book.py

```python
from library_item import LibraryItem

class Book(LibraryItem):
    def __init__(self, item_id: str, title: str, author: str):
        super().__init__(item_id, title)
        self.author = author

    def get_author_or_publisher(self) -> str:
        return self.author
```

#### book_copy.py

```python
from typing import TYPE_CHECKING
from item_states import AvailableState

if TYPE_CHECKING:
    from member import Member
    from library_item import LibraryItem
    from item_states import ItemState

class BookCopy:
    def __init__(self, copy_id: str, item: 'LibraryItem'):
        self.id = copy_id
        self.item = item
        self.current_state: 'ItemState' = AvailableState()
        item.add_copy(self)

    def checkout(self, member: 'Member') -> None:
        self.current_state.checkout(self, member)

    def return_item(self) -> None:
        self.current_state.return_item(self)

    def place_hold(self, member: 'Member') -> None:
        self.current_state.place_hold(self, member)

    def set_state(self, state: 'ItemState') -> None:
        self.current_state = state

    def get_id(self) -> str:
        return self.id

    def get_item(self) -> 'LibraryItem':
        return self.item

    def is_available(self) -> bool:
        return isinstance(self.current_state, AvailableState)
```

#### item_factory.py

```python
from item_type import ItemType
from book import Book
from magazine import Magazine
from library_item import LibraryItem

class ItemFactory:
    @staticmethod
    def create_item(item_type: ItemType, item_id: str, title: str, author: str) -> LibraryItem:
        if item_type == ItemType.BOOK:
            return Book(item_id, title, author)
        elif item_type == ItemType.MAGAZINE:
            return Magazine(item_id, title, author)  # Author might be publisher here
        else:
            raise ValueError("Unknown item type")
```

#### item_states.py

```python
from abc import ABC, abstractmethod
from transaction_service import TransactionService
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from book_copy import BookCopy
    from member import Member

class ItemState(ABC):
    @abstractmethod
    def checkout(self, book_copy: 'BookCopy', member: 'Member') -> None:
        pass

    @abstractmethod
    def return_item(self, book_copy: 'BookCopy') -> None:
        pass

    @abstractmethod
    def place_hold(self, book_copy: 'BookCopy', member: 'Member') -> None:
        pass


class AvailableState(ItemState):
    def checkout(self, book_copy: 'BookCopy', member: 'Member') -> None:
        TransactionService.get_instance().create_loan(book_copy, member)
        book_copy.set_state(CheckedOutState())
        print(f"{book_copy.get_id()} checked out by {member.get_name()}")

    def return_item(self, book_copy: 'BookCopy') -> None:
        print("Cannot return an item that is already available.")

    def place_hold(self, book_copy: 'BookCopy', member: 'Member') -> None:
        print("Cannot place hold on an available item. Please check it out.")


class CheckedOutState(ItemState):
    def checkout(self, book_copy: 'BookCopy', member: 'Member') -> None:
        print(f"{book_copy.get_id()} is already checked out.")

    def return_item(self, book_copy: 'BookCopy') -> None:
        TransactionService.get_instance().end_loan(book_copy)
        print(f"{book_copy.get_id()} returned.")
        
        # If there are holds, move to OnHold state. Otherwise, become Available.
        if book_copy.get_item().has_observers():
            book_copy.set_state(OnHoldState())
            book_copy.get_item().notify_observers()  # Notify members that item is back but on hold
        else:
            book_copy.set_state(AvailableState())

    def place_hold(self, book_copy: 'BookCopy', member: 'Member') -> None:
        book_copy.get_item().add_observer(member)
        print(f"{member.get_name()} placed a hold on '{book_copy.get_item().get_title()}'")


class OnHoldState(ItemState):
    def checkout(self, book_copy: 'BookCopy', member: 'Member') -> None:
        # Only a member who placed the hold can check it out.
        if book_copy.get_item().is_observer(member):
            TransactionService.get_instance().create_loan(book_copy, member)
            book_copy.get_item().remove_observer(member)  # Remove from waiting list
            book_copy.set_state(CheckedOutState())
            print(f"Hold fulfilled. {book_copy.get_id()} checked out by {member.get_name()}")
        else:
            print("This item is on hold for another member.")

    def return_item(self, book_copy: 'BookCopy') -> None:
        print("Invalid action. Item is on hold, not checked out.")

    def place_hold(self, book_copy: 'BookCopy', member: 'Member') -> None:
        print("Item is already on hold.")
```

#### item_type.py

```python
from enum import Enum

class ItemType(Enum):
    BOOK = "BOOK"
    MAGAZINE = "MAGAZINE"
```

#### library_item.py

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from member import Member
from book_copy import BookCopy

class LibraryItem(ABC):
    def __init__(self, item_id: str, title: str):
        self.id = item_id
        self.title = title
        self.copies: List['BookCopy'] = []
        self.observers: List[Member] = []  # Observer Pattern: List of members waiting for this item

    def add_copy(self, book_copy: 'BookCopy') -> None:
        self.copies.append(book_copy)

    def add_observer(self, member: Member) -> None:
        self.observers.append(member)

    def remove_observer(self, member: Member) -> None:
        if member in self.observers:
            self.observers.remove(member)

    def notify_observers(self) -> None:
        print(f"Notifying {len(self.observers)} observers for '{self.title}'...")
        # Use a copy to avoid modification during iteration
        for observer in self.observers.copy():
            observer.update(self)

    def get_available_copy(self) -> Optional['BookCopy']:
        for book_copy in self.copies:
            if book_copy.is_available():
                return book_copy
        return None

    def get_id(self) -> str:
        return self.id

    def get_title(self) -> str:
        return self.title

    def get_copies(self) -> List['BookCopy']:
        return self.copies

    @abstractmethod
    def get_author_or_publisher(self) -> str:
        pass

    def get_available_copy_count(self) -> int:
        return sum(1 for book_copy in self.copies if book_copy.is_available())

    def has_observers(self) -> bool:
        return len(self.observers) > 0

    def is_observer(self, member: Member) -> bool:
        return member in self.observers
```

#### library_management_system.py

```python
from typing import Dict, List, Optional
from library_item import LibraryItem
from member import Member
from book_copy import BookCopy
from item_type import ItemType
from item_factory import ItemFactory
from search_strategy import SearchStrategy

class LibraryManagementSystem:
    _instance: Optional['LibraryManagementSystem'] = None

    def __init__(self):
        if LibraryManagementSystem._instance is not None:
            raise Exception("This class is a singleton!")
        self.catalog: Dict[str, LibraryItem] = {}
        self.members: Dict[str, Member] = {}
        self.copies: Dict[str, BookCopy] = {}

    @staticmethod
    def get_instance() -> 'LibraryManagementSystem':
        if LibraryManagementSystem._instance is None:
            LibraryManagementSystem._instance = LibraryManagementSystem()
        return LibraryManagementSystem._instance

    def add_item(self, item_type: ItemType, item_id: str, title: str, author: str, num_copies: int) -> List[BookCopy]:
        book_copies = []
        item = ItemFactory.create_item(item_type, item_id, title, author)
        self.catalog[item_id] = item
        
        for i in range(num_copies):
            copy_id = f"{item_id}-c{i + 1}"
            book_copy = BookCopy(copy_id, item)
            self.copies[copy_id] = book_copy
            book_copies.append(book_copy)
        
        print(f"Added {num_copies} copies of '{title}'")
        return book_copies

    def add_member(self, member_id: str, name: str) -> Member:
        member = Member(member_id, name)
        self.members[member_id] = member
        return member

    def checkout(self, member_id: str, copy_id: str) -> None:
        member = self.members.get(member_id)
        book_copy = self.copies.get(copy_id)
        
        if member is not None and book_copy is not None:
            book_copy.checkout(member)
        else:
            print("Error: Invalid member or copy ID.")

    def return_item(self, copy_id: str) -> None:
        book_copy = self.copies.get(copy_id)
        if book_copy is not None:
            book_copy.return_item()
        else:
            print("Error: Invalid copy ID.")

    def place_hold(self, member_id: str, item_id: str) -> None:
        member = self.members.get(member_id)
        item = self.catalog.get(item_id)
        
        if member is not None and item is not None:
            # Place hold on any copy that is checked out
            for book_copy in item.get_copies():
                if not book_copy.is_available():
                    book_copy.place_hold(member)
                    break

    def search(self, query: str, strategy: SearchStrategy) -> List[LibraryItem]:
        return strategy.search(query, list(self.catalog.values()))

    def print_catalog(self) -> None:
        print("\n--- Library Catalog ---")
        for item in self.catalog.values():
            print(f"ID: {item.get_id()}, Title: {item.get_title()}, "
                  f"Author/Publisher: {item.get_author_or_publisher()}, "
                  f"Available: {item.get_available_copy_count()}")
        print("-----------------------\n")
```

#### library_management_system_demo.py

```python
from library_management_system import LibraryManagementSystem
from item_type import ItemType
from search_strategy import SearchByTitleStrategy, SearchByAuthorStrategy

class LibraryManagementDemo:
    @staticmethod
    def main():
        library = LibraryManagementSystem.get_instance()

        # === Setup: Add items and members using the Facade ===
        print("=== Setting up the Library ===")

        hobbit_copies = library.add_item(ItemType.BOOK, "B001", "The Hobbit", "J.R.R. Tolkien", 2)
        dune_copies = library.add_item(ItemType.BOOK, "B002", "Dune", "Frank Herbert", 1)
        nat_geo_copies = library.add_item(ItemType.MAGAZINE, "M001", "National Geographic", "NatGeo Society", 3)

        alice = library.add_member("MEM01", "Alice")
        bob = library.add_member("MEM02", "Bob")
        charlie = library.add_member("MEM03", "Charlie")
        library.print_catalog()

        # === Scenario 1: Searching (Strategy Pattern) ===
        print("\n=== Scenario 1: Searching for Items ===")
        print("Searching for title 'Dune':")
        for item in library.search("Dune", SearchByTitleStrategy()):
            print(f"Found: {item.get_title()}")
        
        print("\nSearching for author 'Tolkien':")
        for item in library.search("Tolkien", SearchByAuthorStrategy()):
            print(f"Found: {item.get_title()}")

        # === Scenario 2: Checkout and Return (State Pattern) ===
        print("\n\n=== Scenario 2: Checkout and Return ===")
        library.checkout(alice.get_id(), hobbit_copies[0].get_id())  # Alice checks out The Hobbit copy 1
        library.checkout(bob.get_id(), dune_copies[0].get_id())     # Bob checks out Dune copy 1
        library.print_catalog()

        print("Attempting to checkout an already checked-out book:")
        library.checkout(charlie.get_id(), hobbit_copies[0].get_id())  # Charlie fails to check out The Hobbit copy 1

        print("\nAlice returns The Hobbit:")
        library.return_item(hobbit_copies[0].get_id())
        library.print_catalog()

        # === Scenario 3: Holds and Notifications (Observer Pattern) ===
        print("\n\n=== Scenario 3: Placing a Hold ===")
        print("Dune is checked out by Bob. Charlie places a hold.")
        library.place_hold(charlie.get_id(), "B002")  # Charlie places a hold on Dune

        print("\nBob returns Dune. Charlie should be notified.")
        library.return_item(dune_copies[0].get_id())  # Bob returns Dune

        print("\nCharlie checks out the book that was on hold for him.")
        library.checkout(charlie.get_id(), dune_copies[0].get_id())

        print("\nTrying to check out the same on-hold item by another member (Alice):")
        library.checkout(alice.get_id(), dune_copies[0].get_id())  # Alice fails, it's checked out by Charlie now.

        library.print_catalog()


if __name__ == "__main__":
    LibraryManagementDemo.main()
```

#### loan.py

```python
from datetime import date
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from book_copy import BookCopy
    from member import Member

class Loan:
    def __init__(self, book_copy: 'BookCopy', member: 'Member'):
        self.copy = book_copy
        self.member = member
        self.checkout_date = date.today()

    def get_copy(self) -> 'BookCopy':
        return self.copy

    def get_member(self) -> 'Member':
        return self.member
```

#### magazine.py

```python
from library_item import LibraryItem

class Magazine(LibraryItem):
    def __init__(self, item_id: str, title: str, publisher: str):
        super().__init__(item_id, title)
        self.publisher = publisher

    def get_author_or_publisher(self) -> str:
        return self.publisher
```

#### member.py

```python
from typing import List
from loan import Loan
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from library_item import LibraryItem
    from loan import Loan

class Member:
    def __init__(self, member_id: str, name: str):
        self.id = member_id
        self.name = name
        self.loans: List['Loan'] = []

    def update(self, item: 'LibraryItem') -> None:
        """Observer update method"""
        print(f"NOTIFICATION for {self.name}: The book '{item.get_title()}' you placed a hold on is now available!")

    def add_loan(self, loan: 'Loan') -> None:
        self.loans.append(loan)

    def remove_loan(self, loan: 'Loan') -> None:
        if loan in self.loans:
            self.loans.remove(loan)

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name

    def get_loans(self) -> List['Loan']:
        return self.loans
```

#### search_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from library_item import LibraryItem

class SearchStrategy(ABC):
    @abstractmethod
    def search(self, query: str, items: List[LibraryItem]) -> List[LibraryItem]:
        pass

class SearchByTitleStrategy(SearchStrategy):
    def search(self, query: str, items: List[LibraryItem]) -> List[LibraryItem]:
        return [item for item in items if query.lower() in item.get_title().lower()]

class SearchByAuthorStrategy(SearchStrategy):
    def search(self, query: str, items: List[LibraryItem]) -> List[LibraryItem]:
        return [item for item in items if query.lower() in item.get_author_or_publisher().lower()]
```

#### transaction_service.py

```python
from typing import Dict, Optional, TYPE_CHECKING
from loan import Loan

if TYPE_CHECKING:
    from book_copy import BookCopy
    from member import Member

class TransactionService:
    _instance: Optional['TransactionService'] = None

    def __init__(self):
        if TransactionService._instance is not None:
            raise Exception("This class is a singleton!")
        self.active_loans: Dict[str, Loan] = {}  # Key: BookCopy ID

    @staticmethod
    def get_instance() -> 'TransactionService':
        if TransactionService._instance is None:
            TransactionService._instance = TransactionService()
        return TransactionService._instance

    def create_loan(self, book_copy: 'BookCopy', member: 'Member') -> None:
        if book_copy.get_id() in self.active_loans:
            raise ValueError("This copy is already on loan.")
        
        loan = Loan(book_copy, member)
        self.active_loans[book_copy.get_id()] = loan
        member.add_loan(loan)

    def end_loan(self, book_copy: 'BookCopy') -> None:
        loan = self.active_loans.pop(book_copy.get_id(), None)
        if loan is not None:
            loan.get_member().remove_loan(loan)
```