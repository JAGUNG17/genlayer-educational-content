# GenLayer Intelligent Contracts: Integration Guides

Complete guides for integrating GenLayer with popular tools, platforms, and services.

## Table of Contents

1. [Frontend Integration](#frontend-integration)
2. [API Integrations](#api-integrations)
3. [Database Integration](#database-integration)
4. [Authentication](#authentication)
5. [Monitoring & Analytics](#monitoring--analytics)
6. [CI/CD Integration](#cicd-integration)

---

## Frontend Integration

### 1. React Integration

```javascript
// Install GenLayer SDK
// npm install @genlayer/sdk

import { GenLayerSDK } from '@genlayer/sdk';

const sdk = new GenLayerSDK({
  rpcUrl: 'https://rpc.genlayer.com',
  chainId: 1
});

// Deploy contract
async function deployContract() {
  const contract = await sdk.deployContract({
    name: 'MyContract',
    code: contractCode
  });
  
  return contract.address;
}

// Call contract method
async function callMethod(contractAddress, methodName, args) {
  const contract = await sdk.getContract(contractAddress);
  const result = await contract.methods[methodName](...args).call();
  return result;
}

// React component example
import React, { useState, useEffect } from 'react';

function ContractInteraction() {
  const [contractAddress, setContractAddress] = useState('');
  const [result, setResult] = useState(null);
  
  useEffect(() => {
    deployContract().then(setContractAddress);
  }, []);
  
  const handleCall = async () => {
    const res = await callMethod(contractAddress, 'myMethod', []);
    setResult(res);
  };
  
  return (
    <div>
      <button onClick={handleCall}>Call Contract</button>
      {result && <p>Result: {JSON.stringify(result)}</p>}
    </div>
  );
}

export default ContractInteraction;
```

### 2. Vue.js Integration

```javascript
// Vue 3 composition API
import { ref, onMounted } from 'vue';
import { GenLayerSDK } from '@genlayer/sdk';

export default {
  setup() {
    const sdk = new GenLayerSDK();
    const contractAddress = ref('');
    const result = ref(null);
    
    onMounted(async () => {
      const contract = await sdk.deployContract({
        name: 'MyContract',
        code: contractCode
      });
      contractAddress.value = contract.address;
    });
    
    const callMethod = async (methodName) => {
      const contract = await sdk.getContract(contractAddress.value);
      result.value = await contract.methods[methodName]().call();
    };
    
    return {
      contractAddress,
      result,
      callMethod
    };
  }
};
```

### 3. Web3.js Integration

```javascript
// Integration with Web3.js
import Web3 from 'web3';
import { GenLayerSDK } from '@genlayer/sdk';

const web3 = new Web3('https://rpc.genlayer.com');
const sdk = new GenLayerSDK({ web3 });

// Get account
async function getAccount() {
  const accounts = await web3.eth.getAccounts();
  return accounts[0];
}

// Send transaction
async function sendTransaction(contractAddress, methodName, args) {
  const account = await getAccount();
  const contract = await sdk.getContract(contractAddress);
  
  const tx = await contract.methods[methodName](...args).send({
    from: account,
    gas: 300000
  });
  
  return tx;
}

// Listen to events
async function listenToEvents(contractAddress, eventName) {
  const contract = await sdk.getContract(contractAddress);
  
  contract.events[eventName]()
    .on('data', (event) => {
      console.log('Event:', event);
    })
    .on('error', (error) => {
      console.error('Error:', error);
    });
}
```

---

## API Integrations

### 1. OpenAI Integration

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *
import os

class OpenAIIntegratedContract(gl.Contract):
    """Contract using OpenAI for intelligence"""
    
    api_key: str
    model: str
    
    def __init__(self):
        self.api_key = os.getenv('OPENAI_API_KEY')
        self.model = 'gpt-4'
    
    @gl.public.write
    def analyze_with_openai(self, text: str):
        """Analyze text using OpenAI"""
        
        prompt = f"""Analyze this text and provide insights:
        
{text}

Provide a concise analysis."""
        
        response = gl.eq_principle.prompt_non_comparative(
            lambda: prompt,
            task="Analyze with OpenAI",
            criteria="Response must be analysis of the text"
        )
        
        return response
```

### 2. REST API Integration

```python
class RESTAPIIntegratedContract(gl.Contract):
    """Contract integrating with REST APIs"""
    
    api_endpoints: dict[str, str]
    
    def __init__(self):
        self.api_endpoints = {
            'weather': 'https://api.weather.com/current',
            'stocks': 'https://api.stocks.com/quote',
            'crypto': 'https://api.coingecko.com/api/v3/simple/price'
        }
    
    @gl.public.write
    def fetch_weather(self, city: str):
        """Fetch weather data from REST API"""
        
        url = f"{self.api_endpoints['weather']}?city={city}"
        
        data = gl.nondet.web.get(url)
        
        # Parse and store weather data
        return data
    
    @gl.public.write
    def fetch_stock_price(self, symbol: str):
        """Fetch stock price"""
        
        url = f"{self.api_endpoints['stocks']}?symbol={symbol}"
        
        data = gl.nondet.web.get(url)
        
        return data
```

### 3. GraphQL Integration

```python
class GraphQLIntegratedContract(gl.Contract):
    """Contract using GraphQL APIs"""
    
    graphql_endpoint: str
    
    def __init__(self):
        self.graphql_endpoint = 'https://api.example.com/graphql'
    
    @gl.public.write
    def query_graphql(self, query: str):
        """Execute GraphQL query"""
        
        def fetch_graphql():
            # Construct GraphQL request
            request_body = f'{{"query": "{query}"}}'
            
            # Fetch from GraphQL endpoint
            response = gl.nondet.web.get(self.graphql_endpoint)
            
            return response
        
        result = fetch_graphql()
        
        return result
```

---

## Database Integration

### 1. Data Persistence Pattern

```python
class DatabaseIntegratedContract(gl.Contract):
    """Pattern for database integration"""
    
    database_url: str
    records: dict[str, dict]
    
    def __init__(self, db_url: str):
        self.database_url = db_url
        self.records = {}
    
    @gl.public.write
    def save_record(self, record_id: str, data: dict):
        """Save record to on-chain storage"""
        
        # Store on-chain
        self.records[record_id] = {
            'data': data,
            'timestamp': gl.block.timestamp,
            'hash': self._compute_hash(data)
        }
    
    @gl.public.write
    def sync_to_database(self, record_id: str):
        """Sync on-chain record to external database"""
        
        if record_id not in self.records:
            return False
        
        record = self.records[record_id]
        
        # In production, would sync to database
        # For now, just verify integrity
        
        return True
    
    def _compute_hash(self, data: dict) -> str:
        """Compute hash of data"""
        import hashlib
        import json
        data_str = json.dumps(data, sort_keys=True)
        return hashlib.sha256(data_str.encode()).hexdigest()
```

### 2. Event Logging

```python
class EventLoggingContract(gl.Contract):
    """Log contract events for database sync"""
    
    events_log: list[dict]
    
    def __init__(self):
        self.events_log = []
    
    def _log_event(self, event_type: str, data: dict):
        """Log event for external database"""
        
        event = {
            'type': event_type,
            'data': data,
            'timestamp': gl.block.timestamp,
            'block_number': gl.block.number,
            'sender': msg.sender
        }
        
        self.events_log.append(event)
    
    @gl.public.write
    def perform_action(self, action: str):
        """Perform action and log event"""
        
        # Perform action
        result = self._do_action(action)
        
        # Log event
        self._log_event('action_performed', {
            'action': action,
            'result': result
        })
        
        return result
    
    def _do_action(self, action: str):
        pass
    
    @gl.public.view
    def get_events_log(self) -> list:
        """Get all logged events"""
        return self.events_log
```

---

## Authentication

### 1. Web3 Wallet Authentication

```javascript
// Frontend authentication with Web3 wallet
import { GenLayerSDK } from '@genlayer/sdk';

async function authenticateWithWallet() {
  // Request wallet connection
  const accounts = await window.ethereum.request({
    method: 'eth_requestAccounts'
  });
  
  const userAddress = accounts[0];
  
  // Sign message for authentication
  const message = `Sign this message to authenticate: ${Date.now()}`;
  
  const signature = await window.ethereum.request({
    method: 'personal_sign',
    params: [message, userAddress]
  });
  
  // Verify signature on backend
  const isValid = await verifySignature(userAddress, message, signature);
  
  return { userAddress, signature, isValid };
}

async function verifySignature(address, message, signature) {
  // Send to backend for verification
  const response = await fetch('/api/verify-signature', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ address, message, signature })
  });
  
  const result = await response.json();
  return result.valid;
}
```

### 2. JWT Authentication

```python
class JWTAuthenticatedContract(gl.Contract):
    """Contract with JWT authentication"""
    
    authorized_users: dict[str, dict]
    
    def __init__(self):
        self.authorized_users = {}
    
    @gl.public.write
    def authenticate_user(self, user_id: str, jwt_token: str):
        """Authenticate user with JWT"""
        
        # Verify JWT token
        if self._verify_jwt(jwt_token):
            self.authorized_users[user_id] = {
                'authenticated': True,
                'timestamp': gl.block.timestamp
            }
            return True
        
        return False
    
    def _verify_jwt(self, token: str) -> bool:
        """Verify JWT token"""
        # Implementation depends on JWT library
        return True
    
    def _require_auth(self, user_id: str):
        """Require user to be authenticated"""
        assert user_id in self.authorized_users, "User not authenticated"
    
    @gl.public.write
    def protected_method(self, user_id: str):
        """Method requiring authentication"""
        self._require_auth(user_id)
        # Protected logic
        pass
```

---

## Monitoring & Analytics

### 1. Event Tracking

```python
class EventTrackingContract(gl.Contract):
    """Track contract events for analytics"""
    
    event_metrics: dict[str, dict]
    
    def __init__(self):
        self.event_metrics = {}
    
    def _track_event(self, event_name: str, data: dict):
        """Track event for analytics"""
        
        if event_name not in self.event_metrics:
            self.event_metrics[event_name] = {
                'count': 0,
                'first_occurrence': gl.block.timestamp,
                'last_occurrence': gl.block.timestamp,
                'data': []
            }
        
        metrics = self.event_metrics[event_name]
        metrics['count'] += 1
        metrics['last_occurrence'] = gl.block.timestamp
        metrics['data'].append(data)
    
    @gl.public.view
    def get_metrics(self, event_name: str) -> dict:
        """Get event metrics"""
        return self.event_metrics.get(event_name, {})
```

### 2. Performance Monitoring

```python
class PerformanceMonitoringContract(gl.Contract):
    """Monitor contract performance"""
    
    method_calls: dict[str, dict]
    
    def __init__(self):
        self.method_calls = {}
    
    def _record_method_call(self, method_name: str, execution_time: float):
        """Record method call performance"""
        
        if method_name not in self.method_calls:
            self.method_calls[method_name] = {
                'total_calls': 0,
                'total_time': 0,
                'avg_time': 0,
                'min_time': float('inf'),
                'max_time': 0
            }
        
        metrics = self.method_calls[method_name]
        metrics['total_calls'] += 1
        metrics['total_time'] += execution_time
        metrics['avg_time'] = metrics['total_time'] / metrics['total_calls']
        metrics['min_time'] = min(metrics['min_time'], execution_time)
        metrics['max_time'] = max(metrics['max_time'], execution_time)
    
    @gl.public.view
    def get_performance_metrics(self, method_name: str) -> dict:
        """Get performance metrics for method"""
        return self.method_calls.get(method_name, {})
```

---

## CI/CD Integration

### 1. GitHub Actions Integration

```yaml
# .github/workflows/deploy.yml
name: Deploy GenLayer Contract

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          pip install genlayer-sdk
          pip install -r requirements.txt
      
      - name: Run tests
        run: |
          python -m pytest tests/
      
      - name: Deploy contract
        env:
          GENLAYER_PRIVATE_KEY: ${{ secrets.GENLAYER_PRIVATE_KEY }}
        run: |
          python scripts/deploy.py
      
      - name: Verify deployment
        run: |
          python scripts/verify.py
```

### 2. Automated Testing

```python
# tests/test_contract.py
import pytest
from genlayer import *

class TestMyContract:
    
    def setup_method(self):
        self.contract = MyContract()
    
    def test_initialization(self):
        assert self.contract is not None
    
    def test_basic_operation(self):
        result = self.contract.some_method()
        assert result is not None
    
    def test_state_consistency(self):
        self.contract.update_state()
        assert self.contract._check_consistency()
    
    def test_error_handling(self):
        with pytest.raises(AssertionError):
            self.contract.invalid_operation()
```

### 3. Continuous Monitoring

```python
# monitoring/monitor.py
import requests
import time

def monitor_contract(contract_address):
    """Continuously monitor contract"""
    
    while True:
        try:
            # Get contract state
            state = get_contract_state(contract_address)
            
            # Check health
            if not is_healthy(state):
                send_alert(f"Contract unhealthy: {contract_address}")
            
            # Log metrics
            log_metrics(contract_address, state)
            
            time.sleep(60)  # Check every minute
            
        except Exception as e:
            send_alert(f"Monitoring error: {str(e)}")
            time.sleep(60)

def is_healthy(state):
    """Check if contract state is healthy"""
    return state.get('status') == 'active'

def send_alert(message):
    """Send alert notification"""
    requests.post('https://alerts.example.com/notify', json={'message': message})

def log_metrics(contract_address, state):
    """Log contract metrics"""
    print(f"Contract {contract_address}: {state}")
```

---

## Best Practices for Integration

### 1. Error Handling

```python
async def integrate_with_external_service(contract, service_url):
    """Integrate with error handling"""
    
    try:
        # Call external service
        response = await fetch_from_service(service_url)
        
        # Process response
        result = process_response(response)
        
        # Update contract
        contract.update_state(result)
        
    except TimeoutError:
        # Handle timeout
        contract.use_cached_data()
    
    except ConnectionError:
        # Handle connection error
        contract.use_fallback_data()
    
    except Exception as e:
        # Handle unexpected error
        log_error(str(e))
        contract.set_error_state()
```

### 2. Rate Limiting

```python
class RateLimitedIntegration:
    """Rate-limited external service integration"""
    
    def __init__(self, max_calls_per_minute=60):
        self.max_calls = max_calls_per_minute
        self.call_times = []
    
    async def call_service(self, service_url):
        """Call service with rate limiting"""
        
        current_time = time.time()
        
        # Remove old calls
        self.call_times = [t for t in self.call_times if current_time - t < 60]
        
        # Check rate limit
        if len(self.call_times) >= self.max_calls:
            raise RateLimitError("Rate limit exceeded")
        
        # Make call
        result = await fetch_from_service(service_url)
        
        # Record call
        self.call_times.append(current_time)
        
        return result
```

---

**Last Updated:** March 2026  
**Author:** kuru  
**Version:** 1.0
