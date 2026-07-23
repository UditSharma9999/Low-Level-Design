# Composite Design Pattern


The Composition design pattern means building a complex object by combining smaller, independent objects instead of making one class inherit from another. Think of it as `"has-a"` relationship rather than `"is-a"`. For example, instead of creating a SportsCar class that inherits everything from Car, you can make a Car object that has an Engine, Wheels, MusicSystem, and GPS. 


Two characteristics define the pattern:

1. **Uniform interface**: Both leaf objects (no children) and composite objects (contain children) implement the same interface. The client calls the same methods regardless of which type it is working with.
2. **Recursive composition**: A composite holds a collection of components, which can themselves be composites. This creates tree structures of arbitrary depth without any special handling.


## The Problem: Modeling a File Explorer

Imagine you are building a file explorer application (like Finder on macOS or File Explorer on Windows). The system needs to represent:

- Files that have a name and a size.
- Folders that can hold files and other folders, nested to any depth.

Your goal is to support operations such as:

- `getSize()`: returns the total size of a file or folder (sum of all contents for folders).
- `printStructure()`: prints the name of the item with indentation to show hierarchy.
- `delete()`: deletes a file or a folder and everything inside it.

### The Naive Approach

<details>
<summary>Python</summary>

```python
class File:
    def __init__(self, name: str, size: int):
        self.name = name
        self.size = size

    def get_size(self) -> int:
        return self.size

    def print_structure(self, indent: str):
        print(indent + self.name)

    def delete(self):
        print(f"Deleting file: {self.name}")


class Folder:
    def __init__(self, name: str):
        self.name = name
        self.contents = []

    def add(self, item):
        self.contents.append(item)

    def get_size(self) -> int:
        total = 0
        for item in self.contents:
            if isinstance(item, File):
                total += item.get_size()
            elif isinstance(item, Folder):
                total += item.get_size()
        return total

    def print_structure(self, indent: str):
        print(indent + self.name + "/")
        for item in self.contents:
            if isinstance(item, File):
                item.print_structure(indent + "  ")
            elif isinstance(item, Folder):
                item.print_structure(indent + "  ")

    def delete(self):
        for item in self.contents:
            if isinstance(item, File):
                item.delete()
            elif isinstance(item, Folder):
                item.delete()
        print(f"Deleting folder: {self.name}")
```

</details>

<details>
<summary>C++</summary>

```cpp
class File {
private:
    string name;
    int size;

public:
    File(string name, int size) : name(name), size(size) {}

    int getSize() {
        return size;
    }

    void printStructure(string indent) {
        cout << indent << name << endl;
    }

    void deleteFile() {
        cout << "Deleting file: " << name << endl;
    }
};


class Folder {
private:
    string name;
    vector<void*> contents;
    vector<bool> isFile;

public:
    Folder(string name) : name(name) {}

    void addFile(File* file) {
        contents.push_back(file);
        isFile.push_back(true);
    }

    void addFolder(Folder* folder) {
        contents.push_back(folder);
        isFile.push_back(false);
    }

    int getSize() {
        int total = 0;
        for (size_t i = 0; i < contents.size(); i++) {
            if (isFile[i])
                total += static_cast<File*>(contents[i])->getSize();
            else
                total += static_cast<Folder*>(contents[i])->getSize();
        }
        return total;
    }

    void printStructure(string indent) {
        cout << indent << name << "/" << endl;
        for (size_t i = 0; i < contents.size(); i++) {
            if (isFile[i])
                static_cast<File*>(contents[i])->printStructure(indent + "  ");
            else
                static_cast<Folder*>(contents[i])->printStructure(indent + "  ");
        }
    }

    void deleteFolder() {
        for (size_t i = 0; i < contents.size(); i++) {
            if (isFile[i])
                static_cast<File*>(contents[i])->deleteFile();
            else
                static_cast<Folder*>(contents[i])->deleteFolder();
        }
        cout << "Deleting folder: " << name << endl;
    }
};
```

</details>
<br/>

### What’s Wrong With This Approach?

**1. Repetitive Type Checks** : 
Every operation (`getSize()`, `printStructure()`, `delete()`) requires instanceof checks and downcasting. This logic is duplicated across every method that touches the tree.


**2. No Shared Abstraction**


**3. Violation of Open/Closed Principle**: 
To add new item types (say, `Shortcut` or `CompressedFolder`), you must modify the type-checking logic in every existing method. Each new type means touching every operation in the Folder class.



## Class Diagram


![text19](/assets/19.png)

### Component Interface (e.g., FileSystemItem)
The shared interface that declares operations common to both leaves and composites.

### Leaf (e.g., File)
An end object in the tree that has no children. It implements the Component interface directly.


### Composite (e.g., Folder)
A container that holds child Components and implements the Component interface by delegating to its children.

### Client (e.g., FileExplorerApp)
Works with the tree through the Component interface, without knowing whether it holds a leaf or a composite.



## Implementing the Composite Pattern

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class FileSystemItem(ABC):
    @abstractmethod
    def get_size(self) -> int:
        pass

    @abstractmethod
    def print_structure(self, indent: str):
        pass

    @abstractmethod
    def delete(self):
        pass


class File(FileSystemItem):
    def __init__(self, name: str, size: int):
        self.name = name
        self.size = size

    def get_size(self) -> int:
        return self.size

    def print_structure(self, indent: str):
        print(f"{indent}- {self.name} ({self.size} KB)")

    def delete(self):
        print(f"Deleting file: {self.name}")

        class Folder(FileSystemItem):
    def __init__(self, name: str):
        self.name = name
        self.children: list[FileSystemItem] = []

    def add_item(self, item: FileSystemItem):
        self.children.append(item)

    def remove_item(self, item: FileSystemItem):
        self.children.remove(item)

    def get_size(self) -> int:
        total = 0
        for item in self.children:
            total += item.get_size()
        return total

    def print_structure(self, indent: str):
        print(f"{indent}+ {self.name}/")
        for item in self.children:
            item.print_structure(indent + "  ")

    def delete(self):
        for item in self.children:
            item.delete()
        print(f"Deleting folder: {self.name}")

if __name__ == "__main__":
    file1 = File("readme.txt", 5)
    file2 = File("photo.jpg", 1500)
    file3 = File("data.csv", 300)

    documents = Folder("Documents")
    documents.add_item(file1)
    documents.add_item(file3)

    pictures = Folder("Pictures")
    pictures.add_item(file2)

    home = Folder("Home")
    home.add_item(documents)
    home.add_item(pictures)

    print("---- File Structure ----")
    home.print_structure("")

    print(f"\nTotal Size: {home.get_size()} KB")

    print("\n---- Deleting All ----")
    home.delete()
```

</details>

<details>
<summary>C++</summary>

```cpp
class FileSystemItem {
public:
    virtual int getSize() = 0;
    virtual void printStructure(string indent) = 0;
    virtual void deleteItem() = 0;
    virtual ~FileSystemItem() {}
};


class File : public FileSystemItem {
private:
    string name;
    int size;

public:
    File(string name, int size) : name(name), size(size) {}

    int getSize() override {
        return size;
    }

    void printStructure(string indent) override {
        cout << indent << "- " << name << " (" << size << " KB)" << endl;
    }

    void deleteItem() override {
        cout << "Deleting file: " << name << endl;
    }
};


class Folder : public FileSystemItem {
private:
    string name;
    vector<FileSystemItem*> children;

public:
    Folder(string name) : name(name) {}

    void addItem(FileSystemItem* item) {
        children.push_back(item);
    }

    void removeItem(FileSystemItem* item) {
        children.erase(
            remove(children.begin(), children.end(), item),
            children.end()
        );
    }

    int getSize() override {
        int total = 0;
        for (FileSystemItem* item : children) {
            total += item->getSize();
        }
        return total;
    }

    void printStructure(string indent) override {
        cout << indent << "+ " << name << "/" << endl;
        for (FileSystemItem* item : children) {
            item->printStructure(indent + "  ");
        }
    }

    void deleteItem() override {
        for (FileSystemItem* item : children) {
            item->deleteItem();
        }
        cout << "Deleting folder: " << name << endl;
    }
};


int main() {
    FileSystemItem* file1 = new File("readme.txt", 5);
    FileSystemItem* file2 = new File("photo.jpg", 1500);
    FileSystemItem* file3 = new File("data.csv", 300);

    Folder* documents = new Folder("Documents");
    documents->addItem(file1);
    documents->addItem(file3);

    Folder* pictures = new Folder("Pictures");
    pictures->addItem(file2);

    Folder* home = new Folder("Home");
    home->addItem(documents);
    home->addItem(pictures);

    cout << "---- File Structure ----" << endl;
    home->printStructure("");

    cout << "\nTotal Size: " << home->getSize() << " KB" << endl;

    cout << "\n---- Deleting All ----" << endl;
    home->deleteItem();

    delete file1;
    delete file2;
    delete file3;
    delete documents;
    delete pictures;
    delete home;

    return 0;
}
```

</details>
<br/>


Notice the difference from the naive approach. There are no `instanceof` checks anywhere. The `Folder` simply calls `getSize()` on each child. Polymorphism handles the rest. Whether a child is a `File` or another `Folder`, the correct implementation runs automatically.

## Practical Example: Organization Hierarchy

To show the Composite pattern in a completely different domain, let's build an organization hierarchy system. An OrgComponent interface is shared by Employee (leaf) and Manager (composite). Managers can contain employees and other managers, forming a tree that represents the company structure.


<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class OrgComponent(ABC):
    @abstractmethod
    def get_salary(self) -> int:
        pass

    @abstractmethod
    def get_headcount(self) -> int:
        pass

    @abstractmethod
    def print_hierarchy(self, indent: str):
        pass


class Employee(OrgComponent):
    def __init__(self, name: str, title: str, salary: int):
        self.name = name
        self.title = title
        self.salary = salary

    def get_salary(self) -> int:
        return self.salary

    def get_headcount(self) -> int:
        return 1

    def print_hierarchy(self, indent: str):
        print(f"{indent}- {self.name} ({self.title}, ${self.salary})")


class Manager(OrgComponent):
    def __init__(self, name: str, title: str, salary: int):
        self.name = name
        self.title = title
        self.salary = salary
        self.members: list[OrgComponent] = []

    def add_member(self, member: OrgComponent):
        self.members.append(member)

    def remove_member(self, member: OrgComponent):
        self.members.remove(member)

    def get_salary(self) -> int:
        total = self.salary
        for member in self.members:
            total += member.get_salary()
        return total

    def get_headcount(self) -> int:
        count = 1
        for member in self.members:
            count += member.get_headcount()
        return count

    def print_hierarchy(self, indent: str):
        print(f"{indent}+ {self.name} ({self.title}, ${self.salary})")
        for member in self.members:
            member.print_hierarchy(indent + "  ")


if __name__ == "__main__":
    dev1 = Employee("Alice", "Senior Engineer", 120000)
    dev2 = Employee("Bob", "Engineer", 95000)
    dev3 = Employee("Charlie", "Engineer", 90000)
    designer = Employee("Diana", "Designer", 100000)

    tech_lead = Manager("Eve", "Tech Lead", 140000)
    tech_lead.add_member(dev1)
    tech_lead.add_member(dev2)

    vp_eng = Manager("Frank", "VP Engineering", 200000)
    vp_eng.add_member(tech_lead)
    vp_eng.add_member(dev3)

    vp_product = Manager("Grace", "VP Product", 190000)
    vp_product.add_member(designer)

    ceo = Manager("Hank", "CEO", 300000)
    ceo.add_member(vp_eng)
    ceo.add_member(vp_product)

    print("---- Organization Chart ----")
    ceo.print_hierarchy("")

    print(f"\nTotal Payroll: ${ceo.get_salary()}")
    print(f"Total Headcount: {ceo.get_headcount()}")
    print(f"\nEngineering Payroll: ${vp_eng.get_salary()}")
    print(f"Engineering Headcount: {vp_eng.get_headcount()}")
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

class OrgComponent {
public:
    virtual int getSalary() = 0;
    virtual int getHeadcount() = 0;
    virtual void printHierarchy(string indent) = 0;
    virtual ~OrgComponent() {}
};

class Employee : public OrgComponent {
private:
    string name, title;
    int salary;

public:
    Employee(string name, string title, int salary)
        : name(name), title(title), salary(salary) {}

    int getSalary() override {
        return salary;
    }

    int getHeadcount() override {
        return 1;
    }

    void printHierarchy(string indent) override {
        cout << indent << "- " << name << " (" << title << ", $" << salary << ")" << endl;
    }
};

class Manager : public OrgComponent {
private:
    string name, title;
    int salary;
    vector<OrgComponent*> members;

public:
    Manager(string name, string title, int salary)
        : name(name), title(title), salary(salary) {}

    void addMember(OrgComponent* member) {
        members.push_back(member);
    }

    void removeMember(OrgComponent* member) {
        members.erase(remove(members.begin(), members.end(), member), members.end());
    }

    int getSalary() override {
        int total = salary;
        for (OrgComponent* member : members) total += member->getSalary();
        return total;
    }

    int getHeadcount() override {
        int count = 1;
        for (OrgComponent* member : members) count += member->getHeadcount();
        return count;
    }

    void printHierarchy(string indent) override {
        cout << indent << "+ " << name << " (" << title << ", $" << salary << ")" << endl;
        for (OrgComponent* member : members) member->printHierarchy(indent + "  ");
    }
};

int main() {
    Employee dev1("Alice", "Senior Engineer", 120000);
    Employee dev2("Bob", "Engineer", 95000);
    Employee dev3("Charlie", "Engineer", 90000);
    Employee designer("Diana", "Designer", 100000);

    Manager techLead("Eve", "Tech Lead", 140000);
    techLead.addMember(&dev1);
    techLead.addMember(&dev2);

    Manager vpEng("Frank", "VP Engineering", 200000);
    vpEng.addMember(&techLead);
    vpEng.addMember(&dev3);

    Manager vpProduct("Grace", "VP Product", 190000);
    vpProduct.addMember(&designer);

    Manager ceo("Hank", "CEO", 300000);
    ceo.addMember(&vpEng);
    ceo.addMember(&vpProduct);

    cout << "---- Organization Chart ----" << endl;
    ceo.printHierarchy("");

    cout << "\nTotal Payroll: $" << ceo.getSalary() << endl;
    cout << "Total Headcount: " << ceo.getHeadcount() << endl;
    cout << "\nEngineering Payroll: $" << vpEng.getSalary() << endl;
    cout << "Engineering Headcount: " << vpEng.getHeadcount() << endl;

    return 0;
}
```

</details>
<br/>


