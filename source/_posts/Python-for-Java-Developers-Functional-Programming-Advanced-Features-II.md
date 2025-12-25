---
title: 'Python for Java Developers: Functional Programming & Advanced Features - II'
date: 2025-12-23 20:52:04
categories:
- Python
tags:
- Python
---

# Python for Java Developers: Functional Programming & Advanced Features (Part 2)

## 🎯 Part 2 Focus: Functional Programming & Modern Python Features

If Part 1 was about **syntax translation**, Part 2 is about **paradigm shift**. Java 8 introduced functional programming with Streams, Lambdas, and Optionals. Python has had these features (and more powerful ones) from the beginning!

## 🔄 1. Functional Programming Mindset Shift

### Java vs Python Functional Approach
```java
// Java: Functional programming added later (Java 8+)
// Requires explicit Stream API, often verbose
List<String> filtered = list.stream()
    .filter(s -> s.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

```python
# Python: Functional programming built-in from the start
# More concise, multiple approaches available
filtered = [s.upper() for s in list if s.startswith("A")]  # Comprehension
filtered = list(map(str.upper, filter(lambda s: s.startswith("A"), list)))  # Functional
```

## 🛠️ 2. Functions are First-Class Citizens

### Functions as Variables
```python
# Java: Functional interfaces, method references
# Function<String, Integer> func = String::length;

# Python: Functions are just objects
def greet(name: str) -> str:
    return f"Hello, {name}!"

# Assign function to variable
say_hello = greet
print(say_hello("Alice"))  # Hello, Alice!

# Store functions in collections
functions = [greet, str.upper, len]
for func in functions:
    print(func("test"))  # Calls each function

# Pass functions as arguments
def apply_twice(func, value):
    return func(func(value))

result = apply_twice(str.upper, "hello")  # "HELLO"
```

### Lambda Functions Revisited
```python
# Java: (x, y) -> x + y
# Python: lambda x, y: x + y

# Simple operations
add = lambda x, y: x + y
square = lambda x: x ** 2

# Immediately invoked lambda
result = (lambda x: x * 2)(5)  # 10

# Sort with custom key
students = ["Alice", "Bob", "Charlie"]
students.sort(key=lambda s: len(s))  # Sort by length
students.sort(key=lambda s: s[-1])   # Sort by last character

# Filter with lambda
numbers = [1, 2, 3, 4, 5]
evens = list(filter(lambda x: x % 2 == 0, numbers))
```

## 📦 3. Higher-Order Functions

### Map, Filter, Reduce
```python
from functools import reduce
from typing import List, Callable

# Map (like Java Stream.map())
numbers = [1, 2, 3, 4]
squared = list(map(lambda x: x ** 2, numbers))  # [1, 4, 9, 16]

# Using built-in functions
names = ["alice", "bob", "charlie"]
uppercased = list(map(str.upper, names))  # ['ALICE', 'BOB', 'CHARLIE']

# Filter (like Java Stream.filter())
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# Reduce (like Java Stream.reduce())
sum_all = reduce(lambda acc, x: acc + x, numbers, 0)  # 10
product = reduce(lambda acc, x: acc * x, numbers, 1)  # 24

# But often better with comprehensions...
squared = [x ** 2 for x in numbers]  # More readable!
evens = [x for x in numbers if x % 2 == 0]
```

### Custom Higher-Order Functions
```python
from typing import TypeVar, Callable, List

T = TypeVar('T')
R = TypeVar('R')

def compose(f: Callable[[T], R], g: Callable[[R], R]) -> Callable[[T], R]:
    """Function composition: f ∘ g = f(g(x))"""
    return lambda x: f(g(x))

def pipe(*functions: Callable) -> Callable:
    """Pipe functions left to right"""
    def piped(value):
        result = value
        for func in functions:
            result = func(result)
        return result
    return piped

def memoize(func: Callable) -> Callable:
    """Cache function results (like @Cacheable in Spring)"""
    cache = {}
    
    def memoized(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    return memoized

# Usage
add_one = lambda x: x + 1
double = lambda x: x * 2

composed = compose(add_one, double)  # add_one(double(x))
print(composed(5))  # 11 (5*2=10, 10+1=11)

piped = pipe(double, add_one)  # add_one(double(x))
print(piped(5))  # 11

# Memoization example
@memoize
def expensive_computation(n: int) -> int:
    print(f"Computing for {n}...")
    return sum(range(n))

print(expensive_computation(1000000))  # Computes
print(expensive_computation(1000000))  # Returns cached
```

## 🎭 4. Decorators (Python's Superpower!)

### What are Decorators?
Decorators are **functions that modify other functions**. They're like Java annotations but much more powerful because they can execute code at runtime.

### Basic Decorator
```python
# Java: @Override, @Deprecated, @Test
# Python: @decorator syntax

def debug_decorator(func):
    """Add debugging to any function"""
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@debug_decorator
def add(a: int, b: int) -> int:
    return a + b

print(add(3, 4))
# Output:
# Calling add with args=(3, 4), kwargs={}
# add returned 7
# 7
```

### Common Built-in Decorators
```python
from functools import wraps, lru_cache, total_ordering
import time
from typing import Callable

# @wraps - Preserves function metadata
def timer(func: Callable) -> Callable:
    """Measure execution time"""
    @wraps(func)  # Keeps original function name, docstring
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.3f} seconds")
        return result
    return wrapper

# @lru_cache - Built-in memoization (like @Cacheable)
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    """Compute nth Fibonacci number with caching"""
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast with caching!

# @total_ordering - Generate missing comparison methods
from functools import total_ordering

@total_ordering
class Student:
    def __init__(self, name: str, grade: float):
        self.name = name
        self.grade = grade
    
    def __eq__(self, other):
        return self.grade == other.grade
    
    def __lt__(self, other):
        return self.grade < other.grade
    # @total_ordering generates: >, <=, >= automatically!
```

### Class Decorators
```python
# Decorator that adds methods to a class
def add_logging(cls):
    """Add logging methods to any class"""
    def log(self, message: str):
        print(f"[{cls.__name__}] {message}")
    
    cls.log = log
    cls.info = lambda self, msg: self.log(f"INFO: {msg}")
    cls.error = lambda self, msg: self.log(f"ERROR: {msg}")
    
    return cls

@add_logging
class Database:
    def connect(self):
        self.info("Connecting to database...")
        # Connection logic
        self.info("Connected!")

db = Database()
db.info("Starting operations")  # [Database] INFO: Starting operations
```

### Parameterized Decorators
```python
# Decorators with arguments
def retry(max_attempts: int = 3, delay: float = 1.0):
    """Retry a function on failure"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def unreliable_api_call():
    """Simulate flaky API"""
    import random
    if random.random() < 0.7:
        raise ConnectionError("API failed")
    return "Success"
```

## 🔄 5. Iterators & Generators (Like Java Streams)

### Iterators Protocol
```python
# Java: Iterable<T>, Iterator<T>
# Python: __iter__(), __next__()

class Countdown:
    """Custom iterator"""
    def __init__(self, start: int):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

# Use like Java iterator
for num in Countdown(5):
    print(num)  # 5, 4, 3, 2, 1
```

### Generators (Lazy Evaluation)
```python
# Java: Stream.generate(), Stream.iterate()
# Python: yield keyword

def countdown_gen(start: int):
    """Generator function - returns iterator automatically"""
    current = start
    while current > 0:
        yield current  # Pauses here, resumes on next() call
        current -= 1

# Lazy evaluation - generates values on demand
for num in countdown_gen(5):
    print(num)  # Same as iterator, but simpler!

# Infinite generator
def fibonacci_gen():
    """Infinite Fibonacci sequence"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci_gen()
print(next(fib))  # 0
print(next(fib))  # 1
print(next(fib))  # 1
print(next(fib))  # 2

# Generator expression (like Stream.of()...)
squares = (x ** 2 for x in range(10))  # Lazy!
print(sum(squares))  # 285 - computes on demand
```

### Generator Pipelines (Like Java Stream Pipeline)
```python
# Java: stream().map().filter().collect()
# Python: Generator pipelines with lazy evaluation

def read_lines(filepath: str):
    """Generator: Read file line by line"""
    with open(filepath) as f:
        for line in f:
            yield line.strip()

def filter_comments(lines):
    """Filter out comment lines"""
    for line in lines:
        if not line.startswith("#"):
            yield line

def parse_numbers(lines):
    """Parse numbers from lines"""
    for line in lines:
        try:
            yield float(line)
        except ValueError:
            continue

# Pipeline: read → filter → parse → process
numbers = parse_numbers(filter_comments(read_lines("data.txt")))

# Memory efficient: Only processes what's needed
first_five = []
for i, num in enumerate(numbers):
    if i >= 5:
        break
    first_five.append(num)
```

## 🎨 6. Comprehensions Advanced Patterns

### Nested Comprehensions
```python
# Matrix flattening
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Transpose matrix
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]

# Dictionary comprehensions with conditions
scores = {"Alice": 85, "Bob": 92, "Charlie": 78, "David": 95}
passed = {name: score for name, score in scores.items() if score >= 80}

# Set comprehensions with transformation
words = ["hello", "world", "python", "java", "hello"]
unique_lengths = {len(word) for word in words}  # {5, 6, 4}
```

### Walrus Operator in Comprehensions (Python 3.8+)
```python
# Java: No equivalent, would need temp variable
# Python: := (assignment expression)

# Filter and transform with computed value
data = ["apple", "banana", "cherry", "date"]
result = [upper for fruit in data if (upper := fruit.upper()).startswith("A")]
# ['APPLE']

# Avoid duplicate computation
numbers = [1, 2, 3, 4, 5]
squared_evens = [square for n in numbers if (square := n ** 2) % 2 == 0]
# [4, 16]
```

## ⚡ 7. Async/Await (Like CompletableFuture)

### Basic Async Functions
```java
// Java: CompletableFuture<String> future = CompletableFuture.supplyAsync(...);
//        future.thenApply(...).thenAccept(...);

// Python: async/await (much cleaner!)
```

```python
import asyncio
from typing import Coroutine
import aiohttp  # Async HTTP client

async def fetch_url(url: str) -> str:
    """Async function (coroutine)"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    # Run multiple async operations concurrently
    urls = ["http://example.com", "http://example.org"]
    
    # Like Java's CompletableFuture.allOf()
    tasks = [fetch_url(url) for url in urls]
    results = await asyncio.gather(*tasks)  # Wait for all
    
    # Process results
    for url, content in zip(urls, results):
        print(f"{url}: {len(content)} bytes")

# Run event loop
asyncio.run(main())
```

### Async Generators
```python
async def stream_data(url: str):
    """Stream data as it arrives"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            async for chunk in response.content.iter_chunked(1024):
                yield chunk.decode()

async def process_stream():
    """Process streaming data"""
    async for data in stream_data("http://stream.example.com"):
        print(f"Received: {len(data)} bytes")
        # Process data incrementally
```

### Async Context Managers
```python
# Like Java's try-with-resources but async
import asyncpg  # Async PostgreSQL

class DatabaseConnection:
    async def __aenter__(self):
        self.conn = await asyncpg.connect("postgresql://...")
        return self.conn
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def query_database():
    async with DatabaseConnection() as conn:
        result = await conn.fetch("SELECT * FROM users")
        return result
```

## 📝 8. Type Hints & Static Analysis

### Advanced Type Hints
```python
from typing import (TypeVar, Generic, Protocol, runtime_checkable,
                    Callable, Union, Optional, Any, Literal,
                    TypedDict, NewType, overload)
from dataclasses import dataclass
from abc import ABC, abstractmethod

# Generic types (like Java Generics)
T = TypeVar('T')
U = TypeVar('U')

class Stack(Generic[T]):
    """Generic stack class"""
    def __init__(self):
        self._items: list[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        return self._items.pop()

# Protocol (like Java interface)
@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

def render(shape: Drawable) -> None:
    shape.draw()  # Duck typing with type safety

# TypedDict (for structured dictionaries)
class User(TypedDict):
    name: str
    age: int
    email: Optional[str]

user: User = {"name": "Alice", "age": 30}  # Type checked

# NewType (like value objects)
UserId = NewType('UserId', int)
user_id = UserId(123)  # Distinct type from int

# Literal types
def set_status(status: Literal["active", "inactive", "pending"]) -> None:
    ...

# Overload (multiple signatures)
@overload
def process(data: str) -> str: ...
@overload
def process(data: int) -> int: ...

def process(data: Union[str, int]) -> Union[str, int]:
    if isinstance(data, str):
        return data.upper()
    return data * 2
```

### Type Checking with Mypy
```bash
# Install mypy
pip install mypy

# Create mypy.ini
[mypy]
python_version = 3.12
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true

# Run mypy
mypy . --strict
```

## 🔧 9. Context Managers (Better than try-with-resources)

### Custom Context Managers
```python
# Java: try (AutoCloseable resource) { ... }
# Python: with resource: ...

from contextlib import contextmanager
import sqlite3

# Class-based context manager
class DatabaseConnection:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.connection = None
    
    def __enter__(self):
        self.connection = sqlite3.connect(self.db_path)
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            self.connection.close()
        if exc_type:
            print(f"Exception occurred: {exc_val}")
        return False  # Don't suppress exception

# Usage
with DatabaseConnection("test.db") as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")

# Function-based with @contextmanager
@contextmanager
def temporary_file(content: str):
    """Create temporary file, auto-cleanup"""
    import tempfile
    with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
        f.write(content)
        temp_path = f.name
    try:
        yield temp_path  # Give path to caller
    finally:
        import os
        os.unlink(temp_path)  # Cleanup

with temporary_file("Hello, World!") as filepath:
    with open(filepath) as f:
        print(f.read())  # File auto-deleted after
```

## 🧪 10. Testing with Pytest (Better than JUnit)

### Basic Tests
```python
# test_math.py
import pytest
from typing import List

# Simple test (no classes needed!)
def test_addition():
    assert 1 + 1 == 2

# Test with exceptions
def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        result = 1 / 0

# Parametrized tests (like @ParameterizedTest)
@pytest.mark.parametrize("input_a,input_b,expected", [
    (1, 2, 3),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(input_a: int, input_b: int, expected: int):
    assert input_a + input_b == expected

# Fixtures (like @BeforeEach)
@pytest.fixture
def sample_data() -> List[int]:
    """Setup test data"""
    return [1, 2, 3, 4, 5]

def test_sum(sample_data: List[int]):
    assert sum(sample_data) == 15

# Async tests
@pytest.mark.asyncio
async def test_async_function():
    result = await async_function()
    assert result == expected
```

### Mocking (Like Mockito)
```python
from unittest.mock import Mock, patch, MagicMock
import requests

def test_api_call():
    # Create mock
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"data": "test"}
    
    # Patch requests.get
    with patch('requests.get', return_value=mock_response) as mock_get:
        response = requests.get("http://api.example.com")
        
        # Verify calls
        mock_get.assert_called_once_with("http://api.example.com")
        assert response.status_code == 200
        assert response.json()["data"] == "test"

# Mock objects
def test_user_service():
    mock_db = Mock()
    mock_db.query.return_value = [{"name": "Alice"}]
    
    service = UserService(mock_db)
    users = service.get_users()
    
    assert len(users) == 1
    mock_db.query.assert_called_once_with("SELECT * FROM users")
```

## 🎯 Real-World Example: Functional Data Pipeline

Let's build a complete example showing Java vs Python approaches:

```java
// Java: Process user data with Streams
public class UserProcessor {
    public List<String> processUsers(List<User> users) {
        return users.stream()
            .filter(u -> u.getAge() >= 18)
            .map(u -> u.getName().toUpperCase())
            .sorted()
            .distinct()
            .limit(10)
            .collect(Collectors.toList());
    }
}
```

```python
# Python: Multiple approaches
from typing import List, Set
from dataclasses import dataclass
import itertools

@dataclass
class User:
    name: str
    age: int

# Approach 1: Functional style
def process_users_functional(users: List[User]) -> List[str]:
    return list(
        itertools.islice(
            sorted(
                set(
                    map(
                        str.upper,
                        filter(
                            lambda u: u.age >= 18,
                            users
                        )
                    )
                )
            ),
            10
        )
    )

# Approach 2: Generator pipeline (lazy, memory efficient)
def process_users_generator(users: List[User]) -> List[str]:
    def pipeline():
        # Each step is lazy
        adults = (u for u in users if u.age >= 18)
        names = (u.name.upper() for u in adults)
        unique_names = set(names)  # Forces evaluation for set
        sorted_names = sorted(unique_names)
        return sorted_names[:10]
    
    return list(pipeline())

# Approach 3: Comprehension (most readable)
def process_users_comprehension(users: List[User]) -> List[str]:
    unique_names = {u.name.upper() for u in users if u.age >= 18}
    return sorted(unique_names)[:10]

# Approach 4: Using more-itertools library
from more_itertools import unique_everseen, take

def process_users_itertools(users: List[User]) -> List[str]:
    processed = (
        u.name.upper()
        for u in users
        if u.age >= 18
    )
    unique = unique_everseen(processed)
    sorted_unique = sorted(unique)
    return list(take(10, sorted_unique))
```

## 📚 Cheat Sheet: Java Stream → Python Equivalent

| Java Stream | Python Equivalent | Notes |
|------------|------------------|-------|
| `.stream()` | `iter(collection)` | Create stream/iterator |
| `.map(f)` | `map(f, iterable)` or `(f(x) for x in iterable)` | |
| `.filter(p)` | `filter(p, iterable)` or `(x for x in iterable if p(x))` | |
| `.flatMap(f)` | `itertools.chain.from_iterable(map(f, iterable))` | |
| `.distinct()` | `set(iterable)` or `dict.fromkeys(iterable)` | |
| `.sorted()` | `sorted(iterable)` | |
| `.limit(n)` | `itertools.islice(iterable, n)` | |
| `.skip(n)` | `itertools.islice(iterable, n, None)` | |
| `.peek(f)` | Custom generator with side effects | |
| `.reduce(identity, accumulator)` | `functools.reduce(accumulator, iterable, identity)` | |
| `.collect(Collectors.toList())` | `list(iterable)` | |
| `.collect(Collectors.toSet())` | `set(iterable)` | |
| `.anyMatch(p)` | `any(p(x) for x in iterable)` | |
| `.allMatch(p)` | `all(p(x) for x in iterable)` | |
| `.noneMatch(p)` | `not any(p(x) for x in iterable)` | |
| `.findFirst()` | `next(iter(iterable), None)` | |
| `.count()` | `sum(1 for _ in iterable)` | |

## 🚀 Best Practices for Java Developers

### Do's and Don'ts

✅ **DO**: Use list/dict/set comprehensions for simple transformations
✅ **DO**: Use generators for large datasets (memory efficiency)
✅ **DO**: Use decorators for cross-cutting concerns (logging, caching, timing)
✅ **DO**: Embrace duck typing with Protocol for interfaces
✅ **DO**: Use dataclasses for simple data containers

❌ **DON'T**: Write Java-style getters/setters (use properties or public fields)
❌ **DON'T**: Create deep inheritance hierarchies (favor composition)
❌ **DON'T**: Use classes for everything (functions are fine!)
❌ **DON'T**: Ignore type hints in new code
❌ **DON'T**: Use `map/filter` when comprehensions are clearer

### Code Smells (Java-style Python)

```python
# ❌ Java-style Python (bad)
class UserService:
    def __init__(self):
        self._user_repository = None
    
    def get_user_repository(self):
        return self._user_repository
    
    def set_user_repository(self, repo):
        self._user_repository = repo
    
    def process_users(self, users):
        result = []
        for user in users:
            if user.age >= 18:
                result.append(user.name.upper())
        return sorted(result)[:10]

# ✅ Pythonic style (good)
@dataclass
class UserService:
    user_repository: UserRepository  # Public, no getters/setters
    
    def process_users(self, users: List[User]) -> List[str]:
        return sorted({
            user.name.upper()
            for user in users
            if user.age >= 18
        })[:10]
```

## 📦 Essential Libraries for Java Developers

```python
# Java -> Python library mapping
# Build tools: Maven/Gradle -> Poetry/pipenv
# Testing: JUnit/Mockito -> pytest/unittest.mock
# Web: Spring Boot -> FastAPI/Django
# ORM: Hibernate -> SQLAlchemy
# JSON: Jackson -> pydantic
# HTTP: RestTemplate -> requests/httpx
# Logging: Log4j -> logging
# Serialization: Jackson -> marshmallow
# Validation: Bean Validation -> pydantic
# Configuration: Spring Config -> python-decouple
# Caching: Spring Cache -> functools.lru_cache

# Install with pip
# pip install fastapi sqlalchemy pydantic pytest requests httpx
```

## 🎯 Next Steps

In **Chapater 3**, we'll cover:
- **Web Development**: FastAPI vs Spring Boot
- **Data Science**: NumPy, Pandas (like DataFrame APIs)
- **Concurrency**: Threading vs Multiprocessing
- **Packaging**: Building distributable packages
- **Performance**: Profiling and optimization

## Key Takeaways

1. **Python's functional features are more integrated** than Java's Stream API
2. **Decorators are incredibly powerful** for cross-cutting concerns
3. **Generators provide lazy evaluation** without the complexity of Streams
4. **Type hints + mypy** give you Java-like type safety when you need it
5. **Testing in Python is simpler** and more flexible than JUnit

Remember: **Python is about expressing ideas clearly and concisely**. Don't force Java patterns into Python. Embrace Python's idioms and you'll write cleaner, more maintainable code!