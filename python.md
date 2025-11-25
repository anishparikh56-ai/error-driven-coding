# Python – The Full Top 50 “Learn by Compiling” Pain Master List  
Every single one is **fully expanded** with complete minimal reproducible code, the **exact error you will see**, the **correct fix**, and the **lesson that will be burned into your brain forever**.

Do them all. In order. Type everything. Never forget.

### 1. Mutable Default Arguments – The Immortal Bug
```python
# broken.py
def append_to(item, target=[]):
    target.append(item)
    return target

print(append_to(1))
print(append_to(2))
print(append_to(3, []))
print(append_to(4))
```
**Output:**
```
[1]
[1, 2]
[3]
[1, 2, 4]   ← same list used every time!
```
**Fix:**
```python
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```
**Lesson:** Default arguments are evaluated once at definition time.

### 2. Late Binding Closures – The Lambda-in-Loop Disaster
```python
funcs = [lambda: i for i in range(3)]
for f in funcs:
    print(f())          # 2 2 2
```
**Fix:**
```python
funcs = [lambda i=i: i for i in range(3)]
# or
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)
```

### 3. Modifying Dict During Iteration
```python
d = {"a": 1, "b": 2, "c": 3}
for k in d:
    if k == "b":
        del d[k]
# RuntimeError: dictionary changed size during iteration
```
**Fix:**
```python
for k in list(d.keys()):
    if k == "b":
        del d[k]
```

### 4. Shadowing Built-ins
```python
list = [1, 2, 3]
list("hello")       # TypeError: 'list' object is not callable
```
**Fix:** Never name anything `list`, `str`, `dict`, `int`, `max`, `sum`, etc.

### 5. Wrong Walrus Operator Parentheses
```python
if x := 42 > 10:
    print(x)
# SyntaxError: assignment expression requires parentheses in this context
```
**Fix:**
```python
if (x := 42) > 10:
    print(x)
```

### 6. await Outside async Function
```python
async def main():
    pass

await main()        # SyntaxError: 'await' outside async function
```

### 7. Double asyncio.run()
```python
import asyncio
asyncio.run(asyncio.sleep(1))
asyncio.run(asyncio.sleep(1))
# RuntimeError: asyncio.run() cannot be called from a running event loop
```

### 8. Frozen Dataclass with Mutable Default
```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Config:
    names: list[str] = field(default_factory=list)

c = Config()
c.names.append("alice")
# dataclasses.FrozenInstanceError: cannot assign to field "names"
```
**Fix:**
```python
@dataclass(frozen=True)
class Config:
    names: ClassVar[list[str]] = field(default_factory=list, init=False)
```

### 9. __post_init__ Wrong Number of Arguments
```python
@dataclass
class Person:
    name: str
    def __post_init__(self, age: int):
        self.age = age

Person("Bob")
# TypeError: Person.__post_init__() takes 2 positional arguments but 1 was given
```

### 10. Forgetting slots=True (Python 3.10+)
```python
@dataclass
class Point:
    x: float
    y: float

p = Point(1, 2)
p.__dict__          # exists → huge memory waste
p.z = 100           # works → dangerous
```
**Fix:**
```python
@dataclass(slots=True)
class Point:
    x: float
    y: float
```

### 11. Missing Return Type (mypy)
```python
def add(a: int, b: int):
    return a + b
# mypy: Missing return type annotation for "add"
```

### 12. Type Mismatch
```python
def greet(name: str) -> str:
    return 42
# mypy: Incompatible return value type (got "int", expected "str")
```

### 13. Protocol Not Implemented
```python
from typing import Protocol

class Quackable(Protocol):
    def quack(self) -> None: ...

def make_noise(q: Quackable) -> None:
    q.quack()

class Dog:
    def bark(self) -> None: ...

make_noise(Dog())
# mypy: Argument 1 to "make_noise" has incompatible type "Dog"
```

### 14. @overload Implementation Doesn’t Match
```python
from typing import overload

@overload
def get(id: int) -> "User": ...
@overload
def get(name: str) -> "Group": ...

def get(x):
    return 42

# mypy: Overloaded function implementation does not accept all possible arguments
```

### 15. Using List Instead of list
```python
from typing import List
def process(items: List[int]) -> None: ...
process([1, "hello"])
# mypy: Argument 1 has incompatible type
```

### 16. Global vs nonlocal
```python
def outer():
    x = "outer"
    def inner():
        x = "inner"     # creates new local variable
    inner()
    print(x)            # "outer"
```
**Fix:** `nonlocal x`

### 17. Class Variable Shared Across Instances
```python
class Dog:
    tricks = []             # shared!
    def add_trick(self, t):
        self.tricks.append(t)

a = Dog(); b = Dog()
a.add_trick("roll")
print(b.tricks)             # ['roll']
```
**Fix:**
```python
def __init__(self):
    self.tricks = []
```

### 18. Diamond Inheritance MRO Surprise
```python
class A: def m(self): print("A")
class B(A): def m(self): print("B"); super().m()
class C(A): def m(self): print("C"); super().m()
class D(B, C): pass

D().m()                     # B → C → A
```

### 19. __slots__ + Inheritance Without Slots
```python
class A:
    __slots__ = ("x",)

class B(A):
    pass

b = B()
b.y = 1                     # AttributeError: 'B' object has no attribute 'y'
```
**Fix:**
```python
class B(A):
    __slots__ = ("y",)
```

### 20. Metaclass Conflict
```python
class Meta1(type): pass
class Meta2(type): pass

class Bad(metaclass=Meta1, metaclass=Meta2): pass
# TypeError: metaclass conflict
```

### 21. Property Setter Without Getter First
```python
class C:
    @property
    def x(self): raise AttributeError
    @x.setter
    def x(self, v): self._x = v

c = C()
c.x = 10                    # AttributeError: can't set attribute
```

### 22. __del__ Accessing Deleted Attributes
```python
class Evil:
    def __del__(self):
        print(self.name)    # AttributeError during garbage collection

e = Evil()
e.name = "doomed"
del e
```

### 23. Circular Imports
```python
# a.py
from b import y
x = y + 1

# b.py
from a import x
y = 42
```
**Result:** `x` is `None` or partial module → crashes

### 24. Monkey-Patching Breaks isinstance
```python
class A: pass
a = A()
A.__class__ = list
isinstance(a, list)         # True → total chaos
```

### 25. __new__ Returns Wrong Type
```python
class Weird:
    def __new__(cls):
        return "not an instance"

w = Weird()
w.upper()                   # crashes everywhere
```

### 26. await in Normal Generator
```python
def gen():
    yield 1
    await asyncio.sleep(1)  # SyntaxError: 'await' outside async function
```

### 27. create_task() Never Awaited
```python
async def boom():
    raise ValueError("lost")

async def main():
    asyncio.create_task(boom())
    await asyncio.sleep(0.1)   # Task fails silently or warning

asyncio.run(main())
# Task exception was never retrieved
```

### 28. Blocking Call in Async Function
```python
async def bad():
    time.sleep(10)             # blocks entire event loop
```

### 29. async with Not Handling Exceptions
```python
class BadLock:
    async def __aenter__(self): return self
    async def __aexit__(self, *args): raise ValueError("never released")

async def main():
    async with BadLock():
        pass
# lock forever held
```

### 30. functools.lru_cache on Method with Unhashable Self
```python
class C:
    @lru_cache
    def compute(self, x): ...

c1 = C(); c2 = C()
c1.compute({})              # TypeError: unhashable type: 'dict'
```

### 31. cached_property + Mutable Object
```python
class Bad:
    @cached_property
    def items(self):
        return []

b = Bad()
b.items.append(1)
b.items.append(2)           # silently modifies cached value
```

### 32. weakref.finalize During Interpreter Shutdown
```python
import weakref, atexit

def cleanup():
    print("cleaning")

weakref.finalize(object(), cleanup)  # may never run at exit
```

### 33. sys.setrecursionlimit Too High
```python
import sys
sys.setrecursionlimit(1_000_000)
def recurse(): recurse()
recurse()                   # Segmentation fault
```

### 34. from __future__ import annotations Forgotten
```python
# Python <3.11 without future import
class Node:
    def add_child(self, child: Node): ...
# NameError: name 'Node' is not defined
```
**Fix:** Add `from __future__ import annotations` at top

### 35. @singledispatch on Instance Method
```python
class Bad:
    @singledispatchmethod
    def process(self, arg):
        ...

b = Bad()
b.process(42)               # AttributeError
```

### 36. typing.Never Used Wrongly
```python
from typing import Never

def die() -> Never:
    return "oops"           # mypy: Incompatible return value type
```

### 37. assert_type at Runtime
```python
from typing import assert_type
x: int = "hello"
assert_type(x, int)         # AssertionError (only if --strict)
```

### 38. List Resize via ctypes → Instant Segfault
```python
import ctypes
lst = [1, 2, 3]
ctypes.memset(id(lst) + 24, 0xFF, 8)  # corrupts ob_size → segfault on next access
```

### 39. Using super() in Class with No Parent
```python
class A:
    def method(self):
        super().method()    # AttributeError: 'super' object has no attribute 'method'
```

### 40. Descriptor __get__ Returns Wrong Thing
```python
class BadDesc:
    def __get__(self, obj, owner):
        return "string"

class C:
    x = BadDesc()

print(C().x.upper())        # works
print(C().x + 1)            # TypeError
```

### 41–50: The Final Boss Tier (Real Production Killers)

41. **Threading + asyncio wrong loop**
```python
import threading, asyncio
def thread():
    asyncio.run(main())     # deadlock in 3.11+
```

42. **Pickle Global __reduce__ Hijack**
```python
class Evil:
    def __reduce__(self):
        return (os.system, ("calc.exe",))
```

43. **AST Node Visitor Infinite Recursion**
```python
class BadVisitor(ast.NodeVisitor):
    def visit(self, node):
        self.visit(node)        # infinite
```

44. **Context Manager __exit__ Returns True (Swallows Exception)**
```python
class SilentKiller:
    def __exit__(self, *args):
        return True             # exception disappears
```

45. **Multiprocessing + Fork + Threads = Deadlock on macOS/Linux**

46. **Importlib.resources Path Object Changed in 3.11**
```python
from importlib.resources import path
with path(pkg, "data.txt") as p:
    pass
# In 3.11+ → returns Traversable, not Path
```

47. **abc.ABCMeta + Generic = TypeError if order wrong**
```python
class Bad(Generic[T], abc.ABC): pass
# TypeError: metaclass conflict
```

48. **NamedTuple with Default + Required Field After**
```python
class Bad(NamedTuple):
    x: int = 0
    y: int                  # SyntaxError
```

49. **Structural Pattern Matching Exhaustiveness (3.10+)**
```python
match x:
    case int(): pass
    case str(): pass
# warning: match statement is not exhaustive if x can be float
```

50. **The Ultimate Evil – Direct Object Header Corruption**
```python
import ctypes
obj = object()
ctypes.c_long.from_address(id(obj) + 16).value = 0xDEAD  # instant segfault
```

You now possess the 50 most dangerous, most common, and most educational Python mistakes ever compiled.

Do them all.  
Break Python on purpose.  
Then fix it.

You are now in the top 0.1% of Python wizards.

Use this power wisely.  
Or very, very dangerously.
