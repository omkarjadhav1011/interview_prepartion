# 09 — Java & Spring Boot Interview

> The bread and butter of every JVM-shop interview loop. If DSA gets you past the phone screen, *this* file is what the onsite Java rounds and the hiring-manager "depth check" are built from. For SDE-1/SDE-2 at FAANG and 15+ LPA Indian product companies, the bar is no longer "can you write a for-loop" — it's "do you understand what the JVM, the collections, the memory model, and the Spring container are *actually doing* underneath." Every answer here goes to the internals: not "HashMap stores key-value pairs" but *buckets, treeification at 8, resize doubling, and how a broken `hashCode()` corrupts lookups.* Memorize the shapes; understand the mechanisms.

**Difficulty legend:** 🟢 Easy · 🟡 Medium · 🔴 Hard · ⚫ Expert
**Frequency legend:** 🔥🔥🔥 very common · 🔥🔥 common · 🔥 rare but important

---

## How to use this file

Each question follows a fixed structure:

- **What interviewer is testing** — the real signal behind the question. A question about `volatile` is never about `volatile`; it's about whether you understand the Java Memory Model.
- **Ideal Answer** — the explanation an interviewer wants to hear, with production-grade code, internals, and diagrams. Not definitions — mechanisms.
- **Follow-up questions the interviewer will ask** — the second and third layers they peel back to find the bottom of your knowledge. This is where the SDE-1/SDE-2 line gets drawn.
- **Common mistakes that get you rejected** — the specific wrong answers and half-truths that flip a "hire" to "no hire."

The meta-skill is **depth on demand**. A weak candidate says "ConcurrentHashMap is thread-safe." A strong one says "it uses CAS for the initial insert into an empty bin and synchronizes on the bin head node for collisions, so lock granularity is per-bucket, not per-map like the old `Hashtable`." Aim to be the second candidate on every topic.

---

## Table of Contents

### Core Java

**OOP & Language fundamentals**
1. The four pillars of OOP — with code
2. Abstraction vs Encapsulation — the difference people fumble
3. Abstract class vs Interface (Java 8+)
4. `equals()` and `hashCode()` contract — from scratch
5. How a broken `hashCode()` corrupts a HashMap
6. String immutability and why it matters
7. The String pool, `intern()`, `==` vs `equals()`
8. StringBuilder vs StringBuffer vs String concatenation
9. `final`, `finally`, `finalize`
10. Pass-by-value: Java has no pass-by-reference
11. Static vs instance: initialization order
12. Comparable vs Comparator

**Exceptions**
13. Checked vs unchecked exceptions
14. Custom exceptions — when and how
15. try-with-resources and `AutoCloseable`
16. Exception best practices that get noticed

**Generics**
17. Generics and type erasure
18. Wildcards: `? extends` vs `? super` and PECS
19. Why you can't create a generic array

**Collections internals**
20. HashMap internals — buckets, treeification, resize, load factor
21. ConcurrentHashMap internals — CAS + bin locking
22. ArrayList vs LinkedList
23. TreeMap and the red-black tree
24. Fail-fast vs fail-safe iterators
25. HashSet vs LinkedHashSet vs TreeSet
26. `Hashtable` vs `HashMap` vs `ConcurrentHashMap`

**Multithreading & concurrency**
27. Thread lifecycle and states
28. `Runnable` vs `Callable`, `Thread` vs pool
29. `synchronized` — monitors, intrinsic locks
30. `volatile` — visibility vs atomicity
31. The happens-before relationship and the JMM
32. `wait()`, `notify()`, `notifyAll()`
33. Creating a deadlock — and four ways to prevent it
34. Atomic classes and CAS
35. `ReentrantLock` vs `synchronized`
36. ThreadLocal — uses and the memory leak trap
37. ExecutorService and ThreadPoolExecutor tuning
38. Future vs CompletableFuture
39. CompletableFuture chaining: thenApply / thenCompose / thenCombine
40. The producer-consumer problem with BlockingQueue

**Java 8 functional**
41. Streams — intermediate vs terminal, lazy evaluation
42. Parallel streams and their pitfalls
43. Optional — the right and wrong way
44. Functional interfaces: Predicate / Function / Consumer / Supplier
45. Method references — the four kinds
46. `Comparator.comparing` chains
47. `Collectors.groupingBy` and downstream collectors
48. `reduce` vs `collect`

**Java 17–21 modern**
49. Records
50. Sealed classes and interfaces
51. Pattern matching for `switch`
52. Text blocks
53. Virtual threads (Project Loom) — platform vs virtual
54. When to use virtual threads (and when not to)

**JVM & memory**
55. JVM memory model — heap, stack, metaspace
56. Generational GC — young, old, the collection cycle
57. G1 vs ZGC vs Parallel GC
58. Types of OutOfMemoryError and how to troubleshoot
59. Escape analysis and scalar replacement
60. Class loading and the classloader hierarchy

**Serialization & patterns**
61. Serialization dangers, `transient`, `serialVersionUID`
62. Singleton — enum vs double-checked locking
63. Factory and Abstract Factory
64. Builder pattern
65. Observer pattern
66. Strategy pattern
67. Proxy and dynamic proxy

### Spring Boot

**IoC & DI**
68. What IoC and DI really mean
69. `@Autowired` resolution: by type then by name
70. Constructor vs field vs setter injection
71. Circular dependency — how Spring resolves it (and when it can't)
72. `@Qualifier`, `@Primary`, `@Resource`

**Beans**
73. Bean lifecycle — the full sequence
74. BeanPostProcessor and BeanFactoryPostProcessor
75. Bean scopes — singleton, prototype, request, session
76. `@Component` vs `@Service` vs `@Repository` vs `@Controller`
77. `@Configuration` CGLIB proxy vs lite mode

**Auto-configuration**
78. How Spring Boot auto-configuration works
79. `@ConditionalOnClass` / `@ConditionalOnMissingBean` and overriding
80. Writing your own auto-configuration / starter

**Web MVC**
81. DispatcherServlet request flow — every step
82. `@RestController` vs `@Controller`, `@RequestBody` / `@ResponseBody`
83. Filter vs Interceptor vs AOP

**Transactions**
84. `@Transactional` — the proxy mechanism
85. Propagation: REQUIRED / REQUIRES_NEW / NESTED
86. Isolation levels and the anomalies they prevent
87. The self-invocation trap
88. Rollback rules — why checked exceptions don't roll back

**Data / JPA**
89. Spring Data JPA repository hierarchy
90. Query derivation and `@Query`
91. The N+1 problem and how to fix it
92. Lazy vs eager fetching, `LazyInitializationException`
93. The first-level (persistence-context) cache
94. `save` vs `saveAndFlush`, entity states

**Security**
95. The Spring Security filter chain
96. JWT authentication flow end-to-end
97. OAuth2 / OpenID Connect basics
98. `@PreAuthorize` and method security
99. CSRF — what it is and when to disable it

**Errors, validation, config**
100. `@ControllerAdvice` and `@ExceptionHandler`
101. ProblemDetail / RFC 7807
102. Bean Validation: `@Valid`, custom validators, groups
103. Profiles and externalized configuration
104. `@ConfigurationProperties` vs `@Value`

**Ops & testing**
105. Actuator — endpoints, health, metrics
106. `@SpringBootTest` vs `@WebMvcTest` vs `@DataJpaTest`
107. `@MockBean`, MockMvc, Testcontainers
108. Spring Boot 3.x — Jakarta, AOT, native image, observability

**Microservices**
109. Circuit breaker with Resilience4j
110. Service discovery and client-side load balancing
111. API gateway
112. Config server
113. Distributed tracing

---

# CORE JAVA

The single most important framing for Java rounds: **interviewers assume you can write the code; they're testing whether you know what runs.** When you answer, narrate the mechanism. "I'll override `equals` — and because the contract requires it, `hashCode` too, otherwise the object won't be found in a HashMap even after being put there." That one sentence signals SDE-2.

---

### Q1: The Four Pillars of OOP — with code
**Company:** TCS, Infosys, Accenture, Amazon (phone), Wipro
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Explain the four pillars of object-oriented programming. Don't just define them — show each with code.

**What interviewer is testing:**
This is a warm-up and a filter. They want to hear *applied* understanding, not textbook definitions. The give-away of a weak candidate is reciting "encapsulation is data hiding" with no example. A strong candidate shows polymorphism resolving at runtime and explains *why* each pillar exists.

**Ideal Answer:**

The four pillars are **Encapsulation, Abstraction, Inheritance, and Polymorphism**. Each solves a real design problem.

**1. Encapsulation** — bundle state with the behavior that operates on it, and control access through methods so invariants can't be violated. The point isn't "make fields private"; it's *protecting invariants*.

```java
public class BankAccount {
    private long balanceCents;          // hidden state

    public void deposit(long cents) {
        if (cents <= 0) throw new IllegalArgumentException("must be positive");
        balanceCents += cents;          // invariant enforced here, nowhere else
    }

    public void withdraw(long cents) {
        if (cents > balanceCents) throw new IllegalStateException("insufficient funds");
        balanceCents -= cents;
    }

    public long getBalanceCents() { return balanceCents; }
}
```

If `balanceCents` were public, any code could set it negative — the invariant "balance never goes negative" could be broken anywhere. Encapsulation localizes the rule.

**2. Abstraction** — expose *what* an object does, hide *how*. Interfaces and abstract classes are the tools.

```java
public interface PaymentGateway {
    PaymentResult charge(Money amount, Card card);  // what, not how
}

public class StripeGateway implements PaymentGateway {
    public PaymentResult charge(Money amount, Card card) {
        // HTTP calls, retries, idempotency keys — all hidden from callers
        return new PaymentResult(/* ... */);
    }
}
```

Callers depend on `PaymentGateway`, not `StripeGateway`. Swapping to `PayPalGateway` changes nothing in calling code.

**3. Inheritance** — model an "is-a" relationship and reuse behavior.

```java
public abstract class Employee {
    protected final String name;
    protected Employee(String name) { this.name = name; }
    public abstract double monthlySalary();
    public String label() { return name + ": " + monthlySalary(); } // shared
}

public class SalariedEmployee extends Employee {
    private final double annual;
    public SalariedEmployee(String name, double annual) { super(name); this.annual = annual; }
    public double monthlySalary() { return annual / 12; }
}

public class HourlyEmployee extends Employee {
    private final double rate; private final int hours;
    public HourlyEmployee(String name, double rate, int hours) {
        super(name); this.rate = rate; this.hours = hours;
    }
    public double monthlySalary() { return rate * hours; }
}
```

**4. Polymorphism** — one interface, many implementations, resolved at runtime (dynamic dispatch).

```java
List<Employee> staff = List.of(
    new SalariedEmployee("Asha", 1_200_000),
    new HourlyEmployee("Ravi", 500, 160));

for (Employee e : staff) {
    System.out.println(e.label());   // calls the right monthlySalary() per object
}
```

At compile time the reference type is `Employee`; at runtime the JVM dispatches to the actual object's `monthlySalary()` via the **virtual method table (vtable)**. This is *runtime* (dynamic) polymorphism. **Overloading** is *compile-time* polymorphism — resolved by the compiler from argument types.

**Follow-up questions the interviewer will ask:**

1. *"Difference between overloading and overriding?"* → Overloading: same method name, different parameter lists, resolved at **compile time** by static argument types. Overriding: subclass redefines a superclass method with the same signature, resolved at **runtime** by the object's actual type. Overriding requires the same signature and a covariant-or-same return type; overloading is just a coincidence of names.

2. *"Can you override a static method?"* → No. Static methods belong to the class, not the instance, so they are *hidden*, not overridden. Calling through a subclass reference uses the declared type, not the runtime type — there's no dynamic dispatch for statics.

3. *"Why prefer composition over inheritance?"* → Inheritance couples a subclass to its superclass's implementation (the "fragile base class" problem) and Java allows only single inheritance. Composition (`has-a`) is more flexible: you delegate to a field, can swap it at runtime, and avoid deep brittle hierarchies. The guideline: inherit only for genuine `is-a` with a stable base.

**Common mistakes that get you rejected:**
- Reciting definitions with zero code when asked for code.
- Saying encapsulation is "making fields private" — it's about protecting invariants; private fields are just the mechanism.
- Claiming static methods can be overridden.
- Confusing overloading (compile-time) with overriding (runtime).

---

### Q2: Abstraction vs Encapsulation
**Company:** Infosys, Cognizant, Amazon, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
People mix these up constantly. What's the actual difference between abstraction and encapsulation?

**What interviewer is testing:**
Conceptual clarity. Both "hide" something, so candidates blur them. The distinction: abstraction hides **complexity** (design level — what vs how); encapsulation hides **data** (implementation level — controlling access to state).

**Ideal Answer:**

They operate at different levels:

- **Abstraction** is a *design-time* concern: it hides **complexity** by exposing only the essential interface. You ask "what operations should this expose?" Achieved with interfaces and abstract classes. Example: `List` abstracts away whether it's backed by an array or a linked structure.

- **Encapsulation** is an *implementation-time* concern: it hides **internal state** and bundles it with the methods that mutate it, controlling access via modifiers. You ask "how do I keep this object's data consistent?" Achieved with access modifiers and getters/setters with validation.

A clean way to say it: *abstraction is about the outside view (the contract); encapsulation is about the inside protection (the data).* You can have one without the other — a class with all-public fields behind an interface has abstraction but no encapsulation.

```java
// Abstraction: the interface says WHAT, not HOW
interface Cache<K, V> {
    Optional<V> get(K key);
    void put(K key, V value);
}

// Encapsulation: state is private, access is controlled, invariant (size cap) protected
class LruCache<K, V> implements Cache<K, V> {
    private final int capacity;                              // encapsulated
    private final LinkedHashMap<K, V> map;                   // encapsulated

    LruCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(16, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<K, V> e) {
                return size() > LruCache.this.capacity;      // invariant enforced internally
            }
        };
    }
    public Optional<V> get(K key) { return Optional.ofNullable(map.get(key)); }
    public void put(K key, V value) { map.put(key, value); }
}
```

The `Cache` interface is abstraction. The private `map` and `capacity`, plus the eviction rule living inside the class, are encapsulation.

**Follow-up questions the interviewer will ask:**

1. *"Can abstraction exist without encapsulation?"* → Yes. An interface implemented by a class with public mutable fields gives you abstraction (callers code to the interface) but no encapsulation (state is exposed). They're orthogonal.

2. *"How does Java enforce each?"* → Abstraction: `interface`, `abstract` classes/methods. Encapsulation: access modifiers (`private`, `protected`, package-private, `public`) plus accessor methods with validation.

**Common mistakes that get you rejected:**
- Saying "they're the same thing, both hide stuff."
- Not being able to give the design-level vs implementation-level distinction.

---

### Q3: Abstract Class vs Interface (Java 8+)
**Company:** Amazon, Microsoft, Adobe, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
When would you use an abstract class versus an interface? Java 8 added default methods to interfaces — does that erase the difference?

**What interviewer is testing:**
Whether you know the post-Java-8 reality. Default methods blurred the line, but real differences remain: state, constructors, single vs multiple inheritance.

**Ideal Answer:**

After Java 8, interfaces can have `default` and `static` methods, and Java 9 added `private` interface methods. So "interfaces can't have implementations" is no longer true. But these remain:

| Aspect | Abstract class | Interface |
|---|---|---|
| Instance fields / state | Yes (mutable state) | No (only `public static final` constants) |
| Constructors | Yes | No |
| Multiple inheritance | No (single superclass) | Yes (implement many) |
| Method implementations | Yes | `default` / `static` / `private` only |
| Access modifiers on methods | any | `public` (abstract) by default |
| Use when | shared state + base behavior, "is-a" | capability / contract, "can-do" |

**Rule of thumb:** use an **interface** to define a *capability* that unrelated types can have (`Comparable`, `Serializable`, `Runnable`), and use an **abstract class** when implementations share **state** and a common skeletal implementation.

```java
// Interface: a capability, possibly with default behavior
interface Notifier {
    void send(String message);

    default void sendUrgent(String message) {  // default method (Java 8+)
        send("[URGENT] " + message);
    }
}

// Abstract class: shared STATE + skeletal algorithm (Template Method)
abstract class RetryingNotifier implements Notifier {
    private int retries;                         // state — interfaces can't hold this
    protected RetryingNotifier(int retries) { this.retries = retries; }

    public final void send(String message) {     // template: shared algorithm
        for (int i = 0; i <= retries; i++) {
            if (doSend(message)) return;
        }
        throw new IllegalStateException("send failed after retries");
    }
    protected abstract boolean doSend(String message);  // the varying step
}
```

The interface declares the capability; the abstract class supplies retry *state* and the retry *loop* — neither of which an interface can hold.

**Follow-up questions the interviewer will ask:**

1. *"Diamond problem with default methods — what happens if two interfaces define the same default method?"* → The implementing class **must** override it, otherwise compile error. You can delegate to a specific one: `InterfaceA.super.method()`.

```java
interface A { default String hi() { return "A"; } }
interface B { default String hi() { return "B"; } }
class C implements A, B {
    public String hi() { return A.super.hi(); }  // forced to disambiguate
}
```

2. *"Why can't interfaces have instance state?"* → Multiple inheritance of state causes the classic diamond ambiguity (which copy of the field?). Java avoids it by allowing multiple inheritance of *behavior* (default methods) but not *state*.

3. *"Class extends a class with method `foo` and implements an interface with default `foo` — which wins?"* → **Class wins.** "Class always beats interface" — a concrete method from the superclass takes precedence over an interface default.

**Common mistakes that get you rejected:**
- Saying "interfaces can't have method bodies" (false since Java 8).
- Missing that the real distinguishers are *state* and *constructors*.
- Not knowing the diamond resolution rules.

---

### Q4: `equals()` and `hashCode()` Contract — From Scratch
**Company:** Amazon, Google, Microsoft, Meta, Goldman Sachs, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Write `equals()` and `hashCode()` for a class from scratch. State the contract between them and explain what breaks if you violate it.

**What interviewer is testing:**
This is one of the highest-signal Java questions. They want the **contract**, a correct hand-written implementation (not just `Objects.hash`), and the consequences — specifically how a bad `hashCode` makes objects "disappear" from a HashMap.

**Ideal Answer:**

**The contract** (from `Object`):

1. **Reflexive:** `x.equals(x)` is `true`.
2. **Symmetric:** `x.equals(y)` ⇔ `y.equals(x)`.
3. **Transitive:** `x.equals(y)` and `y.equals(z)` ⇒ `x.equals(z)`.
4. **Consistent:** repeated calls return the same result if the objects don't change.
5. **Non-null:** `x.equals(null)` is `false`.

And the crucial link:
6. **If `x.equals(y)` then `x.hashCode() == y.hashCode()`.** (The reverse is *not* required — unequal objects *may* share a hash code; that's a collision.)

Hand-written implementation:

```java
public final class Money {
    private final long amountCents;
    private final String currency;          // e.g. "USD"

    public Money(long amountCents, String currency) {
        this.amountCents = amountCents;
        this.currency = Objects.requireNonNull(currency);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                       // 1. reference shortcut
        if (o == null || getClass() != o.getClass()) return false; // 2. exact type
        Money other = (Money) o;                          // 3. cast
        return amountCents == other.amountCents           // 4. compare each significant field
            && currency.equals(other.currency);
    }

    @Override
    public int hashCode() {
        int result = Long.hashCode(amountCents);          // start with a field
        result = 31 * result + currency.hashCode();       // 31: odd prime, JIT does (x<<5)-x
        return result;
        // equivalently: return Objects.hash(amountCents, currency);
    }
}
```

**Why 31?** It's an odd prime. Odd avoids information loss that an even multiplier causes (multiplying by an even number shifts bits left, eventually pushing them out). Prime spreads bits well. And `31 * i == (i << 5) - i`, which the JIT optimizes to a shift and subtract. The *exact* multiplier matters far less than including every `equals`-significant field.

**The rule that ties it together:** *fields used in `equals()` must be exactly the fields used in `hashCode()`.* If `equals` compares `amountCents` and `currency`, `hashCode` must derive from both.

**What breaks — the HashMap corruption:** A `HashMap` finds the bucket via `hashCode()`, then confirms with `equals()` inside that bucket. If two equal objects return *different* hash codes, they land in different buckets, so `map.get(key)` looks in the wrong bucket and returns `null` — the entry is effectively *lost* even though you put it there.

```java
class BadKey {
    final int id;
    BadKey(int id) { this.id = id; }
    public boolean equals(Object o) { return o instanceof BadKey b && b.id == id; }
    // NO hashCode override -> uses Object's identity hash
}

Map<BadKey, String> map = new HashMap<>();
map.put(new BadKey(1), "one");
System.out.println(map.get(new BadKey(1))); // null! different identity hash -> wrong bucket
```

The two `BadKey(1)` instances are `equals`, but their `Object.hashCode()` differs (identity-based), so they hash to different buckets. The lookup never even reaches the `equals` check.

**Follow-up questions the interviewer will ask:**

1. *"`getClass()` vs `instanceof` in equals — which and why?"* → `getClass()` enforces *exact* type equality (a `Money` is never equal to a subclass), preserving symmetry under inheritance. `instanceof` allows subclass instances to be equal to superclass instances, which **breaks symmetry** if the subclass adds state — `super.equals(sub)` might be true while `sub.equals(super)` is false. Use `getClass()` for value classes; if you use `instanceof`, the class should generally be `final` or you must be very careful. (Records and Lombok use exact-type semantics.)

2. *"What if a key's hashCode changes after it's in the map (mutable key)?"* → The entry becomes unreachable. It was placed in the bucket for the *old* hash; lookups now compute the *new* hash and search a different bucket. **Never use mutable objects as HashMap keys** — or never mutate the fields that participate in `hashCode` while it's a key. This is the strongest argument for immutable keys.

3. *"Is returning a constant from hashCode legal?"* → Legal (it satisfies the contract — equal objects share it), but catastrophic for performance: every entry collides into one bucket, degrading `HashMap` from O(1) to O(n), or O(log n) after treeification. The contract is about *correctness*; performance needs good distribution.

**Common mistakes that get you rejected:**
- Overriding `equals` but not `hashCode` (the classic bug — entries vanish from hash collections).
- Using different fields in `equals` and `hashCode`.
- Using `instanceof` for a value type with subclasses and breaking symmetry.
- Forgetting the `null` and type checks in `equals` (NPE / ClassCastException).
- Using mutable fields in `hashCode` and then mutating a key.

---

### Q5: How a Broken `hashCode()` Corrupts a HashMap
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Walk me through, step by step, exactly what happens inside a HashMap when you put and then get a key whose `hashCode` is inconsistent with `equals`.

**What interviewer is testing:**
Mechanical understanding of HashMap's two-step lookup (bucket by hash, then equals within bucket). It's Q4 from the data-structure side.

**Ideal Answer:**

A `HashMap` lookup is a **two-stage** process:

1. **Find the bucket:** compute `hash(key)` (HashMap spreads the bits: `h = key.hashCode(); h ^= (h >>> 16);`), then `index = (n - 1) & hash` where `n` is the table size (a power of two).
2. **Search within the bucket:** walk the bucket's linked list (or red-black tree), comparing each node's key with `key.equals(node.key)` (after a fast `==` and hash check).

Now trace the broken case where `equals` says two objects are equal but `hashCode` differs:

```
put(k1, v):   hashCode(k1) = 100 -> bucket 4 -> store (k1, v) in bucket 4
get(k2):      hashCode(k2) = 250 -> bucket 10 -> search bucket 10 -> empty -> return null
              (even though k1.equals(k2) is true!)
```

The `equals` check in step 2 **never runs against k1**, because step 1 already routed the lookup to the wrong bucket. The entry is present in the table but unreachable through `get`.

The mirror failure — `hashCode` consistent but `equals` always `false` — puts both in the same bucket but the in-bucket comparison fails, so again `get` returns `null` and `put` creates duplicates.

**Visual:**

```
Table (n = 16):
 [0] [1] [2] [3] [4]->(k1,v) [5] ... [10](empty) ...
                    ^                  ^
            put landed here     get searched here  -> miss
```

**Complexity impact even when correct:** With a good `hashCode`, entries spread across buckets → average O(1). With a constant `hashCode`, all entries pile into one bucket → O(n) per operation (until treeification at 8 entries makes it O(log n)).

**Follow-up questions the interviewer will ask:**

1. *"Why does HashMap re-hash with `h ^ (h >>> 16)`?"* → Because the bucket index is `(n-1) & hash`, only the low bits select the bucket when `n` is small. XOR-ing in the high 16 bits mixes high-order bits into the low ones, so keys whose hash codes differ only in high bits don't all collide.

2. *"At what point does a bucket become a tree?"* → When a single bucket reaches **8** entries (`TREEIFY_THRESHOLD`) *and* the table size is ≥ 64 (`MIN_TREEIFY_CAPACITY`); below 64 it resizes instead. It reverts to a list when a bucket shrinks to **6** (`UNTREEIFY_THRESHOLD`).

**Common mistakes that get you rejected:**
- Saying "get returns null because equals fails" — no, `equals` is never reached; the *bucket* is wrong.
- Not knowing the two-stage (bucket then equals) structure.

---

### Q6: String Immutability and Why It Matters
**Company:** Amazon, Microsoft, Oracle, Adobe
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Why is `String` immutable in Java? Give concrete benefits.

**What interviewer is testing:**
Whether you understand the *design rationale* — security, caching, thread safety, hashing — not just "you can't change it."

**Ideal Answer:**

`String` is immutable: once created, its character content never changes. `String` is `final`, and internally the backing array (`byte[] value` since Java 9's compact strings, `char[]` before) is `private final` and never exposed or mutated. Methods like `substring`, `toUpperCase`, `replace` return **new** `String` objects.

Why the language designers chose this:

1. **String pool / interning.** Because strings can't change, the JVM can safely share one instance across all references to the same literal. `String a = "hi"; String b = "hi";` point to the *same* pooled object. This is only safe because neither can mutate it under the other.

2. **Security.** Strings carry filenames, URLs, DB connection strings, class names, usernames. If a `String` could change after a security check, a TOCTOU (time-of-check-to-time-of-use) attack would be trivial: validate `"/safe/path"`, then mutate it to `"/etc/passwd"` before use. Immutability closes that hole.

3. **Thread safety.** Immutable objects are inherently thread-safe — no synchronization needed because there's no mutable state to race on. Strings can be shared freely across threads.

4. **Hashcode caching.** `String` caches its hash code (`private int hash;`) after first computation. This is safe only because the content can't change. Since strings are the most common `HashMap` keys, this is a big win.

```java
String s = "hello";
String t = s.toUpperCase();   // new object; s is untouched
System.out.println(s);        // "hello"  (unchanged)
System.out.println(t);        // "HELLO"  (new)

// hash is cached after first use
"hello".hashCode();           // computed
"hello".hashCode();           // returned from cache
```

**Follow-up questions the interviewer will ask:**

1. *"Isn't immutability wasteful — every concatenation makes a new object?"* → Yes, naive `+=` in a loop is O(n²) garbage. That's exactly why `StringBuilder` exists for mutation-heavy building. The compiler also optimizes single-expression `+` concatenations (via `StringConcatFactory` / `invokedynamic` since Java 9).

2. *"Can reflection break String immutability?"* → Yes — you can `setAccessible(true)` on the private `value` array and mutate it, which corrupts the pool and any shared references. But this requires bypassing access control and is a known anti-pattern; the security guarantees assume reflection isn't abused (and the module system / SecurityManager historically restricted it).

3. *"Is `StringBuilder` immutable?"* → No, it's mutable by design — that's the point. It's also *not* thread-safe (unlike `StringBuffer`).

**Common mistakes that get you rejected:**
- Saying only "you can't change it" with no design rationale.
- Claiming concatenation modifies the original string.
- Not mentioning the pool/security/thread-safety/hashing benefits.

---

### Q7: The String Pool, `intern()`, `==` vs `equals()`
**Company:** Amazon, Oracle, Wipro, TCS
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Explain the String pool. What does `new String("x")` create? When is `==` true for strings?

**What interviewer is testing:**
The classic `==` vs `equals` trap and understanding of literal interning. This catches a lot of candidates.

**Ideal Answer:**

The **String pool** (a.k.a. string constant pool, part of the heap since Java 7) holds unique string literals. When the compiler sees a literal `"x"`, it adds it to the pool; all identical literals across the program share that one instance.

- `==` compares **references** (same object?).
- `equals` compares **content** (same characters?).

The traps:

```java
String a = "hello";              // pooled literal
String b = "hello";              // SAME pooled object
System.out.println(a == b);      // true  (same reference)

String c = new String("hello");  // NEW object on the heap, NOT the pooled one
System.out.println(a == c);      // false (different reference)
System.out.println(a.equals(c)); // true  (same content)

String d = c.intern();           // returns the pooled instance
System.out.println(a == d);      // true
```

`new String("hello")` creates **two** objects if `"hello"` wasn't already pooled: the pooled literal and a separate heap object — that's why `new String(...)` is wasteful for literals. `intern()` returns the canonical pooled reference (adding it if absent).

**Compile-time constant folding:**

```java
String x = "he" + "llo";         // folded to "hello" at compile time -> pooled
System.out.println(x == "hello");// true

String part = "he";
String y = part + "llo";         // runtime concatenation -> NEW object, not pooled
System.out.println(y == "hello");// false
final String fpart = "he";
String z = fpart + "llo";        // fpart is a compile-time constant -> folded -> pooled
System.out.println(z == "hello");// true
```

Concatenation of compile-time constants is folded by the compiler and pooled; concatenation involving a *non-final variable* happens at runtime and produces a fresh, unpooled object.

**Follow-up questions the interviewer will ask:**

1. *"When should you use `intern()`?"* → Rarely. It can save memory when you have *many* duplicate strings at runtime (e.g., parsing a file with repeated tokens), letting them share one instance. But it has a cost (pool lookup, and historically pool space pressure), so measure first. Modern alternatives like a `Map<String,String>` deduplication or G1's string deduplication often serve better.

2. *"Always use `==` or `equals` for strings?"* → Almost always `equals` (or `equalsIgnoreCase`). Use `==` only when you *intend* reference identity, which for strings is almost never. For null-safety, `Objects.equals(a, b)` or `"literal".equals(var)`.

3. *"Where does the pool live?"* → In the **heap** since Java 7 (moved from PermGen). So it's subject to GC and `-Xmx`, not a separate fixed region.

**Common mistakes that get you rejected:**
- Using `==` to compare string content in real code.
- Saying `new String("x") == "x"` is true.
- Not knowing compile-time constant folding makes `"a"+"b" == "ab"` true.

---

### Q8: StringBuilder vs StringBuffer vs String Concatenation
**Company:** Amazon, Microsoft, Infosys, Capgemini
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
When do you use `String`, `StringBuilder`, and `StringBuffer`? What's the performance and thread-safety difference?

**What interviewer is testing:**
Practical judgment about mutable string building and awareness that `StringBuffer`'s synchronization is almost always unnecessary overhead.

**Ideal Answer:**

| | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutable | No | Yes | Yes |
| Thread-safe | Yes (immutable) | **No** | Yes (synchronized) |
| Performance | slow for repeated edits | fastest | slower (lock overhead) |
| Since | 1.0 | 1.5 | 1.0 |

**Rule:** build strings with `StringBuilder`. Use `StringBuffer` only if a *single* builder instance is genuinely shared and mutated across threads — which is rare; usually you'd use a different design. Use `String` for fixed text.

The performance trap with `String` in loops:

```java
// O(n^2): each += allocates a new String and copies all prior characters
String s = "";
for (int i = 0; i < n; i++) s += i;       // BAD

// O(n): one growable buffer, amortized doubling
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) sb.append(i); // GOOD
String result = sb.toString();
```

`StringBuilder` keeps an internal `char[]`/`byte[]` that grows by doubling (amortized O(1) append), so building n characters is O(n) total. The `String +=` version reallocates and copies on every iteration → O(n²).

**Note:** the compiler turns a *single* `a + b + c` expression into efficient `StringConcatFactory` calls (Java 9+) or one `StringBuilder` (Java 8), so simple concatenations are fine. The problem is concatenation *inside a loop*, which the compiler can't collapse.

**Follow-up questions the interviewer will ask:**

1. *"Does `StringBuffer`'s thread safety make it safe to use across threads without thought?"* → No. Each *method* is synchronized (atomic individually), but a *sequence* of calls (`if (sb.length() > 0) sb.append(x)`) is still a race — the compound operation isn't atomic. Per-method synchronization rarely gives you the atomicity you actually need, which is why explicit external synchronization or a different design is usually correct.

2. *"How does StringBuilder grow?"* → Default capacity 16; when full it grows to `oldCapacity * 2 + 2`. Pre-size with `new StringBuilder(expectedSize)` to avoid resize copies if you know the size.

**Common mistakes that get you rejected:**
- Recommending `StringBuffer` as the default "thread-safe is safer" choice — it's needless overhead.
- Using `String +=` in a loop.
- Thinking `StringBuffer`'s method-level synchronization gives compound-operation safety.

---

### Q9: `final`, `finally`, `finalize`
**Company:** Infosys, TCS, Wipro, Amazon
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Distinguish `final`, `finally`, and `finalize()`.

**What interviewer is testing:**
A classic "do you know these are unrelated" question, plus awareness that `finalize()` is deprecated.

**Ideal Answer:**

Three unrelated things that share a prefix:

**`final`** — a modifier:
- `final` variable: can be assigned once (constant reference). For objects, the *reference* is fixed, not the object's contents.
- `final` method: cannot be overridden.
- `final` class: cannot be extended (e.g., `String`, `Integer`).

```java
final int MAX = 100;            // can't reassign
final List<String> list = new ArrayList<>();
list.add("ok");                 // allowed — contents mutable
// list = new ArrayList<>();    // compile error — reference is final
```

**`finally`** — a block in `try/catch/finally` that *always* executes (for cleanup), whether or not an exception was thrown — except on `System.exit()` or JVM crash. With try-with-resources, you rarely need it.

```java
try {
    return doWork();
} finally {
    cleanup();                  // runs even if doWork throws or returns
}
```

**`finalize()`** — a method on `Object` the GC *used to* call before reclaiming an object. It's **deprecated since Java 9 and removed/deprecated-for-removal in later versions.** It's unreliable (no guarantee it ever runs, or when), hurts GC performance, and can resurrect objects. Use `try-with-resources` / `AutoCloseable` or `java.lang.ref.Cleaner` instead.

**Follow-up questions the interviewer will ask:**

1. *"Does `finally` run if there's a `return` in `try`?"* → Yes. The `return` value is computed, then `finally` runs, then the method returns. If `finally` itself has a `return`, it *overrides* the try's return (an anti-pattern — never `return` from `finally`).

2. *"Can `finally` swallow an exception?"* → Yes, if `finally` throws or returns, the original exception is lost. This is a real bug source — avoid logic in `finally` that can throw.

3. *"Replacement for finalize?"* → `AutoCloseable` + try-with-resources for deterministic cleanup; `java.lang.ref.Cleaner` for a safety-net for native resources.

**Common mistakes that get you rejected:**
- Confusing the three.
- Recommending `finalize()` for resource cleanup.
- Not knowing `finally` overriding a return loses exceptions.

---

### Q10: Pass-by-Value — Java Has No Pass-by-Reference
**Company:** Amazon, Microsoft, Oracle
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Is Java pass-by-value or pass-by-reference? Prove it.

**What interviewer is testing:**
A precise mental model. Many say "objects are passed by reference" — wrong. Java is *always* pass-by-value; for objects, the *value* passed is the reference (a copy of it).

**Ideal Answer:**

**Java is strictly pass-by-value.** For primitives, the value copied is the primitive. For objects, the value copied is the **reference** (the pointer), not the object. So you get a *copy of the reference* — both copies point to the same object, but reassigning the parameter doesn't affect the caller.

```java
void mutate(StringBuilder sb) { sb.append(" world"); } // affects caller's object
void reassign(StringBuilder sb) { sb = new StringBuilder("new"); } // does NOT

StringBuilder s = new StringBuilder("hello");
mutate(s);
System.out.println(s);      // "hello world"  — we mutated the shared object

reassign(s);
System.out.println(s);      // "hello world"  — unchanged; reassign only rebound the copy
```

The proof is `reassign`: if Java were pass-by-reference, reassigning the parameter would change the caller's variable. It doesn't — because the parameter is a *copy* of the reference. Mutating *through* the reference works because both copies point at the same object; *rebinding* the copy is invisible to the caller.

A swap that doesn't swap clinches it:

```java
void swap(Integer a, Integer b) { Integer t = a; a = b; b = t; } // no effect on caller
```

**Follow-up questions the interviewer will ask:**

1. *"So how do I 'return' two values or mutate a caller's variable?"* → Return an object/record/array, use a holder/wrapper, or restructure. Java gives no out-parameters.

2. *"Why does this confuse people?"* → Because mutation *through* a reference looks like pass-by-reference. The distinction is reassignment vs. mutation: pass-by-value means reassignment of the parameter is local; mutation of the pointed-to object is shared.

**Common mistakes that get you rejected:**
- Saying "primitives by value, objects by reference" — both are by value; objects pass a copy of the reference.

---

### Q11: Static vs Instance — Initialization Order
**Company:** Oracle, Amazon, Adobe
**Difficulty:** 🟡 Medium
**Frequency:** 🔥
**Round:** Phone

**Question:**
What's the exact order of static blocks, instance blocks, constructors, and field initializers when you create an object — including with inheritance?

**What interviewer is testing:**
Precise knowledge of class loading and initialization order. Comes up as a "predict the output" puzzle.

**Ideal Answer:**

Two phases. **Static initialization happens once, when the class is first loaded/initialized.** **Instance initialization happens on every `new`.**

Order with inheritance (`new Child()`):

1. **Static** (once, parent before child, in textual order within each):
   - Parent static fields + static blocks
   - Child static fields + static blocks
2. **Instance** (every construction):
   - Parent instance fields + instance blocks (in order)
   - Parent constructor body
   - Child instance fields + instance blocks (in order)
   - Child constructor body

```java
class Parent {
    static { System.out.println("1 parent static block"); }
    { System.out.println("3 parent instance block"); }
    Parent() { System.out.println("4 parent ctor"); }
}
class Child extends Parent {
    static { System.out.println("2 child static block"); }
    { System.out.println("5 child instance block"); }
    Child() { System.out.println("6 child ctor"); }
}
// new Child(); prints 1,2,3,4,5,6
// a second new Child(); prints only 3,4,5,6 (statics already ran)
```

The key insight: `super()` runs first inside the child constructor (implicit if not written), which is why the parent's instance init and ctor (3,4) finish before the child's instance init and ctor (5,6).

**Follow-up questions the interviewer will ask:**

1. *"When exactly is a class initialized?"* → On first *active use*: first instance creation, first access to a static field (not a compile-time constant) or static method, or reflection. Not merely on classloading — loading and initialization are distinct phases (load → link → initialize).

2. *"Static field that's a compile-time constant — does accessing it trigger init?"* → No. `static final int X = 5;` is inlined at compile time, so reading it doesn't initialize the class.

**Common mistakes that get you rejected:**
- Putting the child constructor before the parent instance block.
- Thinking static blocks run on every `new`.

---

### Q12: Comparable vs Comparator
**Company:** Amazon, Microsoft, Google, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Difference between `Comparable` and `Comparator`? When do you use each?

**What interviewer is testing:**
Practical sorting knowledge and modern `Comparator` composition (Java 8+).

**Ideal Answer:**

- **`Comparable<T>`** — the type's *natural ordering*. Implemented by the class itself via `compareTo(T)`. One per class. Used by `Collections.sort(list)`, `TreeMap`/`TreeSet` by default, `Arrays.sort`.
- **`Comparator<T>`** — an *external*, swappable ordering. A separate object; you can have many. Passed to `sort(list, comparator)`.

```java
record Employee(String name, int age, double salary) implements Comparable<Employee> {
    // natural order: by name
    public int compareTo(Employee o) { return this.name.compareTo(o.name); }
}

List<Employee> list = new ArrayList<>(/* ... */);

Collections.sort(list);                                  // natural order (by name)

// external orderings via Comparator, composed fluently (Java 8+)
list.sort(Comparator.comparingInt(Employee::age));       // by age asc
list.sort(Comparator.comparingDouble(Employee::salary).reversed()); // salary desc
list.sort(Comparator.comparing(Employee::age)
                    .thenComparing(Employee::name));     // age, then name tie-break
list.sort(Comparator.comparing(Employee::name,
                    Comparator.nullsFirst(Comparator.naturalOrder()))); // null-safe
```

**Contract:** `compareTo`/`compare` must return negative/zero/positive and be consistent (transitive, anti-symmetric). It *should* be consistent with `equals` (i.e., `compareTo == 0` iff `equals`) — `TreeMap`/`TreeSet` use `compareTo`, not `equals`, so an inconsistent comparator makes a "duplicate" key silently replace another.

**Follow-up questions the interviewer will ask:**

1. *"Why not `return a - b` for int comparison?"* → **Integer overflow.** If `a = Integer.MAX_VALUE` and `b = -1`, `a - b` overflows to a negative number, giving the wrong sign. Use `Integer.compare(a, b)`.

2. *"TreeSet uses equals or compareTo for uniqueness?"* → `compareTo`/`compare`. If your comparator says two distinct objects are "equal" (returns 0), the set treats them as duplicates. This is a subtle bug when the comparator disagrees with `equals`.

3. *"Sort stability?"* → `Collections.sort`/`Arrays.sort` (for objects) is **stable** (TimSort) — equal elements keep their relative order. Primitive `Arrays.sort` uses dual-pivot quicksort (not stable, but primitives have no identity so it doesn't matter).

**Common mistakes that get you rejected:**
- `a - b` comparators (overflow).
- Not knowing `TreeSet`/`TreeMap` use `compareTo`, not `equals`.
- Manually writing comparators when `Comparator.comparing(...).thenComparing(...)` is cleaner.

---

### Q13: Checked vs Unchecked Exceptions
**Company:** Amazon, Microsoft, Google, Oracle
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
Difference between checked and unchecked exceptions? When should you use each?

**What interviewer is testing:**
The exception hierarchy and *design judgment* — when a failure should be in the method signature vs not.

**Ideal Answer:**

The hierarchy:

```
Throwable
├── Error              (unchecked — JVM problems: OutOfMemoryError, StackOverflowError)
└── Exception
    ├── RuntimeException (unchecked — NPE, IllegalArgument, IndexOutOfBounds)
    └── (other Exception subclasses) (checked — IOException, SQLException)
```

- **Checked** exceptions (`Exception` minus `RuntimeException`) **must** be declared in `throws` or caught — the *compiler* enforces handling. Intent: *recoverable* conditions the caller should consciously handle (file not found, network failure).
- **Unchecked** exceptions (`RuntimeException` and `Error`) need no declaration. Intent: *programming bugs* (`NullPointerException`, `IllegalArgumentException`) that you usually can not recover from and should fix in code, plus catastrophic `Error`s.

```java
// Checked: caller is forced to deal with it
public byte[] readFile(Path p) throws IOException {
    return Files.readAllBytes(p);   // IOException is checked
}

// Unchecked: a bug / precondition violation
public int divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("b must be non-zero"); // unchecked
    return a / b;
}
```

**Design guideline:** use checked for conditions a *well-written* caller can reasonably recover from; use unchecked for precondition violations and bugs. In practice many modern codebases (and Spring) lean heavily on unchecked exceptions because checked exceptions do not compose with lambdas/streams and lead to `catch (Exception e) {}` swallowing.

**Follow-up questions the interviewer will ask:**

1. *"Why does Spring wrap `SQLException` (checked) into `DataAccessException` (unchecked)?"* → To free callers from boilerplate `try/catch` on a low-level checked exception they usually can not recover from, and to provide a *consistent, vendor-agnostic* hierarchy across data stores. It is a deliberate "checked exceptions hurt here" decision.

2. *"Can a lambda throw a checked exception?"* → Not into a standard functional interface like `Function` (its signature does not declare `throws`). You must catch-and-wrap inside the lambda (throw an unchecked wrapper) or use a custom `@FunctionalInterface` that declares the checked exception. This is a major reason modern Java favors unchecked exceptions.

3. *"Is catching `Throwable` ever ok?"* → Almost never in app code — it catches `Error`s like `OutOfMemoryError` you can not sensibly handle. Catch the most specific exception you can act on.

**Common mistakes that get you rejected:**
- Saying `Error` is checked (it is unchecked).
- Catching and swallowing checked exceptions with empty blocks.
- Declaring `throws Exception` everywhere (defeats the purpose).

---

### Q14: Custom Exceptions — When and How
**Company:** Amazon, Microsoft, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
How and when do you create a custom exception? Show one done well.

**What interviewer is testing:**
Whether you can design a meaningful exception (preserve cause, carry context) rather than throwing raw `RuntimeException("error")`.

**Ideal Answer:**

Create a custom exception when you have a *domain-specific* failure that callers may want to catch distinctly, and you want to carry structured context. Decide checked vs unchecked by whether callers can recover — domain exceptions in web apps are usually **unchecked** (cleaner with Spring `@ControllerAdvice`).

```java
public class InsufficientFundsException extends RuntimeException {
    private final String accountId;
    private final long shortfallCents;

    public InsufficientFundsException(String accountId, long shortfallCents) {
        super("Account %s short by %d cents".formatted(accountId, shortfallCents));
        this.accountId = accountId;
        this.shortfallCents = shortfallCents;
    }
    // overload that preserves the underlying cause
    public InsufficientFundsException(String accountId, long shortfallCents, Throwable cause) {
        super("Account %s short by %d cents".formatted(accountId, shortfallCents), cause);
        this.accountId = accountId;
        this.shortfallCents = shortfallCents;
    }
    public String getAccountId() { return accountId; }
    public long getShortfallCents() { return shortfallCents; }
}
```

Key practices:
- **Always offer a `(message, Throwable cause)` constructor** so you never lose the stack trace of the original failure (`throw new XException("context", e)`).
- **Carry structured fields** (`accountId`, `shortfallCents`) so handlers and logs have machine-readable context, not just a string.
- **Name it for the problem**, not the layer (`InsufficientFundsException`, not `ServiceException`).

**Follow-up questions the interviewer will ask:**

1. *"Exception chaining — why pass the cause?"* → So the root cause stack trace is preserved in the chain (`Caused by:` in the log). Swallowing the cause (`throw new XException(e.getMessage())`) discards where it actually broke — a top reason production incidents are hard to debug.

2. *"Should custom exceptions be checked or unchecked?"* → Default to unchecked for domain/business errors in modern apps; reserve checked for genuinely recoverable, expected conditions where you want the compiler to force handling.

**Common mistakes that get you rejected:**
- Dropping the cause (`new RuntimeException(e.getMessage())`), losing the stack trace.
- Generic exceptions named after layers (`DaoException`) carrying no context.
- Extending `Exception` (checked) for things callers can not recover from.

---

### Q15: try-with-resources and `AutoCloseable`
**Company:** Amazon, Oracle, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
What problem does try-with-resources solve? How does it differ from a `finally` block? What is suppressed exception handling?

**What interviewer is testing:**
Resource-leak awareness and a subtle correctness issue (`finally` can mask the real exception).

**Ideal Answer:**

Before Java 7, closing resources meant a `finally` block — verbose and error-prone, and the close call in `finally` could **mask** the real exception:

```java
// OLD — broken masking behavior
InputStream in = null;
try {
    in = new FileInputStream(path);
    read(in);                 // throws IOException A
} finally {
    in.close();               // throws IOException B -> A is LOST, caller sees B
}
```

Try-with-resources fixes this. Any resource implementing `AutoCloseable` declared in the parentheses is **automatically closed** in reverse order of declaration, *and* if both the body and `close()` throw, the body exception wins and the close exception becomes a **suppressed** exception attached to it:

```java
try (InputStream in = new FileInputStream(path);
     OutputStream out = new FileOutputStream(dest)) {   // closed in reverse: out, then in
    in.transferTo(out);
}   // no finally needed; both closed automatically, even on exception
```

If `transferTo` throws A and `in.close()` throws B, the caller gets A with B accessible via `A.getSuppressed()`. The *primary* failure is preserved — the opposite of the old masking bug.

Custom resources just implement `AutoCloseable`:

```java
class Connection implements AutoCloseable {
    public void close() { /* release */ }
}
try (Connection c = new Connection()) { /* use c */ }   // c.close() guaranteed
```

**Follow-up questions the interviewer will ask:**

1. *"`AutoCloseable` vs `Closeable`?"* → `Closeable` (java.io, pre-7) extends `AutoCloseable`; its `close()` throws only `IOException` and should be idempotent. `AutoCloseable.close()` can throw any `Exception`. Try-with-resources works with either.

2. *"Can you use an existing variable in try-with-resources?"* → Since Java 9, yes — if it is effectively final: `try (resource) { }` where `resource` was declared earlier.

3. *"What are suppressed exceptions?"* → When the body throws and a `close()` also throws, the close exception is added to the body exception via `addSuppressed`, retrievable with `getSuppressed()`. The primary exception propagates.

**Common mistakes that get you rejected:**
- Still hand-rolling `finally { x.close(); }` and not knowing about masking.
- Not knowing close order is reverse of declaration.

---

### Q16: Exception Best Practices That Get Noticed
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite / HM

**Question:**
What are your exception-handling best practices in production code?

**What interviewer is testing:**
Production maturity — often an HM-round signal of whether you have operated real systems.

**Ideal Answer:**

The practices that distinguish production-grade code:

1. **Catch specific, not `Exception`/`Throwable`.** Catch only what you can meaningfully handle. Broad catches hide bugs.
2. **Never swallow.** An empty `catch` block is a silent failure. At minimum log with context; usually rethrow or wrap.
3. **Preserve the cause.** `throw new ServiceException("charging card failed for order " + id, e)` — never drop `e`.
4. **Do not use exceptions for control flow.** They are expensive (stack trace capture) and obscure logic.
5. **Fail fast on preconditions.** Validate arguments at method entry (`Objects.requireNonNull`, guard clauses) and throw `IllegalArgumentException`/`IllegalStateException` immediately.
6. **Log OR throw, not both.** Logging and rethrowing the same exception produces duplicate noise. Log at the boundary where you handle it.
7. **Translate layers.** Do not leak `SQLException` to the controller; wrap into a domain/`DataAccessException`.
8. **Clean up with try-with-resources**, not `finally`.

```java
public Receipt charge(OrderId id, Card card) {
    Objects.requireNonNull(card, "card");                 // 5. fail fast
    try {
        return gateway.charge(amountFor(id), card);
    } catch (GatewayTimeoutException e) {                 // 1. specific
        throw new PaymentFailedException("timeout charging order " + id, e); // 3 + 7
    }
    // other gateway exceptions propagate to be handled where it makes sense
}
```

**Follow-up questions the interviewer will ask:**

1. *"How expensive is throwing an exception?"* → The cost is dominated by **filling in the stack trace** (`fillInStackTrace`, walking the stack). You can override it (`super(msg, cause, false, false)`) to skip the trace for high-frequency exceptions — but that is an optimization for a measured hotspot, not a default.

2. *"Where should you handle exceptions in a web app?"* → Centrally — a `@ControllerAdvice` / `@ExceptionHandler` maps exceptions to HTTP responses (Q100). Service code throws meaningful exceptions; the boundary translates them.

**Common mistakes that get you rejected:**
- Empty catch blocks; catching `Exception` broadly.
- Log-and-rethrow duplication.
- Using exceptions for normal flow.

---

# GENERICS

---

### Q17: Generics and Type Erasure
**Company:** Amazon, Google, Microsoft, Oracle
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
What is type erasure? What are its consequences?

**What interviewer is testing:**
Understanding that Java generics are a *compile-time* feature — the runtime knows nothing about type parameters. This explains a whole family of "why can not I..." restrictions.

**Ideal Answer:**

**Type erasure**: the compiler uses generic type information for *compile-time type checking*, then **erases** the type parameters in the bytecode. `List<String>` and `List<Integer>` are both just `List` at runtime. Type variables are replaced with their bounds (or `Object` if unbounded), and the compiler inserts **casts** where needed.

```java
// What you write:
List<String> list = new ArrayList<>();
list.add("hi");
String s = list.get(0);

// What the bytecode effectively is (after erasure):
List list = new ArrayList();
list.add("hi");
String s = (String) list.get(0);   // compiler-inserted cast
```

Why erasure? **Backward compatibility** — generics arrived in Java 5, and erasure let generic code interoperate with pre-generic (raw) code and run on the existing JVM.

Consequences (all flow from "no type info at runtime"):

1. **No runtime type check of type args.** `if (obj instanceof List<String>)` is illegal — only `List<?>` or raw `List`.
2. **Can not create generic arrays.** `new T[10]` is illegal (Q19).
3. **Can not use primitives as type args.** `List<int>` is illegal — only reference types.
4. **No two overloads differing only by type parameter.** Same erased signature → will not compile.
5. **Can not `new T()`** — pass a `Supplier<T>` or `Class<T>`.
6. **Static fields are shared across all parameterizations.**

```java
public <T> T create(Class<T> type) throws Exception {
    return type.getDeclaredConstructor().newInstance();   // workaround for "no new T()"
}
```

**Follow-up questions the interviewer will ask:**

1. *"How do frameworks like Jackson/Spring get the generic type at runtime if it is erased?"* → Erasure removes type *variables*, but generic type info on **fields, method signatures, and superclasses** is retained in class metadata and readable via reflection (`getGenericType`, `ParameterizedType`). The "super type token" trick (`new TypeReference<List<User>>() {}`) captures it by subclassing an anonymous class whose generic superclass *is* recorded.

2. *"What is a bridge method?"* → A synthetic method the compiler generates to preserve polymorphism after erasure. E.g., `Comparable<T>.compareTo(T)` erases to `compareTo(Object)`; the compiler adds a bridge `compareTo(Object)` that casts and calls your `compareTo(MyType)`.

3. *"Reifiable vs non-reifiable types?"* → Reifiable types have full type info at runtime (`String`, `int[]`, `List`, `List<?>`). Non-reifiable types lose info (`List<String>`). Varargs of non-reifiable types trigger `@SafeVarargs`/unchecked warnings (heap pollution risk).

**Common mistakes that get you rejected:**
- Thinking generics exist at runtime (like C# reified generics).
- Trying `instanceof List<String>` or `new T[]`.
- Not knowing the `Class<T>` / super-type-token workarounds.

---

### Q18: Wildcards — `? extends` vs `? super` and PECS
**Company:** Amazon, Google, Microsoft, Goldman Sachs
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Explain `? extends T` vs `? super T`. State the PECS principle and why it holds.

**What interviewer is testing:**
Deep generics — variance. This separates SDE-2 from SDE-1. Most candidates can not explain *why* you can read from `? extends` but not write to it.

**Ideal Answer:**

Generics are **invariant**: `List<Integer>` is **not** a `List<Number>`, even though `Integer` is a `Number`. This is for type safety — if it were allowed, you could add a `Double` to a `List<Integer>` through a `List<Number>` reference. Wildcards reintroduce controlled variance.

**`? extends T` — upper bounded (covariance, a PRODUCER):**
"some unknown type that *is* a T or subtype." You can **read** elements as `T`, but you **cannot add** anything (except `null`), because the compiler does not know the *exact* subtype the list holds.

```java
List<? extends Number> nums = new ArrayList<Integer>();
Number n = nums.get(0);     // OK — whatever it holds, it is a Number
// nums.add(1);             // COMPILE ERROR — could be List<Double>, can not safely add Integer
```

**`? super T` — lower bounded (contravariance, a CONSUMER):**
"some unknown type that *is* a T or supertype." You can **add** a `T` (or subtype), but reading gives only `Object`, because the actual type could be any supertype.

```java
List<? super Integer> sink = new ArrayList<Number>();
sink.add(1);                // OK — Integer fits any super-of-Integer list
sink.add(Integer.valueOf(2));
// Integer x = sink.get(0); // COMPILE ERROR — could be List<Object>; only Object is safe
Object o = sink.get(0);     // OK
```

**PECS — Producer Extends, Consumer Super:**
- If a parameter *produces* values you read out → use `? extends T`.
- If a parameter *consumes* values you put in → use `? super T`.

The canonical example is `Collections.copy`:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (T item : src)        // src PRODUCES T  -> extends
        dest.add(item);       // dest CONSUMES T -> super
}
```

`src` (`? extends T`) is the producer we read from; `dest` (`? super T`) is the consumer we write to. This lets `copy(numberList, integerList)` typecheck safely.

**Why the asymmetry?** With `? extends Number`, the list could be `List<Double>` — adding an `Integer` would corrupt it, so writes are banned; but every element *is* a `Number`, so reads as `Number` are safe. With `? super Integer`, the list could be `List<Object>` — reads give only `Object`, but any `Integer` fits, so writes are safe.

**Follow-up questions the interviewer will ask:**

1. *"What can you add to a `List<? extends Number>`?"* → Only `null` (assignable to anything). Nothing else is provably safe.

2. *"Unbounded `List<?>` vs `List<Object>`?"* → `List<?>` is "list of some unknown type" — you can pass a `List<String>` to it, but can not add (except null). `List<Object>` is a specific list of `Object`s — you *can* add anything, but you can only pass a `List<Object>` to it. Use `List<?>` for read-only/agnostic parameters.

3. *"Where would you use `? super` in real APIs?"* → `Collections.addAll(Collection<? super T>, T...)`, `Comparator` composition, `Stream.collect` suppliers/accumulators, callback sinks.

**Common mistakes that get you rejected:**
- Thinking generics are covariant like arrays.
- Trying to add to a `? extends` collection.
- Reciting "PECS" without explaining the read/write asymmetry.

---

### Q19: Why You Can Not Create a Generic Array
**Company:** Google, Amazon, Oracle
**Difficulty:** 🔴 Hard
**Frequency:** 🔥
**Round:** Onsite

**Question:**
Why is `new T[10]` illegal? Arrays and generics do not mix — explain.

**What interviewer is testing:**
The clash between **reified** arrays (runtime-type-aware) and **erased** generics.

**Ideal Answer:**

Two facts collide:

1. **Arrays are covariant and reified.** `Integer[]` *is-a* `Object[]`, and an array *remembers its component type at runtime*, performing an `ArrayStoreException` check on every write.
2. **Generics are invariant and erased.** `List<Integer>` is not `List<Object>`, and the type parameter is *gone* at runtime.

If `new T[10]` were allowed, erasure would make it `new Object[10]`, and the runtime store-check would use the *erased* type, letting the wrong type slip in undetected. The covariant reified array store-check and erased generics are fundamentally incompatible, so the language bans generic array creation.

```java
@SuppressWarnings("unchecked")
T[] arr = (T[]) new Object[10];   // common workaround — unsafe to RETURN as T[]
```

This compiles with a warning but is only safe if `arr` never escapes as a real `T[]`. The safe approach uses a `Class<T>` token and reflection:

```java
@SuppressWarnings("unchecked")
public static <T> T[] newArray(Class<T> type, int size) {
    return (T[]) java.lang.reflect.Array.newInstance(type, size);  // correct component type
}
```

This is exactly what `ArrayList.toArray(T[])` and `Arrays.copyOf` do.

**Follow-up questions the interviewer will ask:**

1. *"How does `ArrayList` store elements internally then?"* → It uses an `Object[] elementData` and casts on access. Internally it is a raw `Object[]`; the generic type is enforced only at the API boundary by the compiler.

2. *"What is heap pollution?"* → A variable of a parameterized type referring to an object not of that type (e.g., a `List<String>` actually containing an `Integer`), possible via unchecked casts or non-reifiable varargs. It defers the `ClassCastException` to a later, confusing point.

**Common mistakes that get you rejected:**
- Not connecting it to covariance + reification of arrays vs erasure of generics.
- Returning the `(T[]) new Object[]` cast as a public `T[]`.

---

# COLLECTIONS INTERNALS

Collections questions are where interviewers find out if you have actually read the source. "What's the difference between ArrayList and LinkedList" is a softball; "walk me through what happens when a HashMap resizes during a put under high collision" is the real test.

---

### Q20: HashMap Internals — Buckets, Treeification, Resize, Load Factor
**Company:** Amazon, Google, Microsoft, Meta, Flipkart, Goldman Sachs
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
Explain how HashMap works internally in Java 8+. Cover the bucket array, hashing, collision handling, treeification, load factor, and resizing.

**What interviewer is testing:**
The single most-asked Java internals question. They want the *full* mechanism: the table of bins, the bit-spreading hash, collision lists turning into trees at 8, the 0.75 load factor, and doubling resize with the clever rehash split.

**Ideal Answer:**

A `HashMap` is an array of **bins** (buckets), `Node<K,V>[] table`, where each bin holds either a **linked list** or (when large) a **red-black tree** of entries that hash to that index.

**1. Hashing.** HashMap does not use `key.hashCode()` directly. It *spreads* the high bits into the low bits:

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

Why? The bin index is `(n - 1) & hash` and `n` (table length) is always a power of two, so only the **low** bits of the hash pick the bin. XOR-ing the top 16 bits down mixes high-order information in, reducing collisions for keys whose hashes differ mainly in high bits.

**2. Bin index.** `index = (n - 1) & hash`. Because `n` is a power of two, `(n-1)` is a mask of low bits — equivalent to `hash % n` but faster.

**3. Insertion / collision.** On `put`:
- Compute index. If the bin is empty, place the node.
- If occupied (collision), walk the bin: if a node with an equal key (`hash ==` then `equals`) exists, replace its value; else append.
- Java 8 appends to the **tail** of the list (Java 7 used head-insertion, which caused the infamous resize infinite loop in concurrent misuse).

**4. Treeification.** When a single bin's list length reaches **`TREEIFY_THRESHOLD = 8`** *and* the table capacity is at least **`MIN_TREEIFY_CAPACITY = 64`**, that bin converts from a linked list to a **red-black tree**, making worst-case bin operations O(log n) instead of O(n). If capacity < 64, it resizes instead of treeifying. A bin reverts to a list when it shrinks to **`UNTREEIFY_THRESHOLD = 6`** (hysteresis avoids thrashing around the boundary).

**5. Load factor and resize.** Default initial capacity 16, **load factor 0.75**. The `threshold = capacity * loadFactor` (12 for default). When `size` exceeds threshold, the table **doubles** (16 → 32 → 64 ...). 0.75 balances space vs collision probability — higher means fuller bins (more collisions), lower means wasted space.

During resize, each bin splits into two: because capacity doubles, an entry either stays at index `i` or moves to `i + oldCapacity`, decided by a single bit (`hash & oldCapacity`). Java 8 preserves relative order during the split (no rehash of each key needed):

```text
old table (n=16), bin 5:  entries with hash&16 == 0 stay at bin 5
                          entries with hash&16 != 0 move to bin 5+16 = 21
```

**Worst-case complexity:** O(1) average get/put; O(log n) within a treeified bin; O(n) only if all keys collide AND are not Comparable (so cannot tree).

**Follow-up questions the interviewer will ask:**

1. *"Why power-of-two capacity?"* → It makes `(n-1) & hash` a cheap bitmask equal to modulo, and makes the resize split a single-bit decision (`hash & oldCap`). A non-power-of-two would force real modulo and full rehash.

2. *"Why treeify at 8 specifically?"* → With a good hash and load factor 0.75, the probability of a bin reaching 8 entries follows a Poisson distribution and is about 1 in 10 million — so treeification is a rare safety net against pathological/adversarial hashes, not a common path. The list is faster for tiny bins, so 8 is the crossover.

3. *"Is HashMap thread-safe? What breaks?"* → No. Concurrent `put` during resize can lose updates or (in Java 7) create a circular linked list causing an infinite loop in `get` (100% CPU). Java 8 fixed the infinite loop but it is still not safe — use `ConcurrentHashMap`.

4. *"What makes a good key?"* → Immutable, with consistent `hashCode`/`equals`, and a well-distributed `hashCode`. Mutable keys whose hash changes after insertion become unreachable.

**Common mistakes that get you rejected:**
- Saying collisions always make a linked list (Java 8 trees them at 8).
- Forgetting the 64-capacity precondition for treeification.
- Not knowing the bit-spreading hash or the doubling resize split.
- Claiming HashMap is thread-safe.

---

### Q21: ConcurrentHashMap Internals — CAS + Bin Locking
**Company:** Amazon, Google, Microsoft, Meta, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
How does ConcurrentHashMap achieve thread safety in Java 8+? How is it different from the old segment-based design and from `Hashtable`?

**What interviewer is testing:**
Whether you understand fine-grained concurrency: CAS for empty-bin insert, synchronized on the bin head for collisions, no global lock, and that segments are gone since Java 8.

**Ideal Answer:**

`ConcurrentHashMap` (CHM) gives thread safety with *high concurrency* by locking at the **bin (bucket) level**, not the whole map.

**Pre-Java-8 (segmented):** the map was divided into `Segment`s (default 16), each a mini-Hashtable with its own lock. Concurrency was limited to the number of segments — at most 16 writers in parallel. This is the old design candidates often describe; *mention it is gone.*

**Java 8+ (CAS + synchronized bins):** segments were removed. The structure mirrors HashMap (a `Node[] table`, treeification at 8). Synchronization is per-bin:

1. **Empty bin insert via CAS.** If the target bin is `null`, insert the new node with a `compareAndSwap` (lock-free). If it succeeds, done — no lock taken.

```java
// conceptual, from CHM.putVal
if ((f = tabAt(tab, i)) == null) {
    if (casTabAt(tab, i, null, new Node<>(hash, key, value)))
        break;   // CAS succeeded, no lock needed
}
```

2. **Non-empty bin: synchronize on the bin head node.** When the bin already has entries (collision), CHM does `synchronized (f)` where `f` is the **first node of that bin**. So two threads writing to *different* bins never contend; only threads hitting the *same* bin serialize.

```java
synchronized (f) {            // lock just this bin's head, not the whole map
    // walk the list/tree, insert or update
}
```

3. **`size()` uses a striped counter** (`baseCount` + `CounterCell[]`, the LongAdder pattern) to avoid a single hot contended counter; it is an estimate under concurrent mutation.

4. **No null keys or values** — because a `null` return from `get` would be ambiguous (absent vs mapped-to-null) under concurrency, and you can not atomically check-then-act on it.

5. **Concurrent resize is cooperative**: multiple threads help transfer bins (`ForwardingNode` marks moved bins) so resizing does not block all writers.

**Atomic compound operations:** CHM provides `compute`, `computeIfAbsent`, `merge`, `putIfAbsent` that are atomic *per key*, which is the right way to do read-modify-write:

```java
ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();
counts.merge("a", 1, Integer::sum);                 // atomic increment
counts.computeIfAbsent("key", k -> expensiveLoad(k)); // atomic, computed once
```

**Follow-up questions the interviewer will ask:**

1. *"`Hashtable` vs `ConcurrentHashMap`?"* → `Hashtable` synchronizes *every method on the whole object* — one lock, no concurrency, a bottleneck. CHM locks per bin and uses CAS, allowing many concurrent writers. `Hashtable` is legacy; never choose it for new code.

2. *"Is `get()` locked?"* → No. Reads are lock-free. Nodes use `volatile val` and `volatile next`, and the table reference is volatile, so reads see consistent, up-to-date values via the memory model without locking.

3. *"`size()` exact?"* → Not necessarily under concurrent modification — it sums striped counters and is a snapshot estimate. For strict counts you would need external synchronization, which defeats the purpose.

4. *"Why is `computeIfAbsent` better than `get`-then-`put`?"* → `get`-then-`put` is a non-atomic check-then-act: two threads can both see absent and both compute/put, doing duplicate work or overwriting. `computeIfAbsent` does it atomically under the bin lock, computing exactly once.

**Common mistakes that get you rejected:**
- Describing segments as the *current* design (removed in Java 8).
- Saying the whole map is locked on write (it is per-bin).
- Allowing null keys/values.
- Using `get`+`put` for read-modify-write instead of `compute`/`merge`.

---

### Q22: ArrayList vs LinkedList
**Company:** Amazon, Microsoft, Infosys, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥🔥
**Round:** Phone

**Question:**
ArrayList vs LinkedList — internals, complexity, and when to use each.

**What interviewer is testing:**
Whether you know the real-world answer (ArrayList almost always wins) and *why*, not just the textbook Big-O table.

**Ideal Answer:**

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(i)` random access | **O(1)** | O(n) (walk from nearest end) |
| `add` at end (amortized) | **O(1)** | O(1) |
| `add`/`remove` at index | O(n) (shift) | O(n) to *find*, O(1) to *relink* |
| `add`/`remove` at head | O(n) | **O(1)** |
| Memory | compact array | node objects (next/prev/data) — high overhead |
| Cache locality | excellent (contiguous) | poor (scattered nodes) |

**ArrayList** is a growable `Object[]`. On overflow it grows by ~1.5x (`oldCapacity + (oldCapacity >> 1)`) and copies. Random access is O(1) (index arithmetic). Insert/remove in the middle shifts elements — O(n).

**LinkedList** is a doubly-linked list of nodes, also a `Deque`. O(1) insert/remove *given a node reference*, but you almost never have that — you have an index, and reaching it is O(n).

**The real-world verdict:** prefer **ArrayList** almost always. Even for queue/stack patterns, `ArrayDeque` beats `LinkedList`. LinkedList theoretically wins for frequent head insertions or true list-splicing, but its per-node object overhead and terrible cache locality usually make ArrayList faster *even for operations where LinkedList has better Big-O*. Joshua Bloch (its author) has said he basically never uses it.

**Follow-up questions the interviewer will ask:**

1. *"You iterate and remove many elements from the middle — which is faster?"* → Surprisingly often ArrayList still wins on cache effects, but the clean answer is: use `Iterator.remove()`, or batch with `removeIf`. For LinkedList, removing during a `ListIterator` traversal is O(1) per remove; for ArrayList it is O(n) per remove (shift). If removals are truly frequent and positional, LinkedList *can* win — but measure.

2. *"How does ArrayList grow?"* → ~1.5x (`>> 1`). Pre-size with `new ArrayList<>(expected)` to avoid repeated array copies. (HashMap doubles; ArrayList is 1.5x.)

3. *"What should I use for a stack/queue?"* → `ArrayDeque` — faster than `Stack` (legacy, synchronized) and `LinkedList` for both.

**Common mistakes that get you rejected:**
- Recommending LinkedList for "frequent inserts" without qualifying that you rarely have the node reference.
- Citing only Big-O and ignoring cache locality / object overhead.
- Using legacy `Stack`/`Vector`.

---

### Q23: TreeMap and the Red-Black Tree
**Company:** Amazon, Google, Bloomberg
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
How is TreeMap implemented? When would you use it over HashMap?

**What interviewer is testing:**
Sorted-map use cases and awareness that TreeMap is a self-balancing BST giving O(log n) ordered operations and range queries.

**Ideal Answer:**

`TreeMap` is a **red-black tree** — a self-balancing binary search tree. It implements `NavigableMap`/`SortedMap`, keeping keys in **sorted order** (natural ordering or a supplied `Comparator`). All core operations (`get`, `put`, `remove`) are **O(log n)** — slower than HashMap's average O(1), but you gain ordering.

Red-black invariants keep the tree balanced (height O(log n)): every node is red or black, the root is black, red nodes have black children, and every root-to-leaf path has the same number of black nodes. Insertions/deletions re-balance via rotations and recoloring.

Use TreeMap when you need:
- **Sorted iteration** by key.
- **Range queries / navigation:** `firstKey`, `lastKey`, `ceilingKey`, `floorKey`, `higherKey`, `lowerKey`, `headMap`, `tailMap`, `subMap`.

```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(10, "a"); tm.put(20, "b"); tm.put(30, "c");

tm.ceilingKey(15);   // 20  (smallest key >= 15)
tm.floorKey(25);     // 20  (largest key <= 25)
tm.firstKey();       // 10
tm.headMap(20);      // {10=a}  (keys < 20)
tm.subMap(10, 30);   // {10=a, 20=b}
```

These navigation methods are why TreeMap shows up in interval/scheduling and "find nearest" problems.

**Follow-up questions the interviewer will ask:**

1. *"TreeMap uses equals or compareTo for key identity?"* → `compareTo`/`compare`. Two keys are "the same" if the comparator returns 0 — *even if `equals` disagrees*. A comparator inconsistent with `equals` causes surprising behavior (a key not "found" though `equals` would match).

2. *"Null keys in TreeMap?"* → Not allowed (with natural ordering) — comparing null throws NPE. HashMap allows one null key. A custom null-tolerant comparator can permit it.

3. *"Red-black vs AVL?"* → AVL is more strictly balanced (faster lookups) but rebalances more on writes; red-black has cheaper inserts/deletes (fewer rotations), better for write-heavy maps — which is why the JDK uses red-black for TreeMap and treeified HashMap bins.

**Common mistakes that get you rejected:**
- Saying it is a hash structure.
- Forgetting it uses `compareTo`, not `equals`/`hashCode`.
- Not knowing the navigation methods that make it valuable.

---

### Q24: Fail-Fast vs Fail-Safe Iterators
**Company:** Amazon, Microsoft, Oracle
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
What is the difference between fail-fast and fail-safe iterators? Show how ConcurrentModificationException happens.

**What interviewer is testing:**
The `modCount` mechanism and how to mutate a collection safely during iteration.

**Ideal Answer:**

**Fail-fast** iterators (`ArrayList`, `HashMap`, `HashSet`, etc.) throw `ConcurrentModificationException` if the collection is structurally modified *during* iteration by anything other than the iterator's own `remove`. They detect this with a **`modCount`** counter: the iterator snapshots `expectedModCount` at creation and checks `modCount == expectedModCount` on each `next()`. A structural change bumps `modCount`, the check fails, and it throws.

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
for (Integer x : list) {       // for-each uses the iterator
    if (x == 2) list.remove(x); // structural modification -> modCount changes
}                               // -> ConcurrentModificationException on next next()
```

It is "fail-fast" because it fails immediately and visibly rather than risking silent corruption. Note: it is best-effort, not guaranteed (especially across threads).

**The correct ways to remove during iteration:**

```java
// 1. Iterator.remove()
Iterator<Integer> it = list.iterator();
while (it.hasNext()) { if (it.next() == 2) it.remove(); }   // updates modCount in sync

// 2. removeIf (cleanest)
list.removeIf(x -> x == 2);
```

**Fail-safe** iterators (`CopyOnWriteArrayList`, `ConcurrentHashMap`, `ConcurrentSkipListMap`) do **not** throw. They iterate over a **snapshot** or tolerate concurrent changes:
- `CopyOnWriteArrayList` iterates a snapshot of the backing array taken at iterator creation — changes after that are invisible to the iterator.
- `ConcurrentHashMap`'s iterator is **weakly consistent**: it reflects some-but-not-necessarily-all concurrent changes and never throws.

```java
var chm = new ConcurrentHashMap<>(Map.of("a", 1, "b", 2));
for (var k : chm.keySet()) { chm.put("c", 3); }   // no exception (weakly consistent)
```

The trade-off: fail-safe iterators may not see the latest data (snapshot staleness) and `CopyOnWriteArrayList` copies the whole array on every write (expensive for write-heavy use; great for read-heavy, rarely-modified lists like listener registries).

**Follow-up questions the interviewer will ask:**

1. *"Does fail-fast guarantee detection?"* → No. It is best-effort — the doc literally says do not rely on it for correctness, only for bug detection. Across threads the `modCount` check can miss.

2. *"Is `for (x : list) list.remove(x)` always an exception?"* → Subtle: removing the *second-to-last* element can skip the `hasNext`/check and exit without throwing (a known gotcha), which is why you must not rely on it. Always use `Iterator.remove`/`removeIf`.

3. *"Cost of CopyOnWriteArrayList?"* → Every mutation copies the entire backing array — O(n) writes. Only suitable when reads vastly outnumber writes.

**Common mistakes that get you rejected:**
- Modifying a collection in a for-each loop.
- Thinking fail-fast is a guarantee.
- Not knowing `removeIf` / `Iterator.remove`.

---

### Q25: HashSet vs LinkedHashSet vs TreeSet
**Company:** Amazon, Infosys, Microsoft
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Compare HashSet, LinkedHashSet, and TreeSet.

**What interviewer is testing:**
Ordering guarantees and the backing data structures.

**Ideal Answer:**

| | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Backed by | HashMap | LinkedHashMap | TreeMap (red-black) |
| Ordering | none (hash order) | **insertion order** | **sorted order** |
| `add`/`contains` | O(1) avg | O(1) avg | O(log n) |
| Nulls | one null | one null | no null (natural order) |

- **HashSet** — fastest, no ordering. Backed by a `HashMap` whose values are a dummy constant.
- **LinkedHashSet** — HashSet + a doubly-linked list threading entries in **insertion order**, so iteration is predictable. Slightly more memory.
- **TreeSet** — sorted, `NavigableSet` (`first`, `last`, `ceiling`, `floor`, `headSet`, `subSet`). O(log n).

```java
Set<String> hs = new HashSet<>();        // no order
Set<String> lhs = new LinkedHashSet<>(); // insertion order
Set<String> ts = new TreeSet<>();        // sorted
```

Choose by need: no order + fastest → HashSet; preserve insertion order → LinkedHashSet; sorted/range → TreeSet.

**Follow-up questions the interviewer will ask:**

1. *"How does HashSet store elements with a HashMap?"* → Each element is a *key* in the backing map, mapped to a shared dummy `PRESENT` object. `add` returns whether the key was new.

2. *"LinkedHashMap access-order mode?"* → `new LinkedHashMap<>(cap, lf, true)` orders by *access*, not insertion — the basis of an LRU cache (override `removeEldestEntry`).

**Common mistakes that get you rejected:**
- Claiming HashSet preserves insertion order (it does not).
- Putting nulls in a TreeSet with natural ordering.

---

### Q26: Hashtable vs HashMap vs ConcurrentHashMap
**Company:** Amazon, Microsoft, Oracle
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Compare Hashtable, HashMap, and ConcurrentHashMap.

**What interviewer is testing:**
Knowing the legacy/modern split and the right concurrent choice.

**Ideal Answer:**

| | Hashtable | HashMap | ConcurrentHashMap |
|---|---|---|---|
| Thread-safe | Yes (whole-object lock) | No | Yes (per-bin lock + CAS) |
| Null key/value | No | one null key, many null values | No |
| Concurrency | none (single lock) | n/a | high (per-bin) |
| Since / status | 1.0, legacy | 1.2 | 1.5 |
| Iterator | fail-fast (via Enumeration legacy) | fail-fast | weakly consistent |

- **Hashtable** — legacy, every method `synchronized` on the whole object → serial access, bottleneck. Do not use in new code.
- **HashMap** — not synchronized, fastest single-threaded, allows one null key. Use when no concurrency or external synchronization.
- **ConcurrentHashMap** — the modern concurrent choice: per-bin locking + CAS, no null keys/values, weakly-consistent iterators, atomic `compute`/`merge`.

For a synchronized HashMap without CHM you could use `Collections.synchronizedMap(new HashMap<>())`, but that is whole-object locking like Hashtable and still needs manual synchronization for iteration — CHM is almost always better.

**Follow-up questions the interviewer will ask:**

1. *"`Collections.synchronizedMap` vs `ConcurrentHashMap`?"* → `synchronizedMap` wraps each method in a lock on one mutex (whole-map serialization) and *requires you to manually synchronize during iteration*. CHM has fine-grained locking and lock-free reads — far higher throughput.

2. *"Why no null in CHM?"* → A `null` from `get` would be ambiguous (key absent vs mapped to null) and you cannot atomically distinguish them under concurrency.

**Common mistakes that get you rejected:**
- Recommending Hashtable.
- Thinking `synchronizedMap` gives the same concurrency as CHM.

---

# MULTITHREADING & CONCURRENCY

The deepest part of any senior Java loop. The questions probe the **Java Memory Model** — visibility, ordering, atomicity — because that is what separates "I have used `synchronized`" from "I understand why it works."

---

### Q27: Thread Lifecycle and States
**Company:** Amazon, Microsoft, Infosys
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
What are the states of a Java thread? How does it move between them?

**What interviewer is testing:**
The `Thread.State` enum and the transitions — a foundation for everything else.

**Ideal Answer:**

Six states (`Thread.State`):

```
NEW  --start()-->  RUNNABLE  <---->  (running on CPU / ready)
                      |  \
   wait()/join()/     |   \  synchronized blocked on monitor
   park               |    \--------------------> BLOCKED
                      v
                   WAITING / TIMED_WAITING  --notify()/timeout--> RUNNABLE
                      |
                   run() returns / exception
                      v
                  TERMINATED
```

- **NEW** — created but `start()` not called.
- **RUNNABLE** — eligible to run (running or waiting for CPU; Java does not distinguish "ready" from "running").
- **BLOCKED** — waiting to acquire a monitor lock (entering a `synchronized` block held by another thread).
- **WAITING** — indefinitely waiting via `wait()`, `join()`, `LockSupport.park()` until another thread signals.
- **TIMED_WAITING** — waiting with a timeout: `sleep(ms)`, `wait(ms)`, `join(ms)`, `parkNanos`.
- **TERMINATED** — `run()` completed or threw.

```java
Thread t = new Thread(() -> work());   // NEW
t.start();                              // -> RUNNABLE
t.join();                               // current thread WAITING until t TERMINATED
```

**Follow-up questions the interviewer will ask:**

1. *"BLOCKED vs WAITING?"* → BLOCKED is specifically waiting to *acquire a monitor lock* to enter `synchronized`. WAITING/TIMED_WAITING is voluntarily suspended (via `wait`/`join`/`park`) until signaled. Different causes, different wake-ups.

2. *"`start()` vs `run()`?"* → `start()` creates a new thread and the JVM calls `run()` on it. Calling `run()` directly just executes it on the *current* thread — no new thread. Calling `start()` twice throws `IllegalThreadStateException`.

3. *"Can a thread go back to NEW?"* → No. A Thread object cannot be restarted once terminated.

**Common mistakes that get you rejected:**
- Confusing BLOCKED with WAITING.
- Saying `run()` starts a new thread.

---

### Q28: Runnable vs Callable, Thread vs Pool
**Company:** Amazon, Microsoft, Flipkart
**Difficulty:** 🟢 Easy
**Frequency:** 🔥🔥
**Round:** Phone

**Question:**
Runnable vs Callable? Why use a thread pool instead of `new Thread()`?

**Ideal Answer:**

- **`Runnable`** — `void run()`, cannot return a value or throw checked exceptions.
- **`Callable<V>`** — `V call() throws Exception`, returns a value and can throw checked exceptions. Submit to an `ExecutorService` to get a `Future<V>`.

```java
Runnable r = () -> System.out.println("no result");
Callable<Integer> c = () -> compute();   // returns a value

ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(c);
Integer result = f.get();                 // blocks until done
```

**Why a pool, not `new Thread()` per task:**
1. **Thread creation is expensive** (OS thread, ~1MB stack each). Pools reuse threads.
2. **Bounded resource use** — unbounded `new Thread()` under load exhausts memory/OS limits → `OutOfMemoryError: unable to create native thread`.
3. **Queuing and backpressure** — a pool queues excess work instead of spawning unbounded threads.
4. **Lifecycle management** — graceful shutdown, rejection policies, metrics.

**Follow-up questions the interviewer will ask:**

1. *"Why not `Executors.newCachedThreadPool()` everywhere?"* → It has an *unbounded* thread count; under a burst it can spawn thousands of threads and OOM. Prefer a bounded `ThreadPoolExecutor` with an explicit queue and rejection policy in production.

2. *"How does virtual threads change this?"* → With Java 21 virtual threads, the "threads are expensive" premise weakens — you can have millions. Pooling virtual threads is an anti-pattern; use `newVirtualThreadPerTaskExecutor()` (Q53/54).

**Common mistakes that get you rejected:**
- `new Thread().start()` per request in a server.
- Using factory methods without considering the unbounded queue/threads.

---

### Q29: `synchronized` — Monitors and Intrinsic Locks
**Company:** Amazon, Google, Microsoft
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Phone / Onsite

**Question:**
How does `synchronized` work? What is a monitor? What does it guarantee?

**What interviewer is testing:**
Mutual exclusion *and* memory visibility — `synchronized` does both, which many forget.

**Ideal Answer:**

Every Java object has an **intrinsic lock (monitor)**. `synchronized` acquires that monitor on entry and releases it on exit (including via exception). Only one thread can hold an object's monitor at a time, so synchronized blocks on the *same* object are **mutually exclusive**.

It provides **two** guarantees:
1. **Mutual exclusion (atomicity)** — one thread in the critical section at a time.
2. **Visibility (happens-before)** — releasing a monitor *flushes* writes to main memory; acquiring it *invalidates* the cache so the next thread sees those writes. This is the part people forget: `synchronized` is not only about mutual exclusion, it establishes happens-before ordering.

```java
class Counter {
    private int count;
    public synchronized void inc() { count++; }          // locks on `this`
    public synchronized int get() { return count; }       // same lock -> visibility
}

private final Object lock = new Object();
void incBlock() { synchronized (lock) { count++; } }      // block form
```

- **Instance method** `synchronized` → locks on `this`.
- **Static method** `synchronized` → locks on the `Class` object (`Counter.class`).
- **Block** → locks on the specified object.

Intrinsic locks are **reentrant**: a thread holding a monitor can re-enter other synchronized blocks on the same monitor, preventing self-deadlock.

**Follow-up questions the interviewer will ask:**

1. *"Why is `count++` not atomic even though it is one line?"* → It is read-modify-write: load, increment, store. Two threads can interleave and lose an update. `synchronized` (or `AtomicInteger`) makes it atomic.

2. *"How to reduce lock contention?"* → Narrow the critical section, lock striping/splitting, lock-free structures (`Atomic*`, `ConcurrentHashMap`), or read-write locks for read-heavy data.

3. *"Lock on `this` vs a private lock — why prefer private?"* → Locking on `this` (or a public object) lets external code synchronize on the same monitor and accidentally deadlock/contend with you. A `private final Object lock` cannot be hijacked.

**Common mistakes that get you rejected:**
- Saying `synchronized` only gives mutual exclusion (it also gives visibility).
- Synchronizing on a non-final field or a boxed/interned value.
- Synchronizing on `this` for sensitive code.

---

### Q30: `volatile` — Visibility vs Atomicity
**Company:** Amazon, Google, Microsoft, Meta, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
What does `volatile` guarantee and what does it NOT guarantee? When do you use it instead of `synchronized`?

**What interviewer is testing:**
The most misunderstood concurrency keyword. The key insight: `volatile` gives **visibility and ordering** but **not atomicity** for compound operations.

**Ideal Answer:**

`volatile` on a field guarantees:
1. **Visibility** — a write to a volatile is immediately visible to all threads; reads always fetch from main memory (no per-thread cached copy). Without it, a thread can spin forever on a stale cached value.
2. **Ordering** — a volatile write establishes a happens-before with subsequent volatile reads of the same field. Reads/writes are not reordered across the volatile access (it acts as a memory barrier).
3. **Atomicity of a single read or single write** — including `long`/`double` (which are otherwise allowed to tear into two 32-bit writes).

What `volatile` does **NOT** give:
- **Atomicity of compound (read-modify-write) operations.** `count++` on a volatile is still three steps and races. Volatile makes each individual read/write atomic and visible, not the increment as a whole.

The classic correct use — a visibility flag:

```java
class Worker {
    private volatile boolean running = true;   // visibility across threads
    public void run() {
        while (running) { doWork(); }           // sees the update promptly
    }
    public void stop() { running = false; }     // visible immediately
}
// WITHOUT volatile, the worker may cache running=true and loop forever.
```

The classic *wrong* use — a counter:

```java
private volatile int count;
public void inc() { count++; }   // BROKEN — read-modify-write is not atomic; updates lost
```

For that use `AtomicInteger` (CAS) or `synchronized`.

**volatile vs synchronized:**
- `volatile`: visibility + ordering, no locking, only for single reads/writes (flags, safe publication, double-checked locking).
- `synchronized`/locks: visibility + ordering **+ mutual exclusion** for compound operations, at the cost of blocking.

**Follow-up questions the interviewer will ask:**

1. *"Can you implement a thread-safe counter with `volatile`?"* → No, not for increments. Use `AtomicInteger.incrementAndGet()` (lock-free CAS) or `synchronized`.

2. *"Where is volatile essential?"* → The double-checked locking singleton (Q62) — the instance field must be `volatile` so other threads do not see a *partially constructed* object due to reordering of the allocation. Also stop-flags and safe publication.

3. *"Does volatile prevent reordering of *other* operations?"* → Yes, partially: writes before a volatile write are not reordered after it, and reads after a volatile read are not reordered before it (acquire/release semantics). This is how a volatile flag safely publishes data written before it.

**Common mistakes that get you rejected:**
- Saying `volatile` makes `count++` thread-safe.
- Not knowing it provides ordering (memory-barrier) semantics.
- Forgetting `volatile` is required for double-checked locking.

---

### Q31: The Happens-Before Relationship and the JMM
**Company:** Google, Amazon, Meta
**Difficulty:** ⚫ Expert
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
What is the happens-before relationship in the Java Memory Model? Why does it matter?

**What interviewer is testing:**
The theoretical foundation of Java concurrency. Separates people who *understand* concurrency from those who memorized keywords.

**Ideal Answer:**

The **Java Memory Model (JMM)** defines when a write by one thread is *guaranteed visible* to a read by another, and what reorderings are legal. Without it, compilers/CPUs may reorder and cache operations such that one thread never sees another's writes.

**Happens-before (HB)** is a partial ordering: if action A *happens-before* B, then A's memory writes are visible to B, and A is ordered before B. If two actions are *not* ordered by HB, the JVM may reorder them and a data race exists.

The HB rules you must name:
1. **Program order** — within a single thread, each action happens-before the next in program order.
2. **Monitor lock** — unlocking a monitor happens-before every subsequent lock of the *same* monitor. (Why `synchronized` gives visibility.)
3. **Volatile** — a write to a volatile happens-before every subsequent read of that same volatile.
4. **Thread start** — `Thread.start()` happens-before any action in the started thread.
5. **Thread join** — all actions in a thread happen-before another thread's successful return from `join()`.
6. **Transitivity** — if A hb B and B hb C, then A hb C.
7. **final fields** — proper construction publishes `final` fields safely (visible without synchronization once the constructor completes without leaking `this`).

```java
int data;                 // non-volatile
volatile boolean ready;   // volatile

// Thread A:
data = 42;                // (1) ordinary write
ready = true;             // (2) volatile write — HB edge

// Thread B:
if (ready) {              // (3) volatile read — sees true
    use(data);            // (4) GUARANTEED to see 42: (1) hb (2) hb (3) hb (4)
}
```

The volatile write (2) happens-before the volatile read (3); program order gives (1) hb (2) and (3) hb (4); transitivity makes (1) visible to (4). The non-volatile `data` is safely published by the volatile flag.

A **data race** is two accesses to the same non-final, non-volatile variable, at least one a write, not ordered by happens-before. Racy programs have no guarantees.

**Follow-up questions the interviewer will ask:**

1. *"Why can a thread loop forever on a non-volatile flag?"* → No HB edge connects the writer's `flag=false` to the reader's read, so the JIT may hoist the read out of the loop — the reader never observes the change. `volatile` or `synchronized` creates the edge.

2. *"How do `final` fields get safe publication?"* → If `final` fields are set in the constructor and `this` does not escape during construction, any thread seeing a reference to the fully-constructed object sees correct `final` values without synchronization. Why immutable objects are safely shareable.

3. *"Does happens-before mean A executes before B in wall-clock time?"* → No — it is about *visibility and ordering of effects*, not timing.

**Common mistakes that get you rejected:**
- Treating happens-before as wall-clock ordering.
- Not being able to name the monitor and volatile rules.

---

### Q32: `wait()`, `notify()`, `notifyAll()`
**Company:** Amazon, Microsoft, Goldman Sachs
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
How do `wait`/`notify` work? Why must they be called inside `synchronized`? Why a `while` loop, not `if`?

**What interviewer is testing:**
Monitor condition signaling and the spurious-wakeup / missed-signal pitfalls.

**Ideal Answer:**

`wait`, `notify`, `notifyAll` are `Object` methods for inter-thread coordination on a monitor:
- `wait()` — atomically releases the monitor and suspends until notified (or timeout).
- `notify()` — wakes one waiting thread on that monitor.
- `notifyAll()` — wakes all waiting threads (they re-contend for the lock).

**They must be called while holding the monitor** (inside `synchronized` on that object), else `IllegalMonitorStateException`.

**Always wait in a `while` loop, never `if`:**

```java
class BoundedBuffer<T> {
    private final Queue<T> q = new LinkedList<>();
    private final int cap;
    BoundedBuffer(int cap) { this.cap = cap; }

    public synchronized void put(T x) throws InterruptedException {
        while (q.size() == cap) wait();      // WHILE, not IF
        q.add(x);
        notifyAll();
    }
    public synchronized T take() throws InterruptedException {
        while (q.isEmpty()) wait();           // WHILE
        T x = q.remove();
        notifyAll();
        return x;
    }
}
```

Why `while`:
1. **Spurious wakeups** — a thread can wake without any `notify`. The loop re-checks the condition.
2. **Missed/stale condition** — with `notifyAll`, multiple threads wake but the condition may no longer hold when a given thread reacquires the lock. Re-checking is mandatory.

**Prefer `notifyAll` over `notify`** unless all waiters are interchangeable and exactly one should proceed.

**Follow-up questions the interviewer will ask:**

1. *"Modern alternative to wait/notify?"* → `BlockingQueue`, `Lock`+`Condition` (`await`/`signal` with multiple condition queues), `CountDownLatch`, `Semaphore`. Prefer these.

2. *"`notify` vs `notifyAll`?"* → `notify` is cheaper but risky (lost wakeups if waiters are not interchangeable). `notifyAll` is safe but causes a thundering herd. `Condition` objects target specific waiter groups.

3. *"Does `wait()` release the lock? Does `sleep()`?"* → `wait()` releases the monitor. `Thread.sleep()` does **not** — it holds locks while sleeping (a common deadlock cause).

**Common mistakes that get you rejected:**
- Using `if` instead of `while` around `wait()`.
- Calling wait/notify outside `synchronized`.
- Thinking `sleep()` releases locks.

---

### Q33: Creating a Deadlock — and Four Ways to Prevent It
**Company:** Amazon, Google, Microsoft, Meta
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Write code that deadlocks. Then explain how to prevent it.

**What interviewer is testing:**
The four Coffman conditions and concrete prevention strategies — especially **lock ordering**.

**Ideal Answer:**

A deadlock needs all four **Coffman conditions** simultaneously: **mutual exclusion**, **hold-and-wait**, **no preemption**, and **circular wait**. Break any one and deadlock is impossible.

Classic two-lock deadlock (lock-ordering inversion):

```java
Object A = new Object(), B = new Object();

// Thread 1
synchronized (A) {
    Thread.sleep(50);
    synchronized (B) { /* ... */ }   // waits for B (held by Thread 2)
}
// Thread 2
synchronized (B) {
    Thread.sleep(50);
    synchronized (A) { /* ... */ }   // waits for A (held by Thread 1) -> circular wait
}
```

**Prevention strategies:**

1. **Global lock ordering (the standard fix).** Always acquire locks in a fixed total order.

```java
void transfer(Account from, Account to, long amt) {
    Account first  = from.id < to.id ? from : to;   // consistent order by id
    Account second = from.id < to.id ? to : from;
    synchronized (first) {
        synchronized (second) { from.debit(amt); to.credit(amt); }
    }
}
```

2. **Lock timeout / tryLock (breaks hold-and-wait).**

```java
if (lockA.tryLock(1, TimeUnit.SECONDS)) {
    try {
        if (lockB.tryLock(1, TimeUnit.SECONDS)) {
            try { /* work */ } finally { lockB.unlock(); }
        } else { /* back off, release A, retry */ }
    } finally { lockA.unlock(); }
}
```

3. **Avoid nested locks / reduce lock scope.** Do not call alien methods that may lock while holding a lock.

4. **Single coarse lock / lock-free structures.** One lock for related state, or `Atomic*`/`ConcurrentHashMap`.

**Detecting it:** `jstack <pid>` reports "Found one Java-level deadlock". `ThreadMXBean.findDeadlockedThreads()` detects programmatically.

**Follow-up questions the interviewer will ask:**

1. *"Deadlock vs livelock vs starvation?"* → Deadlock: threads blocked forever in a cycle. Livelock: threads keep changing state in response to each other but make no progress. Starvation: a thread never gets the lock/CPU because others monopolize it.

2. *"How does lock ordering scale to many locks?"* → Define a total order (by id, hash, or global rank) and always lock ascending; use `System.identityHashCode` as a tie-breaker with a third "tie lock" for hash collisions.

**Common mistakes that get you rejected:**
- Not knowing the four Coffman conditions.
- Suggesting only "use synchronized carefully" without lock ordering / tryLock.
- Holding a lock while calling alien code.

---

### Q34: Atomic Classes and CAS
**Company:** Amazon, Google, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
How do `AtomicInteger` and friends achieve thread safety without locks? What is CAS and the ABA problem?

**What interviewer is testing:**
Lock-free programming and compare-and-swap, the hardware primitive behind much of `java.util.concurrent`.

**Ideal Answer:**

Atomic classes use **CAS (compare-and-swap)** — a single atomic CPU instruction (`cmpxchg`): "if this location still equals `expected`, set it to `newValue`; return whether it succeeded." No lock, no blocking.

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();   // atomic, lock-free

// what incrementAndGet does internally (CAS retry loop):
int prev, next;
do {
    prev = counter.get();
    next = prev + 1;
} while (!counter.compareAndSet(prev, next));  // retry if another thread changed it
```

This is **optimistic concurrency**: attempt the update, retry on conflict. Under low contention it beats locking (no context switch). Under high contention the retry loop spins — `LongAdder` (striped counters) wins there.

**The ABA problem:** CAS only checks the value *equals* `expected`, not that it never changed. If a value goes A → B → A, a CAS expecting A succeeds, missing the change. Fix with `AtomicStampedReference` (value + version stamp):

```java
AtomicStampedReference<Node> top = new AtomicStampedReference<>(node, 0);
int[] stampHolder = new int[1];
Node current = top.get(stampHolder);
top.compareAndSet(current, newNode, stampHolder[0], stampHolder[0] + 1); // stamp guards ABA
```

**Follow-up questions the interviewer will ask:**

1. *"AtomicInteger vs synchronized counter?"* → Atomic is lock-free and faster under low/moderate contention. Under very high contention the CAS spin wastes CPU; `LongAdder` outperforms both for hot counters.

2. *"LongAdder vs AtomicLong?"* → `AtomicLong` is a single hot CAS target. `LongAdder` keeps striped cells, so threads update different cells with little contention; `sum()` adds them. Use it for write-heavy counters.

3. *"Is CAS wait-free?"* → The algorithm is lock-free (system-wide progress) but an individual thread can starve under constant contention (not wait-free).

**Common mistakes that get you rejected:**
- Thinking atomics use locks internally.
- Not knowing the ABA problem or its `AtomicStampedReference` fix.
- Using `AtomicLong` for an extremely hot counter instead of `LongAdder`.

---

### Q35: ReentrantLock vs synchronized
**Company:** Amazon, Microsoft, Goldman Sachs
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
When would you use `ReentrantLock` over `synchronized`?

**Ideal Answer:**

`ReentrantLock` is an explicit lock with capabilities `synchronized` lacks:

| Feature | `synchronized` | `ReentrantLock` |
|---|---|---|
| Acquire | implicit | explicit `lock()`/`unlock()` |
| Try without blocking | no | `tryLock()` |
| Timed acquire | no | `tryLock(time, unit)` |
| Interruptible wait | no | `lockInterruptibly()` |
| Fairness option | no | `new ReentrantLock(true)` |
| Multiple conditions | one wait set | many `Condition` objects |
| Release | automatic | **must** `unlock()` in `finally` |

```java
private final ReentrantLock lock = new ReentrantLock();
public void doWork() {
    lock.lock();
    try {
        // critical section
    } finally {
        lock.unlock();           // MUST be in finally — no auto-release
    }
}
```

Use `synchronized` by default (simpler, auto-release, JVM-optimized). Reach for `ReentrantLock` when you need `tryLock`/timeout, interruptible locking, fairness, or multiple condition variables.

**Follow-up questions the interviewer will ask:**

1. *"Danger of ReentrantLock?"* → Forgetting `unlock()` (or not in `finally`) leaks the lock → permanent deadlock. `synchronized` cannot leak.

2. *"`ReadWriteLock`?"* → `ReentrantReadWriteLock` allows multiple readers OR one writer — great for read-heavy data. `StampedLock` adds optimistic reads.

3. *"Is `synchronized` slower?"* → Not really — the JVM heavily optimizes it. Choose based on features needed.

**Common mistakes that get you rejected:**
- `unlock()` not in a `finally`.
- Reaching for ReentrantLock when `synchronized` suffices.

---

### Q36: ThreadLocal — Uses and the Memory Leak Trap
**Company:** Amazon, Google, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
What is `ThreadLocal` for? What is the leak risk in a thread pool?

**What interviewer is testing:**
Per-thread state and the classic thread-pool leak.

**Ideal Answer:**

`ThreadLocal<T>` gives each thread its own copy of a value — no sharing, no synchronization. Uses: per-request context (user, trace id), non-thread-safe objects reused per thread (`SimpleDateFormat`), transaction/connection context (Spring's `TransactionSynchronizationManager`).

```java
private static final ThreadLocal<SimpleDateFormat> FMT =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

String format(Date d) { return FMT.get().format(d); }   // each thread its own formatter
```

**The leak trap.** A `ThreadLocal` value lives as long as the *thread* does. In a **thread pool**, threads are reused indefinitely, so:
1. Values set in one task **persist** into the next task on the same thread → stale data, possibly across users (a security risk).
2. The value is never GC'd while the thread lives → memory leak.

**Always clean up in a `finally`:**

```java
try {
    CONTEXT.set(requestContext);
    handle(request);
} finally {
    CONTEXT.remove();        // critical in pooled threads
}
```

**Follow-up questions the interviewer will ask:**

1. *"Why is the leak subtle with weak keys?"* → `ThreadLocalMap` keys are weak references but values are strong. If the ThreadLocal is GC'd but the thread lives, a stale entry (null key, retained value) lingers until the next map operation. Explicit `remove()` avoids relying on that.

2. *"ThreadLocal with virtual threads?"* → Works but discouraged at massive scale. Java 21 introduces **`ScopedValue`** (preview) as a lighter, immutable alternative.

**Common mistakes that get you rejected:**
- Not calling `remove()` in pooled environments → cross-request data leakage.
- Assuming ThreadLocal is cleaned between pool tasks.

---

### Q37: ExecutorService and ThreadPoolExecutor Tuning
**Company:** Amazon, Google, Microsoft, Uber, Netflix
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Explain the `ThreadPoolExecutor` parameters: core/max pool size, queue, keep-alive, rejection policy. How does a task flow through them?

**What interviewer is testing:**
Real production tuning. The *order* in which the pool uses core threads, queue, then max threads is counterintuitive and frequently gotten wrong.

**Ideal Answer:**

```java
new ThreadPoolExecutor(
    corePoolSize,      // threads kept alive even when idle
    maximumPoolSize,   // upper bound on threads
    keepAliveTime,     // idle time before non-core threads die
    TimeUnit.SECONDS,
    workQueue,         // holds tasks waiting for a thread
    threadFactory,     // names threads, sets daemon/priority
    rejectedHandler);  // what to do when saturated
```

**The crucial task-admission order (the gotcha):**
1. If fewer than `corePoolSize` threads running, **create a new core thread** (even if other threads are idle).
2. Else, **enqueue** the task.
3. Only if the queue is **full** does it create threads up to `maximumPoolSize`.
4. If queue full *and* `maximumPoolSize` reached, the **rejection policy** fires.

The counterintuitive part: with an **unbounded** queue (`newFixedThreadPool`'s default), step 2 never fills, so `maximumPoolSize` is **never reached** and tasks pile up unboundedly (OOM). To use `maxPoolSize`, you need a **bounded** queue.

**Rejection policies:**
- `AbortPolicy` (default) — throws `RejectedExecutionException`.
- `CallerRunsPolicy` — runs on the caller's thread (natural backpressure).
- `DiscardPolicy` — silently drops.
- `DiscardOldestPolicy` — drops oldest queued, retries.

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    8, 32, 60L, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),                 // BOUNDED -> max threads usable
    new ThreadFactoryBuilder().setNameFormat("worker-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy());     // backpressure
```

**Sizing:** CPU-bound ≈ cores (+1); I/O-bound: more threads (`cores * (1 + waitTime/computeTime)`), bounded by downstream capacity.

**Follow-up questions the interviewer will ask:**

1. *"Why is `newFixedThreadPool` dangerous?"* → Unbounded `LinkedBlockingQueue`: under overload, tasks queue forever → OOM, no backpressure. Use an explicit bounded `ThreadPoolExecutor` + `CallerRunsPolicy`.

2. *"Size for a web service doing DB/HTTP calls?"* → I/O-bound: many more threads than cores, bounded by downstream (e.g., DB connection pool size), with backpressure.

3. *"Graceful shutdown?"* → `shutdown()` then `awaitTermination(timeout)`, then `shutdownNow()` if needed.

**Common mistakes that get you rejected:**
- Saying tasks go to max threads before filling the queue (wrong order).
- Using `newFixedThreadPool`/`newCachedThreadPool` in production without understanding the unbounded queue/threads.
- Never shutting the pool down.

---

### Q38: Future vs CompletableFuture
**Company:** Amazon, Google, Microsoft, Uber
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
What are the limitations of `Future`, and how does `CompletableFuture` address them?

**What interviewer is testing:**
Why the async API evolved — `Future` is blocking and uncomposable.

**Ideal Answer:**

`Future<V>` (Java 5) limitations:
1. **Blocking get** — `future.get()` blocks; no completion callback.
2. **No chaining** — cannot say "when this finishes, do that" without blocking.
3. **No combining** of multiple futures.
4. **No manual completion / functional error handling.**

```java
Future<Integer> f = executor.submit(() -> compute());
Integer r = f.get();    // BLOCKS the calling thread
```

`CompletableFuture<V>` (Java 8) implements `Future` *and* `CompletionStage`, enabling non-blocking composition:

```java
CompletableFuture
    .supplyAsync(() -> fetchUser(id), pool)
    .thenApply(User::email)
    .thenCompose(email -> sendAsync(email))
    .thenAccept(result -> log.info("sent {}", result))
    .exceptionally(ex -> { log.error("failed", ex); return null; });
```

Abilities `Future` lacks: callbacks (`thenApply`/`thenAccept`/`thenRun`), composition (`thenCompose`/`thenCombine`/`allOf`/`anyOf`), explicit completion (`complete`/`completeExceptionally`), functional recovery (`exceptionally`/`handle`).

**Follow-up questions the interviewer will ask:**

1. *"`thenApply` vs `thenApplyAsync`?"* → `thenApply` runs the callback on the thread that completed the previous stage (or caller). `thenApplyAsync` runs it on the common pool (or supplied executor), freeing the completing thread.

2. *"Default executor for `supplyAsync`?"* → `ForkJoinPool.commonPool()`. For blocking I/O this is dangerous (few threads) — always pass your own executor.

3. *"Wait for several futures?"* → `allOf(cf...)` completes when all complete (returns `CompletableFuture<Void>`); `anyOf` on the first.

**Common mistakes that get you rejected:**
- Blocking with `.get()` inside an async pipeline.
- Using the common pool for blocking I/O.
- Not handling exceptions (a lost exception vanishes silently).

---

### Q39: CompletableFuture Chaining — thenApply / thenCompose / thenCombine
**Company:** Amazon, Google, Microsoft, Netflix, Uber
**Difficulty:** 🔴 Hard
**Frequency:** 🔥🔥🔥
**Round:** Onsite

**Question:**
Explain the difference between `thenApply`, `thenCompose`, and `thenCombine`. When do you use each?

**What interviewer is testing:**
The map vs flatMap distinction (`thenApply` vs `thenCompose`) and combining independent futures (`thenCombine`).

**Ideal Answer:**

**`thenApply(Function<T,R>)` — map.** Transforms with a *synchronous* function returning a plain value. Use when the next step is **not** async.

```java
CompletableFuture<String> name = userFuture.thenApply(User::name);  // User -> String
```

**`thenCompose(Function<T, CompletionStage<R>>)` — flatMap.** The function returns *another CompletableFuture*. Use to chain a **dependent async call** without nesting. `thenApply` with an async function gives `CompletableFuture<CompletableFuture<R>>` (nested); `thenCompose` flattens it.

```java
CompletableFuture<List<Order>> orders =
    userFuture.thenCompose(user -> fetchOrdersAsync(user.id()));   // flattened
```

**`thenCombine(otherCF, BiFunction)` — zip two independent futures** running in **parallel**, combine when both finish.

```java
CompletableFuture<Price> price = supplyAsync(() -> fetchPrice(sku), pool);
CompletableFuture<Stock> stock = supplyAsync(() -> fetchStock(sku), pool);
CompletableFuture<Listing> listing =
    price.thenCombine(stock, (p, s) -> new Listing(p, s));   // concurrent, then join
```

| Method | Function returns | Futures | Analogy |
|---|---|---|---|
| `thenApply` | a value `R` | 1 (chain) | `map` |
| `thenCompose` | a `CompletableFuture<R>` | 1 (dependent chain) | `flatMap` |
| `thenCombine` | a value from two results | 2 (parallel) | `zip` |

Fan-out/fan-in example:

```java
public CompletableFuture<Dashboard> buildDashboard(String userId) {
    CompletableFuture<Profile> profile = supplyAsync(() -> profileSvc.get(userId), pool);
    CompletableFuture<List<Order>> orders =
        profile.thenCompose(p -> supplyAsync(() -> orderSvc.recent(p.id()), pool)); // dependent
    CompletableFuture<List<Notification>> notifs =
        supplyAsync(() -> notifSvc.unread(userId), pool);                            // independent

    return profile
        .thenCombine(orders, DashboardParts::new)
        .thenCombine(notifs, DashboardParts::withNotifs)
        .thenApply(Dashboard::from)
        .exceptionally(ex -> Dashboard.empty());
}
```

**Follow-up questions the interviewer will ask:**

1. *"Exceptions mid-chain?"* → They short-circuit, skipping `thenApply`/`thenCompose` stages, until `exceptionally`/`handle`/`whenComplete` catches. `handle((res, ex) -> ...)` runs on success and failure; `whenComplete` observes without transforming.

2. *"`thenCombine` vs `allOf`?"* → `thenCombine` zips exactly two into a combined value. `allOf(cf...)` waits for N but returns `CompletableFuture<Void>` — read each result manually.

3. *"Does `thenCompose` run the second call only after the first?"* → Yes, dependent/sequential. `thenCombine` runs both concurrently.

**Common mistakes that get you rejected:**
- `thenApply` with an async function → nested `CompletableFuture<CompletableFuture<...>>`.
- `thenCompose` for independent futures (serializes parallelizable work).
- Forgetting exception handling.

---

### Q40: Producer-Consumer with BlockingQueue
**Company:** Amazon, Microsoft, Flipkart
**Difficulty:** 🟡 Medium
**Frequency:** 🔥🔥
**Round:** Onsite

**Question:**
Implement the producer-consumer pattern. Why is `BlockingQueue` better than hand-rolled `wait/notify`?

**Ideal Answer:**

`BlockingQueue` encapsulates bounded-buffer coordination — `put` blocks when full, `take` blocks when empty.

```java
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(1000);  // bounded -> backpressure

class Producer implements Runnable {
    public void run() {
        try { while (true) queue.put(produce()); }            // blocks if full
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}
class Consumer implements Runnable {
    public void run() {
        try { while (true) process(queue.take()); }           // blocks if empty
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}
```

Why over wait/notify: no manual lock/condition management, no missed-signal/spurious-wakeup bugs, built-in backpressure, and implementations for every need (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`, `SynchronousQueue`, `DelayQueue`).

**Follow-up questions the interviewer will ask:**

1. *"Clean shutdown?"* → A "poison pill" sentinel: producers enqueue a terminal token; consumers exit on taking it. Or interrupt and handle `InterruptedException`.

2. *"`offer`/`poll` vs `put`/`take`?"* → `put`/`take` block; `offer`/`poll` return immediately (with optional timeout) — use when you do not want to block indefinitely.

**Common mistakes that get you rejected:**
- Hand-rolling wait/notify when a `BlockingQueue` exists.
- Unbounded queue (no backpressure → OOM).
- Swallowing `InterruptedException` without restoring the flag.

---

## Java 8–21 Modern Features

### Q41: Java 8 Streams — Lazy Evaluation
**Company:** Amazon, Google, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Explain how Java 8 Streams are lazy. What is the difference between intermediate and terminal operations? Give an example where laziness changes correctness.

**What interviewer is testing:** Whether you understand that Streams are pipelines, not data structures — that nothing executes until a terminal operation is called, and that short-circuiting can avoid processing the entire source.

**Ideal Answer:**

A Stream is a description of computation, not a container of data. Intermediate operations (`filter`, `map`, `flatMap`, `sorted`, `distinct`, `peek`, `limit`, `skip`) return a new Stream and are **lazy** — they are not evaluated until a terminal operation fires. Terminal operations (`collect`, `forEach`, `reduce`, `count`, `findFirst`, `anyMatch`, `min`, `max`) consume the pipeline and produce a result or side effect.

**Why laziness matters — short-circuiting:**

```java
// Without laziness you'd process all 1 million elements.
// With laziness, findFirst stops after the first match.
OptionalInt first = IntStream.range(0, 1_000_000)
    .filter(n -> {
        System.out.println("filter: " + n);  // prints only until match
        return n % 1000 == 0 && isPrime(n);
    })
    .findFirst();
```

**Internal pipeline — Spliterator + Sink chain:**

```
Source (Spliterator)
  │
  ▼
filter (StatelessOp) ── Sink.accept() called per element
  │
  ▼
map (StatelessOp)
  │
  ▼
limit (StatefulOp) ── stops upstream when count reached
  │
  ▼
collect (TerminalOp) ── drives the whole chain
```

Stateless ops (`filter`, `map`) pass elements one-at-a-time. Stateful ops (`sorted`, `distinct`) must buffer the entire stream first.

**Laziness gotcha — stream reuse:**

```java
Stream<String> s = list.stream().filter(x -> x.startsWith("A"));
s.count();        // fine
s.findFirst();    // IllegalStateException: stream has already been operated upon
```

**Infinite stream without limit hangs:**

```java
// WRONG — hangs forever
Stream.generate(Math::random).filter(d -> d > 0.999).forEach(System.out::println);

// CORRECT
Stream.generate(Math::random).filter(d -> d > 0.999).limit(5).forEach(System.out::println);
```

**Follow-up questions the interviewer will ask:**

1. *"What is `peek()` for and what is its trap?"* → `peek()` is for debugging — it applies a Consumer without consuming the stream. Trap: if the terminal operation short-circuits (e.g., `findFirst()`), `peek()` may not fire for all elements. Never use it for side effects.

2. *"When does `sorted()` break laziness?"* → `sorted()` is stateful — it must see ALL elements before emitting the first sorted one, buffering the entire stream into an array.

3. *"Difference between `map` and `flatMap`?"* → `map` is 1-to-1; `flatMap` is 1-to-many, flattening nested streams: `orders.stream().flatMap(o -> o.getItems().stream())`.

**Common mistakes that get you rejected:**
- Saying "streams are like collections" — they have no storage.
- Reusing a stream after a terminal operation.
- Using `peek()` for side effects in production.
- Putting `sorted()` early in a large pipeline without realizing it buffers everything.

---

### Q42: Stream.collect() and Collectors
**Company:** Amazon, Microsoft, Goldman Sachs  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Explain `Collectors.groupingBy`, `partitioningBy`, and `toMap`. Write a `toMap` that handles duplicate keys.

**What interviewer is testing:** Depth on the Collector API beyond `toList()` — downstream collectors and merge functions.

**Ideal Answer:**

`collect()` is a mutable reduction — it builds a result container via `supplier` (create container), `accumulator` (fold element in), and `combiner` (merge in parallel).

**groupingBy:**

```java
// Group employees by department
Map<String, List<Employee>> byDept =
    employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));

// Downstream collector: count per department
Map<String, Long> countByDept =
    employees.stream().collect(Collectors.groupingBy(
        Employee::getDepartment, Collectors.counting()));

// Two-level: dept -> level -> list
Map<String, Map<String, List<Employee>>> byDeptAndLevel =
    employees.stream().collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.groupingBy(Employee::getLevel)));
```

**partitioningBy — always returns exactly two keys (true/false):**

```java
Map<Boolean, List<Employee>> split =
    employees.stream().collect(Collectors.partitioningBy(e -> e.getYears() >= 5));
List<Employee> senior = split.get(true);
List<Employee> junior = split.get(false);
```

**toMap — with merge function for duplicates:**

```java
// WRONG — throws IllegalStateException on duplicate key
Map<Long, Employee> byId =
    employees.stream().collect(Collectors.toMap(Employee::getId, e -> e));

// CORRECT — merge function handles duplicates
Map<String, Employee> highestPaid =
    employees.stream().collect(Collectors.toMap(
        Employee::getDepartment,
        e -> e,
        (a, b) -> a.getSalary() >= b.getSalary() ? a : b));

// Control map type (4-arg form):
Map<String, Employee> linked =
    employees.stream().collect(Collectors.toMap(
        Employee::getDepartment, e -> e, (a, b) -> a, LinkedHashMap::new));
```

**Follow-up questions the interviewer will ask:**

1. *"What does `Collectors.toUnmodifiableMap` do differently?"* → Wraps in unmodifiable map; also throws on null values.

2. *"How do you collect to a `TreeMap`?"* → 4-arg `toMap` with `TreeMap::new`.

3. *"How do you get a comma-separated string?"* → `Collectors.joining(",")` or `String.join(",", list)`.

**Common mistakes that get you rejected:**
- Missing merge function in `toMap` when real data has duplicates — `IllegalStateException` in production.
- Confusing `groupingBy` (returns `Map<K, List<V>>`) with `toMap` (returns `Map<K, V>`).
- Treating `partitioningBy` as a filter — it always returns both buckets.

---

### Q43: Parallel Streams
**Company:** Google, Amazon, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** When should you use parallel streams? What are the correctness and performance pitfalls?

**What interviewer is testing:** Whether you know parallel streams require side-effect-free stateless lambdas and that shared mutable state causes silent data corruption.

**Ideal Answer:**

Parallel streams use the common `ForkJoinPool` (parallelism = CPU cores − 1) to split the source via `Spliterator`, process chunks on worker threads, and combine results.

**When to use:**
- Large data sets (rule of thumb: N > 10,000 with non-trivial per-element work).
- CPU-bound work (not I/O-bound).
- Source supports efficient splitting: `ArrayList` and arrays split well; `LinkedList` and `HashSet` split poorly.
- Operations are stateless and non-interfering.

**Pitfall 1 — shared mutable state (silent corruption):**

```java
// WRONG: ArrayList is not thread-safe — elements lost, duplicated, or NPE
List<Integer> results = new ArrayList<>();
IntStream.range(0, 10_000).parallel().filter(n -> n % 2 == 0).forEach(results::add);

// CORRECT: use collect()
List<Integer> results = IntStream.range(0, 10_000).parallel()
    .filter(n -> n % 2 == 0).boxed().collect(Collectors.toList());
```

**Pitfall 2 — ordering not guaranteed:**

```java
// parallel() may not preserve encounter order for some terminal ops
// Use .forEachOrdered() or sequential() if order matters
```

**Pitfall 3 — blocking the common ForkJoinPool with I/O:**

```java
// WRONG — network I/O starves ForkJoinPool workers
urls.parallelStream().map(url -> fetch(url)).collect(toList());

// CORRECT — use custom pool
ForkJoinPool pool = new ForkJoinPool(8);
List<Result> results = pool.submit(
    () -> urls.parallelStream().map(url -> fetch(url)).collect(toList())
).get();
pool.shutdown();
```

**Spliterator efficiency:**

```
ArrayList / int[] → splits at midpoint O(1) → EXCELLENT
LinkedList        → O(n) traversal to split → POOR
HashSet           → partial splits → OK
Stream.generate() → unknown size → POOR
```

**Follow-up questions the interviewer will ask:**

1. *"How do you control parallelism level?"* → Custom `ForkJoinPool`, submit parallel stream as a task. Common pool parallelism = `availableProcessors() - 1`.

2. *"Does `reduce()` work correctly in parallel?"* → Yes, if the combiner is associative and identity is correct. `reduce("", String::concat)` works but is O(n²) — use `joining()`.

3. *"When does parallel stream perform WORSE?"* → Small data sets (fork/join overhead dominates), I/O-bound ops, poor spliterators.

**Common mistakes that get you rejected:**
- Using `forEach` with a shared mutable list in a parallel stream.
- Assuming parallel is always faster — for < 10k elements, overhead typically outweighs gain.
- Doing I/O inside a parallel stream, blocking the common ForkJoinPool.

---

### Q44: Optional — Proper Usage
**Company:** Microsoft, Amazon, Adobe  **Difficulty:** 🟢 Easy  **Frequency:** 🔥🔥🔥  **Round:** Phone

**Question:** What is the difference between `orElse`, `orElseGet`, and `orElseThrow`? What are the Optional anti-patterns?

**What interviewer is testing:** That `orElse` is always eager while `orElseGet` is lazy, and knowledge of when Optional is inappropriate.

**Ideal Answer:**

**orElse vs orElseGet:**

```java
// orElse(T other) — other is ALWAYS evaluated, even when value is present
User user = findUser(id).orElse(createDefaultUser());   // called even if found!

// orElseGet(Supplier<T>) — Supplier called ONLY when Optional is empty
User user = findUser(id).orElseGet(() -> createDefaultUser());  // PREFER THIS

// orElseThrow
User user = findUser(id).orElseThrow(() -> new UserNotFoundException(id));
```

**Rule:** Prefer `orElseGet` when fallback involves computation/object creation. `orElse` is fine only for pre-computed constants: `.orElse(Collections.emptyList())`.

**Chaining:**

```java
Optional<String> city = findUser(id)
    .flatMap(User::getAddress)   // getAddress() returns Optional<Address>
    .map(Address::getCity);

findUser(id).ifPresentOrElse(
    u -> log.info("Found: {}", u.getName()),
    () -> log.warn("User not found: {}", id));
```

**Anti-patterns:**

```java
// 1. Optional as field — serialization breaks, extra memory
private Optional<String> nickname;  // WRONG

// 2. Optional as method parameter
public void process(Optional<String> name) { }  // WRONG — use overloading

// 3. get() without checking
opt.get();  // NoSuchElementException if empty

// 4. Optional.of(null) — NPE immediately
Optional<String> bad = Optional.of(null);  // use Optional.ofNullable(null)

// 5. isPresent() + get() — defeats the purpose
if (opt.isPresent()) { return opt.get().getName(); }
// Write: opt.map(User::getName).orElse("unknown")
```

**Follow-up questions the interviewer will ask:**

1. *"How is Optional different from null?"* → Optional is explicit in the type signature; null is implicit. Optional forces handling; null does not.

2. *"Optional and performance?"* → Optional allocates a heap object per wrap. In hot paths use `OptionalInt`/`OptionalLong`/`OptionalDouble` to avoid boxing.

**Common mistakes that get you rejected:**
- `orElse` with a method call (unnecessary eager evaluation).
- `Optional.of(null)` instead of `Optional.ofNullable(null)`.
- Optional as a field or method parameter.
- `isPresent()` + `get()` pattern.

---

### Q45: Method References — All Four Kinds
**Company:** Amazon, Flipkart, Adobe  **Difficulty:** 🟢 Easy  **Frequency:** 🔥🔥  **Round:** Phone

**Question:** Explain all four kinds of method references in Java 8 with examples.

**What interviewer is testing:** Fluency with functional interfaces and the syntactic sugar method references provide.

**Ideal Answer:**

```
Kind                     Syntax                    Lambda equivalent
─────────────────────── ─────────────────────────  ──────────────────────────────────
1. Static               ClassName::staticMethod    (args) -> ClassName.staticMethod(args)
2. Instance (unbound)   ClassName::instanceMethod  (obj, args) -> obj.instanceMethod(args)
3. Instance (bound)     instance::instanceMethod   (args) -> instance.instanceMethod(args)
4. Constructor          ClassName::new             (args) -> new ClassName(args)
```

**Examples:**

```java
// 1. Static
List<Integer> nums = List.of("1","2","3").stream()
    .map(Integer::parseInt).collect(toList());

// 2. Unbound instance — receiver is the stream element
List<String> upper = names.stream().map(String::toUpperCase).collect(toList());
names.sort(String::compareTo);  // (a, b) -> a.compareTo(b)

// 3. Bound instance — receiver is fixed
names.forEach(System.out::println);  // printer is System.out (bound)
Predicate<String> startsHello = "Hello"::startsWith;

// 4. Constructor
Supplier<List<String>> factory = ArrayList::new;
Function<String, User> userFactory = User::new;
```

**Follow-up questions the interviewer will ask:**

1. *"When can't you use a method reference?"* → When you need to transform arguments or combine multiple calls: `x -> process(x) + 1`.

2. *"What functional interface does `String::length` match?"* → `ToIntFunction<String>`. `Function<String, Integer>` also works but boxes — prefer `ToIntFunction` in tight loops.

3. *"Is `System.out::println` bound or unbound?"* → Bound — `System.out` is a specific `PrintStream` captured at reference creation.

**Common mistakes that get you rejected:**
- Confusing bound vs unbound — in unbound, the object comes from the stream element; in bound, it is fixed at reference creation.

---

### Q46: Default and Static Interface Methods
**Company:** Amazon, Google  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Phone/Onsite

**Question:** Explain default and static interface methods. How does Java resolve the diamond problem?

**What interviewer is testing:** Understanding of Java 8 interface evolution and deterministic conflict resolution rules.

**Ideal Answer:**

`default` methods provide a body in the interface for backward compatibility — existing implementors inherit the method without any code change.

```java
interface Printable {
    default void print() { System.out.println("Printable.print()"); }
    static void log(String msg) { System.out.println("[LOG] " + msg); }
}

class Report implements Printable { }  // inherits print() for free

Printable.log("started");   // OK — called on interface, not instance
// new Report().log(...)    // COMPILE ERROR — static methods not inherited
```

**Diamond problem — three resolution rules (priority order):**

```
Rule 1: Class wins over interface — always.
Rule 2: More specific interface wins (subinterface over superinterface).
Rule 3: Still ambiguous → compiler error; must override explicitly.
```

```java
interface A { default void hello() { System.out.println("A"); } }
interface B extends A { default void hello() { System.out.println("B"); } }
interface C extends A { default void hello() { System.out.println("C"); } }

class D implements B, C {
    // Rule 3: B and C equally specific → COMPILE ERROR — must override
    @Override
    public void hello() { B.super.hello(); }  // explicit delegation syntax
}
```

**Follow-up questions the interviewer will ask:**

1. *"Can default methods be `final`?"* → No. Any implementor can override them. Use abstract classes for non-overridable concrete methods.

2. *"Default method vs abstract class method?"* → Interfaces still cannot have instance fields or constructors. Default methods cannot access per-instance state.

3. *"What is `B.super.hello()` syntax?"* → Explicitly delegates to a specific interface's default implementation. Only valid inside a class that implements `B`.

**Common mistakes that get you rejected:**
- Thinking Java has no diamond problem with interfaces — it does, resolved deterministically.
- Calling a static interface method on an instance — compile error.
- Trying to share state through default methods — interfaces have no instance fields.

---

### Q47: Java 11–17 — var, Records, Sealed Classes, Pattern Matching
**Company:** Google, Amazon, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain `var` (Java 10), text blocks (Java 15), records (Java 16), sealed classes (Java 17), and pattern matching `instanceof` (Java 16).

**What interviewer is testing:** Whether you are current with modern Java idioms and understand their constraints, not just their syntax.

**Ideal Answer:**

**var — local variable type inference (Java 10):**

```java
var list = new ArrayList<String>();   // inferred: ArrayList<String>
var map  = Map.of("a", 1, "b", 2);   // inferred: Map<String, Integer>
// var is still statically typed at compile time
// ILLEGAL: var x;          (needs initializer)
// ILLEGAL: var x = null;   (null has no type)
// Only for local variables — NOT fields, parameters, or return types
```

**Text blocks (Java 15):**

```java
String json = """
        {
            "name": "Alice",
            "age": 30
        }
        """;
// Incidental whitespace stripped by position of closing """
```

**Records (Java 16) — immutable data carriers:**

```java
public record Point(int x, int y) {
    // Compiler generates: canonical constructor, x(), y(), equals(), hashCode(), toString()
    public Point {  // compact constructor for validation
        if (x < 0 || y < 0) throw new IllegalArgumentException("negative coordinate");
    }
    public double distance() { return Math.sqrt(x * x + y * y); }
}
// Records are final — cannot be subclassed
// Fields are private final — immutable
// equals/hashCode by value
Point p = new Point(3, 4);
p.x();   // accessor — NOT getX()
```

**Sealed classes (Java 17) — closed hierarchies:**

```java
public sealed interface Shape permits Circle, Rectangle, Triangle { }
public record Circle(double radius) implements Shape { }
public record Rectangle(double w, double h) implements Shape { }
public final class Triangle implements Shape { }

// Exhaustive switch — no default needed
double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.w() * r.h();
    case Triangle t  -> triangleArea(t);
};
```

**Pattern matching instanceof (Java 16):**

```java
// Old
if (obj instanceof String) { String s = (String) obj; System.out.println(s.length()); }

// Pattern matching — binding variable, no cast
if (obj instanceof String s && s.length() > 5) { System.out.println(s.toUpperCase()); }
```

**Follow-up questions the interviewer will ask:**

1. *"Can records extend classes?"* → No. Records implicitly extend `java.lang.Record` and are `final`. They can implement interfaces.

2. *"What is `non-sealed`?"* → A permitted subclass can declare itself `non-sealed`, reopening the hierarchy below it.

3. *"Is `var` a keyword?"* → No, it is a reserved type name — you can (but shouldn't) have a variable named `var`.

**Common mistakes that get you rejected:**
- Thinking `var` is dynamically typed.
- Trying to subclass a record.
- Forgetting sealed class permitted subtypes must be in the same module (or package in unnamed module).

---

### Q48: Java 21 Virtual Threads (Project Loom)
**Company:** Google, Amazon, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain Java 21 virtual threads. How do they differ from platform threads? What is a carrier thread? When should you NOT use virtual threads?

**What interviewer is testing:** Whether you understand the M:N threading model, scheduler mechanics, and where virtual threads help or hurt.

**Ideal Answer:**

```
Platform thread: 1:1 with OS thread
  Stack: ~1 MB default | Context switch: kernel mode (expensive)
  Blocking I/O: OS thread sits idle | Limit: ~thousands per JVM

Virtual thread: M:N — many virtual on few OS (carrier) threads
  Stack: starts ~1 KB, grows on heap | Context switch: JVM-managed (cheap)
  Blocking I/O: JVM unmounts virtual thread, mounts another | Limit: millions per JVM
```

**How the scheduler works:**

When a virtual thread calls a blocking op (socket read, `sleep`, `lock`), the JVM:
1. Captures the virtual thread's continuation (stack).
2. Unmounts it from the carrier thread.
3. Parks it until the blocking op completes.
4. Mounts another runnable virtual thread on the freed carrier.

```java
// Direct creation
Thread vt = Thread.ofVirtual().start(() -> handleRequest(request));

// Via ExecutorService (preferred for server workloads)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> processRequest(req));
    // Each task gets its own virtual thread
}

// Spring Boot 3.2+ — in application.yml:
// spring.threads.virtual.enabled=true
// All Tomcat request threads become virtual threads automatically
```

**Structured Concurrency (Java 21 preview):**

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<User>   user   = scope.fork(() -> fetchUser(id));
    Future<Orders> orders = scope.fork(() -> fetchOrders(id));
    scope.join().throwIfFailed();
    return new Dashboard(user.resultNow(), orders.resultNow());
}
// If either subtask fails, the other is cancelled automatically.
```

**When NOT to use virtual threads:**

```java
// 1. CPU-bound work — virtual threads don't help; carrier stays pinned during computation
// 2. synchronized blocks/methods — pin the virtual thread to its carrier (carrier cannot be freed)
synchronized (lock) { /* pins carrier */ }
// Fix: use ReentrantLock instead — does NOT pin
lock.lock(); try { /* safe */ } finally { lock.unlock(); }

// 3. ThreadLocal misuse — millions of virtual threads × ThreadLocal values = memory pressure
// Prefer ScopedValue (Java 21 preview)
```

**Follow-up questions the interviewer will ask:**

1. *"Does virtual thread = faster code?"* → Not for CPU-bound work. Benefit is throughput under I/O-bound load — same hardware handles far more concurrent requests.

2. *"What is pinning and how do you detect it?"* → JVM flag `-Djdk.tracePinnedThreads=full` logs pinning events. Occurs inside `synchronized` and native method frames.

3. *"How does Tomcat use virtual threads in Spring Boot 3.2?"* → `TomcatProtocolHandlerCustomizer` replaces the executor with `newVirtualThreadPerTaskExecutor()`. Each HTTP request gets its own virtual thread.

**Common mistakes that get you rejected:**
- Saying virtual threads replace reactive programming for CPU-bound code — they don't.
- Forgetting that `synchronized` pins carriers — a library heavy on `synchronized` negates the benefit.
- Confusing virtual threads with Kotlin coroutines — similar concept, different implementation.

---

### Q49: Java 21 Pattern Matching for Switch
**Company:** Google, Amazon  **Difficulty:** 🟡 Medium  **Frequency:** 🔥  **Round:** Onsite

**Question:** Explain pattern matching for `switch` in Java 21. How does it handle exhaustiveness, guards, and null?

**What interviewer is testing:** Familiarity with modern Java language features and sealed type integration.

**Ideal Answer:**

```java
// Type patterns in switch
static String format(Object obj) {
    return switch (obj) {
        case Integer i -> "int: " + i;
        case Long l    -> "long: " + l;
        case Double d  -> "double: " + String.format("%.2f", d);
        case String s  -> "string: \"" + s + "\"";
        case null      -> "null";           // explicit null handling (Java 21)
        default        -> "other: " + obj;
    };
}

// Guarded patterns (when clause)
static String classify(Object obj) {
    return switch (obj) {
        case Integer i when i < 0  -> "negative int";
        case Integer i when i == 0 -> "zero";
        case Integer i             -> "positive int";
        default                    -> "not an int";
    };
}
```

**Exhaustiveness with sealed types — no default needed:**

```java
sealed interface Expr permits Num, Add, Mul { }
record Num(int value) implements Expr { }
record Add(Expr l, Expr r) implements Expr { }
record Mul(Expr l, Expr r) implements Expr { }

int eval(Expr e) {
    return switch (e) {
        case Num n -> n.value();
        case Add a -> eval(a.l()) + eval(a.r());
        case Mul m -> eval(m.l()) * eval(m.r());
        // No default — sealed hierarchy is exhaustive; compiler verifies
        // Adding a new permitted type without a case = compile error
    };
}
```

**Null handling:**

```java
// Pre-Java 21: switch throws NullPointerException if obj is null
// Java 21: add explicit null case to prevent NPE
switch (obj) {
    case null   -> handleNull();
    case String s -> handleString(s);
    default     -> handleOther(obj);
}
```

**Follow-up questions the interviewer will ask:**

1. *"Can you deconstruct records in switch?"* → Yes (Java 21, finalized): `case Point(int x, int y) when x == y -> "diagonal"`.

2. *"What happens if a switch over a sealed type is non-exhaustive?"* → Compile error — compiler knows all permitted subtypes.

3. *"Difference between pattern switch and if-else instanceof chain?"* → Functionally equivalent, but switch is exhaustiveness-checked with sealed types, more readable, and JVM-optimizable.

**Common mistakes that get you rejected:**
- Putting a catch-all before specific patterns — dominance violation, compile error.
- Forgetting null handling — pre-21 switch NPEs on null input.

---

### Q50: CompletableFuture Advanced — handle, allOf, timeouts
**Company:** Amazon, Google, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain `handle()` vs `exceptionally()`, `allOf()` vs `anyOf()`, and how to add timeouts to a CompletableFuture.

**What interviewer is testing:** Production-grade async error handling and composition.

**Ideal Answer:**

**Error handling — three operators:**

```java
CompletableFuture<String> cf = supplyAsync(() -> fetchData());

// exceptionally — handles exception only, passes success through
cf.exceptionally(ex -> { log.error("failed", ex); return "fallback"; });

// handle — runs on BOTH success and failure
cf.handle((result, ex) -> {
    if (ex != null) { log.error("failed", ex); return "fallback"; }
    return result.toUpperCase();
});

// whenComplete — observe without transforming (cannot change the value)
cf.whenComplete((result, ex) -> {
    if (ex != null) metrics.recordFailure(); else metrics.recordSuccess();
});
```

**allOf vs anyOf:**

```java
CompletableFuture<String> f1 = supplyAsync(() -> fetchA());
CompletableFuture<String> f2 = supplyAsync(() -> fetchB());
CompletableFuture<String> f3 = supplyAsync(() -> fetchC());

// allOf: completes when ALL complete; returns CompletableFuture<Void>
CompletableFuture.allOf(f1, f2, f3).thenRun(() -> {
    String a = f1.join(); String b = f2.join(); String c = f3.join();
    combine(a, b, c);
});

// Typed allOf — collect to list:
List<CompletableFuture<String>> futures = List.of(f1, f2, f3);
CompletableFuture<List<String>> allResults = CompletableFuture
    .allOf(futures.toArray(new CompletableFuture[0]))
    .thenApply(v -> futures.stream().map(CompletableFuture::join).collect(toList()));

// anyOf: completes when FIRST completes; returns CompletableFuture<Object>
CompletableFuture.anyOf(f1, f2, f3).thenAccept(result -> handleFastest((String) result));
// Note: other futures keep running — no automatic cancellation
```

**Timeouts (Java 9+):**

```java
// orTimeout: completes exceptionally with TimeoutException
fetchData().orTimeout(3, TimeUnit.SECONDS);

// completeOnTimeout: completes with fallback value instead of exception
fetchData().completeOnTimeout("default", 3, TimeUnit.SECONDS);

// Full production pattern:
CompletableFuture<Response> response = supplyAsync(() -> callService(), ioPool)
    .completeOnTimeout(Response.empty(), 2, SECONDS)
    .handle((res, ex) -> {
        if (ex != null) { log.error("service call failed", ex); return Response.error(); }
        return res;
    });
```

**Follow-up questions the interviewer will ask:**

1. *"What thread executes the callback in `thenApply`?"* → If the previous stage is already complete when `thenApply` is registered, the calling thread runs it. Otherwise, the thread that completed it runs the callback. Use `thenApplyAsync` to always run on a specific executor.

2. *"Does `allOf` short-circuit on failure?"* → No — it waits for ALL, including failed ones. If one fails, `allOf` fails, but others keep running.

3. *"Difference between `join()` and `get()`?"* → Both block. `get()` throws checked `ExecutionException`/`InterruptedException`. `join()` throws unchecked `CompletionException`. In lambdas, `join()` is more convenient.

**Common mistakes that get you rejected:**
- Calling `join()` on the main/request thread, blocking it — defeats the purpose of async.
- Forgetting `anyOf` returns `CompletableFuture<Object>` — the unchecked cast is a smell.
- Not using `thenApplyAsync` when the callback is expensive.

---

## JVM Internals

### Q51: JVM Memory Areas
**Company:** Google, Goldman Sachs, Amazon  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Describe the JVM memory areas. Explain heap generations, Metaspace, and the anatomy of a stack frame.

**What interviewer is testing:** Whether you can reason about where objects live, how the GC divides the heap, and what the stack stores per method invocation.

**Ideal Answer:**

```
JVM Runtime Data Areas
├── Heap (shared across threads)
│   ├── Young Generation
│   │   ├── Eden Space         ← new objects allocated here
│   │   ├── Survivor 0 (S0)   ← survivors after minor GC
│   │   └── Survivor 1 (S1)   ← survivors alternate each minor GC
│   └── Old Generation (Tenured)  ← objects that survived enough minor GCs
│
├── Metaspace (off-heap, since Java 8 — replaces PermGen)
│   ├── Class metadata (bytecode, method tables)
│   ├── Static variables (reference stored in Class object on heap)
│   └── Interned strings (moved to heap in Java 7+)
│
├── JVM Stacks (one per thread)
│   └── Stack Frame (one per method call)
│       ├── Local Variable Array  ← method args + local vars
│       ├── Operand Stack         ← JVM's "scratchpad" for computation
│       └── Frame Data            ← reference to constant pool, return address
│
├── PC Register (one per thread) ← address of current bytecode instruction
├── Native Method Stack           ← for native (JNI) calls
└── Code Cache                    ← JIT-compiled native code
```

**Heap generation detail:**

Object lifecycle in generational GC:
1. Object allocated in **Eden**.
2. **Minor GC** (Young GC) fires when Eden fills — live objects copied to S0 or S1 (survivors alternate), dead objects collected cheaply.
3. Object's **age** increments each minor GC it survives.
4. When age reaches **tenuring threshold** (default 15), object **promoted** to Old Generation.
5. **Major GC** (Full GC) collects Old Generation — expensive, stop-the-world with older collectors.

**Why generational hypothesis works:** Most objects die young (temporaries, request-scoped data). Young GC only scans the small young generation — fast.

**Metaspace vs PermGen:**

```
PermGen (Java 7 and earlier):
  - Fixed max size (-XX:MaxPermSize)
  - Caused java.lang.OutOfMemoryError: PermGen space on heavy class loading
  - Stored class metadata + interned strings

Metaspace (Java 8+):
  - Off-heap (native memory) — grows dynamically
  - Default: unbounded (controlled via -XX:MaxMetaspaceSize)
  - OOM only if you exhaust native memory or set MaxMetaspaceSize
  - Common cause: classloader leak in app servers (dynamic proxies, Hibernate enhancers)
```

**Stack frame anatomy:**

```java
int add(int a, int b) {
    int result = a + b;  // local variable
    return result;
}
// Frame for add():
//   Local Variable Array: [this(0), a(1), b(2), result(3)]
//   Operand Stack: used during bytecode execution of "a + b"
```

**Follow-up questions the interviewer will ask:**

1. *"Where are static variables stored?"* → The reference is in the `Class` object (on the heap). Metaspace holds class metadata/bytecode, but object references live on the heap.

2. *"What causes StackOverflowError?"* → Unbounded recursion fills the JVM stack — each recursive call adds a frame until no space remains. Controlled by `-Xss` flag.

3. *"What is escape analysis?"* → JIT optimization: if an object does not escape the creating method, JVM can allocate it on the stack (stack allocation) instead of the heap, avoiding GC pressure entirely. Enabled by default since Java 6 (`-XX:+DoEscapeAnalysis`).

**Common mistakes that get you rejected:**
- Confusing PermGen and Metaspace — PermGen was removed in Java 8.
- Saying "static variables are stored in Metaspace" — the reference lives in the heap's Class object.
- Not knowing that each thread has its own stack.

---

### Q52: Garbage Collection Algorithms
**Company:** Google, Goldman Sachs, Amazon  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Compare Serial, Parallel, G1, ZGC, and Shenandoah garbage collectors. When would you choose each?

**What interviewer is testing:** Whether you understand the trade-offs — throughput vs pause time vs memory overhead — and can map a production scenario to the right collector.

**Ideal Answer:**

**The fundamental GC trade-off: throughput vs pause time**

```
HIGH THROUGHPUT ◄─────────────────────────────────────────► LOW PAUSES
Serial → Parallel → G1 → Shenandoah / ZGC
```

**1. Serial GC (`-XX:+UseSerialGC`)**
- Single-threaded, stop-the-world for both minor and major GC.
- Suitable only for: single-core VMs, tiny heaps (< 100 MB), embedded / client apps.

**2. Parallel GC (`-XX:+UseParallelGC`) — Java 8 default**
- Multiple GC threads, but still stop-the-world.
- Optimizes for **throughput** — maximizes work done per unit time.
- Good for: batch jobs, scientific computing where pauses are acceptable.
- Not good for: latency-sensitive services.

**3. G1 GC (`-XX:+UseG1GC`) — Java 9+ default**
- Heap divided into equal-sized **regions** (typically 1–32 MB each).
- Collects the regions with most garbage first ("Garbage First").
- **Concurrent marking** + short stop-the-world pauses for evacuation.
- Target pause: configurable via `-XX:MaxGCPauseMillis=200` (best-effort, not guaranteed).
- Suitable for: 4 GB – 32 GB heaps, general-purpose servers needing balanced throughput + latency.

```
G1 Heap Layout:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ E   │ S   │ Old │ E   │ Old │ H   │ E   │ Old │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
E=Eden  S=Survivor  Old=Tenured  H=Humongous (>0.5 region size)
```

**4. ZGC (`-XX:+UseZGC`) — production since Java 15**
- Concurrent compaction — almost all work done while app threads run.
- **Sub-millisecond pauses** regardless of heap size (tested up to 16 TB).
- Uses **colored pointers** (load barriers on pointer reads) to track object state concurrently.
- Memory overhead: ~6× higher memory bandwidth usage than G1.
- Suitable for: latency-critical services (p99 pause < 1 ms), very large heaps.

**5. Shenandoah (`-XX:+UseShenandoahGC`) — RedHat, available in OpenJDK 12+**
- Also concurrent compaction with very low pauses.
- Uses **Brooks forwarding pointers** (extra word per object) instead of colored pointers.
- Similar goals to ZGC, slightly different design; generally comparable pause times.

**Decision matrix:**

```
Scenario                              Recommended GC
────────────────────────────────────  ──────────────
Batch job, throughput matters most    Parallel GC
General server, heap 4–32 GB          G1 (default)
Latency-critical API (p99 < 10 ms)    ZGC or Shenandoah
Very large heap (> 32 GB)             ZGC or Shenandoah
Embedded / tiny heap                  Serial GC
```

**Follow-up questions the interviewer will ask:**

1. *"What is a Humongous object in G1?"* → An object larger than 50% of a G1 region. Allocated directly in the Old Generation, skipping Young GC. Can cause fragmentation; tune region size with `-XX:G1HeapRegionSize`.

2. *"What is a mixed collection in G1?"* → After concurrent marking, G1 runs mixed GCs that collect both Young and selected Old regions (those with most garbage) to reclaim Old generation space incrementally.

3. *"How does ZGC avoid stop-the-world for compaction?"* → Colored pointers encode GC metadata (mark, relocation bits) directly in pointer bits. Load barriers remap pointers at object access time — so the app never sees an invalid reference even as GC moves objects concurrently.

**Common mistakes that get you rejected:**
- Saying "G1 has zero pauses" — G1 still has stop-the-world phases (initial mark, remark).
- Not knowing G1 is the Java 9+ default (not Parallel).
- Recommending ZGC for a batch job — its memory overhead is wasted when throughput is the goal.

---

### Q53: GC Tuning Flags and OOM Types
**Company:** Amazon, Flipkart, Goldman Sachs  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** What are the key GC tuning flags? How do you read a GC log? What are the types of OutOfMemoryError?

**What interviewer is testing:** Practical ability to diagnose and tune a production JVM — not just knowing flags exist, but knowing what they control and how to read telemetry.

**Ideal Answer:**

**Key JVM flags:**

```bash
# Heap sizing
-Xms2g                    # initial heap size (set equal to Xmx to avoid resize pauses)
-Xmx2g                    # max heap size
-Xss512k                  # stack size per thread (default 512k–1M)

# Collector selection
-XX:+UseG1GC              # G1 (default Java 9+)
-XX:+UseZGC               # ZGC (Java 15+ production)
-XX:MaxGCPauseMillis=200  # G1 pause target (best-effort)

# Young generation sizing
-XX:NewRatio=2            # Old:Young ratio (default 2 = 1/3 young, 2/3 old)
-XX:SurvivorRatio=8       # Eden:Survivor ratio (default 8 = 8/10 Eden, 1/10 each survivor)

# GC logging (Java 9+ unified logging)
-Xlog:gc*:file=/var/log/app/gc.log:time,uptime,level,tags:filecount=5,filesize=20m

# Heap dump on OOM
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/dumps/

# Metaspace cap
-XX:MaxMetaspaceSize=256m
```

**Reading a G1 GC log:**

```
[2.345s][info][gc] GC(42) Pause Young (Normal) (G1 Evacuation Pause) 512M->128M(2048M) 12.345ms
          │            │    │                  │                       │          │       │
          time         GC#  type               reason                  heap       max    pause
```

- `512M->128M`: heap used before → after the GC.
- `(2048M)`: total heap capacity.
- `12.345ms`: stop-the-world pause duration.

**Types of OutOfMemoryError:**

```
1. java.lang.OutOfMemoryError: Java heap space
   Cause: objects filling up the heap; GC cannot reclaim enough
   Fix: increase -Xmx, find memory leaks (heap dump + MAT)

2. java.lang.OutOfMemoryError: GC overhead limit exceeded
   Cause: JVM spending > 98% of time in GC, recovering < 2% heap
   Fix: same as above — heap too small or leak

3. java.lang.OutOfMemoryError: Metaspace
   Cause: class metadata fills Metaspace (classloader leak, too many dynamic proxies)
   Fix: -XX:MaxMetaspaceSize, find classloader leak

4. java.lang.OutOfMemoryError: unable to create native thread
   Cause: OS thread limit reached (-Xss too large × too many threads)
   Fix: reduce -Xss, reduce thread count, use virtual threads (Java 21)

5. java.lang.StackOverflowError (not OOM, but related)
   Cause: deep/infinite recursion fills the JVM stack
   Fix: fix recursion, increase -Xss (rarely)
```

**Follow-up questions the interviewer will ask:**

1. *"How do you set heap to a fixed size and why?"* → Set `-Xms` == `-Xmx`. Prevents JVM from spending time resizing the heap (which requires GC cycles) and avoids unpredictable pause spikes in production.

2. *"What tool do you use to analyze GC logs?"* → GCEasy.io (web UI), GCViewer (desktop), or JDK Mission Control. Look for: frequency of full GCs, pause duration trends, heap utilization after GC (indicates leak if rising).

3. *"What causes GC overhead limit exceeded?"* → Heap is nearly full but objects are all live (a real leak) — GC keeps running and making no progress. The JVM throws this error before it completely runs out of heap.

**Common mistakes that get you rejected:**
- Setting `-Xms` much lower than `-Xmx` in production — causes unnecessary resize GCs.
- Not having `-XX:+HeapDumpOnOutOfMemoryError` in production — you lose the evidence.
- Confusing Metaspace OOM with heap OOM — different cause and fix.

---

### Q54: JIT Compilation — C1, C2, Tiered Compilation
**Company:** Google, Goldman Sachs  **Difficulty:** 🔴 Hard  **Frequency:** 🔥  **Round:** Onsite

**Question:** Explain JIT compilation in the JVM. What are C1 and C2 compilers? What is tiered compilation and OSR? When does the JVM deoptimize?

**What interviewer is testing:** Deep JVM internals — whether you understand how bytecode becomes native code and the speculative optimizations that make Java fast.

**Ideal Answer:**

The JVM starts in **interpreted mode** (fast startup, slow execution). The JIT compiler identifies **hot spots** (methods called frequently) and compiles them to native machine code.

**Two JIT compilers:**

```
C1 (Client compiler):
  - Fast compilation, minimal optimization
  - Good for: startup, short-lived code
  - Produces: lightly optimized native code quickly

C2 (Server compiler):
  - Slow compilation, aggressive optimization (inlining, loop unrolling, escape analysis)
  - Good for: long-running services (peak throughput)
  - Produces: highly optimized native code
```

**Tiered compilation (Java 8+ default, `-XX:+TieredCompilation`):**

```
Level 0: Interpreted
Level 1: C1 (no profiling)
Level 2: C1 (limited profiling)
Level 3: C1 (full profiling) ← most code reaches here initially
Level 4: C2 (fully optimized) ← hot methods eventually promoted here
```

Methods start interpreted, move to C1 quickly (fast startup), and hot methods eventually get promoted to C2 for peak performance. Best of both worlds.

**On-Stack Replacement (OSR):**

If a method has a long-running loop, the JVM can replace the currently executing interpreted frame with a compiled frame **while the loop is still running** — no need to wait for the next invocation. This is OSR.

**JIT optimizations (key ones):**

```
Inlining:        small hot methods are inlined at call site — eliminates call overhead
Escape analysis: objects that don't escape → stack-allocated or scalar-replaced (no GC)
Loop unrolling:  replicate loop body N times, reduce branch overhead
Devirtualization: monomorphic call site → direct call instead of vtable dispatch
Dead code elim:  unreachable branches removed
```

**Deoptimization — when JIT bets wrong:**

JIT uses **speculative optimizations** based on profiling data. If assumptions are violated, the JVM **deoptimizes** — falls back to interpreter, recompiles with less aggressive assumptions.

```
Triggers for deoptimization:
1. Class loading: a new subclass is loaded, breaking "only one implementor" assumption
   → devirtualization invalidated → deoptimize
2. Uncommon trap: a branch assumed "never taken" is hit → deoptimize + recompile
3. CHA (Class Hierarchy Analysis): hierarchy changes at runtime
```

```bash
# Flags to observe JIT behavior
-XX:+PrintCompilation          # log each compiled method
-XX:+UnlockDiagnosticVMOptions
-XX:+PrintInlining             # show inlining decisions
-XX:CompileThreshold=10000     # default hotspot threshold (method calls before C2)
```

**Follow-up questions the interviewer will ask:**

1. *"What is the -server flag?"* → In older JVMs, `-server` selected C2 only (no tiered). In Java 8+, tiered compilation is the default and `-server` is largely a no-op.

2. *"What is intrinsification?"* → Some methods (e.g., `System.arraycopy`, `Math.sqrt`, `String.equals`) are replaced by hand-tuned native instructions by the JIT — faster than what C2 would generate.

3. *"How does escape analysis help GC?"* → If the JIT proves an object does not escape its creating method, it can allocate it on the stack (stack allocation) or decompose it into scalar fields (scalar replacement). No heap allocation = no GC pressure.

**Common mistakes that get you rejected:**
- Thinking Java is always slow because it's interpreted — modern C2 JIT produces code competitive with C++.
- Confusing bytecode compilation (javac, compile time) with JIT compilation (JVM, runtime).
- Not knowing deoptimization exists — this is what causes "JVM warmup" effects in production.

---

### Q55: Class Loading — Delegation Model and Custom ClassLoaders
**Company:** Google, Amazon, Goldman Sachs  **Difficulty:** 🔴 Hard  **Frequency:** 🔥  **Round:** Onsite

**Question:** Explain the ClassLoader hierarchy and delegation model. When would you write a custom ClassLoader?

**What interviewer is testing:** Deep understanding of how the JVM resolves and loads classes — essential for app server and framework developers, and for diagnosing ClassNotFoundException vs NoClassDefFoundError.

**Ideal Answer:**

**ClassLoader hierarchy (Java 9+ modules, but same principle):**

```
Bootstrap ClassLoader (C++ code — not a Java object)
  └── loads: java.lang.*, java.util.*, rt.jar (java.base module)
      │
      Platform ClassLoader (Java 9+ — formerly Extension ClassLoader)
        └── loads: java.se module group, endorsed standards
            │
            Application (App) ClassLoader
              └── loads: classpath classes — your application code
                  │
                  [Your Custom ClassLoader] (optional)
```

**Parent delegation model:**

When a ClassLoader is asked to load class `com.example.Foo`:
1. **Delegates to parent first** (recurses up to Bootstrap).
2. If parent cannot find it (throws `ClassNotFoundException`), the child tries to load it from its own sources.
3. If still not found → `ClassNotFoundException` propagates up.

**Why delegation?**
- Security: core classes (`java.lang.String`) always loaded by Bootstrap — malicious code cannot replace them.
- Consistency: only one copy of a class per ClassLoader chain.

**ClassNotFoundException vs NoClassDefFoundError:**

```
ClassNotFoundException: thrown at runtime when Class.forName() or
  ClassLoader.loadClass() cannot find the class — class not on classpath.

NoClassDefFoundError: the class was present at compile time but is missing
  at runtime — dependency present during build but not at runtime (e.g., missing JAR).
  Also thrown if a class's static initializer threw an exception — the class is
  marked "failed" and subsequent loads throw NoClassDefFoundError.
```

**Custom ClassLoader use cases and implementation:**

```java
// Use cases:
// 1. Load classes from a database, network, or encrypted JAR
// 2. Hot reload: load new version of a class at runtime (different ClassLoader instance)
// 3. Isolation: OSGi, app servers loading plugins with separate class namespaces
// 4. Bytecode transformation: Instrumentation agents (ASM, Byte Buddy)

public class EncryptedClassLoader extends ClassLoader {
    private final Path classDir;

    public EncryptedClassLoader(Path classDir, ClassLoader parent) {
        super(parent);   // always set parent for delegation
        this.classDir = classDir;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            // Convert class name to file path
            Path classFile = classDir.resolve(name.replace('.', '/') + ".class");
            byte[] classBytes = decrypt(Files.readAllBytes(classFile));
            // defineClass converts raw bytes → Class object, registers with this loader
            return defineClass(name, classBytes, 0, classBytes.length);
        } catch (IOException e) {
            throw new ClassNotFoundException(name, e);
        }
    }

    private byte[] decrypt(byte[] encrypted) { /* XOR / AES decrypt */ return encrypted; }
}

// Usage
ClassLoader loader = new EncryptedClassLoader(Paths.get("/secure"), getClass().getClassLoader());
Class<?> clazz = loader.loadClass("com.example.SecureService");
Object instance = clazz.getDeclaredConstructor().newInstance();
```

**Hot reload pattern (break delegation for specific package):**

```java
@Override
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    // Break delegation for plugin classes — load fresh each time
    if (name.startsWith("com.plugin.")) {
        return findClass(name);   // skip parent — load from our source
    }
    return super.loadClass(name, resolve);  // delegate for everything else
}
```

**Follow-up questions the interviewer will ask:**

1. *"What is a classloader leak in a web application?"* → If a ClassLoader is not GC'd when an app is undeployed (e.g., a thread holding a reference to a class it loaded, or a static ThreadLocal), Metaspace fills up with stale class metadata. Diagnosis: heap dump, look for `ClassLoader` instances with unexpected retention.

2. *"What is `Class.forName` vs `ClassLoader.loadClass`?"* → `Class.forName` runs the class's static initializer block. `ClassLoader.loadClass` just loads without initializing. Use `Class.forName` when you need the static initializer to run (e.g., JDBC driver registration).

3. *"How does OSGi use ClassLoaders?"* → Each bundle (JAR) gets its own ClassLoader. Bundle A's ClassLoader only sees its own classes + explicitly imported packages. This provides strict isolation between plugins — different versions of the same library can coexist in one JVM.

**Common mistakes that get you rejected:**
- Not calling `super(parent)` in a custom ClassLoader — breaks delegation and security model.
- Confusing ClassNotFoundException with NoClassDefFoundError — different root causes, different fixes.
- Not knowing that two classes loaded by different ClassLoaders are considered different types — casts between them throw ClassCastException.

---

## Design Patterns in Java

### Q56: Singleton — enum, DCL, static inner class
**Company:** Amazon, Microsoft, Adobe, Goldman Sachs  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Implement Singleton three ways: enum singleton, double-checked locking, and static inner class. Why is `volatile` needed in DCL? Why is enum singleton best?

**What interviewer is testing:** Deep understanding of thread safety, the Java Memory Model, and idiomatic Java design — not just "make the constructor private."

**Ideal Answer:**

**1. Enum singleton (best — Joshua Bloch, Effective Java Item 3):**

```java
public enum DatabaseConnection {
    INSTANCE;

    private final Connection connection;

    DatabaseConnection() {
        // Enum constructor called exactly once by JVM class loading
        this.connection = createConnection();
    }

    public Connection get() { return connection; }
    private Connection createConnection() { /* ... */ return null; }
}

// Usage
DatabaseConnection.INSTANCE.get();
```

Why enum is best:
- JVM guarantees instantiation exactly once (class loading is thread-safe).
- **Serialization-safe**: enum serialization preserves the singleton invariant — `readResolve()` not needed.
- **Reflection-safe**: `Constructor.newInstance()` on an enum throws `IllegalArgumentException`.
- Zero boilerplate.

**2. Double-Checked Locking (DCL) — why `volatile` is required:**

```java
public class Singleton {
    // volatile is MANDATORY — without it, DCL is broken on the JMM
    private static volatile Singleton instance;

    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {                    // first check (no lock — fast path)
            synchronized (Singleton.class) {
                if (instance == null) {            // second check (under lock)
                    instance = new Singleton();    // write
                }
            }
        }
        return instance;                           // read
    }
}
```

**Why `volatile` is mandatory:**

`instance = new Singleton()` is NOT atomic. The JVM may reorder it as:
1. Allocate memory
2. **Write reference to `instance`** ← CPU may reorder this before step 3
3. Execute constructor body

Without `volatile`, Thread B can see a non-null `instance` (step 2 done) but with an incompletely initialized object (step 3 not yet visible to B). `volatile` adds a **happens-before** edge: the write to `instance` happens-before any subsequent read — constructor completion is visible.

**3. Static inner class (Initialization-on-demand holder):**

```java
public class Singleton {
    private Singleton() { }

    // Inner class loaded only when getInstance() is first called
    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;   // class loading is thread-safe by JVM spec
    }
}
```

Why it works: Java class loading is synchronized by the JVM. `Holder` is not loaded until `getInstance()` is called. When it IS loaded, `INSTANCE` is initialized exactly once in a thread-safe manner — no explicit synchronization needed.

Drawback vs enum: not serialization-safe without `readResolve()`.

**Follow-up questions the interviewer will ask:**

1. *"What's the problem with eager initialization?"* → `private static final Singleton INSTANCE = new Singleton()` — simple and thread-safe, but the instance is created at class load time even if it is never used. Fine for most cases; problem only if initialization is expensive and the class might not be used.

2. *"Can reflection break the singleton?"* → Yes for class-based singletons — `Constructor.setAccessible(true)` bypasses the private constructor. Fix: throw `IllegalStateException` from the constructor if the instance already exists. Enum is immune.

3. *"Can serialization break the singleton?"* → Yes for class-based — `readObject` creates a new instance. Fix: add `protected Object readResolve() { return INSTANCE; }`. Enum is immune.

**Common mistakes that get you rejected:**
- DCL without `volatile` — broken on modern CPUs due to instruction reordering.
- Not knowing WHY `volatile` is needed (the memory model explanation is the signal).
- Saying "synchronized getInstance()" — works but serializes every access, not just initialization.

---

### Q57: Factory Method vs Abstract Factory vs Builder
**Company:** Google, Amazon, Microsoft  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Implement Factory Method, Abstract Factory, and Builder for the same domain. When do you use each?

**What interviewer is testing:** Whether you can distinguish the three creational patterns, implement them correctly, and articulate the trade-offs — not just recite GoF definitions.

**Ideal Answer:**

Domain: creating `Notification` objects (Email, SMS, Push).

**1. Factory Method — subclass decides which class to instantiate:**

```java
// Creator with factory method
abstract class NotificationSender {
    // Factory method — subclasses override to create the right product
    protected abstract Notification createNotification(String message);

    public void send(String message) {
        Notification n = createNotification(message);   // call factory method
        n.prepare();
        n.deliver();
    }
}

class EmailSender extends NotificationSender {
    @Override
    protected Notification createNotification(String message) {
        return new EmailNotification(message);
    }
}

class SmsSender extends NotificationSender {
    @Override
    protected Notification createNotification(String message) {
        return new SmsNotification(message);
    }
}
// Client picks a concrete sender; the sender handles product creation.
// Use when: you want subclasses to control which concrete type gets created.
```

**2. Abstract Factory — family of related objects:**

```java
// Abstract factory: creates a family of related products
interface NotificationFactory {
    Notification createNotification(String message);
    NotificationLogger createLogger();   // related product
}

class EmailNotificationFactory implements NotificationFactory {
    public Notification createNotification(String msg) { return new EmailNotification(msg); }
    public NotificationLogger createLogger() { return new EmailLogger(); }
}

class SmsNotificationFactory implements NotificationFactory {
    public Notification createNotification(String msg) { return new SmsNotification(msg); }
    public NotificationLogger createLogger() { return new SmsLogger(); }
}

// Client works with the factory abstraction — no concrete classes
class NotificationService {
    private final NotificationFactory factory;
    NotificationService(NotificationFactory factory) { this.factory = factory; }

    public void send(String message) {
        Notification n  = factory.createNotification(message);
        NotificationLogger l = factory.createLogger();
        n.deliver(); l.log(message);
    }
}
// Use when: you need to ensure products from the same family are used together.
```

**3. Builder — step-by-step construction of complex objects:**

```java
public class EmailNotification {
    private final String recipient;   // required
    private final String subject;     // required
    private final String body;        // optional
    private final String cc;          // optional
    private final boolean htmlEnabled;// optional

    private EmailNotification(Builder b) {
        this.recipient   = b.recipient;
        this.subject     = b.subject;
        this.body        = b.body;
        this.cc          = b.cc;
        this.htmlEnabled = b.htmlEnabled;
    }

    public static class Builder {
        private final String recipient;  // required — in constructor
        private final String subject;
        private String body = "";
        private String cc   = "";
        private boolean htmlEnabled = false;

        public Builder(String recipient, String subject) {
            this.recipient = recipient;
            this.subject   = subject;
        }
        public Builder body(String body)             { this.body = body; return this; }
        public Builder cc(String cc)                 { this.cc = cc; return this; }
        public Builder htmlEnabled(boolean e)        { this.htmlEnabled = e; return this; }
        public EmailNotification build()             { return new EmailNotification(this); }
    }
}

// Usage — fluent, readable, no telescoping constructors
EmailNotification email = new EmailNotification.Builder("bob@example.com", "Hello")
    .body("Meeting at 3pm")
    .htmlEnabled(true)
    .build();
```

**When to use each:**

```
Factory Method:  one product, subclass controls which concrete type
Abstract Factory: family of related products, ensure consistency within family
Builder:         one complex object with many optional fields, step-by-step construction
```

**Follow-up questions the interviewer will ask:**

1. *"Java 8 shortcut for Factory Method?"* → Use a `Function<String, Notification>` or a method reference as the factory: `Function<String, Notification> factory = EmailNotification::new;`. Simple cases don't need a full abstract class.

2. *"Builder vs constructor with many params?"* → Telescoping constructors (many overloads) or constructors with many parameters are hard to read and error-prone (`new User("Alice", null, null, 30, true)`). Builder makes optional fields explicit and readable, and the object can be validated in `build()`.

3. *"How does Lombok's `@Builder` relate?"* → Lombok generates the Builder pattern code at compile time — the final result is exactly what you'd write by hand. It doesn't change the design pattern, just removes boilerplate.

**Common mistakes that get you rejected:**
- Confusing Factory Method (inheritance-based) with Abstract Factory (composition-based).
- Builder with mutable fields — the built object should be immutable (fields `final`).
- Not making required fields part of the Builder constructor (they can be silently omitted).

---

### Q58: Observer / Event-Driven in Java
**Company:** Amazon, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Implement the Observer pattern in Java. Compare `java.util.Observer` (deprecated), `PropertyChangeListener`, and Spring's `ApplicationEvent`.

**What interviewer is testing:** Understanding of event-driven architecture and the idiomatic ways to implement it in Java and Spring.

**Ideal Answer:**

**Core Observer pattern:**

```java
// Subject (Observable)
public class OrderService {
    private final List<OrderEventListener> listeners = new CopyOnWriteArrayList<>();

    public void addListener(OrderEventListener listener) { listeners.add(listener); }
    public void removeListener(OrderEventListener listener) { listeners.remove(listener); }

    public void placeOrder(Order order) {
        // business logic
        saveOrder(order);
        // notify all observers
        OrderPlacedEvent event = new OrderPlacedEvent(this, order);
        listeners.forEach(l -> l.onOrderPlaced(event));
    }
}

@FunctionalInterface
public interface OrderEventListener {
    void onOrderPlaced(OrderPlacedEvent event);
}
```

**PropertyChangeListener (JavaBeans standard):**

```java
public class Portfolio {
    private final PropertyChangeSupport pcs = new PropertyChangeSupport(this);
    private double value;

    public void addPropertyChangeListener(PropertyChangeListener l) { pcs.addPropertyChangeListener(l); }

    public void setValue(double newValue) {
        double old = this.value;
        this.value = newValue;
        pcs.firePropertyChange("value", old, newValue);  // notifies all listeners
    }
}

// Listener
portfolio.addPropertyChangeListener(e ->
    System.out.printf("Portfolio changed: %.2f → %.2f%n", e.getOldValue(), e.getNewValue()));
```

**Spring ApplicationEvent (preferred in Spring apps):**

```java
// Event
public class OrderPlacedEvent extends ApplicationEvent {
    private final Order order;
    public OrderPlacedEvent(Object source, Order order) { super(source); this.order = order; }
    public Order getOrder() { return order; }
}

// Publisher
@Service
public class OrderService {
    private final ApplicationEventPublisher publisher;

    public OrderService(ApplicationEventPublisher publisher) { this.publisher = publisher; }

    public void placeOrder(Order order) {
        saveOrder(order);
        publisher.publishEvent(new OrderPlacedEvent(this, order));
    }
}

// Listener — decoupled, in any Spring bean
@Component
public class EmailListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailSvc.send(event.getOrder().getCustomerEmail(), "Order confirmed!");
    }

    // Async listener — runs in separate thread pool
    @Async
    @EventListener
    public void onOrderPlacedAsync(OrderPlacedEvent event) { /* ... */ }

    // Transactional: fires only after the transaction that published the event commits
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onOrderPlacedAfterCommit(OrderPlacedEvent event) { /* send email */ }
}
```

**Why `@TransactionalEventListener(AFTER_COMMIT)` matters:**

If you publish an event inside a `@Transactional` method and the listener sends an email immediately, but the transaction then rolls back — the email was sent for an order that doesn't exist. `AFTER_COMMIT` delays listener invocation until the transaction commits successfully.

**Follow-up questions the interviewer will ask:**

1. *"What is the difference between `@EventListener` and `@TransactionalEventListener`?"* → `@EventListener` fires synchronously when `publishEvent` is called (within the same transaction). `@TransactionalEventListener(AFTER_COMMIT)` fires after the outer transaction commits — no transaction exists in the listener unless you create one with `@Transactional(REQUIRES_NEW)`.

2. *"How do you order multiple listeners for the same event?"* → Implement `Ordered` or use `@Order(1)` annotation on the listener. Lower value = higher priority.

3. *"Why was `java.util.Observer` deprecated in Java 9?"* → It was poorly designed: `update(Observable, Object)` uses raw `Object`, the `Observable` base class is mutable and thread-unsafe, and the design doesn't support lambda-friendly functional interfaces.

**Common mistakes that get you rejected:**
- Not using `@TransactionalEventListener(AFTER_COMMIT)` when the event implies a committed state — sending emails for rolled-back orders.
- `CopyOnWriteArrayList` for listeners (correct for low-write, high-read patterns) vs `ArrayList` (not thread-safe for concurrent registration).

---

### Q59: Strategy Pattern
**Company:** Amazon, Adobe, Microsoft  **Difficulty:** 🟢 Easy  **Frequency:** 🔥🔥  **Round:** Phone/Onsite

**Question:** Implement the Strategy pattern. How does Java 8 simplify it with functional interfaces?

**What interviewer is testing:** Ability to replace complex if/else chains with clean, extensible designs — and awareness that Java 8 lambdas make Strategy trivial.

**Ideal Answer:**

Strategy encapsulates an algorithm behind an interface, making algorithms interchangeable without changing the client.

**Classic implementation:**

```java
// Strategy interface
public interface PricingStrategy {
    double calculate(double basePrice, int quantity);
}

// Concrete strategies
public class RegularPricing implements PricingStrategy {
    public double calculate(double basePrice, int quantity) { return basePrice * quantity; }
}

public class BulkPricing implements PricingStrategy {
    public double calculate(double basePrice, int quantity) {
        double discount = quantity >= 10 ? 0.9 : 1.0;
        return basePrice * quantity * discount;
    }
}

// Context
public class OrderPricer {
    private PricingStrategy strategy;
    public OrderPricer(PricingStrategy strategy) { this.strategy = strategy; }
    public void setStrategy(PricingStrategy strategy) { this.strategy = strategy; }
    public double price(double base, int qty) { return strategy.calculate(base, qty); }
}
```

**Java 8 simplification — strategy IS a functional interface:**

```java
// PricingStrategy is a @FunctionalInterface — lambdas ARE strategies
@FunctionalInterface
public interface PricingStrategy {
    double calculate(double basePrice, int quantity);
}

// No concrete classes needed — lambdas inline the strategy
OrderPricer regular = new OrderPricer((price, qty) -> price * qty);
OrderPricer bulk     = new OrderPricer((price, qty) -> price * qty * (qty >= 10 ? 0.9 : 1.0));
OrderPricer vip      = new OrderPricer((price, qty) -> price * qty * 0.8);

// Or store strategies in a map for runtime selection
Map<String, PricingStrategy> strategies = Map.of(
    "REGULAR", (p, q) -> p * q,
    "BULK",    (p, q) -> p * q * (q >= 10 ? 0.9 : 1.0),
    "VIP",     (p, q) -> p * q * 0.8
);

// Selection at runtime — no if/else
PricingStrategy strategy = strategies.getOrDefault(customerType, strategies.get("REGULAR"));
double total = strategy.calculate(basePrice, quantity);
```

**vs if/else chain — why Strategy wins:**

```java
// ANTI-PATTERN: if/else chain that grows with every new pricing tier
double price;
if ("VIP".equals(type))     { price = base * qty * 0.8; }
else if ("BULK".equals(type)){ price = base * qty * 0.9; }
else                         { price = base * qty; }
// Adding a new tier requires modifying this method — violates Open/Closed Principle
```

**Follow-up questions the interviewer will ask:**

1. *"Strategy vs Template Method?"* → Strategy uses composition (inject the algorithm object); Template Method uses inheritance (define skeleton in base class, hook methods in subclass). Prefer composition (Strategy) — it is more flexible and testable.

2. *"How do you pick a strategy at runtime?"* → Map from a discriminator string/enum to the strategy instance. `strategies.get(customerType).calculate(...)`. Eliminates if/else entirely.

**Common mistakes that get you rejected:**
- Implementing Strategy with a switch statement in the context class — that's just an if/else with extra steps.
- Not recognizing that a `@FunctionalInterface` is a natural Strategy in Java 8+.

---

### Q60: Proxy Pattern — Static, JDK Dynamic, CGLIB
**Company:** Google, Amazon, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain static proxy vs JDK dynamic proxy vs CGLIB. Which does Spring use and when?

**What interviewer is testing:** Whether you understand how Spring AOP works under the hood — this is the foundation of `@Transactional`, `@Cacheable`, `@Async`, and security.

**Ideal Answer:**

**Static Proxy — compile-time, hand-written:**

```java
// Proxy wraps the real service, adds behavior before/after
public class LoggingUserService implements UserService {
    private final UserService delegate;

    public LoggingUserService(UserService delegate) { this.delegate = delegate; }

    @Override
    public User findById(Long id) {
        log.info("findById called: {}", id);
        User result = delegate.findById(id);
        log.info("findById returned: {}", result);
        return result;
    }
}
// Problem: must implement every method — brittle, scales poorly.
```

**JDK Dynamic Proxy — runtime, interface-based:**

```java
// Works ONLY for interfaces — uses java.lang.reflect.Proxy + InvocationHandler
UserService proxy = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class<?>[]{ UserService.class },  // must be interface(s)
    (proxyObj, method, args) -> {         // InvocationHandler
        log.info("Before: {}", method.getName());
        Object result = method.invoke(realService, args);  // delegate to real
        log.info("After: {}", method.getName());
        return result;
    });
```

**CGLIB (Code Generation Library) — subclass-based, no interface needed:**

```java
// CGLIB generates a subclass at runtime, overrides methods with interceptors
// Spring uses CGLIB when the bean has no interface
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(UserServiceImpl.class);  // concrete class — no interface needed
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    log.info("Before: {}", method.getName());
    Object result = proxy.invokeSuper(obj, args);
    log.info("After: {}", method.getName());
    return result;
});
UserServiceImpl proxied = (UserServiceImpl) enhancer.create();
```

**Spring AOP proxy decision:**

```
Bean implements at least one interface → JDK dynamic proxy (default)
Bean has no interface                   → CGLIB subclass proxy

Override with @EnableAspectJAutoProxy(proxyTargetClass = true) → always CGLIB
Spring Boot 2.x+ default               → CGLIB for all (even interfaces)
```

**Why this matters for @Transactional:**

```java
@Service
public class OrderService {
    @Transactional
    public void placeOrder(Order o) {
        saveOrder(o);
        this.sendEmail(o);       // SELF-INVOCATION — bypasses proxy!
    }

    @Transactional(propagation = REQUIRES_NEW)
    public void sendEmail(Order o) { /* ... */ }  // won't get its own transaction
}
// Fix: inject OrderService into itself, or extract sendEmail to a separate bean
```

The proxy intercepts calls FROM OUTSIDE the bean. `this.sendEmail()` is a direct method call — the proxy is not involved. This is the most common Spring AOP trap.

**Follow-up questions the interviewer will ask:**

1. *"Can CGLIB proxy a final class or final method?"* → No — CGLIB subclasses the class, so `final` prevents override. Spring throws `BeanCreationException` if it tries to proxy a final class. Solution: remove `final`, use interface, or use `@Scope("prototype")` without proxy.

2. *"What does `@EnableAspectJAutoProxy(proxyTargetClass = true)` do?"* → Forces Spring to use CGLIB for all proxies, even beans with interfaces. Spring Boot enables this by default since 2.0.

3. *"What is AspectJ weaving vs Spring AOP proxy?"* → Spring AOP is proxy-based (only intercepts Spring-managed beans, only public methods, no self-invocation). AspectJ weaves bytecode at compile time or load time — it can intercept any method call, including private methods and self-invocations, even in non-Spring code.

**Common mistakes that get you rejected:**
- Not knowing Spring defaults changed to CGLIB in Spring Boot 2.
- Thinking `@Transactional` works on private methods or self-invocations — it does not through proxy AOP.
- Confusing JDK proxy (interface) with CGLIB (subclass).

---

### Q61: Decorator vs Inheritance
**Company:** Amazon, Adobe  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain the Decorator pattern using Java I/O. When do you prefer composition (Decorator) over inheritance?

**What interviewer is testing:** Understanding of the composition-over-inheritance principle and how to recognize Decorator in the wild (Java I/O).

**Ideal Answer:**

Decorator wraps an object with the same interface, adding behavior without changing the original class or creating a subclass explosion.

**Java I/O — canonical Decorator example:**

```java
// Abstract component: InputStream
// Concrete component: FileInputStream
// Decorator base: FilterInputStream (wraps another InputStream)
// Concrete decorators: BufferedInputStream, DataInputStream, GZIPInputStream, CipherInputStream

InputStream raw        = new FileInputStream("data.gz");
InputStream gzipped    = new GZIPInputStream(raw);          // adds decompression
InputStream buffered   = new BufferedInputStream(gzipped);  // adds buffering
DataInputStream data   = new DataInputStream(buffered);     // adds typed reads

// Each decorator wraps the previous — behaviors stack
double value = data.readDouble();
```

**Custom Decorator — logging compression:**

```java
// Component interface
public interface DataProcessor {
    byte[] process(byte[] input);
}

// Concrete component
public class CompressionProcessor implements DataProcessor {
    public byte[] process(byte[] input) { return compress(input); }
}

// Abstract decorator — implements same interface, wraps a component
public abstract class DataProcessorDecorator implements DataProcessor {
    protected final DataProcessor wrapped;
    DataProcessorDecorator(DataProcessor wrapped) { this.wrapped = wrapped; }
}

// Concrete decorator — adds encryption on top of whatever is wrapped
public class EncryptionDecorator extends DataProcessorDecorator {
    EncryptionDecorator(DataProcessor wrapped) { super(wrapped); }

    @Override
    public byte[] process(byte[] input) {
        byte[] intermediate = wrapped.process(input);  // delegate first
        return encrypt(intermediate);                  // then add own behavior
    }
}

// Stack decorators:
DataProcessor pipeline = new EncryptionDecorator(
                            new CompressionProcessor());
// compress → then encrypt
```

**Inheritance explosion — why Decorator wins:**

```
Without Decorator (inheritance):
  InputStream
    ├── BufferedFileInputStream
    ├── CompressedFileInputStream
    ├── BufferedCompressedFileInputStream
    ├── EncryptedFileInputStream
    ├── BufferedEncryptedFileInputStream
    ├── CompressedEncryptedFileInputStream
    └── BufferedCompressedEncryptedFileInputStream   ← 2^N subclasses

With Decorator:
  Wrap a FileInputStream in any combination at runtime — O(N) classes, infinite combinations
```

**When to prefer composition over inheritance:**

1. You need to add behaviors independently and in combination.
2. You want to add behavior to a class you don't own (can't subclass).
3. Subclassing would cause a combinatorial explosion.
4. The new behavior is optional — some clients want it, some don't.

**Follow-up questions the interviewer will ask:**

1. *"Decorator vs Strategy?"* → Both use composition. Decorator adds to the same interface (wrapping calls). Strategy replaces the core algorithm (different implementation of the same job). Decorator layers behavior; Strategy swaps behavior.

2. *"Decorator vs Proxy?"* → Both wrap an object. Proxy controls access to the real object (lazy init, access control, logging on the proxy side). Decorator transparently adds behavior. Subtle distinction: proxy manages the lifecycle of the real object; Decorator receives the real object from outside.

**Common mistakes that get you rejected:**
- Implementing Decorator with inheritance instead of composition — defeats the point.
- Not implementing the same interface as the component — breaks the is-a substitutability.

---

### Q62: Command Pattern
**Company:** Microsoft, Amazon  **Difficulty:** 🟡 Medium  **Frequency:** 🔥  **Round:** Onsite

**Question:** Implement the Command pattern. How does it enable undo/redo and job queuing?

**What interviewer is testing:** Understanding of encapsulating operations as objects — enabling queuing, logging, and reversibility.

**Ideal Answer:**

Command encapsulates a request as an object — decoupling the sender from the receiver and making requests first-class objects that can be stored, queued, logged, and reversed.

```java
// Command interface
public interface Command {
    void execute();
    void undo();
}

// Receiver — the object that knows how to do the work
public class TextEditor {
    private final StringBuilder text = new StringBuilder();

    public void insert(int pos, String str) { text.insert(pos, str); }
    public void delete(int pos, int len)    { text.delete(pos, pos + len); }
    public String getText()                 { return text.toString(); }
}

// Concrete command — encapsulates all state needed to execute AND undo
public class InsertCommand implements Command {
    private final TextEditor editor;
    private final int position;
    private final String text;

    public InsertCommand(TextEditor editor, int position, String text) {
        this.editor   = editor;
        this.position = position;
        this.text     = text;
    }

    @Override public void execute() { editor.insert(position, text); }
    @Override public void undo()    { editor.delete(position, text.length()); }
}

// Invoker — manages command history for undo/redo
public class CommandManager {
    private final Deque<Command> history = new ArrayDeque<>();
    private final Deque<Command> redoStack = new ArrayDeque<>();

    public void execute(Command cmd) {
        cmd.execute();
        history.push(cmd);
        redoStack.clear();   // new command invalidates redo history
    }

    public void undo() {
        if (!history.isEmpty()) {
            Command cmd = history.pop();
            cmd.undo();
            redoStack.push(cmd);
        }
    }

    public void redo() {
        if (!redoStack.isEmpty()) {
            Command cmd = redoStack.pop();
            cmd.execute();
            history.push(cmd);
        }
    }
}
```

**Job queue use case:**

```java
// Commands as tasks in a thread pool — fully decoupled scheduling from execution
BlockingQueue<Command> queue = new LinkedBlockingQueue<>();

// Worker thread
Thread worker = new Thread(() -> {
    try {
        while (!Thread.currentThread().isInterrupted()) {
            Command cmd = queue.take();  // blocks until work arrives
            cmd.execute();
        }
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
});

// Producer
queue.put(new SendEmailCommand(emailService, order));
queue.put(new GenerateInvoiceCommand(invoiceService, order));
// Worker executes them without knowing what they do
```

**Follow-up questions the interviewer will ask:**

1. *"How is Command different from Strategy?"* → Strategy defines HOW to do something (algorithm); Command defines WHAT to do (an action with enough context to do and undo it). Strategy is about behavior variation; Command is about encapsulating and deferring requests.

2. *"How does Spring Batch use Command?"* → Each `Step` in a Job is effectively a Command — it has `execute()` semantics and Spring Batch manages retries, restartability, and state (analogous to undo info).

3. *"Macro commands?"* → A `MacroCommand` holds a list of `Command` objects and executes/undoes them in sequence. E.g., "format document" = [spellCheck, paginate, renderHeaders] as a single undoable command.

**Common mistakes that get you rejected:**
- Not implementing `undo()` — without it the pattern is just a Runnable.
- Storing mutable state by reference in the command — if the receiver changes between execute and undo, undo may have wrong data.

---

## Spring Core & Boot Internals

### Q63: IoC and Dependency Injection
**Company:** Amazon, Microsoft, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Explain IoC and DI. Compare constructor, setter, and field injection. Which does Spring prefer and why? How does Spring resolve `@Autowired` when there are multiple candidates?

**What interviewer is testing:** Understanding of the core Spring programming model — not just "Spring manages beans" but the resolution algorithm and the reasons to prefer constructor injection.

**Ideal Answer:**

**IoC (Inversion of Control):** Instead of your code creating its dependencies, a container creates and injects them. Control of object creation is inverted — from the class to the container.

**DI (Dependency Injection):** The mechanism IoC uses — dependencies are pushed into objects rather than pulled by them.

**Three injection styles:**

```java
// 1. Constructor injection (PREFERRED by Spring team)
@Service
public class OrderService {
    private final PaymentGateway gateway;
    private final EmailService emailService;

    // Spring infers @Autowired with single constructor (Spring 4.3+)
    public OrderService(PaymentGateway gateway, EmailService emailService) {
        this.gateway = gateway;
        this.emailService = emailService;
    }
}
// Why preferred:
// - Dependencies are final — immutable, thread-safe
// - Class cannot be instantiated without all required deps — fail-fast
// - Catches circular dependencies at startup (not silently at runtime)
// - No reflection — works without Spring (plain unit test with new)
// - Works with @Validated constructor parameter constraints

// 2. Setter injection — for OPTIONAL dependencies
@Service
public class ReportService {
    private EmailService emailService;

    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
// Use when: the dependency is optional, or for circular dependency workaround (rare)

// 3. Field injection (DISCOURAGED — Spring never recommends this)
@Service
public class UserService {
    @Autowired private UserRepository repo;  // hidden dependency, not final, no validation
}
// Problems: hides dependencies (can't see them without reading the class body),
// field cannot be final, requires Spring reflection to work — not testable without container
```

**@Autowired resolution order:**

```
Step 1: By TYPE — find all beans assignable to the required type
        → if exactly one: inject it
        → if zero: throw NoSuchBeanDefinitionException (unless required=false)
        → if multiple: proceed to step 2

Step 2: By QUALIFIER — check for @Qualifier("name") annotation
        → if match found: inject it
        → if no @Qualifier: proceed to step 3

Step 3: By NAME — check if any bean name matches the field/parameter name
        → if match found: inject it
        → if still ambiguous: throw NoUniqueBeanDefinitionException
```

```java
// Multiple candidates — explicit @Qualifier
@Service
public class PaymentService {
    @Autowired
    @Qualifier("stripeGateway")         // selects by bean name/qualifier
    private PaymentGateway gateway;
}

// Or @Primary on the preferred implementation
@Primary
@Service("stripeGateway")
public class StripeGateway implements PaymentGateway { }
```

**Follow-up questions the interviewer will ask:**

1. *"What is the difference between `@Autowired`, `@Inject`, and `@Resource`?"* → `@Autowired` is Spring-specific, resolves by type then name. `@Inject` is JSR-330 (javax/jakarta), resolves by type — equivalent to `@Autowired`. `@Resource` is JSR-250, resolves by name first then type. Prefer `@Autowired` or constructor injection in Spring apps.

2. *"What is `@Primary` vs `@Qualifier`?"* → `@Primary` marks one bean as the default for its type — used when you want a sensible default without changing all injection points. `@Qualifier` lets each injection point explicitly name which bean it wants — more precise.

3. *"What happens with circular dependencies in constructor injection?"* → Spring throws `BeanCurrentlyInCreationException` at startup — fail-fast. With field injection, Spring can break the cycle by injecting a partially initialized proxy — the circular dependency silently exists but may cause `NullPointerException` at runtime. Constructor injection exposes the design smell early.

**Common mistakes that get you rejected:**
- Defending field injection — Spring's own documentation recommends against it.
- Not knowing the three-step resolution algorithm.
- Thinking `@Autowired` on a constructor is required (Spring 4.3+ infers it for single-constructor beans).

---

### Q64: Bean Lifecycle
**Company:** Google, Amazon, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Walk through the complete Spring bean lifecycle from instantiation to destruction.

**What interviewer is testing:** Depth on Spring internals — the 10+ step lifecycle is a litmus test for senior candidates.

**Ideal Answer:**

```
Spring Bean Lifecycle (singleton scope):

1.  BeanDefinition loaded (XML / @Component scan / @Bean)

2.  Instantiation
    → constructor called (or factory method)

3.  Populate properties
    → @Autowired / setter injection performed

4.  BeanNameAware.setBeanName(String name) called (if implemented)
5.  BeanClassLoaderAware.setBeanClassLoader(...) called (if implemented)
6.  BeanFactoryAware.setBeanFactory(...) called (if implemented)
7.  ApplicationContextAware.setApplicationContext(...) called (if implemented)

8.  BeanPostProcessor.postProcessBeforeInitialization()
    → ALL registered BPPs run (e.g., @Autowired processing, @Value injection)
    → This is where AOP proxies are NOT yet created

9.  @PostConstruct method called (annotation-driven init)
    (or InitializingBean.afterPropertiesSet() if implemented)
    (or init-method from @Bean(initMethod="...") config)

10. BeanPostProcessor.postProcessAfterInitialization()
    → This is where AOP proxies ARE created (AbstractAutoProxyCreator runs here)
    → The proxy replaces the original bean in the context

11. Bean is READY — injected into other beans, used by application

--- (application shutdown) ---

12. @PreDestroy method called
    (or DisposableBean.destroy() if implemented)
    (or destroy-method from @Bean(destroyMethod="...") config)
```

**Code showing lifecycle hooks:**

```java
@Component
public class DatabaseConnectionPool
    implements BeanNameAware, ApplicationContextAware, InitializingBean, DisposableBean {

    private String beanName;
    private ApplicationContext ctx;
    private HikariDataSource pool;

    @Override
    public void setBeanName(String name) {
        this.beanName = name;   // step 4
        log.info("Bean name: {}", name);
    }

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.ctx = ctx;   // step 7
    }

    @PostConstruct
    public void init() {
        pool = createPool();   // step 9 — after injection, before proxy creation
        log.info("Pool initialized: {}", pool);
    }

    @Override
    public void afterPropertiesSet() {
        // Also step 9 — called after @PostConstruct if both present
        validateConfig();
    }

    @PreDestroy
    public void shutdown() {
        pool.close();   // step 12
    }

    @Override
    public void destroy() {
        // Also step 12 — called after @PreDestroy if both present
    }
}
```

**BeanPostProcessor — how AOP proxies are created:**

```java
// BeanPostProcessor is called for EVERY bean — use it to instrument beans
@Component
public class TimingBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String name) {
        // Return a proxy wrapping the bean
        if (bean instanceof UserService) {
            return Proxy.newProxyInstance(...);  // AOP proxy created here
        }
        return bean;  // return original bean for others
    }
}
```

**Follow-up questions the interviewer will ask:**

1. *"When is the AOP proxy created?"* → In `postProcessAfterInitialization` by `AbstractAutoProxyCreator`. This means `@PostConstruct` runs on the RAW bean (before proxying), which is why calling a `@Transactional` method from `@PostConstruct` does not get the transaction.

2. *"Difference between `@PostConstruct` and `InitializingBean.afterPropertiesSet()`?"* → Functionally the same timing. `@PostConstruct` is JSR-250 — no Spring coupling, preferred. `afterPropertiesSet()` couples the class to Spring's interface. `@Bean(initMethod=...)` is for third-party classes you can't annotate.

3. *"What happens to prototype-scoped bean destruction?"* → Spring does NOT call `@PreDestroy` on prototype beans — the caller owns the lifecycle. Only singleton beans are tracked for destruction.

**Common mistakes that get you rejected:**
- Not knowing WHERE in the lifecycle the AOP proxy is created.
- Thinking `@PostConstruct` runs after the proxy is created — it runs on the raw bean.
- Not knowing prototype beans don't get `@PreDestroy` called.

---

### Q65: Bean Scopes
**Company:** Amazon, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Phone/Onsite

**Question:** Explain Spring bean scopes. How do you inject a prototype-scoped bean into a singleton?

**What interviewer is testing:** Understanding of scope semantics and the scope mismatch problem — a common production bug.

**Ideal Answer:**

```
Scope       Lifecycle                               When to use
──────────  ──────────────────────────────────────  ─────────────────────────────────
singleton   One instance per ApplicationContext     Default — stateless services
prototype   New instance per injection request      Stateful helpers, command objects
request     One per HTTP request (web only)         Request-scoped data (user context)
session     One per HTTP session (web only)         Shopping cart, user preferences
application One per ServletContext                  App-wide cache shared across users
websocket   One per WebSocket session               WebSocket handler state
```

**Scope mismatch problem:**

```java
// PROBLEM: singleton injects prototype once — same prototype instance for all calls
@Component                         // singleton (default)
public class OrderProcessor {
    @Autowired
    private CartValidator validator;  // prototype — but injected ONCE at startup!
}

@Component
@Scope("prototype")
public class CartValidator { private List<String> errors = new ArrayList<>(); }
// Every call to orderProcessor.process() reuses the same CartValidator instance
// → errors accumulate across requests — data leak
```

**Solutions:**

**Option 1: ApplicationContext.getBean() (lookup method pattern — ugly):**

```java
@Component
public class OrderProcessor implements ApplicationContextAware {
    private ApplicationContext ctx;

    public void setApplicationContext(ApplicationContext ctx) { this.ctx = ctx; }

    public void process(Order order) {
        CartValidator validator = ctx.getBean(CartValidator.class);  // new each time
        validator.validate(order);
    }
}
```

**Option 2: Scoped Proxy (preferred):**

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class CartValidator {
    private List<String> errors = new ArrayList<>();
    public void validate(Order o) { /* ... */ }
}

@Component
public class OrderProcessor {
    @Autowired
    private CartValidator validator;  // injects a PROXY

    public void process(Order order) {
        validator.validate(order);  // proxy delegates to a NEW CartValidator each call
    }
}
// The proxy acts as the singleton injection point but creates a new target on each method call
```

**Option 3: Provider<T> (JSR-330, cleanest):**

```java
@Component
public class OrderProcessor {
    @Autowired
    private Provider<CartValidator> validatorProvider;  // javax.inject.Provider

    public void process(Order order) {
        CartValidator validator = validatorProvider.get();  // new instance each call
        validator.validate(order);
    }
}
```

**Follow-up questions the interviewer will ask:**

1. *"What is `proxyMode = ScopedProxyMode.INTERFACES` vs `TARGET_CLASS`?"* → `INTERFACES`: JDK dynamic proxy (requires interface). `TARGET_CLASS`: CGLIB subclass proxy (works for concrete classes). Use `TARGET_CLASS` for classes without interfaces.

2. *"Are request-scoped beans available outside a web request?"* → No — accessing them outside of a request (e.g., in a `@Scheduled` method) throws `IllegalStateException: No thread-bound request found`. Use `RequestContextHolder` carefully or restructure to pass request data as parameters.

**Common mistakes that get you rejected:**
- Not knowing the scope mismatch problem — very common interview trap.
- Solving scope mismatch by making the singleton prototype (wrong — changes behavior for all callers).

---

### Q66: @Configuration and CGLIB Proxy
**Company:** Google, Amazon  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Why are `@Configuration` classes proxied by CGLIB? What is the difference between full `@Configuration` mode and lite mode?

**What interviewer is testing:** Deep Spring internals — why inter-bean method calls inside `@Configuration` produce singletons, and what happens without CGLIB.

**Ideal Answer:**

```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(hikariConfig());
    }

    @Bean
    public HikariConfig hikariConfig() {    // shared dependency
        HikariConfig c = new HikariConfig();
        c.setJdbcUrl("jdbc:postgresql://...");
        return c;
    }

    @Bean
    public TransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());  // calls dataSource() again
    }
}
```

In the code above, `transactionManager()` calls `dataSource()` again. Without CGLIB, this would create a **second** `HikariDataSource` — two connection pools, broken transaction management.

**How @Configuration CGLIB proxy solves this:**

Spring subclasses `AppConfig` with CGLIB. The CGLIB proxy intercepts calls to `@Bean` methods:
- If the bean for `dataSource` already exists in the context → return the cached singleton.
- If not yet created → call the real method, register the bean, return it.

```
CGLIB-enhanced AppConfig:
  transactionManager() calls dataSource()
    ↓ CGLIB intercepts
    ↓ checks ApplicationContext: "dataSource" bean already exists?
    ↓ YES → returns cached DataSource singleton
  → TransactionManager receives the SAME DataSource as other beans
```

**Lite mode — `@Component` + `@Bean` (no CGLIB proxy):**

```java
@Component                  // NOT @Configuration — lite mode
public class AppConfig {
    @Bean
    public DataSource dataSource() { return new HikariDataSource(...); }

    @Bean
    public TransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());  // NEW DataSource!
        // In lite mode, this is a plain Java method call — creates a new instance
    }
}
```

In lite mode, `@Bean` methods are NOT intercepted — each call to `dataSource()` creates a new `HikariDataSource`. Use lite mode only when `@Bean` methods are independent and never called from each other.

**Follow-up questions the interviewer will ask:**

1. *"Can `@Configuration` classes be `final`?"* → No — CGLIB needs to subclass them. Making a `@Configuration` class `final` throws `BeanDefinitionParsingException`.

2. *"What if I want independent beans without CGLIB overhead?"* → Use `@Configuration(proxyBeanMethods = false)` (Spring 5.2+) to opt out of CGLIB. Beans become independent — inter-bean calls create new instances, but startup is faster and the class can be `final`.

3. *"Why does `@SpringBootApplication` include `@Configuration`?"* → `@SpringBootApplication` is `@Configuration + @ComponentScan + @EnableAutoConfiguration`. The main class IS a `@Configuration` class and is CGLIB-proxied.

**Common mistakes that get you rejected:**
- Not knowing why CGLIB is needed — saying "CGLIB is just for AOP" misses the bean method interception story.
- Making `@Configuration` classes `final` in production code.

---

### Q67: Auto-configuration
**Company:** Amazon, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** How does Spring Boot auto-configuration work? Explain `spring.factories`, `@ConditionalOnClass`, and how to write your own starter.

**What interviewer is testing:** Whether you understand the mechanism behind "magic" Spring Boot defaults — essential for debugging misconfigured beans and writing library starters.

**Ideal Answer:**

**Discovery mechanism:**

Spring Boot 2.x: `META-INF/spring.factories` (properties file listing auto-configuration classes):
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.MyAutoConfiguration,\
  com.example.OtherAutoConfiguration
```

Spring Boot 3.x: `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (one class per line):
```
com.example.MyAutoConfiguration
com.example.OtherAutoConfiguration
```

**How @EnableAutoConfiguration processes the list:**

`@SpringBootApplication` → `@EnableAutoConfiguration` → `AutoConfigurationImportSelector` reads the file, filters by active conditions, imports the surviving `@Configuration` classes.

**Conditional annotations (the "magic"):**

```java
@AutoConfiguration
@ConditionalOnClass(DataSource.class)           // only if DataSource is on classpath
@ConditionalOnMissingBean(DataSource.class)     // only if user hasn't defined their own
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties props) {
        return DataSourceBuilder.create()
            .url(props.getUrl())
            .username(props.getUsername())
            .build();
    }
}
```

```
Common @Conditional annotations:
@ConditionalOnClass(Foo.class)       — classpath check (JAR present)
@ConditionalOnMissingClass(Foo.class)
@ConditionalOnBean(Foo.class)        — another bean exists in context
@ConditionalOnMissingBean(Foo.class) — no existing bean of this type
@ConditionalOnProperty("my.feature.enabled=true")
@ConditionalOnExpression("#{...}")   — SpEL expression
@ConditionalOnWebApplication         — running in servlet context
```

**Writing your own starter — file structure:**

```
my-spring-boot-starter/
├── pom.xml  (pom: name ends in -spring-boot-starter, depends on -autoconfigure module)
└── my-autoconfigure/
    ├── pom.xml
    └── src/main/
        ├── java/com/example/MyAutoConfiguration.java
        └── resources/META-INF/spring/
            org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

```java
// Auto-configuration class
@AutoConfiguration
@ConditionalOnClass(MyService.class)
@ConditionalOnMissingBean(MyService.class)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {

    @Bean
    public MyService myService(MyProperties props) {
        return new MyService(props.getApiKey(), props.getTimeout());
    }
}

// Properties binding
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    private String apiKey;
    private Duration timeout = Duration.ofSeconds(30);
    // getters/setters
}
```

**Follow-up questions the interviewer will ask:**

1. *"How do you debug which auto-configurations were applied?"* → `--debug` flag or `logging.level.org.springframework.boot.autoconfigure=DEBUG` — Spring Boot prints the auto-configuration report (CONDITIONS EVALUATION REPORT) showing what matched, what didn't, and why.

2. *"How do you exclude an auto-configuration?"* → `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` or `spring.autoconfigure.exclude=...` in properties.

3. *"What is `@AutoConfiguration` vs `@Configuration` for auto-configs?"* → `@AutoConfiguration` (Spring Boot 2.7+) replaces `@Configuration` for auto-config classes. It sets up proper ordering (`before`/`after` ordering) and is excluded from component scanning (preventing double-registration).

**Common mistakes that get you rejected:**
- Thinking Spring Boot auto-configuration is "scanning all your beans" — it is a discrete list of opt-in configurations loaded by a specific mechanism.
- Not knowing that `@ConditionalOnMissingBean` is what lets users override defaults.

---

### Q68: ApplicationContext Hierarchy
**Company:** Google, Amazon  **Difficulty:** 🔴 Hard  **Frequency:** 🔥  **Round:** Onsite

**Question:** Explain the parent-child ApplicationContext hierarchy. Why did traditional Spring MVC use two contexts?

**What interviewer is testing:** Historical Spring architecture knowledge and understanding of context isolation.

**Ideal Answer:**

Spring supports a parent-child context relationship. A child context can see beans from the parent, but the parent cannot see child beans.

**Traditional Spring MVC dual-context architecture (pre-Spring Boot):**

```
Root ApplicationContext (parent) — created by ContextLoaderListener
  ├── @Service beans (OrderService, UserService)
  ├── @Repository beans (UserRepository, OrderRepository)
  ├── @Component beans
  └── DataSource, TransactionManager

WebApplicationContext (child) — created by DispatcherServlet
  ├── @Controller beans
  ├── ViewResolver
  ├── HandlerMapping
  └── HandlerAdapter

Child can @Autowire from parent (services, repos)
Parent CANNOT see child beans (controllers)
```

**Why the separation?**

1. Multiple `DispatcherServlet` instances (e.g., one for REST API, one for admin UI) each had their own child context — controllers isolated per-servlet.
2. Root context beans (services, repos) shared across all servlets.
3. `@Transactional` AOP worked correctly — services in root context got transaction proxies, controllers in child context did not (no accidental transaction-wrapping of controller methods).

**Spring Boot — single context:**

Spring Boot creates ONE `ApplicationContext` containing everything. The `DispatcherServlet` is configured programmatically (not XML), and multiple servlets are rare. The dual-context complexity is gone.

**Common hierarchy problem (AOP doesn't work on services defined in child context):**

```
If @Service is accidentally defined in the child WebApplicationContext instead of root:
  - @Transactional AOP proxy created in the child context
  - But TransactionManager is in the root context
  - Transaction proxy cannot find the TxManager → @Transactional silently does nothing
```

**Follow-up questions the interviewer will ask:**

1. *"How do you create a parent-child context programmatically?"* → `AnnotationConfigApplicationContext child = new AnnotationConfigApplicationContext(); child.setParent(parentCtx); child.register(ChildConfig.class); child.refresh();`

2. *"When is hierarchy still useful today?"* → Multi-tenant apps where each tenant gets an isolated child context. Plugin architectures where each plugin is isolated. Testing: a test parent context with shared infrastructure, child contexts per test class.

**Common mistakes that get you rejected:**
- Not knowing the dual-context existed before Spring Boot.
- Not understanding child-sees-parent but not the reverse.

---

### Q69: Spring Events — Synchronous vs Async
**Company:** Amazon, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain Spring's event publishing mechanism. What is the difference between synchronous and async events? When do you use `@TransactionalEventListener`?

**What interviewer is testing:** Whether you understand the event threading model and the subtle ordering guarantee that `AFTER_COMMIT` provides.

**Ideal Answer:**

Covered in Q58 (Observer pattern). Key Spring-specific additions:

**Synchronous (default) — same thread, same transaction:**

```java
@Transactional
public void placeOrder(Order order) {
    repo.save(order);
    publisher.publishEvent(new OrderPlacedEvent(this, order));
    // Listener runs HERE — inside the same transaction, before commit
}
```

**@Async — different thread, different transaction:**

```java
@Async                          // runs in async executor thread pool
@EventListener
public void onOrderPlaced(OrderPlacedEvent e) {
    emailService.send(e.getOrder().getEmail(), "Order confirmed!");
}
```

**@TransactionalEventListener(AFTER_COMMIT) — critical for consistency:**

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void sendConfirmationEmail(OrderPlacedEvent e) {
    // Runs AFTER the transaction that published the event commits
    // If the transaction rolls back, this listener never fires
    emailService.send(e.getOrder().getEmail(), "Order confirmed!");
}

// IMPORTANT: If listener also needs a transaction, use REQUIRES_NEW
@Transactional(propagation = Propagation.REQUIRES_NEW)
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void auditOrderPlaced(OrderPlacedEvent e) {
    auditRepo.save(new AuditEntry(e.getOrder()));
}
```

**Transaction phases:**

```
BEFORE_COMMIT   — runs before commit; can still rollback the transaction
AFTER_COMMIT    — runs after successful commit; cannot rollback
AFTER_ROLLBACK  — runs after rollback
AFTER_COMPLETION — runs after commit OR rollback
```

**Follow-up questions the interviewer will ask:**

1. *"What if the outer transaction rolls back — does `AFTER_COMMIT` listener fire?"* → No. The listener only fires on successful commit. This is the key guarantee.

2. *"How do you order multiple listeners for the same event?"* → `@Order(1)` (lower = higher priority) on listener methods or implement `Ordered`.

3. *"Can `@TransactionalEventListener` start its own transaction?"* → Only with `@Transactional(propagation = REQUIRES_NEW)` — it runs after the outer commit, so `REQUIRED` would not find an active transaction and create a new one anyway.

**Common mistakes that get you rejected:**
- Using synchronous `@EventListener` for post-commit actions (emails, external calls) — fires before commit, may be for a rolled-back transaction.
- Expecting `@TransactionalEventListener` to have a transaction by default — it doesn't.

---

### Q70: @Transactional Deep Dive
**Company:** Amazon, Google, Flipkart, Goldman Sachs  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Explain how `@Transactional` works internally. What is the self-invocation trap? Explain propagation levels and rollback rules.

**What interviewer is testing:** The single most important Spring concept for backend developers — whether you know the proxy trap, propagation semantics, and when transactions actually roll back.

**Ideal Answer:**

**Proxy-based AOP — how @Transactional works:**

```
External call → Spring proxy intercepts → begins transaction → calls real method
→ real method returns → proxy commits (or rolls back) → result returned to caller
```

The proxy wraps the bean. Only calls that go through the proxy (from outside the bean) are intercepted.

**Self-invocation trap:**

```java
@Service
public class OrderService {
    @Transactional
    public void placeOrder(Order order) {
        saveOrder(order);
        this.sendConfirmation(order);   // SELF-INVOCATION — bypasses proxy!
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendConfirmation(Order order) {
        // This annotation is IGNORED — called via 'this', not through proxy
        // Runs inside the same transaction as placeOrder()
    }
}
```

Fix options:
```java
// Option 1: Inject the bean into itself (Spring allows this for breaking cycles)
@Autowired private OrderService self;
self.sendConfirmation(order);  // goes through proxy

// Option 2: Extract to a separate @Service bean (cleanest design)
confirmationService.send(order);

// Option 3: Use AopContext.currentProxy() (hacky, requires exposeProxy=true)
((OrderService) AopContext.currentProxy()).sendConfirmation(order);
```

**Propagation levels:**

```java
// REQUIRED (default): join existing, or create new
@Transactional(propagation = Propagation.REQUIRED)

// REQUIRES_NEW: always create a new transaction; suspend the existing one
// Use: audit logging that must commit even if outer transaction rolls back
@Transactional(propagation = Propagation.REQUIRES_NEW)

// NESTED: savepoint in the outer transaction; inner rollback rolls back to savepoint only
// Use: partial rollbacks within a larger unit of work (JPA support varies)
@Transactional(propagation = Propagation.NESTED)

// MANDATORY: must have an existing transaction; throws if none
// NEVER: must NOT have a transaction; throws if one exists
// SUPPORTS: join if exists, otherwise run non-transactionally
// NOT_SUPPORTED: always run non-transactionally; suspend existing
```

**Isolation levels:**

```java
// READ_UNCOMMITTED — dirty reads possible (rarely used)
// READ_COMMITTED  — no dirty reads; non-repeatable reads possible (Postgres default)
// REPEATABLE_READ — no dirty/non-repeatable reads; phantom reads possible (MySQL InnoDB default)
// SERIALIZABLE    — full isolation; highest consistency, lowest concurrency
@Transactional(isolation = Isolation.READ_COMMITTED)
```

**Rollback rules:**

```java
// DEFAULT: rolls back on unchecked (RuntimeException), NOT on checked exceptions
@Transactional
public void process() throws IOException {
    // IOException thrown here → does NOT roll back by default
}

// Override rollback rules:
@Transactional(rollbackFor = Exception.class)       // rollback on any exception
@Transactional(noRollbackFor = ValidationException.class)  // don't rollback on this
```

**Follow-up questions the interviewer will ask:**

1. *"Why doesn't @Transactional work on private methods?"* → The CGLIB proxy overrides public methods. Private methods cannot be overridden — the proxy cannot intercept them. Always put `@Transactional` on public methods.

2. *"What is the difference between REQUIRES_NEW and NESTED?"* → `REQUIRES_NEW` suspends the outer transaction and starts a completely independent one (with its own connection). `NESTED` is a savepoint within the same connection — the inner transaction commits to a savepoint, outer can roll back to it. NESTED is not supported by all JPA providers.

3. *"How do you verify @Transactional is working?"* → `TransactionSynchronizationManager.isActualTransactionActive()` returns true inside a transaction. Also: enable Hibernate SQL logging and look for `BEGIN`/`COMMIT` statements.

**Common mistakes that get you rejected:**
- Not knowing the self-invocation trap — this is THE most common @Transactional bug in production.
- Saying "@Transactional rolls back on any exception" — only unchecked by default.
- Putting @Transactional on private methods — silently ignored.

---
