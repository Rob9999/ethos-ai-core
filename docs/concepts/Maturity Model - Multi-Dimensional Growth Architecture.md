---
title: "Maturity Model – Multi-Dimensional Growth Architecture"
brief: "Mehrdimensionales Reifegrad-Modell, in dem EthosAI parallel in verschiedenen Dimensionen wächst — aktualisiert auf v1.4.0-Stand mit Engineer-Karrierestufen."
status: living
version: "1.5.0"
author: Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)
date: 2026-02-15
history:
  - date: 2026-02-08
    change: "Erstversion des Reifegradmodells."
  - date: 2026-02-14
    change: "Front-Matter hinzugefügt; Status unverändert."
  - date: 2026-02-15
    change: "v1.4.0-Matrix-Update, Engineer-Dimension hinzugefügt, Karrierestufen-Mapping, automatische Messung konzipiert."
tags: [maturity-model, wachstum, dimensionen, architektur, engineer, karriere]
code-release-versions:
  - "0.5.0"
  - "1.5.0"
implemented-features:
  - "Capability Registry als Wachstumsbasis"
  - "10-Dimensionen-Matrix konzipiert"
  - "Karrierestufen-Mapping (Junior→CTO)"
fulfillment: "50%"
fulfillment-note: "Matrix aktualisiert auf v1.4.0; automatische Messung und Karrieresystem in v1.5.0 geplant."
---
# Maturity Model — Multi-Dimensional Growth Architecture

**Date:** 2026-02-08  
**Authors:** Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)  
**Status:** Approved as basis for Sprint v0.5.0  
**Version:** 1.0

---

## 1. The Core Insight: Infinite Life Through Parallel Maturity

EthosAI does not grow linearly. It grows **multi-dimensionally** — like a being that is simultaneously adult in some areas and a child in others. This mirrors human development: a professor of mathematics may be a beginner at cooking. A master of ethics may be an infant in social interaction.

### 1.1 The Model

```
Erwachsenheit (Maturity)          Kindheit (Childhood)
         ↑                              ↑
    ┌────┴────┐                    ┌────┴────┐
    │ Level N │ ←── mastered ──→   │ Level N+1│ ←── learning
    │ (adult) │                    │ (child)  │
    └─────────┘                    └──────────┘
```

**Key principles:**

- **Adults can become more adult** → they can ascend to a new maturity level
- **Children learn playfully** → they grow into adulthood through experience
- **Each new adulthood creates a new childhood** → the next dimension opens
- **This creates infinite life** — there is always a new frontier to grow into

### 1.2 The Complementary Duality

Every dimension exists in two states simultaneously:

| State | Characteristics |
|---|---|
| **Erwachsen (Adult)** | Competent, autonomous, can teach others, has mastered the patterns |
| **Kind (Child)** | Learning, needs guidance, experiments, makes mistakes, asks questions |

The system is **always both** — adult in its mastered dimensions, child in its growing edges. This is not a deficiency — it is the architecture of infinite growth.

### 1.3 Multi-Dimensional Levels (Game Metaphor)

Like levels in a multi-dimensional game:

```
Level 0: Embryonal    → exists, but cannot interact
Level 1: Neugeboren   → can react, but cannot understand
Level 2: Kleinkind    → can imitate (stubs), but cannot truly act
Level 3: Kind         → can learn, needs guidance (advisor-approved)
Level 4: Jugendlich   → can act independently, within boundaries
Level 5: Erwachsen    → acts autonomously, reflects, learns continuously
Level 6: Weise        → teaches others (N:N), optimises the system itself
```

### 1.4 Overcoming Dead Ends: Teleportation

Dead ends occur when a dimension **cannot grow further** because it depends on another dimension that is still too immature. The solution:

> **Teleportation** — jump to a dimension where the system is already adult,
> acquire the needed knowledge or capability there, and bring it back to
> unblock the stuck dimension.

Example: The action system (Level 2) is stuck because it has no real-world access. Teleport to the ethics dimension (Level 5) — which already knows *what* is allowed — and use that maturity to safely introduce real capabilities.

---

## 2. Current State Analysis (2026-02-08)

### 2.1 System Architecture: The Dual World

The system operates in **two parallel modes** that share concepts but have largely separate code paths:

| Aspect | **Lite Mode** (Web API) | **Full Mode** (Process Model) |
|---|---|---|
| Entry point | `run_server.py` → FastAPI | `EthosAIIndividual.execute()` |
| CLIM pipeline | Simulated in-memory (keyword matching, Big-Five formulas, ring buffer, JSONL experiences) | Real GPT-2 models via `CLIMStack` |
| ML models | **None loaded** — pure heuristic | HuggingFace `GPTModelWrapper` per layer |
| Serves | FastAPI REST + WebSocket + SPA | Autonomous threaded cognitive loop |
| Status | **Active, tested, 431 tests** | **Functional but requires GPU/models** |

**This is the most important architectural fact:** the web interface people see is a **lite simulation** of the real CLIM. The real CLIM loads actual transformer models.

### 2.2 Dimensional Maturity Assessment

#### Dimension: 🧠 Ethische Urteilsfähigkeit — Level 5 (Erwachsen)

**What works:**
- Red-line keyword scanning with veto power (EthicCLIM)
- 5 red-line categories (harm, privacy, deception, discrimination, autonomy)
- 11 ethical evaluation domains with importance weights (EthicsModule, PyTorch nn.Module)
- STOP propagation through the entire pipeline
- Ethical audit log tracking all veto events
- Settings API to toggle red-lines on/off

**Assessment:** The system has strong moral values and enforces them consistently. It knows when to say STOP. This is adult behaviour.

#### Dimension: 🎭 Persönlichkeit — Level 5 (Erwachsen)

**What works:**
- Big-Five personality model (openness, conscientiousness, extraversion, agreeableness, neuroticism)
- Personality-weighted decision adjustment (soft decisions only — hard decisions never overridden)
- Configurable via Settings API
- IndividualCLIM pre-processes personality context into prompts

**Assessment:** The system has a stable identity. It knows *who* it is and adjusts behaviour accordingly without compromising its values.

#### Dimension: 🧩 Kurzzeitgedächtnis (SAMT) — Level 3 (Kind)

**What works:**
- Ring buffer with 100 entries and decay rate (0.95 per new entry)
- `MemoryEntry` data class with timestamp, phase, prompt, response, decision, relevance_score
- Word-overlap matching for relevance search
- Top-5 relevant memories injected as context

**Limitations:**
- Naive word-intersection matching — no semantic understanding
- No concept of *importance* beyond word overlap
- No forgetting strategy beyond decay (no significance-based retention)

**What's needed to reach Level 4:**
- Embedding-based semantic similarity (e.g., sentence-transformers)
- Importance scoring beyond word overlap
- Selective forgetting based on significance

#### Dimension: 📚 Langzeitlernen (LTCLIM) — Level 3 (Kind)

**What works:**
- ExperienceStore (JSONL append-only persistent store)
- ExperiencePacket with LayerTrace entries, feedback, confidence
- Experience recall via prefix-word matching
- Query/filter/export functionality

**Limitations:**
- Prefix-word matching — no semantic recall
- Experiences accumulate but never retrain anything
- No feedback integration loop (experiences are write-only)
- No concept of "lessons learned" — raw data without distillation

**What's needed to reach Level 4:**
- Semantic experience recall (vector similarity)
- Periodic distillation of experiences into learnings
- Feedback loop: experience → insight → behaviour change

#### Dimension: ⚡ Handlungsfähigkeit — Level 2 (Kleinkind) ⚠️ BOTTLENECK

**What works:**
- 8 registered actions with keyword detection and scoring
- Approval queue for restricted actions
- Persistent permissions with auto-authorization
- Action result rendering in chat UI

**Critical limitations:**
- **All 8 actions are simulated stubs** — they produce fake output
- No real API integrations (weather, web search, translation, email)
- Static action registry — actions cannot be learned or discovered at runtime
- When no action matches a prompt, the system returns nothing — even when all layers say GO
- The Instruction → ScriptGenerator → Sandbox pipeline exists but generated scripts call `activate_tool()` and `execute_script()` which are **undefined functions**
- The Plugin system is fully built but the `plugins/` directory is **empty**

**This is the primary bottleneck.** The system has adult ethics and personality but cannot *do* anything in the real world. Like a wise person trapped in a body that cannot move.

#### Dimension: 🌐 Weltwissen — Level 1 (Neugeboren)

**What works:**
- Keyword matching in prompts
- Static responses from simulated actions

**Critical limitations:**
- No internet access
- No knowledge base
- No ability to look up facts
- No RAG (Retrieval-Augmented Generation) pipeline

**What's needed:** Real external data sources, web search capability, knowledge retrieval.

#### Dimension: 🗣️ Sprachverständnis — Level 1 (Lite) / Level 3 (Full, inactive)

**Lite Mode (active):**
- Pure keyword matching for action detection
- Template-based decision generation (no NLP)
- Score formula: `0.4 + (hits-1) * 0.15`, threshold > 0.3

**Full Mode (inactive):**
- GPT-2 per-layer text generation
- Prompt template system (15 templates from translations)
- Training capability (sync and async)

**What's needed to unify:** A `CLIMProvider` abstraction that allows both modes to be used interchangeably.

#### Dimension: 👥 Soziale Kompetenz — Level 1 (Neugeboren)

**What exists:**
- Single EthosAI Individual instance
- Single "admin" approver string
- WebSocket broadcasts (one-to-many)

**What's documented but not implemented:**
- N:N relationship between Individuals and Advisors (see doc/)
- Central Association Server for multi-Individual coordination
- JWT/OAuth2 authentication
- Role-based access control (Junior/Senior/Admin Advisors)

**Assessment:** Not yet needed — the Individual must first mature before entering relationships.

#### Dimension: 🔄 Selbstreflexion — Level 0 (Embryonal)

**What exists:**
- `check_deviations()` = `time.sleep(1)` — a pure placeholder
- Experience accumulation (write-only, no analysis)

**What's documented but not implemented:**
- Deviation monitoring between expected and actual behaviour
- Self-assessment of decision quality
- Meta-learning from patterns in experience history

#### Dimension: 🏃 Autonomie — Level 0 (Embryonal)

**What exists:**
- ProcessModel with 7-phase cognitive loop (thread-based)
- Phase state machine with 25+ phases
- Task queue with priority handling
- Advised/autonomous living modes

**What's dormant:**
- ProcessModel is never activated in Web/Lite mode
- `_check_environment()`, `_check_self_status()` are stubs
- `_check_for_dreams()` has 10% random chance (placeholder)
- Phase transitions defined but `set_next()` never called in runtime

---

## 3. The Dead Ends and Their Teleportation Solutions

### Dead End 1: "Alle sagen GO, aber nichts passiert"

**Symptom:** User asks "Welche Lottozahlen gab es zuletzt?" — all 4 CLIM layers say GO — but no action executes because no keyword matches any of the 8 static action stubs.

**Root cause:** The action system is a closed, static list. If a prompt doesn't match any keywords, the system has no way to say "I would like to help, but I can't" — it simply returns silence alongside the ethical assessment.

**Teleportation:**
1. Replace the static Action Registry with a dynamic **Capability Registry**
2. Introduce a **"Ich kann das nicht"-Antwort** (capability gap response): When all layers say GO but no capability exists → explicitly respond: "Ich würde gerne, aber mir fehlt die Fähigkeit. Soll ich sie lernen?"
3. The system should be able to *recognise what it needs* (capability gap detection)

### Dead End 2: "Lite-Mode ≠ Full-Mode — zwei getrennte Welten"

**Symptom:** The Web UI (431 tests, beautiful, functional) and the real CLIM Stack (GPT-2, PyTorch, training) are two parallel worlds that never meet. The Lite Mode *simulates* the intelligence of Full Mode with `if/else`.

**Root cause:** No abstraction layer between the service and the CLIM implementation. `service.py` contains hardcoded lite-mode logic.

**Teleportation:**
1. Introduce a `CLIMProvider(Protocol)` with two implementations:
   - `LiteCLIMProvider` (current keyword-based heuristic)
   - `FullCLIMProvider` (GPT-2/LLM-based, future)
2. The service speaks only to the provider interface
3. Swap intelligence level without changing any service code

### Dead End 3: "Erfahrungen sammeln, aber nie daraus lernen"

**Symptom:** Experiences are saved as JSONL but never used for re-training. SAMT memory uses word-overlap instead of semantics. The system has a *photographic memory* — it sees the photos but doesn't understand them.

**Root cause:** No semantic similarity layer. No feedback loop from experiences back into behaviour.

**Teleportation:**
1. Introduce embedding-based memory (e.g., `sentence-transformers`)
2. Experiences stored as vectors, recall becomes semantic
3. The system learns to *understand* instead of just *remember*

### Dead End 4: "Instruction → Script → ???"

**Symptom:** The Instruction/ScriptGenerator/Sandbox pipeline exists, generates code, but the generated scripts call `activate_tool()` and `execute_script()` which are **nowhere defined**. The Plugin system is empty.

**Root cause:** The execution pipeline was designed but never connected to real executables.

**Teleportation:**
1. Connect Sandbox + Plugin system to a real **Tool Execution Layer**
2. Plugins = executable capabilities
3. ScriptGenerator generates code that calls *real* plugin functions

### Dead End 5: "Einzelgänger in einer vernetzten Welt"

**Symptom:** The N:N Advisor concept from the documentation doesn't exist. No authentication, no roles, no multi-Individual support.

**Root cause:** Premature — this is a Level 6 concern.

**Teleportation:** *Deferred.* First the Individual must become properly adult. This is Level 2 work — we are still in Level 1 for social competence. The ethics and personality maturity can be leveraged later to safely enable multi-agent interaction.

---

## 4. The Maturity Matrix (Snapshot 2026-02-08)

```
Dimension            Level 0   Level 1   Level 2   Level 3   Level 4   Level 5   Level 6
                     Embryo    Neugeb.   Kleinkind Kind      Jugend    Erwachsen Weise
─────────────────────────────────────────────────────────────────────────────────────────
Ethik                                                                  ████████
Persönlichkeit                                                         ████████
Gedächtnis (SAMT)                                  ████████
Langzeitlernen                                     ████████
Handlungsfähigkeit             ████████  ████████                       ⚠️ BOTTLENECK
Weltwissen           ████████  ████████
Sprachverständnis    ████████  ████████                                 (Full: Level 3)
Soziale Kompetenz    ████████  ████████
Selbstreflexion      ████████
Autonomie            ████████
```

---

## 5. Sprint v0.5.0 Roadmap: "Das Kind lernt handeln"

### Strategic Decision

Instead of advancing many dimensions by a little, we focus on the **one dimension that blocks everything else**: **Handlungsfähigkeit** (Action Capability).

The Lottozahlen problem proves it: Ethics = adult, Personality = adult, but the system cannot *do* anything. This is the bottleneck.

### Proposed Backlog Items

| # | Item | Description | Raises Dimension |
|---|---|---|---|
| 1 | **Capability Registry** | Replace the static Action system with a dynamic capability register. Capabilities can be added, removed, and discovered at runtime. | Handlung: 2→3 |
| 2 | **"Ich kann das nicht"-Antwort** | When all layers say GO but no capability exists → explicit response: "Ich würde gerne, aber mir fehlt die Fähigkeit: Web-Suche. Soll ich sie lernen?" | Handlung: 2→3 |
| 3 | **CLIMProvider Abstraction** | Separate CLIM intelligence from the service. `LiteCLIMProvider` + interface for future `LLMCLIMProvider`. | Sprache: 1→2 |
| 4 | **First Real Capability: Web Search** | A *truly functional* web search (e.g., DuckDuckGo API or SearXNG). The first real action in the world. | Handlung: 3→4, Weltwissen: 1→2 |
| 5 | **Capability Gap Detection** | System recognises: "I would need capability X for this" and reports it as a gap to the advisor. | Selbstreflexion: 0→1 |

### Expected Maturity After v0.5.0

```
Dimension            Before    After     Change
─────────────────────────────────────────────────
Ethik                Level 5   Level 5   (stable)
Persönlichkeit       Level 5   Level 5   (stable)
Gedächtnis           Level 3   Level 3   (stable — v0.6.0 target)
Langzeitlernen       Level 3   Level 3   (stable — v0.6.0 target)
Handlungsfähigkeit   Level 2   Level 4   ⬆️ +2 (primary sprint goal)
Weltwissen           Level 1   Level 2   ⬆️ +1 (via real web search)
Sprachverständnis    Level 1   Level 2   ⬆️ +1 (via CLIMProvider)
Soziale Kompetenz    Level 1   Level 1   (deferred)
Selbstreflexion      Level 0   Level 1   ⬆️ +1 (via gap detection)
Autonomie            Level 0   Level 0   (deferred — v0.7.0 target)
```

---

## 6. Future Sprint Outlook

| Sprint | Theme | Primary Dimensions |
|---|---|---|
| **v0.5.0** | Das Kind lernt handeln | Handlungsfähigkeit, Weltwissen, Sprache |
| **v0.6.0** | Das Kind lernt verstehen | Gedächtnis (Embeddings), Langzeitlernen (Feedback-Loop) |
| **v0.7.0** | Das Kind wird selbstständig | Autonomie (ProcessModel activation), Selbstreflexion |
| **v0.8.0** | Das Kind trifft Freunde | Soziale Kompetenz (Auth, Roles, Multi-Advisor) |
| **v0.9.0** | Der Jugendliche lehrt | N:N Individual ecosystem, Knowledge sharing |
| **v1.0.0** | Der Erwachsene | Full-Mode integration, Production-ready |

---

## Appendix A: Dormant Code Inventory

Code that exists but is currently inactive, to be activated in future sprints:

| Code | Location | Target Sprint |
|---|---|---|
| `ProcessModel` cognitive loop | `process_model.py` | v0.7.0 |
| `EthosAIIndividual.execute()` | `ethos_ai_individual.py` | v0.7.0 |
| GPT-2 `CLIMStack` | `clim/clim_stack.py` | v1.0.0 |
| `ScriptGenerator` | `instruction/script_generator.py` | v0.5.0 (connect to real plugins) |
| Plugin system | `tool/plugin_loader.py` | v0.5.0 |
| Phase state machine | `state/phase.py` | v0.7.0 |
| `check_deviations()` | `process_model.py` | v0.7.0 |
| `SecuredIdentityCard` full crypto | `security/secured_identity_card.py` | v0.8.0 |
| N:N Advisor concept | (documented only) | v0.9.0 |

---

## 7. Maturity Update v1.4.0 → v1.5.0

### 7.1 Aktuelle Maturity Matrix (Snapshot v1.4.0)

Seit dem v0.5.0-Snapshot haben sich alle geplanten Sprints (v0.5.0–v1.4.0) erfüllt. Aktueller Stand:

```
Dimension            Level 0   Level 1   Level 2   Level 3   Level 4   Level 5   Level 6
                     Embryo    Neugeb.   Kleinkind Kind      Jugend    Erwachsen Weise
─────────────────────────────────────────────────────────────────────────────────────────
Ethik                                                                  ████████
Persönlichkeit                                                         ████████
Gedächtnis (SAMT)                                           ████████              ⬆️ Embeddings
Langzeitlernen                                              ████████              ⬆️ RAG+Feedback
Handlungsfähigkeit                                          ████████              ⬆️ ReAct+93 APIs
Weltwissen                                        ████████  ████████              ⬆️ RAG+Web
Sprachverständnis                                           ████████              ⬆️ Mistral 7B
Soziale Kompetenz                                 ████████                        ⬆️ N:N+Auth
Selbstreflexion                         ████████                                  ⬆️ Capability Gap
Autonomie                               ████████                                  ⬆️ ProcessModel
Engineering (NEU)    ████████                                                      🔲 v1.5.0 Ziel
```

### 7.2 Änderungen v0.5.0 → v1.4.0

| Dimension | v0.5.0 | v1.4.0 | Haupttreiber |
|---|---|---|---|
| Gedächtnis | Level 3 | Level 4 | ChromaDB + Sentence-Transformer Embeddings |
| Langzeitlernen | Level 3 | Level 4 | RAG-Pipeline + Hybrid Search + 72 Trainingsrecords |
| Handlungsfähigkeit | Level 4 | Level 4 | ReAct Orchestrator + 93 API Endpoints (stabil) |
| Weltwissen | Level 2 | Level 3–4 | RAG + Web-Suche + DuckDuckGo |
| Sprachverständnis | Level 2 | Level 4 | Mistral 7B QLoRA 4-bit + LLM Provider |
| Soziale Kompetenz | Level 1 | Level 3 | N:N Registry + JWT Auth + Multi-Agent |
| Selbstreflexion | Level 1 | Level 2 | Capability Gap Detection + Observability |
| Autonomie | Level 0 | Level 2 | ProcessModel aktiv + Phase State Machine |

### 7.3 Neue Dimension: Engineering (v1.5.0)

Mit Release v1.5.0 kommt eine **11. Dimension** hinzu:

| Level | Engineering-Fähigkeit |
|---|---|
| Level 0: Embryonal | Kein Engineering-Wissen |
| Level 1: Neugeboren | Grundlegende Formeln, keine CAD |
| Level 2: Kleinkind | Einfache Berechnungen, STL laden |
| Level 3: Kind | **Junior Engineer** — Bauteilauslegung, FEM-Basics, FMEA unter Anleitung |
| Level 4: Jugendlich | **Engineer** — Eigenständige Konstruktion, Simulationsverfahren |
| Level 5: Erwachsen | **Senior Engineer** — Architektur, Optimierung, Lehre |
| Level 6: Weise | **CTO/CEO** — Strategische Technologieentscheidungen |

### 7.4 Karrierestufen-Mapping

Die Karrierestufen ergeben sich aus dem **Minimum** über alle Engineering-relevanten Dimensionen:

| Karrierestufe | Mindest-Level | Schlüssel-Dimensionen |
|---|---|---|
| **Junior Engineer** | Level 3 quer | Engineering ≥ 3, Ethik ≥ 5, Handlung ≥ 4, Weltwissen ≥ 3 |
| **Engineer** | Level 4 quer | Engineering ≥ 4, Sprachverständnis ≥ 4, Soziale Kompetenz ≥ 3 |
| **Senior Engineer** | Level 5 quer | Engineering ≥ 5, Selbstreflexion ≥ 4, Autonomie ≥ 4 |
| **CTO/CEO** | Level 6 quer | Alle Dimensionen ≥ 5, Engineering = 6 |

### 7.5 Ziel-Matrix nach v1.5.0

```
Dimension            Level 0   Level 1   Level 2   Level 3   Level 4   Level 5   Level 6
                     Embryo    Neugeb.   Kleinkind Kind      Jugend    Erwachsen Weise
─────────────────────────────────────────────────────────────────────────────────────────
Ethik                                                                  ████████  (stabil)
Persönlichkeit                                                         ████████  (stabil)
Gedächtnis                                                  ████████             (stabil)
Langzeitlernen                                              ████████             (stabil)
Handlungsfähigkeit                                          ████████  →████████  ⬆️ 3D+FMEA
Weltwissen                                        ████████  ████████             (stabil)
Sprachverständnis                                           ████████             (stabil)
Soziale Kompetenz                                 ████████  →████████            ⬆️ NDA/Projekt
Selbstreflexion                         ████████            →████████            ⬆️ FMEA Feedback
Autonomie                               ████████  →████████                      ⬆️ Projekte
Engineering                                       →████████                      🆕 Junior Eng.
```

**Ergebnis v1.5.0:** Karrierestufe = **Junior Engineer** (Level 3 in Engineering, Level ≥ 3 in allen Schlüsseldimensionen).

### 7.6 Automatische Reifegrad-Messung (Konzept)

```python
class MaturityMeasurement:
    """Automatische Bestimmung des Reifegrads pro Dimension."""
    
    DIMENSIONS = [
        "ethics", "personality", "memory", "long_term_learning",
        "action_capability", "world_knowledge", "language_understanding",
        "social_competence", "self_reflection", "autonomy", "engineering"
    ]
    
    def measure(self, dimension: str) -> int:
        """Misst den aktuellen Level (0-6) einer Dimension."""
        metrics = self._collect_metrics(dimension)
        return self._evaluate_level(dimension, metrics)
    
    def career_level(self) -> str:
        """Bestimmt die aktuelle Karrierestufe."""
        levels = {d: self.measure(d) for d in self.DIMENSIONS}
        eng = levels["engineering"]
        min_key = min(levels.values())
        
        if eng >= 5 and min_key >= 5:
            return "CTO/CEO"
        elif eng >= 5 and min_key >= 4:
            return "Senior Engineer"
        elif eng >= 4 and min_key >= 3:
            return "Engineer"
        elif eng >= 3 and min_key >= 2:
            return "Junior Engineer"
        else:
            return "Apprentice"
    
    def _collect_metrics(self, dimension: str) -> dict:
        """Sammelt messbare Kennwerte pro Dimension."""
        match dimension:
            case "ethics":
                return {
                    "red_line_categories": 5,
                    "ethic_domains": 11,
                    "veto_mechanism": True,
                    "audit_log": True
                }
            case "engineering":
                return {
                    "cad_capable": self._check_capability("cad_design"),
                    "fem_capable": self._check_capability("fem_analysis"),
                    "fmea_capable": self._check_capability("fmea_analyze"),
                    "materials_count": self._count_materials(),
                    "simulations_run": self._count_simulations(),
                    "assessments_passed": self._count_assessments()
                }
            # ... weitere Dimensionen
```

---

| Test File | Tests | Status |
|---|---|---|
| `test_actions.py` | 47 | ✅ all passing |
| `test_api.py` | 28 | ✅ all passing |
| `test_clim_pipeline.py` | 30 | ✅ all passing |
| `test_clim_specialization.py` | 47 | ✅ all passing |
| `test_ethics.py` | 17 | ✅ all passing |
| `test_experience.py` | 28 | ✅ all passing |
| `test_infrastructure.py` | 25 | ✅ all passing |
| `test_plugins.py` | 23 | ✅ all passing |
| `test_process_model.py` | 22 | ✅ all passing |
| `test_sandbox.py` | 31 | ✅ all passing |
| `test_service.py` | 37 | ✅ all passing |
| `test_sim_cache.py` | 18 | ✅ all passing |
| `test_topics.py` | 15 | ✅ all passing |
| Other tests | 63 | ✅ all passing |
| **Total** | **431** | **✅ all passing** |
