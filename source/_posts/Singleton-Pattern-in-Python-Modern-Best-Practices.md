---
title: 'Singleton Pattern in Python: Modern Best Practices'
date: 2025-12-23 20:31:46
categories:
- Python
tags:
- Python
---

## What is a Singleton?

A **singleton** is a class that allows **only one instance** to be created throughout the entire application. It provides a global point of access to that instance.

## Real-Life Use Cases for Singletons

### 1. **Configuration Management**
```python
# Single source of truth for application configuration
config = {
    'database_url': 'postgresql://localhost:5432/mydb',
    'api_key': 'secret_key',
    'log_level': 'INFO'
}
# You don't want multiple config objects with different values
```

### 2. **Database Connection Pool**
```python
# One connection pool shared across the application
# Creating multiple pools would exhaust database connections
```

### 3. **Logger Instance**
```python
# Single logger to handle all log messages
# Multiple loggers could write to same file causing corruption
```

### 4. **Cache Manager**
```python
# Global cache store
# Multiple caches would defeat the purpose of caching
```

### 5. **Service Registry (Microservices)**
```python
# Registry of available services
# Should be consistent across the application
```

## ❌ The Old (Problematic) Way: `__new__`

```python
# BAD PRACTICE - Don't use this!
class OldSingleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        # Problem: __init__ gets called multiple times!
        print(f"Initializing {self.__class__.__name__}")

# Test
s1 = OldSingleton()  # Prints: Initializing OldSingleton
s2 = OldSingleton()  # Prints: Initializing OldSingleton AGAIN!
print(s1 is s2)  # True (same object, but __init__ called twice!)
```

## ✅ Modern Singleton Implementations (Python 3.8+)

### Method 1: **Metaclass Approach** (Most Pythonic)

```python
from typing import Any, Dict


class SingletonMeta(type):
    """
    Metaclass for creating singleton classes.
    
    This is the most robust approach as it:
    1. Works with inheritance
    2. Prevents multiple __init__ calls
    3. Thread-safe with proper locking
    """
    
    _instances: Dict[type, Any] = {}
    
    def __call__(cls, *args: Any, **kwargs: Any) -> Any:
        """Override type's __call__ to control instance creation"""
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]


class DatabaseConnection(metaclass=SingletonMeta):
    """
    Real-world example: Database connection pool.
    
    Why singleton?:
    - Expensive to create multiple connections
    - Need to share connection pool across app
    - Prevent connection exhaustion
    """
    
    def __init__(self, connection_string: str = "default"):
        """Initialize only once"""
        self.connection_string = connection_string
        self.connections = []
        print(f"🚀 Creating DatabaseConnection: {connection_string}")
    
    def connect(self):
        """Simulate connection"""
        return f"Connected to {self.connection_string}"


# Test
db1 = DatabaseConnection("postgresql://localhost/mydb")
# Output: 🚀 Creating DatabaseConnection: postgresql://localhost/mydb

db2 = DatabaseConnection("different_string")  # Same instance!
print(db1 is db2)  # True
print(db1.connection_string)  # postgresql://localhost/mydb (not changed!)
print(db2.connection_string)  # postgresql://localhost/mydb
```

### Method 2: **Decorator Approach** (Clean & Reusable)

```python
from typing import Any, Callable, TypeVar
from functools import wraps
import threading

T = TypeVar('T')


def singleton(cls: T) -> T:
    """
    Class decorator for creating singleton classes.
    
    Advantages:
    - Non-invasive (just add @singleton decorator)
    - Works with any class
    - Thread-safe
    """
    instances = {}
    lock = threading.Lock()
    
    @wraps(cls)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        with lock:  # Thread safety
            if cls not in instances:
                instances[cls] = cls(*args, **kwargs)
            return instances[cls]
    
    return wrapper


@singleton
class AppConfig:
    """
    Real-world example: Application configuration.
    
    Why singleton?:
    - Single source of truth for config
    - Load config once from file/env
    - Consistent values across app
    """
    
    def __init__(self, config_file: str = "config.yaml"):
        """Load configuration from file"""
        self.config_file = config_file
        self.settings = self._load_config()
        print(f"📁 Loaded config from {config_file}")
    
    def _load_config(self) -> dict:
        """Simulate config loading"""
        return {
            "database": {"url": "localhost:5432", "pool_size": 10},
            "api": {"timeout": 30, "retries": 3},
            "logging": {"level": "INFO", "file": "app.log"}
        }
    
    def get(self, key: str, default: Any = None) -> Any:
        """Get config value with dot notation"""
        keys = key.split('.')
        value = self.settings
        for k in keys:
            value = value.get(k, {})
        return value if value != {} else default


# Test
config1 = AppConfig("production.yaml")
# Output: 📁 Loaded config from production.yaml

config2 = AppConfig("development.yaml")  # Same instance!
print(config1 is config2)  # True
print(config1.get("database.url"))  # localhost:5432
```

### Method 3: **`__call__` on Class (Borg Pattern Variant)**

```python
class LoggerSingleton:
    """
    Real-world example: Centralized logger.
    
    Why singleton?:
    - Single log file/stream
    - Consistent log format
    - Prevent log corruption from multiple writers
    """
    
    _instance = None
    _initialized = False
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        """Ensure __init__ only runs once"""
        if not self.__class__._initialized:
            self.handlers = []
            self.log_level = "INFO"
            self.__class__._initialized = True
            print("📝 Logger initialized")
    
    def add_handler(self, handler):
        self.handlers.append(handler)
    
    def log(self, message: str, level: str = "INFO"):
        print(f"[{level}] {message}")


# Alternative: Using class as singleton (Python 3.11+)
class ServiceRegistry:
    """
    Using class itself as singleton (no instances).
    
    Best for stateless registries.
    """
    
    _services: dict[str, Any] = {}
    
    @classmethod
    def register(cls, name: str, service: Any) -> None:
        cls._services[name] = service
    
    @classmethod
    def get(cls, name: str) -> Any:
        return cls._services.get(name)
    
    @classmethod
    def list_services(cls) -> list[str]:
        return list(cls._services.keys())


# Test ServiceRegistry (class as singleton)
ServiceRegistry.register("auth", "AuthService")
ServiceRegistry.register("payment", "PaymentService")
print(ServiceRegistry.list_services())  # ['auth', 'payment']
```

### Method 4: **Module-Level Singleton** (Simplest)

```python
# config.py - Module as singleton
"""
Sometimes a module itself can act as a singleton.
Just create your object at module level.
"""

_config_data = None


def init_config(config_file: str):
    global _config_data
    if _config_data is None:
        # Load config once
        _config_data = {"file": config_file, "settings": {}}
        print(f"Config initialized from {config_file}")


def get_config():
    return _config_data


# Usage in other files:
# from config import get_config, init_config
# init_config("settings.yaml")
# config = get_config()
```

### Method 5: **Dependency Injection Container** (Modern Microservices)

```python
from typing import Dict, Type, Any
from dataclasses import dataclass, field
import threading


@dataclass
class Container:
    """
    Modern approach: Dependency Injection container.
    
    Advantages:
    - Explicit dependencies
    - Easy testing (mock dependencies)
    - Lifecycle management
    """
    
    _instances: Dict[str, Any] = field(default_factory=dict)
    _lock: threading.Lock = field(default_factory=threading.Lock)
    
    def singleton(self, cls: Type) -> Any:
        """Register/get a singleton service"""
        with self._lock:
            if cls.__name__ not in self._instances:
                self._instances[cls.__name__] = cls()
            return self._instances[cls.__name__]
    
    def register_instance(self, name: str, instance: Any) -> None:
        """Register an existing instance"""
        with self._lock:
            self._instances[name] = instance
    
    def get(self, name: str) -> Any:
        """Get registered instance"""
        return self._instances.get(name)


# Usage
container = Container()

# Define services
class EmailService:
    def send(self, to: str, message: str) -> bool:
        print(f"📧 Email to {to}: {message}")
        return True

class NotificationService:
    def __init__(self, email_service: EmailService):
        self.email = email_service
    
    def notify(self, user: str, msg: str) -> bool:
        return self.email.send(user, f"Notification: {msg}")


# Get singletons
email1 = container.singleton(EmailService)
email2 = container.singleton(EmailService)
print(email1 is email2)  # True

notification = NotificationService(email1)
notification.notify("user@example.com", "Hello!")
```

## Thread-Safe Singleton with `__init_subclass__` (Python 3.6+)

```python
import threading
from typing import Dict, Any


class ThreadSafeSingleton:
    """
    Base class for thread-safe singletons using __init_subclass__.
    
    Usage:
    class MyService(ThreadSafeSingleton):
        pass
    """
    
    _instances: Dict[type, Any] = {}
    _locks: Dict[type, threading.Lock] = {}
    
    def __init_subclass__(cls, **kwargs):
        """Initialize subclass with singleton behavior"""
        super().__init_subclass__(**kwargs)
        cls._locks[cls] = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        with cls._locks[cls]:
            if cls not in cls._instances:
                instance = super().__new__(cls)
                cls._instances[cls] = instance
                instance._initialized = False
            return cls._instances[cls]
    
    def __init__(self, *args, **kwargs):
        """Ensure initialization happens only once"""
        if not self._initialized:
            super().__init__(*args, **kwargs)
            self._initialized = True


# Real example: Cache Manager
class CacheManager(ThreadSafeSingleton):
    """
    Real-world example: Distributed cache manager.
    
    Why singleton?:
    - Single cache store across app
    - Thread-safe operations
    - Memory management
    """
    
    def __init__(self, max_size: int = 1000):
        if not hasattr(self, '_initialized') or not self._initialized:
            self.cache = {}
            self.max_size = max_size
            self.hits = 0
            self.misses = 0
            super().__init__()
            print(f"💾 CacheManager initialized (max_size={max_size})")
    
    def get(self, key: str):
        if key in self.cache:
            self.hits += 1
            return self.cache[key]
        self.misses += 1
        return None
    
    def set(self, key: str, value: Any):
        if len(self.cache) >= self.max_size:
            # Simple LRU: remove first item
            self.cache.pop(next(iter(self.cache)))
        self.cache[key] = value
    
    def stats(self):
        return {"hits": self.hits, "misses": self.misses, "size": len(self.cache)}
```

## Testing Singletons (Critical!)

```python
import pytest
from unittest.mock import Mock, patch


class TestSingleton:
    """Testing strategies for singleton classes"""
    
    def test_singleton_creates_one_instance(self):
        # Arrange
        class TestClass(metaclass=SingletonMeta):
            pass
        
        # Act
        instance1 = TestClass()
        instance2 = TestClass()
        
        # Assert
        assert instance1 is instance2
    
    def test_singleton_preserves_state(self):
        # Arrange
        @singleton
        class Stateful:
            def __init__(self):
                self.value = 0
        
        # Act
        obj1 = Stateful()
        obj1.value = 42
        obj2 = Stateful()
        
        # Assert
        assert obj2.value == 42
    
    def test_singleton_with_mocks(self):
        """Testing with mocked dependencies"""
        with patch('database.DatabaseConnection') as mock_db:
            # Arrange
            mock_instance = Mock()
            mock_db.return_value = mock_instance
            
            # Act
            from myapp import get_database
            db1 = get_database()
            db2 = get_database()
            
            # Assert
            mock_db.assert_called_once()  # Only instantiated once
            assert db1 is db2
    
    def test_reset_singleton_for_testing(self):
        """Technique to reset singleton between tests"""
        
        # In your singleton class, add a class method:
        class ResettableSingleton:
            _instance = None
            
            @classmethod
            def reset(cls):
                cls._instance = None
        
        # In test setup/teardown:
        def setup_method(self):
            ResettableSingleton.reset()
```

## When NOT to Use Singletons

### ❌ Bad Use Cases:

```python
# 1. As a global variable replacement
# BAD: Using singleton to avoid passing parameters
class UserService:
    def get_user(self):
        # Bad: Hidden dependency on Config singleton
        api_url = Config().get("api.url")  # Hidden coupling!
        return requests.get(api_url)

# GOOD: Explicit dependency injection
class UserService:
    def __init__(self, config: Config):
        self.config = config
    
    def get_user(self):
        api_url = self.config.get("api.url")  # Explicit dependency
        return requests.get(api_url)


# 2. For stateless utility classes
# BAD: Singleton for pure functions
@singleton
class MathUtils:  # Doesn't need to be singleton!
    @staticmethod
    def add(a, b):
        return a + b

# GOOD: Just use module functions
# math_utils.py
def add(a, b):
    return a + b


# 3. When testing is important
# Singletons make unit testing harder due to shared state
```

### ✅ Good Use Cases:

1. **Resource Managers** (Database, Cache, File handles)
2. **Configuration Loaders** (Load once, read many)
3. **Service Locators** (Microservice registries)
4. **Logging Systems** (Single log stream)
5. **Hardware Controllers** (Single physical device)

## Best Practice Summary

### **Recommended Approach** (Python 3.8+):

```python
from typing import Any, Dict
import threading


class SingletonMeta(type):
    """Metaclass for thread-safe singletons"""
    
    _instances: Dict[type, Any] = {}
    _lock: threading.Lock = threading.Lock()
    
    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
            return cls._instances[cls]


class DatabaseConnectionPool(metaclass=SingletonMeta):
    """
    Example: Database connection pool (excellent singleton use case)
    
    Features:
    - Thread-safe initialization
    - Single __init__ call
    - Works with inheritance
    - Clear documentation
    """
    
    def __init__(self, max_connections: int = 10):
        """Initialize once only"""
        self.max_connections = max_connections
        self.connections = []
        self._initialize_pool()
    
    def _initialize_pool(self):
        """Create connection pool"""
        print(f"Initializing pool with {self.max_connections} connections")
        # Actual connection creation logic here
    
    def get_connection(self):
        """Get a database connection"""
        # Pool management logic
        pass
    
    def release_connection(self, conn):
        """Release connection back to pool"""
        pass
```

### **Key Principles**:

1. **Thread Safety**: Always use locking if your app is multi-threaded
2. **Single Initialization**: Ensure `__init__` runs only once
3. **Testability**: Design for easy mocking in tests
4. **Documentation**: Clearly document why it's a singleton
5. **Consider Alternatives**: First consider dependency injection

### **Python 3.11+ Enhancement**:

```python
from typing import Self


class ModernSingleton:
    """Using Self type for better type hints"""
    
    _instance: Self = None
    
    def __new__(cls) -> Self:
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not hasattr(self, '_initialized'):
            # Initialization code
            self._initialized = True
```

## Final Recommendation

**For most cases in Python, you don't need a formal singleton pattern.** Consider these alternatives first:

1. **Module-level variables**: Simple and effective
2. **Dependency injection**: Better for testing
3. **Class methods**: For stateless operations

**Only use singletons when**:
- You have a genuine resource constraint (database connections, file handles)
- You need strict global state management
- The overhead of creating multiple instances is significant

Remember: **"Singletons solve problems you often shouldn't have"**. Use them judiciously and document the rationale clearly.
