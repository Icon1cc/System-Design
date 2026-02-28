# Creational Design Patterns

## What are Creational Patterns?

Creational patterns deal with **object creation mechanisms**. They abstract the instantiation process, making systems independent of how objects are created, composed, and represented.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Creational Patterns Overview                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Pattern     │ Purpose                       │ When to Use          │
│  ────────────┼───────────────────────────────┼───────────────────── │
│  Singleton   │ Ensure single instance        │ Shared resources     │
│  Factory     │ Delegate object creation      │ Unknown type ahead   │
│  Builder     │ Construct complex objects     │ Many parameters      │
│  Prototype   │ Clone existing objects        │ Expensive creation   │
│  Abstract    │ Create families of objects    │ Related products     │
│  Factory     │                               │                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. Singleton Pattern

### Intent
Ensure a class has only **one instance** and provide a global point of access to it.

### Real-World Analogy
There's only one president of a country at a time. No matter how many people refer to "the president," they all mean the same person.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Singleton Structure                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                    ┌─────────────────────┐                          │
│                    │     Singleton       │                          │
│                    ├─────────────────────┤                          │
│                    │ - instance: static  │ ◄── Only one instance    │
│                    ├─────────────────────┤                          │
│                    │ - constructor()     │ ◄── Private!             │
│                    │ + getInstance()     │ ◄── Returns THE instance │
│                    │ + businessMethod()  │                          │
│                    └─────────────────────┘                          │
│                                                                     │
│   Client ─────────────────────────────────► getInstance()           │
│                                                  │                  │
│                                                  ▼                  │
│                                           ┌───────────┐             │
│                                           │ instance  │             │
│                                           └───────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### When to Use
- Database connection pools
- Configuration managers
- Logger instances
- Cache managers
- Thread pools

### When NOT to Use
- When you need multiple instances
- When testing requires isolation
- When global state creates hidden dependencies

### Python Implementation

```python
from threading import Lock

# Method 1: Classic Singleton (Not Thread-Safe)
class SingletonBasic:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        # Warning: __init__ called every time!
        pass

# Method 2: Thread-Safe Singleton with Double-Checked Locking
class DatabaseConnection:
    _instance = None
    _lock = Lock()

    def __new__(cls, *args, **kwargs):
        # First check (without lock for performance)
        if cls._instance is None:
            with cls._lock:
                # Second check (with lock for thread safety)
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, connection_string: str = "default"):
        # Only initialize once
        if not hasattr(self, '_initialized'):
            self._connection_string = connection_string
            self._connection = None
            self._initialized = True
            print(f"Database connection created: {connection_string}")

    def connect(self):
        if self._connection is None:
            self._connection = f"Connection to {self._connection_string}"
        return self._connection

    def execute(self, query: str):
        print(f"Executing: {query}")
        return []

# Method 3: Metaclass Singleton (Pythonic, Thread-Safe)
class SingletonMeta(type):
    _instances = {}
    _lock = Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self, log_file: str = "app.log"):
        self._log_file = log_file
        self._logs = []
        print(f"Logger initialized with file: {log_file}")

    def log(self, message: str, level: str = "INFO"):
        entry = f"[{level}] {message}"
        self._logs.append(entry)
        print(entry)

    def get_logs(self) -> list:
        return self._logs.copy()

# Method 4: Module-Level Singleton (Simplest in Python)
# config.py
class _Config:
    def __init__(self):
        self.debug = False
        self.database_url = "postgres://localhost/db"
        self.api_key = None

config = _Config()  # Single instance at module level

# Usage demonstrations
if __name__ == "__main__":
    # Database Connection
    db1 = DatabaseConnection("postgres://localhost")
    db2 = DatabaseConnection("mysql://other")  # Ignored! Same instance
    print(f"Same instance? {db1 is db2}")  # True

    # Logger
    logger1 = Logger("app.log")
    logger2 = Logger("different.log")  # Ignored!
    logger1.log("Hello from logger1")
    logger2.log("Hello from logger2")
    print(f"Same logger? {logger1 is logger2}")  # True
    print(f"Logs: {logger1.get_logs()}")  # Both messages
```

### Java Implementation

```java
// Method 1: Eager Initialization (Simple, Thread-Safe)
public class EagerSingleton {
    // Instance created at class loading
    private static final EagerSingleton INSTANCE = new EagerSingleton();

    private EagerSingleton() {
        // Private constructor
    }

    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}

// Method 2: Lazy Initialization with Double-Checked Locking
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private final String connectionString;

    private DatabaseConnection(String connectionString) {
        this.connectionString = connectionString;
        System.out.println("Database connection created: " + connectionString);
    }

    public static DatabaseConnection getInstance(String connectionString) {
        // First check (no lock)
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                // Second check (with lock)
                if (instance == null) {
                    instance = new DatabaseConnection(connectionString);
                }
            }
        }
        return instance;
    }

    public void execute(String query) {
        System.out.println("Executing: " + query);
    }
}

// Method 3: Bill Pugh Singleton (Recommended)
public class Logger {
    private final String logFile;
    private final List<String> logs;

    private Logger() {
        this.logFile = "app.log";
        this.logs = new ArrayList<>();
        System.out.println("Logger initialized");
    }

    // Static inner class - loaded only when getInstance() is called
    private static class LoggerHolder {
        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return LoggerHolder.INSTANCE;
    }

    public void log(String message, String level) {
        String entry = String.format("[%s] %s", level, message);
        logs.add(entry);
        System.out.println(entry);
    }

    public List<String> getLogs() {
        return new ArrayList<>(logs);
    }
}

// Method 4: Enum Singleton (Best for Java)
public enum ConfigurationManager {
    INSTANCE;

    private String databaseUrl;
    private boolean debugMode;

    ConfigurationManager() {
        // Load configuration
        this.databaseUrl = "postgres://localhost/db";
        this.debugMode = false;
    }

    public String getDatabaseUrl() {
        return databaseUrl;
    }

    public void setDebugMode(boolean debug) {
        this.debugMode = debug;
    }

    public boolean isDebugMode() {
        return debugMode;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Database Connection
        DatabaseConnection db1 = DatabaseConnection.getInstance("postgres://localhost");
        DatabaseConnection db2 = DatabaseConnection.getInstance("mysql://other");
        System.out.println("Same instance? " + (db1 == db2));  // true

        // Logger
        Logger logger = Logger.getInstance();
        logger.log("Application started", "INFO");

        // Enum Singleton
        ConfigurationManager config = ConfigurationManager.INSTANCE;
        config.setDebugMode(true);
        System.out.println("Debug mode: " + config.isDebugMode());
    }
}
```

### Interview Traps

| Trap | Explanation |
|------|-------------|
| Thread Safety | Basic implementations aren't thread-safe |
| Serialization | Deserialization creates new instances |
| Reflection | Can bypass private constructor |
| Testing | Singletons create global state, hard to test |
| Subclassing | Can create multiple instances via inheritance |

---

## 2. Factory Pattern

### Intent
Define an interface for creating objects, but let subclasses decide which class to instantiate.

### Real-World Analogy
A restaurant menu lists dishes (interface), but the kitchen (factory) decides how to prepare each dish.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Factory Pattern Structure                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────┐         ┌──────────────────────┐                  │
│   │   Client    │────────►│      Creator         │                  │
│   └─────────────┘         ├──────────────────────┤                  │
│                           │ + createProduct()    │ ◄── Factory      │
│                           └──────────┬───────────┘     Method       │
│                                      │                              │
│                          ┌───────────┴───────────┐                  │
│                          ▼                       ▼                  │
│              ┌───────────────────┐   ┌───────────────────┐          │
│              │ ConcreteCreatorA  │   │ ConcreteCreatorB  │          │
│              ├───────────────────┤   ├───────────────────┤          │
│              │ + createProduct() │   │ + createProduct() │          │
│              └─────────┬─────────┘   └─────────┬─────────┘          │
│                        │                       │                    │
│                        ▼                       ▼                    │
│              ┌───────────────────┐   ┌───────────────────┐          │
│              │   ProductA        │   │   ProductB        │          │
│              └───────────────────┘   └───────────────────┘          │
│                        ▲                       ▲                    │
│                        └───────────┬───────────┘                    │
│                                    │                                │
│                           ┌────────┴────────┐                       │
│                           │    Product      │ ◄── Interface         │
│                           │    Interface    │                       │
│                           └─────────────────┘                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### When to Use
- Unknown object types at compile time
- Centralizing complex object creation
- Extending system with new product types
- Different implementations based on configuration

### Python Implementation

```python
from abc import ABC, abstractmethod
from enum import Enum

# Product Interface
class Notification(ABC):
    @abstractmethod
    def send(self, recipient: str, message: str) -> bool:
        pass

    @abstractmethod
    def get_type(self) -> str:
        pass

# Concrete Products
class EmailNotification(Notification):
    def __init__(self, smtp_server: str = "smtp.example.com"):
        self._smtp_server = smtp_server

    def send(self, recipient: str, message: str) -> bool:
        print(f"[EMAIL] Sending to {recipient} via {self._smtp_server}")
        print(f"  Message: {message}")
        return True

    def get_type(self) -> str:
        return "EMAIL"

class SMSNotification(Notification):
    def __init__(self, api_key: str = "default_key"):
        self._api_key = api_key

    def send(self, recipient: str, message: str) -> bool:
        print(f"[SMS] Sending to {recipient}")
        print(f"  Message: {message[:160]}")  # SMS length limit
        return True

    def get_type(self) -> str:
        return "SMS"

class PushNotification(Notification):
    def __init__(self, app_id: str = "my_app"):
        self._app_id = app_id

    def send(self, recipient: str, message: str) -> bool:
        print(f"[PUSH] Sending to device {recipient} via {self._app_id}")
        print(f"  Message: {message}")
        return True

    def get_type(self) -> str:
        return "PUSH"

class SlackNotification(Notification):
    def __init__(self, webhook_url: str):
        self._webhook_url = webhook_url

    def send(self, recipient: str, message: str) -> bool:
        print(f"[SLACK] Sending to channel {recipient}")
        print(f"  Message: {message}")
        return True

    def get_type(self) -> str:
        return "SLACK"

# Method 1: Simple Factory (Static Method)
class NotificationFactory:
    @staticmethod
    def create(notification_type: str, **kwargs) -> Notification:
        notification_type = notification_type.upper()

        if notification_type == "EMAIL":
            return EmailNotification(kwargs.get("smtp_server", "smtp.example.com"))
        elif notification_type == "SMS":
            return SMSNotification(kwargs.get("api_key", "default_key"))
        elif notification_type == "PUSH":
            return PushNotification(kwargs.get("app_id", "my_app"))
        elif notification_type == "SLACK":
            if "webhook_url" not in kwargs:
                raise ValueError("Slack requires webhook_url")
            return SlackNotification(kwargs["webhook_url"])
        else:
            raise ValueError(f"Unknown notification type: {notification_type}")

# Method 2: Factory Method Pattern
class NotificationCreator(ABC):
    @abstractmethod
    def create_notification(self) -> Notification:
        pass

    def notify(self, recipient: str, message: str) -> bool:
        notification = self.create_notification()
        return notification.send(recipient, message)

class EmailNotificationCreator(NotificationCreator):
    def __init__(self, smtp_server: str):
        self._smtp_server = smtp_server

    def create_notification(self) -> Notification:
        return EmailNotification(self._smtp_server)

class SMSNotificationCreator(NotificationCreator):
    def __init__(self, api_key: str):
        self._api_key = api_key

    def create_notification(self) -> Notification:
        return SMSNotification(self._api_key)

# Method 3: Registry-Based Factory (Extensible)
class NotificationRegistry:
    _creators: dict[str, type] = {}

    @classmethod
    def register(cls, notification_type: str, creator: type):
        cls._creators[notification_type.upper()] = creator

    @classmethod
    def create(cls, notification_type: str, **kwargs) -> Notification:
        notification_type = notification_type.upper()
        if notification_type not in cls._creators:
            raise ValueError(f"Unknown type: {notification_type}")
        return cls._creators[notification_type](**kwargs)

# Register notification types
NotificationRegistry.register("EMAIL", EmailNotification)
NotificationRegistry.register("SMS", SMSNotification)
NotificationRegistry.register("PUSH", PushNotification)

# Usage
if __name__ == "__main__":
    # Simple Factory
    print("=== Simple Factory ===")
    email = NotificationFactory.create("email", smtp_server="mail.company.com")
    email.send("user@example.com", "Welcome!")

    sms = NotificationFactory.create("sms", api_key="secret123")
    sms.send("+1234567890", "Your code is 123456")

    # Factory Method
    print("\n=== Factory Method ===")
    creator = EmailNotificationCreator("smtp.company.com")
    creator.notify("user@example.com", "Order confirmed!")

    # Registry Factory
    print("\n=== Registry Factory ===")
    push = NotificationRegistry.create("push", app_id="mobile_app")
    push.send("device_token_123", "You have a new message!")
```

### Java Implementation

```java
// Product Interface
public interface Notification {
    boolean send(String recipient, String message);
    String getType();
}

// Concrete Products
public class EmailNotification implements Notification {
    private final String smtpServer;

    public EmailNotification(String smtpServer) {
        this.smtpServer = smtpServer;
    }

    @Override
    public boolean send(String recipient, String message) {
        System.out.printf("[EMAIL] Sending to %s via %s%n", recipient, smtpServer);
        System.out.printf("  Message: %s%n", message);
        return true;
    }

    @Override
    public String getType() { return "EMAIL"; }
}

public class SMSNotification implements Notification {
    private final String apiKey;

    public SMSNotification(String apiKey) {
        this.apiKey = apiKey;
    }

    @Override
    public boolean send(String recipient, String message) {
        System.out.printf("[SMS] Sending to %s%n", recipient);
        System.out.printf("  Message: %s%n", message.substring(0, Math.min(160, message.length())));
        return true;
    }

    @Override
    public String getType() { return "SMS"; }
}

public class PushNotification implements Notification {
    private final String appId;

    public PushNotification(String appId) {
        this.appId = appId;
    }

    @Override
    public boolean send(String recipient, String message) {
        System.out.printf("[PUSH] Sending to device %s via %s%n", recipient, appId);
        System.out.printf("  Message: %s%n", message);
        return true;
    }

    @Override
    public String getType() { return "PUSH"; }
}

// Simple Factory
public class NotificationFactory {
    public static Notification create(String type, Map<String, String> config) {
        switch (type.toUpperCase()) {
            case "EMAIL":
                return new EmailNotification(config.getOrDefault("smtpServer", "smtp.example.com"));
            case "SMS":
                return new SMSNotification(config.getOrDefault("apiKey", "default"));
            case "PUSH":
                return new PushNotification(config.getOrDefault("appId", "my_app"));
            default:
                throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}

// Factory Method Pattern
public abstract class NotificationCreator {
    public abstract Notification createNotification();

    public boolean notify(String recipient, String message) {
        Notification notification = createNotification();
        return notification.send(recipient, message);
    }
}

public class EmailNotificationCreator extends NotificationCreator {
    private final String smtpServer;

    public EmailNotificationCreator(String smtpServer) {
        this.smtpServer = smtpServer;
    }

    @Override
    public Notification createNotification() {
        return new EmailNotification(smtpServer);
    }
}

// Registry-Based Factory
public class NotificationRegistry {
    private static final Map<String, Supplier<Notification>> creators = new HashMap<>();

    public static void register(String type, Supplier<Notification> creator) {
        creators.put(type.toUpperCase(), creator);
    }

    public static Notification create(String type) {
        Supplier<Notification> creator = creators.get(type.toUpperCase());
        if (creator == null) {
            throw new IllegalArgumentException("Unknown type: " + type);
        }
        return creator.get();
    }

    static {
        register("EMAIL", () -> new EmailNotification("smtp.example.com"));
        register("SMS", () -> new SMSNotification("default_key"));
        register("PUSH", () -> new PushNotification("my_app"));
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Simple Factory
        Notification email = NotificationFactory.create("email", Map.of("smtpServer", "mail.company.com"));
        email.send("user@example.com", "Welcome!");

        // Factory Method
        NotificationCreator creator = new EmailNotificationCreator("smtp.company.com");
        creator.notify("user@example.com", "Order confirmed!");

        // Registry Factory
        Notification push = NotificationRegistry.create("push");
        push.send("device_123", "New message!");
    }
}
```

---

## 3. Builder Pattern

### Intent
Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

### Real-World Analogy
Building a custom pizza: you specify crust, sauce, toppings step by step. The pizza builder handles the construction details.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Builder Pattern Structure                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌───────────┐        ┌────────────────────────┐                   │
│   │  Client   │───────►│       Director         │                   │
│   └───────────┘        │ (Optional Orchestrator)│                   │
│                        └───────────┬────────────┘                   │
│                                    │ uses                           │
│                                    ▼                                │
│                        ┌────────────────────────┐                   │
│                        │    <<interface>>       │                   │
│                        │       Builder          │                   │
│                        ├────────────────────────┤                   │
│                        │ + buildPartA()         │                   │
│                        │ + buildPartB()         │                   │
│                        │ + getResult()          │                   │
│                        └───────────┬────────────┘                   │
│                                    │                                │
│                        ┌───────────┴───────────┐                    │
│                        ▼                       ▼                    │
│            ┌───────────────────┐   ┌───────────────────┐            │
│            │ ConcreteBuilderA  │   │ ConcreteBuilderB  │            │
│            └─────────┬─────────┘   └─────────┬─────────┘            │
│                      │ creates               │ creates              │
│                      ▼                       ▼                      │
│            ┌───────────────────┐   ┌───────────────────┐            │
│            │    ProductA       │   │    ProductB       │            │
│            └───────────────────┘   └───────────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### When to Use
- Objects with many optional parameters
- Step-by-step construction process
- Creating different representations of same type
- Avoiding "telescoping constructor" anti-pattern

### Python Implementation

```python
from dataclasses import dataclass, field
from typing import Optional
from enum import Enum

# The complex product
@dataclass
class HttpRequest:
    method: str
    url: str
    headers: dict = field(default_factory=dict)
    query_params: dict = field(default_factory=dict)
    body: Optional[str] = None
    timeout: int = 30
    retries: int = 3
    auth_token: Optional[str] = None

    def __str__(self):
        return f"""
HttpRequest:
  {self.method} {self.url}
  Headers: {self.headers}
  Query Params: {self.query_params}
  Body: {self.body[:50] if self.body else 'None'}...
  Timeout: {self.timeout}s, Retries: {self.retries}
  Auth: {'Set' if self.auth_token else 'None'}
"""

# Builder with fluent interface
class HttpRequestBuilder:
    def __init__(self):
        self._reset()

    def _reset(self):
        self._method = "GET"
        self._url = ""
        self._headers = {}
        self._query_params = {}
        self._body = None
        self._timeout = 30
        self._retries = 3
        self._auth_token = None

    def method(self, method: str) -> 'HttpRequestBuilder':
        self._method = method.upper()
        return self

    def url(self, url: str) -> 'HttpRequestBuilder':
        self._url = url
        return self

    def header(self, key: str, value: str) -> 'HttpRequestBuilder':
        self._headers[key] = value
        return self

    def headers(self, headers: dict) -> 'HttpRequestBuilder':
        self._headers.update(headers)
        return self

    def query_param(self, key: str, value: str) -> 'HttpRequestBuilder':
        self._query_params[key] = value
        return self

    def body(self, body: str) -> 'HttpRequestBuilder':
        self._body = body
        return self

    def json_body(self, data: dict) -> 'HttpRequestBuilder':
        import json
        self._body = json.dumps(data)
        self._headers["Content-Type"] = "application/json"
        return self

    def timeout(self, seconds: int) -> 'HttpRequestBuilder':
        self._timeout = seconds
        return self

    def retries(self, count: int) -> 'HttpRequestBuilder':
        self._retries = count
        return self

    def auth_token(self, token: str) -> 'HttpRequestBuilder':
        self._auth_token = token
        self._headers["Authorization"] = f"Bearer {token}"
        return self

    def build(self) -> HttpRequest:
        if not self._url:
            raise ValueError("URL is required")

        request = HttpRequest(
            method=self._method,
            url=self._url,
            headers=self._headers.copy(),
            query_params=self._query_params.copy(),
            body=self._body,
            timeout=self._timeout,
            retries=self._retries,
            auth_token=self._auth_token
        )
        self._reset()  # Ready for next build
        return request

# Director (optional) - knows how to build common requests
class HttpRequestDirector:
    def __init__(self, builder: HttpRequestBuilder):
        self._builder = builder

    def build_authenticated_get(self, url: str, token: str) -> HttpRequest:
        return (self._builder
                .method("GET")
                .url(url)
                .auth_token(token)
                .header("Accept", "application/json")
                .timeout(10)
                .build())

    def build_json_post(self, url: str, data: dict, token: str) -> HttpRequest:
        return (self._builder
                .method("POST")
                .url(url)
                .auth_token(token)
                .json_body(data)
                .timeout(30)
                .retries(5)
                .build())

# Another example: SQL Query Builder
class SQLQueryBuilder:
    def __init__(self):
        self._reset()

    def _reset(self):
        self._select = ["*"]
        self._from = ""
        self._where = []
        self._order_by = []
        self._limit = None
        self._offset = None
        self._joins = []

    def select(self, *columns) -> 'SQLQueryBuilder':
        self._select = list(columns) if columns else ["*"]
        return self

    def from_table(self, table: str) -> 'SQLQueryBuilder':
        self._from = table
        return self

    def where(self, condition: str) -> 'SQLQueryBuilder':
        self._where.append(condition)
        return self

    def and_where(self, condition: str) -> 'SQLQueryBuilder':
        return self.where(condition)

    def order_by(self, column: str, desc: bool = False) -> 'SQLQueryBuilder':
        direction = "DESC" if desc else "ASC"
        self._order_by.append(f"{column} {direction}")
        return self

    def limit(self, count: int) -> 'SQLQueryBuilder':
        self._limit = count
        return self

    def offset(self, count: int) -> 'SQLQueryBuilder':
        self._offset = count
        return self

    def join(self, table: str, on: str) -> 'SQLQueryBuilder':
        self._joins.append(f"JOIN {table} ON {on}")
        return self

    def left_join(self, table: str, on: str) -> 'SQLQueryBuilder':
        self._joins.append(f"LEFT JOIN {table} ON {on}")
        return self

    def build(self) -> str:
        if not self._from:
            raise ValueError("FROM table is required")

        query_parts = [
            f"SELECT {', '.join(self._select)}",
            f"FROM {self._from}"
        ]

        if self._joins:
            query_parts.extend(self._joins)

        if self._where:
            query_parts.append(f"WHERE {' AND '.join(self._where)}")

        if self._order_by:
            query_parts.append(f"ORDER BY {', '.join(self._order_by)}")

        if self._limit:
            query_parts.append(f"LIMIT {self._limit}")

        if self._offset:
            query_parts.append(f"OFFSET {self._offset}")

        query = "\n".join(query_parts)
        self._reset()
        return query

# Usage
if __name__ == "__main__":
    # HTTP Request Builder
    print("=== HTTP Request Builder ===")
    builder = HttpRequestBuilder()

    # Fluent API usage
    request = (builder
               .method("POST")
               .url("https://api.example.com/users")
               .header("Accept", "application/json")
               .json_body({"name": "John", "email": "john@example.com"})
               .auth_token("secret_token_123")
               .timeout(60)
               .retries(5)
               .build())
    print(request)

    # Using Director
    director = HttpRequestDirector(builder)
    get_request = director.build_authenticated_get(
        "https://api.example.com/users/123",
        "my_token"
    )
    print(get_request)

    # SQL Query Builder
    print("=== SQL Query Builder ===")
    sql_builder = SQLQueryBuilder()

    query = (sql_builder
             .select("u.id", "u.name", "o.total")
             .from_table("users u")
             .left_join("orders o", "u.id = o.user_id")
             .where("u.active = true")
             .and_where("o.total > 100")
             .order_by("o.total", desc=True)
             .limit(10)
             .build())
    print(query)
```

### Java Implementation

```java
// The complex product
public class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final Map<String, String> queryParams;
    private final String body;
    private final int timeout;
    private final int retries;
    private final String authToken;

    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = new HashMap<>(builder.headers);
        this.queryParams = new HashMap<>(builder.queryParams);
        this.body = builder.body;
        this.timeout = builder.timeout;
        this.retries = builder.retries;
        this.authToken = builder.authToken;
    }

    // Getters...
    public String getMethod() { return method; }
    public String getUrl() { return url; }

    @Override
    public String toString() {
        return String.format("HttpRequest: %s %s, Headers: %s", method, url, headers);
    }

    // Static inner Builder class
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private Map<String, String> queryParams = new HashMap<>();
        private String body;
        private int timeout = 30;
        private int retries = 3;
        private String authToken;

        public Builder(String url) {
            this.url = url;
        }

        public Builder method(String method) {
            this.method = method.toUpperCase();
            return this;
        }

        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }

        public Builder queryParam(String key, String value) {
            this.queryParams.put(key, value);
            return this;
        }

        public Builder body(String body) {
            this.body = body;
            return this;
        }

        public Builder jsonBody(Map<String, Object> data) {
            // Simplified - use Jackson in production
            this.body = data.toString();
            this.headers.put("Content-Type", "application/json");
            return this;
        }

        public Builder timeout(int seconds) {
            this.timeout = seconds;
            return this;
        }

        public Builder retries(int count) {
            this.retries = count;
            return this;
        }

        public Builder authToken(String token) {
            this.authToken = token;
            this.headers.put("Authorization", "Bearer " + token);
            return this;
        }

        public HttpRequest build() {
            if (url == null || url.isEmpty()) {
                throw new IllegalStateException("URL is required");
            }
            return new HttpRequest(this);
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
            .method("POST")
            .header("Accept", "application/json")
            .jsonBody(Map.of("name", "John", "email", "john@example.com"))
            .authToken("secret_token")
            .timeout(60)
            .retries(5)
            .build();

        System.out.println(request);
    }
}
```

---

## Pattern Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│              Creational Patterns Comparison                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Pattern   │ Creates    │ Instance Count │ Construction Complexity  │
│  ──────────┼────────────┼────────────────┼───────────────────────── │
│  Singleton │ One object │ Exactly 1      │ Simple                   │
│  Factory   │ One object │ Many           │ Medium                   │
│  Builder   │ One object │ Many           │ Complex (step-by-step)   │
│  Prototype │ One object │ Many (clones)  │ Medium (via cloning)     │
│  Abstract  │ Family of  │ Many           │ Medium                   │
│  Factory   │ objects    │                │                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Interview Tips

1. **Know when to use each pattern** - Understand the problem each solves
2. **Show variations** - Mention thread-safe, lazy, eager implementations
3. **Discuss trade-offs** - No pattern is perfect for all situations
4. **Code cleanly** - Builder especially shows fluent API skills
5. **Real examples** - Logger (Singleton), Parser (Factory), Query (Builder)

---

**Next:** [04_design_patterns_structural.md](./04_design_patterns_structural.md) - Adapter, Decorator, and Proxy patterns.
