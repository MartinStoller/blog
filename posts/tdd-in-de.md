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
Test-Driven Development to our data pipelines - something that initially felt almost unnatural in a data engineering context. 
It ended up solving a surprising number of problems for us, and fundamentally changed how I think about building pipeline logic.

### Data Engineers, Why Didn't We Test Already?
I have worked at four different companies building data pipelines. None of them had a single unit test in place when I arrived.

At the time, I did not spend much energy questioning why that was. But reflecting on it now, I believe there are several contributing factors.

1. Even in traditional backend development, unit tests are often among the first things to get dropped when deadlines become tight. In environments where testing is not universally recognized as a core engineering practice, teams are even more likely to skip it entirely. Historically, data engineering evolved more from operations and analytics than from classical software engineering, and I think that cultural influence still shows today.
2. Data quality checks, sanity checks, manual validation, and end-to-end validations are already well-established practices in data pipelines. Because of that, many teams feel their systems are "tested enough." Later in this article, I will explain why I believe these approaches are not a substitute for proper unit tests.
3. Pipeline logic can also feel inherently harder to test. Inputs and outputs are often complex structures like DataFrames rather than simple scalar values or clearly isolated objects. That does introduce additional complexity, but it is far from unsolvable - and I will discuss practical ways to deal with it later on.
4. Many data pipelines are not perceived as "real software" in the same way backend services are. SQL transformations, orchestration configs, and Spark jobs often feel more declarative than imperative, which can lead teams to underestimate the need for proper software engineering practices like unit testing.

### Why we should test
Let's talk about 

### How we should test

### The advantages of TDD specifically

Unittests as a contract which data is expected by the pipeline. 
https://www.youtube.com/watch?v=TbWcCyP2MgE
was im TODO obsidian noch dazu steht
