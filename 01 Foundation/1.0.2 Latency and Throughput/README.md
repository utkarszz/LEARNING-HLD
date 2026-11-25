# 1.0.2 - Latency and Throughput

> **Core idea:** Latency is the time one request takes. Throughput is how many requests the system completes per second.

## 1. The two measures

| Measure | Meaning | Highway analogy |
|---|---|---|
| **Latency** | Time for one request to get an answer | How long one car's trip takes |
| **Throughput** | Completed requests per second | How many cars pass a point in an hour |

They are related, but they are not the same. A system can complete many requests per second while some individual requests take a long time.

## 2. Why queues grow near capacity

Suppose one server can handle about **500 requests per second**:

| Request rate | Utilization | What to expect |
|---:|---:|---|
| 350 requests/s | 70% | Some spare capacity; bursts can form short queues |
| 480 requests/s | 96% | Very little spare capacity; small bursts can create long queues |

**Utilization** is the share of available capacity currently in use. As utilization gets close to 100%, incoming requests are more likely to find the server busy. Bursts and slower requests then wait in line, so latency can rise sharply even when throughput increases only a little.

Keep headroom at peak load. A target such as 60-70% utilization can be a useful planning starting point, but the right level depends on traffic bursts, workload, and latency goals.

> Requests per second are the key input here. A user count alone does not determine server load; users can send requests at very different rates.

## 3. Averages hide the slow requests

Requests do not all take the same time. A latency **percentile** tells you the time below which a given share of requests finish:

| Metric | Read it this way | Example if P99 is 800 ms |
|---|---|---|
| **P50** | The median: half of requests are this fast or faster, and half are slower | P50 might be 100 ms: about half finish within 100 ms |
| **P99** | 99 out of 100 requests are this fast or faster; 1 out of 100 takes longer | About 1 in 100 takes more than 800 ms |

The average can look healthy while a small group waits much longer. Track percentiles, especially P99, to see the experience of slow requests. P99 is not the slowest request; the slowest 1% can take even longer.

## 4. Capacity can buy back latency

If demand stays at **350 requests/s** and capacity doubles from 500 to 1,000 requests/s, utilization falls from 70% to 35%. The extra headroom helps queues drain and can lower both P50 and P99. Demand and throughput may stay the same; the improvement comes from less waiting under load.

You can create headroom by adding capacity or by reducing work per request, such as improving an expensive query or avoiding unnecessary calls.

## 5. Distance adds latency

Signals in fiber travel at roughly **200 km per millisecond**. A New York-to-London path of about 5,600 km implies at least around 56 ms of propagation time for a round trip in an idealized direct path; real routes and equipment make it longer, often around 75 ms or more. The exact latency depends on the network path.

Server upgrades cannot remove propagation delay. CDNs and regional deployments reduce latency by serving content or requests closer to users.

## 6. Little's Law: requests in flight

**Requests in flight = throughput x latency** (when the system is in a stable state).

For example, at 350 requests/s and 1 second of latency, about **350 requests** are in flight on average. If a worker or connection pool can handle only 50 at once, the rest may wait or consume other resources. This helps size workers and pools, and explains why slow dependencies can exhaust them.

## 7. Fan-out makes tail latency common

A page that calls 100 index servers in parallel must wait for the slowest response. If each server independently has a 1% chance of exceeding its own P99, then:

**Chance at least one is slower than its P99 = 1 - 0.99^100 = about 63%.**

So a slow response that is rare for one server can affect many fan-out requests. The page's P99 is not automatically equal to one server's P99; it depends on the latency distribution, dependencies, and whether calls are truly independent.

One mitigation is a **hedged request**: if a call has not returned after a threshold, send a duplicate to another replica and use the first response. This can reduce tail latency, but adds load, so use it selectively and cancel the slower duplicate when possible.

## 8. Batching trades latency for throughput

Instead of inserting analytics events one at a time, collect up to 100 rows or wait up to 50 ms, then send one batch. This amortizes per-statement overhead and can raise throughput substantially, while each event may wait up to 50 ms before the batch is sent. The actual gain depends on the database and workload; a 25x gain is an example, not a guarantee.

Batching often fits logs, analytics, and bulk imports. It may be a poor fit for a user waiting on an immediate click response.

## Quick recap

- **Latency:** time per request. **Throughput:** completed requests per second.
- Near full utilization, queues can make latency rise sharply.
- P50 describes the middle request; P99 reveals the slow tail.
- More capacity or less work per request creates headroom.
- Distance adds network delay; regional serving and CDNs can reduce it.
- Little's Law: in-flight requests = throughput x latency.
- Fan-out amplifies tail latency; batching can improve throughput at the cost of waiting.
