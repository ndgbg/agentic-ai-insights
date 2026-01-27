# Error Handling Strategies for Production Agents

Agents fail. Networks timeout. APIs return errors. LLMs hallucinate. Production agent systems need robust error handling.

## The Challenge

Unlike traditional software, agents introduce unique failure modes:
- **Non-deterministic behavior** - Same input, different output
- **Tool failures** - External APIs unavailable
- **Reasoning failures** - Agent chooses wrong tool
- **Partial failures** - Multi-step workflows fail mid-execution
- **Context limits** - Conversation exceeds token limits

## Core Strategies

### 1. Graceful Degradation

Provide reduced functionality instead of complete failure.

```python
@tool
def get_customer_insights(customer_id: str) -> dict:
    """Get comprehensive customer insights"""
    insights = {}
    
    # Try to get purchase history
    try:
        insights["purchases"] = get_purchase_history(customer_id)
    except Exception as e:
        insights["purchases"] = {"error": "Purchase history unavailable"}
        log_error("purchase_history_failed", e)
    
    # Try to get support tickets (independent)
    try:
        insights["support"] = get_support_tickets(customer_id)
    except Exception as e:
        insights["support"] = {"error": "Support data unavailable"}
        log_error("support_tickets_failed", e)
    
    # Always return something useful
    insights["customer_id"] = customer_id
    insights["retrieved_at"] = datetime.now().isoformat()
    
    return insights
```

### 2. Retry with Exponential Backoff

Automatically retry transient failures.

```python
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type
)

@tool
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type((ConnectionError, TimeoutError))
)
def call_external_api(endpoint: str) -> dict:
    """Call external API with retries"""
    response = requests.get(endpoint, timeout=10)
    response.raise_for_status()
    return response.json()
```

### 3. Circuit Breaker

Stop calling failing services to prevent cascading failures.

```python
from circuitbreaker import circuit

@tool
@circuit(failure_threshold=5, recovery_timeout=60, expected_exception=RequestException)
def query_analytics_service(query: str) -> dict:
    """
    Query analytics service with circuit breaker.
    
    After 5 failures, circuit opens for 60 seconds.
    Prevents overwhelming a failing service.
    """
    return analytics_api.query(query)
```

### 4. Fallback Chains

Try multiple approaches in sequence.

```python
@tool
def get_product_info(product_id: str) -> dict:
    """Get product information with fallbacks"""
    
    # Try primary database
    try:
        return primary_db.get_product(product_id)
    except Exception as e:
        log_warning("primary_db_failed", e)
    
    # Fallback to cache
    try:
        cached = cache.get(f"product:{product_id}")
        if cached:
            return {**cached, "source": "cache"}
    except Exception as e:
        log_warning("cache_failed", e)
    
    # Fallback to replica database
    try:
        return replica_db.get_product(product_id)
    except Exception as e:
        log_error("all_sources_failed", e)
    
    # Final fallback
    return {
        "error": "Product information temporarily unavailable",
        "product_id": product_id
    }
```

### 5. Timeout Protection

Prevent hanging operations.

```python
from timeout_decorator import timeout

@tool
@timeout(30)  # 30 second timeout
def process_large_dataset(data: str) -> dict:
    """Process dataset with timeout protection"""
    # Long-running operation
    results = expensive_computation(data)
    return results
```

## Agent-Level Error Handling

### Pattern 1: Structured Error Messages

Help agents understand and recover from errors.

```python
class ToolError:
    def __init__(self, error_type: str, message: str, recoverable: bool, suggestion: str = None):
        self.error_type = error_type
        self.message = message
        self.recoverable = recoverable
        self.suggestion = suggestion
    
    def to_dict(self):
        return {
            "error": self.error_type,
            "message": self.message,
            "recoverable": self.recoverable,
            "suggestion": self.suggestion
        }

@tool
def update_inventory(product_id: str, quantity: int) -> dict:
    """Update product inventory"""
    
    if quantity < 0:
        return ToolError(
            error_type="invalid_input",
            message="Quantity cannot be negative",
            recoverable=True,
            suggestion="Use a positive quantity value"
        ).to_dict()
    
    try:
        result = inventory_db.update(product_id, quantity)
        return {"success": True, "new_quantity": result}
    except ProductNotFound:
        return ToolError(
            error_type="not_found",
            message=f"Product {product_id} not found",
            recoverable=True,
            suggestion="Verify the product ID and try again"
        ).to_dict()
    except DatabaseError as e:
        return ToolError(
            error_type="database_error",
            message="Database temporarily unavailable",
            recoverable=True,
            suggestion="Try again in a few moments"
        ).to_dict()
```

### Pattern 2: Error Recovery Prompts

Guide agents to recover from errors.

```python
def invoke_agent_with_recovery(agent, user_message: str, max_retries: int = 2):
    """Invoke agent with automatic error recovery"""
    
    for attempt in range(max_retries + 1):
        try:
            result = agent(user_message)
            
            # Check if result contains errors
            if "error" in result.message.lower():
                if attempt < max_retries:
                    # Add recovery guidance
                    user_message = f"""
                    Previous attempt failed with: {result.message}
                    
                    Please try a different approach:
                    - Use alternative tools if available
                    - Break the task into smaller steps
                    - Ask for clarification if needed
                    
                    Original request: {user_message}
                    """
                    continue
            
            return result
            
        except Exception as e:
            if attempt < max_retries:
                user_message = f"""
                An error occurred: {str(e)}
                
                Please try again with a simpler approach.
                Original request: {user_message}
                """
                continue
            else:
                raise
    
    return {"error": "Max retries exceeded"}
```

### Pattern 3: Partial Success Handling

Handle multi-step workflows that partially succeed.

```python
@tool
def process_order_batch(order_ids: list[str]) -> dict:
    """Process multiple orders, tracking successes and failures"""
    results = {
        "successful": [],
        "failed": [],
        "total": len(order_ids)
    }
    
    for order_id in order_ids:
        try:
            process_order(order_id)
            results["successful"].append(order_id)
        except Exception as e:
            results["failed"].append({
                "order_id": order_id,
                "error": str(e)
            })
    
    results["success_rate"] = len(results["successful"]) / results["total"]
    
    return results
```

## Monitoring and Observability

### Pattern 1: Structured Logging

```python
import structlog

logger = structlog.get_logger()

@tool
def critical_operation(data: str) -> dict:
    """Operation with comprehensive logging"""
    
    logger.info("operation_started", operation="critical_operation", data_size=len(data))
    
    try:
        result = perform_operation(data)
        logger.info("operation_succeeded", result_size=len(result))
        return result
        
    except ValidationError as e:
        logger.warning("validation_failed", error=str(e), data=data)
        return {"error": "Invalid input"}
        
    except Exception as e:
        logger.error("operation_failed", error=str(e), exc_info=True)
        raise
```

### Pattern 2: Metrics Collection

```python
from prometheus_client import Counter, Histogram
import time

tool_calls = Counter('agent_tool_calls_total', 'Total tool calls', ['tool_name', 'status'])
tool_duration = Histogram('agent_tool_duration_seconds', 'Tool execution time', ['tool_name'])

def monitored_tool(func):
    """Decorator to add monitoring to tools"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        tool_name = func.__name__
        
        try:
            result = func(*args, **kwargs)
            tool_calls.labels(tool_name=tool_name, status='success').inc()
            return result
            
        except Exception as e:
            tool_calls.labels(tool_name=tool_name, status='error').inc()
            raise
            
        finally:
            duration = time.time() - start
            tool_duration.labels(tool_name=tool_name).observe(duration)
    
    return wrapper

@tool
@monitored_tool
def monitored_operation(data: str) -> dict:
    """Operation with automatic metrics"""
    return process(data)
```

### Pattern 3: Distributed Tracing

```python
from opentelemetry import trace
from opentelemetry.trace import Status, StatusCode

tracer = trace.get_tracer(__name__)

@tool
def traced_operation(query: str) -> dict:
    """Operation with distributed tracing"""
    
    with tracer.start_as_current_span("tool.traced_operation") as span:
        span.set_attribute("query", query)
        
        try:
            # Call external service
            with tracer.start_as_current_span("external_api_call"):
                result = external_api.query(query)
            
            span.set_attribute("result_count", len(result))
            span.set_status(Status(StatusCode.OK))
            return result
            
        except Exception as e:
            span.set_status(Status(StatusCode.ERROR, str(e)))
            span.record_exception(e)
            raise
```

## Production Patterns

### Pattern 1: Dead Letter Queue

```python
import boto3

sqs = boto3.client('sqs')
DLQ_URL = "https://sqs.us-west-2.amazonaws.com/123/agent-dlq"

@tool
def process_with_dlq(message: dict) -> dict:
    """Process message with DLQ for failures"""
    
    try:
        result = process_message(message)
        return result
        
    except Exception as e:
        # Send to DLQ for manual review
        sqs.send_message(
            QueueUrl=DLQ_URL,
            MessageBody=json.dumps({
                "original_message": message,
                "error": str(e),
                "timestamp": datetime.now().isoformat()
            })
        )
        
        return {
            "error": "Processing failed, message sent to DLQ",
            "dlq_id": message.get("id")
        }
```

### Pattern 2: Health Checks

```python
@tool
def health_check() -> dict:
    """Check health of all dependencies"""
    health = {
        "status": "healthy",
        "checks": {}
    }
    
    # Check database
    try:
        db.execute("SELECT 1")
        health["checks"]["database"] = "healthy"
    except Exception as e:
        health["checks"]["database"] = f"unhealthy: {str(e)}"
        health["status"] = "degraded"
    
    # Check external API
    try:
        response = requests.get(API_HEALTH_ENDPOINT, timeout=5)
        health["checks"]["external_api"] = "healthy" if response.ok else "unhealthy"
    except Exception as e:
        health["checks"]["external_api"] = f"unhealthy: {str(e)}"
        health["status"] = "degraded"
    
    return health
```

### Pattern 3: Rate Limit Handling

```python
from ratelimit import limits, sleep_and_retry

@tool
@sleep_and_retry
@limits(calls=100, period=60)  # 100 calls per minute
def rate_limited_api_call(endpoint: str) -> dict:
    """API call with rate limiting"""
    return requests.get(endpoint).json()
```

## Best Practices

1. **Fail Fast** - Validate inputs early
2. **Fail Gracefully** - Return useful error messages
3. **Log Everything** - Errors, warnings, and successes
4. **Monitor Metrics** - Track error rates and latency
5. **Set Timeouts** - Prevent hanging operations
6. **Use Circuit Breakers** - Protect failing services
7. **Implement Retries** - Handle transient failures
8. **Provide Fallbacks** - Alternative data sources
9. **Structure Errors** - Make them actionable
10. **Test Failures** - Chaos engineering for agents

## Testing Error Scenarios

```python
import pytest

def test_tool_handles_network_error(mocker):
    """Test tool behavior when network fails"""
    mocker.patch('requests.get', side_effect=ConnectionError("Network unavailable"))
    
    result = call_external_api("https://api.example.com")
    
    assert "error" in result
    assert result["recoverable"] == True

def test_tool_handles_timeout(mocker):
    """Test tool behavior on timeout"""
    mocker.patch('requests.get', side_effect=TimeoutError())
    
    result = call_external_api("https://api.example.com")
    
    assert "error" in result
    assert "timeout" in result["error"].lower()

def test_circuit_breaker_opens():
    """Test circuit breaker opens after failures"""
    # Simulate 5 failures
    for _ in range(5):
        try:
            query_analytics_service("test")
        except:
            pass
    
    # Circuit should be open
    with pytest.raises(CircuitBreakerError):
        query_analytics_service("test")
```

## Conclusion

Production agent systems require comprehensive error handling. Implement these patterns to build resilient agents that gracefully handle failures and provide useful feedback when things go wrong.
