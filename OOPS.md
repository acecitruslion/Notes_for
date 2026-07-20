# OOP in Python — Interview Notes

---

## 1. Class
**Definition:** Blueprint/template for creating objects — defines attributes (data) and methods (behavior).
**Why:** Bundles data + behavior together; enables reusability and modeling real-world entities.
```python
class Car:
    def __init__(self, brand):
        self.brand = brand
    def drive(self):
        print(f"{self.brand} is driving")
```

## 2. Object
**Definition:** Instance of a class — actual entity created in memory with its own state.
**Why:** Class is the design; object is the real, usable thing.
```python
my_car = Car("Toyota")   # object
my_car.drive()
```

## 3. Constructor
**Definition:** Special method called automatically when an object is created; initializes attributes.
**Why:** Ensures object starts in a valid state.
```python
class Car:
    def __init__(self, brand):   # constructor
        self.brand = brand
```
Python has no overloaded constructors like C++/Java — use default args or `@classmethod`.

## 4. Destructor
**Definition:** Called when an object is about to be destroyed (garbage collected). `__del__` in Python.
**Why:** Cleanup resources (files, connections) — though Python's GC usually handles memory itself.
```python
class Car:
    def __del__(self):
        print("Car object destroyed")
```

## 5. Encapsulation
**Definition:** Wrapping data + methods together and restricting direct access to internal state.
**Why:** Protects data integrity, hides implementation details.
```python
class Account:
    def __init__(self, balance):
        self.__balance = balance   # private (name-mangled)
    def get_balance(self):
        return self.__balance
```
- `_var` → convention "protected" (internal use)
- `__var` → name-mangled "private"

## 6. Abstraction
**Definition:** Hiding implementation details, exposing only essential features/interface.
**Why:** Reduces complexity for the user of a class; focus on "what" not "how".
```python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): pass
```
**Encapsulation vs Abstraction:** Encapsulation = *hiding data* (implementation via access control). Abstraction = *hiding complexity* (design level, what to expose).

## 7. Inheritance
**Definition:** A class (child) acquires properties/methods of another class (parent).
**Why:** Code reuse, models "is-a" relationship.
```python
class Animal:
    def speak(self): print("...")
class Dog(Animal):
    def speak(self): print("Bark")
```
Types: single, multiple, multilevel, hierarchical, hybrid.

## 8. Polymorphism
**Definition:** "Many forms" — same interface/method behaves differently based on object/context.
**Why:** Flexibility, extensibility, uniform interface for different types.
```python
for animal in [Dog(), Cat()]:
    animal.speak()   # different behavior, same call
```

## 9. Runtime Polymorphism
**Definition:** Method call resolved at runtime based on actual object type (dynamic binding). Achieved via **method overriding**.
**Why:** Enables extensible code without changing calling logic.
```python
class Animal:
    def speak(self): print("Some sound")
class Dog(Animal):
    def speak(self): print("Bark")   # overridden, resolved at runtime
```

## 10. Compile Time Polymorphism
**Definition:** Resolved at compile time — traditionally via method **overloading**/operator overloading.
**Why:** In C++/Java, choose method version based on signature before execution.
**Python note:** Python doesn't support true compile-time overloading (no static typing/compile step). It's simulated using default args, `*args`, or `functools.singledispatch`.
```python
def add(a, b=0, c=0):
    return a + b + c
```

## 11. Overloading
**Definition:** Multiple methods with same name but different parameters (number/type).
**Why:** Same operation, different input signatures — improves readability.
```python
# Python simulation via default args
class Calc:
    def add(self, a, b=0, c=0):
        return a + b + c
```
Python doesn't support true overloading — last defined method overwrites earlier ones.

## 12. Overriding
**Definition:** Child class redefines a method already defined in parent class, same name & signature.
**Why:** Customize/extend inherited behavior.
```python
class Animal:
    def speak(self): print("Sound")
class Dog(Animal):
    def speak(self): print("Bark")   # overriding
```

### Overloading vs Overriding
| Overloading | Overriding |
|---|---|
| Same class, same name, different params | Parent-child, same name & signature |
| Compile-time (not truly in Python) | Runtime |
| Achieves flexibility of input | Achieves flexibility of behavior |

## 13. Virtual Functions (C++ concept)
**Definition:** Function in base class marked `virtual` so derived class's version is called via base pointer/reference (enables runtime polymorphism).
**Python equivalent:** All methods are "virtual" by default in Python — any overridden method is automatically called via the actual object type (dynamic dispatch).
```python
class Base:
    def show(self): print("Base")
class Derived(Base):
    def show(self): print("Derived")
obj: Base = Derived()
obj.show()  # prints "Derived" — Python always does this
```

## 14. Pure Virtual Functions
**Definition:** Function declared in base with no implementation; forces derived classes to implement it. Makes class abstract.
**Python equivalent:** `@abstractmethod` inside an `ABC` class.
```python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): pass
```

## 15. Abstract Class
**Definition:** Class that cannot be instantiated; may have some implemented methods and some abstract (unimplemented) methods.
**Why:** Defines a common template/contract while allowing shared code.
```python
class Shape(ABC):
    @abstractmethod
    def area(self): pass
    def describe(self):
        print("I am a shape")   # concrete method allowed
```

## 16. Interface
**Definition:** A contract with only method declarations, no implementation, no state. Python has no `interface` keyword — simulated with ABC where **all** methods are abstract.
**Why:** Enforces that implementing classes provide certain behavior; supports multiple "interface" inheritance.
```python
class Flyable(ABC):
    @abstractmethod
    def fly(self): pass
class Swimmable(ABC):
    @abstractmethod
    def swim(self): pass
class Duck(Flyable, Swimmable):
    def fly(self): print("Flying")
    def swim(self): print("Swimming")
```

### Abstract Class vs Interface
| Abstract Class | Interface |
|---|---|
| Can have concrete + abstract methods | Only abstract methods (no implementation) |
| Can have state/instance variables | No state (pure contract) |
| Single inheritance emphasis | Designed for multiple inheritance of behavior contracts |
| Python: `ABC` with some `@abstractmethod` | Python: `ABC` with **all** methods abstract |

## 17. Static
**Definition:** Belongs to the class, not any instance — shared across all objects.
**Why:** Utility methods/data that don't need object state.
```python
class MathUtil:
    pi = 3.14159   # static/class variable

    @staticmethod
    def add(a, b):
        return a + b

    @classmethod
    def get_pi(cls):
        return cls.pi
```
- `@staticmethod` → no access to `self`/`cls`
- `@classmethod` → access to class (`cls`), not instance

## 18. Final
**Definition:** Prevents a class from being subclassed or a method from being overridden.
**Python note:** No native `final` keyword; use `typing.final` (type-checker enforced, not runtime) or convention.
```python
from typing import final
@final
class Config:
    pass

class Base:
    @final
    def core_logic(self): pass
```

## 19. Association
**Definition:** General relationship between two independent classes — objects know about each other but have independent lifecycles, no ownership.
**Why:** Models "uses-a" relationship.
```python
class Teacher:
    pass
class Student:
    def __init__(self, teacher):
        self.teacher = teacher   # association
```

## 20. Aggregation
**Definition:** Special "has-a" association where the child can exist independently of the parent (weak ownership).
**Why:** Model whole-part relationships without strict lifecycle binding.
```python
class Engine:
    pass
class Car:
    def __init__(self, engine: Engine):
        self.engine = engine   # Engine can exist without Car
```

## 21. Composition
**Definition:** Strong "has-a" relationship — part cannot exist without the whole; child's lifecycle is tied to parent.
**Why:** Models tightly coupled ownership.
```python
class Engine:
    pass
class Car:
    def __init__(self):
        self.engine = Engine()   # created & owned by Car
```

### Aggregation vs Composition
| Aggregation | Composition |
|---|---|
| Weak ownership | Strong ownership |
| Part can exist independently | Part dies with the whole |
| e.g., Department has Teachers (teacher exists without dept) | e.g., House has Rooms (room can't exist without house) |

### Inheritance vs Composition
| Inheritance ("is-a") | Composition ("has-a") |
|---|---|
| `Dog(Animal)` — Dog IS an Animal | `Car` HAS an `Engine` |
| Tight coupling, reuses via subclassing | Loose coupling, reuses via object composition |
| Changes in parent can break child | More flexible, easier to change at runtime |
| Use when true hierarchical relationship exists | Preferred generally — "favor composition over inheritance" |

---

## Practical Examples (bank/e-commerce style — good for interviews)

**Class & Object** — Bank account system
```python
class BankAccount:
    def __init__(self, holder, balance):
        self.holder = holder
        self.balance = balance

acc1 = BankAccount("Rahul", 5000)   # object
acc2 = BankAccount("Priya", 12000)  # another object, same class
```

**Constructor** — auto-generate account number on account creation
```python
class BankAccount:
    def __init__(self, holder, balance=0):
        self.holder = holder
        self.balance = balance
        self.acc_no = id(self)   # generated automatically at creation
```

**Destructor** — close DB/file connection when object is deleted
```python
class DBConnection:
    def __del__(self):
        print("Connection closed")

conn = DBConnection()
del conn   # "Connection closed"
```

**Encapsulation** — hide balance, expose only controlled actions
```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance          # private
    def withdraw(self, amount):
        if amount <= self.__balance:
            self.__balance -= amount
        else:
            print("Insufficient funds")
    def get_balance(self):
        return self.__balance
```

**Abstraction** — payment gateway hides internal processing
```python
from abc import ABC, abstractmethod
class Payment(ABC):
    @abstractmethod
    def pay(self, amount): pass

class UpiPayment(Payment):
    def pay(self, amount):
        print(f"Paid {amount} via UPI")   # internal steps hidden from caller
```

**Inheritance** — Manager is a type of Employee
```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)
        self.team_size = team_size
```

**Polymorphism / Runtime Polymorphism** — different payment modes, same method call
```python
class CardPayment(Payment):
    def pay(self, amount): print(f"Paid {amount} by Card")
class CashPayment(Payment):
    def pay(self, amount): print(f"Paid {amount} by Cash")

for method in [UpiPayment(), CardPayment(), CashPayment()]:
    method.pay(500)   # same call, resolved at runtime per object
```

**Compile Time Polymorphism (simulated)** — cart total with optional discount
```python
class Cart:
    def total(self, price, qty=1, discount=0):
        return (price * qty) - discount
```

**Overloading (simulated via default args)**
```python
class Invoice:
    def generate(self, amount, tax=0, discount=0):
        return amount + tax - discount
```

**Overriding** — Manager customizes bonus calculation from Employee
```python
class Employee:
    def bonus(self):
        return self.salary * 0.05
class Manager(Employee):
    def bonus(self):
        return self.salary * 0.10   # overridden
```

**Virtual Functions (Python default behavior)** — notification system
```python
class Notifier:
    def send(self, msg): print("Generic notify:", msg)
class EmailNotifier(Notifier):
    def send(self, msg): print("Email sent:", msg)

n: Notifier = EmailNotifier()
n.send("Order placed")   # "Email sent: Order placed" — dynamic dispatch
```

**Pure Virtual Functions / Abstract Class** — every payment type MUST implement `pay()`
```python
class Payment(ABC):
    @abstractmethod
    def pay(self, amount): pass
    def receipt(self):
        print("Generating receipt...")   # shared concrete method
```

**Interface** — deliverable contract for any shipment provider
```python
class Shippable(ABC):
    @abstractmethod
    def ship(self, order_id): pass

class FedEx(Shippable):
    def ship(self, order_id): print(f"FedEx shipping order {order_id}")
class BlueDart(Shippable):
    def ship(self, order_id): print(f"BlueDart shipping order {order_id}")
```

**Static** — shared bank interest rate for all accounts
```python
class BankAccount:
    interest_rate = 4.5   # same for all accounts

    @staticmethod
    def calculate_interest(principal):
        return principal * BankAccount.interest_rate / 100
```

**Final** — prevent overriding core tax calculation logic
```python
from typing import final
class TaxCalculator:
    @final
    def calculate_gst(self, amount):
        return amount * 0.18
```

**Association** — Doctor and Patient reference each other, independent existence
```python
class Doctor:
    def __init__(self, name):
        self.name = name
class Patient:
    def __init__(self, name, doctor: Doctor):
        self.name = name
        self.doctor = doctor   # associated, not owned
```

**Aggregation** — Department has Employees, but employees can move departments
```python
class Employee:
    def __init__(self, name):
        self.name = name
class Department:
    def __init__(self, name, employees: list):
        self.name = name
        self.employees = employees   # passed in, exists independently
```

**Composition** — Order owns OrderItems; items don't exist without the order
```python
class OrderItem:
    def __init__(self, product, qty):
        self.product = product
        self.qty = qty
class Order:
    def __init__(self, items_data):
        self.items = [OrderItem(p, q) for p, q in items_data]   # created & owned by Order
```

---

## Quick Recap Table

| Concept | One-liner |
|---|---|
| Class | Blueprint |
| Object | Instance of class |
| Constructor | `__init__`, initializes object |
| Destructor | `__del__`, cleanup on deletion |
| Encapsulation | Hide data via access control |
| Abstraction | Hide complexity, show essentials |
| Inheritance | Reuse via parent-child |
| Polymorphism | One interface, many behaviors |
| Runtime Polymorphism | Overriding, resolved at runtime |
| Compile-time Polymorphism | Overloading (simulated in Python) |
| Overloading | Same name, different params |
| Overriding | Same signature, child redefines |
| Virtual Function | Default behavior in Python (dynamic dispatch) |
| Pure Virtual Function | `@abstractmethod` |
| Abstract Class | Mix of abstract + concrete methods |
| Interface | All methods abstract, no state |
| Static | Belongs to class, not instance |
| Final | No override/subclass (`typing.final`) |
| Association | General uses-a relationship |
| Aggregation | Weak has-a (independent lifecycle) |
| Composition | Strong has-a (dependent lifecycle) |
