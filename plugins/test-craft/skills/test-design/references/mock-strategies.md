# Mock Strategies for Unit vs. Integration Tests

This reference provides detailed mock strategy patterns for the two test categories in test-craft.

## Core Principle

The fundamental difference between unit and integration tests in test-craft is the **mock boundary**:

- **Unit tests**: Mock EVERYTHING external to the function/method under test
- **Integration tests**: Only mock other modules within the SAME project; keep real middleware, databases, APIs, etc.

## Unit Test Mock Strategy

### What to Mock

| Dependency Type | Mock? | Example |
|----------------|-------|---------|
| Database access | Yes | Mock repository/DAO layer |
| HTTP/API calls | Yes | Mock HTTP client or service interface |
| Message queue publish/consume | Yes | Mock MQ client |
| Cache (Redis, Memcached) | Yes | Mock cache interface |
| File system I/O | Yes | Mock file reader/writer |
| Time/clock | Yes | Use fixed time provider |
| Random/UUID generation | Yes | Use deterministic generator |
| Other in-project modules | Yes | Mock their interfaces |
| Logging | Usually no | Unless testing log output specifically |
| Pure utility functions | No | Math, string formatting, etc. |

### Language-Specific Patterns

#### Go

```go
// Interface-based mocking
type OrderRepository interface {
    FindByID(ctx context.Context, id string) (*Order, error)
}

// Test with mock
type mockOrderRepo struct {
    orders map[string]*Order
}

func (m *mockOrderRepo) FindByID(ctx context.Context, id string) (*Order, error) {
    o, ok := m.orders[id]
    if !ok {
        return nil, ErrNotFound
    }
    return o, nil
}

// Or use mockgen/testify mock
```

#### Python

```python
# unittest.mock
from unittest.mock import MagicMock, patch

@patch('mymodule.db_client')
def test_create_order(mock_db):
    mock_db.insert.return_value = "order-123"
    result = create_order({"item": "widget"})
    assert result.id == "order-123"

# pytest fixtures
@pytest.fixture
def mock_redis():
    with patch('mymodule.redis_client') as mock:
        mock.get.return_value = None
        yield mock
```

#### TypeScript/JavaScript

```typescript
// Jest mocking
jest.mock('./database', () => ({
  findOrder: jest.fn().mockResolvedValue({ id: '123', status: 'active' }),
}));

// Or dependency injection
const mockRepo = {
  findOrder: jest.fn().mockResolvedValue({ id: '123' }),
};
const service = new OrderService(mockRepo);
```

#### Java

```java
// Mockito
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock private OrderRepository repository;
    @Mock private PaymentGateway gateway;
    @InjectMocks private OrderService service;

    @Test
    void shouldCreateOrder() {
        when(repository.save(any())).thenReturn(new Order("123"));
        Order result = service.create(new CreateOrderRequest());
        assertEquals("123", result.getId());
    }
}
```

## Integration Test Mock Strategy

### What to Mock vs. Keep Real

| Dependency Type | Mock? | Reason |
|----------------|-------|--------|
| Other in-project modules/packages | **Yes** | Isolate the module under test |
| Database (PostgreSQL, MySQL, etc.) | **No** | Test real SQL, schema, constraints |
| Redis/Cache | **No** | Test real caching behavior |
| Message Queue (Kafka, RabbitMQ, etc.) | **No** | Test real pub/sub flow |
| External APIs outside project | **No** | Test real integration (or use sandbox) |
| File system | **No** | Test real file operations |
| HTTP server (for API tests) | **No** | Start real test server |

### Infrastructure Setup Patterns

#### Database

- Use test containers (Testcontainers) or Docker Compose for ephemeral databases
- Create migration/seed scripts for test data
- Clean up between tests (truncate tables or use transactions)

#### Message Queue

- Use test containers for Kafka/RabbitMQ/etc.
- Create test topics/queues with unique names per test run
- Drain queues between tests

#### Cache

- Use test containers for Redis
- Flush between tests
- Use key prefixes per test run

### Separating Integration Tests

#### Go

```go
//go:build integration

package order_test

func TestOrderFlowIntegration(t *testing.T) {
    // Run with: go test -tags=integration ./...
}
```

#### Python

```python
@pytest.mark.integration
class TestOrderFlowIntegration:
    # Run with: pytest -m integration
    pass
```

#### TypeScript/JavaScript

```typescript
// In __tests__/integration/ directory
// Run with: jest --testPathPattern=integration
describe('Order Flow Integration', () => { ... });
```

#### Java

```java
@Tag("integration")
class OrderFlowIntegrationTest {
    // Run with: mvn test -Dgroups=integration
}
```

## Business Flow Identification

For integration tests, identify business flows that can be exercised within the module's boundary:

1. **Entry points**: Find the public API methods that start a business process
2. **Dependencies**: Map which external systems are touched during the flow
3. **Exit points**: Identify observable outputs (database state, MQ messages, API responses)
4. **Completeness**: A flow is "complete" when it starts from an entry point and produces a verifiable outcome

### Example

For an `order` module:
- **Flow**: Create Order → Validate Inventory → Calculate Price → Persist → Publish Event
- **Mock**: `inventory` module (in-project) — stub it to return available
- **Real**: Database (persist order), Message Queue (publish event)
- **Verify**: Order exists in database with correct fields; event published to queue with correct payload
