**Self-Hosting vs Databricks:**

In den letzten Jahren habe ich an zwei sehr ähnlichen agentic-RAG-Projekten gearbeitet. Technisch vergleichbares Zielbild, aber völlig unterschiedliche Realität im Engineering-Alltag.

Einmal:
Self-hosted, Kubernetes, eigener Ingestion-Stack, eigene OCR-Pipelines, eigenes Monitoring.

Einmal:
Databricks.

Meine Erfahrungen und ein etwas ausführlicherer Vergleich:
https://github.com/MartinStoller/blog/blob/main/posts/databricks_as_ai_plattform.md

TL;DR

Mein Eindruck nach beiden Projekten:
- Gerade in Deutschland wirkt „wir sparen Lizenzkosten und hosten selbst“ oft rational - bis mehrere Senior Engineers anfangen, ihre Zeit hauptsächlich in Plattformbetrieb statt Produktentwicklung zu investieren.
- Managed AI-Plattformen kaufen dir nicht nur Infrastruktur ein, sondern vor allem Fokus und Engineering-Geschwindigkeit.

Wobei man fairerweise sagen muss:
Ein Teil meines positiven Eindrucks kam sicher daher, dass ich bereits Erfahrung mit Databricks und PySpark hatte. Wer neu in dem Ökosystem ist, wird die Lernkurve definitiv spüren.

Meiner Erfahrung nach bieten managed Platforms also nicht nur höhere Entwicklungsgeschwindigkeit und stabileren Betrieb, sondern potentiell auch signifikante Kostenersparnis. 

Wer sich aufgrund von Vendor Lock-in, Datenschutz oder regulatorischer Anforderungen trotzdem für self-hosting entscheidet, dem sei gesagt: Viele der nützlichsten Databricks Komponenten, wie etwa MLflow oder Unity Catalog, sind open-source und somit auch unabhängig von Databricks nutzbar.


