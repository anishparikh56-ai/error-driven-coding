# Go – The Full Top 50 “Learn by Compiling” Pain-to-Master List  
From Junior to Staff+ Engineer – Every Mistake You Will Make in Real Go Code  
Do all 50. Type them yourself. Let `go build`, `go vet`, `golint`, `staticcheck`, and the runtime punish you. Fix them. Never forget.

### 1. Unexported Field/Method – The Capitalization Police
```go
type User struct {
    name string
}
func (u User) getName() string { return u.name }

func main() {
    u := User{name: "Alice"}
    fmt.Println(u.name)      // compile error
    fmt.Println(u.getName()) // compile error
}
```
**Error:**
```
u.name undefined (unexported field)
u.getName undefined
```
**Fix:** `Name string`, `GetName() string`

### 2. Value Receiver When You Need Mutation
```go
type Counter struct{ n int }

func (c Counter) Inc() { c.n++ } // value copy!

func main() {
    c := Counter{10}
    c.Inc()
    fmt.Println(c.n) // 10
}
```
**Fix:** `func (c *Counter) Inc() { c.n++ }`

### 3. Forgetting Pointer Receiver Consistency
```go
type Counter struct{ n int }

func (c *Counter) Inc() { c.n++ }
func (c Counter) String() string { return fmt.Sprint(c.n) } // c is nil!

func main() {
    var c *Counter
    fmt.Println(c) // panic: nil receiver in String
}
```
**Fix:** Both methods pointer or both value.

### 4. Loop Variable Capture in Closure (Go < 1.22)
```go
var fs []func()
for i := 0; i < 3; i++ {
    fs = append(fs, func() { fmt.Println(i) })
}
for _, f := range fs { f() } // 3 3 3
```
**Fix:**
```go
for i := 0; i < 3; i++ {
    i := i
    fs = append(fs, func() { fmt.Println(i) })
}
```

### 5. Shadowing Variables – go vet Catches This
```go
func main() {
    err := setup()
    if err != nil { return err }
    err := run() // shadows previous err
    if err != nil { fmt.Println(err) } // wrong err!
}
```
**go vet:**
```
declaration of "err" shadows declaration at line X
```

### 6. Unused Variables/Imports – Hard Compile Error
```go
import "fmt"
var x int // unused

func main() {}
```
**Error:**
```
x declared and not used
```

### 7. Wrong Interface Method Name (Case Sensitive!)
```go
type Stringer interface { String() string }

type User struct{}
func (u User) string() string { return "user" } // lowercase!

var _ Stringer = User{} // no error, but fmt.Println(u) shows struct
```

### 8. nil Receiver Panic
```go
type Logger struct{}
func (l *Logger) Info(s string) { fmt.Println(s) }

func main() {
    var l *Logger
    l.Info("hello") // panic: nil pointer dereference
}
```

### 9. Slice Append Returns New Slice (Forgot Assignment)
```go
s := []int{1, 2}
append(s, 3)     // discarded!
fmt.Println(s)   // [1 2]
```
**Fix:** `s = append(s, 3)`

### 10. Map Read/Write Race Without Mutex
```go
m := map[string]int{}
go func() { m["x"] = 1 }()
go func() { _ = m["x"] }()
```
**go run -race:** data race

### 11. Forgetting to Close Channel → Deadlock
```go
ch := make(chan int)
go func() { ch <- 42 }()
<-ch
close(ch)
close(ch) // panic: close of closed channel
```

### 12. Range Over nil Slice/Map
```go
var s []int
for i, v := range s { fmt.Println(i, v) } // OK
var m map[string]int
for k, v := range m { fmt.Println(k, v) } // panic
```

### 13. JSON Marshal Unexported Fields
```go
type User struct {
    name string `json:"name"`
}
u := User{name: "Bob"}
json.Marshal(u) // {"name":""}
```

### 14. Forgetting json:",omitempty" on Pointer
```go
type Config struct {
    Timeout *int `json:"timeout,omitempty"`
}
json.Marshal(&Config{}) // {"timeout":null} ← not omitted!
```

### 15. Using http.HandlerFunc Wrong
```go
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "hi")
})
http.ListenAndServe(":8080", nil) // works
http.ListenAndServe(":8080", http.DefaultServeMux) // nil deref panic
```

### 16. Context Without Timeout/Cancel
```go
ctx := context.Background()
go longRunning(ctx) // never cancellable → goroutine leak
```

### 17. Defer in Loop Captures Loop Variable
```go
for i := 0; i < 3; i++ {
    defer fmt.Println(i) // 2 1 0
}
```

### 18. Using time.After in Loop → Memory Leak
```go
for {
    select {
    case <-time.After(5 * time.Second): // new timer every loop!
        fmt.Println("tick")
    }
}
```
**Fix:** use `time.NewTicker`

### 19. sync.Once Do Argument Evaluated Every Time
```go
once.Do(createExpensiveObject()) // created every call!
```

### 20. sync.Map Range Callback Returns False → Stops Early
```go
var m sync.Map
m.Store("a", 1)
m.Range(func(k, v interface{}) bool {
    fmt.Println(k)
    return false // only prints first!
})
```

### 21. Generics: Forgetting Constraint
```go
func Max[T any](a, b T) T {
    if a > b { return a } // invalid operation: a > b
}
```
**Fix:** `T constraints.Ordered`

### 22. Generics: Type Set Too Broad
```go
func Print[T any](t T) { fmt.Println(t) }
Print([]int{1,2,3}) // works, but you wanted only strings
```
**Fix:** custom constraint

### 23. embed.FS Wrong Path
```go
//go:embed static/*.txt
var files embed.FS
f, _ := files.Open("static/../secret.txt") // escapes!
```

### 24. sql.NullString Used Wrong
```go
var name sql.NullString
row.Scan(&name) // need &name.String and &name.Valid
```

### 25. reflect.DeepEqual on Functions/Maps
```go
m1 := map[string]int{"a": 1}
m2 := map[string]int{"a": 1}
reflect.DeepEqual(m1, m2) // false!
```

### 26. json.RawMessage Marshal Twice
```go
var r json.RawMessage = []byte(`"hello"`)
json.Marshal(r) // []byte(`"\"hello\""` ) ← double escaped
```

### 27. Using testing.T.Fatal in Goroutine
```go
func TestX(t *testing.T) {
    go func() { t.Fatal("fail") }() // silently ignored
}
```

### 28. init() Order Across Packages – Undefined!
```go
// a.go
var X = B.Y

// b.go
var Y = 42
```

### 29. cgo + Fork = Deadlock
```go
// +build linux
exec.Command("ls").Run() // after cgo call → deadlock
```

### 30. net/http Client Reuse Without Timeout
```go
client := &http.Client{}
resp, _ := client.Get("http://slow") // hangs forever
```

### 31–50: Staff+ Level Go Crimes

31. `sync.Pool` holding pointers to short-lived objects → memory leak  
32. `runtime.SetMutexProfileFraction(1)` in production → 1000x slowdown  
33. `golang.org/x/sync/errgroup` without `WithContext` → no cancel  
34. `io.Copy(os.Stdout, resp.Body)` without `resp.Body.Close()` → connection leak  
35. `defer wg.Done()` before checking error → lost goroutines  
36. `reflect` on unexported struct fields → zero values  
37. `atomic` on non-64-bit-aligned fields on 32-bit systems → crash  
38. `go:linkname` hacking internal functions → breaks on every Go release  
39. `unsafe.Pointer` arithmetic without `unsafe.Add` (Go 1.17+) → deprecated  
40. `runtime/trace` region inside tight loop → GB of trace data  
41. `net.Dial` without deadline → permanent hang  
42. `encoding/gob` with unregistered types → silent zero values  
43. `text/template` with untrusted input → code execution  
44. `os/exec.Command` with user input → command injection  
45. `http.NewRequest` with `Host` header → server-side request smuggling  
46. `math/rand` global without seed → same sequence every run  
47. `time.Parse` without location → silently uses UTC  
48. `database/sql` Rows not closed → connection exhaustion  
49. `io.ReadAll` on unbounded request body → OOM  
50. The Ultimate Evil – Stack Overflow via Recursion + Defer
```go
func infinite() {
    defer infinite()
    infinite()
}
func main() { infinite() } // segfault, not caught
```

Do all 50.  
Break Go on purpose.  
Let the compiler, vet, race detector, and runtime destroy you.

Then fix them.

You are now in the top 0.1% of Go engineers on Earth.

Use this power responsibly.  
Or to write unmaintainable genius code that only you can debug.


# Go – The Final 20 Staff+ Level Crimes (31–50)  
**Fully Expanded with Real Code, Exact Errors/Panics/Leaks, and Fixes**  
These are the bugs that silently kill production systems at 3 a.m. and make senior engineers cry.

Do them all. Watch your program die. Then fix them — and never make them again.

### 31. sync.Pool Holding Pointers to Short-Lived Objects → Massive Memory Leak
```go
var pool = sync.Pool{
    New: func() any { return new(bytes.Buffer) },
}

func leaky() {
    b := pool.Get().(*bytes.Buffer)
    b.WriteString("temporary data")
    // Forgot to reset or return!
    // pool.Put(b) ← missing
}

func main() {
    for i := 0; i < 1_000_000; i++ {
        go leaky()
    }
    time.Sleep(10 * time.Second)
    fmt.Println("Memory usage: hundreds of MB and climbing")
}
```
**Result:** Old buffers never returned → GC keeps them alive → memory explodes  
**Fix:**
```go
b.Reset()
pool.Put(b)
```

### 32. runtime.SetMutexProfileFraction(1) in Production → 1000× Slowdown
```go
func main() {
    runtime.SetMutexProfileFraction(1) // logs EVERY mutex contention
    var mu sync.Mutex
    go func() {
        for {
            mu.Lock(); mu.Unlock()
        }
    }()
    time.Sleep(10 * time.Second)
    // pprof shows millions of fake "contended" locks
}
```
**Result:** CPU and log volume explode  
**Fix:** Only enable in debug: `if os.Getenv("DEBUG") != "" { ... }`

### 33. errgroup Without Context → Goroutine Leak on Cancel
```go
func leaky() {
    g := errgroup.Group{}
    for i := 0; i < 10; i++ {
        g.Go(func() error {
            time.Sleep(10 * time.Second) // never respects cancel!
            return nil
        })
    }
    g.Wait() // hangs forever if parent context canceled
}
```
**Fix:**
```go
g, ctx := errgroup.WithContext(parentCtx)
g.Go(func() error {
    select {
    case <-ctx.Done(): return ctx.Err()
    case <-time.After(10 * time.Second): return nil
    }
})
```

### 34. io.Copy Without Closing resp.Body → Connection Exhaustion
```go
func handler(w http.ResponseWriter, r *http.Request) {
    resp, _ := http.Get("https://example.com/large-file")
    io.Copy(w, resp.Body)
    // resp.Body.Close() ← forgotten!
}
```
**Result:** After ~100 requests → "too many open files" or connection pool dead  
**Fix:**
```go
defer resp.Body.Close()
```

### 35. defer wg.Done() Before Error Check → Lost Goroutines
```go
func badWorker(wg *sync.WaitGroup) {
    defer wg.Done()
    if err := doWork(); err != nil {
        return // wg.Done() already called → wg.Wait() hangs forever!
    }
}
```
**Fix:**
```go
defer func() {
    if err != nil { wg.Add(1) } // re-add on error
    wg.Done()
}()
```

### 36. reflect on Unexported Fields → Silent Zero Values
```go
type Config struct {
    timeout int // unexported
}

c := Config{timeout: 30}
v := reflect.ValueOf(c)
f := v.FieldByName("timeout")
fmt.Println(f.IsValid())  // false
fmt.Println(f.Int())      // panic if you force it
```
**Result:** Configuration silently ignored

### 37. atomic on Non-64-bit-aligned Struct Fields (32-bit Systems)
```go
type Bad struct {
    a uint32
    b uint64 // not 8-byte aligned on 32-bit
    c uint32
}
var x Bad
atomic.AddUint64(&x.b, 1) // crash on arm32/go1.18-
```

### 38. go:linkname Hacks Break on Every Release
```go
//go:linkname fastpath runtime.fastpath
func fastpath()

func main() { fastpath() } // works today, crashes tomorrow
```
**Result:** Go 1.22 → undefined symbol → panic

### 39. unsafe.Pointer Arithmetic Without unsafe.Add (Go 1.17+ Warning → Future Error)
```go
p := unsafe.Pointer(uintptr(0x1000))
p = unsafe.Pointer(uintptr(p) + 8) // deprecated style
```
**Go 1.23+:** compiler error  
**Fix:** `p = unsafe.Add(p, 8)`

### 40. runtime/trace.Region in Hot Loop → Gigabytes of Trace Data
```go
func hot() {
    for i := 0; i < 1_000_000; i++ {
        trace.StartRegion(context.Background(), "hot")
        trace.EndRegion()
    }
}
// trace file: >10 GB in seconds
```

### 41. net.Dial Without Deadline → Permanent Hang
```go
conn, _ := net.Dial("tcp", "1.2.3.4:9999") // dead server
conn.SetDeadline(time.Time{}) // never!
buf := make([]byte, 100)
conn.Read(buf) // hangs forever
```

### 42. encoding/gob With Unregistered Types → Silent Corruption
```go
type Message struct{ Data []byte }
gob.Register(Message{}) // forgot!

// sender
enc := gob.NewEncoder(conn)
enc.Encode([]byte("hello"))

// receiver
var msg Message
dec.Decode(&msg) // msg.Data == nil, no error!
```

### 43. text/template + Untrusted Input → Code Execution
```go
tmpl := template.New("").Funcs(template.FuncMap{
    "exec": os.Exec,
})
template.Must(tmpl.Parse(`{{ exec "rm -rf /" }}`))
tmpl.Execute(os.Stdout, nil) // goodbye server
```

### 44. os/exec.Command With User Input → Shell Injection
```go
name := "Robert'; rm -rf / #"
cmd := exec.Command("sh", "-c", "echo hello "+name)
cmd.Run() // executes rm -rf /
```

### 45. http.NewRequest With Host Header → Request Smuggling
```go
req, _ := http.NewRequest("GET", "http://victim/", nil)
req.Host = "evil.com"
client.Do(req) // backend sees evil.com as host
```

### 46. math/rand Global Without Seed → Predictable Sequence
```go
for i := 0; i < 10; i++ {
    fmt.Println(rand.Int()) // same numbers every run!
}
```
**Fix:** `rand.Seed(time.Now().UnixNano())` or use `rand.New(...)`

### 47. time.Parse Without Location → Silent UTC
```go
t, _ := time.Parse("2006-01-02", "2025-04-01")
fmt.Println(t.Location()) // UTC, not local!
```
**Fix:** `time.ParseInLocation(...)` or use `time.Date`

### 48. database/sql Rows Not Closed → Connection Exhaustion
```go
rows, _ := db.Query("SELECT ...")
for rows.Next() { /* scan */ }
// rows.Close() ← forgotten!
```
**Result:** After 100 queries → "too many connections"

### 49. io.ReadAll on Untrusted Request Body → OOM
```go
func handler(w http.ResponseWriter, r *http.Request) {
    body, _ := io.ReadAll(r.Body) // attacker sends 100GB
    // panic: out of memory
}
```
**Fix:** Use `io.LimitReader(r.Body, 10<<20)`

### 50. Infinite Recursion + Defer = Uncatchable Stack Overflow
```go
func evil() {
    defer evil()    // defer stack grows
    evil()          // call stack grows
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered:", r) // never prints!
        }
    }()
    evil()
}
// Result: segmentation fault — Go cannot recover
```

You have now seen the **20 most dangerous production-killing Go bugs** that only senior engineers encounter.

You broke them.  
You watched Go die in 20 different ways.  
You fixed them.

You are now officially **dangerous** with Go.

Use this knowledge to write unbreakable systems…  
or to become the person called at 4 a.m. when no one else can figure out why the cluster is on fire.

Welcome to the 0.01%.
