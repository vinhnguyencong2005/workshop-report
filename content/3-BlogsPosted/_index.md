---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
includeInReport: false
---
###  [Blog 1 - "THE DYNAMIC DUO" - AWS GLUE & AMAZON ATHENA: WHEN ETL MEETS QUERY ENGINE ON A DATA LAKE](3.1-Blog1/)
This blog introduces the combination of AWS Glue and Amazon Athena — the "dynamic duo" for building serverless data processing and analytics pipelines on Amazon S3. By coupling Glue's automated data crawling and transformation capabilities with Athena's direct SQL query engine via a shared Glue Data Catalog, this article delivers a cost- and performance-optimized Data Lake solution without the need to manage any infrastructure.

### [Blog 2 - AWS ESTIMATED BILLING SHOWED TRILLION-DOLLAR INVOICES – WHAT REALLY HAPPENED?](3.2-Blog2/)
An analysis of the AWS Estimated Billing anomaly that displayed trillion-dollar estimated costs. The article explores the architectural decoupling of Estimate vs. Invoicing pipelines, unit pricing calculations, why code rollback doesn't fix corrupted state, and key distributed system principles for blast radius containment.

###  [Blog 3 - ON-DEMAND VS. PROVISIONED MODE IN AMAZON DYNAMODB – WHICH MODEL SHOULD YOU CHOOSE?](3.3-Blog3/)
This article compares the two capacity modes of Amazon DynamoDB (Provisioned Mode and On-Demand Mode), explains underlying operation mechanisms such as Burst Capacity and Partition Scaling, and provides actionable recommendations to help you select the optimal model for both cost and performance based on real-world scenarios.

###  [Blog 4 - 5 THINGS I LEARNED WHEN EXPLORING AMAZON S3](3.4-Blog4/)
A synthesis of 5 core takeaways from deep-diving into Amazon S3 storage architecture: Flat Namespace without physical directories, hidden billing traps when deleting files (Multipart Uploads & Versioning), request and bandwidth cost optimization via CloudFront, high-concurrency uploading using Pre-signed URLs, and practical tradeoffs of S3 Storage Classes & Intelligent-Tiering.