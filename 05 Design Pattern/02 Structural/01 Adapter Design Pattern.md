# Adapter Design Pattern

The Adapter Design Pattern is a structural design pattern that allows two incompatible interfaces to work together. It acts as a bridge by wrapping an existing class (called the `Adaptee`) and converting its interface into one that the client expects (the `Target interface`). **This pattern is useful when integrating legacy code, third-party libraries, or existing classes without modifying their source code. It promotes code reusability and flexibility by enabling otherwise incompatible components to communicate seamlessly.**

- You need to **bridge the gap between new and old code**, or between systems built with different interface designs.

When faced with incompatible interfaces, developers often resort to rewriting large parts of code or embedding conditionals like `if (legacyType)` to handle special cases. But as more incompatible services or modules are introduced, this approach quickly becomes messy, `tightly coupled, and violates the Open/Closed Principle` making the system hard to scale or refactor.



## 1. The Problem: Incompatible Payment Interfaces
Imagine you are building the checkout component of an e-commerce application. Your checkout service is designed to work with a `PaymentProcessor` interface for handling payments.

### The Expected Interface
Here’s the contract your `CheckoutService` expects any payment provider to follow:

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float, currency: str):
        pass

    @abstractmethod
    def is_payment_successful(self) -> bool:
        pass

    @abstractmethod
    def get_transaction_id(self) -> str:
        pass
```

</details>

<details>
<summary>C++</summary>

```cpp
class PaymentProcessor {
public:
   virtual void processPayment(double amount, string currency) = 0;
   virtual bool isPaymentSuccessful() = 0;
   virtual string getTransactionId() = 0;
   virtual ~PaymentProcessor() {}
};
```

</details>
<br/>


This abstraction makes it easy to swap payment providers without changing any core business logic.

### Your In-House Implementation
Your team already has an internal payment processor that fits this interface perfectly:

<details>
<summary>Python</summary>

```python
import time

class InHousePaymentProcessor(PaymentProcessor):
    def __init__(self):
        self._transaction_id = None
        self._payment_successful = False

    def process_payment(self, amount: float, currency: str):
        print(f"InHouseProcessor: Processing {amount} {currency}")
        self._transaction_id = f"TXN_{int(time.time() * 1000)}"
        self._payment_successful = True
        print(f"InHouseProcessor: Success. Txn ID: {self._transaction_id}")

    def is_payment_successful(self) -> bool:
        return self._payment_successful

    def get_transaction_id(self) -> str:
        return self._transaction_id
```

</details>

<details>
<summary>C++</summary>

```cpp
class InHousePaymentProcessor : public PaymentProcessor {
private:
    string transactionId;
    bool paymentSuccessful = false;

public:
    void processPayment(double amount, string currency) override {
        cout << "InHouseProcessor: Processing " << amount << " " << currency << endl;
        auto now = chrono::duration_cast<chrono::milliseconds>(
            chrono::system_clock::now().time_since_epoch()).count();
        transactionId = "TXN_" + to_string(now);
        paymentSuccessful = true;
        cout << "InHouseProcessor: Success. Txn ID: " << transactionId << endl;
    }

    bool isPaymentSuccessful() override {
        return paymentSuccessful;
    }

    string getTransactionId() override {
        return transactionId;
    }
};
```

</details>
<br/>



Your CheckoutService uses this interface and works beautifully with the in-house payment processor:


<details>
<summary>Python</summary>

```python
class CheckoutService:
    def __init__(self, payment_processor: PaymentProcessor):
        self._processor = payment_processor

    def checkout(self, amount: float, currency: str):
        print(f"Checkout: Processing order for ${amount} {currency}")
        self._processor.process_payment(amount, currency)
        if self._processor.is_payment_successful():
            print(f"Checkout: Order successful! Txn: {self._processor.get_transaction_id()}")
        else:
            print("Checkout: Order failed.")

if __name__ == "__main__":
    processor = InHousePaymentProcessor()
    checkout = CheckoutService(processor)
    checkout.checkout(199.99, "USD")
```

</details>

<details>
<summary>C++</summary>

```cpp
class CheckoutService {
private:
    PaymentProcessor* paymentProcessor;

public:
    CheckoutService(PaymentProcessor* processor) : paymentProcessor(processor) {}

    void checkout(double amount, std::string currency) {
        std::cout << "Checkout: Processing order for $" << amount << " " << currency << std::endl;
        paymentProcessor->processPayment(amount, currency);
        if (paymentProcessor->isPaymentSuccessful()) {
            std::cout << "Checkout: Order successful! Txn: "
                      << paymentProcessor->getTransactionId() << std::endl;
        } else {
            std::cout << "Checkout: Order failed." << std::endl;
        }
    }
};

int main() {
    InHousePaymentProcessor processor;
    CheckoutService checkout(&processor);
    checkout.checkout(199.99, "USD");
    return 0;
}
```

</details>
<br/>


Everything works smoothly. You’ve decoupled your checkout business logic from the underlying payment implementation, allowing future flexibility. Great job so far.

Now management drops a new requirement: integrate with a legacy third-party payment provider. 

### The Incompatible Legacy Gateway
Here’s what that legacy payment class looks like:


<details>
<summary>Python</summary>

```python
class LegacyGateway:
    def __init__(self):
        self._transaction_reference = None
        self._payment_successful = False

    def execute_transaction(self, total_amount: float, currency: str):
        print(f"LegacyGateway: Executing {currency} {total_amount}")
        self._transaction_reference = time.time_ns()
        self._payment_successful = True
        print(f"LegacyGateway: Done. Ref: {self._transaction_reference}")

    def check_status(self, ref: int) -> bool:
        print(f"LegacyGateway: Checking status for ref: {ref}")
        return self._payment_successful

    def get_reference_number(self) -> int:
        return self._transaction_reference
```

</details>

<details>
<summary>C++</summary>

```cpp
class LegacyGateway {
private:
    long transactionReference = 0;
    bool paymentSuccessful = false;

public:
    void executeTransaction(double totalAmount, string currency) {
        cout << "LegacyGateway: Executing " << currency << " " << totalAmount << endl;
        transactionReference = chrono::duration_cast<chrono::nanoseconds>(
            chrono::system_clock::now().time_since_epoch()).count();
        paymentSuccessful = true;
        cout << "LegacyGateway: Done. Ref: " << transactionReference << endl;
    }

    bool checkStatus(long ref) {
        cout << "LegacyGateway: Checking status for ref: " << ref << endl;
        return paymentSuccessful;
    }

    long getReferenceNumber() {
        return transactionReference;
    }
};
```

</details>
<br/>


And here is the constraint:

- You cannot change CheckoutService, it is used system-wide and depends on PaymentProcessor
- You cannot modify LegacyGateway, it is from an external vendor
- But you must make them work together

## What is the Adapter Pattern


Two characteristics define the pattern:

**Interface translation**: The adapter acts like a translator between two incompatible classes. When the client calls a method, the adapter converts that call into the format the existing class understands. It can handle differences in method names, parameters, return values, or how methods are called.

**No source modification**: The adapter allows two classes to work together without changing their original code. The client continues using its expected interface, and the existing (incompatible) class remains unchanged. The adapter simply wraps the existing class and provides the interface the client expects, making them compatible.

### Two Types of Adapters

#### 1. Object Adapter (Uses Composition)

The adapter has an object of the existing class inside it.

Think of it like this:

- You have an old printer.
- You buy an adapter.
- The adapter connects to the printer and translates your computer's commands into commands the printer understands.

The adapter is not the printer. It simply contains (holds) the printer and forwards requests to it.

```
Client → Adapter → Existing Class
```

```py
class OldPrinter:
    def print_text(self, text):
        print(f"Printing: {text}")


class Adapter:
    def __init__(self, printer):
        self.printer = printer  # Has an object of OldPrinter

    def print(self, text):
        # Converts and forwards the request
        self.printer.print_text(text)


# Usage
old_printer = OldPrinter()
adapter = Adapter(old_printer)
adapter.print("Hello, World!")
```

#### 2. Class Adapter (Uses Inheritance)

Instead of holding the existing class, the adapter becomes the existing class by extending it.

```
Client → Adapter (extends Existing Class)
```

Example:

```py
class OldPrinter:
    def print_text(self, text):
        print(f"Printing: {text}")


class Adapter(OldPrinter):
    def print(self, text):
        # Calls the inherited method
        self.print_text(text)


# Usage
adapter = Adapter()

adapter.print("Hello, World!")
```

Here, the adapter inherits all the methods of OldPrinter.

This is called inheritance because the adapter extends the existing class.

### Implementing Adapter

`LegacyGatewayAdapter`. This adapter will implement the `PaymentProcessor` interface, which our `CheckoutService` already depends on.

Internally, it will **translate method calls** into the appropriate operations on the LegacyGateway effectively bridging the gap between incompatible APIs.


<details>
<summary>Python</summary>

```python
class LegacyGatewayAdapter(PaymentProcessor):
   def __init__(self, legacy_gateway):
       self.legacy_gateway = legacy_gateway
       self.current_ref = None

   def process_payment(self, amount, currency):
       print(f"Adapter: Translating processPayment() for {amount} {currency}")
       self.legacy_gateway.execute_transaction(amount, currency)
       self.current_ref = self.legacy_gateway.get_reference_number()

   def is_payment_successful(self):
       return self.legacy_gateway.check_status(self.current_ref)

   def get_transaction_id(self):
       return f"LEGACY_TXN_{self.current_ref}"

class ECommerceAppV2:
   @staticmethod
    def main():
       # Modern processor
       processor = InHousePaymentProcessor()
       modern_checkout = CheckoutService(processor)
       print("--- Using Modern Processor ---")
       modern_checkout.checkout(199.99, "USD")

       # Legacy gateway through adapter
       print("\n--- Using Legacy Gateway via Adapter ---")
       legacy = LegacyGateway()
       processor = LegacyGatewayAdapter(legacy)
       legacy_checkout = CheckoutService(processor)
       legacy_checkout.checkout(75.50, "USD")

if __name__ == "__main__":
   ECommerceAppV2.main()
```

</details>

<details>
<summary>C++</summary>

```cpp
class LegacyGatewayAdapter : public PaymentProcessor {
private:
   LegacyGateway* legacyGateway;
   long currentRef;

public:
   LegacyGatewayAdapter(LegacyGateway* legacyGateway) : legacyGateway(legacyGateway), currentRef(0) {}

   void processPayment(double amount, string currency) override {
       cout << "Adapter: Translating processPayment() for " << amount << " " << currency << endl;
       legacyGateway->executeTransaction(amount, currency);
       currentRef = legacyGateway->getReferenceNumber();
   }

   bool isPaymentSuccessful() override {
       return legacyGateway->checkStatus(currentRef);
   }

   string getTransactionId() override {
       return "LEGACY_TXN_" + to_string(currentRef);
   }
};

class ECommerceAppV2 {
public:
   static void main() {
       // Modern processor
       InHousePaymentProcessor processor;
       CheckoutService modernCheckout(&processor);
       cout << "--- Using Modern Processor ---" << endl;
       modernCheckout.checkout(199.99, "USD");

       // Legacy gateway through adapter
       cout << "\n--- Using Legacy Gateway via Adapter ---" << endl;
       LegacyGateway legacy;
       LegacyGatewayAdapter adapter(&legacy);
       CheckoutService legacyCheckout(&adapter);
       legacyCheckout.checkout(75.50, "USD");
   }
};

int main() {
   ECommerceAppV2::main();
   return 0;
}
```

</details>
<br/>



### What Makes This Adapter Work?

- **Composition over inheritance:** The adapter wraps the `LegacyGateway` object instead of inheriting from it, making the code more flexible and easier to maintain.

- **Method translation:** The adapter converts client method calls into the corresponding legacy API methods, handling differences in method names, parameters, and return values.

- **Type conversion:** The adapter converts the legacy `long` transaction reference into a `String` transaction ID, hiding the legacy data format from the client.

- **Encapsulation:** The adapter hides the complexity of the legacy API, so if the legacy system changes, only the adapter needs to be updated.


## Practical Example: Media Player Adapter
Lets say you are building a media player that natively plays MP3 files. The product team wants to add support for VLC (can play both MP4 and AVI) and MP4 formats.

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

# Target interface
class MediaPlayer(ABC):
    @abstractmethod
    def play(self, filename: str):
        pass


# Native implementation
class Mp3Player(MediaPlayer):
    def play(self, filename: str):
        print(f"MP3 Player: Playing {filename}")


# External codec libraries (Adaptees)
class VlcCodec:
    def play_vlc(self, filename: str):
        print(f"VLC Codec: Decoding and playing {filename}")


class Mp4Codec:
    def play_mp4(self, filename: str):
        print(f"MP4 Codec: Decoding and playing {filename}")


# Adapters
class VlcPlayerAdapter(MediaPlayer):
    def __init__(self, codec: VlcCodec):
        self._codec = codec

    def play(self, filename: str):
        self._codec.play_vlc(filename)


class Mp4PlayerAdapter(MediaPlayer):
    def __init__(self, codec: Mp4Codec):
        self._codec = codec

    def play(self, filename: str):
        self._codec.play_mp4(filename)


# Client
class AudioPlayer:
    def play_file(self, filename: str):
        extension = filename.rsplit(".", 1)[-1].lower()
        players = {
            "mp3": lambda: Mp3Player(),
            "vlc": lambda: VlcPlayerAdapter(VlcCodec()),
            "mp4": lambda: Mp4PlayerAdapter(Mp4Codec()),
        }

        creator = players.get(extension)
        if creator is None:
            print(f"Unsupported format: {extension}")
            return

        creator().play(filename)


if __name__ == "__main__":
    player = AudioPlayer()
    player.play_file("song.mp3")
    player.play_file("movie.mp4")
    player.play_file("documentary.vlc")
    player.play_file("image.png")
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <algorithm>

using namespace std;

// Target interface
class MediaPlayer {
public:
    virtual void play(string filename) = 0;
    virtual ~MediaPlayer() {}
};

// Native implementation
class Mp3Player : public MediaPlayer {
public:
    void play(string filename) override {
        cout << "MP3 Player: Playing " << filename << endl;
    }
};

// External codec libraries (Adaptees)
class VlcCodec {
public:
    void playVlc(string filename) {
        cout << "VLC Codec: Decoding and playing " << filename << endl;
    }
};

class Mp4Codec {
public:
    void playMp4(string filename) {
        cout << "MP4 Codec: Decoding and playing " << filename << endl;
    }
};

// Adapters
class VlcPlayerAdapter : public MediaPlayer {
private:
    VlcCodec* codec;
public:
    VlcPlayerAdapter(VlcCodec* codec) : codec(codec) {}
    void play(string filename) override {
        codec->playVlc(filename);
    }
};

class Mp4PlayerAdapter : public MediaPlayer {
private:
    Mp4Codec* codec;
public:
    Mp4PlayerAdapter(Mp4Codec* codec) : codec(codec) {}
    void play(string filename) override {
        codec->playMp4(filename);
    }
};

// Client
class AudioPlayer {
public:
    void playFile(string filename) {
        string ext = filename.substr(filename.rfind('.') + 1);
        transform(ext.begin(), ext.end(), ext.begin(), ::tolower);

        if (ext == "mp3") {
            Mp3Player player;
            player.play(filename);
        } else if (ext == "vlc") {
            VlcCodec codec;
            VlcPlayerAdapter adapter(&codec);
            adapter.play(filename);
        } else if (ext == "mp4") {
            Mp4Codec codec;
            Mp4PlayerAdapter adapter(&codec);
            adapter.play(filename);
        } else {
            cout << "Unsupported format: " << ext << endl;
        }
    }
};

int main() {
    AudioPlayer player;
    player.playFile("song.mp3");
    player.playFile("movie.mp4");
    player.playFile("documentary.vlc");
    player.playFile("image.png");
    return 0;
}
```

</details>
<br/>

