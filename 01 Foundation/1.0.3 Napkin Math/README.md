# 1.0.3 - Napkin Math

> [!NOTE]
> **Napkin math** turns a product idea into rough estimates of requests, servers, storage, bandwidth, and memory. The goal is the right order of magnitude, not false precision.

## 1. Start with requests per second

Imagine a photo-sharing app with **10 million daily users**, each making **20 requests per day**:

| Estimate | Calculation | Rough result |
|---|---:|---:|
| Requests per day | 10 million x 20 | 200 million |
| Average requests per second | 200 million / 86,400 seconds | about 2,300 |
| Peak, at 3x average | 2,300 x 3 | about 7,000 |

There are 86,400 seconds in a day. For quick mental math, round that to **about 100,000**: **1 million requests/day is about 10-12 requests/second**. Keep track of whether a number is average or peak.

> [!TIP]
> Traffic is uneven. Evening activity or a major event can create a peak several times higher than the daily average. A 3x peak is a planning assumption, not a universal rule; use measured traffic when available.

## 2. Estimate server count

Suppose a load test shows one server handles about **1,000 requests/second** for this workload. A peak of 7,000 requests/second needs at least **7 servers** at that tested capacity. Plan for headroom and a server failure; **10 servers** would leave 9,000 requests/second of nominal capacity after losing one.

```text
baseline servers = peak requests/second / tested requests/second per server
planned servers  = baseline plus capacity for headroom and failures
```

The load balancer spreads requests across the servers. Real capacity depends on request mix, latency targets, and configuration, so measure it rather than guessing.

## 3. Estimate storage

Assume **1 million photo uploads per day**, each averaging **2 megabytes**:

| Estimate | Calculation | Rough result |
|---|---:|---:|
| New photo data per day | 1 million x 2 MB | 2 terabytes (TB) |
| One year, one copy | 2 TB x 365 | 730 TB |
| One year, three copies | 730 TB x 3 | 2,190 TB, or about 2.2 petabytes (PB) |

This excludes metadata, thumbnails, backups beyond the three copies, and storage overhead. Copies improve durability, but replication policy depends on the storage system. **Storage accumulates; daily request volume comes and goes.**

## 4. Estimate bandwidth

Uploads are only part of the traffic. If **10 million users** each view **100 thumbnails/day** at **200 kilobytes (KB)** each:

| Direction | Daily data | Approximate average bandwidth | At 3x peak |
|---|---:|---:|---:|
| Incoming photo uploads | 2 TB/day | about 0.2 gigabits/second (Gbps) | about 0.6 Gbps |
| Outgoing thumbnails | 200 TB/day | about 20 Gbps | about 60 Gbps |

These use decimal units and a 3x peak assumption. For example, 200 TB/day is about 2.3 GB/s on average, or about 18.5 Gbps; rounding gives roughly 20 Gbps. The peak estimate is not the daily average.

> [!IMPORTANT]
> Media-heavy apps can run into bandwidth and storage limits before CPU limits. Estimate **bytes transferred**, not only request counts. A content delivery network (CDN) can serve popular media closer to users and reduce load on origin servers.

## 5. Does the hot data fit in memory?

Suppose 10 million frequently used profiles take about **2 KB each**: that is roughly **20 GB**. If another hot set takes 1 KB for each of 1 million items, that adds about **1 GB**, for roughly **21 GB** before cache overhead.

That may fit in a cache with enough usable memory, plus room for keys, metadata, growth, and a second copy for redundancy. The estimate helps decide whether caching is practical; it does not mean every profile must stay cached.

## 6. Handy numbers to remember

| Thing | Approximate scale |
|---|---:|
| Random access memory (RAM) read | 100 nanoseconds (ns) |
| Solid-state drive (SSD) read | 100 microseconds (µs) |
| Datacenter network round trip | 0.5 milliseconds (ms) |
| Round trip across an ocean | 75-150 ms |
| Seconds in a day | 86,400 (about 100,000 for quick estimates) |

These are order-of-magnitude reminders, not service guarantees. Actual times depend on hardware, network path, and workload. A content delivery network (CDN) round trip is not one fixed number; it depends on the user-to-edge path and what is being measured.

## Quick recap

```text
average requests/s = daily users x requests per user per day / 86,400
peak requests/s    = average requests/s x peak multiplier
baseline servers   = peak requests/s / tested capacity per server
storage            = items/day x size/item x days x copies
bandwidth          = bytes/day / seconds/day, then convert units
in-flight requests = throughput x latency
```

> [!TIP]
> Write down every assumption, round sensibly, and check whether the answer is **kilobytes, gigabytes, terabytes, or petabytes**. A rough estimate that exposes the dominant cost is more useful than a precise-looking guess.
