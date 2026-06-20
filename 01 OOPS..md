# Chapter 1 Enums

Imagine you're building an e-commerce platform and you need to track the status of every order. Orders flow through various states: placed, confirmed, shipped, delivered, or cancelled. How do you represent these states in code?

You could use strings: `"PLACED"`, `"SHIPPED"`, `"DELIVERED"`. But what happens when someone types `"Shiped"` instead of `"SHIPPED"`?

The compiler won't catch it and your code will silently fail at runtime.

You could use integers: 1 for placed, 2 for shipped, 3 for delivered. But now your code is littered with magic numbers. What does if (status == 2) mean?

This is exactly the type of problem enums solve.

## 1. What is an Enum?

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

## Enums with Properties and Methods

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

## Practical Example: 
### Order Processing System

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

// example code
class A {
public:
    static int x;
};
int A::x = 10;   // definition and initialization
//

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

### HTTP Status Code
**Problem**: Create an HttpStatus enum where each status has a numeric code and a message string.

**Requirements**:

- Values: OK(200, "OK"), BAD_REQUEST(400, "Bad Request"), NOT_FOUND(404, "Not Found"), INTERNAL_SERVER_ERROR(500, "Internal Server Error")
- `isSuccess()` method that returns true if the code is less than 400
- `display()` method that prints "CODE MESSAGE" (e.g. "200 OK")
- A `static fromCode(int)` method that returns the HttpStatus for a given code, or null/None if not found


<details>
<summary>Python</summary>

```python
from enum import Enum

class HttpStatus(Enum):
    OK = (200, "OK")
    BAD_REQUEST = (400, "Bad Request")
    NOT_FOUND = (404, "Not Found")
    INTERNAL_SERVER_ERROR = (500, "Internal Server Error")

    def __init__(self, code: int, message: str):
        self.code = code
        self.message = message

    def is_success(self) -> bool:
        return self.code < 400

    def display(self) -> None:
        print(f"{self.code} {self.message}")

    @staticmethod
    def from_code(code: int):
        for status in HttpStatus:
            if status.code == code:
                return status
        return None


if __name__ == "__main__":
    HttpStatus.OK.display()
    HttpStatus.NOT_FOUND.display()

    print(f"Is 200 success? {str(HttpStatus.OK.is_success()).lower()}")
    print(f"Is 404 success? {str(HttpStatus.NOT_FOUND.is_success()).lower()}")

    found = HttpStatus.from_code(500)
    if found is not None:
        print("Found by code 500: ", end="")
        found.display()
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

struct HttpStatus {
    int code;
    string message;

    static const HttpStatus OK;
    static const HttpStatus BAD_REQUEST;
    static const HttpStatus NOT_FOUND;
    static const HttpStatus INTERNAL_SERVER_ERROR;

    bool isSuccess() const {
        return code < 400;
    }

    void display() const {
        cout << code << " " << message << endl;
    }

    static const HttpStatus* fromCode(int code) {
        for (const auto* status : values()) {
            if (status->code == code) {
                return status;
            }
        }
        return nullptr;
    }

    static const vector<const HttpStatus*>& values() {
        static vector<const HttpStatus*> v = {&OK, &BAD_REQUEST, &NOT_FOUND, &INTERNAL_SERVER_ERROR};
        return v;
    }
};

const HttpStatus HttpStatus::OK{200, "OK"};
const HttpStatus HttpStatus::BAD_REQUEST{400, "Bad Request"};
const HttpStatus HttpStatus::NOT_FOUND{404, "Not Found"};
const HttpStatus HttpStatus::INTERNAL_SERVER_ERROR{500, "Internal Server Error"};

int main() {
    HttpStatus::OK.display();
    HttpStatus::NOT_FOUND.display();

    cout << "Is 200 success? " << (HttpStatus::OK.isSuccess() ? "true" : "false") << endl;
    cout << "Is 404 success? " << (HttpStatus::NOT_FOUND.isSuccess() ? "true" : "false") << endl;

    const HttpStatus* found = HttpStatus::fromCode(500);
    if (found != nullptr) {
        cout << "Found by code 500: ";
        found->display();
    }
    return 0;
}
```

</details>


---

# Chapter 2 Interface

A list of methods that any implementing class must provide. It specifies a set of behaviors that a class agrees to implement but leaves the details of those behaviors up to each implementation.

## 2. Key Properties of Interfaces
1. **Defines Behavior Without Dictating Implementation**

2. **Enables Polymorphism:** Different classes can implement the same interface in different ways.

3. **Promotes Decoupling:** Code that depends on interfaces is insulated from changes in the concrete classes that implement them.

## Code Example: Payment Gateway Interface

Let’s say you’re designing a payment processing module that supports multiple providers like Stripe, Razorpay, and PayPal.

You don’t want your business logic to depend on a specific provider. You just want a common way to initiate a payment.

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def initiate_payment(self, amount):
        pass

# Implementing an Interface
class StripePayment(PaymentGateway):
    def initiate_payment(self, amount):
        print(f"Processing payment via Stripe: ${amount}")
class RazorpayPayment(PaymentGateway):
    def initiate_payment(self, amount):
        print(f"Processing payment via Razorpay: ₹{amount}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentGateway {
public:
    virtual ~PaymentGateway() {}  // Virtual destructor for proper cleanup
    // A virtual destructor ensures that when a derived object is deleted through a base-class pointer, both the derived and base destructors are called.
    virtual void initiatePayment(double amount) = 0;  // Pure virtual function 
    // function declared with = 0 that has no implementation in the base class and must be implemented by derived classes.
};

// Implementing an Interface

class StripePayment : public PaymentGateway {
public:
    void initiatePayment(double amount) override {
        cout << "Processing payment via Stripe: $" << amount << endl;
    }
};
class RazorpayPayment : public PaymentGateway {
public:
    void initiatePayment(double amount) override {
        cout << "Processing payment via Razorpay: ₹" << amount << endl;
    }
};
```

</details>

### Programming to the Interface
Here's where the real payoff happens. Instead of having `CheckoutService` depend on `StripePayment` or RazorpayPayment, it depends on the `PaymentGateway` interface. It doesn't know or care which implementation it's using.

<details>
<summary>Python</summary>

```python
class CheckoutService:
    def __init__(self, payment_gateway):
        self.payment_gateway = payment_gateway
    
    def set_payment_gateway(self, payment_gateway):
        self.payment_gateway = payment_gateway
    
    def checkout(self, amount):
        self.payment_gateway.initiate_payment(amount)
```

</details>

<details>
<summary>C++</summary>

```cpp
class CheckoutService {
private:
    PaymentGateway* paymentGateway;
public:
    CheckoutService(PaymentGateway* gateway) : paymentGateway(gateway) {}

    void setPaymentGateway(PaymentGateway* gateway) {
        paymentGateway = gateway;
    }
    
    void checkout(double amount) {
        if (paymentGateway != nullptr) {
            paymentGateway->initiatePayment(amount);
        }
    }
};
```

</details>


Look at the `CheckoutService` constructor. It takes a `PaymentGateway`, not a `StripePayment`. This single decision is what **decouples** the service from any specific provider.

The checkout() method calls initiatePayment() on whatever gateway was injected. It could be Stripe, Razorpay, a mock for testing, or a provider that doesn't even exist yet.

This pattern is called **dependency injection**: instead of creating its own dependencies, the class receives them from the outside. And **it only works because the dependency is typed as an interface**, not a concrete class.


#### Runtime Flexibility
The final piece is wiring everything together. At runtime, you choose which implementation to inject, and you can even swap it out on the fly.

<details>
<summary>Python</summary>

```python
if __name__ == "__main__":
    stripe_gateway = StripePayment()
    checkout_service = CheckoutService(stripe_gateway)
    checkout_service.checkout(120.50)  # Output: Processing payment via Stripe: $120.5
    
    # Switch to Razorpay
    razorpay_gateway = RazorpayPayment()
    checkout_service.set_payment_gateway(razorpay_gateway)
    checkout_service.checkout(150.50)  # Output: Processing payment via Razorpay: ₹150.5
```

</details>

<details>
<summary>C++</summary>

```cpp
class CheckoutService {
private:
    PaymentGateway* paymentGateway;

public:
    CheckoutService(PaymentGateway* gateway) : paymentGateway(gateway) {}
    
    void setPaymentGateway(PaymentGateway* gateway) {
        paymentGateway = gateway;
    }
    
    void checkout(double amount) {
        if (paymentGateway != nullptr) {
            paymentGateway->initiatePayment(amount);
        }
    }
};
```

</details>



## Practical Example: Notification Service
Let's apply interfaces to a different domain. Imagine you're building an alerting system for a DevOps platform. When something goes wrong (server down, high CPU, disk full), the system needs to send notifications. Some teams prefer email, others use Slack, and some have custom webhook integrations.

The alerting service shouldn't know or care which channel is being used. It just sends the notification through whatever channel was configured.

Here's the class diagram for this design:

![Text1](/assets/1.png)


<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class NotificationService(ABC):
    @abstractmethod
    def send(self, recipient: str, message: str) -> None:
        pass

class EmailNotifier(NotificationService):
    def send(self, recipient: str, message: str) -> None:
        print(f"[Email] To: {recipient} | {message}")

class SlackNotifier(NotificationService):
    def send(self, recipient: str, message: str) -> None:
        print(f"[Slack] Channel: {recipient} | {message}")

class WebhookNotifier(NotificationService):
    def send(self, recipient: str, message: str) -> None:
        print(f"[Webhook] URL: {recipient} | {message}")

class AlertService:
    def __init__(self, notifier: NotificationService):
        self._notifier = notifier

    def trigger_alert(self, recipient: str, issue: str) -> None:
        alert_message = f"ALERT: {issue}"
        self._notifier.send(recipient, alert_message)


if __name__ == "__main__":
    email_alerts = AlertService(EmailNotifier())
    email_alerts.trigger_alert("ops@company.com", "CPU usage at 95%")

    slack_alerts = AlertService(SlackNotifier())
    slack_alerts.trigger_alert("#incidents", "Database connection pool exhausted")

    webhook_alerts = AlertService(WebhookNotifier())
    webhook_alerts.trigger_alert("https://hooks.example.com/alerts", "Disk usage at 90%")
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

class NotificationService {
public:
    virtual ~NotificationService() {}
    virtual void send(const std::string& recipient, const std::string& message) = 0;
};

class EmailNotifier : public NotificationService {
public:
    void send(const std::string& recipient, const std::string& message) override {
        std::cout << "[Email] To: " << recipient << " | " << message << std::endl;
    }
};

class SlackNotifier : public NotificationService {
public:
    void send(const std::string& recipient, const std::string& message) override {
        std::cout << "[Slack] Channel: " << recipient << " | " << message << std::endl;
    }
};

class WebhookNotifier : public NotificationService {
public:
    void send(const std::string& recipient, const std::string& message) override {
        std::cout << "[Webhook] URL: " << recipient << " | " << message << std::endl;
    }
};

class AlertService {
private:
    NotificationService* notifier;

public:
    AlertService(NotificationService* notifier) : notifier(notifier) {}

    void triggerAlert(const std::string& recipient, const std::string& issue) {
        std::string alertMessage = "ALERT: " + issue;
        notifier->send(recipient, alertMessage);
    }
};

int main() {
    EmailNotifier emailNotifier;
    AlertService emailAlerts(&emailNotifier);
    emailAlerts.triggerAlert("ops@company.com", "CPU usage at 95%");

    SlackNotifier slackNotifier;
    AlertService slackAlerts(&slackNotifier);
    slackAlerts.triggerAlert("#incidents", "Database connection pool exhausted");

    WebhookNotifier webhookNotifier;
    AlertService webhookAlerts(&webhookNotifier);
    webhookAlerts.triggerAlert("https://hooks.example.com/alerts", "Disk usage at 90%");

    return 0;
}
```

</details>

---

# Chapter 3 Encapsulation

> Encapsulation = Data hiding + Controlled access

## How Encapsulation is Achieved
Encapsulation is primarily implemented using two language features: access modifiers that control visibility, and getters/setters that provide controlled access to private data.

### 1. Access Modifiers
- **Access** modifiers are keywords that control which parts of your code can see and interact with a class's fields and methods. The three most common are:

- **private**: Accessible only within the same class. This is the primary tool for hiding data.
protected: Accessible within the same class and its subclasses. Useful when child classes need access to parent data.
- **public**: Accessible from anywhere. This is what you use for the controlled interface.

The **general rule** is simple: make everything private by default, then selectively expose what needs to be public.


<details>
<summary>Python</summary>

```python
class Product:
    def __init__(self, name: str, price: float):
        self.__name = name       # Name-mangled (private by convention)
        self.__price = price     # Name-mangled (private by convention)

    def get_name(self) -> str:   # Anyone can read the name
        return self.__name

    def get_price(self) -> float:  # Anyone can read the price
        return self.__price
```

</details>

<details>
<summary>C++</summary>

```cpp
class Product {
private:
    string name;         // Only this class can access
    double price;        // Only this class can access

public:
    Product(const string& name, double price)
        : name(name), price(price) {}

    string getName() const {    // Anyone can read the name
        return name;
    }

    double getPrice() const {   // Anyone can read the price
        return price;
    }
};
```

</details>



### 2. Getters and Setters
These are public methods that provide controlled, indirect access to private attributes.

<details>
<summary>Python</summary>

```python
class Product:
    def __init__(self, name: str, price: float):
        self.__name = name
        self.price = price  # Uses the property setter for validation

    @property
    def name(self) -> str:
        return self.__name

    @property
    def price(self) -> float:
        return self.__price

    @price.setter
    def price(self, value: float) -> None:
        if value < 0:
            raise ValueError("Price cannot be negative")
        self.__price = value
```

</details>

<details>
<summary>C++</summary>

```cpp
class Product {
private:
    string name;
    double price;

public:
    Product(const string& name, double price) : name(name), price(0) {
        setPrice(price); // Use the setter for validation
    }

    string getName() const { return name; }

    double getPrice() const { return price; }

    void setPrice(double price) {
        if (price < 0) {
            throw invalid_argument("Price cannot be negative");
        }
        this->price = price;
    }
};
```

</details>


## Practical Example: PaymentProcessor
You're building a PaymentProcessor class that handles credit card transactions. The raw card number must never be stored or visible anywhere in the system. If a developer accidentally logs the payment object or inspects it in a debugger, they should only see a masked version.

<details>
<summary>Python</summary>

```python
class PaymentProcessor:
    def __init__(self, card_number: str, amount: float):
        self.__card_number = self.__mask_card_number(card_number)
        self.__amount = amount

    def __mask_card_number(self, card_number: str) -> str:
        return "****-****-****-" + card_number[-4:]

    def process_payment(self) -> None:
        print(f"Processing payment of ${self.__amount} for card {self.__card_number}")


if __name__ == "__main__":
    payment = PaymentProcessor("1234567812345678", 250.00)
    payment.process_payment()
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentProcessor {
private:
    std::string cardNumber;
    double amount;

    std::string maskCardNumber(const std::string& cardNumber) {
        return "****-****-****-" + cardNumber.substr(cardNumber.length() - 4);
    }

public:
    PaymentProcessor(const std::string& cardNumber, double amount)
        : cardNumber(maskCardNumber(cardNumber)), amount(amount) {}

    void processPayment() {
        std::cout << "Processing payment of $" << amount
                  << " for card " << cardNumber << std::endl;
    }
};

int main() {
    PaymentProcessor payment("1234567812345678", 250.00);
    payment.processPayment();
    return 0;
}
```

</details>


## Exercise 1: TemperatureSensor
**Design Temperature Sensor Class**

Problem: Build a TemperatureSensor class that collects temperature readings and provides statistical access. The sensor should validate that readings fall within a reasonable range and never expose its internal list of readings directly.

**Requirements:**

- Private list of readings
- **addReading(value)**: adds a temperature reading, but only if it's between -50 and 150 degrees (inclusive). Reject out-of-range values.
- **getAverage()**: returns the average of all readings, or 0 if no readings exist
- **getReadingCount()**: returns how many readings have been recorded
- **getReadings()**: returns a copy of the readings list (not the original)

<details>
<summary>Python</summary>

```python
class TemperatureSensor:
    def __init__(self):
        self._readings: list[float] = []

    def add_reading(self, value: float) -> None:
        if -50 <= value <= 150:
            self._readings.append(value)

    def get_average(self) -> float:
        if not self._readings:
            return 0.0
        return round(sum(self._readings) / len(self._readings), 2)

    def get_reading_count(self) -> int:
        return len(self._readings)

    def get_readings(self) -> list[float]:
        return list(self._readings)


if __name__ == "__main__":
    sensor = TemperatureSensor()
    sensor.add_reading(22.5)
    sensor.add_reading(23.1)
    sensor.add_reading(200.0)  # Should be rejected
    sensor.add_reading(-10.0)

    print(f"Count: {sensor.get_reading_count()}")  # 3
    print(f"Average: {sensor.get_average()}")       # 11.87
```

</details>

<details>
<summary>C++</summary>

```cpp
class TemperatureSensor {
private:
    vector<double> readings;

public:
    void addReading(double value) {
        if (value >= -50 && value <= 150) {
            readings.push_back(value);
        }
    }

    double getAverage() const {
        if (readings.empty()) {
            return 0.0;
        }
        double sum = accumulate(readings.begin(), readings.end(), 0.0);
        return round(sum / readings.size() * 100.0) / 100.0;
    }

    int getReadingCount() const {
        return readings.size();
    }

    vector<double> getReadings() const {
        return readings;
    }
};

int main() {
    TemperatureSensor sensor;
    sensor.addReading(22.5);
    sensor.addReading(23.1);
    sensor.addReading(200.0);  // Should be rejected
    sensor.addReading(-10.0);

    cout << "Count: " << sensor.getReadingCount() << endl;  // 3
    cout << "Average: " << sensor.getAverage() << endl;     // 11.87
    return 0;
}
```

</details>


---


# Chapter 4 Abstraction
> Abstraction = Hiding Complexity + Showing Essentials

## Why Abstraction Matters
1. Swap Implementations Without Changing Callers
2. Reduce Complexity for Consumers
3. Extend Without Modifying
4. Share Common Logic Once

## How Abstraction Is Achieved
### 1. Abstract Classes
An abstract class defines a common blueprint for a family of related classes. It can contain both abstract methods (declared but not implemented) and concrete methods (fully implemented). Subclasses must implement the abstract methods but inherit the concrete ones (already implemented and shared by all derived classes.) for free.

This is what makes abstract classes different from interfaces: they let you share behavior, not just a contract.

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod
from datetime import datetime

class Logger(ABC):
    def __init__(self, level: str):
        self._level = level

    # Abstract method: subclasses decide HOW to deliver the message
    @abstractmethod
    def log(self, message: str) -> None:
        pass

    # Concrete method: shared formatting logic inherited by all subclasses
    def format_message(self, message: str) -> str:
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        return f"[{timestamp}] [{self._level}] {message}"

class ConsoleLogger(Logger):
    def __init__(self, level: str):
        super().__init__(level)

    def log(self, message: str) -> None:
        print(self.format_message(message))

class FileLogger(Logger):
    def __init__(self, level: str, file_path: str):
        super().__init__(level)
        self._file_path = file_path

    def log(self, message: str) -> None:
        # In production, this would write to a file
        print(f"Writing to {self._file_path}: {self.format_message(message)}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class Logger {
protected:
    std::string level;

    // Concrete method: shared formatting logic
    std::string formatMessage(const std::string& message) {
        time_t now = time(nullptr);
        char timestamp[20];
        strftime(timestamp, sizeof(timestamp), "%Y-%m-%d %H:%M:%S", localtime(&now));
        return "[" + std::string(timestamp) + "] [" + level + "] " + message;
    }

public:
    Logger(const std::string& level) : level(level) {}
    virtual ~Logger() {}

    // Abstract method: subclasses decide HOW to deliver the message
    virtual void log(const std::string& message) = 0;
};

class ConsoleLogger : public Logger {
public:
    ConsoleLogger(const std::string& level) : Logger(level) {}

    void log(const std::string& message) override {
        std::cout << formatMessage(message) << std::endl;
    }
};

class FileLogger : public Logger {
private:
    std::string filePath;

public:
    FileLogger(const std::string& level, const std::string& filePath)
        : Logger(level), filePath(filePath) {}

    void log(const std::string& message) override {
        std::cout << "Writing to " << filePath << ": "
                  << formatMessage(message) << std::endl;
    }
};
```

</details>


### 2. Interfaces as Abstraction
<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class Exportable(ABC):
    @abstractmethod
    def export(self) -> str:
        pass

class CSVExporter(Exportable):
    def export(self) -> str:
        return "name,email,age\nAlice,alice@example.com,30"

class JSONExporter(Exportable):
    def export(self) -> str:
        return '{"name": "Alice", "email": "alice@example.com"}'
```

</details>

<details>
<summary>C++</summary>

```cpp
class Exportable {
public:
    virtual ~Exportable() {}
    virtual std::string exportData() = 0;
};

class CSVExporter : public Exportable {
public:
    std::string exportData() override {
        return "name,email,age\nAlice,alice@example.com,30";
    }
};

class JSONExporter : public Exportable {
public:
    std::string exportData() override {
        return "{\"name\": \"Alice\", \"email\": \"alice@example.com\"}";
    }
};
```

</details>


### 3. Public APIs as Abstraction
You don't always need abstract classes or interfaces to achieve abstraction. Sometimes a well-designed public API on a regular class is enough. When a class hides its internal complexity behind a few clean public methods, that's abstraction in action.

Consider a `DatabaseClient`. The caller sees `connect()` and `query()`. Behind the scenes, the class manages connection pooling, socket lifecycle, authentication handshakes, query parsing, and retry logic. None of that is the caller's concern.

<details>
<summary>Python</summary>

```python
class DatabaseClient:
    def __init__(self, max_connections: int, retry_attempts: int):
        self.__max_connections = max_connections
        self.__retry_attempts = retry_attempts

    # Clean public API: the caller's view
    def connect(self, host: str, port: int) -> None:
        self.__open_socket(host, port)
        self.__authenticate()
        self.__initialize_connection_pool()

    def query(self, sql: str) -> str:
        parsed_query = self.__parse_query(sql)
        return self.__execute_with_retry(parsed_query)

    # Hidden complexity: the implementation details
    def __open_socket(self, host: str, port: int) -> None: pass
    def __authenticate(self) -> None: pass
    def __initialize_connection_pool(self) -> None: pass
    def __parse_query(self, sql: str) -> str: return sql.strip()
    def __execute_with_retry(self, query: str) -> str:
        for i in range(self.__retry_attempts):
            try:
                return self.__execute_query(query)
            except Exception:
                if i == self.__retry_attempts - 1:
                    raise
        return ""
    def __execute_query(self, query: str) -> str: return "result"
```

</details>

<details>
<summary>C++</summary>

```cpp
class DatabaseClient {
private:
    int maxConnections;
    int retryAttempts;

    void openSocket(const std::string& host, int port) { /* TCP connection */ }
    void authenticate() { /* Credential exchange */ }
    void initializeConnectionPool() { /* Pool management */ }
    std::string parseQuery(const std::string& sql) { return sql; }
    std::string executeWithRetry(const std::string& query) {
        for (int i = 0; i < retryAttempts; i++) {
            try {
                return executeQuery(query);
            } catch (...) {
                if (i == retryAttempts - 1) throw;
            }
        }
        return "";
    }
    std::string executeQuery(const std::string& query) { return "result"; }

public:
    DatabaseClient(int maxConnections, int retryAttempts)
        : maxConnections(maxConnections), retryAttempts(retryAttempts) {}

    void connect(const std::string& host, int port) {
        openSocket(host, port);
        authenticate();
        initializeConnectionPool();
    }

    std::string query(const std::string& sql) {
        std::string parsedQuery = parseQuery(sql);
        return executeWithRetry(parsedQuery);
    }
};
```

</details>



From the caller's perspective, using this class is just few lines:


```cpp
DatabaseClient db = new DatabaseClient(10, 3);
db.connect("localhost", 5432);
String result = db.query("SELECT * FROM users");
```


## Abstraction vs Encapsulation


Think of it this way: **Abstraction is the external view of an object, while Encapsulation is the internal view.**

| Aspect | Encapsulation | Abstraction |
|----------|-------------|-------------|
| Focus | Protecting data within a class | Hiding implementation complexity |
| Goal | Restrict access to internal state | Simplify usage and expose only essentials |
| Level | Implementation-level | Design-level |
| Example | Private `balance` field in `BankAccount` | Exposing only `deposit()` and `withdraw()` without showing how they work |

## Practical Example: Media Player
Let's apply abstraction to a different domain. Imagine you're building a media application that needs to play different types of content: audio files, video files, and streaming content. Each type has a completely different playback mechanism, but they all share certain behaviors: displaying the current status and logging user actions.

Here's the class diagram:

![Text2](/assets/2.png)

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class MediaPlayer(ABC):
    def __init__(self, player_name: str):
        self._player_name = player_name

    @abstractmethod
    def play(self) -> None:
        pass

    @abstractmethod
    def pause(self) -> None:
        pass

    @abstractmethod
    def stop(self) -> None:
        pass

    def display_status(self) -> None:
        print(f"[{self._player_name}] Status: Ready")

    def log_action(self, action: str) -> None:
        print(f"[{self._player_name}] Action: {action}")


class AudioPlayer(MediaPlayer):
    def __init__(self, audio_file: str):
        super().__init__("AudioPlayer")
        self._audio_file = audio_file

    def play(self) -> None:
        self.log_action(f"Playing audio: {self._audio_file}")

    def pause(self) -> None:
        self.log_action(f"Paused audio: {self._audio_file}")

    def stop(self) -> None:
        self.log_action(f"Stopped audio: {self._audio_file}")


class VideoPlayer(MediaPlayer):
    def __init__(self, video_file: str, resolution: str):
        super().__init__("VideoPlayer")
        self._video_file = video_file
        self._resolution = resolution

    def play(self) -> None:
        self.log_action(f"Playing video: {self._video_file} at {self._resolution}")

    def pause(self) -> None:
        self.log_action(f"Paused video: {self._video_file}")

    def stop(self) -> None:
        self.log_action(f"Stopped video: {self._video_file}")


class StreamingPlayer(MediaPlayer):
    def __init__(self, stream_url: str, buffer_size: int):
        super().__init__("StreamingPlayer")
        self._stream_url = stream_url
        self._buffer_size = buffer_size

    def play(self) -> None:
        self.log_action(f"Streaming from: {self._stream_url} (buffer: {self._buffer_size}KB)")

    def pause(self) -> None:
        self.log_action(f"Paused stream: {self._stream_url}")

    def stop(self) -> None:
        self.log_action(f"Stopped stream: {self._stream_url}")


class PlayerController:
    def __init__(self, player: MediaPlayer):
        self._player = player

    def start_playback(self) -> None:
        self._player.display_status()
        self._player.play()

    def pause_playback(self) -> None:
        self._player.pause()

    def stop_playback(self) -> None:
        self._player.stop()


if __name__ == "__main__":
    audio_ctrl = PlayerController(AudioPlayer("song.mp3"))
    audio_ctrl.start_playback()
    audio_ctrl.pause_playback()

    print()

    video_ctrl = PlayerController(VideoPlayer("movie.mp4", "1080p"))
    video_ctrl.start_playback()
    video_ctrl.stop_playback()

    print()

    stream_ctrl = PlayerController(
        StreamingPlayer("https://stream.example.com/live", 2048))
    stream_ctrl.start_playback()
    stream_ctrl.stop_playback()
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

class MediaPlayer {
protected:
    std::string playerName;

    void logAction(const std::string& action) {
        std::cout << "[" << playerName << "] Action: " << action << std::endl;
    }

public:
    MediaPlayer(const std::string& playerName) : playerName(playerName) {}
    virtual ~MediaPlayer() {}

    virtual void play() = 0;
    virtual void pause() = 0;
    virtual void stop() = 0;

    void displayStatus() {
        std::cout << "[" << playerName << "] Status: Ready" << std::endl;
    }
};

class AudioPlayer : public MediaPlayer {
private:
    std::string audioFile;

public:
    AudioPlayer(const std::string& audioFile)
        : MediaPlayer("AudioPlayer"), audioFile(audioFile) {}

    void play() override { logAction("Playing audio: " + audioFile); }
    void pause() override { logAction("Paused audio: " + audioFile); }
    void stop() override { logAction("Stopped audio: " + audioFile); }
};

class VideoPlayer : public MediaPlayer {
private:
    std::string videoFile;
    std::string resolution;

public:
    VideoPlayer(const std::string& videoFile, const std::string& resolution)
        : MediaPlayer("VideoPlayer"), videoFile(videoFile), resolution(resolution) {}

    void play() override {
        logAction("Playing video: " + videoFile + " at " + resolution);
    }
    void pause() override { logAction("Paused video: " + videoFile); }
    void stop() override { logAction("Stopped video: " + videoFile); }
};

class StreamingPlayer : public MediaPlayer {
private:
    std::string streamUrl;
    int bufferSize;

public:
    StreamingPlayer(const std::string& streamUrl, int bufferSize)
        : MediaPlayer("StreamingPlayer"), streamUrl(streamUrl), bufferSize(bufferSize) {}

    void play() override {
        logAction("Streaming from: " + streamUrl + " (buffer: "
            + std::to_string(bufferSize) + "KB)");
    }
    void pause() override { logAction("Paused stream: " + streamUrl); }
    void stop() override { logAction("Stopped stream: " + streamUrl); }
};

class PlayerController {
private:
    MediaPlayer* player;

public:
    PlayerController(MediaPlayer* player) : player(player) {}

    void startPlayback() {
        player->displayStatus();
        player->play();
    }
    void pausePlayback() { player->pause(); }
    void stopPlayback() { player->stop(); }
};

int main() {
    AudioPlayer audio("song.mp3");
    PlayerController audioCtrl(&audio);
    audioCtrl.startPlayback();
    audioCtrl.pausePlayback();

    std::cout << std::endl;

    VideoPlayer video("movie.mp4", "1080p");
    PlayerController videoCtrl(&video);
    videoCtrl.startPlayback();
    videoCtrl.stopPlayback();

    std::cout << std::endl;

    StreamingPlayer stream("https://stream.example.com/live", 2048);
    PlayerController streamCtrl(&stream);
    streamCtrl.startPlayback();
    streamCtrl.stopPlayback();

    return 0;
}
```

</details>

**Why This Design Works**

- **The controller is player-agnostic**. PlayerController doesn't import `AudioPlayer`, `VideoPlayer`, or `StreamingPlayer`. It only knows about `MediaPlayer`. Adding a new player type (say, `PodcastPlayer`) requires zero changes to the controller.

- **Shared behavior is written once**. `displayStatus()` and `logAction()` live in the abstract class. All three concrete players inherit them without reimplementing a single line. If you want to change the status format, you change one method in one place.

- **Each player encapsulates its own complexity**. `StreamingPlayer` manages buffering, `VideoPlayer` handles resolution. The controller doesn't know or care about any of these details. It just calls `play()`.


---

# Chapter 5 Inheritance

> Inheritance enables code reuse by letting you define common logic once in a base class and then extend or specialize it in multiple derived classes.

## Why Inheritance Matters
1. **Code Reusability**
2. **Logical Hierarchy**
3. **Ease of Maintenance**
4. **Polymorphism**: Inheritance is a prerequisite for polymorphism, allowing objects of different subclasses to be treated as objects of the superclass.

## Types of Inheritance
1. **Single Inheritance**
2. **Multi-level Inheritance**
3. **Hierarchical Inheritance**
4. **Multiple Inheritance** is when a child class extends more than one parent. This is where things get complicated. Only C++ and Python support multiple inheritance directly. Java, C#, and TypeScript do not. The reason? The diamond problem.


Imagine `ElectricC`ar extends both `Vehicle` and Machine. Both Vehicle and Machine have a `start()` method. When you call `electricCar.start()`, which version runs? The one from `Vehicle`? The one from `Machine`? Both?

C++ handles this with **virtual inheritance**, which is complex and error-prone. Python handles it with the **Method Resolution Order (MRO)**, a well-defined algorithm (C3 linearization) that determines which parent's method takes priority.
Java and C# sidestep the problem entirely by only allowing single class inheritance, you can implement multiple interfaces, but extend only one class.


To solve the diamond problem in C++, use the `virtual` keyword during inheritance to ensure only one shared instance of the grandparent class is created.

### The Standard Solution: Virtual Inheritance
When two intermediate classes inherit from a common base class, mark the inheritance as `virtual`. This tells the compiler to merge the duplicated paths into a single shared base sub-object.

```cpp
#include <iostream>

class Grandparent {
public:
    void show() { std::cout << "Grandparent class called.\n"; }
};

// Use the virtual keyword here
class Parent1 : virtual public Grandparent {};
class Parent2 : virtual public Grandparent {};

// Child inherits from both intermediate parents
class Child : public Parent1, public Parent2 {};

int main() {
    Child obj;
    obj.show(); // OK: No ambiguity, resolved via virtual inheritance!
    return 0;
}
```

#### Alternative Workarounds
If you cannot modify the inheritance hierarchy to use `virtual`, you can bypass or fix the ambiguity locally using these methods:
- **Scope Resolution Operator (::)**: Explicitly tell the compiler which parent path to follow when calling a method or accessing a variable.
```cpp
// If virtual inheritance is not used, specify the path manually
obj.Parent1::show(); 
```

- **Method Overriding in Child**: Override the ambiguous function directly inside the Child class and choose which parent function to execute.

```cpp
class Child : public Parent1, public Parent2 {
public:
    void show() {
        Parent1::show(); // Solves ambiguity for anyone calling obj.show()
    }
};
```

## When to Use Inheritance

- There is a clear "is-a" relationship
- The parent class defines common behavior or data that children should share. 
- The child class does not violate the behavior expected from the parent.

## Avoid inheritance when:

- The relationship is "has-a" or "uses-a" rather than "is-a".
- You want to combine behaviors from multiple sources dynamically.

- You need runtime flexibility to swap behaviors. With composition, you can inject different implementations. With inheritance, the parent relationship is fixed.   
- You want to avoid tight coupling between child and parent internals.

## Practical Example: Notification System

Imagine you're building a notification system that can send messages through different channels: email, SMS, and push notifications.

All notification types share common properties: a `recipient`, a `message`, and a `timestamp`. They all need a `formatHeader()` method that produces a consistent header format. But the `send()` method works differently for each channel, email needs a subject line, SMS has a character limit, and push notifications have a device token and priority level.

<details>
<summary>Python</summary>

```python
from datetime import datetime


class Notification:
    def __init__(self, recipient: str, message: str):
        self._recipient = recipient
        self._message = message
        self._timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    def format_header(self) -> str:
        return f"[{self._timestamp}] To: {self._recipient}"

    def send(self):
        print(self.format_header())
        print(f"Message: {self._message}")


class EmailNotification(Notification):
    def __init__(self, recipient: str, message: str, subject: str):
        super().__init__(recipient, message)
        self._subject = subject

    def send(self):
        print(self.format_header())
        print(f"Subject: {self._subject}")
        print(f"Body: {self._message}")
        print("Status: Email delivered")


class SMSNotification(Notification):
    MAX_LENGTH = 160

    def __init__(self, recipient: str, message: str, phone_number: str):
        super().__init__(recipient, message)
        self._phone_number = phone_number

    def send(self):
        print(self.format_header())
        print(f"Phone: {self._phone_number}")
        sms_body = (self._message[:self.MAX_LENGTH - 3] + "..."
                    if len(self._message) > self.MAX_LENGTH
                    else self._message)
        print(f"SMS: {sms_body}")
        print(f"Status: SMS sent ({len(sms_body)}/{self.MAX_LENGTH} chars)")


class PushNotification(Notification):
    def __init__(self, recipient: str, message: str,
                 device_token: str, priority: str):
        super().__init__(recipient, message)
        self._device_token = device_token
        self._priority = priority

    def send(self):
        print(self.format_header())
        print(f"Device: {self._device_token[:8]}...")
        print(f"Priority: {self._priority}")
        print(f"Alert: {self._message}")
        print("Status: Push notification delivered")


if __name__ == "__main__":
    email = EmailNotification(
        "alice@example.com", "Your order has been shipped!", "Order Update")
    email.send()

    print()

    sms = SMSNotification(
        "Bob", "Your verification code is 482910.", "+1-555-0123")
    sms.send()

    print()

    push = PushNotification(
        "Charlie", "New message from Alice", "d8a3f4b2c1e5a9b7", "high")
    push.send()
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <ctime>

using namespace std;

class Notification {
protected:
    string recipient;
    string message;
    string timestamp;

public:
    Notification(const string& recipient, const string& message)
        : recipient(recipient), message(message) {
        time_t now = time(nullptr);
        char buf[20];
        strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", localtime(&now));
        timestamp = buf;
    }

    virtual ~Notification() {}

    string formatHeader() {
        return "[" + timestamp + "] To: " + recipient;
    }

    virtual void send() {
        cout << formatHeader() << endl;
        cout << "Message: " << message << endl;
    }
};

class EmailNotification : public Notification {
private:
    string subject;

public:
    EmailNotification(const string& recipient, const string& message,
                      const string& subject)
        : Notification(recipient, message), subject(subject) {}

    void send() override {
        cout << formatHeader() << endl;
        cout << "Subject: " << subject << endl;
        cout << "Body: " << message << endl;
        cout << "Status: Email delivered" << endl;
    }
};

class SMSNotification : public Notification {
private:
    string phoneNumber;
    static const int MAX_LENGTH = 160;

public:
    SMSNotification(const string& recipient, const string& message,
                    const string& phoneNumber)
        : Notification(recipient, message), phoneNumber(phoneNumber) {}

    void send() override {
        cout << formatHeader() << endl;
        cout << "Phone: " << phoneNumber << endl;
        string smsBody = message.length() > MAX_LENGTH
            ? message.substr(0, MAX_LENGTH - 3) + "..."
            : message;
        cout << "SMS: " << smsBody << endl;
        cout << "Status: SMS sent (" << smsBody.length()
                  << "/" << MAX_LENGTH << " chars)" << endl;
    }
};

class PushNotification : public Notification {
private:
    string deviceToken;
    string priority;

public:
    PushNotification(const string& recipient, const string& message,
                     const string& deviceToken, const string& priority)
        : Notification(recipient, message), deviceToken(deviceToken),
          priority(priority) {}

    void send() override {
        cout << formatHeader() << endl;
        cout << "Device: " << deviceToken.substr(0, 8) << "..." << endl;
        cout << "Priority: " << priority << endl;
        cout << "Alert: " << message << endl;
        cout << "Status: Push notification delivered" << endl;
    }
};

int main() {
    EmailNotification email("alice@example.com",
        "Your order has been shipped!", "Order Update");
    email.send();

    cout << endl;

    SMSNotification sms("Bob", "Your verification code is 482910.", "+1-555-0123");
    sms.send();

    cout << endl;

    PushNotification push("Charlie", "New message from Alice",
        "d8a3f4b2c1e5a9b7", "high");
    push.send();

    return 0;
}
```

</details>

<br/>

A **virtual destructor** ensures that when a base-class pointer deletes a derived-class object, the derived-class destructor is called first, then the base-class destructor.

```cpp
virtual ~Notification() {}
```

`n` looks like a `Notification`, but the actual object is an EmailNotification.

virtual `~Notification()` tells C++:

- "When deleting this object, check its real type and destroy the whole object, not just the base-class part."

So:
- With `virtual` → `EmailNotification` and `Notification` are both cleaned up.
- Without `virtual` → only `Notification` is cleaned up.


## Exercise: Shape Hierarchy

#### Design Shape Hierarchy Class
**Problem**: Build a shape hierarchy where the base class provides a shared `describe()` method, and each child class implements its own `area()` and `perimeter()` methods.

**Requirements**:

- Base `Shape` class with a `name` field (protected). A `describe()` method that prints "Shape: [name], Area: [area], Perimeter: [perimeter]". Abstract-like `area()` and `perimeter()` methods that return 0 by default.

- `Circle`: takes a `radius`. Area = pi r^2, Perimeter = 2 pi * r.

- `Rectangle`: takes `width` and `height`. Area = w h, Perimeter = 2 (w + h).

- `describe()` should work for any shape without modification because it calls `area()` and `perimeter()` internally.

<details>
<summary>Python</summary>

```python
import math


class Shape:
    def __init__(self, name: str):
        self._name = name

    def area(self) -> float:
        return 0

    def perimeter(self) -> float:
        return 0

    def describe(self):
        print(f"Shape: {self._name}, Area: {self.area():.2f}, Perimeter: {self.perimeter():.2f}")


class Circle(Shape):
    def __init__(self, radius: float):
        super().__init__("Circle")
        self._radius = radius

    def area(self) -> float:
        return math.pi * self._radius * self._radius

    def perimeter(self) -> float:
        return 2 * math.pi * self._radius


class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        super().__init__("Rectangle")
        self._width = width
        self._height = height

    def area(self) -> float:
        return self._width * self._height

    def perimeter(self) -> float:
        return 2 * (self._width + self._height)


if __name__ == "__main__":
    circle = Circle(5.0)
    circle.describe()

    rect = Rectangle(4.0, 6.0)
    rect.describe()
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <cmath>
#include <cstdio>

using namespace std;

class Shape {
protected:
    string name;

public:
    Shape(const string& name) : name(name) {}
    virtual ~Shape() {}

    virtual double area() {
        return 0;
    }

    virtual double perimeter() {
        return 0;
    }

    void describe() {
        printf("Shape: %s, Area: %.2f, Perimeter: %.2f\n",
               name.c_str(), area(), perimeter());
    }
};

class Circle : public Shape {
private:
    double radius;

public:
    // super class constructor
    Circle(double radius) : Shape("Circle"), radius(radius) {}

    double area() override {
        return M_PI * radius * radius;
    }

    double perimeter() override {
        return 2 * M_PI * radius;
    }
};

class Rectangle : public Shape {
private:
    double width;
    double height;

public:
    Rectangle(double width, double height)
        : Shape("Rectangle"), width(width), height(height) {}

    double area() override {
        return width * height;
    }

    double perimeter() override {
        return 2 * (width + height);
    }
};

int main() {
    Circle circle(5.0);
    circle.describe();

    Rectangle rect(4.0, 6.0);
    rect.describe();

    return 0;
}
```

</details>


---

# Chapter 6 Polymorphism

> Polymorphism lets you call the same method on different objects, and have each object respond in its own way.


## Why Polymorphism Matters

- **Encourages loose coupling**: You interact with abstractions (interfaces or base classes), not specific implementations.

- **Enhances flexibility**: You can introduce new behaviors without modifying existing code, supporting the Open/Closed Principle.


## How Polymorphism Works

### 1. Compile-time Polymorphism (Method Overloading)
Happens when you have multiple methods with the same name in the same class but with different parameter lists.

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
#include <iostream>

class Calculator {
public:
    // Two ints
    int add(int a, int b) {
        return a + b;
    }

    // Two doubles
    double add(double a, double b) {
        return a + b;
    }

    // Three ints
    int add(int a, int b, int c) {
        return a + b + c;
    }
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


### 2. Runtime Polymorphism (Method Overriding / Dynamic Dispatch)
Runtime polymorphism is the more powerful and more important form. It happens when a child class `overrides` a method defined in its parent class, and the decision of which version to call is made `at runtime` based on the actual type of the object, not the declared type of the reference.

**Example**   
Suppose you’re designing a system that sends notifications. You want to support email, SMS, push notifications, etc.




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
#include <iostream>
#include <string>
#include <vector>
#include <memory>
using namespace std;

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

<br/>


The key thing to notice: every element in the list is stored as a `Notification` reference, but the runtime calls the correct child class's `send()`. The variable type says `Notification`. The behavior says `EmailNotification`, `SMSNotification`, or `PushNotification`. That's runtime polymorphism.


#### for cpp code :

`unique_ptr` is a **smart pointer** that owns an object and automatically deletes it when it's no longer needed. It prevents memory leaks and removes the need to write delete manually.

```cpp
unique_ptr<Notification> p(new EmailNotification(...));
```
When p goes out of scope, the EmailNotification object is automatically destroyed.


`make_unique` is a safer and cleaner way to create a `unique_ptr`.

```cpp
// Instead of:
unique_ptr<Notification> p(new EmailNotification(...));

// you write:
auto p = make_unique<EmailNotification>(...);
// It creates the object and wraps it in a unique_ptr in one step.
```

Use `unique_ptr` when you create an object and want C++ to automatically clean it up for you.
Now you don't have to remember `delete`.

```cpp
// Instead of:
Notification* n = new EmailNotification(...);
// ...
delete n;

// you write:
auto n = make_unique<EmailNotification>(...);
```

**Simple rule**: If you use new, prefer unique_ptr so you don't have to manage memory yourself.

## Polymorphism with Interfaces vs Abstract Classes

Both interfaces and abstract classes enable polymorphism. In the notification example, you could define `Notification` as either an abstract class or an interface. The polymorphic behavior, calling `send()` on a base reference and having the child's version execute, works the same either way. So when should you use which?

- Use an **interface** when different types of objects just need to have the same ability. For example, `Email`, `Invoice`, and `Report` are very different things, but they can all have a send() function. The interface only says "you must provide `send()`" and doesn't contain shared data or code.

- Use an abstract class when the objects belong to the same family and share common data or behavior. For example, `EmailNotification`, `SMSNotification`, and `PushNotification` are all notifications. They can share fields like `recipient` and `message`, and methods like `formatHeader()`, while each provides its own version of `send()`.

> 💡Use an Interface when classes are not from the same family, but they share a capability.   
> 💡 Use an Abstract Class when classes are from the same family and can share data or code.

## Exercise 1: Discount Calculator
#### Design Discount Calculator Class
**Problem**: Build a pricing system where an `OrderProcessor` applies discounts polymorphically. Different discount types share a common base class with shared formatting logic, but each one calculates the discounted price differently. The processor works with any discount through the abstract `Discount` type.



<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod


class Discount(ABC):
    def __init__(self, label: str):
        self._label = label

    @abstractmethod
    def apply(self, price: float) -> float:
        pass

    def describe(self, original_price: float):
        discounted_price = self.apply(original_price)
        print(f"{self._label}: ${original_price:.2f} -> ${discounted_price:.2f}")


class PercentageDiscount(Discount):
    def __init__(self, percentage: float):
        super().__init__(f"{percentage:.1f}% off")
        self._percentage = percentage

    def apply(self, price: float) -> float:
        return price * (1 - self._percentage / 100)


class FlatDiscount(Discount):
    def __init__(self, amount: float):
        super().__init__(f"${amount:.1f} off")
        self._amount = amount

    def apply(self, price: float) -> float:
        return max(price - self._amount, 0)


class BuyOneGetOneFree(Discount):
    def __init__(self):
        super().__init__("Buy 1 Get 1 Free")

    def apply(self, price: float) -> float:
        return price / 2


class OrderProcessor:
    def process_order(self, item_name: str, price: float, discount: Discount):
        print(f"Item: {item_name}")
        discount.describe(price)


if __name__ == "__main__":
    processor = OrderProcessor()

    processor.process_order("Laptop", 999.99, PercentageDiscount(20))
    processor.process_order("Headphones", 49.99, FlatDiscount(15))
    processor.process_order("Keyboard", 79.98, BuyOneGetOneFree())
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <cstdio>
#include <algorithm>
using namespace std;

class Discount {
protected:
    string label;

public:
    Discount(const string& label) : label(label) {}
    virtual ~Discount() {}

    virtual double apply(double price) = 0;

    void describe(double originalPrice) {
        double discountedPrice = apply(originalPrice);
        printf("%s: $%.2f -> $%.2f\n", label.c_str(), originalPrice, discountedPrice);
    }
};

class PercentageDiscount : public Discount {
    double percentage;

public:
    PercentageDiscount(double percentage)
        : Discount(string()) {
        char buf[32];
        snprintf(buf, sizeof(buf), "%.1f", percentage);
        this->label = string(buf) + "% off";
        this->percentage = percentage;
    }

    double apply(double price) override {
        return price * (1 - percentage / 100);
    }
};

class FlatDiscount : public Discount {
    double amount;

public:
    FlatDiscount(double amount)
        : Discount(string()) {
        char buf[32];
        snprintf(buf, sizeof(buf), "%.1f", amount);
        this->label = "$" + string(buf) + " off";
        this->amount = amount;
    }

    double apply(double price) override {
        return max(price - amount, 0.0);
    }
};

class BuyOneGetOneFree : public Discount {
public:
    BuyOneGetOneFree() : Discount("Buy 1 Get 1 Free") {}

    double apply(double price) override {
        return price / 2;
    }
};

class OrderProcessor {
public:
    void processOrder(const string& itemName, double price, Discount& discount) {
        cout << "Item: " << itemName << endl;
        discount.describe(price);
    }
};

int main() {
    OrderProcessor processor;

    PercentageDiscount pct(20);
    FlatDiscount flat(15);
    BuyOneGetOneFree bogo;

    processor.processOrder("Laptop", 999.99, pct);
    processor.processOrder("Headphones", 49.99, flat);
    processor.processOrder("Keyboard", 79.98, bogo);
    return 0;
}
```

</details>
<br/>


## Exercise 2: Logging System
#### Design Logging System Class
**Problem**: Build a logging system where the application uses a **Logger** interface polymorphically. Different logger implementations send messages to different destinations, and the application doesn't know or care which one it's using.


<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod


class Logger(ABC):
    @abstractmethod
    def log(self, level: str, message: str) -> None:
        pass

    @abstractmethod
    def get_destination(self) -> str:
        pass


class ConsoleLogger(Logger):
    def log(self, level: str, message: str) -> None:
        print(f"[{level}] {message}")

    def get_destination(self) -> str:
        return "Console"


class FileLogger(Logger):
    def __init__(self, file_path: str):
        self._file_path = file_path

    def log(self, level: str, message: str) -> None:
        print(f"Writing to {self._file_path}: [{level}] {message}")

    def get_destination(self) -> str:
        return f"File: {self._file_path}"


class DatabaseLogger(Logger):
    def __init__(self, table_name: str):
        self._table_name = table_name

    def log(self, level: str, message: str) -> None:
        print(f"INSERT INTO {self._table_name}: [{level}] {message}")

    def get_destination(self) -> str:
        return f"Database: {self._table_name}"


class Application:
    def __init__(self, logger: Logger):
        self._logger = logger

    def run(self):
        self._logger.log("INFO", "Application starting...")
        self._logger.log("INFO", "Processing data...")
        self._logger.log("INFO", "Application shutting down.")


if __name__ == "__main__":
    loggers = [
        ConsoleLogger(),
        FileLogger("/var/log/app.log"),
        DatabaseLogger("app_logs"),
    ]

    for logger in loggers:
        print(f"--- Using {logger.get_destination()} ---")
        app = Application(logger)
        app.run()
        print()
```

</details>

<details>
<summary>C++</summary>

```cpp
class Logger {
public:
    virtual ~Logger() {}
    virtual void log(const string& level, const string& message) = 0;
    virtual string getDestination() = 0;
};

class ConsoleLogger : public Logger {
public:
    void log(const string& level, const string& message) override {
        cout << "[" << level << "] " << message << endl;
    }

    string getDestination() override {
        return "Console";
    }
};

class FileLogger : public Logger {
private:
    string filePath;

public:
    FileLogger(const string& filePath) {
        this->filePath = filePath;
    }

    void log(const string& level, const string& message) override {
        cout << "Writing to " << filePath << ": [" << level << "] " << message << endl;
    }

    string getDestination() override {
        return "File: " + filePath;
    }
};

class DatabaseLogger : public Logger {
private:
    string tableName;

public:
    DatabaseLogger(const string& tableName) {
        this->tableName = tableName;
    }

    void log(const string& level, const string& message) override {
        cout << "INSERT INTO " << tableName << ": [" << level << "] " << message << endl;
    }

    string getDestination() override {
        return "Database: " + tableName;
    }
};

class Application {
private:
    Logger* logger;

public:
    Application(Logger* logger) {
        this->logger = logger;
    }

    void run() {
        logger->log("INFO", "Application starting...");
        logger->log("INFO", "Processing data...");
        logger->log("INFO", "Application shutting down.");
    }
};

int main() {
    ConsoleLogger console;
    FileLogger file("/var/log/app.log");
    DatabaseLogger db("app_logs");

    vector<Logger*> loggers = {&console, &file, &db};

    for (auto* logger : loggers) {
        cout << "--- Using " << logger->getDestination() << " ---" << endl;
        Application app(logger);
        app.run();
        cout << endl;
    }

    return 0;
}
```

</details>
<br/>



---

