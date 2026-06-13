# 07 — System Design Fundamentals

> The building blocks every system design interview is assembled from. SDE-1 rounds test whether you know these pieces exist and can reason about trade-offs; SDE-2 rounds test whether you can compose them under real constraints and defend every decision with numbers. This file is **concept-driven**: each building block gets a full **WHAT → WHEN → HOW (internals) → TRADE-OFFS → DIAGRAM** treatment, plus interview talking points and the follow-ups loops actually ask. Nothing here says "read more online" — the numbers, the internals, and the failure modes are all here.

**Difficulty legend:** 🟢 foundational (must know cold) · 🟡 core (asked in most SD rounds) · 🔴 advanced (SDE-2 / senior signal) · ⚫ deep-dive (staff-level nuance)
**Frequency legend:** 🔥🔥🔥 appears in nearly every round · 🔥🔥 common · 🔥 occasional but high-signal

---

## How to use this file

A system design interview is **not** a knowledge quiz. The interviewer is testing whether you can:

1. **Drive** — take an ambiguous prompt ("design Twitter") and impose structure.
2. **Quantify** — turn "lots of users" into QPS, storage, and bandwidth numbers that *drive design decisions*.
3. **Trade off** — every choice (SQL vs NoSQL, strong vs eventual consistency) costs something; name the cost.
4. **Go deep on demand** — when they poke ("what happens when this node dies?"), you have a real answer.

So this file is organized the way you should *think*: a **framework** first, then a toolbox of **building blocks**. In the interview you pull blocks off the shelf and justify each with the framework's numbers. For each block:

- **WHAT** — one-paragraph precise definition.
- **WHEN to use** — the decision trigger. This is what separates "I read a blog" from "I've built this."
- **HOW it works internally** — the mechanism. SDE-2 signal lives here.
- **TRADE-OFFS** — the cost of the choice, with numbers.
- **ASCII diagram** — because you *will* draw on the whiteboard.
- **Interview talking points** — the 3–4 sentences that earn signal.
- **Common follow-ups** — scripted in real loops; rehearse them.

---

## Table of Contents

**The Framework**
0. The System Design Interview Framework (5 steps + time budgets + estimation numbers)

**Traffic & Distribution**
1. Load Balancers (L4 vs L7, algorithms, health checks, session persistence)
2. Caching (patterns, eviction, stampede, Redis vs Memcached, invalidation)
3. CDN (push vs pull, edge caching, TTL, invalidation)

**Async & Messaging**
4. Message Queues & Streaming (Kafka vs RabbitMQ vs SQS, delivery semantics, ordering, consumer groups, backpressure, DLQ)

**Data**
5. Databases (SQL vs NoSQL framework, data models, replication, sharding, partitioning, indexing B-tree vs LSM)
6. Consistent Hashing (the ring, virtual nodes, worked example)
7. CAP Theorem & PACELC + Consistency Models
8. Database Scaling Patterns (read replicas, federation, sharding, denormalization, CQRS)

**Application-Layer Patterns**
9. Rate Limiting (token bucket, leaky bucket, fixed/sliding window, counter)
10. API Design (REST vs GraphQL vs gRPC, idempotency, pagination, versioning, status codes)
11. Unique ID Generation (UUID v4/v7, Snowflake, DB ticket server, range allocation)

**Storage & Retrieval**
12. Blob / Object Storage (S3 model, presigned URLs, multipart upload)
13. Search (inverted index, Elasticsearch architecture, sharding)

**Operability**
14. Monitoring & Observability (metrics/Prometheus, logging/ELK, tracing/Jaeger, four golden signals)

**Distributed Systems Essentials**
15. Idempotency Keys
16. The Thundering Herd
17. Bloom Filters
18. Write-Ahead Log (WAL)
19. Quorum (W + R > N)
20. Heartbeats & Leader Election
21. Distributed Locks
22. 2PC vs Saga

**Cheat Sheet**
- Numbers every system designer must memorize

---

# 0 — The System Design Interview Framework
**Difficulty:** 🟢 · **Frequency:** 🔥🔥🔥

The single biggest reason candidates fail a 45-minute system design round is **not** lack of knowledge — it's lack of *structure*. They dive into "I'll use Kafka" before anyone has agreed on what the system does. Impose the following five-step framework on every prompt. Announce it out loud at the start: *"Let me spend a few minutes on requirements, then estimation, then a high-level design, then we'll go deep where you're interested, and I'll close on bottlenecks."* That sentence alone signals seniority.

```
 45-minute round time budget
 ┌──────────────────────────────────────────────────────────────┐
 │ 1. Requirements clarification ............ ~5 min   (10%)      │
 │ 2. Back-of-envelope estimation ........... ~5 min   (10%)      │
 │ 3. High-level design (the boxes) ......... ~10 min  (25%)      │
 │ 4. Detailed design (1–2 deep dives) ...... ~15 min  (35%)      │
 │ 5. Bottlenecks / scaling / wrap-up ....... ~10 min  (20%)      │
 └──────────────────────────────────────────────────────────────┘
   Leave the interviewer room to steer — they will redirect you.
```

---

## Step 1 — Requirements clarification (~5 min)

Split into **functional** (what the system *does*) and **non-functional** (the *qualities* it must have). Non-functional requirements are where the design lives — they decide your whole architecture.

**Functional** — the features. For "design Twitter": post a tweet, follow users, view a home timeline, like/retweet. Pin down scope: *"Are we doing the timeline and posting, and treating search and DMs as out of scope for now?"* Narrowing scope is a senior move; trying to build everything is junior.

**Non-functional** — the qualities. Always probe these:

| Dimension | Question to ask | Why it changes the design |
|---|---|---|
| Scale | How many DAU? Read:write ratio? | Decides sharding, caching, read replicas |
| Latency | p99 target? (e.g. < 200 ms) | Decides CDN, caching, sync vs async |
| Availability | 99.9%? 99.99%? | Decides redundancy, multi-region, failover |
| Consistency | Strong or eventual OK? | Decides CAP posture, replication strategy |
| Durability | Can we lose data? | Decides replication factor, WAL, backups |
| Read/write heavy | Which dominates? | Decides read replicas vs write sharding |

**Availability math you must know cold:**

```
 99%      = "two nines"   → 3.65 days/year   downtime
 99.9%    = "three nines" → 8.77 hours/year
 99.99%   = "four nines"  → 52.6 minutes/year
 99.999%  = "five nines"  → 5.26 minutes/year
```

> **Interview talking points:** "Before designing I want to lock the read:write ratio and the p99 latency target — those two numbers decide almost everything downstream. For a Twitter-like feed I'd expect a heavily read-skewed workload, maybe 100:1, which pushes me toward aggressive caching and fan-out-on-write."

---

## Step 2 — Back-of-envelope estimation (~5 min)

The interviewer wants to see that your design is *sized to the load*. You must be able to derive QPS, storage/year, and bandwidth from DAU. Memorize the conversion anchors below — fumbling arithmetic here is a visible negative signal.

### Powers of 2 (for storage/memory sizing)

```
 2^10 = 1 thousand      → 1 KB
 2^20 = 1 million       → 1 MB
 2^30 = 1 billion       → 1 GB
 2^40 = 1 trillion      → 1 TB
 2^50 = 1 quadrillion   → 1 PB
```

### Time anchors (for QPS)

```
 1 day  ≈ 86,400 seconds ≈ 10^5 seconds  (round to 100k — this is THE trick)
 1 month ≈ 2.5 million seconds
 1 year  ≈ 31.5 million seconds ≈ 3 × 10^7
```

### QPS from DAU — the standard derivation

```
 Assume: 100M DAU, each user does 10 writes/day, 100 reads/day.

 Writes/day = 100M × 10        = 10^9 writes/day
 Avg write QPS = 10^9 / 10^5   = 10,000 writes/sec   (use day≈10^5 s)
 Peak QPS ≈ 2–3× average       ≈ 20,000–30,000 writes/sec

 Reads/day = 100M × 100        = 10^10 reads/day
 Avg read QPS = 10^10 / 10^5   = 100,000 reads/sec
 Peak read QPS ≈ 200,000–300,000 reads/sec
```

> **The trick:** divide daily volume by 100,000. That converts "per day" to "per second" in one step. Then multiply by 2–3 for peak.

### Storage estimation

```
 Each tweet: ~300 bytes text + metadata ≈ 0.5 KB (round up for safety)
 Tweets/day = 100M users × 2 tweets = 2 × 10^8 tweets/day
 Storage/day = 2×10^8 × 0.5 KB = 10^8 KB = 100 GB/day
 Storage/year = 100 GB × 365 ≈ 36.5 TB/year (text only)
 With media (images ~200KB, videos MBs) this is 100–1000× larger → object storage.
```

### Bandwidth estimation

```
 Read bandwidth = read QPS × avg response size
 = 100,000 reads/sec × 0.5 KB = 50 MB/sec outbound (text)
 With media in the feed, multiply by the media factor → CDN territory.
```

### Latency numbers every programmer must know (Jeff Dean's table, rounded)

```
 L1 cache reference ............................   0.5 ns
 Branch mispredict .............................   5   ns
 L2 cache reference ............................   7   ns
 Mutex lock/unlock .............................  25   ns
 Main memory (RAM) reference ...................  100  ns   (0.1 µs)
 Compress 1 KB with Snappy .....................  3,000 ns  (3 µs)
 Send 1 KB over 1 Gbps network .................  10,000 ns (10 µs)
 Read 4 KB randomly from SSD ...................  150,000 ns (150 µs)
 Read 1 MB sequentially from memory ............  250,000 ns (250 µs)
 Round trip within same datacenter .............  500,000 ns (0.5 ms)
 Read 1 MB sequentially from SSD ...............  1,000,000 ns (1 ms)
 Disk seek (spinning disk) .....................  10,000,000 ns (10 ms)
 Read 1 MB sequentially from disk (HDD) ........  20,000,000 ns (20 ms)
 Round trip CA → Netherlands → CA ..............  150,000,000 ns (150 ms)
```

**The hierarchy to internalize:** memory is ~100ns, SSD random read ~150µs (≈1000× slower than RAM), same-DC round trip ~0.5ms, cross-continent ~150ms. *This is why we cache in memory and put data near users.*

> **Interview talking points:** "At 100M DAU with a 100:1 read:write ratio I'm looking at ~100k read QPS average, ~200–300k at peak. That's well beyond a single DB, so I'll need read replicas plus a cache layer with a high hit ratio. Storage is ~36 TB/year for text, so I'm sharding by user ID." Saying numbers like this, derived live, is the strongest signal you can give.

**Common follow-up questions:**
1. *"How did you get 100k QPS?"* → Walk the daily-volume ÷ 100,000 division out loud.
2. *"What's your peak factor?"* → 2–3× average; justify with diurnal traffic patterns.
3. *"How much cache memory do you need?"* → 80/20 rule: cache the hot 20%. If hot set is 20% of 36 TB ≈ 7 TB, you shard the cache across many nodes (each Redis node ~ tens of GB).

---

## Step 3 — High-level design (~10 min)

Draw the boxes and arrows: clients → load balancer → app servers (stateless) → caches → databases, plus async workers off a queue, plus blob storage and CDN for media. Keep app servers **stateless** so you can scale them horizontally behind the load balancer. State lives in the DB, cache, and object store.

```
                       ┌─────────┐
       Clients ───────▶│   CDN   │ (static + media, edge-cached)
          │            └─────────┘
          ▼
   ┌──────────────┐
   │ Load Balancer│ (L7)
   └──────┬───────┘
          ▼
   ┌──────────────┐      ┌──────────┐
   │ App Servers  │◀────▶│  Cache   │ (Redis cluster)
   │ (stateless)  │      └──────────┘
   └──────┬───────┘
          │  writes (sync)             writes (async)
          ▼                                  │
   ┌──────────────┐                          ▼
   │   Primary DB │                    ┌────────────┐
   │  (sharded)   │                    │   Queue     │──▶ Workers ──▶ Blob Store
   └──────┬───────┘                    │  (Kafka)    │
          │ replication                └────────────┘
          ▼
   ┌──────────────┐
   │ Read Replicas│
   └──────────────┘
```

Then describe the request flow for one or two key APIs end-to-end. *"A write tweet goes LB → app server → write to primary shard by userID → publish a 'new tweet' event to Kafka → fan-out workers push it into followers' timeline caches."*

---

## Step 4 — Detailed design (~15 min)

Pick the **1–2 most interesting components** and go deep — often the interviewer points. This is where the building blocks below come in: the data model (schema, indexes, sharding key), the caching strategy, the fan-out design, the ID generation scheme. Show the internals. *Don't* try to detail everything; depth on one component beats shallow on ten.

## Step 5 — Bottlenecks & scaling (~10 min)

Proactively name failure modes and how you handle them: the hot key / celebrity problem, single points of failure, cache stampede, the unbounded queue, the replication lag window. For each, state the mitigation. End with monitoring: *"I'd put the four golden signals on every tier and alert on p99 latency and error rate."* Closing on observability is a senior tell.

> **Interview talking points (framework-level):** The framework is your safety net. If you blank, return to it: "Let me re-anchor on the non-functional requirements." Interviewers reward candidates who stay structured under pressure far more than candidates who name-drop technologies.

---

# 1 — Load Balancers
**Difficulty:** 🟢 · **Frequency:** 🔥🔥🔥

## WHAT
A load balancer (LB) sits between clients and a pool of backend servers and distributes incoming traffic across them. It gives you three things: **horizontal scalability** (add servers behind one address), **availability** (route around dead servers), and a **single stable entry point** (clients hit one VIP / DNS name). It also enables zero-downtime deploys (drain a server, deploy, re-add).

## WHEN to use
The moment you have more than one app server — which is *always*, because a single server is a single point of failure. In an interview, the LB appears in the very first high-level diagram. Use a **Layer 7** LB when you need content-based routing (path/host/header), TLS termination, or HTTP-aware features. Use **Layer 4** when you need raw throughput and protocol-agnostic forwarding (e.g. fronting a database proxy or a non-HTTP service).

## HOW it works internally

### L4 (transport layer) vs L7 (application layer)

```
 ┌──────────────────────────────────────────────────────────────────┐
 │  L4 Load Balancer (TCP/UDP)                                        │
 │  • Routes by IP + port only. Cannot see HTTP path/headers.         │
 │  • Forwards packets / TCP connections; very fast, low CPU.         │
 │  • Can do NAT or DSR (direct server return).                       │
 │  • Throughput: millions of packets/sec on commodity HW.            │
 │  Example: AWS NLB, HAProxy in TCP mode.                            │
 ├──────────────────────────────────────────────────────────────────┤
 │  L7 Load Balancer (HTTP/HTTPS)                                     │
 │  • Terminates TCP, parses HTTP. Routes by URL path, host, header,  │
 │    cookie. Can rewrite, compress, cache, terminate TLS.            │
 │  • Higher CPU per request; richer features.                        │
 │  • Enables: /api → service A, /img → service B; canary by header.  │
 │  Example: AWS ALB, NGINX, Envoy, HAProxy in HTTP mode.             │
 └──────────────────────────────────────────────────────────────────┘
```

The key mental model: **L4 moves bytes, L7 understands requests.** L4 establishes a connection to a backend and shuttles bytes without inspecting them. L7 acts as a full reverse proxy: it terminates the client connection, reads the HTTP request, decides where it goes based on content, and opens its own connection to the backend.

### Balancing algorithms

```
 ROUND ROBIN
   Request 1 → S1,  Request 2 → S2,  Request 3 → S3,  Request 4 → S1 ...
   + Dead simple, even distribution when requests are uniform.
   - Ignores actual load: a slow request on S1 still gets the next turn.

 WEIGHTED ROUND ROBIN
   S1(weight 3), S2(weight 1) → S1,S1,S1,S2,S1,S1,S1,S2 ...
   + Send more to beefier servers.

 LEAST CONNECTIONS
   Route to the server with the fewest active connections.
   + Adapts to uneven request durations (long-lived connections, slow queries).
   - Needs the LB to track connection counts.

 LEAST RESPONSE TIME
   Least connections + lowest average latency. Best for heterogeneous backends.

 IP HASH
   server = hash(client_ip) % N
   + Same client always hits the same server → sticky without cookies.
   - Adding/removing a server reshuffles nearly everyone (N changes). Uneven if
     traffic is behind a few NAT egress IPs.

 CONSISTENT HASHING  (see §6)
   Maps clients/keys onto a ring; adding/removing a node moves only ~1/N keys.
   + Used for stateful backends (cache nodes, sharded stores) where you want
     minimal reshuffling on membership change.
```

### Health checks

The LB continuously probes backends and removes unhealthy ones from rotation.

```
 ┌──────────┐  GET /health every 5s   ┌──────────┐
 │    LB    │ ───────────────────────▶│   S2     │
 │          │ ◀── 200 OK (healthy) ────│          │
 │          │                          └──────────┘
 │          │  GET /health             ┌──────────┐
 │          │ ───────────────────────▶│   S3     │  (no response x 3)
 │          │ ◀──── timeout ───────────│  DEAD    │ → removed from pool
 └──────────┘                          └──────────┘

 Two flavors:
  • Passive: observe real traffic; mark unhealthy on errors/timeouts.
  • Active: synthetic probes (HTTP GET /health, TCP connect) on an interval.
 Tunables: interval (5s), timeout (2s), unhealthy threshold (3 fails),
           healthy threshold (2 successes to re-add).  Avoid flapping.
```

A good `/health` endpoint checks downstream dependencies (DB reachable? cache up?) but must be cheap — a heavy health check can itself take down the fleet.

### Session persistence (sticky sessions)

If a server keeps per-user state in memory (a session), subsequent requests must return to the same server. Options:

```
 1. Cookie-based stickiness (L7): LB sets a cookie naming the backend.
    Request carries cookie → LB routes back to that server.
 2. IP hash (L4/L7): hash client IP to a server.

 PROBLEM: stickiness defeats even load balancing and breaks on server death
 (the user's session is gone). PREFERRED: make app servers STATELESS — store
 session in a shared store (Redis) so any server can handle any request.
```

> **The senior move:** "I'd avoid sticky sessions and keep app servers stateless by externalizing session state to Redis. Then the LB can use least-connections freely, and a server dying doesn't log anyone out."

## TRADE-OFFS

| Concern | L4 | L7 |
|---|---|---|
| Throughput | Higher (no parsing) | Lower (parses HTTP) |
| Routing intelligence | IP/port only | Path/host/header/cookie |
| TLS termination | No (passthrough) | Yes |
| Latency added | ~microseconds | ~sub-millisecond |
| Use case | DB proxy, gaming, raw TCP | Web APIs, microservices |

- **Single LB = single point of failure.** Run LBs in active-passive or active-active pairs; use DNS or a floating/virtual IP (VIP) with a heartbeat (e.g. keepalived/VRRP) so a standby takes over.
- **Global vs local.** DNS-based global load balancing (GeoDNS, anycast) routes users to the nearest region; a regional LB then distributes within the datacenter.

```
        DNS / GeoDNS (global)
        ┌────────────┴────────────┐
   us-east region            eu-west region
   ┌────────────┐            ┌────────────┐
   │  LB (a-a)  │            │  LB (a-a)  │
   └─────┬──────┘            └─────┬──────┘
     app servers               app servers
```

> **Interview talking points:** "I'll front the stateless app tier with an L7 load balancer using least-connections, active health checks every few seconds, and TLS termination at the LB. I'll run the LBs as an active-active pair behind a VIP to avoid a single point of failure, and use GeoDNS to route users to the nearest region."

**Common follow-up questions:**
1. *"L4 or L7 here, and why?"* → L7 for HTTP APIs (content routing, TLS); L4 for raw throughput / non-HTTP.
2. *"What if the load balancer itself dies?"* → Redundant LBs (active-active or active-passive) + VIP failover + DNS.
3. *"Round-robin vs least-connections?"* → Least-connections when request durations vary; round-robin when uniform and cheap.
4. *"How do you do a zero-downtime deploy?"* → Drain connections from a server (stop sending new requests, let in-flight finish), deploy, health-check, re-add.
5. *"Sticky sessions — yes or no?"* → Prefer stateless + shared session store; stickiness only as a fallback, knowing it hurts balance and resilience.

---

# 2 — Caching
**Difficulty:** 🟢 · **Frequency:** 🔥🔥🔥

## WHAT
A cache is a small, fast store that holds a copy of frequently accessed data so you can avoid an expensive recomputation or a slow datastore/network hit. Caches exploit two facts: **memory is ~1000x faster than SSD random reads** (100 ns vs 150 µs) and **access is skewed** (the 80/20 rule — a small fraction of data serves most requests). A good cache turns a 100 ms DB query into a sub-millisecond memory lookup.

## WHEN to use
- Read-heavy workloads where the same data is requested repeatedly (timelines, product pages, user profiles).
- Expensive computations whose inputs repeat (rendered HTML, aggregations).
- To absorb read load off a database that can't scale reads cheaply.
- **Not** for write-heavy, low-reuse, or strongly-consistent-critical data where staleness is unacceptable without careful invalidation.

Caches exist at many layers: client/browser → CDN → API gateway → application (in-process) → distributed cache (Redis/Memcached) → database buffer pool. In interviews, "add a cache" almost always means a distributed cache in front of the DB.

## HOW it works internally — caching patterns

### Cache-aside (lazy loading) — the default

The application is responsible for the cache. On read: check cache; on miss, read DB and populate cache.

```
 READ (cache-aside):
   app → cache.get(key)
     hit  → return value
     miss → db.read(key) → cache.set(key, value, ttl) → return value

 WRITE (cache-aside):
   app → db.write(key, value) → cache.delete(key)   # invalidate, don't update
```

```
   ┌─────┐  1.get   ┌───────┐  miss
   │ App │ ───────▶ │ Cache │
   │     │ ◀─────── │       │
   └──┬──┘  4.set   └───────┘
      │ 2.read on miss
      ▼
   ┌──────┐
   │  DB  │  3.return
   └──────┘
```

- **Pro:** only requested data is cached (no waste); cache failure is survivable (fall back to DB); simple.
- **Con:** first read is always a miss (cold cache); risk of stale data if a write updates the DB but the delete fails. On write we **delete** rather than update the cache entry — updating risks a race where a concurrent read repopulates a stale value.

### Write-through

Writes go to the cache *and* the DB synchronously, in one operation. Reads are always served from cache.

```
 WRITE: app → cache.set(key,value) → (cache) → db.write(key,value) → ack
 READ : app → cache.get(key) (always hit for written keys)
```

- **Pro:** cache is never stale; reads are fast and consistent with writes.
- **Con:** every write pays the cache + DB latency; caches data that may never be read (write amplification); cold start still possible for never-written-then-read paths.

### Write-behind (write-back)

Writes go to the cache immediately; the cache asynchronously flushes to the DB later (batched).

```
 WRITE: app → cache.set(key,value) → ACK immediately
                         │ (async, batched)
                         ▼
                    db.write(...)   (e.g. every 1s or per N writes)
```

- **Pro:** extremely fast, low-latency writes; batching reduces DB load (coalesce many writes to the same key).
- **Con:** **data loss risk** — if the cache node dies before flushing, unflushed writes are gone. Use only when some loss is tolerable (metrics, counters) or pair with a durable log.

### Write-around

Writes go straight to the DB, bypassing the cache. The cache is populated only on read (cache-aside read path).

```
 WRITE: app → db.write(...)        (cache untouched)
 READ : cache-aside (miss → db → populate)
```

- **Pro:** avoids flooding the cache with write-once-read-never data (e.g. logs).
- **Con:** recently written data is a guaranteed cache miss on first read.

### Comparison

```
 Pattern        Write path                Read freshness     Loss risk   Best for
 ─────────────────────────────────────────────────────────────────────────────────
 Cache-aside    DB then invalidate cache  may be stale       none        general read-heavy
 Write-through  cache + DB (sync)          always fresh       none        read-after-write
 Write-behind   cache now, DB later        fresh in cache     HIGH        write-heavy, tolerant
 Write-around   DB only                    miss on 1st read   none        write-once data
```

## Eviction policies

A cache has finite memory; when full, it must evict to make room.

```
 LRU (Least Recently Used)  ← the default
   Evict the entry not accessed for the longest time.
   Internals: hashmap + doubly-linked list. On access, move node to head.
   Evict from the tail. O(1) get and put.

   head [most recent] <-> ... <-> [least recent] tail  →  evict tail

 LFU (Least Frequently Used)
   Evict the entry with the fewest accesses. Tracks a frequency count per key.
   Better when popularity is stable; worse for changing access patterns
   (an item popular last week clings to memory). Needs aging/decay to adapt.

 FIFO   Evict oldest inserted (ignores access). Rarely ideal.

 TTL (Time To Live)
   Each entry expires after a fixed duration regardless of access.
   Bounds staleness. Combine with LRU: TTL caps freshness, LRU caps memory.
   Random jitter on TTL avoids synchronized mass expiry (see stampede below).
```

LRU is the standard answer. Mention LFU when popularity is long-tailed and stable, and *always* mention TTL for bounding staleness.

## Cache stampede / thundering herd & mitigations

A **cache stampede** (a.k.a. dog-piling / thundering herd) happens when a popular key expires (or the cache restarts) and thousands of concurrent requests all miss simultaneously, all hit the DB at once, and overwhelm it.

```
        key "trending" expires at t=0
   1000 requests at t=0+ε  → all MISS → all 1000 hit DB → DB melts
   ┌────┐┌────┐┌────┐ ... ┌────┐
   │req ││req ││req │     │req │
   └─┬──┘└─┬──┘└─┬──┘     └─┬──┘
     └─────┴─────┴───────────┘  all stampede → ┌────┐
                                                │ DB │ (overloaded)
                                                └────┘
```

Mitigations:

```
 1. LOCKING / REQUEST COALESCING (single-flight):
    First miss acquires a lock and recomputes; others wait for the result
    (or briefly serve stale). Only ONE DB hit per key.

 2. EARLY / PROBABILISTIC RECOMPUTATION:
    Refresh a key BEFORE it expires, with probability rising as TTL approaches.
    (XFetch algorithm.) Spreads the recompute over time, no synchronized expiry.

 3. STALE-WHILE-REVALIDATE:
    Serve the stale value immediately while ONE background task refreshes it.
    Users never see a miss; DB sees one request.

 4. TTL JITTER:
    Add randomness to TTLs (e.g. 300s ± 30s) so keys don't all expire together.

 5. CACHE WARMING:
    Pre-populate hot keys on deploy/restart so cold start doesn't stampede.
```

A related failure is the **hot key**: one key (a celebrity's profile) gets so much traffic it saturates the single cache node holding it. Mitigate by **replicating** the hot key across nodes or appending a random suffix to spread it (`key#1`..`key#N`) and reading a random replica.

## Redis vs Memcached

```
 ┌──────────────────────────────┬──────────────────────────────────┐
 │ Memcached                    │ Redis                            │
 ├──────────────────────────────┼──────────────────────────────────┤
 │ Data types: strings/blobs    │ Rich: strings, lists, sets,      │
 │   only                       │   sorted sets, hashes, bitmaps,  │
 │                              │   HyperLogLog, streams, geo      │
 │ Multi-threaded               │ Single-threaded core (fast,      │
 │                              │   no lock contention) + I/O      │
 │                              │   threads in newer versions      │
 │ No persistence               │ Persistence: RDB snapshots +     │
 │                              │   AOF append-only log            │
 │ No replication built-in      │ Replication + Sentinel +         │
 │                              │   Cluster (sharding + failover)  │
 │ Pure cache, evict on full    │ Cache OR datastore; pub/sub;     │
 │                              │   Lua scripting; transactions    │
 │ Slightly less memory         │ More features = more overhead    │
 │   overhead per key           │                                  │
 └──────────────────────────────┴──────────────────────────────────┘
```

**Pick Memcached** when you want a dead-simple, multi-threaded, pure key→blob cache and nothing else. **Pick Redis** (the default modern choice) when you want data structures, persistence, replication, pub/sub, atomic operations (rate limiting, leaderboards), or anything beyond a plain cache. In interviews, Redis is almost always the right answer because of its versatility.

## Cache invalidation strategies

> "There are only two hard things in computer science: cache invalidation and naming things." Invalidation is the #1 source of cache bugs.

```
 1. TTL expiry (time-based):
    Simplest. Accept staleness up to TTL. No coordination needed.
    Use when slightly stale data is acceptable (most read-heavy systems).

 2. Write-through invalidation (event-based):
    On every write, delete/update the affected keys. Strong freshness, but
    requires knowing exactly which keys a write touches (hard with aggregates).

 3. Versioned keys:
    Embed a version in the key: user:42:v7. On update, bump version → old key
    is orphaned (and TTL-evicted later). No explicit delete needed; avoids races.

 4. Purge / fan-out invalidation:
    On a write that affects many derived entries, publish an invalidation event
    that workers consume to evict all affected keys.
```

The hard cases: **aggregate caches** (a write to one tweet must invalidate every follower's timeline cache) and **distributed caches** (an invalidation must reach all nodes). Versioned keys sidestep the distributed-delete race elegantly.

> **Interview talking points:** "For a read-heavy timeline I'd use cache-aside with Redis, LRU eviction, and TTLs with jitter to bound staleness and avoid synchronized expiry. To prevent stampedes on hot keys I'd add single-flight locking or stale-while-revalidate so only one request rebuilds a missing key. For the celebrity hot-key problem I'd replicate that key across cache nodes."

**Common follow-up questions:**
1. *"What's your cache hit ratio target and why does it matter?"* → Aim for 90%+; each 1% miss can multiply DB load. Compute backend load = QPS × (1 − hitRatio).
2. *"How do you invalidate on a write?"* → Delete the key (cache-aside) or versioned keys; discuss the multi-key aggregate problem.
3. *"What happens on a cache node failure?"* → Cache-aside survives (fall back to DB, but watch the stampede); use replication / clustering to avoid a cold cliff.
4. *"How do you size the cache?"* → 80/20: cache the hot working set; estimate hot-set size and shard across nodes.
5. *"Write-through vs write-behind?"* → Write-through for freshness, write-behind for write throughput at the cost of durability.
6. *"How do you keep two cache nodes consistent?"* → They shard disjoint keys (consistent hashing); for replicas, use TTL + versioned keys to tolerate brief divergence.

---

# 3 — Content Delivery Network (CDN)
**Difficulty:** 🟢 · **Frequency:** 🔥🔥

## WHAT
A CDN is a geographically distributed network of **edge servers** (Points of Presence, PoPs) that cache content close to users. Instead of every request traveling to your origin server (potentially across the planet, ~150 ms each way), the user is served from a nearby edge (~10–30 ms). CDNs primarily serve **static assets** (images, video, CSS, JS, fonts) but increasingly cache API responses and run edge compute too.

## WHEN to use
- Static/media-heavy content served to a geographically distributed audience.
- To offload bandwidth and reduce latency for global users.
- To absorb traffic spikes / DDoS at the edge before they reach the origin.
- Video streaming (segmented HLS/DASH chunks cache beautifully).
- **Not** for highly dynamic, per-user, uncacheable responses (though edge compute blurs this).

## HOW it works internally

```
   User in India          User in Brazil
        │                       │
        ▼                       ▼
  ┌───────────┐           ┌───────────┐
  │ Edge PoP  │           │ Edge PoP  │   (cache HIT → served locally ~20ms)
  │  Mumbai   │           │ São Paulo │
  └─────┬─────┘           └─────┬─────┘
        │ cache MISS            │ cache MISS
        └───────────┬───────────┘
                    ▼
            ┌────────────────┐
            │  Origin Server │ (your infra, e.g. us-east)
            │  / Object Store│
            └────────────────┘
```

User requests are routed to the nearest edge via **GeoDNS** or **anycast** (the same IP announced from many locations; BGP routes to the topologically nearest one). On a hit the edge serves directly; on a miss it fetches from origin (or a parent/shield cache), stores it, and serves it.

### Push vs Pull CDN

```
 PULL CDN  (origin-pull, lazy):
   Edge fetches from origin on the FIRST request for a URL (a miss), then
   caches it for subsequent requests until TTL expires.
   + Zero upfront work; only requested content is cached; self-managing.
   - First request per region per asset is a slow miss; origin must stay up.
   Best for: large catalogs where most content is rarely accessed (long tail).

 PUSH CDN  (proactive):
   YOU upload/publish content to the CDN ahead of time; it's distributed to
   edges before any user asks.
   + No first-request penalty; full control over what's where; origin can be
     down afterward.
   - You manage distribution and storage; wasteful for rarely-accessed content;
     more operational overhead.
   Best for: predictable, high-traffic assets (a launch video, app bundle).
```

### Edge caching & TTL

Cache behavior is driven by HTTP headers from the origin:

```
 Cache-Control: max-age=86400        → cache 1 day at edge & browser
 Cache-Control: s-maxage=3600        → shared (CDN) cache 1 hour
 Cache-Control: no-store             → never cache
 Cache-Control: private              → browser may cache, CDN may NOT
 Cache-Control: stale-while-revalidate=60  → serve stale up to 60s while refreshing
 ETag: "abc123"  / If-None-Match     → revalidation: 304 Not Modified if unchanged
```

The edge keeps an object until its TTL (`max-age`/`s-maxage`) expires, then revalidates with the origin (conditional GET using `ETag`/`Last-Modified`) — a 304 means "still good, reset the timer" without re-transferring the body.

### Invalidation

Because edges cache by TTL, pushing an update before TTL expiry requires explicit invalidation:

```
 1. TTL expiry: just wait. Simple, but slow to propagate (up to max-age).
 2. PURGE / explicit invalidation: API call tells all edges to drop a URL or
    a path prefix. Propagation takes seconds to minutes across the fleet.
 3. CACHE BUSTING via versioned URLs (the BEST practice):
    /static/app.js?v=8  or  /static/app.a1b2c3.js
    A new version = a new URL = guaranteed fresh, no purge needed, and old
    versions stay cached for clients still using them. Immutable + long TTL.
```

Versioned/fingerprinted URLs are the production-standard answer: set `Cache-Control: max-age=31536000, immutable` and change the filename hash on every deploy. You never invalidate; you just change the URL.

## TRADE-OFFS

```
 + Latency: ~150ms cross-continent → ~10–30ms from a nearby edge.
 + Origin offload: 90%+ of static traffic served at edge → huge bandwidth savings.
 + Resilience: edges absorb spikes & DDoS; origin survives.
 - Staleness: changes take up to TTL to propagate (mitigate with versioned URLs).
 - Cost: per-GB egress + per-request; can be significant at scale.
 - Cache miss penalty: first request per edge is slower than direct (extra hop).
 - Not for dynamic/personalized content (unless using edge compute / ESI).
```

> **Interview talking points:** "I'd put all static assets and media behind a pull CDN with fingerprinted, immutable URLs so deploys never require cache purges and old clients keep working. The CDN offloads ~90% of bandwidth from the origin and cuts global latency from ~150 ms to ~20 ms. For user-uploaded media I'd serve directly from object storage through the CDN."

**Common follow-up questions:**
1. *"Push or pull?"* → Pull by default (self-managing, good for long-tail); push for known high-traffic launch assets.
2. *"How do you invalidate a CDN?"* → Versioned URLs (preferred), explicit purge, or TTL expiry.
3. *"Can a CDN cache dynamic/API responses?"* → Yes with short TTLs and `Vary` headers, or edge compute for personalization; be careful with `private`/`Set-Cookie`.
4. *"How does a user reach the nearest edge?"* → GeoDNS or anycast routing.
5. *"How do you serve user-uploaded images globally?"* → Object store (S3) as origin, CDN in front, presigned/long-TTL URLs.

---

# 4 — Message Queues & Streaming
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
A message queue (or streaming platform) lets one part of a system hand work to another **asynchronously and durably**, decoupling producer from consumer. The producer writes a message and moves on; the consumer reads it when ready. This buys you **decoupling** (services don't call each other directly), **buffering** (absorb spikes — the queue grows instead of the DB melting), **resilience** (a down consumer just means a backlog, not lost requests), and **fan-out** (one event, many consumers).

## WHEN to use
- Any work that doesn't need a synchronous response: sending emails, generating thumbnails, fan-out of a tweet to followers, indexing for search, analytics events.
- To smooth bursty traffic: writes hit the queue at 100k/s, workers drain at a steady 10k/s.
- To decouple a monolith into services that communicate via events.
- Event sourcing / change-data-capture / stream processing.
- **Not** for request/response where the caller blocks for the answer (use RPC/HTTP), nor when end-to-end latency must be sub-10ms.

```
   Synchronous (tight coupling):           Asynchronous (decoupled via queue):
   ┌────────┐    blocks      ┌────────┐    ┌────────┐  enqueue  ┌───────┐  dequeue ┌────────┐
   │Producer│ ─────────────▶ │Consumer│    │Producer│ ────────▶ │ Queue │ ───────▶ │Consumer│
   │        │ ◀───────────── │        │    │        │ ◀ack      │       │          │        │
   └────────┘   waits for    └────────┘    └────────┘           └───────┘          └────────┘
        if consumer slow → producer slow      producer never blocks on consumer speed
```

## HOW it works internally

### Two models: queue vs pub/sub (log)

```
 QUEUE (competing consumers, work distribution):
   One message → delivered to exactly ONE consumer in the group.
   Used for distributing work. (RabbitMQ, SQS)
   ┌─────┐   ┌──────────┐   ┌── consumer A (gets msg 1,3)
   │ Prod│──▶│  Queue   │──▶┤
   └─────┘   └──────────┘   └── consumer B (gets msg 2,4)

 PUB/SUB or LOG (broadcast / replayable):
   One message → delivered to EVERY subscriber/consumer group.
   Kafka is a distributed, append-only, replayable LOG (not a delete-on-read queue).
   ┌─────┐   ┌──────────┐   ┌── group "billing"  (reads all)
   │ Prod│──▶│  Topic   │──▶┤
   └─────┘   └──────────┘   └── group "analytics" (reads all)
```

### Kafka internals — partitions, offsets, consumer groups

Kafka is the streaming heavyweight. A **topic** is split into **partitions**; each partition is an ordered, immutable, append-only log on disk. Producers append; consumers track an **offset** (position) per partition.

```
 Topic "tweets" with 3 partitions:

 Partition 0:  [m0][m1][m2][m3][m4]...   ← append only; each msg has an offset
 Partition 1:  [m0][m1][m2]...
 Partition 2:  [m0][m1][m2][m3]...
                                  ▲
            consumer reads sequentially, commits offset (e.g. "I'm at offset 4")

 KEY → PARTITION:  partition = hash(key) % numPartitions
   Same key (e.g. userId) → same partition → ordered per key.

 CONSUMER GROUP: each partition is consumed by exactly ONE consumer in a group.
   #consumers ≤ #partitions for full parallelism; extra consumers sit idle.
   3 partitions, group of 3 → 1:1. Group of 4 → one idle. Group of 2 → one
   consumer handles 2 partitions.
```

Because data is retained on disk (by time or size, not deleted on read), Kafka supports **replay** — a new consumer group can re-read from offset 0. Throughput is enormous (millions of msgs/sec) because reads/writes are sequential disk I/O and use zero-copy.

### Delivery semantics

This is the single most-asked MQ interview topic.

```
 AT-MOST-ONCE   (fire and forget)
   Deliver, don't retry. Message may be LOST but never duplicated.
   Mechanism: consumer commits offset BEFORE processing. If it crashes
   mid-process, that message is skipped.
   Use: metrics/telemetry where loss is OK and dupes are costly.

 AT-LEAST-ONCE  (the common default)
   Retry until acknowledged. Message is NEVER lost but may be DUPLICATED.
   Mechanism: consumer commits offset AFTER processing. Crash before commit
   → message redelivered.
   Use: most systems. Pair with IDEMPOTENT consumers (see §15) to dedupe.

 EXACTLY-ONCE   (hardest, often "effectively once")
   Each message takes effect exactly once. Achieved via at-least-once delivery
   + idempotency, OR transactional producer/consumer (Kafka transactions:
   atomic "consume-process-produce" + offset commit).
   Use: financial/critical paths. Expensive; prefer at-least-once + idempotency.
```

> **The senior framing:** "True exactly-once delivery across a network is impossible (two generals problem). What we build is *effectively-once*: at-least-once delivery plus idempotent processing, so duplicates are harmless." Saying this is a strong signal.

### Ordering guarantees

```
 Kafka:   ordered WITHIN a partition only. Global order across partitions is
          NOT guaranteed. To order by user, key by userId → same partition.
 RabbitMQ:ordered within a single queue with one consumer; parallel consumers
          break order.
 SQS Standard: NO ordering (best-effort). SQS FIFO: ordered within a
          message-group-id, lower throughput.
```

The fundamental tension: **ordering requires serialization, which limits parallelism.** You get per-key ordering by routing a key to one partition/queue, but global ordering means a single consumer — a throughput bottleneck.

### Consumer groups & scaling

```
 Scale consumers by adding them to a group, up to #partitions.
 Kafka rebalances partitions across the group when a consumer joins/leaves.

 Before: 2 consumers, 4 partitions     After adding C3: rebalance
   C1 ← p0,p1                             C1 ← p0,p1
   C2 ← p2,p3                             C2 ← p2
                                          C3 ← p3
```

### Backpressure

When producers outpace consumers, the queue grows. Backpressure is how the system signals "slow down" or absorbs the imbalance.

```
 Strategies when consumers can't keep up:
  • Buffer (let the queue grow) — bounded by retention/disk; the whole point
    of a queue is to absorb bursts temporarily.
  • Drop / sample — shed load (acceptable for metrics).
  • Block the producer — propagate backpressure upstream (bounded queues).
  • Scale consumers — autoscale on queue depth / consumer lag.
 WATCH: consumer lag (offset gap). Growing lag = consumers falling behind =
        alert and scale, or the backlog grows unbounded.
```

### Dead-letter queue (DLQ)

A message that repeatedly fails processing (poison message) shouldn't block the queue or retry forever. After N failed attempts it's moved to a **dead-letter queue** for inspection.

```
   ┌───────┐  process fail x3   ┌──────────────────┐
   │ Queue │ ─────────────────▶ │ Dead-Letter Queue│ → alert, inspect, replay
   └───────┘                    └──────────────────┘
   Prevents one bad message (malformed, triggers a bug) from blocking the queue
   head or looping forever. Set maxReceiveCount (e.g. 3) → route to DLQ.
```

## Kafka vs RabbitMQ vs SQS — when each

```
 ┌──────────────┬────────────────────┬──────────────────┬───────────────────┐
 │              │ Kafka              │ RabbitMQ         │ SQS (AWS)         │
 ├──────────────┼────────────────────┼──────────────────┼───────────────────┤
 │ Model        │ Distributed log    │ Broker / smart   │ Managed queue     │
 │              │ (pull, replayable) │ routing (push)   │ (pull, managed)   │
 │ Throughput   │ Very high          │ Moderate-high    │ High (auto-scale) │
 │              │ (M msgs/sec)       │ (10s–100k/sec)   │ (nearly unlimited)│
 │ Ordering     │ Per-partition      │ Per-queue        │ FIFO type only    │
 │ Retention    │ Days/weeks; replay │ Until consumed   │ Up to 14 days     │
 │ Routing      │ Topic+partition    │ Rich: exchanges, │ Simple queue;     │
 │              │ (key hash)         │ routing keys,    │ SNS for fan-out   │
 │              │                    │ fanout, topic    │                   │
 │ Delivery     │ At-least / exactly │ At-least / at-   │ At-least-once     │
 │              │ (transactions)     │ most-once        │ (FIFO: dedup)     │
 │ Ops          │ Heavy (ZK/KRaft,   │ Moderate         │ Zero (fully       │
 │              │ partitions)        │                  │ managed)          │
 └──────────────┴────────────────────┴──────────────────┴───────────────────┘
```

**Pick Kafka** for high-throughput event streaming, log aggregation, event sourcing, replay, and multiple independent consumer groups reading the same data (analytics + billing + search off one tweet stream). **Pick RabbitMQ** for complex routing logic (topic/fanout/direct exchanges), task queues with priorities, lower volume, and request/reply patterns. **Pick SQS** when you're on AWS and want zero operational overhead — managed, auto-scaling, pay-per-use; pair with SNS for fan-out.

> **Interview talking points:** "I'll decouple the write path with a queue: the tweet write returns as soon as it's persisted and an event is published; fan-out to followers' timelines happens asynchronously via Kafka consumers. I'll use at-least-once delivery with idempotent consumers (dedupe on tweetId) so retries are safe, key by userId for per-user ordering, and route poison messages to a DLQ after 3 failures. I'd alert on consumer lag and autoscale workers on queue depth."

**Common follow-up questions:**
1. *"At-least-once vs exactly-once — what do you implement?"* → At-least-once + idempotent consumers ("effectively once"); true exactly-once needs transactions and is expensive.
2. *"How do you guarantee ordering?"* → Per-key partitioning (same key → same partition); global order needs a single consumer (throughput cost).
3. *"What happens when consumers fall behind?"* → Monitor consumer lag; autoscale; backpressure; the queue buffers temporarily but is bounded by retention.
4. *"What's a poison message and how do you handle it?"* → A message that always fails; route to a DLQ after N retries to avoid head-of-line blocking.
5. *"Kafka or RabbitMQ here?"* → Kafka for high-volume replayable streams with multiple consumer groups; RabbitMQ for rich routing and task queues.
6. *"How many partitions?"* → Enough for target parallelism (≥ peak #consumers); too many adds overhead and rebalance cost.
7. *"How do you not lose messages?"* → Durable/persisted queue, producer acks (`acks=all`), replication factor ≥ 3, consumer commits offset only after successful processing.

---

# 5 — Databases
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
The database is where durable state lives. The central interview skill is **choosing the right store and explaining why**, then reasoning about how it scales (replication, sharding) and how it stores data physically (indexing). "Use a database" is never an answer; "use a sharded relational store with read replicas because the workload is read-heavy and relational, sharded by userId" is.

## SQL vs NoSQL decision framework

```
 CHOOSE SQL (relational: PostgreSQL, MySQL) WHEN:
   • Data is structured & relational (entities with foreign keys, joins).
   • You need ACID transactions across multiple rows/tables (money, orders).
   • Strong consistency matters (banking, inventory).
   • Complex ad-hoc queries / aggregations / reporting.
   • Schema is well-understood and stable.

 CHOOSE NoSQL WHEN:
   • Massive scale / write throughput beyond one machine, horizontal first.
   • Flexible/evolving schema (varied attributes per record).
   • Simple access patterns (key lookups) at huge volume.
   • You can trade strong consistency for availability + partition tolerance.
   • High write throughput, denormalized data, no complex joins.

 THE HONEST TRUTH for interviews:
   Modern SQL (Postgres) scales further than people think (read replicas,
   partitioning, even sharding). Start with SQL unless a specific requirement
   (scale, write volume, schema flexibility, geo-distribution) forces NoSQL.
```

### ACID vs BASE

```
 ACID (classic SQL guarantees):
   Atomicity   — all-or-nothing transactions.
   Consistency — DB moves from one valid state to another (constraints hold).
   Isolation   — concurrent txns don't interfere (isolation levels).
   Durability  — committed data survives crashes (WAL, see §18).

 BASE (typical NoSQL posture):
   Basically Available — system stays up (returns something).
   Soft state         — state may change without input (replication catching up).
   Eventually consistent — replicas converge given enough time.
```

## NoSQL data models

```
 KEY-VALUE  (Redis, DynamoDB, Riak)
   { key → opaque value }. O(1) get/put by key. No queries on the value.
   Use: sessions, caches, user prefs, shopping carts.

 DOCUMENT  (MongoDB, Couchbase, DynamoDB)
   { key → JSON/BSON document }. Query/index on fields; flexible nested schema.
   Use: content, catalogs, user profiles — data that's read as a whole object.

 WIDE-COLUMN  (Cassandra, HBase, Bigtable)
   Rows keyed by a partition key; each row has dynamic columns grouped in
   column families. Optimized for huge write throughput & range scans.
   Use: time-series, event logs, messaging, analytics at massive scale.

 GRAPH  (Neo4j, Neptune)
   Nodes + edges with properties. Optimized for traversals (friends-of-friends).
   Use: social graphs, recommendations, fraud rings, knowledge graphs.
```

```
 Key-Value:   "user:42" → "{...blob...}"

 Document:    { _id: 42, name: "Ada", tags: ["sql","go"], addr: { city: "NYC" } }

 Wide-column:  rowkey | col:name | col:age | col:lastLogin@ts ...
               42     | Ada      | 36      | 2026-06-01

 Graph:        (Ada) -[:FOLLOWS]-> (Bob) -[:FOLLOWS]-> (Cara)
```

## Replication

Replication keeps copies of data on multiple nodes for **availability** (survive node loss), **read scalability** (serve reads from replicas), and **geo-locality** (replica near users).

### Primary-replica (leader-follower)

```
   writes ──▶ ┌─────────┐  async/sync replication   ┌──────────┐
              │ PRIMARY │ ─────────────────────────▶│ REPLICA 1│ ◀── reads
              │ (leader)│ ─────────────────────────▶│ REPLICA 2│ ◀── reads
              └─────────┘                            └──────────┘
   • All writes go to the primary. Reads can fan out to replicas.
   • Async replication: fast writes, but replicas LAG → stale reads.
   • Sync replication: no lag, but writes wait for replica ack (slower).
   • On primary failure: promote a replica (failover); risk of lost
     un-replicated writes (async) or a split-brain (two primaries).
```

**Replication lag** is the key gotcha: a user writes, then immediately reads from a replica that hasn't caught up → they don't see their own write. Fix with **read-your-writes** consistency (route that user's reads to the primary briefly, or to a replica known to be caught up).

### Multi-primary (multi-leader)

```
   writes ──▶ ┌──────────┐ ◀──▶ ┌──────────┐ ◀── writes
              │ PRIMARY A│      │ PRIMARY B│
              └──────────┘      └──────────┘
   • Multiple nodes accept writes (e.g. one per region for low write latency).
   • PROBLEM: write conflicts (same row edited in two regions). Need conflict
     resolution: last-write-wins (lossy), version vectors, CRDTs, or app logic.
   • Use: multi-region active-active, offline-capable clients.
```

## Sharding (horizontal partitioning across machines)

When data or write throughput exceeds one machine, **shard**: split rows across multiple databases, each holding a subset.

```
 RANGE SHARDING
   Shard by key ranges:  A-H → shard1,  I-P → shard2,  Q-Z → shard3
   + Efficient range scans (sequential keys land together).
   - HOTSPOTS: skewed key distribution overloads one shard (e.g. all recent
     timestamps → newest shard). Rebalancing ranges is manual.

 HASH SHARDING
   shard = hash(key) % N
   + Even distribution (good hash spreads load uniformly).
   - Range queries scatter across all shards. Resharding when N changes moves
     almost everything → use CONSISTENT HASHING (§6) to limit movement.

 GEO / DIRECTORY SHARDING
   Route by region (EU users → EU shard) or by a lookup table mapping
   key → shard (a directory service).
   + Data locality / compliance (GDPR); flexible.
   - Directory is a lookup hop and a potential SPOF; geo skew possible.
```

```
   ┌──────────┐     route by shard key (e.g. hash(userId))
   │  Router  │ ──────────┬──────────────┬──────────────┐
   └──────────┘           ▼              ▼              ▼
                     ┌────────┐     ┌────────┐     ┌────────┐
                     │ Shard0 │     │ Shard1 │     │ Shard2 │
                     │ users  │     │ users  │     │ users  │
                     │ 0..33% │     │34..66% │     │67..100%│
                     └────────┘     └────────┘     └────────┘
```

**Choosing a shard key** is the hardest part: it must (1) distribute load evenly, (2) keep related data together to avoid cross-shard queries, (3) avoid hotspots. Cross-shard joins and transactions are painful — denormalize or avoid them. The **celebrity problem** is a sharding hotspot: one user's data (millions of followers) overwhelms its shard; mitigate with further sub-sharding or caching.

## Partitioning vs sharding

```
 PARTITIONING: splitting a table within ONE database instance (by range/hash/
   list) so the engine scans less. Still one machine.
 SHARDING: splitting across MULTIPLE database instances/machines. A type of
   partitioning that crosses machine boundaries → adds a routing layer.
```

## Indexing — B-tree vs LSM-tree

An index is an auxiliary data structure that turns an O(n) table scan into an O(log n) lookup. The two dominant on-disk index structures have opposite trade-offs.

### B-tree / B+tree (read-optimized) — used by SQL engines

```
 A balanced tree of disk pages; leaves hold (key → row pointer), sorted.
 Lookups, range scans, and ordered iteration are all O(log n).

                 [ 50 | 100 ]                 ← root (internal node)
                /     |      \
         [10|30]   [60|80]   [120|150]        ← internal nodes
         / | \      / | \      / | \
       leaves (sorted keys → row pointers); leaves linked for range scans

 + Excellent reads, range queries, ordered scans. Predictable.
 - Writes do in-place updates → random I/O; page splits; write amplification.
 Best for: read-heavy, range queries, OLTP (Postgres, MySQL/InnoDB).
```

### LSM-tree (write-optimized) — used by Cassandra, RocksDB, LevelDB, Bigtable

```
 Writes go to an in-memory MEMTABLE (+ WAL for durability). When full, it's
 flushed to disk as an immutable, sorted SSTABLE. Background COMPACTION merges
 SSTables and drops deleted/overwritten keys.

   write → [ MemTable (RAM, sorted) ] ──flush──▶ [ SSTable L0 ] ─┐
              + WAL (durability)                  [ SSTable L0 ] ─┤ compaction
                                                  [ SSTable L1 ] ◀┘ merges & sorts

 READ: check MemTable, then SSTables newest→oldest (BLOOM FILTERS, §17, skip
       SSTables that can't contain the key). Slower reads (multiple files).

 + Sequential writes only → very high write throughput, less write amplification.
 - Reads may touch many SSTables (mitigated by bloom filters + compaction).
 - Compaction consumes background I/O.
 Best for: write-heavy workloads (logging, time-series, IoT, messaging).
```

```
 ┌───────────────────────────┬───────────────────────────┐
 │ B-tree                    │ LSM-tree                  │
 ├───────────────────────────┼───────────────────────────┤
 │ Read-optimized            │ Write-optimized           │
 │ In-place updates (random) │ Append-only (sequential)  │
 │ Range scans: excellent    │ Range scans: good         │
 │ Write amplification: high │ Write amplification: low   │
 │ Read path: 1 tree         │ Read path: many SSTables   │
 │ Postgres, MySQL, Oracle   │ Cassandra, RocksDB, HBase │
 └───────────────────────────┴───────────────────────────┘
```

### Read vs write optimization (the mental summary)

```
 Read-heavy  → B-tree indexes, read replicas, caching, denormalize for reads.
 Write-heavy → LSM-tree store, write sharding, write-behind cache, async/batch.
```

> **Interview talking points:** "Given a read-heavy, relational workload with transactional integrity needs, I'd start with PostgreSQL: B-tree indexes on the query columns, read replicas to scale reads, and table partitioning by time. If write volume forces horizontal scale I'd shard by userId using consistent hashing, accepting that cross-user queries become scatter-gather. For a write-heavy event stream I'd reach for a wide-column LSM store like Cassandra instead."

**Common follow-up questions:**
1. *"SQL or NoSQL here, and why?"* → Tie it to: relational + transactions + consistency → SQL; massive write scale + flexible schema + simple access → NoSQL.
2. *"What's your shard key and why?"* → Even distribution, locality, no hotspots; name the cross-shard query cost.
3. *"How do you handle a hot shard / celebrity?"* → Sub-shard, cache, replicate the hot key.
4. *"B-tree vs LSM — which and why?"* → Read-heavy → B-tree; write-heavy → LSM; explain the I/O pattern.
5. *"How do replicas stay consistent? What about replication lag?"* → Async lag → stale reads; fix with read-your-writes routing or sync/semi-sync for critical reads.
6. *"How do you do a cross-shard transaction?"* → Avoid if possible; otherwise 2PC (slow, blocking) or Saga (§22) with compensations.
7. *"What index would you add for this query?"* → A composite index matching the WHERE + ORDER BY columns (left-prefix rule).

---

# 6 — Consistent Hashing
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
Consistent hashing is a technique for distributing keys across nodes such that **adding or removing a node moves only ~1/N of the keys**, not nearly all of them. It's the foundation of distributed caches (Memcached client-side, Redis Cluster), sharded databases (Cassandra, DynamoDB), and any system that must scale its node count without a full data reshuffle.

## WHEN to use
Whenever you distribute keys (cache entries, DB rows, sessions) across a *changing* set of nodes and you want membership changes (scale-up, node failure) to disrupt as few keys as possible. The classic motivation: simple modulo sharding `hash(key) % N` breaks catastrophically when N changes.

## The problem with naive modulo hashing

```
 N = 4 nodes:  node = hash(key) % 4
   key "k1" hash 100 → 100 % 4 = node 0
   key "k2" hash 101 → 101 % 4 = node 1

 Remove one node (N=3): node = hash(key) % 3
   key "k1" hash 100 → 100 % 3 = node 1   (MOVED!)
   key "k2" hash 101 → 101 % 3 = node 2   (MOVED!)

 Almost EVERY key remaps. For a cache: a mass miss → stampede on the DB.
 For a DB: nearly all data must be physically relocated. Catastrophic.
```

## HOW it works internally — the ring

Map both nodes and keys onto a circular hash space (e.g. 0 .. 2^32−1). A key belongs to the **first node found walking clockwise** from the key's position.

```
            0 / 2^32
              ┌───────────┐
       Node C │           │ Node A
        (270°)│           │ (45°)
              │     ●k1   │   k1 → walk CW → lands on Node A
              │           │
              │   k2●     │   k2 → walk CW → lands on Node B
       Node B │           │
        (180°)└───────────┘
                  (ring)

  Each node owns the arc from the PREVIOUS node (CCW) up to itself.
```

When a node is **added** at some ring position, it takes over only the arc between it and its clockwise predecessor — i.e. it steals keys from exactly **one** neighbor. When a node is **removed**, only its keys move to the next node clockwise. Either way: ~1/N of keys move, not all.

```
 ADD Node D between B and C:
   Only keys in (B, D] move from C to D. All other keys stay put.

   before:  ...B────────────C...   (C owns the whole arc B→C)
   after:   ...B────D────────C...   (D owns B→D, C now owns D→C only)
                └ only these keys move ┘
```

## Virtual nodes (the crucial refinement)

With few physical nodes, the ring arcs are uneven → **load imbalance** (one node owns a huge arc). Fix: each physical node is placed at **many** positions on the ring using `hash(nodeId + i)` for i = 1..V. These are **virtual nodes** (vnodes).

```
 Without vnodes (3 nodes): arcs can be wildly uneven.
   A:■■■■■■■■■■■■■■  B:■■■  C:■■■■■■   ← A overloaded

 With vnodes (each node placed ~100–200 times):
   A C B A B C A C B A C B A B ...     ← interleaved → near-uniform load
   Each node owns ~1/N of the ring in aggregate (law of large numbers).

 BONUS: when a node dies, its vnodes are scattered, so its load is redistributed
 ACROSS ALL remaining nodes evenly — not dumped entirely on one neighbor.
```

Typical V = 100–256 virtual nodes per physical node. The standard deviation of load drops as V grows.

## Worked example

```
 Ring space 0..359 (degrees). Physical nodes hashed to:
   A → 45,   B → 180,   C → 270
 Keys hashed to:
   k1 → 30,  k2 → 100,  k3 → 200,  k4 → 300

 Assign each key to the first node clockwise:
   k1(30)  → next CW node ≥ 30  is A(45)   → A
   k2(100) → next CW node ≥ 100 is B(180)  → B
   k3(200) → next CW node ≥ 200 is C(270)  → C
   k4(300) → next CW node ≥ 300 wraps to A(45) → A

 Now ADD node D → 120:
   Only keys in (45, 120] move. k2(100) was → B(180); now D(120) sits before it.
   k2(100) → next CW ≥ 100 is D(120) → D   (MOVED B→D)
   Everything else (k1,k3,k4) UNCHANGED. Just 1 key moved. ✅
```

## TRADE-OFFS

```
 + Minimal reshuffling: ~K/N keys move on a node add/remove (vs ~all with modulo).
 + Incremental scaling: add capacity one node at a time.
 + With vnodes: even load + even redistribution on failure.
 - More complex than modulo. Need to store/lookup the ring (sorted map / BST).
 - Lookup is O(log V·N) (binary search the ring) vs O(1) for modulo.
 - Without vnodes, load can be skewed.
 - Range queries still scatter (keys are hashed, not ordered).
```

> **Interview talking points:** "I'd distribute cache keys (and DB shards) with consistent hashing so adding a node only remaps ~1/N of keys instead of triggering a mass cache miss and DB stampede. I'd use ~150 virtual nodes per physical node so load stays even and a node failure redistributes across all survivors, not just the clockwise neighbor."

**Common follow-up questions:**
1. *"Why not just hash(key) % N?"* → Changing N remaps almost everything; consistent hashing remaps ~1/N.
2. *"What are virtual nodes for?"* → Even load distribution and even redistribution on failure; without them arcs are skewed.
3. *"How do you look up which node owns a key?"* → Binary search a sorted structure of ring positions for the first node clockwise.
4. *"How does this handle a node failure?"* → Its (virtual) ranges pass to the next nodes clockwise; with vnodes that load spreads across all survivors.
5. *"What about replication on the ring?"* → Store each key on the next R nodes clockwise (preference list) — this is how Dynamo/Cassandra replicate.

---

# 7 — CAP Theorem, PACELC & Consistency Models
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT — CAP
The CAP theorem states that a distributed data store can provide at most **two** of three guarantees simultaneously:

```
 C — Consistency:  every read sees the most recent write (or an error).
                   (This is "linearizability," NOT the C in ACID.)
 A — Availability: every request gets a (non-error) response, no guarantee
                   it's the latest data.
 P — Partition tolerance: the system keeps working despite network partitions
                   (messages dropped/delayed between nodes).
```

The subtle, correct framing: **in a distributed system, network partitions are a fact of life, so P is mandatory.** The real choice, *during a partition*, is between **C and A**:

```
 NETWORK PARTITION: nodes can't talk to each other.
   ┌────────┐   X (link down)   ┌────────┐
   │ Node 1 │ ─────╳──────────── │ Node 2 │
   └────────┘                    └────────┘

 A write hits Node 1. Node 2 can't be updated. A read hits Node 2. Now:
   • CP choice: Node 2 REFUSES/errors the read (can't guarantee freshness)
                → consistent but NOT available.
   • AP choice: Node 2 SERVES stale data (responds anyway)
                → available but NOT consistent.
```

```
              CAP triangle (pick the trade-off DURING a partition)
                         C
                        / \
                  CP   /   \  CA  (CA = single-node / no partition; not realistic
                      /     \      for distributed systems)
                     A───────P
                        AP
```

### Concrete system classifications

```
 CP (consistency over availability during partition):
   • HBase, MongoDB (default), Zookeeper, etcd, Spanner (effectively),
     traditional RDBMS in a cluster.
   • Use when correctness > uptime: banking, locks, config, leader election.

 AP (availability over consistency during partition):
   • Cassandra, DynamoDB (default), Riak, CouchDB.
   • Use when uptime > freshness: shopping carts, social feeds, metrics.
   • Reconcile later (eventual consistency, conflict resolution).

 "CA" is a misnomer for distributed systems — a single node has no partition to
   tolerate. Don't claim a real distributed system is CA.
```

## WHAT — PACELC (the more complete model)

CAP only describes behavior *during* a partition. **PACELC** extends it: **if** there's a **P**artition, choose **A** or **C**; **E**lse (normal operation), choose **L**atency or **C**onsistency.

```
 PACELC:  if (Partition)  then choose A or C
          else            then choose L or C    (Latency vs Consistency)

 Why ELSE matters: even with NO partition, synchronous replication for strong
 consistency adds latency (wait for replica acks). You trade speed for freshness
 every single request, not just during failures.
```

```
 System examples (PACELC notation):
   DynamoDB / Cassandra : PA/EL  → favors availability & low latency, eventual consistency.
   MongoDB              : PA/EC  → available under partition, consistent otherwise.
   HBase / BigTable     : PC/EC  → consistent always (pays latency & may reject).
   Spanner              : PC/EC  → strong consistency always (TrueTime; pays latency).
```

## Consistency models (the spectrum)

Consistency isn't binary — it's a spectrum from strongest (most expensive) to weakest (cheapest, most available).

```
 STRONG (linearizable):
   Every read returns the latest committed write, as if there were one copy.
   Cost: coordination/consensus per operation; higher latency; less available.
   Use: bank balance, inventory count, distributed lock.

 SEQUENTIAL / CAUSAL:
   Causally related operations are seen in order by everyone; concurrent ops
   may differ. (If A replies to B's comment, no one sees the reply before B's.)
   Cheaper than strong, preserves "makes sense" ordering.

 READ-YOUR-WRITES (a session guarantee):
   A user always sees their OWN writes immediately (others may lag).
   Implement: route a user's reads to the primary (or a caught-up replica)
   for a short window after they write.
   Use: "I posted and it vanished" bug — the classic replication-lag fix.

 MONOTONIC READS (session guarantee):
   Once you've seen a value, you never see an OLDER one (no going back in time).
   Implement: pin a user to one replica (sticky reads).

 EVENTUAL:
   If writes stop, all replicas eventually converge. No ordering/timing promise.
   Cheapest, most available. Use: feeds, like counts, DNS, view counts.
```

```
  Strength / cost
   high ▲  Strong (linearizable)        ← consensus, slow, CP
        │  Causal
        │  Read-your-writes / Monotonic  ← session guarantees, cheap fixes
   low  │  Eventual                      ← fast, available, AP
        └────────────────────────────▶ availability / performance
```

### Quorum tie-in

You can *tune* consistency in a quorum system (see §19): with N replicas, if **W + R > N** (write quorum + read quorum exceed replica count), reads and writes always overlap on at least one node → strong consistency. Lower W and R → faster but eventual. This is exactly the dial Cassandra/Dynamo expose.

> **Interview talking points:** "Partitions are inevitable, so the real question is what we do during one. For a bank ledger I'd choose CP — reject rather than serve stale balances. For a social feed I'd choose AP — better to show a slightly stale timeline than an error. And PACELC reminds me that even with no partition, strong consistency costs latency on every request, so I'd default to eventual consistency with read-your-writes session guarantees for the user's own actions."

**Common follow-up questions:**
1. *"CAP — explain the real trade-off."* → P is mandatory; choose C or A *during a partition*.
2. *"Is this system CP or AP, and why?"* → Tie to correctness vs uptime priority for the use case.
3. *"What does PACELC add?"* → The latency-vs-consistency trade-off during *normal* operation.
4. *"How do you fix 'I posted but don't see it'?"* → Read-your-writes: route the author's reads to the primary briefly.
5. *"Can you have strong consistency and high availability?"* → Not during a partition (CAP); you tune via quorum (W+R>N) and accept the latency/availability cost.
6. *"Difference between the C in CAP and the C in ACID?"* → CAP-C = linearizability across replicas; ACID-C = constraints/validity within a transaction. Different concepts.

---

# 8 — Database Scaling Patterns
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
The ordered playbook for scaling a database as load grows. You apply these roughly in this sequence; each buys headroom at a cost. Knowing the *order* and *when to escalate* is the senior signal.

```
 Scaling ladder (apply in order, escalate only when needed):
  0. Vertical scaling      — bigger box. Simple, has a ceiling, expensive, SPOF.
  1. Caching               — offload reads (see §2). Cheapest big win.
  2. Read replicas         — scale reads horizontally.
  3. Functional federation — split DBs by feature/domain.
  4. Sharding              — split one dataset across machines (scale writes).
  5. Denormalization       — precompute to avoid joins/aggregates.
  6. CQRS                  — separate read & write models entirely.
```

## 1 — Read replicas (scale reads)

```
   writes ──▶ ┌─────────┐ ──replication──▶ ┌──────────┐ ◀── reads
              │ PRIMARY │ ──────────────▶  │ REPLICA 1│ ◀── reads
              └─────────┘ ──────────────▶  │ REPLICA 2│ ◀── reads
                                            └──────────┘
   App routes WRITES → primary, READS → replicas (round-robin).
   + Scales read-heavy workloads (most web apps) linearly with replicas.
   - Replication lag → stale reads (mitigate: read-your-writes).
   - Does NOT scale writes (all writes still hit one primary).
```

## 2 — Functional partitioning / federation (split by feature)

```
   Monolithic DB              →  Federated by domain
   ┌──────────────┐              ┌────────┐ ┌────────┐ ┌─────────┐
   │ users        │              │ Users  │ │ Orders │ │ Products│
   │ orders       │      →       │  DB    │ │  DB    │ │   DB    │
   │ products     │              └────────┘ └────────┘ └─────────┘
   └──────────────┘
   + Each DB scaled independently; smaller working sets; clearer ownership.
   - Cross-domain joins now happen in the app layer; no cross-DB transactions.
```

## 3 — Sharding (scale writes — see §5)

Split a single large dataset (e.g. `users`) across machines by a shard key. This is the only pattern here that scales *write* throughput, because writes now spread across many primaries. Cost: a routing layer, cross-shard query/transaction pain, rebalancing complexity (use consistent hashing, §6).

## 4 — Denormalization (trade space & write cost for read speed)

```
 Normalized (3NF): joins at read time.
   SELECT u.name, COUNT(t.id) FROM users u JOIN tweets t ...  ← join + aggregate

 Denormalized: store the computed value redundantly.
   users: { id, name, tweet_count }   ← maintained on each write/delete
   Read = single-row lookup, no join.
   + Fast reads (no joins/aggregates at query time).
   - Writes must update multiple places; data can diverge; more storage.
```

## 5 — CQRS (Command Query Responsibility Segregation)

Separate the **write model** (optimized for transactional integrity) from the **read model** (optimized, denormalized, possibly a different store). Writes emit events that asynchronously update read-optimized views.

```
   Commands (writes)                 Queries (reads)
        │                                 ▲
        ▼                                 │
   ┌──────────┐   events    ┌────────────────────────┐
   │ Write DB │ ──────────▶ │ Read store(s)          │
   │ (normalized,           │ (denormalized views,   │
   │  ACID, source          │  search index, cache —  │
   │  of truth)             │  tuned per query)       │
   └──────────┘             └────────────────────────┘
   + Read and write sides scale & optimize independently; great for complex
     read patterns over a transactional core (often with event sourcing).
   - Complexity; the read side is EVENTUALLY consistent (event lag); two models
     to maintain. Use only when read/write needs genuinely diverge.
```

> **Interview talking points:** "I'd scale in order: cache first for the cheapest read offload, then read replicas for the read-heavy timeline, routing the author's own reads to the primary for read-your-writes. If write volume outgrows one primary I'd shard by userId. For the home-timeline read, which is an expensive fan-in, I'd denormalize — precompute each user's timeline (fan-out-on-write) so a read is a single cache lookup. That's effectively CQRS: a write-optimized tweet store and a read-optimized timeline view kept in sync asynchronously."

**Common follow-up questions:**
1. *"In what order do you scale?"* → Cache → read replicas → federation → shard → denormalize/CQRS.
2. *"Read replicas don't scale writes — then what?"* → Shard the write path.
3. *"What's the cost of denormalization?"* → Write amplification + risk of divergence; you maintain consistency in app logic or via events.
4. *"When is CQRS justified?"* → When read and write workloads diverge enough that one model can't serve both well; accept eventual consistency on reads.
5. *"How do you keep the read model fresh?"* → Stream write events to update read views; monitor and bound the lag.

---

# 9 — Rate Limiting
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
Rate limiting caps how many requests a client (user, IP, API key) can make in a time window. It protects services from abuse, accidental floods, scrapers, and cascading overload, and enforces fair usage / billing tiers. The HTTP signal is **429 Too Many Requests**, usually with a `Retry-After` header.

## WHEN to use
- Public APIs (per-key quotas), login endpoints (brute-force defense), expensive operations, any shared resource that must be protected from a single noisy client. In a distributed system the limiter state lives in a shared store (Redis) so all app servers enforce one global limit.

## The five core algorithms

### 1 — Token bucket (the most common; allows bursts)

```
 A bucket holds up to CAPACITY tokens; tokens refill at RATE per second.
 Each request consumes 1 token. Empty bucket → reject (429).

   refill at R tokens/sec
        │
        ▼
   ┌─────────────┐   request takes 1 token
   │ ●●●●● (5/10)│ ───────────────────────▶ allow if ≥1 token, else 429
   └─────────────┘

 capacity=10, rate=1/s: client can burst 10 instantly, then sustain 1/s.
 + Allows bursts up to capacity (good UX), smooth average rate, memory O(1)/key.
 - Two params to tune (capacity, rate).
```

```
 # sketch (per key, stored in Redis)
 now = time()
 tokens = min(capacity, tokens + (now - last_refill) * rate)
 last_refill = now
 if tokens >= 1: tokens -= 1; allow
 else: reject (429)
```

### 2 — Leaky bucket (smooths output to a constant rate)

```
 Requests enter a FIFO queue (the bucket); they "leak" out at a FIXED rate.
 Bucket full → reject. Output rate is perfectly constant (no bursts downstream).

   requests in (bursty) → ┌────────┐ → leak out at fixed rate (smooth)
                          │ queue  │
                          │ ▓▓▓▓   │ full → drop new
                          └────────┘
   + Smooths traffic to a steady rate (protects a downstream that hates bursts).
   - Bursts are delayed (queued) or dropped; needs a queue; less responsive.
```

### 3 — Fixed window counter

```
 Count requests per fixed clock window (e.g. per minute). Reset at boundary.
   12:00:00–12:00:59 → counter; at 12:01:00 → reset to 0.
   counter < limit → allow & increment; else 429.

   + Trivial: one counter + TTL per key. O(1).
   - BURST AT EDGES: limit=100/min. 100 reqs at 12:00:59 + 100 at 12:01:00 =
     200 requests in 1 second across the boundary. Up to 2x the limit briefly.
```

### 4 — Sliding window log

```
 Store a TIMESTAMP for every request (a sorted set). On each request, drop
 timestamps older than the window, then count what remains.
   allow if  count(timestamps in [now - window, now]) < limit.

   [t1 t2 t3 ... ] ← within last 60s? count them. Evict older.

   + EXACT — no edge burst; precise rolling window.
   - Memory O(limit) per key (stores every timestamp); expensive at scale.
```

### 5 — Sliding window counter (the practical sweet spot)

```
 Approximate the sliding window using the current + previous fixed-window
 counts, weighted by overlap. O(1) memory, ~no edge burst.

   count ≈ curWindowCount
         + prevWindowCount * (overlap fraction of previous window)

   e.g. 30% into the current minute: weight previous window by 70%.
   estimate = cur + prev * 0.7

   + Near-exact, O(1) memory, smooths the fixed-window edge burst.
   - Slight approximation (assumes uniform distribution in the prev window).
   THIS is what most production limiters (Cloudflare, etc.) use.
```

### Comparison

```
 Algorithm              Bursts        Memory/key   Accuracy   Edge burst
 ──────────────────────────────────────────────────────────────────────
 Token bucket           up to cap     O(1)         good       no
 Leaky bucket           none (smooth) O(queue)     good       no
 Fixed window           edge bursts   O(1)         coarse     YES (2x)
 Sliding window log     none          O(limit)     exact      no
 Sliding window counter near-none     O(1)         very good  near-no
```

## Distributed rate limiting

```
 Multiple app servers must share ONE limit. Put the counter/bucket in Redis:
   • Atomic ops (INCR, or a Lua script for token bucket) avoid race conditions.
   • One round trip to Redis per request (latency cost).
 Optimization: per-node local limits + periodic sync (slightly leaky but fast),
   or sticky routing so a key's requests land on one node.
```

> **Interview talking points:** "I'd use a token bucket in Redis keyed by API key — it allows reasonable bursts while enforcing an average rate, with O(1) state per client. I'd implement it as an atomic Lua script to avoid races across app servers, return 429 with Retry-After, and expose limit/remaining headers. If I needed strict smoothing into a fragile downstream I'd switch to leaky bucket; for precise limits without the fixed-window edge burst, sliding-window counter."

**Common follow-up questions:**
1. *"Which algorithm and why?"* → Token bucket for bursty-but-bounded APIs; sliding window counter for precise limits; leaky bucket to smooth a downstream.
2. *"Fixed window's flaw?"* → 2x burst across the boundary.
3. *"How do you enforce one limit across many servers?"* → Centralized Redis counter with atomic ops / Lua; or local + sync.
4. *"What do you return when limited?"* → 429 Too Many Requests + Retry-After; rate-limit headers.
5. *"Rate limit by what key?"* → User/API key for fairness; IP for anonymous; layer multiple (per-user AND per-IP AND global).
6. *"What about the Redis hop latency / Redis failure?"* → Fail-open (allow) vs fail-closed (reject) decision; local fallback limits.

---

# 10 — API Design
**Difficulty:** 🟡 · **Frequency:** 🔥🔥

## WHAT
How clients talk to your services. The interview tests whether you can choose between REST / GraphQL / gRPC with reasons, design idempotent and paginated endpoints, version safely, and use the right status codes. Good API design is about **clear contracts, evolvability, and correctness under retries**.

## REST vs GraphQL vs gRPC

```
 ┌──────────────┬─────────────────┬──────────────────┬───────────────────┐
 │              │ REST            │ GraphQL          │ gRPC              │
 ├──────────────┼─────────────────┼──────────────────┼───────────────────┤
 │ Style        │ Resources +     │ Single endpoint, │ RPC (call methods)│
 │              │ HTTP verbs      │ query language   │                   │
 │ Transport    │ HTTP/1.1 + JSON │ HTTP + JSON      │ HTTP/2 + Protobuf │
 │ Payload      │ Fixed per route │ Client picks     │ Binary (compact)  │
 │              │                 │ exact fields     │                   │
 │ Over/under-  │ Over/under-     │ Solves it (ask   │ Fixed (defined in │
 │  fetching    │ fetch common    │ for what you need)│ .proto)          │
 │ Round trips  │ Many (n+1)      │ One (nested)     │ One; streaming    │
 │ Streaming    │ No (SSE/WS bolt-│ Subscriptions    │ Native bi-di      │
 │              │ on)             │                  │ streaming         │
 │ Caching      │ Easy (HTTP/URL) │ Hard (POST)      │ Hard (binary)     │
 │ Schema/typing│ Loose (OpenAPI) │ Strong (SDL)     │ Strong (Protobuf) │
 │ Browser-     │ Excellent       │ Good             │ Needs gRPC-web    │
 │  friendly    │                 │                  │ proxy             │
 └──────────────┴─────────────────┴──────────────────┴───────────────────┘
```

**Pick REST** for public, cacheable, resource-oriented APIs with broad client support — the default. **Pick GraphQL** when clients (esp. mobile, varied UIs) need flexible, precise data shapes and you want to avoid over/under-fetching and n+1 round trips; cost is caching complexity and resolver/N+1 risk on the server. **Pick gRPC** for internal service-to-service communication where you want low latency, compact binary payloads, strong typing, and streaming — not browser-facing without a proxy.

## Idempotency

An operation is **idempotent** if performing it multiple times has the same effect as once. Critical because networks retry, and a retried "charge card" must not double-charge.

```
 GET, PUT, DELETE  → idempotent by HTTP spec.
 POST              → NOT idempotent (creates a new resource each time).

 Make POST safe with an IDEMPOTENCY KEY (see §15):
   Client sends Idempotency-Key: <uuid>. Server stores key→result. A retry with
   the same key returns the stored result instead of acting again.

   POST /payments  Idempotency-Key: abc-123   → charge $10, store result
   (timeout, client retries)
   POST /payments  Idempotency-Key: abc-123   → server sees key → returns same
                                                 result, does NOT charge again.
```

## Pagination — offset vs cursor

```
 OFFSET pagination:  GET /items?limit=20&offset=40   (page 3)
   SQL: ... LIMIT 20 OFFSET 40
   + Simple; can jump to any page.
   - SLOW at deep offsets (DB scans & discards offset rows → O(offset)).
   - INCONSISTENT: if rows are inserted/deleted between pages, items shift —
     you can see duplicates or skip items.

 CURSOR (keyset) pagination:  GET /items?limit=20&after=<cursor>
   cursor encodes the last seen sorted key (e.g. last id/timestamp).
   SQL: ... WHERE id > :cursor ORDER BY id LIMIT 20
   + FAST at any depth (uses the index, O(limit)); STABLE under inserts/deletes.
   - Can't jump to an arbitrary page number; needs a stable sort key.
   → THE choice for infinite scroll / large feeds / APIs at scale.
```

## Versioning

```
 1. URI path:     /v1/users, /v2/users   ← most common, explicit, cache-friendly.
 2. Header:       Accept: application/vnd.api.v2+json   ← clean URLs, less visible.
 3. Query param:  /users?version=2       ← simple but messy for caching.

 Rule: NEVER break a published contract. Add fields (backward compatible) freely;
 removing/renaming/retyping fields requires a new version. Deprecate with a
 sunset timeline and headers (Deprecation, Sunset).
```

## HTTP status codes (the ones to know)

```
 2xx Success
   200 OK            — request succeeded (GET/PUT)
   201 Created       — resource created (POST)
   202 Accepted      — accepted for async processing (queued)
   204 No Content    — success, empty body (DELETE)
 3xx Redirection
   301 Moved Permanently / 304 Not Modified (cache revalidation)
 4xx Client error (the client must change something)
   400 Bad Request   — malformed/invalid input
   401 Unauthorized  — not authenticated (no/invalid credentials)
   403 Forbidden     — authenticated but not allowed
   404 Not Found
   409 Conflict      — version conflict / duplicate (idempotency, optimistic lock)
   422 Unprocessable Entity — well-formed but semantically invalid
   429 Too Many Requests — rate limited
 5xx Server error (the server failed; client may retry)
   500 Internal Server Error
   502 Bad Gateway / 503 Service Unavailable / 504 Gateway Timeout
```

The key distinction: **4xx = caller's fault, don't blindly retry** (except 429/maybe 408); **5xx = server's fault, safe to retry with backoff** (if idempotent).

> **Interview talking points:** "I'd expose a REST API for public clients (cacheable, broadly supported) and use gRPC for internal service-to-service calls (binary, typed, low-latency, streaming). For feeds I'd use cursor pagination so deep pages stay fast and stable under inserts. Write endpoints that mutate money would take an Idempotency-Key so client retries don't double-apply. I'd version via the URI path and only ever add fields within a version."

**Common follow-up questions:**
1. *"REST or GraphQL or gRPC, and why?"* → Map to: public/cacheable → REST; flexible client data → GraphQL; internal/low-latency → gRPC.
2. *"How do you make a POST safe to retry?"* → Idempotency key stored server-side.
3. *"Why cursor over offset pagination?"* → Constant-time deep pages + stability under concurrent writes.
4. *"How do you version without breaking clients?"* → Additive changes within a version; new version for breaking changes; deprecation policy.
5. *"Which status code for X?"* → 409 conflict, 422 validation, 429 rate limit, 401 vs 403 (auth vs authz).
6. *"GraphQL n+1 problem?"* → Batch with DataLoader; depth/complexity limits to prevent abusive queries.

---

# 11 — Unique ID Generation
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
Generating globally unique IDs in a distributed system, ideally **without coordination** (no central bottleneck) and often **roughly time-sortable** (so IDs imply creation order — useful for cursor pagination and timeline ordering). The naive answer (a single DB auto-increment) is a single point of failure and a write bottleneck.

## Requirements to clarify
```
 • Uniqueness (mandatory).
 • Sortable by time? (k-sortable → good for feeds, indexing, pagination).
 • 64-bit (fits a BIGINT, compact) vs 128-bit (UUID)?
 • Throughput: how many IDs/sec, across how many nodes?
 • No coordination / no single point of failure?
```

## Option 1 — UUID

```
 UUID v4 (random): 122 random bits. 2^122 space → collisions astronomically rare.
   + Fully decentralized, zero coordination, generate anywhere instantly.
   - 128 bits (16 bytes) — large; stored as 36-char string is bigger still.
   - NOT sortable → random inserts fragment B-tree indexes (page splits, poor
     locality) → write amplification on the DB.

 UUID v7 (time-ordered, newer):
   [ 48-bit Unix ms timestamp | 74 random bits + version/variant ]
   + Time-sortable (sequential inserts → index-friendly) AND decentralized.
   + Modern best-of-both for many systems; replaces "v4 but unsortable" pain.
   - Still 128 bits; leaks creation time.
```

## Option 2 — Twitter Snowflake (64-bit, k-sortable) — the classic answer

A 64-bit ID composed of time + machine + sequence, generated locally with no coordination, sortable by time.

```
 64-bit Snowflake layout:
 ┌─┬───────────────────────────────────────┬──────────────┬───────────────┐
 │0│   41 bits: timestamp (ms since epoch)  │ 10 bits:     │ 12 bits:      │
 │ │                                        │ machine id   │ sequence #    │
 └─┴───────────────────────────────────────┴──────────────┴───────────────┘
  ▲ sign bit (always 0 → positive)

  • 1 bit sign  : kept 0 so IDs are positive.
  • 41 bits time: ms since a CUSTOM epoch → 2^41 ms ≈ 69 YEARS of IDs.
  • 10 bits node: 2^10 = 1024 machines/generators.
  • 12 bits seq : 2^12 = 4096 IDs per machine PER MILLISECOND
                  → 4096 * 1000 = ~4.1 MILLION IDs/sec PER machine.

  Total per ms across fleet: 1024 * 4096 ≈ 4.2M IDs/ms.
```

```
 Generation logic (per node):
   if same ms as last → seq++; if seq overflows (4096) → busy-wait to next ms.
   if new ms → seq = 0.
   id = (timestamp << 22) | (machineId << 12) | sequence

 + 64-bit (compact, fits BIGINT), TIME-SORTABLE (great for feeds & indexing),
   no coordination, ~4M/sec/node.
 - Clock dependency: clock skew/NTP rollback can produce duplicates or
   out-of-order IDs → must handle (wait, or refuse to go backward).
 - Need to assign unique machine IDs (via Zookeeper/config/etcd).
```

## Option 3 — Database ticket server

```
 A dedicated DB (or a few) hands out auto-increment IDs.
   + Simple, sortable, guaranteed unique.
   - SPOF & bottleneck (one row's lock). Mitigate with MULTIPLE servers using
     offset stepping: server A → 1,3,5,7 (odd), server B → 2,4,6,8 (even).
     (Flickr's approach.) Still a coordination/availability concern.
```

## Option 4 — Range / segment allocation (the pragmatic favorite)

```
 A central allocator hands each node a BLOCK (range) of IDs at a time; the node
 burns through it locally with no further coordination, then asks for the next.
   Node A gets [1..1000], Node B gets [1001..2000], ...
   + Amortizes coordination (1 call per 1000 IDs); sortable-ish; no per-ID hop.
   - IDs lost if a node dies mid-block (gaps — usually fine); central allocator
     must be HA. (This is how DB sequences with cache=N, and "Leaf-segment", work.)
```

## Comparison

```
 Scheme           Bits  Sortable  Coordination       Throughput     Note
 ──────────────────────────────────────────────────────────────────────────
 UUID v4          128   no        none               unlimited      index-unfriendly
 UUID v7          128   yes       none               unlimited      modern default
 Snowflake        64    yes       machine-id only    ~4M/node/sec   clock-dependent
 DB ticket        64    yes       central DB         limited        SPOF risk
 Range allocation 64    ~yes      periodic (batched) very high      gaps on crash
```

> **Interview talking points:** "If I need compact, time-sortable 64-bit IDs at scale with no per-ID coordination, I'd use Snowflake: 41 bits of timestamp for ~69 years and rough sortability, 10 bits of machine ID for 1024 generators, 12 bits of sequence for 4096 IDs/ms/node — about 4 million IDs/sec/node. I'd handle clock skew by refusing to emit IDs if the clock moves backward. If I just need uniqueness without a machine-ID registry, UUID v7 gives decentralized, time-sortable IDs at the cost of 128 bits."

**Common follow-up questions:**
1. *"Why not a DB auto-increment?"* → SPOF + write bottleneck; doesn't scale across nodes/regions.
2. *"Walk me through Snowflake's 64 bits."* → 1 sign + 41 time + 10 machine + 12 sequence; recite the capacities.
3. *"What breaks Snowflake?"* → Clock skew / NTP rollback → duplicates; handle by waiting or refusing to regress.
4. *"Why is sortability useful?"* → Cursor pagination, timeline ordering, index locality (sequential inserts).
5. *"UUID v4 vs v7?"* → v4 random (bad for B-tree index locality); v7 time-ordered (index-friendly).
6. *"How do nodes get unique machine IDs?"* → Zookeeper/etcd/config registry assigns them.

---

# 12 — Blob / Object Storage
**Difficulty:** 🟢 · **Frequency:** 🔥🔥

## WHAT
Object storage (S3, GCS, Azure Blob) stores arbitrarily large unstructured files ("objects") — images, video, backups, logs — as **key → blob + metadata** in flat **buckets** (no real directory tree; "folders" are key prefixes). It is *not* a filesystem and *not* a database: no in-place edits (objects are replaced whole), no transactions, no queries on content. It offers near-infinite capacity, very high durability (S3 advertises 11 nines — 99.999999999%), and is cheap per GB.

## WHEN to use
- Storing user uploads (photos, videos, documents), static website assets, data-lake files, backups, ML datasets — anything large and blobby.
- **Never** store large blobs in your relational DB (bloats the DB, kills cache hit ratios, slows backups). Store the blob in object storage and keep only the **URL/key + metadata** in the DB.

```
   ┌──────────┐   metadata (small)    ┌──────────────┐
   │ Database │ row: {id, owner,      │ "the blob's  │
   │          │       s3_key, size}   │  pointer"    │
   └──────────┘                       └──────────────┘
   ┌──────────────────────┐
   │ Object store (bucket)│ key "uploads/u42/img.jpg" → [ bytes... ] + metadata
   └──────────────────────┘
   ┌──────┐  serves the blob globally, cached at edge
   │ CDN  │ ◀── origin = object store
   └──────┘
```

## HOW it works — the S3 model

```
 • Bucket: a globally-named container. Object: key + value(bytes) + metadata.
 • Flat namespace: key "a/b/c.jpg" is one string; "/" is just a convention.
 • Immutable objects: "edit" = overwrite the whole object (new version).
 • Versioning: keep old versions on overwrite/delete (recover, audit).
 • Storage classes: Standard (hot) → Infrequent Access → Glacier (cold/archive),
   trading retrieval latency/cost for storage cost.
 • Strong read-after-write consistency (modern S3) for new objects.
 • Eventually-consistent listing historically; access control via IAM/bucket
   policies and PRESIGNED URLs.
```

## Presigned URLs (the key interview concept)

To let a client upload/download directly to/from the object store **without proxying bytes through your servers** (which would waste bandwidth and bottleneck), the server generates a **presigned URL**: a time-limited, signed URL granting temporary permission for one specific operation on one specific key.

```
 UPLOAD flow (client → object store directly):
   1. Client → App:  "I want to upload avatar.jpg"
   2. App  → Client: presigned PUT URL (expires in 5 min, scoped to that key)
      (App also records intended metadata in DB)
   3. Client → Object Store:  PUT bytes directly to the presigned URL
   4. Object store → (event) → App confirms / finalizes metadata

   ┌────────┐ 1.request   ┌─────┐
   │ Client │ ───────────▶│ App │ generates signed URL (no bytes through App)
   │        │ ◀───────────│     │
   │        │ 2.url       └─────┘
   │        │ 3.PUT bytes ┌──────────────┐
   │        │ ───────────▶│ Object Store │
   └────────┘             └──────────────┘
```

```
 + Offloads bandwidth from your servers (clients talk to S3 directly).
 + Fine-grained, expiring, least-privilege access — no permanent credentials
   handed to the client.
 + Same pattern for downloads (presigned GET) of private objects.
```

## Multipart upload (for large files)

```
 Large file (GBs) → split into PARTS uploaded in PARALLEL & independently, then
 the store assembles them into one object on completion.

   file → [part1][part2][part3]...[partN]   (each ≥5MB, uploaded concurrently)
   1. Initiate multipart upload → uploadId
   2. Upload each part (parallel) → each returns an ETag
   3. Complete: send the list of (partNumber, ETag) → store assembles the object

   + Parallelism → faster uploads of huge files; throughput scales with parts.
   + RESUMABLE: a failed part is re-uploaded alone, not the whole file.
   + Handles files larger than a single-PUT limit.
```

> **Interview talking points:** "User media goes to object storage, not the database — the DB only stores the key and metadata, and a CDN fronts the bucket for global delivery. Uploads use presigned PUT URLs so clients send bytes straight to S3, keeping that bandwidth off my app servers, and large files use multipart upload for parallel, resumable transfer. I'd enable versioning and lifecycle rules to tier cold objects to Glacier for cost."

**Common follow-up questions:**
1. *"Where do you store user-uploaded images?"* → Object storage; DB holds the key + metadata; CDN in front.
2. *"How does a client upload without overloading your servers?"* → Presigned URL → direct-to-S3 upload.
3. *"How do you handle a 5GB video upload?"* → Multipart upload (parallel, resumable parts).
4. *"How do you secure private objects?"* → Private bucket + presigned GET URLs with short expiry.
5. *"Why not store blobs in the DB?"* → Bloats DB, harms caching/backups, expensive; object storage is purpose-built and cheaper.

---

# 13 — Search (Inverted Index & Elasticsearch)
**Difficulty:** 🟡 · **Frequency:** 🔥🔥

## WHAT
Full-text search lets users find documents by the words they contain ("tweets mentioning 'world cup'"). A relational `LIKE '%term%'` cannot use an index and scans every row — unusable at scale. The solution is an **inverted index**, the data structure behind Elasticsearch, Solr, and Lucene.

## HOW it works — the inverted index

A normal ("forward") index maps document → its words. An **inverted** index flips it: **term → list of documents (and positions) containing it** (a "postings list").

```
 Documents:
   doc1: "the quick brown fox"
   doc2: "quick brown dogs"
   doc3: "lazy fox sleeps"

 Inverted index (term → postings list of docIds [+ positions/freq]):
   "quick" → [doc1, doc2]
   "brown" → [doc1, doc2]
   "fox"   → [doc1, doc3]
   "lazy"  → [doc3]
   ("the" dropped as a stop word)

 Query "quick fox":
   intersect postings("quick")=[1,2] with postings("fox")=[1,3] → [doc1]
   (AND = intersection; OR = union; ranked by relevance)
```

### Build pipeline (analysis)

```
 raw text → TOKENIZE (split into terms) → NORMALIZE (lowercase) →
 STOP-WORD removal ("the","a") → STEMMING/lemmatization ("running"→"run") →
 index each term → docId in the postings lists.

 Search applies the SAME analysis to the query so "Running" matches "run".
```

### Ranking — TF-IDF / BM25

```
 Not all matches are equal. Score documents by relevance:
  • TF  (term frequency): more occurrences in a doc → more relevant.
  • IDF (inverse document frequency): rare terms across the corpus →
        more discriminating → weighted higher ("the" is worthless, "quokka" gold).
  • BM25: the modern refinement (TF saturation + length normalization).
 Return the top-K by score.
```

## Elasticsearch architecture

```
   Index "tweets" split into SHARDS (for scale) each with REPLICAS (for HA):

   ┌──────────── Cluster ────────────────────────────────────┐
   │  Node 1            Node 2            Node 3              │
   │  [Shard0 primary]  [Shard1 primary]  [Shard2 primary]   │
   │  [Shard1 replica]  [Shard2 replica]  [Shard0 replica]   │
   └──────────────────────────────────────────────────────────┘
     • Index = logical search space, split into N shards (a Lucene index each).
     • Sharding: doc routed by shard = hash(routing/_id) % numShards.
     • Replicas: copies of shards for availability + read throughput.
     • Query = SCATTER to all relevant shards → each returns top-K →
               GATHER & merge-rank → final top-K (scatter-gather).
     • Writes go to a primary shard, replicated to replica shards.
     • NEAR-real-time: a "refresh" (default ~1s) makes new docs searchable.
```

```
 SCATTER-GATHER search:
   query → coordinating node → [shard0][shard1][shard2] (parallel)
                                  │ topK    │ topK   │ topK
                                  └────┬─────┴────────┘
                              merge & rerank → global topK → client
```

## TRADE-OFFS

```
 + Sub-second full-text search over huge corpora; relevance ranking; facets,
   aggregations, fuzzy/typo matching, autocomplete.
 - NOT a source-of-truth DB: eventual consistency, near-real-time (not instant),
   weaker durability/transactions. Keep the canonical data in your primary DB
   and feed Elasticsearch via a pipeline (CDC / queue) → CQRS-style read model.
 - Reindexing/mapping changes are heavy. Number of shards is hard to change later.
 - Memory hungry (indexes + caches in RAM).
```

> **Interview talking points:** "For search I'd use an inverted index via Elasticsearch, not LIKE queries. Tweets get analyzed (tokenize, lowercase, stop-words, stem) and indexed term→postings; queries intersect postings and rank by BM25. I'd shard the index by document ID for scale and add replicas for availability and read throughput, with queries doing scatter-gather across shards. Elasticsearch is a read model, not the source of truth — I'd keep canonical data in the primary DB and stream changes into the index, accepting near-real-time freshness."

**Common follow-up questions:**
1. *"Why not SQL LIKE?"* → Can't use an index, full scan, no relevance ranking; doesn't scale.
2. *"What's an inverted index?"* → term → postings list of docs; intersect for AND, union for OR.
3. *"How do you rank results?"* → TF-IDF / BM25 (term frequency × inverse doc frequency + length norm).
4. *"How does Elasticsearch scale?"* → Shards (split index) + replicas (HA/reads) + scatter-gather queries.
5. *"How does data get into the index?"* → Stream from the primary DB (CDC/queue); it's an eventually-consistent read model.
6. *"How do you support typo tolerance / autocomplete?"* → Fuzzy matching (edit distance), edge n-grams / completion suggester.

---

# 14 — Monitoring & Observability
**Difficulty:** 🟡 · **Frequency:** 🔥🔥

## WHAT
Observability is the ability to understand a system's internal state from its external outputs. It rests on **three pillars** — **metrics, logs, traces** — plus alerting. The senior interview move is to *close* your design with: "and here's how I'd know it's healthy and debug it when it isn't." Monitoring tells you *something* is wrong; observability lets you ask *why*.

## The three pillars

```
 METRICS  (aggregated numbers over time)
   Numeric time-series: QPS, p50/p99 latency, error rate, CPU, queue depth.
   Cheap to store, great for dashboards & alerting. "What is happening?"
   Tooling: Prometheus (pull-based scrape) + Grafana (visualize).

 LOGS  (discrete events with context)
   Timestamped records of individual events/errors with detail. Verbose,
   searchable. "What exactly happened in this request?"
   Tooling: ELK (Elasticsearch + Logstash + Kibana) or EFK (Fluentd).
   Structured (JSON) logs > plain text for querying.

 TRACES  (request flow across services)
   Follow ONE request across many services with timing per hop (spans).
   "WHERE is the latency / failure in this distributed call?"
   Tooling: Jaeger / Zipkin / OpenTelemetry. A trace_id threads the request
   through every service; each service adds a span.
```

```
 Distributed trace (one request, trace_id = T):
   [ API gateway        span 5ms ]
      [ auth service    span 2ms ]
      [ tweet service              span 40ms ]
         [ db query    span 30ms ]   ← the latency culprit is visible here
         [ cache get   span 1ms  ]
   Total = 47ms, and you can SEE the db query dominates.
```

## Prometheus model

```
 PULL-based: Prometheus periodically SCRAPES /metrics endpoints exposed by
 each service (counters, gauges, histograms). Stores as time-series labeled by
 dimensions (service, endpoint, status). PromQL queries it; Alertmanager fires
 alerts; Grafana dashboards it.

   service /metrics ◀── scrape every 15s ── Prometheus ──▶ Grafana / Alertmanager
   Metric types: Counter (monotonic, e.g. requests_total),
                 Gauge (up/down, e.g. queue_depth),
                 Histogram (buckets → percentiles, e.g. request_duration).
```

## The Four Golden Signals (Google SRE) — memorize these

```
 1. LATENCY    — how long requests take. Track DISTRIBUTIONS (p50/p95/p99),
                 NOT just the average (averages hide tail pain). Separate
                 success vs error latency (a fast 500 isn't "good").
 2. TRAFFIC    — demand on the system: requests/sec, QPS, transactions/sec.
 3. ERRORS     — rate of failed requests (5xx, timeouts, wrong results).
                 Track error RATE (errors / total), not just count.
 4. SATURATION — how "full" the system is: CPU, memory, disk, queue depth,
                 connection pool usage. The leading indicator of trouble.

 (RED method = Rate, Errors, Duration — a request-centric subset.
  USE method = Utilization, Saturation, Errors — resource-centric.)
```

## Alerting

```
 Alert on SYMPTOMS users feel (p99 latency > 200ms, error rate > 1%), not just
 causes (CPU 90%). Avoid alert fatigue: page only on actionable, urgent issues;
 everything else is a dashboard/ticket. SLI (measured) → SLO (target, e.g.
 99.9% < 200ms) → SLA (contractual promise + penalty). Error budget = the
 allowed unreliability (100% − SLO); spend it on shipping features.
```

> **Interview talking points:** "I'd instrument every tier with the four golden signals — latency (as p99, not average), traffic, errors, and saturation — scraped by Prometheus into Grafana dashboards. Structured logs go to an ELK stack for per-request debugging, and distributed tracing with OpenTelemetry/Jaeger lets me pinpoint which service or query owns the latency in a slow request. I'd alert on user-facing symptoms tied to SLOs, not raw CPU, to avoid alert fatigue."

**Common follow-up questions:**
1. *"What would you monitor?"* → The four golden signals per tier; recite them.
2. *"Why p99 not average latency?"* → Averages hide the tail; the slow 1% is often the most important (and most-loaded) users.
3. *"Metrics vs logs vs traces?"* → Aggregated numbers / discrete events / per-request cross-service flow.
4. *"How do you debug a slow distributed request?"* → Distributed tracing: follow the trace_id, find the span that dominates.
5. *"Push or pull metrics?"* → Prometheus pulls (scrapes); good for service discovery & dead-target detection. Push (statsd) for short-lived jobs.
6. *"What do you alert on?"* → Symptoms (SLO breaches), actionable + urgent only; tie to error budgets.

---

# Distributed Systems Essentials

The grab-bag of patterns that appear as follow-up "gotchas" in nearly every distributed-systems round. Each is short but complete — know the mechanism, the trade-off, and one diagram.

---

# 15 — Idempotency Keys
**Difficulty:** 🟡 · **Frequency:** 🔥🔥🔥

## WHAT
A client-supplied unique token (`Idempotency-Key: <uuid>`) that lets the server **deduplicate retried requests** so a network retry of a non-idempotent operation (charge a card, place an order) takes effect **exactly once**. This is how you achieve "effectively once" on top of at-least-once networks.

## HOW it works
```
   1. Client generates a key (UUID) for the operation, sends it with the request.
   2. Server checks a store (Redis/DB) for that key:
        • not seen → process, store key → {status, result}, return result.
        • seen + completed → return the STORED result (do NOT re-process).
        • seen + in-progress → 409 / wait (a concurrent retry).
   3. Key expires after a TTL (e.g. 24h).

   POST /charge  Idempotency-Key: K1  → charge $10, store K1→{ok, txn#7}
   (timeout; client retries)
   POST /charge  Idempotency-Key: K1  → key seen → return {ok, txn#7}, NO 2nd charge
```

```
   ┌────────┐  K1   ┌──────┐  K1 seen? ┌─────────────┐
   │ Client │ ─────▶│ App  │ ─────────▶│ Idempotency │
   │ (retry)│ ─────▶│      │           │   store     │
   └────────┘       └──────┘           └─────────────┘
              first → do work + store ; retry → replay stored result
```

## TRADE-OFFS
```
 + Safe retries; no double-charge / double-order; the foundation of reliable
   at-least-once + idempotent processing.
 - Server must store keys (memory/TTL); must handle the concurrent in-flight
   case atomically (insert key with a unique constraint / SET NX before work).
 - Key must be stable per logical operation (client generates once, reuses on retry).
```

> **Interview talking points:** "Every money-moving or order-creating endpoint takes an idempotency key. I store it atomically before doing the work (insert-if-absent), so a concurrent retry either replays the stored result or waits. That's how message consumers dedupe at-least-once delivery into effectively-once side effects."

**Common follow-ups:** *Where stored?* (Redis/DB with TTL). *Concurrent duplicates?* (atomic SET NX / unique constraint). *How long to keep keys?* (long enough to cover retry windows, e.g. 24h).

---

# 16 — The Thundering Herd
**Difficulty:** 🟡 · **Frequency:** 🔥🔥

## WHAT
A failure mode where a large number of processes/requests are all woken or unblocked at once and stampede a shared resource, overwhelming it. Two classic forms: **cache stampede** (a hot key expires → all misses hit the DB at once — see §2) and **retry storms** (many clients fail, all retry at the same instant, re-overloading a recovering service).

## HOW it manifests & mitigations
```
   resource recovers / key expires at t0
        │ thousands unblock simultaneously
        ▼
   ┌────────────── all hit at once ──────────────┐
   req req req req req req req req req req req req  →  ┌──────┐ overloaded again
   └──────────────────────────────────────────────┘    │ svc  │ → fails → more retries
                                                        └──────┘   (vicious cycle)

 Mitigations:
  • EXPONENTIAL BACKOFF + JITTER: retry after random(0, base*2^attempt). Jitter
    is the key — it spreads retries so they don't re-synchronize.
  • REQUEST COALESCING / single-flight: only one request rebuilds a missing
    value; others wait for it.
  • TTL JITTER: stagger cache expiry so keys don't all die together.
  • CIRCUIT BREAKER: stop hammering a failing dependency; fail fast, recover gently.
  • RATE LIMITING / load shedding at the entry point.
```

> **Interview talking points:** "Two herds to defend against: cache stampede (mitigate with single-flight + TTL jitter + stale-while-revalidate) and retry storms (mitigate with exponential backoff *with jitter* and circuit breakers). The jitter is what actually breaks the synchronization."

**Common follow-ups:** *Why jitter and not just backoff?* (without jitter, all clients back off the same amount and re-stampede in sync). *Circuit breaker states?* (closed → open on failures → half-open trial → closed).

---

# 17 — Bloom Filters
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
A space-efficient probabilistic data structure that answers **"is X *definitely not* in the set, or *possibly* in the set?"** It has **no false negatives** (if it says "not present," it's truly absent) but allows **false positives** (it may say "possibly present" when it isn't). It trades a small error rate for huge memory savings.

## HOW it works
```
 A bit array of m bits + k independent hash functions.
 ADD(x):    set bits at positions h1(x), h2(x), ..., hk(x) to 1.
 QUERY(x):  if ALL of h1(x)..hk(x) are 1 → "maybe present"
            if ANY is 0                  → "DEFINITELY not present"

 m=16 bits, k=2:
   add "cat"  → set bits 3, 11
   add "dog"  → set bits 7, 11
   index: 0 1 2 [3] 4 5 6 [7] 8 9 10 [11] 12 13 14 15
                 1           1            1
   query "cat" → bits 3 & 11 set → maybe (correct)
   query "cow" → hashes to 3 & 7 → both set (by cat+dog!) → FALSE POSITIVE
   query "ant" → hashes to 5 (=0) → DEFINITELY not present (correct, certain)
```

```
 False-positive rate ≈ (1 - e^(-k·n/m))^k. Tune m and k for target error.
 Rule of thumb: ~10 bits per element → ~1% false positive rate.
 No deletions (clearing a bit could remove others) → use a counting Bloom filter
 if deletes are needed.
```

## WHEN / TRADE-OFFS
```
 Use to AVOID an expensive lookup when the answer is usually "no":
  • LSM-tree / Cassandra: skip reading an SSTable that can't contain a key.
  • Caches/CDNs: "have we ever seen this URL?" before a costly origin fetch.
  • Databases: avoid disk seeks for non-existent keys.
  • Web: "has this user seen this article?" / dedup at scale.
 + Tiny memory (bits per element), O(k) constant-time ops.
 - False positives (must verify a "maybe" with the real lookup); no deletion;
   can't list members or get count.
```

> **Interview talking points:** "A bloom filter front-runs expensive lookups: it definitively rules out absent keys with a few bits per element. Cassandra uses one per SSTable so a read skips files that can't hold the key. The cost is a tunable false-positive rate — a 'maybe' still needs the real check — but never a false negative."

**Common follow-ups:** *False positive vs negative?* (FP possible, FN impossible). *Why no deletes?* (shared bits); use counting bloom filter. *Where used?* (LSM SSTable skip, cache existence checks).

---

# 18 — Write-Ahead Log (WAL)
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
A durability technique: **before** applying any change to the actual data files, **first append a record of that change to a sequential, on-disk log and fsync it.** If the system crashes, it replays the log on restart to recover committed changes. This is how databases provide the **D**urability in ACID (Postgres WAL, MySQL redo log, and the same idea underlies LSM commit logs, Kafka, etcd/Raft logs).

## HOW it works
```
   WRITE:  1. append change to WAL (sequential disk write) → fsync → ACK client
           2. LATER apply the change to data pages (in memory / lazily to disk)

   ┌────────┐  1.append+fsync  ┌──────────────┐
   │ Txn    │ ───────────────▶ │ WAL (on disk,│  durable BEFORE ack
   │        │                  │ append-only) │
   └────────┘                  └──────────────┘
        │ 2.apply (async, batched)
        ▼
   ┌──────────────┐
   │ Data pages   │ (can be flushed lazily; crash-safe because WAL has the truth)
   └──────────────┘

   CRASH RECOVERY: on restart, replay WAL from the last checkpoint → reapply
   committed changes, discard uncommitted. State is reconstructed exactly.
```

## WHY it works / TRADE-OFFS
```
 + Durability: a committed write survives a crash even if data pages weren't
   flushed yet — the log has it.
 + Performance: sequential appends are FAST (no random I/O on the hot path);
   data-file updates are batched/deferred.
 + Enables replication (ship the log to replicas) & point-in-time recovery.
 - The log grows → checkpointing + truncation needed.
 - Recovery replays the log (startup cost proportional to log since checkpoint).
```

> **Interview talking points:** "Durability comes from the write-ahead log: append the change to a sequential on-disk log and fsync before acknowledging, then apply to data pages lazily. A crash is recovered by replaying the WAL from the last checkpoint. The sequential append is also why this is fast, and shipping the WAL to replicas is how replication and point-in-time recovery work."

**Common follow-ups:** *Why log before data?* (so a crash mid-update is recoverable). *Why is it fast?* (sequential I/O vs random). *How does the log not grow forever?* (checkpoints + truncation). *Relation to replication?* (replicas apply the shipped log).

---

# 19 — Quorum (W + R > N)
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
In a replicated store with **N** copies of each item, a **quorum** is the minimum number of nodes that must acknowledge an operation for it to count. By requiring **W** nodes to ack a write and **R** nodes to be read, you tune consistency vs availability. The magic inequality: **if W + R > N, every read overlaps every write on at least one node → you always read the latest write (strong consistency).**

## HOW it works
```
 N = 3 replicas.  Choose W and R.

   W + R > N  → read & write sets OVERLAP → at least one read node has the
                latest write → STRONG consistency.

 Example N=3, W=2, R=2  (W+R=4 > 3):
   write to 2 of 3 nodes [A,B] ack.
   read from 2 of 3 nodes — any 2 of {A,B,C} MUST include A or B (the written
   ones), since only C is un-written. → read sees the latest value. ✅

   ┌─A(new)─┐   ┌─B(new)─┐   ┌─C(old)─┐
   write set = {A,B}; any read set of 2 intersects {A,B}. Guaranteed overlap.
```

```
 Tuning:
   W=N, R=1   → fast reads, slow/less-available writes; strong (W+R>N).
   W=1, R=N   → fast writes, slow reads; strong.
   W=1, R=1   → fastest, but W+R=2 ≤ 3 → EVENTUAL (may miss latest write).
   W=R=quorum (⌈(N+1)/2⌉) → balanced strong consistency, tolerate (N-1)/2 failures.

 This is exactly the dial Cassandra/DynamoDB expose (ONE, QUORUM, ALL).
```

## TRADE-OFFS
```
 + Tunable consistency↔availability↔latency per operation.
 + Tolerates node failures (don't need all N up — just W or R).
 - Higher W/R → more latency & less availability (more nodes must respond).
 - W+R ≤ N → eventual consistency (stale reads possible).
 - Concurrent writes still need conflict resolution (versions/vector clocks).
```

> **Interview talking points:** "With N replicas I tune W and R per operation. Setting W+R>N guarantees the read and write sets overlap, so a read always sees the latest write — strong consistency. For a balanced setup with N=3 I'd use W=R=2: tolerate one node down, strong consistency. If I want fast available writes and can tolerate staleness, W=1,R=1 gives eventual consistency."

**Common follow-ups:** *Why W+R>N?* (guaranteed overlap → read hits a written node). *N=3, W=2, R=2 — failures tolerated?* (one node down still works). *What if W+R≤N?* (eventual; stale reads). *Concurrent writes to the same key?* (resolve via version vectors / LWW).

---

# 20 — Heartbeats & Leader Election
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
**Heartbeats** are periodic "I'm alive" messages a node sends so others can detect when it dies (miss N heartbeats → presumed dead). **Leader election** is how a cluster agrees on a single coordinator (primary, the node that accepts writes / makes decisions) and picks a new one when the leader dies — without ending up with *two* leaders (split-brain).

## HOW it works
```
 HEARTBEAT failure detection:
   follower → leader:  ping every 1s. Miss 3 in a row (timeout) → "leader dead"
   → trigger election. (Tune interval vs detection-speed vs false positives.)

   ┌────────┐  heartbeat 1s   ┌────────┐
   │Follower│ ──────────────▶ │ Leader │   (silence > timeout → start election)
   └────────┘ ◀──────────────└────────┘

 LEADER ELECTION (consensus algorithms — Raft/Paxos):
   • Each node has a randomized election timeout (jitter avoids ties).
   • On timeout, a node becomes CANDIDATE, increments a term, requests votes.
   • A node grants ONE vote per term. Win a MAJORITY (quorum) → become leader.
   • Majority requirement prevents split-brain: two leaders can't both hold a
     majority of an odd-sized cluster.

   Raft states:  Follower ──timeout──▶ Candidate ──majority votes──▶ Leader
                    ▲                                                  │
                    └──────────── higher term seen / step down ────────┘
```

## WHY majority / quorum prevents split-brain
```
 5-node cluster, network splits into {A,B} and {C,D,E}.
   {A,B} = 2 nodes  → cannot get majority (3) → no leader, stays read-only/blocks.
   {C,D,E} = 3 nodes → majority → elects a leader → keeps serving.
 Only ONE side can have a majority → only one leader. (Use ODD node counts.)
```

## TRADE-OFFS
```
 + Automatic failover; no single hand-configured primary; survives node loss.
 - Election window = brief unavailability (no leader for a few timeouts).
 - Needs an odd quorum (3/5) — even counts can deadlock at a tie.
 - Heartbeat tuning: too aggressive → false failovers (flapping); too slow →
   long downtime to detect death.
 Tools: Zookeeper, etcd, Consul implement this so you don't roll your own.
```

> **Interview talking points:** "Nodes heartbeat the leader; missing a few timeouts triggers an election. I'd use Raft via etcd/Zookeeper rather than rolling my own: a candidate needs a majority vote to become leader, and the majority requirement is exactly what prevents split-brain — two partitions can't both hold a majority of an odd-sized cluster. The cost is a brief unavailability window during election."

**Common follow-ups:** *How detect a dead node?* (missed heartbeats / timeout). *How avoid two leaders?* (majority quorum; odd cluster size). *Why randomized timeouts?* (avoid split-vote ties). *Raft vs Paxos?* (same goal; Raft is understandable, term + log + majority).

---

# 21 — Distributed Locks
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
A lock that coordinates **mutual exclusion across machines** — only one process across the whole cluster may hold it at a time (e.g. "only one worker runs this cron job," "only one request mutates this account"). Unlike an in-process mutex, it must survive process crashes (auto-expire) and network issues.

## HOW it works
```
 REDIS lock (single-instance, common):
   acquire: SET lock:resource <uniqueToken> NX PX 30000
            (NX = only if absent; PX = auto-expire in 30s → no deadlock if holder dies)
   release: ONLY if the token matches (a Lua compare-and-delete), so you never
            delete someone else's lock.

   ┌────────┐ SET NX PX  ┌───────┐
   │Worker A│ ──────────▶│ Redis │  got it → do work → DEL (if my token)
   │Worker B│ ──────────▶│       │  NX fails → didn't get it → back off/retry
   └────────┘            └───────┘

 ZooKeeper/etcd lock (stronger):
   Create an EPHEMERAL sequential znode; lowest sequence number holds the lock;
   ephemeral node auto-vanishes if the client session dies → safe release.
```

## The danger: locks are not perfectly safe
```
 PROBLEM: a holder pauses (GC/STW) past the lock's TTL → lock expires →
 another worker acquires it → now TWO workers think they hold it → corruption.

 MITIGATIONS:
  • FENCING TOKENS: lock service returns a monotonically increasing token;
    the protected resource REJECTS any write with a token < the highest seen.
    Even if two holders exist, the stale one's writes are fenced off.
  • Keep critical sections short; set TTL > max expected work + safety margin.
  • Redlock (multi-Redis quorum) for higher availability — but contested for
    correctness; prefer a real consensus store (etcd/ZK) when correctness is key.
```

## TRADE-OFFS
```
 + Cross-machine mutual exclusion for non-idempotent critical sections.
 - HARD to make correct: TTL/pause races, clock issues, the holder dying.
 - A lock is a coordination bottleneck → serializes; avoid if you can.
 BEST ALTERNATIVE: design for idempotency (§15) or optimistic concurrency
   (compare-and-swap / version checks) so you don't NEED a lock.
```

> **Interview talking points:** "For cross-machine mutual exclusion I'd use a lock with an auto-expiring TTL and a unique token, releasing only via compare-and-delete so I never drop someone else's lock. But distributed locks are subtly unsafe — a GC pause past the TTL means two holders — so for correctness I'd add fencing tokens (the resource rejects stale tokens) or, better, avoid the lock entirely with idempotency or optimistic concurrency."

**Common follow-ups:** *Why a TTL?* (avoid deadlock if the holder dies). *Why a unique token?* (don't delete another's lock). *The GC-pause problem?* (fencing tokens). *Redis vs ZooKeeper for locks?* (ZK/etcd safer via consensus + ephemeral nodes; Redis simpler/faster but weaker). *Better than locking?* (idempotency / CAS).

---

# 22 — 2PC vs Saga (Distributed Transactions)
**Difficulty:** 🔴 · **Frequency:** 🔥🔥

## WHAT
How to keep a transaction **atomic across multiple services/databases** when there's no shared single database to give you ACID. Two competing approaches: **Two-Phase Commit (2PC)** — synchronous, blocking, strongly consistent — and the **Saga** pattern — a sequence of local transactions with compensating undo actions, asynchronous and eventually consistent.

## Two-Phase Commit (2PC)
```
 A coordinator drives an all-or-nothing commit across participants.

 Phase 1 (PREPARE): coordinator asks all → "can you commit?" Each does the work,
   locks resources, replies YES (vote-commit) or NO (vote-abort).
 Phase 2 (COMMIT/ABORT): if ALL voted yes → tell all to COMMIT; else → ABORT.

   Coordinator ──prepare──▶ [A] [B] [C]   all YES?
   Coordinator ──commit ──▶ [A] [B] [C]   (or abort if any NO)

 + Strong consistency / true atomicity across services.
 - BLOCKING: participants hold LOCKS through both phases (slow, low throughput).
 - Coordinator is a SPOF: if it dies after prepare, participants are stuck
   holding locks ("in doubt"). Doesn't scale; bad for microservices/high volume.
```

## Saga pattern
```
 Break the distributed transaction into a chain of LOCAL transactions, each
 with a COMPENSATING transaction that semantically undoes it. If a step fails,
 run the compensations for all completed steps (in reverse).

   Order saga:  [Create Order] → [Reserve Inventory] → [Charge Payment] → [Ship]
   If "Charge Payment" FAILS:
     compensate ← [Release Inventory] ← [Cancel Order]   (undo completed steps)

 Two coordination styles:
  • CHOREOGRAPHY: each service emits events others react to (no central brain).
    Decoupled, but flow is implicit/hard to trace at scale.
  • ORCHESTRATION: a central orchestrator tells each service what to do next and
    triggers compensations. Explicit, easier to reason about & monitor.

 + No long-held locks, no global blocking → scales; service autonomy; resilient.
 - EVENTUAL consistency (intermediate states are visible — an order exists
   before payment confirms). Compensations must be designed (and idempotent).
   No isolation: others can see in-between states. More complex to reason about.
```

## Comparison
```
 ┌──────────────────┬───────────────────────┬───────────────────────────┐
 │                  │ 2PC                   │ Saga                      │
 ├──────────────────┼───────────────────────┼───────────────────────────┤
 │ Consistency      │ Strong (atomic)       │ Eventual                  │
 │ Locking          │ Locks across phases   │ No global locks           │
 │ Availability     │ Low (blocking, SPOF)  │ High                      │
 │ Coupling         │ Tight (coordinator)   │ Loose (events/orchestrator)│
 │ Failure recovery │ Coordinator log/retry │ Compensating transactions │
 │ Scale            │ Poor                  │ Good (microservices)      │
 │ Isolation        │ Yes                   │ No (intermediate visible) │
 └──────────────────┴───────────────────────┴───────────────────────────┘
```

> **Interview talking points:** "Across microservices I avoid 2PC — it holds locks through both phases and the coordinator is a SPOF that leaves participants stuck in doubt; it doesn't scale. I'd use a Saga: a chain of local transactions, each with a compensating undo, coordinated by an orchestrator so the flow is explicit. The trade-off is eventual consistency and visible intermediate states, so I design idempotent compensations and surface 'pending' states to users. 2PC only when I truly need cross-service atomicity at low volume."

**Common follow-ups:** *Why avoid 2PC in microservices?* (blocking locks + coordinator SPOF + poor scale). *Saga's downside?* (eventual consistency, no isolation, must design compensations). *Choreography vs orchestration?* (decoupled events vs central controller). *What if a compensation itself fails?* (retry idempotently; alert; manual intervention as last resort).

---

# Numbers Every System Designer Must Memorize

The cheat sheet. If you can recite these and do the back-of-envelope arithmetic live, you signal seniority instantly. Practice the derivations until they're automatic.

## Latency numbers (the hierarchy)

```
 Operation                                  Time        Relative
 ─────────────────────────────────────────────────────────────────────
 L1 cache reference                         0.5 ns      1x
 Branch mispredict                          5   ns      10x
 L2 cache reference                         7   ns      14x
 Mutex lock/unlock                          25  ns      50x
 Main memory (RAM) reference                100 ns      200x   (0.1 µs)
 Compress 1 KB (Snappy)                     3   µs
 Send 1 KB over 1 Gbps network              10  µs
 Read 4 KB random from SSD                  150 µs      ~1,000x RAM
 Read 1 MB sequentially from RAM            250 µs
 Round trip within same datacenter          500 µs      (0.5 ms)
 Read 1 MB sequentially from SSD            1   ms
 Disk seek (spinning HDD)                   10  ms
 Read 1 MB sequentially from HDD            20  ms
 Round trip CA ↔ Netherlands ↔ CA           150 ms
 ─────────────────────────────────────────────────────────────────────
 INTERNALIZE: RAM ~100ns | SSD random ~150µs (1000x slower) |
              same-DC RTT ~0.5ms | cross-continent ~150ms.
 → Cache in memory. Keep data near users. Avoid cross-region on the hot path.
```

## Powers of 2 (storage / capacity)

```
 2^10  = 1,024            ≈ 1 thousand   → 1 KB
 2^20  = 1,048,576        ≈ 1 million    → 1 MB
 2^30  = 1,073,741,824    ≈ 1 billion    → 1 GB
 2^32  = 4,294,967,296    ≈ 4.3 billion  → (max unsigned 32-bit; IPv4 space)
 2^40  ≈ 1 trillion       → 1 TB
 2^50  ≈ 1 quadrillion    → 1 PB
 2^63  ≈ 9.2 × 10^18      → (max signed 64-bit; e.g. BIGINT / Snowflake range)
```

## Time anchors (for QPS math)

```
 1 day   ≈ 86,400 s   ≈ 10^5 s   ← THE trick: daily volume ÷ 100,000 = avg QPS
 1 month ≈ 2.5 million s
 1 year  ≈ 31.5 million s ≈ 3 × 10^7 s
 Peak QPS ≈ 2–3 × average QPS  (diurnal traffic).
```

## Availability (the "nines")

```
 99%     = two nines   → 3.65 days/year down
 99.9%   = three nines → 8.77 hours/year
 99.99%  = four nines  → 52.6 minutes/year
 99.999% = five nines  → 5.26 minutes/year
 Serial dependencies multiply: 99.9% × 99.9% × 99.9% ≈ 99.7% (each hop costs you).
```

## Typical capacity figures (orders of magnitude)

```
 Single commodity server     : ~10k–100k QPS (simple), ~1–10k (DB-bound)
 Single SQL DB instance      : ~5k–20k writes/sec, more reads with replicas
 Redis (single node)         : ~100k+ ops/sec, sub-ms latency, tens of GB RAM
 Kafka                       : millions of messages/sec (sequential disk)
 Memory per cache node       : tens–hundreds of GB
 SSD throughput              : ~500 MB/s–GBs/s; HDD ~100–200 MB/s
 Object storage durability   : 11 nines (99.999999999%)
 1 Gbps link                 : ~125 MB/s   ;  10 Gbps ≈ 1.25 GB/s
```

## Sizing rules of thumb

```
 • 80/20 rule: ~20% of data gets ~80% of traffic → cache the hot 20%.
 • Read:write ratio is the #1 number to establish (decides replica vs shard).
 • char/text ≈ 1–2 bytes; UUID = 16 bytes; a typical row ≈ 100s of bytes;
   an image ≈ 100s of KB–MB; a minute of video ≈ tens of MB.
 • Storage/year = items/day × size/item × 365 (then × replication factor).
 • Cache memory ≈ hot-set size; shard across nodes if it exceeds one node's RAM.
 • Backend load after cache = QPS × (1 − hitRatio). A 90% hit ratio cuts DB
   load 10x; 99% cuts it 100x.
```

## The quick decision reflexes

```
 Read-heavy?        → cache + read replicas + CDN + denormalize.
 Write-heavy?       → shard writes + LSM store + queue/batch + write-behind.
 Need scale + flexible schema + simple access? → NoSQL. Else default SQL.
 Strong consistency required during partition?  → CP. Else AP + eventual.
 Async work / decouple / absorb spikes?         → message queue.
 Static/media to global users?                  → object storage + CDN.
 Full-text search?                               → inverted index / Elasticsearch.
 Unique IDs at scale, sortable, no coordination? → Snowflake (or UUID v7).
 Cross-machine mutual exclusion?                 → distributed lock (or avoid via idempotency).
 Atomic across services?                         → Saga (avoid 2PC at scale).
 Limit request rate?                             → token bucket / sliding window counter.
 Distribute keys over changing nodes?            → consistent hashing + vnodes.
 Cross-service request debugging?                → distributed tracing + 4 golden signals.
```

---

## Final framework reminder

Every answer in this file plugs back into the **5-step framework**: clarify requirements (functional + non-functional), estimate with real numbers (QPS, storage, bandwidth — divide daily by 100k), draw the high-level boxes, go deep on 1–2 components using these building blocks, then close on bottlenecks and observability. **Structure + numbers + named trade-offs** is the entire game. Know these blocks cold, derive the numbers live, and always name what each choice costs.

---

Created 07-system-design-fundamentals.md — 23 building blocks/topics covered (the 5-step framework + load balancers, caching, CDN, message queues/streaming, databases, consistent hashing, CAP/PACELC + consistency models, database scaling patterns, rate limiting, API design, unique ID generation, object storage, search, monitoring/observability, idempotency keys, thundering herd, bloom filters, write-ahead log, quorum, heartbeats/leader election, distributed locks, and 2PC vs Saga), each with WHAT → WHEN → HOW internals → TRADE-OFFS → ASCII diagram, interview talking points, follow-up questions, plus a numbers-to-memorize cheat sheet.
