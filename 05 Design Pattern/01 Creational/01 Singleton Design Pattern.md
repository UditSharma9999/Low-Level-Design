# Singleton Design Pattern

**It guarantees a class has only one instance and provides a global point of access to it.**

**Global access**: Any component can reach the instance without needing it passed through constructors or method parameters.

## Specific Examples:
- **Logger Classes**: Many logging frameworks use the Singleton pattern to provide a global logging object. This ensures that log messages are consistently handled and written to the same output stream.

- **Cache Objects**: In-memory caches are often implemented as Singletons to provide a single point of access for cached data across the application.

- **File System**: File systems often use Singleton objects to represent the file system and provide a unified interface for file operations.

## Class Diagram
To implement the singleton pattern, we must prevent external objects from creating instances of the singleton class. Only the singleton class should be permitted to create its own objects.

Additionally, we need to provide a method for external objects to access the singleton object.

![text12](/assets/12.png)


## Implementation

Singleton implementation varies across languages. The central challenge is thread safety: if two threads call `getInstance()` simultaneously when the instance has not been created yet, both might create separate instances.

We will start with the simplest (but broken) approach and progressively improve it. After the shared implementations, we cover language-specific idioms that are recommended for production use.

### 1. Lazy Initialization (Not Thread-Safe)

This approach creates the singleton instance only when it is needed, saving resources if the singleton is never used in the application.

<details>
<summary>Python</summary>

```python
class LazySingleton:
    # Holds the single shared instance (initially not created)
    _instance = None

    # Constructor prevents direct creation if instance already exists
    def __init__(self):
        if LazySingleton._instance is not None:
            raise Exception("Use get_instance() instead.")

    # Global access point to get the Singleton instance
    @staticmethod
    def get_instance():

        # Create the instance only when first requested (lazy initialization)
        if LazySingleton._instance is None:
            LazySingleton._instance = LazySingleton()

        # Return the shared instance
        return LazySingleton._instance
```

</details>

<details>
<summary>C++</summary>

```cpp
class LazySingleton {
private:
    // Holds the single shared instance (initially not created)
    static LazySingleton* instance;

    // Private constructor prevents creating objects from outside the class
    LazySingleton() {}

public:

    // Global access point to get the Singleton instance
    static LazySingleton* getInstance() {

        // Create the instance only when first requested (lazy initialization)
        if (instance == nullptr) {
            instance = new LazySingleton();
        }

        // Return the shared instance
        return instance;
    }
};
```

</details>
<br/>

### 2. Thread-Safe Singleton

This approach extends lazy initialization by ensuring the Singleton is safe to use in multi-threaded environments.

When multiple threads try to access the instance at the same time, **synchronization (or locking)** ensures that only one thread can create the object, while others wait.

<details>
<summary>Python</summary>

```python
class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __init__(self):
        if ThreadSafeSingleton._instance is not None:
            raise Exception("Use get_instance() instead.")

    @staticmethod
    def get_instance():
        with ThreadSafeSingleton._lock:
            if ThreadSafeSingleton._instance is None:
                ThreadSafeSingleton._instance = ThreadSafeSingleton()
        return ThreadSafeSingleton._instance
```

</details>

<details>
<summary>C++</summary>

```cpp
class ThreadSafeSingleton {
private:
    static ThreadSafeSingleton* instance;
    static mutex lock;

    ThreadSafeSingleton() {}

public:
    static ThreadSafeSingleton* getInstance() {
        lock_guard<mutex> guard(lock);
        if (instance == nullptr) {
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
};
```

</details>
<br/>

- When a thread enters the protected section, it acquires the lock. Other threads must wait until the lock is released.

- This guarantees that only one instance is created, even under concurrent access.

This approach is correct but has a **performance cost**: every call to `getInstance()` acquires a lock, even after the instance has been created. Once the instance exists, there is no reason to synchronize. The next approach fixes this.

### 3. Double-Checked Locking

Double-checked locking reduces the performance overhead by only synchronizing during the first object creation. After the instance exists, threads skip the lock entirely.


<details>
<summary>Python</summary>

```python
class DoubleCheckedSingleton:
    # Holds the single shared instance
    _instance = None
    # Lock used only during first-time creation
    _lock = threading.Lock()

    # Constructor prevents accidental direct creation
    def __init__(self):
        if DoubleCheckedSingleton._instance is not None:
            raise Exception("Use get_instance() instead.")

    # Global access point to get the Singleton instance
    @staticmethod
    def get_instance():
        
		# Fast path: first check without locking
        if DoubleCheckedSingleton._instance is None:
            # Lock only when the instance might need to be created
            with DoubleCheckedSingleton._lock:
                # Second check inside the lock (prevents double creation)
                if DoubleCheckedSingleton._instance is None:
                    DoubleCheckedSingleton._instance = DoubleCheckedSingleton()

        # Return the shared instance
        return DoubleCheckedSingleton._instance
```

</details>

<details>
<summary>C++</summary>

```cpp
class DoubleCheckedSingleton {
private:
    // Holds the single shared instance (needs safe publication in C++)
    static DoubleCheckedSingleton* instance;
    // Lock used only during first-time creation
    static mutex lock;

    // Private constructor prevents external instantiation
    DoubleCheckedSingleton() {}

public:
    // Global access point to get the Singleton instance
    static DoubleCheckedSingleton* getInstance() {
        
		// Fast path: first check without locking
        if (instance == nullptr) {
            // Lock only when the instance might need to be created
            lock_guard<mutex> guard(lock);
            // Second check inside the lock (prevents double creation)
            if (instance == nullptr) {
                instance = new DoubleCheckedSingleton();
            }
        }

        // Return the shared instance
        return instance;
    }
};
```

</details>
<br/>

### 4. Eager Initialization

In eager initialization, the Singleton instance is created **as soon as the class/module is loaded**, before any thread can access it. That makes it inherently thread-safe without explicit locks, because initialization happens once during load/initialization.

<details>
<summary>Python</summary>

```python
class EagerSingleton:
    # Holds the single shared instance (created immediately at module load time)
    _instance = None

    # Private-like constructor guard to prevent direct instantiation
    def __init__(self):
        if EagerSingleton._instance is not None:
            raise Exception("Use get_instance() instead.")

    # Global access point to get the Singleton instance
    @staticmethod
    def get_instance():
        # Return the already-created shared instance
        return EagerSingleton._instance
```

</details>

<details>
<summary>C++</summary>

```cpp
class EagerSingleton {
private:
    // Holds the single shared instance (created immediately during static initialization)
    static EagerSingleton* instance;

    // Private constructor prevents creating objects from outside the class
    EagerSingleton() {}

public:
    // Global access point to get the Singleton instance
    static EagerSingleton* getInstance() {
        // Return the already-created shared instance
        return instance;
    }
};
```

</details>
<br/>



- A class-level/static variable holds the single shared instance.

- The instance is created during class/module initialization, not on first use.

- No locks are needed because the runtime initializes static/class state once.

### 5. Language Specific Implementations


#### Python
In Python, modules are imported once. The interpreter caches modules in **sys.modules** after the first import, so subsequent imports get the same module object. This makes module-level instances natural singletons:

```py
# config_manager.py

class _ConfigManager:
    def __init__(self):
        self._settings = {}

    def set(self, key, value):
        self._settings[key] = value

    def get(self, key, default=None):
        return self._settings.get(key, default)


# Created once when module is first imported
config = _ConfigManager()


# Usage (from other files):
# from config_manager import config
# config.set("debug", True)
```

The underscore prefix on _ConfigManager signals that the class is an implementation detail. Users import config, not the class.

This is the most Pythonic singleton approach: simple, clear, and naturally thread-safe at the module loading level. The Python community generally prefers this over class-based singleton tricks.


##### `__new__` Override
For cases where you want Singleton() to always return the same instance (class-based interface):

```py
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        # Warning: __init__ is called every time Singleton() is called
        pass

# Usage
s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # True
```

##### Metaclass Singleton
A metaclass-based approach separates the singleton logic from the class itself. Any class using SingletonMeta as its metaclass becomes a singleton automatically:

```PY
import threading

class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                if cls not in cls._instances:
                    instance = super().__call__(*args, **kwargs)
                    cls._instances[cls] = instance
        return cls._instances[cls]


class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = "connected"


# Usage
db1 = Database()
db2 = Database()
assert db1 is db2  # True
```


The metaclass intercepts the class call (Database()), checks if an instance already exists, and returns the existing one or creates a new one.


##### Decorator Singleton
A function decorator that caches the first instance and returns it on subsequent calls:

```py
import threading

def singleton(cls):
    instances = {}
    lock = threading.Lock()

    def get_instance(*args, **kwargs):
        if cls not in instances:
            with lock:
                if cls not in instances:
                    instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance


@singleton
class AppConfig:
    def __init__(self):
        self.settings = {}


# Usage
c1 = AppConfig()
c2 = AppConfig()
assert c1 is c2  # True
```

The decorator replaces the class with a wrapper function. Calling `AppConfig()` actually calls `get_instance()`, which returns the cached instance. 

#### CPP Meyers' Singleton (Recommended)
In C++11 and later, the standard guarantees that static local variables are initialized in a thread-safe manner. This makes the Meyers' Singleton the idiomatic approach in modern C++:


```cpp
class Singleton {
private:
    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton& getInstance() {
        static Singleton instance;  // Thread-safe in C++11+
        return instance;
    }

    void doSomething() {
        std::cout << "Singleton operation" << std::endl;
    }
};

// Usage
Singleton::getInstance().doSomething();
```

The C++11 standardguarantees that if multiple threads attempt to initialize the same static local variable concurrently, initialization occurs exactly once. The compiler inserts the necessary synchronization.


