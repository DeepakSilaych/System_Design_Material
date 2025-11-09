# Designing a Task Management System

## Question
Design a Task Management System

## Requirements
1. The task management system should allow users to create, update, and delete tasks.
2. Each task should have a title, description, due date, priority, and status (e.g., pending, in progress, completed).
3. Users should be able to assign tasks to other users and set reminders for tasks.
4. The system should support searching and filtering tasks based on various criteria (e.g., priority, due date, assigned user).
5. Users should be able to mark tasks as completed and view their task history.
6. The system should handle concurrent access to tasks and ensure data consistency.
7. The system should be extensible to accommodate future enhancements and new features.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: User, TaskStatus, Task, TaskManager, TaskManagementSystem.

### Design Details
1. The **User** class represents a user in the task management system, with properties such as id, name, and email.
2. The **TaskStatus** enum defines the possible states of a task, such as pending, in progress, and completed.
3. The **Task** class represents a task in the system, with properties like id, title, description, due date, priority, status, and assigned user.
4. The **TaskManager** class is the core of the task management system and follows the Singleton pattern to ensure a single instance of the task manager.
5. The TaskManager class uses concurrent data structures (ConcurrentHashMap and CopyOnWriteArrayList) to handle concurrent access to tasks and ensure thread safety.
6. The TaskManager class provides methods for creating, updating, deleting, searching, and filtering tasks, as well as marking tasks as completed and retrieving task history for a user.
7. The **TaskManagementSystem** class serves as the entry point of the application and demonstrates the usage of the task management system.

### Implementation (Python)
#### activity_log.py

```python
from datetime import datetime

class ActivityLog:
    def __init__(self, description: str):
        self._description = description
        self._timestamp = datetime.now()
    
    def __str__(self) -> str:
        return f"[{self._timestamp}] {self._description}"
```

#### comment.py

```python
import uuid
from datetime import datetime
from user import User

class Comment:
    def __init__(self, content: str, author: User):
        self._id = str(uuid.uuid4())
        self._content = content
        self._author = author
        self._timestamp = datetime.now()
    
    @property
    def author(self) -> User:
        return self._author
```

#### tag.py

```python
class Tag:
    def __init__(self, name: str):
        self._name = name
    
    @property
    def name(self) -> str:
        return self._name
```

#### task.py

```python
import threading
from typing import List, Optional, Set
from datetime import date
import uuid
from user import User
from task_priority import TaskPriority
from task_status import TaskStatus
from comment import Comment
from activity_log import ActivityLog
from task_state import TodoState, TaskState
from task_observer import TaskObserver
from tag import Tag

class Task:
    def __init__(self, builder: 'TaskBuilder'):
        self._id = builder._id
        self._title = builder._title
        self._description = builder._description
        self._due_date = builder._due_date
        self._priority = builder._priority
        self._created_by = builder._created_by
        self._assignee = builder._assignee
        self._tags = builder._tags
        self._current_state = TodoState()  # Initial state
        self._comments: List[Comment] = []
        self._subtasks: List['Task'] = []
        self._activity_logs: List[ActivityLog] = []
        self._observers: List[TaskObserver] = []
        self._lock = threading.Lock()
        self.add_log(f"Task created with title: {self._title}")
    
    def set_assignee(self, user: User):
        with self._lock:
            self._assignee = user
            self.add_log(f"Assigned to {user.name}")
            self.notify_observers("assignee")
    
    def update_priority(self, priority: TaskPriority):
        with self._lock:
            self._priority = priority
            self.notify_observers("priority")
    
    def add_comment(self, comment: Comment):
        with self._lock:
            self._comments.append(comment)
            self.add_log(f"Comment added by {comment.author.name}")
            self.notify_observers("comment")
    
    def add_subtask(self, subtask: 'Task'):
        with self._lock:
            self._subtasks.append(subtask)
            self.add_log(f"Subtask added: {subtask.get_title()}")
            self.notify_observers("subtask_added")
    
    # State Pattern Methods
    def set_state(self, state: TaskState):
        self._current_state = state
        self.add_log(f"Status changed to: {state.get_status().value}")
        self.notify_observers("status")
    
    def start_progress(self):
        self._current_state.start_progress(self)
    
    def complete_task(self):
        self._current_state.complete_task(self)
    
    def reopen_task(self):
        self._current_state.reopen_task(self)
    
    # Observer Pattern Methods
    def add_observer(self, observer: TaskObserver):
        self._observers.append(observer)
    
    def remove_observer(self, observer: TaskObserver):
        if observer in self._observers:
            self._observers.remove(observer)
    
    def notify_observers(self, change_type: str):
        for observer in self._observers:
            observer.update(self, change_type)
    
    def add_log(self, log_description: str):
        self._activity_logs.append(ActivityLog(log_description))
    
    def is_composite(self) -> bool:
        return len(self._subtasks) > 0
    
    def display(self, indent: str = ""):
        print(f"{indent}- {self._title} [{self.get_status().value}, {self._priority.value}, Due: {self._due_date}]")
        if self.is_composite():
            for subtask in self._subtasks:
                subtask.display(indent + "  ")
    
    # Getters and setters
    def get_id(self) -> str:
        return self._id
    
    def get_title(self) -> str:
        return self._title
    
    def get_description(self) -> str:
        return self._description
    
    def get_priority(self) -> TaskPriority:
        return self._priority
    
    def get_due_date(self) -> date:
        return self._due_date
    
    def get_assignee(self) -> Optional[User]:
        return self._assignee
    
    def set_title(self, title: str):
        self._title = title
    
    def set_description(self, description: str):
        self._description = description
    
    def get_status(self) -> TaskStatus:
        return self._current_state.get_status()
    
    # Builder Pattern
    class TaskBuilder:
        def __init__(self, title: str):
            self._id = str(uuid.uuid4())
            self._title = title
            self._description = ""
            self._due_date = None
            self._priority = None
            self._created_by = None
            self._assignee = None
            self._tags = set()
        
        def description(self, description: str) -> 'Task.TaskBuilder':
            self._description = description
            return self
        
        def due_date(self, due_date: date) -> 'Task.TaskBuilder':
            self._due_date = due_date
            return self
        
        def priority(self, priority: TaskPriority) -> 'Task.TaskBuilder':
            self._priority = priority
            return self
        
        def assignee(self, assignee: User) -> 'Task.TaskBuilder':
            self._assignee = assignee
            return self
        
        def created_by(self, created_by: User) -> 'Task.TaskBuilder':
            self._created_by = created_by
            return self
        
        def tags(self, tags: Set[Tag]) -> 'Task.TaskBuilder':
            self._tags = tags
            return self
        
        def build(self) -> 'Task':
            return Task(self)
```

#### task_list.py

```python
import threading
import uuid
from typing import List
from task import Task

class TaskList:
    def __init__(self, name: str):
        self._id = str(uuid.uuid4())
        self._name = name
        self._tasks: List[Task] = []
        self._lock = threading.Lock()
    
    def add_task(self, task: Task):
        with self._lock:
            self._tasks.append(task)
    
    def get_tasks(self) -> List[Task]:
        with self._lock:
            return self._tasks.copy()  # Return a copy to prevent external modification
    
    @property
    def id(self) -> str:
        return self._id
    
    @property
    def name(self) -> str:
        return self._name
    
    def display(self):
        print(f"--- Task List: {self._name} ---")
        for task in self._tasks:
            task.display("")
        print("-----------------------------------")
```

#### task_management_system.py

```python
import threading
from typing import Dict, List
from user import User
from task import Task
from task_list import TaskList
from task_priority import TaskPriority
from task_status import TaskStatus
from task_sort_strategy import TaskSortStrategy
from task_observer import ActivityLogger
from datetime import date

class TaskManagementSystem:
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
            self._tasks: Dict[str, Task] = {}
            self._task_lists: Dict[str, TaskList] = {}
            self._initialized = True
    
    @classmethod
    def get_instance(cls):
        return cls()
    
    def create_user(self, name: str, email: str) -> User:
        user = User(name, email)
        self._users[user.id] = user
        return user
    
    def create_task_list(self, list_name: str) -> TaskList:
        task_list = TaskList(list_name)
        self._task_lists[task_list.id] = task_list
        return task_list
    
    def create_task(self, title: str, description: str, due_date: date,
                   priority: TaskPriority, created_by_user_id: str) -> Task:
        created_by = self._users.get(created_by_user_id)
        if created_by is None:
            raise ValueError("User not found.")
        
        task = Task.TaskBuilder(title) \
            .description(description) \
            .due_date(due_date) \
            .priority(priority) \
            .created_by(created_by) \
            .build()
        
        task.add_observer(ActivityLogger())
        
        self._tasks[task.get_id()] = task
        return task
    
    def list_tasks_by_user(self, user_id: str) -> List[Task]:
        user = self._users.get(user_id)
        return [task for task in self._tasks.values() 
                if task.get_assignee() == user]
    
    def list_tasks_by_status(self, status: TaskStatus) -> List[Task]:
        return [task for task in self._tasks.values() 
                if task.get_status() == status]
    
    def delete_task(self, task_id: str):
        if task_id in self._tasks:
            del self._tasks[task_id]
    
    def search_tasks(self, keyword: str, sorting_strategy: TaskSortStrategy) -> List[Task]:
        matching_tasks = []
        for task in self._tasks.values():
            if (keyword in task.get_title() or 
                keyword in task.get_description()):
                matching_tasks.append(task)
        
        sorting_strategy.sort(matching_tasks)
        return matching_tasks
```

#### task_management_system_demo.py

```python
from task_management_system import TaskManagementSystem
from task_priority import TaskPriority
from task_status import TaskStatus
from datetime import date
from task_sort_strategy import SortByDueDate

class TaskManagementSystemDemo:
    @staticmethod
    def main():
        task_management_system = TaskManagementSystem.get_instance()
        
        # Create users
        user1 = task_management_system.create_user("John Doe", "john@example.com")
        user2 = task_management_system.create_user("Jane Smith", "jane@example.com")
        
        # Create task lists
        task_list1 = task_management_system.create_task_list("Enhancements")
        task_list2 = task_management_system.create_task_list("Bug Fix")
        
        # Create tasks
        task1 = task_management_system.create_task(
            "Enhancement Task", "Launch New Feature",
            date.today().replace(day=date.today().day + 2), 
            TaskPriority.LOW, user1.id
        )
        subtask1 = task_management_system.create_task(
            "Enhancement sub task", "Design UI/UX",
            date.today().replace(day=date.today().day + 1), 
            TaskPriority.MEDIUM, user1.id
        )
        task2 = task_management_system.create_task(
            "Bug Fix Task", "Fix API Bug",
            date.today().replace(day=date.today().day + 3), 
            TaskPriority.HIGH, user2.id
        )
        
        task1.add_subtask(subtask1)
        
        task_list1.add_task(task1)
        task_list2.add_task(task2)
        
        task_list1.display()
        
        # Update task status
        subtask1.start_progress()
        
        # Assign task
        subtask1.set_assignee(user2)
        
        task_list1.display()
        
        # Search tasks
        search_results = task_management_system.search_tasks("Task", SortByDueDate())
        print("\nTasks with keyword Task:")
        for task in search_results:
            print(task.get_title())
        
        # Filter tasks by status
        filtered_tasks = task_management_system.list_tasks_by_status(TaskStatus.TODO)
        print("\nTODO Tasks:")
        for task in filtered_tasks:
            print(task.get_title())
        
        # Mark a task as done
        subtask1.complete_task()
        
        # Get tasks assigned to a user
        user_task_list = task_management_system.list_tasks_by_user(user2.id)
        print(f"\nTask for {user2.name}:")
        for task in user_task_list:
            print(task.get_title())
        
        task_list1.display()
        
        # Delete a task
        task_management_system.delete_task(task2.get_id())

if __name__ == "__main__":
    TaskManagementSystemDemo.main()
```

#### task_observer.py

```python
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from task import Task

class TaskObserver(ABC):
    @abstractmethod
    def update(self, task: 'Task', change_type: str):
        pass

class ActivityLogger(TaskObserver):
    def update(self, task: 'Task', change_type: str):
        print(f"LOGGER: Task '{task.get_title()}' was updated. Change: {change_type}")
```

#### task_priority.py

```python
from enum import Enum

class TaskPriority(Enum):
    LOW = "LOW"
    MEDIUM = "MEDIUM"
    HIGH = "HIGH"
    CRITICAL = "CRITICAL"
```

#### task_sort_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from task import Task
from task_priority import TaskPriority
from datetime import date

class TaskSortStrategy(ABC):
    @abstractmethod
    def sort(self, tasks: List[Task]):
        pass

class SortByPriority(TaskSortStrategy):
    def sort(self, tasks: List[Task]):
        # Higher priority comes first (CRITICAL > HIGH > MEDIUM > LOW)
        priority_order = {TaskPriority.CRITICAL: 4, TaskPriority.HIGH: 3, 
                         TaskPriority.MEDIUM: 2, TaskPriority.LOW: 1}
        tasks.sort(key=lambda task: priority_order.get(task.get_priority(), 0), reverse=True)

class SortByDueDate(TaskSortStrategy):
    def sort(self, tasks: List[Task]):
        tasks.sort(key=lambda task: task.get_due_date() if task.get_due_date() else date.max)
```

#### task_state.py

```python
from abc import ABC, abstractmethod
from task_status import TaskStatus
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from task import Task

class TaskState(ABC):
    @abstractmethod
    def start_progress(self, task: 'Task'):
        pass
    
    @abstractmethod
    def complete_task(self, task: 'Task'):
        pass
    
    @abstractmethod
    def reopen_task(self, task: 'Task'):
        pass
    
    @abstractmethod
    def get_status(self) -> TaskStatus:
        pass

class TodoState(TaskState):
    def start_progress(self, task: 'Task'):
        task.set_state(InProgressState())
    
    def complete_task(self, task: 'Task'):
        print("Cannot complete a task that is not in progress.")
    
    def reopen_task(self, task: 'Task'):
        print("Task is already in TO-DO state.")
    
    def get_status(self) -> TaskStatus:
        return TaskStatus.TODO

class InProgressState(TaskState):
    def start_progress(self, task: 'Task'):
        print("Task is already in progress.")
    
    def complete_task(self, task: 'Task'):
        task.set_state(DoneState())
    
    def reopen_task(self, task: 'Task'):
        task.set_state(TodoState())
    
    def get_status(self) -> TaskStatus:
        return TaskStatus.IN_PROGRESS

class DoneState(TaskState):
    def start_progress(self, task: 'Task'):
        print("Cannot start a completed task. Reopen it first.")
    
    def complete_task(self, task: 'Task'):
        print("Task is already done.")
    
    def reopen_task(self, task: 'Task'):
        task.set_state(TodoState())
    
    def get_status(self) -> TaskStatus:
        return TaskStatus.DONE
```

#### task_status.py

```python
from enum import Enum

class TaskStatus(Enum):
    TODO = "TODO"
    IN_PROGRESS = "IN_PROGRESS"
    DONE = "DONE"
    BLOCKED = "BLOCKED"
```

#### user.py

```python
import uuid

class User:
    def __init__(self, name: str, email: str):
        self._id = str(uuid.uuid4())
        self._name = name
        self._email = email
    
    @property
    def id(self) -> str:
        return self._id
    
    @property
    def email(self) -> str:
        return self._email
    
    @property
    def name(self) -> str:
        return self._name
```