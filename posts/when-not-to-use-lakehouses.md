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
   
   To answer this question with yes, you should be able to make checkmarks behind at least a few of the following statements:
  - Your current file based storage system suffers from quality issues due to a messy history or concurrent workflows.
  - You work with a diverse team or stakeholders with individual accessibility to the data, making data governance a complex but important issue.
  - Your current runtime environment struggles with parallelism and memory management.
  - Reads and Writes in your current storage system have become too expensive.
  - You need access to Databricks' Machine Learning Functionality and the compute to run it on.
  - You work in a complex landscape, with a large number of tables and pipelines.
  - Storing your data in a file-based format is possible for you. The biggest contraindicator against using databricks is, if for business reasons, some key tables you need to work with have to live in an external Database. Almost always, I/O will become a massive bottleneck for you in those cases.

   If only two or three of these points are applicable to you, please note that most of Databricks' components are open-source and can be used outside of Databricks. So if you only need a distributed compute engine, use Spark on whatever infrastructure you already use. If you mostly need data Governance, add Unity Catalog to your current setup. If you actually only need access to sparks Machine Learning functionality: Just download ML-lib for free and you're good. If you want to introduce ACID transactions to your S3, just use Iceberg or Deltalake. Getting Databricks makes sense, if you plan on working with multiple of these components and you want a platform on top of that, which manages all resources for you.
    The mistake I've seen in practice: A team getting databricks only for the automatic resource management and access to some ML functionality, but their data lived in an external postgres DB, they had no usecase for unity or deltalake and in their notebooks they mostly worked with pandas and python instead of Spark, which leads us to the last point...

3. **_Do you have the team, experience and knowledge to work with it?_**

     While Databricks' marketing team might want to tell you that in the age of AI, anybody can and should work with Databricks, I strongly disagree. While it is true that over the years they made many things simpler and found some strong defaults and auto configurations, you still need data professionals to handle a proper data landscape. And not only that: At least some of them need to have prior exposure to databricks, distributed compute and data governance.
        
     What I've seen in practice: A company's management acquiring a Snowflake environment to migrate their very outdated company wide storage. Only after the Snowflake set-up, they realize there is noone in the team who has ever worked with anything like it and most of the team does not support the decision. There was neither an action plan nor the personal to work with this new environment.
     And as mentioned above, I have seen databricks being introduced into a team that worked with pandas because they had not experience with spark, having all of their code run in the driver node and leaving the executors stale, leading to big inefficiencies.

On the flip-side I have also worked at a company once, who had a solid usecase for a managed data platform like Databricks:
Their Entire Logic was written in PL/SQL and had grown into a 30.000+ line undocumented monster. They generated 2TB of Data a day, 
with their largest frequently used table being 200GB large. They tried to solve it by just installing Spark on their on prem machine, 
but sticking to their OracleDB. Spark did help, but as mentioned above, the external relational DB always posed a massive challenge in terms of I/O bottleneck.
Moreover, there was no platform team that had experience with managing a spark cluster on prem, leading to instabilities there as well.
For them, switching to Databricks would have probably solved a couple of critical problems.

## Conclusion
Clearly, whether to use or not use Databricks can be a tricky decision. Using it, while not really having the usecase
for it, can lead to problems, just like avoiding it even though your current infrastructure is having big problems.

One thing my experience taught me is that the decision to move to Databricks, Snowflake, BigQuery (the borders between warehouses and lakehouses get increasingly blurry) or whatever platform you are
trying to adopt should come mostly from the team. The biggest mistakes happened in my experience when someone high up in management
falls into a marketing trap and wants to make sure the company uses a modern, powerful, AI-Supported Dataplatform only to learn 
that the team doesn't want it or need it. The same is true for the opposite: if your data professionals tells you, you need to upgrade, don't let 
the fact that you have 3 years left on your Oracle license stop you. 

