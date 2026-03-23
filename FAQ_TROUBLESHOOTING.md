# GenLayer Intelligent Contracts: FAQ & Troubleshooting Guide

Comprehensive answers to frequently asked questions and solutions to common problems.

## Table of Contents

1. [General Questions](#general-questions)
2. [Development & Testing](#development--testing)
3. [Consensus & Validation](#consensus--validation)
4. [Performance Issues](#performance-issues)
5. [Security Concerns](#security-concerns)
6. [Deployment & Mainnet](#deployment--mainnet)

---

## General Questions

### Q1: What's the difference between Intelligent Contracts and Smart Contracts?

**A:** Intelligent Contracts extend smart contracts with three key capabilities:

| Aspect | Smart Contracts | Intelligent Contracts |
|--------|-----------------|----------------------|
| Execution | Deterministic only | Deterministic + Non-deterministic |
| External Data | Oracle-dependent | Direct web API access |
| Logic | Code-based | Code + AI-powered |
| Language | Solidity, Vyper | Python |
| Decision Making | Binary/mathematical | Qualitative and complex |

Intelligent Contracts can call LLMs, fetch real-time web data, and make nuanced decisions that were previously impossible on blockchain.

### Q2: Can I use any Python library in GenLayer?

**A:** No, only a restricted set of Python libraries are available for security reasons. Supported libraries include:

- `json` - JSON parsing and serialization
- `datetime` - Date and time operations
- `math` - Mathematical functions
- `hashlib` - Cryptographic hashing
- `base64` - Base64 encoding/decoding
- `re` - Regular expressions
- `urllib.parse` - URL parsing

For external data, use `gl.nondet.web.get()` instead of making direct HTTP requests.

### Q3: How much does it cost to deploy an Intelligent Contract?

**A:** GenLayer uses a gas-based fee model similar to Ethereum:

- **Deployment:** ~50,000 - 200,000 gas depending on contract complexity
- **Method Calls:** 20,000 - 100,000 gas per call
- **LLM Calls:** Additional cost based on token usage
- **Web API Calls:** Minimal additional cost

Exact pricing depends on network conditions and LLM provider rates.

### Q4: What happens if my contract calls an LLM and the API is down?

**A:** The transaction will fail with a timeout error. Best practices:

```python
try:
    response = gl.eq_principle.prompt_non_comparative(prompt_func, ...)
except Exception as e:
    # Fallback behavior
    response = self.default_response
```

Always implement fallback mechanisms for external dependencies.

### Q5: Can I update my contract after deployment?

**A:** No, Intelligent Contracts are immutable once deployed. However:

- You can deploy a new version of the contract
- Migrate state from old to new contract
- Use proxy patterns for upgradeable contracts (advanced)

Plan carefully before deployment.

---

## Development & Testing

### Q6: How do I test my contract locally?

**A:** Use the GenLayer test framework:

```python
from genlayer import *
from genlayer.test import ContractTest

class TestMyContract(ContractTest):
    def setUp(self):
        self.contract = MyContract()
    
    def test_basic_functionality(self):
        self.contract.some_method()
        assert self.contract.some_state == expected_value
    
    def test_web_call(self):
        # Mock web calls in tests
        with self.mock_web_get('https://example.com', '{"data": "value"}'):
            self.contract.fetch_data()
```

### Q7: How do I debug LLM calls?

**A:** Enable debug logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)

# Your contract code will now log LLM interactions
```

Also check:
- LLM response in contract state
- Validation criteria in contract code
- Prompt formatting for clarity

### Q8: What's the maximum size of contract state?

**A:** There's no hard limit, but larger state increases:

- Gas costs for state changes
- Validator verification time
- Network bandwidth

Best practice: Keep state under 1 MB per contract.

### Q9: Can I call another contract from my contract?

**A:** Yes, using contract references:

```python
@gl.public.write
def call_other_contract(self, other_contract_address: str):
    other = gl.contract_at(other_contract_address)
    result = other.some_method()
```

However, this increases complexity and gas costs. Use sparingly.

### Q10: How do I handle large data structures?

**A:** Use pagination and lazy loading:

```python
@gl.public.view
def get_items_paginated(self, page: int, page_size: int = 10) -> list:
    start = page * page_size
    end = start + page_size
    return self.items[start:end]
```

This reduces memory usage and improves performance.

---

## Consensus & Validation

### Q11: What happens if validators disagree on LLM output?

**A:** The transaction enters an appeal process:

1. Initial validators vote on result
2. If consensus not reached, additional validators are called
3. Majority vote determines final result
4. If still no consensus, transaction is rejected

To minimize disagreement:
- Use clear, specific prompts
- Define strict validation criteria
- Test with multiple LLM calls

### Q12: How do I ensure my LLM criteria are clear?

**A:** Use specific, measurable criteria:

```python
# ❌ Bad: Vague criteria
criteria="Response should be reasonable"

# ✅ Good: Specific criteria
criteria="Response must be 'approved', 'rejected', or 'pending' with reasoning"
```

### Q13: What's the difference between comparative and non-comparative validation?

**A:**

**Comparative Validation (Strict Matching):**
- All validators must produce identical results
- Used for deterministic operations
- Example: Checking if website contains text

```python
result = gl.eq_principle.strict_eq(lambda: check_website())
```

**Non-Comparative Validation (Criteria-Based):**
- Validators must meet specified criteria
- Used for LLM outputs
- Example: Classifying content

```python
result = gl.eq_principle.prompt_non_comparative(
    prompt_func,
    criteria="Must classify as A, B, or C"
)
```

### Q14: How many validators verify each transaction?

**A:** Default: 5-10 validators depending on network conditions

- Minimum: 3 validators for consensus
- Maximum: 50+ for critical operations
- Configurable per contract

### Q15: Can I specify which validators verify my transaction?

**A:** No, validators are selected randomly by the network. However, you can:

- Increase validator count for critical operations
- Use staking to incentivize honest validators
- Implement reputation systems

---

## Performance Issues

### Q16: My contract is slow. How do I optimize?

**A:** Common optimization techniques:

1. **Cache Results:**
```python
if key in self.cache:
    return self.cache[key]
result = expensive_operation()
self.cache[key] = result
```

2. **Batch Operations:**
```python
def process_batch(self, items: list):
    for item in items:
        self.process_item(item)
```

3. **Lazy Loading:**
```python
def get_data(self, page: int):
    return self.data[page*10:(page+1)*10]
```

### Q17: LLM calls are timing out. What should I do?

**A:** Solutions:

1. **Simplify Prompts:** Reduce prompt length
2. **Use Faster Models:** Switch to faster LLM (if available)
3. **Implement Timeouts:**
```python
try:
    response = gl.eq_principle.prompt_non_comparative(
        prompt_func,
        timeout=30  # 30 second timeout
    )
except TimeoutError:
    response = self.default_response
```

4. **Batch LLM Calls:** Process multiple items together

### Q18: Web API calls are slow. How do I speed them up?

**A:** Optimization strategies:

1. **Cache API Responses:**
```python
if url in self.api_cache:
    return self.api_cache[url]
data = gl.nondet.web.get(url)
self.api_cache[url] = data
```

2. **Use Faster APIs:** Choose APIs with low latency
3. **Parallel Fetching:** Fetch from multiple sources simultaneously
4. **Implement Fallbacks:** Use cached data if API is slow

### Q19: How do I reduce gas costs?

**A:** Gas optimization techniques:

1. **Minimize State Changes:** Batch updates together
2. **Use Efficient Data Structures:** Prefer lists over nested dicts
3. **Avoid Loops:** Use vectorized operations
4. **Cache Results:** Avoid recalculating

Example:
```python
# ❌ Expensive: Multiple state changes
for item in items:
    self.count += 1

# ✅ Efficient: Single state change
self.count += len(items)
```

### Q20: What's the maximum execution time for a contract method?

**A:** Default: 60 seconds per method call

- Minimum: 10 seconds
- Maximum: 300 seconds (configurable)
- Exceeding limit results in transaction timeout

---

## Security Concerns

### Q21: How do I prevent malicious LLM prompts?

**A:** Input validation and sanitization:

```python
def validate_prompt_input(self, user_input: str) -> str:
    # Remove dangerous characters
    sanitized = user_input.replace('"', '').replace("'", '')
    
    # Limit length
    if len(sanitized) > 1000:
        sanitized = sanitized[:1000]
    
    return sanitized
```

### Q22: Can someone manipulate my contract through LLM responses?

**A:** Unlikely, but possible with poor validation:

```python
# ❌ Vulnerable: Trusts LLM output directly
amount = int(llm_response)

# ✅ Secure: Validates LLM output
try:
    amount = int(llm_response)
    assert 0 < amount < 1000000  # Reasonable bounds
except:
    amount = 0  # Default safe value
```

### Q23: How do I protect sensitive data in my contract?

**A:** Best practices:

1. **Don't Store Secrets:** Never hardcode API keys or passwords
2. **Use Environment Variables:** Store secrets in contract environment
3. **Encrypt Sensitive Data:** Use encryption for sensitive state
4. **Limit Data Exposure:** Only expose necessary data in view methods

```python
# ❌ Bad: Hardcoded secret
API_KEY = "sk-1234567890"

# ✅ Good: Environment variable
API_KEY = gl.env.get('API_KEY')
```

### Q24: What happens if my contract has a bug?

**A:** Bugs can cause:

- **Incorrect Results:** Validators reject transaction
- **State Corruption:** Contract becomes unusable
- **Fund Loss:** Funds locked in contract

Mitigation:
- Thorough testing before deployment
- Gradual rollout with small amounts
- Implement emergency pause mechanism
- Use contract upgrades (proxy pattern)

### Q25: How do I implement access control?

**A:** Use sender verification:

```python
@gl.public.write
def admin_only_method(self):
    assert msg.sender == self.admin, "Only admin can call this"
    # Method implementation
```

For role-based access:

```python
def has_role(self, address: str, role: str) -> bool:
    return role in self.user_roles.get(address, [])

@gl.public.write
def restricted_method(self):
    assert self.has_role(msg.sender, 'moderator'), "Insufficient permissions"
    # Method implementation
```

---

## Deployment & Mainnet

### Q26: How do I deploy my contract to mainnet?

**A:** Deployment steps:

1. **Test Thoroughly:** Use testnet first
2. **Prepare Deployment:**
```bash
genlayer contract deploy --network mainnet --contract MyContract.py
```

3. **Verify Contract:** Check deployment transaction
4. **Monitor:** Watch for errors in first transactions

### Q27: What's the difference between testnet and mainnet?

**A:**

| Aspect | Testnet | Mainnet |
|--------|---------|---------|
| Real Value | No | Yes |
| Gas Cost | Free/Cheap | Real cost |
| Validators | Limited | Full network |
| Finality | Fast | Slower |
| Risk | Low | High |

Always test on testnet before mainnet deployment.

### Q28: How do I migrate state from one contract to another?

**A:** Migration pattern:

```python
class NewContractV2(gl.Contract):
    def __init__(self, old_contract_address: str):
        old_contract = gl.contract_at(old_contract_address)
        
        # Migrate state
        self.data = old_contract.get_all_data()
        self.version = 2
```

### Q29: Can I pause my contract if something goes wrong?

**A:** Yes, implement an emergency pause:

```python
class PausableContract(gl.Contract):
    paused: bool
    
    def __init__(self):
        self.paused = False
    
    @gl.public.write
    def pause(self):
        assert msg.sender == self.admin
        self.paused = True
    
    @gl.public.write
    def unpause(self):
        assert msg.sender == self.admin
        self.paused = False
    
    def _check_not_paused(self):
        assert not self.paused, "Contract is paused"
    
    @gl.public.write
    def critical_method(self):
        self._check_not_paused()
        # Method implementation
```

### Q30: How do I monitor my contract after deployment?

**A:** Monitoring best practices:

1. **Watch Transactions:** Monitor all method calls
2. **Track State Changes:** Alert on unexpected state changes
3. **Monitor Errors:** Track failed transactions
4. **Performance Metrics:** Monitor gas costs and execution time
5. **Validator Health:** Check validator participation

Use GenLayer's monitoring dashboard or build custom monitoring.

---

## Quick Reference: Common Errors

### Error: "Consensus not reached"

**Cause:** Validators disagreed on result

**Solution:**
- Check LLM prompt clarity
- Verify validation criteria
- Test with multiple validators
- Check for non-deterministic code

### Error: "Timeout exceeded"

**Cause:** Method execution took too long

**Solution:**
- Optimize code
- Reduce LLM call complexity
- Implement caching
- Increase timeout limit (if allowed)

### Error: "Invalid state transition"

**Cause:** State change violates contract logic

**Solution:**
- Check contract state machine
- Verify preconditions
- Add state validation
- Review state change logic

### Error: "Insufficient gas"

**Cause:** Not enough gas for transaction

**Solution:**
- Increase gas limit
- Optimize contract code
- Reduce state changes
- Use batch operations

### Error: "LLM API error"

**Cause:** LLM service unavailable

**Solution:**
- Implement retry logic
- Use fallback responses
- Check API status
- Add error handling

---

## Getting Help

### Resources

- **Official Documentation:** https://docs.genlayer.com
- **Community Forum:** https://talks.genlayer.foundation
- **Discord:** Join GenLayer Discord
- **GitHub Issues:** Report bugs on GitHub

### Asking Good Questions

When asking for help:

1. **Provide Context:** Share contract code and error message
2. **Reproduce Issue:** Provide steps to reproduce
3. **Show Attempts:** Explain what you've already tried
4. **Attach Logs:** Include relevant error logs
5. **Specify Version:** Mention GenLayer version

---

**Last Updated:** March 2026  
**Author:** kuru  
**Version:** 1.0
