
# Decorator Design Pattern
The Decorator Design Pattern is a structural pattern that lets you **dynamically add new behavior or responsibilities to objects without modifying their underlying code.**

- You want to **extend the functionality** of a class without subclassing it.
- You need to **compose behaviors at runtime**, in various combinations.
- You want to avoid bloated classes filled with `if-else logic` for optional features.

## The Problem
Imagine you are building a rich text rendering system, something like a simplified word processor or a markdown preview tool. At the core of your system is a `TextView` component that renders plain text on screen.

The first version works great. Then the requirements start growing:

- Support bold text
- Support italic text
- Support underlined text
- Support any combination of the above (bold + italic, bold + underline, italic + underline, all three together)


### The Naive Approach: Subclassing for Every Combination

![text16](..\assets\16.png)

<details>
<summary>Python</summary>

```python
from abc import ABC, abstractmethod

class TextView(ABC):
    @abstractmethod
    def render(self):
        pass

class PlainTextView(TextView):
    def __init__(self, text: str):
        self.text = text

    def render(self):
        print(self.text, end="")

# One subclass per feature...
class BoldTextView(TextView):
    def __init__(self, text: str):
        self.text = text

    def render(self):
        print(f"<b>{self.text}</b>", end="")

# One subclass per combination...
class BoldItalicTextView(TextView):
    def __init__(self, text: str):
        self.text = text

    def render(self):
        print(f"<i><b>{self.text}</b></i>", end="")

# BoldUnderlineTextView, ItalicUnderlineTextView,
# BoldItalicUnderlineTextView... it keeps going.
```

</details>

<details>
<summary>C++</summary>

```cpp
#include <iostream>
#include <string>

using namespace std;

class TextView {
public:
    virtual void render() = 0;
    virtual ~TextView() {}
};

class PlainTextView : public TextView {
private:
    string text;

public:
    PlainTextView(string text) : text(move(text)) {}

    void render() override {
        cout << text;
    }
};

// One subclass per feature...
class BoldTextView : public TextView {
private:
    string text;

public:
    BoldTextView(string text) : text(move(text)) {}

    void render() override {
        cout << "<b>" << text << "</b>";
    }
};

// One subclass per combination...
class BoldItalicTextView : public TextView {
private:
    string text;

public:
    BoldItalicTextView(string text) : text(move(text)) {}

    void render() override {
        cout << "<i><b>" << text << "</b></i>";
    }
};

// BoldUnderlineTextView, ItalicUnderlineTextView,
// BoldItalicUnderlineTextView... it keeps going.
```

</details>
<br/>


### What's Wrong with This Approach?

1. **Class Explosion**: For every new combination of features, you need a new subclass.

2. **Rigid Design**: You cannot change features at runtime. 

3. **Violates the Open/Closed Principle**


## The Decorator Pattern
The Decorator pattern is a structural pattern that attaches additional responsibilities to an object dynamically. Instead of extending a class through inheritance, it wraps the original object inside another object that adds the new behavior.

Two characteristics define the pattern:

1. *Same interface preservation*: Every decorator implements the same interface as the object it wraps. The client cannot tell whether it is talking to a plain object or a decorated one.

2. **Dynamic composition**: Decorators can be stacked at runtime in any combination and any order. You choose what to add and when, not at compile time.

## Class Diagram

![text17](../assets/17.png)


- **Component (e.g., TextView)** : Declares the common interface that both the core object and all decorators implement.

- **ConcreteComponent (e.g., PlainTextView)** : 
The base object that can be wrapped with decorators. It provides the default behavior.

- **Decorator (e.g, TextDecorator)** : 
An abstract class that implements the Component interface and holds a reference to another Component. It forwards calls to the wrapped object.

- **ConcreteDecorator (BoldDecorator, ItalicDecorator, etc.)** : 
Extend the base decorator to add new functionality before/after calling the wrapped component’s method.


## Implementing Decorator Pattern

Let’s implement the Decorator Pattern to enable flexible styling of text elements such as bold, italic, underline, and combinations of these.

Instead of creating a subclass for every combination, we create one class per feature and wrap them around each other. Each decorator adds its behavior and delegates the rest to the next object in the chain.

<details>
<summary>Python</summary>

```python
## Step 1: Define the Component Interface

from abc import ABC, abstractmethod

class TextView(ABC):
    @abstractmethod
    def render(self):
        pass

## Step 2: Implement the Concrete Component
class PlainTextView(TextView):
   def __init__(self, text):
       self.text = text

   def render(self):
       print(self.text, end="")

## Step 3: Create the Abstract Decorator
## This class implements **TextView** and holds a reference to another TextView. It is the bridge between the interface and the concrete decorators.
class TextDecorator(TextView):
   def __init__(self, inner):
       self.inner = inner

## Step 4: Implement Concrete Decorators

class BoldDecorator(TextDecorator):
   def __init__(self, inner):
       super().__init__(inner)

   def render(self):
       ## crazyyyyyy...........
       print("<b>", end="")
       self.inner.render()
       print("</b>", end="")

class ItalicDecorator(TextDecorator):
   def __init__(self, inner):
       super().__init__(inner)

   def render(self):
       print("<i>", end="")
       self.inner.render()
       print("</i>", end="")

class UnderlineDecorator(TextDecorator):
   def __init__(self, inner):
       super().__init__(inner)

   def render(self):
       print("<u>", end="")
       self.inner.render()
       print("</u>", end="")

## Step 5: Using the Decorator from the Client
def main():
    text = PlainTextView("Hello, World!")

    # Plain text
    print("Plain:                   ", end="")
    text.render()
    print()

    # Single decorator: Bold
    print("Bold:                    ", end="")
    bold_text = BoldDecorator(text)
    bold_text.render()
    print()

    # Two decorators: Italic + Underline
    print("Italic + Underline:      ", end="")
    italic_underline = UnderlineDecorator(ItalicDecorator(text))
    italic_underline.render()
    print()

    # Three decorators: Bold + Italic + Underline
    print("Bold + Italic + Underline: ", end="")
    all_styles = UnderlineDecorator(ItalicDecorator(BoldDecorator(text)))
    all_styles.render()
    print()


if __name__ == "__main__":
    main()
```

</details>

<details>
<summary>C++</summary>

```cpp
// Step 1: Define the Component Interface
class TextView {
public:
   virtual void render() = 0;
   virtual ~TextView() {}
};

// Step 2: Implement the Concrete Component
class PlainTextView : public TextView {
private:
   string text;

public:
   PlainTextView(string text) : text(text) {}

   void render() override {
       cout << text;
   }
};

// Step 3: Create the Abstract Decorator
// This class implements TextView and holds a reference to another TextView. It is the bridge between the interface and the concrete decorators.
class TextDecorator : public TextView {
protected:
   TextView* inner;

public:
   TextDecorator(TextView* inner) : inner(inner) {}
};

// Step 4: Implement Concrete Decorators
class ItalicDecorator : public TextDecorator {
public:
   ItalicDecorator(TextView* inner) : TextDecorator(inner) {}

   void render() override {
       cout << "<i>";
       // crazyyyyyy...........
       inner->render();
       cout << "</i>";
   }
};

class BoldDecorator : public TextDecorator {
public:
   BoldDecorator(TextView* inner) : TextDecorator(inner) {}

   void render() override {
       cout << "<b>";
       inner->render();
       cout << "</b>";
   }
};

class UnderlineDecorator : public TextDecorator {
public:
   UnderlineDecorator(TextView* inner) : TextDecorator(inner) {}

   void render() override {
       cout << "<u>";
       inner->render();
       cout << "</u>";
   }
};

// Step 5: Using the Decorator from the Client
int main() {
    PlainTextView text("Hello, World!");

    // Plain text
    cout << "Plain:                   ";
    text.render();
    cout << endl;

    // Single decorator: Bold
    cout << "Bold:                    ";
    BoldDecorator boldText(&text);
    boldText.render();
    cout << endl;

    // Two decorators: Italic + Underline
    cout << "Italic + Underline:      ";
    ItalicDecorator italic(&text);
    UnderlineDecorator italicUnderline(&italic);
    italicUnderline.render();
    cout << endl;

    // Three decorators: Bold + Italic + Underline
    cout << "Bold + Italic + Underline: ";
    BoldDecorator bold(&text);
    ItalicDecorator italicBold(&bold);
    UnderlineDecorator allStyles(&italicBold);
    allStyles.render();
    cout << endl;

    return 0;
}
```

</details>
<br/>


### Output:
```text
Plain:                   Hello, World!
Bold:                    <b>Hello, World!</b>
Italic + Underline:      <u><i>Hello, World!</i></u>
Bold + Italic + Underline: <u><i><b>Hello, World!</b></i></u>
```
