# The Case for TDD in Data Pipelines
Claiming that unit tests are important, and that good test design and meaningful test coverage are desirable in software development, is among the least controversial statements you can make in software engineering.
There may be disagreements about what "proper" coverage means, which parts of a system deserve the most attention, or how tests should be implemented in practice. 
There may be even teams that openly admit they do little or no unit testing at all due to time pressure, delivery constraints, or technical debt.
But you will rarely hear a senior engineer explore a new code base and say:

    "Interesting... you decided to test your application. I don't think we should do that."

Yet for some reason, the world of data engineering often seems to ignore the existence of unit tests entirely.
In my experience, testing is frequently not even discussed as an option when building ETL jobs, Spark transformations, or streaming pipelines. Data quality checks may exist. End-to-end validation may exist. But actual unit tests for transformation logic are surprisingly rare.

In this article, I want to explore why that is, how unit tests can be applied effectively to data pipelines, and why data engineers should start treating them as a fundamental engineering practice rather than an afterthought.
But I don't want to stop at arguing for unit tests in general. In my most recent project, we experimented with applying 
Test-Driven Development (TDD) to our data pipelines - something that initially felt almost unnatural in a data engineering context. 
It ended up solving a surprising number of problems for us, and fundamentally changed how I think about building pipeline logic.

### Data Engineers, Why Didn't We Test Already?
I have worked at four different companies building data pipelines. None of them had a single unit test in place when I arrived.

At the time, I did not spend much energy questioning why that was. But reflecting on it now, I believe there are several contributing factors.

1. Even in traditional backend development, unit tests are often among the first things to get dropped when deadlines become tight. In environments where testing is not universally recognized as a core engineering practice, teams are even more likely to skip it entirely. Historically, data engineering evolved more from operations and analytics than from classical software engineering, and I think that cultural influence still shows today.
2. Many data pipelines are not perceived as "real software" in the same way backend services are. SQL transformations and Spark jobs often feel more declarative than imperative, which can lead teams to underestimate the need for proper software engineering practices like unit testing.
3. Data quality checks, sanity checks, manual validation, and end-to-end validations are already well-established practices in data pipelines. Because of that, many teams feel their systems are "tested enough."
4. Pipeline logic can also feel inherently harder to test. Inputs and outputs are often complex structures like DataFrames rather than simple scalar values or clearly isolated objects. That does introduce additional complexity, but it is far from unsolvable - and I will discuss practical ways to deal with it later on.

### Why we should test
Let's talk about why each of these reasons is actually not a real justification to not write unittest:

1. Culture or tradition is obviously a poor argument against unit testing. So let's move on...
2. It is true that many data pipelines are built using declarative languages or APIs such as SQL or Spark. However, these technologies are only declarative at the level of individual operations. As engineers, we still combine those operations into highly customized business logic, often spanning dozens or hundreds of transformations. At that level, data pipelines are fundamentally no different from traditional software systems. In both backend systems and data pipelines, the goal of unit testing is ultimately the same: identify the boundary between framework functionality and custom business logic - and test the custom logic aggressively.
3. Data quality checks, sanity checks, and similar validations are important. However, they primarily assess the overall validity of a pipeline’s output in a broad sense.
    
    When they fail, you know something is wrong - but you often have little information about where the problem originates. The issue could stem from corrupt input data, incorrect assumptions, a bug in any individual transformation, or even missing or duplicated processing steps.
    
    In that sense, these checks function more like end-to-end or integration tests: they can tell you that something is broken, but not precisely what or where.
    
    Because they operate at such a high level, it is also difficult to design them in a way that reliably captures all relevant failure modes. They are very effective for simple constraints - such as ensuring non-nullability or basic schema validity - but become significantly less precise when it comes to verifying more complex business logic.
    
    Another limitation is that they often fail too late. Ideally, we want to fail as early as possible. Relying solely on downstream sanity checks increases the likelihood that issues are only detected in production, where data is more complex, noisy, and unpredictable than in the mock datasets used during development.
4. Pipeline logic can initially feel harder to test than traditional application code. However, this is not a fundamental limitation. With the right design principles, it becomes entirely manageable. We will explore these later.


### Technical vs contextual complexity (and how TDD helps with the latter)
As mentioned above, non-trivial data pipelines and backend code are similar in the sense that both are software, and the arguments for unit testing largely apply to both in the same way.

That is not to say that the development experience in these two worlds is identical.

Generally speaking, backend systems often have higher technical complexity. You frequently have to deal with more abstraction layers, write more imperative code, handle concurrency or parallelism, work with a larger variety of entities and edge cases, and use patterns such as dependency injection, inheritance, or sophisticated state management.

But because of that, backend development often also has more contextual clarity.
You have the freedom to create your own abstractions, tailor your data models, and structure your application in a way that makes the intent of the system easier to understand - both for humans and for machines. A feature might be technically difficult to implement, but can often still be described clearly in both code and human language.

On the flip side, ETL pipelines often have lower technical depth. Partly because of the declarative nature of the dominant languages and frameworks. Partly because modern runtimes automatically handle many low-level concerns such as optimization, execution planning, and resource management for you.

But that does not necessarily make data engineering simpler overall.
The main challenge in batch processing is often the contextual complexity.

Your inputs are usually large, complex entities that you have limited control over, because upstream systems, customers, or other teams dictate the shape and semantics of the data. Schemas evolve continuously. Business definitions change. Assumptions that were valid yesterday silently become invalid tomorrow - especially during active development.

It is the large number of hidden rules, semantic assumptions, and undocumented contracts between these datasets - many of which require deep business understanding - that can make data engineering so difficult.

Traditional backend systems often allow complexity to be isolated more effectively through abstractions and separation of concerns. When done well, a new engineer can contribute productively to a code base without fully understanding the business domain immediately.

In data engineering, this is much harder.

With column-oriented and transformation-heavy logic - as is common in data pipelines - reasoning locally about the effects of a change becomes difficult. Pipelines and their individual transformations are often deeply interconnected and dependent on each other.

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
- The development process itself often becomes significantly clearer when the test is written first. Counterintuitively, writing the test before the implementation can even be faster overall than writing only the implementation directly. While designing the test, you are forced to understand the data, think through edge cases, clarify assumptions, and define the expected behavior precisely. In other words: the contextual complexity becomes clearer, and what remains afterwards is often “just” the technical implementation. The more context-heavy a system is, the more valuable this becomes.
- In the age of increasingly capable AI coding agents, well-written tests also become an extremely powerful form of specification. In some cases, the tests themselves already contain most of the context required for implementation. The engineer’s role shifts more toward precisely defining behavior and constraints than manually writing every line of implementation code.


### How we should test
Let's start by exploring why pipeline code can feel almost untestable. Consider the following code PySpark snippet:


(mention proper libraries like chispa)


### The advantages of TDD specifically

Unittests as a contract which data is expected by the pipeline. 
https://www.youtube.com/watch?v=TbWcCyP2MgE
was im TODO obsidian noch dazu steht

tests catch bigs that have already happened and prevent them from happining again. dont claim in this articla tests would prevent all bugs

### Conclusion
At the very least I advocate for having the same Testing Standards in Data Engineering as in "classical" Backend Development.
The one uses JOINs, GROUP BYs and Window Functions as individual building blocks, the other uses loops, if-statements and third party API calls. 
But in the end both build custom Software with custom logic and that needs to be tested regardless.
In both cases unit tests are an irreplacable way to not only identify even subtle bugs but also locate them and thus drastically increase long-term 
development speed as well as production stability.

In my experience High test coverage can be even more important in data pipelines due to the high interdependence of units. 
If input data, requirements or some assumption changes this can often affect multiple units in your code and helps avoiding the trap of a bugfix creating two new bugs.
Therefore, TDD specifically can help a lot with keeping test coverage high and reducing bugs by providing clearity on the expected inputs and outputs before the actual implementations.
Finally, it can provide your Coding Agent with essential context about your data, elevating it from a nice little helper to something implementing most - if not all - of your actual business logic.

TDD for complex data pipelines is great! You should try it.

