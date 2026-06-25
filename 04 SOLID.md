# Chapter 1 Single Responsibility Principle (SRP)

#### A class should have one, and only one, reason to change.

## The Problem: The God Class

<details>
<summary>Python</summary>

```python
class UserService:
    def __init__(self, username: str, email: str, password: str):
        self._username = username
        self._email = email
        self._password = password

    def validate_and_hash_password(self):
        # Check password strength
        # Generate salt
        # Hash with bcrypt
        pass

    def save_to_database(self):
        # Connect to database
        # Prepare SQL
        # Execute query
        pass

    def generate_auth_token(self):
        # Create JWT payload
        # Sign with secret key
        # Return token string
        pass

    def send_welcome_email(self):
        # Connect to email server
        # Build welcome template
        # Send email
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class UserService {
private:
    string username;
    string email;
    string password;

public:
    UserService(const string& username, const string& email, const string& password)
        : username(username), email(email), password(password) {}

    string validateAndHashPassword() {
        // Check password strength
        // Generate salt
        // Hash with bcrypt
    }

    void saveToDatabase() {
        // Connect to database
        // Prepare SQL
        // Execute query
    }

    string generateAuthToken() {
        // Create JWT payload
        // Sign with secret key
        // Return token string
    }

    void sendWelcomeEmail() {
        // Connect to email server
        // Build welcome template
        // Send email
    }
};
```

</details>
<br/>

Four distinct responsibilities rolled into one class.

## Applying SRP

Time to fix our original UserService God Class using SRP.

### The Core: User Class

Let's start by extracting a simple User data class. Its only job is to represent a user.

<details>
<summary>Python</summary>

```python
class User:
    def __init__(self, username: str, email: str, password: str):
        self._username = username
        self._email = email
        self._password = password

    def get_username(self) -> str:
        return self._username

    def get_email(self) -> str:
        return self._email

    def get_password(self) -> str:
        return self._password
```

</details>

<details>
<summary>C++</summary>

```cpp
class User {
private:
    string username;
    string email;
    string password;

public:
    User(const string& username, const string& email, const string& password)
        : username(username), email(email), password(password) {}

    string getUsername() const {
        return username;
    }

    string getEmail() const {
        return email;
    }

    string getPassword() const {
        return password;
    }
};
```

</details>
<br/>

### Responsibility 1: Password Hashing

This class handles just the logic of validating and hashing a password.

<details>
<summary>Python</summary>

```python
class PasswordHasher:
    def validate_and_hash(self, password: str) -> str:
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")
        # Generate salt and hash with bcrypt
        return "bcrypt_hashed_" + password  # Simplified for illustration
```

</details>

<details>
<summary>C++</summary>

```cpp
class PasswordHasher {
public:
    string validateAndHash(const string& password) {
        if (password.length() < 8) {
            throw invalid_argument("Password must be at least 8 characters");
        }
        // Generate salt and hash with bcrypt
        return "bcrypt_hashed_" + password; // Simplified for illustration
    }
};
```

</details>
<br/>

### Responsibility 2: Persistence to Database

The responsibility for talking to the database belongs here.

<details>
<summary>Python</summary>

```python
class AuthTokenService:
    def generate_token(self, user: User) -> str:
        # Create JWT payload with user claims
        payload = f'{{"username":"{user.get_username()}","email":"{user.get_email()}"}}'
        # Sign with secret key (simplified)
        return f"eyJhbGciOiJIUzI1NiJ9.{payload}.signature"
```

</details>

<details>
<summary>C++</summary>

```cpp
class UserRepository {
public:
    void save(const User& user) {
        cout << "Saving user " << user.getUsername() << " to database..." << endl;
    }
};
```

</details>
<br/>

### Responsibility 3: Auth Token Generation

This class handles the creation of authentication tokens.

<details>
<summary>Python</summary>

```python
class AuthTokenService:
    def generate_token(self, user: User) -> str:
        # Create JWT payload with user claims
        payload = f'{{"username":"{user.get_username()}","email":"{user.get_email()}"}}'
        # Sign with secret key (simplified)
        return f"eyJhbGciOiJIUzI1NiJ9.{payload}.signature"
```

</details>

<details>
<summary>C++</summary>

```cpp
class AuthTokenService {
public:
    string generateToken(const User& user) {
        // Create JWT payload with user claims
        string payload = "{\"username\":\"" + user.getUsername() + "\",\"email\":\"" + user.getEmail() + "\"}";
        // Sign with secret key (simplified)
        return "eyJhbGciOiJIUzI1NiJ9." + payload + ".signature";
    }
};
```

</details>
<br/>

### Responsibility 4: Sending the Welcome Email

This class is responsible only for sending emails.

<details>
<summary>Python</summary>

```python
class EmailService:
    def send_welcome_email(self, user: User):
        print(f"Sending welcome email to: {user.get_email()}")
        print(f"Welcome to our platform, {user.get_username()}!")
```

</details>

<details>
<summary>C++</summary>

```cpp
class EmailService {
public:
    void sendWelcomeEmail(const User& user) {
        cout << "Sending welcome email to: " << user.getEmail() << endl;
        cout << "Welcome to our platform, " << user.getUsername() << "!" << endl;
    }
};
```

</details>
<br/>

## Common Pitfalls While Applying SRP

1. **Over-Splitting Responsibilities**

2. **Confusing Methods with Responsibilities** : Some developers might try to split this into WelcomeEmailSender and PayslipEmailSender.

3. **Ignoring SRP in Small or Utility Classes** : This class is small and works fine, no need to split it." But a utility class that starts off simple can quietly grow into a mess

---

# Chapter 2 Open-Closed Principle (OCP)

## The Problem: A Growing Payment System

<details>
<summary>Python</summary>

```python
class PaymentProcessor:
    def process_credit_card_payment(self, amount):
        print(f"Processing credit card payment of ${amount}")
        # Complex logic for credit card processing

    def process_paypal_payment(self, amount):
        print(f"Processing PayPal payment of ${amount}")
        # Logic for PayPal processing
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentProcessor {
public:
    void processCreditCardPayment(double amount) {
        cout << "Processing credit card payment of $" << amount << endl;
        // Complex logic for credit card processing
    }

    void processPayPalPayment(double amount) {
        cout << "Processing PayPal payment of $" << amount << endl;
        // Logic for PayPal processing
    }
};
```

</details>
<br/>

Now it works for two methods. But guess what happens when the client wants you to add UPI, Bitcoin, or Apple Pay?

## Introducing the Open-Closed Principle (OCP)

**Software entities should be open for extension, but closed for modification.**

- **Open for Extension**: The behavior of the entity can be extended. As new requirements come in, you should be able to add new behavior without touching existing code.

- **Closed for Modification**: The existing, working code should not be changed. Once it is written, tested, and working, you should not need to go back and alter it to add new features.

## Implementing OCP

### Step 1: Define an Interface

<details>
<summary>Python</summary>

```python
class PaymentMethod(ABC):
    @abstractmethod
    def process_payment(self, amount):
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentMethod {
public:
    virtual void processPayment(double amount) = 0;
    virtual ~PaymentMethod() = default;
};
```

</details>
<br/>

### Step 2: Implement Concrete Strategies

<details>
<summary>Python</summary>

```python
class CreditCardPayment(PaymentMethod):
    def process_payment(self, amount):
        print(f"Processing credit card payment of ${amount}")
        # Complex logic for credit card processing

class PayPalPayment(PaymentMethod):
    def process_payment(self, amount):
        print(f"Processing PayPal payment of ${amount}")
        # Logic for PayPal processing

class UPIPayment(PaymentMethod):
    def process_payment(self, amount):
        print(f"Processing UPI payment of ₹{amount * 80}")  # Assuming conversion rate
        # Logic for UPI processing
```

</details>

<details>
<summary>C++</summary>

```cpp
class CreditCardPayment : public PaymentMethod {
public:
    void processPayment(double amount) override {
        cout << "Processing credit card payment of $" << amount << endl;
        // Complex logic for credit card processing
    }
};

class PayPalPayment : public PaymentMethod {
public:
    void processPayment(double amount) override {
        cout << "Processing PayPal payment of $" << amount << endl;
        // Logic for PayPal processing
    }
};

class UPIPayment : public PaymentMethod {
public:
    void processPayment(double amount) override {
        cout << "Processing UPI payment of ₹" << amount * 80 << endl;
        // Logic for UPI processing
    }
};
```

</details>
<br/>

### Step 3: Modify the PaymentProcessor to Use the Abstraction

Our PaymentProcessor now depends on the PaymentMethod interface, not concrete implementations. It no longer needs to know the specifics of each payment type.

<details>
<summary>Python</summary>

```python
class PaymentProcessor:
    def process(self, payment_method: PaymentMethod, amount):
        # No more if-else! The processor doesn't care about the specific type.
        # It just knows it can call processPayment.
        payment_method.process_payment(amount)
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentProcessor {
public:
    void process(PaymentMethod* paymentMethod, double amount) {
        // No more if-else! The processor doesn't care about the specific type.
        // It just knows it can call processPayment.
        paymentMethod->processPayment(amount);
    }
};
```

</details>
<br/>

### Step 4: Final Checkout Service Implementation

<details>
<summary>Python</summary>

```python
class CheckoutService:
    def process_payment(self, method: PaymentMethod, amount):
        processor = PaymentProcessor()
        processor.process(method, amount)

# Usage
checkout = CheckoutService()
checkout.process_payment(CreditCardPayment(), 100.00)
checkout.process_payment(PayPalPayment(), 100.00)
checkout.process_payment(UPIPayment(), 100.00)
```

</details>

<details>
<summary>C++</summary>

```cpp
class CheckoutService {
public:
    void processPayment(PaymentMethod* method, double amount) {
        PaymentProcessor processor;
        processor.process(method, amount);
    }
};

// Usage
CheckoutService checkout;
CreditCardPayment credit;
PayPalPayment paypal;
UPIPayment upi;

checkout.processPayment(&credit, 100.00);
checkout.processPayment(&paypal, 100.00);
checkout.processPayment(&upi, 100.00);
```

</details>
<br/>

---

# Chapter 3 Liskov Substitution Principle (LSP)

**If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of that program**

## The Problem: A Document System Gone Wrong

<details>
<summary>Python</summary>

```python
class Document:
    def __init__(self, data):
        self.data = data

    def open(self):
        print("Document opened. Data:", self.data[:20] + "...")

    def save(self, new_data):
        self.data = new_data
        print("Document saved.")

    def get_data(self):
        return self.data

class ReadOnlyDocument(Document):
    def __init__(self, data):
        super().__init__(data)

    def save(self, new_data):
        raise UnsupportedOperationException("Cannot save a read-only document!")


class DocumentProcessor:
    def process_and_save(self, doc, additional_info):
        doc.open()
        current_data = doc.get_data()
        new_data = current_data + " | Processed: " + additional_info
        doc.save(new_data)  # Assumes all Documents are savable
        print("Document processing complete.")

if __name__ == "__main__":
    regular_doc = Document("Initial project proposal content.")
    confidential_report = ReadOnlyDocument("Top secret government data.")

    processor = DocumentProcessor()

    print("--- Processing Regular Document ---")
    processor.process_and_save(regular_doc, "Reviewed by Alice")

    print("\n--- Processing ReadOnly Document ---")
    try:
        processor.process_and_save(confidential_report, "Reviewed by Bob")
    except UnsupportedOperationException as e:
        print("Error:", str(e))
```

</details>

<details>
<summary>C++</summary>

```cpp
class Document {
protected:
    string data;

public:
    Document(const string& data) : data(data) {}

    virtual void open() const {
        cout << "Document opened. Data: " << data.substr(0, min((size_t)20, data.length())) << "..." << endl;
    }

    virtual void save(const string& newData) {
        data = newData;
        cout << "Document saved." << endl;
    }

    string getData() const {
        return data;
    }

    virtual ~Document() = default;
};


class ReadOnlyDocument : public Document {
public:
    ReadOnlyDocument(const string& data) : Document(data) {}

    void save(const string& /*newData*/) override {
        throw runtime_error("Cannot save a read-only document!");
    }
};


class DocumentProcessor {
public:
    void processAndSave(Document* doc, const string& additionalInfo) {
        doc->open();
        string currentData = doc->getData();
        string newData = currentData + " | Processed: " + additionalInfo;
        doc->save(newData); // Assumes all Documents are savable
        cout << "Document processing complete." << endl;
    }
};

int main() {
    Document* regularDoc = new Document("Initial project proposal content.");
    Document* confidentialReport = new ReadOnlyDocument("Top secret government data.");

    DocumentProcessor processor;

    cout << "--- Processing Regular Document ---" << endl;
    processor.processAndSave(regularDoc, "Reviewed by Alice");

    cout << "\n--- Processing ReadOnly Document ---" << endl;
    try {
        processor.processAndSave(confidentialReport, "Reviewed by Bob");
    } catch (const exception& e) {
        cerr << "Error: " << e.what() << endl;
    }

    delete regularDoc;
    delete confidentialReport;

    return 0;
}
```

</details>
<br/>

**Output**

```text
--- Processing Regular Document ---
Document opened. Data: Initial project proposal content.
Document saved.
Document processing complete.

--- Processing ReadOnly Document ---
Document opened. Data: Top secret government data.
Error: Cannot save a read-only document!
```

## Implementing LSP

Let’s refactor our design so that subtypes like `ReadOnlyDocument`

### Step 1: Define Behavior Interfaces

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class Document(ABC):
    @abstractmethod
    def open(self):
        pass

    @abstractmethod
    def get_data(self):
        pass

class Editable(Document):
    @abstractmethod
    def save(self, new_data):
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class Document {
public:
    virtual void open() const = 0;
    virtual string getData() const = 0;
    virtual ~Document() = default;
};

class Editable : public Document {
public:
    virtual void save(const string& newData) = 0;
};
```

</details>
<br/>

### Step 2: Implement EditableDocument and ReadOnlyDocument

<details>
<summary>Python</summary>

```python
class EditableDocument(Editable):
    def __init__(self, data):
        self.data = data

    def open(self):
        print("Editable Document opened. Data:", self._preview())

    def save(self, new_data):
        self.data = new_data
        print("Document saved.")

    def get_data(self):
        return self.data

    def _preview(self):
        return self.data[:20] + "..."


class ReadOnlyDocument(Document):
    def __init__(self, data):
        self.data = data

    def open(self):
        print("Read-Only Document opened. Data:", self._preview())

    def get_data(self):
        return self.data

    def _preview(self):
        return self.data[:20] + "..."
```

</details>

<details>
<summary>C++</summary>

```cpp
class EditableDocument : public Editable {
private:
    string data;

    string preview() const {
        return data.substr(0, min((size_t)20, data.length())) + "...";
    }

public:
    EditableDocument(const string& data) : data(data) {}

    void open() const override {
        cout << "Editable Document opened. Data: " << preview() << endl;
    }

    void save(const string& newData) override {
        data = newData;
        cout << "Document saved." << endl;
    }

    string getData() const override {
        return data;
    }
};


class ReadOnlyDocument : public Document {
private:
    string data;

    string preview() const {
        return data.substr(0, min((size_t)20, data.length())) + "...";
    }

public:
    ReadOnlyDocument(const string& data) : data(data) {}

    void open() const override {
        cout << "Read-Only Document opened. Data: " << preview() << endl;
    }

    string getData() const override {
        return data;
    }
};
```

</details>
<br/>

![text11](/assets/11.png)

### Step 3: Refactor the Client Code

The `processor` has two methods: `process()` accepts any Document for reading, and `processAndSave()` accepts an Editable, which guarantees both reading and writing capabilities. If you try to pass a `ReadOnlyDocument` to `processAndSave()`, the compiler rejects it before the program ever runs.

<details>
<summary>Python</summary>

```python
class DocumentProcessor:
    def process(self, doc: Document):
        doc.open()
        print("Document processed.")

    def process_and_save(self, doc: Editable, additional_info: str):
        doc.open()
        current_data = doc.get_data()
        new_data = current_data + " | Processed: " + additional_info
        doc.save(new_data)
        print("Editable document processed and saved.")

if __name__ == "__main__":
    editable = EditableDocument("Draft proposal for Q3.")
    read_only = ReadOnlyDocument("Top secret strategy.")

    processor = DocumentProcessor()

    print("--- Processing Editable Document ---")
    processor.process_and_save(editable, "Reviewed by Alice")

    print("\n--- Processing Read-Only Document ---")
    processor.process(read_only)  # This works fine
```

</details>

<details>
<summary>C++</summary>

```cpp
class DocumentProcessor {
public:
    void process(const Document* doc) const {
        doc->open();
        cout << "Document processed." << endl;
    }

    void processAndSave(Editable& doc, const string& additionalInfo) {
        doc.open();
        string currentData = doc.getData();
        string newData = currentData + " | Processed: " + additionalInfo;
        doc.save(newData);
        cout << "Editable document processed and saved." << endl;
    }
};

int main() {
    EditableDocument editable("Draft proposal for Q3.");
    ReadOnlyDocument readOnly("Top secret strategy.");
    DocumentProcessor processor;

    cout << "--- Processing Editable Document ---" << endl;
    processor.processAndSave(editable, "Reviewed by Alice");

    cout << "\n--- Processing Read-Only Document ---" << endl;
    processor.process(&readOnly); // Works fine

    // processor.processAndSave(readOnly, "Reviewed by Bob");
    // Won't compile! ReadOnlyDocument doesn't have save().

    return 0;
}
```

</details>
<br/>

<br/>

**The “Is-A” Linguistic Trap**:
Just because something sounds like it "is a" something else in natural language does not mean it is a valid subtype in code.

Take this classic example:

> A penguin is a bird, but penguins can’t fly.If your Bird class has a fly() method, and you override it in Penguin to throw an exception or do nothing—you've violated LSP.

## Exercise 2: The Bird/Penguin Problem

Problem: A `Bird` class has both `eat()` and `fly()` methods. A Penguin subclass extends `Bird` but overrides `fly()` to throw an exception, since penguins cannot fly. Any client code that calls `fly()` on a Bird reference will crash at runtime when it gets a `Penguin`.

<details>
<summary>Python</summary>

```python
# Before: Penguin extends Bird but can't fly
class Bird:
    def eat(self):
        print(f"{self.__class__.__name__} is eating")

    def fly(self):
        print(f"{self.__class__.__name__} is flying")

class Sparrow(Bird):
    pass

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins can't fly!")

def make_bird_fly(bird):
    bird.fly()  # Crashes for Penguin!

make_bird_fly(Sparrow())  # Works fine
make_bird_fly(Penguin())  # NotImplementedError!

# TODO: Split Bird into a Bird ABC (eat) and a FlyingBird ABC (fly).
# TODO: Sparrow implements FlyingBird, Penguin implements only Bird.
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

// Before: Penguin extends Bird but can't fly
class Bird {
public:
    virtual void eat() {
        std::cout << "Bird is eating" << std::endl;
    }

    virtual void fly() {
        std::cout << "Bird is flying" << std::endl;
    }

    virtual ~Bird() = default;
};

class Sparrow : public Bird {
public:
    void eat() override { std::cout << "Sparrow is eating" << std::endl; }
    void fly() override { std::cout << "Sparrow is flying" << std::endl; }
};

class Penguin : public Bird {
public:
    void eat() override { std::cout << "Penguin is eating" << std::endl; }
    void fly() override {
        throw std::runtime_error("Penguins can't fly!");
    }
};

void makeBirdFly(Bird& bird) {
    bird.fly(); // Crashes for Penguin!
}

int main() {
    Sparrow sparrow;
    Penguin penguin;
    makeBirdFly(sparrow); // Works fine
    makeBirdFly(penguin); // runtime_error!
    return 0;
}

// TODO: Split Bird into a Bird interface (eat) and a FlyingBird interface (fly).
// TODO: Sparrow implements FlyingBird, Penguin implements only Bird.
```

</details>
<br/>

**Solution**:

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod


class Bird(ABC):
    @abstractmethod
    def eat(self) -> None:
        pass


class FlyingBird(Bird):
    @abstractmethod
    def fly(self) -> None:
        pass


class Sparrow(FlyingBird):
    def eat(self) -> None:
        print("Sparrow is eating")

    def fly(self) -> None:
        print("Sparrow is flying")


class Penguin(Bird):
    def eat(self) -> None:
        print("Penguin is eating")


if __name__ == "__main__":
    sparrow = Sparrow()
    sparrow.eat()
    sparrow.fly()

    penguin = Penguin()
    penguin.eat()
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
using namespace std;

class Bird {
public:
    virtual void eat() = 0;
    virtual ~Bird() = default;
};

class FlyingBird : public Bird {
public:
    virtual void fly() = 0;
};

class Sparrow : public FlyingBird {
public:
    void eat() override {
        cout << "Sparrow is eating" << endl;
    }

    void fly() override {
        cout << "Sparrow is flying" << endl;
    }
};

class Penguin : public Bird {
public:
    void eat() override {
        cout << "Penguin is eating" << endl;
    }
};

int main() {
    Sparrow sparrow;
    sparrow.eat();
    sparrow.fly();

    Penguin penguin;
    penguin.eat();

    return 0;
}
```

</details>
<br/>

---


# Chapter 4 Interface Segregation Principle (ISP)

**Clients should not be forced to depend on methods they do not use.**  Each interface should represent a specific capability or behavior. If a class doesn’t need a method, it shouldn’t be forced to implement it.



## The Problem: A Fat Interface

Imagine you are building a media player app that supports different types of media:

- Audio files (MP3, WAV)
- Video files (MP4, AVI)

You might start with what feels like a convenient design: a single, unified interface that handles everything.


<details>
<summary>Python</summary>

```python
class MediaPlayer(ABC):
    @abstractmethod
    def play_audio(self, audio_file):
        pass

    @abstractmethod
    def stop_audio(self):
        pass

    @abstractmethod
    def adjust_audio_volume(self, volume):
        pass

    @abstractmethod
    def play_video(self, video_file):
        pass

    @abstractmethod
    def stop_video(self):
        pass

    @abstractmethod
    def adjust_video_brightness(self, brightness):
        pass

    @abstractmethod
    def display_subtitles(self, subtitle_file):
        pass

class AudioOnlyPlayer(MediaPlayer):
    def play_audio(self, audio_file):
        print(f"Playing audio file: {audio_file}")

    def stop_audio(self):
        print("Audio stopped.")

    def adjust_audio_volume(self, volume):
        print(f"Audio volume set to: {volume}")

    # Unwanted methods
    def play_video(self, video_file):
        raise NotImplementedError("Not supported.")

    def stop_video(self):
        raise NotImplementedError("Not supported.")

    def adjust_video_brightness(self, brightness):
        raise NotImplementedError("Not supported.")

    def display_subtitles(self, subtitle_file):
        raise NotImplementedError("Not supported.")
```

</details>

<details>
<summary>C++</summary>

```cpp
class MediaPlayer {
public:
    virtual void playAudio(const string& audioFile) = 0;
    virtual void stopAudio() = 0;
    virtual void adjustAudioVolume(int volume) = 0;

    virtual void playVideo(const string& videoFile) = 0;
    virtual void stopVideo() = 0;
    virtual void adjustVideoBrightness(int brightness) = 0;
    virtual void displaySubtitles(const string& subtitleFile) = 0;

    virtual ~MediaPlayer() = default;
};


class AudioOnlyPlayer : public MediaPlayer {
public:
    void playAudio(const string& audioFile) override {
        cout << "Playing audio file: " << audioFile << endl;
    }

    void stopAudio() override {
        cout << "Audio stopped." << endl;
    }

    void adjustAudioVolume(int volume) override {
        cout << "Audio volume set to: " << volume << endl;
    }

    // Unwanted methods forced by the interface
    void playVideo(const string& /*videoFile*/) override {
        throw runtime_error("Not supported.");
    }

    void stopVideo() override {
        // no-op
    }

    void adjustVideoBrightness(int /*brightness*/) override {
        throw runtime_error("Not supported.");
    }

    void displaySubtitles(const string& /*subtitleFile*/) override {
        throw runtime_error("Not supported.");
    }
};
```

</details>
<br/>


### What’s Wrong With This?
#### Interface Pollution
The MediaPlayer interface is doing too much. It combines multiple unrelated responsibilities.

Any class that implements this interface must carry the weight of all seven methods, even when it only needs three. 

#### Fragile Code
Now, imagine you add a new method to the interface, like `enablePictureInPicture()`. Suddenly, all existing implementations, audio-only, video-only, or otherwise, must be updated.

This tight coupling slows you down and increases the risk of bugs. One method addition forces changes across files that have nothing to do with picture-in-picture mode.

#### Violates Liskov Substitution
A client may expect any `MediaPlayer` to support video, but passing in an `AudioOnlyPlayer` will crash the program with an `UnsupportedOperationException`.



## Why Does ISP Matter?
- #### Increased Cohesion, Reduced Coupling
- #### Improved Flexibility & Reusability
- #### Avoids "Interface Pollution" and LSP Violations



## Applying ISP

### Step 1: Define Smaller, Cohesive Interfaces
Instead of one bloated MediaPlayer interface, we’ll create multiple focused ones:


<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

# Audio-only capabilities
class AudioPlayerControls(ABC):
    @abstractmethod
    def play_audio(self, audio_file):
        pass

    @abstractmethod
    def stop_audio(self):
        pass

    @abstractmethod
    def adjust_audio_volume(self, volume):
        pass

# Video-only capabilities
class VideoPlayerControls(ABC):
    @abstractmethod
    def play_video(self, video_file):
        pass

    @abstractmethod
    def stop_video(self):
        pass

    @abstractmethod
    def adjust_video_brightness(self, brightness):
        pass

    @abstractmethod
    def display_subtitles(self, subtitle_file):
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
// Audio-only capabilities
class AudioPlayerControls {
public:
    virtual void playAudio(const string& audioFile) = 0;
    virtual void stopAudio() = 0;
    virtual void adjustAudioVolume(int volume) = 0;
    virtual ~AudioPlayerControls() = default;
};

// Video-only capabilities
class VideoPlayerControls {
public:
    virtual void playVideo(const string& videoFile) = 0;
    virtual void stopVideo() = 0;
    virtual void adjustVideoBrightness(int brightness) = 0;
    virtual void displaySubtitles(const string& subtitleFile) = 0;
    virtual ~VideoPlayerControls() = default;
};

```

</details>
<br/>



### Step 2: Classes Implement Only the Interfaces They Need
Now our specific player classes can implement only the relevant interfaces.

<details>
<summary>Python</summary>

```python
class ModernAudioPlayer(AudioPlayerControls):
    def play_audio(self, audio_file):
        print(f"ModernAudioPlayer: Playing audio - {audio_file}")

    def stop_audio(self):
        print("ModernAudioPlayer: Audio stopped.")

    def adjust_audio_volume(self, volume):
        print(f"ModernAudioPlayer: Volume set to {volume}")

class SilentVideoPlayer(VideoPlayerControls):
    def play_video(self, video_file):
        print(f"SilentVideoPlayer: Playing video - {video_file}")

    def stop_video(self):
        print("SilentVideoPlayer: Video stopped.")

    def adjust_video_brightness(self, brightness):
        print(f"SilentVideoPlayer: Brightness set to {brightness}")

    def display_subtitles(self, subtitle_file):
        print(f"SilentVideoPlayer: Subtitles from {subtitle_file}")

# ComprehensiveMediaPlayer (Both audio + video)

class ComprehensiveMediaPlayer(AudioPlayerControls, VideoPlayerControls):
    def play_audio(self, audio_file):
        print(f"ComprehensiveMediaPlayer: Playing audio - {audio_file}")

    def stop_audio(self):
        print("ComprehensiveMediaPlayer: Audio stopped.")

    def adjust_audio_volume(self, volume):
        print(f"ComprehensiveMediaPlayer: Audio volume set to {volume}")

    def play_video(self, video_file):
        print(f"ComprehensiveMediaPlayer: Playing video - {video_file}")

    def stop_video(self):
        print("ComprehensiveMediaPlayer: Video stopped.")

    def adjust_video_brightness(self, brightness):
        print(f"ComprehensiveMediaPlayer: Brightness set to {brightness}")

    def display_subtitles(self, subtitle_file):
        print(f"ComprehensiveMediaPlayer: Subtitles from {subtitle_file}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class ModernAudioPlayer : public AudioPlayerControls {
public:
    void playAudio(const string& audioFile) override {
        cout << "ModernAudioPlayer: Playing audio - " << audioFile << endl;
    }

    void stopAudio() override {
        cout << "ModernAudioPlayer: Audio stopped." << endl;
    }

    void adjustAudioVolume(int volume) override {
        cout << "ModernAudioPlayer: Volume set to " << volume << endl;
    }
};

class SilentVideoPlayer : public VideoPlayerControls {
public:
    void playVideo(const string& videoFile) override {
        cout << "SilentVideoPlayer: Playing video - " << videoFile << endl;
    }

    void stopVideo() override {
        cout << "SilentVideoPlayer: Video stopped." << endl;
    }

    void adjustVideoBrightness(int brightness) override {
        cout << "SilentVideoPlayer: Brightness set to " << brightness << endl;
    }

    void displaySubtitles(const string& subtitleFile) override {
        cout << "SilentVideoPlayer: Subtitles from " << subtitleFile << endl;
    }
};

// ComprehensiveMediaPlayer (Both audio + video)

class ComprehensiveMediaPlayer : public AudioPlayerControls, public VideoPlayerControls {
public:
    void playAudio(const string& audioFile) override {
        cout << "ComprehensiveMediaPlayer: Playing audio - " << audioFile << endl;
    }

    void stopAudio() override {
        cout << "ComprehensiveMediaPlayer: Audio stopped." << endl;
    }

    void adjustAudioVolume(int volume) override {
        cout << "ComprehensiveMediaPlayer: Audio volume set to " << volume << endl;
    }

    void playVideo(const string& videoFile) override {
        cout << "ComprehensiveMediaPlayer: Playing video - " << videoFile << endl;
    }

    void stopVideo() override {
        cout << "ComprehensiveMediaPlayer: Video stopped." << endl;
    }

    void adjustVideoBrightness(int brightness) override {
        cout << "ComprehensiveMediaPlayer: Brightness set to " << brightness << endl;
    }

    void displaySubtitles(const string& subtitleFile) override {
        cout << "ComprehensiveMediaPlayer: Subtitles from " << subtitleFile << endl;
    }
};
```

</details>
<br/>

This is the essence of the Interface Segregation Principle in action. The interfaces are small, focused, and composable.


## Common Pitfalls While Applying ISP

- **Over-Segregation**: This is a problem because you end up with too many tiny interfaces that are hard to manage and understand. 

- **Lack of Cohesion**: The mistake is creating interfaces that are not tightly related, mixing unrelated methods together.

    Low cohesion makes interfaces confusing and hard to reason about. If an interface has methods that don't logically belong together, you have the same problem as a fat interface, just with a different name.

    Make sure every method in an interface relates to a single, well-defined responsibility. Think of your interface as a role. Would it make sense for all these actions to be part of that role?

## Exercise 1: Fat MultiFunctionDevice

**Refactor MultiFunctionDevice**

**Problem**: You have a `MultiFunctionDevice` interface with `print()`, `scan()`, `fax()`, and `staple()` methods. A BasicPrinter only prints. Refactor into separate `Printable`, `Scannable`, and `Faxable` interfaces so that `BasicPrinter` only implements the capabilities it actually supports.

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

# Before: Fat interface forces BasicPrinter to implement everything
class MultiFunctionDevice(ABC):
    @abstractmethod
    def print_doc(self, document):
        pass

    @abstractmethod
    def scan(self, document):
        pass

    @abstractmethod
    def fax(self, document, number):
        pass

    @abstractmethod
    def staple(self, document):
        pass

class BasicPrinter(MultiFunctionDevice):
    def print_doc(self, document):
        print(f"Printing: {document}")

    def scan(self, document):
        raise NotImplementedError("BasicPrinter cannot scan.")

    def fax(self, document, number):
        raise NotImplementedError("BasicPrinter cannot fax.")

    def staple(self, document):
        raise NotImplementedError("BasicPrinter cannot staple.")

if __name__ == "__main__":
    printer = BasicPrinter()
    printer.print_doc("report.pdf")

# TODO: Create Printable, Scannable, Faxable, and Stapleable interfaces.
# TODO: Refactor BasicPrinter to implement only Printable.
# TODO: Create an OfficePrinter that implements Printable, Scannable, and Faxable.
# TODO: Create a FullDevice that implements all four interfaces.
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

using namespace std;

// Before: Fat interface forces BasicPrinter to implement everything
class MultiFunctionDevice {
public:
    virtual void print(const string& document) = 0;
    virtual void scan(const string& document) = 0;
    virtual void fax(const string& document, const string& number) = 0;
    virtual void staple(const string& document) = 0;
    virtual ~MultiFunctionDevice() = default;
};

class BasicPrinter : public MultiFunctionDevice {
public:
    void print(const string& document) override {
        cout << "Printing: " << document << endl;
    }

    void scan(const string& /*document*/) override {
        throw runtime_error("BasicPrinter cannot scan.");
    }

    void fax(const string& /*document*/, const string& /*number*/) override {
        throw runtime_error("BasicPrinter cannot fax.");
    }

    void staple(const string& /*document*/) override {
        throw runtime_error("BasicPrinter cannot staple.");
    }
};

int main() {
    BasicPrinter printer;
    printer.print("report.pdf");
    return 0;
}

// TODO: Create Printable, Scannable, Faxable, and Stapleable interfaces.
// TODO: Refactor BasicPrinter to implement only Printable.
// TODO: Create an OfficePrinter that implements Printable, Scannable, and Faxable.
// TODO: Create a FullDevice that implements all four interfaces.
```

</details>
<br/>

#### Solution

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

# Segregated interfaces - each with a single capability
class Printable(ABC):
    @abstractmethod
    def print_doc(self, document):
        pass

class Scannable(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class Faxable(ABC):
    @abstractmethod
    def fax(self, document, number):
        pass

class Stapleable(ABC):
    @abstractmethod
    def staple(self, document):
        pass

# BasicPrinter only implements Printable
class BasicPrinter(Printable):
    def print_doc(self, document):
        print(f"BasicPrinter -> Printing: {document}")

# OfficePrinter implements three interfaces
class OfficePrinter(Printable, Scannable, Faxable):
    def print_doc(self, document):
        print(f"OfficePrinter -> Printing: {document}")

    def scan(self, document):
        print(f"OfficePrinter -> Scanning: {document}")

    def fax(self, document, number):
        print(f"OfficePrinter -> Faxing: {document} to {number}")

# FullDevice implements all four interfaces
class FullDevice(Printable, Scannable, Faxable, Stapleable):
    def print_doc(self, document):
        print(f"FullDevice -> Printing: {document}")

    def scan(self, document):
        print(f"FullDevice -> Scanning: {document}")

    def fax(self, document, number):
        print(f"FullDevice -> Faxing: {document} to {number}")

    def staple(self, document):
        print(f"FullDevice -> Stapling: {document}")

if __name__ == "__main__":
    basic = BasicPrinter()
    basic.print_doc("report.pdf")

    office = OfficePrinter()
    office.print_doc("memo.pdf")
    office.scan("memo.pdf")
    office.fax("memo.pdf", "555-1234")

    full = FullDevice()
    full.print_doc("contract.pdf")
    full.scan("contract.pdf")
    full.fax("contract.pdf", "555-5678")
    full.staple("contract.pdf")
```

</details>

<details>
<summary>C++</summary>

```cpp
class Printable {
public:
    virtual void print(const string& document) = 0;
    virtual ~Printable() = default;
};

class Scannable {
public:
    virtual void scan(const string& document) = 0;
    virtual ~Scannable() = default;
};

class Faxable {
public:
    virtual void fax(const string& document, const string& number) = 0;
    virtual ~Faxable() = default;
};

class Stapleable {
public:
    virtual void staple(const string& document) = 0;
    virtual ~Stapleable() = default;
};

// BasicPrinter only implements Printable
class BasicPrinter : public Printable {
public:
    void print(const string& document) override {
        cout << "BasicPrinter -> Printing: " << document << endl;
    }
};

// OfficePrinter implements three interfaces
class OfficePrinter : public Printable, public Scannable, public Faxable {
public:
    void print(const string& document) override {
        cout << "OfficePrinter -> Printing: " << document << endl;
    }

    void scan(const string& document) override {
        cout << "OfficePrinter -> Scanning: " << document << endl;
    }

    void fax(const string& document, const string& number) override {
        cout << "OfficePrinter -> Faxing: " << document << " to " << number << endl;
    }
};

// FullDevice implements all four interfaces
class FullDevice : public Printable, public Scannable, public Faxable, public Stapleable {
public:
    void print(const string& document) override {
        cout << "FullDevice -> Printing: " << document << endl;
    }

    void scan(const string& document) override {
        cout << "FullDevice -> Scanning: " << document << endl;
    }

    void fax(const string& document, const string& number) override {
        cout << "FullDevice -> Faxing: " << document << " to " << number << endl;
    }

    void staple(const string& document) override {
        cout << "FullDevice -> Stapling: " << document << endl;
    }
};

int main() {
    BasicPrinter basic;
    basic.print("report.pdf");

    OfficePrinter office;
    office.print("memo.pdf");
    office.scan("memo.pdf");
    office.fax("memo.pdf", "555-1234");

    FullDevice full;
    full.print("contract.pdf");
    full.scan("contract.pdf");
    full.fax("contract.pdf", "555-5678");
    full.staple("contract.pdf");

    return 0;
}
```

</details>
<br/>



---


# Chapter 5 Dependency Inversion Principle (DIP)


## The Problem: A Tightly Coupled EmailService

Imagine you are building an `EmailService`. Your first task is to send emails using Gmail. So you write something like this.

Here is the low-level module, a `GmailClient` that knows how to talk to Gmail's servers.


<details>
<summary>Python</summary>

```python
class GmailClient:
    def send_gmail(self, to_address, subject_line, email_body):
        print("Connecting to Gmail SMTP server...")
        print(f"Sending email via Gmail to: {to_address}")
        print(f"Subject: {subject_line}")
        print(f"Body: {email_body}")
        # ... actual Gmail API interaction logic ...
        print("Gmail email sent successfully!")

## And here is the high-level module, the EmailService that handles business logic like sending welcome emails and password resets.

class EmailService:
    def __init__(self):
        self.gmail_client = GmailClient()

    def send_welcome_email(self, user_email, user_name):
        subject = f"Welcome, {user_name}!"
        body = "Thanks for signing up to our awesome platform. We're glad to have you!"
        self.gmail_client.send_gmail(user_email, subject, body)

    def send_password_reset_email(self, user_email):
        subject = "Reset Your Password"
        body = "Please click the link below to reset your password..."
        self.gmail_client.send_gmail(user_email, subject, body)
```

</details>

<details>
<summary>C++</summary>

```cpp
class GmailClient {
public:
    void sendGmail(const string& toAddress, const string& subjectLine, const string& emailBody) {
        cout << "Connecting to Gmail SMTP server..." << endl;
        cout << "Sending email via Gmail to: " << toAddress << endl;
        cout << "Subject: " << subjectLine << endl;
        cout << "Body: " << emailBody << endl;
        // ... actual Gmail API interaction logic ...
        cout << "Gmail email sent successfully!" << endl;
    }
};

// And here is the high-level module, the EmailService that handles business logic like sending welcome emails and password resets.

class EmailService {
private:
    GmailClient gmailClient;

public:
    void sendWelcomeEmail(const string& userEmail, const string& userName) {
        string subject = "Welcome, " + userName + "!";
        string body = "Thanks for signing up to our awesome platform. We're glad to have you!";
        gmailClient.sendGmail(userEmail, subject, body);
    }

    void sendPasswordResetEmail(const string& userEmail) {
        string subject = "Reset Your Password";
        string body = "Please click the link below to reset your password...";
        gmailClient.sendGmail(userEmail, subject, body);
    }
};
```

</details>
<br/>



## The Dependency Inversion Principle

- #### High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., interfaces).

- #### Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.


You might wonder what exactly is being "inverted." It is the direction of dependency. Without DIP, high-level modules depend directly on low-level modules. With DIP, both the high-level module and the low-level module depend on a shared abstraction (an interface or abstract class).

The control flow might still go from high to low, but the source code dependency is inverted. High-level modules define what they need (the contract/interface), and low-level modules provide the how (the implementation of that interface).


### Why Does DIP Matter?
- Decoupling
- Flexibility and Extensibility
- Parallel Development

## Applying DIP

### Step 1: Define the Abstraction (The Contract)

<details>
<summary>Python</summary>

```python
class EmailClient(ABC):
    @abstractmethod
    def send_email(self, to, subject, body):
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class EmailClient {
public:
    virtual void sendEmail(const string& to, const string& subject, const string& body) = 0;
    virtual ~EmailClient() = default;
};
```

</details>
<br/>


### Step 2: Concrete Implementations

<details>
<summary>Python</summary>

```python
class GmailClientImpl(EmailClient):
    def send_email(self, to, subject, body):
        print("Connecting to Gmail SMTP server...")
        print(f"Sending email via Gmail to: {to}")
        print(f"Subject: {subject}")
        print(f"Body: {body}")
        # ... actual Gmail API interaction logic ...
        print("Gmail email sent successfully!")

class OutlookClientImpl(EmailClient):
    def send_email(self, to, subject, body):
        print("Connecting to Outlook Exchange server...")
        print(f"Sending email via Outlook to: {to}")
        print(f"Subject: {subject}")
        print(f"Body: {body}")
        # ... actual Outlook API interaction logic ...
        print("Outlook email sent successfully!")
```

</details>

<details>
<summary>C++</summary>

```cpp
class GmailClientImpl : public EmailClient {
public:
    void sendEmail(const string& to, const string& subject, const string& body) override {
        cout << "Connecting to Gmail SMTP server..." << endl;
        cout << "Sending email via Gmail to: " << to << endl;
        cout << "Subject: " << subject << endl;
        cout << "Body: " << body << endl;
        cout << "Gmail email sent successfully!" << endl;
    }
};

class OutlookClientImpl : public EmailClient {
public:
    void sendEmail(const string& to, const string& subject, const string& body) override {
        cout << "Connecting to Outlook Exchange server..." << endl;
        cout << "Sending email via Outlook to: " << to << endl;
        cout << "Subject: " << subject << endl;
        cout << "Body: " << body << endl;
        cout << "Outlook email sent successfully!" << endl;
    }
};
```

</details>
<br/>


### Step 3: Update the High-Level Module

Now comes the key change. Our `EmailService` will no longer know about `GmailClientImpl` or `OutlookClientImpl`. It will only know about the `EmailClient` interface. The actual implementation gets "injected" into it from the outside.


<details>
<summary>Python</summary>

```python
class EmailService:
    def __init__(self, email_client: EmailClient):
        self.email_client = email_client

    def send_welcome_email(self, user_email, user_name):
        subject = f"Welcome, {user_name}!"
        body = "Thanks for signing up to our awesome platform. We're glad to have you!"
        self.email_client.send_email(user_email, subject, body)

    def send_password_reset_email(self, user_email):
        subject = "Reset Your Password"
        body = "Please click the link below to reset your password..."
        self.email_client.send_email(user_email, subject, body)


if __name__ == "__main__":
    print("--- Using Gmail ---")
    gmail_service = EmailService(GmailClientImpl())
    gmail_service.send_welcome_email("test@example.com", "Alice")

    print("\n--- Using Outlook ---")
    outlook_service = EmailService(OutlookClientImpl())
    outlook_service.send_welcome_email("test@example.com", "Alice")
```

</details>

<details>
<summary>C++</summary>

```cpp
class EmailService {
private:
    shared_ptr<EmailClient> emailClient;

public:
    EmailService(shared_ptr<EmailClient> client) : emailClient(move(client)) {}

    void sendWelcomeEmail(const string& userEmail, const string& userName) {
        string subject = "Welcome, " + userName + "!";
        string body = "Thanks for signing up to our awesome platform. We're glad to have you!";
        emailClient->sendEmail(userEmail, subject, body);
    }

    void sendPasswordResetEmail(const string& userEmail) {
        string subject = "Reset Your Password";
        string body = "Please click the link below to reset your password...";
        emailClient->sendEmail(userEmail, subject, body);
    }
};


int main() {
    cout << "--- Using Gmail ---" << endl;
    shared_ptr<EmailClient> gmail = make_shared<GmailClientImpl>();
    EmailService gmailService(gmail);
    gmailService.sendWelcomeEmail("test@example.com", "Alice");

    cout << "\n--- Using Outlook ---" << endl;
    shared_ptr<EmailClient> outlook = make_shared<OutlookClientImpl>();
    EmailService outlookService(outlook);
    outlookService.sendWelcomeEmail("test@example.com", "Alice");

    return 0;
}
```

</details>
<br/>

## Common Pitfalls While Applying DIP
- **Over-Abstraction**

- **Leaky Abstractions**:  
    The mistake is exposing implementation-specific logic in your interface. For example, adding a method like configureGmailSpecificSetting() to the EmailClient interface defeats the entire purpose of the abstraction.

-  **Interfaces Owned by Low-Level Modules**:  
    The mistake is letting the low-level module define the interface it implements. For example, if GmailClient defines IGmailClient, and then EmailService depends on IGmailClient, the high-level module is still tied to the low-level module's namespace and structure.

    The abstraction should be defined by the high-level module (or in a neutral shared module), not by the implementation.


## Common Questions About DIP

### "Is DIP the same as Dependency Injection (DI)?"

Not exactly.

Dependency Inversion (DIP) is a principle:  “Depend on abstractions, not concrete implementations.”

Dependency Injection (DI) is a technique used to achieve DIP: You inject dependencies into a class (via constructor, setter, or method) instead of the class creating them itself.


### "Is DIP the same as Inversion of Control (IoC)?"

Nope, but they’re related.

Inversion of Control (IoC) is a broader design concept where the flow of control is inverted. Instead of your code calling libraries, a framework or container calls your code 

DIP is one specific way to achieve IoC — by inverting who depends on whom (high-level modules depend on abstractions, not implementations).


