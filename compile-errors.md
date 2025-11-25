# C++ Tutorial: Learning Through Compiler Errors

This tutorial is designed to teach C++ by intentionally introducing common compiler errors at each level. We'll start with basic concepts, move to intermediate, and end with advanced topics. For each example:

- I'll show **broken code** that triggers a compiler error.
- Explain the error message (simplified, as actual messages vary by compiler like g++ or MSVC).
- Provide the **fixed code** and explain why it works.
- Include a brief exercise to reinforce the lesson.

Assume you're using a modern C++ compiler (C++11 or later). Compile with commands like `g++ file.cpp -o output` and run `./output`. Errors will appear during compilation.

## Basic Level: Fundamentals of Syntax and Control Flow

At this level, we'll cover variables, data types, loops, and functions. Focus on syntax basics.

### 1. Missing Semicolon
Broken code often forgets the `;` at the end of statements.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!"
    return 0;
}
```

**Expected Error:** Something like "error: expected ';' before 'return'". The compiler sees the statement as incomplete.

**Explanation:** Every C++ statement must end with a semicolon. It's like a period in English.

**Fixed Code:**
```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!";
    return 0;
}
```

**Exercise:** Write a program that prints your name without a semicolon after the print statement. Fix the error and add a second print line.

### 2. Undeclared Variable
Using a variable without declaring it first.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    x = 5;
    std::cout << x << std::endl;
    return 0;
}
```

**Expected Error:** "error: 'x' was not declared in this scope". The compiler doesn't know what `x` is.

**Explanation:** Variables must be declared with a type (e.g., `int`) before use. C++ is statically typed.

**Fixed Code:**
```cpp
#include <iostream>

int main() {
    int x = 5;  // Declaration and initialization
    std::cout << x << std::endl;
    return 0;
}
```

**Exercise:** Declare a `double` variable for a price, assign 9.99, and print it. Intentionally omit the declaration and fix it.

### 3. Type Mismatch
Assigning incompatible types.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    int num = "five";
    std::cout << num << std::endl;
    return 0;
}
```

**Expected Error:** "error: invalid conversion from 'const char*' to 'int'". Strings aren't integers.

**Explanation:** C++ enforces type safety. Use `std::string` for text.

**Fixed Code:**
```cpp
#include <iostream>
#include <string>

int main() {
    std::string word = "five";
    std::cout << word << std::endl;
    return 0;
}
```

**Exercise:** Try assigning a float (e.g., 3.14) to an `int`. Observe the warning/error, then use `double`.

### 4. Loop Syntax Error
Incorrect `for` loop structure.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 5) {
        std::cout << i << std::endl;
    }
    return 0;
}
```

**Expected Error:** "error: expected ';' before ')' token". The loop needs three parts: init, condition, increment.

**Explanation:** A `for` loop requires `for (init; condition; update)`.

**Fixed Code:**
```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 5; ++i) {  // Added increment
        std::cout << i << std::endl;
    }
    return 0;
}
```

**Exercise:** Create a `while` loop that counts down from 10. Forget the decrement and fix it.

### 5. Function Without Return Type
Defining a function incorrectly.

**Broken Code:**
```cpp
#include <iostream>

add(int a, int b) {
    return a + b;
}

int main() {
    std::cout << add(3, 4) << std::endl;
    return 0;
}
```

**Expected Error:** "error: ISO C++ forbids declaration of 'add' with no type". Functions need a return type.

**Explanation:** Specify what the function returns (e.g., `int`).

**Fixed Code:**
```cpp
#include <iostream>

int add(int a, int b) {  // Added 'int' return type
    return a + b;
}

int main() {
    std::cout << add(3, 4) << std::endl;
    return 0;
}
```

**Exercise:** Write a function to multiply two floats. Omit the return type, compile, and correct it.

## Intermediate Level: Pointers, Arrays, and Classes

Building on basics, we'll introduce memory management, arrays, and object-oriented concepts.

### 1. Dereferencing Null Pointer
Using a pointer without initialization.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    int* ptr;
    std::cout << *ptr << std::endl;  // Dereference
    return 0;
}
```

**Expected Error:** This might compile but crash at runtime. Compiler may warn: "warning: 'ptr' is used uninitialized".

**Explanation:** Pointers must point to valid memory. Initialize to `nullptr` or allocate.

**Fixed Code:**
```cpp
#include <iostream>

int main() {
    int* ptr = nullptr;
    if (ptr != nullptr) {
        std::cout << *ptr << std::endl;
    } else {
        std::cout << "Pointer is null!" << std::endl;
    }
    return 0;
}
```

**Exercise:** Allocate an `int` with `new`, dereference it without assignment, and fix by assigning a value.

### 2. Array Out-of-Bounds
Accessing beyond array size.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    int arr[3] = {1, 2, 3};
    std::cout << arr[3] << std::endl;  // Index 3 is out of bounds
    return 0;
}
```

**Expected Error:** Compiles but undefined behavior (runtime issue). Some compilers warn about bounds.

**Explanation:** Arrays are zero-indexed; size 3 means indices 0-2.

**Fixed Code:**
```cpp
#include <iostream>
#include <array>  // For safer std::array

int main() {
    std::array<int, 3> arr = {1, 2, 3};
    if (arr.size() > 3) {  // Check bounds
        std::cout << arr[3] << std::endl;
    } else {
        std::cout << "Out of bounds!" << std::endl;
    }
    return 0;
}
```

**Exercise:** Use a loop to print an array. Intentionally go one index too far and use bounds checking to fix.

### 3. Class Member Access Without Object
Accessing class members statically.

**Broken Code:**
```cpp
#include <iostream>

class MyClass {
public:
    int value = 10;
};

int main() {
    std::cout << value << std::endl;  // No object
    return 0;
}
```

**Expected Error:** "error: 'value' was not declared in this scope".

**Explanation:** Non-static members require an instance (object).

**Fixed Code:**
```cpp
#include <iostream>

class MyClass {
public:
    int value = 10;
};

int main() {
    MyClass obj;  // Create instance
    std::cout << obj.value << std::endl;
    return 0;
}
```

**Exercise:** Add a private member to a class. Try accessing it directly and fix with a getter method.

### 4. Missing Constructor
Implicit constructor issues with initialization.

**Broken Code:**
```cpp
#include <iostream>

class Person {
public:
    std::string name;
    int age;
};

int main() {
    Person p("Alice", 30);  // No matching constructor
    return 0;
}
```

**Expected Error:** "error: no matching function for call to 'Person::Person(const char [6], int)'".

**Explanation:** Default constructor doesn't take arguments; define one.

**Fixed Code:**
```cpp
#include <iostream>
#include <string>

class Person {
public:
    std::string name;
    int age;
    Person(std::string n, int a) : name(n), age(a) {}  // Constructor
};

int main() {
    Person p("Alice", 30);
    std::cout << p.name << " is " << p.age << std::endl;
    return 0;
}
```

**Exercise:** Create a class with a default constructor. Omit it and try initializing without args; add it back.

### 5. Inheritance Without Virtual Destructor
Base class without virtual destructor.

**Broken Code:**
```cpp
#include <iostream>

class Base {
public:
    ~Base() { std::cout << "Base destroyed" << std::endl; }
};

class Derived : public Base {
public:
    ~Derived() { std::cout << "Derived destroyed" << std::endl; }
};

int main() {
    Base* b = new Derived();
    delete b;  // Only calls Base destructor
    return 0;
}
```

**Expected Error:** Compiles, but runtime: Only "Base destroyed" prints (memory leak potential).

**Explanation:** For polymorphism, make base destructors virtual.

**Fixed Code:**
```cpp
#include <iostream>

class Base {
public:
    virtual ~Base() { std::cout << "Base destroyed" << std::endl; }  // Virtual
};

class Derived : public Base {
public:
    ~Derived() { std::cout << "Derived destroyed" << std::endl; }
};

int main() {
    Base* b = new Derived();
    delete b;  // Now calls both
    return 0;
}
```

**Exercise:** Add another derived class. Delete via base pointer without virtual and observe; add virtual.

## Advanced Level: Templates, STL, and Modern Features

Now, we'll tackle generics, standard library, exceptions, and concurrency.

### 1. Template Syntax Error
Incorrect template declaration.

**Broken Code:**
```cpp
#include <iostream>

template <T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << max(3, 5) << std::endl;
    return 0;
}
```

**Expected Error:** "error: 'T' does not name a type". Missing `typename` or `class`.

**Explanation:** Templates use `template <typename T>` or `template <class T>`.

**Fixed Code:**
```cpp
#include <iostream>

template <typename T>  // Added 'typename'
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << max(3, 5) << std::endl;
    return 0;
}
```

**Exercise:** Make a template function for min. Omit `typename` and fix; test with floats.

### 2. STL Container Misuse
Using vector without proper include or access.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    vector<int> v = {1, 2, 3};
    std::cout << v[3] << std::endl;
    return 0;
}
```

**Expected Error:** "error: 'vector' was not declared in this scope". Need `#include <vector>` and `std::`.

**Explanation:** STL requires specific headers and namespace.

**Fixed Code:**
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    if (v.size() > 3) {
        std::cout << v[3] << std::endl;
    } else {
        std::cout << "Out of bounds!" << std::endl;
    }
    return 0;
}
```

**Exercise:** Use `std::map`. Forget the include and fix; insert keys and access invalid one.

### 3. Exception Without Catch
Throwing without handling.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    throw std::runtime_error("Error occurred!");
    return 0;
}
```

**Expected Error:** Compiles, but runtime: Uncaught exception, program terminates.

**Explanation:** Use `try-catch` to handle exceptions.

**Fixed Code:**
```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("Error occurred!");
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }
    return 0;
}
```

**Exercise:** Throw a custom exception class. Omit catch and observe termination; add try-catch.

### 4. Lambda Capture Error
Incorrect capture in lambda.

**Broken Code:**
```cpp
#include <iostream>

int main() {
    int x = 10;
    auto lambda = []() { return x; };  // x not captured
    std::cout << lambda() << std::endl;
    return 0;
}
```

**Expected Error:** "error: 'x' is not captured".

**Explanation:** Lambdas need to capture variables: `[=]` for by-value, `[&]` for by-reference.

**Fixed Code:**
```cpp
#include <iostream>

int main() {
    int x = 10;
    auto lambda = [x]() { return x; };  // Capture by value
    std::cout << lambda() << std::endl;
    return 0;
}
```

**Exercise:** Capture by reference and modify `x` inside lambda. Try without capture and fix.

### 5. Multithreading Without Mutex
Race condition in threads.

**Broken Code:**
```cpp
#include <iostream>
#include <thread>

int counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        ++counter;  // Race condition
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << counter << std::endl;  // May not be 2000
    return 0;
}
```

**Expected Error:** Compiles, but runtime: Inconsistent output due to data race.

**Explanation:** Use `std::mutex` to protect shared data.

**Fixed Code:**
```cpp
#include <iostream>
#include <thread>
#include <mutex>

int counter = 0;
std::mutex mtx;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        mtx.lock();
        ++counter;
        mtx.unlock();
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << counter << std::endl;  // Always 2000
    return 0;
}
```

**Exercise:** Add more threads without mutex and observe varying results; add mutex and verify consistency.

This tutorial builds progressively, so practice each section before moving on. Experiment with variations to see new errors—it's the best way to learn! If you encounter real errors, search for the message online or ask for clarification.
