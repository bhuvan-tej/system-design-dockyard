# Back-of-Envelope Estimation

## 1. What it is
Quick order-of-magnitude math to decide *whether* you need caching, sharding, CDNs, etc. The number matters less than the **conclusion you draw from it**.

## 2. Numbers to memorise

### Powers of two / data sizes
| Power | Approx        | Name |
|-------|---------------|------|
| 2^10  | 1 thousand    | 1 KB |
| 2^20  | 1 million     | 1 MB |
| 2^30  | 1 billion     | 1 GB |
| 2^40  | 1 trillion    | 1 TB |
| 2^50  | 1 quadrillion | 1 PB |

### Time
- 1 day ≈ **86,400 s ≈ 10^5 s** (use 10^5 for mental math)
- 1 month ≈ 2.5 × 10^6 s, 1 year ≈ 3 × 10^7 s

### Latency numbers (order of magnitude)
| Operation | Latency |
|---|---|
| L1 cache reference | ~1 ns |
| Main memory reference | ~100 ns |
| Read 1 MB sequentially from memory | ~10 µs |
| SSD random read | ~100 µs |
| Round trip within same datacenter | ~0.5 ms |
| Read 1 MB sequentially from SSD | ~1 ms |
| Disk (HDD) seek | ~10 ms |
| Round trip cross-continent | ~150 ms |

**Takeaways:** memory ≫ SSD ≫ network ≫ HDD; avoid cross-region round-trips on the hot path; compress before sending over network.

### Rough single-node capacities (sanity checks)
| Component | Ballpark |
|---|---|
| Web/app server (simple API) | 1K – 10K RPS |
| Relational DB (well-indexed reads) | ~5K – 10K QPS; writes lower |
| Redis single node | ~100K ops/s |
| Kafka broker | ~100s of MB/s |

### Availability "nines"
| Availability | Downtime / year |
|---|---|
| 99% | 3.65 days |
| 99.9% | 8.76 hours |
| 99.99% | 52.6 minutes |
| 99.999% | 5.26 minutes |

Components in **series** multiply availability (0.999 × 0.999 = 0.998). Redundant components in **parallel**: 1 − (1 − a)^n.

## 3. The formulas
```
Avg QPS       = DAU × actions_per_user_per_day / 10^5
Peak QPS      = Avg QPS × 2..5   (say 2x-3x unless told otherwise)
Storage/yr    = writes_per_day × object_size × 365
Bandwidth     = QPS × response_size
Cache memory  = 20% of daily read volume (80/20 rule)
Servers       = Peak QPS / QPS_per_server  (+ headroom)
```

## 4. Worked example — URL shortener
Assume 100M DAU, each creates 0.1 URLs/day and does 10 redirects/day. URL record ≈ 500 bytes.

- Writes: 100M × 0.1 / 10^5 = **100 writes/s** (peak ~300)
- Reads: 100M × 10 / 10^5 = **10K reads/s** (peak ~30K) → **read:write = 100:1**
- Storage: 10M new URLs/day × 500 B = 5 GB/day → ~**2 TB/yr**, ~10 TB over 5 yrs
- Cache: 1B reads/day; caching the hottest 20% → 0.2 × 1B × 500 B ≈ **100 GB** (upper bound — many reads hit the same URLs) → fits in a small Redis cluster
- Short code length: 62^7 ≈ 3.5 × 10^12 → 7 base62 chars is plenty for 10 yrs

**Conclusions to say aloud:**
1. Read-heavy → cache in front of DB, read replicas.
2. 10 TB over 5 yrs → a single node is borderline; plan to shard by short code.
3. 30K peak read QPS → ~5–10 stateless app servers behind an LB.

## 5. Interview tips
- Round aggressively (86,400 → 10^5). Say "roughly".
- State assumptions out loud and ask if they're OK.
- Always convert numbers into a **design decision**.
- Don't spend more than ~5 minutes here.

## 6. Practice
Estimate for: Twitter timeline reads, WhatsApp messages/day storage, YouTube upload bandwidth, Uber location updates/sec (drivers send GPS every 4s).
