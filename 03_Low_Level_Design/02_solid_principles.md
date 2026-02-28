# SOLID Principles

## What are SOLID Principles?

SOLID is an acronym for five design principles that help create maintainable, flexible, and scalable object-oriented software. These principles are **essential** for LLD interviews.

```
┌─────────────────────────────────────────────────────────────────────┐
│                       SOLID Principles                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   S  ─────  Single Responsibility Principle (SRP)                   │
│              "A class should have only ONE reason to change"        │
│                                                                     │
│   O  ─────  Open/Closed Principle (OCP)                             │
│              "Open for extension, closed for modification"          │
│                                                                     │
│   L  ─────  Liskov Substitution Principle (LSP)                     │
│              "Subtypes must be substitutable for their base types"  │
│                                                                     │
│   I  ─────  Interface Segregation Principle (ISP)                   │
│              "Many specific interfaces > one general interface"     │
│                                                                     │
│   D  ─────  Dependency Inversion Principle (DIP)                    │
│              "Depend on abstractions, not concretions"              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. Single Responsibility Principle (SRP)

> **"A class should have only one reason to change."**

A class should do ONE thing and do it well. If a class has multiple responsibilities, changes to one responsibility might break another.

### Real-World Analogy
A chef cooks food. A waiter serves food. A cashier handles payments. If one person did all three, they'd be overwhelmed and make mistakes.

### Bad Example - Violating SRP

**Python:**
```python
# BAD: This class has TOO MANY responsibilities
class Employee:
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary

    # Responsibility 1: Employee data
    def get_name(self) -> str:
        return self.name

    # Responsibility 2: Salary calculation
    def calculate_pay(self) -> float:
        return self.salary * 1.1  # With bonus

    # Responsibility 3: Database operations
    def save_to_database(self):
        # Database connection, SQL queries...
        print(f"INSERT INTO employees VALUES ('{self.name}', {self.salary})")

    # Responsibility 4: Report generation
    def generate_report(self) -> str:
        return f"Employee Report\nName: {self.name}\nSalary: ${self.salary}"

    # Responsibility 5: Email sending
    def send_email(self, message: str):
        # SMTP connection, email formatting...
        print(f"Sending email to {self.name}: {message}")

# Problems:
# 1. Changes to database schema affect this class
# 2. Changes to email format affect this class
# 3. Changes to report format affect this class
# 4. Testing is complex - need database, email server
```

**Java:**
```java
// BAD: Too many responsibilities
public class Employee {
    private String name;
    private double salary;

    // Responsibility 1: Employee data
    public String getName() { return name; }

    // Responsibility 2: Salary calculation
    public double calculatePay() {
        return salary * 1.1;
    }

    // Responsibility 3: Database operations
    public void saveToDatabase() {
        // SQL, connections, etc.
    }

    // Responsibility 4: Report generation
    public String generateReport() {
        return "Employee Report\n...";
    }

    // Responsibility 5: Email sending
    public void sendEmail(String message) {
        // SMTP, formatting, etc.
    }
}
```

### Good Example - Following SRP

**Python:**
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol

# Responsibility 1: Employee Data (pure data class)
@dataclass
class Employee:
    employee_id: str
    name: str
    email: str
    base_salary: float

# Responsibility 2: Salary Calculation
class PayCalculator:
    def __init__(self, tax_rate: float = 0.2):
        self._tax_rate = tax_rate

    def calculate_gross_pay(self, employee: Employee) -> float:
        return employee.base_salary

    def calculate_net_pay(self, employee: Employee) -> float:
        gross = self.calculate_gross_pay(employee)
        return gross * (1 - self._tax_rate)

# Responsibility 3: Database Operations
class EmployeeRepository:
    def __init__(self, connection_string: str):
        self._connection = connection_string

    def save(self, employee: Employee) -> bool:
        # Only handles database operations
        print(f"Saving {employee.name} to database")
        return True

    def find_by_id(self, employee_id: str) -> Employee:
        # Query database
        return Employee("E001", "John", "john@email.com", 50000)

    def delete(self, employee_id: str) -> bool:
        return True

# Responsibility 4: Report Generation
class ReportGenerator:
    def generate_employee_report(self, employee: Employee) -> str:
        return f"""
        ╔══════════════════════════════════╗
        ║       EMPLOYEE REPORT            ║
        ╠══════════════════════════════════╣
        ║ ID:     {employee.employee_id:<20} ║
        ║ Name:   {employee.name:<20} ║
        ║ Email:  {employee.email:<20} ║
        ║ Salary: ${employee.base_salary:<19,.2f} ║
        ╚══════════════════════════════════╝
        """

# Responsibility 5: Email Notification
class EmailService:
    def __init__(self, smtp_server: str):
        self._smtp_server = smtp_server

    def send_email(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending email to {to}: {subject}")
        return True

# Usage - each class does ONE thing
employee = Employee("E001", "Alice", "alice@company.com", 75000)

repository = EmployeeRepository("postgres://...")
repository.save(employee)

calculator = PayCalculator(tax_rate=0.25)
net_pay = calculator.calculate_net_pay(employee)

reporter = ReportGenerator()
report = reporter.generate_employee_report(employee)

emailer = EmailService("smtp.company.com")
emailer.send_email(employee.email, "Welcome!", "Welcome to the company!")
```

**Java:**
```java
// Responsibility 1: Pure Data
public class Employee {
    private final String employeeId;
    private final String name;
    private final String email;
    private final double baseSalary;

    public Employee(String id, String name, String email, double salary) {
        this.employeeId = id;
        this.name = name;
        this.email = email;
        this.baseSalary = salary;
    }

    // Only getters - no business logic
    public String getEmployeeId() { return employeeId; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public double getBaseSalary() { return baseSalary; }
}

// Responsibility 2: Pay Calculation
public class PayCalculator {
    private final double taxRate;

    public PayCalculator(double taxRate) {
        this.taxRate = taxRate;
    }

    public double calculateNetPay(Employee employee) {
        return employee.getBaseSalary() * (1 - taxRate);
    }
}

// Responsibility 3: Database Operations
public class EmployeeRepository {
    private final DataSource dataSource;

    public void save(Employee employee) { /* DB logic only */ }
    public Employee findById(String id) { /* DB logic only */ }
    public void delete(String id) { /* DB logic only */ }
}

// Responsibility 4: Report Generation
public class ReportGenerator {
    public String generateEmployeeReport(Employee employee) {
        return String.format("Report for %s...", employee.getName());
    }
}

// Responsibility 5: Email
public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        // Email logic only
    }
}
```

### Benefits of SRP
- **Easier Testing:** Each class tests one thing
- **Lower Coupling:** Changes isolated to one class
- **Better Organization:** Clear where code belongs
- **Parallel Development:** Teams work on separate responsibilities

---

## 2. Open/Closed Principle (OCP)

> **"Software entities should be open for extension but closed for modification."**

You should be able to add new functionality WITHOUT changing existing code.

### Real-World Analogy
A smartphone lets you install new apps without modifying the phone's operating system. The OS is "closed" for modification but "open" for extension via apps.

### Bad Example - Violating OCP

**Python:**
```python
# BAD: Adding a new discount type requires modifying existing code
class DiscountCalculator:
    def calculate_discount(self, customer_type: str, amount: float) -> float:
        if customer_type == "regular":
            return amount * 0.1
        elif customer_type == "premium":
            return amount * 0.2
        elif customer_type == "vip":
            return amount * 0.3
        # Adding new customer type requires modifying this class!
        # elif customer_type == "enterprise":
        #     return amount * 0.4
        else:
            return 0

# Every new discount type = change to this class = risk of bugs
```

### Good Example - Following OCP

**Python:**
```python
from abc import ABC, abstractmethod

# Abstract base for discount strategies
class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

    @property
    @abstractmethod
    def name(self) -> str:
        pass

# Concrete implementations - each can be added WITHOUT modifying existing code
class RegularDiscount(DiscountStrategy):
    @property
    def name(self) -> str:
        return "Regular"

    def calculate(self, amount: float) -> float:
        return amount * 0.1

class PremiumDiscount(DiscountStrategy):
    @property
    def name(self) -> str:
        return "Premium"

    def calculate(self, amount: float) -> float:
        return amount * 0.2

class VIPDiscount(DiscountStrategy):
    @property
    def name(self) -> str:
        return "VIP"

    def calculate(self, amount: float) -> float:
        return amount * 0.3

# NEW: Add enterprise discount without changing existing code!
class EnterpriseDiscount(DiscountStrategy):
    @property
    def name(self) -> str:
        return "Enterprise"

    def calculate(self, amount: float) -> float:
        return amount * 0.4

# Calculator is CLOSED for modification
class DiscountCalculator:
    def calculate_discount(self, strategy: DiscountStrategy, amount: float) -> float:
        return strategy.calculate(amount)

# Usage
calculator = DiscountCalculator()

# Works with any discount strategy - open for extension!
print(calculator.calculate_discount(RegularDiscount(), 100))    # 10.0
print(calculator.calculate_discount(PremiumDiscount(), 100))    # 20.0
print(calculator.calculate_discount(EnterpriseDiscount(), 100)) # 40.0
```

**Java:**
```java
// Strategy interface
public interface DiscountStrategy {
    double calculate(double amount);
    String getName();
}

// Implementations
public class RegularDiscount implements DiscountStrategy {
    @Override
    public double calculate(double amount) { return amount * 0.1; }
    @Override
    public String getName() { return "Regular"; }
}

public class PremiumDiscount implements DiscountStrategy {
    @Override
    public double calculate(double amount) { return amount * 0.2; }
    @Override
    public String getName() { return "Premium"; }
}

// NEW: Just add new class, no existing code changes!
public class EnterpriseDiscount implements DiscountStrategy {
    @Override
    public double calculate(double amount) { return amount * 0.4; }
    @Override
    public String getName() { return "Enterprise"; }
}

// Calculator never changes
public class DiscountCalculator {
    public double calculateDiscount(DiscountStrategy strategy, double amount) {
        return strategy.calculate(amount);
    }
}
```

---

## 3. Liskov Substitution Principle (LSP)

> **"Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."**

If class B is a subclass of class A, you should be able to use B anywhere A is expected without unexpected behavior.

### Real-World Analogy
If you ask for "a vehicle," you should be able to drive a car, truck, or motorcycle without changing how you drive.

### Bad Example - Violating LSP

**Python:**
```python
# BAD: Square violates LSP when inheriting from Rectangle
class Rectangle:
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height

    @property
    def width(self) -> float:
        return self._width

    @width.setter
    def width(self, value: float):
        self._width = value

    @property
    def height(self) -> float:
        return self._height

    @height.setter
    def height(self, value: float):
        self._height = value

    def area(self) -> float:
        return self._width * self._height

# Square "is a" Rectangle mathematically, but...
class Square(Rectangle):
    def __init__(self, side: float):
        super().__init__(side, side)

    @Rectangle.width.setter
    def width(self, value: float):
        # Squares must maintain equal sides!
        self._width = value
        self._height = value  # This breaks expectations!

    @Rectangle.height.setter
    def height(self, value: float):
        self._width = value
        self._height = value

# LSP violation demonstrated
def double_width(rect: Rectangle):
    """This function expects Rectangle behavior"""
    rect.width = rect.width * 2
    # For Rectangle: area doubles
    # For Square: area quadruples! Unexpected!

rect = Rectangle(5, 10)
print(f"Rectangle area: {rect.area()}")  # 50
double_width(rect)
print(f"After doubling width: {rect.area()}")  # 100 ✓

square = Square(5)
print(f"Square area: {square.area()}")  # 25
double_width(square)
print(f"After doubling width: {square.area()}")  # 100, not 50! ✗
```

### Good Example - Following LSP

**Python:**
```python
from abc import ABC, abstractmethod

# Better design: Shape is the abstraction
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

# Rectangle is a Shape with independent width and height
class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height

    def area(self) -> float:
        return self._width * self._height

    def scale_width(self, factor: float):
        self._width *= factor

    def scale_height(self, factor: float):
        self._height *= factor

# Square is a Shape with one dimension
class Square(Shape):
    def __init__(self, side: float):
        self._side = side

    def area(self) -> float:
        return self._side ** 2

    def scale(self, factor: float):
        self._side *= factor

# Now both can be used as Shapes without breaking expectations
def print_area(shape: Shape):
    print(f"Area: {shape.area()}")

print_area(Rectangle(5, 10))  # Area: 50
print_area(Square(5))         # Area: 25
# Both work correctly!
```

**Java:**
```java
// Better design with LSP
public interface Shape {
    double area();
}

public class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }

    public void scaleWidth(double factor) {
        this.width *= factor;
    }
}

public class Square implements Shape {
    private double side;

    public Square(double side) {
        this.side = side;
    }

    @Override
    public double area() {
        return side * side;
    }

    public void scale(double factor) {
        this.side *= factor;
    }
}

// Both work correctly as Shapes
public void printArea(Shape shape) {
    System.out.println("Area: " + shape.area());
}
```

### LSP Rules of Thumb
1. **Preconditions** cannot be strengthened in a subtype
2. **Postconditions** cannot be weakened in a subtype
3. **Invariants** must be preserved by the subtype
4. If it doesn't make sense to substitute, don't inherit

---

## 4. Interface Segregation Principle (ISP)

> **"Clients should not be forced to depend on interfaces they do not use."**

Many small, specific interfaces are better than one large, general-purpose interface.

### Real-World Analogy
A universal remote has 50 buttons, but you only use 5. A simple remote with just the buttons you need is better.

### Bad Example - Violating ISP

**Python:**
```python
from abc import ABC, abstractmethod

# BAD: One fat interface forces all implementers to define everything
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def sleep(self):
        pass

    @abstractmethod
    def receive_salary(self):
        pass

class HumanWorker(Worker):
    def work(self):
        print("Human working")

    def eat(self):
        print("Human eating")

    def sleep(self):
        print("Human sleeping")

    def receive_salary(self):
        print("Human receiving salary")

class RobotWorker(Worker):
    def work(self):
        print("Robot working")

    def eat(self):
        # Robots don't eat! Forced to implement anyway
        raise NotImplementedError("Robots don't eat")

    def sleep(self):
        # Robots don't sleep!
        raise NotImplementedError("Robots don't sleep")

    def receive_salary(self):
        # Robots don't get paid!
        raise NotImplementedError("Robots don't receive salary")
```

### Good Example - Following ISP

**Python:**
```python
from abc import ABC, abstractmethod

# GOOD: Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Sleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass

class Payable(ABC):
    @abstractmethod
    def receive_salary(self):
        pass

# Human implements all relevant interfaces
class HumanWorker(Workable, Eatable, Sleepable, Payable):
    def __init__(self, name: str):
        self.name = name

    def work(self):
        print(f"{self.name} is working")

    def eat(self):
        print(f"{self.name} is eating lunch")

    def sleep(self):
        print(f"{self.name} is sleeping")

    def receive_salary(self):
        print(f"{self.name} received salary")

# Robot only implements what makes sense
class RobotWorker(Workable):
    def __init__(self, model: str):
        self.model = model

    def work(self):
        print(f"Robot {self.model} is working")

# Volunteer works but doesn't get paid
class Volunteer(Workable, Eatable, Sleepable):
    def __init__(self, name: str):
        self.name = name

    def work(self):
        print(f"Volunteer {self.name} is working")

    def eat(self):
        print(f"Volunteer {self.name} is eating")

    def sleep(self):
        print(f"Volunteer {self.name} is sleeping")

# Functions depend only on what they need
def assign_work(workers: list[Workable]):
    for worker in workers:
        worker.work()

def pay_workers(workers: list[Payable]):
    for worker in workers:
        worker.receive_salary()

# Usage
human = HumanWorker("Alice")
robot = RobotWorker("R2D2")
volunteer = Volunteer("Bob")

all_workers: list[Workable] = [human, robot, volunteer]
assign_work(all_workers)  # Works for all

paid_workers: list[Payable] = [human]  # Only humans get paid
pay_workers(paid_workers)
```

**Java:**
```java
// Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public interface Payable {
    void receiveSalary();
}

// Human implements all
public class HumanWorker implements Workable, Eatable, Sleepable, Payable {
    private String name;

    public void work() { System.out.println(name + " is working"); }
    public void eat() { System.out.println(name + " is eating"); }
    public void sleep() { System.out.println(name + " is sleeping"); }
    public void receiveSalary() { System.out.println(name + " got paid"); }
}

// Robot only implements Workable
public class RobotWorker implements Workable {
    private String model;

    public void work() { System.out.println("Robot " + model + " is working"); }
}

// Type-safe functions
public void assignWork(List<Workable> workers) {
    workers.forEach(Workable::work);
}

public void payWorkers(List<Payable> workers) {
    workers.forEach(Payable::receiveSalary);
}
```

---

## 5. Dependency Inversion Principle (DIP)

> **"High-level modules should not depend on low-level modules. Both should depend on abstractions."**

Don't let your core business logic depend directly on infrastructure details like databases or external services.

### Real-World Analogy
A lamp doesn't care if it's connected to solar power or city power - it just needs electricity. The lamp depends on the "electricity interface," not a specific power source.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Dependency Inversion                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   WITHOUT DIP:                    WITH DIP:                         │
│   ────────────                    ────────────                      │
│                                                                     │
│   ┌────────────┐                  ┌────────────┐                    │
│   │ OrderService│                 │ OrderService│                   │
│   └──────┬─────┘                  └──────┬─────┘                    │
│          │ depends on                    │ depends on               │
│          ▼                               ▼                          │
│   ┌────────────┐                  ┌────────────┐                    │
│   │MySQLDatabase│                 │ <<interface>>│                  │
│   └────────────┘                  │ Repository │                    │
│                                   └──────┬─────┘                    │
│   Problem: Can't switch                  │ implemented by           │
│   to PostgreSQL easily     ┌─────────────┼─────────────┐            │
│                            ▼             ▼             ▼            │
│                    ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│                    │ MySQL    │  │PostgreSQL│  │  Mock    │         │
│                    │Repository│  │Repository│  │Repository│         │
│                    └──────────┘  └──────────┘  └──────────┘         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Bad Example - Violating DIP

**Python:**
```python
# BAD: High-level module depends on low-level module
class MySQLDatabase:
    def __init__(self, connection_string: str):
        self.connection = connection_string

    def execute_query(self, query: str) -> list:
        print(f"Executing MySQL query: {query}")
        return []

    def save(self, table: str, data: dict):
        print(f"Saving to MySQL table {table}: {data}")

class OrderService:
    def __init__(self):
        # Direct dependency on concrete class!
        self.database = MySQLDatabase("mysql://localhost/orders")

    def create_order(self, order_data: dict):
        # Tightly coupled to MySQL
        self.database.save("orders", order_data)

    def get_order(self, order_id: str):
        return self.database.execute_query(f"SELECT * FROM orders WHERE id = {order_id}")

# Problems:
# 1. Can't switch to PostgreSQL without changing OrderService
# 2. Can't unit test without a real database
# 3. OrderService knows too much about database implementation
```

### Good Example - Following DIP

**Python:**
```python
from abc import ABC, abstractmethod
from typing import Optional
from dataclasses import dataclass

@dataclass
class Order:
    order_id: str
    customer_id: str
    items: list
    total: float

# Abstraction - high-level defines what it needs
class OrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> bool:
        pass

    @abstractmethod
    def find_by_id(self, order_id: str) -> Optional[Order]:
        pass

    @abstractmethod
    def find_by_customer(self, customer_id: str) -> list[Order]:
        pass

# Low-level module implements the abstraction
class MySQLOrderRepository(OrderRepository):
    def __init__(self, connection_string: str):
        self._connection = connection_string

    def save(self, order: Order) -> bool:
        print(f"[MySQL] Saving order {order.order_id}")
        # MySQL-specific implementation
        return True

    def find_by_id(self, order_id: str) -> Optional[Order]:
        print(f"[MySQL] Finding order {order_id}")
        return Order(order_id, "C001", ["item1"], 100.0)

    def find_by_customer(self, customer_id: str) -> list[Order]:
        print(f"[MySQL] Finding orders for customer {customer_id}")
        return []

class PostgreSQLOrderRepository(OrderRepository):
    def __init__(self, connection_string: str):
        self._connection = connection_string

    def save(self, order: Order) -> bool:
        print(f"[PostgreSQL] Saving order {order.order_id}")
        return True

    def find_by_id(self, order_id: str) -> Optional[Order]:
        print(f"[PostgreSQL] Finding order {order_id}")
        return Order(order_id, "C001", ["item1"], 100.0)

    def find_by_customer(self, customer_id: str) -> list[Order]:
        print(f"[PostgreSQL] Finding orders for customer {customer_id}")
        return []

# For testing - no real database needed!
class InMemoryOrderRepository(OrderRepository):
    def __init__(self):
        self._orders: dict[str, Order] = {}

    def save(self, order: Order) -> bool:
        self._orders[order.order_id] = order
        return True

    def find_by_id(self, order_id: str) -> Optional[Order]:
        return self._orders.get(order_id)

    def find_by_customer(self, customer_id: str) -> list[Order]:
        return [o for o in self._orders.values() if o.customer_id == customer_id]

# High-level module depends on ABSTRACTION
class OrderService:
    def __init__(self, repository: OrderRepository):
        self._repository = repository  # Dependency injected!

    def create_order(self, customer_id: str, items: list) -> Order:
        order = Order(
            order_id=f"ORD-{hash(customer_id)}",
            customer_id=customer_id,
            items=items,
            total=sum(item.get('price', 0) for item in items)
        )
        self._repository.save(order)
        return order

    def get_order(self, order_id: str) -> Optional[Order]:
        return self._repository.find_by_id(order_id)

# Usage - easy to switch implementations
# Production with MySQL
mysql_repo = MySQLOrderRepository("mysql://prod-server/orders")
service = OrderService(mysql_repo)
service.create_order("C001", [{"name": "Widget", "price": 10}])

# Production with PostgreSQL - just change the repository!
postgres_repo = PostgreSQLOrderRepository("postgres://prod-server/orders")
service = OrderService(postgres_repo)
service.create_order("C001", [{"name": "Widget", "price": 10}])

# Testing - no database needed!
test_repo = InMemoryOrderRepository()
test_service = OrderService(test_repo)
order = test_service.create_order("C001", [{"name": "Widget", "price": 10}])
assert test_service.get_order(order.order_id) == order
```

**Java:**
```java
// Abstraction
public interface OrderRepository {
    boolean save(Order order);
    Optional<Order> findById(String orderId);
    List<Order> findByCustomer(String customerId);
}

// MySQL Implementation
public class MySQLOrderRepository implements OrderRepository {
    private final String connectionString;

    public MySQLOrderRepository(String connectionString) {
        this.connectionString = connectionString;
    }

    @Override
    public boolean save(Order order) {
        System.out.println("[MySQL] Saving order " + order.getOrderId());
        return true;
    }

    @Override
    public Optional<Order> findById(String orderId) {
        return Optional.of(new Order(orderId, "C001", List.of(), 100.0));
    }

    @Override
    public List<Order> findByCustomer(String customerId) {
        return List.of();
    }
}

// In-Memory for testing
public class InMemoryOrderRepository implements OrderRepository {
    private final Map<String, Order> orders = new HashMap<>();

    @Override
    public boolean save(Order order) {
        orders.put(order.getOrderId(), order);
        return true;
    }

    @Override
    public Optional<Order> findById(String orderId) {
        return Optional.ofNullable(orders.get(orderId));
    }

    @Override
    public List<Order> findByCustomer(String customerId) {
        return orders.values().stream()
            .filter(o -> o.getCustomerId().equals(customerId))
            .collect(Collectors.toList());
    }
}

// High-level depends on abstraction
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;  // Injected!
    }

    public Order createOrder(String customerId, List<Item> items) {
        Order order = new Order(/* ... */);
        repository.save(order);
        return order;
    }
}
```

---

## SOLID Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SOLID Quick Reference                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Principle    │ Violation Sign           │ Solution                 │
│  ─────────────┼──────────────────────────┼────────────────────────  │
│  SRP          │ Class has many reasons   │ Split into smaller       │
│               │ to change                │ classes                  │
│  ─────────────┼──────────────────────────┼────────────────────────  │
│  OCP          │ Adding feature requires  │ Use polymorphism and     │
│               │ modifying existing code  │ abstractions             │
│  ─────────────┼──────────────────────────┼────────────────────────  │
│  LSP          │ Subclass breaks parent's │ Rethink hierarchy;       │
│               │ contract/expectations    │ use composition          │
│  ─────────────┼──────────────────────────┼────────────────────────  │
│  ISP          │ Class forced to implement│ Break into smaller       │
│               │ unused methods           │ interfaces               │
│  ─────────────┼──────────────────────────┼────────────────────────  │
│  DIP          │ High-level depends on    │ Introduce abstractions;  │
│               │ concrete low-level       │ inject dependencies      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Interview Tips

1. **Don't over-engineer** - Apply SOLID when it adds value, not everywhere
2. **Know trade-offs** - More abstractions = more complexity
3. **Give examples** - Show violation then solution
4. **Explain "why"** - What problem does each principle solve?
5. **Recognize violations** - Interviewers may show code and ask what's wrong

---

**Next:** [03_design_patterns_creational.md](./03_design_patterns_creational.md) - Singleton, Factory, and Builder patterns.
