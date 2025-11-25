### Difference Between Compile-Time and Runtime

| Aspect              | Compile-Time                                      | Runtime                                          |
|---------------------|---------------------------------------------------|--------------------------------------------------|
| **When it happens** | Before the program runs, during compilation       | When the program is actually executing           |
| **Who performs it** | The compiler (e.g., gcc, javac, clang, csc)       | The operating system + runtime environment (JVM, .NET CLR, Python interpreter, etc.) |
| **Errors detected** | Compile-time errors (syntax errors, type errors, etc.) | Runtime errors/exceptions (NullPointerException, division by zero, file not found, etc.) |
| **Performance impact** | Affects only build time                           | Directly affects how fast and correctly the program runs |
| **Can the code change it?** | No — once compiled, it's fixed (except generics type erasure in Java) | Yes — depends on user input, environment, etc. |
| **Optimization**    | Compiler can optimize (constant folding, inlining, dead code elimination) | JIT compiler (in Java/.NET) can optimize at runtime |

### Examples in Different Languages

#### 1. C/C++ (Classic compiled languages)

```cpp
// Compile-time example
constexpr int SIZE = 10;           // Decided at compile time
int arr[SIZE];                     // Array size must be known at compile time

int arr2[0];                       // Compile-time ERROR: array size must be > 0

// Runtime example
int n;
std::cin >> n;                     // Value known only when user types it
int* dynamic = new int[n];         // Size decided at runtime

int x = 10 / 0;                    // Compile-time: OK (just a warning sometimes)
                   // Runtime: Crash (division by zero)
```

#### 2. Java

```java
// Compile-time
List<String> list = new ArrayList<>(); // OK
List<int> wrong = new ArrayList<>();    // Compile-time ERROR (generics don't allow primitives)

// Runtime
String s = null;
System.out.println(s.length());        // Compiles fine → NullPointerException at runtime

Integer[] arr = new Integer[5];
arr[10] = 100;                         // Compiles fine → ArrayIndexOutOfBoundsException at runtime
```

#### 3. Python (Interpreted – but still has the concept)

```python
# "Compile-time" in Python = when the bytecode (.pyc) is generated (very early, still before execution)
def func():
    return undeclared_var    # No error yet! Only when function is called

# Runtime errors
print(10 / 0)                # ZeroDivisionError at runtime
my_list = [1, 2, 3]
print(my_list[99])           # IndexError at runtime

# Some things are caught when the module is imported (closer to compile-time)
x = "hello"hello  # SyntaxError when the file is parsed/compiled to bytecode
```

#### 4. Constant Folding (Compile-time optimization)

```java
// Java / C++ / C#
int a = 5 + 3 * 2;     // Compiler calculates 5 + 6 = 11 at compile time
final int x = 10;      // Java: treated as compile-time constant
int b = x + 20;        // b is also computed at compile time → 30
```

#### 5. Templates / Generics (Compile-time polymorphism)

```cpp
// C++ template – code generated at compile time for each type
template<typename T>
T add(T a, T b) { return a + b; }

auto x = add(5, 3);     // instantiates add<int>
auto y = add(3.5, 2.1); // instantiates add<double>
```

```java
// Java generics – type erasure happens at compile time
List<String> strings = new ArrayList<>();
strings.add("hello");
// strings.add(123);    // Compile-time error
```

### Summary Table of Common Scenarios

| Scenario                          | Detected at Compile-Time? | Detected at Runtime? | Example Language |
|-----------------------------------|---------------------------|-----------------------|------------------|
| Syntax error                      | Yes                       | No                    | All              |
| Type mismatch (statically typed)  | Yes                       | No                    | Java, C++, C#    |
| Division by zero (constant)       | Sometimes (warning/error) | Yes (crash)           | C/C++            |
| Division by zero (variable)       | No                        | Yes                   | All              |
| Array index out of bounds         | Sometimes (if constant)   | Usually Yes           | Java/C# (yes), C/C++ (no) |
| Calling undefined function        | Yes (if not declared)     | Yes (if dynamic)      | C++ vs Python    |
| Memory allocation size            | No (if variable)          | Yes                   | All              |

### Key Takeaway
- **Compile-time** = "Can the compiler figure this out without running the program?" → Yes → do it early, fail early, optimize early."
- **Runtime** = "Only known when the program is actually running → must be handled while executing (exceptions, dynamic dispatch, reflection, etc.)."

Understanding this difference is crucial for writing safe, performant, and maintainable code!
