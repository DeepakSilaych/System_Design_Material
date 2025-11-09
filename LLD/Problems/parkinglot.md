# Designing a Parking Lot System

## Question
Design a Parking Lot System

## Requirements
1. The parking lot should have multiple levels, each level with a certain number of parking spots.
2. The parking lot should support different types of vehicles, such as cars, motorcycles, and trucks.
3. Each parking spot should be able to accommodate a specific type of vehicle.
4. The system should assign a parking spot to a vehicle upon entry and release it when the vehicle exits.
5. The system should track the availability of parking spots and provide real-time information to customers.
6. The system should handle multiple entry and exit points and support concurrent access.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: ParkingLot, Level, ParkingSpot, Vehicle, VehicleType, Main.

### Design Details
1. The **ParkingLot** class follows the Singleton pattern to ensure only one instance of the parking lot exists. It maintains a list of levels and provides methods to park and unpark vehicles.
2. The **Level** class represents a level in the parking lot and contains a list of parking spots. It handles parking and unparking of vehicles within the level.
3. The **ParkingSpot** class represents an individual parking spot and tracks the availability and the parked vehicle.
4. The **Vehicle** class is an abstract base class for different types of vehicles. It is extended by Car, Motorcycle, and Truck classes.
5. The **VehicleType** enum defines the different types of vehicles supported by the parking lot.
6. Multi-threading is achieved through the use of synchronized keyword on critical sections to ensure thread safety.
7. The **Main** class demonstrates the usage of the parking lot system.

### Implementation (Python)
#### bike.py

```python
from vehicle import Vehicle
from vehicle_size import VehicleSize

class Bike(Vehicle):
    def __init__(self, license_number: str):
        super().__init__(license_number, VehicleSize.SMALL)
```

#### car.py

```python
from vehicle import Vehicle
from vehicle_size import VehicleSize

class Car(Vehicle):
    def __init__(self, license_number: str):
        super().__init__(license_number, VehicleSize.MEDIUM)
```

#### fee_strategy.py

```python
from abc import ABC, abstractmethod
from parking_ticket import ParkingTicket
from vehicle_size import VehicleSize

class FeeStrategy(ABC):
    @abstractmethod
    def calculate_fee(self, parking_ticket: ParkingTicket) -> float:
        pass

class FlatRateFeeStrategy(FeeStrategy):
    RATE_PER_HOUR = 10.0

    def calculate_fee(self, parking_ticket: ParkingTicket) -> float:
        duration = parking_ticket.get_exit_timestamp() - parking_ticket.get_entry_timestamp()
        hours = (duration // (1000 * 60 * 60)) + 1
        return hours * self.RATE_PER_HOUR

class VehicleBasedFeeStrategy(FeeStrategy):
    HOURLY_RATES = {
        VehicleSize.SMALL: 10.0,
        VehicleSize.MEDIUM: 20.0,
        VehicleSize.LARGE: 30.0
    }

    def calculate_fee(self, parking_ticket: ParkingTicket) -> float:
        duration = parking_ticket.get_exit_timestamp() - parking_ticket.get_entry_timestamp()
        hours = (duration // (1000 * 60 * 60)) + 1
        return hours * self.HOURLY_RATES[parking_ticket.get_vehicle().get_size()]
```

#### parking_floor.py

```python
import threading
from parking_spot import ParkingSpot
from vehicle import Vehicle
from vehicle_size import VehicleSize
from typing import Dict, Optional
from collections import defaultdict

class ParkingFloor:
    def __init__(self, floor_number: int):
        self.floor_number = floor_number
        self.spots: Dict[str, ParkingSpot] = {}
        self._lock = threading.Lock()

    def add_spot(self, spot: ParkingSpot):
        self.spots[spot.get_spot_id()] = spot

    def find_available_spot(self, vehicle: Vehicle) -> Optional[ParkingSpot]:
        with self._lock:
            available_spots = [
                spot for spot in self.spots.values()
                if not spot.is_occupied_spot() and spot.can_fit_vehicle(vehicle)
            ]
            if available_spots:
                # Sort by spot size (smallest first)
                available_spots.sort(key=lambda x: x.get_spot_size().value)
                return available_spots[0]
            return None

    def display_availability(self):
        print(f"--- Floor {self.floor_number} Availability ---")
        available_counts = defaultdict(int)
        
        for spot in self.spots.values():
            if not spot.is_occupied_spot():
                available_counts[spot.get_spot_size()] += 1

        for size in VehicleSize:
            print(f"  {size.value} spots: {available_counts[size]}")
```

#### parking_lot.py

```python
import threading
from parking_floor import ParkingFloor
from parking_ticket import ParkingTicket
from fee_strategy import FlatRateFeeStrategy, FeeStrategy
from parking_strategy import NearestFirstStrategy, ParkingStrategy
from vehicle import Vehicle
from typing import List, Dict, Optional

class ParkingLot:
    _instance = None
    _lock = threading.Lock()

    def __init__(self):
        if ParkingLot._instance is not None:
            raise Exception("This class is a singleton!")
        self.floors: List[ParkingFloor] = []
        self.active_tickets: Dict[str, ParkingTicket] = {}
        self.fee_strategy = FlatRateFeeStrategy()
        self.parking_strategy = NearestFirstStrategy()
        self._main_lock = threading.Lock()

    @staticmethod
    def get_instance():
        if ParkingLot._instance is None:
            with ParkingLot._lock:
                if ParkingLot._instance is None:
                    ParkingLot._instance = ParkingLot()
        return ParkingLot._instance

    def add_floor(self, floor: ParkingFloor):
        self.floors.append(floor)

    def set_fee_strategy(self, fee_strategy: FeeStrategy):
        self.fee_strategy = fee_strategy

    def set_parking_strategy(self, parking_strategy: ParkingStrategy):
        self.parking_strategy = parking_strategy

    def park_vehicle(self, vehicle: Vehicle) -> Optional[ParkingTicket]:
        with self._main_lock:
            spot = self.parking_strategy.find_spot(self.floors, vehicle)
            if spot is not None:
                spot.park_vehicle(vehicle)
                ticket = ParkingTicket(vehicle, spot)
                self.active_tickets[vehicle.get_license_number()] = ticket
                print(f"Vehicle {vehicle.get_license_number()} parked at spot {spot.get_spot_id()}")
                return ticket
            else:
                print(f"No available spot for vehicle {vehicle.get_license_number()}")
                return None

    def unpark_vehicle(self, license_number: str) -> Optional[float]:
        with self._main_lock:
            ticket = self.active_tickets.pop(license_number, None)
            if ticket is None:
                print(f"Ticket not found for vehicle {license_number}")
                return None

            ticket.get_spot().unpark_vehicle()
            ticket.set_exit_timestamp()
            fee = self.fee_strategy.calculate_fee(ticket)
            print(f"Vehicle {license_number} unparked from spot {ticket.get_spot().get_spot_id()}")
            return fee
```

#### parking_lot_demo.py

```python
from parking_lot import ParkingLot
from parking_floor import ParkingFloor
from parking_spot import ParkingSpot
from vehicle_size import VehicleSize
from fee_strategy import VehicleBasedFeeStrategy
from bike import Bike
from car import Car
from truck import Truck

class ParkingLotDemo:
    @staticmethod
    def main():
        parking_lot = ParkingLot.get_instance()

        # 1. Initialize the parking lot with floors and spots
        floor1 = ParkingFloor(1)
        floor1.add_spot(ParkingSpot("F1-S1", VehicleSize.SMALL))
        floor1.add_spot(ParkingSpot("F1-M1", VehicleSize.MEDIUM))
        floor1.add_spot(ParkingSpot("F1-L1", VehicleSize.LARGE))

        floor2 = ParkingFloor(2)
        floor2.add_spot(ParkingSpot("F2-M1", VehicleSize.MEDIUM))
        floor2.add_spot(ParkingSpot("F2-M2", VehicleSize.MEDIUM))

        parking_lot.add_floor(floor1)
        parking_lot.add_floor(floor2)

        parking_lot.set_fee_strategy(VehicleBasedFeeStrategy())

        # 2. Simulate vehicle entries
        print("\n--- Vehicle Entries ---")
        floor1.display_availability()
        floor2.display_availability()

        bike = Bike("B-123")
        car = Car("C-456")
        truck = Truck("T-789")

        bike_ticket = parking_lot.park_vehicle(bike)
        car_ticket = parking_lot.park_vehicle(car)
        truck_ticket = parking_lot.park_vehicle(truck)

        print("\n--- Availability after parking ---")
        floor1.display_availability()
        floor2.display_availability()

        # 3. Simulate another car entry (should go to floor 2)
        car2 = Car("C-999")
        car2_ticket = parking_lot.park_vehicle(car2)

        # 4. Simulate a vehicle entry that fails (no available spots)
        bike2 = Bike("B-000")
        failed_bike_ticket = parking_lot.park_vehicle(bike2)

        # 5. Simulate vehicle exits and fee calculation
        print("\n--- Vehicle Exits ---")

        if car_ticket is not None:
            fee = parking_lot.unpark_vehicle(car.get_license_number())
            if fee is not None:
                print(f"Car C-456 unparked. Fee: ${fee:.2f}")

        print("\n--- Availability after one car leaves ---")
        floor1.display_availability()
        floor2.display_availability()


if __name__ == "__main__":
    ParkingLotDemo.main()
```

#### parking_spot.py

```python
import threading
from vehicle import Vehicle
from vehicle_size import VehicleSize

class ParkingSpot:
    def __init__(self, spot_id: str, spot_size: VehicleSize):
        self.spot_id = spot_id
        self.spot_size = spot_size
        self.is_occupied = False
        self.parked_vehicle = None
        self._lock = threading.Lock()

    def get_spot_id(self) -> str:
        return self.spot_id

    def get_spot_size(self) -> VehicleSize:
        return self.spot_size

    def is_available(self) -> bool:
        with self._lock:
            return not self.is_occupied

    def is_occupied_spot(self) -> bool:
        return self.is_occupied

    def park_vehicle(self, vehicle: Vehicle):
        with self._lock:
            self.parked_vehicle = vehicle
            self.is_occupied = True

    def unpark_vehicle(self):
        with self._lock:
            self.parked_vehicle = None
            self.is_occupied = False

    def can_fit_vehicle(self, vehicle: Vehicle) -> bool:
        if self.is_occupied:
            return False

        if vehicle.get_size() == VehicleSize.SMALL:
            return self.spot_size == VehicleSize.SMALL
        elif vehicle.get_size() == VehicleSize.MEDIUM:
            return self.spot_size == VehicleSize.MEDIUM or self.spot_size == VehicleSize.LARGE
        elif vehicle.get_size() == VehicleSize.LARGE:
            return self.spot_size == VehicleSize.LARGE
        else:
            return False
```

#### parking_strategy.py

```python
from abc import ABC, abstractmethod
from parking_floor import ParkingFloor
from vehicle import Vehicle
from parking_spot import ParkingSpot
from vehicle_size import VehicleSize
from typing import List, Optional

class ParkingStrategy(ABC):
    @abstractmethod
    def find_spot(self, floors: List[ParkingFloor], vehicle: Vehicle) -> Optional[ParkingSpot]:
        pass

class NearestFirstStrategy(ParkingStrategy):
    def find_spot(self, floors: List[ParkingFloor], vehicle: Vehicle) -> Optional[ParkingSpot]:
        for floor in floors:
            spot = floor.find_available_spot(vehicle)
            if spot is not None:
                return spot
        return None

class FarthestFirstStrategy(ParkingStrategy):
    def find_spot(self, floors: List[ParkingFloor], vehicle: Vehicle) -> Optional[ParkingSpot]:
        reversed_floors = list(reversed(floors))
        for floor in reversed_floors:
            spot = floor.find_available_spot(vehicle)
            if spot is not None:
                return spot
        return None

class BestFitStrategy(ParkingStrategy):
    def find_spot(self, floors: List[ParkingFloor], vehicle: Vehicle) -> Optional[ParkingSpot]:
        best_spot = None

        for floor in floors:
            spot_on_this_floor = floor.find_available_spot(vehicle)

            if spot_on_this_floor is not None:
                if best_spot is None:
                    best_spot = spot_on_this_floor
                else:
                    # A smaller spot size enum ordinal means a tighter fit
                    if list(VehicleSize).index(spot_on_this_floor.get_spot_size()) < list(VehicleSize).index(best_spot.get_spot_size()):
                        best_spot = spot_on_this_floor

        return best_spot
```

#### parking_ticket.py

```python
import uuid
import time
from vehicle import Vehicle
from parking_spot import ParkingSpot

class ParkingTicket:
    def __init__(self, vehicle: Vehicle, spot: ParkingSpot):
        self.ticket_id = str(uuid.uuid4())
        self.vehicle = vehicle
        self.spot = spot
        self.entry_timestamp = int(time.time() * 1000)
        self.exit_timestamp = 0

    def get_ticket_id(self) -> str:
        return self.ticket_id

    def get_vehicle(self) -> Vehicle:
        return self.vehicle

    def get_spot(self) -> ParkingSpot:
        return self.spot

    def get_entry_timestamp(self) -> int:
        return self.entry_timestamp

    def get_exit_timestamp(self) -> int:
        return self.exit_timestamp

    def set_exit_timestamp(self):
        self.exit_timestamp = int(time.time() * 1000)
```

#### truck.py

```python
from vehicle import Vehicle
from vehicle_size import VehicleSize

class Truck(Vehicle):
    def __init__(self, license_number: str):
        super().__init__(license_number, VehicleSize.LARGE)
```

#### vehicle.py

```python
from abc import ABC
from vehicle_size import VehicleSize

class Vehicle(ABC):
    def __init__(self, license_number: str, size: VehicleSize):
        self.license_number = license_number
        self.size = size

    def get_license_number(self) -> str:
        return self.license_number

    def get_size(self) -> VehicleSize:
        return self.size
```

#### vehicle_size.py

```python
from enum import Enum

class VehicleSize(Enum):
    SMALL = "SMALL"
    MEDIUM = "MEDIUM"
    LARGE = "LARGE"
```