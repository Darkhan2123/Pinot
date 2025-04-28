https://www.youtube.com/watch?v=mwAH6t772E4&ab_channel=GOTOConferences

# Why Apache Pinot??

## Introduction

Apache Pinot revolutionizes real-time analytics by delivering sub-second query responses, even under high-concurrency workloads. Its architecture supports near-real-time ingestion, enabling you to analyze streaming data instantly. Unlike other tools, it excels in scalability, handling petabyte-scale datasets without compromising performance.

Adoption across industries highlights its impact. For example, Apache Pinot boasts 12.9 million Docker pulls and 8,000 Meetup members globally. These metrics reflect its growing popularity as a reliable solution for real-time insights. Whether you need interactive dashboards or time-sensitive analytics, Apache Pinot empowers you to act on data faster than ever.

Key Takeaways

    Apache Pinot gives super-fast answers to questions in less than a second, perfect for live data checks and dashboards.

    It can grow bigger easily, letting companies manage huge data without slowing down.

    Smart ways of organizing data in Pinot make it faster to find answers, even when many people use it at the same time.

    Pinot works well with tools like Kafka, helping handle and study live data quickly.

    Companies can save money by using Pinot's special storage and automatic data copying.


### Background

30 years ago, analytics consisted primarily of daily reports prepared for executives. These reports, often printed and bound, contained basic visualizations like pie charts summarizing the previous day's business activities. This was the birth of modern analytics and ETL (Extract, Transform, Load) processes.
Having insight into business activities from just a day ago was revolutionary for decision-makers at that time. However, the analytics landscape has evolved dramatically since then.

Traditional analytics databases, designed primarily for internal reporting and batch processing, struggled to meet these new requirements. This gap in capabilities prompted the engineering team at LinkedIn to develop what would eventually become Apache Pinot.

### Scope

This report covers:
- The fundamental concepts and features of Apache Pinot
- A detailed explanation of Pinot's architecture and components
- How data flows through Pinot's system
- Real-world use cases and implementations
- Comparisons with other analytics technologies
- Basic deployment information

## What is Apache Pinot?

Apache Pinot is a real-time distributed OLAP datastore specifically designed for low-latency, high-throughput analytics. It was created to power user-facing analytics applications that require immediate insights from fresh data.

Unlike traditional data warehouses that primarily serve internal business users, Pinot was built from the ground up to support analytics features that end users interact with directly. This fundamental difference drives many of Pinot's architectural decisions.

### Core Features

Pinot offers several key features that differentiate it from other analytics databases:

**Fast Queries**: Pinot can filter and aggregate petabytes of data with P90 latencies in the tens of milliseconds—fast enough to support interactive user interfaces.

**High Concurrency**: With its distributed architecture, Pinot can handle hundreds of thousands of concurrent queries per second, making it suitable for user-facing applications with large user bases.

**Batch and Streaming Ingest**: Pinot can ingest data from streaming sources like Apache Kafka, Apache Pulsar, and AWS Kinesis in real-time, while also supporting batch ingestion from Hadoop, Spark, AWS S3, and other sources. It can combine both types of data sources into a single queryable view.

**Rich Indexing Options**: Pinot offers pluggable indexes including timestamp, inverted, StarTree, Bloom filter, range, text, JSON, and geospatial indexes. These enable ultra-fast query execution across various data types and query patterns.

**Built for Scale**: Horizontally scalable and fault-tolerant, Pinot can adapt to workloads across the storage and throughput spectrum.

**SQL Query Interface**: Pinot provides a highly standardized SQL query interface accessible through a built-in query editor and REST API, making it approachable for developers and analysts.

**Multitenancy Support**: Built-in multitenancy allows management and security isolation in logical namespaces, supporting cloud-friendly resource management.

### Key Applications

Pinot excels in scenarios where:

- Analytics must be delivered directly to end users
- Query results must be returned in milliseconds
- Data freshness is measured in seconds rather than hours or days
- The system must support thousands or millions of concurrent users
- Both historical and real-time data must be queryable through a unified interface
- Data volumes are very large (terabytes to petabytes)

## Pinot Architecture

At its core, Apache Pinot is a distributed system designed to handle massive scale while maintaining performance. Understanding Pinot's architecture is like understanding how a busy restaurant kitchen works - different specialized stations work together to deliver orders quickly even during peak hours.

### Core Components

Pinot's architecture consists of four main components:

**Controller**: The brain of the system that manages the cluster and orchestrates other components. The controller handles cluster management, table creation, segment assignment, and other administrative tasks. Think of the controller as the head chef who oversees the entire kitchen, making sure all stations are running smoothly and coordinating the overall operation.

**Broker**: Receives queries from clients, routes them to the appropriate servers, collects and merges results, and returns the final answer. The broker is like the server at a restaurant who takes your order, brings it to the kitchen, collects your food when it's ready, and delivers it to your table.

**Server**: Stores data segments and processes queries on them. Servers are organized into groups that specialize in either real-time or offline (batch) data processing. Servers are like the line cooks who actually prepare different parts of your meal - some might specialize in grilling while others handle salads or desserts.

**Minion**: Executes background tasks like converting data from raw formats into optimized segment files, purging old data, and other maintenance work. Minions are like the prep cooks and dishwashers who work behind the scenes to keep everything running smoothly.

![Pinot Architecture](https://i.imgur.com/example.jpg)

Pinot relies on two additional important technologies:

**Apache Helix**: Manages cluster state and coordinates the distributed system. Helix is like the communication system that allows all kitchen staff to coordinate their activities.

**Apache ZooKeeper**: Provides reliable distributed coordination for the cluster. ZooKeeper is like the central bulletin board where important information is posted for everyone to see.

### Data Flow

Data flows through Pinot in two primary ways: real-time and batch.

**Real-time Ingestion**:
1. Data is produced by source systems and sent to a streaming platform like Kafka
2. Pinot's real-time servers consume directly from the streaming platform
3. Incoming data is stored in-memory in "consuming segments"
4. Data is immediately available for querying, even before it's written to disk
5. When certain thresholds are reached (time, size, or number of rows), consuming segments are "completed" and persisted to disk
6. Completed segments are uploaded to deep storage (like HDFS or S3)

This process is similar to a food delivery service where orders are taken and food preparation begins immediately. Customers can check the status of their order at any point in the process.

**Batch Ingestion**:
1. Data is collected in batch storage systems like HDFS or S3
2. Pinot's minion tasks (or external jobs) process this data into Pinot segments
3. Segments are uploaded to deep storage
4. The controller notifies offline servers to download and load these segments
5. Once loaded, the data is available for querying

This is comparable to a restaurant preparing ingredients in advance during off-peak hours to handle the dinner rush more efficiently.

### Query Processing

When a query arrives at Pinot, the following sequence occurs:

1. A client sends a SQL query to a Pinot broker
2. The broker analyzes the query and determines which segments (and therefore which servers) contain relevant data
3. The broker creates a query plan and sends appropriate sub-queries to the selected servers
4. Each server processes its portion of the query in parallel, scanning and filtering data from its local segments
5. Servers return their results to the broker
6. The broker merges and aggregates all results into a final answer
7. The final result is returned to the client

This distributed processing approach is key to Pinot's performance, allowing it to divide and conquer complex queries across many servers. It's like a restaurant kitchen where different stations work in parallel to prepare different components of a meal simultaneously.

For especially complex queries, Pinot can use its multi-stage query engine, which works like an assembly line with multiple processing stages.

### Data Storage

Pinot stores data in a columnar format, which provides several advantages for analytics workloads:

1. **Efficient Storage**: Similar data types compress better together
2. **Query Performance**: Only relevant columns need to be read for a query
3. **Vectorized Processing**: Operations can be performed on entire columns efficiently

Data in Pinot is organized into:

**Tables**: Logical collections of related data with a defined schema

**Segments**: The basic unit of data storage and parallelism in Pinot. A table is divided into many segments based on time ranges or other partitioning schemes. Segments are like the containers food is stored in - each containing a specific type of ingredient for quick access.

**Shards**: Segments can be further divided into shards for even greater parallelism

Pinot employs immutable segments as its primary storage model. When data needs to be updated (as in upsert operations), Pinot uses special techniques to ensure the latest version is returned at query time while maintaining the performance benefits of immutability.

### Indexing Strategies

One of Pinot's most powerful features is its rich set of indexing options. These indexes dramatically accelerate query performance, similar to how a well-organized kitchen allows chefs to find ingredients instantly rather than searching through everything.

**Forward Index**: The basic storage format mapping document IDs to values

**Inverted Index**: Maps values back to document IDs, accelerating equality filters (WHERE column = value)

**Range Index**: Optimizes range filters (WHERE column BETWEEN x AND y)

**StarTree Index**: A specialized index that pre-computes aggregations along different dimension combinations, dramatically speeding up common OLAP queries

**Bloom Filter**: A probabilistic data structure that quickly determines if a value might exist in a segment

**Text Index**: Based on Apache Lucene, enables full-text search capabilities

**JSON Index**: Supports efficient queries on nested JSON data

**Geospatial Index**: Optimizes location-based queries

Each index type serves specific query patterns, and tables can use multiple index types simultaneously. The appropriate indexing strategy depends on the query patterns, data characteristics, and performance requirements of each specific use case.

## Real-world Applications

### LinkedIn Case Study

As Pinot's birthplace, LinkedIn employs it to power numerous user-facing analytics features:

**Who Viewed Your Profile**: This flagship analytics product shows LinkedIn members who has viewed their profile in real-time. It uses Pinot to run complex queries on large volumes of profile view data to identify interesting insights dynamically.

**Company Page Analytics**: Helps company administrators understand the demographics of their followers and analyze follower trends.

**Content Engagement Metrics**: Provides authors with real-time insights into how their posts and articles perform.

**Ad Campaign Reporting**: Offers advertisers near-real-time visibility into campaign performance.

LinkedIn reports that Pinot powers 50+ user-facing products, ingesting millions of events per second and serving 100k+ queries per second at millisecond latency.

### Uber Case Study

Uber has implemented Pinot for several critical analytics applications:

**UberEats Restaurant Manager**: Provides restaurant owners with real-time insights into their performance on the platform, including order volumes, customer satisfaction, and delivery times.

**Driver and Rider Experience Monitoring**: Analyzes trip data in real-time to identify and address service issues.

**Dynamic Pricing**: Supports algorithms that adjust prices based on current market conditions.

### Other Organizations

Many other major technology companies have adopted Pinot:

**Microsoft**: Uses Pinot for real-time service monitoring and user engagement analytics.

**Slack**: Employs Pinot for workspace analytics and feature usage tracking.

**WeChat**: Leverages Pinot to analyze user interactions across its massive platform.

**Stripe**: Utilizes Pinot for real-time financial analytics.

**Walmart**: Implements Pinot for inventory and sales analytics.

## Comparison With Other Systems

To understand Pinot's positioning in the analytics landscape, it's helpful to compare it with other technologies:

| Feature | Apache Pinot | Apache Druid | ClickHouse | Traditional Data Warehouse |
|---------|-------------|--------------|------------|---------------------------|
| Query latency | Milliseconds | Milliseconds to seconds | Seconds | Seconds to minutes |
| Data freshness | Seconds | Minutes | Minutes to hours | Hours to days |
| Concurrent queries | 100,000+ | 10,000+ | 1,000+ | 100+ |
| Streaming ingest | Native | Yes | Limited | Very limited |
| SQL support | Yes (most of SQL-92) | Yes (limited) | Yes (extensive) | Yes (extensive) |
| Primary use case | User-facing analytics | Interactive analytics | Analytical reporting | Business intelligence |
| Scaling model | Horizontal | Horizontal | Horizontal | Vertical & Horizontal |
| Pre-aggregation | Yes (StarTree) | Yes (data cubes) | Yes (materialized views) | Limited |

This comparison highlights Pinot's specialization in ultra-low-latency, high-concurrency scenarios, particularly for user-facing applications. While other systems may excel in different areas, Pinot occupies a unique position in the real-time, user-facing analytics niche.

## Getting Started With Pinot

Getting started with Pinot is relatively straightforward. For a basic development environment, Docker provides the simplest approach:

```
docker run -p 9000:9000 apachepinot/pinot:1.3.0 QuickStart -type hybrid
```

This command launches a self-contained Pinot instance with example data and a web interface accessible at http://localhost:9000.

For production deployments, Kubernetes is the recommended approach. The Pinot Helm chart provides a configurable way to deploy all required components:

```
helm repo add pinot https://raw.githubusercontent.com/apache/pinot/master/kubernetes/helm
helm install pinot pinot/pinot
```

Detailed setup instructions and configuration options are available in the official Apache Pinot documentation.

## Conclusions

Apache Pinot represents a significant advancement in analytics infrastructure, enabling a new category of data-driven applications and experiences. Its unique architecture and design choices address the specific challenges of user-facing analytics, where traditional data warehousing approaches fall short.

Key findings from this analysis include:

1. Pinot's distributed architecture, with specialized components for different functions, enables it to achieve both extreme scale and low latency simultaneously.

2. The combination of real-time and batch ingestion within a unified query layer provides a "best of both worlds" approach to data freshness and completeness.

3. Pinot's rich indexing options, particularly the StarTree index, deliver dramatic performance improvements for common analytics queries.

4. The system's focus on immutability as a core principle simplifies many aspects of distributed data management while still supporting update operations where needed.

5. Pinot's success at companies like LinkedIn and Uber demonstrates its viability for production-scale, mission-critical analytics workloads.

Organizations seeking to implement real-time analytics capabilities, particularly for customer-facing features, should consider Apache Pinot as a primary technology choice. Its combination of performance, scale, and flexibility makes it well-suited to the increasing demands for immediate, data-driven insights.

## References

1. Apache Pinot Official Documentation. [Online] Available at: https://docs.pinot.apache.org/

2. LinkedIn Engineering (2014). "Real-time Analytics at Massive Scale with Pinot." [Online] Available at: https://engineering.linkedin.com/analytics/real-time-analytics-massive-scale-pinot

3. Berglund, T. "What is Apache Pinot (and User-Facing Analytics)." StarTree Lightboard Series.

4. Apache Pinot GitHub Repository. [Online] Available at: https://github.com/apache/pinot

5. Neppalli Naga, P. (2014). "Real-time Analytics at Massive Scale with Pinot." LinkedIn Engineering Blog.
