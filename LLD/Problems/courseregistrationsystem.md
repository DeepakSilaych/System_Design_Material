# Designing a University Course Registration System

## Question
Design a University Course Registration System

## Requirements
1. The course registration system should allow students to register for courses and view their registered courses.
2. Each course should have a course code, name, instructor, and maximum enrollment capacity.
3. Students should be able to search for courses based on course code or name.
4. The system should prevent students from registering for courses that have reached their maximum enrollment capacity.
5. The system should handle concurrent registration requests from multiple students.
6. The system should ensure data consistency and prevent race conditions.
7. The system should be extensible to accommodate future enhancements and new features.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: Student, Course, Registration, CourseRegistrationSystem,
CourseRegistrationDemo.

### Design Details
1. The **Student** class represents a student in the course registration system, with properties such as ID, name, email, and a list of registered courses.
2. The **Course** class represents a course offered in the system, with properties such as code, name, instructor, maximum capacity, and the number of enrolled students.
3. The **Registration** class represents a registration record, associating a student with a course and capturing the registration timestamp.
4. The **CourseRegistrationSystem** class is the main class that manages the course registration system. It follows the Singleton pattern to ensure only one instance of the system exists.
5. The CourseRegistrationSystem class provides methods for adding courses and students, searching for courses, registering students for courses, and retrieving registered courses for a student.
6. Multi-threading is implemented using concurrent data structures (ConcurrentHashMap and CopyOnWriteArrayList) to handle concurrent access to shared data, such as courses and registrations.
7. The registerCourse method is synchronized to ensure thread safety when multiple students are registering for courses simultaneously.
8. The notifyObservers method is a placeholder for notifying observers (e.g., UI components) about updates to course enrollment.
9. The **CourseRegistrationDemo** class demonstrates the usage of the course registration system by creating courses and students, searching for courses, registering students for courses, and retrieving registered courses for a student.

### Implementation (Python)
#### course.py

```python
from datetime import datetime
from typing import List, Dict

class Course:
    def __init__(self, code: str, name: str, instructor: str, max_capacity: int, enrolled_students: int):
        self.code = code
        self.name = name
        self.instructor = instructor
        self.max_capacity = max_capacity
        self.enrolled_students = enrolled_students

    def set_enrolled_students(self, enrolled_students: int):
        self.enrolled_students = enrolled_students

    def get_code(self) -> str:
        return self.code

    def get_name(self) -> str:
        return self.name

    def get_instructor(self) -> str:
        return self.instructor

    def get_max_capacity(self) -> int:
        return self.max_capacity

    def get_enrolled_students(self) -> int:
        return self.enrolled_students
```

#### course_registration_demo.py

```python
from course_registration_system import CourseRegistrationSystem
from course import Course
from student import Student

class CourseRegistrationDemo:
    @staticmethod
    def run():
        registration_system = CourseRegistrationSystem.get_instance()

        # Create courses
        course1 = Course("CS101", "Introduction to Programming", "John Doe", 50, 0)
        course2 = Course("CS201", "Data Structures and Algorithms", "Jane Smith", 30, 0)
        registration_system.add_course(course1)
        registration_system.add_course(course2)

        # Create students
        student1 = Student(1, "Alice", "alice@example.com", [])
        student2 = Student(2, "Bob", "bob@example.com", [])
        registration_system.add_student(student1)
        registration_system.add_student(student2)

        # Search for courses
        search_results = registration_system.search_courses("CS")
        print("Search Results:")
        for course in search_results:
            print(f"{course.get_code()} - {course.get_name()}")

        # Register courses for students
        registered1 = registration_system.register_course(student1, course1)
        registered2 = registration_system.register_course(student2, course1)
        registered3 = registration_system.register_course(student1, course2)

        print("Registration Results:")
        print(f"Student 1 - Course 1: {registered1}")
        print(f"Student 2 - Course 1: {registered2}")
        print(f"Student 1 - Course 2: {registered3}")

        # Get registered courses for a student
        registered_courses = registration_system.get_registered_courses(student1)
        print("Registered Courses for Student 1:")
        for course in registered_courses:
            print(f"{course.get_code()} - {course.get_name()}")

if __name__ == "__main__":
    CourseRegistrationDemo.run()
```

#### course_registration_system.py

```python
from typing import Dict, List
from course import Course
from student import Student
from registration import Registration
from datetime import datetime

class CourseRegistrationSystem:
    instance = None
    courses: Dict[str, Course] = {}
    students: Dict[int, Student] = {}
    registrations: List[Registration] = []

    @staticmethod
    def get_instance():
        if CourseRegistrationSystem.instance is None:
            CourseRegistrationSystem.instance = CourseRegistrationSystem()
        return CourseRegistrationSystem.instance

    def add_course(self, course: Course):
        self.courses[course.get_code()] = course

    def add_student(self, student: Student):
        self.students[student.get_id()] = student

    def search_courses(self, query: str) -> List[Course]:
        result = []
        for course in self.courses.values():
            if query in course.get_code() or query in course.get_name():
                result.append(course)
        return result

    def register_course(self, student: Student, course: Course) -> bool:
        if course.get_enrolled_students() < course.get_max_capacity():
            registration = Registration(student, course, datetime.now())
            self.registrations.append(registration)
            student.get_registered_courses().append(course)
            course.set_enrolled_students(course.get_enrolled_students() + 1)
            self._notify_observers(course)
            return True
        return False

    def get_registered_courses(self, student: Student) -> List[Course]:
        return student.get_registered_courses()

    def _notify_observers(self, course: Course):
        # Notify observers (e.g., UI) about the updated course enrollment
        # ...
        pass
```

#### registration.py

```python
from student import Student
from course import Course
import datetime

class Registration:
    def __init__(self, student: Student, course: Course, registration_time: datetime):
        self.student = student
        self.course = course
        self.registration_time = registration_time
```

#### student.py

```python
from typing import List
from course import Course

class Student:
    def __init__(self, student_id: int, name: str, email: str, registered_courses: List[Course]):
        self.id = student_id
        self.name = name
        self.email = email
        self.registered_courses = registered_courses

    def get_id(self) -> int:
        return self.id

    def get_name(self) -> str:
        return self.name

    def get_email(self) -> str:
        return self.email

    def get_registered_courses(self) -> List[Course]:
        return self.registered_courses
```