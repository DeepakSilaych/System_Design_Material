# Designing a Movie Ticket Booking System like BookMyShow

## Question
Design a Movie Ticket Booking System like BookMyShow

## Requirements
1. The system should allow users to view the list of movies playing in different theaters.
2. Users should be able to select a movie, theater, and show timing to book tickets.
3. The system should display the seating arrangement of the selected show and allow users to choose seats.
4. Users should be able to make payments and confirm their booking.
5. The system should handle concurrent bookings and ensure seat availability is updated in real-time.
6. The system should support different types of seats (e.g., normal, premium) and pricing.
7. The system should allow theater administrators to add, update, and remove movies, shows, and seating arrangements.
8. The system should be scalable to handle a large number of concurrent users and bookings.

## Solution
### Design overview
The booking flow is: discover shows → lock seats atomically → pay → confirm booking → release locks.
Seat allocation must be concurrency-safe and time-bound (locks expire).

- Per-show locking prevents cross-show contention and narrows the critical section.
- Pricing via a strategy lets us vary rules (weekday/weekend).
- Observer on `Movie` supports user subscriptions/notifications.

### Core model
- `Movie`, `City`, `Cinema`, `Screen`, `Show`
- `Seat` (`AVAILABLE` → `LOCKED` → `BOOKED`)
- `SeatLockManager` (time-bound locks)
- `BookingManager` (orchestrates lock → pay → confirm → unlock)
- `PaymentStrategy` variants; `Payment` outcome
- `MovieBookingService` façade and `MovieBookingDemo`

### Key flows
1. Search shows by city and movie
2. Lock seats (reject if any requested seat is unavailable)
3. Compute price via strategy and process payment
4. Confirm booking (mark seats booked) and release locks

### Design Details
1. The **Movie** class represents a movie with properties such as ID, title, description, and duration.
2. The **Theater** class represents a theater with properties such as ID, name, location, and a list of shows.
3. The **Show** class represents a movie show in a theater, with properties such as ID, movie, theater, start time, end time, and a map of seats.
4. The **Seat** class represents a seat in a show, with properties such as ID, row, column, type, price, and status.
5. The **SeatType** enum defines the different types of seats (normal or premium).
6. The **SeatStatus** enum defines the different statuses of a seat (available or booked).
7. The **Booking** class represents a booking made by a user, with properties such as ID, user, show, selected seats, total price, and status.
8. The **BookingStatus** enum defines the different statuses of a booking (pending, confirmed, or cancelled).
9. The **User** class represents a user of the booking system, with properties such as ID, name, and email.
10. The **MovieTicketBookingSystem** class is the main class that manages the movie ticket booking system. It follows the Singleton pattern to ensure only one instance of the system exists.
11. The MovieTicketBookingSystem class provides methods for adding movies, theaters, and shows, as well as booking tickets, confirming bookings, and cancelling bookings.
12. Multi-threading is achieved using concurrent data structures such as ConcurrentHashMap to handle concurrent access to shared resources like shows and bookings.
13. The **MovieTicketBookingDemo** class demonstrates the usage of the movie ticket booking system by adding movies, theaters, shows, booking tickets, and confirming or cancelling bookings.

### Implementation (Python)
#### booking.py

```python
from typing import List, Optional
from user import User
from show import Show
from seat import Seat
from payment import Payment
from enums import SeatStatus

class Booking:
    def __init__(self, booking_id: str, user: User, show: Show, seats: List[Seat], total_amount: float, payment: Payment):
        self.id = booking_id
        self.user = user
        self.show = show
        self.seats = seats
        self.total_amount = total_amount
        self.payment = payment

    def confirm_booking(self) -> None:
        """Marks seats as BOOKED upon successful booking creation"""
        for seat in self.seats:
            seat.set_status(SeatStatus.BOOKED)

    def get_id(self) -> str:
        return self.id

    def get_user(self) -> User:
        return self.user

    def get_show(self) -> Show:
        return self.show

    def get_seats(self) -> List[Seat]:
        return self.seats

    def get_total_amount(self) -> float:
        return self.total_amount

    def get_payment(self) -> Payment:
        return self.payment

    class BookingBuilder:
        def __init__(self):
            self.id: Optional[str] = None
            self.user: Optional[User] = None
            self.show: Optional[Show] = None
            self.seats: Optional[List[Seat]] = None
            self.total_amount: Optional[float] = None
            self.payment: Optional[Payment] = None

        def set_id(self, booking_id: str) -> 'Booking.BookingBuilder':
            self.id = booking_id
            return self

        def set_user(self, user: User) -> 'Booking.BookingBuilder':
            self.user = user
            return self

        def set_show(self, show: Show) -> 'Booking.BookingBuilder':
            self.show = show
            return self

        def set_seats(self, seats: List[Seat]) -> 'Booking.BookingBuilder':
            self.seats = seats
            return self

        def set_total_amount(self, total_amount: float) -> 'Booking.BookingBuilder':
            self.total_amount = total_amount
            return self

        def set_payment(self, payment: Payment) -> 'Booking.BookingBuilder':
            self.payment = payment
            return self

        def build(self) -> 'Booking':
            # Validations can be added here
            return Booking(self.id, self.user, self.show, self.seats, self.total_amount, self.payment)
```

#### booking_manager.py

```python
from typing import List, Optional
from user import User
from show import Show
from seat import Seat
from payment_strategy import PaymentStrategy
from seat_lock_manager import SeatLockManager
from enums import PaymentStatus
from booking import Booking

class BookingManager:
    def __init__(self, seat_lock_manager: SeatLockManager):
        self.seat_lock_manager = seat_lock_manager

    def create_booking(self, user: User, show: Show, seats: List[Seat], payment_strategy: PaymentStrategy) -> Optional[Booking]:
        # 1. Lock the seats
        self.seat_lock_manager.lock_seats(show, seats, user.get_id())

        # 2. Calculate the total price
        total_amount = show.get_pricing_strategy().calculate_price(seats)

        # 3. Process Payment
        payment = payment_strategy.pay(total_amount)

        # 4. If payment is successful, create the booking
        if payment.get_status() == PaymentStatus.SUCCESS:
            booking = Booking.BookingBuilder() \
                .set_user(user) \
                .set_show(show) \
                .set_seats(seats) \
                .set_total_amount(total_amount) \
                .set_payment(payment) \
                .build()

            # 5. Confirm the booking (mark seats as BOOKED)
            booking.confirm_booking()

            # Clean up the lock map
            self.seat_lock_manager.unlock_seats(show, seats, user.get_id())

            return booking
        else:
            print("Payment failed. Please try again.")
            return None
```

#### cinema.py

```python
from typing import List
from city import City
from screen import Screen

class Cinema:
    def __init__(self, cinema_id: str, name: str, city: City, screens: List[Screen]):
        self.id = cinema_id
        self.name = name
        self.city = city
        self.screens = screens

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name

    def get_city(self) -> City:
        return self.city

    def get_screens(self) -> List[Screen]:
        return self.screens
```

#### city.py

```python
class City:
    def __init__(self, city_id: str, name: str):
        self.id = city_id
        self.name = name

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name
```

#### enums.py

```python
from enum import Enum

class PaymentStatus(Enum):
    SUCCESS = "SUCCESS"
    FAILURE = "FAILURE"
    PENDING = "PENDING"

class SeatStatus(Enum):
    AVAILABLE = "AVAILABLE"
    LOCKED = "LOCKED"  # Temporarily held during booking process
    BOOKED = "BOOKED"

class SeatType(Enum):
    REGULAR = 50.0
    PREMIUM = 80.0
    RECLINER = 120.0

    def get_price(self) -> float:
        return self.value
```

#### movie.py

```python
from movie_observer import MovieSubject

class Movie(MovieSubject):
    def __init__(self, movie_id: str, title: str, duration_in_minutes: int):
        super().__init__()
        self.id = movie_id
        self.title = title
        self.duration_in_minutes = duration_in_minutes

    def get_id(self) -> str:
        return self.id

    def get_title(self) -> str:
        return self.title
```

#### movie_booking_demo.py

```python
from movie_booking_service import MovieBookingService
from movie import Movie
from screen import Screen
from seat import Seat, SeatType
from enums import SeatStatus
from payment_strategy import CreditCardPaymentStrategy
from movie_observer import UserObserver
from datetime import datetime, timedelta
from pricing_strategy import WeekdayPricingStrategy

class MovieBookingDemo:
  @staticmethod
  def main():
      # Setup
      service = MovieBookingService.get_instance()

      nyc = service.add_city("city1", "New York")
      la = service.add_city("city2", "Los Angeles")

      # 2. Add movies
      matrix = Movie("M1", "The Matrix", 120)
      avengers = Movie("M2", "Avengers: Endgame", 170)
      service.add_movie(matrix)
      service.add_movie(avengers)

      # Add Seats for a Screen
      screen1 = Screen("S1")

      for i in range(1, 11):
          seat_type = SeatType.REGULAR if i <= 5 else SeatType.PREMIUM
          screen1.add_seat(Seat(f"A{i}", 1, i, seat_type))
          screen1.add_seat(Seat(f"B{i}", 2, i, seat_type))

      # Add Cinemas
      amc_nyc = service.add_cinema("cinema1", "AMC Times Square", nyc.get_id(), [screen1])

      # Add Shows
      matrix_show = service.add_show("show1", matrix, screen1, datetime.now() + timedelta(hours=2), WeekdayPricingStrategy())
      avengers_show = service.add_show("show2", avengers, screen1, datetime.now() + timedelta(hours=5), WeekdayPricingStrategy())

      # User and Observer Setup
      alice = service.create_user("Alice", "alice@example.com")
      alice_observer = UserObserver(alice)
      avengers.add_observer(alice_observer)

      # Simulate movie release
      print("\n--- Notifying Observers about Movie Release ---")
      avengers.notify_observers()

      # User Story: Alice books tickets
      print("\n--- Alice's Booking Flow ---")
      city_name = "New York"
      movie_title = "Avengers: Endgame"

      # 1. Search for shows
      available_shows = service.find_shows(movie_title, city_name)
      if not available_shows:
          print(f"No shows found for {movie_title} in {city_name}")
          return
      
      selected_show = available_shows[0]  # Alice selects the first show

      # 2. View available seats
      available_seats = [seat for seat in selected_show.get_screen().get_seats() if seat.get_status() == SeatStatus.AVAILABLE]
      print(f"Available seats for '{selected_show.get_movie().get_title()}' at {selected_show.get_start_time()}: {[seat.get_id() for seat in available_seats]}")

      # 3. Select seats
      desired_seats = [available_seats[2], available_seats[3]]
      print(f"Alice selects seats: {[seat.get_id() for seat in desired_seats]}")

      # 4. Book Tickets
      booking = service.book_tickets(
          alice.get_id(),
          selected_show.get_id(),
          desired_seats,
          CreditCardPaymentStrategy("1234-5678-9876-5432", "123")
      )

      if booking:
          print("\n--- Booking Successful! ---")
          print(f"Booking ID: {booking.get_id()}")
          print(f"User: {booking.get_user().get_name()}")
          print(f"Movie: {booking.get_show().get_movie().get_title()}")
          print(f"Seats: {[seat.get_id() for seat in booking.get_seats()]}")
          print(f"Total Amount: ${booking.get_total_amount()}")
          print(f"Payment Status: {booking.get_payment().get_status().value}")
      else:
          print("Booking failed.")

      # 5. Verify seat status after booking
      print("\nSeat status after Alice's booking:")
      for seat in desired_seats:
          print(f"Seat {seat.get_id()} status: {seat.get_status().value}")

      # 6. Shut down the system to release resources like the scheduler.
      service.shutdown()


if __name__ == "__main__":
    MovieBookingDemo.main()
```

#### movie_booking_service.py

```python
from typing import Dict, List, Optional
from city import City
from cinema import Cinema
from movie import Movie
from screen import Screen
from show import Show
from user import User
from seat_lock_manager import SeatLockManager
from booking_manager import BookingManager
from payment_strategy import PaymentStrategy
from datetime import datetime
from booking import Booking
import threading
from pricing_strategy import PricingStrategy
from seat import Seat

class MovieBookingService:
    _instance: Optional['MovieBookingService'] = None
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

        self.cities: Dict[str, City] = {}
        self.cinemas: Dict[str, Cinema] = {}
        self.movies: Dict[str, Movie] = {}
        self.users: Dict[str, User] = {}
        self.shows: Dict[str, Show] = {}

        self.seat_lock_manager = SeatLockManager()
        self.booking_manager = BookingManager(self.seat_lock_manager)
        self.initialized = True

    @classmethod
    def get_instance(cls) -> 'MovieBookingService':
        return cls()

    def get_booking_manager(self) -> BookingManager:
        return self.booking_manager

    # Data Management Methods
    def add_city(self, city_id: str, name: str) -> City:
        city = City(city_id, name)
        self.cities[city.get_id()] = city
        return city

    def add_cinema(self, cinema_id: str, name: str, city_id: str, screens: List[Screen]) -> Cinema:
        city = self.cities[city_id]
        cinema = Cinema(cinema_id, name, city, screens)
        self.cinemas[cinema.get_id()] = cinema
        return cinema

    def add_movie(self, movie: Movie) -> None:
        self.movies[movie.get_id()] = movie

    def add_show(self, show_id: str, movie: Movie, screen: Screen, start_time: datetime, pricing_strategy: PricingStrategy) -> Show:
        show = Show(show_id, movie, screen, start_time, pricing_strategy)
        self.shows[show.get_id()] = show
        return show

    def create_user(self, name: str, email: str) -> User:
        user = User(name, email)
        self.users[user.get_id()] = user
        return user

    def book_tickets(self, user_id: str, show_id: str, desired_seats: List[Seat], payment_strategy: PaymentStrategy) -> Optional[Booking]:
        return self.booking_manager.create_booking(
            self.users[user_id],
            self.shows[show_id],
            desired_seats,
            payment_strategy
        )

    # Search Functionality
    def find_shows(self, movie_title: str, city_name: str) -> List[Show]:
        result = []
        for show in self.shows.values():
            if show.get_movie().get_title().lower() == movie_title.lower():
                cinema = self._find_cinema_for_show(show)
                if cinema and cinema.get_city().get_name().lower() == city_name.lower():
                    result.append(show)
        return result

    def _find_cinema_for_show(self, show: Show) -> Optional[Cinema]:
        # This is inefficient. In a real system, shows would have a direct link to the cinema.
        # For this example, we traverse the cinema list.
        for cinema in self.cinemas.values():
            if show.get_screen() in cinema.get_screens():
                return cinema
        return None

    def shutdown(self) -> None:
        self.seat_lock_manager.shutdown()
        print("MovieTicketBookingSystem has been shut down.")
```

#### movie_observer.py

```python
from abc import ABC, abstractmethod
from typing import List
from user import User
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from movie import Movie

class MovieObserver(ABC):
    @abstractmethod
    def update(self, movie: 'Movie') -> None:
        pass

class MovieSubject:
    def __init__(self):
        self.observers: List[MovieObserver] = []

    def add_observer(self, observer: MovieObserver) -> None:
        self.observers.append(observer)

    def remove_observer(self, observer: MovieObserver) -> None:
        if observer in self.observers:
            self.observers.remove(observer)

    def notify_observers(self) -> None:
        for observer in self.observers:
            observer.update(self)

class UserObserver(MovieObserver):
    def __init__(self, user: User):
        self.user = user

    def update(self, movie: 'Movie') -> None:
        print(f"Notification for {self.user.get_name()} ({self.user.get_id()}): Movie '{movie.get_title()}' is now available for booking!")
```

#### payment.py

```python
import uuid
from enums import PaymentStatus

class Payment:
    def __init__(self, amount: float, status: PaymentStatus, transaction_id: str):
        self.id = str(uuid.uuid4())
        self.amount = amount
        self.status = status
        self.transaction_id = transaction_id

    def get_status(self) -> PaymentStatus:
        return self.status
```

#### payment_strategy.py

```python
from abc import ABC, abstractmethod
from payment import Payment
from enums import PaymentStatus
import random
import uuid

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> Payment:
        pass

class CreditCardPaymentStrategy(PaymentStrategy):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number
        self.cvv = cvv

    def pay(self, amount: float) -> Payment:
        print(f"Processing credit card payment of ${amount:.2f}")
        # Simulate payment gateway interaction
        payment_success = random.random() > 0.05  # 95% success rate
        return Payment(
            amount,
            PaymentStatus.SUCCESS if payment_success else PaymentStatus.FAILURE,
            f"TXN_{uuid.uuid4()}"
        )
```

#### pricing_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from seat import Seat

class PricingStrategy(ABC):
    @abstractmethod
    def calculate_price(self, seats: List[Seat]) -> float:
        pass

class WeekdayPricingStrategy(PricingStrategy):
    def calculate_price(self, seats: List[Seat]) -> float:
        return sum(seat.get_type().get_price() for seat in seats)

class WeekendPricingStrategy(PricingStrategy):
    WEEKEND_SURCHARGE = 1.2  # 20% surcharge

    def calculate_price(self, seats: List[Seat]) -> float:
        base_price = sum(seat.get_type().get_price() for seat in seats)
        return base_price * self.WEEKEND_SURCHARGE
```

#### screen.py

```python
from typing import List
from seat import Seat

class Screen:
    def __init__(self, screen_id: str):
        self.id = screen_id
        self.seats: List[Seat] = []

    def add_seat(self, seat: Seat) -> None:
        self.seats.append(seat)

    def get_id(self) -> str:
        return self.id

    def get_seats(self) -> List[Seat]:
        return self.seats
```

#### seat.py

```python
from enums import SeatStatus, SeatType

class Seat:
    def __init__(self, seat_id: str, row: int, col: int, seat_type: SeatType):
        self.id = seat_id
        self.row = row
        self.col = col
        self.type = seat_type
        self.status = SeatStatus.AVAILABLE

    def get_id(self) -> str:
        return self.id

    def get_row(self) -> int:
        return self.row

    def get_col(self) -> int:
        return self.col

    def get_type(self) -> SeatType:
        return self.type

    def get_status(self) -> SeatStatus:
        return self.status

    def set_status(self, status: SeatStatus) -> None:
        self.status = status
```

#### seat_lock_manager.py

```python
from typing import Dict, List
from show import Show
from seat import Seat
from enums import SeatStatus
import threading
from concurrent.futures import ThreadPoolExecutor
import time

class SeatLockManager:
    def __init__(self):
        self.locked_seats: Dict[Show, Dict[Seat, str]] = {}
        self.executor = ThreadPoolExecutor(max_workers=5)
        self.LOCK_TIMEOUT_SECONDS = 0.5  # 0.5 seconds. In real world, timeout would be in minutes

    def lock_seats(self, show: Show, seats: List[Seat], user_id: str) -> None:
        # Use a lock per show to ensure atomicity for that specific show
        show_lock = getattr(show, '_lock', None)
        if show_lock is None:
            show._lock = threading.Lock()
            show_lock = show._lock

        with show_lock:
            # Check if any of the requested seats are already locked or booked
            for seat in seats:
                if seat.get_status() != SeatStatus.AVAILABLE:
                    print(f"Seat {seat.get_id()} is not available.")
                    return

            # Lock the seats
            for seat in seats:
                seat.set_status(SeatStatus.LOCKED)

            if show not in self.locked_seats:
                self.locked_seats[show] = {}
            
            for seat in seats:
                self.locked_seats[show][seat] = user_id

            # Schedule a task to unlock the seats after a timeout
            self.executor.submit(self._unlock_after_timeout, show, seats, user_id)
            print(f"Locked seats: {[seat.get_id() for seat in seats]} for user {user_id}")

    def _unlock_after_timeout(self, show: Show, seats: List[Seat], user_id: str) -> None:
        time.sleep(self.LOCK_TIMEOUT_SECONDS)
        self.unlock_seats(show, seats, user_id)

    def unlock_seats(self, show: Show, seats: List[Seat], user_id: str) -> None:
        show_lock = getattr(show, '_lock', None)
        if show_lock is None:
            return

        with show_lock:
            show_locks = self.locked_seats.get(show)
            if show_locks is not None:
                for seat in seats:
                    # Only unlock if it's still locked by the same user (prevents race conditions)
                    if seat in show_locks and show_locks[seat] == user_id:
                        del show_locks[seat]
                        if seat.get_status() == SeatStatus.LOCKED:
                            seat.set_status(SeatStatus.AVAILABLE)
                            print(f"Unlocked seat: {seat.get_id()} due to timeout.")
                        else:
                            print(f"Unlocked seat: {seat.get_id()} due to booking completion.")
                
                if not show_locks:
                    del self.locked_seats[show]

    def shutdown(self) -> None:
        print("Shutting down SeatLockProvider scheduler.")
        self.executor.shutdown(wait=True)
```

#### show.py

```python
from datetime import datetime
from movie import Movie
from screen import Screen
from pricing_strategy import PricingStrategy

class Show:
    def __init__(self, show_id: str, movie: Movie, screen: Screen, start_time: datetime, pricing_strategy: PricingStrategy):
        self.id = show_id
        self.movie = movie
        self.screen = screen
        self.start_time = start_time
        self.pricing_strategy = pricing_strategy

    def get_id(self) -> str:
        return self.id

    def get_movie(self) -> Movie:
        return self.movie

    def get_screen(self) -> Screen:
        return self.screen

    def get_start_time(self) -> datetime:
        return self.start_time

    def get_pricing_strategy(self) -> PricingStrategy:
        return self.pricing_strategy
```

#### user.py

```python
import uuid

class User:
    def __init__(self, name: str, email: str):
        self.id = str(uuid.uuid4())
        self.name = name
        self.email = email

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name
```