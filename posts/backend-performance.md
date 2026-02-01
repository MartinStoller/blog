# Backend-Performance in High-Throughput Systems - A Data Engineer's Perspective
Throughout my career in IT consulting, I have seen this pattern repeat itself over and over again:
Many high-load systems start out as small, inconspicuous prototypes.

In their early stages—when fundamental decisions about architecture, technologies, and design are made—the scale at which 
these systems will eventually operate is often ignored or simply impossible to anticipate.

As a result, they are typically built using the default patterns of classical backend development:

- a relational database with a highly normalized data model,
- an imperative, deterministic, object-oriented codebase,
- row-wise, case-by-case business logic,
- and a request–response architecture exposed via REST APIs and optimized for minimal latency.

These defaults exist for good reasons, and in many scenarios they work extremely well. However, some systems—especially 
those facing massive data volumes or extreme request rates—are better served by a fundamentally different approach.

In this article, we will explore how to identify such systems and how backend developers can address performance challenges 
in high-throughput environments by deliberately breaking away from their default mental models. 
In particular, we will examine how core principles and technologies from the field of data engineering—such as batch 
processing, declarative computation, and denormalized data models—can be applied to build backend systems with 
near-unbounded scalability.

## Aunt Emma
Let us begin with an imaginary example, which is heavily based on real Experiences of mine...

Let´s imagine our Aunt Emma has a small business. One Store, Two Employees, only a hand full of different Products... nothing fancy.
For many of her daily business decisions she relies on KPIs, Sales Forecasts and Price Recommendations by quantitative
business models. This is a lot of work to calculate by hand or with her limited Excel skills, so as the good niece/nephew 
we are, we decide to help her out and automate the process in a Web App.

You can imagine the UI looking more or less like this (Thanks, ChatGPT!):
![img.png](../assets/ui_aunt_emma.png)
Essentially, the App consists of 2 components:
- An overview, which lists all sold products with a little bit of preview data (left side),
- When clicking on a product, we see all available KPIs and data for that product (right side).

Our first step is to pick a proper architecture. We go for what I would call a simple and intuitive approach:




Note this: How hard it can be sometimes to anticipate whether something will every become a high-load system or foreever 
remain a small nice prototype.

