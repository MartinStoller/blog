# Warum Data Engineering TDD braucht
In der Softwareentwicklung käme niemand auf die Idee, komplexe Business-Logik ohne Unit-Tests in Produktion zu schicken. Im Data Engineering sieht die Realität oft anders aus: ETL-Jobs, Spark-Transformationen und Streaming-Pipelines laufen erstaunlich oft komplett ohne automatisierte Unit-Tests.
Warum ist das so – und warum bricht uns das am Ende das Genick?

# Das Kernproblem: Kontextuelle Komplexität
Während klassische Backend-Systeme oft eine hohe technische Komplexität besitzen (Concurrency, State Management, etc.), kämpfen Datenpipelines mit extremer kontextueller Komplexität.
- Wir kontrollieren die Form der Eingangsdaten selten selbst.
- Schemas und Business-Definitionen ändern sich ständig - insbesondere während der Entwicklung.
- Pipelines sind tief miteinander verwoben – eine kleine Änderung an einer Transformation kann "downstream" massive Ripple-Effekte auslösen.

# Warum "Datenqualitäts-Checks" nicht reichen
- Zu oberflächlich
- Zu langsam
- Zu ungenau

# Der Case für TDD (Test-Driven Development)
Unit-Tests eliminieren keine zukünftigen Bugs, aber sie konservieren Wissen. Sie machen versteckte Annahmen, Edge Cases und Marktregeln direkt im Code als ausführbare Dokumentation sichtbar.

Wenn wir Tests vor der Implementierung schreiben (TDD), zwingt uns das, die Datenstruktur und die Business-Logik vorab komplett zu durchdringen. Das Ergebnis:
- Weniger Regressionen: Alte Fehler tauchen nach Refactorings nicht plötzlich wieder auf.
- Besserer Code: Wir bauen automatisch modularere Funktionen (z. B. Column-Expressions statt gigantischer DataFrame-Funktionen).
- Perfekt für AI-Agents: Gut geschriebene Tests sind die ultimative, präzise Spezifikation, die moderne KI-Coding-Tools brauchen, um fehlerfreien Code zu generieren.


Ich könnte mir gut vorstellen, hierüber noch einen Tech-Talk für die nächste Konferenz-Saison vorzubereiten.
Was meint ihr dazu?

👇 Lust auf die Details?
Im vollständigen Artikel erfährst du, wie wir ein kollabierendes Tourismus-MVP kurz vor dem Go-Live durch TDD gerettet haben – inklusive konkreter PySpark- & SQL-Codebeispiele und Best Practices (wie der Nutzung von chispa).

