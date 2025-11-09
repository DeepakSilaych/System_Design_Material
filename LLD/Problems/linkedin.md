# Designing a Professional Networking Platform like LinkedIn

## Question
Design a Professional Networking Platform like LinkedIn

## Requirements
#### User Registration and Authentication:
- Users should be able to create an account with their professional information, such as name, email, and password.
- Users should be able to log in and log out of their accounts securely.
#### User Profiles:
- Each user should have a profile with their professional information, such as profile picture, headline, summary, experience, education, and skills.
- Users should be able to update their profile information.
#### Connections:
- Users should be able to send connection requests to other users.
- Users should be able to accept or decline connection requests.
- Users should be able to view their list of connections.
#### Messaging:
- Users should be able to send messages to their connections.
- Users should be able to view their inbox and sent messages.
#### Job Postings:
- Employers should be able to post job listings with details such as title, description, requirements, and location.
- Users should be able to view and apply for job postings.
#### Search Functionality:
- Users should be able to search for other users, companies, and job postings based on relevant criteria.
- Search results should be ranked based on relevance and user preferences.
#### Notifications:
- Users should receive notifications for events such as connection requests, messages, and job postings.
- Notifications should be delivered in real-time.
#### Scalability and Performance:
- The system should be designed to handle a large number of concurrent users and high traffic load.
- The system should be scalable and efficient in terms of resource utilization.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Singleton collaborators are used where a single coordinating instance (for example managers or
processors) simplifies shared-state management across the system.

Key components include: User, Profile, Experience, Education, Skill, Connection, Message,
JobPosting, Notification, NotificationType, LinkedInService, LinkedInService, LinkedInDemo.

### Design Details
1. The **User** class represents a user in the LinkedIn system, containing properties such as ID, name, email, password, profile, connections, inbox, and sent messages.
2. The **Profile** class represents a user's profile, containing properties such as profile picture, headline, summary, experiences, educations, and skills.
3. The **Experience**, **Education**, and **Skill** classes represent different components of a user's profile.
4. The **Connection** class represents a connection between two users, containing the user and the connection date.
5. The **Message** class represents a message sent between users, containing properties such as ID, sender, receiver, content, and timestamp.
6. The **JobPosting** class represents a job listing posted by an employer, containing properties such as ID, title, description, requirements, location, and post date.
7. The **Notification** class represents a notification generated for a user, containing properties such as ID, user, notification type, content, and timestamp.
8. The **NotificationType** enum defines the different types of notifications, such as connection request, message, and job posting.
9. The **LinkedInService** class is the main class that manages the LinkedIn system. It follows the Singleton pattern to ensure only one instance of the service exists.
10. The **LinkedInService** class provides methods for user registration, login, profile updates, connection requests, job postings, user and job search, messaging, and notifications.
11. Multi-threading is achieved using concurrent data structures such as ConcurrentHashMap and CopyOnWriteArrayList to handle concurrent access to shared resources.
12. The **LinkedInDemo** class demonstrates the usage of the LinkedIn system by registering users, logging in, updating profiles, sending connection requests, posting job listings, searching for users and jobs, sending messages, and retrieving notifications.

### Implementation (Python)
#### connection.py

```python
from datetime import datetime
from typing import Optional
from member import Member
from enums import ConnectionStatus

class Connection:
    def __init__(self, from_member: Member, to_member: Member):
        self.from_member = from_member
        self.to_member = to_member
        self.status = ConnectionStatus.PENDING
        self.requested_at = datetime.now()
        self.accepted_at: Optional[datetime] = None

    def get_from_member(self) -> Member:
        return self.from_member

    def get_to_member(self) -> Member:
        return self.to_member

    def get_status(self) -> ConnectionStatus:
        return self.status

    def set_status(self, status: ConnectionStatus) -> None:
        self.status = status
        if status == ConnectionStatus.ACCEPTED:
            self.accepted_at = datetime.now()
```

#### connection_service.py

```python
from typing import Dict
from member import Member
from connection import Connection
from notification_service import NotificationService
from enums import ConnectionStatus, NotificationType
import uuid
import threading
from notification import Notification

class ConnectionService:
    def __init__(self, notification_service: NotificationService):
        self.notification_service = notification_service
        self.connection_requests: Dict[str, Connection] = {}
        self.lock = threading.Lock()

    def send_request(self, from_member: Member, to_member: Member) -> str:
        connection = Connection(from_member, to_member)
        request_id = str(uuid.uuid4())
        
        with self.lock:
            self.connection_requests[request_id] = connection

        print(f"{from_member.get_name()} sent a connection request to {to_member.get_name()}.")

        notification = Notification(
            to_member.get_id(),
            NotificationType.CONNECTION_REQUEST,
            f"{from_member.get_name()} wants to connect with you. Request ID: {request_id}"
        )
        self.notification_service.send_notification(to_member, notification)

        return request_id

    def accept_request(self, request_id: str) -> None:
        with self.lock:
            request = self.connection_requests.get(request_id)
            
            if request and request.get_status() == ConnectionStatus.PENDING:
                request.set_status(ConnectionStatus.ACCEPTED)

                from_member = request.get_from_member()
                to_member = request.get_to_member()

                from_member.add_connection(to_member)
                to_member.add_connection(from_member)

                print(f"{to_member.get_name()} accepted the connection request from {from_member.get_name()}.")
                del self.connection_requests[request_id]
            else:
                print("Invalid or already handled request ID.")
```

#### education.py

```python
class Education:
    def __init__(self, school: str, degree: str, start_year: int, end_year: int):
        self.school = school
        self.degree = degree
        self.start_year = start_year
        self.end_year = end_year

    def __str__(self) -> str:
        return f"{self.degree}, {self.school} ({self.start_year} - {self.end_year})"
```

#### enums.py

```python
from enum import Enum

class ConnectionStatus(Enum):
    PENDING = "PENDING"
    ACCEPTED = "ACCEPTED"
    REJECTED = "REJECTED"
    WITHDRAWN = "WITHDRAWN"

class NotificationType(Enum):
    CONNECTION_REQUEST = "CONNECTION_REQUEST"
    POST_LIKE = "POST_LIKE"
    POST_COMMENT = "POST_COMMENT"
```

#### experience.py

```python
from datetime import date
from typing import Optional

class Experience:
    def __init__(self, title: str, company: str, start_date: date, end_date: Optional[date]):
        self.title = title
        self.company = company
        self.start_date = start_date
        self.end_date = end_date

    def __str__(self) -> str:
        end_str = "Present" if self.end_date is None else str(self.end_date)
        return f"{self.title} at {self.company} ({self.start_date} to {end_str})"
```

#### feed_sorting_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from post import Post

class FeedSortingStrategy(ABC):
    @abstractmethod
    def sort(self, posts: List[Post]) -> List[Post]:
        pass

class ChronologicalSortStrategy(FeedSortingStrategy):
    def sort(self, posts: List[Post]) -> List[Post]:
        return sorted(posts, key=lambda post: post.get_created_at(), reverse=True)
```

#### linkedin_demo.py

```python
from linkedin_system import LinkedInSystem
from member import Member
from experience import Experience
from education import Education
from datetime import date

class LinkedInDemo:
    @staticmethod
    def main():
        system = LinkedInSystem.get_instance()

        # 1. Create Members using the Builder Pattern
        print("--- 1. Member Registration ---")
        alice = Member.Builder("Alice", "alice@example.com") \
            .with_summary("Senior Software Engineer with 10 years of experience.") \
            .add_experience(Experience("Sr. Software Engineer", "Google", date(2018, 1, 1), None)) \
            .add_experience(Experience("Software Engineer", "Microsoft", date(2014, 6, 1), date(2017, 12, 31))) \
            .add_education(Education("Princeton University", "M.S. in Computer Science", 2012, 2014)) \
            .build()

        bob = Member.Builder("Bob", "bob@example.com") \
            .with_summary("Product Manager at Stripe.") \
            .add_experience(Experience("Product Manager", "Stripe", date(2020, 2, 1), None)) \
            .add_education(Education("MIT", "B.S. in Business Analytics", 2015, 2019)) \
            .build()

        charlie = Member.Builder("Charlie", "charlie@example.com").build()

        system.register_member(alice)
        system.register_member(bob)
        system.register_member(charlie)

        alice.display_profile()

        # 2. Connection Management
        print("\n--- 2. Connection Management ---")
        # Alice sends requests to Bob and Charlie
        request_id1 = system.send_connection_request(alice, bob)
        request_id2 = system.send_connection_request(alice, charlie)

        bob.view_notifications()  # Bob sees Alice's request

        print("\nBob accepts Alice's request.")
        system.accept_connection_request(request_id1)
        print("Alice and Bob are now connected.")

        # 3. Posting and News Feed
        print("\n--- 3. Posting & News Feed ---")
        bob.display_profile()  # Bob has 1 connection
        system.create_post(bob.get_id(), "Excited to share we've launched our new feature! #productmanagement")

        # Alice views her news feed. She should see Bob's post.
        system.view_news_feed(alice.get_id())

        # Charlie views his feed. It should be empty as he is not connected to anyone.
        system.view_news_feed(charlie.get_id())

        # 4. Interacting with a Post (Observer Pattern in action)
        print("\n--- 4. Post Interaction & Notifications ---")
        bobs_post = system.get_latest_post_by_member(bob.get_id())
        if bobs_post:
            bobs_post.add_like(alice)
            bobs_post.add_comment(alice, "This looks amazing! Great work!")

        # Bob checks his notifications. He should see a like and a comment from Alice.
        bob.view_notifications()

        # 5. Searching for Members
        print("\n--- 5. Member Search ---")
        search_results = system.search_member_by_name("ali")
        print("Search results for 'ali':")
        for member in search_results:
            print(f" - {member.get_name()}")

if __name__ == "__main__":
    LinkedInDemo.main()
```

#### linkedin_system.py

```python
import threading
from typing import Dict, List, Optional
from member import Member
from connection_service import ConnectionService
from newsfeed_service import NewsFeedService
from search_service import SearchService
from notification_service import NotificationService
from feed_sorting_strategy import ChronologicalSortStrategy
from post import Post

class LinkedInSystem:
    _instance: Optional['LinkedInSystem'] = None
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

        self.members: Dict[str, Member] = {}
        self.connection_service = ConnectionService(NotificationService())
        self.news_feed_service = NewsFeedService()
        self.search_service = SearchService(self.members.values())
        self.initialized = True

    @classmethod
    def get_instance(cls) -> 'LinkedInSystem':
        return cls()

    def register_member(self, member: Member) -> None:
        self.members[member.get_id()] = member
        print(f"New member registered: {member.get_name()}")

    def get_member(self, name: str) -> Optional[Member]:
        for member in self.members.values():
            if member.get_name() == name:
                return member
        return None

    def send_connection_request(self, from_member: Member, to_member: Member) -> str:
        return self.connection_service.send_request(from_member, to_member)

    def accept_connection_request(self, request_id: str) -> None:
        self.connection_service.accept_request(request_id)

    def create_post(self, member_id: str, content: str) -> None:
        author = self.members[member_id]
        post = Post(author, content)
        self.news_feed_service.add_post(author, post)
        print(f"{author.get_name()} created a new post.")

    def get_latest_post_by_member(self, member_id: str) -> Optional[Post]:
        member_posts = self.news_feed_service.get_member_posts(self.members[member_id])
        if not member_posts:
            return None
        return member_posts[-1]

    def view_news_feed(self, member_id: str) -> None:
        member = self.members[member_id]
        print(f"\n--- News Feed for {member.get_name()} ---")
        self.news_feed_service.display_feed_for_member(member, ChronologicalSortStrategy())

    def search_member_by_name(self, name: str) -> List[Member]:
        return self.search_service.search_by_name(name)
```

#### member.py

```python
from typing import List, Set
from profile import Profile
from notification import Notification
from notification_observer import NotificationObserver
from experience import Experience
from education import Education
import uuid

class Member(NotificationObserver):
    def __init__(self, member_id: str, name: str, email: str, profile: Profile):
        self.id = member_id
        self.name = name
        self.email = email
        self.profile = profile
        self.connections: Set['Member'] = set()
        self.notifications: List['Notification'] = []

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name

    def get_email(self) -> str:
        return self.email

    def get_connections(self) -> Set['Member']:
        return self.connections

    def get_profile(self) -> Profile:
        return self.profile

    def add_connection(self, member: 'Member') -> None:
        self.connections.add(member)

    def display_profile(self) -> None:
        print(f"\n--- Profile for {self.name} ({self.email}) ---")
        self.profile.display()
        print(f"  Connections: {len(self.connections)}")

    def view_notifications(self) -> None:
        print(f"\n--- Notifications for {self.name} ---")
        unread_notifications = [n for n in self.notifications if not n.is_read()]
        
        if not unread_notifications:
            print("  No new notifications.")
            return

        for notification in unread_notifications:
            print(f"  - {notification.get_content()}")
            notification.mark_as_read()

    def update(self, notification: 'Notification') -> None:
        self.notifications.append(notification)
        print(f"Notification pushed to {self.name}: {notification.get_content()}")

    class Builder:
        def __init__(self, name: str, email: str):
            self.id = str(uuid.uuid4())
            self.name = name
            self.email = email
            self.profile = Profile()

        def with_summary(self, summary: str) -> 'Member.Builder':
            self.profile.set_summary(summary)
            return self

        def add_experience(self, experience: Experience) -> 'Member.Builder':
            self.profile.add_experience(experience)
            return self

        def add_education(self, education: Education) -> 'Member.Builder':
            self.profile.add_education(education)
            return self

        def build(self) -> 'Member':
            return Member(self.id, self.name, self.email, self.profile)
```

#### newsfeed.py

```python
from typing import List
from post import Post
from feed_sorting_strategy import FeedSortingStrategy

class NewsFeed:
    def __init__(self, posts: List[Post]):
        self.posts = posts

    def display(self, strategy: FeedSortingStrategy) -> None:
        sorted_posts = strategy.sort(self.posts)
        if not sorted_posts:
            print("  Your news feed is empty.")
            return

        for post in sorted_posts:
            print("----------------------------------------")
            print(f"Post by: {post.get_author().get_name()} (at {post.get_created_at().date()})")
            print(f"Content: {post.get_content()}")
            print(f"Likes: {len(post.get_likes())}, Comments: {len(post.get_comments())}")
            print("----------------------------------------")
```

#### newsfeed_service.py

```python
from typing import Dict, List
from member import Member
from post import Post
from feed_sorting_strategy import FeedSortingStrategy
from newsfeed import NewsFeed
from collections import defaultdict
import threading

class NewsFeedService:
    def __init__(self):
        self.all_posts: Dict[str, List[Post]] = defaultdict(list)
        self.lock = threading.Lock()

    def add_post(self, member: Member, post: Post) -> None:
        with self.lock:
            self.all_posts[member.get_id()].append(post)

    def get_member_posts(self, member: Member) -> List[Post]:
        return self.all_posts.get(member.get_id(), [])

    def display_feed_for_member(self, member: Member, feed_sorting_strategy: FeedSortingStrategy) -> None:
        feed_posts = []
        
        for connection in member.get_connections():
            connection_posts = self.all_posts.get(connection.get_id(), [])
            feed_posts.extend(connection_posts)

        news_feed = NewsFeed(feed_posts)
        news_feed.display(feed_sorting_strategy)
```

#### notification.py

```python
from enums import NotificationType
import uuid
from datetime import datetime

class Notification:
    def __init__(self, member_id: str, notification_type: NotificationType, content: str):
        self.id = str(uuid.uuid4())
        self.member_id = member_id
        self.type = notification_type
        self.content = content
        self.created_at = datetime.now()
        self._is_read = False

    def get_content(self) -> str:
        return self.content

    def mark_as_read(self) -> None:
        self._is_read = True

    def is_read(self) -> bool:
        return self._is_read
```

#### notification_observer.py

```python
from abc import ABC, abstractmethod
from typing import List
from notification import Notification

class NotificationObserver(ABC):
    @abstractmethod
    def update(self, notification: 'Notification') -> None:
        pass

class Subject:
    def __init__(self):
        self.observers: List[NotificationObserver] = []

    def add_observer(self, observer: NotificationObserver) -> None:
        self.observers.append(observer)

    def remove_observer(self, observer: NotificationObserver) -> None:
        if observer in self.observers:
            self.observers.remove(observer)

    def notify_observers(self, notification: 'Notification') -> None:
        for observer in self.observers:
            observer.update(notification)
```

#### notification_service.py

```python
from member import Member
from notification import Notification

class NotificationService:
    def send_notification(self, member: Member, notification: Notification) -> None:
        member.update(notification)
```

#### post.py

```python
from typing import List
from member import Member
from notification import Notification
from notification_observer import Subject
from enums import NotificationType
import uuid
from datetime import datetime

class Like:
    def __init__(self, member: Member):
        self.member = member
        self.created_at = datetime.now()

    def get_member(self) -> Member:
        return self.member

class Comment:
    def __init__(self, author: Member, text: str):
        self.author = author
        self.text = text
        self.created_at = datetime.now()

    def get_author(self) -> Member:
        return self.author

    def get_text(self) -> str:
        return self.text

class Post(Subject):
    def __init__(self, author: Member, content: str):
        super().__init__()
        self.id = str(uuid.uuid4())
        self.author = author
        self.content = content
        self.created_at = datetime.now()
        self.likes: List[Like] = []
        self.comments: List[Comment] = []
        self.add_observer(author)

    def add_like(self, member: Member) -> None:
        self.likes.append(Like(member))
        notification_content = f"{member.get_name()} liked your post."
        notification = Notification(self.author.get_id(), NotificationType.POST_LIKE, notification_content)
        self.notify_observers(notification)

    def add_comment(self, member: Member, text: str) -> None:
        self.comments.append(Comment(member, text))
        notification_content = f"{member.get_name()} commented on your post: \"{text}\""
        notification = Notification(self.author.get_id(), NotificationType.POST_COMMENT, notification_content)
        self.notify_observers(notification)

    def get_id(self) -> str:
        return self.id

    def get_author(self) -> Member:
        return self.author

    def get_content(self) -> str:
        return self.content

    def get_created_at(self) -> datetime:
        return self.created_at

    def get_likes(self) -> List[Like]:
        return self.likes

    def get_comments(self) -> List[Comment]:
        return self.comments
```

#### profile.py

```python
from typing import List, Optional
from experience import Experience
from education import Education

class Profile:
    def __init__(self):
        self.summary: Optional[str] = None
        self.experiences: List[Experience] = []
        self.educations: List[Education] = []

    def set_summary(self, summary: str) -> None:
        self.summary = summary

    def add_experience(self, experience: Experience) -> None:
        self.experiences.append(experience)

    def add_education(self, education: Education) -> None:
        self.educations.append(education)

    def display(self) -> None:
        print(f"  Summary: {self.summary if self.summary else 'N/A'}")

        print("  Experience:")
        if not self.experiences:
            print("    - None")
        else:
            for exp in self.experiences:
                print(f"    - {exp}")

        print("  Education:")
        if not self.educations:
            print("    - None")
        else:
            for edu in self.educations:
                print(f"    - {edu}")
```

#### search_service.py

```python
from typing import Collection, List
from member import Member

class SearchService:
    def __init__(self, members: Collection[Member]):
        self.members = members

    def search_by_name(self, name: str) -> List[Member]:
        return [member for member in self.members if name.lower() in member.get_name().lower()]
```