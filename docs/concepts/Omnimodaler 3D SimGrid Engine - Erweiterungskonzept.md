---
title: "Omnimodaler 3D SimGrid Engine — Erweiterungskonzept"
brief: "World-native Architektur zur Erweiterung des Digital Twin SimGrid um physikalisch korrekte 3D-Simulation, CAD-Integration und Engineering-Visualisierung — EU-konform, inkrementell, auf dem Worlds-Framework aufbauend."
status: living
version: "2.0.0"
author: Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)
date: 2026-03-21
history:
  - date: 2026-02-15
    version: "1.0.0"
    change: "Erstversion des 3D SimGrid Extension Konzepts (monolithischer Server-Ansatz)."
  - date: 2026-03-21
    version: "2.0.0"
    change: "Komplette Neuausrichtung: World-native Architektur statt monolithischem Backend. Gap-Analyse v1.5.0→v1.90.0; EU-Compliance-Bewertung aller Komponenten; Hybrid Client/Server-Strategie (Rapier.js + optionale Backend-Services); Integration mit ECS, WorldBase, SimGrid-Text-Pipeline. Altes Konzept als Appendix A erhalten."
  - date: 2026-03-21
    version: "2.0.1"
    change: "Phase 2 CAD-Konverter umgesetzt: CadQueryImporter (STEP/IGES/BREP→GLB, SHA-256 LRU-Cache), CAD REST-API (3 Endpoints), 6 neue DIN/EN-Werkstoffe (22 gesamt), MCP-Tool simgrid_import_cad, Frontend CAD-Import-Dialog. Fulfillment 30%→75%."
tags: [simgrid, 3d-simulation, physics, rapier, cad, worlds, engineering, eu-compliance]
code-release-versions:
  - "1.5.0"
  - "1.90.0"
  - "1.92.0"
implemented-features:
  - "SimGrid3D ABC-Interfaces (engine, physics_sim, cad_importer, fem_solver, materials)"
  - "Material-Datenbank (6 Werkstoffe, SI-Einheiten)"
  - "ECS mit rigidbody/collider/transform Components"
  - "Physics-Presets (5 Profile, 8 Material-Presets, Per-Zone-Overrides)"
  - "Three.js 3D-Rendering in WorldBase (STL, GLTF)"
  - "World Packs: simgrid-strukturmechanik, material-prueflab"
  - "CadQueryImporter (STEP/IGES/BREP → GLB, SHA-256 Cache, Lazy-Dependencies)"
  - "CAD REST-API (3 Endpoints: convert, result, formats)"
  - "6 DIN/EN-Werkstoffe (AlMg3, GJS-400-15, X5CrNi18-10, PA6-GF30, S355J2, Ti-6Al-4V)"
  - "MCP-Tool simgrid_import_cad (8 Tools gesamt)"
  - "Frontend CAD-Import-Dialog"
fulfillment: "75%"
fulfillment-note: "Phase 1 (ABC-Interfaces, Material-DB, ECS, Three.js-Rendering, Rapier.js-Physik) + Phase 2 (CadQueryImporter STEP/IGES/BREP→GLB, CAD REST-API, 22 Werkstoffe, MCP-Tools, Frontend-Import) vollständig. FEM-Integration (Phase 3) und MuJoCo Concrete Physics (Phase 4) fehlen."
---
# Omnimodaler 3D SimGrid Engine — Erweiterungskonzept

**Version:** 2.0.0
**Datum:** 2026-03-21
**Autoren:** Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)
**Status:** Living — World-native Architektur

---

## 1 · Motivation

Der Digital Twin SimGrid (v1.3.0, 100% umgesetzt) arbeitet rein textbasiert: Er simuliert Perspektiven, bewertet Konsequenzen und synthetisiert Entscheidungen — alles in natürlicher Sprache. Für den **Junior Engineer** im Maschinenbau fehlt die physikalische Dimension:

- **Keine Echtzeit-Physik** — Kräfte, Kontakte, Kollisionen werden nicht berechnet
- **Kein CAD-Import** — STEP/IGES → 3D-Darstellung fehlt
- **Keine FEM** — Strukturanalyse (Spannungen, Verformungen) ist nicht möglich
- **Keine Physik-Visualisierung** — Spannungsverteilungen, Kraftpfeile, Verformungsplots fehlen

Die Erweiterung macht den SimGrid **omnimodal**: Text + 3D + Physik + Materialien = eine vollständige Engineering-Simulationsumgebung.

### 1.1 Was sich seit v1.5.0 geändert hat

Das Konzept v1.0.0 (Februar 2026) entwarf einen monolithischen Server-Ansatz mit MuJoCo + PyVista + CadQuery als Backend-Services. Seitdem ist das Projekt auf **v1.90.0** gewachsen, und das **Worlds-Framework** hat vieles davon auf elegantere Weise gelöst:

| Fähigkeit | v1.5.0 (geplant) | v1.90.0 (IST) |
|---|---|---|
| 3D-Rendering | PyVista (Server-Side) | ✅ Three.js in 3 Welten (Client-Side, ~4.600 LOC) |
| Scene Graph | Nicht spezifiziert | ✅ ECS mit 12 Component-Schemas (315 LOC) |
| Materialien | Konzept | ✅ `simgrid3d/materials.py` + `physics_configurator.py` (8 Presets) |
| STL/GLTF Import | Geplant | ✅ WorkbenchWorld (STL-Parser, GLTFLoader, 5 Koordinaten-Frames) |
| Multi-User | Nicht geplant | ✅ WebSocket + WorldCursors + WorldSessionManager |
| Standalone | Nicht geplant | ✅ Docker-ready (Worlds Standalone, Mode A/B) |
| Marketplace | Nicht geplant | ✅ Ed25519-signierte World Packs + MarketplaceRegistry |
| SEC-Tier-System | Nicht geplant | ✅ 3-Tier-Enforcement + Zertifizierung |
| World Packs | Nicht geplant | ✅ 5 Packs (u.a. `simgrid-strukturmechanik`, `material-prueflab`) |
| SimGrid3D ABCs | Nicht vorhanden | ✅ engine.py, physics_sim.py, cad_importer.py, fem_solver.py |
| Concrete Physics | MuJoCo (Server) | 🔴 Kein konkreter Simulator verdrahtet |
| CAD STEP→GLTF | CadQuery (Server) | 🔴 Kein Konverter implementiert |
| FEM-Solver | SfePy (Server) | 🔴 Kein Solver verdrahtet |
| API-Endpoints | 10 geplant | 🔴 0 implementiert (via existing World Router stattdessen) |

**Fazit:** Die Visualisierungs-, Scene-Management- und Infrastruktur-Lücken sind geschlossen. Was fehlt: **Physik-Berechnung, CAD-Konvertierung und FEM-Analyse**.

---

## 2 · Gap-Analyse: Was noch fehlt

### 2.1 Offene Lücken (rot)

| # | Lücke | Beschreibung | Dringlichkeit |
|---|---|---|---|
| G1 | **Client-Physics** | Kein Rigid-Body-Simulator im Browser (Kräfte, Kontakte, Gelenke) | HIGH |
| G2 | **STEP-Import** | Kein STEP/IGES → GLTF Konverter (Maschinenbau-Standardformat) | HIGH |
| G3 | **FEM-Analyse** | Kein Spannungs-/Verformungs-Solver | MEDIUM |
| G4 | **Spannungs-Visualisierung** | Kein Vertex-Coloring für von-Mises / Hauptspannungen | MEDIUM |
| G5 | **FMEA-Modul** | Kein automatisches Failure Mode & Effects Assessment | LOW |
| G6 | **SimGrid-Brücke** | Text-Pipeline (6 Stufen) nicht mit 3D-Ergebnissen verknüpft | MEDIUM |

### 2.2 Vorhandene Bausteine (grün)

| # | Baustein | Status | Wo? |
|---|---|---|---|
| B1 | Three.js 3D-Engine | ✅ Produktion | WorldBase (1.048 LOC) |
| B2 | ECS (Entity-Component-System) | ✅ Produktion | `ecs.py` (315 LOC, 12 Components) |
| B3 | Physics-Presets | ✅ Produktion | `physics_configurator.py` (5 Profile, 8 Materialien) |
| B4 | STL/GLTF Import | ✅ Produktion | WorkbenchWorld (stl-parser.js, GLTFLoader) |
| B5 | Materialien-DB | ✅ Produktion | `simgrid3d/materials.py` (6 Werkstoffe, SI) |
| B6 | SimGrid Text-Pipeline | ✅ 100% | 6 Stufen (Complexity→Experience) |
| B7 | SimGrid3D ABCs | ✅ Interfaces | engine.py, physics_sim.py, cad_importer.py, fem_solver.py |
| B8 | Multi-User | ✅ Produktion | WorldSession + WorldCursors |
| B9 | Measurement-Tool | ✅ Produktion | measurement-tool.js |
| B10 | World Packs | ✅ Produktion | simgrid-strukturmechanik, material-prueflab |
| B11 | MCP-Gateway | ✅ 24 Tools | mcp_worlds.py (17 World-Tools) |

---

## 3 · Neue Architektur: World-native Hybrid-Ansatz

### 3.1 Entwurfsprinzip

Statt des monolithischen Server-Ansatzes aus v1.0.0 nutzt v2.0.0 eine **World-native Hybrid-Architektur**:

- **Client-First**: Echtzeit-Physik läuft im Browser (Rapier.js WASM) — keine Roundtrips
- **Server-on-Demand**: Schwere Operationen (CAD-Konvertierung, FEM) sind Backend-Services
- **World-native**: SimGrid 3D ist eine EthosAI World (`SimGridWorld extends WorldBase`)
- **SimGrid-Brücke**: Text-Pipeline bewertet 3D-Ergebnisse mit AI-Reasoning

### 3.2 Schichtenmodell v2.0.0

```
┌──────────────────────────────────────────────────────────────────┐
│                    SimGrid 3D World (Browser)                    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ SimGridWorldView extends WorldBase                          │ │
│  │  ┌──────────┐  ┌──────────────┐  ┌───────────────────────┐  │ │
│  │  │Three.js  │  │ Rapier.js    │  │ Stress Colormap       │  │ │
│  │  │Renderer  │  │ WASM Physics │  │ Vertex-Coloring       │  │ │
│  │  └──────────┘  └──────────────┘  └───────────────────────┘  │ │
│  │  ┌──────────┐  ┌──────────────┐  ┌───────────────────────┐  │ │
│  │  │STL/GLTF  │  │ Material     │  │ Measurement & Labels  │  │ │
│  │  │Import    │  │ Inspector    │  │ (shared)              │  │ │
│  │  └──────────┘  └──────────────┘  └───────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ World Framework (shared)                                    │ │
│  │  WorldBase · WorldBridge · WorldStorage · WorldEventBus     │ │
│  │  WorldSession · WorldCursors · ECS · Physics-Presets        │ │
│  └─────────────────────────────────────────────────────────────┘ │
├──────────────────────────────────────────────────────────────────┤
│             Backend Services (on-demand, optional)               │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐   │
│  │ CAD Converter│  │ FEM Service  │  │ Physics Batch         │   │
│  │ STEP → GLTF  │  │ SfePy        │  │ MuJoCo (HiFi)         │   │
│  │ (CadQuery)   │  │ (Statik)     │  │ (optional)            │   │
│  └──────────────┘  └──────────────┘  └───────────────────────┘   │
├──────────────────────────────────────────────────────────────────┤
│             SimGrid Text-Pipeline (AI Reasoning)                 │
│  ComplexityRouter → ScenarioGenerator → RoleSimulator            │
│  → ConsequenceSimulator → DecisionSynthesizer                    │
│  → ExperienceHarvester                                           │
│                                                                  │
│  NEU: Physik-Ergebnisse als Fakten in die Pipeline einspeisen    │
│  → RoleSimulator.ENGINEER bewertet Spannungen, Sicherheitsfaktor │
│  → ConsequenceSimulator projiziert Materialalterung              │
│  → DecisionSynthesizer: Ethik (40%) + Technik (20%) + ...        │
└──────────────────────────────────────────────────────────────────┘
```

### 3.3 Datenfluss: Engineering-Aufgabe (v2.0.0)

```
Aufgabe: "Analysiere diese Welle auf Biegung und Torsion"
    │
    ▼
┌───────────────────────┐
│ SimGridWorld (Browser)│  → Bauteil laden (STL/GLTF)
│ 3D-Viewport           │  → Material zuweisen (Material-Inspector)
│ Rapier.js Physics     │  → Lastfall interaktiv anlegen (Kräfte, Lager)
└──────────┬────────────┘
           │ (Echtzeit-Physik im Browser: Kontakte, Kollisionen, Kinematik)
           ▼
┌───────────────────────┐
│ Backend: FEM Service  │  → POST /api/fem/analyze (Mesh + Lasten + Material)
│ SfePy (Server)        │  → Berechnet: von-Mises, Hauptspannungen, Verformung
│                       │  → Gibt JSON-Ergebnis mit Vertex-Daten zurück
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ SimGridWorld (Browser)│  → Vertex-Coloring: Spannungsverteilung
│ Stress Colormap       │  → Clipping-Planes für Schnittansichten
│ Report-Generator      │  → Markdown-Report mit Screenshots
└──────────┬────────────┘
           ▼
┌───────────────────────┐
│ SimGrid Text-Pipeline │  → ENGINEER-Rolle bewertet Ergebnisse
│ AI Reasoning          │  → Sicherheitsfaktor vs. Normanforderung
│                       │  → Alternativvorschläge (Material, Geometrie)
└───────────────────────┘
```

### 3.4 Vergleich: v1.0.0 (alt) vs. v2.0.0 (neu)

| Aspekt | v1.0.0 (Feb 2026) | v2.0.0 (Mrz 2026) |
|--------|-------------------|---------------------|
| Dependencies | ~500 MB sofort (MuJoCo+VTK+OCCT) | ~200 KB (Rapier WASM), Rest on-demand |
| Visualisierung | PyVista (Server-Side Rendering) | Three.js (Client-Side, bereits vorhanden) |
| Physics | MuJoCo (Server-only) | Rapier.js (Client) + MuJoCo (Server, optional) |
| Architektur | Monolithisch (`/api/simgrid3d/*`) | World-nativ (WorldBase, ECS, World Pack) |
| Multi-User | Nicht im Konzept | ✅ Bereits da (WorldSession) |
| Marketplace | Nicht im Konzept | ✅ World Pack als verteilbare Einheit |
| Standalone | Nicht explizit | ✅ Docker-ready via Worlds Standalone |
| EU-Konformität | Nicht adressiert | ✅ Alle Komponenten EU-Herkunft |
| AI-Reasoning | Nicht integriert | SimGrid Text-Pipeline bewertet Ergebnisse |
| Inkrementell | 9 Wochen Wasserfall (P1–P9) | 4 Phasen, jede standalone nutzbar |

---

## 4 · EU-konforme Technologie-Auswahl

### 4.1 Prinzip

Gemäß der EthosAI-Worlds-LLM-Policy (§5.2 der Produkt-Architektur-Analyse): **Nur EU-konforme Komponenten**. Alle Libraries müssen permissive Lizenzen haben und dürfen keine Telemetrie an Nicht-EU-Server senden.

### 4.2 Ausgewählte Technologie-Stacks

#### Client-seitig (Browser, Zero Dependencies)

| Komponente | Library | Herkunft | Lizenz | Größe |
|---|---|---|---|---|
| **3D-Rendering** | Three.js r152 | 🌐 Open Source (MIT) | MIT | ✅ Bereits geladen |
| **Echtzeit-Physik** | Rapier.js (WASM) | 🇫🇷 Dimforge (Paris) | Apache 2.0 | ~200 KB |
| **Mesh-Processing** | three-bvh-csg | 🌐 Open Source | MIT | ~30 KB |

**Rapier.js** ist die primäre Empfehlung für Client-Physics:
- **EU-Herkunft**: Dimforge SAS, Paris, Frankreich
- **Apache 2.0**: Volle kommerzielle Nutzung
- **Features**: Rigid Bodies, Joints (Revolute, Prismatic, Fixed, Ball), Contact Events, CCD, SIMD
- **WASM**: Läuft in jedem Browser ohne Server-Roundtrip
- **Deterministic**: Gleiche Inputs → gleiche Outputs (reproduzierbare Ingenieur-Ergebnisse)
- Die `physics_configurator.py` erwähnt Rapier bereits als Referenzarchitektur (BVH, SIMD)

#### Server-seitig (Backend, on-demand)

| Komponente | Library | Herkunft | Lizenz | Wann nötig? |
|---|---|---|---|---|
| **CAD-Konverter** | CadQuery + OpenCascade | 🇫🇷 Open Cascade SAS + Community | Apache 2.0 / LGPL | STEP/IGES Import |
| **Mesh-Exchange** | Trimesh | 🌐 Open Source | MIT | Format-Konvertierung |
| **FEM-Solver** | SfePy | 🇨🇿 Czech Academy of Sciences | BSD | Spannungsanalyse |
| **HiFi-Physics** | MuJoCo (optional) | 🏴 Google DeepMind | Apache 2.0 | Kontakt-intensive Simulationen |

#### LLM-Reasoning (SimGrid Text-Pipeline)

| Quelle | Herkunft | Lizenz | Tier 1–3 |
|---|---|---|---|
| Mistral 7B / 8×7B | 🇫🇷 Paris | Apache 2.0 | ✅ Alle |
| Aleph Alpha Luminous | 🇩🇪 Heidelberg | Commercial | ✅ Alle |
| Lokales Open-Source (Ollama) | 🇪🇺 | Variiert | ✅ Alle |

### 4.3 EU-Compliance-Matrix

| Kriterium | Status | Begründung |
|---|---|---|
| **Keine US-Cloud-Abhängigkeit** | ✅ | Alles lokal oder EU-hosted |
| **Keine Telemetrie** | ✅ | Rapier, Three.js, SfePy: Zero Telemetry |
| **DSGVO-konform** | ✅ | Alle Daten lokal, SEC-Tier-Enforcement |
| **Permissive Lizenz** | ✅ | Apache 2.0, MIT, BSD — keine GPL im Kern |
| **Auditierbar** | ✅ | Rapier (Rust, ~80K LOC), Tauri (~50K LOC) |
| **Air-Gapped-fähig** | ✅ | Alle WASM-Module lokal bundlebar |

---

## 5 · Open-Source-Module — Bewertung (aktualisiert)

### 5.1 Tier 1 — Sofort einsetzbar (Kern-Stack)

#### Rapier.js (Client-Physics, WASM) ⭐ NEU in v2.0.0

| Eigenschaft | Details |
|---|---|
| **Repository** | [dimforge/rapier](https://github.com/dimforge/rapier) |
| **Lizenz** | Apache 2.0 |
| **Herkunft** | 🇫🇷 Dimforge SAS, Paris |
| **Stars** | 4.500+ |
| **JS API** | `@dimforge/rapier3d-compat` (WASM, ~200 KB) |
| **Stärken** | Rigid-Body-Dynamik, Contact Events, Gelenke (Revolute, Prismatic, Fixed, Ball), CCD, SIMD-Beschleunigung, deterministisch |
| **Browserkompatibilität** | Alle modernen Browser (Chrome, Firefox, Safari, Edge) |

**Warum Rapier statt MuJoCo im Browser:** MuJoCo hat keinen WASM-Build. Rapier bietet 90% der Rigid-Body-Features, läuft direkt im Browser und ist EU-Herkunft. MuJoCo bleibt als optionaler HiFi-Backend-Service verfügbar.

#### CadQuery + OCP (CAD Kernel, Server-side)

| Eigenschaft | Details |
|---|---|
| **Repository** | [CadQuery/cadquery](https://github.com/CadQuery/cadquery) |
| **Lizenz** | Apache 2.0 |
| **Basis** | Open Cascade Technology (🇫🇷 Open Cascade SAS, Guyancourt) |
| **Stärken** | Parametrisches 3D-CAD, STEP/IGES/DXF/STL/AMF-Import/Export, Assembly-Support |
| **Einsatz** | STEP → GLTF Konvertierung im Backend |

#### Trimesh (Mesh Processing, Server-side)

| Eigenschaft | Details |
|---|---|
| **Repository** | [mikedh/trimesh](https://github.com/mikedh/trimesh) |
| **Lizenz** | MIT |
| **Stärken** | STL/OBJ/GLTF/PLY laden, Boolean-Operationen, Masse/Trägheit, Convex Hulls |
| **Einsatz** | Format-Konvertierung CadQuery → GLTF für Three.js |

### 5.2 Tier 2 — On-Demand Backend-Services

#### SfePy (FEM-Solver)

| Eigenschaft | Details |
|---|---|
| **Herkunft** | 🇨🇿 Czech Academy of Sciences, Nové Město |
| **Lizenz** | BSD |
| **Stärken** | PDEs via FEM, Strukturmechanik, NumPy/SciPy-basiert, Windows-nativ |
| **Einsatz** | Statische Spannungsanalyse, Modalanalyse |

#### MuJoCo (HiFi-Physics, optional)

| Eigenschaft | Details |
|---|---|
| **Repository** | [google-deepmind/mujoco](https://github.com/google-deepmind/mujoco) |
| **Lizenz** | Apache 2.0 |
| **Stärken** | Beste Kontakt-Physik, Tendons/Actuators, MJCF-Format |
| **Einsatz** | Batch-Simulation für kontaktintensive Mechanismen (Getriebe, Robotik) |

### 5.3 Tier 3 — Spezialisiert (unverändert aus v1.0.0)

| Tool | Lizenz | Zweck | Einschränkung |
|---|---|---|---|
| **FEniCSx** | LGPL-3.0 | HPC-FEM (akademischer Goldstandard) | Docker/WSL empfohlen |
| **PyBullet** | zlib | Alternative Collision-Detection, URDF | Ergänzt Rapier bei URDF-Modellen |
| **OpenFOAM** | GPL-3 | CFD (Turbulenzen, Strömung) | Nur Linux/WSL |
| **Blender bpy** | GPL 2+ | Fotorealistische Renders | Als externer Service |

---

## 6 · SimGridWorld — Die neue Engineering-World

### 6.1 Klassen-Design

```javascript
// SimGridWorldView (frontend module)
class SimGridWorldView extends WorldBase {

    // === Lifecycle (WorldBase Hooks) ===
    _createScene()          // Three.js Scene + Grid + Axis + Lighting
    _createCamera()         // Engineering-Kamera (Ortho + Perspektive umschaltbar)
    _animate(dt)            // Rapier.js Physik-Step + Three.js Render
    _dispose()              // Rapier World + Three.js Cleanup

    // === Rapier.js Physics ===
    _initPhysics(preset)    // Rapier.World erstellen (Gravity aus physics_configurator)
    _addRigidBody(entity)   // ECS-Entity → Rapier RigidBody + Three.js Mesh
    _applyForce(bodyId, force, point) // Kraft auf Body anwenden
    _createJoint(type, bodyA, bodyB, config) // Gelenk-Verbindung

    // === CAD Integration ===
    _importModel(file)      // STL/GLTF direkt, STEP via Backend-Konverter
    _assignMaterial(bodyId, materialId) // Material aus materials.py zuweisen

    // === FEM Integration ===
    _requestFEMAnalysis(mesh, loads, material) // POST /api/fem/analyze
    _applyStressColormap(vertices, stressData) // von-Mises Vertex-Coloring
    _showDeformation(vertices, displacements, scale) // Überhöhte Verformung

    // === SimGrid Bridge ===
    _sendToSimGridPipeline(physicsResults) // Ergebnisse an Text-Pipeline
    _displayAIAssessment(report)           // AI-Bewertung im PAD anzeigen

    // === Interaction ===
    _onRaycasterHit(event)  // Part-Selection + Force-Application-Mode
    _createForceArrow(point, direction, magnitude) // Kraftpfeil als 3D-Helper
    _createSupportMarker(point, type) // Festlager / Loslager / Einspannung
}
```

### 6.2 ECS-Integration

SimGridWorld nutzt die bestehenden ECS-Components und erweitert sie:

| Component | Bestehend? | Beschreibung |
|---|---|---|
| `transform` | ✅ Ja | Position, Rotation, Scale |
| `mesh` | ✅ Ja | Three.js Mesh-Referenz |
| `collider` | ✅ Ja | Collision-Shape (box, sphere, cylinder, mesh) |
| `rigidbody` | ✅ Ja | Masse, Velocity, is_static |
| `material_eng` | 🆕 Neu | Engineering-Material (ID → materials.py) |
| `loads` | 🆕 Neu | Angewandte Kräfte & Momente [{point, force, type}] |
| `constraints` | 🆕 Neu | Lagerungen [{point, type: fixed/pinned/roller}] |
| `fem_result` | 🆕 Neu | FEM-Ergebnis {von_mises[], displacement[], factor_of_safety} |

### 6.3 World Pack: `simgrid-3d`

```
world-packages/simgrid-3d/
├── assets/
│   ├── rapier3d-compat.wasm       # Rapier WASM Bundle (~200 KB)
│   └── sample-models/             # Beispiel-STL/GLTF für Demo
├── metadata.json                  # sec_tier_min: 1, requires_network: false
├── scene.json                     # Default-Scene: Testfeld mit Messgeräten
├── permissions/
│   ├── admin.json
│   ├── editor.json
│   └── visitor.json
├── scripts/
│   └── simgrid-physics.js         # Rapier.js Integration + Force-UI
├── state/
│   └── materials.json             # Erweiterte Material-DB (10+ Werkstoffe)
└── tool-layouts/
    └── default.json               # PAD-Layout: Material, Forces, FEM Results
```

### 6.4 MCP-Tools (SimGrid-spezifisch)

| Tool | Beschreibung | Beispiel-Aufruf |
|---|---|---|
| `simgrid_create_body` | Rigid Body in der Szene erstellen | `{shape: "cylinder", radius: 0.025, height: 0.5, material: "S235JR"}` |
| `simgrid_apply_force` | Kraft auf Body anwenden | `{body: "shaft_01", force: [0, -5000, 0], point: [0, 0.25, 0]}` |
| `simgrid_add_constraint` | Lagerung definieren | `{body: "shaft_01", type: "pinned", point: [0, 0, 0]}` |
| `simgrid_run_physics` | Physik-Simulation starten | `{duration: 2.0, timestep: 0.001}` |
| `simgrid_request_fem` | FEM-Analyse anfordern | `{bodies: ["shaft_01"], analysis: "static"}` |
| `simgrid_get_results` | Ergebnisse abfragen | `{body: "shaft_01", type: "von_mises"}` |
| `simgrid_assess` | SimGrid-Pipeline für AI-Bewertung | `{context: "Biegung unter 5 kN Querkraft"}` |

---

## 7 · Materialien-Datenbank (erweitert)

### 7.1 Schema (unverändert aus v1.0.0)

```json
{
  "id": "S235JR",
  "name": "Baustahl S235JR",
  "category": "steel",
  "properties": {
    "density_kg_m3": 7850,
    "yield_strength_mpa": 235,
    "tensile_strength_mpa": 360,
    "youngs_modulus_gpa": 210,
    "poissons_ratio": 0.3,
    "fracture_toughness_mpa_sqrt_m": 140,
    "fatigue_limit_mpa": 160,
    "thermal_conductivity_w_mk": 50,
    "thermal_expansion_1_k": 1.2e-5,
    "max_service_temp_c": 300,
    "brittleness_index": 0.15
  },
  "standards": ["EN 10025-2", "DIN 17100"],
  "source": "Stahlschlüssel 2024"
}
```

### 7.2 Datensatz (10 + 6 bereits in materials.py)

**Bereits in `simgrid3d/materials.py`:** Steel S235, Aluminum 6061-T6, Titanium Gr2, Copper, Brass, Stainless 304L

**Erweiterung (DIN/EN-Normen):**

| Werkstoff | Kategorie | Norm |
|---|---|---|
| S355J2 | Baustahl | EN 10025-2 |
| C45 | Vergütungsstahl | EN 10083-2 |
| 42CrMo4 | Vergütungsstahl | EN 10083-3 |
| X5CrNi18-10 (1.4301) | Edelstahl | EN 10088-3 |
| AlMg3 (5754) | Aluminium | EN 573-3 |
| AlMgSi1 (6082) | Aluminium | EN 573-3 |
| PA6 GF30 | Kunststoff | ISO 1874 |
| GJL-250 | Gusseisen (Lamellen) | EN 1561 |
| GJS-400-15 | Gusseisen (Kugel) | EN 1563 |
| Ti-6Al-4V (Gr5) | Titan-Legierung | ECSS-Q-70-71A |

---

## 8 · Backend-Services (on-demand)

### 8.1 CAD-Konverter

```
POST /api/cad/convert
Content-Type: multipart/form-data

file: shaft.step
target_format: gltf

→ 200 OK  { "url": "/api/cad/result/abc123.glb", "vertices": 12480, "faces": 24960 }
```

**Implementation:** `ethos_ai_worlds/api/cad_router.py` (~300 LOC)
- CadQuery liest STEP/IGES → Trimesh konvertiert → GLTF/GLB Export
- Timeout: 30s, Max-Dateigröße: 100 MB
- Ergebnis-Cache: LRU mit SHA-256-Hash des Inputs

### 8.2 FEM-Service

```
POST /api/fem/analyze
Content-Type: application/json

{
  "mesh_url": "/api/cad/result/abc123.glb",
  "material": "S235JR",
  "loads": [{"point": [0, 0.25, 0], "force": [0, -5000, 0]}],
  "constraints": [{"point": [0, 0, 0], "type": "fixed"}],
  "analysis_type": "static"
}

→ 200 OK  {
    "max_von_mises_mpa": 187.3,
    "max_displacement_mm": 0.42,
    "factor_of_safety": 1.26,
    "vertex_stress": [12.1, 15.3, ...],  // pro Vertex
    "vertex_displacement": [[0.01, -0.02, 0], ...]
}
```

**Implementation:** `ethos_ai_worlds/api/fem_router.py` (~400 LOC)
- SfePy für statische Analyse (Spannung, Verformung)
- Rückgabe als JSON mit Per-Vertex-Daten für Colormap
- Timeout: 60s

### 8.3 Physics-Batch (optional, MuJoCo)

Nur für Kunden, die akkurate kontaktintensive Simulation benötigen (Getriebe, Robotik). Der Client schickt eine Konfiguration, der Server simuliert und streamt die Ergebnisse via WebSocket.

---

## 9 · SimGrid-Brücke: Text meets Physik

### 9.1 Integration mit der bestehenden 6-Stufen-Pipeline

Die SimGrid-Text-Pipeline (ComplexityRouter → ExperienceHarvester) erhält physikalische Fakten als kontextuellen Input:

```python
# Physik-Ergebnisse als SimGrid-Input
physics_context = {
    "modality": "CAD_MODEL",  # ComplexityRouter → L2 SIMULATIVE
    "engineering_data": {
        "component": "Welle Ø50 × 500 mm, S235JR",
        "load_case": "Biegung: 5 kN Querkraft bei L/2",
        "max_von_mises_mpa": 187.3,
        "yield_strength_mpa": 235,
        "factor_of_safety": 1.26,
        "max_displacement_mm": 0.42,
        "norm_reference": "DIN 743 (Wellen/Achsen)"
    }
}

# RoleSimulator.ENGINEER evaluiert spezifisch:
# → "Sicherheitsfaktor 1.26 liegt unter DIN-Empfehlung 1.5 für dynamische Last"
# → "Empfehlung: S355J2 oder Durchmesser auf 55 mm erhöhen"
```

### 9.2 Neue Modalität in ComplexityRouter

Der `ComplexityRouter` kennt bereits die Modalität `CAD_MODEL`. Diese routet automatisch auf **L2 SIMULATIVE** (volle SimGrid-Pipeline mit Consequence-Trees).

### 9.3 ENGINEER-Rolle erweitert

Der `RoleSimulator` hat bereits die Rolle **ENGINEER**. Diese wird um physikalische Bewertungskriterien erweitert:

| Kriterium | Gewichtung | Quelle |
|---|---|---|
| Sicherheitsfaktor vs. Norm | 30% | DIN 743, VDI 2230, FKM-Richtlinie |
| Materialausnutzung | 25% | $\sigma_{vM} / R_{p0.2}$ |
| Verformung vs. Toleranz | 20% | Kundenspezifikation |
| Ermüdungsfestigkeit | 15% | DIN 50100, Wöhler-Kurve |
| Herstellbarkeit | 10% | Fertigungstechnologie + Kosten |

---

## 10 · Sicherheitsüberlegungen (aktualisiert)

| Aspekt | Maßnahme | SEC-Tier |
|---|---|---|
| **Client-Physics** | Rapier.js läuft isoliert im Browser-Tab (Same-Origin-Policy) | Alle |
| **STEP-Upload** | Max. 100 MB, MIME-Validierung, kein Executable-Content | Alle |
| **FEM-Timeout** | Max. 60s pro Analyse, Worker-Process mit Resource-Limits | Alle |
| **CAD-Konverter** | Subprocess mit Timeout + Memory-Limit (512 MB) | Alle |
| **Daten-Isolation** | Jeder World-Pack hat eigenes Datenverzeichnis | Alle |
| **GPU-Zugriff** | Nur WebGL via Three.js, keine native GPU-API | Alle |
| **SEC-Tier-Enforcement** | `min_tier` in World-Manifest, `enforce_tier_policy()` im Router | ≥ Tier 1 |
| **Air-Gap** | Rapier WASM + Three.js lokal bundlebar, kein CDN | Tier 3 |

---

## 11 · Implementierungsplan v2.0.0

### Phase 1 — SimGrid Engineering World (Client-seitig)

**Scope:** Interaktive 3D-Physik im Browser — kein neuer Backend-Code nötig.

| # | Deliverable | LOC (Schätzung) |
|---|---|---|
| 1.1 | `SimGridWorldView (frontend module)` (SimGridWorldView extends WorldBase) | ~1.500 |
| 1.2 | Rapier.js WASM Integration (Physics-Step in `_animate()`) | ~400 |
| 1.3 | Force-Application-UI (Kraftpfeile, Lager-Marker) | ~300 |
| 1.4 | Material-Inspector PAD (Auswahl aus materials.py) | ~200 |
| 1.5 | ECS-Erweiterung: `material_eng`, `loads`, `constraints` Components | ~100 |
| 1.6 | World Pack `simgrid-3d` (metadata.json, scene.json, Rapier WASM) | ~50 |
| 1.7 | 7 MCP-Tools (simgrid_create_body, _apply_force, etc.) | ~300 |
| 1.8 | Tests (≥ 30) | ~500 |
| | **Gesamt Phase 1** | **~3.350** |

**Ergebnis:** Interaktive 3D-Physik-Simulation direkt im Browser — World-nativ, Multi-User-fähig, Marketplace-ready.

### Phase 2 — CAD-Konverter (STEP → GLTF)

| # | Deliverable | LOC (Schätzung) |
|---|---|---|
| 2.1 | `ethos_ai_worlds/api/cad_router.py` (POST /api/cad/convert) | ~300 |
| 2.2 | CadQuery + Trimesh Integration | ~200 |
| 2.3 | Frontend: STEP-Upload-Button in SimGridWorld | ~100 |
| 2.4 | Tests (≥ 15) | ~250 |
| | **Gesamt Phase 2** | **~850** |

**Neue Dependencies:** `cadquery`, `trimesh` (~100 MB mit OpenCascade)

### Phase 3 — FEM-Service (Spannungsanalyse)

| # | Deliverable | LOC (Schätzung) |
|---|---|---|
| 3.1 | `ethos_ai_worlds/api/fem_router.py` (POST /api/fem/analyze) | ~400 |
| 3.2 | SfePy Integration (statische Analyse) | ~300 |
| 3.3 | Frontend: Stress-Colormap (Vertex-Coloring) | ~250 |
| 3.4 | Frontend: Deformation-View (überhöht) | ~150 |
| 3.5 | SimGrid-Brücke: Physics→Text-Pipeline | ~200 |
| 3.6 | Tests (≥ 20) | ~400 |
| | **Gesamt Phase 3** | **~1.700** |

**Neue Dependencies:** `sfepy` (~50 MB)

### Phase 4 — HiFi-Physics (optional, MuJoCo)

| # | Deliverable | LOC (Schätzung) |
|---|---|---|
| 4.1 | MuJoCo Concrete Implementation für `PhysicsSimulator` ABC | ~400 |
| 4.2 | WebSocket-Streaming (Server→Client Simulation-Frames) | ~300 |
| 4.3 | Frontend: Frame-by-Frame-Playback | ~200 |
| 4.4 | Tests (≥ 10) | ~200 |
| | **Gesamt Phase 4** | **~1.100** |

**Neue Dependencies:** `mujoco` (~350 MB)

### Meilenstein-Übersicht

| Phase | Fulfillment danach | Dependencies | Standalone-nutzbar? |
|---|---|---|---|
| **Phase 1** | 🟡 60% | Rapier.js WASM (~200 KB) | ✅ Ja |
| **Phase 2** | 🟡 75% | + CadQuery, Trimesh (~100 MB) | ✅ Ja |
| **Phase 3** | 🟢 90% | + SfePy (~50 MB) | ✅ Ja |
| **Phase 4** | 🟢 100% | + MuJoCo (~350 MB) | ✅ Ja |

---

## 12 · Abhängigkeiten (gesamt, alle Phasen)

### Client-seitig (Phase 1)
```
@dimforge/rapier3d-compat    # Apache 2.0, 🇫🇷, ~200 KB WASM
# Three.js r152 bereits geladen
```

### Server-seitig (Phase 2–4, on-demand)
```
cadquery >= 2.4              # Apache 2.0, 🇫🇷 OCC Basis
cadquery-ocp >= 7.9          # Apache 2.0, 🇫🇷
trimesh >= 4.5               # MIT
sfepy >= 2024.1              # BSD, 🇨🇿
mujoco >= 3.5.0              # Apache 2.0 (optional, Phase 4)
```

| Phase | Neuer Disk-Footprint |
|---|---|
| Phase 1 (Client) | ~200 KB |
| Phase 2 (+ CAD) | ~100 MB |
| Phase 3 (+ FEM) | ~150 MB |
| Phase 4 (+ MuJoCo) | ~500 MB |

---

## 13 · Bestehende World Packs als Basis

Zwei bereits existierende World Packs bilden den inhaltlichen Rahmen:

### `simgrid-strukturmechanik`
- **Titel:** SimGrid Structural Mechanics Test Field
- **Zweck:** Virtuelles Prüflabor für Zug, Biegung, Torsion, Ermüdung, Schwingung
- **Normen:** DIN EN ISO 6892, DIN 50100, VDI 2230
- **Status:** Pack-Struktur vorhanden, 3D-Physik fehlt → wird durch Phase 1 aktiviert

### `material-prueflab`
- **Titel:** Material Test Lab
- **Zweck:** Werkstoffprüfung und Materialkennwerte
- **Status:** Pack-Struktur vorhanden → wird durch Phase 1 + 3 aktiviert

---

---

## Appendix A — Altes Konzept v1.0.0 (Februar 2026)

> *Die folgenden Abschnitte dokumentieren den ursprünglichen monolithischen Ansatz aus v1.0.0. Sie bleiben als Referenz erhalten. Die empfohlene Architektur ist v2.0.0 (§3–§12 oben).*

### A.1 Ursprüngliches Schichtenmodell (v1.0.0)

```
┌──────────────────────────────────────────────────────────────┐
│                       API Layer                              │
│         FastAPI Endpoints: /api/simgrid3d/*                  │
├──────────────────────────────────────────────────────────────┤
│                    Orchestrator Layer                        │
│          SimGrid3DOrchestrator                               │
│   ┌──────────────┬──────────────┬──────────────────────┐     │
│   │ CAD Service  │ Physics Svc  │ Visualization Svc    │     │
│   │ (CadQuery)   │ (MuJoCo)     │ (PyVista)            │     │
│   └──────────────┴──────────────┴──────────────────────┘     │
├──────────────────────────────────────────────────────────────┤
│                    Mesh Exchange Layer                       │
│                    Trimesh (Converter)                       │
│         STL ↔ OBJ ↔ GLTF ↔ PLY ↔ MJCF ↔ STEP                 │
├──────────────────────────────────────────────────────────────┤
│                    Analysis Layer                            │
│   ┌──────────────┬──────────────┬──────────────────────┐     │
│   │ FEM Service  │ FMEA Module  │ Dynamics Service     │     │
│   │ (SfePy)      │ (fmea.py)    │ (PyDy)               │     │
│   └──────────────┴──────────────┴──────────────────────┘     │
├──────────────────────────────────────────────────────────────┤
│              Bestehender Digital Twin SimGrid                │
│    ComplexityRouter → ScenarioGenerator → RoleSimulator      │
│    → ConsequenceSimulator → DecisionSynthesizer              │
│    → ExperienceHarvester                                     │
└──────────────────────────────────────────────────────────────┘
```

### A.2 Ursprünglicher Implementierungsplan (9 Wochen)

| Phase | Woche | Deliverable |
|---|---|---|
| **P1** | KW 09 | SimGrid 3D Orchestrator Module |
| **P2** | KW 10 | `simulation/cad_service` module — CadQuery Integration |
| **P3** | KW 10 | `simulation/physics_service` module — MuJoCo Integration |
| **P4** | KW 11 | `simulation/visualization_service` module — PyVista |
| **P5** | KW 11 | `simulation/mesh_exchange` module — Trimesh Converter |
| **P6** | KW 12 | `simulation/fem_service` module — SfePy FEM |
| **P7** | KW 12 | `simulation/fmea_module` module — FMEA Engine |
| **P8** | KW 13 | Materials Database (JSON, 10+ Werkstoffe) |
| **P9** | KW 13 | API-Endpoints + Tests (≥ 50 Tests) |

### A.3 Ursprüngliche Abhängigkeiten (~500 MB)

```
mujoco >= 3.5.0        # Apache 2.0
pyvista >= 0.44         # MIT
trimesh >= 4.5          # MIT
cadquery >= 2.4         # Apache 2.0
cadquery-ocp >= 7.9     # Apache 2.0
sfepy >= 2024.1         # BSD
pydy >= 0.7             # BSD
```

### A.4 Warum v1.0.0 nicht umgesetzt wurde

| Problem | Beschreibung |
|---|---|
| **Monolithisch** | Alle 500 MB Dependencies auf einmal nötig, auch nur für einfache Physik |
| **Server-Rendering** | PyVista rendert auf dem Server — kein Echtzeit-Feedback im Browser |
| **Keine World-Integration** | Parallel zum World-Framework statt darauf aufbauend |
| **Kein Multi-User** | Nicht vorgesehen — Worlds hat es inzwischen |
| **Keine EU-Prüfung** | Herkunft und Lizenz-Compliance nicht bewertet |
| **Kein Marketplace** | Nicht verteilbar als World Pack |
| **Overtaken by events** | Worlds-Framework (v1.68.0–v1.90.0) hat Infrastruktur-Lücken geschlossen |

---

*Dieses Konzept ist ein lebendes Dokument. Nächster Meilenstein: Phase 1 (SimGridWorld + Rapier.js).*
