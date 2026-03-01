# When (not) to use Databricks
I am not kidding: There is a German multi-billion company storing almost all of its central data on a single physical machine, 
which stands in their DB Admins private basement.

Modern Lakehouse Platforms such as Databricks or Snowflake are amazingly powerful and as a Data Engineer they are currently 
under my favorite tools to work with. But the way I`ve seen them (not) being used or miss-understood by multiple companies
was at times shocking. Here are three examples:

- A company telling me proudly about their state-of-the-art modern cloud environment including Databricks "for some advanced Machine Learning workflows". 
All of their data lived in an external relational Database and noone in the team had ever written a single line 
of Spark Code. Neither using Databricks' Storage Systems nor its' primary Compute Engine, they essentially used it for nothing else than running regular python scripts that could be hosted anywhere.

- I once worked in a project which used 30.000+ lines of undocumented PL/SQL Code for a set of highly complex pipelines, generating multiple Terabytes a day.
Thankfully, we made the decision to refactor and migrate to PySpark, hoping for cleaner code and much better performance. However, they were unwilling to move away from their relational Database as the primary
storage system and refused to host spark anywhere but on their on premise server with scalability issues. At the same time they were unable to provide an infrastructure specialist 
to host, scale and maintain a spark cluster in that environment.

- Regarding the company I mention in the beginning: The good news is, they understood at some point that having all your data in this guy's basement, is not a
great idea and wanted to fix it by setting up Snowflake. The bad news: Only after they had Snowflake all set-up, they realized
not only is there no plan at all about how to migrate the data there, but there is noone who can work with it.

In the hopes of giving people a better understanding of when (not) to use modern lakehouse architectures, but at the very least as a coping 
mechanism for these traumatizing experiences of mine, let me summarize of how Databricks works, what
problems it solves and a few guidelines to figure out whether it's a proper tool for your team or not.

