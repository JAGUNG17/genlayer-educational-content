# Getting Started with GenLayer Intelligent Contracts

## Introduction

GenLayer represents a paradigm shift in blockchain development by introducing **Intelligent Contracts**—a revolutionary evolution of traditional smart contracts. Unlike conventional smart contracts that operate deterministically with limited access to external data, Intelligent Contracts harness the power of Large Language Models (LLMs), web connectivity, and sophisticated consensus mechanisms to enable complex, real-world decision-making on the blockchain.

This tutorial will guide you through the fundamental concepts and practical implementation of GenLayer Intelligent Contracts, equipping you with the knowledge to build AI-powered decentralized applications.

## What Are Intelligent Contracts?

Intelligent Contracts are Python-based smart contracts that combine three core capabilities:

### 1. Natural Language Processing (NLP)

Intelligent Contracts leverage Large Language Models to interpret and process human language inputs. This enables contracts to understand qualitative criteria and make nuanced decisions based on contextual understanding, moving far beyond the rigid conditional logic of traditional smart contracts.

**Example Use Case:** A contract can evaluate whether a piece of content meets subjective quality criteria, such as determining if a submitted article is "well-written" or "technically accurate," without requiring explicit code-based rules.

### 2. Web Connectivity

These contracts can actively fetch and process real-time data from web APIs, enabling dynamic decision-making based on current information. This capability bridges the gap between on-chain and off-chain environments without relying on traditional oracle services.

**Example Use Case:** A contract can fetch current cryptocurrency prices from an API, verify weather conditions from a weather service, or check social media metrics in real-time to trigger contract actions.

### 3. Non-Deterministic Operations

Intelligent Contracts introduce sophisticated handling of non-deterministic operations—tasks that may produce varying results across different validators. Through the **Equivalence Principle**, multiple validators can reach consensus even when processing LLM outputs or variable external data.

**Example Use Case:** When multiple validators call an LLM with the same prompt, they may receive semantically similar but textually different responses. The Equivalence Principle allows validators to agree that these responses are equivalent without requiring exact matches.

## Core Concepts

### The Equivalence Principle

The Equivalence Principle is the foundation of GenLayer's consensus mechanism for non-deterministic operations. It provides two validation approaches:

**Comparative Validation (`gl.eq_principle.strict_eq`):** Used for operations with deterministic outputs (like boolean checks). Validators must agree on exactly the same result.

**Non-Comparative Validation (`gl.eq_principle.prompt_non_comparative`):** Used for LLM outputs and complex data. Validators evaluate whether results meet specified criteria rather than requiring exact matches.

### Contract Structure

All Intelligent Contracts follow a consistent Python-based structure:

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class MyContract(gl.Contract):
    # State variables with type annotations
    state_variable: str
    counter: int
    
    def __init__(self):
        self.state_variable = "initial value"
        self.counter = 0
    
    @gl.public.view
    def read_method(self) -> str:
        """Read-only method that doesn't modify state"""
        return self.state_variable
    
    @gl.public.write
    def write_method(self, new_value: str):
        """Method that modifies contract state"""
        self.state_variable = new_value
        self.counter += 1
    
    @gl.public.write.payable
    def payable_method(self, amount: int):
        """Method that can receive value"""
        self.state_variable = f"Received {amount}"
```

### Method Decorators

GenLayer provides three decorator types for contract methods:

| Decorator | Purpose | State Modification | Can Receive Value |
|-----------|---------|-------------------|------------------|
| `@gl.public.view` | Read-only operations | No | No |
| `@gl.public.write` | State modification | Yes | No |
| `@gl.public.write.payable` | State modification with value transfer | Yes | Yes |

## Your First Intelligent Contract

Let's build a simple contract that demonstrates web connectivity and the Equivalence Principle:

### Step 1: Basic Web Verification Contract

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class WebVerificationContract(gl.Contract):
    verified_content: bool
    
    def __init__(self):
        self.verified_content = False
    
    @gl.public.write
    def verify_website(self):
        """Verify that a website contains specific content"""
        website_url = 'https://example.org'
        
        def fetch_and_verify():
            # Fetch webpage content
            web_data = gl.nondet.web.render(website_url, mode='html')
            # Check if specific content exists
            return 'iana' in web_data
        
        # Use strict equivalence for boolean results
        self.verified_content = gl.eq_principle.strict_eq(fetch_and_verify)
    
    @gl.public.view
    def get_verification_status(self) -> bool:
        return self.verified_content
```

**Key Points:**

- Non-deterministic operations (web fetching) must occur within a function passed to `gl.eq_principle.*`
- The function must have no arguments—external variables are captured automatically
- `gl.eq_principle.strict_eq` is appropriate for boolean or deterministic results
- All validators will agree on the same boolean result

### Step 2: LLM Integration with Non-Comparative Validation

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *
import typing

class LLMAnalysisContract(gl.Contract):
    analysis_result: str
    
    def __init__(self):
        self.analysis_result = ""
    
    @gl.public.write
    def analyze_content(self):
        """Use LLM to analyze content with non-comparative validation"""
        self.analysis_result = gl.eq_principle.prompt_non_comparative(
            lambda: "Analyze this text and provide a brief summary",
            task="Provide a concise summary of the given text",
            criteria="Summary should be 1-3 sentences and capture main points"
        )
    
    @gl.public.view
    def get_analysis(self) -> str:
        return self.analysis_result
```

**Key Points:**

- Non-comparative validation allows different validators to accept different LLM outputs as long as they meet the criteria
- The lambda function provides the system prompt
- The `task` parameter describes what the LLM should do
- The `criteria` parameter defines what validators will check

## Advanced Features

### Web Data Access

Intelligent Contracts can fetch data from web APIs in two modes:

**HTML Mode:** Retrieves the full HTML structure, useful for parsing specific elements.

```python
def fetch_html():
    web_data = gl.nondet.web.render('https://api.example.com', mode='html')
    return web_data
```

**Text Mode:** Retrieves rendered text content, useful for reading human-readable information.

```python
def fetch_text():
    web_data = gl.nondet.web.render('https://api.example.com', mode='text')
    return web_data
```

### Storage and State Management

Intelligent Contracts support various data types for state management:

```python
class AdvancedStorageContract(gl.Contract):
    # Primitive types
    name: str
    count: int
    balance: float
    is_active: bool
    
    # Collection types
    items: list[str]
    user_scores: dict[str, int]
    
    def __init__(self):
        self.name = "Contract"
        self.count = 0
        self.balance = 0.0
        self.is_active = True
        self.items = []
        self.user_scores = {}
    
    @gl.public.write
    def add_item(self, item: str):
        self.items.append(item)
    
    @gl.public.write
    def record_score(self, user: str, score: int):
        self.user_scores[user] = score
```

### Error Handling

Proper error handling ensures contract reliability:

```python
class ErrorHandlingContract(gl.Contract):
    last_error: str
    
    def __init__(self):
        self.last_error = ""
    
    @gl.public.write
    def process_with_error_handling(self, value: int):
        try:
            if value < 0:
                raise ValueError("Value must be positive")
            result = 100 / value
            self.last_error = ""
        except ValueError as e:
            self.last_error = str(e)
        except ZeroDivisionError as e:
            self.last_error = "Cannot divide by zero"
    
    @gl.public.view
    def get_last_error(self) -> str:
        return self.last_error
```

## Best Practices

### 1. Prompt Engineering

When using LLMs in Intelligent Contracts, craft clear, specific prompts:

```python
# Good: Specific and unambiguous
prompt = "Extract the price in USD from the following text"

# Avoid: Vague and open-ended
prompt = "Tell me about the price"
```

### 2. Validation Criteria

Define clear validation criteria for non-comparative validation:

```python
# Good: Specific criteria
criteria = "Response must be a valid JSON object with 'price' and 'currency' fields"

# Avoid: Vague criteria
criteria = "Response should be reasonable"
```

### 3. Security Considerations

Be cautious with external data and user inputs:

```python
class SecureContract(gl.Contract):
    @gl.public.write
    def process_user_input(self, user_input: str):
        # Validate input length
        if len(user_input) > 1000:
            raise ValueError("Input too long")
        
        # Use input carefully in prompts
        safe_prompt = f"Analyze this user-provided text: {user_input}"
```

### 4. Gas Efficiency

Minimize expensive operations:

```python
class EfficientContract(gl.Contract):
    cached_result: str
    last_update: int
    
    @gl.public.write
    def update_if_needed(self, force_update: bool = False):
        # Avoid redundant LLM calls
        if force_update or not self.cached_result:
            self.cached_result = gl.eq_principle.prompt_non_comparative(
                lambda: "Generate result",
                task="Generate a result",
                criteria="Result should be valid"
            )
```

## Deployment

### Local Testing

Before deploying, test your contract locally:

```bash
# Install GenLayer CLI
pip install genlayer

# Initialize a new project
genlayer init my_contract

# Test your contract
genlayer test
```

### Deployment Steps

1. **Prepare your contract:** Ensure all code is tested and validated
2. **Configure network:** Specify the GenLayer network endpoint
3. **Deploy:** Use the GenLayer CLI to deploy your contract

```bash
genlayer deploy --contract MyContract --network testnet
```

## Common Patterns

### Pattern 1: Data Verification

```python
class DataVerificationContract(gl.Contract):
    verified_data: dict
    
    @gl.public.write
    def verify_external_data(self, data_url: str):
        def fetch_and_verify():
            data = gl.nondet.web.get(data_url)
            # Verify data integrity
            return len(data) > 0 and 'status' in data
        
        if gl.eq_principle.strict_eq(fetch_and_verify):
            self.verified_data = {'status': 'verified'}
```

### Pattern 2: Content Analysis

```python
class ContentAnalysisContract(gl.Contract):
    analysis_cache: dict[str, str]
    
    @gl.public.write
    def analyze_content(self, content_id: str, content: str):
        if content_id not in self.analysis_cache:
            analysis = gl.eq_principle.prompt_non_comparative(
                lambda: f"Analyze: {content}",
                task="Provide content analysis",
                criteria="Analysis should be 2-4 sentences"
            )
            self.analysis_cache[content_id] = analysis
```

### Pattern 3: Multi-Source Verification

```python
class MultiSourceContract(gl.Contract):
    verification_results: list[bool]
    
    @gl.public.write
    def verify_from_multiple_sources(self, urls: list[str]):
        results = []
        for url in urls:
            def verify_url():
                data = gl.nondet.web.get(url)
                return 'success' in data.lower()
            
            results.append(gl.eq_principle.strict_eq(verify_url))
        
        self.verification_results = results
```

## Troubleshooting

### Issue: Non-deterministic operations outside of `gl.eq_principle`

**Error:** `RuntimeError: Web operations must be called within gl.eq_principle`

**Solution:** Ensure all `gl.nondet.*` calls are within functions passed to `gl.eq_principle.*`

```python
# Wrong
data = gl.nondet.web.get('https://example.com')

# Correct
def fetch():
    return gl.nondet.web.get('https://example.com')
result = gl.eq_principle.strict_eq(fetch)
```

### Issue: Validators disagreeing on LLM output

**Error:** `ConsensusError: Validators could not reach agreement`

**Solution:** Review and refine your validation criteria to be more specific

```python
# Better criteria
criteria = "Response must be a single word, either 'yes' or 'no'"

# Vague criteria
criteria = "Response should be reasonable"
```

### Issue: Timeout during contract execution

**Error:** `TimeoutError: Contract execution exceeded time limit`

**Solution:** Optimize your contract to reduce computational overhead

- Minimize LLM calls
- Cache results when possible
- Reduce external API calls

## Next Steps

Now that you understand the fundamentals of Intelligent Contracts, explore these advanced topics:

1. **Vector Storage:** Learn how to implement semantic search and similarity matching
2. **Advanced Consensus:** Understand appeal processes and validator staking
3. **DApp Development:** Build complete decentralized applications with frontend integration
4. **Production Deployment:** Deploy contracts to mainnet with proper security audits

## Resources

- **Official Documentation:** https://docs.genlayer.com
- **GitHub Repository:** https://github.com/genlayer
- **Community Forum:** https://talks.genlayer.foundation
- **Discord Community:** Join the GenLayer Discord for support and discussions

---

**Last Updated:** March 2026  
**Author:** Manus AI  
**Version:** 1.0
