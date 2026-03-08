| Date       | Target Audience                                                                 |
|------------|---------------------------------------------------------------------------------|
| 10.03.2026 | Data Engineers, Architects or anybody involved in strategic decisions like this |

# When (not) to move into a lake house 🌊🏡
Throughout my data engineering career, I’ve seen modern lakehouse platforms such as Databricks introduced into organizations with great enthusiasm — and very mixed results. Not because the technology is flawed, but because the problems it solves are often misunderstood.

I’ve worked with teams that should have modernized their infrastructure but didn’t. I’ve also seen companies introduce the hottest big data platform on the market without having the use case — or the team — to support it.

These experiences made me realize that the real challenge isn’t choosing the “most modern” platform. It’s understanding what problem you are actually trying to solve.

This article is my attempt to explain how Databricks - as the leading lake house platform - works, what problems it is designed to solve, and what has — and hasn’t — worked in the projects I’ve been involved in. 
After reading this, you should not only have a better theoretical understanding of my favorite data platform, but also be better equipped to answer a more important question:

Is it a good fit for YOUR team?

## The Lakehouse Idea
To understand whether Databricks is the right tool for your team, it helps to first understand the problems that motivated its development.
Historically, organizations had two very different options for centralizing their data:

- Data Lakes – file-based storage systems, which are fast, flexible and cheap. They can store large volumes of raw data but often lack structure, governance, and reliability.

- Data Warehouses – structured systems optimized for analytics and reporting. They provide ACID transactions, schema enforcement, and query optimization, but are less flexible and often expensive at scale. Essentially, they are read-optimized relational Databases.

Choosing one or the other meant making significant trade-offs — trade-offs that more and more organizations were unwilling to accept. In response, Databricks introduced the lakehouse in 2020: an approach designed to combine the best of both worlds, offering both scalable file-based storage and structured, reliable analytics.

With this in mind, the first question to ask when considering a lakehouse platform is:

_Does your system or organization truly have the demands — both in terms of data volume and computational complexity — that justify this level of infrastructure?_ (don't worry, we'll dissect this question later)

## Databricks under the hood

The lakehouse concept achieves a balance between data lake and warehouse through three main components layered on top of your data lake:

1. **Table Format Layer (also called Open Table Format)**

   Provides ACID transactions, schema enforcement, versioning, and optimized file management. Without this layer, your data lake would remain just a collection of files with limited reliability. With it, concurrent reads and writes are handled safely, and changes are tracked accurately.  

   Databricks uses Delta Lake to implement this layer. Delta Lake stores raw data as **immutable** .parquet files and maintains a **_delta-log_** directory containing metadata about the table’s transactions. This metadata enables ACID transactions, time travel, schema enforcement, and rollback, turning a raw data lake into a reliable, analytics-ready foundation.

2. **Metadata & Governance Layer**

   In Databricks, this layer is implemented through Unity Catalog. It centralizes metadata management across your lakehouse and supports improved query planning, fine-grained access control, and consistent governance policies.

    Beyond access management, Unity Catalog provides data lineage capabilities, allowing you to trace dependencies between tables, jobs, and other assets. This transparency is essential in larger environments where understanding how data flows through the system becomes increasingly complex.
    
    More recently, Unity Catalog has introduced features such as UC functions and volumes, which allow controlled access not only to datasets but also to defined functionality. These capabilities make it possible to expose data and operations to automated workflows or AI-driven applications in a governed and auditable way.

3. **Distributed Compute Engine**

   This layer is responsible for executing queries and transformations across large datasets, supporting both batch and streaming workloads.

    In Databricks, Spark serves as the primary compute engine. However, other engines such as Trino, Flink, or Databricks SQL can also operate on a lakehouse, as long as they respect the transactional guarantees provided by the underlying table format.
    
    The key advantage of a distributed compute engine is its ability to automatically parallelize workloads across multiple nodes. Instead of manually partitioning data, orchestrating batch splits, or dealing with memory limitations on a single machine, distributed engines are designed to scale horizontally and handle large volumes of data more efficiently.
    
    If you frequently find yourself manually batching workloads, running into memory constraints, or repeatedly adjusting infrastructure to keep pipelines running, it may be a sign that your current setup has outgrown itself. In such cases, adopting a distributed engine like Spark can significantly simplify both performance optimization and operational complexity. ([Dont end up building your own engine](posts/backend-performance.md))

This brings us to the second key question when it comes to using Databricks:

_Do you have a usecase for most of these components?_ (more on that in the next section)

With this increased capability comes increased responsibility. Delta Lake metadata must be handled appropriately through careful data retention and optimization. Unity Catalog needs to be understood in terms of both its capabilities and proper usage. Distributed compute engines must also be used with knowledge of their architecture and best practices. 
Features like Delta’s automatic optimizations, Spark’s Adaptive Query Execution (AQE), and liquid clustering have simplified management compared with the past, but it is still not advisable to deploy these technologies on a team with little prior experience.

Which leads to another key question:

_Do you have the team, experience and knowledge to work with it?_

## Learn from the mistakes of other companies
Now that we have established a few key questions to ask when considering Databricks, let’s look at how these questions play out in practice.
Over the past few years, I’ve seen situations where organizations ignored each of them in one way or another. Sharing these experiences may help you evaluate your own situation more realistically — and hopefully avoid some common pitfalls.

1. **_Does your system or organization truly have the demands — both in terms of data volume and computational complexity — that justify this level of infrastructure?_**
    
    To confidently answer this question with yes, you should be able to check at least a few of the following statements:
    - You already experience — or expect to experience — performance issues with your current stack.
    - These issues are primarily caused by infrastructure limitations or increasing data volumes.
    - You manage a large number of tables and data pipelines that require orchestration.
    - Multiple teams depend on the same data platform.
    - Some of your datasets exceed 100 GB in size.
    - Your workflows go beyond simple ingestion and querying. For example:
      - transforming and remodeling data for downstream users 
      - generating new datasets from existing ones 
      - running machine learning pipelines 
      - processing real-time data streams

    **What I've seen in practice**: 
    In one case, a small two-person team decided they needed a Databricks environment to run some experimental pipelines on a 20 GB dataset.
    For a few weeks, they explored the data and experimented with different ideas — something that could easily have been done using many simpler tools. Eventually, the project was abandoned, and the Databricks environment remained mostly unused.

    Occasionally, someone would run a small experiment there, but it never became part of any production workflow, and nobody was responsible for maintaining or governing it.

    In situations like this, the order is often reversed.

    A better approach is usually to prototype using lightweight tools first, and only move to a platform like Databricks once the workload proves valuable enough to justify a production-grade environment.

    There is nothing wrong with prototyping in Databricks. However, if none of those experiments ever transition into production workloads, it may be a sign that the platform was introduced before the problem truly required it.

2. **_Do you have a usecase for most of its components?_**
   
    To answer this question with yes, you should be able to check at least a few of the following statements:Your current file-based storage system suffers from data quality issues due to a messy history or conflicting workflows.

   - Your current file-based storage system suffers from data quality issues due to a messy history or conflicting workflows. 
   - You work with multiple teams or stakeholders who require controlled access to shared datasets, making governance an important concern. 
   - Your current runtime environment struggles with parallelism or memory management when processing large workloads. 
   - Reads and writes in your current storage system have become too slow or expensive. 
   - You plan to make use of Databricks’ machine learning tooling and require scalable compute to run those workloads. 
   - Your environment contains a large number of tables and pipelines that need orchestration and coordination. 
   - Storing your data in a file-based format (e.g., object storage) is a viable option.
   
   One of the strongest counterindicators for adopting Databricks is when key datasets must remain in an external relational database for business or technical reasons. In such cases, the constant data movement between systems often turns I/O into a major bottleneck and undermines many of the advantages a lakehouse architecture provides. 

   If only two or three of the points above apply to your situation, it is worth remembering that many of Databricks’ core components are available as open-source technologies and can be used independently.

    For example:
   - If you primarily need distributed compute, you can run Spark on your existing infrastructure.
   - If data governance is your main concern, you may only need a metadata and governance layer.
   - If you need machine learning tooling, Spark’s MLlib is freely available.
   - If your goal is to introduce ACID transactions to object storage, table formats like Iceberg or Delta Lake can be used without adopting the full Databricks platform.
   
   Databricks becomes most valuable when you plan to use several of these components together and want a managed platform that integrates them and handles infrastructure, orchestration, and scaling.

    **What I've seen in practice:**

    In one project, a team adopted Databricks primarily for its automatic resource management and access to machine learning tooling. However, all of their data remained in an external PostgreSQL database. They had no practical use for Unity Catalog or Delta Lake, and most of their notebooks relied on Pandas and standard Python rather than Spark. 

    As a result, the platform’s main strengths — distributed processing and integrated data management — were never really utilized. Which leads to the final question...

3. **_Do you have the team, experience and knowledge to work with it?_**

     While marketing campaigns sometimes suggest that modern platforms make large-scale data processing accessible to everyone, in practice a lakehouse environment still requires experienced data professionals.

    Over the years, Databricks has certainly improved usability through better defaults, automation, and managed infrastructure. However, building and operating a reliable data platform still requires people who understand distributed computing, data governance, and modern data pipelines. Ideally, at least some members of the team should already have experience working with these concepts or with similar platforms.

     **What I've seen in practice:**

    In one case, a company’s management decided to acquire a Snowflake environment to replace their outdated company-wide storage system. Only after the platform had been set up did they realize that no one on the team had experience with anything like it. Most team members were skeptical about the decision, and there was neither a concrete migration plan nor the necessary expertise to operate the new environment.

    In another project, Databricks was introduced into a team that primarily worked with Pandas because they had little experience with Spark. As a result, most workloads ran entirely on the driver node while the executors remained largely unused. The platform technically worked, but many of its advantages — especially distributed computation — were never utilized.

    **The opposite Situation:**

    I have also seen the opposite scenario: teams that clearly had the need for a managed data platform but tried to solve the problem with minimal changes to their existing setup. 
    
    In one company, the entire data processing logic was implemented in PL/SQL and had grown into an undocumented codebase of more than 30,000 lines. The system generated around 2 TB of data per day, with some frequently accessed tables exceeding 200 GB.

    The team attempted to address performance issues by installing Spark on an on-premise machine while keeping their Oracle database as the central storage system. While Spark did provide some improvements, the external relational database quickly became a major I/O bottleneck. At the same time, the organization did not have a platform team with experience managing a distributed Spark cluster, which led to additional operational instability.

    In a situation like this, adopting a managed platform such as Databricks would likely have solved several of their most critical problems.


## Conclusion
Clearly, deciding whether or not to use Databricks can be tricky. Introducing it without a real use case can create unnecessary complexity, just as avoiding it despite clear limitations in your current infrastructure can hold a team back.

One thing my experience has taught me is that the decision to move to platforms such as Databricks, Snowflake, or BigQuery — especially as the boundaries between data warehouses and lakehouses become increasingly blurry — should largely come from the team working with the data.

In my experience, the biggest mistakes tend to happen when someone high up in management falls into the marketing trap of wanting the company to adopt a modern, powerful, AI-supported data platform, only to discover later that the team neither needs nor wants it.

The opposite can be just as problematic. If your data professionals tell you that your current infrastructure is reaching its limits and that an upgrade is necessary, it may be worth listening — even if you still have three years left on your Oracle license.