# 02 — Back of the Envelope Estimation

> **Status:** Completed 🟢  &nbsp;•&nbsp; **Learned on:** _2026-09-02_

A rough estimate built from a thought experiment plus a few common performance numbers — good enough to size a system in an interview.

---

##  Handwritten Notes
 [My handwritten notes (PDF)](./assets/back-of-the-envelope-handwritten-notes.pdf)

---

##  Key Learnings

### 1. Powers of Two
Use these when converting users, QPS, and bytes. The power-of-two value is close enough to the round number that either one is fine in an estimate.

| Power | Approximate value | Full name | Short name |
|------:|-------------------|-----------|------------|
| 10 | 1 thousand | 1 Kilobyte | 1 KB |
| 20 | 1 million | 1 Megabyte | 1 MB |
| 30 | 1 billion | 1 Gigabyte | 1 GB |
| 40 | 1 trillion | 1 Terabyte | 1 TB |
| 50 | 1 quadrillion | 1 Petabyte | 1 PB |

So `2^10 ≈ 10^3`, `2^20 ≈ 10^6`, `2^30 ≈ 10^9`, and so on. In the Twitter estimate below, `10^6 MB = 1 TB`.

### 2. Latency Numbers
Orders of magnitude matter more than exact digits. A few anchors:

| Operation | Time |
|-----------|------|
| L1 cache reference | 0.5 ns |
| L2 cache reference | 7 ns |
| Main memory reference | 100 ns |
| Compress 1 KB with Zippy | 10,000 ns = 10 μs |
| Send 2 KB over a 1 Gbps network | 20,000 ns = 20 μs |
| Read 1 MB sequentially from memory | 250,000 ns = 250 μs |
| Round trip within the same datacenter | 500,000 ns = 500 μs |
| Disk seek | 10,000,000 ns = 10 ms |
| Read 1 MB sequentially from the network | 10,000,000 ns = 10 ms |
| Read 1 MB sequentially from disk | 30,000,000 ns = 30 ms |
| Send a packet CA → Netherlands → CA | 150,000,000 ns = 150 ms |

**After looking at these:**
- Memory is faster than disk.
- Avoid disk seeks if possible.
- Simple compression is fast.
- Compress data before sending it, when you can.
- Datacenters are far apart, and that round trip is time-consuming (~150 ms coast to coast).

### 3. Availability and SLA
- **High availability** means the system stays operational for a long time.
- It is measured as a percentage. **100% = 0 downtime.**
- **SLA (Service Level Agreement)** is the agreement between the service provider and the customer about the level of service (including that availability) the provider commits to.

How the usual "nines" turn into downtime:

| Availability | Downtime / year | Downtime / day |
|-------------:|-----------------|----------------|
| 99% | 3.65 days | ~14.4 min |
| 99.9% | 8.76 hours | ~1.4 min |
| 99.99% | 52.6 min | ~8.6 sec |
| 99.999% | 5.26 min | ~0.86 sec |

A year is about `365 × 24 × 60 ≈ 526,000` minutes, so each extra nine cuts the allowed downtime by 10×.

### 4. Twitter — QPS and Storage
Interview question: estimate tweet QPS and how much storage the media needs.

**Assumptions**
- Monthly active users = 300 million
- Daily users = 50% of MAU
- Tweets per user per day, on average = 2
- Tweets that contain media = 10%
- Tweets are stored for 5 years

**Daily active users**

```
DAU = 50% of 300M = 150M
```

**Tweet QPS**

```
QPS = 150M × 2 tweets / 24 hr / 3600 sec
    = 300,000,000 / 86,400
    ≈ 3,500
```

**Peak QPS** — assume the peak is about 2× the average:

```
Peak QPS = 2 × 3,500 ≈ 7,000
```

**Average tweet size**
- `tweet_id` = 64 bytes
- text = 140 bytes
- media = 1 MB

Text and the id are negligible next to media, so the storage estimate only counts media.

**Media storage per day**

```
= (daily active users) × (tweets / day) × (10% contain media) × (media size)
= 150M × 2 × 10% × 1 MB
= 30 × 10^6 MB
= 30 TB / day
```

**Over 5 years**

```
= 30 TB × 365 × 5
= 54,750 TB
≈ 55 PB
```

---

##  Open Questions
- Peak was taken as a flat 2×. When is that too crude, and when do you need an hourly traffic curve?
- At what point does tweet text and metadata storage stop being ignorable next to media?
- An SLA of 99.99% allows ~52 minutes a year. How do you turn that into an error budget the team can actually spend?

##  References
- *System Design Interview – An Insider's Guide* (Vol. 1) — Alex Xu
- Latency numbers: Jeff Dean, "Numbers Everyone Should Know" (via the book)
