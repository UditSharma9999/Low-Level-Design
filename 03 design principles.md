# Chapter 1 DRY Principle (Don’t Repeat Yourself)

The DRY principle says that each piece of knowledge in your system should live in exactly one place. When you need that knowledge somewhere else, you reference the single source rather than creating a second copy.

## The Rule of Three

The idea is simple. Before extracting shared logic, wait until you see the same pattern three times. Two occurrences might be coincidental. Maybe those two pieces of code look similar today but will diverge tomorrow as their respective features evolve. Three occurrences, though, that is a pattern.

---
**Poor Test Coverage:** When logic is repeated, each copy needs its own tests. 

Redundant logic adds noise. When reading through a codebase, you want to quickly identify what is unique versus what is shared. If the same 10-line validation block appears in 15 files, those 150 lines contribute nothing new. 

If a line of code is extremely simple and unlikely to change, extracting it into a shared utility can actually make things worse


## Practical Example: Notification System

### The Problem

<details>
<summary>Python</summary>

```python
class OrderService:
    def notify_order_confirmation(self, user_id: str, order_id: str) -> None:
        # Duplicated: message formatting
        message = f"[Order] Hi {user_id}, your order {order_id} has been confirmed."
        formatted = message[0].upper() + message[1:]

        # Duplicated: sending logic
        print("Connecting to notification API...")
        print(f"Sending to {user_id}: {formatted}")
        print("Notification sent successfully.")

class ShippingService:
    def notify_shipment_update(self, user_id: str, tracking_id: str) -> None:
        # Duplicated: message formatting
        message = f"[Shipping] Hi {user_id}, your shipment {tracking_id} is on its way."
        formatted = message[0].upper() + message[1:]

        # Duplicated: sending logic
        print("Connecting to notification API...")
        print(f"Sending to {user_id}: {formatted}")
        print("Notification sent successfully.")

class SupportService:
    def notify_ticket_resolution(self, user_id: str, ticket_id: str) -> None:
        # Duplicated: message formatting
        message = f"[Support] Hi {user_id}, your ticket {ticket_id} has been resolved."
        formatted = message[0].upper() + message[1:]

        # Duplicated: sending logic
        print("Connecting to notification API...")
        print(f"Sending to {user_id}: {formatted}")
        print("Notification sent successfully.")
```

</details>

<details>
<summary>C++</summary>

```cpp
class OrderService {
public:
    void notifyOrderConfirmation(const string& userId,
                                  const string& orderId) {
        // Duplicated: message formatting
        string message = "[Order] Hi " + userId + ", your order "
            + orderId + " has been confirmed.";
        message[0] = toupper(message[0]);

        // Duplicated: sending logic
        cout << "Connecting to notification API..." << endl;
        cout << "Sending to " << userId << ": " << message << endl;
        cout << "Notification sent successfully." << endl;
    }
};

class ShippingService {
public:
    void notifyShipmentUpdate(const string& userId,
                               const string& trackingId) {
        // Duplicated: message formatting
        string message = "[Shipping] Hi " + userId + ", your shipment "
            + trackingId + " is on its way.";
        message[0] = toupper(message[0]);

        // Duplicated: sending logic
        cout << "Connecting to notification API..." << endl;
        cout << "Sending to " << userId << ": " << message << endl;
        cout << "Notification sent successfully." << endl;
    }
};

class SupportService {
public:
    void notifyTicketResolution(const string& userId,
                                 const string& ticketId) {
        // Duplicated: message formatting
        string message = "[Support] Hi " + userId + ", your ticket "
            + ticketId + " has been resolved.";
        message[0] = toupper(message[0]);

        // Duplicated: sending logic
        cout << "Connecting to notification API..." << endl;
        cout << "Sending to " << userId << ": " << message << endl;
        cout << "Notification sent successfully." << endl;
    }
};
```

</details>
<br/>

![text6](/assets/6.png)

### After: DRY Applied

<details>
<summary>Python</summary>

```python
class MessageFormatter:
    @staticmethod
    def format(category: str, user_id: str, detail: str) -> str:
        message = f"[{category}] Hi {user_id}, {detail}"
        return message[0].upper() + message[1:]

class NotificationSender:
    @staticmethod
    def send(user_id: str, message: str) -> None:
        print("Connecting to notification API...")
        print(f"Sending to {user_id}: {message}")
        print("Notification sent successfully.")

class OrderService:
    def notify_order_confirmation(self, user_id: str, order_id: str) -> None:
        message = MessageFormatter.format(
            "Order", user_id, f"your order {order_id} has been confirmed.")
        NotificationSender.send(user_id, message)

class ShippingService:
    def notify_shipment_update(self, user_id: str, tracking_id: str) -> None:
        message = MessageFormatter.format(
            "Shipping", user_id, f"your shipment {tracking_id} is on its way.")
        NotificationSender.send(user_id, message)

class SupportService:
    def notify_ticket_resolution(self, user_id: str, ticket_id: str) -> None:
        message = MessageFormatter.format(
            "Support", user_id, f"your ticket {ticket_id} has been resolved.")
        NotificationSender.send(user_id, message)
```

</details>

<details>
<summary>C++</summary>

```cpp
class MessageFormatter {
public:
    static string format(const string& category,
                               const string& userId,
                               const string& detail) {
        string message = "[" + category + "] Hi " + userId + ", " + detail;
        message[0] = toupper(message[0]);
        return message;
    }
};

class NotificationSender {
public:
    static void send(const string& userId, const string& message) {
        cout << "Connecting to notification API..." << endl;
        cout << "Sending to " << userId << ": " << message << endl;
        cout << "Notification sent successfully." << endl;
    }
};

class OrderService {
public:
    void notifyOrderConfirmation(const string& userId,
                                  const string& orderId) {
        string message = MessageFormatter::format(
            "Order", userId, "your order " + orderId + " has been confirmed.");
        NotificationSender::send(userId, message);
    }
};

class ShippingService {
public:
    void notifyShipmentUpdate(const string& userId,
                               const string& trackingId) {
        string message = MessageFormatter::format(
            "Shipping", userId, "your shipment " + trackingId + " is on its way.");
        NotificationSender::send(userId, message);
    }
};

class SupportService {
public:
    void notifyTicketResolution(const string& userId,
                                 const string& ticketId) {
        string message = MessageFormatter::format(
            "Support", userId, "your ticket " + ticketId + " has been resolved.");
        NotificationSender::send(userId, message);
    }
};
```

</details>
<br/>

##  ConfigLoader
### Refactor ConfigLoader

**Problem**: Your application loads configuration from three sources: a config file, environment variables, and default values. Each source currently has its own parsing and validation pipeline, but the pipeline logic (read, parse, validate, return) is identical. Your task is to eliminate this duplication.

**Requirements:**

- Create a `ConfigSource` interface with a `loadValue(key)` method
- Implement `FileConfigSource`, `EnvConfigSource`, and `DefaultConfigSource`
- Create a `ConfigLoader` that tries each source in priority order (file first, then env, then defaults) and returns the first non-null value
- Add validation: config values must be non-empty strings


### before:
<details>
<summary>Python</summary>

```python
import os

# Before: Each source has its own load-parse-validate pipeline
class AppConfig:
    def __init__(self):
        self.file_config = {"db.host": "localhost", "db.port": "5432"}
        self.defaults = {"db.host": "127.0.0.1", "db.port": "3306", "db.timeout": "30"}

    def get_from_file(self, key: str):
        value = self.file_config.get(key)
        if value is None or value == "":  # Duplicated validation
            return None
        return value

    def get_from_env(self, key: str):
        value = os.environ.get(key.replace(".", "_").upper())
        if value is None or value == "":  # Duplicated validation
            return None
        return value

    def get_from_defaults(self, key: str):
        value = self.defaults.get(key)
        if value is None or value == "":  # Duplicated validation
            return None
        return value

# TODO: Extract a ConfigSource interface (ABC) and create a ConfigLoader.

if __name__ == "__main__":
    # After refactoring, usage should look like:
    # loader = ConfigLoader([file_source, env_source, default_source])
    # host = loader.get("db.host")
    # print(f"db.host = {host}")
    pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class AppConfig {
private:
    map<string, string> fileConfig;
    map<string, string> defaults;

public:
    AppConfig() {
        fileConfig["db.host"] = "localhost";
        fileConfig["db.port"] = "5432";
        defaults["db.host"] = "127.0.0.1";
        defaults["db.port"] = "3306";
        defaults["db.timeout"] = "30";
    }

    string getFromFile(const string& key) {
        auto it = fileConfig.find(key);
        if (it == fileConfig.end() || it->second.empty()) return ""; // Duplicated
        return it->second;
    }

    string getFromEnv(const string& key) {
        const char* value = getenv(key.c_str());
        if (value == nullptr || string(value).empty()) return ""; // Duplicated
        return string(value);
    }

    string getFromDefaults(const string& key) {
        auto it = defaults.find(key);
        if (it == defaults.end() || it->second.empty()) return ""; // Duplicated
        return it->second;
    }
};

// TODO: Extract a ConfigSource interface and create a ConfigLoader.

int main() {
    // After refactoring, usage should look like:
    // ConfigLoader loader({fileSource, envSource, defaultSource});
    // string host = loader.get("db.host");
    return 0;
}
```

</details>


### After:

<details>
<summary>Python</summary>

```python
import os
from abc import ABC, abstractmethod

class ConfigSource(ABC):
    @abstractmethod
    def load_value(self, key: str):
        pass

class FileConfigSource(ConfigSource):
    def __init__(self, config: dict):
        self.config = config

    def load_value(self, key: str):
        return self.config.get(key)

class EnvConfigSource(ConfigSource):
    def load_value(self, key: str):
        return os.environ.get(key.replace(".", "_").upper())

class DefaultConfigSource(ConfigSource):
    def __init__(self, defaults: dict):
        self.defaults = defaults

    def load_value(self, key: str):
        return self.defaults.get(key)

class ConfigLoader:
    def __init__(self, sources: list):
        self.sources = sources

    def get(self, key: str):
        for source in self.sources:
            value = source.load_value(key)
            if value is not None and value != "":
                return value
        return None

if __name__ == "__main__":
    file_config = {"db.host": "localhost", "db.port": "5432"}
    defaults = {"db.host": "127.0.0.1", "db.port": "3306", "db.timeout": "30"}

    loader = ConfigLoader([
        FileConfigSource(file_config),
        EnvConfigSource(),
        DefaultConfigSource(defaults),
    ])

    print(f"db.host = {loader.get('db.host')}")
    print(f"db.port = {loader.get('db.port')}")
    print(f"db.timeout = {loader.get('db.timeout')}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class ConfigSource {
public:
    virtual ~ConfigSource() = default;
    virtual string loadValue(const string& key) const = 0;
};

class FileConfigSource : public ConfigSource {
private:
    map<string, string> config;

public:
    FileConfigSource(const map<string, string>& config)
        : config(config) {}

    string loadValue(const string& key) const override {
        auto it = config.find(key);
        if (it != config.end()) return it->second;
        return "";
    }
};

class EnvConfigSource : public ConfigSource {
public:
    string loadValue(const string& key) const override {
        string envKey = key;
        for (auto& c : envKey) {
            if (c == '.') c = '_';
            else c = toupper(c);
        }
        const char* value = getenv(envKey.c_str());
        return value ? string(value) : "";
    }
};

class DefaultConfigSource : public ConfigSource {
private:
    map<string, string> defaults;

public:
    DefaultConfigSource(const map<string, string>& defaults)
        : defaults(defaults) {}

    string loadValue(const string& key) const override {
        auto it = defaults.find(key);
        if (it != defaults.end()) return it->second;
        return "";
    }
};

class ConfigLoader {
private:
    vector<unique_ptr<ConfigSource>> sources;

public:
    ConfigLoader(vector<unique_ptr<ConfigSource>> sources)
        : sources(move(sources)) {}

    string get(const string& key) const {
        for (const auto& source : sources) {
            string value = source->loadValue(key);
            if (!value.empty()) {
                return value;
            }
        }
        return "";
    }
};

int main() {
    map<string, string> fileConfig = {
        {"db.host", "localhost"}, {"db.port", "5432"}
    };
    map<string, string> defaults = {
        {"db.host", "127.0.0.1"}, {"db.port", "3306"}, {"db.timeout", "30"}
    };

    vector<unique_ptr<ConfigSource>> sources;
    sources.push_back(make_unique<FileConfigSource>(fileConfig));
    sources.push_back(make_unique<EnvConfigSource>());
    sources.push_back(make_unique<DefaultConfigSource>(defaults));

    ConfigLoader loader(move(sources));

    cout << "db.host = " << loader.get("db.host") << endl;
    cout << "db.port = " << loader.get("db.port") << endl;
    cout << "db.timeout = " << loader.get("db.timeout") << endl;

    return 0;
}
```

</details>
<br/>



----


# Chapter 2 KISS Principle (Keep It Simple, Stupid.)



## The Complexity Cycle


![text7](/assets/7.png)

The Complexity Cycle means that software or systems usually do not become complicated overnight. Instead, complexity grows little by little. It often starts when a piece of code becomes slightly difficult to understand. Because it is harder to follow, developers are more likely to make mistakes or miss bugs. When a bug appears, instead of fixing the root cause, a quick workaround is added to solve the immediate problem. However, that workaround makes the code even more complicated. As complexity increases, understanding the code becomes even harder, which leads to more bugs and more workarounds. This creates a repeating cycle where the system becomes increasingly difficult to maintain. Eventually, the code may become so messy that the only practical solution is to rewrite it completely. The KISS (Keep It Simple, Stupid) principle helps prevent this cycle by encouraging simple, clear, and easy-to-understand solutions from the beginning.

## Signs You’re Violating KISS
A good sign that you're moving away from KISS is when you start adding complexity without a clear need. For example, creating an interface when there is only one implementation, using reflection instead of a straightforward method call, or adding extra layers of code "just in case" they might be useful in the future can make the code harder to understand and maintain.

**Favor Composition Over Inheritance**

-----


# Chapter 3 YAGNI Principle (You Aren’t Gonna Need It.)

> **Always implement things when you actually need them, never when you just foresee that you need them.**


---

# Chapter 4 Law of Demeter (LoD)

> **"Only talk to your direct friends, not to your friends' friends."**

this means that an object should only interact with objects it directly knows about. It should not reach deep into another object's internal structure to get information.

Think of it like ordering food in a restaurant. You tell the waiter what you want. You do not walk into the kitchen, open the refrigerator, find the ingredients, and tell the chef how to cook. The waiter acts as an intermediary and hides all those internal details from you. This makes the restaurant easier to run because the kitchen can change how it works without affecting customers.

The same idea applies in software.

Imagine an e-commerce application where a Customer owns a ShoppingCart, the cart contains CartItems, each item has a Product, and the product has a Price.

A bad approach would be:

```py
customer.getCart().getItems()[0].getProduct().getPrice()
```

This code is navigating through many objects to get a single piece of information. 

This creates a strong dependency on the internal structure of multiple classes.

The problem appears when the system changes.

Suppose the ShoppingCart stops using a list and starts using a dictionary. Or Product changes how prices are stored. Even though these changes have nothing to do with the service requesting the price, that service will break because it was directly relying on those internal details.

This is called **tight coupling**. Many parts of the system become dependent on each other, making the code harder to maintain.

The Law of Demeter suggests a better approach. Instead of digging through objects, ask the object that owns the information to provide what you need.

```py
customer.getFirstItemPrice()
```

Now the service only talks to the Customer. The Customer can internally ask the ShoppingCart, and the ShoppingCart can internally ask the CartItem and Product.

The responsibility becomes distributed properly:

```text
OrderService
    ↓
Customer
    ↓
ShoppingCart
    ↓
CartItem
    ↓
Product
```

Each class only communicates with the objects it directly owns.

#### Does LoD Mean Getters Are Bad?
No. 
Simple getters are perfectly fine. These are direct properties of the object itself. 
The issue starts when getters are chained
```py
customer.getAddress()
        .getCity()
        .getCountry()
        .getName()
```

Now the caller depends on the structure of multiple objects.

The deeper the chain, the stronger the coupling.


## The Problem
Imagine you're building a simple e-commerce system.

You have:

- A `Customer` who owns a `ShoppingCart`
- The cart contains a list of `CartItems`
- Each `CartItem` refers to a `Product`
- And every `Product` has a `Price`

![Text8](/assets/8.png)

```py
price = customer.get_shopping_cart().get_items()[0].get_product().get_price()
```


## Refactoring with LoD in Mind
Let's rewrite the e-commerce example in a cleaner, more respectful way.

The strategy is to **push the responsibility down** to the classes that own the data. Each class will expose a meaningful method instead of exposing its internals.

### 1. Add a method to ShoppingCart
The ShoppingCart knows about its items. So it should be the one to answer questions about them.

<details>
<summary>Python</summary>

```python
class ShoppingCart:
    def __init__(self):
        self._items = []

    # ... other methods ...

    def get_first_item_price(self):
        if not self._items:
            return Money.ZERO
        return self._items[0].get_product().get_price()
```

</details>

<details>
<summary>C++</summary>

```cpp
class ShoppingCart {
private:
    vector<CartItem> items;

public:
    // ... constructor, other methods ...

    Money getFirstItemPrice() const {
        if (items.empty()) return Money::ZERO;
        return items[0].getProduct().getPrice();
    }
};
```

</details>
<br/>

Notice that `ShoppingCart` still reaches into `CartItem` and `Product`. That is fine here because `ShoppingCart` owns the items. The chain stays within the cart's own responsibility boundary. The important thing is that **external callers no longer need** to know about these internals.


### Step 2: Add a method to Customer
The Customer owns the ShoppingCart, so it should be the one to delegate cart-related queries.


<details>
<summary>Python</summary>

```python
class Customer:
    def __init__(self, shopping_cart):
        self._shopping_cart = shopping_cart

    # ... other methods ...

    def get_first_cart_item_price(self):
        return self._shopping_cart.get_first_item_price()
```

</details>

<details>
<summary>C++</summary>

```cpp
class Customer {
private:
    ShoppingCart shoppingCart;

public:
    // ... constructor, other methods ...

    Money getFirstCartItemPrice() const {
        return shoppingCart.getFirstItemPrice();
    }
};
```

</details>
<br/>

### Step 3: Update the OrderService
Now the OrderService only talks to its direct friend: the Customer.

<details>
<summary>Python</summary>

```python
def display_first_item_price(customer: Customer):
    price = customer.get_first_cart_item_price()
    print("Price of the first item:", price.get_amount())
```

</details>

<details>
<summary>C++</summary>

```cpp
void displayFirstItemPrice(const Customer& customer) {
    Money price = customer.getFirstCartItemPrice();
    cout << "Price of the first item: " << price.getAmount() << endl;
}
```

</details>
<br/>


`OrderService` only talks to `Customer`. `Customer` only talks to `ShoppingCart`. Each layer hides the next one. Now `OrderService` does not care about:

- How the cart stores items
- What a CartItem contains
- How the Product holds its price

## When is it okay to “violate” the Law of Demeter?
LoD is a guideline—not a hard rule.

- It’s acceptable to traverse simple data carriers where behavior isn't expected.

- **Stable, Low-Level Libraries**: Using well-known APIs (like Map.get() or List.size()) is typically safe.

- **Fluent APIs / Builders**: Method chaining in fluent interfaces is usually an intentional design, not a violation.

## Practical Example: Ride Notification

### The Problem
A NotificationService class needs three pieces of information to send a ride update to a passenger:

- The driver's name
- The car's license plate
- The passenger's phone number

<details>
<summary>Python</summary>

```python
class NotificationService:
    def send_ride_update(self, ride):
        # Train wreck 1: reaching through Driver -> Profile to get name
        driver_name = (ride.get_driver()
                           .get_profile()
                           .get_full_name())

        # Train wreck 2: reaching through Driver -> Vehicle -> Registration
        plate = (ride.get_driver()
                     .get_vehicle()
                     .get_registration()
                     .get_license_plate())

        # Train wreck 3: reaching through Passenger -> ContactInfo
        phone = (ride.get_passenger()
                     .get_contact_info()
                     .get_phone_number())

        message = f"Your driver {driver_name} is arriving in a {plate}. Contact: {phone}"
        print(f"SMS to {phone}: {message}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class NotificationService {
public:
    void sendRideUpdate(const Ride& ride) {
        // Train wreck 1: reaching through Driver -> Profile to get name
        string driverName = ride.getDriver()
                                .getProfile()
                                .getFullName();

        // Train wreck 2: reaching through Driver -> Vehicle -> Registration
        string plate = ride.getDriver()
                           .getVehicle()
                           .getRegistration()
                           .getLicensePlate();

        // Train wreck 3: reaching through Passenger -> ContactInfo
        string phone = ride.getPassenger()
                           .getContactInfo()
                           .getPhoneNumber();

        cout << "SMS to " << phone << ": Your driver " << driverName
             << " is arriving in a " << plate
             << ". Contact: " << phone << endl;
    }
};
```

</details>
<br/>

![text9](/assets/9.png)

### The Fix: Delegation Methods

Instead of letting `NotificationService` navigate the entire object graph, we add delegation methods to `Ride`. The `Ride` class already has references to its driver and passenger, so it is the natural place to answer these questions.

<details>
<summary>Python</summary>

```python
class Ride:
    def __init__(self, driver, passenger):
        self._driver = driver
        self._passenger = passenger

    # ... other methods ...

    def get_driver_name(self):
        return self._driver.get_profile().get_full_name()

    def get_vehicle_plate(self):
        return self._driver.get_vehicle().get_registration().get_license_plate()

    def get_passenger_phone(self):
        return self._passenger.get_contact_info().get_phone_number()
```

</details>

<details>
<summary>C++</summary>

```cpp
class Ride {
private:
    Driver driver;
    Passenger passenger;

public:
    // ... constructor, other methods ...

    string getDriverName() const {
        return driver.getProfile().getFullName();
    }

    string getVehiclePlate() const {
        return driver.getVehicle().getRegistration().getLicensePlate();
    }

    string getPassengerPhone() const {
        return passenger.getContactInfo().getPhoneNumber();
    }
};
```

</details>
<br/>



Now the NotificationService becomes simple and clean:


<details>
<summary>Python</summary>

```python
class NotificationService:
    def send_ride_update(self, ride):
        driver_name = ride.get_driver_name()
        plate = ride.get_vehicle_plate()
        phone = ride.get_passenger_phone()

        message = f"Your driver {driver_name} is arriving in a {plate}. Contact: {phone}"
        print(f"SMS to {phone}: {message}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class NotificationService {
public:
    void sendRideUpdate(const Ride& ride) {
        string driverName = ride.getDriverName();
        string plate = ride.getVehiclePlate();
        string phone = ride.getPassengerPhone();

        cout << "SMS to " << phone << ": Your driver " << driverName
             << " is arriving in a " << plate
             << ". Contact: " << phone << endl;
    }
};
```

</details>
<br/>


![text10](/assets/10.png)

---

## Coupling

Coupling refers to the degree of dependency between different classes, modules, or components. It answers the question:

> "How much does one part of the system know about or rely on another part?"

## Cohesion

Cohesion refers to how closely related the responsibilities within a single class or module are. It answers the question:

> "Does this component have one clear purpose, or is it doing many unrelated things?"

<br/>

**Good software design typically aims for:**
- Low Coupling
- High Cohesion

