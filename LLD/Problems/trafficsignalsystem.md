# Designing a Traffic Signal Control System

## Question
Design a Traffic Signal Control System

## Requirements
1. The traffic signal system should control the flow of traffic at an intersection with multiple roads.
2. The system should support different types of signals, such as red, yellow, and green.
3. The duration of each signal should be configurable and adjustable based on traffic conditions.
4. The system should handle the transition between signals smoothly, ensuring safe and efficient traffic flow.
5. The system should be able to detect and handle emergency situations, such as an ambulance or fire truck approaching the intersection.
6. The system should be scalable and extensible to support additional features and functionality.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: Signal, Road, TrafficLight, TrafficController, TrafficSignalSystemDemo.

### Design Details
1. The **Signal** enum represents the different states of a traffic light: red, yellow, and green.
2. The **Road** class represents a road in the traffic signal system, with properties such as ID, name, and an associated traffic light.
3. The **TrafficLight** class represents a traffic light, with properties such as ID, current signal, and durations for each signal state. It provides methods to change the signal and notify observers (e.g., roads) about signal changes.
4. The **TrafficController** class serves as the central controller for the traffic signal system. It follows the Singleton pattern to ensure a single instance of the controller. It manages the roads and their associated traffic lights, starts the traffic control process, and handles emergency situations.
5. The **TrafficSignalSystemDemo** class is the main entry point of the application. It demonstrates the usage of the traffic signal system by creating roads, traffic lights, assigning traffic lights to roads, and starting the traffic control process.

### Implementation (Python)
#### direction.py

```python
from enum import Enum

class Direction(Enum):
    NORTH = "NORTH"
    SOUTH = "SOUTH"
    EAST = "EAST"
    WEST = "WEST"
```

#### intersection_controller.py

```python
from direction import Direction
from traffic_light import TrafficLight
from intersection_state import IntersectionState, NorthSouthGreenState
from observer import TrafficObserver
from typing import Dict, List
import threading

class IntersectionController:
    def __init__(self, intersection_id: int, traffic_lights: Dict[Direction, TrafficLight], 
                 green_duration: int, yellow_duration: int):
        self._id = intersection_id
        self._traffic_lights = traffic_lights
        self._green_duration = green_duration
        self._yellow_duration = yellow_duration
        self._current_state = NorthSouthGreenState()  # Initial state for the intersection
        self._running = True

    def get_id(self) -> int:
        return self._id

    def get_green_duration(self) -> int:
        return self._green_duration

    def get_yellow_duration(self) -> int:
        return self._yellow_duration

    def get_light(self, direction: Direction) -> TrafficLight:
        return self._traffic_lights[direction]

    def set_state(self, state: IntersectionState):
        self._current_state = state

    def start(self):
        thread = threading.Thread(target=self.run)
        thread.start()

    def stop(self):
        self._running = False

    def run(self):
        while self._running:
            try:
                self._current_state.handle(self)
            except Exception as e:
                print(f"Intersection {self._id} encountered an error: {e}")
                self._running = False

    # Builder Pattern
    class Builder:
        def __init__(self, intersection_id: int):
            self._id = intersection_id
            self._green_duration = 5000  # default 5s
            self._yellow_duration = 2000  # default 2s
            self._observers: List[TrafficObserver] = []

        def with_durations(self, green: int, yellow: int) -> 'IntersectionController.Builder':
            self._green_duration = green
            self._yellow_duration = yellow
            return self

        def add_observer(self, observer: TrafficObserver) -> 'IntersectionController.Builder':
            self._observers.append(observer)
            return self

        def build(self) -> 'IntersectionController':
            lights = {}
            for direction in Direction:
                light = TrafficLight(self._id, direction)
                # Attach all registered observers to each light
                for observer in self._observers:
                    light.add_observer(observer)
                lights[direction] = light
            return IntersectionController(self._id, lights, self._green_duration, self._yellow_duration)
```

#### intersection_state.py

```python
from abc import ABC, abstractmethod
from direction import Direction
from light_color import LightColor
from typing import TYPE_CHECKING
import time

if TYPE_CHECKING:
    from intersection_controller import IntersectionController

class IntersectionState(ABC):
    @abstractmethod
    def handle(self, context: 'IntersectionController'):
        pass

class EastWestGreenState(IntersectionState):
    def handle(self, context: 'IntersectionController'):
        print(f"\n--- INTERSECTION {context.get_id()}: Cycle -> East-West GREEN ---")

        # Turn East and West green, ensure North and South are red
        context.get_light(Direction.EAST).start_green()
        context.get_light(Direction.WEST).start_green()
        context.get_light(Direction.NORTH).set_color(LightColor.RED)
        context.get_light(Direction.SOUTH).set_color(LightColor.RED)

        # Wait for green light duration
        time.sleep(context.get_green_duration() / 1000.0)

        # Transition East and West to Yellow
        context.get_light(Direction.EAST).transition()
        context.get_light(Direction.WEST).transition()

        # Wait for yellow light duration
        time.sleep(context.get_yellow_duration() / 1000.0)

        # Transition East and West to Red
        context.get_light(Direction.EAST).transition()
        context.get_light(Direction.WEST).transition()

        # Change the intersection's state back to let North-South go
        context.set_state(NorthSouthGreenState())

class NorthSouthGreenState(IntersectionState):
    def handle(self, context: 'IntersectionController'):
        print(f"\n--- INTERSECTION {context.get_id()}: Cycle Start -> North-South GREEN ---")

        # Turn North and South green, ensure East and West are red
        context.get_light(Direction.NORTH).start_green()
        context.get_light(Direction.SOUTH).start_green()
        context.get_light(Direction.EAST).set_color(LightColor.RED)
        context.get_light(Direction.WEST).set_color(LightColor.RED)

        # Wait for green light duration
        time.sleep(context.get_green_duration() / 1000.0)

        # Transition North and South to Yellow
        context.get_light(Direction.NORTH).transition()
        context.get_light(Direction.SOUTH).transition()

        # Wait for yellow light duration
        time.sleep(context.get_yellow_duration() / 1000.0)

        # Transition North and South to Red
        context.get_light(Direction.NORTH).transition()
        context.get_light(Direction.SOUTH).transition()

        # Change the intersection's state to let East-West go
        context.set_state(EastWestGreenState())
```

#### light_color.py

```python
from enum import Enum

class LightColor(Enum):
    GREEN = "GREEN"
    YELLOW = "YELLOW"
    RED = "RED"
```

#### observer.py

```python
from abc import ABC, abstractmethod
from direction import Direction
from light_color import LightColor

class TrafficObserver(ABC):
    @abstractmethod
    def update(self, intersection_id: int, direction: Direction, color: LightColor):
        pass

class CentralMonitor(TrafficObserver):
    def update(self, intersection_id: int, direction: Direction, color: LightColor):
        print(f"[MONITOR] Intersection {intersection_id}: Light for {direction.value} direction changed to {color.value}.")
```

#### signal_state.py

```python
from abc import ABC, abstractmethod
from light_color import LightColor
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from traffic_light import TrafficLight

class SignalState(ABC):
    @abstractmethod
    def handle(self, context: 'TrafficLight'):
        pass

class YellowState(SignalState):
    def handle(self, context: 'TrafficLight'):
        context.set_color(LightColor.YELLOW)
        # After being yellow, the next state is red.
        context.set_next_state(RedState())

class GreenState(SignalState):
    def handle(self, context: 'TrafficLight'):
        context.set_color(LightColor.GREEN)
        # After being green, the next state is yellow.
        context.set_next_state(YellowState())

class RedState(SignalState):
    def handle(self, context: 'TrafficLight'):
        context.set_color(LightColor.RED)
        # Red is a stable state, it transitions to green only when the intersection controller commands it.
        # So, the next state is self.
        context.set_next_state(RedState())
```

#### traffic_control_system.py

```python
from intersection_controller import IntersectionController
from observer import CentralMonitor
from typing import List
import threading
from concurrent.futures import ThreadPoolExecutor

class TrafficControlSystem:
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
            self._intersections: List[IntersectionController] = []
            self._executor_service = None
            self._initialized = True

    @classmethod
    def get_instance(cls):
        return cls()

    def add_intersection(self, intersection_id: int, green_duration: int, yellow_duration: int):
        intersection = IntersectionController.Builder(intersection_id) \
            .with_durations(green_duration, yellow_duration) \
            .add_observer(CentralMonitor()) \
            .build()
        self._intersections.append(intersection)

    def start_system(self):
        if not self._intersections:
            print("No intersections to manage. System not starting.")
            return

        print("--- Starting Traffic Control System ---")
        self._executor_service = ThreadPoolExecutor(max_workers=len(self._intersections))
        
        for intersection in self._intersections:
            self._executor_service.submit(intersection.run)

    def stop_system(self):
        print("\n--- Shutting Down Traffic Control System ---")
        
        for intersection in self._intersections:
            intersection.stop()
        
        if self._executor_service:
            self._executor_service.shutdown(wait=True)
        
        print("All intersections stopped. System shut down.")
```

#### traffic_light.py

```python
from direction import Direction
from light_color import LightColor
from signal_state import SignalState
from observer import TrafficObserver
from typing import List
from signal_state import RedState, GreenState

class TrafficLight:
    def __init__(self, intersection_id: int, direction: Direction):
        self._intersection_id = intersection_id
        self._direction = direction
        self._current_color = None
        self._current_state = RedState()  # Default state is Red
        self._next_state = None
        self._observers: List[TrafficObserver] = []
        self._current_state.handle(self)

    # This is called by the IntersectionController to initiate a G-Y-R cycle
    def start_green(self):
        self._current_state = GreenState()
        self._current_state.handle(self)

    # This is called by the IntersectionController to transition from G->Y or Y->R
    def transition(self):
        self._current_state = self._next_state
        self._current_state.handle(self)

    def set_color(self, color: LightColor):
        if self._current_color != color:
            self._current_color = color
            self._notify_observers()

    def set_next_state(self, state: SignalState):
        self._next_state = state

    def get_current_color(self) -> LightColor:
        return self._current_color

    def get_direction(self) -> Direction:
        return self._direction

    # Observer pattern methods
    def add_observer(self, observer: TrafficObserver):
        self._observers.append(observer)

    def remove_observer(self, observer: TrafficObserver):
        if observer in self._observers:
            self._observers.remove(observer)

    def _notify_observers(self):
        for observer in self._observers:
            observer.update(self._intersection_id, self._direction, self._current_color)
```

#### traffic_system_demo.py

```python
import time
from traffic_control_system import TrafficControlSystem

class TrafficSystemDemo:
    @staticmethod
    def main():
        # 1. Get the singleton TrafficControlSystem instance
        system = TrafficControlSystem.get_instance()

        # 2. Add intersections to the system
        system.add_intersection(1, 500, 200)
        system.add_intersection(2, 700, 150)

        # 3. Start the system
        system.start_system()

        # 4. Let the simulation run for a while (e.g., 5 seconds)
        try:
            time.sleep(5)
        except KeyboardInterrupt:
            pass

        # 5. Stop the system gracefully
        system.stop_system()

if __name__ == "__main__":
    TrafficSystemDemo.main()
```