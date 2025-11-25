# C++ Tutorial – Part 2: Runtime Errors & Memory Leaks  
(Learn by Crashing, Valgrinding, and Fixing the Invisible Bugs)

Now that you survived the compiler, meet the real enemies: bugs that **compile perfectly** but explode at runtime.

We’ll trigger every classic runtime disaster on purpose, show you exactly what happens, and teach you how to detect and destroy them forever.

Tools you need right now:
```bash
# Ubuntu/Debian
sudo apt install valgrind gdb clang-tidy

# macOS
brew install valgrind gdb llvm

# Windows
# Use Visual Studio + built-in debugger, or WSL + Valgrind
```

## LEVEL 1: Use-After-Free (The #1 C++ Killer)

```cpp
int* create() {
    int x = 42;
    return &x;               // ← local variable address
}

int main() {
    int* p = create();
    std::cout << *p << '\n'; // Undefined Behavior → often prints 42... then crashes later
}
```

**Valgrind says:**
```
Invalid read of size 4
   at 0x... main (...)
   by 0x... create (...)
Address 0x... is 4 bytes inside a block of size 8 alloc'd ... and freed
==12345==    by 0x... create (...)
```

**Fix:** Never return pointers/references to stack objects.

## LEVEL 2: Double Free & Memory Corruption

```cpp
int main() {
    int* p = new int(42);
    delete p;
    delete p;                // ← double delete
}
```

**Valgrind:**
```
Invalid free() / delete
   at 0x... operator delete (...)
   by 0x... main (...)
Address 0x... is 0 bytes inside a block of size 8 previously freed
```

**Even worse version (real-world corruption):**
```cpp
int* p = new int[10];
delete p;        // ← delete instead of delete[] !!!
delete[] p;      // now heap is corrupted
```

**Valgrind:**
```
Mismatched free() / delete / delete []
```

**Rule:** `new` → `delete`, `new[]` → `delete[]`. Never mix.

## LEVEL 3: Classic Memory Leaks

```cpp
void leak_forever() {
    while(true) {
        new int[1000];       // 4 KB leak per iteration
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}
```

**Valgrind summary after 5 seconds:**
```
HEAP SUMMARY:
    definitely lost: 19,531,200 bytes in 4,882 blocks
    indirectly lost: 0 bytes
    possibly lost:    0 bytes
    still reachable:  0 bytes
```

**Real-world leak that fools beginners:**
```cpp
void false_positive() {
    static int* p = new int(42);  // never deleted → leak
}
```

**Valgrind marks it as “still reachable”** → easy to ignore → real leak in production.

**Best practice:** In modern C++, raw `new`/`delete` = code smell.

## LEVEL 4: Buffer Overflows (Stack & Heap)

### Stack overflow
```cpp
void boom(int n) {
    char buffer[16];
    sprintf(buffer, "Number is %d and more text that overflows", n);
    std::cout << buffer << '\n';
    boom(n+1);               // + recursion = segfault in seconds
}
```

### Heap overflow
```cpp
int* p = new int[10];
p[20] = 666;                 // 10 ints after the end → corrupts heap metadata
delete[] p;                  // → immediate crash or later corruption
```

**Valgrind catches both instantly:**
```
Invalid write of size 4
   at 0x... main (...)
Address 0x... is 40 bytes past the end of allocated block
```

## LEVEL 5: Uninitialized Memory (The Silent Killer)

```cpp
int main() {
    int x;                       // not initialized!
    if (x == 42)                 // 50% chance on some systems
        std::cout << "The universe is broken\n";
}
```

**Valgrind:**
```
Conditional jump or move depends on uninitialised value(s)
   at 0x... main (...)
```

**Real example that passes tests 99% of the time:**
```cpp
bool is_positive() {
    int value;
    std::cin >> value;           // what if user presses Ctrl+D?
    return value > 0;            // uses garbage if input fails
}
```

## LEVEL 6: The Most Common Leak Patterns in Real Code

| Pattern                        | Leak Type           | How to Spot (Valgrind)         | Modern Fix                     |
|--------------------------------|---------------------|--------------------------------|--------------------------------|
| `new` without `delete`         | Definite leak       | "definitely lost"              | `std::unique_ptr`              |
| Forgotten `delete` in exception| Definite leak       | leak only on exception path    | RAII (any smart pointer)       |
| `malloc()` in C library        | Definite leak       | shows up as leak               | `free()` or wrap in unique_ptr |
| Thread-local `new`             | Still reachable     | often ignored                  | `thread_local` + RAII          |

## LEVEL 7: Modern C++ Solutions (Never Leak Again)

### Replace ALL raw pointers with smart pointers

```cpp
#include <memory>
#include <vector>

// Old dangerous code
std::vector<int*>* create_leaks() {
    auto* v = new std::vector<int*>;
    v->push_back(new int(1));
    v->push_back(new int(2));
    return v;                     // ← leak if exception thrown
}

// New bulletproof code
auto create_safe() {
    auto v = std::make_unique<std::vector<std::unique_ptr<int>>>();
    v->push_back(std::make_unique<int>(1));
    v->push_back(std::make_unique<int>(2));
    return v;                     // no leak possible
}
```

### Custom RAII class example

```cpp
class FileHandle {
    FILE* f;
public:
    explicit FileHandle(const char* name) : f(fopen(name, "r")) {
        if (!f) throw std::runtime_error("open failed");
    }
    ~FileHandle() { if(f) fclose(f); }
    FileHandle(const FileHandle&) = delete;
    // ... move ctor if needed
};
```

## LEVEL 8: Runtime Detection Tools Cheat Sheet

| Tool           | Catches                                  | How to run                              |
|----------------|------------------------------------------|-----------------------------------------|
| Valgrind       | All memory errors, leaks, UAF            | `valgrind --leak-check=full ./a.out`    |
| AddressSanitizer (ASan) | Faster than Valgrind, catches same bugs | `g++ -fsanitize=address -g`             |
| UndefinedBehaviorSanitizer (UBSan) | Signed overflow, null deref, etc.     | `g++ -fsanitize=undefined`              |
| clang-tidy     | Static analysis (some leaks)             | `clang-tidy file.cpp -checks=*`         |
| Visual Studio  | Excellent debugger + CRT checks          | `_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF)`  |

**My daily compile command (2025):**
```bash
g++ -std=c++23 -Wall -Wextra -Werror -fsanitize=address,undefined -g -O1 *.cpp
```

## FINAL CHALLENGE: Find & Fix All Runtime Bugs

```cpp
// runtime-monster.cpp — compile with ASan/Valgrind and fix everything
#include <iostream>
#include <vector>

int* make_dangling() {
    int x = 42;
    return &x;
}

struct Bad {
    int* data;
    Bad() { data = new int[10]; }
    ~Bad() { delete data; }               // wrong delete!
};

int main() {
    int* p = make_dangling();
    std::cout << *p << '\n';              // UAF

    {
        Bad b;
        // b goes out of scope → wrong delete
    }

    int* leak = new int(100);
    leak = nullptr;                       // definite leak

    std::vector<int> v(1000);
    std::cout << v[10000] << '\n';        // out of bounds (UB, sometimes no crash)

    return 0;
}
```

Run with AddressSanitizer → it will scream at every line.

**After fixing:**
```cpp
#include <iostream>
#include <memory>
#include <vector>

auto make_safe() { return std::make_unique<int>(42); }

struct Good {
    std::unique_ptr<int[]> data;
    Good() : data(std::make_unique<int[]>(10)) {}
    // destructor automatically correct
};

int main() {
    auto p = make_safe();
    std::cout << *p << '\n';

    {
        Good g;
    } // auto cleanup

    // no manual new → no leaks possible

    std::vector<int> v(1000);
    std::cout << v.at(10000) << '\n';  // .at() throws instead of UB
}
```

## Summary: You Are Now Immune to 95% of C++ Bugs

| Bug Type               | Detection Tool       | Prevention Technique           |
|------------------------|----------------------|--------------------------------|
| Use-after-free         | ASan / Valgrind      | Smart pointers                 |
| Double / mismatched delete | Valgrind / ASan   | Never use raw new/delete       |
| Memory leaks           | Valgrind --leak-check=full | RAII + unique_ptr       |
| Buffer overflow        | ASan                 | std::span, .at(), bounds checking |
| Uninitialized vars    | Valgrind / ASan      | =default initialization, {}    |

**One rule to rule them all (2025+):**
> If you type the word `new` or `delete` in production code → you’re doing it wrong.

Now go run your old projects under AddressSanitizer.  
You will cry.  
Then you will fix them.  
And you will become a C++ wizard.

Happy (leak-free) hacking! 🧙‍♂️
