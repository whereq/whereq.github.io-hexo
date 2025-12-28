---
title: 'Deep Dive: Python Multi-Threading'
date: 2025-12-28 11:29:49
tags:
---

Python threading is **NOT true parallelism** due to the Global Interpreter Lock (GIL), but it's **incredibly useful for I/O-bound tasks**. In Python 3.11+, threading has evolved with better performance, new APIs, and improved synchronization primitives.

## 📊 The Python Threading Model: GIL Reality

### The Global Interpreter Lock (GIL) Explained

```python
"""
GIL Architecture:
┌──────────────────────────────────────────────────────────────────────┐
│                    Python Interpreter                                │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │               Global Interpreter Lock                         │   │
│  │     (Only ONE thread executes Python bytecode                 │   │
│  │                at any given time)                             │   │
│  └───────────────────────────────────────────────────────────────┘   │
│           │                      │                                   │
│   Thread 1 (Running)     Thread 2 (Waiting)     Thread 3 (Waiting)   |
└──────────────────────────────────────────────────────────────────────┘

When a thread releases the GIL:
1. After executing ~100 ticks of bytecode
2. During I/O operations (file, network, sleep)
3. When explicitly calling time.sleep(0)
"""

# Visualizing GIL contention
import threading
import time

def gil_demo():
    """Demonstrate GIL limitations for CPU-bound tasks."""
    
    def cpu_intensive(n):
        """CPU-bound task - suffers from GIL."""
        count = 0
        for i in range(n):
            count += i
        return count
    
    def io_bound(delay):
        """I/O-bound task - benefits from threading."""
        time.sleep(delay)
        return f"Slept for {delay}s"
    
    # CPU-bound threading (NO speedup)
    start = time.time()
    threads = [
        threading.Thread(target=cpu_intensive, args=(10_000_000,))
        for _ in range(4)
    ]
    
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    
    cpu_time = time.time() - start
    print(f"CPU-bound (4 threads): {cpu_time:.2f}s")
    
    # I/O-bound threading (BIG speedup)
    start = time.time()
    threads = [
        threading.Thread(target=io_bound, args=(1,))
        for _ in range(4)
    ]
    
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    
    io_time = time.time() - start
    print(f"I/O-bound (4 threads): {io_time:.2f}s (Expected: ~1s)")
```

## 🎯 When to Use Threading vs Multiprocessing vs Async

```
Decision Tree for Concurrency in Python:

Start
  ↓
Is the task I/O-bound? → YES → Consider Threading or Async
  ↓ NO                       │
Is the task CPU-bound? → YES → Use Multiprocessing
  ↓ NO                       │
Single-threaded is fine      │
                             │
                    ┌────────┴────────┐
                    │                 │
              Need callback-style?    Many connections?
                    │                 │
                    ▼                 ▼
               Async (asyncio)    Threading
                    │                 │
                    └────────┬────────┘
                             ↓
                    Use concurrent.futures
```

### Performance Comparison Matrix
```python
import concurrent.futures
import multiprocessing
import threading
import asyncio
import time
import requests

class ConcurrencyBenchmark:
    """Benchmark different concurrency approaches."""
    
    @staticmethod
    def benchmark_approaches():
        """Compare threading, multiprocessing, and async."""
        
        def cpu_task(n):
            """CPU-bound task."""
            return sum(i * i for i in range(n))
        
        async def async_io_task(delay):
            """Async I/O task."""
            await asyncio.sleep(delay)
            return delay
        
        def sync_io_task(delay):
            """Sync I/O task."""
            time.sleep(delay)
            return delay
        
        results = {}
        
        # CPU-bound: Multiprocessing wins
        print("CPU-bound benchmark (sum of squares):")
        n = 10_000_000
        tasks = 4
        
        # Single-threaded baseline
        start = time.time()
        for _ in range(tasks):
            cpu_task(n)
        results["single"] = time.time() - start
        
        # Threading (GIL-limited)
        start = time.time()
        with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
            list(executor.map(cpu_task, [n] * tasks))
        results["threading"] = time.time() - start
        
        # Multiprocessing (true parallelism)
        start = time.time()
        with concurrent.futures.ProcessPoolExecutor(max_workers=4) as executor:
            list(executor.map(cpu_task, [n] * tasks))
        results["multiprocessing"] = time.time() - start
        
        print(f"Single-threaded: {results['single']:.2f}s")
        print(f"Threading: {results['threading']:.2f}s ({results['single']/results['threading']:.1f}x)")
        print(f"Multiprocessing: {results['multiprocessing']:.2f}s ({results['single']/results['multiprocessing']:.1f}x)")
        
        return results

# Run benchmark
benchmark = ConcurrencyBenchmark()
benchmark.benchmark_approaches()
```

## 🏗️ Modern Threading APIs (Python 3.7+)

### 1. **`concurrent.futures.ThreadPoolExecutor`** (Recommended)
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from typing import List, Dict, Any
import requests
import time

class ModernThreadPoolExample:
    """Modern thread pool usage with error handling."""
    
    def fetch_urls_concurrently(self, urls: List[str], 
                               max_workers: int = 5) -> Dict[str, Any]:
        """
        Fetch multiple URLs concurrently using ThreadPoolExecutor.
        
        Thread Pool Architecture:
        ┌────────────────────────────────────────────────┐
        │          ThreadPoolExecutor                    │
        │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  │
        │  │Worker│ │Worker│ │Worker│ │Worker│ │Worker│  │
        │  │  1   │ │  2   │ │  3   │ │  4   │ │  5   │  │
        │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘  │
        │       │       │       │       │       │        │
        │  ┌────┴───────┴───────┴───────┴───────┴──┐     │
        │  │           Task Queue                  │     │
        │  │  [url1, url2, url3, url4, url5, ...]  │     │
        │  └───────────────────────────────────────┘     │
        └────────────────────────────────────────────────┘
        """
        
        def fetch_url(url: str) -> Dict[str, Any]:
            """Worker function to fetch a single URL."""
            start_time = time.time()
            try:
                response = requests.get(url, timeout=10)
                return {
                    "url": url,
                    "status": response.status_code,
                    "content_length": len(response.content),
                    "time_taken": time.time() - start_time,
                    "success": True
                }
            except Exception as e:
                return {
                    "url": url,
                    "error": str(e),
                    "time_taken": time.time() - start_time,
                    "success": False
                }
        
        results = {}
        
        # Using ThreadPoolExecutor with context manager
        with ThreadPoolExecutor(max_workers=max_workers) as executor:
            # Submit all tasks
            future_to_url = {
                executor.submit(fetch_url, url): url
                for url in urls
            }
            
            # Process results as they complete
            for future in as_completed(future_to_url):
                url = future_to_url[future]
                try:
                    results[url] = future.result(timeout=15)
                except Exception as e:
                    results[url] = {
                        "url": url,
                        "error": f"Future exception: {e}",
                        "success": False
                    }
        
        return results
    
    def batch_process_with_limits(self, items: List[str], 
                                 process_func: callable,
                                 max_workers: int = 10,
                                 batch_size: int = 100) -> List[Any]:
        """
        Process items in batches with rate limiting.
        
        Batch Processing Flow:
        ┌─────────┐     ┌──────────────┐     ┌─────────────┐
        │ Input   │───>>│  Batch Split │───>>│ Thread Pool │
        │ Items   │     │   (size=100) │     │ (max=10)    │
        └─────────┘     └──────────────┘     └─────────────┘
                                                           
        ┌─────────────┐     ┌─────────────┐     ┌─────────┐
        │ Process     │───>>│ Collect     │───>>│ Output  │
        │ Batch       │     │ Results     │     │ Results │
        └─────────────┘     └─────────────┘     └─────────┘
        """
        
        all_results = []
        
        # Process in batches to control memory usage
        for i in range(0, len(items), batch_size):
            batch = items[i:i + batch_size]
            
            with ThreadPoolExecutor(max_workers=max_workers) as executor:
                # Using map for simpler cases
                batch_results = list(executor.map(process_func, batch))
                all_results.extend(batch_results)
            
            # Optional: Add delay between batches
            if i + batch_size < len(items):
                time.sleep(0.1)  # Small delay to prevent overwhelming systems
        
        return all_results

# Usage
processor = ModernThreadPoolExample()
urls = [
    "https://httpbin.org/get",
    "https://httpbin.org/delay/2",
    "https://httpbin.org/status/404",
    "https://httpbin.org/status/500",
]

results = processor.fetch_urls_concurrently(urls, max_workers=3)
for url, result in results.items():
    status = "✓" if result["success"] else "✗"
    print(f"{status} {url}: {result.get('status', 'Error')}")

```

### 2. **Advanced ThreadPoolExecutor Patterns**
```python
from concurrent.futures import ThreadPoolExecutor, wait, FIRST_COMPLETED
from typing import List, Tuple, Callable, Any
import time
from dataclasses import dataclass
from enum import Enum

class TaskPriority(Enum):
    HIGH = 1
    MEDIUM = 2
    LOW = 3

@dataclass
class PrioritizedTask:
    """Task with priority for advanced scheduling."""
    func: Callable
    args: Tuple
    kwargs: dict
    priority: TaskPriority
    created_at: float = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = time.time()
    
    def __lt__(self, other):
        # Priority comparison for queue ordering
        if self.priority.value != other.priority.value:
            return self.priority.value < other.priority.value
        return self.created_at < other.created_at

class AdvancedThreadPool:
    """Advanced thread pool with priority scheduling and monitoring."""
    
    def __init__(self, max_workers: int = None, name_prefix: str = "Worker"):
        self.max_workers = max_workers or (multiprocessing.cpu_count() * 2)
        self.name_prefix = name_prefix
        self._stats = {
            "tasks_completed": 0,
            "tasks_failed": 0,
            "total_time": 0.0
        }
    
    def execute_with_priority(self, 
                            tasks: List[PrioritizedTask],
                            timeout: float = None) -> List[Any]:
        """
        Execute tasks with priority scheduling.
        
        Priority Queue Behavior:
        ┌─────────────────────────────────────────┐
        │      Priority Queue (Min-Heap)          │
        ├─────────────────────────────────────────┤
        │ [HIGH] Task A (created: 10:00:00)       │
        │ [HIGH] Task B (created: 10:00:01)       │
        │ [MEDIUM] Task C (created: 10:00:00)     │
        │ [LOW] Task D (created: 10:00:02)        │
        └─────────────────────────────────────────┘
              │
              ▼ (Thread picks highest priority)
        ┌─────────────────────────────────────────┐
        │           Thread Pool                   │
        │  Worker 1: Task A (HIGH)                │
        │  Worker 2: Task B (HIGH)                │
        │  Worker 3: Task C (MEDIUM)              │
        └─────────────────────────────────────────┘
        """
        
        # Sort tasks by priority (and creation time for tie-breaking)
        sorted_tasks = sorted(tasks)
        
        results = []
        start_time = time.time()
        
        with ThreadPoolExecutor(
            max_workers=self.max_workers,
            thread_name_prefix=self.name_prefix
        ) as executor:
            
            # Submit all tasks
            future_to_task = {}
            for task in sorted_tasks:
                future = executor.submit(task.func, *task.args, **task.kwargs)
                future_to_task[future] = task
            
            # Wait for completion with timeout
            if timeout:
                done, not_done = wait(future_to_task.keys(), timeout=timeout)
                
                # Handle timeout
                if not_done:
                    print(f"Warning: {len(not_done)} tasks timed out")
                    for future in not_done:
                        future.cancel()
            else:
                done = future_to_task.keys()
            
            # Collect results
            for future in done:
                task = future_to_task[future]
                try:
                    result = future.result(timeout=1)
                    results.append((task, result, None))
                    self._stats["tasks_completed"] += 1
                except Exception as e:
                    results.append((task, None, str(e)))
                    self._stats["tasks_failed"] += 1
        
        self._stats["total_time"] = time.time() - start_time
        return results
    
    def get_statistics(self) -> dict:
        """Get execution statistics."""
        stats = self._stats.copy()
        if stats["tasks_completed"] + stats["tasks_failed"] > 0:
            stats["success_rate"] = (
                stats["tasks_completed"] / 
                (stats["tasks_completed"] + stats["tasks_failed"])
            )
        return stats

# Usage
def process_item(item: str, weight: int = 1) -> str:
    """Simulate processing with variable weight."""
    time.sleep(weight * 0.1)
    return f"Processed: {item}"

pool = AdvancedThreadPool(max_workers=3)

tasks = [
    PrioritizedTask(process_item, ("urgent",), {"weight": 1}, TaskPriority.HIGH),
    PrioritizedTask(process_item, ("important",), {"weight": 2}, TaskPriority.HIGH),
    PrioritizedTask(process_item, ("normal",), {"weight": 3}, TaskPriority.MEDIUM),
    PrioritizedTask(process_item, ("background",), {"weight": 5}, TaskPriority.LOW),
]

results = pool.execute_with_priority(tasks)
stats = pool.get_statistics()

print(f"Completed {stats['tasks_completed']} tasks in {stats['total_time']:.2f}s")
print(f"Success rate: {stats.get('success_rate', 0):.1%}")
```

## 🔒 Thread Synchronization: Modern Patterns

### 1. **Lock Hierarchy and Deadlock Prevention**
```python
import threading
from typing import Dict, Any
from contextlib import contextmanager
from enum import Enum
import time

class LockType(Enum):
    READ = "read"
    WRITE = "write"

class ThreadSafeResourceManager:
    """
    Modern thread-safe resource manager with lock hierarchy.
    
    Lock Hierarchy Design:
    ┌─────────────────────────────────────────┐
    │           Global Order Lock             │
    │   (Ensures consistent lock ordering)    │
    └─────────────────────────────────────────┘
                │           │
        ┌───────┴───────┐ ┌─┴────────────┐
        │ Database Lock │ │ Cache Lock   │
        │   (Level 2)   │ │  (Level 1)   │
        └───────┬───────┘ └─┬────────────┘
                │           │
        ┌───────┴───────┐ ┌─┴────────────┐
        │   Acquire     │ │   Acquire    │
        │  in order:    │ │  Cache first │
        │  1. Cache     │ │  2. Database │
        │  2. Database  │ │  (AVOID!)    │
        └───────────────┘ └──────────────┘
    """
    
    def __init__(self):
        # Define lock hierarchy (lower level = acquired first)
        self._locks = {
            "cache": threading.RLock(),      # Level 1
            "database": threading.RLock(),   # Level 2
            "file_system": threading.RLock() # Level 3
        }
        
        # Lock order must be consistent to prevent deadlocks
        self._lock_order = ["cache", "database", "file_system"]
        
        # Statistics for monitoring
        self._lock_stats: Dict[str, Dict[str, int]] = {
            lock_name: {"acquires": 0, "contention": 0, "wait_time": 0.0}
            for lock_name in self._locks
        }
    
    @contextmanager
    def acquire_locks(self, *lock_names: str, timeout: float = 5.0):
        """
        Acquire multiple locks in consistent order.
        
        Context Manager Flow:
        ┌─────────────────────────────────────────┐
        │    Enter acquire_locks() context        │
        │  ┌───────────────────────────────────┐  │
        │  │ 1. Sort locks by hierarchy order  │  │
        │  │ 2. Try acquire each with timeout  │  │
        │  │ 3. If any fails, release all      │  │
        │  │ 4. If all succeed, yield control  │  │
        │  └───────────────────────────────────┘  │
        │               │                         │
        │        Code executes with               │
        │        all locks held                   │
        │               │                         │
        │  ┌───────────────────────────────────┐  │
        │  │ 1. Release locks in reverse order │  │
        │  │ 2. Update statistics              │  │
        │  └───────────────────────────────────┘  │
        └─────────────────────────────────────────┘
        """
        acquired = []
        start_wait = time.time()
        
        try:
            # Sort locks by hierarchy to prevent deadlocks
            sorted_locks = sorted(
                lock_names,
                key=lambda x: self._lock_order.index(x) if x in self._lock_order else 999
            )
            
            for lock_name in sorted_locks:
                if lock_name not in self._locks:
                    raise ValueError(f"Unknown lock: {lock_name}")
                
                lock = self._locks[lock_name]
                lock_start = time.time()
                
                # Try to acquire with timeout
                acquired_lock = lock.acquire(timeout=timeout)
                if not acquired_lock:
                    # Timeout - release all previously acquired locks
                    for acquired_lock_name in reversed(acquired):
                        self._locks[acquired_lock_name].release()
                    raise TimeoutError(f"Timeout acquiring lock: {lock_name}")
                
                lock_wait = time.time() - lock_start
                if lock_wait > 0.001:  # Contention detected
                    self._lock_stats[lock_name]["contention"] += 1
                    self._lock_stats[lock_name]["wait_time"] += lock_wait
                
                self._lock_stats[lock_name]["acquires"] += 1
                acquired.append(lock_name)
            
            total_wait = time.time() - start_wait
            if total_wait > 0.01:
                print(f"Warning: High lock contention ({total_wait:.3f}s)")
            
            yield  # Execute the critical section
            
        finally:
            # Release locks in reverse order
            for lock_name in reversed(acquired):
                self._locks[lock_name].release()
    
    def update_cache_and_database(self, key: str, value: Any):
        """Example of safe multi-lock acquisition."""
        with self.acquire_locks("cache", "database"):
            # Both locks are held, safe to update both resources
            print(f"Updating {key} = {value} in cache and database")
            # Simulate updates
            time.sleep(0.01)
    
    def get_statistics(self) -> Dict[str, Any]:
        """Get lock usage statistics."""
        stats = {}
        for lock_name, lock_data in self._lock_stats.items():
            if lock_data["acquires"] > 0:
                avg_wait = lock_data["wait_time"] / lock_data["acquires"]
                stats[lock_name] = {
                    "acquires": lock_data["acquires"],
                    "contention_events": lock_data["contention"],
                    "avg_wait_time_ms": avg_wait * 1000,
                    "contention_percent": (lock_data["contention"] / lock_data["acquires"]) * 100
                }
        return stats

# Usage
manager = ThreadSafeResourceManager()

# Multiple threads updating resources safely
def worker(worker_id: int):
    for i in range(5):
        manager.update_cache_and_database(f"key_{worker_id}_{i}", f"value_{i}")
        time.sleep(0.005)

threads = [threading.Thread(target=worker, args=(i,)) for i in range(3)]
for t in threads:
    t.start()
for t in threads:
    t.join()

stats = manager.get_statistics()
print("\nLock Statistics:")
for lock_name, data in stats.items():
    print(f"{lock_name}: {data['acquires']} acquires, "
          f"{data['contention_percent']:.1f}% contention")
```

### 2. **Read-Write Locks (Python 3.8+)**
```python
import threading
from typing import List
from contextlib import contextmanager
from collections import defaultdict

class ReadWriteLock:
    """
    Read-Write Lock implementation.
    
    Multiple readers OR single writer can hold the lock.
    
    State Machine:
    ┌─────────┐   Writer requests   ┌─────────┐
    │  FREE   │───────────────────>>│ WRITING │
    │         │<<───────────────────│         │
    └─────────┘   Writer releases   └─────────┘
          │            │                 │
          │            │ Reader requests │
    Reader│requests    │           Reader│releases
          │            │                 │
          ▼            ▼                 ▼
    ┌─────────┐   Writer waits   ┌─────────┐
    │ READING │<<─────────────── │WAITING  │
    │         │────────────────>>│WRITER   │
    └─────────┘ Last reader      └─────────┘
                    releases
    """
    
    def __init__(self):
        self._read_ready = threading.Condition(threading.Lock())
        self._readers = 0
        self._writers_waiting = 0
        self._writer_active = False
    
    @contextmanager
    def read_lock(self):
        """Acquire a read lock."""
        with self._read_ready:
            while self._writer_active or self._writers_waiting > 0:
                self._read_ready.wait()
            self._readers += 1
        
        try:
            yield
        finally:
            with self._read_ready:
                self._readers -= 1
                if self._readers == 0:
                    self._read_ready.notify_all()
    
    @contextmanager
    def write_lock(self):
        """Acquire a write lock."""
        with self._read_ready:
            self._writers_waiting += 1
            while self._readers > 0 or self._writer_active:
                self._read_ready.wait()
            self._writers_waiting -= 1
            self._writer_active = True
        
        try:
            yield
        finally:
            with self._read_ready:
                self._writer_active = False
                self._read_ready.notify_all()

class ThreadSafeDictionary:
    """Dictionary protected by read-write lock."""
    
    def __init__(self):
        self._lock = ReadWriteLock()
        self._data = {}
        self._access_stats = defaultdict(int)
    
    def get(self, key, default=None):
        """Thread-safe get (multiple readers allowed)."""
        with self._lock.read_lock():
            self._access_stats['reads'] += 1
            return self._data.get(key, default)
    
    def set(self, key, value):
        """Thread-safe set (single writer)."""
        with self._lock.write_lock():
            self._access_stats['writes'] += 1
            self._data[key] = value
    
    def update_batch(self, items: dict):
        """Update multiple items atomically."""
        with self._lock.write_lock():
            self._access_stats['batch_writes'] += 1
            self._data.update(items)
    
    def get_stats(self):
        """Get access statistics."""
        with self._lock.read_lock():
            return dict(self._access_stats)

# Usage
def reader(dict_obj: ThreadSafeDictionary, reader_id: int):
    """Reader thread function."""
    for i in range(100):
        value = dict_obj.get(f"key_{i % 10}")
        time.sleep(0.001)

def writer(dict_obj: ThreadSafeDictionary, writer_id: int):
    """Writer thread function."""
    for i in range(20):
        dict_obj.set(f"key_{writer_id}_{i}", f"value_{i}")
        time.sleep(0.005)

safe_dict = ThreadSafeDictionary()

# Start multiple readers and a few writers
reader_threads = [threading.Thread(target=reader, args=(safe_dict, i)) 
                  for i in range(5)]
writer_threads = [threading.Thread(target=writer, args=(safe_dict, i)) 
                  for i in range(2)]

for t in reader_threads + writer_threads:
    t.start()
for t in reader_threads + writer_threads:
    t.join()

stats = safe_dict.get_stats()
print(f"Access stats: {stats}")
```

## 🚨 Thread Safety Best Practices

### 1. **Immutable Data Patterns**
```python
from typing import NamedTuple, FrozenSet
from dataclasses import dataclass, field
from datetime import datetime
import threading

@dataclass(frozen=True)  # Immutable dataclass
class ImmutableConfig:
    """Thread-safe immutable configuration."""
    timeout: int
    retries: int
    endpoints: FrozenSet[str]
    created_at: datetime = field(default_factory=datetime.utcnow)
    
    def with_updates(self, **kwargs):
        """Create new instance with updates (copy-on-write)."""
        return dataclasses.replace(self, **kwargs)

class ThreadSafeConfigManager:
    """
    Configuration manager using immutable objects.
    
    Copy-on-Write Pattern:
    ┌─────────────────────────────────────────┐
    │   Thread 1 reads config v1              │
    │   Thread 2 reads config v1              │
    │   Thread 3 reads config v1              │
    └─────────────────────────────────────────┘
                      │
          ┌───────────┴───────────┐
          │ Admin updates config  │
          │ (creates new v2)      │
          └───────────┬───────────┘
                      │
    ┌─────────────────┴─────────────────┐
    │ All new reads get config v2       │
    │ Existing threads continue with v1 │
    └───────────────────────────────────┘
    """
    
    def __init__(self):
        self._config = ImmutableConfig(
            timeout=30,
            retries=3,
            endpoints=frozenset(["api.example.com"])
        )
        self._lock = threading.RLock()
        self._version = 1
    
    def get_config(self) -> ImmutableConfig:
        """Thread-safe config access (no lock needed for reads!)."""
        # No lock needed because config is immutable!
        return self._config
    
    def update_config(self, **kwargs) -> int:
        """Thread-safe config update (atomic swap)."""
        with self._lock:
            self._config = self._config.with_updates(**kwargs)
            self._version += 1
            return self._version
    
    def get_version(self) -> int:
        """Get current config version."""
        return self._version

# Usage
config_manager = ThreadSafeConfigManager()

def worker(worker_id: int):
    """Worker thread using immutable config."""
    config = config_manager.get_config()
    print(f"Worker {worker_id} using config v{config_manager.get_version()}: "
          f"timeout={config.timeout}, endpoints={config.endpoints}")
    time.sleep(0.1)

# Start workers
threads = [threading.Thread(target=worker, args=(i,)) for i in range(3)]
for t in threads:
    t.start()

# Update config while workers are running
time.sleep(0.05)
new_version = config_manager.update_config(timeout=60, retries=5)
print(f"\nUpdated config to v{new_version}")

for t in threads:
    t.join()
```

### 2. **Thread-Local Storage**
```python
import threading
from typing import Dict, Any
from contextvars import ContextVar
import random

class ThreadLocalManager:
    """
    Managing thread-local data with modern Python.
    
    Thread-Local Storage Architecture:
    ┌─────────────────────────────────────────┐
    │        Thread 1                         │
    │  ┌──────────────────────────────┐       │
    │  │  thread_local_storage:       │       │
    │  │  {'request_id': 'req_123',   │       │
    │  │   'user_id': 'user_456'}     │       │
    │  └──────────────────────────────┘       │
    └─────────────────────────────────────────┘
    
    ┌─────────────────────────────────────────┐
    │        Thread 2                         │
    │  ┌──────────────────────────────┐       │
    │  │  thread_local_storage:       │       │
    │  │  {'request_id': 'req_789',   │       │
    │  │   'user_id': 'user_012'}     │       │
    │  └──────────────────────────────┘       │
    └─────────────────────────────────────────┘
    """
    
    def __init__(self):
        # Python 3.7+: Use contextvars for async/thread compatibility
        self.request_id: ContextVar[str] = ContextVar('request_id', default='')
        self.user_id: ContextVar[str] = ContextVar('user_id', default='')
        
        # Legacy thread-local for comparison
        self._legacy_thread_local = threading.local()
    
    def set_request_context(self, request_id: str, user_id: str):
        """Set context for current thread."""
        self.request_id.set(request_id)
        self.user_id.set(user_id)
        
        # Legacy approach
        self._legacy_thread_local.request_id = request_id
        self._legacy_thread_local.user_id = user_id
    
    def get_request_info(self) -> Dict[str, str]:
        """Get context from current thread."""
        return {
            "request_id": self.request_id.get(),
            "user_id": self.user_id.get(),
            "thread_id": threading.get_ident()
        }
    
    def process_request(self, request_data: Dict[str, Any]):
        """Process request with thread-local context."""
        # Set thread-local context
        self.set_request_context(
            request_id=request_data.get('request_id', 'unknown'),
            user_id=request_data.get('user_id', 'anonymous')
        )
        
        # Access thread-local data
        context = self.get_request_info()
        print(f"[Thread {context['thread_id']}] Processing request {context['request_id']} "
              f"for user {context['user_id']}")
        
        # Simulate work
        time.sleep(random.uniform(0.01, 0.05))
        
        return {"status": "processed", "context": context}

# Usage
manager = ThreadLocalManager()

def handle_request(request_data):
    """Request handler using thread-local storage."""
    return manager.process_request(request_data)

# Simulate multiple concurrent requests
requests = [
    {"request_id": f"req_{i}", "user_id": f"user_{i % 5}"}
    for i in range(10)
]

with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(handle_request, requests))

print(f"\nProcessed {len(results)} requests across threads")
```

## 📈 Performance Monitoring and Debugging

### 1. **Thread Profiling and Monitoring**
```python
import threading
import time
from typing import Dict, List
from dataclasses import dataclass
from collections import defaultdict
import psutil
import os

@dataclass
class ThreadMetrics:
    """Thread performance metrics."""
    thread_id: int
    thread_name: str
    cpu_percent: float
    memory_mb: float
    state: str
    user_time: float
    system_time: float
    io_counters: Dict[str, int]

class ThreadMonitor:
    """
    Real-time thread monitoring and profiling.
    
    Monitoring Architecture:
    ┌─────────────────────────────────────────┐
    │        Thread Monitor Daemon            │
    │  ┌──────────────────────────────────┐   │
    │  │  Polling Loop (every 100ms)      │   │
    │  │  1. Get all thread stats         │   │
    │  │  2. Calculate metrics            │   │
    │  │  3. Detect anomalies             │   │
    │  │  4. Log/alert if needed          │   │
    │  └──────────────────────────────────┘   │
    └─────────────────────────────────────────┘
               │
        ┌──────┴───────────────┐
        │                      │
    ┌──────────┐      ┌───────────────────────┐
    │ CSV Log  |      │ Real time dash board  |
    └──────────┘      └───────────────────────┘
    """
    
    def __init__(self, update_interval: float = 0.1):
        self.update_interval = update_interval
        self.metrics_history: Dict[int, List[ThreadMetrics]] = defaultdict(list)
        self._stop_event = threading.Event()
        self._monitor_thread = None
        
    def start_monitoring(self):
        """Start background monitoring thread."""
        self._monitor_thread = threading.Thread(
            target=self._monitor_loop,
            name="ThreadMonitor",
            daemon=True
        )
        self._monitor_thread.start()
    
    def stop_monitoring(self):
        """Stop monitoring."""
        self._stop_event.set()
        if self._monitor_thread:
            self._monitor_thread.join(timeout=2)
    
    def _monitor_loop(self):
        """Main monitoring loop."""
        process = psutil.Process(os.getpid())
        
        while not self._stop_event.is_set():
            try:
                # Get thread metrics
                thread_metrics = self._collect_metrics(process)
                
                # Store in history
                for metric in thread_metrics:
                    self.metrics_history[metric.thread_id].append(metric)
                    
                    # Keep only last 1000 samples per thread
                    if len(self.metrics_history[metric.thread_id]) > 1000:
                        self.metrics_history[metric.thread_id].pop(0)
                
                # Detect issues
                self._detect_anomalies(thread_metrics)
                
                time.sleep(self.update_interval)
                
            except Exception as e:
                print(f"Monitoring error: {e}")
                time.sleep(1)
    
    def _collect_metrics(self, process) -> List[ThreadMetrics]:
        """Collect metrics for all threads."""
        metrics = []
        
        # Get native thread IDs and stats
        thread_stats = {}
        try:
            # This requires psutil 5.7.0+
            for thread in process.threads():
                thread_stats[thread.id] = {
                    'user_time': thread.user_time,
                    'system_time': thread.system_time,
                }
        except AttributeError:
            # Fallback for older psutil
            pass
        
        # Get Python thread info
        for thread in threading.enumerate():
            thread_id = thread.ident
            if thread_id:
                try:
                    # Get CPU and memory for thread (approximate)
                    cpu_percent = 0.0
                    memory_mb = 0.0
                    
                    # Note: Per-thread CPU/memory is platform specific
                    # This is a simplified approximation
                    
                    metrics.append(ThreadMetrics(
                        thread_id=thread_id,
                        thread_name=thread.name,
                        cpu_percent=cpu_percent,
                        memory_mb=memory_mb,
                        state=str(thread),
                        user_time=thread_stats.get(thread_id, {}).get('user_time', 0),
                        system_time=thread_stats.get(thread_id, {}).get('system_time', 0),
                        io_counters={}  # Would require platform-specific code
                    ))
                except Exception as e:
                    print(f"Error collecting metrics for thread {thread.name}: {e}")
        
        return metrics
    
    def _detect_anomalies(self, metrics: List[ThreadMetrics]):
        """Detect threading issues."""
        # Check for deadlocks (simplified)
        long_running = []
        for metric in metrics:
            if metric.thread_name.startswith("Worker") and metric.state == "WAITING":
                # Check if this thread has been waiting too long
                thread_history = self.metrics_history.get(metric.thread_id, [])
                if len(thread_history) > 10:
                    # Check if thread has been in same state for last 10 samples
                    recent_states = [m.state for m in thread_history[-10:]]
                    if all(s == metric.state for s in recent_states):
                        long_running.append(metric.thread_name)
        
        if long_running:
            print(f"Warning: Threads may be stuck: {long_running}")
    
    def generate_report(self) -> Dict[str, Any]:
        """Generate monitoring report."""
        report = {
            "total_threads": len(self.metrics_history),
            "thread_details": {},
            "summary": {
                "avg_threads": 0,
                "max_threads": 0,
                "potential_issues": []
            }
        }
        
        for thread_id, history in self.metrics_history.items():
            if history:
                latest = history[-1]
                report["thread_details"][latest.thread_name] = {
                    "state": latest.state,
                    "cpu_percent": latest.cpu_percent,
                    "memory_mb": latest.memory_mb,
                    "samples_collected": len(history)
                }
        
        report["summary"]["avg_threads"] = len(self.metrics_history)
        report["summary"]["max_threads"] = len(self.metrics_history)
        
        return report

# Usage
monitor = ThreadMonitor(update_interval=0.5)
monitor.start_monitoring()

# Run some threaded workload
def workload_task(task_id: int):
    """Simulate workload."""
    for i in range(100):
        time.sleep(0.01)
        _ = i * i  # Some CPU work

with ThreadPoolExecutor(max_workers=4, thread_name_prefix="Worker") as executor:
    futures = [executor.submit(workload_task, i) for i in range(8)]
    for future in futures:
        future.result()

time.sleep(1)  # Let monitor collect final data
monitor.stop_monitoring()

report = monitor.generate_report()
print(f"Thread monitoring report: {report}")
```

## 🚀 Production-Ready Threading Patterns

### 1. **Producer-Consumer with Priority Queue**
```python
import threading
import queue
import time
from typing import Any, Optional
from dataclasses import dataclass, field
from enum import IntEnum
import random

class Priority(IntEnum):
    HIGH = 1
    NORMAL = 2
    LOW = 3

@dataclass(order=True)
class WorkItem:
    """Work item with priority for queue."""
    priority: Priority
    timestamp: float = field(default_factory=time.time)
    data: Any = field(compare=False)
    
    def process(self):
        """Process this work item."""
        time.sleep(random.uniform(0.01, 0.1))  # Simulate work
        return f"Processed: {self.data} (priority: {self.priority.name})"

class ThreadPoolWithPriority:
    """
    Production thread pool with priority queue.
    
    Architecture:
    ┌─────────────────────────────────────────┐
    │         Priority Queue                  │
    │  ┌──────────────────────────────────┐   │
    │  │ [HIGH] Item A (created: 10:00)   │   │
    │  │ [HIGH] Item B (created: 10:01)   │   │
    │  │ [NORMAL] Item C (created: 10:00) │   │
    │  │ [LOW] Item D (created: 10:02)    │   │
    │  └──────────────────────────────────┘   │
    └─────────────────────────────────────────┘
               │
        Workers take highest
        priority items first
               │
    ┌─────────────────────────────────────────┐
    │          Worker Threads                 │
    │  Worker 1: [HIGH] Item A                │
    │  Worker 2: [HIGH] Item B                │
    │  Worker 3: [NORMAL] Item C              │
    └─────────────────────────────────────────┘
    """
    
    def __init__(self, num_workers: int = 4):
        self.task_queue = queue.PriorityQueue()
        self.workers: List[threading.Thread] = []
        self.shutdown_flag = threading.Event()
        self.results = queue.Queue()
        self._stats = {
            "tasks_processed": 0,
            "tasks_failed": 0,
            "avg_processing_time": 0.0
        }
        
        # Create worker threads
        for i in range(num_workers):
            worker = threading.Thread(
                target=self._worker_loop,
                name=f"Worker-{i}",
                daemon=True
            )
            worker.start()
            self.workers.append(worker)
    
    def submit(self, item: WorkItem):
        """Submit a work item to the queue."""
        self.task_queue.put(item)
    
    def submit_batch(self, items: List[WorkItem]):
        """Submit multiple work items."""
        for item in items:
            self.task_queue.put(item)
    
    def _worker_loop(self):
        """Worker thread main loop."""
        while not self.shutdown_flag.is_set():
            try:
                # Get task with timeout to allow shutdown check
                item = self.task_queue.get(timeout=0.1)
                
                start_time = time.time()
                try:
                    result = item.process()
                    self.results.put((item, result, None))
                    self._stats["tasks_processed"] += 1
                except Exception as e:
                    self.results.put((item, None, str(e)))
                    self._stats["tasks_failed"] += 1
                
                processing_time = time.time() - start_time
                # Update running average
                self._stats["avg_processing_time"] = (
                    0.9 * self._stats["avg_processing_time"] + 
                    0.1 * processing_time
                )
                
                self.task_queue.task_done()
                
            except queue.Empty:
                continue  # Timeout, check shutdown flag
    
    def shutdown(self, wait: bool = True):
        """Shutdown the thread pool."""
        self.shutdown_flag.set()
        if wait:
            for worker in self.workers:
                worker.join(timeout=2)
    
    def get_results(self, timeout: Optional[float] = None):
        """Get results from the queue."""
        results = []
        try:
            while True:
                result = self.results.get(timeout=timeout)
                results.append(result)
                self.results.task_done()
        except queue.Empty:
            pass
        return results
    
    def get_statistics(self):
        """Get pool statistics."""
        return self._stats.copy()

# Usage
pool = ThreadPoolWithPriority(num_workers=3)

# Submit work with different priorities
work_items = [
    WorkItem(Priority.HIGH, data="Critical system update"),
    WorkItem(Priority.LOW, data="Background cleanup"),
    WorkItem(Priority.NORMAL, data="User report generation"),
    WorkItem(Priority.HIGH, data="Payment processing"),
    WorkItem(Priority.NORMAL, data="Data sync"),
]

pool.submit_batch(work_items)

# Wait for completion
time.sleep(1)

# Get results
results = pool.get_results()
for item, result, error in results:
    if error:
        print(f"Error processing {item.data}: {error}")
    else:
        print(f"Success: {result}")

stats = pool.get_statistics()
print(f"\nPool statistics: {stats}")

pool.shutdown()
```

## 📊 Threading Anti-Patterns and Solutions

### Common Anti-Patterns:
```python
# ❌ ANTI-PATTERN 1: Creating too many threads
def bad_massive_threading():
    """Creating thousands of threads - terrible idea!"""
    threads = []
    for i in range(10000):  # ❌ DON'T DO THIS!
        t = threading.Thread(target=lambda: time.sleep(1))
        threads.append(t)
        t.start()

# ✅ SOLUTION: Use ThreadPoolExecutor with reasonable limits
def good_thread_pool():
    """Use thread pool with limited workers."""
    with ThreadPoolExecutor(max_workers=50) as executor:  # ✅ REASONABLE
        futures = [executor.submit(lambda: time.sleep(1)) 
                  for _ in range(10000)]
        # Executor manages thread reuse

# ❌ ANTI-PATTERN 2: Not handling exceptions in threads
def bad_unhandled_exceptions():
    """Exceptions in threads are silent by default."""
    def buggy_worker():
        raise ValueError("This will be lost!")
    
    t = threading.Thread(target=buggy_worker)
    t.start()
    t.join()  # Exception disappears!

# ✅ SOLUTION: Proper exception handling
def good_exception_handling():
    """Capture exceptions from threads."""
    def worker(result_holder):
        try:
            # Do work
            raise ValueError("Test error")
        except Exception as e:
            result_holder.append(e)
    
    exceptions = []
    t = threading.Thread(target=worker, args=(exceptions,))
    t.start()
    t.join()
    
    if exceptions:
        print(f"Thread failed: {exceptions[0]}")

# ❌ ANTI-PATTERN 3: Busy waiting
def bad_busy_wait():
    """Busy waiting wastes CPU."""
    ready = False
    
    def setter():
        nonlocal ready
        time.sleep(1)
        ready = True
    
    threading.Thread(target=setter).start()
    
    # ❌ Busy wait - wastes CPU
    while not ready:
        pass  # Spins CPU!

# ✅ SOLUTION: Use proper synchronization
def good_synchronization():
    """Use Condition or Event for waiting."""
    ready = threading.Event()
    
    def setter():
        time.sleep(1)
        ready.set()
    
    threading.Thread(target=setter).start()
    ready.wait()  # ✅ Efficient wait
```

## 🎯 Final Recommendations

### When to Use Threading:
1. **I/O-bound operations** (network, file, database calls)
2. **Background tasks** that don't need CPU
3. **GUI applications** (keep UI responsive)
4. **Web servers** handling multiple requests
5. **Any task that spends time waiting**

### When NOT to Use Threading:
1. **CPU-bound computations** (use multiprocessing)
2. **Simple sequential tasks**
3. **When shared state is complex** (consider async)
4. **When you need true parallelism** (use multiprocessing)

### Modern Python Threading Best Practices:
1. **Always use `concurrent.futures.ThreadPoolExecutor`** for new code
2. **Set reasonable `max_workers`** (CPU count × 2-5 for I/O)
3. **Use context managers** for resource cleanup
4. **Implement proper error handling** in worker functions
5. **Monitor thread health** in production
6. **Use immutable data** where possible
7. **Profile before optimizing** - threading has overhead

## 📚 Resources and Further Reading

1. **Official Python Documentation**: `threading`, `concurrent.futures`
2. **"Python Concurrency with asyncio"** by Matthew Fowler
3. **"High Performance Python"** by Micha Gorelick and Ian Ozsvald
4. **Real Python Threading Tutorials**: Comprehensive guides
5. **PyCon Talks**: Search for "Python threading" and "GIL" talks

---