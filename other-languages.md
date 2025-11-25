# Learn by Compiling: The Ultimate Pain-to-Master Series  
Now covering **Go, Python, Java, and C#** — from basics to bleeding-edge advanced concepts.  
Every single example is structured exactly like your favorite C++ smart-pointer section:  
**broken code → real compiler/runtime error → fix → lesson burned into your soul.**

### New Section: Go – The Compiler That Teaches Through Strictness and Silence

#### 1. Exported vs Unexported – The Capitalization Police
**Broken Code:**
```go
type User struct {
    name string
    age  int
}

func (u User) getName() string {
    return u.name
}

func main() {
    u := User{name: "Alice", age: 30}
    fmt.Println(u.name)      // ← boom
    fmt.Println(u.getName()) // ← boom
}
```
**Compiler Error:**
```
u.name (unexport35ed field) undefined
u.getName undefined (type User has no method getName)
```
**Fixed:**
```go
type User struct {
    Name string
    Age  int
}
func (u User) GetName() string { return u.Name }
```
**Lesson:** Go exports ONLY uppercase identifiers. No exceptions.

#### 2. Pointer vs Value Receiver – The Most Common Silent Bug
**Broken Code:**
```go
type Counter struct{ n int }

func (c Counter) Inc() { c.n++ } // value receiver!

func main() {
    c := Counter{10}
    c.Inc()
    fmt.Println(c.n) // still 10!
}
```
**No compile error** — runs perfectly wrong.  
**Fix:**
```go
func (c *Counter) Inc() { c.n++ }
```
**Lesson:** If a method mutates the receiver → pointer receiver. Always.

#### 3. Embedding & Method Promotion – The “Inheritance” Go Never Admits To
**Broken Code:**
```go
type Person struct{ Name string }
func (p Person) Greet() { fmt.Println("Hi", p.Name) }

type Admin struct {
    Person
    Level int
}

func main() {
    a := Admin{Person{"Bob"}, 9000}
    a.Greeting() // ← no such method
}
```
**Compiler Error:**
```
a.Greeting undefined (type Admin has no field or method Greeting)
```
**Fixed:**
```go
a.Greet() // promoted automatically!
```
**Lesson:** Embedding = composition + automatic method promotion.

#### 4. Interface Satisfaction – Silent and Deadly
**Broken Code:**
```go
type Stringer interface {
    String() string
}

type User struct{ Name string }

func (u User) string() string { return u.Name } // lowercase!

func main() {
    var _ Stringer = User{"Alice"} // ← should fail loudly
}
```
**No error at all** — but `fmt.Println(user)` prints `{Alice}` instead of name.  
**Fix:**
```go
func (u User) String() string { return u.Name } // exact name + exported
```

#### 5. The Infamous Loop Variable Capture (Pre-Go 1.22)
**Broken Code:**
```go
var funcs []func()
for i := 0; i < 3; i++ {
    funcs = append(funcs, func() { fmt.Println(i) })
}
for _, f := range funcs { f() } // 3 3 3
```
**Fix (old style):**
```go
for i := 0; i < 3; i++ {
    i := i // new variable per iteration
    funcs = append(funcs, func() { fmt.Println(i) })
}
```
**Go 1.22+ fixes this automatically.**

#### 6. Empty Interface + Type Assertion Panic
**Broken Code:**
```go
func print(x interface{}) {
    s := x.(string) // no check!
    fmt.Println(s)
}
print(42) // runtime panic
```
**Fix:**
```go
if s, ok := x.(string); ok {
    fmt.Println(s)
}
```

#### 7. Generics – Forgetting Constraints
**Broken Code (Go 1.18):**
```go
func Max[T any](a, b T) T {
    if a > b { return a } // operator > not defined
}
```
**Compiler Error:**
```
invalid operation: operator > not defined on a (variable of type T constrained to any)
```
**Fix:**
```go
func Max[T constraints.Ordered](a, b T) T {
    if a > b { return a }
    return b
}
```

### New Section: Python – Where the Interpreter Punishes You at Runtime (and mypy screams)

#### 1. Mutable Default Arguments – The Immortal Bug
**Broken Code:**
```python
def append_to(item, target=[]):
    target.append(item)
    return target

a = append_to(1)
b = append_to(2)
print(b)  # [1, 2] → shared list!
```
**Fix:**
```python
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```

#### 2. Late Binding Closures – The Classic Lambda in Loop
**Broken Code:**
```python
funcs = [lambda: i for i in range(3)]
for f in funcs: print(f())  # 2 2 2
```
**Fix:**
```python
funcs = [lambda i=i: i for i in range(3)]
```

#### 3. Dataclass Frozen + Mutable Field = Explosion
**Broken Code:**
```python
@dataclass(frozen=True)
class Config:
    allowed: list[str] = field(default_factory=list)

c = Config()
c.allowed.append("admin")  # freezes, but list is mutable!
```
**Runtime Error:**
```
FrozenInstanceError: cannot assign to field 'allowed'
```
**Fix:**
```python
allowed: ClassVar[list[str]] = field(default_factory=list, init=False)
```

#### 4. Protocol Violation – mypy Catches What Runtime Won’t
**Broken Code:**
```python
from typing import Protocol

class Duck(Protocol):
    def quack(self) -> str: ...

def make_quack(d: Duck): ...

class Dog:
    def bark(self): ...

make_quack(Dog())  # mypy: Argument 1 has incompatible type
```

#### 5. @overload Gone Wrong
**Broken Code:**
```python
from typing import overload

@overload
def get(id: int) -> User: ...
@overload
def get(name: str) -> User: ...

def get(x):  # missing type
    ...
```
**mypy error:**
```
Overloaded function implementation does not accept all possible arguments
```

### New Section: Java – Where javac Is Your Strict Professor

#### 1. Forgetting to Override (No @Override)
**Broken Code:**
```java
class Animal { void speak() { } }
class Dog extends Animal {
    void speak() { System.out.println("Woof"); } // typo!
}
Animal a = new Dog();
a.speak(); // silence → no override!
```
**Fix:**
```java
@Override
void speak() { ... } // javac error if signature wrong
```

#### 2. Generics Type Erasure Disaster
**Broken Code:**
```java
List<String> strings = new ArrayList<>();
List raw = strings;
raw.add(42); // compiles!
strings.get(0).toUpperCase(); // ClassCastException at runtime
```
**Fix (never do this):**
```java
List<String> safe = new ArrayList<>();
// don't mix raw types
```

#### 3. Sealed Classes – Forgetting permits
**Broken Code:**
```java
public sealed class Expr permits Constant { }

public final class Constant extends Expr { }

public class Variable extends Expr { } // in another file
```
**Error:**
```
class is not allowed to extend sealed class Expr
```

#### 4. Record with Mutable Component
**Broken Code:**
```java
public record Range(List<Integer> values) { }

var r = new Range(new ArrayList<>());
r.values().add(999); // mutation escapes!
```
**Fix:**
```java
public record Range(List<Integer> values) {
    public Range { values = List.copyOf(values); }
}
```

### New Section: C# – Roslyn Is Watching Your Every Mistake

#### 1. async All the Way Down – Fire and Forget Hell
**Broken Code:**
```csharp
public async Task<long> ComputeAsync() {
    await Task.Delay(100);
    return 42;
}

public void Bad() {
    ComputeAsync(); // exceptions swallowed!
}
```
**Runtime:** UnobservedTaskException → app crash

#### 2. record vs record class vs record struct
**Broken Code:**
```csharp
public record Person(string Name); // reference type
public record struct Point(int X, int Y); // value type

Person p1 = new("A");
Person p2 = new("A");
Console.WriteLine(p1 == p2); // true (with synthesis)

Point pt1 = new(1,2);
Point pt2 = new(1,2);
Console.WriteLine(pt1 == pt2); // true
```

#### 3. Pattern Matching Exhaustiveness
**Broken Code (C# 10+):**
```csharp
sealed class Shape { }
class Circle : Shape { public int R { get; } = 1; }
class Rectangle : Shape { }

string Describe(Shape s) => s switch {
    Circle c => "circle",
    // missing Rectangle!
};
```
**Compiler Error:**
```
CS8505: switch expression does not handle all possible input values
```

#### 4. Source Generators – Wrong Attribute
**Broken Code:**
```csharp
[GenerateSerializer] // doesn't exist
public partial class User { public string Name { get; set; } }
```
**Build error:** attribute not found → zero code generated

#### 5. Primary Constructors (C# 12)
**Broken Code:**
```csharp
class Service(ILogger logger); // syntax error in older C#
```
**C# 12+ only:**
```csharp
class Service(ILogger logger) {
    public void Log(string msg) => logger.LogInformation(msg);
}
```

Do every single one of these 50+ broken examples.  
The compiler and runtime will become your greatest teachers.  
You’ll never make these mistakes in production again.  

Happy breaking (and mastering)!
