**Self-Hosting vs Databricks in enterprise RAG: Die versteckten Kosten von 'gratis' Infrastruktur**

In den letzten Jahren habe ich an zwei ähnlichen agentic-RAG-Projekten gearbeitet. Technisch vergleichbares Zielbild, aber völlig unterschiedliche Realität im Engineering-Alltag.

Einmal:
Self-hosted, Kubernetes, eigener Ingestion-Stack, eigene OCR-Pipelines, eigenes Monitoring.

Einmal:
Databricks.

Meine Erfahrungen und Learnings habe ich ausführlich in einem Blog-Artikel zusammengefasst (Link in den Kommentaren).

**TL;DR**

"Wir sparen Lizenzkosten und hosten selbst" klingt oft rational - bis mehrere Senior Engineers anfangen, ihre Zeit hauptsächlich in Plattformbetrieb statt Produktentwicklung zu investieren. 
Managed AI-Plattformen kaufen dir nicht nur Infrastruktur ein, sondern vor allem Fokus und Engineering-Geschwindigkeit. Dadurch kann man mit ihnen sogar Kosten einsparen.

Wobei man fairerweise sagen muss:
Ein Teil meines positiven Eindrucks kam sicher daher, dass ich bereits Erfahrung mit Databricks und PySpark hatte. Wer neu in dem Ökosystem ist, wird die Lernkurve definitiv spüren.

Wer sich aufgrund von Vendor Lock-in, Datenschutz oder regulatorischer Anforderungen trotzdem für self-hosting entscheidet, dem sei gesagt: Viele der nützlichsten Databricks Komponenten, wie etwa MLflow oder Unity Catalog, sind open-source und somit auch unabhängig von Databricks nutzbar.

Ich freue mich über andere Erfahrungsberichte und Meinungen in den Kommentaren :)
