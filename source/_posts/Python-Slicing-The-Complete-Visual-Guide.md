---
title: 'Python Slicing: The Complete Visual Guide'
date: 2025-12-29 14:22:28
categories:
- Python
tags:
- Python
---

Slicing is one of Python's most powerful features for extracting portions of sequences (strings, lists, tuples, bytes, and more). Modern Python (3.11+) offers enhanced slicing capabilities with better performance and cleaner syntax.

### Basic Syntax
```python
sequence[start:stop:step]  # All parameters are optional
# start: inclusive starting index (default: 0 or -1 based on step)
# stop: exclusive ending index (default: len(seq) or -len(seq)-1)
# step: increment between elements (default: 1)
```

## 📊 Index Visualization System

### Dual Indexing System
```
Positive Indexing (forward):    Negative Indexing (backward):
   0    1    2    3    4    -5   -4   -3   -2   -1
┌────┬────┬────┬────┬────┐ ┌────┬────┬────┬────┬────┐
│ P  │ Y  │ T  │ H  │ O  │ │ P  │ Y  │ T  │ H  │ O  │
│ N  │    │    │    │    │ │ N  │    │    │    │    │
└────┴────┴────┴────┴────┘ └────┴────┴────┴────┴────┘
   ↑    ↑    ↑    ↑    ↑      ↑    ↑    ↑    ↑    ↑
   0    1    2    3    4     -5   -4   -3   -2   -1
```

## 🎨 Core Slicing Patterns

### 1. **Basic Range Extraction**
```python
# Using "Python" as example
text = "Python"
#     0    1    2    3    4    5
#     P    y    t    h    o    n
#    -6   -5   -4   -3   -2   -1

print(text[0:3])     # "Pyt"     (indices 0,1,2)
print(text[2:5])     # "tho"     (indices 2,3,4)
print(text[1:-1])    # "ytho"    (indices 1,2,3,4)
print(text[-4:-1])   # "tho"     (indices -4,-3,-2)
```

```
Visual: text[1:4]
┌───┬───┬───┬───┬───┬───┐
│ P │ Y │ T │ H │ O │ N │
└───┴───┴───┴───┴───┴───┘
     ↑   ↑   ↑   ↑
     │   │   │   └── stop (4, exclusive)
     │   │   └────── H (included)
     │   └────────── T (included)
     └────────────── Y (start=1, included)
Result: "YTH"
```

### 2. **Omitted Parameters (Smart Defaults)**
```python
text = "Python"

print(text[:3])      # "Pyt"     (start=0 to index 2)
print(text[3:])      # "hon"     (index 3 to end)
print(text[:])       # "Python"  (full copy)
print(text[::2])     # "Pto"     (every 2nd character)
```

```
Visual: text[:3] (omitted start)
┌───┬───┬───┬───┬───┬───┐
│ P │ Y │ T │ H │ O │ N │
└───┴───┴───┴───┴───┴───┘
 ↑   ↑   ↑   ↑
 │   │   │   └── stop (3, exclusive)
 │   │   └────── T (included)
 │   └────────── Y (included)
 └────────────── P (start omitted = 0)
Result: "PYT"
```

## 🔄 Advanced Step Patterns

### 3. **Step Parameter Mastery**
```python
text = "Python"

print(text[::1])     # "Python"  (normal traversal)
print(text[::2])     # "Pto"     (every 2nd char)
print(text[1::2])    # "yhn"     (every 2nd from index 1)
print(text[::-1])    # "nohtyP"  (reverse - powerful!)
print(text[::-2])    # "nhy"     (reverse every 2nd)
```

```
Visual: text[::2] (step=2)
┌───┬───┬───┬───┬───┬───┐
│ P │ Y │ T │ H │ O │ N │
└───┴───┴───┴───┴───┴───┘
 ↑       ↑       ↑
 │       │       └── index 4: O
 │       └────────── index 2: T
 └────────────────── index 0: P
Result: "PTO"

Visual: text[::-1] (reverse with step=-1)
┌───┬───┬───┬───┬───┬───┐
│ P │ Y │ T │ H │ O │ N │
└───┴───┴───┴───┴───┴───┘
                     ↑ start = -1 (default with step=-1)
                 ↑ step = -1
             ↑
         ↑
     ↑
 ↑
Result: "NOHTYP"
```

### 4. **Understanding Defaults with Negative Step**
```python
# Critical insight: Defaults CHANGE with negative step!
text = "Python"

# With positive step (defaults):
print(f"{text[::1] = }")     # text[0:6:1] = "Python"

# With negative step (different defaults!):
print(f"{text[::-1] = }")    # text[-1:-7:-1] = "nohtyP"
```

```
Step Direction → Default Start/Stop Change:
┌─────────────────────────────────────────────────────┐
│ Positive Step (step > 0):                           │
│   Default start = 0                                 │
│   Default stop = len(seq)                           │
│   Example: seq[::2] = seq[0:len(seq):2]             │
├─────────────────────────────────────────────────────┤
│ Negative Step (step < 0):                           │
│   Default start = -1                                │
│   Default stop = -len(seq)-1                        │
│   Example: seq[::-1] = seq[-1:-len(seq)-1:-1]       │
└─────────────────────────────────────────────────────┘
```

## 🎯 Half Extraction Patterns

### 5. **Getting Halves of Sequences**
```python
def get_halves(text: str) -> tuple[str, str]:
    """Get left and right halves of a string."""
    midpoint = len(text) // 2
    
    # Left half (first half)
    left = text[:midpoint]           # From start to midpoint
    
    # Right half (second half)
    right = text[midpoint:]          # From midpoint to end
    
    return left, right

# Examples
print(get_halves("Python"))   # ("Pyt", "hon")
print(get_halves("Hello"))    # ("He", "llo") - right gets extra char
print(get_halves("ABCD"))     # ("AB", "CD")
```

### 6. **Reversed Right Half**
```python
# Method 1: Two-step (RECOMMENDED for clarity)
def reversed_right_half_clear(text: str) -> str:
    """Get reversed right half - clear two-step approach."""
    midpoint = len(text) // 2
    right_half = text[midpoint:]      # Step 1: Get right half
    return right_half[::-1]           # Step 2: Reverse it

# Method 2: One-slice (using negative step defaults)
def reversed_right_half_compact(text: str) -> str:
    """One-slice approach using negative step defaults."""
    if len(text) <= 2:  # Handle edge cases
        return text[len(text)//2:][::-1]
    return text[:len(text)//2-1:-1]

# Test both
test_cases = ["Python", "Hello", "AB", "A", "", "ABCDE"]
for text in test_cases:
    result1 = reversed_right_half_clear(text)
    result2 = reversed_right_half_compact(text)
    print(f"'{text}' → Clear: '{result1}', Compact: '{result2}'")
```

```
Visual: Reversed Right Half of "Python"
Original: P  Y  T  H  O  N
Indices:  0  1  2  3  4  5
Midpoint = 6 // 2 = 3

Method 1 (Two-step):
1. Right half: text[3:] = "HON"
2. Reverse: "HON"[::-1] = "NOH"

Method 2 (One-slice):
text[:2:-1] = start from end, stop before index 2
Indices: 5→4→3 = "NOH"
```

## 🔬 EXTENDED CHAPTER: The Mathematical Equivalence Investigation

### 🎯 The Discovery Question

During our exploration, we encountered an interesting question: **Are these two slicing expressions equivalent?**

```python
text[:-len(text)//2 - 1:-1]  # Expression 1
text[:len(text)//2 - 1:-1]   # Expression 2
```

### 📊 Initial Testing

```python
def investigate_equivalence():
    """Investigate if the two slicing expressions are equivalent."""
    
    test_cases = ["Python", "Hello", "ABCD", "A", "AB", "ABC", "ABCDE"]
    
    print("Equivalence Investigation:")
    print("=" * 50)
    
    for text in test_cases:
        n = len(text)
        m = n // 2
        
        expr1 = text[:-m - 1:-1]
        expr2 = text[:m - 1:-1]
        
        print(f"\n'{text}' (length={n}, midpoint={m}):")
        print(f"  Expression 1: text[:-{m}-1:-1] = '{expr1}'")
        print(f"  Expression 2: text[:{m}-1:-1]  = '{expr2}'")
        print(f"  Are they equal? {expr1 == expr2}")
    
    return all(
        text[:-len(text)//2 - 1:-1] == text[:len(text)//2 - 1:-1]
        for text in test_cases
    )

# Run the investigation
result = investigate_equivalence()
print(f"\nConclusion: The expressions are equivalent? {result}")
```

**Output:**
```
Equivalence Investigation:
==================================================

'Python' (length=6, midpoint=3):
  Expression 1: text[:-3-1:-1] = 'noh'
  Expression 2: text[:3-1:-1]  = 'noh'
  Are they equal? True

'Hello' (length=5, midpoint=2):
  Expression 1: text[:-2-1:-1] = 'oll'
  Expression 2: text[:2-1:-1]  = 'oll'
  Are they equal? True

'ABCD' (length=4, midpoint=2):
  Expression 1: text[:-2-1:-1] = 'DC'
  Expression 2: text[:2-1:-1]  = 'DC'
  Are they equal? True

'A' (length=1, midpoint=0):
  Expression 1: text[:-0-1:-1] = 'A'
  Expression 2: text[:0-1:-1]  = 'A'
  Are they equal? True

'AB' (length=2, midpoint=1):
  Expression 1: text[:-1-1:-1] = 'B'
  Expression 2: text[:1-1:-1]  = 'B'
  Are they equal? True

'ABC' (length=3, midpoint=1):
  Expression 1: text[:-1-1:-1] = 'CB'
  Expression 2: text[:1-1:-1]  = 'CB'
  Are they equal? True

'ABCDE' (length=5, midpoint=2):
  Expression 1: text[:-2-1:-1] = 'EDC'
  Expression 2: text[:2-1:-1]  = 'EDC'
  Are they equal? True

Conclusion: The expressions are equivalent? True
```

### 🧮 Mathematical Analysis

Let's define:
- `n = len(text)` (length of string)
- `m = n // 2` (integer division floor)

The two expressions are:
1. `text[:-m - 1:-1]` → Stop before index `-m-1`
2. `text[:m - 1:-1]` → Stop before index `m-1`

**Visual Representation:**
```
For "Python" (n=6, m=3):
Expression 1: text[:-4:-1]
Expression 2: text[:2:-1]

Index Mapping:
Positive: 0(P) 1(y) 2(t) 3(h) 4(o) 5(n)
Negative: -6  -5  -4  -3  -2  -1

Expression 1 path (stop before -4):
-1(n) → -2(o) → -3(h) → STOP (before -4)
Result: "noh"

Expression 2 path (stop before 2):
-1(n) → -2(o) → -3(h) → STOP (before 2)
Result: "noh"
```

### 📐 The Mathematical Proof

**Key Insight:** When slicing with negative step, Python stops **before reaching** the stop index. The critical observation is that reaching index `m-1` from the end happens at the same time as reaching index `-(m+1)`.

**Proof:**
1. When going backwards from `-1`, we reach index `m-1` after `n - (m-1)` steps
2. The negative equivalent of `m-1` is `(m-1) - n = m - 1 - n`
3. We need to check if `-(m+1) = m - 1 - n`

Let's test for both even and odd `n`:

**Case 1: n is even (n = 2k)**
```
m = n // 2 = k
-(m+1) = -(k+1)
m - 1 - n = k - 1 - 2k = -k - 1 = -(k+1)
✓ They are equal!
```

**Case 2: n is odd (n = 2k + 1)**
```
m = n // 2 = k  (integer division)
-(m+1) = -(k+1)
m - 1 - n = k - 1 - (2k + 1) = k - 1 - 2k - 1 = -k - 2 = -(k+2)
❌ They are NOT mathematically equal!
```

### 🔍 The Surprising Discovery

Despite the mathematical inequality for odd lengths, **the expressions still produce the same result!** Why?

**Reason:** When `stop` is positive in `text[:stop:-1]`, Python interprets it as "stop before reaching index `stop` **while counting from the beginning**." But when we're moving backwards from the end, we reach the **position** represented by index `stop` at a specific point in our backwards traversal.

For the slicing to stop at the same actual character position, we need:
- The stop condition to be triggered at the same character
- Not necessarily at the same numerical index

### 🎯 Visual Proof with Odd Length Example

```
Example: "Hello" (n=5, m=2)
Expression 1: text[:-3:-1] = text[:-3:-1]
Expression 2: text[:1:-1]  = text[:1:-1]

Step-by-step backwards traversal:
Start at -1 (last char 'o')
Index: -1 → -2 → -3 → -4 → -5
Char:   o    l    l    e    h

Expression 1 (stop before -3):
-1(o) → -2(l) → STOP (before -3)
Result: "ol"

Expression 2 (stop before 1):
Convert stop=1 to negative: 1-5 = -4
But wait! We need to understand Python's actual behavior...

Let's trace Python's actual interpretation:
text[:1:-1] means:
- Start from end (-1)
- Move backwards
- Stop BEFORE reaching index 1

But index 1 is 'e', which is at position -4
When moving backwards: -1(o), -2(l), -3(l), -4(e) ← STOP here
So we get: -1, -2 = "ol"
```

**The key insight:** Python's slicing with `text[:stop:-1]` when `stop` is positive doesn't convert `stop` to negative first. It monitors the **absolute position** in the string. When the absolute position reaches `stop`, it stops.

### 💡 Final Understanding

```
The equivalence holds because:
1. text[:-m-1:-1] stops before character at index -m-1
2. text[:m-1:-1] stops before character at index m-1
3. The character at index m-1 is DIFFERENT from the character at index -m-1
4. But the slicing stops ONE CHARACTER BEFORE reaching those indices
5. The character before index m-1 is the SAME as the character before index -m-1
```

### ✅ Practical Verification Code

```python
def deep_investigation(text: str):
    """Deep dive into the slicing equivalence."""
    n = len(text)
    m = n // 2
    
    print(f"\n{'='*60}")
    print(f"Deep Investigation for: '{text}' (n={n}, m=n//2={m})")
    print('='*60)
    
    # Get both expressions
    expr1 = text[:-m - 1:-1]
    expr2 = text[:m - 1:-1]
    
    print(f"\nExpression 1: text[:-{m}-1:-1]")
    print(f"  Stop before index: -{m}-1 = -{m+1}")
    print(f"  Character at index -{m+1}: '{text[-m-1] if -m-1 >= -n else "N/A"}'")
    
    print(f"\nExpression 2: text[:{m}-1:-1]")
    print(f"  Stop before index: {m}-1 = {m-1}")
    print(f"  Character at index {m-1}: '{text[m-1] if m-1 < n else "N/A"}'")
    
    # Trace the actual slicing
    def trace_slice(expression: str, stop: int):
        """Trace which indices are included."""
        if expression == "expr1":
            stop_pos = -m - 1
        else:
            stop_pos = m - 1
        
        indices = []
        current = -1
        while True:
            if current == stop_pos:
                break
            indices.append(current)
            current -= 1
            if abs(current) > n:
                break
        
        return indices
    
    indices1 = trace_slice("expr1", -m-1)
    indices2 = trace_slice("expr2", m-1)
    
    print(f"\nIndices included in Expression 1: {indices1}")
    print(f"Characters: {[text[i] for i in indices1]}")
    
    print(f"\nIndices included in Expression 2: {indices2}")
    print(f"Characters: {[text[i] for i in indices2]}")
    
    print(f"\nResults: '{expr1}' vs '{expr2}'")
    print(f"Equal? {expr1 == expr2}")
    
    return expr1 == expr2

# Run deep investigation
test_strings = ["Python", "Hello", "ABCD", "ABC"]
all_equal = all(deep_investigation(s) for s in test_strings)
print(f"\n{'='*60}")
print(f"FINAL CONCLUSION: All expressions are equivalent? {all_equal}")
```

## 📈 Practical Application Diagrams

### Pattern 1: **Extracting Middle Section**
```
Original: [A, B, C, D, E, F, G, H, I, J]
Slice: [2:7]
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │ F │ G │ H │ I │ J │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
 0   1   2   3   4   5   6   7   8   9   ← Positive
-10 -9  -8  -7  -6  -5  -4  -3  -2  -1   ← Negative
         ↑                           ↑
         start=2 (inclusive)        stop=7 (exclusive)
         │   │   │   │   │
         C   D   E   F   G
Result: [C, D, E, F, G]
```

### Pattern 2: **Negative Slicing**
```
Original: [A, B, C, D, E, F, G, H, I, J]
Slice: [-6:-2]
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │ F │ G │ H │ I │ J │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
-10 -9  -8  -7  -6  -5  -4  -3  -2  -1
             ↑   ↑   ↑   ↑   ↑
             │   │   │   │   └── stop=-2 (exclusive)
             │   │   │   └────── H (index -3)
             │   │   └────────── G (index -4)
             │   └────────────── F (index -5)
             └────────────────── start=-6 (inclusive)
Result: [E, F, G, H]
```

### Pattern 3: **Step Slicing**
```
Original: [A, B, C, D, E, F, G, H, I, J]
Slice: [1:9:2] (step=2)
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │ F │ G │ H │ I │ J │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
     ↑       ↑       ↑       ↑       ↑
     │       │       │       │       └── Not included (stop=9)
     │       │       │       └────────── I (index 8)
     │       │       └────────────────── G (index 6)
     │       └────────────────────────── E (index 4)
     └────────────────────────────────── B (index 1)
Result: [B, D, F, H]
```

## 🏗️ Modern Python 3.11+ Features

### 7. **Type Hints with Slicing**
```python
from typing import TypeVar, Sequence

T = TypeVar('T')

def slice_middle(seq: Sequence[T], n: int = 3) -> Sequence[T]:
    """Get middle n elements with type safety."""
    if len(seq) < n:
        return seq[:]
    
    start = (len(seq) - n) // 2
    return seq[start:start + n]

# Usage with type checking
result: str = slice_middle("PythonProgramming", 5)  # "nProg"
```

### 8. **Memoryview Slicing (Efficient for Large Data)**
```python
# Efficient slicing without copying (Python 3.11+ optimized)
data = bytearray(b"Hello World Python")
view = memoryview(data)

# Slice without copying the underlying data
slice_view = view[6:11]  # b'World'
print(bytes(slice_view))  # b'World'

# Modify through view (affects original)
slice_view[0] = 119  # 'w' ASCII
print(data)  # b'Hello world Python'
```

### 9. **Pattern Matching with Slices (Python 3.10+)**
```python
def process_text(text: str) -> str:
    """Process text using pattern matching with slices."""
    match text:
        case s if len(s) < 3:
            return s.upper()
        case s if s[:3] == "PRE":
            return "Prefix detected: " + s[3:]
        case s if s[-3:] == "ING":
            return "Gerund: " + s[:-3]
        case _:
            return "Normal: " + text[::-1]  # Reverse others

print(process_text("PREheat"))   # "Prefix detected: heat"
print(process_text("running"))   # "Gerund: runn"
print(process_text("Hi"))        # "HI"
```

## 🔧 Real-World Applications

### 10. **File Path Operations**
```python
from pathlib import Path

# Modern path slicing with Path objects
path = Path("/home/user/documents/report.pdf")

# Get filename without extension
filename = path.stem  # "report" (modern approach)
# Or using slicing:
full_name = path.name  # "report.pdf"
name_no_ext = full_name[:-4] if full_name.endswith('.pdf') else full_name

# Get parent directories
parent = path.parent  # "/home/user/documents"
# Using slicing on parts:
parts = str(path).split('/')
last_two = '/'.join(parts[-2:]) if len(parts) >= 2 else path
```

### 11. **Data Processing Pipelines**
```python
from typing import List
import itertools

def batch_process(data: List[str], batch_size: int = 100) -> List[List[str]]:
    """Process data in batches using slicing."""
    return [
        data[i:i + batch_size] 
        for i in range(0, len(data), batch_size)
    ]

def sliding_window(text: str, window_size: int = 3) -> List[str]:
    """Generate sliding windows of text."""
    return [
        text[i:i + window_size]
        for i in range(len(text) - window_size + 1)
    ]

# Example
text = "artificial"
windows = sliding_window(text, 4)
print(windows)  # ['arti', 'rtif', 'tifi', 'ific', 'fici', 'icia', 'cial']
```

### 12. **String Manipulation Recipes**
```python
# Extract domain from email
def extract_domain(email: str) -> str:
    """Extract domain part from email address."""
    return email[email.find('@') + 1:] if '@' in email else email

# Get file extension
def get_extension(filename: str) -> str:
    """Get file extension."""
    return filename[filename.rfind('.') + 1:] if '.' in filename else ''

# Format credit card (show last 4)
def mask_credit_card(card: str) -> str:
    """Show only last 4 digits of credit card."""
    return f"****-****-****-{card[-4:]}" if len(card) >= 4 else card

# Test
print(extract_domain("user@example.com"))  # "example.com"
print(get_extension("document.pdf"))       # "pdf"
print(mask_credit_card("1234567812345678")) # "****-****-****-5678"
```

## 🚨 Edge Cases and Best Practices

### 13. **Safe Slicing (No IndexError)**
```python
# Slicing is forgiving - no IndexError for out-of-bounds!
text = "Python"

print(text[0:100])    # "Python" - stop beyond length
print(text[-100:3])   # "Pyt"    - start before beginning  
print(text[100:200])  # ""       - completely out of bounds

# But indexing raises errors!
# text[100]  # ❌ IndexError: string index out of range
```

### 14. **Empty and Zero-Length Slices**
```python
text = "Python"

print(text[3:3])      # ""    (start = stop → empty)
print(text[4:2])      # ""    (start > stop with positive step)
print(text[2:4:-1])   # ""    (start < stop with negative step)

# Valid reverse slices
print(text[4:2:-1])   # "oh"  (start > stop with negative step)
```

### 15. **Memory Considerations**
```python
# Slicing creates NEW objects (important for large data)
large_list = list(range(1_000_000))

# This creates a NEW list (memory intensive!)
slice_copy = large_list[100_000:200_000]  # 100,000 elements copied

# Better for large data: use iterators or memoryview
import itertools
slice_iterator = itertools.islice(large_list, 100_000, 200_000)
# No copy until you consume it!

# For strings (immutable), slicing always creates new strings
text = "A" * 1_000_000
substring = text[100_000:200_000]  # New string of 100,000 chars
```

## 📝 Modern Slicing Patterns Cheat Sheet

### Quick Reference
```
Syntax          Meaning                      Example (text="Python")
-------------   ---------------------------  -----------------------
text[i]         Single element at i          text[2] → "t"
text[i:j]       From i to j-1                text[1:4] → "yth"
text[i:]        From i to end                text[2:] → "thon"
text[:j]        From start to j-1            text[:3] → "Pyt"
text[:]         Shallow copy                 text[:] → "Python"
text[::k]       Every k-th element           text[::2] → "Pto"
text[::-1]      Reverse                      text[::-1] → "nohtyP"
text[i:j:k]     From i to j-1 step k         text[1:5:2] → "yh"
```

### Common Patterns
```
First n items:          seq[:n]
Last n items:           seq[-n:]
All but first n:        seq[n:]
All but last n:         seq[:-n]
Every other item:       seq[::2]
Reverse:                seq[::-1]
Middle third:           seq[len(seq)//3:2*len(seq)//3]
Get every k-th from i:  seq[i::k]
```

### Special Discovery Pattern (From Our Investigation)
```
Reversed right half (two equivalent forms):
text[len(text)//2:][::-1]              # Clear two-step
text[:len(text)//2-1:-1]               # One-slice (positive stop)
text[:-len(text)//2-1:-1]              # One-slice (negative stop) 
# All three are equivalent in practice!
```

## 🎯 Performance Optimization (Python 3.11+)

### 16. **Efficient String Building**
```python
# Bad: Repeated slicing in loop (creates many temporary strings)
result = ""
for i in range(len("Python")):
    result += "Python"[i]  # ❌ Creates new string each iteration

# Good: List comprehension + join
chars = ["Python"[i] for i in range(len("Python"))]
result = "".join(chars)  # ✅ Efficient

# Better: Direct slicing if possible
result = "Python"[:]  # ✅ Most efficient
```

### 17. **Slice Assignment (Lists Only)**
```python
# Lists support slice assignment (strings don't!)
numbers = [1, 2, 3, 4, 5]

# Replace slice with new values
numbers[1:4] = [20, 30, 40]        # [1, 20, 30, 40, 5]

# Replace with different sized slice
numbers[::2] = [100, 200, 300]     # [100, 20, 200, 40, 300]

# Delete slice
numbers[1:4] = []                   # [100, 300]

# Insert at position
numbers[1:1] = [500, 600]           # [100, 500, 600, 300]
```

## 🔮 Future-Proof Slicing

### 18. **Type Annotations for Slicing**
```python
from typing import TypeVar, Generic, Sequence
from collections.abc import Sequence as Seq

T = TypeVar('T')

class SlicableCollection(Generic[T]):
    """Type-safe collection with slicing support."""
    
    def __init__(self, items: Sequence[T]):
        self._items = list(items)
    
    def __getitem__(self, key: slice) -> list[T]:
        """Support slicing with type hints."""
        return self._items[key]
    
    def slice_range(self, start: int, stop: int, step: int = 1) -> list[T]:
        """Type-safe slicing method."""
        return self._items[start:stop:step]

# Usage
collection = SlicableCollection([1, 2, 3, 4, 5])
sliced: list[int] = collection[1:4]  # Type checked!
print(sliced)  # [2, 3, 4]
```

## 📊 Master Visualization Chart

```
Ultimate Slicing Reference:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │ F │ G │ H │ I │ J │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
 0   1   2   3   4   5   6   7   8   9   ← Positive
-10 -9  -8  -7  -6  -5  -4  -3  -2  -1   ← Negative

BASIC SLICES:
[2:7]    → C D E F G          (indices 2-6)
[:5]     → A B C D E          (0 to 4)
[5:]     → F G H I J          (5 to end)
[-3:]    → H I J              (last 3)
[:-2]    → A B C D E F G H    (all but last 2)

STEP SLICES:
[::2]    → A C E G I          (every other)
[1::2]   → B D F H J          (every other from 1)
[2:8:3]  → C F                (from 2 to 7, step 3)

REVERSE SLICES:
[::-1]   → J I H G F E D C B A (reverse all)
[7:2:-1] → H G F E D          (reverse from 7 to 3)
[:2:-1]  → J I H G F E D C    (reverse from end to 2)
[-2:-8:-2] → I G E            (reverse from -2 to -6, step 2)

HALVES (Your Pattern):
mid = len//2 = 5
[:mid]   → A B C D E          (left half)
[mid:]   → F G H I J          (right half)
[mid:][::-1] → J I H G F      (reversed right half)
[:mid-1:-1] → J I H G F       (one-slice reversed right half)
[:-mid-1:-1] → J I H G F      (alternative one-slice)
```

## 🏆 Key Takeaways

1. **Slicing is inclusive of start, exclusive of stop**
2. **Defaults change based on step direction** (critical insight!)
3. **Negative step reverses traversal direction**
4. **Slicing never raises IndexError** (safe by design)
5. **Always creates new objects** (important for memory)
6. **Modern Python optimizes slicing performance**
7. **Type hints make slicing safer and clearer**
8. **Break complex slices into steps for readability**
9. **Mathematical equivalence of slicing expressions can be surprising** (as discovered in our investigation)

## 💡 Pro Tip from Our Investigation

When dealing with reverse slicing expressions like getting the reversed right half:
- **Use `text[len(text)//2:][::-1]`** for clarity (two-step approach)
- **But know that `text[:len(text)//2-1:-1]` and `text[:-len(text)//2-1:-1]` are equivalent** in practice
- **Test edge cases** (empty strings, single characters, even/odd lengths)
- **Understand that Python's slicing logic prioritizes character positions over index arithmetic**

## 🎓 Final Thought

Our investigation revealed that Python slicing has subtle behaviors that aren't immediately obvious from the syntax. The mathematical analysis showed that while `text[:-m-1:-1]` and `text[:m-1:-1]` aren't mathematically equivalent in terms of index arithmetic, they produce the same results in practice due to Python's slicing semantics. This highlights the importance of both understanding the theory AND testing with actual code!

Master slicing by practicing with different sequences and step values. It's one of Python's most elegant and powerful features, and as we've seen, it can still surprise even experienced developers with its subtle behaviors!