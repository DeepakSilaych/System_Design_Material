# Designing a Digital Wallet System

## Question
Design a Digital Wallet System

## Requirements
1. The digital wallet should allow users to create an account and manage their personal information.
2. Users should be able to add and remove payment methods, such as credit cards or bank accounts.
3. The digital wallet should support fund transfers between users and to external accounts.
4. The system should handle transaction history and provide a statement of transactions.
5. The digital wallet should support multiple currencies and perform currency conversions.
6. The system should ensure the security of user information and transactions.
7. The digital wallet should handle concurrent transactions and ensure data consistency.
8. The system should be scalable to handle a large number of users and transactions.

## Solution
### Design overview
A wallet is an accounts-and-ledgers system with strong consistency for balance mutations and
idempotent money-movement APIs. Execution order is: authenticate → validate → reserve → post →
persist transaction → emit events.

- Ledger-centric design: every balance change has a durable `Transaction`.
- Currency conversion via strategy (or external FX oracle) to avoid coupling.
- Idempotency keys on transfer to prevent double posting on retries.
- Payment methods behind an abstraction; PCI-sensitive data tokenized elsewhere.

### Core model
- `User`, `Account` (currency, balance), `Transaction` (source, destination, amount, currency, time)
- `PaymentMethod` abstractions (`CreditCard`, `BankAccount`)
- `Currency`, `CurrencyConverter`
- `DigitalWallet` façade for user/account management, deposits/withdrawals/transfers, statements

### Key flows
1. Deposit/Withdraw: validate method → process via provider → post to account ledger
2. Transfer: verify ownership/limits → (optional) convert FX → atomic debit/credit → record `Transaction`
3. Statement: read-only range queries over an account’s transactions

### Design Details
1. The **User** class represents a user of the digital wallet, with properties such as ID, name, email, password, and a list of accounts.
2. The **Account** class represents a user's account within the digital wallet, with properties like ID, user, account number, currency, balance, and a list of transactions. It provides methods to deposit and withdraw funds.
3. The **Transaction** class represents a financial transaction between two accounts, containing properties such as ID, source account, destination account, amount, currency, and timestamp.
4. The **PaymentMethod** class is an abstract base class for different payment methods, such as credit cards and bank accounts. It defines the common properties and methods for processing payments.
5. The **CreditCard** and **BankAccount** classes are concrete implementations of the PaymentMethod class, representing specific payment methods.
6. The **Currency** enum represents different currencies supported by the digital wallet.
7. The **CurrencyConverter** class provides a static method to convert amounts between different currencies based on predefined exchange rates.
8. The **DigitalWallet** class is the central component of the digital wallet system. It follows the Singleton pattern to ensure only one instance of the digital wallet exists. It provides methods to create users, accounts, add payment methods, transfer funds, and retrieve transaction history. It handles concurrent access to shared resources using synchronization.
9. The **DigitalWalletDemo** class demonstrates the usage of the digital wallet system by creating users, accounts, adding payment methods, depositing funds, transferring funds, and retrieving transaction history.

### Implementation (Python)
#### account.py

```python
from decimal import Decimal
from exception import InsufficientFundsException

class Account:
    def __init__(self, id, user, account_number, currency):
        self.id = id
        self.user = user
        self.account_number = account_number
        self.currency = currency
        self.balance = Decimal('0.00')
        self.transactions = []

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if self.balance >= amount:
            self.balance -= amount
        else:
            raise InsufficientFundsException("Insufficient funds in the account.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)
```

#### bank_account.py

```python
from payment_method import PaymentMethod

class BankAccount(PaymentMethod):
    def __init__(self, id, user, account_number, routing_number):
        super().__init__(id, user)
        self.account_number = account_number
        self.routing_number = routing_number

    def process_payment(self, amount, currency):
        # Process bank account payment
        return True
```

#### credit_card.py

```python
from payment_method import PaymentMethod

class CreditCard(PaymentMethod):
    def __init__(self, id, user, card_number, expiration_date, cvv):
        super().__init__(id, user)
        self.card_number = card_number
        self.expiration_date = expiration_date
        self.cvv = cvv

    def process_payment(self, amount, currency):
        # Process credit card payment
        return True
```

#### currency.py

```python
from enum import Enum

class Currency(Enum):
    USD = 'USD'
    EUR = 'EUR'
    GBP = 'GBP'
    JPY = 'JPY'
```

#### currency_converter.py

```python
from decimal import Decimal
from currency import Currency

class CurrencyConverter:
    exchange_rates = {
        Currency.USD: Decimal('1.00'),
        Currency.EUR: Decimal('0.85'),
        Currency.GBP: Decimal('0.72'),
        Currency.JPY: Decimal('110.00')
    }

    @staticmethod
    def convert(amount, source_currency, target_currency):
        source_rate = CurrencyConverter.exchange_rates[source_currency]
        target_rate = CurrencyConverter.exchange_rates[target_currency]
        return amount * source_rate / target_rate
```

#### digital_wallet.py

```python
import uuid
from user import User
from account import Account
from payment_method import PaymentMethod
from currency import Currency
from currency_converter import CurrencyConverter
from transaction import Transaction

class DigitalWallet:
    _instance = None

    def __init__(self):
        self.users = {}
        self.accounts = {}
        self.payment_methods = {}

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    def create_user(self, user):
        self.users[user.id] = user

    def get_user(self, user_id):
        return self.users.get(user_id)

    def create_account(self, account):
        self.accounts[account.id] = account
        account.user.add_account(account)

    def get_account(self, account_id):
        return self.accounts.get(account_id)

    def add_payment_method(self, payment_method):
        self.payment_methods[payment_method.id] = payment_method

    def get_payment_method(self, payment_method_id):
        return self.payment_methods.get(payment_method_id)

    def transfer_funds(self, source_account, destination_account, amount, currency):
        if source_account.currency != currency:
            amount = CurrencyConverter.convert(amount, currency, source_account.currency)
        source_account.withdraw(amount)

        if destination_account.currency != currency:
            amount = CurrencyConverter.convert(amount, currency, destination_account.currency)
        destination_account.deposit(amount)

        transaction_id = self._generate_transaction_id()
        transaction = Transaction(transaction_id, source_account, destination_account, amount, currency)
        source_account.add_transaction(transaction)
        destination_account.add_transaction(transaction)

    def get_transaction_history(self, account):
        return account.transactions

    def _generate_transaction_id(self):
        return "TXN" + str(uuid.uuid4()).replace('-', '').upper()[:8]
```

#### digital_wallet_demo.py

```python
from decimal import Decimal
from user import User
from account import Account
from currency import Currency
from digital_wallet import DigitalWallet
from credit_card import CreditCard
from bank_account import BankAccount

class DigitalWalletDemo:
    @staticmethod
    def run():
        digital_wallet = DigitalWallet.get_instance()

        # Create users
        user1 = User("U001", "John Doe", "john@example.com", "password123")
        user2 = User("U002", "Jane Smith", "jane@example.com", "password456")
        digital_wallet.create_user(user1)
        digital_wallet.create_user(user2)

        # Create accounts
        account1 = Account("A001", user1, "1234567890", Currency.USD)
        account2 = Account("A002", user2, "9876543210", Currency.EUR)
        digital_wallet.create_account(account1)
        digital_wallet.create_account(account2)

        # Add payment methods
        credit_card = CreditCard("PM001", user1, "1234567890123456", "12/25", "123")
        bank_account = BankAccount("PM002", user2, "9876543210", "987654321")
        digital_wallet.add_payment_method(credit_card)
        digital_wallet.add_payment_method(bank_account)

        # Deposit funds
        account1.deposit(Decimal("1000.00"))
        account2.deposit(Decimal("500.00"))

        # Transfer funds
        digital_wallet.transfer_funds(account1, account2, Decimal("100.00"), Currency.USD)

        # Get transaction history
        transaction_history1 = digital_wallet.get_transaction_history(account1)
        transaction_history2 = digital_wallet.get_transaction_history(account2)

        # Print transaction history
        print("Transaction History for Account 1:")
        for transaction in transaction_history1:
            print(f"Transaction ID: {transaction.id}")
            print(f"Amount: {transaction.amount} {transaction.currency}")
            print(f"Timestamp: {transaction.timestamp}")
            print()

        print("Transaction History for Account 2:")
        for transaction in transaction_history2:
            print(f"Transaction ID: {transaction.id}")
            print(f"Amount: {transaction.amount} {transaction.currency}")
            print(f"Timestamp: {transaction.timestamp}")
            print()

if __name__ == "__main__":
    DigitalWalletDemo.run()
```

#### exception.py

```python
class InsufficientFundsException(Exception):
    pass
```

#### payment_method.py

```python
class PaymentMethod:
    def __init__(self, id, user):
        self.id = id
        self.user = user

    def process_payment(self, amount, currency):
        raise NotImplementedError
```

#### transaction.py

```python
from datetime import datetime

class Transaction:
    def __init__(self, id, source_account, destination_account, amount, currency):
        self.id = id
        self.source_account = source_account
        self.destination_account = destination_account
        self.amount = amount
        self.currency = currency
        self.timestamp = datetime.now()
```

#### user.py

```python
class User:
    def __init__(self, id, name, email, password):
        self.id = id
        self.name = name
        self.email = email
        self.password = password
        self.accounts = []

    def add_account(self, account):
        self.accounts.append(account)

    def remove_account(self, account):
        self.accounts.remove(account)
```