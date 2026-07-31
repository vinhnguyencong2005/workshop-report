---
title: "Blog 3"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
includeInReport: false
---
# On-Demand vs Provisioned Mode in Amazon DynamoDB - Which Model Should You Choose?

One of the most important decisions when designing a NoSQL database with Amazon DynamoDB is choosing the right **Read/Write Capacity Mode**. Making the right choice not only keeps your system running smoothly but can also **save up to 70% in costs**.

This article compares the two models - **Provisioned Mode** and **On-Demand Mode** - along with underlying mechanisms (Burst Capacity, Partition Scaling) and practical application examples.

---

## 1. Provisioned Capacity Mode

**How it works:** You explicitly specify the number of **RCU** (Read Capacity Units) and **WCU** (Write Capacity Units) your table needs per second.

### Special Feature: Burst Capacity

Many people think that if you provision 100 WCU but only use 20 WCU, the remaining 80 WCU are "wasted." However, DynamoDB has a mechanism called **Burst Capacity**:

- DynamoDB automatically accumulates unused RCU/WCU during low-usage minutes and stores them in a **"reserve pool"**.
- This pool can hold up to the equivalent of **5 minutes (300 seconds)** of provisioned capacity.
- **Effect:** When your system suddenly experiences a short traffic burst exceeding the provisioned threshold, DynamoDB draws WCU/RCU from this reserve pool to serve requests immediately - **NO Throttling** and **NO extra cost**.

### Pros

- **Cost-Effective:** If your system load is stable or has predictable cycles (e.g., high during business hours, low at night), Provisioned mode is significantly cheaper than On-Demand.
- **Deeper savings with Reserved Capacity:** Commit to 1-year or 3-year terms for discounts of up to **70%**.

### Cons

- **Risk of Throttling Exception** (`ProvisionedThroughputExceededException`): If traffic spikes persist beyond 5 minutes (exhausting Burst Capacity) and Auto Scaling hasn't scaled out yet (can take 1–5 minutes to react).

---

## 2. On-Demand Capacity Mode

**How it works:** You don't need to specify RCU/WCU. DynamoDB automatically manages resources, and you pay only for the actual requests made (billed as **RRU/WRU** - Request Units).

### Special Feature: Lag / Throttling on Sudden Spikes

Many people mistakenly believe On-Demand mode can scale from 0 to infinity instantly in 1 millisecond. The reality is slightly different:

- **On-Demand scaling mechanism:** DynamoDB On-Demand can automatically serve up to **2x** the previous peak traffic ever recorded.
- **Issue with extremely fast traffic spikes** (e.g., from 500 req/s to 50,000 req/s in 2 seconds): When traffic surges far beyond the previous peak, DynamoDB needs time to automatically split partitions (**Partition Splitting**) and allocate additional physical servers in the background.
- During the seconds-to-minutes "warm-up" / partition split process, requests exceeding the threshold may experience increased **Latency** or temporary **Throttling**.

### Pros

- **Zero-Management:** No need to configure WCU/RCU or manage Auto Scaling rules.
- **Pay-per-request:** If your application has no traffic (e.g., Dev/Staging environments or an e-commerce app at night), costs are **$0**.

### Cons

- **Higher per-request cost:** The unit price per 1 million Request Units in On-Demand is roughly **70% more expensive** than continuously running 1 RCU/WCU in Provisioned mode under the same conditions.
- **Beware of massive traffic spikes:** You still need to warm up in advance if you know a major event (e.g., flash sale, deal hunting) is coming.

---

## 3. Practical Examples: When to Choose Which Model?

### Choose **On-Demand** Mode when:

- **Launching a new product:** You have no historical data to estimate user access patterns.
- **Applications with long idle periods:** The system receives sporadic customer input and is mostly idle at night.
- **Serverless environments:** Event-driven applications with highly unpredictable request patterns.

### Choose **Provisioned** Mode when:

- **Your application has stable, predictable traffic.**
- **You've been running On-Demand and have CloudWatch Metrics:** After monitoring CloudWatch for 1–2 months, you can determine baseline traffic → Switch to Provisioned + Auto Scaling to reduce costs.
- **You need strict cost optimization:** Combine Provisioned Mode with DynamoDB Reserved Capacity.

---

## References

- [Read/Write Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/capacity-mode.html)
- [Provisioned Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/provisioned-capacity-mode.html)
- [Burst & Adaptive Capacity](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/burst-adaptive-capacity.html)
- [On-Demand Capacity Mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/on-demand-capacity-mode.html)
- [DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)
- [Post on Facebook](https://www.facebook.com/share/p/18qTVu8i7b/)