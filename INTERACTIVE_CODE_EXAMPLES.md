# GenLayer Intelligent Contracts: Interactive Code Examples

A collection of advanced, production-ready code examples demonstrating GenLayer's capabilities with detailed explanations.

## Table of Contents

1. [Web Data Integration](#web-data-integration)
2. [LLM-Powered Decision Making](#llm-powered-decision-making)
3. [Complex State Management](#complex-state-management)
4. [Error Handling & Recovery](#error-handling--recovery)
5. [Performance Optimization](#performance-optimization)
6. [Real-Time Data Processing](#real-time-data-processing)

---

## Web Data Integration

### Example 1: Real-Time Price Tracking

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *
from datetime import datetime

class PriceTrackerContract(gl.Contract):
    """Track cryptocurrency prices and trigger alerts"""
    
    prices: dict[str, dict]
    alerts: dict[str, list]
    
    def __init__(self):
        self.prices = {}
        self.alerts = {}
    
    @gl.public.write
    def track_crypto_price(self, symbol: str, threshold: float):
        """Track a cryptocurrency and set price threshold"""
        
        def fetch_price():
            # Fetch current price from CoinGecko API
            response = gl.nondet.web.get(
                f'https://api.coingecko.com/api/v3/simple/price?ids={symbol.lower()}&vs_currencies=usd'
            )
            return response
        
        price_data = fetch_price()
        
        # Parse and store price
        self.prices[symbol] = {
            'current_price': price_data.get(symbol.lower(), {}).get('usd', 0),
            'threshold': threshold,
            'timestamp': gl.block.timestamp,
            'alert_triggered': False
        }
        
        # Check if threshold exceeded
        if self.prices[symbol]['current_price'] > threshold:
            self._trigger_alert(symbol)
    
    def _trigger_alert(self, symbol: str):
        """Internal method to trigger price alert"""
        if symbol not in self.alerts:
            self.alerts[symbol] = []
        
        self.alerts[symbol].append({
            'timestamp': gl.block.timestamp,
            'price': self.prices[symbol]['current_price'],
            'threshold': self.prices[symbol]['threshold']
        })
        
        self.prices[symbol]['alert_triggered'] = True
    
    @gl.public.view
    def get_price_status(self, symbol: str) -> dict:
        """Get current price and alert status"""
        return self.prices.get(symbol, {})
    
    @gl.public.view
    def get_alerts(self, symbol: str) -> list:
        """Get all price alerts for a symbol"""
        return self.alerts.get(symbol, [])
```

**Key Concepts:**
- Uses `gl.nondet.web.get()` for real-time API calls
- Stores structured data with timestamps
- Implements internal helper methods
- Provides view methods for data retrieval

---

### Example 2: Multi-Source Data Aggregation

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class DataAggregatorContract(gl.Contract):
    """Aggregate data from multiple sources"""
    
    aggregated_data: dict[str, dict]
    source_reliability: dict[str, float]
    
    def __init__(self):
        self.aggregated_data = {}
        self.source_reliability = {
            'source_a': 0.95,
            'source_b': 0.88,
            'source_c': 0.92
        }
    
    @gl.public.write
    def aggregate_data(self, data_key: str):
        """Fetch and aggregate data from multiple sources"""
        
        sources = {
            'source_a': 'https://api.example.com/data-a',
            'source_b': 'https://api.example.com/data-b',
            'source_c': 'https://api.example.com/data-c'
        }
        
        collected_data = {}
        
        # Fetch from all sources
        for source_name, url in sources.items():
            try:
                data = gl.nondet.web.get(url)
                collected_data[source_name] = {
                    'data': data,
                    'reliability': self.source_reliability[source_name],
                    'timestamp': gl.block.timestamp
                }
            except Exception as e:
                collected_data[source_name] = {
                    'error': str(e),
                    'reliability': 0
                }
        
        # Aggregate with weighted average
        self.aggregated_data[data_key] = self._weighted_aggregate(collected_data)
    
    def _weighted_aggregate(self, sources: dict) -> dict:
        """Calculate weighted aggregate from multiple sources"""
        
        total_weight = 0
        weighted_sum = 0
        
        for source_name, source_info in sources.items():
            if 'error' not in source_info:
                weight = source_info['reliability']
                # Assuming numeric data for aggregation
                value = float(source_info['data'].get('value', 0))
                weighted_sum += value * weight
                total_weight += weight
        
        return {
            'aggregated_value': weighted_sum / total_weight if total_weight > 0 else 0,
            'sources_used': len([s for s in sources.values() if 'error' not in s]),
            'total_sources': len(sources),
            'timestamp': gl.block.timestamp
        }
    
    @gl.public.view
    def get_aggregated_data(self, data_key: str) -> dict:
        """Retrieve aggregated data"""
        return self.aggregated_data.get(data_key, {})
```

**Key Concepts:**
- Fetches from multiple sources simultaneously
- Implements error handling for failed requests
- Uses weighted aggregation for data fusion
- Tracks source reliability metrics

---

## LLM-Powered Decision Making

### Example 3: Intelligent Content Classification

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class ContentClassifierContract(gl.Contract):
    """Classify content using LLM with validation"""
    
    classifications: dict[str, dict]
    classification_stats: dict[str, int]
    
    def __init__(self):
        self.classifications = {}
        self.classification_stats = {
            'educational': 0,
            'entertainment': 0,
            'news': 0,
            'technical': 0,
            'other': 0
        }
    
    @gl.public.write
    def classify_content(self, content_id: str, content_url: str):
        """Classify content using LLM"""
        
        def fetch_and_classify():
            # Fetch content
            content = gl.nondet.web.get(content_url)
            
            # Create classification prompt
            prompt = f"""Analyze this content and classify it into ONE category:
            
Content:
{content[:500]}  # Limit to first 500 chars

Categories: educational, entertainment, news, technical, other

Respond with ONLY the category name."""
            
            return prompt
        
        # Use non-comparative validation for LLM output
        classification = gl.eq_principle.prompt_non_comparative(
            fetch_and_classify,
            task="Classify content into predefined categories",
            criteria="Response must be exactly one of: educational, entertainment, news, technical, other"
        )
        
        # Extract category from response
        category = self._extract_category(classification)
        
        # Store classification
        self.classifications[content_id] = {
            'category': category,
            'raw_response': classification,
            'timestamp': gl.block.timestamp,
            'url': content_url
        }
        
        # Update statistics
        if category in self.classification_stats:
            self.classification_stats[category] += 1
    
    def _extract_category(self, response: str) -> str:
        """Extract category from LLM response"""
        
        categories = ['educational', 'entertainment', 'news', 'technical', 'other']
        response_lower = response.lower()
        
        for category in categories:
            if category in response_lower:
                return category
        
        return 'other'
    
    @gl.public.view
    def get_classification(self, content_id: str) -> dict:
        """Get classification for content"""
        return self.classifications.get(content_id, {})
    
    @gl.public.view
    def get_statistics(self) -> dict:
        """Get classification statistics"""
        return self.classification_stats
```

**Key Concepts:**
- Uses `gl.eq_principle.prompt_non_comparative()` for LLM calls
- Defines clear validation criteria
- Implements response parsing logic
- Tracks statistics across classifications

---

### Example 4: Sentiment Analysis with Confidence Scoring

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class SentimentAnalyzerContract(gl.Contract):
    """Analyze sentiment with confidence scoring"""
    
    sentiment_records: dict[str, dict]
    confidence_threshold: float
    
    def __init__(self):
        self.sentiment_records = {}
        self.confidence_threshold = 0.7
    
    @gl.public.write
    def analyze_sentiment(self, text_id: str, text_content: str):
        """Analyze sentiment of text content"""
        
        def analyze():
            prompt = f"""Analyze the sentiment of this text and provide:
1. Sentiment (positive, negative, neutral)
2. Confidence score (0.0 to 1.0)
3. Key sentiment indicators

Text: "{text_content}"

Format your response as:
SENTIMENT: [sentiment]
CONFIDENCE: [score]
INDICATORS: [list of indicators]"""
            
            return prompt
        
        response = gl.eq_principle.prompt_non_comparative(
            analyze,
            task="Analyze text sentiment with confidence scoring",
            criteria="Response must include SENTIMENT, CONFIDENCE, and INDICATORS fields"
        )
        
        # Parse response
        parsed = self._parse_sentiment_response(response)
        
        # Store if confidence exceeds threshold
        if parsed['confidence'] >= self.confidence_threshold:
            self.sentiment_records[text_id] = {
                'sentiment': parsed['sentiment'],
                'confidence': parsed['confidence'],
                'indicators': parsed['indicators'],
                'raw_response': response,
                'timestamp': gl.block.timestamp,
                'accepted': True
            }
        else:
            self.sentiment_records[text_id] = {
                'sentiment': parsed['sentiment'],
                'confidence': parsed['confidence'],
                'indicators': parsed['indicators'],
                'raw_response': response,
                'timestamp': gl.block.timestamp,
                'accepted': False,
                'reason': 'Confidence below threshold'
            }
    
    def _parse_sentiment_response(self, response: str) -> dict:
        """Parse sentiment analysis response"""
        
        lines = response.split('\n')
        result = {
            'sentiment': 'neutral',
            'confidence': 0.0,
            'indicators': []
        }
        
        for line in lines:
            if 'SENTIMENT:' in line:
                sentiment = line.split('SENTIMENT:')[1].strip().lower()
                if sentiment in ['positive', 'negative', 'neutral']:
                    result['sentiment'] = sentiment
            elif 'CONFIDENCE:' in line:
                try:
                    confidence = float(line.split('CONFIDENCE:')[1].strip())
                    result['confidence'] = min(1.0, max(0.0, confidence))
                except:
                    result['confidence'] = 0.0
            elif 'INDICATORS:' in line:
                indicators = line.split('INDICATORS:')[1].strip()
                result['indicators'] = [i.strip() for i in indicators.split(',')]
        
        return result
    
    @gl.public.view
    def get_sentiment(self, text_id: str) -> dict:
        """Get sentiment analysis result"""
        return self.sentiment_records.get(text_id, {})
```

**Key Concepts:**
- Implements confidence-based filtering
- Parses complex LLM responses
- Stores validation metadata
- Tracks acceptance criteria

---

## Complex State Management

### Example 5: Multi-Level Approval Workflow

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class ApprovalWorkflowContract(gl.Contract):
    """Multi-level approval workflow with state transitions"""
    
    requests: dict[str, dict]
    approvers: list[str]
    
    def __init__(self):
        self.requests = {}
        self.approvers = []
    
    @gl.public.write
    def create_request(self, request_id: str, description: str, amount: float):
        """Create a new approval request"""
        
        self.requests[request_id] = {
            'description': description,
            'amount': amount,
            'status': 'pending',
            'approvals': [],
            'rejections': [],
            'created_at': gl.block.timestamp,
            'updated_at': gl.block.timestamp
        }
    
    @gl.public.write
    def approve_request(self, request_id: str, approver_id: str, comments: str = ""):
        """Approve a request"""
        
        if request_id not in self.requests:
            return
        
        request = self.requests[request_id]
        
        # Check if already approved by this approver
        if any(a['approver_id'] == approver_id for a in request['approvals']):
            return
        
        # Add approval
        request['approvals'].append({
            'approver_id': approver_id,
            'timestamp': gl.block.timestamp,
            'comments': comments
        })
        
        # Check if all approvers have approved
        if len(request['approvals']) >= len(self.approvers):
            request['status'] = 'approved'
        
        request['updated_at'] = gl.block.timestamp
    
    @gl.public.write
    def reject_request(self, request_id: str, approver_id: str, reason: str):
        """Reject a request"""
        
        if request_id not in self.requests:
            return
        
        request = self.requests[request_id]
        
        # Add rejection
        request['rejections'].append({
            'approver_id': approver_id,
            'timestamp': gl.block.timestamp,
            'reason': reason
        })
        
        # Mark as rejected
        request['status'] = 'rejected'
        request['updated_at'] = gl.block.timestamp
    
    @gl.public.view
    def get_request_status(self, request_id: str) -> dict:
        """Get request status and approval progress"""
        
        if request_id not in self.requests:
            return {}
        
        request = self.requests[request_id]
        
        return {
            'status': request['status'],
            'approvals': len(request['approvals']),
            'required_approvals': len(self.approvers),
            'rejections': len(request['rejections']),
            'approval_percentage': (len(request['approvals']) / len(self.approvers) * 100) if self.approvers else 0
        }
```

**Key Concepts:**
- Manages complex state transitions
- Tracks multiple approval stages
- Implements conditional logic based on state
- Provides progress tracking

---

## Error Handling & Recovery

### Example 6: Resilient Data Fetching with Fallbacks

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class ResilientDataFetcherContract(gl.Contract):
    """Fetch data with fallback mechanisms"""
    
    data_cache: dict[str, dict]
    fallback_data: dict[str, str]
    
    def __init__(self):
        self.data_cache = {}
        self.fallback_data = {}
    
    @gl.public.write
    def fetch_with_fallback(self, data_key: str, primary_url: str, fallback_url: str):
        """Fetch data with fallback URL"""
        
        primary_data = None
        fallback_data = None
        used_source = None
        
        # Try primary source
        try:
            primary_data = gl.nondet.web.get(primary_url)
            used_source = 'primary'
        except Exception as e:
            # Try fallback source
            try:
                fallback_data = gl.nondet.web.get(fallback_url)
                used_source = 'fallback'
            except Exception as e2:
                # Use cached data if available
                if data_key in self.data_cache:
                    used_source = 'cache'
                else:
                    used_source = 'none'
        
        # Store result
        self.data_cache[data_key] = {
            'data': primary_data or fallback_data or self.data_cache.get(data_key, {}).get('data'),
            'source': used_source,
            'timestamp': gl.block.timestamp,
            'primary_url': primary_url,
            'fallback_url': fallback_url
        }
    
    @gl.public.view
    def get_data(self, data_key: str) -> dict:
        """Get cached data with source information"""
        return self.data_cache.get(data_key, {})
```

**Key Concepts:**
- Implements try-except error handling
- Uses fallback mechanisms
- Maintains data cache for recovery
- Tracks data source lineage

---

## Performance Optimization

### Example 7: Batch Processing with Caching

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class BatchProcessorContract(gl.Contract):
    """Process data in batches with caching"""
    
    processed_items: dict[str, dict]
    batch_cache: dict[str, list]
    cache_ttl: int
    
    def __init__(self):
        self.processed_items = {}
        self.batch_cache = {}
        self.cache_ttl = 3600  # 1 hour in seconds
    
    @gl.public.write
    def process_batch(self, batch_id: str, items: list[str]):
        """Process multiple items in a batch"""
        
        results = []
        
        for item in items:
            # Check cache first
            if item in self.processed_items:
                cached = self.processed_items[item]
                # Check if cache is still valid
                if gl.block.timestamp - cached['timestamp'] < self.cache_ttl:
                    results.append({
                        'item': item,
                        'result': cached['result'],
                        'source': 'cache'
                    })
                    continue
            
            # Process item
            processed_result = self._process_item(item)
            
            # Cache result
            self.processed_items[item] = {
                'result': processed_result,
                'timestamp': gl.block.timestamp
            }
            
            results.append({
                'item': item,
                'result': processed_result,
                'source': 'processed'
            })
        
        # Store batch results
        self.batch_cache[batch_id] = results
    
    def _process_item(self, item: str) -> dict:
        """Process a single item"""
        # Simulate processing
        return {
            'processed': True,
            'item': item,
            'timestamp': gl.block.timestamp
        }
    
    @gl.public.view
    def get_batch_results(self, batch_id: str) -> list:
        """Get batch processing results"""
        return self.batch_cache.get(batch_id, [])
```

**Key Concepts:**
- Implements caching with TTL
- Processes items in batches
- Tracks cache hits vs. fresh processing
- Optimizes repeated operations

---

## Real-Time Data Processing

### Example 8: Event Stream Processing

```python
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
from genlayer import *

class EventStreamProcessorContract(gl.Contract):
    """Process real-time event streams"""
    
    events: list[dict]
    event_stats: dict[str, int]
    last_processed_timestamp: int
    
    def __init__(self):
        self.events = []
        self.event_stats = {}
        self.last_processed_timestamp = 0
    
    @gl.public.write
    def process_event_stream(self, stream_url: str):
        """Fetch and process event stream"""
        
        def fetch_stream():
            return gl.nondet.web.get(stream_url)
        
        stream_data = fetch_stream()
        
        # Parse events
        if isinstance(stream_data, list):
            for event in stream_data:
                self._process_event(event)
        
        self.last_processed_timestamp = gl.block.timestamp
    
    def _process_event(self, event: dict):
        """Process individual event"""
        
        event_type = event.get('type', 'unknown')
        
        # Update statistics
        if event_type not in self.event_stats:
            self.event_stats[event_type] = 0
        self.event_stats[event_type] += 1
        
        # Store event with metadata
        self.events.append({
            'type': event_type,
            'data': event,
            'processed_at': gl.block.timestamp,
            'index': len(self.events)
        })
    
    @gl.public.view
    def get_event_statistics(self) -> dict:
        """Get event processing statistics"""
        
        return {
            'total_events': len(self.events),
            'event_types': self.event_stats,
            'last_processed': self.last_processed_timestamp
        }
    
    @gl.public.view
    def get_recent_events(self, limit: int = 10) -> list:
        """Get most recent events"""
        return self.events[-limit:] if limit > 0 else []
```

**Key Concepts:**
- Processes event streams in real-time
- Maintains event statistics
- Tracks processing timestamps
- Provides event retrieval with limits

---

## Best Practices Summary

### 1. Error Handling
```python
try:
    data = gl.nondet.web.get(url)
except Exception as e:
    # Handle error gracefully
    data = self.fallback_data
```

### 2. LLM Response Parsing
```python
response = gl.eq_principle.prompt_non_comparative(
    prompt_func,
    task="Clear task description",
    criteria="Specific validation criteria"
)
# Always parse and validate response
parsed = self._parse_response(response)
```

### 3. State Management
```python
# Use dictionaries for flexible data storage
self.data[key] = {
    'value': value,
    'timestamp': gl.block.timestamp,
    'metadata': {}
}
```

### 4. Performance
```python
# Cache expensive operations
if key in self.cache:
    return self.cache[key]

result = expensive_operation()
self.cache[key] = result
```

---

## Conclusion

These examples demonstrate advanced patterns for building production-ready Intelligent Contracts. Key takeaways:

- Use appropriate error handling for external calls
- Implement caching for performance
- Parse LLM responses carefully
- Manage complex state with clear data structures
- Track metadata for debugging and analytics

---

**Last Updated:** March 2026  
**Author:** kuru  
**Version:** 1.0
