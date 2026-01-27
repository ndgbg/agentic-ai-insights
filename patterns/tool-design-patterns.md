# Tool Design Patterns for AI Agents

Effective agent systems depend on well-designed tools. This guide covers patterns for creating tools that agents can use reliably.

## Core Principles

### 1. Single Responsibility
Each tool should do one thing well.

```python
# ❌ Bad: Tool does too much
@tool
def manage_customer(action: str, customer_id: str, data: dict):
    """Create, update, delete, or query customers"""
    if action == "create":
        # ...
    elif action == "update":
        # ...

# ✅ Good: Separate tools
@tool
def get_customer(customer_id: str) -> dict:
    """Get customer details by ID"""
    return db.query("SELECT * FROM customers WHERE id = ?", customer_id)

@tool
def update_customer(customer_id: str, updates: dict) -> bool:
    """Update customer information"""
    return db.update("customers", customer_id, updates)
```

### 2. Clear Descriptions
Agents rely on descriptions to choose tools.

```python
# ❌ Bad: Vague description
@tool
def process_data(data: str) -> str:
    """Process data"""
    pass

# ✅ Good: Specific description
@tool
def calculate_revenue_by_region(sales_data: str) -> dict:
    """
    Calculate total revenue grouped by geographic region.
    
    Args:
        sales_data: JSON string with sales records containing 'region' and 'amount'
    
    Returns:
        Dictionary mapping region names to total revenue
        Example: {"North": 150000, "South": 120000}
    """
    pass
```

### 3. Type Safety
Use type hints for validation.

```python
from typing import Literal

@tool
def set_priority(
    ticket_id: str,
    priority: Literal["low", "medium", "high", "critical"]
) -> bool:
    """Set ticket priority level"""
    # Type system prevents invalid priorities
    return update_ticket(ticket_id, {"priority": priority})
```

## Common Patterns

### Pattern 1: Query-Transform-Return

For data retrieval and processing:

```python
@tool
def get_top_customers(limit: int = 10) -> list:
    """Get top customers by total purchase amount"""
    # Query
    results = db.query("""
        SELECT customer_id, name, SUM(amount) as total
        FROM orders
        GROUP BY customer_id
        ORDER BY total DESC
        LIMIT ?
    """, limit)
    
    # Transform
    customers = [
        {
            "id": row[0],
            "name": row[1],
            "total_spent": float(row[2])
        }
        for row in results
    ]
    
    # Return
    return customers
```

### Pattern 2: Validate-Execute-Confirm

For actions with side effects:

```python
@tool
def send_email(recipient: str, subject: str, body: str) -> dict:
    """Send email to customer"""
    # Validate
    if not is_valid_email(recipient):
        return {"success": False, "error": "Invalid email address"}
    
    if len(body) > 10000:
        return {"success": False, "error": "Email body too long"}
    
    # Execute
    try:
        email_service.send(recipient, subject, body)
    except Exception as e:
        return {"success": False, "error": str(e)}
    
    # Confirm
    return {
        "success": True,
        "message": f"Email sent to {recipient}",
        "timestamp": datetime.now().isoformat()
    }
```

### Pattern 3: Paginated Results

For large datasets:

```python
@tool
def search_orders(
    query: str,
    page: int = 1,
    page_size: int = 20
) -> dict:
    """Search orders with pagination"""
    offset = (page - 1) * page_size
    
    results = db.query("""
        SELECT * FROM orders
        WHERE description LIKE ?
        LIMIT ? OFFSET ?
    """, f"%{query}%", page_size, offset)
    
    total = db.query("SELECT COUNT(*) FROM orders WHERE description LIKE ?", f"%{query}%")[0][0]
    
    return {
        "results": results,
        "page": page,
        "page_size": page_size,
        "total_results": total,
        "total_pages": (total + page_size - 1) // page_size,
        "has_more": offset + len(results) < total
    }
```

### Pattern 4: Idempotent Operations

For safe retries:

```python
@tool
def create_or_update_user(email: str, name: str) -> dict:
    """Create user if not exists, otherwise update"""
    # Check if exists
    existing = db.query("SELECT id FROM users WHERE email = ?", email)
    
    if existing:
        # Update
        user_id = existing[0][0]
        db.update("users", user_id, {"name": name})
        return {"action": "updated", "user_id": user_id}
    else:
        # Create
        user_id = db.insert("users", {"email": email, "name": name})
        return {"action": "created", "user_id": user_id}
```

### Pattern 5: Composite Tools

Combine simple tools for complex operations:

```python
@tool
def onboard_customer(email: str, name: str, plan: str) -> dict:
    """Complete customer onboarding process"""
    results = {}
    
    # Step 1: Create account
    user = create_or_update_user(email, name)
    results["user_created"] = user
    
    # Step 2: Set up subscription
    subscription = create_subscription(user["user_id"], plan)
    results["subscription"] = subscription
    
    # Step 3: Send welcome email
    email_result = send_email(
        email,
        "Welcome!",
        f"Hi {name}, welcome to our platform!"
    )
    results["email_sent"] = email_result
    
    return results
```

## Error Handling Patterns

### Pattern 1: Graceful Degradation

```python
@tool
def get_weather_with_forecast(location: str) -> dict:
    """Get current weather and 5-day forecast"""
    result = {}
    
    # Try to get current weather
    try:
        result["current"] = weather_api.get_current(location)
    except Exception as e:
        result["current"] = {"error": str(e)}
    
    # Try to get forecast (independent of current)
    try:
        result["forecast"] = weather_api.get_forecast(location)
    except Exception as e:
        result["forecast"] = {"error": str(e)}
    
    return result
```

### Pattern 2: Retry with Backoff

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@tool
@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def call_external_api(endpoint: str) -> dict:
    """Call external API with automatic retries"""
    response = requests.get(endpoint, timeout=10)
    response.raise_for_status()
    return response.json()
```

### Pattern 3: Circuit Breaker

```python
from circuitbreaker import circuit

@tool
@circuit(failure_threshold=5, recovery_timeout=60)
def query_slow_service(query: str) -> dict:
    """Query service with circuit breaker protection"""
    # If service fails 5 times, circuit opens for 60 seconds
    return slow_service.query(query)
```

## Performance Patterns

### Pattern 1: Caching

```python
from functools import lru_cache

@tool
@lru_cache(maxsize=100)
def get_product_details(product_id: str) -> dict:
    """Get product details (cached)"""
    # Expensive database query
    return db.query("SELECT * FROM products WHERE id = ?", product_id)
```

### Pattern 2: Batch Operations

```python
@tool
def get_multiple_customers(customer_ids: list[str]) -> list[dict]:
    """Get multiple customers in one query"""
    # Single query instead of N queries
    placeholders = ",".join(["?" for _ in customer_ids])
    results = db.query(
        f"SELECT * FROM customers WHERE id IN ({placeholders})",
        *customer_ids
    )
    return results
```

### Pattern 3: Async Operations

```python
import asyncio

@tool
async def fetch_all_data(sources: list[str]) -> dict:
    """Fetch data from multiple sources concurrently"""
    async def fetch_one(source):
        return await api.get(source)
    
    results = await asyncio.gather(*[fetch_one(s) for s in sources])
    return dict(zip(sources, results))
```

## Security Patterns

### Pattern 1: Input Validation

```python
import re

@tool
def search_users(query: str) -> list:
    """Search users by name or email"""
    # Prevent SQL injection
    if not re.match(r'^[a-zA-Z0-9@._-]+$', query):
        return {"error": "Invalid search query"}
    
    # Use parameterized query
    return db.query("SELECT * FROM users WHERE name LIKE ? OR email LIKE ?", 
                    f"%{query}%", f"%{query}%")
```

### Pattern 2: Permission Checks

```python
@tool
def delete_customer(customer_id: str, requesting_user: str) -> dict:
    """Delete customer (admin only)"""
    # Check permissions
    if not has_permission(requesting_user, "delete_customer"):
        return {"error": "Insufficient permissions"}
    
    # Perform action
    db.delete("customers", customer_id)
    
    # Audit log
    audit_log.record("customer_deleted", {
        "customer_id": customer_id,
        "deleted_by": requesting_user
    })
    
    return {"success": True}
```

### Pattern 3: Rate Limiting

```python
from functools import wraps
import time

def rate_limit(max_calls: int, period: int):
    """Rate limit decorator"""
    calls = []
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            # Remove old calls
            calls[:] = [c for c in calls if c > now - period]
            
            if len(calls) >= max_calls:
                return {"error": f"Rate limit exceeded: {max_calls} calls per {period}s"}
            
            calls.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator

@tool
@rate_limit(max_calls=10, period=60)
def expensive_operation(data: str) -> dict:
    """Expensive operation with rate limiting"""
    return process(data)
```

## Testing Patterns

### Pattern 1: Mock External Dependencies

```python
@tool
def get_stock_price(symbol: str) -> float:
    """Get current stock price"""
    return stock_api.get_price(symbol)

# Test
def test_get_stock_price(mocker):
    mocker.patch('stock_api.get_price', return_value=150.25)
    assert get_stock_price("AAPL") == 150.25
```

### Pattern 2: Validate Tool Outputs

```python
from pydantic import BaseModel

class CustomerResponse(BaseModel):
    id: str
    name: str
    email: str
    status: Literal["active", "inactive"]

@tool
def get_customer(customer_id: str) -> dict:
    """Get customer details"""
    result = db.query("SELECT * FROM customers WHERE id = ?", customer_id)
    
    # Validate output schema
    return CustomerResponse(**result).dict()
```

## Best Practices

1. **Keep tools focused** - One tool, one purpose
2. **Document thoroughly** - Agents rely on descriptions
3. **Handle errors gracefully** - Return structured error messages
4. **Validate inputs** - Prevent invalid operations
5. **Make tools idempotent** - Safe to retry
6. **Log operations** - Track tool usage for debugging
7. **Test extensively** - Unit test each tool
8. **Monitor performance** - Track latency and errors
9. **Version tools** - Handle breaking changes carefully
10. **Provide examples** - Show expected inputs/outputs in docstrings

## Anti-Patterns to Avoid

❌ **God Tools** - Tools that do everything
❌ **Unclear Names** - `process_data()`, `handle_request()`
❌ **Silent Failures** - Returning None instead of error messages
❌ **Side Effects Without Confirmation** - Deleting data without validation
❌ **Unstructured Returns** - Returning raw strings instead of structured data
❌ **Missing Error Handling** - Letting exceptions bubble up
❌ **No Input Validation** - Trusting all inputs
❌ **Blocking Operations** - Long-running synchronous calls
❌ **Hardcoded Values** - Configuration mixed with logic
❌ **No Logging** - Can't debug tool usage

## Conclusion

Well-designed tools are the foundation of reliable agent systems. Follow these patterns to create tools that agents can use effectively and safely in production.
