---
name: api-client-tester-waruna834
description: Complete API testing tool for REST, GraphQL, and WebSocket with collections support - lightweight Postman alternative
triggers:
  - test REST API endpoints
  - send GraphQL queries
  - test WebSocket connections
  - manage API request collections
  - authenticate API requests
  - export API test collections
  - inspect HTTP responses
  - build API testing suite
---

# API Client & Tester

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

API Client & Tester is a complete API testing tool for REST, GraphQL, and WebSocket protocols. It provides a lightweight alternative to Postman with support for multiple HTTP methods, authentication strategies, request collections, and import/export functionality.

## Installation

### From Source

```bash
git clone https://github.com/waruna834/API_Client___Tester.git
cd API_Client___Tester
pip install -r requirements.txt
python main.py
```

### Requirements

- Python 3.7+
- Windows 10/11 (64-bit)
- Dependencies managed via `requirements.txt`

## Core Components

### HTTP Client

The tool supports standard HTTP methods with authentication:

```python
import requests
import json

class APIClient:
    def __init__(self, base_url=""):
        self.base_url = base_url
        self.headers = {}
        self.auth = None
    
    def set_auth(self, auth_type, credentials):
        """Set authentication for requests"""
        if auth_type == "basic":
            from requests.auth import HTTPBasicAuth
            self.auth = HTTPBasicAuth(credentials['username'], credentials['password'])
        elif auth_type == "bearer":
            self.headers['Authorization'] = f"Bearer {credentials['token']}"
        elif auth_type == "api_key":
            self.headers[credentials['key_name']] = credentials['key_value']
    
    def request(self, method, endpoint, **kwargs):
        """Send HTTP request"""
        url = f"{self.base_url}{endpoint}"
        kwargs.setdefault('headers', {}).update(self.headers)
        if self.auth:
            kwargs['auth'] = self.auth
        
        response = requests.request(method, url, **kwargs)
        return {
            'status_code': response.status_code,
            'headers': dict(response.headers),
            'body': response.text,
            'json': response.json() if response.headers.get('content-type', '').startswith('application/json') else None
        }

# Usage
client = APIClient(base_url="https://api.example.com")
client.set_auth("bearer", {"token": os.getenv("API_TOKEN")})

response = client.request("GET", "/users", params={"page": 1})
print(f"Status: {response['status_code']}")
print(f"Data: {response['json']}")
```

### GraphQL Support

```python
class GraphQLClient:
    def __init__(self, endpoint, headers=None):
        self.endpoint = endpoint
        self.headers = headers or {}
    
    def query(self, query, variables=None):
        """Execute GraphQL query"""
        payload = {
            'query': query,
            'variables': variables or {}
        }
        
        response = requests.post(
            self.endpoint,
            json=payload,
            headers=self.headers
        )
        
        result = response.json()
        if 'errors' in result:
            raise Exception(f"GraphQL errors: {result['errors']}")
        
        return result['data']
    
    def mutation(self, mutation, variables=None):
        """Execute GraphQL mutation"""
        return self.query(mutation, variables)

# Usage
gql_client = GraphQLClient(
    endpoint="https://api.example.com/graphql",
    headers={"Authorization": f"Bearer {os.getenv('GRAPHQL_TOKEN')}"}
)

query = """
query GetUser($id: ID!) {
    user(id: $id) {
        id
        name
        email
    }
}
"""

result = gql_client.query(query, variables={"id": "123"})
print(result['user'])
```

### WebSocket Testing

```python
import websocket
import json
import threading

class WebSocketClient:
    def __init__(self, url):
        self.url = url
        self.ws = None
        self.messages = []
    
    def on_message(self, ws, message):
        """Handle incoming message"""
        self.messages.append({
            'timestamp': datetime.now().isoformat(),
            'data': message
        })
        print(f"Received: {message}")
    
    def on_error(self, ws, error):
        """Handle error"""
        print(f"Error: {error}")
    
    def on_close(self, ws, close_status_code, close_msg):
        """Handle connection close"""
        print(f"Connection closed: {close_status_code} - {close_msg}")
    
    def on_open(self, ws):
        """Handle connection open"""
        print("WebSocket connection opened")
    
    def connect(self):
        """Establish WebSocket connection"""
        self.ws = websocket.WebSocketApp(
            self.url,
            on_open=self.on_open,
            on_message=self.on_message,
            on_error=self.on_error,
            on_close=self.on_close
        )
        
        ws_thread = threading.Thread(target=self.ws.run_forever)
        ws_thread.daemon = True
        ws_thread.start()
    
    def send(self, data):
        """Send message"""
        if self.ws:
            self.ws.send(json.dumps(data) if isinstance(data, dict) else data)
    
    def close(self):
        """Close connection"""
        if self.ws:
            self.ws.close()

# Usage
ws_client = WebSocketClient("wss://echo.websocket.org")
ws_client.connect()

# Send message
ws_client.send({"type": "ping", "timestamp": time.time()})

# Wait for responses
time.sleep(2)

# Close connection
ws_client.close()
```

### Collections Management

```python
import json
import os

class CollectionManager:
    def __init__(self, collections_dir="./collections"):
        self.collections_dir = collections_dir
        os.makedirs(collections_dir, exist_ok=True)
    
    def create_collection(self, name, requests=None):
        """Create a new collection"""
        collection = {
            'name': name,
            'created_at': datetime.now().isoformat(),
            'requests': requests or []
        }
        return collection
    
    def add_request(self, collection, request_data):
        """Add request to collection"""
        request = {
            'id': str(uuid.uuid4()),
            'name': request_data.get('name', 'Untitled'),
            'method': request_data['method'],
            'url': request_data['url'],
            'headers': request_data.get('headers', {}),
            'body': request_data.get('body', ''),
            'auth': request_data.get('auth', None),
            'params': request_data.get('params', {})
        }
        collection['requests'].append(request)
        return collection
    
    def save_collection(self, collection):
        """Save collection to file"""
        filename = f"{collection['name'].replace(' ', '_')}.json"
        filepath = os.path.join(self.collections_dir, filename)
        
        with open(filepath, 'w') as f:
            json.dump(collection, f, indent=2)
        
        return filepath
    
    def load_collection(self, name):
        """Load collection from file"""
        filename = f"{name.replace(' ', '_')}.json"
        filepath = os.path.join(self.collections_dir, filename)
        
        with open(filepath, 'r') as f:
            return json.load(f)
    
    def export_collection(self, collection, output_path):
        """Export collection to JSON"""
        with open(output_path, 'w') as f:
            json.dump(collection, f, indent=2)
    
    def import_collection(self, input_path):
        """Import collection from JSON"""
        with open(input_path, 'r') as f:
            collection = json.load(f)
        
        self.save_collection(collection)
        return collection

# Usage
manager = CollectionManager()

# Create collection
collection = manager.create_collection("User API Tests")

# Add requests
collection = manager.add_request(collection, {
    'name': 'Get All Users',
    'method': 'GET',
    'url': 'https://api.example.com/users',
    'auth': {'type': 'bearer', 'token': '${API_TOKEN}'}
})

collection = manager.add_request(collection, {
    'name': 'Create User',
    'method': 'POST',
    'url': 'https://api.example.com/users',
    'headers': {'Content-Type': 'application/json'},
    'body': json.dumps({'name': 'John Doe', 'email': 'john@example.com'}),
    'auth': {'type': 'bearer', 'token': '${API_TOKEN}'}
})

# Save collection
manager.save_collection(collection)

# Export for sharing
manager.export_collection(collection, "./exports/user_api_tests.json")
```

## Authentication Patterns

### Basic Authentication

```python
client = APIClient(base_url="https://api.example.com")
client.set_auth("basic", {
    'username': os.getenv('API_USERNAME'),
    'password': os.getenv('API_PASSWORD')
})

response = client.request("GET", "/protected")
```

### Bearer Token

```python
client = APIClient(base_url="https://api.example.com")
client.set_auth("bearer", {
    'token': os.getenv('BEARER_TOKEN')
})

response = client.request("GET", "/me")
```

### API Key (Header)

```python
client = APIClient(base_url="https://api.example.com")
client.set_auth("api_key", {
    'key_name': 'X-API-Key',
    'key_value': os.getenv('API_KEY')
})

response = client.request("GET", "/data")
```

### OAuth2 Flow

```python
import requests
from urllib.parse import urlencode

class OAuth2Client:
    def __init__(self, client_id, client_secret, auth_url, token_url):
        self.client_id = client_id
        self.client_secret = client_secret
        self.auth_url = auth_url
        self.token_url = token_url
        self.access_token = None
    
    def get_authorization_url(self, redirect_uri, scope=None):
        """Generate authorization URL"""
        params = {
            'client_id': self.client_id,
            'redirect_uri': redirect_uri,
            'response_type': 'code'
        }
        if scope:
            params['scope'] = scope
        
        return f"{self.auth_url}?{urlencode(params)}"
    
    def exchange_code_for_token(self, code, redirect_uri):
        """Exchange authorization code for access token"""
        data = {
            'grant_type': 'authorization_code',
            'code': code,
            'redirect_uri': redirect_uri,
            'client_id': self.client_id,
            'client_secret': self.client_secret
        }
        
        response = requests.post(self.token_url, data=data)
        token_data = response.json()
        self.access_token = token_data['access_token']
        return token_data

# Usage
oauth = OAuth2Client(
    client_id=os.getenv('OAUTH_CLIENT_ID'),
    client_secret=os.getenv('OAUTH_CLIENT_SECRET'),
    auth_url="https://oauth.example.com/authorize",
    token_url="https://oauth.example.com/token"
)

auth_url = oauth.get_authorization_url(
    redirect_uri="http://localhost:8080/callback",
    scope="read write"
)
print(f"Visit: {auth_url}")
```

## Request Execution Patterns

### Batch Request Execution

```python
class BatchExecutor:
    def __init__(self, client):
        self.client = client
    
    def execute_collection(self, collection, environment=None):
        """Execute all requests in a collection"""
        results = []
        env_vars = environment or {}
        
        for request in collection['requests']:
            # Replace environment variables
            url = self._replace_vars(request['url'], env_vars)
            headers = {k: self._replace_vars(v, env_vars) 
                      for k, v in request.get('headers', {}).items()}
            
            try:
                response = self.client.request(
                    request['method'],
                    url,
                    headers=headers,
                    json=json.loads(request.get('body', '{}')) if request.get('body') else None
                )
                
                results.append({
                    'request': request['name'],
                    'status': 'success',
                    'response': response
                })
            except Exception as e:
                results.append({
                    'request': request['name'],
                    'status': 'error',
                    'error': str(e)
                })
        
        return results
    
    def _replace_vars(self, text, env_vars):
        """Replace ${VAR} with environment values"""
        for key, value in env_vars.items():
            text = text.replace(f"${{{key}}}", str(value))
        return text

# Usage
executor = BatchExecutor(client)

environment = {
    'BASE_URL': 'https://api.example.com',
    'API_TOKEN': os.getenv('API_TOKEN'),
    'USER_ID': '12345'
}

results = executor.execute_collection(collection, environment)

for result in results:
    print(f"{result['request']}: {result['status']}")
    if result['status'] == 'success':
        print(f"  Status Code: {result['response']['status_code']}")
```

### Response Validation

```python
class ResponseValidator:
    @staticmethod
    def validate_status(response, expected_status):
        """Validate status code"""
        return response['status_code'] == expected_status
    
    @staticmethod
    def validate_json_schema(response, schema):
        """Validate JSON response against schema"""
        from jsonschema import validate, ValidationError
        
        try:
            validate(instance=response['json'], schema=schema)
            return True
        except ValidationError as e:
            return False, str(e)
    
    @staticmethod
    def validate_response_time(response, max_time_ms):
        """Validate response time"""
        return response.get('elapsed_ms', 0) <= max_time_ms

# Usage
validator = ResponseValidator()

response = client.request("GET", "/users/123")

# Validate status
assert validator.validate_status(response, 200), "Expected 200 status"

# Validate schema
schema = {
    "type": "object",
    "properties": {
        "id": {"type": "string"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"}
    },
    "required": ["id", "name", "email"]
}

is_valid = validator.validate_json_schema(response, schema)
assert is_valid, "Response doesn't match schema"
```

## Configuration

### Environment Variables

```python
# .env file
API_BASE_URL=https://api.example.com
API_TOKEN=your_token_here
API_USERNAME=your_username
API_PASSWORD=your_password
OAUTH_CLIENT_ID=your_client_id
OAUTH_CLIENT_SECRET=your_client_secret
GRAPHQL_ENDPOINT=https://api.example.com/graphql
WEBSOCKET_URL=wss://api.example.com/ws
```

### Config File

```python
import json

class Config:
    def __init__(self, config_path="config.json"):
        self.config_path = config_path
        self.config = self.load()
    
    def load(self):
        """Load configuration"""
        try:
            with open(self.config_path, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return self.get_defaults()
    
    def get_defaults(self):
        """Default configuration"""
        return {
            'timeout': 30,
            'retry_count': 3,
            'verify_ssl': True,
            'follow_redirects': True,
            'default_headers': {
                'User-Agent': 'API-Client-Tester/1.0'
            }
        }
    
    def save(self):
        """Save configuration"""
        with open(self.config_path, 'w') as f:
            json.dump(self.config, f, indent=2)

# Usage
config = Config()
print(f"Timeout: {config.config['timeout']}")
```

## Troubleshooting

### SSL Certificate Errors

```python
# Disable SSL verification (not recommended for production)
response = requests.get(url, verify=False)

# Or provide custom CA bundle
response = requests.get(url, verify='/path/to/ca-bundle.crt')
```

### Connection Timeouts

```python
client = APIClient(base_url="https://api.example.com")

try:
    response = client.request(
        "GET", 
        "/slow-endpoint",
        timeout=10  # 10 seconds timeout
    )
except requests.exceptions.Timeout:
    print("Request timed out")
```

### Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls=10, period=60):
    """Rate limiting decorator"""
    timestamps = []
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            timestamps[:] = [t for t in timestamps if now - t < period]
            
            if len(timestamps) >= calls:
                sleep_time = period - (now - timestamps[0])
                time.sleep(sleep_time)
            
            timestamps.append(time.time())
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls=10, period=60)
def make_api_call(endpoint):
    return client.request("GET", endpoint)
```

### WebSocket Reconnection

```python
class ResilientWebSocketClient(WebSocketClient):
    def __init__(self, url, max_retries=5):
        super().__init__(url)
        self.max_retries = max_retries
        self.retry_count = 0
    
    def on_error(self, ws, error):
        """Handle error with reconnection"""
        print(f"Error: {error}")
        if self.retry_count < self.max_retries:
            self.retry_count += 1
            time.sleep(2 ** self.retry_count)  # Exponential backoff
            self.connect()
        else:
            print("Max retries reached")
    
    def on_open(self, ws):
        """Reset retry count on successful connection"""
        self.retry_count = 0
        print("WebSocket connection established")
```

## Best Practices

1. **Use environment variables** for sensitive data (tokens, passwords)
2. **Implement retry logic** for transient network errors
3. **Validate responses** before processing data
4. **Save collections** for regression testing
5. **Use timeouts** to prevent hanging requests
6. **Handle rate limits** appropriately
7. **Log requests and responses** for debugging
8. **Version your collections** for tracking changes
