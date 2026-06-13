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

---

## 18 — Google Docs / Collaborative Editing
**Asked at:** Google, Microsoft  **Difficulty:** 🔴  **Level:** SDE-2/Senior

**1. Requirements**
*Functional:*
- Multiple users can edit the same document simultaneously in real time
- Changes from any user appear on all other users' screens within ~500 ms
- Each user's cursor position and selection is visible to others (presence)
- Full revision history — view any past version, diff between versions, restore
- Offline editing: buffer local changes and sync on reconnect
- Comments and suggestions (track-changes) mode
- Sharing with permission levels: viewer, commenter, editor, owner

*Non-functional:*
- Latency: local keystroke response < 16 ms (must feel instant); remote propagation < 500 ms P99
- Availability: 99.99% (documents are business-critical)
- Consistency: eventual consistency for concurrent edits; conflict-free merge required
- Scale: 1 B documents, 50 M DAU, peak 100 K concurrent collaborative sessions
- Durability: zero data loss — every committed operation persisted to durable log

**2. Back-of-envelope estimation**
- 50 M DAU; avg 3 documents opened/day = 150 M doc-open events/day
- Collaborative sessions: assume 5% of sessions are concurrent multi-user = 7.5 M collaborative sessions/day; peak 100 K concurrent
- Operations (keystrokes): avg active user types 40 ops/min during a 20-min session = 800 ops/session; 50 M * 800 = 40 B ops/day = ~460 K ops/sec peak (10x avg) = 460 K write ops/sec
- Storage per op: ~50 bytes (op type, position, author, timestamp) = 460 K * 50 = 23 MB/sec raw operation log; ~700 GB/day
- Document snapshots: full snapshot every 1000 ops; avg doc 50 KB * 1 B docs = 50 TB snapshot storage total
- Bandwidth: 100 K concurrent sessions * 10 ops/sec broadcast to avg 3 peers = 3 M op-messages/sec; at 100 bytes each = 300 MB/sec outbound

**3. API design**
```
# Open document session (returns WebSocket URL + doc state)
GET /docs/{docId}/session
Response: { wsUrl, docId, snapshotVersion, snapshot, revision }

# WebSocket message: client sends an operation
WS SEND: { type:"op", docId, revision, op: { type:"insert"|"delete"|"retain", pos, chars, author } }

# WebSocket message: server broadcasts transformed operation
WS RECV: { type:"op", op: { ... }, serverRevision, authorId }

# WebSocket message: presence broadcast
WS RECV: { type:"presence", userId, cursor:{ pos, selStart, selEnd }, color }

# HTTP: fetch revision history
GET /docs/{docId}/revisions?from=100&to=200
Response: [{ revision, op, userId, timestamp }]

# HTTP: restore to a revision
POST /docs/{docId}/restore
Body: { targetRevision: 150 }

# HTTP: get snapshot near a revision (for fast load)
GET /docs/{docId}/snapshot?nearRevision=500
Response: { snapshot, snapshotRevision }
```

**4. Data model / schema**
```
documents
  doc_id        UUID PK
  owner_id      UUID FK users
  title         TEXT
  created_at    TIMESTAMPTZ
  updated_at    TIMESTAMPTZ
  current_rev   BIGINT          -- monotonically increasing server revision counter

-- Append-only operation log (sharded by doc_id)
operations
  doc_id        UUID  (shard key)
  revision      BIGINT           -- server-assigned, sequential per doc
  op_data       JSONB            -- { type, pos, chars, authorId }
  author_id     UUID
  client_rev    BIGINT           -- revision client believed they were at
  created_at    TIMESTAMPTZ
  PRIMARY KEY (doc_id, revision)
  INDEX (doc_id, revision)       -- range scan for catchup

-- Periodic snapshots to avoid replaying the full log on every open
snapshots
  doc_id          UUID  (shard key)
  snapshot_rev    BIGINT
  content         TEXT / BYTEA   -- full document state at snapshot_rev
  created_at      TIMESTAMPTZ
  PRIMARY KEY (doc_id, snapshot_rev)

-- Sharing / permissions
permissions
  doc_id      UUID
  user_id     UUID
  role        ENUM(viewer, commenter, editor, owner)
  PRIMARY KEY (doc_id, user_id)
```
Shard/partition key: `doc_id` — all operations for a document are co-located. Use consistent hashing across Cassandra or CockroachDB nodes.

**5. High-level architecture**
```
Clients (Browser / Mobile)
        |  HTTPS for REST  |  WebSocket for real-time ops
        v                  v
   +----------------------------------------------+
   |           Load Balancer                       |
   |  (sticky sessions by docId hash)             |
   +---------------+------------------------------+
                   |
   +---------------v--------------------------------------------------+
   |               Collaboration Service (stateful)                    |
   |  One process owns each active document session                    |
   |  * Receives raw ops from clients over WS                         |
   |  * Runs OT transform engine                                      |
   |  * Broadcasts transformed ops to all clients in session          |
   |  * Maintains in-memory document state + revision counter         |
   +-------+------------------+---------------------------------------+
           | persist ops       | presence & routing
   +-------v----------+  +----v---------------------+
   |  Operation Log   |  |  Redis Pub/Sub            |
   |  (Cassandra /    |  |  channel: doc:{docId}     |
   |   CockroachDB)   |  |  for cross-node fan-out   |
   |  sharded by      |  +---------------------------+
   |  doc_id          |
   +-------+----------+
           | periodic snapshot trigger
   +-------v----------+
   |  Snapshot Store  |
   |  (S3 / GCS)      |
   |  snapshot every  |
   |  1000 ops        |
   +------------------+

   +---------------------------------------------+
   |           Document Metadata Service          |
   |  (permissions, sharing, doc list, REST API)  |
   |  backed by PostgreSQL                        |
   +---------------------------------------------+

   +---------------------------------------------+
   |                CDN (CloudFront)              |
   |  Serves JS bundle, static assets            |
   +---------------------------------------------+
```

**6. Deep dive**

**Operational Transformation (OT) vs CRDTs**

OT transforms operations relative to concurrent operations so that all sites converge to the same state. Example: Alice inserts "X" at position 5; concurrently Bob deletes char at position 3. Bob's delete shifts Alice's insert position. OT transforms Alice's op to insert at position 4 before applying at Bob's replica.

The transform function T(op_a, op_b) -> op_a' satisfies: apply(apply(doc, op_a), T(op_b, op_a)) == apply(apply(doc, op_b), T(op_a, op_b)). This is the "diamond property." Google's Wave algorithm (Jupiter protocol) simplifies this by having a central server: clients only need to transform against the server's history, not each other directly. The server maintains a single revision sequence; every client op is transformed against all server ops since the client's base revision.

CRDTs (Conflict-free Replicated Data Types) — specifically sequence CRDTs like RGA or Logoot — assign every character a unique immutable identifier (site ID + counter). Insertions are always relative to identifiers, never positions, so concurrent inserts never conflict; ordering is deterministic by ID. CRDTs enable true peer-to-peer sync without a central transform server. Downside: tombstones (deleted chars stay as markers) bloat memory; harder to implement correctly; position queries are O(n). Figma uses CRDTs; Google Docs uses OT.

**WebSocket real-time sync architecture**

Each active document session is owned by a single Collaboration Service node (selected by consistent hash on docId). All clients editing the same document connect to the same node — the LB uses sticky routing. The node holds in-memory: current document state, current server revision R, and a queue of unacknowledged client ops. When a client sends op(clientRev, op), the node: (1) fetches all server ops with revision > clientRev from its in-memory buffer, (2) transforms op against those, (3) assigns server revision R+1, (4) writes to Cassandra, (5) broadcasts transformed op to all connected clients. Crash recovery: on node restart, load latest snapshot from S3, replay Cassandra log since snapshot_rev.

**Cursor/presence broadcasting**

Presence (cursor positions) is ephemeral and high-frequency — not persisted. Each client sends a presence heartbeat every 1 s over the same WebSocket. The Collaboration Service holds a cursor map { userId -> {pos, color} } in memory and broadcasts diffs to all peers. If a user disconnects, remove from map after 5 s timeout.

**7. Data flow** — Inserting a character
1. User types "A" at position 47. Browser sends WS message: `{type:"op", clientRev:312, op:{type:"insert", pos:47, chars:"A", authorId:"u1"}}`.
2. Collaboration Service receives op. Current serverRev = 315. It fetches ops 313, 314, 315 from its in-memory buffer.
3. OT engine transforms op against those 3 ops: position adjusts from 47 to 49 after accounting for two earlier inserts from another user.
4. Server assigns revision 316. Writes to Cassandra: `(doc_id, rev=316, op_data, author)`.
5. Server sends ack to sender: `{type:"ack", serverRev:316}`.
6. Server broadcasts to all other clients in session: `{type:"op", op:{type:"insert", pos:49, chars:"A"}, serverRev:316}`.
7. Each receiving client applies the op at position 49 to their local state and advances their known serverRev to 316.
8. Every 1000 revisions, Snapshot Worker serializes full doc state, uploads to S3, writes snapshot record to Cassandra.

**8. Scaling**
- **10x (500 K concurrent sessions):** Horizontal scale of Collaboration Service nodes; consistent hash ensures each doc maps to one node; Redis pub/sub handles cases where clients of same doc are on different nodes (rare with sticky routing).
- **100x (5 M concurrent sessions):** Introduce a dedicated presence pub/sub tier (Redis Cluster); offload snapshot generation to async workers; Cassandra cluster scaled to 20+ nodes with RF=3; operation log partitioned with vnodes.
- **1000x (50 M concurrent sessions):** Geo-distributed deployment; each region handles its own doc sessions with async cross-region replication for durability; edge nodes for WS termination to reduce latency globally.

**9. Trade-offs**
- **OT (centralized) vs CRDT (peer-to-peer):** OT chosen — simpler to reason about correctness in a client-server model; server is the single source of truth for revision ordering; no tombstone bloat. CRDTs suit fully offline-first / P2P scenarios but are operationally harder.
- **Cassandra vs PostgreSQL for op log:** Cassandra chosen — write-heavy append-only workload with high throughput; natural partitioning by doc_id; no need for cross-doc transactions.
- **Eventual vs strong consistency:** Eventual consistency is acceptable for collaborative text — users tolerate ~500 ms propagation delay. Strong consistency would require distributed locking per character, killing throughput.
- **Sticky routing cost:** Sticky LB makes node failure require session migration. Mitigation: detect failure, reconnect clients to new node, replay from Cassandra log. Acceptable trade-off for eliminating cross-node OT complexity.

**10. Follow-up questions**

**Q: How do you handle a client that was offline for 2 hours and reconnects?**
A: Client sends its last known serverRev. Server loads nearest snapshot at or before that rev, then streams all ops from snapshot_rev to current serverRev. Client replays the op log to reconstruct current state. Buffered local ops are then transformed against the full catch-up delta and sent to server.

**Q: How do you implement version history / "see all changes"?**
A: The operation log in Cassandra IS the version history. To reconstruct document at revision N: load nearest snapshot with snapshot_rev <= N, replay ops from snapshot_rev to N. For diff view: reconstruct state at rev N and N-1 (or use a named snapshot) and run a text diff algorithm (Myers diff).

**Q: What if two users simultaneously rename the document title?**
A: Document metadata (title, sharing settings) is not subject to OT — it's a last-write-wins field stored in PostgreSQL with optimistic locking (version column). Last committer wins; this is acceptable for metadata updates which are rare.

**Q: How do you keep memory usage bounded on the Collaboration Service?**
A: Evict sessions idle for > 30 min from memory; persist final state to Cassandra and S3 snapshot. On next access, reload from snapshot. In-memory op buffer is capped at last 1000 ops (enough for OT transform window).

---

## 19 — Distributed Job Scheduler
**Asked at:** LinkedIn, Uber, Amazon, Microsoft  **Difficulty:** 🟡  **Level:** SDE-2

**1. Requirements**
*Functional:*
- Schedule one-time or recurring jobs using cron expressions (e.g., `0 9 * * 1` = every Monday 9 AM)
- Jobs execute at (or very near) their scheduled time with at-most-once or at-least-once delivery guarantees, configurable per job
- Support job types: HTTP callback, queue message, arbitrary script/container
- Job state transitions: PENDING -> RUNNING -> SUCCEEDED / FAILED / TIMED_OUT
- Retry on failure with configurable retry count and backoff policy (fixed / exponential)
- Dead-letter queue for jobs that exhausted retries
- Job history: last N executions with status, start/end time, logs URL
- Pause, resume, delete individual jobs or job groups

*Non-functional:*
- Scale: 10 M registered jobs; 100 K jobs firing per minute at peak
- Scheduling accuracy: fire within +/-1 second of scheduled time P95
- Availability: 99.99% — scheduler must not be a single point of failure
- Exactly-once execution (or idempotent at-least-once) — critical for financial jobs
- Low scheduler overhead: scanning 10 M jobs every second is not acceptable

**2. Back-of-envelope estimation**
- 10 M registered jobs; assume avg job fires once per hour -> 10 M / 3600 = ~2,778 jobs/sec average; peak (e.g., top-of-hour burst): 100 K jobs/min = 1,667 jobs/sec
- Job record size: ~200 bytes. 10 M * 200 = 2 GB — fits in memory; also store in DB
- Execution history: 100 K executions/min * 500 bytes/record = 50 MB/min -> 72 GB/day; retain 30 days = 2.16 TB
- Scheduler nodes needed: each node can poll and dispatch ~10 K jobs/sec; need ~17 nodes at peak with 2x headroom -> 35 nodes
- Redis sorted set for time index: 10 M members at 50 bytes each = 500 MB — fits in a single Redis instance (use cluster for redundancy)

**3. API design**
```
# Create a new job
POST /jobs
Body: {
  name: "daily-report",
  type: "http",              // http | sqs | container
  schedule: "0 9 * * *",    // cron expression or ISO8601 for one-time
  timezone: "America/New_York",
  target: { url: "https://...", method:"POST", body:{} },
  retryPolicy: { maxAttempts: 3, backoff:"exponential", initialDelay:30 },
  timeout: 300,              // seconds
  deliveryGuarantee: "at-least-once"  // or "at-most-once"
}
Response: { jobId, nextFireAt }

# Get job + last 10 executions
GET /jobs/{jobId}
Response: { jobId, schedule, status, nextFireAt, recentExecutions:[...] }

# Pause / resume
PATCH /jobs/{jobId}/pause
PATCH /jobs/{jobId}/resume

# Force trigger immediately
POST /jobs/{jobId}/trigger

# Delete
DELETE /jobs/{jobId}

# List executions
GET /jobs/{jobId}/executions?limit=50&cursor=...
```

**4. Data model / schema**
```
jobs  (PostgreSQL — source of truth)
  job_id            UUID PK
  name              TEXT
  owner_id          UUID
  cron_expr         TEXT        -- null for one-time
  timezone          TEXT
  type              ENUM(http, sqs, container)
  target_config     JSONB       -- url/queue/image depending on type
  retry_policy      JSONB       -- { maxAttempts, backoff, initialDelay }
  timeout_secs      INT
  delivery          ENUM(at_least_once, at_most_once)
  status            ENUM(active, paused, deleted)
  next_fire_at      TIMESTAMPTZ  -- computed; indexed
  created_at        TIMESTAMPTZ
  INDEX (status, next_fire_at)  -- key query: active jobs due soon

executions  (TimescaleDB / append-only partitioned by month)
  execution_id      UUID PK
  job_id            UUID FK jobs
  attempt           INT
  status            ENUM(running, succeeded, failed, timed_out)
  scheduled_at      TIMESTAMPTZ
  started_at        TIMESTAMPTZ
  finished_at       TIMESTAMPTZ
  worker_id         TEXT
  logs_url          TEXT
  INDEX (job_id, scheduled_at DESC)

-- Redis: time-indexed queue of due jobs
SORTED SET  "scheduler:due_jobs"
  member: job_id (string)
  score:  next_fire_at as Unix timestamp (float)
```
Partition key for executions: `job_id` % N shards if using Cassandra instead of PG.

**5. High-level architecture**
```
Clients (API / SDK / UI)
        |
   +----v------------------------------------------+
   |           API Service (stateless)              |
   |  CRUD for jobs; validates cron; writes PG      |
   |  Updates Redis sorted set on create/update     |
   +----+------------------------------------------+
        | write                 | read
   +----v--------------+  +----v------------------+
   |  PostgreSQL        |  |  Redis Cluster         |
   |  (jobs table)      |  |  Sorted Set:           |
   |  source of truth   |<-|  score=fire_time       |
   +-------------------+  |  for fast due-job      |
                          |  lookup                |
                          +--------+---------------+
                                   | ZRANGEBYSCORE poll
                        +----------v-----------------------------------+
                        |      Scheduler Nodes (fleet, 35 nodes)       |
                        |                                               |
                        |  Each node runs:                              |
                        |  1. Poller: every 1s ZRANGEBYSCORE            |
                        |     score <= now+5s LIMIT 500                |
                        |  2. Locker: SET NX job:{id}:lock 30s         |
                        |     (distributed lock via Redis)             |
                        |  3. Dispatcher: fan-out to Worker Queue      |
                        |  4. Leader Election: one node is "master"    |
                        |     for re-computing next_fire_at            |
                        +----------+-----------------------------------+
                                   | enqueue
                        +----------v---------------------------------+
                        |       Job Queue (Kafka)                     |
                        |  Topic: job_executions                      |
                        |  Partitioned by job_id                      |
                        +----------+---------------------------------+
                                   | consume
                        +----------v---------------------------------+
                        |       Worker Pool                           |
                        |  Pulls from Kafka                           |
                        |  Executes: HTTP call / SQS msg /            |
                        |  container invocation                       |
                        |  Writes execution record to PG              |
                        |  On success: ACK Kafka offset               |
                        |  On failure: retry or DLQ                   |
                        +----------+---------------------------------+
                                   |
                        +----------v---------------------------------+
                        |  Dead-Letter Queue (Kafka DLQ)             |
                        |  + Alert service                            |
                        +--------------------------------------------+
```

**6. Deep dive**

**Cron expression parsing and next_fire_at computation**
Parse the cron expression (5 or 6 fields: sec, min, hour, dom, month, dow) into a schedule object. Use a library (e.g., Quartz in Java, croniter in Python). To find the next fire time: given current time T and timezone, advance field by field from second to minute to hour to day until all constraints are satisfied. Store the resulting UTC timestamp in `next_fire_at` and in the Redis sorted set as the score. After each execution, the scheduler recomputes the next_fire_at from the completed-execution timestamp (not from original next_fire_at, to avoid drift).

**Distributed lock for exactly-once execution**
The core problem: multiple scheduler nodes will see the same due job in the Redis sorted set at the same second. Only one should execute it. Solution: use Redis SET NX (set if not exists) with a TTL equal to job timeout + buffer: `SET job:{jobId}:lock {workerId} NX EX 330`. Only the node that wins the SET NX proceeds to enqueue the job. The lock TTL ensures a crashed worker eventually releases the lock so a retry can happen. For at-most-once: do not retry if lock acquisition fails. For at-least-once: retry after lock expiry if execution record shows no success.

**Job state machine**
```
PENDING (queued in Kafka)
   | worker picks up
   v
RUNNING (execution record created; lock held)
   | completes successfully
   +---> SUCCEEDED (write execution record; release lock; compute next_fire_at; re-insert to sorted set)
   | exceeds timeout
   +---> TIMED_OUT (worker kills task; write record; attempt < maxAttempts -> re-queue with backoff delay)
   | HTTP 5xx / exception
   +---> FAILED (attempt < maxAttempts -> re-queue with exponential delay; else -> DLQ)
```

**Retry with exponential backoff**
On FAILED: delay = initialDelay * 2^(attempt-1) + jitter. For initialDelay=30s, maxAttempts=3: delays are 30s, 60s, 120s. Retry is enqueued by inserting a new entry into the Redis sorted set with score = now + delay. The job in PG gets a `next_retry_at` field updated.

**7. Data flow** — Recurring job fires
1. Job "send-weekly-report" has next_fire_at = 2026-06-13 09:00:00 UTC.
2. At 08:59:58, Scheduler Node A's poller runs `ZRANGEBYSCORE scheduler:due_jobs -inf [now+5]` and retrieves the job.
3. Node A attempts `SET job:abc123:lock nodeA NX EX 330`. Wins the lock.
4. Node A enqueues message `{jobId:"abc123", executionId:"exec-xyz", attempt:1}` to Kafka topic `job_executions`.
5. Node A computes next_fire_at = 2026-06-20 09:00:00 UTC, updates PG `jobs.next_fire_at`, and does `ZADD scheduler:due_jobs <next_ts> abc123`.
6. Worker node consumes from Kafka. Writes `executions(exec-xyz, running, ...)` to PG.
7. Worker makes HTTP POST to target URL. Receives 200 OK within 5 seconds.
8. Worker writes `executions(exec-xyz, succeeded, finished_at=now)` to PG. ACKs Kafka offset.
9. Worker releases lock: `DEL job:abc123:lock`.

**8. Scaling**
- **10x (1 M jobs/min):** Shard Redis sorted set by (job_id hash % N) — each scheduler node owns a subset of shards; horizontal scale of scheduler nodes and workers.
- **100x:** Partition Kafka topics by job type; dedicated worker pools per job type (HTTP workers vs container workers); PG read replicas for history queries; TimescaleDB for execution history with automatic partition pruning.
- **1000x:** Multi-region deployment with region affinity for jobs (job registered in US fires from US region); use etcd or ZooKeeper for cluster membership and leader election instead of Redis pub/sub.

**9. Trade-offs**
- **Redis sorted set vs DB polling:** DB polling (SELECT * WHERE next_fire_at <= NOW()) requires a full index scan under heavy load and doesn't scale to 10 M jobs. Redis sorted set gives O(log N) range queries and in-memory speed. Downside: Redis is not the source of truth — PG is. Both must be kept in sync (handled at write time by API service).
- **Kafka vs SQS for job queue:** Kafka chosen for replayability (replay failed jobs), ordering within a partition, and high throughput. SQS is simpler but lacks replay and has per-message cost at scale.
- **At-least-once vs exactly-once:** True exactly-once requires distributed transactions (Kafka transactions + DB write atomically). Instead, design jobs to be idempotent, use at-least-once delivery, and detect duplicates via execution record (INSERT OR IGNORE on execution_id).

**10. Follow-up questions**

**Q: How do you handle a "poison" job that always times out and monopolizes a worker?**
A: After maxAttempts failures, move to DLQ. Worker marks the execution as DEAD in PG. Alert fires to job owner. The job is not re-scheduled until owner explicitly re-enables it. DLQ is a separate Kafka topic with a dedicated consumer for inspection.

**Q: What if the scheduler node crashes after acquiring the lock but before enqueuing to Kafka?**
A: The Redis lock TTL expires after (timeout + buffer) seconds. A watchdog process on other nodes periodically scans for jobs whose lock has expired but whose execution record shows no `started_at` — these are re-queued. This is the at-least-once retry path.

**Q: How do you ensure top-of-the-hour burst (millions of cron jobs set to "0 * * * *") doesn't overwhelm the system?**
A: (1) Add random jitter to scheduled times (+/-30s optional, configured per job). (2) Pre-warm the worker pool before the hour. (3) Kafka absorbs the burst naturally — producers (scheduler nodes) enqueue quickly; consumers (workers) process at their rate. (4) Rate-limit dispatch per target domain to avoid thundering-herd on downstream services.

**Q: How do you implement distributed leader election for the scheduler's "master" role?**
A: Use Redis SETNX with a TTL for a `scheduler:leader` key. The node that holds it is leader; all others are followers. Leader heartbeats the key every TTL/3 seconds. If the leader dies, the key expires and another node wins the next SETNX race. Alternatively use ZooKeeper ephemeral nodes or etcd leases for stronger guarantees.

---

## 20 — Google Search
**Asked at:** Google  **Difficulty:** 🔴  **Level:** Senior

**1. Requirements**
*Functional:*
- Crawl the web continuously and index discovered pages
- Accept keyword queries and return ranked list of relevant URLs with title + snippet
- Handle query operators: site:, filetype:, quotes for exact match, minus for exclusion
- Autocomplete / query suggestions as user types
- Freshness: breaking news pages indexed within minutes; regular pages within days
- Personalized ranking based on user history (optional / signed-in users)
- Localized results: language detection, region-based ranking

*Non-functional:*
- Scale: crawl 20 B pages; index 100 B terms; serve 9 B queries/day (~100 K QPS)
- Latency: search result < 200 ms P99; autocomplete < 50 ms
- Availability: 99.999%
- Index freshness: high-priority pages re-crawled every 1 hour; avg every 3 days
- Storage: inverted index ~20 PB (compressed); raw page store ~100 PB

**2. Back-of-envelope estimation**
- 9 B queries/day = 104 K QPS; peak 3x = 312 K QPS
- Index size: 20 B pages * avg 500 distinct terms/page = 10 T term-document pairs; at 8 bytes each = 80 TB uncompressed; with variable-byte + delta encoding ~20 TB per replica; 3 replicas = 60 TB per shard group
- Crawler: crawl 20 B pages, re-crawl every 3 days avg -> 20 B / (3 * 86400) = 77 K pages/sec to crawl
- Raw HTML: avg page 50 KB * 20 B = 1 PB; store in GCS/S3
- Postings list per term: avg 1 M documents per common term; with docID + score = 12 bytes -> 12 MB per term; top 1 M terms * 12 MB = 12 TB in hot index tier
- Serving: 312 K QPS * avg 10 shard fan-out * 1 ms/shard = ~3 M shard-ops/sec

**3. API design**
```
# Public search API
GET /search?q=distributed+systems&page=1&num=10&hl=en&gl=US
Response: {
  totalResults: 4200000000,
  timeTakenMs: 147,
  results: [
    { url, title, snippet, lastCrawled, rank },
    ...
  ],
  relatedQueries: ["..."],
  spellCorrection: "did you mean: ..."
}

# Autocomplete
GET /complete?q=dist&hl=en
Response: { suggestions: ["distributed systems", "disney+", ...] }

# Internal: Crawler submits page
POST /indexer/ingest
Body: { url, rawHtml, crawledAt, httpStatus, contentHash }

# Internal: Force re-index a URL (for publishers)
POST /indexer/submit
Body: { url }
```

**4. Data model / schema**
```
-- Crawl frontier (URL queue, distributed)
frontier
  url_hash    BIGINT PK   -- MurmurHash of normalized URL
  url         TEXT
  priority    FLOAT       -- based on PageRank estimate + freshness need
  last_crawled TIMESTAMPTZ
  next_crawl  TIMESTAMPTZ
  etag        TEXT        -- for conditional GET
  INDEX (next_crawl) WHERE status='active'

-- Raw page store: object storage (S3/GCS)
  Key: {url_hash}/{crawl_timestamp}.gz
  Value: gzip(rawHtml)

-- Forward index: docId -> terms + positions (used during indexing)
-- Stored in BigTable / HBase: rowKey = doc_id
forward_index
  doc_id      BIGINT
  url         TEXT
  title       TEXT
  terms       BYTES      -- serialized: [{term, tf, positions:[]}]
  pagerank    FLOAT
  crawled_at  TIMESTAMPTZ

-- Inverted index: term -> postings list
-- Stored in custom segment files (like Lucene), sharded by term hash
inverted_index_segment
  term        TEXT  (shard key: hash(term) % N_shards)
  postings    BYTES -- delta-encoded [(doc_id, tf, positions)...], sorted by score desc
  df          INT   -- document frequency (for IDF)
```

**5. High-level architecture**
```
Internet
   |
   v
+------------------------------------------------------------------+
|                      CRAWL PIPELINE                               |
|                                                                   |
|  URL Frontier ---> Crawler Workers (100K parallel HTTP clients)  |
|  (distributed    |  * politeness: 1 req/sec per domain           |
|   priority Q)    |  * robots.txt / noindex respect               |
|                  |  * DNS cache + HTTP/2                          |
|                  v                                               |
|            Raw Page Store (GCS) ---> Content Dedup (simhash)    |
+---------------------------+--------------------------------------+
                            | raw HTML
                            v
+------------------------------------------------------------------+
|                    INDEXING PIPELINE                              |
|                                                                   |
|  HTML Parser -> Tokenizer -> Stemmer -> Stop-word filter         |
|       |              |                                           |
|       v              v                                           |
|  Link Extractor  Term Extractor -> TF computation               |
|  (feeds Frontier)    |                                           |
|                      v                                           |
|             Forward Index Writer (BigTable)                      |
|                      |                                           |
|             Periodic Map-Reduce / Dataflow job:                  |
|             Merge forward index -> build inverted index segments  |
|             Delta encode postings; sort by BM25 score desc       |
+---------------------------+--------------------------------------+
                            | index segments
                            v
+------------------------------------------------------------------+
|                     INDEX SERVING TIER                            |
|                                                                   |
|  Leaf Nodes (index shards, 10K+ servers)                         |
|  Each holds: subset of inverted index in RAM + SSD               |
|  Handle: lookup term -> fetch postings -> compute local scores   |
|                  |                                               |
|  Root / Mixer node (per datacenter)                              |
|  Receives query -> fans out to all leaf nodes in parallel        |
|  Collects top-K from each leaf -> merge-sorts -> reranks         |
|  -> applies personalization -> returns top 10                    |
+---------------------------+--------------------------------------+
                            |
   +------------------------v----------------------------------+
   |              Query Understanding Layer                    |
   |  * Spell correction (n-gram model)                       |
   |  * Query tokenization + stemming                         |
   |  * Named entity recognition                              |
   |  * Query rewriting / expansion (synonyms)                |
   |  * SafeSearch classification                             |
   +------------------------+----------------------------------+
                            |
   +------------------------v----------------------------------+
   |              Frontend / Serving                           |
   |  Load Balancer -> Query servers (stateless)              |
   |  Snippet generation (KWIC extraction from doc)           |
   |  Universal search: merge web + images + news             |
   |  Cache layer (Memcached) for popular queries              |
   +-----------------------------------------------------------+
```

**6. Deep dive**

**Inverted index structure**
An inverted index maps each term to a postings list: a sorted array of (docID, term_frequency, [positions]) entries. Positions enable phrase queries ("new york" — require term1 position + 1 = term2 position). DocIDs are delta-encoded (store differences between consecutive IDs, not absolute values) because docIDs are large 64-bit numbers but differences are small — variable-byte encoding reduces 8 bytes to 1-3 bytes per entry. Skip pointers every sqrt(N) entries speed up AND-merge of two long postings lists (instead of scanning entire list, jump past blocks). Segmented index: new documents go into a small delta index; periodically merged with main index via log-structured merge (like LevelDB/Lucene).

**Index sharding strategies**
Two approaches: (1) shard by doc_id (document partitioning) — each shard holds all terms for a subset of documents; a query must fan out to ALL shards and merge results. (2) shard by term (term partitioning) — each shard owns all postings for a subset of terms; a query hits only shards containing its query terms. Google uses document partitioning because it gives better load balance, allows replication of popular shards, and simplifies geo-distribution.

**Ranking overview**
BM25 score(d, q) = Sum_t IDF(t) * (tf(t,d) * (k1+1)) / (tf(t,d) + k1 * (1-b+b*|d|/avgdl)) where k1~1.2, b~0.75. IDF = log((N - df + 0.5) / (df + 0.5)). PageRank adds a global quality signal. Learning-to-rank (GBDT / neural) takes O(100) features (BM25, PageRank, anchor text, click-through rate, freshness, HTTPS, mobile-friendly) and produces final score. This reranking happens on the mixer after initial BM25 retrieval.

**7. Data flow** — Query serving
1. User types "distributed systems interview" -> browser sends GET /search?q=...
2. Query server applies spell-check, tokenization, stemming -> tokens: ["distribut", "system", "interview"].
3. Mixer node receives query. Fans out to all leaf shard nodes: "get top-200 docs for each token."
4. Each leaf fetches postings for each token, computes BM25 scores, merges (AND/OR based on query), returns top-200 (docId, score, title_snippet).
5. Mixer merges responses from all shards (merge-sort by score), deduplicates, takes top-1000.
6. Reranker applies learning-to-rank model: adds PageRank, freshness, personalization features -> reranked top-10.
7. Snippet service: for each top-10 docId, fetches stored snippet (pre-computed) or extracts KWIC from stored doc. Highlights query terms.
8. Response assembled and returned. Total time: ~150 ms.
9. Popular query result cached in Memcached with TTL 5 min.

**8. Scaling**
- **10x crawl / index:** Add more crawler workers; increase indexing pipeline parallelism (more Dataflow workers); add more leaf shard nodes.
- **100x query load:** Add more mixer nodes; replicate each index shard 3x and load-balance across replicas; geo-distribute index to serve from nearest datacenter.
- **1000x:** Hierarchical serving tree (root -> mixer -> leaves at 3 levels); partition index by content type (web, news, images, video separately); serve from 10+ global datacenters with anycast routing.

**9. Trade-offs**
- **Push vs pull freshness:** Pull (scheduled re-crawl by priority) chosen over push (publishers ping Google) — push can be abused; pull gives full control over freshness schedule.
- **Document vs term partitioning:** Document partitioning chosen — simpler load balancing, better fault isolation (a shard failure loses only its docs, not all queries containing a term).
- **Approximate top-K:** Leaf nodes return approximate top-200 rather than exact scores. Some relevant docs below rank 200 in a leaf may be missed. Acceptable trade-off for 10-100x latency reduction vs exact merge.
- **Index freshness vs throughput:** Serving a real-time index for every crawled doc would require continuous merges. Instead: delta index (seconds-fresh for breaking news) + main index (days-fresh for everything else) + periodic merges at off-peak hours.

**10. Follow-up questions**

**Q: How do you crawl 20 billion pages without being blocked?**
A: Respect robots.txt and crawl-delay. One request per second per domain. Identify as Googlebot with a legitimate user-agent. Distribute crawl IPs across large CIDR ranges. Prioritize sitemaps. Use conditional GET (If-Modified-Since / ETag) to skip unchanged pages.

**Q: How do you detect and handle near-duplicate content?**
A: Compute SimHash (a locality-sensitive hash) of each page's content. Two pages with SimHash Hamming distance < 3 are near-duplicates. Keep highest-quality version (shortest URL, highest PageRank). Store SimHash in frontier DB for fast lookup.

**Q: How do you handle the PageRank computation at scale?**
A: PageRank is computed periodically (weekly) via a distributed iterative graph algorithm (like Pregel or Spark GraphX) over the link graph. The web graph has ~20 B nodes and ~300 B edges. Each iteration: PR(v) = (1-d)/N + d * Sum PR(u)/out_degree(u). Converges in ~50 iterations. Store final PR scores in BigTable indexed by docId; join with BM25 scores at ranking time.

**Q: How would you design the autocomplete feature?**
A: Store top-N query suggestions per prefix in a trie or sorted structure. Backed by offline analysis of query logs (top 1 M queries by frequency). Serve from Redis with prefix as key -> list of suggestions. For freshness (trending queries), run a streaming job (Kafka + Flink) that detects queries rising in frequency and inserts them into the autocomplete index in real time.

---

## 21 — Distributed Cache
**Asked at:** Amazon, Google, Meta  **Difficulty:** 🔴  **Level:** SDE-2/Senior

**1. Requirements**
*Functional:*
- Key-value store: get(key), set(key, value, ttl), delete(key)
- Support for TTL (time-to-live) with lazy expiry + active expiry sweep
- Atomic operations: incr/decr (for counters), setnx (set-if-not-exists), compare-and-swap
- Multiple eviction policies: LRU, LFU, allkeys-random, volatile-lru (configurable per namespace)
- Clustering: distribute data across N nodes with configurable replication factor
- Client-side routing: clients know which node holds a key (no proxy hop needed)

*Non-functional:*
- Latency: get/set < 1 ms P99 (in-process network round-trip)
- Throughput: 10 M ops/sec per cluster
- Availability: 99.99% — node failures must be handled without manual intervention
- Consistency: eventual (replication is async) — acceptable for cache use cases
- Memory efficiency: minimize per-key overhead; support up to 1 TB of data per cluster

**2. Back-of-envelope estimation**
- 10 M ops/sec; 70% reads, 30% writes = 7 M reads + 3 M writes/sec
- Avg value size: 1 KB. 3 M writes/sec * 1 KB = 3 GB/sec write throughput per cluster
- Memory per cluster: 1 TB = 1 T keys at 1 KB avg; with 50-byte overhead per key (key + metadata + pointers) = 1.05 TB raw
- Nodes: each node handles 500 K ops/sec and holds 100 GB data -> need 20 nodes for throughput; 10 TB / 100 GB = 10 nodes for storage. Bottleneck is throughput -> 20 nodes + 2x headroom = 40 nodes
- Replication: RF=3, each key on 3 nodes -> ~60 TB aggregate memory across cluster
- Cache hit rate target: 95% -> 5% misses at 10 M ops/sec = 500 K DB queries/sec — need fast DB backend

**3. API design**
```
# Client API (binary protocol — Redis RESP or Memcached protocol)
SET key value [EX seconds] [NX|XX]
GET key
DEL key [key ...]
MGET key [key ...]          -- multi-get (pipeline)
MSET key value [key value]  -- multi-set
INCR key
INCRBY key delta
SETNX key value             -- set if not exists (atomic)
TTL key                     -- remaining TTL in seconds
SCAN cursor [MATCH pattern] [COUNT count]   -- keyspace iteration

# Cluster management API (HTTP)
GET  /cluster/nodes          -- list all nodes + slots
POST /cluster/rebalance      -- trigger shard rebalancing
GET  /cluster/stats          -- ops/sec, hit rate, memory usage per node
POST /cluster/nodes/{id}/evict  -- emergency memory reclaim
```

**4. Data model / schema**
```
In-memory per-node data structure:

Hash table (open addressing, power-of-2 sizing)
  Each slot:
    key_ptr   -> slab-allocated byte string
    val_ptr   -> slab-allocated byte string
    ttl       UINT32    -- absolute expiry epoch seconds (0 = no TTL)
    freq      UINT16    -- for LFU: access frequency counter
    last_used UINT32    -- for LRU: last access epoch seconds
    flags     UINT8     -- type flags

Slab allocator:
  Memory divided into 1 MB slabs, each holding objects of one size class
  Size classes: 64B, 128B, 256B, 512B, 1KB, 2KB, 4KB, ... (each ~1.25x previous)
  Eliminates fragmentation; enables O(1) free

LRU linked list (doubly-linked) maintained per hash table
  Head = MRU; tail = LRU candidate for eviction

Cluster routing table (on each client):
  consistent_hash_ring: sorted array of (virtual_node_hash, node_id)
  Replication positions: primary + next 2 clockwise on ring
```

**5. High-level architecture**
```
Application Servers
  |  Client library (consistent hash -> direct TCP to node)
  |  No proxy hop for hot path
  v
+---------------------------------------------------------------+
|                  Cache Cluster (40 nodes)                      |
|                                                                |
|  +-----------+  +-----------+  +-----------+                  |
|  |  Node 1   |  |  Node 2   |  |  Node 3   |  ... Node 40    |
|  |  slots:   |  |  slots:   |  |  slots:   |                 |
|  |  0 - 4095 |  | 4096-8191 |  | 8192-12287|                 |
|  |  RF=3:    |  |           |  |           |                 |
|  |  also     |  |           |  |           |                 |
|  |  holds    |  |           |  |           |                 |
|  |  replicas |  |           |  |           |                 |
|  |  of N2,N3 |  |           |  |           |                 |
|  +-----------+  +-----------+  +-----------+                  |
|                                                                |
|  Gossip protocol between nodes:                               |
|    * Failure detection (heartbeat every 1s; dead after 3)    |
|    * Cluster state propagation (new node, removed node)      |
|    * Slot migration tracking                                  |
+---------------------------------------------------------------+
  |
  v
+---------------------------+
|  Config / Cluster Manager |   (ZooKeeper or etcd)
|  Stores:                  |   Elects coordinator node
|  * Node list + slots      |   Detects node failures
|  * Rebalance state        |   Triggers slot migration
+---------------------------+
```

**6. Deep dive**

**Consistent hashing ring with virtual nodes — worked example**
Three physical nodes: N1 (10.0.0.1), N2 (10.0.0.2), N3 (10.0.0.3). Use 150 virtual nodes per physical node = 450 total virtual nodes. Hash space: 0 to 2^32 - 1. Virtual node hashes are evenly distributed around the ring. For a key "user:1234": hash("user:1234") = 2,147,483,789 (example). Walk the ring clockwise to find the first virtual node with hash >= 2,147,483,789. Suppose that's virtual node "N2-vn47" at position 2,147,500,000. So the primary is N2. The next two clockwise virtual nodes map to N3 and N1 -> those are the replica holders. When N2 fails: its virtual nodes are removed from the ring. Only keys in N2's slots need to be re-served from the replicas on N3 and N1. Only ~1/3 of N2's keys are affected rather than all keys (because N2 has virtual nodes scattered across the ring, and each virtual node's successor is a different physical node). This is the core advantage of virtual nodes over simple consistent hashing — even load distribution even with heterogeneous node capacities.

**Hot-key problem and mitigation**
A viral post's like-count key might receive 100 K reads/sec on a single cache node. Mitigations: (1) Local shadow cache — application servers keep a tiny LRU (e.g., 1000 entries) in process memory; hot keys are cached in every app server's local dict, reducing cache node load from 100 K to 100 K/N_app_servers. (2) Key sharding with suffix — instead of "post:viral", clients randomly choose from "post:viral:0" through "post:viral:9"; reads are spread across 10 keys on potentially 10 different nodes; the "writer" (like counter) writes to all 10 shards; reads sum them up (eventual consistency). (3) Probabilistic early expiry — to prevent thundering herd on expiry, expire the key slightly before its TTL with probability proportional to how close it is to expiry, so individual clients regenerate it before the full burst.

**Cache-aside vs write-through vs write-behind**
Cache-aside: app reads DB on miss, writes to cache manually. Simple but race condition on concurrent writes: thread A reads old DB value -> thread B writes new value to DB + cache -> thread A overwrites cache with stale value. Fix: use cache-aside with version check or short TTL. Write-through: write to cache and DB synchronously on every update. Keeps cache warm but doubles write latency. Write-behind (write-back): write to cache immediately, return success; async write to DB in background. Fastest writes but risk data loss on cache node crash before async flush.

**7. Data flow** — Cache read (cache-aside pattern)
1. App calls `cache.get("user:profile:42")`. Client library hashes key -> maps to Node 7 (primary).
2. TCP request to Node 7. Node 7 looks up key in hash table -> found. Checks TTL: not expired. Moves to head of LRU list. Returns value.
3. Total latency: ~0.3 ms.
4. If Node 7 is down: client library detects TCP failure (within 100 ms). Reads from replica on Node 8.
5. If key is not found (cache miss): client returns null. App queries PostgreSQL -> gets user profile -> calls `cache.set("user:profile:42", value, EX 3600)` -> stored on Node 7 + async replicated to Nodes 8, 9.

**8. Scaling**
- **10x traffic (100 M ops/sec):** Add more nodes; client library automatically rebalances slots via consistent hashing; slot migration is online (live data moved in background while serving traffic).
- **100x data (10 PB):** Introduce tiered caching: hot tier (pure in-memory, NVMe-backed for overflow), warm tier (SSD with memory index); keys automatically migrate between tiers by access frequency.
- **1000x:** Geo-distributed caches per region; cross-region replication for global hot keys; regional cache clusters are independent (no cross-DC synchronous reads).

**9. Trade-offs**
- **Consistent hashing vs range partitioning:** Consistent hashing chosen — minimal key migration when nodes join/leave (only adjacent slots affected); range partitioning gives sequential key locality (useful for scans) but not needed for a pure key-value cache.
- **Async vs sync replication:** Async chosen — sync replication requires all replicas to ack before returning, adding 1-2 ms per write. Cache is not the source of truth; slight staleness on failover is acceptable.
- **LRU vs LFU eviction:** LRU evicts items not accessed recently, which may include still-popular items that happened to not be accessed in the last window (scan pollution). LFU tracks frequency — better for stable hot-key workloads. Redis 4.0+ uses LFU with a frequency counter that decays over time.

**10. Follow-up questions**

**Q: How do you handle the thundering herd on a cache cold start?**
A: (1) Request coalescing: use a per-key mutex (or promise/future) so only one goroutine/thread fetches from DB; others wait and get the same result. (2) Probabilistic early expiry: as TTL approaches zero, randomly expire the cache entry slightly early so it gets repopulated before the formal expiry, spreading the regeneration across time. (3) Warm the cache proactively from a snapshot of DB data before taking traffic.

**Q: How do you implement atomic increment (INCR) in a distributed cache?**
A: On the primary node: the hash table entry for the key is updated atomically using a mutex (or via a single-threaded event loop like Redis). The increment is applied in-place, then async-replicated. No two-phase commit needed — the primary always wins; replicas are eventually consistent for counters.

**Q: What happens during a slot migration (adding a new node)?**
A: The cluster coordinator assigns a slot range to the new node. The source node begins streaming keys in that range to the destination node. During migration, reads/writes for the migrating slot are first tried on the source; if the key has already migrated, source redirects client to destination (MOVED error). Client updates its routing table. Once migration completes, source deletes the migrated keys. This is the Redis Cluster migration protocol — fully online, no downtime.

**Q: How do you ensure cache consistency when the DB is updated?**
A: Pattern: write DB first, then delete cache entry (not update it). Deleting avoids the race where a stale write overwrites a fresher DB value. Use short TTL as a backstop. For strong consistency needs, use a "refresh-ahead" pattern: a background job subscribes to DB change events (CDC via Debezium) and proactively updates the cache entry.

---

## 22 — Payment System (Stripe / Razorpay)
**Asked at:** Razorpay, PhonePe, Stripe, Goldman Sachs, PayPal  **Difficulty:** 🔴  **Level:** Senior

**1. Requirements**
*Functional:*
- Accept payments from customers via cards, UPI, net banking, wallets
- Create charges, refunds, and payouts to merchants
- Support multi-step payments: authorize -> capture -> settle
- Transfer money between internal accounts (ledger debit/credit)
- Reconciliation: detect missing or duplicate settlements daily
- Webhooks to notify merchants of payment status changes
- Idempotent API — retrying the same request never double-charges

*Non-functional:*
- Scale: 10 M transactions/day = 116 TPS avg; peak 10x = 1,160 TPS
- Latency: payment API response < 3 s (includes bank round-trip)
- Consistency: STRONG — data loss or double-charge is catastrophic
- Durability: all transactions persisted before response; zero data loss
- Availability: 99.99% (< 1 hr downtime/year)
- Auditability: every state transition logged immutably; full audit trail

**2. Back-of-envelope estimation**
- 10 M txns/day; avg txn value Rs.1,000; total GMV Rs.10 B/day
- Payment record: ~500 bytes; 10 M * 500 = 5 GB/day; 1.8 TB/year -> manageable in PostgreSQL
- Ledger entries: 4 rows per transaction (debit + credit * 2 parties) = 40 M rows/day; at 100 bytes/row = 4 GB/day
- Idempotency table: 1 row per API call; at TTL 24 hrs, ~10 M active rows at any time; 10 M * 200 bytes = 2 GB in memory / Redis
- Webhook events: 10 M/day * 3 delivery attempts avg = 30 M HTTP calls/day = 347 calls/sec outbound
- Reconciliation: runs nightly over 10 M records — batch job, not latency-sensitive

**3. API design**
```
# Create a payment intent (authorization)
POST /v1/payment_intents
Headers: Idempotency-Key: <client-generated-UUID>
Body: {
  amount: 10000,       // in smallest currency unit (paise)
  currency: "INR",
  paymentMethod: "card",
  customerId: "cust_abc",
  metadata: { orderId: "order_xyz" }
}
Response: { paymentIntentId, status:"requires_payment_method", clientSecret }

# Confirm payment (capture)
POST /v1/payment_intents/{id}/confirm
Headers: Idempotency-Key: <UUID>
Body: { paymentMethodId: "pm_123", returnUrl: "https://..." }
Response: { paymentIntentId, status:"succeeded"|"requires_action", nextAction }

# Refund
POST /v1/refunds
Headers: Idempotency-Key: <UUID>
Body: { paymentIntentId: "pi_abc", amount: 5000, reason: "customer_request" }
Response: { refundId, status:"pending"|"succeeded" }

# Get payment status
GET /v1/payment_intents/{id}
Response: { paymentIntentId, status, amount, currency, createdAt, updatedAt, timeline:[...] }

# Webhook registration
POST /v1/webhooks
Body: { url: "https://merchant.com/webhook", events:["payment.succeeded","refund.created"] }
```

**4. Data model / schema**
```
-- Idempotency keys (prevent double-charge)
idempotency_keys
  idempotency_key  TEXT PK           -- client-provided UUID
  request_hash     TEXT              -- SHA256 of (endpoint + body)
  response_body    TEXT              -- cached response JSON
  status_code      INT
  created_at       TIMESTAMPTZ
  expires_at       TIMESTAMPTZ       -- typically now+24h
  INDEX (expires_at) for TTL cleanup

-- Payment intents (one per checkout attempt)
payment_intents
  id               TEXT PK           -- pi_xxx
  merchant_id      UUID FK merchants
  customer_id      UUID FK customers
  amount           BIGINT            -- in smallest unit (paise/cents)
  currency         CHAR(3)
  status           ENUM(created, processing, requires_action, succeeded, failed, refunded)
  payment_method   ENUM(card, upi, netbanking, wallet)
  idempotency_key  TEXT FK idempotency_keys
  metadata         JSONB
  created_at       TIMESTAMPTZ
  updated_at       TIMESTAMPTZ
  INDEX (merchant_id, created_at DESC)
  INDEX (status, created_at) for reconciliation

-- Double-entry ledger (immutable; never UPDATE or DELETE)
ledger_entries
  entry_id       BIGSERIAL PK
  txn_id         TEXT              -- groups debit + credit pair
  account_id     UUID FK accounts
  type           ENUM(debit, credit)
  amount         BIGINT            -- always positive
  currency       CHAR(3)
  reference_id   TEXT              -- payment_intent_id or refund_id
  created_at     TIMESTAMPTZ
  INDEX (account_id, created_at DESC)
  INDEX (txn_id)

-- Accounts (merchant float, platform, bank settlement)
accounts
  account_id     UUID PK
  owner_type     ENUM(merchant, customer, platform, bank)
  owner_id       UUID
  currency       CHAR(3)
  -- balance is NOT stored here; derived from SUM(credits) - SUM(debits)
  -- for performance, maintain a materialized balance with optimistic locking:
  balance        BIGINT
  balance_version BIGINT            -- incremented on each update (optimistic lock)
```
Shard key: `merchant_id` for payment_intents; `account_id` for ledger_entries.

**5. High-level architecture**
```
Client (Browser / Mobile App)
       | HTTPS
       v
+----------------------------------------------------------------------+
|                    API Gateway (Kong / Envoy)                         |
|  Auth (API key / OAuth), rate limiting, TLS termination              |
+---------------------+-------------------------------------------------+
                      |
   +------------------v------------------------------------------------+
   |               Payment Service (stateless, N instances)            |
   |  1. Check idempotency key (Redis or PG)                           |
   |  2. Validate request, create payment_intent record                |
   |  3. Call Payment Router                                           |
   +-----------+-------------------------------------------------------+
               |
   +-----------v--------------------------------------------+
   |          Payment Router                                |
   |  Selects PSP (Razorpay/Stripe/Banks)                   |
   |  based on: currency, method, %success                  |
   +-----------+--------------------------------------------+
               | external HTTP call (< 2s timeout)
   +-----------v--------------------------------------------+
   |  PSP (Payment Service Provider) / Bank                |
   |  Visa/Mastercard networks, UPI NPCI                   |
   +-----------+--------------------------------------------+
               | async callback / webhook from PSP
   +-----------v-------------------------------------------------------+
   |              Ledger Service (double-entry accounting)              |
   |  Atomic DB transaction:                                            |
   |    INSERT ledger_entry(debit, customer_account)                   |
   |    INSERT ledger_entry(credit, merchant_account)                  |
   |    INSERT ledger_entry(debit, merchant_account, fee)              |
   |    INSERT ledger_entry(credit, platform_account, fee)             |
   |    UPDATE payment_intents SET status=succeeded                    |
   +----------------------------+--------------------------------------+
                                |
   +----------------------------v--------------------------------------+
   |            Event Bus (Kafka)                                      |
   |  payment.succeeded, refund.created, payout.initiated             |
   +--------+----------------------------------------------------------+
            |                                                   |
   +--------v--------------------------------+  +----------------v-----------+
   |   Webhook Delivery Service              |  |   Reconciliation Service   |
   |   Reads events, delivers to             |  |   Nightly batch: compare   |
   |   merchant URLs with retry              |  |   PSP settlement report    |
   |   (exponential backoff, DLQ)            |  |   vs internal ledger       |
   +-----------------------------------------+  +----------------------------+

   +------------------------------------------------------------------+
   |                  PostgreSQL (Primary + 2 Replicas)                |
   |  Tables: idempotency_keys, payment_intents, ledger_entries,      |
   |          accounts                                                |
   |  Sharded by merchant_id using Citus or application-level hash    |
   +------------------------------------------------------------------+
```

**6. Deep dive**

**Idempotency keys — the most critical piece**
Client generates a UUID v4 before sending the payment request and sends it as the `Idempotency-Key` header. On the server, the first thing the Payment Service does is: `INSERT INTO idempotency_keys (idempotency_key, request_hash, ...) VALUES (...) ON CONFLICT (idempotency_key) DO NOTHING RETURNING *`. If the insert returns a row: this is the first attempt. Proceed with payment processing. If the insert returns nothing (key already exists): a duplicate request. Retrieve and return the stored response: `SELECT response_body, status_code FROM idempotency_keys WHERE idempotency_key = ?`. The request_hash (SHA256 of endpoint + sorted body) detects misuse: same idempotency key with different body -> 422 Unprocessable Entity. The row is set to expire after 24 hours (cleaned by a background job). This atomic INSERT-or-skip prevents any race condition — two concurrent retries both do the INSERT; only one succeeds; the loser waits for the winner to write the response.

**Double-entry ledger accounting**
Every money movement creates exactly two ledger entries: one debit and one credit. They always balance: sum of all debits = sum of all credits (accounting identity). Example — customer pays Rs.1,000, platform fee Rs.20:
```
txn_id  account               type    amount
T1      customer_wallet       DEBIT   1000
T1      merchant_float        CREDIT  980
T1      merchant_float        DEBIT   20
T1      platform_fees         CREDIT  20
```
The ledger is append-only (immutable). To find a merchant's current balance: `SELECT SUM(CASE WHEN type='credit' THEN amount ELSE -amount END) FROM ledger_entries WHERE account_id = ?`. For performance, maintain a materialized balance column in the `accounts` table, updated via optimistic locking (CAS on balance_version) within the same DB transaction as the ledger insert.

**Saga pattern for multi-step payment**
A payment involves multiple services: (1) Charge customer, (2) Reserve merchant inventory (optional), (3) Fulfill order, (4) Settle funds to merchant. Each step can fail. Saga pattern: each step has a compensating transaction. If step 3 fails: compensating T3 (cancel fulfillment) + compensating T1 (refund customer). Implemented as an orchestrator-based saga: a saga coordinator persists the current step and drives transitions. On crash, the coordinator resumes from the last persisted step. Each step is idempotent (safe to re-run). This avoids distributed transactions (2PC) while still achieving eventual consistency with compensation on failure.

**7. Data flow** — Successful card payment
1. Customer clicks "Pay Rs.1,000." Client generates idempotency key `ik_abc123`. Calls POST /v1/payment_intents with the key.
2. API Gateway authenticates merchant API key. Passes to Payment Service.
3. Payment Service: INSERT idempotency_key -> succeeds (first attempt). Validate amount/currency. INSERT payment_intents(status=created). Return paymentIntentId + clientSecret to client.
4. Client confirms payment: POST /confirm with card tokenized by client-side SDK. Payment Service sets status=processing.
5. Payment Router selects Visa network for card payment. Calls Visa authorization API. Awaits response (< 2s).
6. Visa responds: approved, authorization code ABC. Payment Service calls Ledger Service.
7. Ledger Service: in a single ACID transaction, INSERT 4 ledger rows + UPDATE payment_intents status=succeeded + UPDATE account balances with CAS. COMMIT.
8. Idempotency key updated with response body (status=succeeded).
9. Ledger Service publishes event `payment.succeeded` to Kafka.
10. Webhook Delivery Service consumes event, makes HTTP POST to merchant's webhook URL. Merchant's server receives status update, fulfills order.
11. Payment Service returns 200 response to client with status=succeeded.

**8. Scaling**
- **10x (1,160 TPS -> 11,600 TPS):** Shard PostgreSQL by merchant_id (Citus); connection pooling via PgBouncer; read replicas for reporting queries; Redis for idempotency key lookup to avoid hot PG row.
- **100x:** Dedicated ledger DB (append-only, optimized for writes); separate OLTP DB for payment_intents; async ledger posting via Kafka (event-driven ledger update); partition ledger by account_id range.
- **1000x (UPI-scale: millions TPS):** Multi-region active-active with regional sharding; synchronous cross-region replication only for idempotency keys and ledger; PSP routing with regional fallbacks.

**9. Trade-offs**
- **Strong consistency (ACID) vs availability:** Payments chosen strong consistency. During a DB partition, reject new payments rather than risk double-charging. "CP" in CAP terms.
- **Synchronous PSP call vs async:** Synchronous chosen — user expects immediate confirmation. If PSP call times out, return pending status; a background poller checks PSP for final status and reconciles.
- **Saga vs 2PC:** Saga chosen — 2PC requires all participants to support distributed transactions, is slow (lock held during inter-service calls), and fails catastrophically if coordinator crashes. Saga is slower to compensate but more resilient and doesn't require distributed locking.

**10. Follow-up questions**

**Q: How do you handle a network timeout on the PSP call — you don't know if the charge went through?**
A: Enter "uncertain" state. Persist payment_intent with status=processing. Background reconciliation job polls PSP API every 30 s for up to 5 min with the original PSP reference ID. If PSP confirms success: run Ledger Service to post entries, emit webhook. If PSP confirms failure: update status=failed, release idempotency key. If no response after 5 min: trigger auto-refund if PSP confirms charge, or mark as failed if not. Never leave a payment in processing state for > 10 min.

**Q: How does reconciliation work?**
A: Every night, PSP sends a settlement CSV (or the system polls PSP's settlement API). The reconciliation job compares each PSP transaction ID against the internal ledger. Three discrepancy types: (1) PSP has a charge we have no record of -> create internal record + alert. (2) We have a record but PSP does not -> investigate (may indicate a bug or fraud). (3) Amount mismatch -> alert finance team. Discrepancies trigger alerts and are reviewed by finance within 24 hrs.

**Q: How do you prevent fraud?**
A: Layered defense: (1) Velocity limits: max N transactions per card per hour (Redis counter). (2) Risk scoring: ML model scores each transaction in < 100 ms based on device fingerprint, IP, card BIN, historical patterns. (3) 3DS authentication for high-risk transactions. (4) Address Verification Service (AVS) for card-not-present. (5) Real-time anomaly detection (spike in declines from one merchant -> flag for review).

**Q: How do you handle currency conversion for international payments?**
A: Maintain a rates service that fetches FX rates from market data providers every minute. Rates are valid for 30 s from quote time. Payment intent stores the quoted rate + quote expiry. If user confirms after quote expiry, re-quote and show new rate. Lock in the rate at authorization time; settlement at actual bank rate; FX risk absorbed by platform with daily hedging.

---

## 23 — Ad Click Aggregator
**Asked at:** Meta, Google, Amazon  **Difficulty:** 🔴  **Level:** SDE-2/Senior

**1. Requirements**
*Functional:*
- Track ad clicks from browsers/apps in real time
- Aggregate click counts per (ad_id, hour) and per (advertiser_id, day)
- Expose dashboard queries: "clicks for ad X in the last 7 days, grouped by hour"
- Deduplicate clicks: same user clicking same ad within 1 hour counts as one click
- Support late-arriving events (up to 2 hours late) and correct aggregates
- Provide real-time (< 5 min lag) and historical (up to 3 years) query capability
- Alert advertisers when daily budget is exhausted (real-time)

*Non-functional:*
- Scale: 10 B click events/day = 115 K events/sec; peak 500 K events/sec
- Query latency: dashboard query < 2 s; real-time alert < 30 s
- Availability: 99.99%
- Data accuracy: <=0.1% error on aggregate counts (approximate OK for real-time dashboards; exact for billing)
- Storage: 3 years of hourly aggregates; event log for 30 days for reprocessing

**2. Back-of-envelope estimation**
- 115 K events/sec; each event: 100 bytes (user_id, ad_id, timestamp, IP, device, creative_id) = 11.5 MB/sec = ~1 TB/day raw event log
- Kafka: 115 K events/sec * 100 bytes = 11.5 MB/sec -> 3 Kafka brokers with 10x headroom
- Deduplication window: Redis SET with TTL 1 hr. Key: "ded:{user_id}:{ad_id}:{hour}". Size: if 1 M unique (user, ad) pairs per hour, 1 M * 50 bytes = 50 MB/hr in Redis -> trivially small
- Aggregates storage: 10 B clicks/day / 24 hrs / (unique ads per hour). Assume 10 M active ads; 10 M * 24 * 365 * 3 = 262 B hourly rows/3 years; at 24 bytes/row = 6.3 TB for hourly agg table
- OLAP queries: ClickHouse handles 100 M rows/sec scan -> 6.3 TB / 4 KB/block = fast enough
- Flink cluster: 115 K events/sec / 10 K per task = 12 parallel tasks; 3-node cluster sufficient

**3. API design**
```
# Track a click (called by ad SDK embedded in publisher pages)
POST /v1/clicks
Body: { adId, userId, publisherId, pageUrl, timestamp, creativeId, deviceType }
Response: 202 Accepted   -- fire-and-forget; no blocking

# Query click aggregates (advertiser dashboard)
GET /v1/aggregates/clicks?adId=xxx&from=2026-06-01&to=2026-06-13&granularity=hour
Response: {
  adId: "xxx",
  data: [{ ts: "2026-06-01T00:00:00Z", clicks: 1423 }, ...]
}

# Query by advertiser
GET /v1/aggregates/clicks?advertiserId=yyy&from=2026-06-01&to=2026-06-13&granularity=day
Response: { advertiserId:"yyy", data:[{ date, clicks, impressions, ctr }...] }

# Internal: budget check (called by ad server before showing ad)
GET /v1/budgets/{advertiserId}/remaining?date=2026-06-13
Response: { remaining: 4500, currency:"USD", exhausted:false }
```

**4. Data model / schema**
```
-- Raw event stream (Kafka topic, retained 30 days)
Topic: ad_clicks
Partition key: ad_id (ensures per-ad ordering)
Schema (Avro):
  user_id       STRING
  ad_id         STRING
  advertiser_id STRING
  publisher_id  STRING
  timestamp     LONG (epoch ms)
  ip            STRING
  device_type   ENUM(desktop, mobile, tablet)
  creative_id   STRING
  session_id    STRING

-- Deduplicated click events (Kafka topic, after dedup filter)
Topic: ad_clicks_deduped (same schema)

-- Hourly aggregates (ClickHouse)
TABLE click_agg_hourly (
  ad_id         String,
  advertiser_id String,
  hour_bucket   DateTime,   -- truncated to hour
  clicks        UInt64,
  impressions   UInt64
)
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(hour_bucket)
ORDER BY (advertiser_id, ad_id, hour_bucket)

-- Daily budget consumption (ClickHouse materialized view)
TABLE budget_daily (
  advertiser_id String,
  date          Date,
  clicks        UInt64,
  spend_usd     Decimal(18,6)
)
ENGINE = SummingMergeTree()
ORDER BY (advertiser_id, date)
```

**5. High-level architecture**
```
Browser / App (ad SDK)
        | POST /v1/clicks (fire-and-forget, HTTPS)
        v
+---------------------------------------------------------------+
|               Click Collector (stateless, 50+ servers)         |
|  Validates, normalizes, assigns server-side timestamp          |
|  Produces to Kafka topic "ad_clicks"                          |
+---------------------------------------------------------------+
                         |
                    +----v----------------------------------------------+
                    |           Kafka Cluster (ad_clicks topic)          |
                    |  Partitioned by ad_id; 30-day retention            |
                    +----+----------------------------------------------+
                         | consume
        +----------------v----------------------------------------------------+
        |              Flink Streaming Job (Dedup + Aggregate)                  |
        |                                                                        |
        |  Stage 1: Deduplication                                                |
        |    Per event: check Redis SET "ded:{user}:{ad}:{hour}"               |
        |    SETNX -> if key already exists, drop event                        |
        |    Else mark + forward to deduped topic                              |
        |                                                                        |
        |  Stage 2: Windowed Aggregation                                         |
        |    Tumbling window: 1-minute counts per (ad_id)                       |
        |    Sliding window: 1-hour counts (for budget check)                   |
        |    Watermark: event time; allow 2-hr late events                      |
        |    Late events trigger aggregate correction                            |
        |                                                                        |
        |  Stage 3: Sink                                                         |
        |    Write minute-aggregates -> ClickHouse (ad_clicks_1min)             |
        |    Write hour-aggregates -> ClickHouse (click_agg_hourly)             |
        |    Write budget alerts -> Kafka "budget_alerts" topic                 |
        +----------------+----------------------------------------------------+
                         |
              +----------v-------------------------------------------------+
              |         ClickHouse OLAP Cluster                             |
              |  click_agg_hourly (SummingMergeTree)                        |
              |  Replicated across 3 nodes; sharded by ad_id hash           |
              |  Dashboard queries: sub-second for 7-day range              |
              +----------+-------------------------------------------------+
                         |
              +----------v-------------------------------------------------+
              |     Dashboard API (stateless query layer)                   |
              |  Query ClickHouse; cache popular range queries              |
              |  in Redis with 60s TTL                                      |
              +------------------------------------------------------------+

              +--------------------------------------------------+
              |    Budget Alert Service                           |
              |  Consumes "budget_alerts" Kafka topic             |
              |  Calls Ad Server API to pause overspent ads       |
              |  Sends email/push to advertiser                   |
              +--------------------------------------------------+
```

**6. Deep dive**

**Lambda vs Kappa architecture**
Lambda architecture: two parallel pipelines — a speed layer (stream processing for real-time approximate counts) and a batch layer (periodic exact recomputation over raw data). Query layer merges results. Pro: batch layer ensures accuracy; speed layer gives freshness. Con: complex — two codebases to maintain; batch and stream logic can diverge. Kappa architecture: single stream processing pipeline; raw events are replayed from Kafka (30-day retention) when reprocessing is needed. Pro: one codebase; simpler operations. Con: Kafka retention cost; reprocessing is slower than batch MR. Chosen: Kappa — Flink stream processing + Kafka replay is sufficient; ClickHouse SummingMergeTree handles late-arrival corrections by re-merging.

**Count-min sketch for approximate real-time counts**
When 500 K events/sec makes exact in-memory counting infeasible for the full space of (ad_id, hour) tuples, a count-min sketch provides approximate counts using O(w*d) memory where w=width, d=depth. For w=10K, d=5: 50K counters * 8 bytes = 400 KB per sketch. Error bound: estimate <= true_count + epsilon*N with probability 1 - delta where epsilon=e/w, delta=e^(-d). For w=10K, d=5: error <=0.027% of total events. Used in Flink for real-time budget-remaining estimates; exact ClickHouse aggregates are used for billing.

**Late-arriving events handling with watermarks**
Flink tracks event-time watermarks: a monotonically increasing timestamp that represents "we have seen all events with timestamp <= watermark." Watermark = max(event.timestamp seen) - 2 hours (allowed lateness). Events arriving within the 2-hour window after their hour-bucket's window close time still update the aggregate. After the window closes beyond the lateness threshold, a "late" event can still correct the aggregate: Flink re-triggers the window computation and overwrites ClickHouse with the corrected count. For billing (exact counts), always use the 24-hr-delayed finalized aggregate, not the real-time one.

**7. Data flow** — Click event processing
1. User clicks ad. Ad SDK fires POST /v1/clicks with {adId, userId, timestamp, ...}.
2. Click Collector validates fields, assigns server_timestamp, produces Avro message to Kafka `ad_clicks` partition = hash(ad_id) % 120.
3. Flink Dedup stage reads event. Computes dedup key = "ded:user123:ad456:2026061309" (hour bucket). Redis SETNX: if returns 0 (key exists), drop event. If returns 1, forward to deduped stream.
4. Flink Aggregation stage: adds event to tumbling 1-min window for (ad456, 2026-06-13T09:00). Window fires at :01. Emits {ad_id:"ad456", minute:"2026-06-13T09:00", clicks:347}.
5. Flink sinks 1-min aggregate to ClickHouse via JDBC bulk insert.
6. ClickHouse SummingMergeTree merges: click_agg_hourly for (ad456, 2026-06-13T09:00) accumulates minute rows.
7. Budget check: Flink also maintains a sliding 24-hr window per advertiser_id. When spend crosses 95% of daily budget, emits alert to Kafka `budget_alerts`.
8. Budget Alert Service consumes, calls Ad Server to soft-pause advertiser's campaigns.
9. Dashboard query: advertiser queries GET /aggregates/clicks?adId=ad456&granularity=hour. API queries ClickHouse: `SELECT hour_bucket, SUM(clicks) FROM click_agg_hourly WHERE ad_id='ad456' AND hour_bucket BETWEEN ... GROUP BY hour_bucket`. Returns in ~200 ms.

**8. Scaling**
- **10x events (1.15 M/sec):** Scale Kafka to 30+ brokers; scale Flink to 100+ parallel tasks; Redis Cluster for deduplication; ClickHouse sharding across 10+ nodes.
- **100x (11.5 M/sec):** Multi-datacenter Kafka with mirroring; hierarchical aggregation (per-DC aggregation -> global merge); abandon per-event dedup (too expensive) -> switch to probabilistic sampling + statistical correction.
- **1000x:** Pre-aggregate at edge (CDN-level click counting before sending to origin); only send pre-aggregated minute-counts to central pipeline; accept higher approximation error.

**9. Trade-offs**
- **Lambda vs Kappa:** Kappa chosen — one pipeline to maintain; ClickHouse SummingMergeTree handles corrections from late events; Kafka 30-day retention enables full replay if bugs found.
- **Exact vs approximate real-time counts:** Count-min sketch for real-time budget estimates (acceptable ~0.03% error); exact ClickHouse aggregates for billing (finalized 24 hrs after day end). Never approximate billing data.
- **Dedup window size:** 1-hour window balances accuracy vs Redis memory. A smaller window misses multi-click fraud within the window. A larger window requires more Redis memory and increases false-dedup risk for legitimate repeat visits across days.

**10. Follow-up questions**

**Q: How do you handle click fraud (bots)?**
A: Multi-layer: (1) IP velocity check: >100 clicks/min from one IP -> block (Redis counter). (2) User-agent analysis: headless browser signatures. (3) ML model: features include click-to-impression ratio, geographic anomaly, device fingerprint, time distribution. (4) Honeypot ads: invisible ads that legitimate users can't click — any click is a bot. (5) Post-hoc analysis: daily job that invalidates bot-detected clicks and adjusts billing.

**Q: How do you ensure click data is not lost if the Click Collector crashes?**
A: Click Collector is stateless. Kafka producers use acks=all (wait for all ISR replicas to confirm). The Collector itself writes nothing to disk — Kafka is the durable store. If a Collector instance crashes mid-batch, the producer's retry logic re-sends unacknowledged messages. Duplicate events are handled by the deduplication stage.

**Q: How do you backfill aggregates when a bug in Flink produces wrong counts?**
A: (1) Fix the Flink job. (2) Reset Kafka consumer group offset to 30 days ago (or to the point before the bug was introduced). (3) Clear the affected ClickHouse partitions: `ALTER TABLE click_agg_hourly DROP PARTITION '202606'`. (4) Replay events through the fixed Flink job. ClickHouse partitions are rebuilt from clean data. This is the key advantage of Kappa architecture + Kafka retention.

**Q: ClickHouse SummingMergeTree — how does it handle corrections?**
A: SummingMergeTree merges rows with the same primary key by summing numeric columns. To correct a count: insert a row with the delta (negative delta for downward correction). For example, if we over-counted by 100: INSERT (ad456, 2026-06-13T09:00, clicks=-100). ClickHouse merges it with existing rows on background merge. For billing finalization, run `OPTIMIZE TABLE click_agg_hourly FINAL` to force merge before extracting billing figures.

---

## 24 — Google Maps / Location & Routing
**Asked at:** Uber, Google, Lyft, Microsoft  **Difficulty:** 🔴  **Level:** Senior

**1. Requirements**
*Functional:*
- Display a navigable map with tiles at multiple zoom levels
- Search for places by name, address, category (restaurants, ATMs)
- Get driving / walking / transit directions between two points with ETA
- Real-time traffic layer: show congestion on road segments
- Driver/delivery tracking: update and query current location of a driver (Uber use case)
- Offline maps: download a region for offline use
- Street View / satellite imagery layer

*Non-functional:*
- Scale: 1 B MAU; 20 M DAU; 5 M concurrent navigation sessions; 1 M driver location updates/sec
- Latency: tile load < 100 ms; route < 500 ms; driver location query < 100 ms
- Availability: 99.999% (navigation failure is safety-critical)
- Map data update: road changes reflected within 24 hours globally

**2. Back-of-envelope estimation**
- Map tiles: world map at zoom 18 = ~4.5 T tiles; each tile 50 KB -> ~225 PB (not stored all — only populated areas + lower zooms); in practice ~10 PB for all zoom levels
- Tile serving: 20 M DAU; each user loads ~200 tiles/session = 4 B tile requests/day = 46 K tile QPS; peak 5x = 230 K QPS -> CDN-served
- Route queries: 20 M DAU * 3 routes/day = 60 M routes/day = 694 route QPS
- Driver location updates: 1 M drivers * 1 update/4 sec = 250 K writes/sec to Redis
- Location reads: 5 M concurrent passengers checking driver location every 3 s = 1.67 M reads/sec -> Redis geospatial index
- Road graph: 50 M road segments worldwide; each node 16 bytes, edge 32 bytes = ~3.2 GB total graph — fits in RAM per routing server

**3. API design**
```
# Tile API (slippy map standard)
GET /tiles/{z}/{x}/{y}.png          -- raster tile (PNG)
GET /tiles/{z}/{x}/{y}.pbf          -- vector tile (Protobuf)
Response: binary tile data; CDN-cached; Cache-Control: max-age=86400

# Place search
GET /places/search?q=pizza&lat=37.7749&lng=-122.4194&radius=2000
Response: { places:[{ placeId, name, address, lat, lng, category, rating }] }

# Directions / routing
GET /directions?origin=37.77,-122.41&destination=37.33,-121.88&mode=driving&departAt=now
Response: {
  routes: [{
    distance: 82000,    // meters
    duration: 3840,     // seconds
    polyline: "encoded_polyline",
    steps: [{ instruction, distance, duration, maneuver }]
  }]
}

# Driver location update (Uber-like)
PUT /drivers/{driverId}/location
Body: { lat, lng, heading, speed, timestamp }
Response: 200 OK

# Find nearby drivers
GET /drivers/nearby?lat=37.77&lng=-122.41&radius=2000&type=uberX
Response: { drivers:[{ driverId, lat, lng, eta, vehicleType }] }

# ETA estimate
GET /eta?origin=37.77,-122.41&destination=37.33,-121.88
Response: { etaSeconds: 720, distanceMeters: 12000 }
```

**4. Data model / schema**
```
-- Road network graph (in-memory on routing servers; also stored in PostGIS)
nodes (road intersections)
  node_id   BIGINT PK
  lat       DOUBLE
  lng       DOUBLE
  geohash6  CHAR(6)     -- for spatial filtering
  INDEX (geohash6)

edges (road segments)
  edge_id      BIGINT PK
  from_node    BIGINT FK nodes
  to_node      BIGINT FK nodes
  distance_m   INT
  speed_limit  SMALLINT    -- km/h
  road_class   ENUM(motorway, trunk, primary, secondary, residential)
  oneway       BOOLEAN
  INDEX (from_node)

-- Real-time traffic weights (updated every 1 min from probe data)
traffic_weights  (Redis sorted set or in-memory on routing servers)
  edge_id -> current_travel_time_secs (replaces speed_limit * distance)

-- Place index (Elasticsearch)
places
  place_id    TEXT
  name        TEXT (full-text indexed)
  category    TEXT
  address     TEXT
  location    GEO_POINT (lat, lng)
  rating      FLOAT
  INDEX: full-text on name, geo on location

-- Driver locations (Redis, NOT persisted to DB for location history)
Redis GEOADD "drivers:online" lng lat driverId
Redis GEOPOS / GEORADIUS for proximity queries
Expire key after driver goes offline (TTL 60s refreshed on each update)

-- Map tiles (object storage + CDN)
S3/GCS key: tiles/{z}/{x}/{y}/{style}.pbf
CloudFront CDN: global PoPs serve cached tiles
TTL: 24h for frequently updated areas; 1 week for static terrain
```

**5. High-level architecture**
```
Mobile App / Browser
  | Tile requests (static assets)  | API requests (routing, search, location)
  v                                v
+--------------------+    +-------------------------------------------+
|  CloudFront CDN    |    |  API Gateway (geo-load-balanced)           |
|  200+ PoPs globally|    |  Routes to nearest regional cluster       |
|  Serves 99% tiles  |    +------------------+------------------------+
+--------------------+                       |
                                             |
              +------------------------------v--------------------------+
              |              Regional API Cluster                        |
              |                                                          |
              |  +--------------+  +----------------+                   |
              |  |  Tile Service|  |  Place Search  |                   |
              |  |  Reads S3    |  |  (Elasticsearch)|                  |
              |  |  on CDN miss |  +----------------+                   |
              |  +--------------+                                        |
              |  +-------------------------------------------------------+|
              |  |           Routing Service                             ||
              |  |  Holds full road graph in RAM (3.2 GB)               ||
              |  |  A* algorithm with real-time traffic weights          ||
              |  |  Weighted graph updated every 1 min via Redis        ||
              |  +-------------------------------------------------------+|
              |  +-------------------------------------------------------+|
              |  |      Location Service (Driver tracking)               ||
              |  |  Accepts PUT /drivers/{id}/location                  ||
              |  |  GEOADD to Redis geospatial index                    ||
              |  |  Broadcast update to Kafka -> passenger apps         ||
              |  +-------------------------------------------------------+|
              +-----------------------------------------------------------+
                                        |
              +-------------------------v-------------------------------+
              |               Data Stores                               |
              |  Redis Cluster: driver geospatial index (250K writes/s) |
              |  Elasticsearch: place index (full-text + geo)           |
              |  PostGIS (PostgreSQL): road graph (source of truth)     |
              |  S3/GCS: map tiles (10 PB), satellite imagery          |
              |  ClickHouse: traffic probe analytics                    |
              +----------------------------------------------------------+
                                        |
              +-------------------------v-------------------------------+
              |                 Offline Pipeline                        |
              |  OSM / HERE / TomTom -> ETL -> PostGIS + tiles         |
              |  Traffic probe data (GPS breadcrumbs) -> edge weights  |
              |  Runs nightly; updated edges pushed to routing servers  |
              +---------------------------------------------------------+
```

**6. Deep dive**

**Geohash encoding — worked example**
Geohash encodes a (lat, lng) pair as a short alphanumeric string where common prefix = geographic proximity. Algorithm: interleave binary representations of longitude and latitude bits, then base32-encode. Example: San Francisco (lat=37.7749, lng=-122.4194). Step 1: lng range [-180, 180]. -122.4194 -> bisect 20 times: 0=[-180,0], 1=[0,180] -> starts with 0 (lng < 0). Continue bisecting: binary string for lng ~= 01001010110110001101. Step 2: lat range [-90, 90]. 37.7749 -> binary ~= 10110001100011011001. Step 3: Interleave (lng bit, lat bit): 001101000101100010... -> first 30 bits. Step 4: Group into 5-bit chunks -> base32 characters. Result: "9q8yy" (level 5, ~4.9 km x 4.9 km). Level 6 ("9q8yyz") -> ~1.2 km x 0.6 km. Level 8 -> ~38 m x 19 m (precise enough for street-level search). For nearby driver search: query geohash at level 6 and the 8 neighboring geohash cells (to avoid boundary misses).

**Geohash vs Quadtree for spatial indexing**
Geohash: fixed grid cells at each level; cells are rectangular, varying in size by latitude; easy to compute (pure string prefix); stored as indexed column in DB or Redis. Weakness: boundary problem (two nearby points may have very different geohash strings if they straddle a cell boundary — solved by querying 9 cells). Quadtree: adaptive subdivisions — a quad subdivides only when it has more than k points (typically k=100 for a leaf node). This concentrates resolution in dense areas (cities) and keeps coarse resolution for sparse areas (oceans). Better than geohash for highly non-uniform point distributions (like real-world POIs or drivers). S2 geometry (Google): sphere-based, not flat; cells have more uniform areas than geohash; used in production Google Maps for its improved locality guarantees.

**ETA computation with A* on road graph**
Dijkstra's algorithm finds shortest path in O((V + E) log V). A* adds a heuristic h(v) = straight-line distance(v, destination) / max_speed. This reduces explored nodes from the full graph to a directed frontier toward the destination. For a city-level route: ~10 K nodes explored vs 50 M total. Real-time traffic: instead of distance/speed_limit as edge weight, use current_travel_time (from probe vehicles GPS breadcrumbs averaged over last 5 min). Updated edge weights pushed to routing servers via Redis pub/sub every 60 s. Historical traffic patterns (day-of-week, hour) used when real-time data is sparse (late night).

**7. Data flow** — "Get directions from A to B"
1. App calls GET /directions?origin=37.77,-122.41&destination=37.33,-121.88&mode=driving.
2. API Gateway routes to nearest regional Routing Service.
3. Routing Service: snap origin/destination to nearest road nodes using geospatial index (find node within 100 m radius).
4. Load current edge weights from Redis (or in-memory cache refreshed every 60 s).
5. Run A* on road graph: explored ~8,000 nodes, found optimal path. Total route: 22 edges, 12.4 km, 18 min with current traffic.
6. Encode path as polyline (Google Encoded Polyline Algorithm). Generate turn-by-turn instructions.
7. Return response. Client decodes polyline, draws route on map tiles.
8. Client begins navigation: every 5 s, re-evaluate remaining route with updated traffic (re-run A* from current position).

**8. Scaling**
- **10x routes (7K QPS):** Scale Routing Service horizontally; route servers are stateless (road graph is read-only, loaded from S3 at startup); add more route server instances behind LB.
- **100x driver updates (25 M/sec):** Partition Redis geospatial index by geographic region (geohash level-4 cells); each Redis node handles a geographic shard; route update to correct shard via consistent hash on driver's current geohash.
- **1000x (global Google Maps scale):** Geo-distributed clusters (NA, EU, APAC, etc.); road graph partitioned by country/region; routing queries that cross regions use a hierarchical routing algorithm (precompute highway-level routes between partitions, then local routes within).

**9. Trade-offs**
- **Raster vs vector tiles:** Vector tiles chosen for client-side rendering — smaller size (Protobuf vs PNG), client can rotate/style dynamically, supports accessibility features. Raster tiles simpler to serve but fixed style, larger size, no client customization.
- **A* vs Dijkstra for routing:** A* always chosen — heuristic dramatically reduces search space (10x fewer nodes explored for typical city routes). Only falls back to Dijkstra when destination is unclear (multi-destination queries).
- **Redis geo index vs PostGIS for driver locations:** Redis chosen for driver locations — in-memory, sub-millisecond, built-in GEORADIUS; 1 M driver updates/sec is not feasible for PostGIS disk writes. PostGIS is used as the source of truth for road network and places (not for ephemeral driver locations).

**10. Follow-up questions**

**Q: How do you keep map tiles fresh when roads change?**
A: Map edit pipeline: road changes submitted -> validated -> applied to PostGIS road graph -> trigger tile regeneration for affected tile coordinates at all zoom levels. Tile generation is a batch job (MapReduce over changed areas). Updated tiles are pushed to S3 and CDN cache is invalidated with a versioned URL or `Cache-Control: no-cache` + CDN purge. High-priority changes (new highway) go through an accelerated pipeline; minor changes batched nightly.

**Q: How do you route for very long distances (e.g., cross-country)?**
A: Hierarchical routing: (1) Precompute a highway-level contracted graph (Contraction Hierarchies algorithm — precompute shortcut edges that bypass intermediate nodes). (2) For long routes: use A* on the contracted highway graph first (fast, thousands of nodes instead of millions); then expand with local roads at origin and destination. Contraction Hierarchies reduce query time from seconds to milliseconds for cross-continent routes.

**Q: How do you handle the "map too big for one server" problem?**
A: Road graph is partitioned by geographic bounding box (e.g., US West, US East, EU, APAC). Each partition's routing server holds its sub-graph. Cross-partition routing uses a tier-0 router that stitches together routes from partition boundary to partition boundary, then fills in local segments.

**Q: How does ETA account for traffic signals and turns?**
A: Each edge in the road graph has turn cost metadata: left turn penalty (e.g., 15 s), right turn (5 s), U-turn (60 s), traffic signal (add avg wait time derived from historical probe data for that intersection). These penalties are added to edge weights in A* as the algorithm considers each neighbor node.

---

## 25 — Live Streaming / Twitch
**Asked at:** Amazon (Twitch), Meta, YouTube  **Difficulty:** 🟡  **Level:** SDE-1/2

**1. Requirements**
*Functional:*
- Streamers broadcast live video from a desktop/mobile encoder
- Viewers can watch live streams with low latency (< 30 s behind live for broadcast; < 3 s for interactive)
- Video is transcoded to multiple quality levels (1080p, 720p, 480p, 360p) for Adaptive Bitrate (ABR)
- Live chat: viewers send and receive messages in real time; channel-level fan-out
- Viewer count displayed on stream; updated every 10 s
- Stream recording: save VOD (video on demand) for playback after stream ends
- Stream discovery: browse live channels by category, viewer count, game

*Non-functional:*
- Scale: 10 M concurrent viewers; 100 K concurrent streamers; peak chat 100 K messages/sec per popular channel
- Latency: HLS broadcast latency ~20-30 s acceptable; Low-latency HLS ~3-5 s for interactive streams
- Availability: 99.99% — stream interruption directly impacts streamer revenue
- Video quality: auto-adapt to viewer bandwidth (ABR); smooth startup < 2 s
- Storage: 1 TB/hr per 1080p60fps stream; 100 K streamers * 4 hrs avg = 400 PB raw/day (transcode + compress -> ~40 PB/day for HLS segments)

**2. Back-of-envelope estimation**
- 100 K streamers; avg bitrate 6 Mbps ingest = 600 Gbps ingest bandwidth
- Transcoded output: 4 quality levels * avg 3 Mbps = 12 Mbps per stream * 100 K streams = 1.2 Tbps transcoder output bandwidth
- Viewer bandwidth: 10 M viewers * avg 4 Mbps = 40 Tbps outbound — must be CDN-served; cannot come from origin
- HLS segment: 2-second segments; 10 M viewers requesting new segment every 2 s = 5 M requests/sec — CDN handles this easily
- Chat: 100 K messages/sec across all channels; avg message 200 bytes = 20 MB/sec; fan-out: avg channel 1K viewers receiving each message = 100 M message deliveries/sec
- Viewer count: 10 M concurrent viewers across 100 K channels; update every 10 s = 1 M updates/sec to viewer counter service
- VOD storage: 100 K streamers * 4 hrs * 1.5 GB/hr (compressed) = 600 TB/day

**3. API design**
```
# Streamer starts broadcasting (returns RTMP ingest URL)
POST /v1/streams
Headers: Authorization: Bearer <streamer_token>
Body: { title, categoryId, language, lowLatencyMode:true }
Response: { streamId, rtmpUrl:"rtmp://ingest.twitch.tv/live/{stream_key}", playbackUrl }

# Get stream info
GET /v1/streams/{streamId}
Response: { streamId, title, streamerId, viewerCount, startedAt, category, hlsUrl }

# Browse live streams
GET /v1/streams?category=gaming&sort=viewer_count&limit=20&cursor=...
Response: { streams:[...], nextCursor }

# Chat: send message
POST /v1/streams/{streamId}/chat
Body: { message: "PogChamp", emoticons:["PogChamp"] }
Response: 202 Accepted

# Chat: WebSocket subscription
WS  /v1/streams/{streamId}/chat/ws
RECV: { userId, username, message, badges, timestamp, color }

# VOD playback
GET /v1/vods/{vodId}
Response: { vodId, hlsUrl, duration, recordedAt, thumbnailUrl }
```

**4. Data model / schema**
```
-- Stream metadata (PostgreSQL)
streams
  stream_id     UUID PK
  streamer_id   UUID FK users
  title         TEXT
  category_id   UUID FK categories
  status        ENUM(live, ended)
  started_at    TIMESTAMPTZ
  ended_at      TIMESTAMPTZ
  rtmp_key      TEXT UNIQUE    -- secret ingest key
  vod_id        UUID FK vods   -- populated when stream ends
  INDEX (status, started_at DESC) for discovery
  INDEX (category_id, status)

-- Chat messages (Cassandra, append-only)
chat_messages
  stream_id   UUID  (partition key)
  sent_at     TIMEUUID (clustering key, DESC)
  user_id     UUID
  username    TEXT
  message     TEXT
  badges      LIST<TEXT>
  color       TEXT
  PRIMARY KEY ((stream_id), sent_at)  -- range query: last N messages

-- Viewer count (Redis)
INCR  "viewers:{streamId}"    -- on join
DECR  "viewers:{streamId}"    -- on leave
GET   "viewers:{streamId}"    -- for display (updated every 10s snapshot to DB)

-- HLS segments (S3 / GCS)
  Key: segments/{streamId}/{rendition}/{sequenceNum}.ts
  Playlist: segments/{streamId}/master.m3u8
             segments/{streamId}/1080p/playlist.m3u8
             segments/{streamId}/720p/playlist.m3u8  ...

-- VOD metadata
vods
  vod_id        UUID PK
  stream_id     UUID FK streams
  hls_url       TEXT    -- points to final assembled HLS on S3
  duration_secs INT
  size_bytes    BIGINT
  created_at    TIMESTAMPTZ
```

**5. High-level architecture**
```
Streamer (OBS / mobile encoder)
    | RTMP stream (video + audio)
    v
+------------------------------------------------------------------+
|               RTMP Ingest Service (100K+ connections)             |
|  * Authenticates stream_key                                       |
|  * Accepts RTMP connection; buffers incoming video chunks         |
|  * Pushes raw stream to Transcoder queue (Kafka or direct)       |
|  * Geo-distributed ingest PoPs to minimize streamer upload lag   |
+-------------------------+-----------------------------------------+
                          | raw video chunks
                          v
+------------------------------------------------------------------+
|                   Transcoder Fleet                                |
|  * FFmpeg-based workers (GPU-accelerated)                        |
|  * Input: raw H.264/HEVC stream                                  |
|  * Output: HLS segments at 1080p60, 720p60, 480p30, 360p30      |
|  * Segment duration: 2 s (standard HLS) or 0.5 s (LL-HLS)      |
|  * Pushes segments to S3; updates HLS playlist files            |
|  * One transcoder per active stream (dedicated or shared)       |
+-------------------------+-----------------------------------------+
                          | HLS segments -> S3
                          v
+------------------------------------------------------------------+
|                 S3 / Object Storage (Origin)                      |
|  Stores HLS .ts segment files + .m3u8 playlist files             |
|  Playlist files updated every 2 s (new segment appended)        |
|  CDN pulls from here on cache miss                               |
+-------------------------+-----------------------------------------+
                          | CDN pull
                          v
+------------------------------------------------------------------+
|             CDN (Akamai / CloudFront, 200+ PoPs globally)        |
|  Caches .ts segment files (popular segments served from edge)    |
|  Playlist (.m3u8) cached for 2 s only (must stay fresh)         |
|  Viewers pull from nearest CDN PoP                               |
|  10 M viewers @ 40 Tbps — CDN absorbs all bandwidth             |
+-------------------------+-----------------------------------------+
                          | HLS playback (HTTP byte-range)
                          v
                  Viewer (HLS player, ABR)
                  Requests new playlist every 2 s
                  Downloads next segment
                  Auto-adjusts quality based on bandwidth

CHAT SYSTEM (separate path):
Viewer --WS--> Chat Gateway (100s of servers) ---> Redis Pub/Sub (per channel)
                                               +--> Cassandra (persist messages)
Chat Gateway subscribes to channel's Redis pub/sub topic
Fan-out: Viewer sends -> Chat Gateway -> Redis PUBLISH -> all other Chat Gateways
         subscribed to same channel -> forward to their WS connections

VIEWER COUNT:
Chat Gateway INCR/DECR Redis counter on join/leave
Viewer Count Service reads Redis every 10s -> broadcasts to streamers + CDN metadata
```

**6. Deep dive**

**RTMP ingest -> HLS transcoding pipeline**
RTMP (Real-Time Messaging Protocol) is a low-latency TCP-based protocol used by encoders (OBS Studio) to push live video to ingest servers. It carries H.264 video + AAC audio in interleaved chunks. The Ingest Service receives these, authenticates the stream key, and immediately forwards to the Transcoder. The Transcoder (FFmpeg with NVENC GPU acceleration) reads the incoming stream in real time. It produces multiple output renditions simultaneously (same input, multiple output resolutions/bitrates). Every 2 seconds, it finalizes a segment (.ts file — MPEG-TS container) for each rendition and uploads to S3. It then updates the HLS master playlist (master.m3u8) and per-rendition playlists (e.g., 1080p/playlist.m3u8) to include the new segment. The playlist is a rolling window of the last 5 segments (10 s of buffer). Viewer players poll the playlist every 2 s, download the newest segment not yet played.

**Adaptive Bitrate (ABR) streaming**
The HLS player measures download throughput continuously. If the last segment downloaded at 8 Mbps and the current rendition requires 6 Mbps -> stay at 1080p. If bandwidth drops to 3 Mbps -> switch to 720p (4 Mbps). Segment boundaries are switching points. The master playlist lists all renditions with their bandwidth and resolution — the player chooses. This ensures smooth playback on variable network connections (mobile users, congested networks) without buffering.

**Chat at scale — fan-out via pub/sub**
A popular streamer has 500 K concurrent viewers all watching the same channel. When one viewer sends a chat message: (1) HTTP POST to Chat Gateway. (2) Chat Gateway publishes to Redis pub/sub channel "chat:{streamId}". (3) All Chat Gateway instances subscribed to this channel receive the message. (4) Each instance forwards to its connected WebSocket clients. Fan-out = 1 publish -> N gateway instances -> M viewers per instance. For 500 K viewers across 500 gateway instances = 1000 viewers/instance -> 1 publish fans out to 500 instances -> each forwards to 1000 WS connections. Redis pub/sub handles this at ~100 K messages/sec per channel. For mega-events (1 M+ viewers): use a hierarchical pub/sub or dedicate Redis nodes per channel.

**Viewer count aggregation — HyperLogLog**
Exact viewer counting at scale (10 M concurrent across 100 K channels) is memory-intensive if using exact sets. HyperLogLog (HLL) provides approximate distinct count using O(1.5 KB) memory per counter with ~0.81% standard error. For Twitch's use case: Redis PFADD "viewers_hll:{streamId}" userId on each join; PFCOUNT "viewers_hll:{streamId}" for current count. This handles de-duplication (same user on two devices counted once) and gives accurate approximate counts for large streams. For small streams (<1K viewers), exact INCR/DECR counters are fine.

**7. Data flow** — Viewer watching a live stream
1. Viewer opens stream page. Browser requests GET /v1/streams/{streamId} -> gets hlsUrl.
2. HLS player fetches master.m3u8 from CDN (Cache-Control: max-age=2). Gets list of renditions.
3. Player selects 1080p initially. Fetches 1080p/playlist.m3u8 (CDN-cached for 2 s). Gets last 5 segment URLs.
4. Player fetches the 3 newest .ts segments it hasn't played. These are ~6 s of video. Begins playback.
5. Every 2 s: player refetches playlist. New segment at the end. Downloads it. Smooth continuous playback.
6. Viewer's bandwidth drops. Player detects segment download time exceeds segment duration. Switches to 720p. Requests 720p/playlist.m3u8.
7. Viewer opens chat. WS connection to Chat Gateway. Gateway subscribes to Redis "chat:{streamId}".
8. Another viewer types a message. POST to Chat Gateway -> Redis PUBLISH -> viewer's Chat Gateway receives -> forwards over WS -> viewer sees message in < 200 ms.
9. Viewer joins: Chat Gateway increments Redis "viewers:{streamId}". Viewer Count Service reads counter every 10 s and pushes update to Streamer's dashboard via WS.

**8. Scaling**
- **10x streamers (1 M):** Scale Ingest Service horizontally (each server handles ~500 concurrent streams); scale Transcoder fleet with auto-scaling based on active stream count; S3 scales automatically; CDN scales automatically.
- **100x viewers (1 B):** CDN handles it — add more PoPs; optimize segment size (2 s standard) for high cache hit rate; pre-warm CDN for scheduled events (predictable load).
- **100x chat (10 M messages/sec):** Partition Redis pub/sub by channel; introduce message rate limiting per channel per user; for mega-events, switch to dedicated real-time messaging clusters (Ably, Pusher at scale); drop messages gracefully under extreme load (viewers accept occasional missed chat messages).

**9. Trade-offs**
- **HLS vs WebRTC for delivery:** HLS chosen for broadcast (> 30 s latency acceptable for most streams; CDN-friendly, scales to billions of viewers). WebRTC chosen for interactive/low-latency streams (< 1 s latency, peer-to-peer or SFU, harder to scale to millions). Twitch uses Low-Latency HLS as a middle ground (~3 s with 0.5 s segments).
- **RTMP vs SRT for ingest:** RTMP is widely supported (all encoders); SRT (Secure Reliable Transport) is newer, handles packet loss better, preferred for unstable connections. Twitch supports both; OBS defaults to RTMP.
- **Redis pub/sub vs Kafka for chat fan-out:** Redis pub/sub is simpler and lower latency (< 10 ms) but no persistence and no replay. Kafka provides durability and replay but adds latency. Chat messages are written to Cassandra for history, so Redis pub/sub for live fan-out is sufficient — it's fire-and-forget for real-time delivery; Cassandra is the durable store.

**10. Follow-up questions**

**Q: How do you handle a streamer's encoder disconnecting mid-stream?**
A: Ingest Service detects TCP disconnect. Marks stream as buffering. Holds a 60-second reconnect window — streamer's encoder reconnects and stream resumes. If no reconnect within 60 s, stream is marked as ended. Transcoder flushes its segment buffer, finalizes the HLS playlist with an `#EXT-X-ENDLIST` tag. VOD assembly begins automatically from the recorded segments.

**Q: How do you record a VOD while live streaming simultaneously?**
A: The Transcoder writes segments to S3 in real time. A parallel VOD Assembler watches the S3 prefix for the stream, collecting all .ts segments as they appear. When the stream ends, it creates the final VOD .m3u8 playlist pointing to all segments in order. No post-processing step required — segments are already transcoded. The VOD is available within seconds of stream end.

**Q: How do you handle chat moderation at scale?**
A: (1) Client-side: browser filters known bad words. (2) Server-side: ML classifier scores each message in < 50 ms before fan-out. (3) AutoMod: pre-configured word/phrase block lists per channel. (4) Timeout system: moderators (humans) can ban/timeout users; ban list stored in Redis per channel; Chat Gateway checks on each message. (5) Rate limiting: max 1 message per second per user per channel (Redis token bucket).

**Q: How would you implement a "clip" feature (clip last 30 s of a stream)?**
A: HLS segments are already stored in S3. A clip is created by identifying the last ~15 segments (30 s at 2 s/segment). The Clip Service creates a new .m3u8 playlist file referencing those 15 segments (no re-encoding needed — segments are already transcoded). The clip URL points to this playlist. Clips are stored permanently as VODs in S3. Total clip creation time: < 1 s (just a new playlist file write).

---

## Design Trade-off Cheat Sheet

| Problem | Key decision | Choice | Why |
|---|---|---|---|
| Social feed | Fanout-on-write vs fanout-on-read | Write for <=10K followers; read for celebrities | Write keeps read fast; read avoids amplifying celebrity posts |
| Chat | Storage model | Append-only message table sharded by conversation_id | Chat is write-heavy; reads are always recent; sharding by convo keeps related data together |
| Search/Autocomplete | Trie vs search DB | Trie in memory for <10M terms; Elasticsearch otherwise | Trie is O(L) prefix lookup but hard to distribute; ES scales horizontally |
| Payments | Consistency model | Strong (ACID, single-region DB) | Money: never sacrifice correctness for availability |
| Notifications | Delivery guarantee | At-least-once + idempotent receivers | Best-effort loses messages; exactly-once is too expensive at scale |
| Cache | Eviction policy | LRU for access-frequency workloads, LFU for skewed hotspot workloads | LRU evicts things not used recently; LFU keeps truly popular items |
| URL shortener | DB choice | SQL (PostgreSQL) | Simple key-value but need ACID; Postgres with a single table is fine until 10B URLs |
| Video streaming | CDN strategy | Pull CDN with long TTL for popular; bypass CDN for live | Popular VOD benefits from caching; live can't be cached meaningfully |
| Ride sharing | Location store | Redis GEOADD (in-memory geo index) | Sub-millisecond proximity queries; persistence via AOF |
| Rate limiter | Algorithm | Sliding window counter (Redis + Lua) | Fixed window has boundary spikes; token bucket requires per-user state; sliding window is the best balance |

---

Created **08-system-design-problems.md** — 25 designs covered (8 easy, 12 medium, 5 hard), each with requirements, estimation with real arithmetic, API design, schema with sharding key, full ASCII architecture diagram, deep dives into critical components, step-by-step data flow, scaling strategy up to 1000x, trade-off analysis, and follow-up Q&A.
