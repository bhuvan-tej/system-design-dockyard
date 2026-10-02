# Non-Functional Requirements (NFRs)

## 1. What it is
Functional requirements say **what** the system does ("users can post a photo").
Non-functional requirements say **how well** it must do it — speed, uptime, durability, scale. **NFRs drive the architecture**; two systems with identical features but different NFRs look completely different.

## 2. Why it matters in interviews
Step 1 of the [framework](01-interview-framework.md) is incomplete without NFRs. Every later decision (cache? replicas? strong consistency? multi-region?) should trace back to one of them. Interviewers explicitly score whether you asked.

## 3. The NFRs you must know

### Scalability
Ability to handle growth (users, data, traffic) by adding resources.
- Ask: DAU/MAU, peak vs average, growth rate, read:write ratio, data size.
- Vertical vs horizontal → see [Scaling](../tier-1-core-building-blocks/01-scaling-and-load-balancing.md).

### Latency
Time for one request (client → response).
- Always talk in **percentiles**, not averages:
  - **p50** (median): typical user.
  - **p95 / p99**: tail — the slow requests that users actually notice and complain about.
  - **p99.9**: matters at scale (1 in 1000 of 1B requests = 1M bad experiences).
- **Tail latency amplification**: a page calling 100 services in parallel is as slow as the slowest one → p99 of each becomes p50-ish of the page. Fixes: hedged requests, timeouts, fewer fan-out calls.
- Typical targets: API read < 100–200 ms p99; search/autocomplete < 100 ms; payments can be slower (~1–2 s).

### Throughput
Work done per unit time — **RPS/QPS**, messages/s, MB/s.
Latency and throughput are different: batching raises throughput but can raise latency.

### Availability
Fraction of time the system serves requests successfully.
```
Availability = uptime / (uptime + downtime)
             ≈ MTBF / (MTBF + MTTR)
```
- Improve by raising **MTBF** (mean time between failures — redundancy, testing) or cutting **MTTR** (mean time to recover — auto-failover, fast rollbacks, good alerting).
- Nines table + series/parallel math → see [Estimation](02-back-of-envelope-estimation.md).
- Techniques: redundancy (no SPOF), replication, multi-AZ / multi-region, health checks + failover, graceful degradation, load shedding.

### Reliability vs availability
- **Available** = it responds. **Reliable** = it responds *correctly*, consistently, over time.
- A system returning wrong balances is available but not reliable.

### Fault tolerance & resilience
Keep working (maybe degraded) when components fail.
- Redundancy, replication, retries with backoff, timeouts, circuit breakers, bulkheads, fallbacks.
- **Graceful degradation**: recommendations service down → show popular items instead of an error page.

### Durability
Once acknowledged, data is never lost.
- Replication (sync / quorum), write-ahead logs, backups, cross-region copies.
- S3 advertises **11 nines of durability** (99.999999999%) — different from its availability (99.99%).
- Ask: "Can we ever lose a write?" Chat message → no. View counter → a few is fine.

### Consistency
How up-to-date reads are → see [CAP & Consistency Models](../tier-1-core-building-blocks/05-cap-and-consistency-models.md). Decide **per feature**.

### Security & privacy (mention briefly)
AuthN (who are you — OAuth2/OIDC, JWT) vs AuthZ (what can you do — RBAC/ABAC), TLS in transit, encryption at rest, rate limiting, input validation, PII handling, GDPR data residency.

### Maintainability, observability, cost (mention if time)
- **Observability**: metrics, logs, traces; alert on SLOs.
- **Maintainability**: simple components, clear ownership, easy deploys.
- **Cost**: cheaper storage tiers for cold data, autoscaling, CDN offload.

## 4. SLI / SLO / SLA
| Term | Meaning | Example |
|---|---|---|
| **SLI** — indicator | What you *measure* | % of requests < 200 ms; % successful requests |
| **SLO** — objective | Internal *target* for an SLI | 99.9% of requests succeed over 30 days |
| **SLA** — agreement | *Contract* with customers, with penalties | 99.9% monthly uptime or 10% credit |
| **Error budget** | 100% − SLO | 0.1% ≈ 43 min/month of allowed failure → spend on risky deploys, freeze when exhausted |

Rule: SLA ≤ SLO ≤ what you actually achieve — so you breach your internal target before the contract.

## 5. Turning NFRs into design decisions (the actual skill)
| If the requirement is… | Then the design needs… |
|---|---|
| Read-heavy (100:1) | Cache, read replicas, CDN |
| Write-heavy | Sharding, LSM-based stores, queues to absorb bursts |
| p99 < 100 ms | Cache hot path, avoid cross-region calls, precompute, async non-critical work |
| 99.99% availability | No SPOF, multi-AZ, auto-failover, degrade gracefully |
| Never lose data | Sync/quorum replication, WAL, backups, idempotent retries |
| Strong consistency | Single leader / consensus, transactions — accept higher latency |
| Global users | Multi-region, GeoDNS, CDN, regional data placement |
| Spiky traffic (flash sale) | Queue-based load levelling, autoscaling, rate limiting |
| Huge data (PBs) | Blob storage, tiered/cold storage, partitioning |

## 6. Question checklist to ask in minute 1–5
1. How many users (DAU) and how fast is it growing?
2. Read-heavy or write-heavy? Ratio?
3. Latency target for the main operation?
4. Is availability or consistency more important for the core feature?
5. Can we lose data? Any durability constraints?
6. Single region or global?
7. Any compliance/security requirements (payments, PII)?
8. Average object size? Media involved?

## 7. How to say it
> "Non-functionally: 50M DAU, read-heavy at ~100:1, p99 read latency under 200 ms, 99.99% availability for reads. Writes must be durable. Eventual consistency is fine for the feed, but a user must see their own posts immediately."

## 8. Interview follow-ups
1. Why p99 and not average latency?
2. Availability vs durability — give a system that has high one and low the other.
3. What's an error budget and how does a team use it?
4. Your system has 3 services in series at 99.9% each — overall availability? *(≈ 99.7%)*
5. How would you move from 99.9% to 99.99%? *(remove SPOFs, multi-AZ, faster detection & failover — cut MTTR)*
6. Difference between fault tolerance and high availability?

## 9. Common mistakes
- Listing NFRs as buzzwords ("scalable, reliable, available") without numbers.
- Averages instead of percentiles.
- Confusing availability with durability.
- Never referring back to the NFRs when making design choices.
