---
title: "Learning Packages — Scalable Extension Architecture"
brief: "Konzept für skalierbare Lernpakete: Wissen, Fähigkeiten und Trainingsdaten als installierbare Module auf EU-Servern."
status: draft
version: "1.5.0"
author: Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)
date: 2026-02-15
history:
  - date: 2026-02-15
    change: "Erstversion des Learning Package Konzepts."
tags: [learning, packages, extension, skalierung, eu-server, training]
code-release-versions:
  - "1.5.0"
implemented-features: []
fulfillment: "0%"
fulfillment-note: "Konzeptdokument — Implementierung in v1.5.0 geplant."
---
# Learning Packages — Scalable Extension Architecture

**Version:** 1.0.0  
**Datum:** 2026-02-15  
**Autoren:** Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)  
**Status:** Draft — Zielrelease v1.5.0

---

## 1 · Motivation

EthosAI's Wachstum folgt dem Maturity Model: Neue Fähigkeiten entstehen durch Lernen, keine statische Programmierung. Dafür braucht es ein standardisiertes System, um **Wissen, Fähigkeiten und Trainingsdaten** als installierbare Module bereitzustellen.

### Anforderungen

| Anforderung | Lösung |
|---|---|
| **Skalierbar** | Hunderte Pakete gleichzeitig verwaltbar |
| **Plattform-unabhängig** | Auf EthosAI selbst + externer Server |
| **EU-konform** | DSGVO, Hosting auf EU-Servern |
| **Signiert** | Kryptographisch verifizierte Integrität |
| **Versioniert** | SemVer mit Dependency-Resolution |
| **Isoliert** | Pakete beeinflussen sich nicht gegenseitig |

---

## 2 · Package-Struktur

### 2.1 Ein Learning Package besteht aus:

```
learning_package/
├── package.yaml          # Metadaten, Version, Dependencies
├── knowledge/            # Wissensmodule
│   ├── formulas.yaml     # Formelsammlungen
│   ├── standards.yaml    # Normen und Richtlinien
│   ├── materials.json    # Werkstoff-Daten
│   └── domain_text.md    # Fachtext (für RAG-Embedding)
├── skills/               # Fähigkeiten (Code)
│   ├── __init__.py
│   ├── capability_A.py   # Neue Capability-Implementierung
│   └── capability_B.py
├── training/             # Trainingsdaten
│   ├── scenarios.jsonl   # CLIM-Trainingsszenarien
│   ├── conversations.jsonl # Dialog-Trainingsbeispiele
│   └── test_cases.json   # Validierungsszenarien
├── assessment/           # Prüfungsszenarien
│   ├── exam.yaml         # Prüfungsfragen + Erwartungen
│   └── rubric.yaml       # Bewertungsschema
├── signature.ed25519     # Kryptographische Signatur
└── CHANGELOG.md          # Änderungshistorie
```

### 2.2 package.yaml

```yaml
name: mechanical-engineering-basics
version: "1.0.0"
display_name: "Maschinenbau — Grundlagen"
description: "Werkstoffkunde, Technische Mechanik, Festigkeitslehre"
author: "EthosAI Learning Team"
license: "CC BY-SA 4.0"
language: "de"

# Maturity Requirements
required_maturity:
  min_overall: 3
  dimensions:
    ethics: 5        # Muss verstehen: Werkstoffversagen → Gefahr
    world_knowledge: 2

# Dependencies
dependencies:
  - name: math-fundamentals
    version: ">=1.0.0"
  - name: physics-basics
    version: ">=1.0.0"

# Provided Capabilities
capabilities:
  - beam_bending_analysis
  - material_selection
  - stress_calculation
  - safety_factor_check

# Training Configuration
training:
  scenarios_count: 50
  estimated_training_time: "2h"
  assessment_threshold: 0.8   # 80% bestehen

# Classification
classification: PUBLIC
tags: [maschinenbau, mechanik, werkstoffe, festigkeit]
```

---

## 3 · Package Registry

### 3.1 Architektur

```
┌─────────────────────────────────────────────┐
│           Package Registry Server            │
│         (EU-Server, DSGVO-konform)           │
├─────────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐    │
│  │         Catalog API                  │    │
│  │  GET  /api/packages                  │    │
│  │  GET  /api/packages/{name}           │    │
│  │  GET  /api/packages/{name}/versions  │    │
│  │  GET  /api/packages/{name}/download  │    │
│  │  POST /api/packages/search           │    │
│  │  POST /api/packages/publish          │    │
│  └─────────────────────────────────────┘    │
│  ┌─────────────────────────────────────┐    │
│  │         Storage (S3-compatible)      │    │
│  │  Hetzner Object Storage / MinIO      │    │
│  │  Verschlüsselt (AES-256)            │    │
│  └─────────────────────────────────────┘    │
│  ┌─────────────────────────────────────┐    │
│  │         Signature Verification       │    │
│  │  Ed25519 für alle Pakete            │    │
│  │  Trusted Publisher Registry          │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### 3.2 Hosting — EU-Server-Optionen

| Anbieter | Standort | DSGVO | Preis (geschätzt) |
|---|---|---|---|
| **Hetzner Cloud** | Falkenstein/Nürnberg, DE | ✅ | ab 5€/Monat (CX22) |
| **IONOS** | Frankfurt, DE | ✅ | ab 4€/Monat |
| **OVH** | Straßburg, FR / Frankfurt, DE | ✅ | ab 3,50€/Monat |
| **netcup** | Nürnberg, DE | ✅ | ab 3€/Monat |

**Empfehlung:** Hetzner Cloud — beste Preis-Leistung, deutsches Unternehmen, DSGVO-Vorzeigeanbieter, S3-kompatibles Object Storage.

---

## 4 · Installation & Lifecycle

### 4.1 Package Manager CLI

```bash
# Verfügbare Pakete suchen
ethosai-pkg search "maschinenbau"

# Paket installieren
ethosai-pkg install mechanical-engineering-basics

# Paket aktualisieren
ethosai-pkg upgrade mechanical-engineering-basics

# Installierte Pakete auflisten
ethosai-pkg list

# Paket entfernen
ethosai-pkg uninstall mechanical-engineering-basics

# Prüfung nach Installation
ethosai-pkg assess mechanical-engineering-basics
```

### 4.2 Installations-Workflow

```
ethosai-pkg install mechanical-engineering-basics
    │
    ▼
┌─────────────────────────┐
│ 1. Registry-Abfrage      │  → Metadaten + Dependencies laden
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 2. Dependency-Resolution │  → Fehlende Pakete identifizieren
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 3. Download + Verify     │  → Paket laden, Ed25519-Signatur prüfen
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 4. Maturity-Check        │  → Reifegrad-Voraussetzung prüfen
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 5. Knowledge Integration │  → Wissen in RAG/Embedding laden
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 6. Skill Registration    │  → Capabilities registrieren
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 7. Training Execution    │  → Szenarien durchlaufen
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 8. Assessment            │  → Prüfung ablegen (≥ 80% pass)
└────────┬────────────────┘
         ▼
┌─────────────────────────┐
│ 9. Certification         │  → Package als "gelernt" markieren
│                          │  → Maturity-Update im Model
└─────────────────────────┘
```

---

## 5 · Initiale Package-Bibliothek (v1.5.0)

### 5.1 Kern-Pakete

| Package | Beschreibung | Dependencies |
|---|---|---|
| `math-fundamentals` | Lineare Algebra, Analysis, Statistik | — |
| `physics-basics` | Mechanik, Thermodynamik, Strömungslehre | `math-fundamentals` |
| `mechanical-engineering-basics` | Werkstoffkunde, TM, Festigkeit | `math-fundamentals`, `physics-basics` |
| `cad-fundamentals` | 3D-Modellierung, STEP/IGES, Toleranzen | `mechanical-engineering-basics` |
| `fem-basics` | Finite-Elemente-Methode, Netzgenerierung | `mechanical-engineering-basics` |
| `safety-engineering` | FMEA, Fehlerbaumanalyse, Normen | `mechanical-engineering-basics` |

### 5.2 Erweiterungs-Pakete (Zukunft)

| Package | Beschreibung | Karrierestufe |
|---|---|---|
| `automation-engineering` | SPS, Robotik, Sensorik | Engineer |
| `project-management` | Gantt, Agile, Kalkulation | Engineer |
| `coding-python` | Python-Programmierung | Senior |
| `system-architecture` | Systemdesign, Modularisierung | Senior |
| `leadership-skills` | Teamführung, Kommunikation | CTO |

---

## 6 · Sicherheit

### 6.1 Signatur-Verifizierung

```python
import nacl.signing

def verify_package(package_path: str, signature_path: str, 
                   trusted_key: nacl.signing.VerifyKey) -> bool:
    """Verifiziert Ed25519-Signatur eines Learning Package."""
    with open(package_path, 'rb') as f:
        package_bytes = f.read()
    with open(signature_path, 'rb') as f:
        signature = f.read()
    
    try:
        trusted_key.verify(package_bytes, signature)
        return True
    except nacl.exceptions.BadSignatureError:
        return False
```

### 6.2 Sandbox-Ausführung

Skills (Python-Code) aus Learning Packages werden in der bestehenden Sandbox (v1.2.0, 100%) ausgeführt:
- Kein Dateisystemzugriff außerhalb des Projekt-Namespace
- Kein Netzwerkzugriff ohne explizite Freigabe
- CPU/Memory-Limits
- Timeout-Schutz

---

## 7 · Monitoring & Lernfortschritt

### 7.1 Metriken

| Metrik | Messung |
|---|---|
| **Packages Installed** | Anzahl installierter Pakete |
| **Assessment Score** | Durchschnittliche Prüfungsnote |
| **Capability Count** | Aktive Fähigkeiten aus Packages |
| **Knowledge Size** | Gesamtgröße der eingebetteten Wissensbasis |
| **Training Hours** | Kumulierte Trainingszeit |
| **Career Progress** | % zum nächsten Karrierelevel |

### 7.2 Dashboard-Integration

Die Metriken fließen in das bestehende KPI Health Dashboard (v1.2.0) ein:

```
GET /api/learning/status
GET /api/learning/packages
GET /api/learning/progress
GET /api/learning/career-level
```

---

## 8 · Implementierungsplan

| Phase | KW | Deliverable |
|---|---|---|
| **L1** | KW 13 | `ethos_ai/learning/package_manager.py` — Package Manager |
| **L2** | KW 13 | `ethos_ai/learning/package_registry.py` — Registry Client |
| **L3** | KW 14 | `ethos_ai/learning/package_installer.py` — Installer + Verifier |
| **L4** | KW 14 | `ethos_ai/learning/assessment_engine.py` — Prüfungssystem |
| **L5** | KW 15 | 3 initiale Packages erstellen (math, physics, mech-eng) |
| **L6** | KW 15 | API-Endpoints + Tests (≥ 30 Tests) |
| **L7** | KW 16 | Registry-Server Setup (Hetzner, FastAPI) |

---

## 9 · Beziehung zu bestehenden Modulen

| Bestehendes Modul | Erweiterung |
|---|---|
| `api/capabilities.py` | Skills aus Packages als dynamische Capabilities |
| `tool/plugin_loader.py` | Packages als spezielle Plugin-Klasse |
| `clim/train_list_generator.py` | Training-Szenarien aus Packages laden |
| `clim/experience_store.py` | Assessment-Ergebnisse als Erfahrungen speichern |
| `api/vector_store.py` | Knowledge-Texte als Embeddings laden |

---

*Dieses Konzept ist Teil des Release v1.5.0 "Going to Be An Engineer".*
