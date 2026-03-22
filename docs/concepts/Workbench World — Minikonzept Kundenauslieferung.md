---
title: "Workbench World — Minikonzept Kundenauslieferung"
brief: "Konzept für die Auslieferung der EthosAI Workbench World als standalone 3D-Review-Plattform für Kundenprojekte. Browser-basiert, kein Blender, kein CAD-Programm nötig."
status: draft
version: "0.1.0"
author: Robert Alexander Massinger
date: 2026-03-09
tags: [workbench, delivery, customer, concept, 3d-review, standalone]
code-references:
  - "ethos_ai/api/workbench_router.py"
  - "ethos_ai/api/static/js/components/workbenchworld.js"
  - "doc/api/Workbench-API.md"
---
# Workbench World — Minikonzept Kundenauslieferung

## 1 · Ausgangslage

Die **Workbench World** ist ein browser-basierter 3D-Projektviewer innerhalb von EthosAI CLIM. Aktueller Stand (v1.63.0):

| Feature | Status |
|---------|--------|
| STL-Viewer (multi-part, binär) | ✅ produktiv |
| Projektbaum (Auto-Discovery) | ✅ produktiv |
| Avatar-System (Anthropometrie, 22 DOF) | ✅ produktiv |
| Part-Animation (Keyframe, Timeline-UI) | ✅ produktiv |
| Ergonomie-Overlay (Reach, FOV, Heatmap) | ✅ produktiv |
| Ruler-Tool, Clipping-Planes | ✅ produktiv |
| FPV-Kamera, Egress-Route | ✅ produktiv |
| REST-API (10 Endpoints, dokumentiert) | ✅ produktiv |

**Ziel:** Workbench World als eigenständiges Lieferpaket für Kunden bereitstellen — ein 3D-Modell-Review-System, das im Browser läuft und keine CAD-Lizenz, kein Blender, keine Spezialsoftware benötigt.

---

## 2 · Liefermodell

### 2.1 Drei Stufen

```
┌─────────────────────────────────────────────────────────┐
│  Stufe 1: Viewer-Paket (read-only)                      │
│  ────────────────────────────────────────────            │
│  • Statisches HTML/JS/CSS Bundle                        │
│  • STL-Dateien als Payload                              │
│  • Kein Server nötig (file://) oder Mini-Server         │
│  • Kunden öffnen index.html → 3D-Review sofort          │
│  → Lieferbar: SOFORT (Sprint 1)                         │
├─────────────────────────────────────────────────────────┤
│  Stufe 2: Server-Paket (interaktiv)                     │
│  ────────────────────────────────────────────            │
│  • EthosAI Workbench als Docker-Container               │
│  • REST-API aktiv (Avatare, Animationen)                │
│  • Kein LLM, kein CLIM — nur Workbench-Router           │
│  • JWT-Auth für Zugangskontrolle                        │
│  → Lieferbar: Sprint 2 (+4 BL-Items)                    │
├─────────────────────────────────────────────────────────┤
│  Stufe 3: Vollintegration (AI-Review)                   │
│  ────────────────────────────────────────────            │
│  • Workbench + EthosAI CLIM                             │
│  • AI-gestützte Ergonomie-Analyse                       │
│  • Automatische Findings-Generierung                    │
│  • Profession-Package: „Review-Ingenieur"               │
│  → Lieferbar: Sprint 3+ (Roadmap)                       │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Stufe 1 – Viewer-Paket (MVP)

**Was wird geliefert:**

```
workbench-review/
├── index.html              ← Standalone SPA
├── css/
│   └── workbench.css
├── js/
│   ├── three.min.js        ← Three.js r152
│   ├── STLLoader.js
│   ├── OrbitControls.js
│   └── workbenchworld.js   ← Viewer-Kern (gekürzt)
├── models/                 ← Kundenprojekt-STLs
│   ├── cockpit.stl
│   ├── hull_main.stl
│   ├── solar_left.stl
│   └── ...
└── manifest.json           ← Part-Liste, Farben, Animationen
```

**Technische Umsetzung:**

1. **Build-Skript** — extrahiert relevante JS/CSS aus EthosAI, bündelt mit STLs
2. **manifest.json** — deklariert Parts, Farben, Animationspresets, Kameraposition
3. **Offline-fähig** — kein Netzwerk nötig, alles lokal
4. **Kein Backend** — STLs werden direkt per `file://` oder eingebetteten Data-URLs geladen

**manifest.json Beispiel:**

```json
{
  "project": "Front Module evol-1-2-0",
  "version": "1.2.0",
  "created": "2026-03-09",
  "parts": [
    { "file": "models/cockpit.stl", "name": "cockpit", "color": "#c0c0c0" },
    { "file": "models/hull_main.stl", "name": "hull_main", "color": "#e8e8e8" },
    { "file": "models/solar_left.stl", "name": "solar_left", "color": "#1a237e" }
  ],
  "camera": { "x": 8000, "y": 4000, "z": 6000, "target": [0, 0, 0] },
  "animations": ["FM Landing", "Deploy Sequence"]
}
```

### 2.3 Stufe 2 – Server-Paket

**Zusätzlich zu Stufe 1:**

- **Docker Image** (`ethosai/workbench:latest`) — Python 3.11 + FastAPI, ~150 MB
- **REST-API** — alle 10 Workbench-Endpoints (siehe Workbench-API.md)
- **JWT-Auth** — Kunden erhalten Login-Token
- **Multi-Projekt** — mehrere Projekte gleichzeitig nach `customers/` mounten
- **Avatar-Kollaboration** — mehrere Benutzer können gleichzeitig Avatare spawnen

**Docker Compose:**

```yaml
services:
  workbench:
    image: ethosai/workbench:latest
    ports:
      - "8000:8000"
    volumes:
      - ./customer-data:/app/customers:ro
    environment:
      - ETHOSAI_MODE=workbench-only
      - ETHOSAI_AUTH=jwt
```

---

## 3 · Kundendatenschutz

| Maßnahme | Umsetzung |
|----------|-----------|
| Keine Kundennamen im Quellcode | ⛔ how-to-release.md Regel |
| Keine Kundendaten im Git-Repo | `.gitignore: customers/` |
| Viewer-Paket enthält nur STLs | Kein Quellcode (.scad) ausgeliefert |
| Server-Paket: read-only API | Kein File-Upload, kein Write-Zugriff |
| Path-Traversal-Schutz | `_safe_resolve()` in jedem Endpoint |
| JWT-Auth im Server-Modus | Kein anonymer Zugriff |

---

## 4 · API-Dokumentation als Lieferbestandteil

Jede Kundenauslieferung (Stufe 2+) enthält:

1. **Workbench-API.md** — vollständige REST-API-Referenz (10 Endpoints, Schemas, Beispiele)
2. **README.md** — Quickstart, Docker-Setup, Screenshots
3. **CHANGELOG.md** — Versions-Historie des gelieferten Pakets

Die API-Dokumentation ist bereits erstellt: `doc/api/Workbench-API.md`

---

## 5 · Aufwandsschätzung

| Arbeitspaket | Aufwand | Abhängigkeiten |
|-------------|---------|----------------|
| Stufe 1: Build-Skript (Extract + Bundle) | 1 Sprint | — |
| Stufe 1: manifest.json Schema + Generator | 0.5 Sprint | Build-Skript |
| Stufe 1: Offline-Index.html (standalone) | 0.5 Sprint | Build-Skript |
| Stufe 2: Dockerfile + Compose | 1 Sprint | Stufe 1 |
| Stufe 2: Workbench-Only-Modus (FastAPI) | 0.5 Sprint | — |
| Stufe 2: JWT-Auth für Kunden-Login | 0.5 Sprint | Auth vorhanden |
| Stufe 2: API-Dokumentation | ✅ erledigt | — |
| Stufe 3: AI-Review-Integration | 3+ Sprints | CLIM, Profession-Package |

**Stufe 1 MVP: ~2 Sprints**
**Stufe 2 Server: +2 Sprints**

---

## 6 · Nächste Schritte

1. ✅ **API-Dokumentation erstellen** → `doc/api/Workbench-API.md`
2. ✅ **Minikonzept erstellen** → dieses Dokument
3. **Backlog-Einträge anlegen** → BL-283 ff. im Projekt-Backlog
4. **Sprint planen** → Stufe 1 MVP als nächsten Workbench-Sprint
5. **Build-Skript prototypen** → `scripts/build_workbench_package.py`
6. **Manifest-Schema definieren** → `workbench_manifest.schema.json`
