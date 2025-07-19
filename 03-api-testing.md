
# API Testing 🔌

## What is API Testing?

API (Application Programming Interface) testing validates data exchange between different software systems. It focuses on business logic, data validation, and integration points without testing the user interface.

## Key Concepts

### Types of API Testing
- **REST API Testing**: Testing RESTful web services
- **GraphQL Testing**: Testing GraphQL queries and mutations
- **SOAP API Testing**: Testing XML-based web services
- **Reverse API Testing**: Testing undocumented or third-party APIs
- **Microservices Testing**: Testing service-to-service communication

### Testing Levels
- **Unit Level**: Individual API endpoints
- **Integration Level**: API interactions with databases/services
- **System Level**: Complete API ecosystem testing
- **Contract Testing**: API contract validation between services

## Tools & Frameworks

### Manual API Testing Tools
- **Postman** - Most popular API testing platform
- **Insomnia** - Lightweight REST client
- **Thunder Client** - VS Code extension
- **Paw** - macOS API testing tool
- **Advanced REST Client** - Chrome extension

### Automation Frameworks

#### REST API Automation
```python
# Python with requests and pytest
import requests
import pytest
import json

class TestUserAPI:
    BASE_URL = "https://jsonplaceholder.typicode.com"
    
    def test_get_all_users(self):
        """Test retrieving all users"""
        response = requests.get(f"{self.BASE_URL}/users")
        
        assert response.status_code == 200
        assert len(response.json()) > 0
        assert isinstance(response.json(), list)
    
    def test_get_user_by_id(self):
        """Test retrieving specific user"""
        user_id = 1
        response = requests.get(f"{self.BASE_URL}/users/{user_id}")
        
        assert response.status_code == 200
        user_data = response.json()
        assert user_data["id"] == user_id
        assert "name" in user_data
        assert "email" in user_data
    
    def test_create_user(self):
        """Test creating new user"""
        new_user = {
            "name": "Test User",
            "username": "testuser",
            "email": "test@example.com"
        }
        
        response = requests.post(f"{self.BASE_URL}/users", json=new_user)
        
        assert response.status_code == 201
        created_user = response.json()
        assert created_user["name"] == new_user["name"]
        assert "id" in created_user
    
    def test_update_user(self):
        """Test updating existing user"""
        user_id = 1
        updated_data = {
            "name": "Updated Name",
            "email": "updated@example.com"
        }
        
        response = requests.put(f"{self.BASE_URL}/users/{user_id}", json=updated_data)
        
        assert response.status_code == 200
        updated_user = response.json()
        assert updated_user["name"] == updated_data["name"]
    
    def test_delete_user(self):
        """Test deleting user"""
        user_id = 1
        response = requests.delete(f"{self.BASE_URL}/users/{user_id}")
        
        assert response.status_code == 200
```

#### GraphQL Testing
```python
# GraphQL API testing with Python
import requests

class TestGraphQLAPI:
    GRAPHQL_URL = "https://api.github.com/graphql"
    
    def test_user_query(self):
        """Test GraphQL user query"""
        query = """
        query {
            user(login: "octocat") {
                name
                login
                repositories(first: 5) {
                    nodes {
                        name
                        description
                    }
                }
            }
        }
        """
        
        headers = {
            "Authorization": "Bearer YOUR_TOKEN",
            "Content-Type": "application/json"
        }
        
        response = requests.post(
            self.GRAPHQL_URL,
            json={"query": query},
            headers=headers
        )
        
        assert response.status_code == 200
        data = response.json()
        assert "data" in data
        assert data["data"]["user"]["login"] == "octocat"
    
    def test_mutation(self):
        """Test GraphQL mutation"""
        mutation = """
        mutation {
            createIssue(input: {
                repositoryId: "REPO_ID",
                title: "Test Issue",
                body: "This is a test issue"
            }) {
                issue {
                    id
                    title
                    body
                }
            }
        }
        """
        
        response = requests.post(
            self.GRAPHQL_URL,
            json={"query": mutation},
            headers={"Authorization": "Bearer YOUR_TOKEN"}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert "data" in data
```

#### Java REST Assured
```java
// REST Assured framework for Java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class APITests {
    
    @Test
    public void testGetUsers() {
        given()
            .baseUri("https://jsonplaceholder.typicode.com")
        .when()
            .get("/users")
        .then()
            .statusCode(200)
            .body("size()", greaterThan(0))
            .body("[0].name", notNullValue())
            .body("[0].email", containsString("@"));
    }
    
    @Test
    public void testCreateUser() {
        String requestBody = "{\n" +
            "\"name\": \"Test User\",\n" +
            "\"username\": \"testuser\",\n" +
            "\"email\": \"test@example.com\"\n" +
        "}";
        
        given()
            .baseUri("https://jsonplaceholder.typicode.com")
            .header("Content-Type", "application/json")
            .body(requestBody)
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .body("name", equalTo("Test User"))
            .body("id", notNullValue());
    }
}
```

## Real-World Examples

### E-commerce API Testing Suite
```python
class TestEcommerceAPI:
    
    def test_product_catalog_workflow(self):
        """Test complete product catalog workflow"""
        # 1. Get all categories
        categories_response = requests.get(f"{BASE_URL}/categories")
        assert categories_response.status_code == 200
        categories = categories_response.json()
        
        # 2. Get products in first category
        category_id = categories[0]["id"]
        products_response = requests.get(f"{BASE_URL}/categories/{category_id}/products")
        assert products_response.status_code == 200
        products = products_response.json()
        
        # 3. Get detailed product information
        product_id = products[0]["id"]
        product_response = requests.get(f"{BASE_URL}/products/{product_id}")
        assert product_response.status_code == 200
        
        product = product_response.json()
        assert product["id"] == product_id
        assert "price" in product
        assert "inventory" in product
    
    def test_shopping_cart_workflow(self):
        """Test shopping cart operations"""
        # 1. Create cart
        cart_response = requests.post(f"{BASE_URL}/carts")
        assert cart_response.status_code == 201
        cart_id = cart_response.json()["id"]
        
        # 2. Add items to cart
        item_data = {
            "product_id": 123,
            "quantity": 2
        }
        add_response = requests.post(f"{BASE_URL}/carts/{cart_id}/items", json=item_data)
        assert add_response.status_code == 200
        
        # 3. Update item quantity
        update_data = {"quantity": 3}
        update_response = requests.put(f"{BASE_URL}/carts/{cart_id}/items/123", json=update_data)
        assert update_response.status_code == 200
        
        # 4. Calculate total
        total_response = requests.get(f"{BASE_URL}/carts/{cart_id}/total")
        assert total_response.status_code == 200
        assert "total" in total_response.json()
```

### Authentication & Authorization Testing
```python
class TestAuthAPI:
    
    def test_login_workflow(self):
        """Test user authentication"""
        login_data = {
            "username": "testuser",
            "password": "password123"
        }
        
        response = requests.post(f"{BASE_URL}/auth/login", json=login_data)
        assert response.status_code == 200
        
        token_data = response.json()
        assert "access_token" in token_data
        assert "refresh_token" in token_data
        
        return token_data["access_token"]
    
    def test_protected_endpoint(self):
        """Test endpoint requiring authentication"""
        # Get token first
        token = self.test_login_workflow()
        
        headers = {
            "Authorization": f"Bearer {token}"
        }
        
        response = requests.get(f"{BASE_URL}/user/profile", headers=headers)
        assert response.status_code == 200
        
        profile = response.json()
        assert "user_id" in profile
        assert "email" in profile
    
    def test_unauthorized_access(self):
        """Test endpoint without proper authentication"""
        response = requests.get(f"{BASE_URL}/user/profile")
        assert response.status_code == 401
        
        error = response.json()
        assert "error" in error
        assert "Unauthorized" in error["error"]
```

## Reverse API Testing

### Discovering API Endpoints
```python
class TestReverseAPI:
    
    def test_endpoint_discovery(self):
        """Discover API endpoints through exploration"""
        # Common endpoint patterns to test
        common_endpoints = [
            "/api/v1/users",
            "/api/users",
            "/users",
            "/api/v1/health",
            "/health",
            "/status",
            "/version",
            "/docs",
            "/swagger"
        ]
        
        discovered_endpoints = []
        
        for endpoint in common_endpoints:
            try:
                response = requests.get(f"{BASE_URL}{endpoint}")
                if response.status_code != 404:
                    discovered_endpoints.append({
                        "endpoint": endpoint,
                        "status": response.status_code,
                        "content_type": response.headers.get("content-type", "")
                    })
            except:
                continue
        
        return discovered_endpoints
    
    def test_parameter_fuzzing(self):
        """Test different parameter combinations"""
        base_endpoint = "/api/users"
        
        # Test different parameter types
        test_params = [
            {"id": "1"},
            {"id": "abc"},
            {"id": ""},
            {"limit": "10"},
            {"limit": "-1"},
            {"limit": "abc"},
            {"sort": "name"},
            {"sort": "invalid"},
            {"filter": "active"},
            {"search": "test"}
        ]
        
        results = []
        for params in test_params:
            response = requests.get(f"{BASE_URL}{base_endpoint}", params=params)
            results.append({
                "params": params,
                "status": response.status_code,
                "response_size": len(response.content)
            })
        
        return results
```

## Resume Showcase Tips

### Strong API Testing Experience
```
✅ Excellent Examples:
• "Automated 150+ REST API endpoints using Python/requests, improving test coverage by 70%"
• "Implemented comprehensive GraphQL testing suite validating queries, mutations, and subscriptions"
• "Developed API contract testing framework preventing 25+ integration issues in microservices architecture"
• "Created data-driven API tests supporting multiple environments (dev, staging, production)"

✅ Technical Skills to Highlight:
• REST/GraphQL/SOAP API testing
• Tools: Postman, REST Assured, requests, newman
• Authentication methods: OAuth, JWT, API keys
• API documentation tools: Swagger, OpenAPI
• Performance testing: API load testing
• Security testing: API penetration testing
```

### Quantifiable Achievements
- Number of APIs automated
- Test coverage percentage
- Bug detection rate
- Performance improvements
- Integration test suite size

## Learning Path 📚

### Beginner (0-3 months)
1. **HTTP Fundamentals**
   ```
   - HTTP methods (GET, POST, PUT, DELETE)
   - Status codes (200, 400, 401, 404, 500)
   - Headers and authentication
   - Request/response body formats (JSON, XML)
   ```

2. **Manual Testing with Postman**
   - Create collections
   - Write tests with JavaScript
   - Environment variables
   - Data-driven testing

3. **Basic Automation**
   - Python requests library
   - Simple GET/POST tests
   - Response validation

### Intermediate (3-6 months)
1. **Advanced Automation**
   ```python
   # Configuration management
   class APIConfig:
       def __init__(self, env="dev"):
           self.configs = {
               "dev": {"base_url": "https://dev-api.com"},
               "staging": {"base_url": "https://staging-api.com"},
               "prod": {"base_url": "https://api.com"}
           }
           self.config = self.configs[env]
   ```

2. **Testing Frameworks**
   - pytest for Python
   - TestNG for Java
   - Jest for JavaScript
   - Framework patterns and best practices

3. **CI/CD Integration**
   - Newman for Postman collections
   - API testing in build pipelines
   - Test reporting and metrics

### Advanced (6+ months)
1. **Microservices Testing**
   - Contract testing with Pact
   - Service virtualization
   - Chaos engineering for APIs

2. **Security Testing**
   - OWASP API security testing
   - Authentication bypass testing
   - Input validation testing

3. **Performance Testing**
   - Load testing APIs
   - Stress testing endpoints
   - Performance monitoring

## Best Practices

### Test Design
```python
# Good: Clear test structure
class TestUserManagement:
    def setup_method(self):
        """Setup test data"""
        self.base_url = "https://api.example.com"
        self.headers = {"Content-Type": "application/json"}
    
    def test_create_user_with_valid_data(self):
        """Test user creation with valid input"""
        user_data = {
            "name": "John Doe",
            "email": "john@example.com",
            "role": "user"
        }
        
        response = requests.post(
            f"{self.base_url}/users",
            json=user_data,
            headers=self.headers
        )
        
        # Assertions
        assert response.status_code == 201
        created_user = response.json()
        assert created_user["name"] == user_data["name"]
        assert "id" in created_user
        assert created_user["id"] > 0
    
    def teardown_method(self):
        """Cleanup test data"""
        # Clean up any test data created
        pass
```

### Error Handling
```python
def make_api_request(method, url, **kwargs):
    """Wrapper for API requests with error handling"""
    try:
        response = requests.request(method, url, **kwargs)
        response.raise_for_status()
        return response
    except requests.exceptions.RequestException as e:
        print(f"API request failed: {e}")
        raise
    except ValueError as e:
        print(f"Invalid JSON response: {e}")
        raise
```

### Data Management
- Use test data factories
- Implement data cleanup
- Separate test data from test logic
- Use environment-specific configurations

### Contract Testing Implementation
```python
# Consumer contract test using Pact
import pact
from pact import Consumer, Provider

pact = Consumer('UserService').has_pact_with(Provider('PaymentService'))

def test_payment_processing():
    """Test payment service contract"""
    # Define expected interaction
    pact.given('user has valid payment method') \
        .upon_receiving('a payment request') \
        .with_request('POST', '/api/v1/payments') \
        .will_respond_with(200, body={
            'transaction_id': pact.like('txn_123'),
            'status': 'approved',
            'amount': pact.like(99.99)
        })
    
    with pact:
        # Make actual API call
        result = payment_service.process_payment({
            'amount': 99.99,
            'currency': 'USD',
            'payment_method': 'card_123'
        })
        
        assert result['status'] == 'approved'
        assert 'transaction_id' in result

# Provider verification test
def test_provider_verification():
    """Verify provider fulfills consumer contracts"""
    verifier = pact.Verifier()
    
    verifier.verify_pacts(
        pact_url='./pacts/userservice-paymentservice.json',
        provider_base_url='http://localhost:8080',
        provider_states_setup_url='http://localhost:8080/provider-states'
    )
```

### Advanced API Testing Patterns
```python
# API testing with data-driven approach
import pytest
import pandas as pd

class AdvancedAPITestSuite:
    
    @pytest.fixture
    def test_data(self):
        """Load test data from CSV/Excel"""
        return pd.read_csv('api_test_data.csv')
    
    @pytest.mark.parametrize('test_case', [
        {'endpoint': '/users', 'method': 'GET', 'expected_status': 200},
        {'endpoint': '/users/999', 'method': 'GET', 'expected_status': 404},
        {'endpoint': '/users', 'method': 'POST', 'expected_status': 201}
    ])
    def test_api_endpoints(self, test_case):
        """Data-driven API testing"""
        response = requests.request(
            method=test_case['method'],
            url=f"{BASE_URL}{test_case['endpoint']}",
            json=test_case.get('payload', {})
        )
        assert response.status_code == test_case['expected_status']
    
    def test_api_performance_benchmarks(self):
        """Test API performance benchmarks"""
        endpoints = ['/users', '/products', '/orders']
        performance_results = {}
        
        for endpoint in endpoints:
            start_time = time.time()
            response = requests.get(f"{BASE_URL}{endpoint}")
            end_time = time.time()
            
            performance_results[endpoint] = {
                'response_time': end_time - start_time,
                'status_code': response.status_code,
                'response_size': len(response.content)
            }
            
            # Assert performance requirements
            assert performance_results[endpoint]['response_time'] < 2.0
            assert response.status_code == 200
        
        return performance_results
    
    def test_api_rate_limiting(self):
        """Test API rate limiting implementation"""
        # Make rapid requests to test rate limiting
        responses = []
        for i in range(100):
            response = requests.get(f"{BASE_URL}/api/users")
            responses.append({
                'status_code': response.status_code,
                'headers': dict(response.headers)
            })
            
            if response.status_code == 429:  # Rate limited
                assert 'Retry-After' in response.headers
                break
        
        # Verify rate limiting is working
        rate_limited = any(r['status_code'] == 429 for r in responses)
        assert rate_limited, "Rate limiting should be enforced"
```

### API Security Testing
```python
class APISecurityTestSuite:
    
    def test_sql_injection_prevention(self):
        """Test SQL injection vulnerability prevention"""
        sql_payloads = [
            "'; DROP TABLE users; --",
            "' OR '1'='1",
            "' UNION SELECT * FROM admin_users --"
        ]
        
        for payload in sql_payloads:
            response = requests.get(f"{BASE_URL}/users", params={
                'search': payload
            })
            
            # Should handle malicious input gracefully
            assert response.status_code in [200, 400, 422]
            assert 'error' not in response.text.lower() or \
                   'sql' not in response.text.lower()
    
    def test_authentication_bypass(self):
        """Test authentication bypass attempts"""
        bypass_attempts = [
            {'header': 'Authorization', 'value': 'Bearer invalid_token'},
            {'header': 'X-API-Key', 'value': 'admin'},
            {'header': 'Authorization', 'value': 'Basic YWRtaW46YWRtaW4='}
        ]
        
        for attempt in bypass_attempts:
            response = requests.get(
                f"{BASE_URL}/admin/users",
                headers={attempt['header']: attempt['value']}
            )
            
            # Should reject unauthorized access
            assert response.status_code in [401, 403]
    
    def test_data_exposure_prevention(self):
        """Test sensitive data exposure prevention"""
        response = requests.get(f"{BASE_URL}/users/1")
        
        if response.status_code == 200:
            user_data = response.json()
            
            # Sensitive fields should not be exposed
            sensitive_fields = ['password', 'ssn', 'credit_card']
            for field in sensitive_fields:
                assert field not in user_data
```

### API Monitoring and Observability
```python
class APIMonitoringTest:
    
    def test_api_health_monitoring(self):
        """Test API health monitoring endpoints"""
        health_response = requests.get(f"{BASE_URL}/health")
        assert health_response.status_code == 200
        
        health_data = health_response.json()
        required_fields = ['status', 'timestamp', 'version']
        
        for field in required_fields:
            assert field in health_data
        
        assert health_data['status'] in ['healthy', 'degraded', 'unhealthy']
    
    def test_api_metrics_collection(self):
        """Test API metrics collection"""
        metrics_response = requests.get(f"{BASE_URL}/metrics")
        
        if metrics_response.status_code == 200:
            metrics_text = metrics_response.text
            
            # Check for common metrics
            expected_metrics = [
                'http_requests_total',
                'http_request_duration_seconds',
                'http_response_size_bytes'
            ]
            
            for metric in expected_metrics:
                assert metric in metrics_text
    
    def test_distributed_tracing(self):
        """Test distributed tracing implementation"""
        trace_id = str(uuid.uuid4())
        
        response = requests.get(
            f"{BASE_URL}/users/1",
            headers={'X-Trace-ID': trace_id}
        )
        
        # Verify trace ID is propagated
        assert 'X-Trace-ID' in response.headers
        assert response.headers['X-Trace-ID'] == trace_id
```

---

**Next Step**: Ready to test performance? Continue to [Load & Performance Testing](./04-load-performance-testing.md) to learn JMeter, Locust, and stress testing techniques.
