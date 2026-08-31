# 01 — Scaling

> **Status:** Completed 🟢  &nbsp;•&nbsp; **Learned on:** _2026-08-29_

How a single-server app evolves into a highly-available, large-scale system — one bottleneck at a time.

---

## ✍️ Handwritten Notes
📄 [My handwritten notes (PDF)](./assets/scaling-handwritten-notes.pdf)

---

## 🧠 Key Learnings

### 1. Horizontal vs Vertical Scaling
- **Vertical scaling (scale up)** → add more power (CPU, RAM) to a *single* server.
  - Best for **low traffic** and **simplicity**.
- **Horizontal scaling (scale out)** → add *more servers*.
  - Best for **large-scale apps** with **huge traffic**.

**Limitations of vertical scaling:**
- Impossible to add *unlimited* CPU & RAM to a single server.
- **No failover / backup** — if that one server goes down, the whole website goes down with it.

### 2. Load Balancer
- Sits between users and the web servers; users hit a **public IP**, the LB talks to servers over **private IPs**.
- **Problem without a load balancer** (users connected directly to one server):
  - When the server is offline, users can't reach the site.
  - When many users hit at the same time → **slow response** and **failure to connect**.
- **Solution:** LB distributes traffic across multiple web servers (Server 1, Server 2, …) via their private IPs, so adding a 2nd web server improves availability and capacity.

### 3. Database Replication
- **Master–Slave** setup:
  - **Master** = original DB → handles **write operations** (insert, delete, update).
  - **Slave(s)** = copies → handle **read operations**, getting copies from the master.
- Master replicates changes out to Slave DB 1, Slave DB 2, Slave DB 3, … (**DB replication**).
- **Advantages:**
  - **Better performance** (reads spread across slaves).
  - **Reliability** (data survives on copies).
  - **High availability** (site still serves reads if one DB fails).

### 4. CDN (Content Delivery Network)
- A website serves two kinds of content:
  - **Static content** → images, videos, CSS, JS (rarely changes).
  - **Dynamic content** → changes per request / per user.
- **How a CDN works:** a CDN server (geographically **closest to the user**) caches and serves **static content**.
  - On a **cache miss**, the CDN fetches from the **origin server**, caches it, and returns it.
  - Next user gets it directly from the CDN → faster.
- **Things to consider when using a CDN:**
  - **Cost** — get rid of infrequently used content (don't cache what's rarely served).
  - **Expiry time (TTL)** — not too long, not too short.
  - **CDN fallback / backup** — origin as backup when the CDN fails.
  - **Invalidating files** — via CDN vendor **API** or **versioning** the file URLs.

### 5. Stateless vs Stateful Architecture
- **Session data / state** = data that represents a user's session on the server (used to **identify** and **authenticate** a user).
- **Stateful server:** stores each user's state on a specific server (e.g., Server 1 holds User 1's state).
  - User requests must keep hitting the *same* server → **overhead, not simple**.
- **Stateless server:** does **not** remember client data / store any info.
  - Session state is pushed to a **shared storage** (shared DB/cache); any server can fetch it.
  - **Simple, robust** — any web server can handle any request.

---

## 🏗️ The Full Picture
After applying all of the above, the service looks like:
- **Multiple servers** behind a **Load Balancer**
- **Master–Slave DB** replication
- **Cache / Cache DB**
- **CDN** for static content
- **Stateless** web tier (session state in shared storage)

---

## ❓ Open Questions
- How does the load balancer detect an unhealthy server (health checks)?
- Sync vs async replication — what's the consistency trade-off on slave reads?
- Where exactly does the cache sit relative to the DB (read-through vs cache-aside)?

## 🔗 References
- *System Design Interview – An Insider's Guide* (Vol. 1) — Alex Xu
