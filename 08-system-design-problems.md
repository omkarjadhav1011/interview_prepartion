# 08 — System Design Problems

> The onsite that decides your level. SDE-1 loops include 1 system design round; SDE-2 loops include 2, and your level (and pay band) is often set here more than in coding. Every problem below gets a **complete** end-to-end design — requirements, real back-of-envelope arithmetic, API, schema with sharding keys, an ASCII architecture diagram, deep dives, request lifecycles, a 10x/100x/1000x scaling story, explicit CAP trade-offs, and the follow-ups real interviewers fire. No hand-waving.

**Difficulty legend:** 🟢 Easy (SDE-1) · 🟡 Medium (SDE-1/2) · 🔴 Hard (SDE-2+)

This file builds on the vocabulary from **`07-system-design-fundamentals.md`** — load balancers (L4/L7), CDNs, caching (cache-aside/write-through/write-back), consistent hashing, database replication (leader-follower, multi-leader), sharding strategies, the CAP/PACELC theorems, quorums (W+R>N), message queues (Kafka/SQS), CDC, bloom filters, and back-of-envelope cheat-numbers. Where a building block appears below I name it and reference file 07 rather than re-deriving it; the focus here is *composing* those blocks into a working system.

---

## The 5-step framework (memorize this)

Every design round follows the same arc. Spend ~45 min like this:

1. **Requirements (5 min)** — Clarify functional ("what does it do?") and non-functional ("how fast / how available / how consistent / how big?"). *Drive* this — the interviewer is deliberately vague. Pin down scale with one number: DAU.
2. **Estimation (5 min)** — DAU → QPS (read & write) → storage/year → bandwidth → cache memory. The numbers decide your architecture (do you need sharding? a CDN? a cache?). Don't skip this even if it feels mechanical — it's where you justify every later choice.
3. **API + Data model (10 min)** — A handful of endpoints with request/response shapes. Then tables/collections with **indexes** and the **partition/sharding key** (the single most-probed schema decision).
4. **High-level architecture (10 min)** — Draw the boxes: clients → LB → app servers → cache → DB → queue → workers. Show data flow. This is the picture you keep extending in the deep dive.
5. **Deep dive + scaling + trade-offs (15 min)** — The interviewer picks 1-2 components to go deep on. Have an opinion, state the trade-off, name the CAP decision. Then 10x/100x growth: where does it break and what do you change?

**The cheat-numbers (from file 07) you'll reuse constantly:**

| Quantity | Number |
|---|---|
| 1 day | 86,400 s ≈ 10⁵ s |
| L1 cache ref | 0.5 ns |
| Main memory ref | 100 ns |
| SSD random read | 16 µs (~150 µs older) |
| Disk seek | 10 ms |
| Network round trip same DC | 0.5 ms |
| Round trip CA↔Netherlands | 150 ms |
| Read 1 MB sequential from memory | 250 µs |
| Read 1 MB sequential from SSD | 1 ms |
| Read 1 MB sequential from disk | 20 ms |
| 1 char ≈ 1 byte; 1 KB tweet, 200 KB photo, 2 MB image, 5 MB song |
| Throughput of one well-tuned SQL box | ~1–5 K writes/s, ~10–50 K reads/s |
| One Redis node | ~100 K ops/s, ~25 GB usable RAM |
| One Kafka partition | ~10 MB/s, tens of K msgs/s |

**Availability table (the "nines"):**

| Nines | Downtime/year |
|---|---|
| 99% (two nines) | 3.65 days |
| 99.9% | 8.76 hours |
| 99.99% | 52.6 minutes |
| 99.999% (five nines) | 5.26 minutes |

---

## Table of Contents

**🟢 EASY (SDE-1)**
1. [URL Shortener (TinyURL)](#1--url-shortener-tinyurl)
2. [Pastebin](#2--pastebin)
3. [Rate Limiter](#3--rate-limiter)
4. [Distributed Key-Value Store](#4--distributed-key-value-store)
5. [Unique ID Generator](#5--unique-id-generator)
6. [Leaderboard](#6--leaderboard)
7. [Notification System](#7--notification-system)
8. [File Storage Service (Dropbox-lite)](#8--file-storage-service-dropbox-lite)

**🟡 MEDIUM (SDE-1/2)**
9. [Twitter / News Feed](#9--twitter--news-feed)
10. [Instagram / Photo Sharing](#10--instagram--photo-sharing)
11. [WhatsApp / Chat System](#11--whatsapp--chat-system)
12. [YouTube / Video Streaming](#12--youtube--video-streaming)
13. [Uber / Ride Sharing](#13--uber--ride-sharing)
14. [Yelp / Proximity Service](#14--yelp--proximity-service)
15. [Typeahead / Autocomplete](#15--typeahead--autocomplete)
16. [Web Crawler](#16--web-crawler)
17. [Ticketmaster / Booking System](#17--ticketmaster--booking-system)
18. [Google Docs / Collaborative Editing](#18--google-docs--collaborative-editing)
19. [Distributed Job Scheduler](#19--distributed-job-scheduler)

**🔴 HARD (SDE-2+)**
20. [Google Search](#20--google-search)
21. [Distributed Cache](#21--distributed-cache)
22. [Payment System (Stripe/Razorpay)](#22--payment-system-striperazorpay)
23. [Ad Click Aggregator](#23--ad-click-aggregator)
24. [Google Maps / Proximity + Routing](#24--google-maps--proximity--routing)

---

# 🟢 EASY (SDE-1)

---

## 1 — URL Shortener (TinyURL)

**Asked at:** Amazon, Google, Microsoft, Bloomberg, Atlassian, Flipkart  **Difficulty:** 🟢 Easy  **Level:** SDE-1

The canonical warm-up. It looks trivial ("just hash the URL") but every sub-decision — how to generate the short code, how to avoid collisions, read-vs-write skew, custom aliases — is a probe. Get this one *crisp*.

### 1. Requirements

**Functional**
- Given a long URL, return a short URL (e.g. `https://tiny.co/aZ3kP9`).
- Redirect: hitting the short URL 301/302-redirects to the original.
- Optional **custom alias** (`tiny.co/my-brand`).
- Optional **expiration** (link dies after a TTL).
- Analytics (click counts) — nice-to-have, often deferred.

**Non-functional**
- **Latency:** redirect must be <100 ms (it's on the critical path of a user clicking a link).
- **Availability:** 99.99% — a dead shortener breaks every link ever made. Availability > consistency here.
- **Consistency:** eventual is fine for analytics; the mapping itself should be read-your-writes (create then immediately share).
- **Durability:** never lose a mapping. A lost mapping = a dead link forever.
- **Scale target:** 100M new URLs/month; read:write ≈ 100:1 (links are created once, clicked many times).

### 2. Back-of-envelope estimation

```
Writes:  100M new URLs / month
         100M / (30 × 86,400) ≈ 100M / 2.6M s ≈ 38 writes/s   (avg)
         Peak ≈ 5× avg ≈ ~200 writes/s

Reads:   read:write = 100:1  →  3,800 reads/s avg, ~20,000 reads/s peak

Storage (per row):
   short_code      7 bytes
   long_url        ~500 bytes (URLs can be long; budget 500)
   metadata        ~100 bytes (created_at, expiry, owner, click_count)
   ─────────────────────────
   ≈ ~600 bytes/row, round to 1 KB with overhead/index

   Per year:  100M/mo × 12 = 1.2B URLs/year
   1.2B × 1 KB = 1.2 TB/year
   Over 10 years ≈ 12 TB  → trivially fits a sharded DB, or even one big box.

Key space (how long must the code be?):
   Use base62 [a-zA-Z0-9] = 62 symbols.
   62^6 = 56.8 billion   → enough for ~47 years at 1.2B/yr
   62^7 = 3.5 trillion   → safe headroom. Use 7 chars.

Bandwidth:
   Read response is a redirect (~tiny header, ~500 B). 20K rps × 500 B ≈ 10 MB/s. Negligible.

Cache memory (cache the hot 20% of reads — Pareto):
   Daily reads ≈ 3,800 × 86,400 ≈ 328M reads/day across maybe ~10M distinct hot URLs.
   Cache 10M hot mappings × ~1 KB = 10 GB → fits comfortably in one Redis node (25 GB usable).
```

**Takeaway from the numbers:** writes are tiny (38/s), reads are 100× heavier and latency-critical → this is a **read-heavy, cache-first** system. Storage is small. So the *real* design problems are (a) short-code generation and (b) read latency, not storage scale.

### 3. API design

```
POST /api/v1/urls
   Request:  { "longUrl": "https://example.com/very/long/path?x=1",
               "customAlias": "my-brand"  (optional),
               "expiresAt": "2027-01-01T00:00:00Z"  (optional) }
   Response: 201 { "shortUrl": "https://tiny.co/aZ3kP9",
                   "shortCode": "aZ3kP9",
                   "expiresAt": "..." }
   Errors:   409 if customAlias is taken; 400 if longUrl invalid.

GET /{shortCode}
   Response: 302 Found, Location: <longUrl>
             (or 301 if you want browsers/CDNs to cache the redirect permanently)
   Errors:   404 if not found or expired.

DELETE /api/v1/urls/{shortCode}   (auth required, owner only)
GET    /api/v1/urls/{shortCode}/stats   (click analytics)
```

**301 vs 302 — a real follow-up:** A **301 (permanent)** is cached by the browser, so subsequent clicks never hit your server — great for load, terrible for analytics (you stop seeing clicks). A **302 (found/temporary)** forces every click through your server — you keep analytics but carry the full read load. *Decision:* use **302** if analytics matter (most products do), **301** if you only care about offloading. State this trade-off out loud; interviewers love it.

### 4. Data model / schema

A URL shortener is a **key-value lookup** by short_code. That points at NoSQL (or a KV store), but a single indexed SQL table is also fine at this scale. I'll use a relational table and note the NoSQL alternative.

```
TABLE url_mappings
─────────────────────────────────────────────────────────
  short_code     VARCHAR(7)   PRIMARY KEY      ← partition/shard key
  long_url       VARCHAR(2048) NOT NULL
  creator_id     BIGINT       (nullable)
  created_at     TIMESTAMP
  expires_at     TIMESTAMP    (nullable, indexed for TTL sweeps)
  click_count    BIGINT       DEFAULT 0        ← hot counter, see note

INDEX idx_expires (expires_at)        -- for the expiry janitor
INDEX idx_creator (creator_id)        -- "my links" page
```

**Sharding key = `short_code`.** It's the only thing the hot path (`GET /{shortCode}`) queries by, it's high-cardinality, and lookups are point reads. Shard by `hash(short_code) % N` (consistent hashing — see file 07) so reads fan to exactly one shard.

**`click_count` is a trap:** a hot link clicked 20K/s would hammer one row with `UPDATE ... SET click_count = click_count + 1` → row-lock contention. *Don't* update synchronously. Emit a click event to a queue/Kafka and aggregate counts asynchronously (see Ad Click Aggregator, #23). Keep the redirect path read-only.

### 5. High-level architecture

```
                                          ┌──────────────────────┐
   Client ──── GET /aZ3kP9 ─────────────▶ │   CDN / Edge cache   │  (301-cached redirects
     │                                     │  (optional)          │   can short-circuit here)
     │                                     └──────────┬───────────┘
     │                                                │ miss
     ▼                                                ▼
 ┌────────────────┐        ┌───────────────────────────────────────┐
 │  L7 Load        │───────▶│         Stateless App Servers          │
 │  Balancer       │        │  (Redirect Service + Create Service)   │
 └────────────────┘        └───────┬───────────────────────┬────────┘
                                    │ 1. read cache          │ writes
                          (cache-aside)                      │
                                    ▼                         ▼
                          ┌──────────────────┐    ┌────────────────────────┐
                          │  Redis cluster   │    │  ID/Code Generator      │
                          │  code → longUrl  │    │  (Counter via Zookeeper │
                          │  (hot 20%)       │    │   range / Snowflake)    │
                          └────────┬─────────┘    └───────────┬────────────┘
                                   │ miss → DB                │
                                   ▼                          ▼
                          ┌─────────────────────────────────────────────┐
                          │  Sharded KV / SQL  (shard by hash(code))     │
                          │  Leader + Followers per shard (replication)  │
                          └─────────────────────────────────────────────┘
                                   │
        click events ─────────────┘ (async) ──▶ Kafka ──▶ Aggregator ──▶ Analytics store
```

### 6. Deep dive

**(A) Short-code generation — the heart of the problem.** Three approaches:

1. **Hash the long URL (MD5/SHA → base62, take first 7 chars).**
   - *Pro:* same URL → same code (free dedup).
   - *Con:* truncating a hash → **collisions**. Handle by: on collision, append a salt and rehash, or probe (`code`, `code+1`...). Collision-check requires a DB read on every write (a `SELECT` before `INSERT`), adding latency. Also "same URL same code" leaks that two people shortened the same link and breaks per-user analytics/expiry.

2. **Counter + base62 encode (recommended).** Keep a global monotonic counter; each new URL gets the next integer, base62-encoded → the code. ID `125` → `"cb"`, ID `3,521,614,606,208` → `"zzzzzz"`.
   - *Pro:* zero collisions by construction, no pre-read.
   - *Con:* sequential codes are **guessable/enumerable** (scrape all links). Fix: don't use a raw counter — use a **ranged counter** (each app server grabs a block of 1,000 IDs from Zookeeper/a counter service, hands them out locally → no per-write coordination) and optionally **scramble** the integer with a reversible permutation (e.g. multiply by a large prime mod 62^7, or Feistel/XOR) before base62 so codes look random but stay collision-free.

3. **Pre-generated keys (Key Generation Service, KGS).** A standalone service pre-computes random unique 7-char codes into a `unused_keys` table. Writers pop a key (mark it used in one transaction). 
   - *Pro:* O(1) write, codes look random, no collisions.
   - *Con:* the KGS is a single point of failure → replicate it; concurrency on "pop a key" needs care (each app server caches a batch of keys in memory).

**Recommendation:** ranged counter + base62 + light scrambling. It's collision-free, coordination-light, and non-enumerable.

**(B) Read path / caching.** The redirect is the hot path. Cache-aside (file 07): app checks Redis (`code → longUrl`); on hit, redirect immediately (<5 ms). On miss, read the shard, populate Redis with a TTL, redirect. Pareto says ~20% of links get ~80% of clicks, so a 10 GB cache absorbs the vast majority of the 20K rps, leaving the DB lightly loaded. Optionally front the whole thing with a **CDN** that caches 301 redirects at the edge.

**(C) Custom alias.** Check the `url_mappings` table for the alias as a primary key; `INSERT ... IF NOT EXISTS` (or a unique-constraint catch) returns 409 on conflict. Custom aliases and generated codes share the same key space — generated codes are length-7, so reserve custom aliases to a different length or a prefix to avoid future collisions with the generator's space.

### 7. Data flow

**Create (write):**
1. `POST /api/v1/urls` hits the LB → a stateless app server.
2. Validate the long URL. If `customAlias` present, attempt insert with alias as PK → 409 on conflict.
3. Else: grab the next ID from the server's local block (refill the block from the counter service if empty), base62-encode, scramble.
4. `INSERT (short_code, long_url, ...)` into the shard for `hash(short_code)`.
5. Write-through to Redis (so a freshly created link is instantly readable — read-your-writes).
6. Return `201` with the short URL.

**Redirect (read):**
1. `GET /aZ3kP9` → LB → app server.
2. `GET aZ3kP9` from Redis. Hit → go to step 5.
3. Miss → `SELECT long_url FROM url_mappings WHERE short_code='aZ3kP9'` on the right shard.
4. If found, populate Redis (TTL ~24h). If not found / expired → `404`.
5. Emit a click event to Kafka (async, fire-and-forget — never block the redirect).
6. Return `302 Location: <long_url>`.

### 8. Scaling

- **10x (2K writes/s, 200K reads/s):** Reads are absorbed by scaling out Redis (add nodes, consistent hashing) and adding read-replicas / a CDN. Writes at 2K/s still fit a handful of shards.
- **100x (20K writes/s, 2M reads/s):** Shard the DB by `hash(short_code)` across N nodes; each handles ~its slice of writes. Reads are >99% cache; size Redis cluster for the hot set (~100 GB → 4-5 nodes). CDN edge-caches redirects globally to cut latency for far users (the 150 ms cross-ocean RTT).
- **1000x:** Multi-region active-active. Each region has its own Redis + DB replica; mappings are immutable after creation so cross-region replication is trivially eventually-consistent (no conflicts — a code maps to exactly one URL forever). Generate IDs per-region with a region prefix in the ID space to avoid cross-region coordination.
- **Bottlenecks:** (1) the global counter — fix with ranged allocation; (2) the hot `click_count` row — fix by moving counts off the write path into a stream aggregator; (3) cross-region latency — fix with CDN + regional replicas.

### 9. Trade-offs

- **CAP:** Choose **AP**. A redirect must work even during a partition; serving a possibly-slightly-stale mapping is fine because mappings are immutable. Availability > consistency.
- **SQL vs NoSQL:** At 12 TB / point-lookup-by-key, a wide-column/KV store (DynamoDB, Cassandra) is the natural fit and shards effortlessly. But a single well-indexed Postgres handles this scale too and gives you transactions for custom-alias uniqueness for free. *Justification:* pick KV/NoSQL when you expect 100x+ growth and want painless sharding; pick SQL for simplicity and strong alias-uniqueness at SDE-1 scale.
- **301 vs 302:** analytics (302) vs load-offload (301), covered above.
- **Hash vs counter codes:** dedup-for-free (hash) vs collision-free + simple (counter). Counter wins for production.

### 10. Follow-up questions

**Q: How do you prevent two app servers from generating the same code?**
A: They don't share a counter at write time. Each server reserves a *range* (block of 1,000 IDs) from a central counter (Zookeeper/Redis `INCRBY 1000`) and hands them out locally. Ranges never overlap, so codes are globally unique with zero per-write coordination.

**Q: How do you handle link expiration / cleanup?**
A: Store `expires_at`; the read path checks it and returns 404 if past. A background **janitor** job sweeps `WHERE expires_at < now()` in batches and deletes (or DynamoDB/Cassandra TTL auto-expires rows). Also evict from cache on expiry.

**Q: What if the same long URL is shortened twice — same code or different?**
A: Different (with counter approach). If the product wants dedup, keep a secondary index `hash(long_url) → short_code` and look up first. Trade-off: per-user expiry/analytics break if codes are shared, so most products *don't* dedup.

**Q: How do you make codes non-guessable?**
A: Scramble the counter integer with a reversible permutation (multiply by a large coprime mod 62^7, or a small Feistel network) before base62. Codes look random; still collision-free and reversible if needed.

**Q: How do you do analytics without slowing redirects?**
A: Fire a click event to Kafka on the redirect path (non-blocking). A stream processor aggregates per-code counts into an analytics store (see #23). The redirect never writes to the DB.

---

## 2 — Pastebin

**Asked at:** Amazon, Microsoft, LinkedIn, Adobe  **Difficulty:** 🟢 Easy  **Level:** SDE-1

Pastebin = TinyURL + a big text blob. The new wrinkle is **storing large content** (text up to a few MB) cheaply, which introduces **blob storage (S3) vs database** as the central trade-off.

### 1. Requirements

**Functional**
- User pastes text (up to, say, 10 MB), gets a short URL.
- Anyone with the URL reads the paste.
- Optional expiration (burn-after-reading, 1 day, 1 week, never).
- Optional privacy (unlisted/public), optional syntax highlighting (client-side).
- Custom alias (like TinyURL).

**Non-functional**
- **Latency:** read <200 ms; write <500 ms (uploading content is heavier than a URL row).
- **Availability:** 99.9%.
- **Consistency:** read-your-writes for the creator; eventual elsewhere.
- **Durability:** high — losing a paste loses user data. 11 nines if backed by S3.
- **Scale:** ~10M new pastes/month, read:write ≈ 10:1 (lighter skew than TinyURL).

### 2. Back-of-envelope estimation

```
Writes:  10M/month ÷ 2.6M s ≈ 4 writes/s avg, ~20/s peak
Reads:   10:1 → 40 reads/s avg, ~200/s peak

Content size: assume average paste = 10 KB (most are code snippets), cap 10 MB.
Storage/year: 10M/mo × 12 × 10 KB = 120M × 10 KB = 1.2 TB/year of content
              + metadata rows (~500 B each) × 120M = 60 GB/year

Bandwidth: reads 200/s × 10 KB = 2 MB/s (avg paste). Spiky for large pastes.

Cache: hot pastes (recently created, trending). Cache top ~1M pastes × 10 KB = 10 GB → one Redis node.
```

**Takeaway:** content (1.2 TB/yr) dwarfs metadata. Don't store 10 MB blobs *in the database* (kills row size, backup time, replication). Put **content in object storage (S3)**, **metadata + the S3 pointer in the DB**.

### 3. API design

```
POST /api/v1/pastes
   Request:  { "content": "....", "expiresIn": "1d"|"1w"|"never",
               "visibility": "public"|"unlisted", "customAlias": "..." }
   Response: 201 { "pasteId": "x7Kp2a", "url": "https://pb.co/x7Kp2a" }
   (For very large content, return a presigned S3 PUT URL and have the client upload directly.)

GET /{pasteId}            → 200 { content, language, createdAt, expiresAt } or 404
DELETE /api/v1/pastes/{pasteId}   (owner only)
```

### 4. Data model / schema

```
TABLE pastes                          (metadata only)
─────────────────────────────────────────────────
  paste_id      VARCHAR(8)  PRIMARY KEY   ← shard key (hash)
  s3_key        VARCHAR(256)             ← pointer to blob in S3
  creator_id    BIGINT (nullable)
  size_bytes    INT
  language      VARCHAR(32)
  visibility    ENUM('public','unlisted')
  created_at    TIMESTAMP
  expires_at    TIMESTAMP (nullable, indexed)

INDEX idx_expires (expires_at)

OBJECT STORE (S3):  bucket/pastes/{paste_id}  →  raw text content
```

**Shard key = `paste_id`** (same reasoning as TinyURL). Content lives in S3, keyed by `paste_id` — S3 handles its own durability (11 nines) and scaling.

### 5. High-level architecture

```
 Client ──POST──▶ ┌──────────┐    ┌─────────────────────┐
                  │   LB     │───▶│   App Servers        │
 Client ──GET───▶ └──────────┘    │ (Paste + Read svc)   │
                                  └───┬──────────┬───────┘
                       paste_id gen   │          │ small content read
                  (counter→base62)    │          ▼
                                      │   ┌───────────────┐  miss   ┌──────────────┐
                                      │   │ Redis (hot     │────────▶│ Metadata DB   │
                                      │   │ pastes + meta) │         │ (sharded)     │
                                      │   └───────────────┘         └──────────────┘
                                      │                                     │ s3_key
                                      ▼                                     ▼
                              ┌─────────────────────────────────────────────────┐
                              │     Object Storage (S3)  —  raw paste content    │
                              │     fronted by CDN for popular/large pastes      │
                              └─────────────────────────────────────────────────┘
                              Expiry janitor: sweeps expires_at, deletes S3 + row
```

### 6. Deep dive

**(A) DB vs S3 for content.** Storing 10 MB blobs in Postgres/MySQL bloats rows, slows full-table backups, and inflates replication traffic. **S3 (object storage)** is purpose-built: cheap ($/GB), 11-nines durability, infinite scale, and CDN-friendly. So: **metadata + pointer in DB, bytes in S3.** Small pastes (<~64 KB) *can* be inlined in the DB or cache to save an S3 round trip; large pastes always go to S3.

**(B) Direct-to-S3 upload for large pastes.** For multi-MB content, don't proxy bytes through your app servers (wastes their bandwidth/CPU). Issue a **presigned S3 PUT URL**; the client uploads directly to S3, then calls your API to commit the metadata row. Symmetric on read: serve a presigned GET or a CDN URL.

**(C) Read path.** Look up metadata (Redis → DB). For small inlined content, you're done from cache. For S3-backed content, return a CDN URL or stream from S3. Popular pastes get cached at the CDN edge, so a viral paste doesn't melt your origin.

### 7. Data flow

**Create:** validate → generate `paste_id` (counter+base62) → if large, presign S3 PUT and let client upload; else upload server-side → write metadata row (`s3_key`, size, expiry) → cache metadata → return URL.

**Read:** `GET /{id}` → Redis metadata hit? → fetch content (inlined cache, or S3/CDN) → check `expires_at` (404 if expired) → return.

### 8. Scaling

- Content scale is handled by S3 (effectively infinite) + CDN (read offload). Metadata is small; shard by `paste_id` past 100x.
- **Hot paste (a viral snippet):** CDN absorbs it. Without CDN, cache the content in Redis with a short TTL.
- **Write spikes:** app servers are stateless → autoscale behind the LB; S3 absorbs blob writes directly.

### 9. Trade-offs

- **CAP:** AP — reads should work during partitions; pastes are immutable so staleness is harmless.
- **SQL vs NoSQL:** metadata is tiny and relational-friendly → SQL is fine; NoSQL (Dynamo) if you want zero-ops sharding. **Content must be object storage, not a DB** — this is the load-bearing decision.
- **Inline-vs-S3 threshold:** inlining small pastes saves a round trip but bloats the DB/cache; pick a threshold (~64 KB) and measure.

### 10. Follow-up questions

**Q: Why not store content in the database?**
A: 10 MB blobs destroy row size, backup/restore time, and replication bandwidth; databases charge premium $/GB. S3 is cheaper, more durable (11 nines), infinitely scalable, and CDN-integrable. Keep the DB for small, queryable metadata.

**Q: How do you handle burn-after-reading?**
A: On first successful read, mark the paste consumed (atomic flag flip) and enqueue deletion of the S3 object + row. Subsequent reads → 404. Use a conditional update so concurrent reads don't double-serve.

**Q: How do you stop someone uploading a 10 GB "paste"?**
A: Enforce a size cap in the presigned URL (S3 supports content-length-range conditions) and validate `size_bytes` server-side before committing metadata.

**Q: Public/unlisted listing?**
A: "Unlisted" = reachable only by URL (not indexed); "public" = also appears in a discover feed. Keep a separate `public_pastes` index/feed table; never list unlisted ones.

---

## 3 — Rate Limiter

**Asked at:** Stripe, Amazon, Google, Cloudflare, Atlassian, Razorpay  **Difficulty:** 🟢 Easy (the *distributed* version is 🟡)  **Level:** SDE-1

Deceptively deep. The interviewer wants the **algorithms** (token bucket vs sliding window) *and* the **distributed coordination** problem (how do 50 app servers share one limit without a race?). Be ready to write the token-bucket math.

### 1. Requirements

**Functional**
- Limit requests per client (by user ID / API key / IP) to N per time window.
- Support multiple rules (e.g. 10 req/s burst, 1000 req/hour sustained; different tiers).
- Return `429 Too Many Requests` + a `Retry-After` header when exceeded.
- Rules configurable without redeploy.

**Non-functional**
- **Latency:** the limiter is on *every* request → must add <5 ms. This dominates the design.
- **Availability:** must not become a single point of failure; **fail-open vs fail-closed** is a real decision.
- **Accuracy:** approximate is usually acceptable (a few extra requests through is OK); some cases (payments) need exactness.
- **Scale:** if the API does 1M req/s, the limiter sees 1M req/s.

### 2. Back-of-envelope estimation

```
Suppose API peak = 1,000,000 req/s, 10M distinct API keys.

State per key: token-bucket = {tokens (float, 8B), last_refill_ts (8B)} ≈ 16 B + key overhead ~ 50 B.
10M keys × 50 B = 500 MB → easily fits one Redis node's RAM.

Throughput: the limiter must do ~1M ops/s. One Redis node ≈ 100K ops/s →
   need ~10+ Redis nodes (shard by key), OR push the check to the edge / in-process.

Latency budget: each check = 1 Redis round trip (~0.5 ms same-DC) + script eval. Fine under 5 ms.
```

**Takeaway:** state is tiny (500 MB), but **op throughput (1M/s)** forces either a Redis *cluster* sharded by key, or a *local* token bucket per app server with periodic sync. Latency budget forbids anything slow.

### 3. API design

The rate limiter is usually **middleware**, not a public API. But it exposes:

```
Internal:  allow(clientId, ruleId) → { allowed: bool, remaining: int, retryAfter: seconds }

Config:    PUT /admin/rules/{ruleId}  { "limit": 1000, "windowSec": 3600, "burst": 10 }

On the wire, when blocked:
   HTTP/1.1 429 Too Many Requests
   Retry-After: 12
   X-RateLimit-Limit: 1000
   X-RateLimit-Remaining: 0
   X-RateLimit-Reset: 1718000000
```

### 4. Data model

```
Redis key per client:rule:
   rl:{ruleId}:{clientId}  →  HASH { tokens: 9.5, last_refill: 1718000000.123 }
   (token bucket)
   — or —
   rl:{ruleId}:{clientId}  →  ZSET of request timestamps   (sliding-window log)
   — or —
   rl:{ruleId}:{clientId}:{windowStart} → counter INT with TTL  (fixed window)

Rules config (small, cached in every app server, refreshed every few sec):
   rules: { "free_tier": {limit:1000, window:3600, burst:10}, ... }
```

### 5. The algorithms (the deep dive that *is* the problem)

**1) Fixed Window Counter.** Bucket time into windows (e.g. each minute). `INCR rl:{client}:{minute}`; if > limit, reject. TTL the key to the window length.
- *Pro:* trivial, O(1), tiny memory.
- *Con:* **boundary burst** — a client can send `limit` requests at 0:59 and another `limit` at 1:00 = 2× limit in 1 second straddling the boundary.

```
limit = 5/min
 0:00 .............. 0:59 | 1:00 .............. 1:59
        [5 reqs at 0:59]   [5 reqs at 1:00]
        └──────── 10 requests in ~1 second ────────┘   ← the bug
```

**2) Sliding Window Log.** Store a sorted set of request timestamps; on each request, drop timestamps older than `now - window`, count what remains, allow if `< limit`, then add `now`.
- *Pro:* exact, no boundary burst.
- *Con:* memory = O(requests) per client (store every timestamp) — expensive for high-volume clients.

**3) Sliding Window Counter (the sweet spot).** Approximate the sliding window using the current and previous fixed-window counts, weighted by overlap:

```
count ≈ curr_window_count
      + prev_window_count × (overlap fraction of prev window still in the sliding window)

e.g. limit 100/min, now is 30s into the current minute:
   estimate = curr(  in this minute so far) + prev × (1 − 30/60)
            = 18 + 80 × 0.5 = 58   →  allow (58 < 100)
```
- *Pro:* near-exact, O(1) memory (two counters), smooths the boundary burst. **Used by Cloudflare.**

**4) Token Bucket (most common answer).** A bucket holds up to `B` tokens, refilled at rate `r` tokens/sec. Each request consumes 1 token; if the bucket is empty, reject. Allows **bursts** up to `B` while enforcing average rate `r`.

```
on request at time t:
   elapsed       = t − last_refill
   tokens        = min(B, tokens + elapsed × r)   # refill (lazily, no background timer)
   last_refill   = t
   if tokens >= 1:
        tokens -= 1;  ALLOW
   else:
        retryAfter = (1 − tokens) / r;  REJECT (429)
```
- *Pro:* allows controlled bursts (good UX), O(1) memory (2 numbers), no timer needed (lazy refill).
- *Con:* two clients sharing a key need atomic refill+decrement → use a **Redis Lua script** to make read-modify-write atomic.

**5) Leaky Bucket.** Requests enter a fixed-size FIFO queue, processed at a constant rate; queue full → reject. Smooths output to a steady rate (good for protecting a downstream that needs even load), but adds latency and no bursting.

**Recommendation:** **Token bucket** for APIs (burst-friendly, cheap). **Sliding-window counter** when you must cap precisely without bursts.

### 6. Deep dive — distributed coordination

The hard part: 50 stateless app servers must enforce *one* shared limit per client.

**Option A — Centralized Redis (recommended default).** All servers run the same atomic Lua script against a shared Redis (sharded by client key). The Lua script does refill+decrement+check in one round trip → no race.

```lua
-- KEYS[1]=bucket key, ARGV: now, rate, burst
local b = redis.call('HMGET', KEYS[1], 'tokens', 'ts')
local tokens = tonumber(b[1]) or tonumber(ARGV[3])   -- start full
local ts     = tonumber(b[2]) or tonumber(ARGV[1])
local elapsed = tonumber(ARGV[1]) - ts
tokens = math.min(tonumber(ARGV[3]), tokens + elapsed * tonumber(ARGV[2]))
local allowed = 0
if tokens >= 1 then tokens = tokens - 1; allowed = 1 end
redis.call('HMSET', KEYS[1], 'tokens', tokens, 'ts', ARGV[1])
redis.call('EXPIRE', KEYS[1], 3600)
return allowed
```
- *Cost:* one ~0.5 ms RTT per request. *Risk:* Redis is a dependency — if it's down, do you fail-open (allow all) or fail-closed (block all)? Usually **fail-open** for availability (a brief over-limit beats an outage), unless the limit protects something critical (payments → fail-closed).

**Option B — Local + sync (lower latency, approximate).** Each server keeps a local token bucket sized to `globalLimit / numServers`, and periodically reconciles with a central store. Zero per-request RTT, but the limit is only globally approximate and breaks if traffic isn't evenly balanced across servers. Good for ultra-high-throughput edge limiting.

**Option C — Sticky routing.** Route a given client always to the same server (consistent hashing at the LB), so that server owns the client's bucket locally with no coordination. Breaks on server failure / rebalancing.

### 7. Data flow

1. Request arrives → middleware extracts `clientId` (API key/user/IP) and matches a `ruleId`.
2. Run the atomic token-bucket Lua against Redis shard `hash(clientId)`.
3. Allowed → forward request, add `X-RateLimit-*` headers.
4. Rejected → return `429` + `Retry-After`, don't forward.
5. If Redis is unreachable → fail-open (allow) or fail-closed per rule criticality.

### 8. Scaling

- **Throughput:** shard Redis by `clientId`; add nodes (consistent hashing). 1M req/s ÷ 100K/node → ~10–15 nodes.
- **Latency at the edge:** move limiting to the CDN/edge (Cloudflare-style) so it happens before traffic reaches your origin; use local buckets with eventual global sync.
- **Hot key (one client hammering):** that client's bucket lives on one shard — it gets the load but rejects fast (O(1)); not a system-wide problem.

### 9. Trade-offs

- **Accuracy vs latency/availability:** centralized Redis = accurate but a dependency + RTT; local = fast but approximate. Most APIs accept approximate.
- **Fail-open vs fail-closed:** availability vs protection. Default open; closed for security/payment limits.
- **CAP:** the limiter favors **AP** — better to occasionally over-admit than to block all traffic when the coordination store is partitioned.
- **Burst vs smooth:** token bucket allows bursts (good UX); leaky bucket smooths (protects fragile downstreams).

### 10. Follow-up questions

**Q: Why is fixed-window counter dangerous?**
A: Boundary burst — a client can fire `limit` requests just before the window flips and `limit` again just after, doubling the effective rate across the boundary. Sliding-window counter or token bucket fixes it.

**Q: How do you make the token bucket atomic across servers?**
A: A Redis Lua script: read tokens+timestamp, refill, decrement, write — all server-side in one atomic eval, so concurrent requests can't both see "1 token left."

**Q: What if Redis goes down?**
A: Decide fail-open (allow, preserve availability) or fail-closed (block, preserve protection) per rule. Add a local in-process fallback bucket so you degrade gracefully rather than 500.

**Q: Rate-limit by what key?**
A: Layered — by IP (anti-DDoS), by API key/user (fairness/billing tiers), by endpoint (protect expensive routes), even globally (protect the whole service). Compose multiple buckets; reject if *any* trips.

**Q: How do you push limits to the edge?**
A: Run local token buckets in edge PoPs (CDN workers) sized to a share of the global limit, syncing counts to a central store every few seconds. Cuts latency to ~0 and stops abuse before it reaches origin, at the cost of exactness.

---

## 4 — Distributed Key-Value Store

**Asked at:** Amazon (DynamoDB team), Google, Confluent, Databricks  **Difficulty:** 🟢 Easy to frame, 🔴 to do well  **Level:** SDE-1 (intro) / SDE-2

This is the "build DynamoDB/Cassandra" question. It's the *foundational* distributed-systems design — consistent hashing, replication, quorums, and conflict resolution — and everything else (cache, queue, DB) reuses these ideas. Reference file 07's CAP/quorum sections heavily.

### 1. Requirements

**Functional**
- `put(key, value)` and `get(key)` — that's the core API.
- Values are opaque blobs (up to ~1 MB).
- Highly available writes (always accept a write, even during partitions).
- Tunable consistency (let the caller pick strong vs eventual).
- Automatic partitioning + replication across nodes.

**Non-functional**
- **Latency:** single-digit ms get/put (p99 < 10 ms).
- **Availability:** 99.99%+ — "always writeable" is the headline feature.
- **Consistency:** tunable (eventual by default — this is an AP store like Dynamo).
- **Durability:** N replicas; survive node + rack + AZ failure.
- **Scale:** petabytes, millions of ops/s, linearly scalable by adding nodes.

### 2. Back-of-envelope estimation

```
Target: 1M ops/s, 100 TB data, avg value 1 KB.

Nodes: one commodity node ≈ 10K writes/s heavy, ~50K reads/s.
   1M ops/s ÷ ~50K/node ≈ 20–100 nodes depending on read/write mix.
Storage: 100 TB ÷ (per-node ~2 TB SSD) × replication factor 3 = 300 TB raw
   → 300 TB ÷ 2 TB ≈ 150 nodes.
Replication: RF=3 standard. Quorum N=3, common W=2, R=2 (W+R>N → strong-ish reads).
Memory: memtable + bloom filters + key cache per node, sized to RAM (~10s of GB/node).
```

### 3. API design

```
PUT  /kv/{key}    body=value   [?consistency=ONE|QUORUM|ALL]   → {version: vectorClock}
GET  /kv/{key}    [?consistency=ONE|QUORUM|ALL]                → {value, version}
DELETE /kv/{key}                                               → tombstone write
```

### 4. Data model / partitioning

There are no tables — it's a ring of nodes, each owning a range of the hash space.

```
Partitioning: consistent hashing (file 07).
   hash(key) → position on a 0..2^160 ring.
   Walk clockwise to the first node = the key's coordinator.
   Use VIRTUAL NODES (each physical node owns many ring positions) to spread load
   evenly and rebalance smoothly when nodes join/leave.

Replication: store the key on the coordinator + the next (RF−1) distinct physical
   nodes clockwise = the "preference list." RF=3.

Per-node storage engine: LSM-tree (memtable → SSTables on disk) — write-optimized,
   exactly what Cassandra/RocksDB use. Bloom filter per SSTable to skip disk reads.
```

### 5. High-level architecture

```
            ┌──────── Consistent-hash ring (with virtual nodes) ────────┐
            │                                                            │
   put(k,v) │     N1 ◄── coordinator for k                              │
  ──────────┼──▶  │  replicates to N2, N3 (preference list)             │
 (any node  │     ▼                                                     │
  can be    │   ┌────┐   ┌────┐   ┌────┐   ┌────┐   ┌────┐   ...        │
  coord via │   │ N1 │──▶│ N2 │──▶│ N3 │──▶│ N4 │──▶│ N5 │              │
  request   │   └────┘   └────┘   └────┘   └────┘   └────┘              │
  routing)  │     ▲ gossip protocol (membership, failure detection)     │
            └──────────────────────────────────────────────────────────┘

   Each node: ── LSM write path ──▶ commit log ─▶ memtable ─▶ flush ─▶ SSTables
                                                              + bloom filter + compaction
```

### 6. Deep dive

**(A) Consistent hashing + virtual nodes.** A naive `hash(key) % N` reshuffles *everything* when N changes. Consistent hashing maps both keys and nodes onto a ring; adding/removing a node only moves the keys between two adjacent points. **Virtual nodes** (each physical node = 100s of ring tokens) fix two problems: uneven key distribution and uneven load when a node dies (its load spreads across many nodes, not just its one neighbor). See file 07 for the full derivation.

**(B) Quorum reads/writes (tunable consistency).** With RF = N, require W replicas to ack a write and R replicas to answer a read. **W + R > N ⇒ read-your-writes / strong-ish consistency** (read and write sets overlap). Common: N=3, W=2, R=2. Tune for the workload: `W=1` (fast writes, weak), `R=1` (fast reads), `W=N` (durable, slow). This is the AP/CP knob — *the caller* picks per request.

**(C) Conflict resolution.** Under AP + concurrent writes during a partition, two replicas may diverge. Resolve with:
- **Last-write-wins (LWW)** by timestamp — simple, but can silently drop a concurrent write (clock skew is the enemy).
- **Vector clocks** — track causality; if two versions are concurrent (neither descends from the other), surface *both* to the client (or the app) to merge. Dynamo does this; clients resolve.
- **CRDTs** — data types (counters, sets) that merge deterministically without coordination.

**(D) Failure handling.** **Gossip** spreads membership/failure info (no central coordinator). **Hinted handoff:** if a replica is down at write time, a healthy node accepts the write *on its behalf* and forwards it when the dead node recovers → preserves write availability. **Read repair:** on a quorum read, if replicas disagree, push the newest value to the stale ones. **Anti-entropy** with **Merkle trees** periodically reconciles whole ranges between replicas cheaply (compare tree hashes, only sync differing leaves).

### 7. Data flow

**Put:** client hits any node (the coordinator) → `hash(key)` → find preference list → send write to all RF replicas → wait for **W** acks → return success (attach version/vector clock). Down replicas get **hinted handoff**.

**Get:** coordinator → request from RF replicas → wait for **R** responses → if versions agree, return; if they conflict, **read-repair** the stale ones and return the newest (or both, with vector clocks).

### 8. Scaling

- **Add a node:** it claims virtual-node tokens around the ring; only the key ranges adjacent to those tokens migrate (a small fraction). Linear scaling — double nodes ≈ double throughput/capacity.
- **Hot key:** one key = one preference list = a few nodes. Mitigate with a caching layer in front, or replicate hot keys more widely.
- **Cross-DC:** replicate asynchronously to another region's ring; tune `LOCAL_QUORUM` so latency stays regional while you keep a remote copy for DR.

### 9. Trade-offs

- **CAP:** explicitly **AP** with tunable consistency — the defining choice. During a partition you keep accepting writes (availability) and reconcile later. Flip toward CP by setting `W=N` / `R=N` (slower, may reject during partition).
- **LSM vs B-tree:** LSM = fast sequential writes, read amplification mitigated by bloom filters + compaction; B-tree = fast reads, slower random writes. KV stores pick LSM (write-heavy, append-friendly).
- **LWW vs vector clocks:** simplicity vs correctness under concurrency.

### 10. Follow-up questions

**Q: Why virtual nodes?**
A: Even load distribution and graceful rebalancing. Without them, one node owns one big arc; when it dies, its entire load lands on a single neighbor. With vnodes, each node's tokens are scattered, so a failure's load spreads across the whole cluster.

**Q: How does W+R>N give consistency?**
A: Pigeonhole — if the write set (W nodes) and read set (R nodes) together exceed N, they must overlap by at least one node, so a read always touches a replica that has the latest write.

**Q: How do you detect node failures without a master?**
A: Gossip — nodes randomly exchange heartbeats/membership state; a node not heard from is marked suspect then dead. Decentralized, no SPOF.

**Q: What's hinted handoff vs read repair vs anti-entropy?**
A: Hinted handoff = store a missed write for a down node, replay on recovery (write-time). Read repair = fix stale replicas detected during a read (read-time). Anti-entropy/Merkle = background full reconciliation of ranges (periodic). Three layers of convergence.

---

## 5 — Unique ID Generator

**Asked at:** Twitter (Snowflake), Amazon, Instagram, Flipkart  **Difficulty:** 🟢 Easy  **Level:** SDE-1

Generate 64-bit unique IDs across a distributed fleet — no collisions, roughly time-ordered, no central bottleneck. The expected answer is **Snowflake**; knowing *why* each bit field exists is the signal.

### 1. Requirements

**Functional**
- Generate globally unique 64-bit IDs.
- IDs are **roughly time-sortable** (k-sorted) — useful for DB primary keys, sorting feeds by recency.
- High throughput (10K+/s per node), low latency (in-process, no network call ideally).

**Non-functional**
- **Uniqueness:** absolute — zero collisions, ever.
- **Availability:** generation must not depend on a single central service.
- **Latency:** sub-microsecond (local) is ideal.
- **Ordering:** monotonic-ish by time (not strictly total, but close).

### 2. Estimation

```
Throughput target: say 100K IDs/s globally, peak 1M/s.
Snowflake gives 4096 IDs/ms/node = 4.09M IDs/s per node → one node covers peak easily.
With 1024 node IDs available, ceiling ≈ 4 billion IDs/s. Never a throughput problem.
ID space: 2^63 ≈ 9.2 × 10^18 IDs — inexhaustible.
```

### 3. Approaches

**1) UUID v4 (random 128-bit).**
- *Pro:* no coordination, generate anywhere.
- *Con:* 128 bits (big PK, hurts index locality), **not sortable**, random → bad B-tree insert locality (page splits everywhere).

**2) DB auto-increment.**
- *Pro:* simple, sortable, unique.
- *Con:* single DB = bottleneck + SPOF. Multi-DB with offset stepping (server A: 1,3,5…; B: 2,4,6…) scales poorly and breaks when you add nodes.

**3) Ticket server (Flickr).** One DB whose only job is `REPLACE INTO ... ; SELECT LAST_INSERT_ID()`. Run two with odd/even offsets for HA.
- *Pro:* simple, 64-bit, sortable. *Con:* still a central hop + SPOF risk.

**4) Snowflake (recommended).** Build the 64-bit ID from time + machine + sequence, fully in-process:

```
 0 │ 41 bits timestamp (ms since custom epoch) │ 10 bits machine ID │ 12 bits sequence
─┴──────────────────────────────────────────────┴───────────────────┴─────────────────
 1   41 bits → 2^41 ms ≈ 69 years of IDs          10 → 1024 nodes      12 → 4096 IDs/ms/node
 sign
```

- **41-bit timestamp** (custom epoch, not 1970, to maximize lifespan) → IDs sort by time.
- **10-bit machine ID** → up to 1024 generators; assigned via Zookeeper/config at boot.
- **12-bit sequence** → 4096 IDs per millisecond per machine; resets each ms.

Two IDs on the same machine in the same ms differ in the sequence; across machines they differ in machine ID; across ms they differ in timestamp → globally unique, no coordination after the machine ID is assigned.

### 4. The generation algorithm

```java
synchronized long nextId() {
    long now = currentTimeMillis();
    if (now < lastTs) throw new ClockMovedBackwards();   // NTP rollback → refuse/wait
    if (now == lastTs) {
        seq = (seq + 1) & 0xFFF;          // 12-bit sequence
        if (seq == 0) now = waitNextMillis(lastTs);  // sequence exhausted this ms → spin to next ms
    } else {
        seq = 0;
    }
    lastTs = now;
    return ((now - EPOCH) << 22) | (machineId << 12) | seq;
}
```

### 5. Architecture

```
   App instance ──┐
   App instance ──┼── each embeds an ID generator (in-process, no network call)
   App instance ──┘            │
                               ▼
                     ┌──────────────────────┐
                     │ Zookeeper / etcd      │  ← assigns a unique 10-bit machineId
                     │ (only at startup)     │     to each instance on boot
                     └──────────────────────┘
```

### 6. Deep dive — the clock problem

The Achilles heel of Snowflake is **clock skew / NTP rollback**. If the machine clock jumps backward, you can regenerate timestamps you already used → collisions.
- **Detect:** if `now < lastTs`, the clock moved back. Options: (a) refuse to generate until the clock catches up (small stall, safest); (b) borrow from a logical clock; (c) if the drift is tiny (a few ms), wait it out.
- **Machine ID assignment:** must be unique per running instance. Zookeeper ephemeral sequential nodes hand out IDs and reclaim them when an instance dies, so 1024 IDs are reused safely.
- **Sequence exhaustion:** 4096/ms is plenty; if you somehow exceed it, spin-wait to the next millisecond.

### 7. Data flow

Boot: instance registers with Zookeeper → receives a free 10-bit machine ID. Runtime: `nextId()` is a local, lock-light (synchronized/atomic) bit-twiddle — no network, sub-microsecond.

### 8. Scaling

- Throughput scales with node count (each node = 4M IDs/s); 1024-node ceiling = 4B/s. Beyond that, steal bits (fewer sequence bits + more machine bits) or shorten the epoch granularity.
- No central bottleneck at generation time → linear.

### 9. Trade-offs

- **Sortable vs random:** Snowflake is time-sortable (good PK locality, good for feeds) but **leaks creation time and rate** (competitors can infer your sign-up volume — Snowflake IDs are partially enumerable). UUIDv4 is opaque but unsortable and bigger.
- **Coordination:** Snowflake coordinates only at boot (machine ID); UUID never coordinates; ticket server coordinates every call.
- **CAP:** generation is local → effectively AP and partition-immune once machine IDs are assigned.

### 10. Follow-up questions

**Q: Why 41 bits for time?**
A: 2^41 ms ≈ 69 years. With a custom epoch set to your launch date, you get ~69 years before overflow — enough, and it keeps IDs 64-bit (fits a BIGINT, half a UUID's size).

**Q: What happens if the clock goes backward?**
A: You risk reusing a (timestamp, sequence) pair → collision. Detect `now < lastTs` and either stall until the clock recovers or fail fast. Never blindly generate.

**Q: Why not just UUIDs?**
A: 128 bits bloats every index/PK; random UUIDs destroy B-tree insert locality (random page writes) and aren't sortable, so you can't range-scan by time. Snowflake is half the size, sortable, and index-friendly.

**Q: How are machine IDs assigned uniquely?**
A: Zookeeper/etcd ephemeral sequential znodes — each booting instance claims the next free ID; when it dies, the ephemeral node vanishes and the ID is reclaimed.

---

## 6 — Leaderboard

**Asked at:** Riot, EA, Zynga, Amazon, Dream11, gaming companies  **Difficulty:** 🟢 Easy  **Level:** SDE-1

"Top-K players by score, with each player's own rank, updated in real time." The whole question hinges on knowing the right data structure: a **sorted set** (Redis ZSET / skip list). Then the scaling story is about millions of players.

### 1. Requirements

**Functional**
- Update a player's score (absolute set or increment).
- Get the **top K** players (e.g. top 10).
- Get a specific player's **rank** and score.
- Get a window around a player ("players ranked 995–1005 near me").
- Time-bounded boards (daily/weekly/all-time).

**Non-functional**
- **Latency:** all ops <10 ms — leaderboards are shown live in-game.
- **Availability:** 99.9%; minor staleness OK.
- **Consistency:** eventual is fine (a rank off by one for a moment is harmless).
- **Scale:** 50M players, 10M DAU, score updates flooding in during peak events.

### 2. Back-of-envelope estimation

```
DAU 10M, each plays ~10 games/day, score update per game:
   writes = 10M × 10 / 86,400 ≈ 1,160 updates/s avg, ~5,000/s peak.
Reads (everyone checks rank often): ~10× writes ≈ 50,000 reads/s peak.

Memory for a Redis ZSET of 50M players:
   per member ≈ key(~20 B) + score(8 B) + skiplist/dict overhead (~64 B) ≈ ~90–100 B
   50M × 100 B ≈ 5 GB → fits one Redis node; shard if multiple boards.
```

**Takeaway:** entirely in-memory in a ZSET. The naive "SQL `ORDER BY score`" approach would scan/sort on every read → dies at this scale.

### 3. API design

```
POST /leaderboard/{boardId}/score   { userId, score }     // or {delta} to increment
GET  /leaderboard/{boardId}/top?k=10                       → [{rank,userId,score}...]
GET  /leaderboard/{boardId}/rank/{userId}                  → {rank, score}
GET  /leaderboard/{boardId}/around/{userId}?range=5        → 11 entries centered on user
```

### 4. Data model

```
Redis sorted set per board:
   ZADD  board:weekly  <score>  <userId>
   ZREVRANGE  board:weekly  0 9  WITHSCORES        → top 10
   ZREVRANK   board:weekly  <userId>               → rank (0-based)
   ZINCRBY    board:weekly  <delta>  <userId>      → increment
   ZREVRANGE  board:weekly  (rank-5) (rank+5)      → window around user

Source-of-truth DB (durable, ZSET is a fast index over this):
   TABLE scores(user_id PK, board_id, score, updated_at)   -- for recovery/audit
```

A **ZSET is a skip list + hash map**: O(log N) inserts/updates and rank queries, O(log N + K) for top-K. That's the entire reason this problem is "easy" once you name the structure.

### 5. Architecture

```
   Game server ──score update──▶ ┌──────────┐    ┌──────────────────────────┐
                                  │   API    │───▶│ Redis ZSET (the board)    │
   Player app ──get rank/top────▶ │  servers │    │ skiplist: O(logN) ops     │
                                  └────┬─────┘    └──────────────────────────┘
                                       │ async write-behind
                                       ▼
                              ┌────────────────────┐
                              │ Durable DB (scores) │  ← rebuild ZSET on restart
                              └────────────────────┘
                              Time boards: separate keys board:daily:{date}, TTL'd
```

### 6. Deep dive

**(A) Why ZSET, not SQL.** `SELECT ... ORDER BY score DESC LIMIT 10` re-sorts (or scans an index) on every read, and computing *one player's rank* (`COUNT(*) WHERE score > mine`) is O(N). A skip-list-backed sorted set keeps elements ordered, so top-K is O(log N + K) and rank is O(log N) — orders of magnitude faster, and it stays correct under a flood of updates.

**(B) Time-bounded boards.** Use a separate ZSET per period: `board:daily:2026-06-11`, `board:weekly:2026-W24`. TTL daily/weekly keys for auto-cleanup. All-time is its own permanent ZSET. Score updates fan to all relevant boards.

**(C) Durability.** Redis is the fast index, not the source of truth. Write the score to a durable DB (or rely on Redis AOF/RDB persistence) so you can **rebuild the ZSET** after a crash. On restart, replay scores from the DB into the ZSET.

### 7. Data flow

**Update:** `POST score` → `ZADD/ZINCRBY` to all relevant period boards (O(log N) each) → async-persist to the durable DB.
**Top-K:** `ZREVRANGE board 0 K-1 WITHSCORES` → return.
**My rank:** `ZREVRANK board userId` (+1 for 1-based) and `ZSCORE` for the score.
**Around me:** `ZREVRANK` to find rank r, then `ZREVRANGE board r-w r+w`.

### 8. Scaling

- **One ZSET caps at one node's RAM/throughput (~5 GB / 100K ops/s).** For 100M+ players or higher throughput:
  - **Shard by score range** ("bucketing"): board partitioned into score buckets across nodes; top-K reads the top bucket(s); rank = sum of counts in higher buckets + local rank. More complex but parallel.
  - **Replicas for reads:** reads (50K/s) dwarf writes — add Redis read replicas; writes go to the primary.
  - **Approximate ranks at extreme scale:** for "you're in the top 5%," exact rank past #10,000 rarely matters — bucket coarsely below the top.
- **Hot board (a big tournament):** replicate the board widely; the top-K slice is tiny and cache-friendly (cache top-100 for a few seconds).

### 9. Trade-offs

- **Exact vs approximate rank:** exact is O(log N) in one ZSET but hard to shard; approximate (bucketed) scales horizontally. Beyond the top few thousand, approximate is fine.
- **In-memory vs durable:** ZSET gives speed but needs a durable backing store for recovery.
- **CAP:** AP — stale-by-a-moment ranks are acceptable; availability matters more during a live event spike.

### 10. Follow-up questions

**Q: Why not SQL `ORDER BY`?**
A: Computing a single player's rank is O(N) (count everyone above them) and top-K re-sorts/scans on every read. At 50K reads/s over 50M rows it collapses. A skip-list sorted set makes both O(log N).

**Q: How do you scale past one Redis node?**
A: Shard by score-range buckets across nodes (top-K hits the top buckets, rank sums higher buckets), and add read replicas since the workload is read-heavy.

**Q: How do you support daily/weekly resets?**
A: A ZSET key per period with a TTL; updates fan out to current daily/weekly/all-time keys. Expired keys auto-clean.

**Q: How do you survive a Redis crash?**
A: Persist every score to a durable DB (or enable Redis AOF). On restart, rebuild the ZSET by replaying scores. Redis is a fast index, not the system of record.

---

## 7 — Notification System

**Asked at:** Amazon, Uber, Swiggy, LinkedIn, Microsoft  **Difficulty:** 🟢 Easy (🟡 at fan-out scale)  **Level:** SDE-1

Send notifications across **multiple channels** (push, SMS, email) reliably, at scale, with retries, dedup, and user preferences. The design is a classic **queue + workers + third-party-provider** pipeline.

### 1. Requirements

**Functional**
- Send notifications via **push (APNs/FCM), SMS, email**, in-app.
- Accept from many internal services (order shipped, OTP, marketing blast).
- Respect **user preferences** (opt-out per channel/category) and quiet hours.
- **Retries** on transient provider failure; **dedup** (don't send the same thing twice).
- Templating + localization.
- Track delivery status (sent/delivered/failed/opened).

**Non-functional**
- **Latency:** transactional (OTP) within seconds; marketing can be minutes.
- **Availability:** 99.9% — but the system should *buffer*, not drop, during provider outages.
- **Reliability:** at-least-once delivery (with dedup → effectively once).
- **Scale:** 10M+ notifications/day, with spikes (a marketing blast = millions at once).

### 2. Back-of-envelope estimation

```
10M notifications/day = 10M / 86,400 ≈ 116/s avg.
Marketing blast: 5M in 10 min = 5M / 600 ≈ 8,300/s burst → queue must absorb this.

Storage (status records): 10M/day × ~300 B × 90-day retention
   = 10M × 90 × 300 B ≈ 270 GB → modest, shardable.

Provider rate limits: APNs/FCM/Twilio/SES each cap throughput → workers must
   rate-limit per provider and queue overflow.
```

**Takeaway:** the burst (8,300/s vs 116/s avg) is the design driver → a **durable queue** decouples producers from the slower third-party providers and smooths spikes.

### 3. API design

```
POST /notifications
   { userId, channel: "push"|"sms"|"email"|"auto",
     templateId, params: {...}, category: "transactional"|"marketing",
     idempotencyKey: "order-123-shipped" }
   → 202 Accepted { notificationId }     (async — we accept then deliver)

GET /notifications/{id}        → status
PUT /users/{id}/preferences    → channel/category opt-outs
```

`idempotencyKey` is how producers guarantee they don't double-enqueue (e.g. on retry).

### 4. Data model

```
TABLE notifications
  id PK, user_id, channel, template_id, category, status, idempotency_key (unique),
  created_at, sent_at, provider_msg_id, retry_count
  INDEX (user_id, created_at);  UNIQUE (idempotency_key)   ← dedup at the door

TABLE user_preferences
  user_id PK, push_enabled, sms_enabled, email_enabled, marketing_opt_in, quiet_hours, locale

TABLE device_tokens
  user_id, platform (ios/android), token, updated_at     ← for push targeting

TABLE templates
  template_id PK, channel, locale, subject, body_template
```

### 5. High-level architecture

```
 Internal services ──POST──▶ ┌────────────────────┐
 (order, auth/OTP, mktg)     │ Notification API    │  validate, dedup (idempotencyKey),
                             │ (ingestion)         │  resolve prefs+template, enqueue
                             └─────────┬───────────┘
                                       ▼
                        ┌──────────────────────────────┐
                        │  Message Queue (Kafka/SQS)     │  ← buffers bursts, durable
                        │  topics per channel + priority │     (transactional > marketing)
                        └───┬──────────┬──────────┬──────┘
                            ▼          ▼          ▼
                       ┌────────┐ ┌────────┐ ┌────────┐
                       │ Push   │ │ SMS    │ │ Email  │   channel workers
                       │ worker │ │ worker │ │ worker │   (rate-limit per provider,
                       └───┬────┘ └───┬────┘ └───┬────┘    retry w/ backoff, dedup)
                           ▼          ▼          ▼
                       ┌────────┐ ┌────────┐ ┌────────┐
                       │APNs/FCM│ │ Twilio │ │  SES   │   3rd-party providers
                       └───┬────┘ └───┬────┘ └───┬────┘
                           └──────────┴──────────┘
                                      ▼ delivery webhooks/callbacks
                            ┌──────────────────────┐
                            │ Status DB + Analytics │  sent/delivered/opened
                            └──────────────────────┘
                            Failed after N retries ──▶ Dead Letter Queue
```

### 6. Deep dive

**(A) Why a queue.** Producers (your services) are fast; providers (APNs, Twilio) are slow and rate-limited and *go down*. A durable queue (Kafka/SQS) **decouples** them: the API returns `202` instantly, and workers drain the queue at the providers' pace. A 5M marketing blast lands in the queue in seconds and is delivered over minutes without dropping anything or overloading providers. Use **priority topics** so a marketing blast never delays an OTP.

**(B) Reliability: retries, dedup, idempotency.** Delivery is **at-least-once** — a worker may crash after sending but before acking, so the message is redelivered. Make it **effectively-once** with: (1) the producer's `idempotencyKey` (unique index rejects duplicates at ingestion), and (2) workers checking "already sent?" before calling the provider. Transient failures (provider 5xx, timeout) → **exponential backoff retry**; after N failures → **Dead Letter Queue** for inspection/alerting.

**(C) Preferences + dedup of intent.** Before enqueueing, check `user_preferences` (channel opt-out, marketing opt-in, quiet hours → defer). "auto" channel resolves to the user's preferred reachable channel (push if a device token exists, else SMS/email). This keeps you compliant and avoids annoying users.

**(D) Fan-out.** A single "promo" request targeting 5M users explodes into 5M individual notification jobs. Do the fan-out in a worker (read the audience in batches, enqueue per-user jobs) — never synchronously in the API request.

### 7. Data flow

1. Service `POST /notifications` with an `idempotencyKey`.
2. API: dedup (unique key) → load prefs (opted out? quiet hours? → drop/defer) → render template → write a `notifications` row (status=QUEUED) → enqueue to the channel+priority topic → return `202`.
3. Channel worker pulls a batch, rate-limited per provider; checks "not already sent"; calls the provider; on success marks SENT + stores `provider_msg_id`; on transient failure re-queues with backoff; on permanent failure → DLQ.
4. Provider delivery webhook → update status to DELIVERED/OPENED/BOUNCED.

### 8. Scaling

- **Bursts:** the queue absorbs them; scale workers horizontally (more consumers per partition) to drain faster.
- **Provider limits:** per-provider token-bucket rate limiting in workers; shard across multiple provider accounts/regions if one isn't enough.
- **Fan-out (millions):** batch-read the audience, parallelize enqueue across many fan-out workers, partition the topic by user-hash so workers parallelize.
- **Status writes (10M/day):** shard the status DB by `notification_id`; or stream status into an analytics store rather than a hot OLTP table.

### 9. Trade-offs

- **At-least-once vs exactly-once:** true exactly-once across third parties is impossible; we do at-least-once + idempotency = effectively-once.
- **Sync vs async:** always async (`202` + queue) — you can't block a request on a flaky third party.
- **Priority isolation:** separate queues/topics so marketing volume never starves transactional OTPs (a latency-vs-cost trade).
- **CAP:** AP — buffer and retry rather than fail; brief delivery delay beats dropping notifications.

### 10. Follow-up questions

**Q: How do you avoid sending the same notification twice?**
A: Two layers — producers supply an `idempotencyKey` enforced by a unique index at ingestion, and workers re-check "already sent" before calling the provider (because queue delivery is at-least-once).

**Q: How do you keep OTPs fast during a marketing blast?**
A: Separate priority queues/topics. Transactional consumers are isolated from the marketing flood, so an 8K/s blast in the marketing topic never delays the OTP topic.

**Q: A provider (Twilio) is down — what happens?**
A: Workers get errors, retry with exponential backoff; messages stay in the durable queue (not lost). If the outage is long, messages buffer; after N retries they go to a DLQ and you alert/fallback to another channel/provider.

**Q: How do you fan out a notification to 5M users?**
A: Asynchronously — a fan-out worker reads the audience in batches and enqueues per-user jobs (partitioned by user hash). Never expand the audience synchronously in the API call.

---

## 8 — File Storage Service (Dropbox-lite)

**Asked at:** Dropbox, Google (Drive), Microsoft (OneDrive), Box  **Difficulty:** 🟢 Easy to frame / 🟡 to do sync well  **Level:** SDE-1

Upload, download, and **sync** files across a user's devices. The core ideas: **chunking + dedup**, **metadata vs blob separation**, and **sync via a change journal**.

### 1. Requirements

**Functional**
- Upload/download files (up to GBs).
- **Sync** changes across all the user's devices automatically.
- Share files/folders with other users.
- File versioning (restore previous versions).
- Work offline; reconcile on reconnect.

**Non-functional**
- **Durability:** never lose a byte (11 nines via S3).
- **Availability:** 99.9%.
- **Latency:** fast metadata ops; uploads/downloads bandwidth-bound.
- **Efficiency:** don't re-upload unchanged data (delta sync), dedup identical chunks.
- **Scale:** 100M users, hundreds of PB.

### 2. Back-of-envelope estimation

```
100M users, 10M DAU, avg 100 files each, avg file 1 MB (skewed — some GBs).
Total stored: 100M × 100 × 1 MB ≈ 10 PB (before dedup; dedup cuts ~30%).

Uploads: 10M DAU × 10 file-changes/day = 100M writes/day ÷ 86,400 ≈ 1,160/s avg.
   With chunking (4 MB chunks), large files = many chunk PUTs.
Downloads/syncs: read:write ≈ 10:1 → ~12K/s.
Metadata ops (list, stat, sync poll): much higher, ~100K/s — these are tiny and DB-bound.
Bandwidth: 12K downloads/s × 1 MB ≈ 12 GB/s → CDN + S3 handle this, not your servers.
```

**Takeaway:** blobs go to **S3** (10 PB, CDN-fronted); metadata (small, high-QPS, relational) goes to a **sharded DB**; the interesting engineering is **chunking, dedup, and sync**.

### 3. API design

```
POST /files/upload/init     { fileName, size, chunkHashes:[...] }
   → { uploadId, missingChunks:[hashes the server doesn't already have] }   ← dedup!
PUT  /chunks/{hash}          (presigned S3 URL) — upload only missing chunks
POST /files/upload/commit    { uploadId, chunkList } → { fileId, version }

GET  /files/{fileId}         → metadata + chunk list → download chunks from CDN/S3
GET  /sync?cursor=<token>    → changes since cursor (long-poll/websocket)  ← the sync journal
POST /files/{id}/share       { targetUser, permission }
```

### 4. Data model

```
TABLE files
  file_id PK, owner_id, name, parent_folder_id, current_version, size,
  is_deleted, updated_at
  INDEX (owner_id, parent_folder_id)            -- list a folder
  SHARD KEY: owner_id  (a user's files colocated)

TABLE file_versions
  file_id, version, chunk_list (ordered hashes), created_at, device_id
  PK (file_id, version)

TABLE chunks                                    -- content-addressed dedup table
  chunk_hash PK (SHA-256 of content), s3_key, size, ref_count
  -- identical content anywhere → one stored chunk

TABLE sync_journal                              -- the change feed per user
  user_id, seq (monotonic), file_id, change_type, version, ts
  PK (user_id, seq)                             -- clients poll "give me changes after seq N"

OBJECT STORE (S3): bucket/chunks/{chunk_hash}   -- the actual bytes, content-addressed
```

### 5. High-level architecture

```
   Device A ──┐                ┌──────────────┐    ┌──────────────────────────┐
   Device B ──┼── sync client ─│  API Gateway  │───▶│ Metadata Service          │
   Device C ──┘  (chunk, hash, │  / LB         │    │ (files, versions, chunks) │
                  watch FS)    └──────┬────────┘    └──────────┬───────────────┘
                                      │                         ▼ shard by owner_id
                        upload missing chunks            ┌──────────────────┐
                        (presigned, direct)              │ Metadata DB       │
                                      │                  │ (sharded SQL)     │
                                      ▼                  └──────────────────┘
                          ┌────────────────────────┐
                          │  Block/Chunk Storage    │  content-addressed by hash
                          │  S3 (+ CDN for download) │  dedup via ref_count
                          └────────────────────────┘
                                      ▲
                          ┌───────────┴────────────┐
                          │  Notification/Sync svc  │  pushes "you have changes"
                          │  (long-poll / WebSocket)│  to a user's other devices
                          └─────────────────────────┘
```

### 6. Deep dive

**(A) Chunking + content-addressed dedup.** Split each file into fixed (4 MB) chunks; hash each chunk (SHA-256). The hash *is* the storage key (content-addressed). On upload, the client sends only the chunk *hashes* first; the server replies which chunks it doesn't already have (`missingChunks`), and the client uploads only those. Benefits: (1) **dedup** — if a chunk's bytes already exist (same file shared, or a common library), `ref_count++` and skip the upload; (2) **delta sync** — editing one chunk of a 1 GB file re-uploads only that 4 MB chunk; (3) **resumable** uploads — retry only missing chunks. (Production Dropbox uses *content-defined* chunking so an insertion doesn't shift all subsequent chunk boundaries.)

**(B) Sync via a change journal.** Each user has a monotonic `sync_journal` (seq, file_id, change). Devices keep a **cursor** (last seq seen) and ask "what changed since cursor?" via **long-poll or WebSocket**. When device A commits a change, the journal advances and the **notification service** pings A's other devices, which then pull the delta and download only the changed chunks. This is far cheaper than re-scanning the whole tree.

**(C) Metadata vs blobs.** Metadata (files, folders, versions, chunk lists) is small, relational, and high-QPS → sharded SQL keyed by `owner_id` (a user's files/folders colocated for fast listing). Blobs are huge and immutable → S3, content-addressed, CDN-fronted for downloads. Never store blobs in the DB.

**(D) Conflict resolution.** Two devices edit the same file offline → on reconnect, both versions exist in `file_versions`. Strategy: keep both ("filename (conflicted copy from Device B)") rather than silently overwrite — last-write-wins loses data. For text, optionally offer a merge.

### 7. Data flow

**Upload:** client chunks + hashes the file → `upload/init` with hashes → server returns `missingChunks` (dedup) → client PUTs only missing chunks to S3 (presigned, direct) with `ref_count++` → `upload/commit` writes a new `file_version` (chunk list) and appends to the user's `sync_journal` → notification service pings other devices.

**Sync (other device):** notified (or long-poll) → `GET /sync?cursor=N` → receives changed file_ids/versions → fetches new chunk lists → downloads only chunks it lacks from CDN/S3 → updates local copy → advances cursor.

### 8. Scaling

- **Blobs:** S3 + CDN are effectively infinite; dedup cuts ~30% storage.
- **Metadata:** shard by `owner_id`; hot users (huge folders) are rare and bounded.
- **Sync fan-out:** notification service holds millions of long-poll/WebSocket connections → horizontal connection servers (like a chat system's gateway, see #11).
- **Hot file (a shared 1 GB file downloaded by 10K people):** CDN edge-caches the chunks; origin S3 barely sees it.

### 9. Trade-offs

- **Fixed vs content-defined chunking:** fixed is simple but a 1-byte insertion at the start shifts every boundary (re-uploads everything); content-defined chunking keeps boundaries stable across edits (better dedup) at more CPU.
- **Strong vs eventual sync consistency:** sync is eventually consistent (devices converge); conflicts kept as copies, not auto-merged, to avoid data loss.
- **SQL (metadata) + object store (blobs):** the canonical split — relational queries for folders/sharing, infinite cheap durable storage for bytes.
- **CAP:** metadata leans CP (you want consistent folder structure); blob layer is AP/eventual.

### 10. Follow-up questions

**Q: How do you avoid re-uploading a huge file when one line changes?**
A: Chunking + content-addressed hashing. Only the changed chunk's hash differs, so `upload/init` reports just that chunk as missing; you upload 4 MB, not 1 GB.

**Q: How does dedup work across users?**
A: Chunks are stored keyed by their content hash with a `ref_count`. If user B uploads a chunk whose bytes already exist (uploaded by user A), the server just increments `ref_count` — one physical copy, two references. Delete decrements; GC when it hits zero.

**Q: How do devices learn about changes without polling constantly?**
A: A per-user monotonic sync journal + a notification service over WebSocket/long-poll. Devices hold a cursor and are pushed a "you have changes" signal, then pull only the delta since their cursor.

**Q: How do you handle two devices editing offline?**
A: Both versions land in `file_versions` on reconnect; we keep both as "conflicted copies" rather than last-write-wins, because silently overwriting loses a user's work. Optionally offer a merge for text.

**Q: Why not store files in the database?**
A: GB-scale blobs destroy DB row size, backup, and replication. Object storage (S3) is cheaper, 11-nines durable, infinitely scalable, content-addressable for dedup, and CDN-frontable. DB holds only small queryable metadata.

---

# 🟡 MEDIUM (SDE-1/2)

---

## 9 — Twitter / News Feed

**Asked at:** Twitter/X, Meta (Facebook feed, Instagram feed), LinkedIn, Pinterest, ShareChat, Amazon  **Difficulty:** 🟡 Medium  **Level:** SDE-1/2

The single most important medium-difficulty design. The entire problem reduces to one decision: **fan-out-on-write vs fan-out-on-read**, and the celebrity (hot-key) problem that forces a hybrid. Nail this and you can derive Instagram, LinkedIn, and most social feeds on the spot.

### 1. Requirements

**Functional**
- Post a tweet (text + media), up to 280 chars.
- **Home timeline:** a user sees a reverse-chronological (or ranked) feed of tweets from people they follow.
- Follow/unfollow users.
- User timeline: all tweets by one user.
- Like, retweet, reply (engagement).
- Search/trending (often deferred).

**Non-functional**
- **Latency:** home timeline load <200 ms (p99) — this is the product. Posting can be slightly slower.
- **Availability:** 99.99% — feed must always render (stale beats blank).
- **Consistency:** eventual — it's fine if a tweet takes a few seconds to appear in followers' feeds.
- **Read-heavy:** read:write ≈ 100:1 (people scroll far more than they post).
- **Scale:** 300M MAU, 150M DAU, ~500M tweets/day.

### 2. Back-of-envelope estimation

```
DAU = 150M.
Tweets/day = 500M  →  500M / 86,400 ≈ 5,800 tweets/s avg, ~12K/s peak (write QPS).

Home-timeline reads: each DAU refreshes ~10×/day:
   150M × 10 = 1.5B reads/day  →  1.5B / 86,400 ≈ 17,000 reads/s avg, ~50K/s peak.
   (Read:write ≈ 17K : 5.8K ≈ ~3:1 at the timeline API; far higher if you count scroll/pagination.)

Storage per tweet:
   tweet_id 8 B + user_id 8 B + text 280 B + metadata ~100 B ≈ ~400 B; round to 1 KB w/ index.
   500M/day × 1 KB = 500 GB/day  →  ~180 TB/year (text only; media separate in S3).

Fan-out math (the crux):
   Avg followers ≈ 200. Each tweet by an average user must be written into ~200 timelines.
   500M tweets/day × 200 = 100B timeline writes/day = 100B/86,400 ≈ 1.16M writes/s to timeline cache.
   A celebrity with 100M followers → one tweet = 100M timeline writes. THIS is why pure
   fan-out-on-write breaks.

Timeline cache memory: cache ~800 recent tweet-IDs per active user.
   150M DAU × 800 IDs × 8 B = 150M × 6.4 KB ≈ ~1 TB of timeline IDs → a Redis cluster (~40 nodes).
   (Store only tweet IDs in the timeline; hydrate tweet content from a separate cache.)
```

**Takeaway:** writes are modest (12K/s) but the **fan-out amplification** (1.16M timeline writes/s, spiking to 100M for one celebrity tweet) is the whole game. Reads must be cheap (precomputed) because they dominate the product experience.

### 3. API design

```
POST /v1/tweet
   { text, mediaIds:[...] }  (auth → author from token)
   → 201 { tweetId, createdAt }

GET /v1/feed?cursor=<tweetId>&limit=50      ← home timeline (the hot path)
   → { tweets:[{id, author, text, media, likeCount, ...}], nextCursor }

POST   /v1/follow/{userId}
DELETE /v1/follow/{userId}
GET    /v1/user/{userId}/tweets?cursor=...   ← user timeline
POST   /v1/tweet/{id}/like
```

Cursor-based pagination (not offset) — offsets break when new tweets are inserted at the head.

### 4. Data model / schema

```
TABLE tweets                                   (source of truth, sharded by tweet_id)
  tweet_id BIGINT PK (Snowflake — time-sortable!),  author_id, text, media_ids,
  created_at, reply_to, retweet_of
  SHARD KEY: tweet_id (hash)        — point lookups by id when hydrating timelines

TABLE follows                                  (the social graph)
  follower_id, followee_id, created_at
  PK (follower_id, followee_id)
  INDEX (followee_id)               — "who follows X?" (needed for fan-out)
  SHARD KEY: follower_id            — "who do I follow?" reads are colocated

TABLE user_timeline   (a user's own tweets — list by author)
  author_id, tweet_id  — INDEX(author_id, tweet_id DESC)

REDIS  home_timeline:{userId} → LIST/ZSET of recent tweet_ids (the precomputed feed)
REDIS  tweet:{tweetId}        → cached tweet content (hydration)
```

**Why `tweet_id` is a Snowflake ID:** it's time-sortable, so a timeline is just "the highest tweet_ids," and cursor pagination is a simple `id < cursor` — no separate timestamp sort needed.

### 5. High-level architecture

```
                                   ┌──────────────────────────────────────────────┐
  POST /tweet ──▶ ┌──────────┐     │              WRITE PATH                        │
                  │   LB     │────▶│  Tweet Service ─▶ tweets DB (sharded by id)     │
                  └──────────┘     │       │                                         │
                                   │       └─▶ Fan-out Service ──▶ Kafka             │
                                   │             (reads followers,                   │
                                   │              pushes tweet_id into                │
                                   │              each follower's home_timeline)      │
                                   └───────────────────┬──────────────────────────────┘
                                                       ▼
                                          ┌────────────────────────────┐
                                          │ Redis: home_timeline:{uid}  │  precomputed feeds
                                          │   = [tweetId, tweetId, ...] │  (fan-out-on-write)
                                          └─────────────┬──────────────┘
  GET /feed ────▶ ┌──────────┐                          │
                  │   LB     │────▶ Timeline Service ────┘ 1. read tweet_ids from Redis
                  └──────────┘            │                2. MERGE in celebrity tweets (pull)
                                          │                3. hydrate content from tweet cache
                                          ▼                4. rank, return page
                              ┌────────────────────────┐
                              │ tweet content cache +   │
                              │ tweets DB (hydration)   │
                              └────────────────────────┘
   Media (images/video) ──▶ uploaded to S3, served via CDN; tweets store media_ids only.
```

### 6. Deep dive — fan-out-on-write vs fan-out-on-read (THE decision)

**Fan-out-on-write (push model).** When a user tweets, immediately **push** the tweet_id into the precomputed `home_timeline` list of every follower (via the fan-out service + Kafka workers).
- *Read* = trivial: `GET home_timeline:{me}` → already-assembled list of IDs → hydrate → done. **O(1) read, blazing fast.**
- *Write* = expensive: one tweet = (number of followers) timeline writes. Fine for 200 followers; **catastrophic for a celebrity** (100M writes for one tweet → "fan-out storm," and most of those followers may never log in).
- *Wastes* work writing into inactive users' timelines.

**Fan-out-on-read (pull model).** Store only each user's own tweets. At read time, **pull** the latest tweets from everyone you follow and merge them on the fly.
- *Write* = trivial: just append to your own tweet list.
- *Read* = expensive: fetch + merge tweets from (say) 1,000 followees on every feed load → slow, hammers the DB, hard to cache. Bad for the read-heavy product.

**Hybrid (what Twitter actually does — the expected answer).**
- **Push for normal users** (≤ a threshold of followers, e.g. <10K): fan-out-on-write into followers' timelines. Cheap and gives fast reads for the common case.
- **Pull for celebrities** (followers above the threshold): do **not** fan out their tweets. Instead, at read time, the timeline service **merges** a small number of "followed celebrities' recent tweets" (pulled live, easily cached) into the user's mostly-precomputed timeline.
- This caps fan-out write amplification (no 100M-write storms) while keeping reads fast (the precomputed part is already there; the celebrity merge is a handful of cached lookups).

```
Home feed assembly for user U:
   precomputed = home_timeline:{U}                     (push, from non-celebrity followees)
   live        = for each celebrity C that U follows:  (pull)
                    recent tweets of C  (cached: celeb_recent:{C})
   feed = merge-by-tweet_id-desc(precomputed, live)    (tweet_id is time-sortable)
   hydrate top 50, rank, return.
```

**(B) Timeline storage.** Store only **tweet IDs** in `home_timeline:{uid}` (a capped list/ZSET of ~800 ids), not full tweet content — otherwise a popular tweet is duplicated into millions of timelines (storage explosion) and edits/deletes are impossible. Hydrate content separately from `tweet:{id}` cache. Cap the list (LRU/trim) so memory is bounded.

**(C) Handling the fan-out asynchronously.** Posting returns immediately after writing to the `tweets` DB. The fan-out is a background job: Tweet Service emits the new tweet to Kafka; fan-out workers read the author's followers (from `follows` indexed by `followee_id`) and push the tweet_id into each follower's Redis timeline. Eventual consistency: a tweet appears in feeds within ~seconds.

### 7. Data flow

**Post a tweet:**
1. `POST /tweet` → Tweet Service → write to `tweets` DB (Snowflake id) + tweet content cache.
2. Emit tweet event to Kafka. Return `201` to the user (don't wait for fan-out).
3. Fan-out workers: is author a celebrity (followers > threshold)? If yes → skip fan-out (pull at read). If no → look up followers, push `tweet_id` into each follower's `home_timeline` Redis list (trim to cap).

**Load home feed:**
1. `GET /feed` → Timeline Service.
2. Read precomputed `home_timeline:{me}` IDs from Redis.
3. Pull recent tweets of the celebrities `me` follows (from `celeb_recent:{C}` cache), merge by tweet_id desc.
4. Hydrate the top `limit` tweet IDs from the tweet content cache (batch `MGET`).
5. Apply ranking (recency, engagement) if not purely chronological.
6. Return the page + `nextCursor`.

### 8. Scaling

- **10x (50K tweets/s):** more fan-out workers + more Kafka partitions; shard `tweets` and `follows` further; grow the Redis timeline cluster.
- **100x:** the bottleneck is fan-out write amplification → the celebrity threshold and hybrid model are doing the heavy lifting; tune the threshold down as the graph gets more skewed. Shard timelines by `userId` across the Redis cluster (consistent hashing).
- **1000x / global:** multi-region. Keep a user's timeline in their home region; replicate tweets cross-region. Rank with an ML service (out of scope, but mention it: the feed becomes ranked, not chronological, with a feature store).
- **Bottlenecks:** (1) celebrity fan-out → hybrid pull; (2) timeline cache memory → store IDs only + cap; (3) hydration → batch MGET + tweet content cache; (4) `follows` "who follows X" for a 100M-follower account → don't materialize it, use the pull path.

### 9. Trade-offs

- **Push vs pull:** push = fast reads/expensive writes (good for the 99% with few followers); pull = cheap writes/expensive reads (good for celebrities). Hybrid takes the best of each. This is *the* trade-off the interviewer wants articulated.
- **CAP:** **AP** — eventual consistency is acceptable (a tweet appearing in your feed a few seconds late is fine); availability of the feed is paramount.
- **SQL vs NoSQL:** tweets and the timeline are append-mostly, point-lookup-by-id, massive → wide-column/NoSQL (Cassandra/Manhattan) or sharded SQL. The social graph (`follows`) is also huge; often a dedicated graph/KV store. Redis for the hot precomputed timelines.
- **Store IDs vs full tweets in timelines:** IDs (dedup, editable, bounded memory) vs full content (one fewer hydration hop but storage explosion). IDs win.

### 10. Follow-up questions

**Q: How do you handle a celebrity with 100M followers?**
A: Don't fan out their tweets on write (a 100M-write storm). Use the pull path: at read time, merge the celebrity's recent tweets (cheaply cached) into each follower's otherwise-precomputed timeline. Hybrid model.

**Q: A new user follows 500 people — their timeline is empty. How do you backfill?**
A: On follow (or first feed load), do a one-time pull: fetch recent tweets from the newly-followed users and merge into their timeline, then let the push model maintain it going forward.

**Q: How do you keep the timeline in time order if fan-out is async and parallel?**
A: Sort by `tweet_id` (Snowflake → time-ordered) at read time, not by arrival into the timeline. So even if pushes land out of order, the merged feed is correctly ordered.

**Q: How do you handle deletes/edits if the tweet is duplicated in millions of timelines?**
A: That's exactly why timelines store only tweet *IDs*, not content. Delete/edit the single source-of-truth tweet (and content cache); at hydration time a deleted ID is just skipped. No need to touch millions of timelines.

**Q: Chronological vs ranked feed?**
A: Chronological = merge by tweet_id. Ranked = a scoring service ranks candidate tweets by predicted engagement (recency, affinity, content features) via an ML model + feature store. The candidate-generation (fan-out) part is unchanged; ranking is a layer on top.

**Q: How big do you make the celebrity threshold?**
A: Tune by measuring fan-out cost vs read-merge cost. A common value is ~10K–100K followers: above it, the write amplification outweighs the read-merge cost, so pull wins.

---

## 10 — Instagram / Photo Sharing

**Asked at:** Meta (Instagram), Pinterest, Snap, Flipkart  **Difficulty:** 🟡 Medium  **Level:** SDE-1/2

Instagram = Twitter's feed (#9) + a heavy **media pipeline** (upload, process, store, CDN-serve photos/videos). Reuse the fan-out model wholesale; the new depth is media handling.

### 1. Requirements

**Functional**
- Upload photos/videos with captions.
- Home feed of posts from followed accounts (same fan-out problem as Twitter).
- Follow/unfollow; user profile grid.
- Like, comment.
- Stories (ephemeral, 24h) — optional.
- Explore/search — optional.

**Non-functional**
- **Latency:** feed <200 ms; image load fast via CDN.
- **Availability:** 99.99%.
- **Durability:** never lose a user's photos (S3, 11 nines).
- **Read-heavy:** ~100:1.
- **Scale:** 500M DAU, 100M photos/day.

### 2. Back-of-envelope estimation

```
100M photos/day → 100M/86,400 ≈ 1,160 uploads/s avg, ~3K/s peak.
Avg photo (post-compression, multiple resolutions stored): ~1.5 MB across variants.
Media storage/year: 100M × 1.5 MB × 365 ≈ 55 PB/year → S3.

Feed reads: 500M DAU × 20 refreshes = 10B/day ≈ 116K reads/s avg, ~300K/s peak.
Image bandwidth: 300K img-loads/s × 200 KB (a feed thumbnail) ≈ 60 GB/s → CDN absorbs this.
Metadata DB: 100M posts/day × ~1 KB = 100 GB/day metadata → sharded.
```

**Takeaway:** the feed reuses #9's fan-out. The new scale driver is **55 PB/year of media** and **60 GB/s of image bandwidth** → the answer is **S3 + aggressive CDN + multi-resolution processing**.

### 3. API design

```
POST /v1/media/upload/init   { type, size } → { uploadId, presignedUrl }  ← direct-to-S3
POST /v1/posts               { mediaIds:[...], caption } → 201 { postId }
GET  /v1/feed?cursor=...                                → posts (same as Twitter feed)
GET  /v1/users/{id}/posts?cursor=...                    → profile grid
POST /v1/posts/{id}/like  |  POST /v1/posts/{id}/comments
```

### 4. Data model

```
TABLE posts
  post_id PK (Snowflake), author_id, caption, media_ids[], created_at, like_count, comment_count
  SHARD KEY post_id;  INDEX (author_id, created_at) for the profile grid

TABLE media
  media_id PK, post_id, owner_id, type, s3_keys{thumb,medium,full}, status(processing/ready)

TABLE follows / home_timeline / tweet-content-cache   ← identical to #9 (reuse)

OBJECT STORE (S3): original + generated variants (thumbnail 150px, medium 600px, full 1080px)
```

### 5. High-level architecture

```
  Client ──init──▶ App Server ──presigned──▶  ┌──────────┐
  Client ──PUT bytes────────────────────────▶ │   S3      │ (original)
                                               └────┬─────┘
                              S3 event ────────────▶│
                                                    ▼
                                        ┌────────────────────────┐
                                        │ Media Processing Workers │ resize → thumb/medium/full
                                        │ (queue-driven)           │ transcode video, strip EXIF
                                        └───────────┬─────────────┘
                                                    ▼  write variants back to S3, mark media ready
   POST /posts ──▶ Post Service ─▶ posts DB ─▶ Fan-out (Kafka) ─▶ home_timeline:{follower} (Redis)
                                                    │
   GET /feed ──▶ Timeline Service (push/pull hybrid, #9) ─▶ hydrate post metadata
                                                    ▼
                                        ┌────────────────────────┐
                                        │  CDN  ◀── S3 (variants)  │  serves images/video globally
                                        └────────────────────────┘
```

### 6. Deep dive

**(A) Media upload + processing pipeline.** Never proxy bytes through your app servers. Client gets a **presigned S3 URL** and uploads directly. An **S3 event** (or a queue message) triggers **processing workers** that generate multiple resolutions (thumbnail/medium/full), transcode video to adaptive formats (HLS/DASH), strip EXIF/geo, run safety checks, then mark `media.status = ready`. The post can be created with `status=processing` and "go live" once media is ready (or block the post until ready — product choice).

**(B) Serving via CDN.** The feed references S3 keys; images are served from a **CDN** (CloudFront/Akamai) that edge-caches them globally → a viral photo is served from edge PoPs near users, not from origin. This is what makes 60 GB/s affordable and fast worldwide (avoids the 150 ms cross-ocean RTT). Pick the right variant per device (thumbnail in the grid, full in detail view) to save bandwidth.

**(C) Feed = #9's hybrid fan-out.** Identical: push post IDs into followers' timelines for normal users; pull for celebrities; merge by `post_id` (Snowflake, time-ordered); hydrate metadata; CDN serves the media. Don't re-derive it — say "same as the Twitter feed."

**(D) Stories (ephemeral).** Stored with a 24h TTL (Redis TTL or a DynamoDB TTL). A separate, lighter timeline; auto-expire rather than fan-out/delete.

### 7. Data flow

**Upload + post:** init → presigned S3 PUT (client uploads directly) → S3 event → processing workers generate variants + mark ready → `POST /posts` writes metadata → fan-out to followers' timelines.
**View feed:** hybrid timeline assembly (#9) → hydrate post metadata → client requests image variants from the **CDN**.

### 8. Scaling

- **Media (55 PB/yr):** S3 (infinite) + lifecycle policies (move cold media to cheaper storage tiers). CDN offloads ~95%+ of read bandwidth.
- **Feed:** scales exactly like #9 (more fan-out workers, bigger timeline cache, celebrity hybrid).
- **Processing:** queue-driven workers autoscale with upload volume; a backlog just delays "go live," it doesn't drop uploads.
- **Hot post (viral):** CDN absorbs it; metadata + counters cached; `like_count` updated via async aggregation (not a hot row update — see #1/#23).

### 9. Trade-offs

- **Pre-generate variants vs on-the-fly resize:** pre-generating (at upload) costs storage but makes reads cheap/cacheable; on-the-fly (an image proxy at the CDN edge) saves storage but adds read latency/CPU. Instagram pre-generates the common sizes.
- **Same fan-out trade-offs as #9** (push/pull hybrid, store IDs not content).
- **CAP:** AP for feed/media; counters eventually consistent.

### 10. Follow-up questions

**Q: How do you handle a photo going viral (millions of views)?**
A: The CDN edge-caches the image variants, so origin S3 sees almost none of the traffic. The post metadata is cached; the like counter is aggregated asynchronously rather than hammering one DB row.

**Q: Why generate multiple resolutions at upload?**
A: So reads are cheap and CDN-cacheable — serve a 150px thumbnail in the grid and a 1080px full image in detail view. On-the-fly resizing would add latency and CPU to every read; pre-generating amortizes it once per upload.

**Q: How is this different from Twitter?**
A: The feed/fan-out is identical. The difference is the media pipeline: direct-to-S3 presigned upload, async multi-resolution processing, and CDN delivery of large binaries — Twitter's payload is tiny text, Instagram's is heavy media.

**Q: How do Stories expire?**
A: Store them with a 24h TTL (Redis/DynamoDB TTL auto-expiry). No fan-out cleanup needed — they vanish automatically; the stories timeline simply stops returning expired entries.

---

## 11 — WhatsApp / Chat System

**Asked at:** Meta (WhatsApp/Messenger), Slack, Discord, Telegram, Hike, Microsoft (Teams)  **Difficulty:** 🟡 Medium  **Level:** SDE-1/2

The defining real-time system. The core challenges: **persistent connections at massive scale** (how do you hold 1B simultaneous WebSocket connections?), **message delivery + ordering + dedup**, **online/last-seen presence**, and **offline delivery**. Be very thorough — this is a frequent SDE-2 question.

### 1. Requirements

**Functional**
- 1:1 messaging, real-time delivery.
- Group chats (up to ~256 members).
- **Delivery receipts:** sent (one tick), delivered (two ticks), read (blue ticks).
- Online/last-seen **presence**.
- Offline message delivery (recipient gets messages on reconnect).
- Media messages (images/video/voice).
- Optional: end-to-end encryption.

**Non-functional**
- **Latency:** message delivery <100 ms when both online — it must *feel* instant.
- **Availability:** 99.99%.
- **Durability:** never lose an undelivered message (must survive recipient being offline for days).
- **Consistency:** per-conversation message **ordering** must be preserved; delivery exactly-once perceived (no dupes).
- **Scale:** 2B users, 1B concurrent connections, 100B messages/day.

### 2. Back-of-envelope estimation

```
100B messages/day → 100B / 86,400 ≈ 1.16M messages/s avg, ~3M/s peak.

Concurrent connections: ~1B users online → need to hold 1B persistent WebSocket/TCP connections.
   One connection ≈ a few KB of memory + a socket FD. A tuned box holds ~500K–1M connections.
   1B / ~500K per box ≈ ~2,000 connection-gateway servers (the "chat servers").

Storage: messages are mostly transient (delivered then can be pruned), but assume 30-day retention
   for undelivered/history: avg message ~100 B.
   100B/day × 100 B = 10 TB/day → ~300 TB for 30 days → sharded NoSQL.

Presence updates: every online user heartbeats every ~30s and changes status on connect/disconnect.
   1B users × (1/30) ≈ 33M presence ops/s — huge; must be cheap (in-memory, sampled).
```

**Takeaway:** the two dominating problems are **(1) holding 1B persistent connections** (→ a fleet of stateful connection gateways + a way to route a message to the gateway holding the recipient's connection) and **(2) cheap presence** (33M ops/s → can't hit a DB; keep in memory, possibly sample/coalesce).

### 3. API design

Chat is **WebSocket-based** (persistent, bidirectional), not request/response REST.

```
WebSocket frames (client ⇄ chat server):
  → SEND      { tempId, toUserId|groupId, type:text, body, ts }
  ← ACK       { tempId, messageId, serverTs }          (server persisted it → one tick)
  ← MESSAGE   { messageId, fromUserId, body, ts }       (incoming message push)
  → DELIVERED { messageId }   → relayed to sender (two ticks)
  → READ      { messageId }   → relayed to sender (blue ticks)
  ← PRESENCE  { userId, status:online|offline, lastSeen }

REST (out of band):
  POST /media/upload  (presigned S3, like #8/#10)
  GET  /conversations/{id}/messages?before=<cursor>   (history/pagination)
```

### 4. Data model

```
TABLE messages                                  (sharded by conversation/chat_id)
  message_id (Snowflake — gives per-shard time ordering),
  chat_id, sender_id, body (or media ref), created_at, type
  SHARD KEY: chat_id        — all messages of a conversation colocated & ordered
  PK (chat_id, message_id)

TABLE inbox / undelivered                        (per-recipient queue for offline delivery)
  user_id, message_id, chat_id, delivered(bool)
  SHARD KEY: user_id        — "what hasn't this user received yet?"

TABLE group_members
  group_id, user_id   — fan-out targets for group messages

REDIS  presence:{userId}        → {status, lastSeen}  (TTL'd heartbeat)
REDIS  session:{userId}         → which chat-gateway server holds this user's connection
```

### 5. High-level architecture

```
   User A (online) ═══WebSocket═══╗
                                  ▼
                         ┌──────────────────┐        ┌─────────────────────────┐
                         │ Chat Gateway #17  │───────▶│ Session Registry (Redis) │
                         │ (holds A's conn)  │        │ userId → gateway server   │
                         └────────┬─────────┘         └─────────────────────────┘
                                  │ 1. persist msg            ▲ lookup B's gateway
                                  ▼                           │
                         ┌──────────────────┐                │
                         │  Message Service  │────────────────┘
                         │  + Kafka (durable │  2. find B's gateway via registry
                         │   message log)    │  3. route message to that gateway
                         └────────┬─────────┘
                                  ▼ persist
                  ┌──────────────────────────────┐
                  │ messages DB (sharded chat_id) │   + inbox DB (offline delivery)
                  └──────────────────────────────┘
                                  │ 4. push to B's gateway
                                  ▼
                         ┌──────────────────┐
                         │ Chat Gateway #42  │═══WebSocket═══▶ User B (online)
                         │ (holds B's conn)  │                (if B offline → stays in inbox,
                         └──────────────────┘                 delivered on B's reconnect)
   Presence service: gateways report connect/disconnect/heartbeat → Redis presence:{uid}
   Media: uploaded to S3 (presigned), message carries a media ref + thumbnail.
```

### 6. Deep dive

**(A) Holding 1B persistent connections — the connection gateway.** Clients open a **WebSocket** (or long-lived TCP) to a **chat gateway** server and keep it open. Each gateway holds ~500K–1M connections (tune kernel: file descriptors, `epoll`, memory per socket). The fleet is ~2,000 gateways behind an L4 load balancer that does **connection-aware routing** (sticky — a user stays on one gateway for the connection's life). The critical piece is a **session registry** (`userId → gatewayId`, in Redis) so the system can find *which gateway holds a given recipient's connection* to deliver a message. Gateways are **stateful** (they own live sockets) — a different scaling discipline than stateless app servers.

**(B) Message send → deliver flow (1:1).**
1. A sends over its WebSocket to Gateway-17.
2. Gateway-17 → Message Service: persist the message (assign a Snowflake `message_id` → ordering), write to `messages` DB and B's `inbox`. ACK back to A (one tick).
3. Look up B in the session registry. **B online** → registry says "Gateway-42" → route the message to Gateway-42 → push down B's WebSocket → B's client sends `DELIVERED` → relayed to A (two ticks). **B offline** → message stays in B's `inbox`; delivered when B reconnects.
4. B reads it → `READ` → relayed to A (blue ticks).

**(C) Ordering, dedup, and exactly-once feel.** Per-conversation ordering: assign `message_id` from a per-chat monotonic source (or Snowflake, sorting by id). Clients send a `tempId`; the server returns the canonical `messageId` in the ACK so the client can dedup retries (it resent the same `tempId` after a flaky network → server recognizes and returns the existing id). Delivery is at-least-once on the wire (retries on missing ACK), made **effectively-once** by the client deduping on `messageId`.

**(D) Offline delivery.** The recipient's `inbox` (sharded by `user_id`) holds undelivered message refs. On reconnect, the client syncs: "give me everything since my last `message_id`." Messages are pushed, marked delivered, and pruned. This is why messages must be durably persisted, not just relayed in memory.

**(E) Presence (online/last-seen) — cheap by design.** 33M ops/s can't touch a DB. Keep `presence:{userId}` in Redis with a **TTL** refreshed by the gateway's heartbeat; if the key expires (no heartbeat), the user is offline and `lastSeen` is frozen. Don't broadcast every presence change to everyone (that's O(N²)); push presence only to **active conversation peers** / those currently viewing the chat, and **coalesce** rapid flips. Presence is best-effort (eventually consistent) — a slightly stale "last seen" is acceptable.

**(F) Group messages.** A group message fans out to all members: persist once (keyed by group `chat_id`), then for each member look up their gateway (online → push) or inbox (offline → queue). Cap group size (~256) keeps fan-out bounded. For huge broadcast groups, treat like the celebrity pull model (#9).

### 7. Data flow

Covered in (B)/(D) above. Summary: **persist first (durable), then deliver** — never deliver-then-persist, or a crash loses the message. ACK on persist (one tick), route via session registry (online) or queue in inbox (offline), relay DELIVERED/READ receipts back.

### 8. Scaling

- **Connections:** add gateways (each +~500K conns); L4 LB spreads new connections; session registry scales as a Redis cluster.
- **Messages (3M/s peak):** Kafka as the durable message backbone partitioned by `chat_id`; shard `messages`/`inbox` DBs by `chat_id`/`user_id`. NoSQL (Cassandra/HBase) fits the append-heavy, ordered-by-id, point-read pattern.
- **Presence (33M ops/s):** in-memory Redis with TTL heartbeats + targeted (not global) presence push + coalescing. Don't persist presence to disk.
- **Geo:** place gateways in regions near users (cut RTT); route cross-region messages through a backbone. Recipient's gateway may be in another region → registry stores region too.
- **Bottlenecks:** (1) gateway connection limits → more boxes + kernel tuning; (2) presence fan-out → targeted + coalesced; (3) hot group → bounded size or pull model; (4) finding the recipient's gateway → fast registry (Redis).

### 9. Trade-offs

- **Stateful gateways vs stateless app tier:** chat *requires* stateful connection servers (they own live sockets) — harder to deploy/rebalance (a gateway restart drops its connections → clients reconnect to another). Trade simplicity for real-time push.
- **Push vs poll for delivery:** push (WebSocket) gives <100 ms delivery vs polling's latency/overhead. Worth the connection cost.
- **CAP:** message persistence leans **CP** (don't lose/duplicate/reorder messages — ack on durable persist); presence is **AP** (best-effort, stale OK). Different subsystems, different choices.
- **SQL vs NoSQL:** messages = NoSQL (Cassandra) — append-heavy, ordered by id, sharded by chat, point reads. Relational joins aren't needed.

### 10. Follow-up questions

**Q: How do you deliver a message to a user who is offline?**
A: Persist it durably and queue a reference in the recipient's `inbox` (sharded by user_id). On reconnect, the client syncs everything since its last seen `message_id`; messages are pushed, marked delivered, and pruned. Persistence is mandatory precisely so offline users don't lose messages.

**Q: How does the system know which server holds the recipient's connection?**
A: A session registry (`userId → gatewayId`, in Redis), updated on connect/disconnect. To deliver, the message service looks up the recipient's gateway and routes the push there. If absent → user is offline → inbox.

**Q: How do you guarantee message ordering?**
A: Assign each message a monotonic id per conversation (Snowflake or a per-chat sequence) and store/sort by it. Clients render by id, so even out-of-order arrival is reordered correctly.

**Q: How do you avoid duplicate messages on retry?**
A: At-least-once on the wire + client-side dedup. Client attaches a `tempId`; server returns the canonical `messageId` in the ACK. A retried send with the same `tempId` maps to the same `messageId`, so the client (and server) dedup it. Net effect: exactly-once as perceived.

**Q: How do you make presence cheap at 33M ops/s?**
A: In-memory Redis keys with a heartbeat-refreshed TTL (expiry ⇒ offline). Don't write to disk, don't broadcast every flip globally — push presence only to active conversation peers and coalesce rapid changes. Presence is best-effort, not strongly consistent.

**Q: How would you add end-to-end encryption?**
A: Encrypt on the client (Signal protocol — per-conversation symmetric keys exchanged via asymmetric key agreement). The server only relays ciphertext and never holds plaintext or keys. Server-side search/processing of message content becomes impossible — a deliberate trade-off for privacy.

**Q: What happens when a chat gateway crashes?**
A: It drops ~500K live connections; those clients detect the dropped socket and reconnect (LB routes them to a healthy gateway, session registry updates). Because messages are persisted (not just in gateway memory) and undelivered ones sit in inboxes, nothing is lost — clients resync on reconnect.

---

## 12 — YouTube / Video Streaming

**Asked at:** Google (YouTube), Netflix, Meta, Amazon (Prime Video), Hotstar  **Difficulty:** 🟡 Medium  **Level:** SDE-1/2

The two halves: a massive **upload + transcode pipeline** and **adaptive streaming delivery at planetary scale via CDN**. The metadata side is small; the bytes side is everything.

### 1. Requirements

**Functional**
- Upload videos (up to GBs/hours).
- Stream/watch with **adaptive bitrate** (auto-adjust quality to bandwidth).
- Search, recommendations (recommendations out of scope; mention).
- View counts, likes, comments.
- Resolutions from 144p to 4K; multiple codecs/devices.

**Non-functional**
- **Latency:** video should **start playing** in <2 s; no rebuffering.
- **Availability:** 99.99%.
- **Durability:** never lose an uploaded video.
- **Read-heavy:** watch:upload is astronomically skewed (millions watch, few upload).
- **Scale:** 2B users, 500 hours of video uploaded/minute, 1B watch-hours/day.

### 2. Back-of-envelope estimation

```
Uploads: 500 hours/minute = 500 × 60 min of video /min. As files: say 5M uploads/day
   → 5M/86,400 ≈ 58 uploads/s. Each triggers heavy transcode (CPU-bound, the real cost).

Storage: a 10-min 1080p video ≈ ~500 MB original; transcoding to ~6 renditions
   (144p..4K) ≈ ~2–3× the original in total variants.
   500 hrs/min × 60 × 24 = 720K hrs/day. At ~1 GB/hr (avg across renditions) ≈ ~700 TB/day raw,
   ~2 PB/day with all renditions → exabytes over years → S3/Blobstore + tiering.

Watch bandwidth: 1B watch-hours/day. At ~3 Mbps avg (mixed quality):
   1B hrs × 3,600 s × 3 Mbps / 86,400 s ≈ enormous → ~hundreds of Tbps globally.
   This MUST be served by CDN/edge caches; origin would never survive it.
```

**Takeaway:** two engines — a **transcoding pipeline** (CPU-heavy, async, queue-driven) and **CDN delivery** (the only way to serve hundreds of Tbps). Metadata is comparatively trivial.

### 3. API design

```
POST /v1/videos/upload/init  { title, size } → { uploadId, presignedUrls[] }  (multipart→S3)
POST /v1/videos/{id}/complete                  → kicks off transcoding
GET  /v1/videos/{id}                           → metadata + manifest URL (HLS/DASH .m3u8/.mpd)
GET  /watch?v={id}                             → player loads manifest → fetches segments from CDN
POST /v1/videos/{id}/view                      → async view-count increment (#23 style)
```

### 4. Data model

```
TABLE videos
  video_id PK, uploader_id, title, description, duration, status(uploading/transcoding/ready),
  thumbnail_url, manifest_url, created_at, view_count
  INDEX (uploader_id, created_at)

TABLE video_renditions
  video_id, resolution(144p..4K), codec, s3_prefix (segmented files), bitrate
  PK (video_id, resolution, codec)

OBJECT STORE: original upload + transcoded renditions, each chopped into 2–10s SEGMENTS
   + a manifest (.m3u8 HLS / .mpd DASH) listing segments per bitrate.
```

### 5. High-level architecture

```
  Uploader ──multipart presigned──▶ ┌──────┐
                                     │  S3   │ (original)
                                     └──┬───┘
                          upload event  │
                                        ▼
                         ┌───────────────────────────────────┐
                         │ Transcoding Pipeline (queue-driven) │
                         │  split → transcode to N renditions  │  (massively parallel,
                         │  → segment (2–10s) → generate HLS/   │   CPU/GPU farm)
                         │  DASH manifest → write to S3         │
                         └──────────────────┬──────────────────┘
                                            ▼ mark video "ready"
                  ┌──────────────────────────────────────────────┐
   Viewer ──────▶ │ Metadata Service ─▶ returns manifest URL       │
   GET /watch     └──────────────────────────────────────────────┘
        │                              │
        │ player fetches manifest, then segments adaptively
        ▼                              ▼
   ┌─────────────────────────────────────────────────────────┐
   │   CDN (edge PoPs)  ◀── origin S3   serves video segments  │  (95%+ of bytes from edge)
   └─────────────────────────────────────────────────────────┘
```

### 6. Deep dive

**(A) Transcoding pipeline.** A raw upload is useless for streaming — you must transcode it into multiple **renditions** (144p…4K), in multiple codecs (H.264 for compatibility, VP9/AV1 for efficiency), each **segmented** into 2–10 s chunks, plus a **manifest** (HLS `.m3u8` / DASH `.mpd`) that lists the segments at each bitrate. This is CPU/GPU-heavy and embarrassingly parallel: split the source into chunks and transcode chunks in parallel across a worker farm (a DAG of jobs orchestrated by a pipeline). It's async — the video is "processing" until renditions are ready. Failure of one chunk → retry just that chunk.

**(B) Adaptive Bitrate Streaming (ABR).** The player downloads the manifest, which lists the same content at multiple bitrates. The **player** measures available bandwidth and buffer health and requests the next 2–10 s **segment** at the appropriate quality — dropping to 480p on a weak connection, climbing to 4K on a fast one, switching seamlessly at segment boundaries. This is why videos start fast (grab a low-bitrate first segment) and don't stall. The server just serves static segments; intelligence lives in the client + manifest.

**(C) CDN delivery — the only way to serve the bandwidth.** Segments are static files → perfectly CDN-cacheable. Edge PoPs near users serve the bytes; popular videos live hot at the edge (origin S3 sees a trickle). YouTube/Netflix even place caching appliances *inside ISPs* (Open Connect). This both makes the bandwidth affordable and slashes start latency (segments come from a nearby PoP, not across an ocean). Pre-position popular content to edges proactively.

**(D) Direct, resumable, multipart upload.** Big files upload directly to S3 via presigned **multipart** URLs (upload parts in parallel, resume failed parts) — never through your app servers.

### 7. Data flow

**Upload:** init → multipart presigned PUTs to S3 (resumable) → complete → enqueue transcode job → pipeline splits/transcodes/segments/manifests in parallel → writes renditions to S3 → mark `ready` → notify uploader.
**Watch:** `GET /watch?v=` → metadata service returns manifest URL → player fetches manifest → player requests segments from the **CDN**, adapting bitrate to bandwidth → view event fired async (counter aggregation, #23).

### 8. Scaling

- **Storage (exabytes):** S3/Blobstore + tiering (hot popular content fast; cold long-tail to cheaper/archival storage). Delete or down-rendition unwatched long-tail.
- **Bandwidth (Tbps):** CDN + ISP-embedded caches; pre-warm popular content. Origin only serves cache misses.
- **Transcoding:** autoscaling worker farm; parallelize per-chunk; prioritize popular/creator uploads.
- **View counts (a viral video):** async aggregation via a stream processor (#23), never a hot-row UPDATE.
- **Bottlenecks:** (1) bandwidth → CDN; (2) transcode CPU → parallel chunked farm; (3) start latency → low-bitrate first segment + nearby edge; (4) storage cost → tiering.

### 9. Trade-offs

- **Pre-transcode all renditions vs on-demand:** pre-transcoding the common renditions makes playback instant/cacheable but costs storage/compute up front; rare resolutions can be transcoded lazily. Mostly pre-transcode.
- **HLS vs DASH:** HLS (Apple ecosystem, ubiquitous) vs DASH (codec-agnostic standard). Often serve both. Both are segment+manifest ABR.
- **CAP:** AP for watch (serve cached, eventually consistent metadata/counters); the upload/transcode side is a durable async pipeline.
- **SQL vs NoSQL:** metadata is small and relational-friendly (SQL fine); the bytes are object storage. Counts via stream aggregation.

### 10. Follow-up questions

**Q: How does video start playing so fast?**
A: Adaptive bitrate — the player grabs a low-bitrate first segment from a nearby CDN edge (small, fast) to start playback within ~2 s, then ramps quality up as it measures bandwidth and fills the buffer.

**Q: Why segment videos into chunks?**
A: Segments enable (1) adaptive bitrate switching at segment boundaries, (2) CDN caching of static chunks, (3) parallel transcoding, and (4) resumable/seekable playback. The manifest ties segments+bitrates together.

**Q: How do you serve the enormous watch bandwidth?**
A: CDN edge caches (and ISP-embedded caches like Netflix Open Connect). Segments are static and highly cacheable; popular content is served from PoPs near users, so origin handles only a sliver. Origin alone could never push hundreds of Tbps.

**Q: How do you handle a 4-hour 4K upload reliably?**
A: Multipart resumable upload directly to S3 (upload parts in parallel, retry only failed parts). Transcoding then splits the source and transcodes chunks in parallel across a worker farm, retrying individual failed chunks.

**Q: How do view counts work without melting a DB row?**
A: Fire a view event to a stream (Kafka), aggregate counts in a stream processor, and periodically materialize the total (see Ad Click Aggregator, #23). Never `UPDATE views = views+1` on a hot row for a viral video.

---

## 13 — Uber / Ride Sharing

**Asked at:** Uber, Lyft, Ola, Grab, DoorDash, Swiggy, Zomato (delivery dispatch)  **Difficulty:** 🟡 Medium (🔴 to do dispatch + geo well)  **Level:** SDE-2

The flagship geo problem. Two hard parts: **efficiently finding nearby drivers** (spatial indexing — geohash / quadtree / S2) over millions of constantly-moving drivers, and **matching/dispatch** (assign the right driver to a rider without double-booking). Be very thorough on the spatial index.

### 1. Requirements

**Functional**
- Drivers continuously report GPS location.
- Rider requests a ride from point A → B.
- System finds **nearby available drivers** and matches one.
- Real-time trip tracking (rider sees driver moving).
- Fare calculation, ETA, surge pricing.
- Trip lifecycle: requested → matched → en route → in trip → completed.

**Non-functional**
- **Latency:** find-nearby + match in <2–3 s (riders won't wait).
- **Availability:** 99.99% — outage = no rides = lost revenue.
- **Consistency:** a driver must be matched to **exactly one** rider (no double-booking — strong consistency on the match).
- **Scale:** 10M drivers, 100M riders, 1M concurrent rides, drivers ping location every 4 s.

### 2. Back-of-envelope estimation

```
Drivers: 10M, ~3M online at peak. Each pings location every 4 s:
   location updates = 3M / 4 ≈ 750,000 writes/s  ← the dominant write load.
   (Naively writing each to a DB is impossible — needs in-memory geo-index.)

Ride requests: 1M concurrent rides, avg ride 20 min → throughput
   ≈ matches: say 5M rides/day → 5M/86,400 ≈ 58 matches/s avg, ~200/s peak.
   (Matching is low QPS but latency-critical and must be exactly-once.)

Storage:
   location (ephemeral, in-memory) — not durably stored at full resolution.
   trips: 5M/day × ~1 KB = 5 GB/day → 1.8 TB/yr → sharded SQL/NoSQL.

Geo-index memory: 3M online drivers × ~100 B (id, lat, lng, status) ≈ 300 MB → in-memory.
```

**Takeaway:** **750K location writes/s** dominate → location lives in an **in-memory spatial index**, updated constantly, *not* a durable DB. Matching is rare but must be **strongly consistent** (one driver, one rider). Two very different subsystems.

### 3. API design

```
POST /v1/drivers/location   { driverId, lat, lng, status }   → 200   (4s heartbeat)
POST /v1/rides/request      { riderId, pickup:{lat,lng}, dropoff:{lat,lng} }
   → { rideId, status:"matching" }   then async push when matched
GET  /v1/rides/{id}                  → status, driver location, ETA
POST /v1/rides/{id}/accept           (driver accepts)
WebSocket: live driver-location → rider; ride-offer → driver.
```

### 4. Data model

```
IN-MEMORY GEO-INDEX (Redis GEO / custom)         ← the hot location store
   driver location keyed by geohash cell:  geo:{cell} → {driverId: (lat,lng,status)}
   (Redis: GEOADD drivers <lng> <lat> <driverId>;  GEOSEARCH ... BYRADIUS)

TABLE trips                                       (durable, sharded by trip_id/city)
  trip_id PK, rider_id, driver_id, status, pickup, dropoff, fare, requested_at, ...
  SHARD KEY: city_id or hash(trip_id)

TABLE drivers / riders                            (profile, vehicle, rating — relational)

REDIS  driver_status:{driverId} → available|on_trip   (for atomic match)
```

### 5. The spatial index — the core deep dive

**The problem:** given a rider's location, find available drivers within ~3 km, fast, while 3M drivers move and re-report every 4 s. A naive `WHERE distance(driver, rider) < 3km` scans every driver → O(N) per query, impossible. You need a **spatial index** that buckets the world into cells so you only examine nearby cells.

**Geohash (most common interview answer).** Recursively divide the world into a grid; encode each cell as a base-32 string. **Longer prefix = smaller cell = more precision.** Nearby points share a geohash prefix.

```
Geohash precision:
   4 chars ≈ 39 km × 19 km cell
   5 chars ≈ 4.9 km × 4.9 km
   6 chars ≈ 1.2 km × 0.6 km
   7 chars ≈ 153 m × 153 m

To find drivers near a rider:
   1. Compute the rider's geohash at the precision matching the search radius (~6 chars for ~1km).
   2. Look up drivers in that cell + the 8 NEIGHBORING cells (a point near a cell edge has
      close drivers in the adjacent cell — must check neighbors!).
   3. Compute exact distance for candidates, filter to radius, sort by distance/ETA.
```

- *Pro:* simple, prefix-based, works great with a KV store (`geo:{6-char-hash} → set of driverIds`); Redis has it built-in (`GEOADD`/`GEOSEARCH`).
- *Con:* fixed grid → dense cities have overloaded cells, empty rural cells are wasteful; edge cases at cell boundaries (mitigated by checking neighbors).

**Quadtree (the other classic answer).** A tree that recursively splits a region into 4 quadrants **only where density is high** — a cell splits when it exceeds a capacity (e.g. 100 drivers). Dense downtown = deep subtree (fine cells); empty desert = one shallow cell.
- *Pro:* adapts to density (no overloaded cells); efficient range queries (descend to the rider's leaf, check siblings).
- *Con:* must rebalance as drivers move (a moving driver may cross cell boundaries → update the tree); more complex than geohash.

**S2 (Google) / H3 (Uber's actual choice).** Hierarchical cell systems mapping the sphere to cells (S2 via a Hilbert curve, H3 via hexagons). H3's hexagons have uniform neighbor distance (no corner/edge asymmetry like squares) — Uber uses H3 in production. Mention it as the "what they really use."

**Recommendation:** geohash (or Redis GEO) for an interview — simplest correct answer; mention quadtree for adaptive density and H3 as production reality.

### 6. Deep dive — matching / dispatch (exactly-once)

Finding candidates is half the job; **assigning** one is the other half, and it needs **strong consistency** — two riders must never get the same driver.
1. Rider requests → find nearby available drivers via the geo-index (above).
2. Rank candidates (distance/ETA, driver rating, acceptance likelihood).
3. **Offer** the ride to the top driver. To avoid double-booking, **atomically reserve** the driver: `SET driver_status:{id} = on_trip IF available` (a compare-and-set / Redis `SET NX`, or a DB row lock). Only one rider wins the CAS.
4. Driver accepts within a timeout → confirm trip. Driver declines / times out → release the reservation (`status=available`) and offer the next candidate.
5. Persist the trip (durable) and switch rider+driver to a live WebSocket channel for tracking.

This is the **CP island** in an otherwise AP system: the match itself must be linearizable (one driver ⇒ one rider).

### 7. Architecture & data flow

```
  Driver app ──loc every 4s──▶ ┌──────────────────┐
                               │ Location Service  │──▶ In-memory Geo-Index (Redis GEO / quadtree)
                               └──────────────────┘     (750K writes/s, ephemeral)
  Rider app ──request ride──▶  ┌──────────────────┐
                               │ Matching Service  │  1. GEOSEARCH nearby available drivers
                               └────────┬─────────┘  2. rank by ETA/rating
                                        │            3. CAS-reserve top driver (exactly-once)
                                        ▼            4. offer → driver accepts/declines
                               ┌──────────────────┐
                               │ Trip Service ──▶ trips DB (durable, sharded by city)
                               └────────┬─────────┘
                                        ▼ live tracking
                               ┌──────────────────┐
                               │ WebSocket Gateway │  driver location ⇄ rider in real time
                               └──────────────────┘
   Surge/ETA: separate services consuming supply/demand from the geo-index per area.
```

**Flow:** driver pings location (4 s) → updates in-memory geo-index. Rider requests → matching service GEOSEARCHes nearby cells + neighbors → ranks → CAS-reserves the best driver → offers → on accept, persists the trip and opens a live WebSocket channel for both. During the trip, the driver's location streams to the rider via WebSocket.

### 8. Scaling

- **Location writes (750K/s):** keep them in-memory (Redis GEO cluster) sharded **by region/city** — a driver in Mumbai never affects the Delhi index. Drop intermediate pings (only the latest location matters); don't durably persist every ping.
- **Geo-index:** shard by city/region (geographic sharding) — queries are inherently local, so this scales perfectly. Hot city → more shards/replicas for that region.
- **Matching:** low QPS but latency-critical; the CAS reservation is the only contention point and it's per-driver (no global lock).
- **Bottlenecks:** (1) location write volume → in-memory + regional sharding + drop stale pings; (2) dense-city cell overload → quadtree/adaptive cells; (3) double-booking → CAS/lock on driver status; (4) cross-region rare (rides are local).

### 9. Trade-offs

- **Geohash vs quadtree vs H3:** fixed grid (simple, KV-friendly) vs density-adaptive (handles cities) vs hexagonal (uniform neighbors, production). Pick by density skew and complexity budget.
- **In-memory vs durable location:** location is ephemeral and high-volume → in-memory (lose it on crash, drivers re-ping in 4 s, no harm). Trips are durable.
- **CAP:** **mixed** — location/geo-index is **AP** (stale-by-4s is fine); the **match is CP** (exactly-once, linearizable). Articulating this split is the senior signal.
- **SQL vs NoSQL:** trips are relational-ish but sharded by city (SQL or NoSQL both fine); location is a specialized in-memory geo store, not a general DB.

### 10. Follow-up questions

**Q: How do you find nearby drivers efficiently?**
A: A spatial index. Bucket drivers into geohash cells (or a quadtree); to search, compute the rider's cell and examine it **plus its 8 neighbors** (drivers near a cell edge), compute exact distances on the small candidate set, and sort by ETA. This turns an O(N) scan into an O(candidates) lookup.

**Q: Why check neighboring cells?**
A: A rider standing near a cell boundary may have the closest driver just across the line in an adjacent cell. Searching only the rider's own cell would miss them, so you union the rider's cell with its 8 neighbors.

**Q: Geohash vs quadtree?**
A: Geohash is a fixed grid — simple, prefix-based, KV-friendly, but dense cities overload cells. A quadtree splits adaptively by density (deep where crowded, shallow where empty) but must rebalance as drivers move. Uber actually uses H3 hexagons for uniform neighbor geometry.

**Q: How do you prevent two riders from matching the same driver?**
A: Atomically reserve the driver with a compare-and-set on their status (`SET on_trip IF available`, or a row lock). Only one rider's CAS succeeds; the match is the strongly-consistent (CP) part of the system.

**Q: How do you handle 750K location updates/second?**
A: Keep locations in an in-memory geo-index (Redis GEO) sharded by city/region; queries are local so geographic sharding scales linearly. Don't persist every ping — only the latest matters, and a lost index just costs one 4 s re-ping.

**Q: How does surge pricing work?**
A: A separate service computes supply (available drivers) vs demand (open requests) per geo cell from the geo-index, and applies a multiplier when demand outstrips supply in an area. It's a read over the same spatial buckets.

---

## 14 — Yelp / Proximity Service

**Asked at:** Yelp, Google (Places/Maps), Zomato, Foursquare, Swiggy  **Difficulty:** 🟡 Medium  **Level:** SDE-1/2

"Find businesses near me." It's the **read-only, static-data** cousin of Uber (#13): same spatial-indexing core, but the entities (restaurants) **don't move**, so the index is mostly static and read-heavy — which changes everything about how you build it.

### 1. Requirements

**Functional**
- Find businesses (restaurants, shops) within a radius / in a viewport.
- Filter by category, rating, price, open-now.
- View business details, reviews, photos.
- Add/edit businesses and reviews.

**Non-functional**
- **Latency:** search <200 ms.
- **Availability:** 99.9%.
- **Read-heavy:** vastly more searches than edits (businesses rarely change).
- **Consistency:** eventual is fine (a new review/business appearing minutes later is OK).
- **Scale:** 200M businesses, 500M users, high search QPS.

### 2. Back-of-envelope estimation

```
Searches: 500M users × a few searches/day = say 1B searches/day
   → 1B/86,400 ≈ 11,500 searches/s avg, ~50K/s peak.
Writes: businesses + reviews are rare: maybe 1M/day → ~12/s. Negligible vs reads.

Storage: 200M businesses × ~2 KB (name, location, hours, attrs) = 400 GB.
   Reviews: say 1B reviews × 1 KB = 1 TB. Photos → S3 (PB-scale), CDN-served.

Geo-index: 200M businesses, but STATIC → precompute and cache aggressively.
```

**Takeaway:** unlike Uber, the data is **static and read-heavy** → the spatial index can be **precomputed and heavily cached** (no 750K writes/s churn). The whole design optimizes reads.

### 3. API design

```
GET /v1/search?lat=&lng=&radius=2km&category=restaurant&openNow=true&sort=rating
   → [{businessId, name, distance, rating, ...}]
GET /v1/businesses/{id}                         → details + reviews + photos
POST /v1/businesses/{id}/reviews   { rating, text }
```

### 4. Data model

```
TABLE businesses
  business_id PK, name, lat, lng, geohash, category, price, rating, hours, address
  INDEX (geohash)              -- spatial lookup
  INDEX (category, geohash)    -- filtered search
  SHARD KEY: region / geohash-prefix (geographic sharding)

TABLE reviews
  review_id PK, business_id, user_id, rating, text, created_at
  INDEX (business_id, created_at);  SHARD KEY business_id

GEO-INDEX: geohash/quadtree of businesses (static, precomputed, cached).
OBJECT STORE: business photos → S3 + CDN.
```

### 5. Architecture

```
  User ──search──▶ ┌──────────┐    ┌────────────────────────┐
                   │   LB     │───▶│ Search Service          │
                   └──────────┘    │ 1. geohash of query loc │
                                   │ 2. lookup cell+neighbors│
                                   │ 3. filter (cat/rating)  │
                                   │ 4. sort by distance     │
                                   └──────┬─────────┬────────┘
                                          ▼         ▼
                              ┌─────────────────┐  ┌──────────────────────┐
                              │ Geo-index cache  │  │ Search engine (ES)    │  for text/filter
                              │ (Redis, static)  │  │ with geo_distance     │  queries
                              └─────────────────┘  └──────────┬───────────┘
                                          │                    ▼
                                  ┌────────────────────────────────────┐
                                  │ Businesses DB + Reviews DB (sharded) │
                                  └────────────────────────────────────┘
                              Photos → S3 → CDN
```

### 6. Deep dive

**(A) Spatial index (same core as #13, but static).** Bucket businesses into geohash cells (or a quadtree). Search = geohash the query point at the radius-appropriate precision, fetch the cell + 8 neighbors, compute exact distances, filter and sort. Because businesses **don't move**, the index is **precomputed once and cached** — no constant rewriting like Uber's driver pings. This makes a Redis/in-memory geo-index or a precomputed cell→businesses map extremely effective.

**(B) Combining geo with filters — use a search engine.** Real queries are "Italian restaurants, 4+ stars, open now, within 2 km, sorted by rating." That's a **geo query + attribute filters + sort** — exactly what **Elasticsearch** (with `geo_distance`/`geo_bounding_box` + filters) is built for. Index businesses into ES; let it do the spatial + filter + rank in one query. The DB remains the source of truth; ES is the read-optimized search index, kept in sync via CDC.

**(C) Caching.** Static data + read-heavy = cache everything: hot search results (popular areas) cached for seconds–minutes, business details cached, photos on CDN. A search in Times Square at lunch is asked thousands of times — cache it.

### 7. Data flow

**Search:** geohash the query location → fetch candidate businesses from the geo-index/ES for the cell + neighbors → apply filters (category, rating, open-now) → compute exact distances → sort → return (cache the result).
**Add business / review:** write to the DB → CDC updates the ES index + invalidates relevant geo-cache cells. Eventually consistent (appears in search shortly after).

### 8. Scaling

- **Reads (50K/s):** cache heavily + scale ES read replicas; geographic sharding means a query only touches its region's shard/index.
- **Static index:** precompute, replicate widely; rebuilds are batch jobs, not hot-path.
- **Photos:** S3 + CDN (like #10).
- **Hot area:** cache popular-area results; the geo-cell is small and cache-friendly.

### 9. Trade-offs

- **vs Uber:** static read-heavy data ⇒ precomputed, cached index + a search engine for filters; Uber's data is high-churn ⇒ in-memory mutable index. Same spatial primitive, opposite write profile.
- **CAP:** **AP** — eventual consistency is fine (a new business/review showing up minutes later is acceptable); availability of search matters.
- **DB + search engine split:** DB = source of truth (consistent writes); ES = denormalized read index (fast geo+filter+sort), synced via CDC. Classic CQRS-style split.

### 10. Follow-up questions

**Q: How is this different from Uber's proximity?**
A: Same spatial index (geohash/quadtree), but Yelp's entities are static, so the index is precomputed and aggressively cached — no 750K writes/s churn. The design is read-optimized (search engine + caching) rather than write-optimized (in-memory mutable geo store).

**Q: How do you do "Italian, 4+ stars, open now, within 2 km, by rating" in one query?**
A: A search engine like Elasticsearch with `geo_distance` plus attribute filters and a sort — it combines spatial, filtering, and ranking in one indexed query. Keep the DB as source of truth and sync ES via CDC.

**Q: How do you keep search fast under 50K QPS?**
A: Cache hot-area results (popular places are queried repeatedly), geographically shard the index (queries are local), scale ES read replicas, and serve photos from a CDN. Static data caches extremely well.

**Q: How do you handle businesses near a cell boundary?**
A: Same as Uber — search the query's geohash cell plus its 8 neighbors, then filter by exact distance, so a business just across a cell line isn't missed.

---


---

## 15 -- Typeahead / Autocomplete
**Asked at:** Google, Amazon, Twitter  **Difficulty:** 🟡 Medium  **Level:** SDE-1/SDE-2

**1. Requirements**

*Functional:*
- As a user types in a search box, return the top-5 completions within 100 ms.
- Completions are ranked by query frequency (globally) and optionally personalized.
- Support prefix matching (not substring).
- New queries affect suggestions within a few hours (near-real-time index update).

*Non-functional:*
- P99 latency <= 100 ms end-to-end.
- Availability 99.99%.
- Read-heavy: billions of keystrokes/day; writes (new trending queries) are batch.
- Scale: 500 M DAU; autocomplete fires on every keystroke.

**2. Back-of-envelope estimation**

- 500 M DAU x 5 searches/day x 5 keystrokes = 25 events/user/day.
- Autocomplete QPS: 500M x 25 / 86,400 = **145,000 QPS** (read). Peak 3x = ~435,000 QPS.
- Unique prefixes: top 1 M queries x 10 prefix lengths = 10 M entries, ~5 GB in RAM.
- Write pipeline: log ~8.5 B queries/day -> batch aggregate hourly -> patch Redis. Write QPS to suggestion store: negligible.

**3. API design**

```
GET /autocomplete?prefix=sys&limit=5&lang=en
Response: { "prefix": "sys", "suggestions": [
  {"query": "system design", "score": 9821043},
  {"query": "system of a down", "score": 4102881}
]}
```

No write API on the hot path -- frequency updates come from an offline pipeline.

**4. Data model / schema**

Suggestion store (Redis):
```
Key:   prefix:<normalized_prefix>
Value: [("system design", 9821043), ...]  top-5, pre-ranked
TTL:   1 hour (refreshed by batch pipeline)
```

Frequency store (Cassandra):
```
query_frequencies: query_text TEXT PK, count BIGINT, updated_at TIMESTAMP
```

Query log: Kafka -> S3, append-only, never queried on hot path.

**5. High-level architecture**

```
  Client
    | debounce 150 ms
    v
  CDN / Edge Cache (popular prefixes cached, TTL 60 s)
    | miss
    v
  Load Balancer
    v
  Autocomplete Service (stateless)
    | 1. normalize prefix
    | 2. GET prefix:<prefix> from Redis
    | 3. hit  -> return top-5
    | 4. miss -> Trie Service (cold start only)
    v
  Redis Cluster (pre-computed top-5 per prefix, sharded by prefix hash)
    |
    v (miss only)
  Trie Service (in-memory, read-only per AZ)
    | traverse -> rank -> top-5 -> warm Redis
    v
  +------------------------------------------+
  |  Offline Pipeline                        |
  |  queries -> Kafka -> Flink (hourly agg)  |
  |                -> Cassandra (freq store) |
  |                -> Trie Builder           |
  |                   -> writes Redis        |
  +------------------------------------------+
```

**6. Deep dive**

**(A) Trie vs prefix hash table.**
A classic trie requires a DFS subtree traversal per request. At scale this is too slow. The production approach: **pre-compute top-K results for every prefix** and store them as a plain Redis key-value lookup. The trie exists only in the offline pipeline; the hot path is a single Redis GET.

**(B) Ranking by frequency + freshness.**
Score = frequency x recency_decay. Trending queries get a boost. The offline Flink job computes scores hourly; the Trie Builder rewrites top-5 lists into Redis. Suggestions reflect the last hour of trends without any real-time writes on the hot path.

**(C) Client debounce + CDN caching.**
A 150 ms debounce converts 8 keystrokes into 1-3 HTTP requests. Popular prefixes ("the", "how") cached at the CDN edge 60 s -- a large fraction of traffic never reaches origin.

**7. Data flow**

1. User types "sys" -> debounce 150 ms -> GET /autocomplete?prefix=sys.
2. CDN miss -> Autocomplete Service -> GET prefix:sys from Redis -> hit -> return top-5.
3. CDN caches result 60 s.
4. Offline: Kafka receives search query -> Flink hourly aggregate -> Cassandra -> Trie Builder rewrites Redis top-5 for affected prefixes.

**8. Scaling**

- **10x:** Redis read replicas; more CDN PoPs; scale stateless app servers.
- **100x:** Shard Redis by prefix hash (16 shards); regional Redis clusters with async replication.
- **1000x:** Push top-100K prefixes to CDN edge; only long-tail reaches origin.

**9. Trade-offs**

- **Pre-computed vs real-time trie:** Pre-computed is fast (single Redis GET) but stale by up to 1 hour. Real-time trie is fresh but too slow at 435K QPS. Pre-computed wins at Google scale.
- **CAP: AP.** Redis down -> fall back to trie service or empty suggestions. Never block the search box.
- **Personalization:** Re-rank global top-10 on app server using user history (small per-user cache). No per-user prefix tables.

**10. Follow-up questions**

**Q: How do you handle personalization without exploding storage?**
A: Return global top-10 from prefix cache, then re-rank on the app server using the user recent query history fetched from a small per-user profile cache.

**Q: How do you deal with offensive/spam completions?**
A: A denylist (bloom filter) at the app server before returning results. Offline moderation pipeline removes flagged queries from the frequency store before the next Trie Builder run.

**Q: How do you support multiple languages?**
A: Separate prefix namespaces per language: `prefix:en:sys` vs `prefix:ja:word`. Language detected from Accept-Language header or user setting.

**Q: What if a query suddenly trends (breaking news)?**
A: Reduce batch interval to 5-15 min, or run a separate "trending now" Flink stream (5-min window) writing to a small Redis key blended into top-5 at the app server.

---

## 16 -- Web Crawler
**Asked at:** Google, Amazon  **Difficulty:** 🟡 Medium  **Level:** SDE-1/SDE-2

**1. Requirements**

*Functional:*
- Crawl the public web, downloading HTML pages.
- Extract and follow hyperlinks to discover new URLs.
- Respect robots.txt and crawl-delay directives.
- Re-crawl pages periodically; detect and skip duplicate content.
- Store raw HTML for downstream processing (indexing, etc.).

*Non-functional:*
- Scale: index ~5 B pages; crawl 1 B new/updated pages/day.
- Politeness: no more than 1 request per domain per crawl-delay window (1-10 s).
- Deduplication: never store the same content twice.
- Workers can fail and restart without losing progress.
- Throughput: ~12,000 pages/s sustained.

**2. Back-of-envelope estimation**

- 1 B pages/day / 86,400 s = **11,600 pages/s**.
- Avg page 100 KB HTML. Storage/day: 1 B x 100 KB = **100 TB/day** (compressed ~30 TB).
- Visited URL store: 5 B URLs x 100 bytes = **500 GB**.
- Bloom filter: 5 B entries at 1% FP rate = **~6 GB RAM**.

**3. API design**

Internal control APIs:
```
POST /crawl/seed   { "urls": ["https://example.com"] }
GET  /crawl/status -> { "pages_crawled": 1000000000, "frontier_size": 50000000 }
GET  /robots?domain=example.com -> parsed robots.txt rules
```

**4. Data model / schema**

URL Frontier (Kafka, partitioned by domain hash):
```
frontier: url TEXT, priority FLOAT, discovered_at TIMESTAMP, depth INT
```

Visited URL store (bloom filter + Cassandra):
```
visited_urls: url_hash BYTES(16) PK, crawled_at TIMESTAMP, http_status INT, content_hash BYTES(16)
Bloom filter: 5 B entries, ~6 GB RAM, 1% FP rate
```

Raw content: S3 `s3://crawl/{year}/{month}/{day}/{url_hash}.html.gz`

Robots.txt cache (Redis): `Key: robots:{domain}  TTL: 24 h`

**5. High-level architecture**

```
  Seed URLs
      |
      v
  +----------------------------------------------+
  |  URL Frontier Service                        |
  |  Kafka topics, partitioned by domain hash    |
  +-------------------+--------------------------+
                      | dequeue batches
                      v
  +----------------------------------------------+
  |  Crawler Worker Pool (1000s of pods)         |
  |  1. Dequeue URL from Kafka partition         |
  |  2. Check bloom filter -> skip if seen       |
  |  3. Fetch robots.txt (Redis cache)           |
  |  4. Wait for per-domain token bucket         |
  |  5. HTTP GET (timeout 10 s)                  |
  |  6. Hash body -> content dedup check        |
  |  7. Store HTML to S3                         |
  |  8. Parse links -> push to Kafka frontier    |
  |  9. Mark visited in bloom filter + Cassandra |
  +----------------------------------------------+
         |                    |
         v                    v
  +-------------+   +--------------------+
  | Bloom Filter|   | Content Dedup Store|
  | (Redis,     |   | content_hash->S3   |
  |  ~6 GB RAM) |   | (Cassandra)        |
  +-------------+   +--------------------+
         |
         v
  +------------------+
  | Raw HTML (S3)    | -> indexing pipeline
  +------------------+
```

**6. Deep dive**

**(A) BFS frontier queue with politeness.**
Partition the Kafka frontier by domain hash so each partition owns a set of domains. Each crawler worker maintains a **per-domain token bucket** -- it may only send to domain X after crawl-delay elapsed. This serializes per-domain requests without global coordination.

**(B) Duplicate URL detection with bloom filter.**
Check bloom filter before fetching (O(1), ~6 GB RAM). False positives occasionally skip a new URL -- acceptable. URL normalization (lowercase, strip fragments, sort query params) before hashing collapses equivalent URLs. Cassandra is the durable ground truth; the bloom filter is the fast-path gate.

**(C) Content deduplication.**
Different URLs may serve identical content (mirrors, pagination). SHA-256 the body; look up in the content dedup store. If found, store only a pointer to the original S3 key. Cuts storage 20-30%.

**7. Data flow**

1. Seed service enqueues URLs to Kafka, partitioned by hash(domain) % N.
2. Worker dequeues URL -> bloom filter check -> skip if seen.
3. Fetch robots.txt from Redis (or fetch + cache 24 h).
4. Wait for per-domain token bucket.
5. HTTP GET page; on error log and mark visited.
6. Hash body -> content dedup check -> skip storage if duplicate, but mark URL visited.
7. Store HTML to S3; write url_hash -> Cassandra; add to bloom filter.
8. Extract links -> normalize -> push unseen URLs to Kafka frontier.

**8. Scaling**

- **10x:** Add crawler pods (stateless); increase Kafka partitions.
- **100x:** Shard bloom filter across Redis cluster; shard Cassandra by url_hash; per-worker DNS cache.
- **1000x:** Multi-region crawl clusters per TLD; custom priority frontier with compaction; S3 intelligent tiering.

**9. Trade-offs**

- **Bloom filter FP:** ~1% skip rate on new URLs in exchange for O(1) lookup. For archival crawlers, use persistent store only.
- **CAP: AP.** Kafka partition failover causes some URL re-enqueue (at-least-once). Idempotency (bloom + visited store) absorbs duplicates.
- **BFS vs priority:** Production crawlers use priority scoring (estimated PageRank, freshness) to crawl high-value pages first.

**10. Follow-up questions**

**Q: How do you handle JavaScript-rendered pages?**
A: A separate headless-browser worker tier (Puppeteer/Chromium) for URLs flagged as JS-heavy. ~10x slower, so kept separate from the main crawl path.

**Q: How do you avoid losing progress on worker restart?**
A: Cassandra visited store is durable. On restart, rebuild bloom filter from a recent Cassandra snapshot; accept slightly higher re-crawl rate for a few hours.

**Q: How do you re-crawl for freshness?**
A: Track last_crawled_at and change_frequency. A scheduler queries Cassandra for pages where last_crawled_at + ttl < now and re-enqueues with a freshness priority boost.

**Q: How do you detect crawler traps?**
A: Limit crawl depth (max 8 hops), cap URLs per domain, detect incrementing-parameter patterns (regex fingerprinting), and honor robots.txt Disallow.

---

## 17 -- Ticketmaster / Event Booking System
**Asked at:** Amazon, Flipkart, BookMyShow  **Difficulty:** 🟡 Medium  **Level:** SDE-1/SDE-2

**1. Requirements**

*Functional:*
- Users browse events and view available seats on a venue map.
- Users reserve a seat (hold 10 min), then complete purchase.
- Seat is locked exclusively during hold; no double-booking.
- Support flash sales: 100K users simultaneously for one event.
- Admins create events, upload seating charts, set pricing tiers.

*Non-functional:*
- No double-booking ever; consistency on seat state is CP.
- Hold timeout: unreserved seat released after 10 min.
- Browse: ~3K QPS steady; booking spike: 100K concurrent.
- Latency: seat selection < 200 ms; booking confirmation < 2 s.
- Availability 99.99%.

**2. Back-of-envelope estimation**

- 50 M DAU browsing x 5 pages / 86,400 = ~2,900 browse QPS.
- Flash sale: **100,000 concurrent** reservation attempts for 30-60 s.
- Seats: 100K events x 10K seats = 1 B seat records x 200 bytes = **200 GB** (partitioned by event).
- Bookings: 500K/day x 500 bytes = 250 MB/day; years of history ~100 GB.

**3. API design**

```
GET  /events?city=NYC&date=2025-01-01
GET  /events/{eventId}/seats
POST /reservations
  Body:     { "eventId": "e1", "seatId": "s42", "userId": "u99" }
  Response: { "reservationId": "r1", "expiresAt": "2025-01-01T10:10:00Z", "price": 85.00 }
POST /bookings
  Body:     { "reservationId": "r1", "paymentToken": "tok_xxx" }
  Response: { "bookingId": "b1", "confirmationCode": "TKT-9F3A" }
DELETE /reservations/{reservationId}
GET    /bookings/{bookingId}
```

**4. Data model / schema**

```sql
events (event_id UUID PK, name TEXT, venue_id UUID, starts_at TIMESTAMP, status ENUM)

seats (
  seat_id UUID PK, event_id UUID,  -- event_id is the shard key
  section TEXT, row TEXT, number TEXT, price_tier TEXT,
  status ENUM(available, held, booked),
  held_by UUID NULL, held_until TIMESTAMP NULL,
  INDEX: (event_id, status)
)

reservations (
  reservation_id UUID PK, seat_id UUID, event_id UUID, user_id UUID,
  created_at TIMESTAMP, expires_at TIMESTAMP,
  status ENUM(pending, confirmed, released, expired),
  INDEX: (expires_at)
)

bookings (
  booking_id UUID PK, reservation_id UUID, user_id UUID,
  payment_id UUID, confirmed_at TIMESTAMP, amount DECIMAL
)
```

Shard seats + reservations by event_id -- a flash sale for one event hits one shard.

**5. High-level architecture**

```
  Users
      |
      v
  CDN (event pages, venue images, static seat maps)
      |
      v
  Load Balancer (L7)
      |                        |
      v                        v
  Browse Service          Reservation Service
  cache-aside Redis       atomic seat locks
  reads replicas          writes to primary
      |                        |
      v                        v
  Read Replicas (PG)      Primary PostgreSQL (sharded by event_id)
                               |
                          Flash Sale Queue (Redis list per event)
                          absorbs 100K burst -> reservation workers
                          at safe rate (500 req/s)
                               |
                               v
                          Payment Service (external gateway)
                               |
                               v
                          Notification Service (email/SMS)
                               |
                               v
                          TTL Sweeper (cron every 30 s)
                          releases expired holds
```

**6. Deep dive**

**(A) Seat reservation with optimistic locking.**
The core invariant: a seat cannot be double-booked. Implement with a single atomic UPDATE:

```sql
UPDATE seats
SET    status = 'held', held_by = :userId,
       held_until = NOW() + INTERVAL '10 minutes'
WHERE  seat_id = :seatId
  AND  event_id = :eventId
  AND  status = 'available'   -- guard: this is the lock check
RETURNING seat_id;
```

If UPDATE returns 0 rows, the seat was already taken. No SELECT-then-UPDATE (TOCTOU race). Under normal load this works well. Under a flash sale where many users fight for the same seat, the queue below absorbs the spike.

**(B) Flash-sale handling with a virtual waiting room.**
100K simultaneous requests would cause DB lock contention storms. Solution:
1. User clicks "Reserve" -> Reservation Service pushes to a Redis list per event (Redis handles 100K/s easily).
2. Service returns immediately: { "status": "queued", "position": 4821, "estimatedWait": "2 min" }.
3. Reservation Workers dequeue at a controlled rate (e.g., 500/s) and attempt the DB UPDATE.
4. User polls /reservations/status/{requestId} or receives a WebSocket push on success or sell-out.

**(C) Idempotency for payments.**
A payment call may time out -- was the charge made? The booking endpoint requires reservationId as an idempotency key. Before calling the payment gateway, write a payment_attempts record with (idempotency_key=reservationId, status=pending). On repeat calls: if pending -> "in progress"; if succeeded -> return existing bookingId; if failed -> allow retry. Prevents double-charging.

**7. Data flow**

1. Browse /events/{id}/seats -> Browse Service -> Redis cache (seat bitmask, TTL 5 s) -> returns seat map.
2. POST /reservations -> Reservation Service. Flash-sale mode? Push to Redis queue, return queued. Normal? Attempt DB UPDATE.
3. DB UPDATE succeeds (status -> held, held_until = now+10m) -> reservation record created -> return reservationId + price.
4. POST /bookings -> write payment_attempts(pending) -> call Payment Service -> on success: DB txn: seats status=booked, reservations status=confirmed, INSERT booking.
5. TTL Sweeper (every 30 s): UPDATE seats SET status=available WHERE held_until < NOW() AND status=held.

**8. Scaling**

- **10x:** Shard PG by event_id; cache seat maps in Redis; scale stateless services horizontally.
- **100x:** Redis queue for all flash sales; read replicas per shard; CDN for seat-map SVGs.
- **1000x:** Multi-region DB clusters; popular events get dedicated shard clusters.

**9. Trade-offs**

- **CP for seat locking:** Correctness over availability. DB unavailable -> selection fails, not double-books.
- **Optimistic vs pessimistic locking:** Pessimistic (SELECT FOR UPDATE) blocks the row for the entire checkout (minutes) -- terrible for concurrency. Optimistic (condition in UPDATE WHERE) fails fast.
- **Queue vs direct DB:** Queue absorbs burst and gives fair ordering, but adds latency. Direct DB is faster for normal load. Hybrid: direct attempt first, queue only when event is in high-demand mode.

**10. Follow-up questions**

**Q: How do you prevent a user from holding 50 seats?**
A: Per-user-per-event hold limit (e.g., max 4) enforced at the app layer. Store (user_id, event_id) -> active_hold_count in Redis; reject if over limit.

**Q: How do you handle a hold expiring mid-checkout?**
A: expires_at is checked at booking time. If expired, reject with "hold expired." UI shows countdown and redirects to seat selection if it lapses.

**Q: What if payment succeeds but the booking DB write fails?**
A: The payment_attempts record is written first. A reconciliation job (every minute) detects payment_attempts.status=paid with no matching booking record and either finalizes or refunds it.

**Q: How do you serve seat map reads at 50K QPS for a hot event?**
A: Cache the seat map bitmask in Redis per event, refreshed every 3-5 s. Users accept slight staleness on display; the authoritative check is the atomic UPDATE at reservation time.

---
