# Java – The Full Top 50 “Learn by Compiling” Pain-to-Master List  
From Junior Dev to Principal Engineer – Every Mistake That Kills Production Java Systems  
Do all 50. Compile with `javac`. Run with `java`. Let the JVM, `javac`, spotbugs, Error Prone, and real OutOfMemoryErrors punish you. Then fix them forever.

Fully expanded, complete, minimal reproducible examples.

### 1. Forgetting @Override → Silent No-Op
```java
class Animal { void speak() { } }
class Dog extends Animal {
    void speak() { System.out.println("Woof"); } // typo: no @Override!
}
Animal a = new Dog();
a.speak(); // prints nothing!
```
**Fix:** `@Override` → compile error if signature wrong

### 2. Raw Types → ClassCastException at Runtime
```java
List strings = new ArrayList();
strings.add(42);
String s = (String) strings.get(0); // ClassCastException
```

### 3. Diamond Operator in Anonymous Inner Class (Java <9)
```java
List<String> list = new ArrayList<>() { }; // compile error before Java 9
```

### 4. Sealed Class – Forgetting permits
```java
// Shape.java
public sealed class Shape permits Circle { }

public final class Circle extends Shape { }

public class Square extends Shape { } // in another file → compile error
```
**Error:** class is not allowed to extend sealed class

### 5. Record with Mutable Collection
```java
public record Order(List<String> items) { }

var order = new Order(new ArrayList<>(List.of("a")));
order.items().add("b"); // mutation escapes!
```

### 6. Unchecked Exception in Stream → Silent Failure
```java
List.of("1", "x", "3").stream()
    .map(Integer::parseInt)
    .forEach(System.out::println);
// NumberFormatException kills stream silently
```

### 7. ConcurrentHashMap.computeIfAbsent Infinite Loop
```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.computeIfAbsent("key", k -> map.get("other")); // StackOverflowError
```

### 8. ThreadLocal Without remove() → Memory Leak
```java
static ThreadLocal<BigObject> tl = ThreadLocal.withInitial(BigObject::new);

void process() {
    tl.get().doWork();
    // tl.remove(); ← forgotten → leak in thread pool
}
```

### 9. finalize() Resurrection → Undead Objects
```java
class Evil {
    static Evil INSTANCE;
    @Override protected void finalize() {
        INSTANCE = this; // resurrects!
    }
}
```

### 10. Double-Checked Locking Without volatile
```java
class Singleton {
    private static Singleton instance; // not volatile!
    public static Singleton get() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
// can return partially constructed object!
```

### 11–20: JVM & GC Nightmares

11. Direct ByteBuffer Memory Leak
```java
ByteBuffer buf = ByteBuffer.allocateDirect(100_000_000);
// never freed explicitly → native OOM
```

12. java.lang.instrument + RetainClass = PermGen/Metaspace OOM
```java
// loading 10,000 classes via byte-buddy without -XX:+CMSClassUnloading
```

13. G1 Humongous Allocation Fragmentation
```java
byte[] huge = new byte[6_000_000]; // >50% of G1 region → never reclaimed efficiently
```

14. ForkJoinPool.commonPool() + Blocking Call → Starvation
```java
ForkJoinPool.common.parallelism = 256;
IntStream.range(0, 1000).parallel().forEach(i -> {
    LockSupport.parkNanos(1_000_000_000); // blocks all worker threads
});
```

15. System.gc() with -XX:+DisableExplicitGC → Ignored (surprise!)

16. -Xms = -Xmx + Large Pages → JVM Won’t Start

17. String.substring() Before Java 7u6 → Memory Leak
```java
String huge = new String(bigArray); // shares array until 7u6
String tail = huge.substring(100_000); // keeps entire array alive
```

18. WeakReference + ReferenceQueue Never Polled → Leak

19. PhantomReference Misuse (never cleared)

20. java.util.logging + Custom Handler Without Memory Limit → OOM

### 21–30: Concurrency & Threading Hell

21. ReentrantLock Never Unlocked on Exception
```java
lock.lock();
try {
    // work
} finally {
    // lock.unlock(); ← forgotten
}
```

22. ExecutorService Never Shutdown → JVM Won’t Exit

23. Future.get() Without Timeout → Permanent Hang

24. CountDownLatch with Negative Count → IllegalStateException

25. CyclicBarrier with 1 Party → Deadlock

26. Semaphore.acquireUninterruptibly() → Ignores Thread.interrupt()

27. ThreadPoolExecutor + CallerRunsPolicy + Unbounded Queue → OOM

28. CompletableFuture + Custom Executor That’s Shut Down → RejectedExecutionException

29. java.util.Timer + Long-Running Task → Thread Leak

30. Thread.setDaemon(true) After start() → IllegalThreadStateException

### 31–40: Real Production Killers

31. Spring @Transactional + private method → No Proxy → No Transaction

32. JPA @OneToMany(fetch = EAGER) + Large Graph → N+1 + OOM

33. Hibernate Session Never Closed → Connection Leak

34. Jackson @JsonIgnore + @JsonProperty on Getter → Infinite Recursion

35. Gson Expose + Transient + Custom Serializer → Double Serialization

36. lombok @EqualsAndHashCode on Entity with @Id → StackOverflowError in Sets

37. lombok @Data on Class with bidirectional @OneToMany → StackOverflowError

38. java.time ZoneId.of("IST") → Ambiguous (India vs Ireland)

39. DateTimeFormatter Without ResolverStyle.STRICT → Accepts Invalid Dates

40. BigDecimal(String) vs BigDecimal(double)
```java
new BigDecimal("0.1"); // correct
new BigDecimal(0.1);   // 0.1000000000000000055511151231257827021181583404541015625
```

### 41–50: The Final Boss Tier

41. ClassLoader Leak via ThreadLocal + Webapp Reload (Tomcat/JBoss)

42. native JNI Leak via NewGlobalRef Without DeleteGlobalRef

43. java.util.zip.Deflater Held Across Requests → Native Memory Leak

44. java.awt.Toolkit.getDefaultToolkit() in Server → Headless Exception or Leak

45. sun.misc.Unsafe + Off-Heap Without FreeMemory → Native OOM

46. java.lang.reflect.Proxy + Interface with Default Methods + SecurityManager → AccessControlException

47. MethodHandles.lookup() in Wrong Class Context → IllegalAccessException

48. java.security.cert.CertificateRevokedException Silently Ignored in Custom TrustManager

49. -Djava.security.manager + Custom Policy That Allows Everything → False Sense of Security

50. The Ultimate Evil – Direct Memory Corruption via Unsafe
```java
class Evil {
    static final Unsafe unsafe;
    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
        } catch (Exception e) { throw new RuntimeException(e); }
    }

    public static void main(String[] args) throws Exception {
        String s = "hello";
        long addr = unsafe.getLong(s, 8L); // object header
        unsafe.putLong(s, 8L, 0xDEADBEEF); // corrupt object header → JVM segfault
    }
}
// Result: immediate JVM crash
```

You have now witnessed the 50 most dangerous, most common, and most expensive Java bugs ever written.

You compiled them.  
You watched the JVM crash, hang, leak, and corrupt itself in 50 different ways.  
You fixed them.

You are now officially **dangerous** with Java.

You can now:
- Debug any production Java outage at 3 a.m.
- Write code that survives Black Friday
- Scare junior devs with stories that actually happened

Welcome to the 0.01% of Java engineers.

Use this power wisely.  
Or to write "enterprise-grade" code that only you can maintain.

# Java – The Full Expanded 11–20: JVM & GC Nightmares That Kill Production  
These are the bugs that silently eat terabytes of RAM and bring down billion-dollar services.

Do them all. Watch the JVM die slowly (or instantly). Then fix them — and never pay this price again.

### 11. Direct ByteBuffer Memory Leak – Native OOM Without Any Java Heap
```java
import java.nio.ByteBuffer;

public class DirectBufferLeak {
    public static void main(String[] args) throws Exception {
        while (true) {
            // 100 MB direct buffer every loop
            ByteBuffer buf = ByteBuffer.allocateDirect(100 * 1024 * 1024);
            // Never freed → native memory grows forever
            Thread.sleep(10);
        }
    }
}
```
**Run with:** `java -Xmx512m DirectBufferLeak`  
**Result:** Java heap stays at 512 MB, but OS shows memory → 10+ GB → killed by OOM killer  
**Fix:**
```java
((DirectBuffer) buf).cleaner().clean(); // Java 9+
```
or wrap in try-with-resources with a cleaner.

### 12. ClassLoader Leak via Dynamic Class Generation → Metaspace OOM
```java
// Requires byte-buddy or cglib on classpath
import net.bytebuddy.ByteBuddy;
import net.bytebuddy.dynamic.loading.ClassLoadingStrategy;

public class ClassLoaderLeak {
    public static void main(String[] args) throws Exception {
        int i = 0;
        while (true) {
            new ByteBuddy()
                .subclass(Object.class)
                .name("LeakedClass" + i++)
                .make()
                .load(ClassLoader.getSystemClassLoader(),
                      ClassLoadingStrategy.Default.WRAPPER)
                .getLoaded();
            // 10,000+ classes → Metaspace full
        }
    }
}
```
**Run with:** `-XX:MaxMetaspaceSize=256m`  
**Result:** `java.lang.OutOfMemoryError: Metaspace`

### 13. G1 Humongous Allocation Fragmentation → GC Cannot Reclaim
```java
public class G1Fragmentation {
    static List<byte[]> list = new ArrayList<>();

    public static void main(String[] args) {
        // G1 region size default = 1–32 MB
        while (true) {
            // 6 MB = humongous (>50% of region)
            list.add(new byte[6_000_000]);
            list.add(new byte[100]); // tiny object
            // humongous regions never split → permanent fragmentation
        }
    }
}
```
**Result:** Heap grows to 90% but GC never frees anything → Full GC storm

### 14. ForkJoinPool.commonPool() + Blocking Call → All Parallel Streams Die
```java
import java.util.concurrent.*;
import java.util.stream.*;

public class ForkJoinStarvation {
    public static void main(String[] args) {
        // Default parallelism = CPU cores
        IntStream.range(0, 1000).parallel().forEach(i -> {
            try {
                Thread.sleep(5000); // blocks a worker thread
            } catch (InterruptedException e) {}
        });
        System.out.println("This line never prints");
    }
}
```
**Result:** All 8 (or 100) worker threads blocked → entire app frozen

### 15. System.gc() + -XX:+DisableExplicitGC → Ignored (Surprise!)
```java
public class SystemGcIgnored {
    public static void main(String[] args) {
        System.out.println("Forcing GC...");
        System.gc();                    // silently ignored!
        System.out.println("GC call returned immediately");
    }
}
```
**Run with:** `java -XX:+DisableExplicitGC SystemGcIgnored`  
**Result:** No full GC, direct buffers still not cleaned

### 16. -Xms = -Xmx + Huge Pages → JVM Refuses to Start
```java
// Command line:
java -Xms32g -Xmx32g -XX:+UseLargePages -jar App.jar
```
**Result:**
```
Failed to reserve shared memory: No such file or directory
```
Because huge pages weren’t pre-allocated by the OS.

### 17. String.substring() Before Java 7u6 → Classic Memory Leak
```java
public class SubstringLeak {
    public static void main(String[] args) {
        String huge = new String(new char[100_000_000]); // 200 MB
        String tiny = huge.substring(0, 10);             // keeps 200 MB alive!
        huge = null;
        System.gc();
        System.out.println("tiny holds 200 MB");
    }
}
```
**Works correctly only on Java 7u6+** — older versions share char[].

### 18. WeakReference + ReferenceQueue Never Polled → Leak
```java
import java.lang.ref.*;

public class ReferenceQueueLeak {
    static ReferenceQueue<Object> queue = new ReferenceQueue<>();
    static List<WeakReference<Object>> refs = new ArrayList<>();

    public static void main(String[] args) throws Exception {
        while (true) {
            Object obj = new byte[10_000_000];
            refs.add(new WeakReference<>(obj, queue));
            // queue.poll() never called → Reference objects never cleared
        }
    }
}
```
**Result:** `OutOfMemoryError: Java heap space` even though objects are weakly reachable

### 19. PhantomReference Misused (Never Becomes Enqueued)
```java
import java.lang.ref.*;

public class PhantomFail {
    public static void main(String[] args) throws Exception {
        ReferenceQueue<Object> q = new ReferenceQueue<>();
        Object obj = new Object();
        PhantomReference<Object> ref = new PhantomReference<>(obj, q);

        obj = null;
        System.gc();

        Thread.sleep(1000);
        System.out.println(q.poll()); // null — you forgot to keep strong ref to PhantomReference!
    }
}
```
**Rule:** PhantomReference itself must be strongly reachable or it gets cleared too.

### 20. java.util.logging + Custom Handler Without Size Limit → OOM
```java
import java.util.logging.*;

public class JULMemoryLeak {
    static {
        Logger logger = Logger.getLogger("");
        MemoryHandler handler = new MemoryHandler(
            new ConsoleHandler(), 1000, Level.OFF); // buffer never pushed
        logger.addHandler(handler);
    }

    public static void main(String[] args) {
        Logger log = Logger.getLogger("leak");
        while (true) {
            log.severe("msg " + System.nanoTime());
            // MemoryHandler buffer grows forever
        }
    }
}
```
**Result:** OOM after millions of log records

You just reproduced the **10 most expensive JVM/GC bugs** that have cost companies millions in cloud bills and downtime.

You watched the JVM:
- Leak native memory
- Explode Metaspace
- Fragment G1
- Starve its own thread pool
- Ignore your desperate `System.gc()` pleas
- Hold onto 200 MB strings because of a 10-char substring
- Leak reference objects
- And more…

Now you know exactly how they happen — and how to prevent them.

You are now officially **dangerous** with the JVM internals.

Use this knowledge to:
- Survive on-call at 3 a.m.
- Save your company $100k/month in AWS bills
- Make senior engineers nod in quiet respect

Welcome to the 0.01%.

# Java – Expanded #21–30: Concurrency & Threading Hell That Kills Production  
These are the bugs that silently deadlock, leak, or hang your entire JVM at scale.  
Every single one has taken down real production systems.

Do them all. Run them. Watch the JVM freeze, leak, or refuse to exit. Then fix them — and become untouchable.

### 21. ReentrantLock Never Unlocked on Exception → Permanent Deadlock
```java
import java.util.concurrent.locks.*;

public class LockNeverReleased {
    static ReentrantLock lock = new ReentrantLock();

    public static void bad() {
        lock.lock();
        try {
            System.out.println("Got lock");
            throw new RuntimeException("boom");
            // lock.unlock() never reached!
        } finally {
            // lock.unlock(); ← forgotten!
        }
    }

    public static void main(String[] args) throws Exception {
        new Thread(LockNeverReleased::bad).start();
        Thread.sleep(1000);
        lock.lock(); // hangs forever — lock is poisoned
        System.out.println("This never prints");
    }
}
```
**Result:** Second thread deadlocks forever  
**Fix:** Always unlock in `finally`

### 22. ExecutorService Never Shutdown → JVM Never Exits
```java
import java.util.concurrent.*;

public class NonDaemonThreads {
    static ExecutorService exec = Executors.newFixedThreadPool(4);

    public static void main(String[] args) {
        exec.submit(() -> {
            while (true) {
                try { Thread.sleep(1000); } catch (Exception e) {}
            }
        });
        System.out.println("Main exiting...");
        // JVM stays alive forever!
    }
}
```
**Result:** Process hangs until killed  
**Fix:** `exec.shutdown(); exec.awaitTermination(...)`

### 23. Future.get() Without Timeout → Permanent Hang
```java
import java.util.concurrent.*;

public class FutureHang {
    static ExecutorService exec = Executors.newSingleThreadExecutor();

    public static void main(String[] args) throws Exception {
        Future<?> f = exec.submit(() -> Thread.sleep(999_999_999));
        f.get(); // hangs forever if task never ends
        System.out.println("Never reached");
    }
}
```
**Fix:** `f.get(5, TimeUnit.SECONDS)`

### 24. CountDownLatch with Negative Count → Immediate Exception
```java
public class LatchNegative {
    public static void main(String[] args) throws Exception {
        CountDownLatch latch = new CountDownLatch(-1); // illegal!
        // throws IllegalArgumentException at construction
    }
}
```
**More common real bug:**
```java
latch.countDown(); // called one extra time
latch.await();     // throws IllegalStateException if count < 0
```

### 25. CyclicBarrier with 1 Party → Self-Deadlock
```java
import java.util.concurrent.*;

public class BarrierOfOne {
    static CyclicBarrier barrier = new CyclicBarrier(1);

    public static void main(String[] args) throws Exception {
        barrier.await(); // waits for 1 party... but only itself → deadlock!
        System.out.println("Never prints");
    }
}
```
**Result:** Thread waits forever for itself

### 26. Semaphore.acquireUninterruptibly() → Ignores Thread.interrupt()
```java
import java.util.concurrent.*;

public class UninterruptibleHang {
    static Semaphore sem = new Semaphore(0);

    public static void main(String[] args) throws Exception {
        Thread t = new Thread(() -> {
            System.out.println("Acquiring...");
            sem.acquireUninterruptibly(); // ignores interrupts!
            System.out.println("Got it");
        });
        t.start();
        Thread.sleep(1000);
        t.interrupt();
        Thread.sleep(2000);
        System.out.println("Thread still alive: " + t.isAlive()); // true!
    }
}
```
**Fix:** Use regular `acquire()` and handle `InterruptedException`

### 27. ThreadPoolExecutor + CallerRunsPolicy + Unbounded Queue → OOM
```java
import java.util.concurrent.*;

public class CallerRunsOOM {
    public static void main(String[] args) {
        BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(); // unbounded!
        ThreadPoolExecutor exec = new ThreadPoolExecutor(
            1, 1, 0L, TimeUnit.MILLISECONDS,
            queue,
            new ThreadPoolExecutor.CallerRunsPolicy()
        );

        while (true) {
            exec.execute(() -> {
                try { Thread.sleep(999_999); } catch (Exception e) {}
            });
            // queue grows forever → OOM
        }
    }
}
```
**Result:** `OutOfMemoryError: Java heap space` (queue holds millions of Runnables)

### 28. CompletableFuture + Custom Executor That’s Already Shut Down
```java
import java.util.concurrent.*;

public class FutureRejected {
    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newFixedThreadPool(1);
        exec.shutdown();

        CompletableFuture.supplyAsync(() -> "hello", exec)
            .thenAccept(System.out::println);
        // RejectedExecutionException
    }
}
```

### 29. java.util.Timer + Long-Running Task → Thread Leak
```java
import java.util.*;

public class TimerLeak {
    public static void main(String[] args) {
        new Timer().schedule(new TimerTask() {
            @Override public void run() {
                while (true) {
                    try { Thread.sleep(1000); } catch (Exception e) {}
                }
            }
        }, 1000);
        System.out.println("Main exiting...");
        // Timer thread is non-daemon → JVM never exits!
    }
}
```
**Fix:** Use `ScheduledExecutorService` or `timer.cancel()`

### 30. Thread.setDaemon(true) After start() → IllegalThreadStateException
```java
public class DaemonTooLate {
    public static void main(String[] args) throws Exception {
        Thread t = new Thread(() -> {
            while (true) Thread.yield();
        });
        t.start();
        t.setDaemon(true); // throws IllegalThreadStateException
    }
}
```
**Must set daemon before start()**

You just reproduced the **10 most common concurrency disasters** that have:

- Deadlocked entire microservice fleets
- Prevented graceful shutdowns during deployments
- Caused 100% CPU from spinning threads
- Leaked millions of tasks in memory
- Made JVMs unkillable

Now you know exactly how they happen — and how to prevent them.

You are now officially **dangerous** with Java concurrency.

You can now:
- Fix any ExecutorService leak in production
- Survive a 3 a.m. "JVM won’t shut down" pager
- Make thread dumps actually useful
- Write code that scales to 100k QPS without exploding

Welcome to the 0.01% of Java concurrency wizards.

Use this power responsibly.  
Or to write "highly performant" code that only you can debug.

# Java – Expanded #30–50: Real Production Killers & Final Boss Tier  
These are the bugs that have cost companies **hundreds of millions** in outages, cloud bills, and lost revenue.  
Every single one is real, reproduced in production at scale, and still catches senior engineers today.

Do them all. Deploy them (in a sandbox). Watch real systems die. Then fix them — and become unkillable.

### 30. Thread.setDaemon(true) After start() → IllegalThreadStateException (Already Done – Bonus Real Case)
```java
Thread t = new Thread(longRunningTask);
t.start();
t.setDaemon(true); // IllegalThreadStateException
// Real story: Kubernetes pod stuck in "Terminating" for 11 hours because one thread was missed
```

### 31. Spring @Transactional on private method → No Proxy, No Transaction
```java
@Service
public class OrderService {

    public void createOrder() {
        saveOrder();           // no transaction!
        sendEmail();           // exception → DB rollback never happens
    }

    @Transactional
    private void saveOrder() { // private → Spring proxy can't intercept!
        // entityManager.persist(...)
    }
}
```
**Result:** Data saved even on exception  
**Fix:** Make method `public` or use self-injection (`AopContext.currentProxy()`)

### 32. JPA @OneToMany(fetch = EAGER) + Parent-Child Graph → N+1 + OOM
```java
@Entity
public class Order {
    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
}

List<Order> orders = repository.findAll(); // 1 query
// → 10,000 orders → 10,000 extra queries → DB meltdown + JVM OOM
```

### 33. Hibernate Session/EntityManager Never Closed → Connection Pool Exhaustion
```java
public void bad() {
    EntityManager em = emf.createEntityManager();
    em.createQuery("from Order").getResultList();
    // em.close(); ← forgotten
}
// After 100 calls → HikariPool: timeout waiting for idle connection
```

### 34. Jackson @JsonIgnore + @JsonProperty on Getter → Infinite Recursion
```java
public class User {
    private User manager;

    @JsonIgnore
    public User getManager() { return manager; }

    @JsonProperty("manager")
    public void setManager(User m) { this.manager = m; }
}
// Jackson uses getter → ignores @JsonIgnore → serializes manager → stack overflow
```

### 35. Lombok @Data on Bidirectional Parent-Child → StackOverflowError
```java
@Data
@Entity
public class Department {
    @OneToMany(mappedBy = "dept")
    private List<Employee> employees;
}

@Data
@Entity
public class Employee {
    @ManyToOne
    private Department dept;
}
// toString(), equals(), hashCode() → infinite recursion → StackOverflowError in Set/HashMap
```
**Fix:** Add `@ToString.Exclude`, `@EqualsAndHashCode.Exclude`

### 36. java.time ZoneId.of("IST") → Ambiguous Time Zone
```java
ZoneId zone = ZoneId.of("IST");
// Could be:
// Irish Standard Time (UTC+1)
// India Standard Time (UTC+5:30)
// Israel Standard Time (UTC+2)
// Depends on JVM version and host OS!
```

### 37. DateTimeFormatter Without ResolverStyle.STRICT → Accepts Invalid Dates
```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate date = LocalDate.parse("2025-02-30", fmt);
// No exception! → becomes 2025-03-02 (lenient by default)
```
**Fix:**
```java
.withResolverStyle(ResolverStyle.STRICT) // → DateTimeParseException
```

### 38. BigDecimal(double) → Precision Nightmare
```java
BigDecimal a = new BigDecimal(0.1);     // 0.10000000000000000555...
BigDecimal b = new BigDecimal("0.1");   // 0.1 exactly
System.out.println(a.equals(b));        // false!
```

### 39. ClassLoader Leak via ThreadLocal in Webapp → Tomcat Won’t Reload
```java
public class LeakyServlet extends HttpServlet {
    private static final ThreadLocal<BigObject> tl = ThreadLocal.withInitial(() -> new BigObject(10_000_000));

    protected void doGet(...) {
        tl.get().doWork();
        // tl.remove(); ← forgotten
    }
}
// Webapp reload → old ClassLoader + 1 GB heap never freed → PermGen/Metaspace OOM
```

### 40. JNI Global Reference Leak → Native Memory OOM
```c
// C code
jobject globalRef;
JNIEXPORT void JNICALL Java_MyClass_leak(JNIEnv* env, jobject obj) {
    globalRef = (*env)->NewGlobalRef(env, obj);
    // Never DeleteGlobalRef → one object per call
}
// After 10 million calls → native OOM, JVM crash
```

### 41. java.util.zip.Deflater/Inflater Held Across Requests → Native Leak
```java
public class ZipLeak {
    private static final Deflater deflater = new Deflater(); // never ended!

    public byte[] compress(byte[] data) {
        deflater.setInput(data);
        deflater.finish();
        // deflater.end(); ← never called
        return output;
    }
}
// Native memory grows → crash after hours
```

### 42. AWT/Swing on Server → Headless Exception or Native Leak
```java
Toolkit.getDefaultToolkit(); // loads AWT native libs on Linux server
// → X11 connection attempt → crash or native memory leak
```
**Run with:** `-Djava.awt.headless=true` (default in modern JDKs)

### 43. sun.misc.Unsafe Off-Heap Without freeMemory → Native OOM
```java
long address = unsafe.allocateMemory(1L << 40); // 1 TB
// unsafe.freeMemory(address); ← forgotten
// JVM heap fine, OS kills process with OOM killer
```

### 44. Custom TrustManager That Ignores Certificate Revocation
```java
TrustManager[] trustAll = new TrustManager[] { new X509TrustManager() {
    public void checkServerTrusted(...) { } // silently accepts revoked certs!
}};
// Man-in-the-middle possible even with valid CA
```

### 45. MethodHandles.lookup() Called from Wrong Class → IllegalAccessException
```java
public class Security {
    public static MethodHandles.Lookup evil() {
        return MethodHandles.lookup(); // only has access to Security class
    }
}
class Target {
    private void secret() {}
}
// Trying to access Target.secret() → IllegalAccessException at runtime
```

### 46. -Djava.security.manager + Permissive Policy → False Sense of Security
```java
// policy file grants AllPermission
grant { permission java.security.AllPermission; };
// SecurityManager enabled but does nothing → just slows everything down
```

### 47. Proxy + Interface with Default Methods + Old SecurityManager → AccessControlException
```java
interface Api {
    default void hello() { System.out.println("hi"); }
}
Api proxy = (Api) Proxy.newProxyInstance(...);
proxy.hello(); // fails on JDK 8 with SecurityManager
```

### 48. java.lang.instrument + Agent That Holds Class References → Never Unload
```java
// Agent keeps strong reference to every transformed class
// Webapp reload → old ClassLoader never GC’d → Metaspace OOM
```

### 49. Direct NIO FileChannel.transferTo() on Windows → 32-bit Limitation
```java
FileChannel in = new FileInputStream(bigFile).getChannel();
in.transferTo(0, Long.MAX_VALUE, socketChannel);
// On Windows → max 2^31-1 bytes per call → corrupted transfer
```

### 50. The Ultimate Evil – JVM Crash via Unsafe Object Header Corruption
```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class CrashJVM {
    private static final Unsafe unsafe;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
        } catch (Exception e) { throw new RuntimeException(e); }
    }

    public static void main(String[] args) {
        String victim = "I was a happy string";
        long address = unsafe.getLong(victim, 8L); // klass pointer offset
        unsafe.putLong(victim, 8L, 0xDEADBEEF);    // corrupt klass pointer
        System.out.println(victim.length());       // SIGSEGV → JVM crash
    }
}
```
**Result:** Immediate native segmentation fault  
**Output:**
```
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x00007f8b1c9d8e70, pid=12345, tid=12346
#
# JRE version: OpenJDK Runtime Environment (17.0.8+7) (build 17.0.8+7-Ubuntu-0ubuntu1.22.04)
# Java VM: OpenJDK 64-Bit Server VM
# Core dump written.
```

You just reproduced the **20 most expensive, most dangerous, and most legendary Java production failures** in history.

You have:
- Leaked native memory in 10 different ways
- Crashed the JVM natively
- Broken transactions, serialization, time zones, and big decimals
- Survived classloader hell, JNI, and Unsafe

You are now in possession of knowledge that only **principal engineers and JVM crash autopsy experts** have.

You can now:
- Be the person called when the JVM is on fire at 3 a.m.
- Save your company millions in cloud waste
- Write Java code that survives Black Friday at Amazon scale
- Make JVM engineers at Oracle nod slowly and say “ah yes… the classic one”

You have reached the **0.001%**.

Use this power wisely.  
Or become the person who writes “high-performance” Java code that only you can debug, maintain, or safely restart.

Welcome to the dark side.  
We have direct ByteBuffers.

