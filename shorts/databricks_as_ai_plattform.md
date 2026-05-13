| Date       | Target Audience                                                 |
|------------|-----------------------------------------------------------------|
| 13.03.2026 | Infrastructure decision makers, Software-, Data- & ML-Engineers |

# Is databricks about to become the dominant RAG platform?
Introduction: 
- More and more companies want to bild agentic systems and RAG applications
  - Over the past 2 years I had 2 similar and yet very different projects: Both wanted an Agentic/RAG Application, but:
    - 1 was building a RAG application almost from scratch with a python backend (we forked an open source RAG Project and went from there)
    - The other one used Databricks as a Platform for everything but the frontend.
    - The difference was shocking

Main Part:
- Self built Version Problems:
  - Hosted on premise in Kubernetes: Yes, self hosting is cheaper, but can it really justify the amount of money(wages) and work needed to manage your own on prem kubernetes cluster? But maybe you want to self host also for data privacy reasons, so fair enough.
  - Scraping performance: Scraping 20 different data source systems for hundreds of thousands of files was a big challenge performance wise. One time the database is the bottleck, then parallelism gives you a headache and once you fixed that you had network issues.
  - Serving performance: After going live our user base grew quickly. Needless to say that we ran into scalability issues once the user base had 10x increased while we were adding new experimental agentic features and support for more data sources.
  - OCR Issues: Different logic needed for all kinds of file types. pdfs need to use different functions to extract their contents properly than images or pptx presentations. Wanting to try, compare and imlement different OCR technique for .png files could take days.
  - Governance Issues: Adding the right metadata in the right way to ensure data governance was tricky. Testing whether it worked properly and if it didnt, why was even more complicated.
  - Monitoring & LLMOps complicated: To track performance, obtain user feedback, observe costs and compare all of the above among different models, retrieval strategies or chunking techniques required complicated solutions. Some of them took multiple weeks to implement.
  

- Then a while later I was working on another Project where we built a somewhat compareable Application with Databricks as its platform. The difference was impressive:
  - Hardly any infrastructure issues: Servers are reliable. Run your code serverless or on a server depending on what fits each peace of code better. And while IaC remains a part of Software Engineering I cant get myself to enjoy, databricks asset bundles made powerful CI/CD pipelines much simpler to set up than in our self hosted environment.
  - Scraping performance: No need to worry about how to run your code in parallel and orchestrate your workers. Just define your scraping function as udf and spark does the autoscaling for you. Ok, this might be a little bit oversimplified and at times we did have to think about parallel workflows and performance, but not to the same extent.
  - Serving performance: Exposing the final Models and Agents in a highly scalable way was almost no work at all.
  - OCR Issues: Databricks delivers strong inbuilt OCR functions which delivered not only high quality results but where also easy to use at scale.
  - Governance: Unity Catalog was very pleasant to use not only for who can see what but even who can DO what in agentic features (e.g. real time querying data or scraping code repositories). Very reliable, secure and easy to configure.
  - Monitoring & LLMOps: Easy to integrate Feedback tools for user feedback, llm as a judge or dedicated feedback sessions, mlflows automatic tracing including the option to add custom spans, easy to set up option to A/B test different models, retrieval strategies or chunking techniques.
  - Automatic Indexing: Another nice example of how databricks just makes your life a bit easier: You dont need to worry about hosting a vector DB and embed your contents in there. Databricks does this automatically under the hood on teh click of a button.
  - Guardrails and Security: Databricks had refreshingly simple yet powerful solutions to protecting against prompt injection, handling authentication or implementing proper guardrails for ratelimits

Summary:
- Building proper RAG applications is not all of a sudden completly frictionless and easy and you still cant build one over night - It remains a complex product and even on databricks we had at times bugs, blockers and questionmarks. But overall much less of them.
  - Yes Databricks on top of your hyperscaler is gonna be more expensive to host than your self hosted version on premise. WHether it outways the increased workload - especially in a high wages market like the German market - I cant say.
  - What i can say is that developing in a selfhosted environment was often painful and had almost certainly more blockers than a cloud native version would have had.
  - In any case: Databricks specifically seems to have shifted its strategy heavily to becoming the dominant platform fpr building your custom RAG and Agentic Applications and from my experience it really felt like they are getting there.
  - If you are intrigued but for some reason (maybe you prefer the privacy only self hosting can provide, maybe you want to avoid the vendor lock in or maybe you are simply already working with a different stack and just cant start over) cant use databricks, please note that many components are open source and can be incorporated in your custom stack such as mlflow for LLMops or unity catalog for data governance.


