| Date       | Target Audience                                          |
|------------|----------------------------------------------------------|
| 27.05.2026 | Data Engineers & Everyone writing complex Data Pipelines |


# The Case for TDD in Data Pipelines
Claiming that unit tests are important, and that good test design and meaningful test coverage are desirable in software development, is among the least controversial statements you can make in software engineering.
There may be disagreements about what "proper" coverage means, which parts of a system deserve the most attention, or how tests should be implemented in practice. 
There may be even teams that openly admit they do little or no unit testing at all due to time pressure, delivery constraints, or technical debt.
But you will rarely hear an experienced software engineer claim that testing would be unnecessary or undesirable.

Yet for some reason, the world of data engineering often seems to ignore the existence of unit tests entirely.
In my experience, testing is frequently not even discussed as an option when building ETL jobs, Spark transformations, or streaming pipelines. Data quality checks may exist. End-to-end validation may exist. But actual unit tests for transformation logic are surprisingly rare.

In this article, I want to explore why that is, how unit tests can be applied effectively to data pipelines, and why data engineers should start treating them as a fundamental engineering practice rather than an afterthought.
But I don't want to stop at arguing for unit tests in general. In my most recent project, we experimented with applying 
Test-Driven Development (TDD) to our data pipelines - something that initially felt almost unnatural in a data engineering context. 
It ended up solving a surprising number of problems for us, and fundamentally changed how I think about building pipeline logic.

### TDD! What madness drove them in there?
I am not going to lie: until about a year ago, I was also guilty of not prioritizing tests nearly enough.
And even when building "classical" backend systems, I had never seriously considered using Test-Driven Development.

That changed in my most recent project.

We were building custom pricing models for tourism companies from scratch. Customers would provide highly individual input data about bookings, capacities, prices, seasonal trends, and more. 
Based on that, we built fairly complex batch-processing pipelines that computed daily pricing updates for thousands of bookable units.

The project followed a highly iterative process.
We developed an initial prototype, reviewed bugs and unexpected behavior together with the customer, adjusted the business logic, 
and gradually evolved the system into a production-ready MVP. From there, we kept refining and extending it until the 
final product emerged. 

This meant we constantly revisited the same functions over and over again. Over time, edge cases, dependencies, and hidden business rules started piling up. Eventually it became difficult to remember 
why certain features had been implemented in the first place. Why exactly did we apply rule X only to the British market during weekends?
Was this behavior intentional? Was it a workaround for an earlier issue? Or was it simply a bug nobody had noticed yet?

Two weeks before go-live, everything looked great. The customer had only identified two remaining minor issues related to special pricing rules during the Christmas season. Everything else appeared stable.
I pushed what was supposed to be one of our final fixes to the staging environment. That fix reintroduced an older issue.
Fixing that issue caused another regression. Fixing that triggered even larger problems somewhere else in the pipeline.

What followed were two consecutive 60-hour work weeks, multiple emergency meetings, and a painful attempt to reconstruct decisions and assumptions from months earlier.

Thankfully, shortly before go-live, we managed to untangle the situation and stabilize the system again. 
In the end, the problem turned out to be a combination of hidden regressions, deeply interconnected logic, and contradictory business requirements that had accumulated over time.

After the successful launch, I spent some time reflecting on what had actually happened.
What allowed a nearly finished system to destabilize so dramatically from what initially looked like a relatively small change?
And more importantly:
How could we prevent this from happening again - not only in this system, but in future data pipeline projects as well?

My conclusion was that proper unit testing could have prevented a large part of this chaos.
Not because tests magically eliminate bugs, but because they preserve knowledge.
Every edge case, bug fix, business rule, and previously discovered failure mode could have been documented directly in the codebase as executable behavior. Instead of relying on tribal knowledge and memory, the system itself could have continuously verified whether those assumptions still held true.

And that naturally led me toward TDD.

We tried it, and it worked surprisingly well. Whenever a ticket required changes to a transformation, we started by translating the expected behavior into a test. Over time, this made the system noticeably more stable and cut down on regressions. It also made the business logic itself easier to reason about and brought a few benefits I hadn't anticipated at all...

### Data Engineers, Why Don't We Test?
I have worked at four different companies building data pipelines (among other things). None of them had a single unit test in place when I arrived.

At the time, I did not spend much energy questioning why that was. But reflecting on it now, I believe there are several contributing factors.

1. Even in traditional backend development, unit tests are often among the first things to get dropped when deadlines become tight. In environments where testing is not even universally recognized as a core engineering practice, teams are even more likely to skip it entirely. Historically, data engineering evolved more from operations and analytics than from classical software engineering, and I think that cultural influence still shows today.
2. Many data pipelines are not perceived as "real software" in the same way backend services are. SQL transformations and Spark jobs often feel more declarative than imperative, which can lead teams to underestimate the need for proper software engineering practices like unit testing.
3. Data quality checks, sanity checks, manual validation, and end-to-end validations are already well-established practices in data pipelines. Because of that, many teams feel their systems are "tested enough."
4. Pipeline logic can also feel inherently harder to test. Inputs and outputs are often complex structures like DataFrames rather than simple scalar values or clearly isolated objects. That does introduce additional complexity, but it is far from unsolvable - and I will discuss practical ways to deal with it later on.

### Why we should test
Let's talk about why each of these reasons is actually not a real justification to not write unit tests:

1. Culture or tradition is obviously a poor argument against unit testing. So let's move on to more rational arguments.
2. It is true that many data pipelines are built using declarative languages or APIs such as SQL or Spark. However, these technologies are only declarative at the level of individual operations. As engineers, we still combine those operations into highly customized business logic, often spanning dozens or hundreds of transformations. At that level, data pipelines are fundamentally no different from traditional software systems. In both backend systems and data pipelines, the goal of unit testing is ultimately the same: identify the boundary between framework functionality and custom business logic - and test the custom logic aggressively.
3. Data quality checks, sanity checks, and similar validations are important. However, they primarily assess the overall validity of a pipeline’s output in a broad sense.
    
    When they fail, you know something is wrong - but you often have little information about where the problem originates. The issue could stem from corrupt input data, incorrect assumptions, a bug in any individual transformation, or even missing or duplicated processing steps.
    
    In that sense, these checks function more like end-to-end or integration tests: they can tell you that something is broken, but not precisely what or where.
    
    Because they operate at such a high level, it is also difficult to design them in a way that reliably captures all relevant failure modes. They are very effective for simple constraints - such as ensuring non-nullability or basic schema validity - but become significantly less precise when it comes to verifying more complex business logic.
    
    Another limitation is that they often fail too late. Ideally, we want to fail as early as possible. Relying solely on downstream sanity checks increases the likelihood that issues are only detected in production, where data is more complex, noisy, and unpredictable than in the known datasets used during development.
    In data-intensive environments, failing late is particularly problematic because feedback cycles are slower, reruns are expensive, and downstream systems may already depend on corrupted outputs.
4. Pipeline logic can initially feel harder to test than traditional application code. However, this is not a fundamental limitation. With the right design principles, it becomes entirely manageable. We will explore these later.

### Technical vs contextual complexity (and how TDD helps with the latter)
As mentioned above, non-trivial data pipelines and backend code are similar in the sense that both are software, and the arguments for unit testing largely apply to both in the same way.

That is not to say that the development experience in these two worlds is identical.

Generally speaking, backend systems often have **higher technical complexity**. You frequently have to deal with more abstraction layers, write more imperative code, handle concurrency or parallelism, work with a larger variety of entities and edge cases, and use patterns such as dependency injection, inheritance, or sophisticated state management.

But because of that, backend development often also has **more contextual clarity**.
You have the freedom to create your own abstractions, tailor your data models, and structure your application in a way that makes the intent of the system easier to understand - both for humans and agents.

On the flip side, ETL pipelines often have **lower technical depth**. Partly because of the declarative nature of the dominant languages and frameworks. Partly because modern runtimes automatically handle many low-level concerns such as optimization, execution planning, and resource management for you.
But that does not necessarily make data engineering simpler overall.
The main challenge in batch processing is often the **contextual complexity**.

Your inputs are usually large, complex entities that you have limited control over, because upstream systems, customers, or other teams dictate the shape and semantics of the data. Schemas evolve continuously. Business definitions change. Assumptions that were valid yesterday silently become invalid tomorrow - especially during active development.

It is the large number of hidden rules, semantic assumptions, and undocumented contracts between these datasets - many of which require deep business understanding - that can make data engineering so difficult.

Traditional backend systems often allow complexity to be isolated more effectively through abstractions and separation of concerns. When done well, a new engineer can contribute productively to a code base without fully understanding the business domain immediately.

In data engineering, this is much harder. With column-oriented and holistic logic - as is common in data pipelines - reasoning locally about the effects of a change becomes difficult. Pipelines and their individual transformations are often deeply interconnected and dependent on each other.
As a result, even small changes in code or business requirements can create surprisingly large ripple effects throughout the system.

This is where unit tests become incredibly valuable.
Not only do they help ensure consistent behavior and protect against regressions, they also act as executable documentation 
for many of these hidden assumptions and dependencies. Writing tests forces engineers to formalize expectations explicitly, 
think about edge cases earlier, and clarify business semantics while implementing the logic itself.

Technically, you could argue that it does not matter much whether tests are written before the implementation (TDD) or afterwards. And to some extent, I agree.
As Tech Lead in my last project, I told my team that unit tests were mandatory, but that they could decide for themselves whether to write them before or after the implementation.

At the same time, I also told them that in practice I see several strong arguments for writing tests rigorously before implementation:

- Once the implementation is finished, most developers naturally want to move on. There is a strong psychological tendency to think: 
    "It works now, so why spend more time writing tests?" As a result, tests written afterwards are often fewer in number, lower in quality, or skipped entirely - especially under time pressure. 
- The development process itself often becomes significantly clearer when the test is written first. Counterintuitively, writing the test before the implementation can even be faster overall than writing only the implementation directly. While designing the test, you are forced to understand the data, think through edge cases, clarify assumptions, and define the expected behavior precisely. In other words: the contextual complexity becomes clearer, and what remains afterwards is often a relatively simple technical implementation. The more context-heavy a system is, the more valuable this becomes.
- In the age of increasingly capable AI coding agents, well-written tests also become an extremely powerful form of specification. In some cases, the tests themselves already contain most of the context required for implementation. The engineer’s role shifts more toward precisely defining behavior and constraints than manually writing every line of implementation code.

Summarizing, with TDD we usually get better tests, more reliable implementations and (potentially) a faster development process, while
also documenting many assumptions in our code, which can be used both by developers, other stakeholders and AI.

### How we should test
So far this was all a little abstract. Let’s have a look at some concrete examples.

We start by exploring why pipeline code can feel hard to test. Similar to regular backend development, the testability of your code is an important indicator of its quality.

Consider this example, where we want to convert German umlauts (ä, ö, ü to ae, oe, ue) and extract city and street from an address string:

So we want to transform this:

| Cust_id | Address                                   |
|--------:|-------------------------------------------|
| 1 | Max-Schär-Str, 50733, Köln, Germany       |
| 2 | Eichenweg, 80909, München, Germany        |
| 3 | Klötzlmüllerstr, 84034, Landshut, Germany |

to this:

| Cust_id | street             | city     
|--------:|--------------------|------------|
| 1 | Max-Schaer-Str     | Koeln     |
| 2 | Eichenweg          | Muenchen  |
| 3 | Kloetzlmuellerstr  | Landshut  |

A developer might attempt to solve this with the following function:
```python
def extract_cleaned_address_details(df: DataFrame) -> DataFrame:
    # Replace German umlauts
    replacements = {
        "ä": "ae",
        "ö": "oe",
        "ü": "ue"
    }

    address_clean = reduce(
        lambda col, kv: F.regexp_replace(col, kv[0], kv[1]),
        replacements.items(),
        F.col("Address")
    )
    
    # Split by comma
    parts = F.split(address_clean, r",\s*")
    
    # Build transformed dataframe
    result = (
        df.withColumn("parts", parts)
          .withColumn("street", F.col("parts")[0])
          .withColumn(
              "city",
              F.when(F.size("parts") == 4, F.col("parts")[2])
               .otherwise(F.col("parts")[1])
          )
          .select("Cust_id", "street", "city")
    )

    return result
```

We can immediately see how this code is not easy to read (especially without comments helping out).
This is mainly due to two things:

1. Our function has two tasks: cleaning and splitting the string.
2. The functionality for handling the umlauts returns a full DataFrame when it actually only needs to return a single Column.

If we instead write the two tasks into separate functions and turn the umlaut parsing function into a column expression, we will immediately see how we introduce reusability, better readability, and testability:
```python
def parse_umlauts(col: Column) -> Column:
    replacements = {
        "ä": "ae",
        "ö": "oe",
        "ü": "ue"
    }

    return reduce(
        lambda c, kv: F.regexp_replace(c, kv[0], kv[1]),
        replacements.items(),
        col
    )

def extract_cleaned_address_details(df: DataFrame) -> DataFrame:
    parts = F.split(F.col("Address"), r",\s*")

    return df.select(
        "Cust_id",
        parse_umlauts(parts[0]).alias("street"),
        parse_umlauts(parts[2]).alias("city")
    )
```
This is just a demo, so don't get hung up if you would have written these functions differently.
The point is: be very careful about the single responsibility principle, and be aware of functions returning DataFrames when they could just return columns.

During development, I always try to think: what is the minimum viable input and output of a function. This also helps avoid growing individual functions over time as new requirements are introduced.

So now that we have testable code, how do we actually test it?

PySpark does have a testing module, but generally I like to use the [chispa](https://github.com/MrPowers/chispa) library for most assertions.

When writing tests, I usually start by classifying whether I am writing a test for a column expression or a DataFrame-returning function.

This is because column expressions can be nicely asserted by comparing two columns of the same DataFrame like so:
```python
def test_parse_umlauts():
    data = [
        ("Köln", "Koeln"),
        ("München", "Muenchen"),
        ("Schär", "Schaer"),
        ("Klötzlmüllerstr", "Kloetzlmuellerstr"),
        (None, None)
    ]

    df = (
        spark.createDataFrame(data, ["city", "expected_city"])
        .withColumn("clean_city", parse_umlauts(F.col("city")))
    )

    assert_column_equality(df, "clean_city", "expected_city")
```

DataFrame functions, on the other hand, usually require asserting equality between two DataFrames:
```python
def test_extract_cleaned_address_details():
    source_data = [
        (1, "Max-Schär-Str, 50733, Köln, Germany"),
        (2, "Eichenweg, 81122, München, Germany"),
        (3, "Klötzlmüllerstr, 84034, Landshut, Germany"),
        (4, None)
    ]

    source_df = spark.createDataFrame(source_data, ["Cust_id", "Address"])

    actual_df = extract_cleaned_address_details(source_df)

    expected_data = [
        (1, "Max-Schaer-Str", "Koeln"),
        (2, "Eichenweg", "Muenchen"),
        (3, "Kloetzlmuellerstr", "Landshut"),
        (4, None, None)
    ]

    expected_df = spark.createDataFrame(
        expected_data,
        ["Cust_id", "street", "city"]
    )

    assert_df_equality(actual_df, expected_df)
```

Another practical tip for testing: make sure you have a proper local Spark configuration for your tests. This mostly means allocating enough memory to your setup and reducing the default shuffle partitions to around 2 (instead of the default 200, which is therefore much less efficient).

Moreover, you obviously do not have to hardcode your test data directly in the test. Especially when reusing data or working with larger datasets, it can make sense to load test fixtures from external files.
However, in my experience, large or complex test datasets inside a unit test are often a warning sign. They can indicate that the function under test is doing too much, or that its logic is too hard to reason about in isolation.
Keeping test data directly inside the test also improves readability and makes it easier to understand the intent at a glance. For that reason, I prefer this as the default approach, and only move test data into external fixtures when there is a clear benefit.

##### Testing SQL

Using PySpark, you can even test your SQL queries. The concept is quite similar. Let’s use the PySpark testing package instead of chispa this time:
```python
from pyspark.testing import assertDataFrameEqual

query = f"SELECT * from {df} where age > {age}"  # arbitrarily complex SQL query

test_data = spark.createDataFrame(
    [
        (1, "Hans", 22),
        (2, "Franz", 30),
        (3, "Günther", None),
    ],
    ["id", "name", "age"],
)

expected = spark.createDataFrame(
    [
        (2, "Franz", 30),
    ],
    ["id", "name", "age"],
)

actual = spark.sql(query, df=test_data, age=25)
assertDataFrameEqual(actual, expected)
```
Writing these tests before implementing the actual transformation forces you to treat your data schemas as a strict contract. By defining the exact input shapes and expected outputs upfront, you stop guessing how a transformation should behave. Instead, the test suite becomes the single source of truth for how data moves through your system – a contract that is automatically verified every time you run your CI/CD pipeline.

### Managing Expectations
In my experience, unit tests are sometimes misunderstood as a mechanism that somehow prevents new bugs from appearing altogether.

That is usually not how testing works in practice.
Most bugs are, by definition, unexpected. You typically did not write a test for a bug you could not yet anticipate.

The primary value of unit tests is therefore not that they magically eliminate all future bugs, but that they document and preserve behavior that is already known and understood. Once a bug has been discovered and fixed, a corresponding test ensures that the same regression does not silently reappear later.

This becomes especially important in ETL pipelines, where systems are often highly interconnected and tightly coupled. Seemingly small changes in one transformation can create unexpected side effects throughout the rest of the pipeline.
In this kind of environment, a strong test suite becomes less about proving absolute correctness and more about creating confidence that changes do not unintentionally break existing behavior.

### Conclusion
Observability may ultimately matter more than unit testing in many data systems because real-world data is messy and unpredictable. But that does not make unit tests unimportant. It simply means that data engineering requires multiple complementary layers of validation rather than relying on a single strategy alone.
I advocate for applying at least similar testing standards in data engineering as we already expect in “classical” backend development.

One world uses JOINs, GROUP BYs, and window functions as its primary building blocks. The other relies more heavily on loops, conditionals, state management, and third-party APIs. But in the end, both domains build custom software containing custom business logic - and custom logic needs to be tested regardless of the underlying technology.

Specifically writing Tests **before** their implementation worked very well for us. I am looking forward to gathering even more experience with TDD. 

Of course, every system is different, and no engineering practice is universally optimal in every situation. My argument is not that TDD magically solves all problems in data engineering.
But I do believe this:

TDD is massively underrated in data pipelines. Most ETL teams probably have never seriously tried it. They should.