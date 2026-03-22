# Living Toolbox System — Konzept

**Version:** 2.0
**Datum:** 2026-02-21
**Status:** Aktiv (v1.21.0)
**Autor:** EthosAI PO/Architect
**Sprache:** DE / EN (Language Switch)

---

## 1. Vision

EthosAI soll viele Berufe abdecken — vom Maschinenbauingenieur über den Statiker
bis zum Softwareentwickler. Jeder Beruf arbeitet mit eigenen **Werkzeugen**.

Das **Living Toolbox System** macht daraus ein **lebendes, modulares, selbst-
erweiterbares Werkzeugkastensystem**:

- **Sparten-unterteilt:** Jede Toolbox gehört zu einer Domäne (Maschinenbau,
  Software, Elektrotechnik, Bauingenieurwesen, …)
- **Selbst-erweiterbar:** EthosAI kann (bei vorhandener Softwareentwicklungs-
  Capability) eigene Tools schreiben, testen und in eine Toolbox einstellen
- **Standardisierte Einbindung:** Manifest-basierte Discovery mit `toolbox.yaml`
- **Hot-Reload:** Toolboxen können zur Laufzeit (nach-)geladen werden
- **UI-fähig:** Web-Interface zeigt alle Toolboxen, deren Tools, Status
- **Dokumentiert:** Jede Toolbox hat ein eigenes `docs/`-Verzeichnis und `README.md`
- **Self-Documenting:** Bilingual DE/EN mit Language Switch

---

## 2. Architektur

### 2.1 Verzeichnisstruktur (kanonisch)

```
toolbox-packages/                          ← Root-Package
├── __init__.py
│
├── engineering/                           ← Maschinenbau-Toolbox (v1.0.0)
│   ├── toolbox.yaml                       ← Manifest
│   ├── README.md                          ← Capability TOC + Kurzbeschreibung
│   ├── __init__.py                        ← EngineeringDomain enum
│   ├── gear_calc.py                       ← Tool: Zahnradberechnung DIN 3990
│   ├── fmea.py                            ← Singleton: FMEA-Analyse
│   ├── standards_db.py                    ← Singleton: Normen-Datenbank  
│   ├── knowledge_graph.py                 ← Singleton: Wissens-Graph
│   ├── gear_standards.py                  ← Startup-Hook: Zahnrad-Normen
│   ├── gear_fmea.py                       ← Startup-Hook: Zahnrad-FMEA
│   ├── openscad_gen.py                    ← Tool: OpenSCAD-Generator
│   ├── bom_generator.py                   ← Tool: Stücklisten-Generator
│   ├── report_generator.py                ← Tool: Technischer Bericht
│   └── docs/                              ← Toolbox-Dokumentation
│       ├── concept_de.md                  ← Konzept (Deutsch)
│       ├── concept_en.md                  ← Concept (English)
│       └── guide_de.md                    ← Anleitung (Deutsch)
│
├── software_dev/                          ← Software-Entwicklung Toolbox (v1.0.0)
│   ├── toolbox.yaml
│   ├── README.md
│   ├── __init__.py
│   ├── code_analyzer.py                   ← Tool: Code-Analyse
│   ├── test_generator.py                  ← Tool: Test-Generierung
│   ├── refactoring.py                     ← Tool: Refactoring-Assistent
│   ├── dependency_checker.py              ← Tool: Abhängigkeiten prüfen
│   ├── doc_generator.py                   ← Tool: Dokumentations-Generator
│   └── docs/
│       ├── concept_de.md
│       ├── concept_en.md
│       └── guide_de.md
│
├── electrical/                            ← Elektrotechnik (geplant v1.23.0)
│   └── ...
│
└── civil-engineering/                     ← Bauingenieurwesen (geplant v1.24.0)
    └── ...
```

### 2.2 Kompatibilitätsschicht

```
engineering/ package                      ← Compatibility Proxy (seit v1.21.0)
├── __init__.py                            ← re-exports EngineeringDomain
├── fmea.py                                ← re-exports → toolboxes.engineering.fmea
├── gear_calc.py                           ← re-exports → toolboxes.engineering.gear_calc
├── ...                                    ← Alle Module sind dünne Proxies
└── (DeprecationWarning bei Import)
```

**Alle bestehenden Imports (`from ethos_ai.engineering.X import Y`) funktionieren
weiterhin**, leiten aber intern an `toolboxes.engineering.X` weiter.
Eine `DeprecationWarning` weist auf den neuen Import-Pfad hin.

---

## 3. Manifest: `toolbox.yaml`

Jede Toolbox wird durch ein YAML-Manifest vollständig beschrieben:

```yaml
name: engineering                           # Eindeutiger Name
version: "1.0.0"                           # SemVer
display_name: "Maschinenbau-Werkzeugkasten"
description: "... nach DIN/ISO"
domain: MECHANICAL                          # EngineeringDomain-Wert
icon: "⚙️"
author: EthosAI
created: "2026-02-20"

requires: []                                # Andere Toolbox-Abhängigkeiten
pip_requires: []                            # Externe pip-Pakete

singletons:                                 # Langlebige Service-Objekte
  knowledge_graph:
    class: knowledge_graph.EngineeringKnowledgeGraph
    kwargs: { state_dir: state }

startup:                                    # Init-Funktionen beim Laden
  - function: gear_standards.ensure_gear_standards
    singleton_args: [standards_db]

tools:                                      # Aufrufbare Werkzeuge
  - name: gear_calculate
    function: gear_calc.design_gear_pair
    display_name: "Zahnradberechnung DIN 3990"
    description: "Vollständige Zahnradberechnung"
    keywords: [zahnrad, gear, DIN 3990]

capabilities:                               # Registrierte Fähigkeiten
  - id: gear-design-full
    name: "Zahnradgetriebe-Auslegung"
    description: "Berechnung → CAD → BOM → Report"
```

---

## 4. Lifecycle

```
       ┌──────────┐     ┌───────────┐     ┌─────────┐
       │DISCOVERED│────>│VALIDATING │────>│ LOADING │
       └──────────┘     └───────────┘     └────┬────┘
                              │                │
                              ▼                ▼
                        ┌──────────┐      ┌──────────┐
                        │ REJECTED │      │  ACTIVE  │<─────┐
                        └──────────┘      └────┬─────┘      │
                                               │            │
                                               ▼            │
                                          ┌─────────┐ reload│
                                          │UNLOADED │───────┘
                                          └─────────┘
                                              
                         ┌─────────┐     ┌─────────┐
                         │  ERROR  │     │ SANDBOX │
                         └─────────┘     └─────────┘
```

**States:**
| State | Beschreibung |
|-------|-------------|
| DISCOVERED | YAML-Manifest gefunden, nicht validiert |
| VALIDATING | Strukturprüfung läuft |
| REJECTED | Validierung fehlgeschlagen |
| LOADING | Singletons, Hooks, Tools werden geladen |
| ACTIVE | Voll funktionsfähig |
| UNLOADED | Entladen, Ressourcen freigegeben |
| ERROR | Laden fehlgeschlagen |
| SANDBOX | KI-generiert, noch nicht freigegeben |

---

## 5. Kernkomponenten

| Komponente | Datei | Verantwortung |
|-----------|-------|---------------|
| **Models** | `toolbox/models` module | Pydantic-Datenmodelle (~170 Zeilen) |
| **Loader** | `toolbox/loader` module | YAML-Discovery + Validierung (~230 Zeilen) |
| **Registry** | `toolbox/registry` module | Lifecycle-Manager (~530 Zeilen) |

### 5.1 ToolboxRegistry API

```python
class ToolboxRegistry:
    async def discover()                    # Scan toolbox-packages/ für toolbox.yaml
    async def validate(name)                # Strukturprüfung
    async def load(name)                    # Singletons + Hooks + Tools laden
    async def unload(name)                  # Ressourcen freigeben
    async def reload(name)                  # Hot-Reload (unload → discover → load)
    
    def inject_singletons(name, dict)       # Vorgefüllte Singletons teilen
    def list_all() → list[LoadedToolbox]    # Alle Toolboxen
    def list_active() → list[LoadedToolbox] # Nur aktive
    def get(name) → LoadedToolbox           # Einzelne Toolbox
    def get_singleton(tb, name)             # Singleton-Zugriff
    def get_tool_callable(tb, tool)         # Tool-Callable
    def list_all_tools() → list[ToolInfo]   # Alle Tools über alle Toolboxen
    def find_tools(query) → list[ToolInfo]  # Keyword-Suche
```

---

## 6. Import-Kompatibilität

### Migrationsstrategie (abgeschlossen v1.21.0)

| Phase | Version | Aktion |
|-------|---------|--------|
| 1 | v1.20.0 | Dateien nach `engineering-toolbox/` kopiert (relative Imports) |
| 2 | v1.20.0 | `toolbox.yaml` + Registry + Loader implementiert |
| 3 | v1.21.0 | `engineering/` package zu Compatibility Proxy konvertiert |
| 4 | v1.21.0 | Alle Tests bestehen mit Proxy-Imports |
| 5 | v1.22.0+ | DeprecationWarning → Imports schrittweise auf `toolboxes.*` migrieren |
| 6 | v2.0.0 | Proxy-Layer entfernen |

### Proxy-Beispiel

```python
# engineering/fmea module (v1.21.0 — Proxy)
"""FMEA Module — Compatibility proxy.
Canonical source: engineering-toolbox/fmea.py
"""
from toolboxes.engineering.fmea import (
    FMEAAction, FMEAAnalysis, FMEAEntry, RiskLevel,
)
__all__ = ["RiskLevel", "FMEAAction", "FMEAEntry", "FMEAAnalysis"]
```

---

## 7. Selbst-Erweiterbarkeit (CodeGen → Toolbox)

EthosAI kann eigene Tools schreiben, wenn sie über die Software-Entwicklungs-
Toolbox verfügt:

```
┌──────────────┐    ┌─────────────┐    ┌──────────────┐    ┌──────────┐
│ Advisor gibt │    │ CodeGen     │    │ ToolboxMgr   │    │ Sandbox  │
│ Auftrag:     │───>│ erzeugt     │───>│ hot-reload   │───>│ Test +   │
│ "Schreib ein │    │ neues Tool  │    │ + validate   │    │ Freigabe │
│  Statik-Tool"│    │ → .py file  │    │ + register   │    │          │
└──────────────┘    └─────────────┘    └──────────────┘    └──────────┘
```

**Sicherheits-Gate:**
1. Neues Tool wird in `SANDBOX`-Modus geladen
2. Automatische Tests werden ausgeführt
3. Advisor-Freigabe erforderlich für `ACTIVE`-Modus
4. Fehlerhafte Tools können sofort entladen werden

---

## 8. Web-UI & SSH-Interface

### 8.1 Web-UI: Toolbox-View (#/toolboxes)

- **Karten-Ansicht**: Jede Toolbox als Karte mit Icon, Name, Version, Status
- **Detail-Dialog**: Metadaten, Tools-Tabelle, Singletons, Capabilities
- **Live-Suche**: Echtzeit-Suche über alle Toolboxen
- **Aktionsbuttons**: Laden, Entladen, Reload
- **Responsiv**: Desktop + Tablet Layout

### 8.2 SSH-Interface (geplant v1.25.0)

```bash
ethos toolbox list                          # Übersicht
ethos toolbox info engineering              # Details
ethos toolbox reload engineering            # Hot-Reload
ethos toolbox run engineering.gear_calculate --force_kn=100
ethos toolbox add-tool engineering new_tool.py
```

---

## 9. REST-API

| Methode | Pfad | Beschreibung |
|---------|------|-------------|
| GET | `/api/toolbox-list` | Alle Toolboxen mit Status |
| GET | `/api/toolbox-detail/{name}` | Toolbox-Detail |
| GET | `/api/toolbox-packages/{name}/tools` | Werkzeug-Liste |
| POST | `/api/toolbox-packages/{name}/load` | Toolbox laden |
| POST | `/api/toolbox-packages/{name}/unload` | Toolbox entladen |
| POST | `/api/toolbox-packages/{name}/reload` | Hot-Reload |
| GET | `/api/toolbox-list/tools/search?q=…` | Cross-Toolbox-Suche |

---

## 10. Toolbox-Dokumentationsstandard

Jede Toolbox MUSS folgende Dokumentation enthalten:

```
toolbox-packages/{name}/
├── README.md                    ← Capability TOC + Kurzbeschreibung (DE | EN)
└── docs/
    ├── concept_de.md            ← Toolbox-Konzept (Deutsch)
    ├── concept_en.md            ← Toolbox Concept (English)
    ├── guide_de.md              ← Anleitung (Deutsch)
    └── guide_en.md              ← Guide (English)
```

### README.md Template

```markdown
# {Toolbox Display Name}
> {Kurzbeschreibung}

| | DE | EN |
|---|---|---|
| Konzept | [concept_de.md](docs/concept_de.md) | [concept_en.md](docs/concept_en.md) |
| Anleitung | [guide_de.md](docs/guide_de.md) | [guide_en.md](docs/guide_en.md) |

## Capabilities
| ID | Name | Beschreibung / Description |
|---|---|---|
| ... | ... | ... |

## Tools
| Name | Beschreibung / Description |
|---|---|
| ... | ... |

## Version
- **Version:** {version}
- **Domain:** {domain}
- **Author:** {author}
```

---

## 11. Roadmap

### v1.21.0 — Living Toolbox System (aktuell)
- ✅ Legacy-Migration: `engineering/` package → Compatibility Proxy
- ✅ `engineering-toolbox/` ist kanonische Quelle
- ✅ Software-Entwicklung Toolbox (`software-dev-toolbox/`)
- ✅ Toolbox-Dokumentationsstandard (README.md + docs/)
- ✅ Konzeptdokument v2.0

### v1.22.0 — Self-Extensibility
- 🔲 CodeGen-Integration: EthosAI erstellt eigene Tools via CodeGen
- 🔲 Sandbox-Modus für KI-generierte Tools
- 🔲 Automatische Test-Generierung für neue Tools
- 🔲 `toolbox.yaml`-Auto-Update nach Tool-Erstellung

### v1.23.0 — Elektrotechnik-Toolbox
- 🔲 `electrical-toolbox/` mit circuit_calc, emc_check, power_analysis
- 🔲 Normen: DIN VDE 0100, IEC 60364, DIN EN 61000
- 🔲 Schaltungssimulation (SPICE-Anbindung)

### v1.24.0 — Bauingenieurwesen-Toolbox
- 🔲 `civil-engineering-toolbox/` mit structural_calc, load_analysis
- 🔲 Normen: DIN EN 1990-1992, Eurocode
- 🔲 Statik-Berechnung, Tragwerksplanung

### v1.25.0 — SSH-Terminal-Interface
- 🔲 `ethos toolbox` CLI-Befehle
- 🔲 Interactive Shell für Tool-Ausführung
- 🔲 Batch-Verarbeitung

### v1.26.0 — Toolbox-Marketplace
- 🔲 Import/Export von Toolboxen als ZIP/TAR
- 🔲 Versionierung und Kompatibilitätsprüfung
- 🔲 Community-Marketplace (Registry-Server)
- 🔲 Signierung und Vertrauensstufen

---

## 12. Akzeptanzkriterien (v1.21.0)

1. ✅ `engineering/` package ist Compatibility Proxy
2. ✅ `engineering-toolbox/` ist kanonische Quelle
3. ✅ Alle bestehenden Tests bestehen (mit DeprecationWarning)
4. ✅ Software-Entwicklung Toolbox existiert und ist ladbar
5. ✅ Jede Toolbox hat README.md + docs/
6. ✅ Konzeptdokument v2.0 mit Roadmap v1.22–1.26
