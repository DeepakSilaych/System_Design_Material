# Designing Stack Overflow

## Question
Design Stack Overflow

## Requirements
1. Users can post questions, answer questions, and comment on questions and answers.
2. Users can vote on questions and answers.
3. Questions should have tags associated with them.
4. Users can search for questions based on keywords, tags, or user profiles.
5. The system should assign reputation score to users based on their activity and the quality of their contributions.
6. The system should handle concurrent access and ensure data consistency.

## Solution
### Approach and Principles
The reference design models the domain with cohesive classes that each own a clear responsibility.
Interactions between components are mediated through well-defined interfaces, aligning with SOLID
principles.

Key components include: User, Question, Answer, Comment, Tag, Vote, StackOverflow,
StackOverflowDemo.

### Design Details
1. The **User** class represents a user of the Stack Overflow system, with properties such as id, username, email, and reputation.
2. The **Question** class represents a question posted by a user, with properties such as id, title, content, author, answers, comments, tags, votes and creation date.
3. The **Answer** class represents an answer to a question, with properties such as id, content, author, associated question, comments, votes and creation date.
4. The **Comment** class represents a comment on a question or an answer, with properties such as id, content, author, and creation date.
5. The **Tag** class represents a tag associated with a question, with properties such as id and name.
6. The **Vote** class represents vote associated with a question/answer.
7. The **StackOverflow** class is the main class that manages the Stack Overflow system. It provides methods for creating user, posting questions, answers, and comments, voting on questions and answers, searching for questions, and retrieving questions by tags and users.
8.  The **StackOverflowDemo** class demonstrates the usage of the Stack Overflow system by creating users, posting questions and answers, voting, searching for questions, and retrieving questions by tags and users.

### Implementation (Python)
#### content.py

```python
import threading
import uuid
from abc import ABC
from datetime import datetime
from typing import Dict, List, Optional, Set, TYPE_CHECKING

from enums import VoteType, EventType
from user import User
from tag import Tag

if TYPE_CHECKING:
    from event import Event
    from post_observer import PostObserver


class Content(ABC):
    def __init__(self, content_id: str, body: str, author: User):
        self.id = content_id
        self.body = body
        self.author = author
        self.creation_time = datetime.now()

    def get_id(self) -> str:
        return self.id

    def get_body(self) -> str:
        return self.body

    def get_author(self) -> User:
        return self.author


class Post(Content):
    def __init__(self, post_id: str, body: str, author: User):
        super().__init__(post_id, body, author)
        self.vote_count = 0
        self.voters: Dict[str, VoteType] = {}
        self.comments: List['Comment'] = []
        self.observers: List['PostObserver'] = []
        self._lock = threading.Lock()

    def add_observer(self, observer: 'PostObserver'):
        self.observers.append(observer)

    def notify_observers(self, event: 'Event'):
        for observer in self.observers:
            observer.on_post_event(event)

    def vote(self, user: User, vote_type: VoteType):
        with self._lock:
            user_id = user.get_id()
            if self.voters.get(user_id) == vote_type:
                return  # Already voted

            score_change = 0
            if user_id in self.voters:  # User is changing their vote
                score_change = 2 if vote_type == VoteType.UPVOTE else -2
            else:  # New vote
                score_change = 1 if vote_type == VoteType.UPVOTE else -1

            self.voters[user_id] = vote_type
            self.vote_count += score_change

            # Import here to avoid circular dependency
            from event import Event
            
            if isinstance(self, Question):
                event_type = EventType.UPVOTE_QUESTION if vote_type == VoteType.UPVOTE else EventType.DOWNVOTE_QUESTION
            else:
                event_type = EventType.UPVOTE_ANSWER if vote_type == VoteType.UPVOTE else EventType.DOWNVOTE_ANSWER

            self.notify_observers(Event(event_type, user, self))

    def get_vote_count(self) -> int:
        return self.vote_count

    def add_comment(self, comment: 'Comment'):
        self.comments.append(comment)

    def get_comments(self) -> List['Comment']:
        return self.comments


class Question(Post):
    def __init__(self, title: str, body: str, author: User, tags: Set[Tag]):
        super().__init__(str(uuid.uuid4()), body, author)
        self.title = title
        self.tags = tags
        self.answers: List['Answer'] = []
        self.accepted_answer: Optional['Answer'] = None

    def add_answer(self, answer: 'Answer'):
        self.answers.append(answer)

    def accept_answer(self, answer: 'Answer'):
        with self._lock:
            # Only the question author can accept an answer, and it shouldn't be their own answer
            if (self.author.get_id() != answer.get_author().get_id() and 
                self.accepted_answer is None):
                self.accepted_answer = answer
                answer.set_accepted(True)
                
                # Import here to avoid circular dependency
                from event import Event
                self.notify_observers(Event(EventType.ACCEPT_ANSWER, answer.get_author(), answer))

    def get_title(self) -> str:
        return self.title

    def get_tags(self) -> Set[Tag]:
        return self.tags

    def get_answers(self) -> List['Answer']:
        return self.answers

    def get_accepted_answer(self) -> Optional['Answer']:
        return self.accepted_answer


class Answer(Post):
    def __init__(self, body: str, author: User):
        super().__init__(str(uuid.uuid4()), body, author)
        self.is_accepted = False

    def set_accepted(self, accepted: bool):
        self.is_accepted = accepted

    def is_accepted_answer(self) -> bool:
        return self.is_accepted


class Comment(Content):
    def __init__(self, body: str, author: User):
        super().__init__(str(uuid.uuid4()), body, author)
```

#### enums.py

```python
from enum import Enum

class VoteType(Enum):
    UPVOTE = "UPVOTE"
    DOWNVOTE = "DOWNVOTE"

class EventType(Enum):
    UPVOTE_QUESTION = "UPVOTE_QUESTION"
    DOWNVOTE_QUESTION = "DOWNVOTE_QUESTION"
    UPVOTE_ANSWER = "UPVOTE_ANSWER"
    DOWNVOTE_ANSWER = "DOWNVOTE_ANSWER"
    ACCEPT_ANSWER = "ACCEPT_ANSWER"
```

#### event.py

```python
from enums import EventType
from user import User
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from post import Post

class Event:
    def __init__(self, event_type: EventType, actor: User, target_post: 'Post'):
        self.type = event_type
        self.actor = actor
        self.target_post = target_post

    def get_type(self) -> EventType:
        return self.type

    def get_actor(self) -> User:
        return self.actor

    def get_target_post(self) -> 'Post':
        return self.target_post
```

#### post_observer.py

```python
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from event import Event

class PostObserver(ABC):
    @abstractmethod
    def on_post_event(self, event: 'Event'):
        pass
```

#### reputation_manager.py

```python
from post_observer import PostObserver
from event import Event
from enums import EventType

class ReputationManager(PostObserver):
    QUESTION_UPVOTE_REP = 5
    ANSWER_UPVOTE_REP = 10
    ACCEPTED_ANSWER_REP = 15
    DOWNVOTE_REP_PENALTY = -1  # Penalty for the voter
    POST_DOWNVOTED_REP_PENALTY = -2  # Penalty for the post author

    def on_post_event(self, event: Event):
        post_author = event.get_target_post().get_author()
        
        if event.get_type() == EventType.UPVOTE_QUESTION:
            post_author.update_reputation(self.QUESTION_UPVOTE_REP)
        elif event.get_type() == EventType.DOWNVOTE_QUESTION:
            post_author.update_reputation(self.DOWNVOTE_REP_PENALTY)
            event.get_actor().update_reputation(self.POST_DOWNVOTED_REP_PENALTY)  # voter penalty
        elif event.get_type() == EventType.UPVOTE_ANSWER:
            post_author.update_reputation(self.ANSWER_UPVOTE_REP)
        elif event.get_type() == EventType.DOWNVOTE_ANSWER:
            post_author.update_reputation(self.DOWNVOTE_REP_PENALTY)
            event.get_actor().update_reputation(self.POST_DOWNVOTED_REP_PENALTY)
        elif event.get_type() == EventType.ACCEPT_ANSWER:
            post_author.update_reputation(self.ACCEPTED_ANSWER_REP)
```

#### search_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from tag import Tag
from user import User
from content import Question

class SearchStrategy(ABC):
    @abstractmethod
    def filter(self, questions: List[Question]) -> List[Question]:
        pass

class KeywordSearchStrategy(SearchStrategy):
    def __init__(self, keyword: str):
        self.keyword = keyword.lower()

    def filter(self, questions: List[Question]) -> List[Question]:
        return [q for q in questions 
                if self.keyword in q.get_title().lower() or self.keyword in q.get_body().lower()]

class TagSearchStrategy(SearchStrategy):
    def __init__(self, tag: Tag):
        self.tag = tag

    def filter(self, questions: List[Question]) -> List[Question]:
        return [q for q in questions 
                if any(t.get_name().lower() == self.tag.get_name().lower() for t in q.get_tags())]

class UserSearchStrategy(SearchStrategy):
    def __init__(self, user: User):
        self.user = user

    def filter(self, questions: List[Question]) -> List[Question]:
        return [q for q in questions if q.get_author().get_id() == self.user.get_id()]
```

#### stack_overflow_demo.py

```python
from stack_overflow_service import StackOverflowService
from enums import VoteType
from search_strategy import UserSearchStrategy, TagSearchStrategy
from tag import Tag

class StackOverflowDemo:
    @staticmethod
    def main():
        service = StackOverflowService()

        # 1. Create Users
        alice = service.create_user("Alice")
        bob = service.create_user("Bob")
        charlie = service.create_user("Charlie")

        # 2. Alice posts a question
        print("--- Alice posts a question ---")
        java_tag = Tag("java")
        design_patterns_tag = Tag("design-patterns")
        tags = {java_tag, design_patterns_tag}
        question = service.post_question(alice.get_id(), "How to implement Observer Pattern?", "Details about Observer Pattern...", tags)
        StackOverflowDemo.print_reputations(alice, bob, charlie)

        # 3. Bob and Charlie post answers
        print("\n--- Bob and Charlie post answers ---")
        bob_answer = service.post_answer(bob.get_id(), question.get_id(), "You can use the java.util.Observer interface.")
        charlie_answer = service.post_answer(charlie.get_id(), question.get_id(), "A better way is to create your own Observer interface.")
        StackOverflowDemo.print_reputations(alice, bob, charlie)

        # 4. Voting happens
        print("\n--- Voting Occurs ---")
        service.vote_on_post(alice.get_id(), question.get_id(), VoteType.UPVOTE)  # Alice upvotes her own question
        service.vote_on_post(bob.get_id(), charlie_answer.get_id(), VoteType.UPVOTE)  # Bob upvotes Charlie's answer
        service.vote_on_post(alice.get_id(), bob_answer.get_id(), VoteType.DOWNVOTE)  # Alice downvotes Bob's answer
        StackOverflowDemo.print_reputations(alice, bob, charlie)

        # 5. Alice accepts Charlie's answer
        print("\n--- Alice accepts Charlie's answer ---")
        service.accept_answer(question.get_id(), charlie_answer.get_id())
        StackOverflowDemo.print_reputations(alice, bob, charlie)

        # 6. Search for questions
        print("\n--- (C) Combined Search: Questions by 'Alice' with tag 'java' ---")
        filters_c = [
            UserSearchStrategy(alice),
            TagSearchStrategy(java_tag)
        ]
        search_results = service.search_questions(filters_c)
        for q in search_results:
            print(f"  - Found: {q.get_title()}")

    @staticmethod
    def print_reputations(*users):
        print("--- Current Reputations ---")
        for user in users:
            print(f"{user.get_name()}: {user.get_reputation()}")


if __name__ == "__main__":
    StackOverflowDemo.main()
```

#### stack_overflow_service.py

```python
from typing import Dict, Set, List
from user import User
from content import Question
from content import Answer
from tag import Tag
from enums import VoteType
from search_strategy import SearchStrategy
from reputation_manager import ReputationManager
from content import Post

class StackOverflowService:
    def __init__(self):
        self.users: Dict[str, User] = {}
        self.questions: Dict[str, Question] = {}
        self.answers: Dict[str, Answer] = {}
        self.reputation_manager = ReputationManager()

    def create_user(self, name: str) -> User:
        user = User(name)
        self.users[user.get_id()] = user
        return user

    def post_question(self, user_id: str, title: str, body: str, tags: Set[Tag]) -> Question:
        author = self.users[user_id]
        question = Question(title, body, author, tags)
        question.add_observer(self.reputation_manager)
        self.questions[question.get_id()] = question
        return question

    def post_answer(self, user_id: str, question_id: str, body: str) -> Answer:
        author = self.users[user_id]
        question = self.questions[question_id]
        answer = Answer(body, author)
        answer.add_observer(self.reputation_manager)
        question.add_answer(answer)
        self.answers[answer.get_id()] = answer
        return answer

    def vote_on_post(self, user_id: str, post_id: str, vote_type: VoteType):
        user = self.users[user_id]
        post = self.find_post_by_id(post_id)
        post.vote(user, vote_type)

    def accept_answer(self, question_id: str, answer_id: str):
        question = self.questions[question_id]
        answer = self.answers[answer_id]
        question.accept_answer(answer)

    def search_questions(self, strategies: List[SearchStrategy]) -> List[Question]:
        results = list(self.questions.values())

        for strategy in strategies:
            results = strategy.filter(results)

        return results

    def get_user(self, user_id: str) -> User:
        return self.users[user_id]

    def find_post_by_id(self, post_id: str) -> Post:
        if post_id in self.questions:
            return self.questions[post_id]
        elif post_id in self.answers:
            return self.answers[post_id]
        
        raise KeyError("Post not found")
```

#### tag.py

```python
class Tag:
    def __init__(self, name: str):
        self.name = name

    def get_name(self) -> str:
        return self.name
```

#### user.py

```python
import threading
import uuid

class User:
    def __init__(self, name: str):
        self.id = str(uuid.uuid4())
        self.name = name
        self.reputation = 0
        self._lock = threading.Lock()

    def update_reputation(self, change: int):
        with self._lock:
            self.reputation += change

    def get_id(self) -> str:
        return self.id

    def get_name(self) -> str:
        return self.name

    def get_reputation(self) -> int:
        with self._lock:
            return self.reputation
```