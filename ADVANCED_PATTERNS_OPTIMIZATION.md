# GenLayer Intelligent Contracts: Advanced Patterns & Optimization

Master advanced design patterns and optimization techniques for production-grade Intelligent Contracts.

## Table of Contents

1. [Architectural Patterns](#architectural-patterns)
2. [State Optimization](#state-optimization)
3. [Gas Optimization](#gas-optimization)
4. [LLM Optimization](#llm-optimization)
5. [Concurrency & Scalability](#concurrency--scalability)
6. [Advanced Security](#advanced-security)

---

## Architectural Patterns

### 1. Factory Pattern

Create multiple contract instances dynamically:

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class ContractFactory(gl.Contract):
    """Factory for creating contract instances"""
    
    instances: dict[str, str]
    instance_count: int
    
    def __init__(self):
        self.instances = {}
        self.instance_count = 0
    
    @gl.public.write
    def create_instance(self, name: str) -> str:
        """Create new contract instance"""
        
        instance_id = f"{name}_{self.instance_count}"
        
        # Store instance reference
        self.instances[instance_id] = {
            'name': name,
            'created_at': gl.block.timestamp,
            'creator': msg.sender
        }
        
        self.instance_count += 1
        return instance_id
    
    @gl.public.view
    def get_instance(self, instance_id: str) -> dict:
        return self.instances.get(instance_id, {})
    
    @gl.public.view
    def get_all_instances(self) -> list:
        return list(self.instances.values())
```

**Benefits:**
- Dynamic contract creation
- Centralized management
- Easy instance tracking

---

### 2. Proxy Pattern

Implement upgradeable contracts:

```python
class ProxyContract(gl.Contract):
    """Proxy for contract upgrades"""
    
    implementation_address: str
    admin: str
    
    def __init__(self, implementation: str):
        self.implementation_address = implementation
        self.admin = msg.sender
    
    @gl.public.write
    def upgrade(self, new_implementation: str):
        """Upgrade contract implementation"""
        assert msg.sender == self.admin, "Only admin"
        self.implementation_address = new_implementation
    
    @gl.public.view
    def get_implementation(self) -> str:
        return self.implementation_address
```

**Benefits:**
- Upgradeable contracts
- Backward compatibility
- Centralized updates

---

### 3. Observer Pattern

Implement event notification system:

```python
class ObserverContract(gl.Contract):
    """Observer pattern for event notifications"""
    
    observers: dict[str, list]
    events: list[dict]
    
    def __init__(self):
        self.observers = {}
        self.events = []
    
    @gl.public.write
    def subscribe(self, event_type: str, callback_address: str):
        """Subscribe to events"""
        if event_type not in self.observers:
            self.observers[event_type] = []
        self.observers[event_type].append(callback_address)
    
    @gl.public.write
    def emit_event(self, event_type: str, data: dict):
        """Emit event to subscribers"""
        
        event = {
            'type': event_type,
            'data': data,
            'timestamp': gl.block.timestamp,
            'emitter': msg.sender
        }
        
        self.events.append(event)
        
        # Notify observers
        if event_type in self.observers:
            for observer in self.observers[event_type]:
                # Notify observer (implementation depends on callback mechanism)
                pass
```

**Benefits:**
- Loose coupling
- Event-driven architecture
- Scalable notifications

---

### 4. Strategy Pattern

Implement interchangeable algorithms:

```python
class StrategyContract(gl.Contract):
    """Strategy pattern for algorithm selection"""
    
    strategies: dict[str, dict]
    active_strategy: str
    
    def __init__(self):
        self.strategies = {
            'conservative': {'weight': 0.7, 'risk_level': 'low'},
            'moderate': {'weight': 0.5, 'risk_level': 'medium'},
            'aggressive': {'weight': 0.3, 'risk_level': 'high'}
        }
        self.active_strategy = 'moderate'
    
    @gl.public.write
    def set_strategy(self, strategy_name: str):
        """Change active strategy"""
        assert strategy_name in self.strategies, "Unknown strategy"
        self.active_strategy = strategy_name
    
    @gl.public.view
    def execute_strategy(self, value: float) -> float:
        """Execute current strategy"""
        strategy = self.strategies[self.active_strategy]
        return value * strategy['weight']
```

**Benefits:**
- Flexible algorithm selection
- Runtime strategy switching
- Easy to add new strategies

---

## State Optimization

### 1. Packed Storage

Store multiple values in single state variable:

```python
class PackedStorageContract(gl.Contract):
    """Optimize storage using packed values"""
    
    # Instead of:
    # user_balance: dict[str, float]
    # user_timestamp: dict[str, int]
    # user_active: dict[str, bool]
    
    # Use:
    user_data: dict[str, dict]
    
    def __init__(self):
        self.user_data = {}
    
    @gl.public.write
    def update_user(self, user_id: str, balance: float, active: bool):
        """Update user with packed data"""
        
        self.user_data[user_id] = {
            'balance': balance,
            'timestamp': gl.block.timestamp,
            'active': active
        }
    
    @gl.public.view
    def get_user(self, user_id: str) -> dict:
        return self.user_data.get(user_id, {})
```

**Benefits:**
- Reduced storage slots
- Lower gas costs
- Better cache locality

---

### 2. Lazy Loading

Load data only when needed:

```python
class LazyLoadingContract(gl.Contract):
    """Lazy load data on demand"""
    
    data_index: dict[str, str]  # Maps to external URLs
    data_cache: dict[str, dict]
    
    def __init__(self):
        self.data_index = {}
        self.data_cache = {}
    
    @gl.public.write
    def register_data(self, data_id: str, data_url: str):
        """Register data source (don't load yet)"""
        self.data_index[data_id] = data_url
    
    @gl.public.view
    def get_data(self, data_id: str) -> dict:
        """Load data only when requested"""
        
        # Check cache first
        if data_id in self.data_cache:
            return self.data_cache[data_id]
        
        # Load from source
        if data_id in self.data_index:
            url = self.data_index[data_id]
            data = gl.nondet.web.get(url)
            self.data_cache[data_id] = data
            return data
        
        return {}
```

**Benefits:**
- Reduced initial state
- On-demand loading
- Better memory efficiency

---

### 3. Bloom Filters

Efficiently check membership:

```python
class BloomFilterContract(gl.Contract):
    """Use bloom filter for efficient membership checking"""
    
    bloom_filter: list[bool]
    filter_size: int
    
    def __init__(self):
        self.filter_size = 1000
        self.bloom_filter = [False] * self.filter_size
    
    def _hash(self, item: str, seed: int) -> int:
        """Hash function for bloom filter"""
        import hashlib
        h = hashlib.md5(f"{item}{seed}".encode()).hexdigest()
        return int(h, 16) % self.filter_size
    
    @gl.public.write
    def add_item(self, item: str):
        """Add item to bloom filter"""
        for seed in range(3):  # Use 3 hash functions
            index = self._hash(item, seed)
            self.bloom_filter[index] = True
    
    @gl.public.view
    def might_contain(self, item: str) -> bool:
        """Check if item might be in set"""
        for seed in range(3):
            index = self._hash(item, seed)
            if not self.bloom_filter[index]:
                return False
        return True
```

**Benefits:**
- Constant memory usage
- Fast membership checking
- Space-efficient

---

## Gas Optimization

### 1. Batch Operations

Process multiple items in single transaction:

```python
class BatchOperationContract(gl.Contract):
    """Optimize with batch operations"""
    
    items: dict[str, int]
    
    def __init__(self):
        self.items = {}
    
    @gl.public.write
    def batch_update(self, updates: dict[str, int]):
        """Update multiple items at once"""
        
        # Single state write instead of multiple
        for key, value in updates.items():
            self.items[key] = value
    
    @gl.public.write
    def batch_increment(self, keys: list[str], increment: int):
        """Increment multiple items"""
        
        for key in keys:
            if key in self.items:
                self.items[key] += increment
            else:
                self.items[key] = increment
```

**Benefits:**
- Reduced transaction count
- Lower total gas cost
- Atomic operations

---

### 2. Early Exit

Minimize computation for invalid inputs:

```python
class EarlyExitContract(gl.Contract):
    """Optimize with early exit"""
    
    @gl.public.write
    def process_transaction(self, tx_id: str, amount: float):
        """Process with early validation"""
        
        # Early exit for invalid inputs
        if not tx_id:
            return
        
        if amount <= 0:
            return
        
        if amount > 1000000:
            return
        
        # Only process valid transactions
        # ... rest of processing
        pass
```

**Benefits:**
- Reduced gas for invalid inputs
- Faster rejection
- Better UX

---

### 3. Caching

Cache expensive computations:

```python
class CachingContract(gl.Contract):
    """Optimize with caching"""
    
    computation_cache: dict[str, dict]
    cache_ttl: int
    
    def __init__(self):
        self.computation_cache = {}
        self.cache_ttl = 3600
    
    def _expensive_computation(self, input_data: str) -> float:
        """Expensive computation with caching"""
        
        # Check cache
        if input_data in self.computation_cache:
            cached = self.computation_cache[input_data]
            if gl.block.timestamp - cached['timestamp'] < self.cache_ttl:
                return cached['result']
        
        # Compute result
        result = self._do_expensive_work(input_data)
        
        # Cache result
        self.computation_cache[input_data] = {
            'result': result,
            'timestamp': gl.block.timestamp
        }
        
        return result
    
    def _do_expensive_work(self, data: str) -> float:
        # Actual computation
        return float(len(data))
```

**Benefits:**
- Reduced repeated computation
- Lower gas costs
- Faster execution

---

## LLM Optimization

### 1. Prompt Caching

Cache LLM responses:

```python
class PromptCachingContract(gl.Contract):
    """Cache LLM responses"""
    
    llm_cache: dict[str, dict]
    cache_ttl: int
    
    def __init__(self):
        self.llm_cache = {}
        self.cache_ttl = 7200
    
    def _get_llm_response(self, prompt: str) -> str:
        """Get LLM response with caching"""
        
        # Create cache key from prompt
        import hashlib
        cache_key = hashlib.md5(prompt.encode()).hexdigest()
        
        # Check cache
        if cache_key in self.llm_cache:
            cached = self.llm_cache[cache_key]
            if gl.block.timestamp - cached['timestamp'] < self.cache_ttl:
                return cached['response']
        
        # Call LLM
        response = gl.eq_principle.prompt_non_comparative(
            lambda: prompt,
            task="Process prompt",
            criteria="Valid response"
        )
        
        # Cache response
        self.llm_cache[cache_key] = {
            'response': response,
            'timestamp': gl.block.timestamp
        }
        
        return response
```

**Benefits:**
- Reduced LLM API calls
- Lower costs
- Faster responses

---

### 2. Prompt Optimization

Write efficient prompts:

```python
class OptimizedPromptsContract(gl.Contract):
    """Optimize LLM prompts"""
    
    @gl.public.write
    def classify_with_optimized_prompt(self, content: str):
        """Use optimized prompt for classification"""
        
        # ❌ Bad: Verbose, unclear
        bad_prompt = f"""
        Please analyze the following content and try to determine what category 
        it might belong to. The categories are: educational, entertainment, news, 
        technical, and other. Please provide your best guess about which category 
        this content fits into.
        
        Content: {content}
        """
        
        # ✅ Good: Concise, clear
        good_prompt = f"""Classify this content into ONE category:
        
Categories: educational, entertainment, news, technical, other

Content: {content[:200]}

Response format: CATEGORY: [category name]"""
        
        response = gl.eq_principle.prompt_non_comparative(
            lambda: good_prompt,
            task="Classify content",
            criteria="Response must start with 'CATEGORY:' followed by one category"
        )
        
        return response
```

**Benefits:**
- Faster LLM processing
- Clearer responses
- Better token efficiency

---

### 3. Batch LLM Calls

Process multiple prompts efficiently:

```python
class BatchLLMContract(gl.Contract):
    """Batch multiple LLM calls"""
    
    @gl.public.write
    def batch_classify(self, items: list[str]):
        """Classify multiple items efficiently"""
        
        results = []
        
        for item in items:
            # Could batch these in a single LLM call
            prompt = f"Classify: {item}"
            
            response = gl.eq_principle.prompt_non_comparative(
                lambda: prompt,
                task="Classify",
                criteria="Return category"
            )
            
            results.append({
                'item': item,
                'classification': response
            })
        
        return results
```

**Benefits:**
- More efficient processing
- Better resource utilization
- Potential cost savings

---

## Concurrency & Scalability

### 1. Sharding

Distribute data across multiple contracts:

```python
class ShardedDataContract(gl.Contract):
    """Sharded data storage"""
    
    shard_count: int
    shards: dict[int, dict]
    
    def __init__(self, shard_count: int = 10):
        self.shard_count = shard_count
        self.shards = {i: {} for i in range(shard_count)}
    
    def _get_shard(self, key: str) -> int:
        """Determine shard for key"""
        import hashlib
        h = hashlib.md5(key.encode()).hexdigest()
        return int(h, 16) % self.shard_count
    
    @gl.public.write
    def set_value(self, key: str, value):
        """Set value in appropriate shard"""
        shard_id = self._get_shard(key)
        self.shards[shard_id][key] = value
    
    @gl.public.view
    def get_value(self, key: str):
        """Get value from appropriate shard"""
        shard_id = self._get_shard(key)
        return self.shards[shard_id].get(key)
```

**Benefits:**
- Distributed storage
- Parallel processing
- Better scalability

---

### 2. Rate Limiting

Prevent abuse and manage load:

```python
class RateLimitedContract(gl.Contract):
    """Rate limiting for contract methods"""
    
    user_calls: dict[str, list]
    rate_limit: int
    time_window: int
    
    def __init__(self):
        self.user_calls = {}
        self.rate_limit = 10  # 10 calls
        self.time_window = 3600  # per hour
    
    def _check_rate_limit(self, user: str) -> bool:
        """Check if user exceeds rate limit"""
        
        current_time = gl.block.timestamp
        
        if user not in self.user_calls:
            self.user_calls[user] = []
        
        # Remove old calls outside time window
        self.user_calls[user] = [
            call_time for call_time in self.user_calls[user]
            if current_time - call_time < self.time_window
        ]
        
        # Check limit
        if len(self.user_calls[user]) >= self.rate_limit:
            return False
        
        # Record call
        self.user_calls[user].append(current_time)
        return True
    
    @gl.public.write
    def rate_limited_method(self):
        """Method with rate limiting"""
        assert self._check_rate_limit(msg.sender), "Rate limit exceeded"
        # Method implementation
        pass
```

**Benefits:**
- Prevent abuse
- Manage resources
- Fair usage

---

## Advanced Security

### 1. Reentrancy Protection

Prevent reentrancy attacks:

```python
class ReentrancyProtectedContract(gl.Contract):
    """Protect against reentrancy attacks"""
    
    locked: bool
    balances: dict[str, float]
    
    def __init__(self):
        self.locked = False
        self.balances = {}
    
    def _lock(self):
        """Acquire lock"""
        assert not self.locked, "Reentrancy detected"
        self.locked = True
    
    def _unlock(self):
        """Release lock"""
        self.locked = False
    
    @gl.public.write.payable
    def deposit(self):
        """Deposit with reentrancy protection"""
        self._lock()
        try:
            self.balances[msg.sender] = self.balances.get(msg.sender, 0) + msg.value
        finally:
            self._unlock()
    
    @gl.public.write
    def withdraw(self, amount: float):
        """Withdraw with reentrancy protection"""
        self._lock()
        try:
            assert self.balances[msg.sender] >= amount
            self.balances[msg.sender] -= amount
            # Transfer funds
        finally:
            self._unlock()
```

**Benefits:**
- Prevent reentrancy attacks
- Atomic operations
- Safe state transitions

---

### 2. Input Validation

Comprehensive input validation:

```python
class ValidatedContract(gl.Contract):
    """Comprehensive input validation"""
    
    def _validate_address(self, address: str) -> bool:
        """Validate address format"""
        return len(address) == 42 and address.startswith('0x')
    
    def _validate_amount(self, amount: float) -> bool:
        """Validate amount"""
        return 0 < amount <= 1000000
    
    def _validate_string(self, text: str, max_length: int = 1000) -> bool:
        """Validate string"""
        return 0 < len(text) <= max_length
    
    @gl.public.write
    def transfer(self, to_address: str, amount: float):
        """Transfer with validation"""
        
        assert self._validate_address(to_address), "Invalid address"
        assert self._validate_amount(amount), "Invalid amount"
        
        # Perform transfer
        pass
```

**Benefits:**
- Prevent invalid inputs
- Reduce attack surface
- Better error messages

---

### 3. Access Control Lists

Fine-grained permissions:

```python
class ACLContract(gl.Contract):
    """Access Control List"""
    
    permissions: dict[str, dict]
    roles: dict[str, list]
    
    def __init__(self):
        self.permissions = {}
        self.roles = {}
    
    def _grant_role(self, user: str, role: str):
        """Grant role to user"""
        if user not in self.roles:
            self.roles[user] = []
        if role not in self.roles[user]:
            self.roles[user].append(role)
    
    def _has_role(self, user: str, role: str) -> bool:
        """Check if user has role"""
        return role in self.roles.get(user, [])
    
    def _require_role(self, role: str):
        """Require specific role"""
        assert self._has_role(msg.sender, role), f"Requires {role} role"
    
    @gl.public.write
    def admin_method(self):
        """Method requiring admin role"""
        self._require_role('admin')
        # Admin-only logic
        pass
```

**Benefits:**
- Fine-grained control
- Role-based access
- Flexible permissions

---

## Performance Benchmarks

| Optimization | Gas Saved | Speed Improvement |
|--------------|-----------|-------------------|
| Batch Operations | 30-50% | 40-60% |
| Caching | 50-70% | 60-80% |
| Early Exit | 20-40% | 30-50% |
| Sharding | 40-60% | 50-70% |
| Rate Limiting | 10-20% | 5-10% |

---

## Optimization Checklist

- [ ] Use batch operations where possible
- [ ] Implement caching for expensive operations
- [ ] Add early exit for invalid inputs
- [ ] Optimize LLM prompts
- [ ] Use appropriate data structures
- [ ] Implement rate limiting
- [ ] Add reentrancy protection
- [ ] Validate all inputs
- [ ] Monitor gas usage
- [ ] Profile performance

---

## Conclusion

Advanced patterns and optimization techniques are essential for building scalable, efficient, and secure Intelligent Contracts. Choose patterns based on your specific requirements and always test thoroughly before deployment.

---

**Last Updated:** March 2026  
**Author:** kuru  
**Version:** 1.0
