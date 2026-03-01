# When (not) to use Databricks
Throughout my career as a data engineering consultant, I’ve seen modern lakehouse platforms such as Databricks introduced into organizations with great enthusiasm — and very mixed results. Not because the technology is flawed, but because the problems it solves are often misunderstood.

These experiences made me realize that the real challenge isn’t choosing the “most modern” platform. It’s understanding what problem you are actually trying to solve.

I’ve worked with teams that should have modernized their infrastructure but didn’t. I’ve also seen companies introduce the hottest big data platform on the market without having the use case — or the team — to support it.

This article is my attempt to explain how Databricks works, what problems lakehouse platforms are designed to solve, and what has — and hasn’t — worked in the projects I’ve been involved in. 
After reading this, you should not only have a better theoretical understanding of my favorite data platform, but also be better equipped to answer a more important question:

Should you use it at all?





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

