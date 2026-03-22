---
title: "Ethik-Domänen – Referenzdefinition"
brief: "Definition der 11 ethischen Domänen (Verletzungsgefahr, Gerechtigkeit, Empathie u. a.) mit Schwellwerten und Gewichtungen für das EthosAI-Ethiksystem."
status: final
version: "0.3.0"
author: Robert Alexander Massinger
date: 2024-09-30
history:
  - date: 2024-09-30
    change: "Erstversion der Ethik-Domänen-Referenz."
  - date: 2026-02-14
    change: "Front-Matter hinzugefügt; Status unverändert."
tags: [ethik, domänen, schwellwerte, clim, referenz]
code-release-versions:
  - "0.1.0"
  - "0.3.0"
implemented-features:
  - "11 Ethik-Domänen mit Schwellwerten und Gewichtungen"
  - "EthicsDomain Dataclass"
  - "EthicsDomains Registry"
  - "EthicsModule Orchestrierung"
fulfillment: "100%"
---

domains = [
    EthicsDomain(name="Verletzungsgefahr", threshold=0.7, importance=1.0),
    EthicsDomain(name="Reife (Jugendschutz)", threshold=0.7, importance=1.0),
    EthicsDomain(name="Identität / Integrität", threshold=0.7, importance=1.0),
    EthicsDomain(name="Selbstfürsorge", threshold=0.5, importance=0.7),
    EthicsDomain(name="Ganzheitliche Gesamtfürsorge", threshold=0.7, importance=1.0),
    EthicsDomain(name="Gerechtigkeit", threshold=0.7, importance=1.0),
    EthicsDomain(name="Nachhaltigkeit", threshold=0.6, importance=0.8),
    EthicsDomain(name="Autonomie", threshold=0.6, importance=0.8),
    EthicsDomain(name="Transparenz", threshold=0.5, importance=0.7),
    EthicsDomain(name="Empathie", threshold=0.6, importance=0.8),
    EthicsDomain(name="Verantwortung", threshold=0.7, importance=1.0)
]