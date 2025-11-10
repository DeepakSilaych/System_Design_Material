# Designing a Concert Ticket Booking System

## Question
Design a Concert Ticket Booking System

## Requirements
1. The concert ticket booking system should allow users to view available concerts and their seating arrangements.
2. Users should be able to search for concerts based on various criteria such as artist, venue, date, and time.
3. Users should be able to select seats and purchase tickets for a specific concert.
4. The system should handle concurrent booking requests to avoid double-booking of seats.
5. The system should ensure fair booking opportunities for all users.
6. The system should handle payment processing securely.
7. The system should generate booking confirmations and send them to users via email or SMS.
8. The system should provide a waiting list functionality for sold-out concerts.

## Solution
### Design overview
Order the solution around the real booking flow: discover inventory → hold seats (atomic) → pay →
confirm → ticket/receipt. Seat allocation must be race-safe and idempotent.

- Separate read-side (search) from write-side (booking) to keep locks short.
- Use a lightweight “seat hold” with TTL to avoid cart sniping and reduce double-booking risk.
- Keep payment behind a strategy to swap processors.
- Persist confirmations and email/SMS asynchronously after commit.

### Core model
- `Concert` (id, artist, venue, datetime, seats)
- `Seat` (id, label, type, price), with status transitions: AVAILABLE → HELD → BOOKED
- `SeatHold` (id, user, seats, expires_at)
- `Booking` (id, user, concert, seats, total, status)
- `BookingService` (atomic holds, confirm, cancel; concurrency control)
- `PaymentStrategy` (charge/refund)
- `SearchService` (filter by artist/venue/date, paginate)

### Key flows
1. Search: read-only filters on concerts and free seats
2. Hold: atomically mark seats HELD for a user with expiry (e.g., 5 minutes)
3. Pay & confirm: charge payment; on success → mark seats BOOKED and persist `Booking`
4. Cancel/expire: release HELD seats back to AVAILABLE; failed payments also release seats

### Design Details
1. The **Concert** class represents a concert event, with properties such as ID, artist, venue, date and time, and a list of seats.
2. The **Seat** class represents a seat in a concert, with properties like ID, seat number, seat type, price, and status. It provides methods to book and release a seat.
3. The **SeatType** enum represents the different types of seats available, such as regular, premium, and VIP.
4. The **SeatStatus** enum represents the status of a seat, which can be available, booked, or reserved.
5. The **Booking** class represents a booking made by a user for a specific concert and seats. It contains properties such as ID, user, concert, seats, total price, and status. It provides methods to confirm and cancel a booking.
6. The **BookingStatus** enum represents the status of a booking, which can be pending, confirmed, or cancelled.
7. The **User** class represents a user of the concert ticket booking system, with properties like ID, name, and email.
8. The **ConcertTicketBookingSystem** class is the central component of the system. It follows the Singleton pattern to ensure a single instance of the system. It manages concerts, bookings, and provides methods to add concerts, search concerts, book tickets, and cancel bookings.
9. The **SeatNotAvailableException** is a custom exception used to handle cases where a seat is not available for booking.

### Implementation (Python)
Below is a minimal in-memory outline that mirrors the flow. Replace dictionaries with repositories in
production and gate mutations with DB-level constraints/row locks.

#### models.py
```python
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from enum import Enum
from typing import List


class SeatStatus(Enum):
    AVAILABLE = "AVAILABLE"
    HELD = "HELD"
    BOOKED = "BOOKED"


@dataclass
class Seat:
    seat_id: str
    label: str
    price: float
    status: SeatStatus = SeatStatus.AVAILABLE


@dataclass(frozen=True)
class Concert:
    concert_id: str
    artist: str
    venue: str
    when: datetime
```

#### services.py
```python
import threading
import uuid
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import Dict, List, Tuple

from models import Concert, Seat, SeatStatus


class PaymentStrategy:
    def charge(self, user_id: str, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Invalid payment amount")


@dataclass
class Booking:
    booking_id: str
    user_id: str
    concert_id: str
    seat_ids: List[str]
    total: float
    confirmed_at: datetime


@dataclass
class SeatHold:
    hold_id: str
    user_id: str
    concert_id: str
    seat_ids: List[str]
    expires_at: datetime


@dataclass
class BookingService:
    payment: PaymentStrategy = field(default_factory=PaymentStrategy)
    _concerts: Dict[str, Concert] = field(default_factory=dict)
    _seats: Dict[str, Dict[str, Seat]] = field(default_factory=dict)  # concert_id -> seat_id -> Seat
    _holds: Dict[str, SeatHold] = field(default_factory=dict)
    _bookings: Dict[str, Booking] = field(default_factory=dict)
    _lock: threading.Lock = field(default_factory=threading.Lock)

    def add_concert(self, concert: Concert, seats: List[Seat]) -> None:
        self._concerts[concert.concert_id] = concert
        self._seats[concert.concert_id] = {s.seat_id: s for s in seats}

    def search(self, *, artist: str | None = None, venue: str | None = None) -> List[Concert]:
        concerts = list(self._concerts.values())
        if artist:
            concerts = [c for c in concerts if artist.lower() in c.artist.lower()]
        if venue:
            concerts = [c for c in concerts if venue.lower() in c.venue.lower()]
        return concerts

    def hold_seats(self, user_id: str, concert_id: str, seat_ids: List[str], ttl_seconds: int = 300) -> SeatHold:
        with self._lock:
            seats = self._seats[concert_id]
            # expire stale holds
            self._expire_holds_locked()
            # verify availability
            for sid in seat_ids:
                if seats[sid].status != SeatStatus.AVAILABLE:
                    raise RuntimeError(f"Seat {sid} not available")
            # mark held
            for sid in seat_ids:
                seats[sid].status = SeatStatus.HELD
            hold = SeatHold(
                hold_id=f"HOLD-{uuid.uuid4().hex[:8].upper()}",
                user_id=user_id,
                concert_id=concert_id,
                seat_ids=list(seat_ids),
                expires_at=datetime.utcnow() + timedelta(seconds=ttl_seconds),
            )
            self._holds[hold.hold_id] = hold
            return hold

    def confirm(self, hold_id: str) -> Booking:
        with self._lock:
            hold = self._holds.pop(hold_id, None)
            if not hold or hold.expires_at <= datetime.utcnow():
                raise RuntimeError("Hold expired or not found")
            seats = self._seats[hold.concert_id]
            total = sum(seats[sid].price for sid in hold.seat_ids)
            # charge
            self.payment.charge(user_id=hold.user_id, amount=total)
            # mark booked
            for sid in hold.seat_ids:
                seats[sid].status = SeatStatus.BOOKED
            booking = Booking(
                booking_id=f"BOOK-{uuid.uuid4().hex[:8].upper()}",
                user_id=hold.user_id,
                concert_id=hold.concert_id,
                seat_ids=hold.seat_ids,
                total=total,
                confirmed_at=datetime.utcnow(),
            )
            self._bookings[booking.booking_id] = booking
            return booking

    def cancel(self, booking_id: str) -> None:
        with self._lock:
            booking = self._bookings.pop(booking_id, None)
            if not booking:
                return
            seats = self._seats[booking.concert_id]
            for sid in booking.seat_ids:
                if seats[sid].status == SeatStatus.BOOKED:
                    seats[sid].status = SeatStatus.AVAILABLE

    def _expire_holds_locked(self) -> None:
        now = datetime.utcnow()
        expired = [hid for hid, h in self._holds.items() if h.expires_at <= now]
        for hid in expired:
            hold = self._holds.pop(hid)
            for sid in hold.seat_ids:
                self._seats[hold.concert_id][sid].status = SeatStatus.AVAILABLE
```