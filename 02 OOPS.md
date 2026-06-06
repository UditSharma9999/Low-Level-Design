# Chapter 1

## Enums

Imagine you're building an e-commerce platform and you need to track the status of every order. Orders flow through various states: placed, confirmed, shipped, delivered, or cancelled. How do you represent these states in code?

You could use strings: `"PLACED"`, `"SHIPPED"`, `"DELIVERED"`. But what happens when someone types `"Shiped"` instead of `"SHIPPED"`?

The compiler won't catch it and your code will silently fail at runtime.

You could use integers: 1 for placed, 2 for shipped, 3 for delivered. But now your code is littered with magic numbers. What does if (status == 2) mean?

This is exactly the type of problem enums solve.

### 1. What is an Enum?

An enum (short for enumeration) is a special data type that defines a fixed set of named constants. Unlike strings or integers, enums are type-safe, meaning the compiler ensures you can only use values that actually exist in your defined set.

They ensure that a variable can only take **one out of a predefined set of valid options**.

<details>
<summary>Python</summary>

```python
from enum import Enum

# Simple Enum
class OrderStatus(Enum):
    PLACED = "PLACED"
    CONFIRMED = "CONFIRMED"
    SHIPPED = "SHIPPED"
    DELIVERED = "DELIVERED"
    CANCELLED = "CANCELLED"

status = OrderStatus.SHIPPED

if status == OrderStatus.SHIPPED:
    print("Your package is on the way!")
```

</details>

<details>
<summary>C++</summary>

```cpp
enum class OrderStatus {
    PLACED,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED
};

OrderStatus status = OrderStatus::SHIPPED;

if (status == OrderStatus::SHIPPED) {
    cout << "Your package is on the way!" << endl;
}
```

</details>

### Enums with Properties and Methods

In many languages, each enum value can hold additional data and even define behavior. This makes them surprisingly powerful for modeling domain concepts.

<details>
<summary>Python</summary>

```python
from enum import Enum

class Coin(Enum):
    PENNY = 1
    NICKEL = 5
    DIME = 10
    QUARTER = 25

    def __init__(self, value):
        self.coin_value = value

    def get_value(self):
        return self.coin_value

total = Coin.DIME.get_value() + Coin.QUARTER.get_value()  # 35
```

</details>

<details>
<summary>C++</summary>

```cpp
enum class Coin {
    PENNY,
    NICKEL,
    DIME,
    QUARTER
};

// C++ enums can't hold fields, so use a helper function
int getCoinValue(Coin coin) {
    switch (coin) {
        case Coin::PENNY:   return 1;
        case Coin::NICKEL:  return 5;
        case Coin::DIME:    return 10;
        case Coin::QUARTER: return 25;
        default:            return 0;
    }
}

int total = getCoinValue(Coin::DIME) + getCoinValue(Coin::QUARTER); // 35
```

</details>

### Practical Example: Order Processing System

Let's build a small order processing system that uses two enums: OrderStatus (which we've already seen) and PaymentMethod. Together, they demonstrate how enums bring structure and safety to a real domain model.

The Order class tracks an order's status, payment method, and total amount. It provides methods to advance the status through its lifecycle, cancel the order, and display order information.

The key insight is that enums control the valid transitions: an order can only move forward through the status chain (PLACED to CONFIRMED to SHIPPED to DELIVERED), and cancellation is only allowed before shipping.

<details>
<summary>Python</summary>

```python
from enum import Enum

class OrderStatus(Enum):
    PLACED = "PLACED"
    CONFIRMED = "CONFIRMED"
    SHIPPED = "SHIPPED"
    DELIVERED = "DELIVERED"
    CANCELLED = "CANCELLED"

class PaymentMethod(Enum):
    CREDIT_CARD = ("Credit Card", 2.5)
    DEBIT_CARD = ("Debit Card", 1.0)
    UPI = ("UPI", 0.0)
    NET_BANKING = ("Net Banking", 1.5)

    def __init__(self, display_name: str, fee_percent: float):
        self.display_name = display_name
        self.fee_percent = fee_percent

class Order:
    _status_transitions = {
        OrderStatus.PLACED: OrderStatus.CONFIRMED,
        OrderStatus.CONFIRMED: OrderStatus.SHIPPED,
        OrderStatus.SHIPPED: OrderStatus.DELIVERED,
    }

    def __init__(self, order_id: str, payment_method: PaymentMethod, amount: float):
        self._order_id = order_id
        self._status = OrderStatus.PLACED
        self._payment_method = payment_method
        self._amount = amount

    def advance_status(self) -> bool:
        next_status = self._status_transitions.get(self._status)
        if next_status:
            self._status = next_status
            return True
        return False

    def cancel(self) -> bool:
        if self._status in (OrderStatus.PLACED, OrderStatus.CONFIRMED):
            self._status = OrderStatus.CANCELLED
            return True
        return False

    def get_total_with_fees(self) -> float:
        return self._amount + (self._amount * self._payment_method.fee_percent / 100)

    def display_info(self) -> None:
        print(f"Order {self._order_id} | Status: {self._status.value} | "
              f"Payment: {self._payment_method.display_name} | "
              f"Amount: ${self._amount:.2f} (with fees: ${self.get_total_with_fees():.2f})")


if __name__ == "__main__":
    order = Order("ORD-001", PaymentMethod.CREDIT_CARD, 99.99)
    order.display_info()

    order.advance_status()  # PLACED -> CONFIRMED
    order.advance_status()  # CONFIRMED -> SHIPPED
    order.display_info()

    print(f"Cancel after shipping: {order.cancel()}")  # False
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

enum class OrderStatus {
    PLACED, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
};

std::string orderStatusToString(OrderStatus s) {
    switch (s) {
        case OrderStatus::PLACED:    return "PLACED";
        case OrderStatus::CONFIRMED: return "CONFIRMED";
        case OrderStatus::SHIPPED:   return "SHIPPED";
        case OrderStatus::DELIVERED: return "DELIVERED";
        case OrderStatus::CANCELLED: return "CANCELLED";
        default:                     return "UNKNOWN";
    }
}

struct PaymentMethod {
    std::string displayName;
    double feePercent;

    static const PaymentMethod CREDIT_CARD;
    static const PaymentMethod DEBIT_CARD;
    static const PaymentMethod UPI;
    static const PaymentMethod NET_BANKING;
};

const PaymentMethod PaymentMethod::CREDIT_CARD{"Credit Card", 2.5};
const PaymentMethod PaymentMethod::DEBIT_CARD{"Debit Card", 1.0};
const PaymentMethod PaymentMethod::UPI{"UPI", 0.0};
const PaymentMethod PaymentMethod::NET_BANKING{"Net Banking", 1.5};

class Order {
private:
    std::string orderId;
    OrderStatus status;
    PaymentMethod paymentMethod;
    double amount;

public:
    Order(const std::string& orderId, const PaymentMethod& paymentMethod, double amount)
        : orderId(orderId), status(OrderStatus::PLACED),
          paymentMethod(paymentMethod), amount(amount) {}

    bool advanceStatus() {
        switch (status) {
            case OrderStatus::PLACED:
                status = OrderStatus::CONFIRMED; return true;
            case OrderStatus::CONFIRMED:
                status = OrderStatus::SHIPPED; return true;
            case OrderStatus::SHIPPED:
                status = OrderStatus::DELIVERED; return true;
            default:
                return false;
        }
    }

    bool cancel() {
        if (status == OrderStatus::PLACED || status == OrderStatus::CONFIRMED) {
            status = OrderStatus::CANCELLED;
            return true;
        }
        return false;
    }

    double getTotalWithFees() const {
        return amount + (amount * paymentMethod.feePercent / 100);
    }

    void displayInfo() const {
        printf("Order %s | Status: %s | Payment: %s | Amount: $%.2f (with fees: $%.2f)\n",
            orderId.c_str(), orderStatusToString(status).c_str(),
            paymentMethod.displayName.c_str(), amount, getTotalWithFees());
    }
};

int main() {
    Order order("ORD-001", PaymentMethod::CREDIT_CARD, 99.99);
    order.displayInfo();

    order.advanceStatus(); // PLACED -> CONFIRMED
    order.advanceStatus(); // CONFIRMED -> SHIPPED
    order.displayInfo();

    std::cout << "Cancel after shipping: " << (order.cancel() ? "true" : "false") << std::endl;
    return 0;
}
```

## </details>

## Chapter 2

### Encapsulation: Why Hiding Matters

Encapsulation means keeping an object's data private and letting the object control how that data is used. You interact with it through methods instead of reaching in and changing its fields directly.

**Encapsulation isn't just about validation. In multi-threaded systems, encapsulation gives you a single place to add synchronization.**

### When to Use Records / Data Classes vs Full Encapsulation

Not every class needs getters, setters, and validation. Some objects are just data bundles. Modern languages have constructs for this.

<details>
<summary>Python</summary>

```python
# Python dataclass (frozen = immutable)
from dataclasses import dataclass
from datetime import datetime

@dataclass(frozen=True)
class Ticket:
    spot_id: int
    license_plate: str
    entry_time: datetime
```

</details>

<details>
<summary>C++</summary>

```cpp
// C++ struct: data carrier (public by default)
struct Ticket {
    const int spotId;
    const std::string licensePlate;
    const TimePoint entryTime;
};
```

</details>

| Use a record/data class when...                                | Use a full class with encapsulation when...                     |
| -------------------------------------------------------------- | --------------------------------------------------------------- |
| The data is immutable (never changes after creation)           | The state changes over time (balances, occupancy)               |
| There are no invariants to enforce                             | Business rules constrain valid states                           |
| It's a value being passed around (Ticket, Receipt, Coordinate) | It manages a collection or resource (ParkingLot, Cart, Account) |
| All fields are set at construction and never modified          | Methods mutate internal state with validation                   |

---

# Chapter 3 Inheritance vs Composition

Modern practice strongly favors composition because it's more flexible and less fragile.

![text3](/assets/3.png)

## Inheritance: The "Is-A" Relationship

Inheritance lets a child class reuse the implementation of a parent class. A `SavingsAccount` IS A `BankAccount`, so it makes sense for it to inherit shared behavior like deposit() and` withdraw()`.

<details>
<summary>Python</summary>

```python
class BankAccount:
    def __init__(self, initial_balance_cents: int):
        self._balance_cents = initial_balance_cents

    def deposit(self, amount_cents: int) -> None:
        if amount_cents <= 0:
            raise ValueError("Must be positive")
        self._balance_cents += amount_cents

    def withdraw(self, amount_cents: int) -> None:
        if amount_cents <= 0:
            raise ValueError("Must be positive")
        if amount_cents > self._balance_cents:
            raise ValueError("Insufficient funds")
        self._balance_cents -= amount_cents

    def get_balance_cents(self) -> int:
        return self._balance_cents

class SavingsAccount(BankAccount):
    def __init__(self, initial_balance_cents: int, interest_rate: float):
        super().__init__(initial_balance_cents)
        self._interest_rate = interest_rate

    def apply_interest(self) -> None:
        interest = int(self.get_balance_cents() * self._interest_rate)
        self.deposit(interest)
```

</details>

<details>
<summary>C++</summary>

```cpp
class BankAccount {
private:
    long balanceCents;

public:
    BankAccount(long initialBalanceCents) : balanceCents(initialBalanceCents) {}

    void deposit(long amountCents) {
        if (amountCents <= 0) throw std::invalid_argument("Must be positive");
        balanceCents += amountCents;
    }

    void withdraw(long amountCents) {
        if (amountCents <= 0) throw std::invalid_argument("Must be positive");
        if (amountCents > balanceCents) throw std::invalid_argument("Insufficient funds");
        balanceCents -= amountCents;
    }

    long getBalanceCents() const { return balanceCents; }
};

class SavingsAccount : public BankAccount {
    double interestRate;

public:
    SavingsAccount(long initialBalanceCents, double interestRate)
        : BankAccount(initialBalanceCents), interestRate(interestRate) {}

    void applyInterest() {
        long interest = (long)(getBalanceCents() * interestRate);
        deposit(interest);
    }
};
```

</details>

This works well because `SavingsAccount` and `CheckingAccount` genuinely are bank accounts. The shared `deposit()` and `withdraw()` logic belongs in the parent.

## Composition: The "Has-A" Relationship

Composition means a class contains an instance of another class instead of extending it. A `Car` HAS AN `Engine` -- it doesn't inherit from `Engine`.

<details>
<summary>Python</summary>

```python
class Engine:
    def __init__(self, horsepower: int):
        self._horsepower = horsepower

    def start(self) -> None:
        print("Engine started")

class Car:
    def __init__(self, engine: Engine):
        self._engine = engine  # Car HAS an Engine

    def start(self) -> None:
        self._engine.start()
        print("Car is ready to drive")
```

</details>

<details>
<summary>C++</summary>

```cpp
class Engine {
    int horsepower;

public:
    Engine(int horsepower) : horsepower(horsepower) {}
    void start() { std::cout << "Engine started\n"; }
    int getHorsepower() const { return horsepower; }
};

class Car {
    Engine engine;  // Car HAS an Engine

public:
    Car(Engine engine) : engine(std::move(engine)) {}

    void start() {
        engine.start();
        std::cout << "Car is ready to drive\n";
    }
};
```

</details>

Inheritance creates tight coupling. When you change a parent class, you risk breaking every child that extends it.

## The Diamond Problem

When a class inherits from two parents that share a common ancestor, which version of the shared method does it get? This is the diamond problem.

```text
      Animal
      /    \
   Flyer  Swimmer
      \    /
     FlyingFish
```

C++ allows multiple inheritance, which makes this a real issue:

```cpp
class Animal {
public:
    virtual void eat() { std::cout << "Animal eating\n"; }
};

class Flyer : public Animal {
public:
    void eat() override { std::cout << "Flyer eating\n"; }
};

class Swimmer : public Animal {
public:
    void eat() override { std::cout << "Swimmer eating\n"; }
};

// Which eat() does FlyingFish get? Ambiguous!
class FlyingFish : public Flyer, public Swimmer {};
// C++ fix: use virtual inheritance or explicitly disambiguate
```

Python resolves it with Method Resolution Order (MRO), which follows a deterministic left-to-right, depth-first linearization. Java avoids the problem entirely by not allowing multiple class inheritance -- you can only implement multiple interfaces, not extend multiple classes.

> Here's the most common mistake candidates make: using inheritance to reuse code even when there's no "is-a" relationship.

```java
// BAD: NotificationService inherits from EmailSender
// A notification service IS NOT an email sender
class EmailSender {
    public void sendEmail(String to, String body) { /* SMTP logic */ }
}

class NotificationService extends EmailSender {
    public void notifyUser(String userId, String message) {
        String email = lookupEmail(userId);
        sendEmail(email, message);  // "Convenient" reuse via inheritance
    }
}
// Problem: NotificationService is permanently locked to email.
// What if you need SMS? Push notifications? You can't swap them in.
```

<details>
<summary>Python</summary>

```python
# GOOD: Composition with a protocol/ABC
from abc import ABC, abstractmethod

class MessageSender(ABC):
    @abstractmethod
    def send(self, destination: str, body: str) -> None: ...

class EmailSender(MessageSender):
    def send(self, destination: str, body: str) -> None:
        pass  # SMTP logic

class SmsSender(MessageSender):
    def send(self, destination: str, body: str) -> None:
        pass  # SMS API logic

class NotificationService:
    def __init__(self, sender: MessageSender):
        self._sender = sender  # Composed, not inherited

    def notify_user(self, user_id: str, message: str) -> None:
        destination = self._lookup_destination(user_id)
        self._sender.send(destination, message)
```

</details>

<details>
<summary>C++</summary>

```cpp
// GOOD: Composition with an abstract base class (C++ "interface")
class MessageSender {
public:
    virtual void send(const std::string& destination, const std::string& body) = 0;
    virtual ~MessageSender() = default;
};

class EmailSender : public MessageSender {
public:
    void send(const std::string& destination, const std::string& body) override {}
};

class NotificationService {
    std::unique_ptr<MessageSender> sender;

public:
    NotificationService(std::unique_ptr<MessageSender> sender)
        : sender(std::move(sender)) {}

    void notifyUser(const std::string& userId, const std::string& message) {
        std::string dest = lookupDestination(userId);
        sender->send(dest, message);
    }
};
```

</details>

## The Practical Rule

Use inheritance when:

- The relationship is genuinely "is-a" (SavingsAccount IS A BankAccount)
- Subclasses share implementation that naturally belongs in the parent
- The hierarchy is shallow (one or two levels deep)

Use composition for everything else. Especially when:

- You want to swap behavior at runtime
- The "reuse" is just code convenience, not a real type relationship
- You find yourself building hierarchies three or more levels deep

---

# Chapter 4 Polymorphism (many forms) and Interfaces

![text4](/assets/4.png)

Polymorphism allows the same method name or interface to exhibit different behaviors depending on the object that is invoking it.

## How Polymorphism Works

Polymorphism in OOP comes in two forms: compile-time (decided before the program runs) and runtime (decided while the program runs). Both allow the same method name to behave differently, but the mechanism is fundamentally different.

## 1. Compile-time Polymorphism (Method Overloading)

Compile-time polymorphism, also called method overloading, happens when you have multiple methods with the same name in the same class but with different parameter lists.

The compiler determines which version to call based on the number, types, or order of arguments at the call site. The decision is made before the program runs.

<details>
<summary>Python</summary>

```python
# Python does NOT support method overloading natively.
# If you define multiple methods with the same name, only the last one survives.
# The standard workaround is to use default arguments or *args.

class Calculator:
    def add(self, *args):
        return sum(args)
calc = Calculator()
print(calc.add(2, 3))        # 5
print(calc.add(2.5, 3.5))    # 6.0
print(calc.add(1, 2, 3))     # 6
```

</details>

<details>
<summary>C++</summary>

```cpp
class Calculator {
public:
    // Two ints
    int add(int a, int b) {return a + b;}
    // Two doubles
    double add(double a, double b) {return a + b;}
    // Three ints
    int add(int a, int b, int c) {return a + b + c;}
};
int main() {
    Calculator calc;
    std::cout << calc.add(2, 3) << std::endl;        // Calls add(int, int) -> 5
    std::cout << calc.add(2.5, 3.5) << std::endl;    // Calls add(double, double) -> 6
    std::cout << calc.add(1, 2, 3) << std::endl;     // Calls add(int, int, int) -> 6
    return 0;
}
```

</details>

## 2. Runtime Polymorphism (Method Overriding / Dynamic Dispatch)

It happens when a child class overrides a method defined in its parent class, and the decision of which version to call is made at runtime based on the actual type of the object, not the declared type of the reference.

**Example**
Suppose you’re designing a system that sends notifications. You want to support email, SMS, push notifications, etc.

![text5](/assets/5.png)

<details>
<summary>Python</summary>

```python
class Notification:
    def __init__(self, recipient: str, message: str):
        self._recipient = recipient
        self._message = message

    def send(self):
        print(f"Sending generic notification to {self._recipient}")


class EmailNotification(Notification):
    def __init__(self, recipient: str, message: str, subject: str):
        super().__init__(recipient, message)
        self._subject = subject

    def send(self):
        print(f"Sending EMAIL to {self._recipient} | Subject: {self._subject}")


class SMSNotification(Notification):
    def __init__(self, recipient: str, message: str, phone_number: str):
        super().__init__(recipient, message)
        self._phone_number = phone_number

    def send(self):
        print(f"Sending SMS to {self._phone_number} | Message: {self._message}")


class PushNotification(Notification):
    def __init__(self, recipient: str, message: str, device_token: str):
        super().__init__(recipient, message)
        self._device_token = device_token

    def send(self):
        print(f"Sending PUSH to device {self._device_token[:8]}"
              f"... | Alert: {self._message}")


if __name__ == "__main__":
    notifications = [
        EmailNotification("alice@example.com", "Your order shipped!", "Order Update"),
        SMSNotification("Bob", "Code: 482910", "+1-555-0123"),
        PushNotification("Charlie", "New message", "d8a3f4b2c1e5a9b7"),
    ]

    for n in notifications:
        n.send()
```

</details>

<details>
<summary>C++</summary>

```cpp
class Notification {
protected:
    string recipient;
    string message;

public:
    Notification(const string& recipient, const string& message)
        : recipient(recipient), message(message) {}

    virtual ~Notification() {}

    virtual void send() {
        cout << "Sending generic notification to " << recipient << endl;
    }
};

class EmailNotification : public Notification {
    string subject;

public:
    EmailNotification(const string& recipient, const string& message,
                      const string& subject)
        : Notification(recipient, message), subject(subject) {}

    void send() override {
        cout << "Sending EMAIL to " << recipient
             << " | Subject: " << subject << endl;
    }
};

class SMSNotification : public Notification {
    string phoneNumber;

public:
    SMSNotification(const string& recipient, const string& message,
                    const string& phoneNumber)
        : Notification(recipient, message), phoneNumber(phoneNumber) {}

    void send() override {
        cout << "Sending SMS to " << phoneNumber
             << " | Message: " << message << endl;
    }
};

class PushNotification : public Notification {
    string deviceToken;

public:
    PushNotification(const string& recipient, const string& message,
                     const string& deviceToken)
        : Notification(recipient, message), deviceToken(deviceToken) {}

    void send() override {
        cout << "Sending PUSH to device " << deviceToken.substr(0, 8)
             << "... | Alert: " << message << endl;
    }
};

int main() {
    vector<unique_ptr<Notification>> notifications;
    notifications.push_back(make_unique<EmailNotification>(
        "alice@example.com", "Your order shipped!", "Order Update"));
    notifications.push_back(make_unique<SMSNotification>(
        "Bob", "Code: 482910", "+1-555-0123"));
    notifications.push_back(make_unique<PushNotification>(
        "Charlie", "New message", "d8a3f4b2c1e5a9b7"));

    for (auto& n : notifications) {
        n->send();
    }
    return 0;
}
```

</details>

## 3. Polymorphism with Interfaces vs Abstract Classes

Both interfaces and abstract classes enable polymorphism. In the notification example, you could define Notification as either an abstract class or an interface. The polymorphic behavior, calling send() on a base reference and having the child's version execute, works the same either way. So when should you use which?

| Aspect               | Interface                                              | Abstract Class                                    |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------- |
| Relationship         | "can do" (capability)                                  | "is a" (family)                                   |
| Shared behavior      | None (contract only)                                   | Yes (concrete methods + fields)                   |
| Multiple inheritance | A class can implement many                             | A class can extend only one                       |
| When to use          | Unrelated classes share a capability                   | Related classes share logic                       |
| Example              | `Sendable` implemented by `Email`, `Invoice`, `Report` | `Notification` extended by `Email`, `SMS`, `Push` |

Use an interface when the implementing classes are fundamentally different but share a capability. `Email`, `Invoice`, and `Report` have nothing in common structurally, but they can all `send()`. An interface defines that contract without forcing a shared hierarchy.

Use an abstract class when the implementing classes are a family with shared logic. All notifications need the same `formatHeader()` method, the same `recipient` and `message` fields, and the same constructor pattern. An abstract class provides all of that, plus the abstract `send()` that each child implements differently.

---

# Chapter 5 Abstract Classes vs Interfaces

> Abstract classes share both a contract and implementation across subclasses. Interfaces share only a contract. If the implementations share actual code, use an abstract class. If they only share method signatures, use an interface.

![Text6](/assets/6.png)

## Abstract Class: Shared Implementation + Shared Contract

Use an abstract class when subclasses share actual behavior, not just method signatures. The abstract class provides the common code so subclasses don't duplicate it.

**Example: File System**. Both `File` and `Folder` have a name, a parent reference, and a `getPath()` method that walks up the parent chain. That's real shared code.

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class FileSystemEntry(ABC):
    def __init__(self, name: str, parent: "FileSystemEntry | None"):
        self._name = name
        self._parent = parent

    def get_name(self) -> str:
        return self._name

    # Shared implementation
    def get_path(self) -> str:
        if self._parent is None:
            return "/" + self._name
        return self._parent.get_path() + "/" + self._name

    @abstractmethod
    def get_size(self) -> int: ...

class File(FileSystemEntry):
    def __init__(self, name: str, parent: FileSystemEntry | None, size_bytes: int):
        super().__init__(name, parent)
        self._size_bytes = size_bytes

    def get_size(self) -> int:
        return self._size_bytes

class Folder(FileSystemEntry):
    def __init__(self, name: str, parent: FileSystemEntry | None):
        super().__init__(name, parent)
        self._children: list[FileSystemEntry] = []

    def add_entry(self, entry: FileSystemEntry) -> None:
        self._children.append(entry)

    def get_size(self) -> int:
        return sum(child.get_size() for child in self._children)
```

</details>

<details>
<summary>C++</summary>

```cpp
class FileSystemEntry {
protected:
    std::string name;
    FileSystemEntry* parent;

public:
    FileSystemEntry(std::string name, FileSystemEntry* parent)
        : name(std::move(name)), parent(parent) {}

    virtual ~FileSystemEntry() = default;

    std::string getName() const { return name; }

    // Shared implementation
    std::string getPath() const {
        if (!parent) return "/" + name;
        return parent->getPath() + "/" + name;
    }

    virtual long getSize() const = 0;  // Pure virtual — subclasses must implement
};

class File : public FileSystemEntry {
    long sizeBytes;
public:
    File(std::string name, FileSystemEntry* parent, long sizeBytes)
        : FileSystemEntry(std::move(name), parent), sizeBytes(sizeBytes) {}

    long getSize() const override { return sizeBytes; }
};

class Folder : public FileSystemEntry {
    std::vector<FileSystemEntry*> children;
public:
    Folder(std::string name, FileSystemEntry* parent)
        : FileSystemEntry(std::move(name), parent) {}

    void addEntry(FileSystemEntry* entry) { children.push_back(entry); }

    long getSize() const override {
        long total = 0;
        for (auto* child : children) total += child->getSize();
        return total;
    }
};
```

</details>

## Interface: Shared Contract Only

Use an interface when classes share behavior but have zero common implementation. They agree on what they do, not how they do it.

**Example: Rate Limiter**. A `TokenBucketLimiter` and a `SlidingWindowLogLimiter` both answer the question "should this request be allowed?" But their internal state and algorithms are completely different.

<details>
<summary>Python</summary>

```python
from typing import Protocol

class Limiter(Protocol):
    def allow_request(self, client_id: str) -> bool: ...

class TokenBucketLimiter:
    def __init__(self, max_tokens: int, refill_rate: int):
        self._token_counts: dict[str, int] = {}
        self._max_tokens = max_tokens
        self._refill_rate = refill_rate

    def allow_request(self, client_id: str) -> bool:
        tokens = self._token_counts.get(client_id, self._max_tokens)
        if tokens <= 0:
            return False
        self._token_counts[client_id] = tokens - 1
        return True

class SlidingWindowLogLimiter:
    def __init__(self, max_requests: int, window_millis: int):
        self._request_logs: dict[str, list[float]] = {}
        self._max_requests = max_requests
        self._window_millis = window_millis

    def allow_request(self, client_id: str) -> bool:
        import time
        now = time.time() * 1000
        log = self._request_logs.setdefault(client_id, [])
        log[:] = [ts for ts in log if ts >= now - self._window_millis]
        if len(log) >= self._max_requests:
            return False
        log.append(now)
        return True
```

</details>

<details>
<summary>C++</summary>

```cpp
// C++ has no "interface" keyword — use a pure virtual class
class Limiter {
public:
    virtual ~Limiter() = default;
    virtual bool allowRequest(const std::string& clientId) = 0;
};

class TokenBucketLimiter : public Limiter {
    std::unordered_map<std::string, long> tokenCounts;
    long maxTokens;
    long refillRate;

public:
    TokenBucketLimiter(long maxTokens, long refillRate)
        : maxTokens(maxTokens), refillRate(refillRate) {}

    bool allowRequest(const std::string& clientId) override {
        long tokens = tokenCounts.count(clientId) ? tokenCounts[clientId] : maxTokens;
        if (tokens <= 0) return false;
        tokenCounts[clientId] = tokens - 1;
        return true;
    }
};

class SlidingWindowLogLimiter : public Limiter {
    std::unordered_map<std::string, std::vector<long>> requestLogs;
    int maxRequests;
    long windowMillis;

public:
    SlidingWindowLogLimiter(int maxRequests, long windowMillis)
        : maxRequests(maxRequests), windowMillis(windowMillis) {}

    bool allowRequest(const std::string& clientId) override {
        auto now = std::chrono::system_clock::now().time_since_epoch().count();
        auto& log = requestLogs[clientId];
        log.erase(std::remove_if(log.begin(), log.end(),
            [&](long ts) { return ts < now - windowMillis; }), log.end());
        if ((int)log.size() >= maxRequests) return false;
        log.push_back(now);
        return true;
    }
};
```

</details>

## Language-Specific Details

### C++: Pure Virtual Classes as Interfaces

C++ has no `interface` keyword. The convention is a class with all pure virtual methods and a virtual destructor:

```cpp
// This IS an interface in C++ — all methods are pure virtual
class Limiter {
public:
    virtual ~Limiter() = default;
    virtual bool allowRequest(const std::string& clientId) = 0;
};

// C++ allows multiple inheritance, so a class can "implement" multiple interfaces
class AuditableLimiter : public Limiter, public Auditable {
    // Must implement all pure virtual methods from both
};
```

C++ allows multiple inheritance of classes with state, which can lead to the diamond problem. Prefer pure virtual classes (interfaces) for multiple inheritance and single inheritance for abstract classes with shared state.

### Python: ABC and Protocol

Python offers two approaches:

ABC (Abstract Base Class) -- explicit inheritance, checked at instantiation:

```python
from abc import ABC, abstractmethod

class FileSystemEntry(ABC):
    @abstractmethod
    def get_size(self) -> int: ...

# TypeError at instantiation if get_size() is not implemented
```

**Protocol** -- structural typing, no inheritance required:

```py
from typing import Protocol

class Limiter(Protocol):
    def allow_request(self, client_id: str) -> bool: ...

# Any class with allow_request(str) -> bool satisfies Limiter
# No need to explicitly inherit from it
```

Python's duck typing means you often don't need formal interfaces at all. If it has an `allow_request` method, it works. `Protocol` adds static type checking on top of that flexibility.

> Interview tip: In a Python LLD interview, use ABC when you want to enforce that subclasses implement certain methods (the class will refuse to instantiate otherwise). Use Protocol when you want type-safe duck typing without forcing an inheritance relationship.

---

## Chapter 6 C++ and Python Toolkit for LLD

### The Translation Table

| Java Feature                   | C++ Equivalent                                | Python Equivalent                                                        |
| ------------------------------ | --------------------------------------------- | ------------------------------------------------------------------------ |
| interface                      | Class with pure virtual functions (`= 0`)     | ABC with `@abstractmethod` or `Protocol`                                 |
| abstract class                 | Class with at least one pure virtual function | ABC with mix of abstract and concrete methods                            |
| implements / extends           | `: public Base` (inheritance)                 | `class Child(Base):`                                                     |
| Multiple interfaces            | Multiple inheritance from abstract classes    | Multiple inheritance from ABCs                                           |
| private field                  | `private:` section                            | `_name` convention (no enforcement)                                      |
| protected field                | `protected:` section                          | `_name` convention (same as private)                                     |
| public method                  | `public:` section                             | Default (no prefix)                                                      |
| enum with behavior             | `enum class` + separate map/function          | `Enum` with methods and `__init__`                                       |
| Generics (`<T>`)               | Templates (`template<typename T>`)            | Type hints (`list[T]`, `Generic[T]`)                                     |
| final class                    | `final` keyword (C++11)                       | No built-in equivalent                                                   |
| @Override                      | `override` keyword (C++11)                    | No keyword; just redefine the method                                     |
| Collections.unmodifiableList() | Return `const` reference                      | Return `tuple()` or `frozenset()`                                        |
| HashMap                        | `std::unordered_map`                          | `dict`                                                                   |
| TreeMap                        | `std::map` (ordered by default)               | `dict` (insertion-ordered; use `sortedcontainers.SortedDict` for sorted) |
| ArrayList                      | `std::vector`                                 | `list`                                                                   |
| HashSet                        | `std::unordered_set`                          | `set`                                                                    |
| Queue / PriorityQueue          | `std::queue` / `std::priority_queue`          | `collections.deque` / `heapq`                                            |

## C++ Equivalents in Detail

### Interfaces via Pure Virtual Functions

C++ doesn't have an `interface` keyword. Instead, you create a class where every method is pure virtual:

```cpp
// This IS an interface -- all methods are pure virtual
class PaymentProcessor {
public:
    virtual bool charge(Money amount, PaymentMethod method) = 0;
    virtual bool refund(const std::string& transactionId) = 0;
    virtual ~PaymentProcessor() = default;  // Always add virtual destructor
};

class StripeProcessor : public PaymentProcessor {
public:
    bool charge(Money amount, PaymentMethod method) override {
        // Stripe-specific logic
        return true;
    }

    bool refund(const std::string& transactionId) override {
        // Stripe-specific logic
        return true;
    }
};
```

The `= 0` makes a method pure virtual, meaning any subclass must implement it. The `override` keyword (C++11) tells the compiler to verify you're actually overriding a virtual method -- use it every time.

Critical: Always declare a virtual destructor in your base class. **Without it, deleting a derived object through a base pointer causes undefined behavior**.

### Abstract Classes

Mix pure virtual methods (subclass must implement) with concrete methods (shared behavior):

```cpp
class Vehicle {
private:
    std::string licensePlate;
    VehicleType type;

public:
    Vehicle(std::string plate, VehicleType type)
        : licensePlate(std::move(plate)), type(type) {}

    virtual ~Vehicle() = default;

    std::string getLicensePlate() const { return licensePlate; }
    VehicleType getType() const { return type; }

    // Subclasses must implement
    virtual int getRequiredSpots() const = 0;
};
```

### Smart Pointers for Ownership

In Java, garbage collection handles memory. In C++, you need smart pointers:

```cpp
// Unique ownership -- one owner, automatically deleted
std::unique_ptr<Vehicle> vehicle = std::make_unique<Car>("ABC-123");

// Shared ownership -- reference counted
std::shared_ptr<PaymentProcessor> processor = std::make_shared<StripeProcessor>();

// Never use raw new/delete in LLD interviews
```

In an interview, default to `std::unique_ptr` unless multiple objects need to share ownership.

## Python Equivalents in Detail

### Interfaces via ABC and Protocol

Python gives you two approaches. Use ABC when you want enforced contracts:

```Python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount: Money, method: PaymentMethod) -> bool:
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        pass

class StripeProcessor(PaymentProcessor):
    def charge(self, amount: Money, method: PaymentMethod) -> bool:
        # Stripe-specific logic
        return True

    def refund(self, transaction_id: str) -> bool:
        # Stripe-specific logic
        return True
```

If you forget to implement `refund()`, Python raises `TypeError` when you try to instantiate `StripeProcessor`. The check happens at instantiation time, not at class definition time -- a key difference from Java.

`Protocol` (from `typing`) provides structural typing -- "if it has these methods, it qualifies":

```py
from typing import Protocol

class PaymentProcessor(Protocol):
    def charge(self, amount: Money, method: PaymentMethod) -> bool: ...
    def refund(self, transaction_id: str) -> bool: ...

# StripeProcessor doesn't need to inherit from PaymentProcessor.
# It just needs to have the right methods.
class StripeProcessor:
    def charge(self, amount: Money, method: PaymentMethod) -> bool:
        return True

    def refund(self, transaction_id: str) -> bool:
        return True
```

For interviews, `ABC` is clearer and more explicit. Use `Protocol` when you want duck typing with type checker support.

### Enums with Behavior

Python enums can have methods and custom `__init__`:

```py
from enum import Enum

class VehicleType(Enum):
    MOTORCYCLE = 1
    CAR = 2
    BUS = 5

    def __init__(self, required_spots: int):
        self._required_spots = required_spots

    @property
    def required_spots(self) -> int:
        return self._required_spots

# Usage
spots = VehicleType.CAR.required_spots  # 2
```

### Dataclasses for Value Objects

When a class is mostly data with minimal behavior, dataclass eliminates boilerplate:

```Python
from dataclasses import dataclass

@dataclass(frozen=True)  # frozen=True makes it immutable
class Money:
    amount_cents: int
    currency: str = "USD"

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount_cents + other.amount_cents, self.currency)
```

**frozen=True** makes the object immutable and hashable -- useful for value objects that serve as map keys.

### The Privacy Convention

Python has no access enforcement. It relies on convention:

```py
class BankAccount:
    def __init__(self, balance_cents: int):
        self._balance_cents = balance_cents   # "private" by convention
        self.__internal = "name-mangled"       # Harder to access, but not truly private

    @property
    def balance_cents(self) -> int:
        return self._balance_cents
```

Single underscore `_name` means "don't touch this from outside." Double underscore `__name` triggers name mangling (`_ClassName__name`), making accidental access harder but not impossible.

In interviews, use single underscore and `@property`. Mention that Python trusts developers to respect conventions rather than enforcing access.

## Common Pitfalls by Language

### Python: No True Private

```py
account = BankAccount(1000)
account._balance_cents = -9999  # Python won't stop you
```

In an interview, acknowledge this. Say: "Python uses conventions instead of enforcement. The underscore signals intent, but nothing prevents external access. For this design, we rely on the team respecting the interface."

### C++: Multiple Inheritance Complexity

Java allows single class inheritance plus multiple interfaces. C++ allows multiple inheritance from any classes, which introduces the diamond problem:

```C++
class A { public: void doSomething(); };
class B : public A {};
class C : public A {};
class D : public B, public C {};  // Two copies of A -- ambiguous!
```

The fix is virtual inheritance, but it adds complexity. In interviews, stick to single inheritance from one concrete class and multiple inheritance only from pure abstract classes (interfaces). This mirrors the Java model and avoids the diamond problem entirely.

### Python: ABC Checks Happen Late

```Python
class Broken(PaymentProcessor):
    pass  # Forgot to implement charge() and refund()

# This line is fine -- no error yet
x = None

# Error only happens HERE, at instantiation
x = Broken()  # TypeError: Can't instantiate abstract class
```

Java and C++ catch this at compile time. In Python, you won't know until you actually create an instance. In interviews, mention that you'd rely on type checkers like mypy to catch this earlier.

---