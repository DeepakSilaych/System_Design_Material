# Designing Splitwise

## Question
Design Splitwise

## Requirements
1. The system should allow users to create accounts and manage their profile information.
2. Users should be able to create groups and add other users to the groups.
3. Users should be able to add expenses within a group, specifying the amount, description, and participants.
4. The system should automatically split the expenses among the participants based on their share.
5. Users should be able to view their individual balances with other users and settle up the balances.
6. The system should support different split methods, such as equal split, percentage split, and exact amounts.
7. Users should be able to view their transaction history and group expenses.
8. The system should handle concurrent transactions and ensure data consistency.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: User, Group, Expense, Split, Transaction, SplitwiseService, SplitwiseDemo.

### Design Details
1. The **User** class represents a user in the Splitwise system, with properties such as ID, name, email, and a map to store balances with other users.
2. The **Group** class represents a group in Splitwise, containing a list of member users and a list of expenses.
3. The **Expense** class represents an expense within a group, with properties such as ID, amount, description, the user who paid, and a list of splits.
4. The **Split** class is an abstract class representing the split of an expense. It is extended by EqualSplit, PercentSplit, and ExactSplit classes to handle different split methods.
5. The **Transaction** class represents a transaction between two users, with properties such as ID, sender, receiver, and amount.
6. The **SplitwiseService** class is the main class that manages the Splitwise system. It follows the Singleton pattern to ensure only one instance of the service exists.
7. The SplitwiseService class provides methods for adding users, groups, and expenses, splitting expenses, updating balances, settling balances, and creating transactions.
8. Multi-threading is achieved using concurrent data structures such as ConcurrentHashMap and CopyOnWriteArrayList to handle concurrent access to shared resources.
9. The **SplitwiseDemo** class demonstrates the usage of the Splitwise system by creating users, a group, adding an expense, settling balances, and printing user balances.

### Implementation (Python)
#### balance_sheet.py

```python
import threading
from typing import Dict, TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class BalanceSheet:
    def __init__(self, owner: 'User'):
        self._owner = owner
        self._balances: Dict['User', float] = {}
        self._lock = threading.Lock()
    
    def get_balances(self) -> Dict['User', float]:
        return self._balances
    
    def adjust_balance(self, other_user: 'User', amount: float):
        with self._lock:
            if self._owner == other_user:
                return  # Cannot owe yourself
            
            if other_user in self._balances:
                self._balances[other_user] += amount
            else:
                self._balances[other_user] = amount
    
    def show_balances(self):
        print(f"--- Balance Sheet for {self._owner.get_name()} ---")
        if not self._balances:
            print("All settled up!")
            return
        
        total_owed_to_me = 0
        total_i_owe = 0
        
        for other_user, amount in self._balances.items():
            if amount > 0.01:
                print(f"{other_user.get_name()} owes {self._owner.get_name()} ${amount:.2f}")
                total_owed_to_me += amount
            elif amount < -0.01:
                print(f"{self._owner.get_name()} owes {other_user.get_name()} ${-amount:.2f}")
                total_i_owe += (-amount)
        
        print(f"Total Owed to {self._owner.get_name()}: ${total_owed_to_me:.2f}")
        print(f"Total {self._owner.get_name()} Owes: ${total_i_owe:.2f}")
        print("---------------------------------")
```

#### expense.py

```python
from datetime import datetime
from typing import List, Optional
from user import User
from split import Split
from split_strategy import SplitStrategy

class Expense:
    def __init__(self, builder: 'ExpenseBuilder'):
        self._id = builder._id
        self._description = builder._description
        self._amount = builder._amount
        self._paid_by = builder._paid_by
        self._timestamp = datetime.now()
        
        # Use the strategy to calculate splits
        self._splits = builder._split_strategy.calculate_splits(
            builder._amount, builder._paid_by, builder._participants, builder._split_values
        )
    
    def get_id(self) -> str:
        return self._id
    
    def get_description(self) -> str:
        return self._description
    
    def get_amount(self) -> float:
        return self._amount
    
    def get_paid_by(self) -> User:
        return self._paid_by
    
    def get_splits(self) -> List[Split]:
        return self._splits
    
    class ExpenseBuilder:
        def __init__(self):
            self._id: Optional[str] = None
            self._description: Optional[str] = None
            self._amount: Optional[float] = None
            self._paid_by: Optional[User] = None
            self._participants: Optional[List[User]] = None
            self._split_strategy: Optional[SplitStrategy] = None
            self._split_values: Optional[List[float]] = None
        
        def set_id(self, expense_id: str) -> 'Expense.ExpenseBuilder':
            self._id = expense_id
            return self
        
        def set_description(self, description: str) -> 'Expense.ExpenseBuilder':
            self._description = description
            return self
        
        def set_amount(self, amount: float) -> 'Expense.ExpenseBuilder':
            self._amount = amount
            return self
        
        def set_paid_by(self, paid_by: User) -> 'Expense.ExpenseBuilder':
            self._paid_by = paid_by
            return self
        
        def set_participants(self, participants: List[User]) -> 'Expense.ExpenseBuilder':
            self._participants = participants
            return self
        
        def set_split_strategy(self, split_strategy: SplitStrategy) -> 'Expense.ExpenseBuilder':
            self._split_strategy = split_strategy
            return self
        
        def set_split_values(self, split_values: List[float]) -> 'Expense.ExpenseBuilder':
            self._split_values = split_values
            return self
        
        def build(self) -> 'Expense':
            if self._split_strategy is None:
                raise ValueError("Split strategy is required.")
            return Expense(self)
```

#### group.py

```python
import uuid
from typing import List, TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class Group:
    def __init__(self, name: str, members: List['User']):
        self._id = str(uuid.uuid4())
        self._name = name
        self._members = members
    
    def get_id(self) -> str:
        return self._id
    
    def get_name(self) -> str:
        return self._name
    
    def get_members(self) -> List['User']:
        return self._members.copy()
```

#### split.py

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class Split:
    def __init__(self, user: 'User', amount: float):
        self._user = user
        self._amount = amount
    
    def get_user(self) -> 'User':
        return self._user
    
    def get_amount(self) -> float:
        return self._amount
```

#### split_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List, Optional, TYPE_CHECKING
from split import Split
from user import User

class SplitStrategy(ABC):
    @abstractmethod
    def calculate_splits(self, total_amount: float, paid_by: 'User', participants: List['User'], split_values: Optional[List[float]]) -> List['Split']:
        pass

class EqualSplitStrategy(SplitStrategy):
    def calculate_splits(self, total_amount: float, paid_by: 'User', participants: List['User'], split_values: Optional[List[float]]) -> List['Split']:
        splits = []
        amount_per_person = total_amount / len(participants)
        for participant in participants:
            splits.append(Split(participant, amount_per_person))
        return splits

class ExactSplitStrategy(SplitStrategy):
    def calculate_splits(self, total_amount: float, paid_by: 'User', participants: List['User'], split_values: Optional[List[float]]) -> List['Split']:
        if len(participants) != len(split_values):
            raise ValueError("Number of participants and split values must match.")
        if abs(sum(split_values) - total_amount) > 0.01:
            raise ValueError("Sum of exact amounts must equal the total expense amount.")
        
        splits = []
        for i in range(len(participants)):
            splits.append(Split(participants[i], split_values[i]))
        return splits

class PercentageSplitStrategy(SplitStrategy):
    def calculate_splits(self, total_amount: float, paid_by: 'User', participants: List['User'], split_values: Optional[List[float]]) -> List['Split']:
        if len(participants) != len(split_values):
            raise ValueError("Number of participants and split values must match.")
        if abs(sum(split_values) - 100.0) > 0.01:
            raise ValueError("Sum of percentages must be 100.")
        
        splits = []
        for i in range(len(participants)):
            amount = (total_amount * split_values[i]) / 100.0
            splits.append(Split(participants[i], amount))
        return splits
```

#### splitwise_demo.py

```python
from splitwise_service import SplitwiseService
from expense import Expense
from split_strategy import EqualSplitStrategy, ExactSplitStrategy, PercentageSplitStrategy

class SplitwiseDemo:
    @staticmethod
    def main():
        # 1. Setup the service
        service = SplitwiseService.get_instance()
        
        # 2. Create users and groups
        alice = service.add_user("Alice", "alice@a.com")
        bob = service.add_user("Bob", "bob@b.com")
        charlie = service.add_user("Charlie", "charlie@c.com")
        david = service.add_user("David", "david@d.com")
        
        friends_group = service.add_group("Friends Trip", [alice, bob, charlie, david])
        
        print("--- System Setup Complete ---\n")
        
        # 3. Use Case 1: Equal Split
        print("--- Use Case 1: Equal Split ---")
        service.create_expense(Expense.ExpenseBuilder()
                              .set_description("Dinner")
                              .set_amount(1000)
                              .set_paid_by(alice)
                              .set_participants([alice, bob, charlie, david])
                              .set_split_strategy(EqualSplitStrategy()))
        
        service.show_balance_sheet(alice.get_id())
        service.show_balance_sheet(bob.get_id())
        print()
        
        # 4. Use Case 2: Exact Split
        print("--- Use Case 2: Exact Split ---")
        service.create_expense(Expense.ExpenseBuilder()
                              .set_description("Movie Tickets")
                              .set_amount(370)
                              .set_paid_by(alice)
                              .set_participants([bob, charlie])
                              .set_split_strategy(ExactSplitStrategy())
                              .set_split_values([120.0, 250.0]))
        
        service.show_balance_sheet(alice.get_id())
        service.show_balance_sheet(bob.get_id())
        print()
        
        # 5. Use Case 3: Percentage Split
        print("--- Use Case 3: Percentage Split ---")
        service.create_expense(Expense.ExpenseBuilder()
                              .set_description("Groceries")
                              .set_amount(500)
                              .set_paid_by(david)
                              .set_participants([alice, bob, charlie])
                              .set_split_strategy(PercentageSplitStrategy())
                              .set_split_values([40.0, 30.0, 30.0]))  # 40%, 30%, 30%
        
        print("--- Balances After All Expenses ---")
        service.show_balance_sheet(alice.get_id())
        service.show_balance_sheet(bob.get_id())
        service.show_balance_sheet(charlie.get_id())
        service.show_balance_sheet(david.get_id())
        print()
        
        # 6. Use Case 4: Simplify Group Debts
        print("--- Use Case 4: Simplify Group Debts for 'Friends Trip' ---")
        simplified_debts = service.simplify_group_debts(friends_group.get_id())
        if not simplified_debts:
            print("All debts are settled within the group!")
        else:
            for debt in simplified_debts:
                print(debt)
        print()
        
        service.show_balance_sheet(bob.get_id())
        
        # 7. Use Case 5: Partial Settlement
        print("--- Use Case 5: Partial Settlement ---")
        # From the simplified debts, we see Bob should pay Alice. Let's say Bob pays 100.
        service.settle_up(bob.get_id(), alice.get_id(), 100)
        
        print("--- Balances After Partial Settlement ---")
        service.show_balance_sheet(alice.get_id())
        service.show_balance_sheet(bob.get_id())

if __name__ == "__main__":
    SplitwiseDemo.main()
```

#### splitwise_service.py

```python
import threading
from typing import Dict, List, Optional
from user import User
from group import Group
from expense import Expense
from transaction import Transaction

class SplitwiseService:
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
            self._users: Dict[str, User] = {}
            self._groups: Dict[str, Group] = {}
            self._initialized = True
    
    @classmethod
    def get_instance(cls):
        return cls()
    
    def add_user(self, name: str, email: str) -> User:
        user = User(name, email)
        self._users[user.get_id()] = user
        return user
    
    def add_group(self, name: str, members: List[User]) -> Group:
        group = Group(name, members)
        self._groups[group.get_id()] = group
        return group
    
    def get_user(self, user_id: str) -> Optional[User]:
        return self._users.get(user_id)
    
    def get_group(self, group_id: str) -> Optional[Group]:
        return self._groups.get(group_id)
    
    def create_expense(self, builder: Expense.ExpenseBuilder):
        with self._lock:
            expense = builder.build()
            paid_by = expense.get_paid_by()
            
            for split in expense.get_splits():
                participant = split.get_user()
                amount = split.get_amount()
                
                if paid_by != participant:
                    paid_by.get_balance_sheet().adjust_balance(participant, amount)
                    participant.get_balance_sheet().adjust_balance(paid_by, -amount)
            
            print(f"Expense '{expense.get_description()}' of amount {expense.get_amount()} created.")
    
    def settle_up(self, payer_id: str, payee_id: str, amount: float):
        with self._lock:
            payer = self._users[payer_id]
            payee = self._users[payee_id]
            print(f"{payer.get_name()} is settling up {amount} with {payee.get_name()}")
            
            # Settlement is like a reverse expense. payer owes less to payee.
            payee.get_balance_sheet().adjust_balance(payer, -amount)
            payer.get_balance_sheet().adjust_balance(payee, amount)
    
    def show_balance_sheet(self, user_id: str):
        user = self._users[user_id]
        user.get_balance_sheet().show_balances()
    
    def simplify_group_debts(self, group_id: str) -> List[Transaction]:
        group = self._groups.get(group_id)
        if group is None:
            raise ValueError("Group not found")
        
        # Calculate net balance for each member within the group context
        net_balances = {}
        for member in group.get_members():
            balance = 0
            for other_user, amount in member.get_balance_sheet().get_balances().items():
                # Consider only balances with other group members
                if other_user in group.get_members():
                    balance += amount
            net_balances[member] = balance
        
        # Separate into creditors and debtors
        creditors = [(user, balance) for user, balance in net_balances.items() if balance > 0]
        debtors = [(user, balance) for user, balance in net_balances.items() if balance < 0]
        
        creditors.sort(key=lambda x: x[1], reverse=True)
        debtors.sort(key=lambda x: x[1])
        
        transactions = []
        i = j = 0
        
        while i < len(creditors) and j < len(debtors):
            creditor_user, creditor_amount = creditors[i]
            debtor_user, debtor_amount = debtors[j]
            
            amount_to_settle = min(creditor_amount, -debtor_amount)
            transactions.append(Transaction(debtor_user, creditor_user, amount_to_settle))
            
            creditors[i] = (creditor_user, creditor_amount - amount_to_settle)
            debtors[j] = (debtor_user, debtor_amount + amount_to_settle)
            
            if abs(creditors[i][1]) < 0.01:
                i += 1
            if abs(debtors[j][1]) < 0.01:
                j += 1
        
        return transactions
```

#### transaction.py

```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from user import User

class Transaction:
    def __init__(self, from_user: 'User', to_user: 'User', amount: float):
        self._from = from_user
        self._to = to_user
        self._amount = amount
    
    def __str__(self) -> str:
        return f"{self._from.get_name()} should pay {self._to.get_name()} ${self._amount:.2f}"
```

#### user.py

```python
import uuid
from balance_sheet import BalanceSheet
    
class User:
    def __init__(self, name: str, email: str):
        self._id = str(uuid.uuid4())
        self._name = name
        self._email = email
        self._balance_sheet = BalanceSheet(self)
    
    def get_id(self) -> str:
        return self._id
    
    def get_name(self) -> str:
        return self._name
    
    def get_balance_sheet(self) -> 'BalanceSheet':
        return self._balance_sheet
```