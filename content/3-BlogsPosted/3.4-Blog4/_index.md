---
title: "Blog 4"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---

# 5 Things I Learned When Exploring Amazon S3

Hello everyone!

Recently, I spent time reading official documentation and diving deeper into AWS storage architecture, specifically Amazon S3. Initially, I viewed S3 simply as a basic file storage repository. However, after delving into technical docs, I realized there are numerous underlying mechanisms that, if overlooked, can lead to severe design mistakes and unexpected billing costs.

Below are 5 core insights I accumulated while researching S3, which I hope will be useful for anyone studying or building cloud projects:

### 1. S3 Has No Physical Directory System

When looking at the AWS S3 Console, folders appear intuitive. However, S3 is actually an Object Storage system with a **Flat Namespace** architecture.

What I learned is that a path like `images/2026/avatar.png` is NOT a file named `avatar.png` sitting inside two nested directory folders. The entire string is a single unique Key. Slashes (`/`) are merely Prefixes used to simulate a directory-like interface. Understanding this fundamental concept helped me realize that "renaming a folder" in S3 actually forces the system to copy every single object to a new key string and delete old objects — an extraordinarily expensive operation if prefixing is not planned properly upfront.

### 2. Clicking Delete Doesn't Mean AWS Stops Charging You

One surprising discovery regarding S3 Billing is that deleting files doesn't always stop storage charges:

* **Incomplete Multipart Uploads:** When large file uploads are enabled, the process is split into smaller parts. If the network drops mid-upload, incomplete parts remain stuck and hidden, yet AWS continues charging for them. The standard solution is configuring Lifecycle Rules to clean them up automatically.
* **Versioning:** If Versioning is enabled on a bucket, clicking delete merely attaches a "Delete Marker" to the file without removing the original data. The takeaway is always pairing Versioning with Lifecycle Rules to expire old versions after a set duration.

### 3. Storage Is Cheap, But Requests & Data Transfer Add Up

When drafting cost estimates, S3 Standard storage rates look remarkably cheap. However, budget overruns typically happen elsewhere:

* **Request Cost:** S3 charges per 1,000 API calls. If an application stores millions of tiny static files with high query rates, API call costs will far exceed storage fees.
* **Data Transfer Out:** Egress bandwidth pushing data from S3 to the Internet carries relatively high pricing.

Hence, the architectural principle is always placing a Content Delivery Network like **Amazon CloudFront** in front of S3 to cache static assets, blocking massive direct request spikes to S3.

### 4. Optimal Upload Architecture: Bypass the Backend

This is my favorite takeaway regarding System Design. Initially, my intuition was: `Client` uploads file to `Backend` → `Backend` uses SDK to push file to `S3`.

However, reading performance best practices revealed that this pattern creates heavy bottlenecks on Backend servers under concurrent upload loads. Modern architectures prefer **Pre-signed URLs**. The Backend merely validates permissions and generates a short-lived URL. The Client then uses that URL to upload heavy data streams directly to S3 servers.

### 5. S3 Offers Multiple Storage Classes — Don't Just Use Standard

I initially assumed all files uploaded to S3 reside in S3 Standard. AWS docs explain that S3 provides multiple Storage Classes tailored for different access patterns. For infrequently accessed data, Standard-IA or Glacier tiers dramatically lower storage costs.

My favorite feature is **S3 Intelligent-Tiering**. Simply selecting this Storage Class enables S3 to automatically monitor access patterns and shift objects between access tiers to optimize costs, without incurring Retrieval Fees. However, practical lessons suggest avoiding blind adoption:

* **Very Small Files:** For tiny files, AWS imposes automated management fees per object. If your system holds millions of tiny files, management fees can surpass S3 Standard costs.
* **Early Deletion:** Cheaper tiers come with minimum storage duration commitments. Deleting objects shortly after uploading still incurs full cycle billing.

---

These are the key insights I gained from exploring Amazon S3 theory. If anyone here has hands-on experience, encountered edge cases, or has optimization tips, feel free to share and discuss!

Thank you for reading!

### References
- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Amazon S3 Pricing](https://aws.amazon.com/s3/pricing/)