# LLD Notes — Infosys L2/L3

---

## 1. OOD Principles

### SOLID

| Principle | Idea | Quick Example |
|---|---|---|
| **S** – Single Responsibility | A class should have only one reason to change | `Invoice` class should only handle invoice data; printing logic goes in a separate `InvoicePrinter` class |
| **O** – Open/Closed | Open for extension, closed for modification | Add new shape types by creating new classes implementing `Shape`, not by editing existing `AreaCalculator` |
| **L** – Liskov Substitution | Subclass should be replaceable for its parent without breaking behavior | If `Bird` has `fly()`, don't make `Penguin extends Bird` — it breaks the contract |
| **I** – Interface Segregation | Don't force a class to implement methods it doesn't need | Split a fat `Machine` interface into `Printer`, `Scanner`, `Fax` instead of one interface with all methods |
| **D** – Dependency Inversion | Depend on abstractions, not concrete classes | `NotificationService` should depend on `IMessageSender` interface, not directly on `EmailSender` |

**Quick code — SRP + OCP + DIP together:**
```python
from abc import ABC, abstractmethod

# DIP: depend on abstraction, not concrete class
class IMessageSender(ABC):
    @abstractmethod
    def send(self, message: str): ...

# OCP: add new senders without changing existing code
class EmailSender(IMessageSender):
    def send(self, message):
        print(f"Email sent: {message}")

class SMSSender(IMessageSender):
    def send(self, message):
        print(f"SMS sent: {message}")

# SRP: NotificationService only handles sending, not creating senders
class NotificationService:
    def __init__(self, sender: IMessageSender):
        self.sender = sender

    def notify(self, message):
        self.sender.send(message)

NotificationService(EmailSender()).notify("Hello!")   # Email sent: Hello!
NotificationService(SMSSender()).notify("Hello!")      # SMS sent: Hello!
```

### DRY (Don't Repeat Yourself)
Avoid duplicating logic — extract common code into reusable methods/classes.
*Example:* Instead of writing validation logic in 3 places, create one `Validator` utility class.

### KISS (Keep It Simple, Stupid)
Prefer the simplest solution that works — avoid over-engineering.
*Example:* Don't build a plugin architecture for a script that runs once a day.

---

## 2. OOP Relationships (Association, Aggregation, Composition)

| Relationship | Strength | Lifecycle Dependency | Real-life Example |
|---|---|---|---|
| **Association** | Weakest — just "uses" or "knows about" | Independent objects | A `Teacher` and a `Student` — they interact, but neither owns the other |
| **Aggregation** | "Has-a" (whole-part, but loosely coupled) | Part can exist without the whole | A `Department` has `Professors` — if the department shuts down, professors still exist |
| **Composition** | Strongest "has-a" (whole-part, tightly coupled) | Part cannot exist without the whole | A `House` has `Rooms` — if the house is destroyed, the rooms cease to exist |

**One-liner to remember:**
- Association → related objects
- Aggregation → owns, but parts survive independently (shared/detachable)
- Composition → owns, and parts die with the parent (exclusive ownership)

**Quick code:**
```python
# Association — Teacher just "knows" a Student, no ownership
class Student:
    def __init__(self, name):
        self.name = name

class Teacher:
    def teach(self, student: Student):     # relationship exists only during the call
        print(f"Teaching {student.name}")

# Aggregation — Department "has" Professors, but they're passed in (can exist independently)
class Professor:
    def __init__(self, name):
        self.name = name

class Department:
    def __init__(self, professors: list):
        self.professors = professors        # created outside, just referenced here

profs = [Professor("Dr. Rao")]
dept = Department(profs)                    # if dept is deleted, Dr. Rao still exists

# Composition — House "owns" Rooms; rooms are created & destroyed with the house
class Room:
    def __init__(self, name):
        self.name = name

class House:
    def __init__(self):
        self.rooms = [Room("Bedroom"), Room("Kitchen")]   # created inside, tightly bound

house = House()                              # if house is deleted, rooms go with it
```

---

## 3. Design Patterns

| Pattern | Type | Definition | Use Case |
|---|---|---|---|
| **Singleton** | Creational | Ensures only one instance of a class exists, with a global access point | Database connection pool, Logger, Configuration manager |
| **Factory** | Creational | Creates objects without exposing the instantiation logic; a common interface decides which class to instantiate | Creating different `PaymentMethod` objects (`CreditCard`, `UPI`, `NetBanking`) based on input |
| **Strategy** | Behavioral | Defines a family of algorithms, encapsulates each, and makes them interchangeable at runtime | Different sorting strategies, or different discount calculation strategies for an e-commerce app |
| **Observer** | Behavioral | One-to-many dependency — when one object (subject) changes state, all its dependents (observers) are notified | YouTube channel subscribers getting notified on new video upload; Event listeners |

*Interview tip:* For each, be ready to say **definition + one real example + why it's useful** in 2–3 lines. No need to code it fully unless asked.

**Singleton:**
```python
class Logger:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

l1 = Logger()
l2 = Logger()
print(l1 is l2)   # True — same instance
```
*Use when exactly one object should exist (Logger, Config, Database Connection).*

**Factory:**
```python
class CreditCardPayment:
    def pay(self, amount): print(f"Paid {amount} via Credit Card")

class UPIPayment:
    def pay(self, amount): print(f"Paid {amount} via UPI")

class PaymentFactory:
    @staticmethod
    def get_payment_method(method):
        if method == "credit_card":
            return CreditCardPayment()
        elif method == "upi":
            return UPIPayment()

payment = PaymentFactory.get_payment_method("upi")
payment.pay(500)   # Paid 500 via UPI
```
*Use when object creation depends on runtime input.*

**Strategy:**
```python
class FlatDiscount:
    def apply(self, price): return price - 50

class PercentDiscount:
    def apply(self, price): return price * 0.9

class Cart:
    def __init__(self, strategy):
        self.strategy = strategy            # interchangeable at runtime

    def checkout(self, price):
        return self.strategy.apply(price)

print(Cart(FlatDiscount()).checkout(500))      # 450
print(Cart(PercentDiscount()).checkout(500))   # 450.0
```
*Use when multiple algorithms can solve the same problem.*

**Observer:**
```python
class Channel:
    def __init__(self):
        self.subscribers = []

    def subscribe(self, sub):
        self.subscribers.append(sub)

    def notify_all(self, video_title):
        for sub in self.subscribers:
            sub.update(video_title)

class Subscriber:
    def __init__(self, name):
        self.name = name

    def update(self, video_title):
        print(f"{self.name} notified: New video '{video_title}' uploaded")

channel = Channel()
channel.subscribe(Subscriber("Alice"))
channel.subscribe(Subscriber("Bob"))
channel.notify_all("LLD Basics")
```
*Use when one object should notify many others after a state change.*

---

## 4. UML Diagrams

| Diagram | What it Shows |
|---|---|
| **Class Diagram** | Static structure — classes, their attributes, methods, and relationships (association/aggregation/composition/inheritance) between them |
| **Sequence Diagram** | Dynamic behavior — the order of messages/method calls exchanged between objects over time, for a specific scenario |
| **Use Case Diagram** | Functional requirements — actors (users/systems) and the use cases (actions) they can perform with the system |

---

## 5. Basic Design Questions

### A. Library Management System

**Classes:**
- `Book` — attributes: `title`, `author`, `ISBN`, `isAvailable`
- `Member` — attributes: `memberId`, `name`, `booksIssued`
- `Library` — attributes: `books`, `members`; methods: `addBook()`, `removeBook()`, `registerMember()`
- `Librarian` — methods: `issueBook()`, `returnBook()`
- `BookIssue` (transaction record) — attributes: `book`, `member`, `issueDate`, `dueDate`

**Relationships:**
- `Library` **aggregates** `Book` and `Member` (library has books/members, but they can conceptually exist without it)
- `Library` **composition** with `BookIssue` records (issue records belong exclusively to the library)
- `Librarian` **association** with `Book`/`Member` (interacts with them to issue/return)

**Responsibilities:**
- `Book`: hold book data, track availability
- `Member`: track which books are currently issued
- `Librarian`: perform issue/return operations, enforce rules (due date, fine)
- `Library`: overall catalog and member management

---

### B. Parking Lot

**Classes:**
- `ParkingLot` — attributes: `floors`, `entryGates`; methods: `parkVehicle()`, `unparkVehicle()`
- `ParkingFloor` — attributes: `floorNumber`, `spots`
- `ParkingSpot` — attributes: `spotId`, `spotType`, `isOccupied`
- `Vehicle` (base) → `Car`, `Bike`, `Truck` (inheritance) — attributes: `licensePlate`, `type`
- `Ticket` — attributes: `entryTime`, `spot`, `vehicle`
- `Payment` — methods: `calculateFee()`, `processPayment()`

**Relationships:**
- `ParkingLot` **composition** with `ParkingFloor` (floors don't exist without the lot)
- `ParkingFloor` **composition** with `ParkingSpot`
- `Vehicle` **inheritance** → `Car`, `Bike`, `Truck`
- `Ticket` **association** with `Vehicle` and `ParkingSpot` (links them temporarily)

**Responsibilities:**
- `ParkingSpot`: track occupancy, spot type compatibility
- `Vehicle` subclasses: define vehicle-specific size/type
- `Ticket`: record entry/exit details for fee calculation
- `ParkingLot`: orchestrate spot allocation across floors

---

### C. Student Management System

**Classes:**
- `Student` — attributes: `studentId`, `name`, `grade`; methods: `enroll()`, `viewGrades()`
- `Course` — attributes: `courseId`, `courseName`, `credits`
- `Teacher` — attributes: `teacherId`, `name`; methods: `assignGrade()`
- `Department` — attributes: `departmentName`, `courses`
- `Enrollment` — attributes: `student`, `course`, `grade`, `semester`

**Relationships:**
- `Department` **aggregates** `Course` (courses can be moved/exist under restructuring)
- `Department` **aggregates** `Teacher`
- `Student` **association** with `Course` via `Enrollment` (many-to-many relationship handled through the join class)
- `Teacher` **association** with `Course` (teaches)

**Responsibilities:**
- `Student`: personal data, enrollment requests, view grades
- `Teacher`: manage assigned courses, assign grades
- `Course`: hold course metadata
- `Enrollment`: link student ↔ course ↔ grade (many-to-many resolver)
- `Department`: organize courses and teachers

---

## Interview Approach for Design Questions
1. Clarify scope/requirements briefly (e.g., "Should we handle multiple vehicle types?")
2. List core entities/classes
3. Assign attributes + key methods to each
4. Define relationships between classes (inheritance/association/aggregation/composition)
5. State responsibilities in one line per class (SRP mindset)

No need for full code — a clear verbal/whiteboard walkthrough is enough for L2/L3.
