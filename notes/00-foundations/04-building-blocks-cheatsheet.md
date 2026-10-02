# Building Blocks & Trade-offs Cheat Sheet

One-page revision before an interview. Each row: what it is → when to reach for it → what to say about it. Deep dives live in the Tier 1 / Tier 2 notes.

## 1. Building blocks

| Block | What it does | Reach for it when | Say / watch out for | Examples |
|---|---|---|---|---|
| **DNS** | Name → IP; geo/latency/weighted routing | Always; multi-region routing | TTL caching slows failover | Route 53, Cloudflare DNS |
| **CDN** | Edge cache for static/media content | Images, video, JS/CSS, global users | Pull vs push; invalidate via versioned URLs | CloudFront, Akamai, Cloudflare |
| **Load balancer** | Spread traffic, health checks | >1 server | L4 vs L7; LB itself must be HA | ALB/NLB, Nginx, HAProxy, Envoy |
| **API gateway** | Single entry: auth, rate limit, routing, TLS | Microservices, public APIs | Can become a bottleneck/SPOF; keep it thin | Kong, AWS API Gateway, Spring Cloud Gateway |
| **App servers** | Business logic | Always | Keep **stateless** → horizontal scale | Spring Boot on K8s |
| **Cache** | In-memory hot data | Read-heavy, latency SLOs | Cache-aside, TTL, invalidation, stampede, hot keys | Redis, Memcached, Caffeine |
| **Relational DB** | ACID, joins, schema | Transactions, relations, money | Replicas for reads; sharding is hard | Postgres, MySQL |
| **Key-value store** | Get/put by key, massive scale | Sessions, carts, simple lookups | Model around access pattern | DynamoDB, Redis |
| **Wide-column store** | Partitioned, write-optimised (LSM) | Huge write volume, time-series, messages | Partition key design, no joins | Cassandra, ScyllaDB, HBase |
| **Document store** | JSON documents | Flexible/nested schema | Denormalisation, weaker multi-doc txns | MongoDB |
| **Graph DB** | Nodes + edges traversal | Social graph, recommendations, fraud | Hard to shard | Neo4j, Neptune |
| **Search index** | Inverted index, full-text, ranking | Search, filters, autocomplete | Not source of truth; sync via CDC/events | Elasticsearch, OpenSearch |
| **Message queue** | Async point-to-point work | Background jobs, load levelling | At-least-once → idempotent consumers; DLQ | SQS, RabbitMQ |
| **Event log / stream** | Durable, replayable, partitioned log | Fan-out, event-driven, CDC, analytics | Ordering per partition; consumers ≤ partitions | Kafka, Kinesis, Pulsar |
| **Pub/Sub** | Broadcast to many subscribers | Notifications, realtime fan-out | Usually no replay (Redis Pub/Sub) | SNS, Redis Pub/Sub, GCP Pub/Sub |
| **Blob / object storage** | Cheap, durable large files | Images, video, backups, logs | Store metadata in DB, file in blob; presigned URLs | S3, GCS, Azure Blob |
| **Coordination service** | Consensus: leader election, config, locks | Leader election, service registry, distributed locks | CP; keep traffic low | ZooKeeper, etcd, Consul |
| **Service discovery** | Service name → healthy instances | Dynamic microservices | Client-side vs server-side | Consul, Eureka, K8s DNS |
| **Rate limiter** | Throttle per key | Public APIs, abuse, cost | Token bucket + Redis Lua; fail-open vs closed | Envoy, Bucket4j |
| **Batch / stream processing** | Large-scale compute on data | Analytics, ETL, aggregation, ML features | Batch = latency hours; stream = seconds | Spark, Flink, Kafka Streams |
| **Data warehouse / lake** | OLAP analytics on history | Reporting, BI | Separate from OLTP DB | BigQuery, Snowflake, Redshift, S3 + Athena |
| **Observability stack** | Metrics, logs, traces, alerts | Always (mention at the end) | Alert on SLOs, correlation IDs | Prometheus, Grafana, ELK, Jaeger/OTel |
| **Scheduler / cron** | Run jobs at time T | Reminders, expiry, retries | Distributed → lock or leader to avoid double-runs | Quartz, K8s CronJob, Airflow |

## 2. Classic trade-offs (have a one-liner for each)

| Trade-off | Choose A when… | Choose B when… |
|---|---|---|
| **Consistency vs availability** (CAP) | Money, inventory, uniqueness | Feeds, likes, carts, DNS |
| **Latency vs consistency** (PACELC) | Correctness beats speed | Speed beats freshness |
| **SQL vs NoSQL** | Relations, transactions, ad-hoc queries | Known access patterns, massive scale, flexible schema |
| **Normalise vs denormalise** | Write-heavy, integrity | Read-heavy, avoid joins at scale |
| **Vertical vs horizontal scaling** | Early stage, DB quick win | Stateless tier, beyond one box |
| **Sync vs async communication** | Caller needs the result now | Slow/unreliable work, fan-out, spikes |
| **Push vs pull** (fan-out on write vs read) | Normal users: precompute feeds on write | Celebrities: merge at read time (hybrid) |
| **Polling vs WebSockets vs SSE** | Rare updates → polling | Bi-directional → WS; server→client only → SSE |
| **Cache-aside vs write-through** | General read-heavy, tolerate brief staleness | Must read fresh right after write |
| **Strong vs eventual replication** (sync vs async) | Can't lose writes | Low write latency, tolerate lag |
| **Monolith vs microservices** | Small team, early product | Many teams, independent scaling/deploys |
| **REST vs gRPC** | Public/browser APIs | Internal high-throughput service calls |
| **Batch vs stream processing** | Hourly/daily reports, cheaper | Real-time dashboards, fraud detection |
| **Range vs hash sharding** | Range scans (time-series) | Even distribution, point lookups |
| **Optimistic vs pessimistic locking** | Low contention | High contention, short critical sections |
| **Throughput vs latency** | Batch/buffer for throughput | Process individually for latency |
| **Accuracy vs cost/speed** | Billing, exact counts | Approximate: HyperLogLog, Count-Min Sketch, sampled analytics |

## 3. Probabilistic data structures (bonus points)
| Structure | Answers | Use |
|---|---|---|
| **Bloom filter** | "Definitely not in set" / "maybe in set" | Cache penetration guard, crawler visited-URLs, LSM SSTable lookup |
| **HyperLogLog** | Approx count of distinct items (~12 KB, ~1% error) | Unique visitors, DAU |
| **Count-Min Sketch** | Approx frequency of items | Heavy hitters, trending topics, top-K |
| **Geohash / Quadtree** | Nearby points | Uber drivers nearby, Yelp |
| **Trie** | Prefix lookup | Autocomplete / typeahead |
| **Merkle tree** | Detect which ranges differ between replicas | Anti-entropy in Cassandra/Dynamo, Git, blockchains |

## 4. Common patterns → where they show up
| Pattern | Case studies |
|---|---|
| Cache-aside + CDN | URL shortener, news feed, video streaming |
| Fan-out on write / read (hybrid) | News feed, Twitter timeline |
| WebSockets + presence registry | Chat, ride sharing, collaborative docs |
| Queue + workers + DLQ | Notification system, video transcoding, web crawler |
| Consistent hashing + replication | Distributed KV store, distributed cache |
| Presigned URLs + blob storage + chunking | File storage (Dropbox), video upload |
| Geo-indexing (geohash/quadtree) | Ride sharing, proximity search |
| Snowflake IDs + base62 | URL shortener, chat message IDs |
| Idempotency keys + outbox | Payments, order processing |
| Token bucket in Redis | Rate limiter, API gateway |

## 5. The 30-second pre-interview checklist
- [ ] Clarify functional scope + NFRs with numbers
- [ ] Estimate QPS, storage → draw a conclusion
- [ ] API + data model before boxes
- [ ] Walk one read and one write path
- [ ] For each choice: bottleneck → options → choice → trade-off
- [ ] Failures: what if X dies? SPOFs? retries/idempotency?
- [ ] Close with monitoring, and what you'd improve next
