| Date       | Target Audience                                                                 |
|------------|---------------------------------------------------------------------------------|
| 06.03.2026 | Data Engineers, Architects or anybody involved in strategic decisions like this |

# When (not) to move into a lake house 🌊🏡
Throughout my career as a data engineering consultant, I’ve seen modern lakehouse platforms such as Databricks introduced into organizations with great enthusiasm — and very mixed results. Not because the technology is flawed, but because the problems it solves are often misunderstood.

These experiences made me realize that the real challenge isn’t choosing the “most modern” platform. It’s understanding what problem you are actually trying to solve.

I’ve worked with teams that should have modernized their infrastructure but didn’t. I’ve also seen companies introduce the hottest big data platform on the market without having the use case — or the team — to support it.

This article is my attempt to explain how Databricks works, what problems lakehouse platforms are designed to solve, and what has — and hasn’t — worked in the projects I’ve been involved in. 
After reading this, you should not only have a better theoretical understanding of my favorite data platform, but also be better equipped to answer a more important question:

Is it a good fit for YOUR team?

## The Lakehouse Idea
To understand whether Databricks is the right tool for your team, it helps to first understand the problems that motivated its development.
Historically, organizations had two very different options for centralizing their data:

- Data Lakes – file-based storage systems designed for scalability and cost-efficiency. They can store large volumes of raw data but often lack structure, governance, and reliability.

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
    
    If you frequently find yourself manually batching workloads, running into memory constraints, or repeatedly adjusting infrastructure to keep pipelines running, it may be a sign that your current setup has outgrown itself. In such cases, adopting a distributed engine like Spark can significantly simplify both performance optimization and operational complexity. ([I wrote a full article about this](posts/backend-performance.md))

This brings us to the second key question when it comes to using Databricks:

_Do you have a usecase for all of these components?_ (more on that in the next section)

With this increased capability comes increased responsibility. Delta Lake metadata must be handled appropriately through careful data retention and optimization. Unity Catalog needs to be understood in terms of both its capabilities and proper usage. Distributed compute engines must also be used with knowledge of their architecture and best practices. 
Features like Delta’s automatic optimizations, Spark’s Adaptive Query Execution (AQE), and liquid clustering have simplified management compared with the past, but it is still not advisable to deploy these technologies on a team with little prior experience.

Which leads to another key question:

_Do you have the team, experience and knowledge to work with it?_

## Learn from the mistakes of other companies
Now that we determined three key questions to ask yourself, when considering to use Databricks, lets dissect and support them with actual experiences of mine.

I have experienced violations against all three questions in practice. Hopefully, sharing these experiences makes it easier for you to answer them for yourself and enables you to learn from the mistakes of others.

here i can put my horrorstories and summarize the decisive questions whether to use databricks or not

Examples to mention throughout article:

- I am not kidding: There is a German multi-billion dollar company storing almost all of its central data on a single physical machine, 
which stands in their DB Admins private basement. The good news is, they understood at some point that having all their data in this guy's basement, is not a
great idea and decided to fix it by setting up a Snowflake cloud environment. The bad news: Only after they had Snowflake all set-up, they realized
not only is there no plan at all on how to migrate the data there, but there is not a single person with Snowflake experience in the team.

- A company telling me proudly about their state-of-the-art modern cloud environment including Databricks "for some advanced Machine Learning workflows". 
All of their data lived in an external relational Database and noone in the team had ever written a single line 
of Spark Code. Neither using Databricks' Storage Systems nor its' primary Compute Engine, they essentially used it for nothing else than running regular python scripts that could be hosted anywhere.

- I once worked in a project which used 30.000+ lines of undocumented PL/SQL Code for a set of highly complex batch processing pipelines, generating multiple Terabytes a day.
Thankfully, we made the decision to refactor and migrate to PySpark, hoping for cleaner code and much better performance. However, they were unwilling to move away from their relational Database as the primary
storage system and refused to host spark anywhere but on their on premise server with scalability issues. At the same time they were unable to provide an infrastructure specialist 
to host, scale and maintain a spark cluster in that environment.

In the hopes of giving people a better understanding of when (not) to use modern lakehouse architectures, but at the very least as a coping 
mechanism for these traumatizing experiences, let me summarize how Databricks works, what
problems it solves and a few guidelines to figure out whether it's a proper tool for your team or not.

## Conclusion
Lakehouses have proven so powerful so that even Data Warehouses such as Snowflake or BigQuery keep adding lakehouse-like features.

Once you have a usecase for a lakehouse architecture the next questions would be which one to pick. This is a much more detailed but
also less important question. I might write another Snowflake vs Databricks article one day, but generally speaking you should usually
be fine with either of them.
