# Chapter 1 Association

Association is a relationship between two independent classes where one object uses or interacts with another object. Both objects can exist independently of each other.

If Class A must interact with Class B to fulfill its purpose, then Class A is associated with Class B.


- Represents a `Uses-A` or `has-a` relationship.
- Objects are loosely coupled.
- Both classes can exist independently.


#### Types of Association

- **Unidirectional Association**: One class knows about another, but not vice versa. Example: A Student has a LibraryCard, but the LibraryCard doesn't know about the Student.


- **Bidirectional Association**: Both classes know about and interact with each other. Example: A Teacher is assigned to a Classroom, and the Classroom knows its Teacher.


## 2. UML Representation

| Symbol       | Meaning                         | Example Scenario                     |
| ------------ | ------------------------------- | ------------------------------------ |
| `---`        | An association between classes  | `Student --- Teacher`                |
| `-->`        | Directionality (who knows whom) | `Order --> PaymentGateway`           |
| No arrowhead | Bidirectional association       | `Team --- Developer`                 |
| `1`          | Exactly one                     | Each User has one Profile            |
| `0..1`       | Zero or one (optional)          | An Employee may have a Manager       |
| `*`          | Many (zero or more)             | A Project can have many Tasks        |
| `1..*`       | At least one                    | Each Course has one or more Students |



## Practical Example: Hospital Appointment System

A hospital manages doctors, patients, rooms, and appointments. The relationships between these entities demonstrate unidirectional, bidirectional, one-to-many, and many-to-many associations working together.

Here's how the classes connect:

- `Appointment` holds a reference to a `Room` (unidirectional, the room doesn't know about its appointments).  

- `Doctor` has a list of `Appointment` objects, and each `Appointment` points back to its `Doctor` (bidirectional one-to-many).

- `Patient` has a list of `Appointment` objects, and each Appointment points back to its `Patient` (bidirectional one-to-many).

- `Doctor` and `Patient` are connected many-to-many through Appointment as an intermediary. A doctor sees many patients, and a patient can visit many doctors, but they don't reference each other directly.

![Text3](/assets/3.png)


<details>
<summary>Python</summary>

```python
class Room:
    def __init__(self, number: str, floor: int):
        self.number = number
        self.floor = floor

class Appointment:
    def __init__(self, doctor, patient, room: Room, time: str):
        self.doctor = doctor
        self.patient = patient
        self.room = room
        self.time = time
        doctor.add_appointment(self)
        patient.add_appointment(self)

class Doctor:
    def __init__(self, name: str, specialization: str):
        self.name = name
        self.specialization = specialization
        self.appointments = []

    def add_appointment(self, appt: Appointment):
        self.appointments.append(appt)

    def get_patients(self):
        seen = set()
        result = []
        for appt in self.appointments:
            if id(appt.patient) not in seen:
                seen.add(id(appt.patient))
                result.append(appt.patient)
        return result

class Patient:
    def __init__(self, name: str):
        self.name = name
        self.appointments = []

    def add_appointment(self, appt: Appointment):
        self.appointments.append(appt)

    def get_doctors(self):
        seen = set()
        result = []
        for appt in self.appointments:
            if id(appt.doctor) not in seen:
                seen.add(id(appt.doctor))
                result.append(appt.doctor)
        return result

# Usage
dr_smith = Doctor("Dr. Smith", "Cardiology")
dr_patel = Doctor("Dr. Patel", "Neurology")

alice = Patient("Alice")
bob = Patient("Bob")

room_101 = Room("101", 1)
room_205 = Room("205", 2)

Appointment(dr_smith, alice, room_101, "9:00 AM")
Appointment(dr_smith, bob, room_101, "10:00 AM")
Appointment(dr_patel, alice, room_205, "2:00 PM")

print(f"{dr_smith.name}'s patients:")
for p in dr_smith.get_patients():
    print(f"  - {p.name}")

print(f"{alice.name}'s doctors:")
for d in alice.get_doctors():
    print(f"  - {d.name} ({d.specialization})")

print(f"{dr_smith.name}'s schedule:")
for a in dr_smith.appointments:
    print(f"  - {a.time} with {a.patient.name} in Room {a.room.number}")
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

class Room {
private:
    string number;
    int floor;
public:
    Room(const string& number, int floor) : number(number), floor(floor) {}
    string getNumber() const { return number; }
    int getFloor() const { return floor; }
};

class Doctor;
class Patient;

class Appointment {
private:
    Doctor* doctor;
    Patient* patient;
    Room* room;
    string time;
public:
    Appointment(Doctor* doctor, Patient* patient, Room* room, const string& time);
    Doctor* getDoctor() const { return doctor; }
    Patient* getPatient() const { return patient; }
    Room* getRoom() const { return room; }
    string getTime() const { return time; }
};

class Doctor {
private:
    string name;
    string specialization;
    vector<Appointment*> appointments;
public:
    Doctor(const string& name, const string& specialization)
        : name(name), specialization(specialization) {}

    void addAppointment(Appointment* appt) {
        appointments.push_back(appt);
    }

    vector<Patient*> getPatients() const;

    string getName() const { return name; }
    string getSpecialization() const { return specialization; }
    vector<Appointment*> getAppointments() const { return appointments; }
};

class Patient {
private:
    string name;
    vector<Appointment*> appointments;
public:
    Patient(const string& name) : name(name) {}

    void addAppointment(Appointment* appt) {
        appointments.push_back(appt);
    }

    vector<Doctor*> getDoctors() const;

    string getName() const { return name; }
    vector<Appointment*> getAppointments() const { return appointments; }
};

Appointment::Appointment(Doctor* doctor, Patient* patient, Room* room, const string& time)
    : doctor(doctor), patient(patient), room(room), time(time) {
    doctor->addAppointment(this);
    patient->addAppointment(this);
}

vector<Patient*> Doctor::getPatients() const {
    vector<Patient*> result;
    for (auto* appt : appointments) {
        auto* p = appt->getPatient();
        if (find(result.begin(), result.end(), p) == result.end())
            result.push_back(p);
    }
    return result;
}

vector<Doctor*> Patient::getDoctors() const {
    vector<Doctor*> result;
    for (auto* appt : appointments) {
        auto* d = appt->getDoctor();
        if (find(result.begin(), result.end(), d) == result.end())
            result.push_back(d);
    }
    return result;
}

int main() {
    Doctor drSmith("Dr. Smith", "Cardiology");
    Doctor drPatel("Dr. Patel", "Neurology");

    Patient alice("Alice");
    Patient bob("Bob");

    Room room101("101", 1);
    Room room205("205", 2);

    Appointment a1(&drSmith, &alice, &room101, "9:00 AM");
    Appointment a2(&drSmith, &bob, &room101, "10:00 AM");
    Appointment a3(&drPatel, &alice, &room205, "2:00 PM");

    cout << drSmith.getName() << "'s patients:" << endl;
    for (auto* p : drSmith.getPatients())
        cout << "  - " << p->getName() << endl;

    cout << alice.getName() << "'s doctors:" << endl;
    for (auto* d : alice.getDoctors())
        cout << "  - " << d->getName() << " (" << d->getSpecialization() << ")" << endl;

    cout << drSmith.getName() << "'s schedule:" << endl;
    for (auto* a : drSmith.getAppointments())
        cout << "  - " << a->getTime() << " with " << a->getPatient()->getName()
             << " in Room " << a->getRoom()->getNumber() << endl;

    return 0;
}
```

</details>
<br/>

## Why This Design Works
- The `Appointment` class is the intermediary. Instead of `Doctor` and `Patient` holding direct references to each other (which would create a tangled many-to-many), they connect through Appointment. This is a common pattern for modeling many-to-many relationships in code, analogous to a join table in a relational database. 


----


# Chapter 2 Aggregation

> Pass by constructor !!!!!!!!!!!

Aggregation is a special form of Association that represents a Has-A relationship with weak ownership. The contained object can exist independently of the container object.

- Represents a **Has-A** relationship.
- It is a weak association.
- Child objects can exist independently of the parent object.


## UML Representation
In UML class diagrams, aggregation is represented by a hollow diamond (◊) on the "whole" side of the relationship. The diamond connects to the class that contains or references the other objects.

![Text4](/assets/4.png)

- Playlist holds references to multiple `Song` objects. The 1 to * multiplicity means one playlist can group many songs.
- Song is an independent entity with its own data (title, artist, duration). It doesn't know which playlists reference it.
- The hollow diamond (`o--`) on the `Playlist` side is the UML notation for aggregation. It signals that `Playlist` is the "whole" and `Song` is the "part," but the songs are not owned by the playlist.

## Code Example
Let’s model a real-world aggregation: The relationship between a university Department and its Professors.

A department "has" professors, but the professors are independent entities. If the department is restructured or closed, the professors (as university employees) still exist and can be assigned to other departments. The department does not own the lifecycle of the professors.

<details>
<summary>Python</summary>

```python
class Professor:
    def __init__(self, name):
        self.name = name

    def get_name(self):
        return self.name
		
class Department:
    def __init__(self, name, professors):
        self.name = name
        self.professors = professors

    def print_professors(self):
        print(f"Professors in {self.name} Department:")
        for professor in self.professors:
            print(f"- {professor.get_name()}")
			
if __name__ == "__main__":
    p1 = Professor("Dr. Smith")
    p2 = Professor("Dr. Johnson")

    profs = [p1, p2]

    cs_dept = Department("Computer Science", profs)
    cs_dept.print_professors()

    # cs_dept can go out of scope or be deleted...
    # but p1 and p2 still exist and can be used elsewhere.			
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

class Professor {
private:
    string name;
public:
    Professor(const string& name) : name(name) {}

    string getName() const {
        return name;
    }
};

class Department {
private:
    string name;
    vector<Professor*> professors;
public:
    Department(const string& name, const vector<Professor*>& professors)
        : name(name), professors(professors) {}

    void printProfessors() const {
        cout << "Professors in " << name << " Department:" << endl;
        for (const auto& professor : professors) {
            cout << "- " << professor->getName() << endl;
        }
    }
};

int main() {
    Professor* p1 = new Professor("Dr. Smith");
    Professor* p2 = new Professor("Dr. Johnson");

    vector<Professor*> profs = {p1, p2};

    Department csDept("Computer Science", profs);
    csDept.printProfessors();

    // csDept can go out of scope or be deleted...
    // but p1 and p2 still exist and can be used elsewhere.

    delete p1;
    delete p2;

    return 0;
}
```

</details>
<br/>


## Bad → Good → Great Example

- **Bad**: A Team class has a method createNewDeveloper(), creating and destroying Developer objects internally. This creates tight coupling, making it behave like composition.

- **Good**: A Team class holds a reference to Developer instances that are created elsewhere and passed to it. This is standard aggregation.

- **Great**: A Team's dependencies (the list of Developers) are provided via its constructor or a setter method (**Dependency Injection**). This is the most flexible approach, promoting high modularity and making the Team class easy to test with mock Developer objects.


----


# Chapter 3 Composition

**Composition** is a special type of association that signifies **strong ownership** between objects. The “whole” class is fully responsible for creating, managing, and destroying the “part” objects. In fact, the parts cannot exist without the whole.


- Represents a **strong “has-a”** relationship.

- The **whole owns** the part and controls its lifecycle.

- When the whole is destroyed, the parts are also destroyed.

- The parts are **not shared** with any other object.

- The part has **no independent meaning** or identity outside the whole.

## UML Representation

 composition is represented by a filled **diamond (◆)** at the “whole” end of the relationship. This is in contrast to aggregation's hollow diamond (◊) and association's plain solid line.



## Code Example
Let's model the ordering scenario. An `Order` composes multiple `LineItem` objects. The order creates line items internally when items are added, and destroys them when the order is destroyed.

<details>
<summary>Python</summary>

```python
class LineItem:
    def __init__(self, product_name, quantity, unit_price):
        self.product_name = product_name
        self.quantity = quantity
        self.unit_price = unit_price

    def get_subtotal(self):
        return self.quantity * self.unit_price

    def describe(self):
        print(f"{self.product_name} x{self.quantity} "
              f"@ ${self.unit_price:.2f} = ${self.get_subtotal():.2f}")

class Order:
    def __init__(self, order_id):
        self.order_id = order_id
        self.line_items = []

    def add_item(self, product, quantity, unit_price):
        self.line_items.append(LineItem(product, quantity, unit_price))

    def remove_item(self, product):
        self.line_items = [
            item for item in self.line_items
            if item.product_name != product
        ]

    def get_total(self):
        return sum(item.get_subtotal() for item in self.line_items)

    def print_receipt(self):
        print(f"Order: {self.order_id}")
        for item in self.line_items:
            item.describe()
        print(f"Total: ${self.get_total():.2f}")

if __name__ == "__main__":
    order = Order("ORD-1001")
    order.add_item("Wireless Mouse", 2, 29.99)
    order.add_item("USB-C Cable", 3, 9.99)
    order.add_item("Laptop Stand", 1, 49.99)

    order.print_receipt()

    # When order is deleted, all LineItems are destroyed with it.
    # No LineItem exists outside of an Order.
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

class LineItem {
private:
    string productName;
    int quantity;
    double unitPrice;

public:
    LineItem(const string& productName, int quantity, double unitPrice)
        : productName(productName), quantity(quantity), unitPrice(unitPrice) {}

    double getSubtotal() const {
        return quantity * unitPrice;
    }

    string getProductName() const {
        return productName;
    }

    void describe() const {
        cout << productName << " x" << quantity
             << " @ $" << unitPrice
             << " = $" << getSubtotal() << endl;
    }
};

class Order {
private:
    string orderId;
    vector<LineItem> lineItems;

public:
    Order(const string& orderId) : orderId(orderId) {}

    void addItem(const string& product, int quantity, double unitPrice) {
        lineItems.emplace_back(product, quantity, unitPrice);
    }

    void removeItem(const string& product) {
        lineItems.erase(
            remove_if(lineItems.begin(), lineItems.end(),
                [&](const LineItem& item) {
                    return item.getProductName() == product;
                }),
            lineItems.end());
    }

    double getTotal() const {
        double total = 0;
        for (const auto& item : lineItems) {
            total += item.getSubtotal();
        }
        return total;
    }

    void printReceipt() const {
        cout << "Order: " << orderId << endl;
        for (const auto& item : lineItems) {
            item.describe();
        }
        cout << "Total: $" << getTotal() << endl;
    }
};

int main() {
    Order order("ORD-1001");
    order.addItem("Wireless Mouse", 2, 29.99);
    order.addItem("USB-C Cable", 3, 9.99);
    order.addItem("Laptop Stand", 1, 49.99);

    order.printReceipt();

    // When order goes out of scope, all LineItems are destroyed
    // automatically (RAII). No manual cleanup needed.

    return 0;
}
```

</details>
<br/>

- **The order creates its own line items.**: The addItem() method takes raw data and internally creates a new LineItem(...).

- **Line items have no independent existence.**: There is no LineItem floating around in the system outside of an Order. No other class holds a reference to these line items. They are born inside the order and die with the order.


- **Destroying the order destroys all line items.**: When the Order object is garbage collected, all its LineItem objects are destroyed too.

> This is a true composition relationship: the parts exist only within the context of the whole, and their lifecycle is completely controlled by it.


Composition is a relationship where one object is made up of other objects and completely owns them. A simple way to think about it is that the part cannot meaningfully exist without the whole. For example, a house is composed of rooms. A room is created as part of a house, belongs to that house, and if the house is destroyed, the room no longer exists independently.

Composition is often preferred over inheritance because it makes software more flexible and easier to maintain. Instead of building long inheritance hierarchies, you can create a class by combining smaller, reusable components.   
For example, a vehicle can contain an engine object rather than inheriting from different vehicle types. This allows the engine to be changed easily, such as switching between a petrol engine, electric engine, or hybrid engine, without modifying the vehicle class itself.

<br/>

| Feature            | Association              | Aggregation                   | Composition                      |
| ------------------ | ------------------------ | ----------------------------- | -------------------------------- |
| Ownership          | None                     | Weak — has-a                  | Strong — owns-a                  |
| Lifecycle          | Independent              | Independent                   | Dependent — part dies with whole |
| Tightness          | Loose coupling           | Moderate coupling             | Tight coupling                   |
| Multiplicity       | Flexible (1:1, 1:N, N:N) | Whole can group many parts    | Whole composed of integral parts |
| Reusability        | High — parts reusable    | Moderate — parts often reused | Low — parts not reused outside   |
| UML Symbol         | Solid Line               | Hollow Diamond (◊)            | Filled Diamond (◆)               |
| Who creates parts? | Either side or external  | External — passed in          | Whole — created internally       |
| Real Example       | Student ↔ Course         | Playlist → Song               | Order → LineItem                 |


**Think of it like this:**
- **Association** is a general connection: two classes simply know about each other.

- **Aggregation** is a grouping: the whole and parts can exist independently.

- **Composition** is an ownership: the part’s existence is bound to the whole.

![Text5](/assets/5.png)


## Exercise 1: Computer System
Design Computer System Class
Problem: Build a Computer that composes a CPU, RAM, and HardDrive. The computer creates these parts internally based on specs passed to its constructor. No component exists outside of a computer, and destroying the computer destroys all its components.


<details>
<summary>Python</summary>

```python
class CPU:
    def __init__(self, model, cores):
        self.model = model
        self.cores = cores

    def describe(self):
        print(f"  CPU: {self.model} ({self.cores} cores)")

class RAM:
    def __init__(self, size_gb):
        self.size_gb = size_gb

    def describe(self):
        print(f"  RAM: {self.size_gb} GB")

class HardDrive:
    def __init__(self, capacity_gb):
        self.capacity_gb = capacity_gb

    def describe(self):
        print(f"  Storage: {self.capacity_gb} GB")

class Computer:
    def __init__(self, name, cpu_model, cpu_cores, ram_gb, storage_gb):
        self.name = name
        self.cpu = CPU(cpu_model, cpu_cores)
        self.ram = RAM(ram_gb)
        self.hard_drive = HardDrive(storage_gb)

    def describe_specs(self):
        print(f"Computer: {self.name}")
        self.cpu.describe()
        self.ram.describe()
        self.hard_drive.describe()

    def upgrade_ram(self, new_size_gb):
        self.ram = RAM(new_size_gb)

if __name__ == "__main__":
    pc = Computer("Dev Workstation", "Intel i7-13700K", 16, 32, 1000)

    pc.describe_specs()

    pc.upgrade_ram(64)
    print("\nAfter RAM upgrade:")
    pc.describe_specs()
```

</details>

<details>
<summary>C++</summary>

```cpp
class CPU {
private:
    string model;
    int cores;
public:
    CPU(const string& model, int cores) : model(model), cores(cores) {}

    void describe() const {
        cout << "  CPU: " << model << " (" << cores << " cores)" << endl;
    }
};

class RAM {
private:
    int sizeGB;
public:
    RAM(int sizeGB) : sizeGB(sizeGB) {}

    void describe() const {
        cout << "  RAM: " << sizeGB << " GB" << endl;
    }

    int getSizeGB() const { return sizeGB; }
};

class HardDrive {
private:
    int capacityGB;
public:
    HardDrive(int capacityGB) : capacityGB(capacityGB) {}

    void describe() const {
        cout << "  Storage: " << capacityGB << " GB" << endl;
    }
};

class Computer {
private:
    string name;
    CPU cpu;
    RAM ram;
    HardDrive hardDrive;
public:
    Computer(const string& name, const string& cpuModel, int cpuCores,
             int ramGB, int storageGB)
        : name(name),
          cpu(cpuModel, cpuCores),
          ram(ramGB),
          hardDrive(storageGB) {}

    void describeSpecs() const {
        cout << "Computer: " << name << endl;
        cpu.describe();
        ram.describe();
        hardDrive.describe();
    }

    void upgradeRAM(int newSizeGB) {
        ram = RAM(newSizeGB);
    }
};

int main() {
    Computer pc("Dev Workstation", "Intel i7-13700K", 16, 32, 1000);

    pc.describeSpecs();

    pc.upgradeRAM(64);
    cout << "\nAfter RAM upgrade:" << endl;
    pc.describeSpecs();

    return 0;
}
```

</details>
<br/>



---

# Chapter 4 Dependency

What happens when a class needs to use another class for a brief moment to get a job done, without needing to hold onto it forever?  
That’s **dependency**

Dependency exists when one class relies on another to fulfill a responsibility, but does so without retaining a permanent reference to it.

This typically happens when:

- A class **accepts another class as a method parameter**.

- A class **instantiates or uses another class inside a method**.

- A class **returns an object of another class from a method**.


#### UML Representation dashed arrow (`..>`)

## Recognizing Dependencies in Code
### 1. As Method Parameters
The dependent class receives another class as a parameter, uses it during the method, and lets it go.

```py
class ReportGenerator:
    def generate(self, source):
        data = source.fetch_all()
        # Format data into report...
        return formatted_report
```




### 2. As Local Variables
Class creates another class inside a method, uses it, and discards it. The created object never escapes the method scope.

```py
class OrderProcessor:
    def process(self, order):
        formatter = JsonFormatter()
        json = formatter.format(order)
        # Send json to external API...
```

### 3. As Return Types

```py
class UserFactory:
    def create_user(self, name, email):
        return User(name, email)
```

### 4. As Static Method Calls
A class can depend on another class by calling its static methods. There's no object reference at all, just a class-level dependency.


```Python
class PasswordService:
    def verify(self, input_text, hash_value):
        return HashUtils.sha256(input_text) == hash_value
```


# Dependency Injection (DI)

**Dependency Injection** is a design technique where a class receives the objects it depends on, instead of creating them itself.


This leads to:
- Better testability
- Greater modularity
- Loose coupling


### Example
Consider a `NotificationService` that sends email notifications. A straightforward implementation might create its own `EmailSender` internally:

```py
class NotificationService:
    def __init__(self):
        self.sender = EmailSender()  # Creates its own dependency

    def notify_user(self, message):
        self.sender.send(message)
```

This looks reasonable, but it has several problems:

- **Can't switch implementations**. Want to send SMS instead of email?

- **Violates the Open/Closed Principle**. Adding a new notification channel requires changing existing code rather than extending it.

### The Solution: Inject from Outside

Instead of letting the class create its own dependencies, you provide them from outside. This is Dependency Injection: a design technique where a class receives the objects it depends on rather than creating them itself.

```py
from abc import ABC, abstractmethod

class Sender(ABC):
    @abstractmethod
    def send(self, message: str) -> None:
        pass

class EmailSender(Sender):
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SmsSender(Sender):
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

class NotificationService:
    def __init__(self, sender: Sender):
        self.sender = sender  # Injected from outside

    def notify_user(self, message: str) -> None:
        self.sender.send(message)
```

Now the `NotificationService` depends on a Sender interface, not a concrete `EmailSender`. The specific implementation is provided from outside through the constructor.


This gives you three concrete benefits:

- **Swappable implementations**

- **Loose coupling**

- **Easy testing**. Pass a MockSender in unit tests that records messages instead of actually sending them.


## File Conversion Service
Design File Conversion Service
Problem: Build a FileConverter that converts files from one format to another. During the convert() method, it depends on a FileReader to read the source file, a FormatParser to parse the content, and a FileWriter to write the output. None of these are stored as fields.

### Requirements:

- `FileReader` with a `read(filePath)` method that returns the file content as a string.

- `FormatParser` with a `parse(content, targetFormat)` method that returns the converted content.

- `FileWriter` with a `write(filePath, content)` method that writes the content to a file.  

- `FileConverter` with a `convert(sourcePath, targetPath, targetFormat, reader, parser, writer)` method that coordinates the three dependencies. No fields.

<details>
<summary>Python</summary>

```python
class FileReader:
    def read(self, file_path):
        print(f"Reading file: {file_path}")
        content = "name,age,city"
        print(f"Content: {content}")
        return content

class FormatParser:
    def parse(self, content, target_format):
        print(f"Parsing content to {target_format} format")
        parsed = '[{"name":"Alice","age":30,"city":"NYC"}]'
        print(f"Parsed: {parsed}")
        return parsed

class FileWriter:
    def write(self, file_path, content):
        print(f"Writing to file: {file_path}")

class FileConverter:
    def convert(self, source_path, target_path, target_format,
                reader, parser, writer):
        content = reader.read(source_path)
        parsed = parser.parse(content, target_format)
        writer.write(target_path, parsed)
        print(f"File conversion complete: {source_path} -> {target_path}")

if __name__ == "__main__":
    converter = FileConverter()

    reader = FileReader()
    parser = FormatParser()
    writer = FileWriter()

    converter.convert("data.csv", "output.json", "JSON", reader, parser, writer)
```

</details>

<details>
<summary>C++</summary>

```cpp
class FileReader {
public:
    string read(const string& filePath) {
        cout << "Reading file: " << filePath << endl;
        string content = "name,age,city";
        cout << "Content: " << content << endl;
        return content;
    }
};

class FormatParser {
public:
    string parse(const string& content, const string& targetFormat) {
        cout << "Parsing content to " << targetFormat << " format" << endl;
        string parsed = "[{\"name\":\"Alice\",\"age\":30,\"city\":\"NYC\"}]";
        cout << "Parsed: " << parsed << endl;
        return parsed;
    }
};

class FileWriter {
public:
    void write(const string& filePath, const string& content) {
        cout << "Writing to file: " << filePath << endl;
    }
};

class FileConverter {
public:
    void convert(const string& sourcePath, const string& targetPath,
                 const string& targetFormat, FileReader& reader,
                 FormatParser& parser, FileWriter& writer) {
        string content = reader.read(sourcePath);
        string parsed = parser.parse(content, targetFormat);
        writer.write(targetPath, parsed);
        cout << "File conversion complete: " << sourcePath << " -> " << targetPath << endl;
    }
};

int main() {
    FileConverter converter;

    FileReader reader;
    FormatParser parser;
    FileWriter writer;

    converter.convert("data.csv", "output.json", "JSON", reader, parser, writer);

    return 0;
}
```

</details>
<br/>

---


