# Designing high-throughput systems - what backend developers can learn from data engineering

One pattern I've repeatedly seen throughout my career in IT consulting:

Many high-load systems start as small, harmless prototypes.

At the beginning, nobody expects terabytes of data, millions of requests, or hour-long processing pipelines. So naturally, systems are built using the default patterns of traditional backend development:

* relational databases
* normalized data models
* object-oriented services
* request–response architectures
* row-wise business logic

And to be clear: these defaults exist for good reasons. In many cases, they are exactly the right choice.

But some systems eventually hit a point where optimizing “within the paradigm” is no longer enough.

I've often seen teams respond to scalability problems by reaching for familiar tools:
better algorithms, caching, more hardware, more parallelism, GC tuning, chunking, etc.

Sometimes that helps.

But often the bigger gains come from questioning the underlying assumptions entirely.

Should this computation happen on demand at all?
Do we actually need normalized models here?
Would batch processing fit better than request-driven execution?
Are we solving an OLAP problem with OLTP thinking?

That's where ideas from data engineering become incredibly valuable.

**On June 18th, I'll be speaking at the Java User Group Nürnberg** about exactly this topic in my talk:

_“Backend Performance in High-Throughput Systems - A Data Engineer's Perspective”_

Using a simple prototype system that gradually scales from “small business app” to “Walmart-scale nightmare”, I'll walk through:

* when request–response architectures start breaking down
* why precomputation and batch processing can fundamentally change scalability
* where declarative and column-oriented processing shines
* how denormalization changes large-scale system design
* and why technologies like Parquet, Iceberg, Spark, Trino, or DuckDB matter far beyond “big data” teams

The talk is aimed at backend developers who want a more practical understanding of *when* data-engineering patterns become useful - and how to apply them without turning every project into a giant data platform.

Really looking forward to the discussion.
