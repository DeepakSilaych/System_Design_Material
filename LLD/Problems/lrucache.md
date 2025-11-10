# Designing a LRU Cache

## Question
Design a LRU Cache

## Requirements
1. The LRU cache should support the following operations:
- put(key, value): Insert a key-value pair into the cache. If the cache is at capacity, remove the least recently used item before inserting the new item.
- get(key): Get the value associated with the given key. If the key exists in the cache, move it to the front of the cache (most recently used) and return its value. If the key does not exist, return -1.
2. The cache should have a fixed capacity, specified during initialization.
3. The cache should be thread-safe, allowing concurrent access from multiple threads.
4. The cache should be efficient in terms of time complexity for both put and get operations, ideally O(1).

## Solution
### Design overview
Use a hash map for O(1) lookup and a doubly linked list for O(1) recency updates/evictions. The
execution order is: read/write → adjust DLL nodes → evict tail when at capacity.

### Core model
- `Node` (key, value, prev, next)
- `DoublyLinkedList` with sentinel `head`/`tail` and operations: `add_first`, `remove`, `move_to_front`, `remove_last`
- `LRUCache` (capacity, map, dll, lock)

### Key flows
1. `get(k)`: lookup map → if hit, move node to head → return value; else return None
2. `put(k,v)`: if key exists → update and move to head; else evict tail when full, then insert new at head

### Design Details
1. The **Node** class represents a node in the doubly linked list, containing the key, value, and references to the previous and next nodes.
2. The **LRUCache** class implements the LRU cache functionality using a combination of a hash map (cache) and a doubly linked list (head and tail).
3. The get method retrieves the value associated with a given key. If the key exists in the cache, it is moved to the head of the linked list (most recently used) and its value is returned. If the key does not exist, null is returned.
4. The put method inserts a key-value pair into the cache. If the key already exists, its value is updated, and the node is moved to the head of the linked list. If the key does not exist and the cache is at capacity, the least recently used item (at the tail of the linked list) is removed, and the new item is inserted at the head.
5. The addToHead, removeNode, moveToHead, and removeTail methods are helper methods to manipulate the doubly linked list.
6. The synchronized keyword is used on the get and put methods to ensure thread safety, allowing concurrent access from multiple threads.
7. The **LRUCacheDemo** class demonstrates the usage of the LRU cache by creating an instance of LRUCache with a capacity of 3, performing various put and get operations, and printing the results.

### Implementation (Python)
#### dll.py

```python
from typing import TypeVar, Generic, Optional
from node import Node

K = TypeVar('K')
V = TypeVar('V')

class DoublyLinkedList(Generic[K, V]):
    def __init__(self):
        self.head: Node[K, V] = Node(None, None)  # Dummy head
        self.tail: Node[K, V] = Node(None, None)  # Dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head

    def add_first(self, node: Node[K, V]) -> None:
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def remove(self, node: Node[K, V]) -> None:
        node.prev.next = node.next
        node.next.prev = node.prev

    def move_to_front(self, node: Node[K, V]) -> None:
        self.remove(node)
        self.add_first(node)

    def remove_last(self) -> Optional[Node[K, V]]:
        if self.tail.prev == self.head:
            return None
        last = self.tail.prev
        self.remove(last)
        return last
```

#### lru_cache.py

```python
import threading
from typing import TypeVar, Generic, Optional, Dict
from dll import DoublyLinkedList
from node import Node

K = TypeVar('K')
V = TypeVar('V')

class LRUCache(Generic[K, V]):
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.map: Dict[K, Node[K, V]] = {}
        self.dll: DoublyLinkedList[K, V] = DoublyLinkedList()
        self.lock = threading.Lock()

    def get(self, key: K) -> Optional[V]:
        with self.lock:
            if key not in self.map:
                return None
            node = self.map[key]
            self.dll.move_to_front(node)
            return node.value

    def put(self, key: K, value: V) -> None:
        with self.lock:
            if key in self.map:
                node = self.map[key]
                node.value = value
                self.dll.move_to_front(node)
            else:
                if len(self.map) == self.capacity:
                    lru = self.dll.remove_last()
                    if lru is not None:
                        del self.map[lru.key]
                new_node = Node(key, value)
                self.dll.add_first(new_node)
                self.map[key] = new_node

    def remove(self, key: K) -> None:
        with self.lock:
            if key not in self.map:
                return
            node = self.map[key]
            self.dll.remove(node)
            del self.map[key]
```

#### lru_cache_demo.py

```python
from lru_cache import LRUCache

class LRUCacheDemo:
    @staticmethod
    def main():
        cache: LRUCache[str, int] = LRUCache(3)

        cache.put("a", 1)
        cache.put("b", 2)
        cache.put("c", 3)

        # Accessing 'a' makes it the most recently used
        print(cache.get("a"))  # 1

        # Adding 'd' will cause 'b' (the current LRU item) to be evicted
        cache.put("d", 4)

        # Trying to get 'b' should now return None
        print(cache.get("b"))  # None

if __name__ == "__main__":
    LRUCacheDemo.main()
```

#### node.py

```python
from typing import TypeVar, Generic, Optional

K = TypeVar('K')
V = TypeVar('V')

class Node(Generic[K, V]):
    def __init__(self, key: K, value: V):
        self.key = key
        self.value = value
        self.prev: Optional['Node[K, V]'] = None
        self.next: Optional['Node[K, V]'] = None
```