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

### Why didn't we test already?
