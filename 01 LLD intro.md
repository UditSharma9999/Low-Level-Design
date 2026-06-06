# Chapter 1
## Relationships Between Classes

### Key relationships include:
1. **Association**: A general "uses-a" relationship. A `Doctor` uses a `Stethoscope`.
2. **Aggregation (Weak "has-a")**: An object contains other objects, but they can exist independently. A `Department` has `Professors`. If the department is closed, the professors still exist.
3. **Composition (Strong "has-a")**: An object is composed of other objects, and their lifecycles are tied. A `House` is composed of `Rooms`. If you demolish the house, the rooms are destroyed with it.

4. **Inheritance ("is-a")**: A class inherits properties and behaviors from a parent. A Car is a Vehicle.

### Method Signatures
Once your classes and relationships are defined, the next step is deciding how they behave using methods. A well-designed method signature is self-documenting and intuitive.

You’ll need to decide:

- What methods should each class expose?
- What are the method's input parameters, and return types?
- Should the method be public, private, or protected?
- What exceptions might they throw?
- Is it synchronous or asynchronous?



## Types of LLD Interviews

![text1](/assets/1.png)
<br/>
![text1](/assets/2.png)


