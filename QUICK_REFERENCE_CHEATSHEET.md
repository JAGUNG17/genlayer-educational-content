# GenLayer Intelligent Contracts: Quick Reference Cheat Sheet

A fast reference guide for common patterns and syntax.

## Contract Structure

```python
from genlayer import *

class MyContract(gl.Contract):
    # State variables
    data: dict[str, int]
    
    def __init__(self):
        self.data = {}
    
    # Public write method (modifies state)
    @gl.public.write
    def set_value(self, key: str, value: int):
        self.data[key] = value
    
    # Public view method (read-only)
    @gl.public.view
    def get_value(self, key: str) -> int:
        return self.data.get(key, 0)
    
    # Private method
    def _private_helper(self):
        pass
```

---

## Method Decorators

| Decorator | Purpose | Modifies State | Payable |
|-----------|---------|----------------|---------|
| `@gl.public.view` | Read-only access | No | No |
| `@gl.public.write` | Modify state | Yes | No |
| `@gl.public.write.payable` | Modify state + receive value | Yes | Yes |

---

## Web Data Access

### Fetch JSON Data
```python
data = gl.nondet.web.get('https://api.example.com/data')
```

### Render HTML
```python
html = gl.nondet.web.render('https://example.com', mode='html')
```

### Check Website Content
```python
content = gl.nondet.web.get('https://example.com')
if 'search_text' in content:
    # Content found
    pass
```

### Parse JSON Response
```python
response = gl.nondet.web.get('https://api.example.com/data')
data = json.loads(response)
value = data.get('key', 'default')
```

---

## LLM Integration

### Basic LLM Call
```python
response = gl.eq_principle.prompt_non_comparative(
    lambda: "Your prompt here",
    task="Task description",
    criteria="Validation criteria"
)
```

### LLM with Context
```python
def prompt_func():
    data = gl.nondet.web.get('https://api.example.com')
    return f"Analyze this data: {data}"

response = gl.eq_principle.prompt_non_comparative(
    prompt_func,
    task="Data analysis",
    criteria="Response must classify as A, B, or C"
)
```

### Parse LLM Response
```python
response = gl.eq_principle.prompt_non_comparative(...)
lines = response.split('\n')
for line in lines:
    if 'KEY:' in line:
        value = line.split('KEY:')[1].strip()
```

### Deterministic Check
```python
result = gl.eq_principle.strict_eq(
    lambda: check_condition()
)
```

---

## State Management

### Store Simple Value
```python
self.count = 0
self.name = "example"
self.active = True
```

### Store Dictionary
```python
self.data = {}
self.data['key'] = 'value'
```

### Store List
```python
self.items = []
self.items.append({'id': 1, 'name': 'item'})
```

### Store Nested Structure
```python
self.records = {
    'user1': {
        'balance': 100,
        'timestamp': gl.block.timestamp,
        'active': True
    }
}
```

### Update State
```python
# Direct assignment
self.count = 10

# Increment
self.count += 1

# Dictionary update
self.data['key'] = new_value

# List append
self.items.append(new_item)
```

---

## Block & Transaction Info

```python
# Current block timestamp
timestamp = gl.block.timestamp

# Transaction sender
sender = msg.sender

# Transaction value (for payable methods)
amount = msg.value

# Block number
block_num = gl.block.number
```

---

## Error Handling

### Try-Except
```python
try:
    data = gl.nondet.web.get(url)
except Exception as e:
    data = default_data
```

### Assertions
```python
assert condition, "Error message"
assert msg.sender == self.admin, "Only admin allowed"
```

### Conditional Logic
```python
if condition:
    # Do something
    pass
elif other_condition:
    # Do something else
    pass
else:
    # Default behavior
    pass
```

---

## Common Patterns

### Counter
```python
class CounterContract(gl.Contract):
    count: int
    
    def __init__(self):
        self.count = 0
    
    @gl.public.write
    def increment(self):
        self.count += 1
    
    @gl.public.view
    def get_count(self) -> int:
        return self.count
```

### Key-Value Store
```python
class KVStoreContract(gl.Contract):
    store: dict[str, str]
    
    def __init__(self):
        self.store = {}
    
    @gl.public.write
    def set(self, key: str, value: str):
        self.store[key] = value
    
    @gl.public.view
    def get(self, key: str) -> str:
        return self.store.get(key, "")
```

### Approval Tracking
```python
class ApprovalContract(gl.Contract):
    approvals: dict[str, list]
    
    @gl.public.write
    def approve(self, item_id: str):
        if item_id not in self.approvals:
            self.approvals[item_id] = []
        self.approvals[item_id].append(msg.sender)
    
    @gl.public.view
    def get_approvals(self, item_id: str) -> list:
        return self.approvals.get(item_id, [])
```

### Caching
```python
class CachedContract(gl.Contract):
    cache: dict[str, dict]
    cache_ttl: int
    
    def __init__(self):
        self.cache = {}
        self.cache_ttl = 3600
    
    def get_cached(self, key: str):
        if key in self.cache:
            cached = self.cache[key]
            if gl.block.timestamp - cached['time'] < self.cache_ttl:
                return cached['value']
        return None
    
    def set_cache(self, key: str, value):
        self.cache[key] = {
            'value': value,
            'time': gl.block.timestamp
        }
```

---

## Data Types

```python
# Integer
count: int = 0

# String
name: str = "example"

# Boolean
active: bool = True

# Float
price: float = 99.99

# List
items: list[str] = []

# Dictionary
data: dict[str, int] = {}

# Optional
value: int | None = None
```

---

## String Operations

```python
# Concatenation
result = "Hello " + "World"

# String methods
text = "example"
upper = text.upper()  # "EXAMPLE"
lower = text.lower()  # "example"
length = len(text)    # 7

# String formatting
formatted = f"Value: {value}"
formatted = "Value: {}".format(value)

# String splitting
parts = "a,b,c".split(',')  # ['a', 'b', 'c']

# String contains
if 'search' in text:
    pass

# String replacement
result = text.replace('old', 'new')
```

---

## List Operations

```python
# Create list
items = []
items = [1, 2, 3]

# Append
items.append(4)

# Length
count = len(items)

# Access
first = items[0]
last = items[-1]

# Slice
subset = items[1:3]

# Iterate
for item in items:
    print(item)

# Check membership
if 1 in items:
    pass
```

---

## Dictionary Operations

```python
# Create dictionary
data = {}
data = {'key': 'value'}

# Set value
data['key'] = 'value'

# Get value
value = data.get('key', 'default')

# Check key exists
if 'key' in data:
    pass

# Iterate
for key, value in data.items():
    print(key, value)

# Keys and values
keys = list(data.keys())
values = list(data.values())
```

---

## Comparison & Logic

```python
# Comparison
a == b      # Equal
a != b      # Not equal
a > b       # Greater than
a < b       # Less than
a >= b      # Greater or equal
a <= b      # Less or equal

# Logic
a and b     # Both true
a or b      # Either true
not a       # Opposite

# Ternary
result = value_if_true if condition else value_if_false
```

---

## Type Conversions

```python
# String to integer
num = int("123")

# String to float
decimal = float("3.14")

# Integer to string
text = str(123)

# String to boolean
flag = bool("true")

# List to string
text = ",".join(["a", "b", "c"])

# String to list
items = "a,b,c".split(",")
```

---

## JSON Operations

```python
import json

# Parse JSON
data = json.loads('{"key": "value"}')

# Serialize to JSON
json_str = json.dumps(data)

# Access nested data
value = data.get('key', {}).get('nested', 'default')
```

---

## Common Validations

```python
# Check not empty
assert len(text) > 0, "Text cannot be empty"

# Check range
assert 0 <= value <= 100, "Value out of range"

# Check type
assert isinstance(value, int), "Must be integer"

# Check sender
assert msg.sender == self.admin, "Only admin"

# Check state
assert self.initialized, "Not initialized"

# Check amount
assert msg.value > 0, "Must send value"
```

---

## Debugging Tips

### Print Values
```python
# Use logging or state storage
self.debug_log = f"Value: {value}"
```

### Store Intermediate Results
```python
self.last_result = result
self.last_error = error_message
```

### Validate Assumptions
```python
assert condition, "Assumption violated"
```

### Check State Consistency
```python
@gl.public.view
def get_state_info(self) -> dict:
    return {
        'count': len(self.items),
        'total': self.total,
        'timestamp': gl.block.timestamp
    }
```

---

## Performance Tips

### Cache Results
```python
if key not in self.cache:
    self.cache[key] = expensive_operation()
return self.cache[key]
```

### Batch Operations
```python
for item in items:
    self.process(item)
```

### Avoid Nested Loops
```python
# ❌ Slow: O(n²)
for i in items1:
    for j in items2:
        if i == j:
            pass

# ✅ Fast: O(n)
set1 = set(items1)
for j in items2:
    if j in set1:
        pass
```

### Limit Data Size
```python
# Paginate large results
def get_items(self, page: int, size: int = 10):
    start = page * size
    return self.items[start:start + size]
```

---

## Security Checklist

- [ ] Validate all inputs
- [ ] Check sender permissions
- [ ] Verify external data
- [ ] Use safe defaults
- [ ] Handle errors gracefully
- [ ] Test edge cases
- [ ] Audit before deployment
- [ ] Monitor after deployment

---

## Useful Links

- **Docs:** https://docs.genlayer.com
- **API Reference:** https://docs.genlayer.com/api
- **Examples:** https://github.com/genlayer/examples
- **Community:** https://talks.genlayer.foundation

---

**Last Updated:** March 2026  
**Author:** kuru  
**Version:** 1.0
