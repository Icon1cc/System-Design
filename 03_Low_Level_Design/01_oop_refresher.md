# Object-Oriented Programming Refresher

## Why OOP for System Design?

Object-Oriented Programming is the foundation of Low Level Design. Before diving into design patterns and SOLID principles, you need a solid grasp of OOP concepts.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    The Four Pillars of OOP                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│         ┌─────────────┐                  ┌─────────────┐            │
│         │ENCAPSULATION│                  │ ABSTRACTION │            │
│         │             │                  │             │            │
│         │ Hide data   │                  │ Hide        │            │
│         │ behind      │                  │ complexity  │            │
│         │ methods     │                  │ behind      │            │
│         │             │                  │ simple      │            │
│         │             │                  │ interface   │            │
│         └─────────────┘                  └─────────────┘            │
│                                                                     │
│         ┌─────────────┐                  ┌─────────────┐            │
│         │ INHERITANCE │                  │POLYMORPHISM │            │
│         │             │                  │             │            │
│         │ Reuse code  │                  │ Same        │            │
│         │ through     │                  │ interface,  │            │
│         │ parent-     │                  │ different   │            │
│         │ child       │                  │ behavior    │            │
│         │ relationship│                  │             │            │
│         └─────────────┘                  └─────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. Encapsulation

**Definition:** Bundling data and methods that operate on that data within a single unit (class), and restricting direct access to internal state.

### Real-World Analogy
Think of a car. You don't need to understand how the engine works to drive. You interact with a simple interface (steering wheel, pedals) while the complex mechanics are hidden.

### Bad Example (No Encapsulation)

**Python:**
```python
class BankAccount:
    def __init__(self):
        self.balance = 0  # Public - anyone can modify directly!

# Problem: No validation, no protection
account = BankAccount()
account.balance = -1000  # Invalid state allowed!
account.balance = "hello"  # Type safety violated!
```

**Java:**
```java
public class BankAccount {
    public double balance = 0;  // Public - dangerous!
}

// Problem: Anyone can corrupt the state
BankAccount account = new BankAccount();
account.balance = -1000;  // No validation!
```

### Good Example (Proper Encapsulation)

**Python:**
```python
class BankAccount:
    def __init__(self, initial_balance: float = 0):
        self._balance = initial_balance  # Private by convention
        self._transaction_history = []

    @property
    def balance(self) -> float:
        """Read-only access to balance"""
        return self._balance

    def deposit(self, amount: float) -> bool:
        """Deposit with validation"""
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self._balance += amount
        self._transaction_history.append(f"Deposit: {amount}")
        return True

    def withdraw(self, amount: float) -> bool:
        """Withdraw with validation"""
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self._balance:
            raise ValueError("Insufficient funds")
        self._balance -= amount
        self._transaction_history.append(f"Withdrawal: {amount}")
        return True

# Usage
account = BankAccount(100)
account.deposit(50)      # Works: balance = 150
account.withdraw(30)     # Works: balance = 120
# account._balance = -1000  # Still possible but clearly violates convention
```

**Java:**
```java
public class BankAccount {
    private double balance;
    private List<String> transactionHistory;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
        this.transactionHistory = new ArrayList<>();
    }

    // Getter only - no setter for balance
    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        balance += amount;
        transactionHistory.add("Deposit: " + amount);
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
        transactionHistory.add("Withdrawal: " + amount);
    }
}
```

### Key Benefits
1. **Data Validation** - Control how data is modified
2. **Flexibility** - Change internal implementation without affecting clients
3. **Debugging** - All changes go through methods (easy to add logging)
4. **Thread Safety** - Can add synchronization in methods

---

## 2. Abstraction

**Definition:** Hiding complex implementation details and showing only the necessary features of an object.

### Real-World Analogy
When you use a coffee machine, you press a button and get coffee. You don't need to know about water temperature, pressure, or brewing time.

### Python Example

```python
from abc import ABC, abstractmethod

# Abstraction: Define WHAT, not HOW
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        """Process a payment. Implementation details hidden."""
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        """Process a refund."""
        pass

# Concrete implementations hide complexity
class CreditCardProcessor(PaymentProcessor):
    def __init__(self, api_key: str):
        self._api_key = api_key
        self._gateway = self._initialize_gateway()

    def _initialize_gateway(self):
        # Complex gateway initialization hidden
        return "gateway_connection"

    def _validate_card(self, card_number: str) -> bool:
        # Luhn algorithm, network checks - all hidden
        return True

    def _fraud_check(self, amount: float) -> bool:
        # ML model, risk scoring - all hidden
        return True

    def process_payment(self, amount: float) -> bool:
        # Client just calls this simple method
        # Complex logic is abstracted away
        if self._fraud_check(amount):
            # Process through gateway...
            return True
        return False

    def refund(self, transaction_id: str) -> bool:
        # Complex refund logic hidden
        return True

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float) -> bool:
        # Different implementation, same interface
        return True

    def refund(self, transaction_id: str) -> bool:
        return True

# Client code - doesn't know implementation details
def checkout(processor: PaymentProcessor, amount: float):
    if processor.process_payment(amount):
        print("Payment successful!")
    else:
        print("Payment failed!")

# Works with any PaymentProcessor
checkout(CreditCardProcessor("api_key"), 100.00)
checkout(PayPalProcessor(), 100.00)
```

**Java:**
```java
// Abstraction through interface
public interface PaymentProcessor {
    boolean processPayment(double amount);
    boolean refund(String transactionId);
}

public class CreditCardProcessor implements PaymentProcessor {
    private final String apiKey;
    private final PaymentGateway gateway;

    public CreditCardProcessor(String apiKey) {
        this.apiKey = apiKey;
        this.gateway = initializeGateway();  // Complex, hidden
    }

    private PaymentGateway initializeGateway() {
        // Complex initialization hidden from clients
        return new PaymentGateway();
    }

    private boolean fraudCheck(double amount) {
        // ML model, risk scoring - all hidden
        return true;
    }

    @Override
    public boolean processPayment(double amount) {
        // Simple interface, complex implementation hidden
        if (fraudCheck(amount)) {
            return gateway.charge(amount);
        }
        return false;
    }

    @Override
    public boolean refund(String transactionId) {
        return gateway.refund(transactionId);
    }
}

// Client code - no knowledge of implementation
public class CheckoutService {
    public void checkout(PaymentProcessor processor, double amount) {
        if (processor.processPayment(amount)) {
            System.out.println("Payment successful!");
        }
    }
}
```

---

## 3. Inheritance

**Definition:** A mechanism where a new class (child/subclass) derives properties and behaviors from an existing class (parent/superclass).

### When to Use Inheritance

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Inheritance Decision Tree                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Is there an "IS-A" relationship?                                   │
│         │                                                           │
│    ┌────┴────┐                                                      │
│   YES        NO ──────────────► Don't use inheritance               │
│    │                            Consider composition                │
│    ▼                                                                │
│  Will the child use MOST of                                         │
│  the parent's behavior?                                             │
│         │                                                           │
│    ┌────┴────┐                                                      │
│   YES        NO ──────────────► Don't use inheritance               │
│    │                            Extract an interface                │
│    ▼                                                                │
│  Is the parent designed for                                         │
│  extension?                                                         │
│         │                                                           │
│    ┌────┴────┐                                                      │
│   YES        NO ──────────────► Be careful!                         │
│    │                            May break encapsulation             │
│    ▼                                                                │
│  ✓ Inheritance is appropriate                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Python Example

```python
from abc import ABC, abstractmethod
from datetime import datetime

class Employee(ABC):
    """Base class for all employees"""

    def __init__(self, employee_id: str, name: str, base_salary: float):
        self._employee_id = employee_id
        self._name = name
        self._base_salary = base_salary
        self._hire_date = datetime.now()

    @property
    def employee_id(self) -> str:
        return self._employee_id

    @property
    def name(self) -> str:
        return self._name

    @abstractmethod
    def calculate_salary(self) -> float:
        """Each employee type calculates salary differently"""
        pass

    def get_info(self) -> str:
        """Common behavior for all employees"""
        return f"ID: {self._employee_id}, Name: {self._name}"

class FullTimeEmployee(Employee):
    """Full-time employee with benefits"""

    def __init__(self, employee_id: str, name: str, base_salary: float,
                 bonus_percentage: float = 0.1):
        super().__init__(employee_id, name, base_salary)
        self._bonus_percentage = bonus_percentage

    def calculate_salary(self) -> float:
        return self._base_salary * (1 + self._bonus_percentage)

class ContractEmployee(Employee):
    """Contract employee paid hourly"""

    def __init__(self, employee_id: str, name: str, hourly_rate: float,
                 hours_worked: float):
        super().__init__(employee_id, name, hourly_rate)
        self._hourly_rate = hourly_rate
        self._hours_worked = hours_worked

    def calculate_salary(self) -> float:
        return self._hourly_rate * self._hours_worked

class Manager(FullTimeEmployee):
    """Manager with team management responsibilities"""

    def __init__(self, employee_id: str, name: str, base_salary: float,
                 team_size: int):
        super().__init__(employee_id, name, base_salary, bonus_percentage=0.2)
        self._team_size = team_size
        self._team_members: list[Employee] = []

    def add_team_member(self, employee: Employee):
        self._team_members.append(employee)

    def calculate_salary(self) -> float:
        # Managers get extra for team size
        base = super().calculate_salary()
        management_bonus = 1000 * self._team_size
        return base + management_bonus

# Usage
employees = [
    FullTimeEmployee("E001", "Alice", 80000),
    ContractEmployee("C001", "Bob", 50, 160),
    Manager("M001", "Charlie", 100000, team_size=5)
]

for emp in employees:
    print(f"{emp.get_info()} - Salary: ${emp.calculate_salary():,.2f}")
```

**Java:**
```java
public abstract class Employee {
    protected String employeeId;
    protected String name;
    protected double baseSalary;
    protected LocalDate hireDate;

    public Employee(String employeeId, String name, double baseSalary) {
        this.employeeId = employeeId;
        this.name = name;
        this.baseSalary = baseSalary;
        this.hireDate = LocalDate.now();
    }

    public abstract double calculateSalary();

    public String getInfo() {
        return String.format("ID: %s, Name: %s", employeeId, name);
    }

    // Getters
    public String getEmployeeId() { return employeeId; }
    public String getName() { return name; }
}

public class FullTimeEmployee extends Employee {
    private double bonusPercentage;

    public FullTimeEmployee(String id, String name, double salary) {
        this(id, name, salary, 0.1);
    }

    public FullTimeEmployee(String id, String name, double salary, double bonus) {
        super(id, name, salary);
        this.bonusPercentage = bonus;
    }

    @Override
    public double calculateSalary() {
        return baseSalary * (1 + bonusPercentage);
    }
}

public class ContractEmployee extends Employee {
    private double hoursWorked;

    public ContractEmployee(String id, String name, double hourlyRate, double hours) {
        super(id, name, hourlyRate);
        this.hoursWorked = hours;
    }

    @Override
    public double calculateSalary() {
        return baseSalary * hoursWorked;  // baseSalary is hourly rate here
    }
}

public class Manager extends FullTimeEmployee {
    private int teamSize;
    private List<Employee> teamMembers;

    public Manager(String id, String name, double salary, int teamSize) {
        super(id, name, salary, 0.2);  // Higher bonus for managers
        this.teamSize = teamSize;
        this.teamMembers = new ArrayList<>();
    }

    public void addTeamMember(Employee employee) {
        teamMembers.add(employee);
    }

    @Override
    public double calculateSalary() {
        double base = super.calculateSalary();
        double managementBonus = 1000 * teamSize;
        return base + managementBonus;
    }
}
```

---

## 4. Polymorphism

**Definition:** The ability of objects of different classes to respond to the same method call in different ways.

### Types of Polymorphism

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Types of Polymorphism                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────┐    ┌─────────────────────────┐         │
│  │   COMPILE-TIME          │    │   RUNTIME               │         │
│  │   (Static)              │    │   (Dynamic)             │         │
│  ├─────────────────────────┤    ├─────────────────────────┤         │
│  │                         │    │                         │         │
│  │  Method Overloading     │    │  Method Overriding      │         │
│  │  ─────────────────      │    │  ──────────────────     │         │
│  │  Same method name,      │    │  Child class provides   │         │
│  │  different parameters   │    │  different implementa-  │         │
│  │                         │    │  tion of parent method  │         │
│  │  Example:               │    │                         │         │
│  │  add(int, int)          │    │  Example:               │         │
│  │  add(double, double)    │    │  Animal.speak()         │         │
│  │  add(String, String)    │    │  Dog.speak() → "Bark"   │         │
│  │                         │    │  Cat.speak() → "Meow"   │         │
│  └─────────────────────────┘    └─────────────────────────┘         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Python Example - Runtime Polymorphism

```python
from abc import ABC, abstractmethod
from typing import List

class Shape(ABC):
    """Abstract base class for all shapes"""

    @abstractmethod
    def area(self) -> float:
        pass

    @abstractmethod
    def perimeter(self) -> float:
        pass

    @abstractmethod
    def draw(self) -> str:
        pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height

    def area(self) -> float:
        return self._width * self._height

    def perimeter(self) -> float:
        return 2 * (self._width + self._height)

    def draw(self) -> str:
        return f"Drawing rectangle {self._width}x{self._height}"

class Circle(Shape):
    def __init__(self, radius: float):
        self._radius = radius

    def area(self) -> float:
        import math
        return math.pi * self._radius ** 2

    def perimeter(self) -> float:
        import math
        return 2 * math.pi * self._radius

    def draw(self) -> str:
        return f"Drawing circle with radius {self._radius}"

class Triangle(Shape):
    def __init__(self, a: float, b: float, c: float):
        self._a = a
        self._b = b
        self._c = c

    def area(self) -> float:
        # Heron's formula
        s = self.perimeter() / 2
        return (s * (s-self._a) * (s-self._b) * (s-self._c)) ** 0.5

    def perimeter(self) -> float:
        return self._a + self._b + self._c

    def draw(self) -> str:
        return f"Drawing triangle with sides {self._a}, {self._b}, {self._c}"

# Polymorphism in action - same interface, different behavior
class Canvas:
    def __init__(self):
        self._shapes: List[Shape] = []

    def add_shape(self, shape: Shape):
        self._shapes.append(shape)

    def total_area(self) -> float:
        # Works with ANY shape - polymorphism!
        return sum(shape.area() for shape in self._shapes)

    def render_all(self):
        for shape in self._shapes:
            print(shape.draw())  # Each shape draws differently

# Usage
canvas = Canvas()
canvas.add_shape(Rectangle(10, 5))
canvas.add_shape(Circle(7))
canvas.add_shape(Triangle(3, 4, 5))

print(f"Total area: {canvas.total_area():.2f}")
canvas.render_all()
```

**Java:**
```java
public abstract class Shape {
    public abstract double area();
    public abstract double perimeter();
    public abstract String draw();
}

public class Rectangle extends Shape {
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

    @Override
    public double perimeter() {
        return 2 * (width + height);
    }

    @Override
    public String draw() {
        return String.format("Drawing rectangle %.1fx%.1f", width, height);
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }

    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }

    @Override
    public String draw() {
        return String.format("Drawing circle with radius %.1f", radius);
    }
}

// Polymorphism in action
public class Canvas {
    private List<Shape> shapes = new ArrayList<>();

    public void addShape(Shape shape) {
        shapes.add(shape);
    }

    public double totalArea() {
        return shapes.stream()
                     .mapToDouble(Shape::area)
                     .sum();
    }

    public void renderAll() {
        for (Shape shape : shapes) {
            System.out.println(shape.draw());  // Polymorphic call
        }
    }
}
```

---

## Composition vs Inheritance

**Important Interview Topic:** When should you use composition instead of inheritance?

```
┌─────────────────────────────────────────────────────────────────────┐
│              Composition vs Inheritance                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  INHERITANCE ("IS-A")              COMPOSITION ("HAS-A")            │
│  ─────────────────────             ─────────────────────            │
│                                                                     │
│  class Dog extends Animal          class Car:                       │
│      # Dog IS AN Animal                engine: Engine               │
│                                        wheels: List[Wheel]          │
│                                        # Car HAS AN Engine          │
│                                                                     │
│  USE WHEN:                         USE WHEN:                        │
│  • True "is-a" relationship        • "Has-a" relationship           │
│  • Child uses ALL parent           • Want runtime flexibility       │
│    behavior                        • Need to combine behaviors      │
│  • Hierarchy is stable             • Avoid tight coupling           │
│                                                                     │
│  PROBLEMS:                         BENEFITS:                        │
│  • Tight coupling                  • Loose coupling                 │
│  • Fragile base class             • Easy to change                 │
│  • Can't change at runtime        • More flexible                  │
│  • Diamond problem                • Easier testing                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example: Composition Over Inheritance

**Bad: Inheritance Abuse**
```python
# Bad: Using inheritance for code reuse
class Vehicle:
    def start_engine(self):
        print("Engine started")

class Boat(Vehicle):
    def sail(self):
        print("Sailing")

class Car(Vehicle):
    def drive(self):
        print("Driving")

# Problem: What about electric vehicles? Bicycles?
class Bicycle(Vehicle):  # Bicycles don't have engines!
    pass
```

**Good: Composition**
```python
from abc import ABC, abstractmethod

# Define capabilities as interfaces/components
class Engine(ABC):
    @abstractmethod
    def start(self):
        pass

class GasEngine(Engine):
    def start(self):
        print("Gas engine started")

class ElectricMotor(Engine):
    def start(self):
        print("Electric motor started")

class NoEngine(Engine):
    def start(self):
        print("No engine to start")

# Compose vehicles from components
class Vehicle:
    def __init__(self, engine: Engine):
        self._engine = engine

    def start(self):
        self._engine.start()

# Now we can create any combination
gas_car = Vehicle(GasEngine())
electric_car = Vehicle(ElectricMotor())
bicycle = Vehicle(NoEngine())

# Easy to change at runtime
gas_car._engine = ElectricMotor()  # Converted to electric!
```

---

## Interview Tips for OOP Questions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OOP Interview Checklist                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✓ Identify key entities (nouns in the problem)                     │
│  ✓ Define clear interfaces before implementations                   │
│  ✓ Use encapsulation - make fields private                          │
│  ✓ Prefer composition over deep inheritance                         │
│  ✓ Apply polymorphism for flexible designs                          │
│  ✓ Consider edge cases (null values, empty collections)             │
│  ✓ Think about extensibility (new types, new behaviors)             │
│  ✓ Discuss trade-offs of your design choices                        │
│                                                                     │
│  RED FLAGS:                                                         │
│  ✗ Public fields without getters/setters                            │
│  ✗ Very deep inheritance hierarchies (>3 levels)                    │
│  ✗ God classes that do everything                                   │
│  ✗ Tight coupling between unrelated classes                         │
│  ✗ Not using interfaces/abstract classes                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Definition | Key Benefit |
|---------|------------|-------------|
| Encapsulation | Hide internal state | Data protection |
| Abstraction | Hide complexity | Simplicity |
| Inheritance | Reuse through hierarchy | Code reuse |
| Polymorphism | Same interface, different behavior | Flexibility |

---

**Next:** [02_solid_principles.md](./02_solid_principles.md) - The five principles of good OO design.
