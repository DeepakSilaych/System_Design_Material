# Designing a Car Rental System

## Question
Design a Car Rental System

## Requirements
1. The car rental system should allow customers to browse and reserve available cars for specific dates.
2. Each car should have details such as make, model, year, license plate number, and rental price per day.
3. Customers should be able to search for cars based on various criteria, such as car type, price range, and availability.
4. The system should handle reservations, including creating, modifying, and canceling reservations.
5. The system should keep track of the availability of cars and update their status accordingly.
6. The system should handle customer information, including name, contact details, and driver's license information.
7. The system should handle payment processing for reservations.
8. The system should be able to handle concurrent reservations and ensure data consistency.

## Solution
### Design overview
The refactored design keeps the model small but expressive and orders the components the same way the
business flow executes:

1. **Catalogue** – describe the fleet and expose search capabilities.
2. **Reservation lifecycle** – block a vehicle for a date range, release it, and capture payment.
3. **Facade** – orchestrate the user journey (search → reserve → pay) and expose a cohesive API to
   UI or automation clients.

The code favours simple, immutable data structures where possible, delegates policy-specific logic to
strategy interfaces (for payment), and validates dates and availability before mutating any shared
state.  Responsibilities are kept small to preserve the SRP, and collaborators are injected rather
than hidden behind singletons so that the design is testable.

### Core model
| Component | Responsibility |
| --- | --- |
| `Car` | Value object capturing inventory details (make, model, year, category, pricing, mileage). |
| `Customer` | Lightweight record of rental customer identity and driver licence. |
| `ReservationRequest` | Input DTO carrying search and booking intent (vehicle id, dates). |
| `Reservation` | Aggregate that links customer, vehicle and booking window, and derives total price. |
| `Inventory` | Repository of vehicles. Performs filtering by availability, class, and budget. |
| `ReservationService` | Transaction script that validates dates, interacts with inventory, creates bookings, processes payments and emits receipts. |
| `PaymentStrategy` | Strategy interface; concrete implementations (credit card, wallet) can be swapped without touching booking logic. |
| `CarRentalSystem` | Thin façade used by UI/client code. Coordinates the happy-path flow and exposes query + command methods. |

### Key flows
1. **Search**: `CarRentalSystem.find_cars` delegates to `Inventory` with a filter tuple (`CarType`, price range, required dates). The inventory first prunes by metadata and then by overlapping reservations.
2. **Reserve**: `ReservationService.reserve` validates date ordering, double checks availability (defensive) and creates a `Reservation`. The reservation is persisted in-memory (dictionary keyed by booking id) for simplicity.
3. **Payment**: Before marking a reservation confirmed the service charges the configured `PaymentStrategy`. Failures raise an exception and roll back the tentative block.
4. **Cancel**: Removes the booking, releases the car in the inventory and records a cancellation timestamp.

### Implementation (Python)
The modules below are ordered in the same sequence they compose the flow: domain objects first,
followed by services, then the façade and finally a usage example.

#### models.py
```python
from dataclasses import dataclass, field
from datetime import date
from enum import Enum
from typing import Tuple


class CarType(Enum):
    SEDAN = "sedan"
    SUV = "suv"
    HATCHBACK = "hatchback"
    CONVERTIBLE = "convertible"


@dataclass(frozen=True)
class Car:
    car_id: str
    make: str
    model: str
    year: int
    category: CarType
    daily_rate: float
    mileage_limit: int | None = None


@dataclass(frozen=True)
class Customer:
    customer_id: str
    name: str
    email: str
    licence_number: str


@dataclass(frozen=True)
class ReservationRequest:
    car_id: str
    customer_id: str
    start_date: date
    end_date: date

    def validate(self) -> None:
        if self.start_date >= self.end_date:
            raise ValueError("End date must be after start date")


@dataclass
class Reservation:
    reservation_id: str
    car: Car
    customer: Customer
    start_date: date
    end_date: date
    total_cost: float = field(init=False)

    def __post_init__(self) -> None:
        rental_days = (self.end_date - self.start_date).days
        if rental_days <= 0:
            raise ValueError("Reservation must span at least one day")
        self.total_cost = rental_days * self.car.daily_rate

    def overlaps(self, other_window: Tuple[date, date]) -> bool:
        other_start, other_end = other_window
        return not (self.end_date <= other_start or self.start_date >= other_end)
```

#### inventory.py
```python
from dataclasses import dataclass, field
from datetime import date
from typing import Dict, Iterable, List, Tuple

from models import Car, CarType


@dataclass
class Inventory:
    _cars: Dict[str, Car] = field(default_factory=dict)

    def add_car(self, car: Car) -> None:
        self._cars[car.car_id] = car

    def remove_car(self, car_id: str) -> None:
        self._cars.pop(car_id, None)

    def list_cars(self) -> Iterable[Car]:
        return self._cars.values()

    def find_available(
        self,
        existing_bookings: Dict[str, List[Tuple[date, date]]],
        *,
        car_type: CarType | None = None,
        price_between: Tuple[float, float] | None = None,
        rental_window: Tuple[date, date] | None = None,
    ) -> List[Car]:
        matches: List[Car] = []
        for car in self._cars.values():
            if car_type and car.category != car_type:
                continue
            if price_between:
                low, high = price_between
                if not (low <= car.daily_rate <= high):
                    continue
            if rental_window and any(
                _overlaps(rental_window, window)
                for window in existing_bookings.get(car.car_id, [])
            ):
                continue
            matches.append(car)
        return matches


def _overlaps(window_a: Tuple[date, date], window_b: Tuple[date, date]) -> bool:
    start_a, end_a = window_a
    start_b, end_b = window_b
    return not (end_a <= start_b or start_a >= end_b)
```

#### payments.py
```python
from abc import ABC, abstractmethod


class PaymentStrategy(ABC):
    @abstractmethod
    def charge(self, *, customer_id: str, amount: float) -> None:
        """Raise an exception if the payment fails."""


class CreditCardPayment(PaymentStrategy):
    def charge(self, *, customer_id: str, amount: float) -> None:
        # Here we would integrate with a gateway. For the demo we assume success.
        if amount <= 0:
            raise ValueError("Amount must be positive")


class WalletPayment(PaymentStrategy):
    def charge(self, *, customer_id: str, amount: float) -> None:
        # Stub for illustration.
        if amount > 5000:
            raise RuntimeError("Wallet limit exceeded")
```

#### reservation_service.py
```python
import uuid
from dataclasses import dataclass, field
from datetime import date
from typing import Dict, List, Tuple

from inventory import Inventory
from models import Customer, Reservation, ReservationRequest
from payments import PaymentStrategy


@dataclass
class ReservationService:
    inventory: Inventory
    payment: PaymentStrategy
    customers: Dict[str, Customer] = field(default_factory=dict)
    _reservations: Dict[str, Reservation] = field(default_factory=dict)
    _car_bookings: Dict[str, List[Tuple[date, date]]] = field(default_factory=dict)

    def register_customer(self, customer: Customer) -> None:
        self.customers[customer.customer_id] = customer

    def reserve(self, request: ReservationRequest) -> Reservation:
        request.validate()
        customer = self.customers.get(request.customer_id)
        if not customer:
            raise LookupError(f"Unknown customer {request.customer_id}")

        booking_window = (request.start_date, request.end_date)
        available = self.inventory.find_available(
            self._car_bookings, car_type=None, price_between=None, rental_window=booking_window
        )
        if request.car_id not in {car.car_id for car in available}:
            raise ValueError("Car is not available for the selected window")

        car = next(car for car in self.inventory.list_cars() if car.car_id == request.car_id)
        reservation = Reservation(
            reservation_id=f"RES-{uuid.uuid4().hex[:8].upper()}",
            car=car,
            customer=customer,
            start_date=request.start_date,
            end_date=request.end_date,
        )

        self.payment.charge(customer_id=customer.customer_id, amount=reservation.total_cost)

        self._reservations[reservation.reservation_id] = reservation
        self._car_bookings.setdefault(car.car_id, []).append(booking_window)
        return reservation

    def cancel(self, reservation_id: str) -> None:
        reservation = self._reservations.pop(reservation_id, None)
        if not reservation:
            return
        window = (reservation.start_date, reservation.end_date)
        self._car_bookings[reservation.car.car_id] = [
            existing for existing in self._car_bookings.get(reservation.car.car_id, []) if existing != window
        ]

    def reservations_for(self, customer_id: str) -> List[Reservation]:
        return [res for res in self._reservations.values() if res.customer.customer_id == customer_id]

    def booked_windows(self, car_id: str) -> List[Tuple[date, date]]:
        return list(self._car_bookings.get(car_id, []))
```

#### facade.py
```python
from datetime import date
from typing import List, Tuple

from inventory import Inventory
from models import Car, CarType, Customer, Reservation, ReservationRequest
from payments import CreditCardPayment
from reservation_service import ReservationService


class CarRentalSystem:
    def __init__(self) -> None:
        self.inventory = Inventory()
        self.reservations = ReservationService(
            inventory=self.inventory,
            payment=CreditCardPayment(),
        )

    def add_car(self, car: Car) -> None:
        self.inventory.add_car(car)

    def register_customer(self, customer: Customer) -> None:
        self.reservations.register_customer(customer)

    def find_cars(
        self,
        *,
        car_type: CarType | None = None,
        price_between: Tuple[float, float] | None = None,
        rental_window: Tuple[date, date] | None = None,
    ) -> List[Car]:
        return self.inventory.find_available(
            self.reservations._car_bookings,
            car_type=car_type,
            price_between=price_between,
            rental_window=rental_window,
        )

    def make_reservation(self, request: ReservationRequest) -> Reservation:
        return self.reservations.reserve(request)

    def cancel_reservation(self, reservation_id: str) -> None:
        self.reservations.cancel(reservation_id)
```

#### demo.py
```python
from datetime import date, timedelta

from facade import CarRentalSystem
from models import Car, CarType, Customer, ReservationRequest


def main() -> None:
    system = CarRentalSystem()
    system.add_car(Car("CAR-001", "Toyota", "Camry", 2022, CarType.SEDAN, daily_rate=55))
    system.add_car(Car("CAR-002", "Honda", "CR-V", 2023, CarType.SUV, daily_rate=75))

    system.register_customer(Customer("CUS-001", "John Doe", "john@example.com", "DL123"))

    rental_window = (date.today(), date.today() + timedelta(days=4))
    available = system.find_cars(car_type=CarType.SUV, rental_window=rental_window)
    if not available:
        print("No SUVs available for the chosen dates")
        return

    request = ReservationRequest(
        car_id=available[0].car_id,
        customer_id="CUS-001",
        start_date=rental_window[0],
        end_date=rental_window[1],
    )
    reservation = system.make_reservation(request)
    print(f"Reservation confirmed: {reservation.reservation_id} for {reservation.total_cost} USD")


if __name__ == "__main__":
    main()
```

### Extensibility notes
- The booking service persists data in-memory to keep the example compact; swapping to a
  database-backed repository only requires replacing the `Inventory` and `ReservationService` storage
  dictionaries.
- Pricing rules and add-ons (insurance, mileage penalties) can be encapsulated in a `PricingStrategy`
  that the service calls before charging the payment method.
- The payment layer is strategy-based, so support for new gateways or offline payments is isolated to
  new concrete classes.