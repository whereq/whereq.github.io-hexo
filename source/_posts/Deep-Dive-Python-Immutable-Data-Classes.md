---
title: 'Deep Dive: Python Immutable Data Classes'
date: 2025-12-16 22:44:42
categories:
- Python
tags:
- Python
---

## 📋 Table of Contents
1. [Introduction & Fundamentals](#introduction--fundamentals)
2. [How It Works Internally](#how-it-works-internally)
3. [Practical Benefits & Use Cases](#practical-benefits--use-cases)
4. [The Shallow Freezing Caveat](#the-shallow-freezing-caveat)
5. [Working with Immutable Objects](#working-with-immutable-objects)
6. [Advanced Patterns & Alternatives](#advanced-patterns--alternatives)
7. [Performance Considerations](#performance-considerations)
8. [Best Practices](#best-practices)

## Introduction & Fundamentals

### What is `@dataclass(frozen=True)`?

The `@dataclass(frozen=True)` decorator in Python creates **immutable data classes** - classes primarily designed for storing data while enforcing **strict immutability** on their instances. Introduced in Python 3.7's `dataclasses` module, it combines automatic method generation with functional programming principles.

### Core Concept: Immutability vs. Mutability

```python
# Mutable class (standard dataclass)
@dataclass
class MutablePoint:
    x: float
    y: float

p1 = MutablePoint(1, 2)
p1.x = 3  # ✅ Modification allowed

# Immutable class (frozen dataclass)
@dataclass(frozen=True)
class ImmutablePoint:
    x: float
    y: float

p2 = ImmutablePoint(1, 2)
p2.x = 3  # ❌ Raises FrozenInstanceError
```

## How It Works Internally

### Automatic Method Generation

When you use `@dataclass(frozen=True)`, Python automatically generates these methods:

| Method | Purpose | Generated With `frozen=True` |
|--------|---------|-----------------------------|
| `__init__` | Constructor | ✅ Yes |
| `__repr__` | String representation | ✅ Yes |
| `__eq__` | Equality comparison | ✅ Yes |
| `__hash__` | Hashing support | ✅ Yes (crucial!) |
| `__setattr__` | Attribute assignment | ✅ Modified to raise error |
| `__delattr__` | Attribute deletion | ✅ Modified to raise error |
| `__lt__`, `__le__`, etc. | Comparisons | ✅ If `order=True` |

### The Magic of `__hash__`

The most important addition is `__hash__`:

```python
@dataclass(frozen=True)
class User:
    id: int
    name: str
    
# Objects are hashable and can be used in sets/dicts
users_set = {User(1, "Alice"), User(2, "Bob")}
users_dict = {User(1, "Alice"): "admin", User(2, "Bob"): "user"}
```

**Key Insight**: Without `frozen=True`, dataclasses are **unhashable by default** unless you explicitly set `unsafe_hash=True`. With `frozen=True`, hashability comes for free.

### Implementation Example

Here's what Python essentially creates under the hood:

```python
class ImmutablePoint:
    def __init__(self, x: float, y: float):
        object.__setattr__(self, 'x', x)
        object.__setattr__(self, 'y', y)
    
    def __setattr__(self, name, value):
        raise FrozenInstanceError(f"cannot assign to field '{name}'")
    
    def __delattr__(self, name):
        raise FrozenInstanceError(f"cannot delete field '{name}'")
    
    def __repr__(self):
        return f"ImmutablePoint(x={self.x}, y={self.y})"
    
    def __eq__(self, other):
        if not isinstance(other, ImmutablePoint):
            return NotImplemented
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):
        return hash((self.x, self.y))
```

## Practical Benefits & Use Cases

### 1. Thread Safety & Concurrency

```python
from threading import Thread
from dataclasses import dataclass
import time

@dataclass(frozen=True)
class SharedConfig:
    max_connections: int
    timeout: int
    api_key: str

config = SharedConfig(max_connections=100, timeout=30, api_key="secret")

# Multiple threads can safely read without locks
def worker(config: SharedConfig, thread_id: int):
    for _ in range(3):
        print(f"Thread {thread_id}: Max connections = {config.max_connections}")
        time.sleep(0.1)

threads = [Thread(target=worker, args=(config, i)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

### 2. Dictionary Keys & Set Elements

```python
@dataclass(frozen=True)
class Coordinate:
    x: int
    y: int
    z: int = 0

# Create a cache of computed values
cache = {}

def compute_value(coord: Coordinate) -> float:
    if coord in cache:  # ✅ Can use as dict key
        return cache[coord]
    
    # Expensive computation
    result = (coord.x ** 2 + coord.y ** 2 + coord.z ** 2) ** 0.5
    cache[coord] = result
    return result

# Create a set of unique coordinates
unique_points = {
    Coordinate(1, 2),
    Coordinate(1, 2),  # Duplicate - will be ignored
    Coordinate(3, 4),
    Coordinate(1, 2, 5),  # Different because z=5
}
print(f"Unique points: {len(unique_points)}")  # 3
```

### 3. Functional Programming Patterns

```python
from dataclasses import dataclass
from typing import List

@dataclass(frozen=True)
class ShoppingCart:
    items: tuple[str, ...]  # Use tuple for immutability
    discount: float = 0.0
    
    def add_item(self, item: str) -> 'ShoppingCart':
        """Return a NEW cart with the added item"""
        from dataclasses import replace
        return replace(self, items=self.items + (item,))
    
    def apply_discount(self, new_discount: float) -> 'ShoppingCart':
        """Return a NEW cart with updated discount"""
        from dataclasses import replace
        return replace(self, discount=new_discount)

# Original cart remains unchanged
cart1 = ShoppingCart(items=("apple", "banana"))
cart2 = cart1.add_item("orange")
cart3 = cart2.apply_discount(0.1)

print(f"Cart1: {cart1}")  # Items: ('apple', 'banana'), Discount: 0.0
print(f"Cart2: {cart2}")  # Items: ('apple', 'banana', 'orange'), Discount: 0.0
print(f"Cart3: {cart3}")  # Items: ('apple', 'banana', 'orange'), Discount: 0.1
```

### 4. Configuration Objects

```python
@dataclass(frozen=True)
class DatabaseConfig:
    host: str
    port: int
    database: str
    username: str
    password: str  # In reality, use a proper secrets manager
    
    @property
    def connection_string(self) -> str:
        return f"postgresql://{self.username}:{self.password}@{self.host}:{self.port}/{self.database}"

# Safe to pass around - no accidental modifications
config = DatabaseConfig(
    host="localhost",
    port=5432,
    database="mydb",
    username="admin",
    password="secret123"
)

# Can be safely stored in a module-level constant
DEFAULT_CONFIG = DatabaseConfig(
    host="prod-db.example.com",
    port=5432,
    database="production",
    username="readonly",
    password="readonly123"
)
```

## The Shallow Freezing Caveat

### Understanding the Limitation

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Company:
    name: str
    employees: list[str]  # 🚨 Mutable type!

# Create an instance
google = Company("Google", ["Alice", "Bob"])

# This is blocked (good):
# google.name = "Alphabet"  # ❌ FrozenInstanceError

# This still works (problematic):
google.employees.append("Charlie")  # ✅ No error!
print(google.employees)  # ['Alice', 'Bob', 'Charlie']

# The list itself is mutable, even though the field reference is frozen
```

### Solutions for Deep Immutability

**Option 1: Use Immutable Types**

```python
from typing import Tuple

@dataclass(frozen=True)
class ImmutableCompany:
    name: str
    employees: Tuple[str, ...]  # ✅ Tuple is immutable
    
company = ImmutableCompany("Google", ("Alice", "Bob"))
# company.employees.append("Charlie")  # ❌ AttributeError: 'tuple' object has no attribute 'append'
```

**Option 2: Convert to Immutable in `__post_init__`**

```python
from dataclasses import dataclass, field
from typing import List

@dataclass(frozen=True)
class SafeCompany:
    name: str
    employees: List[str] = field(default_factory=list)
    
    def __post_init__(self):
        # Convert list to tuple after initialization
        object.__setattr__(self, 'employees', tuple(self.employees))

company = SafeCompany("Google", ["Alice", "Bob"])
print(company.employees)  # ('Alice', 'Bob')
# company.employees.append("Charlie")  # ❌ AttributeError
```

**Option 3: Use `functools.cached_property` for Computed Mutable Data**

```python
from dataclasses import dataclass
from functools import cached_property
from typing import Dict

@dataclass(frozen=True)
class Analytics:
    user_ids: Tuple[int, ...]
    
    @cached_property
    def user_id_to_index(self) -> Dict[int, int]:
        """Compute once, cache forever"""
        return {user_id: idx for idx, user_id in enumerate(self.user_ids)}

analytics = Analytics((101, 102, 103))
print(analytics.user_id_to_index)  # {101: 0, 102: 1, 103: 2}
```

## Working with Immutable Objects

### The `replace()` Function Pattern

```python
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class UserProfile:
    username: str
    email: str
    is_active: bool = True
    permissions: Tuple[str, ...] = ()
    
    def deactivate(self) -> 'UserProfile':
        """Functional style: return new object with changes"""
        return replace(self, is_active=False)
    
    def add_permission(self, permission: str) -> 'UserProfile':
        return replace(
            self, 
            permissions=self.permissions + (permission,)
        )

# Chain updates functionally
user = UserProfile("alice", "alice@example.com")
user = user.add_permission("read").add_permission("write")
user = user.deactivate()

print(user)  # UserProfile(username='alice', email='alice@example.com', is_active=False, permissions=('read', 'write'))
```

### Bulk Updates with Dictionary Unpacking

```python
@dataclass(frozen=True)
class Settings:
    theme: str
    language: str
    notifications: bool
    timeout: int

settings = Settings("dark", "en", True, 30)

# Update multiple fields at once
new_settings = replace(
    settings,
    **{"theme": "light", "notifications": False}  # Can use dict unpacking
)

# Or with variable updates
updates = {}
if user_prefers_light_mode:
    updates["theme"] = "light"
if user_is_busy:
    updates["notifications"] = False
    
updated_settings = replace(settings, **updates)
```

### Versioned Data with Inheritance

```python
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True)
class UserV1:
    id: int
    name: str

@dataclass(frozen=True)
class UserV2(UserV1):
    email: Optional[str] = None
    phone: Optional[str] = None

# Backward compatible
v1_user = UserV1(1, "Alice")
v2_user = UserV2(1, "Alice", email="alice@example.com")

# Type checking works
def process_user(user: UserV1):
    print(f"Processing: {user.name}")

process_user(v1_user)  # ✅
process_user(v2_user)  # ✅ (inheritance)
```

## Advanced Patterns & Alternatives

### Pattern 1: Builder Pattern for Complex Objects

```python
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass(frozen=True)
class Query:
    table: str
    columns: Tuple[str, ...]
    filters: Tuple['Filter', ...]
    limit: Optional[int] = None
    
    @dataclass
    class Builder:
        """Mutable builder for immutable Query"""
        table: str = ""
        columns: List[str] = field(default_factory=list)
        filters: List['Filter'] = field(default_factory=list)
        limit: Optional[int] = None
        
        def build(self) -> 'Query':
            return Query(
                table=self.table,
                columns=tuple(self.columns),
                filters=tuple(self.filters),
                limit=self.limit
            )
    
    @dataclass(frozen=True)
    class Filter:
        column: str
        operator: str
        value: any

# Usage
builder = Query.Builder()
builder.table = "users"
builder.columns = ["id", "name", "email"]
builder.filters.append(Query.Filter("age", ">", 18))
builder.limit = 100

query = builder.build()  # ✅ Immutable Query object
```

### Pattern 2: Registry Pattern with Frozen Classes

```python
from dataclasses import dataclass
from typing import Dict, ClassVar

@dataclass(frozen=True)
class ErrorCode:
    code: int
    message: str
    
    # Registry of all error codes
    _registry: ClassVar[Dict[int, 'ErrorCode']] = {}
    
    def __post_init__(self):
        # Register the instance
        ErrorCode._registry[self.code] = self
    
    @classmethod
    def get(cls, code: int) -> 'ErrorCode':
        return cls._registry[code]

# Define error codes (these become singletons)
NOT_FOUND = ErrorCode(404, "Resource not found")
UNAUTHORIZED = ErrorCode(401, "Authentication required")
FORBIDDEN = ErrorCode(403, "Insufficient permissions")

# Safe to use anywhere
print(ErrorCode.get(404))  # ErrorCode(code=404, message='Resource not found')
```

### Alternative 1: `collections.namedtuple`

```python
from collections import namedtuple
from typing import NamedTuple

# Basic immutable structure
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
# p.x = 3  # ❌ AttributeError: can't set attribute

# With type hints (Python 3.6+)
class PointNT(NamedTuple):
    x: float
    y: float
    z: float = 0.0

# Limitations: Less flexible than dataclasses, no easy field defaults
```

### Alternative 2: `attrs` Library

```python
import attr

@attr.s(frozen=True, slots=True)
class AttrsPoint:
    x: float = attr.ib()
    y: float = attr.ib()
    color: str = attr.ib(default="blue")

# Additional features like validators, converters
@attr.s(frozen=True)
class ValidatedPoint:
    x: float = attr.ib(validator=attr.validators.instance_of(float))
    y: float = attr.ib(validator=attr.validators.instance_of(float))
```

## Performance Considerations

### Memory and Speed Implications

```python
import timeit
from dataclasses import dataclass, make_dataclass
from pympler import asizeof

@dataclass(frozen=True)
class FrozenUser:
    id: int
    name: str
    email: str

@dataclass
class MutableUser:
    id: int
    name: str
    email: str

# Memory usage
frozen = FrozenUser(1, "Alice", "alice@example.com")
mutable = MutableUser(1, "Alice", "alice@example.com")

print(f"Frozen size: {asizeof.asizeof(frozen)} bytes")
print(f"Mutable size: {asizeof.asizeof(mutable)} bytes")
# Typically very similar - frozen might be slightly larger due to __hash__

# Creation speed
setup = "from __main__ import FrozenUser, MutableUser"
stmt_frozen = "FrozenUser(1, 'Alice', 'alice@example.com')"
stmt_mutable = "MutableUser(1, 'Alice', 'alice@example.com')"

time_frozen = timeit.timeit(stmt_frozen, setup=setup, number=1000000)
time_mutable = timeit.timeit(stmt_mutable, setup=setup, number=1000000)

print(f"Frozen creation: {time_frozen:.3f}s")
print(f"Mutable creation: {time_mutable:.3f}s")
# Frozen is slightly slower due to additional checks
```

### When Performance Matters

```python
# For high-performance scenarios, consider __slots__
@dataclass(frozen=True)
class OptimizedPoint:
    __slots__ = ('x', 'y', 'z')  # Reduces memory overhead
    x: float
    y: float
    z: float = 0.0

# Or use tuple directly for maximum performance
from typing import NamedTuple

class PointNT(NamedTuple):
    x: float
    y: float
    z: float = 0.0

# Benchmark: NamedTuple is fastest for creation and memory
```

## Best Practices

### 1. **Default to Frozen**

```python
# Good: Start with frozen, relax only if needed
@dataclass(frozen=True)
class Config:
    api_key: str
    timeout: int = 30
    retries: int = 3

# Only make mutable if you have a proven need
@dataclass  # No frozen - use sparingly!
class Session:
    user_id: int
    token: str
    expiry: float
```

### 2. **Use Immutable Field Types**

```python
from typing import Tuple, FrozenSet

@dataclass(frozen=True)
class ImmutableData:
    # Good: Immutable types
    ids: Tuple[int, ...]
    tags: FrozenSet[str]
    name: str
    count: int
    
    # Avoid (unless converted in __post_init__):
    # items: List[str]  # ❌
    # config: Dict[str, Any]  # ❌
```

### 3. **Implement Domain Logic as Methods**

```python
@dataclass(frozen=True)
class Money:
    amount: float
    currency: str
    
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currencies must match")
        return replace(self, amount=self.amount + other.amount)
    
    def convert(self, rate: float, target_currency: str) -> 'Money':
        return Money(amount=self.amount * rate, currency=target_currency)

usd = Money(100, "USD")
eur = usd.convert(0.85, "EUR")
total = usd.add(Money(50, "USD"))
```

### 4. **Combine with `kw_only` for Clarity**

```python
@dataclass(frozen=True, kw_only=True)
class ServerConfig:
    host: str
    port: int
    ssl: bool = True
    timeout: int = 30
    
# Forces explicit parameter names
config = ServerConfig(host="example.com", port=443)
# Not: ServerConfig("example.com", 443)  # ❌
```

### 5. **Document Frozen Assumptions**

```python
@dataclass(frozen=True)
class APIResponse:
    """Immutable response from API.
    
    This class is frozen to ensure:
    1. Thread safety when sharing between components
    2. Hashability for caching
    3. Predictable behavior in functional pipelines
    
    Use `dataclasses.replace()` to create modified copies.
    """
    status: int
    data: dict
    headers: Tuple[Tuple[str, str], ...]
```

## Common Pitfalls and Solutions

### Pitfall 1: Inadvertent Mutation of Nested Objects

```python
# ❌ Problem
@dataclass(frozen=True)
class Graph:
    nodes: dict

# ✅ Solution
@dataclass(frozen=True)
class Graph:
    nodes: frozendict  # Requires 'frozendict' package
    # OR
    nodes: Mapping[str, Any]  # Read-only interface

# ✅ Better Solution
@dataclass(frozen=True)
class Graph:
    _nodes: dict = field(default_factory=dict, repr=False)
    
    @property
    def nodes(self) -> Mapping[str, Any]:
        return MappingProxyType(self._nodes)  # Read-only view
```

### Pitfall 2: Circular References

```python
from typing import Optional

@dataclass(frozen=True)
class TreeNode:
    value: any
    left: Optional['TreeNode'] = None
    right: Optional['TreeNode'] = None
    
    # This works! Frozen doesn't prevent object graph formation
    # It only prevents reassignment of the fields themselves
```

### Pitfall 3: Serialization Issues

```python
import json
from dataclasses import dataclass, asdict

@dataclass(frozen=True)
class Data:
    x: int
    y: int

d = Data(1, 2)

# Standard serialization works
json_str = json.dumps(asdict(d))

# For custom serialization, implement __json__ or use a mixin
import json
from abc import ABC

class JSONSerializable(ABC):
    def to_json(self) -> str:
        return json.dumps(asdict(self))

@dataclass(frozen=True)
class JsonData(JSONSerializable):
    x: int
    y: int
```

## Conclusion

`@dataclass(frozen=True)` is a powerful tool that brings functional programming benefits to Python's object-oriented world. It provides:

1. **Safety** through immutability
2. **Convenience** through automatic method generation
3. **Expressiveness** through clean, declarative syntax
4. **Performance** through hashability and thread safety

While the "shallow freezing" limitation requires attention, following the patterns and best practices outlined above will help you create robust, maintainable, and bug-resistant data models in Python.

**When to use it**: Configuration objects, value objects, messages/events, cache keys, shared state in concurrent systems, and anywhere you want to enforce the principle of "no surprises."

**When to avoid it**: When you genuinely need mutable state, when working with large object graphs that change frequently, or when the performance overhead of copying is prohibitive.