Long Method and Class: you should take this code and put it in a new method / Splitting it up.

If you have a large variety of primitive fields, it may be possible to logically group some of them into their own class

Long Parameter List: 

Sometimes different parts of the code contain identical groups of variables.

# 1. Composing Methods
## Extract Method
### Problem
You have a code fragment that can be grouped together.

### Solution
Move this code to a separate new method (or function) and replace the old code with a call to the method.



## Inline Method
### Problem
When a method body is more obvious than the method itself, use this technique.

By minimizing the number of unneeded methods, you make the code more straightforward.

```py
class PizzaDelivery:
    # ...
    def getRating(self):
        return 2 if self.moreThanFiveLateDeliveries() else 1

    def moreThanFiveLateDeliveries(self):
        return self.numberOfLateDeliveries > 5
```

### Solution
Replace calls to the method with the method’s content and delete the method itself. 

```py
class PizzaDelivery:
  # ...
  def getRating(self):
    return 2 if self.numberOfLateDeliveries > 5 else 1
```

## Extract Variable
### Problem
You have an expression that’s hard to understand.

```py
def renderBanner(self):
    if (self.platform.toUpperCase().indexOf("MAC") > -1) and \
       (self.browser.toUpperCase().indexOf("IE") > -1) and \
       self.wasInitialized() and (self.resize > 0):
        # do something
```

### Solution
Place the result of the expression or its parts in separate variables that are self-explanatory.

More variables are present in your code. But this is counterbalanced by the ease of reading your code.

```py
def renderBanner(self):
    isMacOs = self.platform.toUpperCase().indexOf("MAC") > -1
    isIE = self.browser.toUpperCase().indexOf("IE") > -1
    wasResized = self.resize > 0

    if isMacOs and isIE and self.wasInitialized() and wasResized:
        # do something
```

## Inline Temp
### Problem
You have a temporary variable that’s assigned the result of a simple expression and nothing more.

```py
def hasDiscount(order):
    basePrice = order.basePrice()
    return basePrice > 1000
```

### Solution
Replace the references to the variable with the expression itself.

```py
def hasDiscount(order):
    return order.basePrice() > 1000
```

> ##### Drawbacks: Sometimes seemingly useless temps are used to cache the result of an expensive operation that’s reused several times. So before using this refactoring technique, make sure that simplicity won’t come at the cost of performance.


## Replace Temp with Query
### Problem
You place the result of an expression in a local variable for later use in your code.

```py
def calculateTotal():
    basePrice = quantity * itemPrice
    if basePrice > 1000:
        return basePrice * 0.95
    else:
        return basePrice * 0.98
```


### Solution
Move the entire expression to a separate method and return the result from it. Query the method instead of using a variable. Incorporate the new method in other methods, if necessary.

```py
def calculateTotal():
    if basePrice() > 1000:
        return basePrice() * 0.95
    else:
        return basePrice() * 0.98

def basePrice():
    return quantity * itemPrice
```
#### Performance

This refactoring may prompt the question of whether this approach is liable to cause a performance hit. The honest answer is: yes, it is, since the resulting code may be burdened by querying a new method. But with today’s fast CPUs and excellent compilers, the burden will almost always be minimal. By contrast, readable code and the ability to reuse this method in other places in program code—thanks to this refactoring approach—are very noticeable benefits.

Nonetheless, if your temp variable is used to cache the result of a truly time-consuming expression, you may want to stop this refactoring after extracting the expression to a new method.

## Split Temporary Variable
### Problem
You have a local variable that’s used to store various intermediate values inside a method (except for cycle variables).

```py
temp = 2 * (height + width)
print(temp)
temp = height * width
print(temp)
```

### Solution
Use different variables for different values. Each variable should be responsible for only one particular thing.

```py
perimeter = 2 * (height + width)
print(perimeter)
area = height * width
print(area)
```

## Remove Assignments to Parameters
### Problem
Some value is assigned to a parameter inside method’s body.

```py
def discount(inputVal, quantity):
    if quantity > 50:
        inputVal -= 2
    # ...
```

### Solution
Use a local variable instead of a parameter.

```py
def discount(inputVal, quantity):
    result = inputVal
    if quantity > 50:
        result -= 2
    # ...
```

## Replace Method with Method Object
### Problem
You have a long method in which the local variables are so intertwined that you can’t apply Extract Method.

```py
class Order:
    # ...
    def price(self):
        primaryBasePrice = 0
        secondaryBasePrice = 0
        tertiaryBasePrice = 0
        # Perform long computation.
```

### Solution
Transform the method into a separate class so that the local variables become fields of the class. Then you can split the method into several methods within the same class.

```py
class Order:
    # ...
    def price(self):
        return PriceCalculator(self).compute()


class PriceCalculator:
    def __init__(self, order):
        self._primaryBasePrice = 0
        self._secondaryBasePrice = 0
        self._tertiaryBasePrice = 0
        # Copy relevant information from the
        # order object.

    def compute(self):
        # Perform long computation.
```

Firstly, this allows isolating the problem at the class level. Secondly, it paves the way for splitting a large and unwieldy method into smaller ones that wouldn’t fit with the purpose of the original class anyway.

##### Benefits: Isolating a long method in its own class allows stopping a method from ballooning in size.  
##### Drawbacks: Another class is added, increasing the overall complexity of the program.


## Substitute Algorithm
### Problem
So you want to replace an existing algorithm with a new one?

```py
def foundPerson(people):
    for i in range(len(people)):
        if people[i] == "Don":
            return "Don"
        if people[i] == "John":
            return "John"
        if people[i] == "Kent":
            return "Kent"
    return ""
```

### Solution
Replace the body of the method that implements the algorithm with a new algorithm.

```py
def foundPerson(people):
    candidates = ["Don", "John", "Kent"]
    return people if people in candidates else ""

```


# 2. Moving Features between Objects

## Move Method

### Problem
A method is used more in another class than in its own class.

### Solution
Create a new method in the class that uses the method the most, then move code from the old method to there. Turn the code of the original method into a reference to the new method in the other class or else remove it entirely.

## Move Field
### Problem
A field is used more in another class than in its own class.

### Solution
Create a field in a new class and redirect all users of the old field to it.

## Extract Class
### Problem
When one class does the work of two, awkwardness results.
### Solution
Instead, create a new class and place the fields and methods responsible for the relevant functionality in it.

## Inline Class
### Problem
A class does almost nothing and isn’t responsible for anything, and no additional responsibilities are planned for it.

### Solution
Move all features from the class to another one.

## Hide Delegate
### Problem
The client gets object B from a field or method of object А. Then the client calls a method of object B.

### Solution
Create a new method in class A that delegates the call to object B. Now the client doesn’t know about, or depend on, class B.

## Remove Middle Man
### Problem
A class has too many methods that simply delegate to other objects.

### Solution
Delete these methods and force the client to call the end methods directly.


## Introduce Foreign Method
### Problem
A utility class doesn’t contain the method that you need and you can’t add the method to the class.

### Solution
Add the method to a client class and pass an object of the utility class to it as an argument.

## Introduce Local Extension
### Problem
A utility class doesn’t contain some methods that you need. But you can’t add these methods to the class.

### Solution
Create a new class containing the methods and make it either the child or wrapper of the utility class.

