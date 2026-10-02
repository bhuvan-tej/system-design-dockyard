# System Design Dockyard

[![SYSTEM DESIGN](https://img.shields.io/badge/SYSTEM%20DESIGN-2563EB?style=for-the-badge)](#roadmap)
[![INTERVIEW PREP](https://img.shields.io/badge/INTERVIEW%20PREP-7C3AED?style=for-the-badge)](INTERVIEW-QUESTIONS.md)
[![JAVA](https://img.shields.io/badge/JAVA-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](implementations/)
[![MERMAID DIAGRAMS](https://img.shields.io/badge/MERMAID%20DIAGRAMS-FF3670?style=for-the-badge&logo=mermaid&logoColor=white)](case-studies/)

Personal reference repo for system design interview prep. Structure and priority order below.

## Folder structure
```
system-design-dockyard/
├── README.md                              ← this file — roadmap + checklist
├── INTERVIEW-QUESTIONS.md                  ← on-demand Tier 1 + Tier 2 scenarios and answer guides
├── notes/
│   ├── _template.md                       ← copy this for every new topic
│   ├── 00-foundations/                    ← interview framework + estimation (read first)
│   ├── tier-1-core-building-blocks/       ← highest priority, one file per topic
│   ├── tier-2-distributed-patterns/       ← secondary depth, usually follow-ups
│   └── tier-3-ai-conditional/             ← only if an AI-facing role is active
├── case-studies/                          ← end-to-end designs (requirements → estimate → diagram → trade-offs)
└── implementations/                       ← small runnable Java components (cap at 3-5)
```

## Roadmap

### 0 — Foundations (do first, revisit often)
- [x] [Interview framework](notes/00-foundations/01-interview-framework.md) — the 45-min structure
- [x] [Back-of-envelope estimation](notes/00-foundations/02-back-of-envelope-estimation.md) — QPS, storage, bandwidth
- [x] [Non-functional requirements](notes/00-foundations/03-non-functional-requirements.md) — latency percentiles, availability, durability, SLI/SLO/SLA
- [x] [Building blocks & trade-offs cheat sheet](notes/00-foundations/04-building-blocks-cheatsheet.md) — one-page revision before interviews

### Tier 1 — Core Building Blocks (asked in nearly every round)
- [ ] [Scaling & Load Balancing](notes/tier-1-core-building-blocks/01-scaling-and-load-balancing.md)
- [ ] [Caching & CDN](notes/tier-1-core-building-blocks/02-caching-and-cdn.md)
- [ ] [Databases — SQL vs NoSQL, Indexing, Replication](notes/tier-1-core-building-blocks/03-databases.md)
- [ ] [Sharding & Consistent Hashing](notes/tier-1-core-building-blocks/04-sharding-and-consistent-hashing.md)
- [ ] [CAP Theorem & Consistency Models](notes/tier-1-core-building-blocks/05-cap-and-consistency-models.md)
- [ ] [Message Queues & Event-Driven Architecture](notes/tier-1-core-building-blocks/06-message-queues-and-event-driven.md)
- [ ] [Rate Limiting](notes/tier-1-core-building-blocks/07-rate-limiting.md)
- [ ] [Networking & Communication — DNS, HTTP, WebSockets, polling, gRPC](notes/tier-1-core-building-blocks/08-networking-and-communication.md)
- [ ] [Unique ID Generation & Idempotency](notes/tier-1-core-building-blocks/09-unique-ids-and-idempotency.md)

> Notes are written; tick a box only after you can explain the topic out loud in ~3 minutes
> and answer the "Interview follow-ups" section without looking.

For Tier 1, first study the concepts, worked example, and diagram. Then use the
**"Say this out loud - interview practice"** section in each note: explain the
sample answer in your own words, state the trade-off, and walk through the failure
question. Java sketches teach the flow; only the files in `implementations/` are
standalone runnable examples.

### Tier 2 — Distributed Patterns (secondary depth, usually follow-ups)
- [ ] [Monolith vs Microservices](notes/tier-2-distributed-patterns/01-monolith-vs-microservices.md)
- [ ] [API Gateway & Service Discovery](notes/tier-2-distributed-patterns/02-api-gateway-and-service-discovery.md)
- [ ] [Saga vs 2-Phase Commit (+ Outbox pattern)](notes/tier-2-distributed-patterns/03-saga-vs-two-phase-commit.md)
- [ ] [Circuit Breaker vs Retry (+ timeouts, backoff, bulkheads)](notes/tier-2-distributed-patterns/04-circuit-breaker-and-retry.md)
- [ ] [Search Architecture (Elasticsearch / OpenSearch, inverted index)](notes/tier-2-distributed-patterns/05-search-architecture.md)
- [ ] [Blob / Object Storage (S3-style)](notes/tier-2-distributed-patterns/06-blob-and-object-storage.md)
- [ ] [Observability (metrics, logs, tracing, SLOs)](notes/tier-2-distributed-patterns/07-observability.md)
- [ ] [Distributed Locks & Leader Election (ZooKeeper / etcd / Redis)](notes/tier-2-distributed-patterns/08-distributed-locks-and-leader-election.md)

Study in the order above after the relevant Tier 1 basics. For each topic, explain
the normal flow, walk through a failure, and answer the follow-ups aloud before
checking it off. Java snippets in these notes are learning flows with explicit
adapter contracts, not standalone applications; runnable examples remain in
`implementations/`.

### Tier 3 — Conditional (only if an AI-facing role is active)
- [ ] RAG Architecture Basics
- [ ] Vector DB Fundamentals
- [ ] AI Observability

### Case Studies (interleave with notes — don't wait until Tier 1 is "done")
| Case study                                            | Do it after                                |
|-------------------------------------------------------|--------------------------------------------|
| [ ] [URL shortener](case-studies/01-url-shortener.md) | Estimation, Databases, Caching, Unique IDs |
| [ ] Rate limiter                                      | Rate Limiting                              |
| [ ] Distributed key-value store                       | Sharding, CAP, Replication                 |
| [ ] Notification system                               | Message Queues                             |
| [ ] Chat system                                       | Networking (WebSockets), Message Queues    |
| [ ] News feed / timeline                              | Caching, Message Queues (fan-out)          |
| [ ] Typeahead / autocomplete                          | Caching, Tries                             |
| [ ] Web crawler                                       | Message Queues, Consistent Hashing         |
| [ ] File storage (Dropbox / Google Drive)             | Blob Storage, Consistency                  |
| [ ] Video streaming (YouTube)                         | CDN, Blob Storage                          |
| [ ] Ride sharing (Uber)                               | Geo-indexing, WebSockets                   |

Case studies are complete design walkthroughs, not full applications. Architecture
and sequence diagrams are embedded directly as Mermaid blocks. Start with the
URL shortener: requirements, estimates, APIs, data model, evolving architecture,
request flows, failures, focused Java, and out-loud interview practice.

### Implementations (Java, runnable with `java <File>.java`)
- [x] [LRU Cache](implementations/LRUCache.java)
- [x] [Token Bucket + Sliding Window Rate Limiter](implementations/RateLimiters.java)
- [x] [Consistent Hash Ring](implementations/ConsistentHashRing.java)

## How to use this repo
1. Read `00-foundations` first — every answer follows that framework.
2. For each Tier 1 or Tier 2 topic: read the note → re-derive the diagrams/code yourself → answer the follow-ups aloud.
3. After 2-3 topics, do the matching case study end-to-end on paper in 45 minutes, then write it up in `case-studies/`.
4. For every design decision, run the loop: **clarify what "good" means → estimate scale → name the bottleneck → state the trade-off you're accepting.** That loop is the actual interview skill — the notes are ammunition for it, not a substitute.

## On-demand interview practice
[Interview question bank](INTERVIEW-QUESTIONS.md) covers Tier 1 and Tier 2 with
30 scenario-based questions, expandable answer guides, follow-ups, conceptual
checks, and optional Java reasoning drills. Use the topic index to pick a question
when you need it; attempt an answer before opening the guide, then revisit only
the linked notes where your understanding is weak.
