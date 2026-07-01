# Factory Method Design Pattern

**The Factory Method Design Pattern is a creational pattern that provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.**

- The exact type of object to be created isn't known until runtime.
- Object creation logic is complex, repetitive, or needs encapsulation.
- You **want to follow the Open/Closed Principle**, open for extension, closed for modification.


Instead of one central factory deciding what to create, you delegate the responsibility to specialized classes that know exactly what they need to produce.

## Class Diagram


![text13](/assets/13.png)



### 1. Product (e.g., Notification)
The interface or abstract class that defines the contract for all objects the factory method creates. Every concrete product implements this interface, which means the rest of the system can work with any product without knowing its concrete type.

`Notification` interface with its send() method. The creator and client code only ever reference this interface, never `EmailNotification` or `SMSNotification` directly.

### 2. ConcreteProduct (e.g., EmailNotification)
The actual classes that implement the Product interface. Each one provides its own behavior.

### 3. Creator (e.g., NotificationCreator)
An abstract class (or an interface) that declares the factory method, which returns an object of type Product.

1. **Declares the factory method** (createNotification()) that subclasses must implement.
2. **Contains shared logic** that uses the product. For example, the send() method in

### 4. ConcreteCreator (e.g., EmailNotificationCreator)
Subclasses of Creator that override the factory method to return a specific ConcreteProduct. Each creator is paired with exactly one product type.

- EmailNotificationCreator returns new EmailNotification().

- SMSNotificationCreator returns new SMSNotification()

## Implementing Factory Method

<details>
<summary>Python</summary>

```python

## 1. Define the Product Interface
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None:
        pass

## 2. Define Concrete Products
class EmailNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Sending email: {message}")

class SMSNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Sending SMS: {message}")

class PushNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Sending push notification: {message}")

class SlackNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Sending Slack message: {message}")

## 3 Define an Abstract Creator
class NotificationCreator(ABC):
    # Factory Method - subclasses decide what to create
    @abstractmethod
    def create_notification(self) -> Notification:
        pass

    # Shared logic that uses the factory method
    def send(self, message: str) -> None:
        notification = self.create_notification()
        notification.send(message)


## 4. Define Concrete Creators
class EmailNotificationCreator(NotificationCreator):
    def create_notification(self) -> Notification:
        return EmailNotification()

class SMSNotificationCreator(NotificationCreator):
    def create_notification(self) -> Notification:
        return SMSNotification()

class PushNotificationCreator(NotificationCreator):
    def create_notification(self) -> Notification:
        return PushNotification()

class SlackNotificationCreator(NotificationCreator):
    def create_notification(self) -> Notification:
        return SlackNotification()

## 5. Client Code
def main():
    # Send Email
    creator = EmailNotificationCreator()
    creator.send("Welcome to our platform!")

    # Send SMS
    creator = SMSNotificationCreator()
    creator.send("Your OTP is 123456")

    # Send Push Notification
    creator = PushNotificationCreator()
    creator.send("You have a new follower!")

    # Send Slack Message
    creator = SlackNotificationCreator()
    creator.send("Standup in 10 minutes!")

if __name__ == "__main__":
    main()


```

</details>

<details>
<summary>C++</summary>

```cpp

// 1. Define the Product Interface
class Notification {
public:
    virtual void send(const string& message) = 0;
    virtual ~Notification() {}
};


// 2. Define Concrete Products
class EmailNotification : public Notification {
public:
    void send(const string& message) override {
        cout << "Sending email: " << message << endl;
    }
};

class SMSNotification : public Notification {
public:
    void send(const string& message) override {
        cout << "Sending SMS: " << message << endl;
    }
};

class PushNotification : public Notification {
public:
    void send(const string& message) override {
        cout << "Sending push notification: " << message << endl;
    }
};

class SlackNotification : public Notification {
public:
    void send(const string& message) override {
        cout << "Sending Slack message: " << message << endl;
    }
};


// 3 Define an Abstract Creator
class NotificationCreator {
public:
    // Factory Method - subclasses decide what to create
    virtual unique_ptr<Notification> createNotification() = 0;

    // Shared logic that uses the factory method
    void send(const string& message) {
        auto notification = createNotification();
        notification->send(message);
    }

    virtual ~NotificationCreator() = default;
};


// 4. Define Concrete Creators
class EmailNotificationCreator : public NotificationCreator {
public:
    unique_ptr<Notification> createNotification() override {
        return make_unique<EmailNotification>();
    }
};

class SMSNotificationCreator : public NotificationCreator {
public:
    unique_ptr<Notification> createNotification() override {
        return make_unique<SMSNotification>();
    }
};

class PushNotificationCreator : public NotificationCreator {
public:
    unique_ptr<Notification> createNotification() override {
        return make_unique<PushNotification>();
    }
};

class SlackNotificationCreator : public NotificationCreator {
public:
    unique_ptr<Notification> createNotification() override {
        return make_unique<SlackNotification>();
    }
};


// 5. Client Code
int main() {
    // Send Email
    unique_ptr<NotificationCreator> creator = make_unique<EmailNotificationCreator>();
    creator->send("Welcome to our platform!");

    // Send SMS
    creator = make_unique<SMSNotificationCreator>();
    creator->send("Your OTP is 123456");

    // Send Push Notification
    creator = make_unique<PushNotificationCreator>();
    creator->send("You have a new follower!");

    // Send Slack Message
    creator = make_unique<SlackNotificationCreator>();
    creator->send("Standup in 10 minutes!");

    return 0;
}
```

</details>
<br/>


# Abstract Factory Design Pattern

***The Abstract Factory Design Pattern is a creational pattern that provides an interface for creating families of related or dependent objects without specifying their concrete classes.***



The **Factory Method Pattern** is used when a class needs to create a single object, but the exact type of that object can vary. It defines a factory method that subclasses or different implementations use to create the required object.

The **Abstract Factory Pattern**, on the other hand, is used when you need to create multiple related objects that must work together. Instead of creating just one object, it creates an entire family of compatible objects.

![text14](/assets/14.png)

## Practical Example: Notification System

Let's build a notification system that supports Email and SMS channels. Each channel produces two related objects: a Message and a Sender. Mixing an email message with an SMS sender would produce garbled output, so family consistency matters.

![text15](/assets/15.png)

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

# Abstract Products
class Message(ABC):
    @abstractmethod
    def set_content(self, to: str, body: str):
        pass

    @abstractmethod
    def format(self) -> str:
        pass

class Sender(ABC):
    @abstractmethod
    def send(self, message: Message):
        pass

# Email Products
class EmailMessage(Message):
    def set_content(self, to: str, body: str):
        self.to = to
        self.body = body

    def format(self) -> str:
        return f"Email to <{self.to}>: {self.body}"

class EmailSender(Sender):
    def send(self, message: Message):
        print(f"Sending via SMTP: {message.format()}")

# SMS Products
class SmsMessage(Message):
    def set_content(self, to: str, body: str):
        self.to = to
        self.body = body[:160]

    def format(self) -> str:
        return f"SMS to {self.to}: {self.body}"

class SmsSender(Sender):
    def send(self, message: Message):
        print(f"Sending via carrier API: {message.format()}")

# Abstract Factory
class NotificationFactory(ABC):
    @abstractmethod
    def create_message(self) -> Message:
        pass

    @abstractmethod
    def create_sender(self) -> Sender:
        pass

# Concrete Factories
class EmailFactory(NotificationFactory):
    def create_message(self) -> Message:
        return EmailMessage()

    def create_sender(self) -> Sender:
        return EmailSender()

class SmsFactory(NotificationFactory):
    def create_message(self) -> Message:
        return SmsMessage()

    def create_sender(self) -> Sender:
        return SmsSender()

# Client
class NotificationService:
    def __init__(self, factory: NotificationFactory):
        self.factory = factory

    def notify(self, to: str, body: str):
        message = self.factory.create_message()
        message.set_content(to, body)
        sender = self.factory.create_sender()
        sender.send(message)

# Entry Point
if __name__ == "__main__":
    print("=== Email Notification ===")
    email_service = NotificationService(EmailFactory())
    email_service.notify("alice@example.com", "Your order has been shipped!")

    print()

    print("=== SMS Notification ===")
    sms_service = NotificationService(SmsFactory())
    sms_service.notify("+1-555-0123", "Your order has been shipped!")
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

using namespace std;

// Abstract Products
class Message {
public:
    virtual void setContent(const string& to, const string& body) = 0;
    virtual string format() const = 0;
    virtual ~Message() = default;
};

class Sender {
public:
    virtual void send(const Message& message) = 0;
    virtual ~Sender() = default;
};

// Email Products
class EmailMessage : public Message {
    string to, body;
public:
    void setContent(const string& to, const string& body) override {
        this->to = to;
        this->body = body;
    }
    string format() const override {
        return "Email to <" + to + ">: " + body;
    }
};

class EmailSender : public Sender {
public:
    void send(const Message& message) override {
        cout << "Sending via SMTP: " << message.format() << endl;
    }
};

// SMS Products
class SmsMessage : public Message {
    string to, body;
public:
    void setContent(const string& to, const string& body) override {
        this->to = to;
        this->body = body.length() > 160 ? body.substr(0, 160) : body;
    }
    string format() const override {
        return "SMS to " + to + ": " + body;
    }
};

class SmsSender : public Sender {
public:
    void send(const Message& message) override {
        cout << "Sending via carrier API: " << message.format() << endl;
    }
};

// Abstract Factory
class NotificationFactory {
public:
    virtual Message* createMessage() = 0;
    virtual Sender* createSender() = 0;
    virtual ~NotificationFactory() = default;
};

// Concrete Factories
class EmailFactory : public NotificationFactory {
public:
    Message* createMessage() override { return new EmailMessage(); }
    Sender* createSender() override { return new EmailSender(); }
};

class SmsFactory : public NotificationFactory {
public:
    Message* createMessage() override { return new SmsMessage(); }
    Sender* createSender() override { return new SmsSender(); }
};

// Client
class NotificationService {
    NotificationFactory* factory;
public:
    NotificationService(NotificationFactory* factory) : factory(factory) {}

    void notify(const string& to, const string& body) {
        Message* message = factory->createMessage();
        message->setContent(to, body);
        Sender* sender = factory->createSender();
        sender->send(*message);
        delete message;
        delete sender;
    }
};

int main() {
    cout << "=== Email Notification ===" << endl;
    EmailFactory emailFactory;
    NotificationService emailService(&emailFactory);
    emailService.notify("alice@example.com", "Your order has been shipped!");

    cout << endl;

    cout << "=== SMS Notification ===" << endl;
    SmsFactory smsFactory;
    NotificationService smsService(&smsFactory);
    smsService.notify("+1-555-0123", "Your order has been shipped!");

    return 0;
}
```

</details>
<br/>

