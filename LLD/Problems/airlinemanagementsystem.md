# Designing an Airline Management System

## Question
Design an Airline Management System

## Requirements
1. The airline management system should allow users to search for flights based on source, destination, and date.
2. Users should be able to book flights, select seats, and make payments.
3. The system should manage flight schedules, aircraft assignments, and crew assignments.
4. The system should handle passenger information, including personal details and baggage information.
5. The system should support different types of users, such as passengers, airline staff, and administrators.
6. The system should be able to handle cancellations, refunds, and flight changes.
7. The system should ensure data consistency and handle concurrent access to shared resources.
8. The system should be scalable and extensible to accommodate future enhancements and new features.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: Flight, Aircraft, Passenger, Booking, Seat, Payment, FlightSearch,
BookingManager, PaymentProcessor, AirlineManagementSystem.

### Design Details
1. The **Flight** class represents a flight in the airline management system, with properties such as flight number, source, destination, departure time, arrival time, and available seats.
2. The **Aircraft** class represents an aircraft, with properties like tail number, model, and total seats.
3. The **Passenger** class represents a passenger, with properties such as ID, name, email, and phone number.
4. The **Booking** class represents a booking made by a passenger for a specific flight and seat, with properties such as booking number, flight, passenger, seat, price, and booking status.
5. The **Seat** class represents a seat on a flight, with properties like seat number, seat type, and seat status.
6. The **Payment** class represents a payment made for a booking, with properties such as payment ID, payment method, amount, and payment status.
7. The **FlightSearch** class provides functionality to search for flights based on source, destination, and date.
8. The **BookingManager** class manages the creation and cancellation of bookings. It follows the Singleton pattern to ensure a single instance of the booking manager.
9. The **PaymentProcessor** class handles the processing of payments. It follows the Singleton pattern to ensure a single instance of the payment processor.
10. The **AirlineManagementSystem** class serves as the main entry point of the system, combining all the components and providing methods for flight management, booking, payment processing, and other operations.

### Implementation (Python)
#### aircraft.py

```python
class Aircraft:
    def __init__(self, tail_number, model, total_seats):
        self.tail_number = tail_number
        self.model = model
        self.total_seats = total_seats
```

#### airline_management_system.py

```python
from flight import Flight
from aircraft import Aircraft
from flight_search import FlightSearch
from booking_manager import BookingManager
from payment_processor import PaymentProcessor

class AirlineManagementSystem:
    def __init__(self):
        self.flights = []
        self.aircrafts = []
        self.flight_search = FlightSearch(self.flights)
        self.booking_manager = BookingManager()
        self.payment_processor = PaymentProcessor()

    def add_flight(self, flight):
        self.flights.append(flight)

    def add_aircraft(self, aircraft):
        self.aircrafts.append(aircraft)

    def search_flights(self, source, destination, date):
        return self.flight_search.search_flights(source, destination, date)

    def book_flight(self, flight, passenger, seat, price):
        return self.booking_manager.create_booking(flight, passenger, seat, price)

    def cancel_booking(self, booking_number):
        self.booking_manager.cancel_booking(booking_number)

    def process_payment(self, payment):
        self.payment_processor.process_payment(payment)
```

#### airline_management_system_demo.py

```python
from datetime import datetime, timedelta
from typing import List
from airline_management_system import AirlineManagementSystem
from passenger import Passenger
from flight import Flight
from aircraft import Aircraft
from seat import Seat, SeatType

class AirlineManagementSystemDemo:
    @staticmethod
    def run():
        airline_management_system = AirlineManagementSystem()

        # Create users
        passenger1 = Passenger("U001", "John Doe", "john@example.com", "1234567890")

        # Create flights
        departure_time1 = datetime.now() + timedelta(days=1)
        arrival_time1 = departure_time1 + timedelta(hours=2)
        flight1 = Flight("F001", "New York", "London", departure_time1, arrival_time1)

        departure_time2 = datetime.now() + timedelta(days=3)
        arrival_time2 = departure_time2 + timedelta(hours=5)
        flight2 = Flight("F002", "Paris", "Tokyo", departure_time2, arrival_time2)

        airline_management_system.add_flight(flight1)
        airline_management_system.add_flight(flight2)

        # Create aircrafts
        aircraft1 = Aircraft("A001", "Boeing 747", 300)
        aircraft2 = Aircraft("A002", "Airbus A380", 500)
        airline_management_system.add_aircraft(aircraft1)
        airline_management_system.add_aircraft(aircraft2)

        # Search flights
        search_date = datetime.now().date() + timedelta(days=1)
        search_results: List[Flight] = airline_management_system.search_flights("New York", "London", search_date)
        print("Search Results:")
        for flight in search_results:
            print(f"Flight: {flight.flight_number} - {flight.source} to {flight.destination}")

        seat = Seat("25A", SeatType.ECONOMY)

        # Book a flight
        booking = airline_management_system.book_flight(flight1, passenger1, seat, 100)
        if booking:
            print(f"Booking successful. Booking ID: {booking.booking_number}")
        else:
            print("Booking failed.")

        # Cancel a booking
        airline_management_system.cancel_booking(booking.booking_number)
        print("Booking cancelled.")

if __name__ == "__main__":
    AirlineManagementSystemDemo.run()
```

#### booking.py

```python
from enum import Enum

class BookingStatus(Enum):
    CONFIRMED = 1
    CANCELLED = 2
    PENDING = 3
    EXPIRED = 4

class Booking:
    def __init__(self, booking_number, flight, passenger, seat, price):
        self.booking_number = booking_number
        self.flight = flight
        self.passenger = passenger
        self.seat = seat
        self.price = price
        self.status = BookingStatus.CONFIRMED

    def cancel(self):
        self.status = BookingStatus.CANCELLED
```

#### booking_manager.py

```python
import datetime
from booking import Booking
from threading import Lock

class BookingManager:
    _instance = None
    _lock = Lock()

    def __new__(cls):
        if not cls._instance:
            with cls._lock:
                if not cls._instance:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self.bookings = {}
        self.booking_counter = 0

    def create_booking(self, flight, passenger, seat, price):
        booking_number = self._generate_booking_number()
        booking = Booking(booking_number, flight, passenger, seat, price)
        with self._lock:
            self.bookings[booking_number] = booking
        return booking

    def cancel_booking(self, booking_number):
        with self._lock:
            booking = self.bookings.get(booking_number)
            if booking:
                booking.cancel()

    def _generate_booking_number(self):
        self.booking_counter += 1
        timestamp = datetime.datetime.now().strftime("%Y%m%d%H%M%S")
        return f"BKG{timestamp}{self.booking_counter:06d}"
```

#### flight.py

```python
from datetime import datetime

class Flight:
    def __init__(self, flight_number, source, destination, departure_time, arrival_time):
        self.flight_number = flight_number
        self.source = source
        self.destination = destination
        self.departure_time = departure_time
        self.arrival_time = arrival_time
        self.available_seats = []

    def get_source(self):
        return self.source

    def get_destination(self):
        return self.destination

    def get_departure_time(self):
        return self.departure_time
```

#### flight_search.py

```python
from datetime import date

class FlightSearch:
    def __init__(self, flights):
        self.flights = flights

    def search_flights(self, source, destination, date):
        return [flight for flight in self.flights
                if flight.get_source().lower() == source.lower()
                and flight.get_destination().lower() == destination.lower()
                and flight.get_departure_time().date() == date]
```

#### passenger.py

```python
class Passenger:
    def __init__(self, passenger_id, name, email, phone):
        self.id = passenger_id
        self.name = name
        self.email = email
        self.phone = phone
```

#### payment.py

```python
from enum import Enum

class PaymentStatus(Enum):
    PENDING = 1
    COMPLETED = 2
    FAILED = 3
    REFUNDED = 4

class Payment:
    def __init__(self, payment_id, payment_method, amount):
        self.payment_id = payment_id
        self.payment_method = payment_method
        self.amount = amount
        self.status = PaymentStatus.PENDING

    def process_payment(self):
        # Process payment logic
        self.status = PaymentStatus.COMPLETED
```

#### payment_processor.py

```python
from threading import Lock

class PaymentProcessor:
    _instance = None
    _lock = Lock()

    def __new__(cls):
        if not cls._instance:
            with cls._lock:
                if not cls._instance:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def process_payment(self, payment):
        # Process payment using the selected payment method
        payment.process_payment()
```

#### seat.py

```python
from enum import Enum

class SeatStatus(Enum):
    AVAILABLE = 1
    RESERVED = 2
    OCCUPIED = 3

class SeatType(Enum):
    ECONOMY = 1
    PREMIUM_ECONOMY = 2
    BUSINESS = 3
    FIRST_CLASS = 4

class Seat:
    def __init__(self, seat_number, seat_type):
        self.seat_number = seat_number
        self.type = seat_type
        self.status = SeatStatus.AVAILABLE

    def reserve(self):
        self.status = SeatStatus.RESERVED

    def release(self):
        self.status = SeatStatus.AVAILABLE
```