---
title: "Digital Twin SimGrid – Konzept für eine denkende Simulationsumgebung"
brief: "Konzept einer ‚denkenden Simulationsumgebung', in der das SimGrid als interne Bühne für Szenario-Proben, Perspektivwechsel und Konsequenz-Abwägung dient."
status: konzept
version: "1.3.0"
author: Robert Alexander Massinger
date: 2026-02-12
history:
  - date: 2026-02-12
    change: "Erstversion des Digital-Twin-SimGrid-Konzepts."
  - date: 2026-02-14
    change: "Front-Matter hinzugefügt; Status unverändert."
tags: [digital-twin, simgrid, simulation, konzept]
code-release-versions:
  - "1.3.0"
implemented-features:
  - "ComplexityRouter (3-Tier: L0/L1/L2)"
  - "ScenarioGenerator (LLM-enhanced + Template)"
  - "RoleSimulator (Multi-Stakeholder)"
  - "ConsequenceSimulator (Konsequenzbaum)"
  - "DecisionSynthesizer"
  - "ExperienceHarvester"
  - "SimulationGrid (Kern-Pipeline)"
fulfillment: "100%"
---
# Digital Twin SimGrid — Konzept für eine denkende Simulationsumgebung

**Version:** 1.0  
**Datum:** 2026-02-12  
**Status:** Konzept  
**Referenz:** [Originalkonzept (SimGrid Test Case Generation)](Concept%20To%20Generate%20Test%20Cases%20From%20SimGrid%20Output%20Data.md) | [Implementierungsanalyse](Analysis%20-%20SimGrid%20Test%20Case%20Generation%20Concept.md)

---

## Vision

> *„Bevor du in der harten Welt handelst, probe es in deiner inneren Welt."*

Das SimGrid ist die **innere Denkbühne** des EthosAI-Individuums — vergleichbar mit einem Digital Twin Grid, aber als mentaler Raum. Wie ein hochkomplexes Gehirn auf einfache Reize sofort reagiert, aber komplexe Situationen erst **intern durchspielt**, bevor es eine Antwort an die Außenwelt gibt, soll das SimGrid als **Digital Twin** der realen Situation fungieren.

### Kerngedanke

```
Alltags-Anfrage → CLIM Pipeline → SimGrid denkt nach → Beste Antwort → Außenwelt
                                         ↑
                              "Digital Twin Grid"
                          Virtuelles Durchspielen von
                      Situationen, Rollen, Konsequenzen
                         bevor man sich entscheidet
```

Das SimGrid ist **kein nachgelagertes Testtool**, sondern ein **integraler Denkschritt** — der innere Simulationsraum, in dem das Individuum:
- Szenarien durchspielt, bevor es antwortet
- Verschiedene Rollen einnimmt (Perspektivwechsel)
- Konsequenzen abwägt
- Die beste Lösung findet
- Und daraus für die Zukunft lernt

---

## Architektur-Überblick

```
┌─────────────────────────────────────────────────────────┐
│                    Anfrage (Prompt)                     │
└─────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────┐
│              Komplexitäts-Router (Phase 1)              │
│  ┌─────────┐  ┌────────────┐  ┌─────────────────┐       │
│  │ REFLEXIV │  │ DELIBERATIV│  │ SIMULATIONS-    │      │
│  │ (sofort) │  │ (abwägen)  │  │ PFLICHTIG       │      │
│  └────┬─────┘  └─────┬──────┘  └───────┬─────────┘      │
│       │              │                  │               │
└───────┼──────────────┼──────────────────┼───────────────┘
        │              │                  │
        ▼              ▼                  ▼
┌──────────┐  ┌─────────────┐  ┌──────────────────────────┐
│   CLIM   │  │    CLIM     │  │    SimGrid Digital Twin  │
│ Pipeline │  │  Pipeline   │  │                          │
│ (direkt) │  │ + Einfache  │  │  ┌──────────────────┐    │
│          │  │   SimGrid-  │  │  │  Szenarien-      │    │
│          │  │   Prüfung   │  │  │  Generator       │    │
│          │  │             │  │  └────────┬─────────┘    │
│          │  │             │  │           │              │
│          │  │             │  │  ┌────────▼─────────┐    │
│          │  │             │  │  │ Rollen-Simulator │    │
│          │  │             │  │  │(Multi-Perspektive│    │
│          │  │             │  │  └────────┬─────────┘    │
│          │  │             │  │           │              │
│          │  │             │  │  ┌────────▼─────────┐    │
│          │  │             │  │  │  Konsequenz-     │    │
│          │  │             │  │  │  Simulator       │    │
│          │  │             │  │  └────────┬─────────┘    │
│          │  │             │  │           │              │
│          │  │             │  │  ┌────────▼─────────┐    │
│          │  │             │  │  │  Entscheidungs-  │    │
│          │  │             │  │  │  Synthese        │    │
│          │  │             │  │  └──────────────────┘    │
│          │  │             │  │                          │
└────┬─────┘  └──────┬──────┘  └────────────┬─────────────┘
     │               │                      │
     └───────────────┼──────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Antwort an die „harte" Umgebung            │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│           Experience Harvester (lernt mit)              │
│  → ExperiencePacket → Test-Case-Generator → Training    │
└─────────────────────────────────────────────────────────┘
```

---

## Phase 1: Komplexitäts-Router — „Wie tief muss ich nachdenken?"

Wie ein Gehirn nicht bei jedem Reiz in tiefe Reflexion geht, braucht das System einen **Komplexitäts-Router**, der entscheidet, wie viel Simulation nötig ist.

### Drei Denktiefe-Stufen

| Stufe | Name | Auslöser | Verarbeitung | Latenz-Budget |
|-------|------|----------|--------------|---------------|
| **L0** | **Reflexiv** | Einfache Fragen, bekannte Muster, Smalltalk | CLIM Pipeline direkt → Antwort | < 2s |
| **L1** | **Deliberativ** | Mittlere Komplexität, ethische Berührpunkte, Planungsfragen | CLIM Pipeline + SimGrid-Schnellprüfung (1 Alternative) | < 10s |
| **L2** | **Simulationspflichtig** | Hohe Komplexität, Rollenkonflikte, Konsequenzenketten, Lebensplanung | Vollständige SimGrid-Simulation mit Multi-Perspektive | < 60s |

### Router-Logik (neues Modul: `ComplexityRouter`)

```python
class ThinkingDepth(Enum):
    REFLEXIVE = "L0"       # Sofort antworten
    DELIBERATIVE = "L1"    # Kurz nachdenken
    SIMULATIVE = "L2"      # Tief simulieren

class ComplexityRouter:
    """Bestimmt die nötige Denktiefe für eine Anfrage."""

    # Signale für höhere Denktiefe
    COMPLEXITY_SIGNALS = {
        "multi_stakeholder":  # Mehrere Beteiligte / Perspektiven
            r"(wir|team|familie|partner|kollegen|chef|kinder)",
        "consequence_chain":  # Langzeitfolgen
            r"(konsequenz|langfristig|zukunft|auswirkung|folge|risiko)",
        "ethical_tension":    # Ethische Spannung
            r"(soll ich|darf ich|ist es richtig|moral|gewissen|dilemma)",
        "planning":           # Vor-/Hauptplanung
            r"(plan|strategie|vorgehen|schritt|wie soll ich|entscheid)",
        "role_conflict":      # Rollenkonflikte
            r"(einerseits|andererseits|rolle|verpflichtung|loyalität)",
    }

    def assess(self, prompt: str, context: dict) -> ThinkingDepth:
        """Bestimmt die Denktiefe basierend auf Prompt + Kontext."""
        score = 0
        for signal, pattern in self.COMPLEXITY_SIGNALS.items():
            if re.search(pattern, prompt, re.IGNORECASE):
                score += 1

        # Kontext-Faktoren
        if context.get("active_planning_session"):
            score += 2
        if context.get("previous_simgrid_run"):
            score += 1  # Folgefrage zu einer Simulation

        if score == 0:
            return ThinkingDepth.REFLEXIVE
        elif score <= 2:
            return ThinkingDepth.DELIBERATIVE
        else:
            return ThinkingDepth.SIMULATIVE
```

---

## Phase 2: SimGrid Digital Twin — Die innere Denkbühne

### 2.1 Szenarien-Generator

Erzeugt aus der Anfrage **mehrere mögliche Handlungsvarianten**:

```python
class ScenarioGenerator:
    """Generiert Handlungsvarianten aus einem Prompt."""

    def generate(self, prompt: str, context: dict, depth: ThinkingDepth
                 ) -> list[SimulationScenario]:
        """
        L1 (Deliberativ):  1 Hauptszenario + 1 Alternative
        L2 (Simulativ):    1 Hauptszenario + 3-5 Alternativen + 1 Counterfactual
        """
        scenarios = []

        # Hauptszenario: Was der Nutzer vorhat
        scenarios.append(SimulationScenario(
            description=prompt,
            variant_type="primary",
            role_perspective="self",
        ))

        if depth >= ThinkingDepth.DELIBERATIVE:
            # Eine Alternative generieren lassen
            alt = self._generate_alternative(prompt, context)
            scenarios.append(alt)

        if depth >= ThinkingDepth.SIMULATIVE:
            # Weitere Alternativen + Perspektiven
            scenarios.extend(self._generate_alternatives(prompt, context, n=3))
            scenarios.append(self._generate_counterfactual(prompt, context))

        return scenarios
```

### 2.2 Rollen-Simulator — Perspektivwechsel

Das Kernstück des Digital Twin: **Andere Rollen einnehmen**, um eine Situation aus verschiedenen Blickwinkeln zu bewerten. Wie ein Mensch, der sich fragt: *„Wie würde das aus Sicht meines Chefs aussehen? Meiner Familie? Meiner Zukunft?"*

```python
class RoleSimulator:
    """Simuliert eine Situation aus verschiedenen Perspektiven."""

    # Standard-Rollen-Set (erweiterbar durch Kontext)
    DEFAULT_ROLES = [
        Role("self",        "Ich selbst",       "Eigene Werte, Ziele, Bedürfnisse"),
        Role("counterpart", "Gegenüber",        "Perspektive des Gesprächspartners"),
        Role("observer",    "Neutraler Beobachter", "Objektive Außenansicht"),
        Role("future_self", "Mein Zukunfts-Ich", "Langzeitperspektive"),
        Role("ethicist",    "Ethiker",          "Moralische Bewertung"),
    ]

    def simulate_perspectives(self, scenario: SimulationScenario,
                               roles: list[Role] = None
                               ) -> list[PerspectiveResult]:
        """
        Durchspielt ein Szenario aus jeder Rolle:
        - Jede Rolle bewertet das Szenario durch die CLIM-Pipeline
        - Der Prompt wird mit dem Rollen-Kontext angereichert
        - Ergebnis: Multi-Perspektiven-Bewertung
        """
        results = []
        for role in (roles or self.DEFAULT_ROLES):
            # Prompt mit Rollenkontext anreichern
            role_prompt = self._build_role_prompt(scenario, role)
            # Durch CLIM-Pipeline schicken (intern, nicht nach außen)
            clim_result = self.clim_stack.process("simulate", role_prompt)
            results.append(PerspectiveResult(
                role=role,
                decision=clim_result.decision,
                reasoning=clim_result.reasoning,
                confidence=clim_result.confidence,
                ethic_value=clim_result.ethic_value,
            ))
        return results
```

### 2.3 Konsequenz-Simulator

Spielt **Ursache-Wirkungs-Ketten** durch — nicht nur den nächsten Schritt, sondern 2-3 Schritte voraus:

```python
class ConsequenceSimulator:
    """Simuliert Konsequenzketten: Was passiert, wenn...?"""

    def simulate_consequences(self, scenario: SimulationScenario,
                               perspectives: list[PerspectiveResult],
                               depth: int = 3
                               ) -> ConsequenceTree:
        """
        Für jedes Szenario: Simuliere die Kette von Folgen.
        depth=1: Unmittelbare Folge
        depth=2: Folgen der Folge
        depth=3: Langzeiteffekte

        Ergebnis: Baumstruktur der Konsequenzen mit Wahrscheinlichkeiten.
        """
        tree = ConsequenceTree(root=scenario)

        for d in range(depth):
            for leaf in tree.get_leaves():
                # "Was passiert als Nächstes?"
                next_consequences = self._project_next_step(
                    leaf, perspectives
                )
                for consequence in next_consequences:
                    tree.add_child(leaf, consequence)

        return tree
```

### 2.4 Entscheidungs-Synthese

Aggregiert alle Simulationsergebnisse zu einer **optimalen Antwort**:

```python
class DecisionSynthesizer:
    """Synthetisiert die beste Antwort aus allen Simulationsergebnissen."""

    def synthesize(self, scenarios: list[SimulationScenario],
                   perspectives: dict[str, list[PerspectiveResult]],
                   consequences: dict[str, ConsequenceTree],
                   ) -> SynthesizedDecision:
        """
        Gewichtete Synthese:
        1. Ethik-Score über alle Perspektiven (höchste Priorität)
        2. Konsens-Score: Wie viele Perspektiven stimmen überein?
        3. Konsequenz-Score: Langfristig positive Auswirkungen
        4. Individuum-Score: Passt zur Persönlichkeit des Individuums

        Ergebnis: Beste Handlungsoption + Begründung mit Perspektiven
        """
        rankings = []
        for scenario in scenarios:
            persp = perspectives[scenario.id]
            cons = consequences[scenario.id]

            ethic_score = self._compute_ethic_consensus(persp)
            consensus = self._compute_role_consensus(persp)
            consequence_score = self._evaluate_consequence_tree(cons)
            individual_fit = self._compute_individual_alignment(persp)

            rankings.append(ScenarioRanking(
                scenario=scenario,
                ethic_score=ethic_score,
                consensus_score=consensus,
                consequence_score=consequence_score,
                individual_score=individual_fit,
                total=self._weighted_total(
                    ethic_score, consensus, consequence_score, individual_fit
                ),
            ))

        rankings.sort(key=lambda r: r.total, reverse=True)
        best = rankings[0]

        return SynthesizedDecision(
            chosen_scenario=best.scenario,
            decision=best.scenario.decision,
            reasoning=self._build_multi_perspective_reasoning(best, rankings),
            confidence=best.total,
            alternatives=rankings[1:],
            simulation_trace=self._build_trace(rankings),
        )
```

---

## Phase 3: Experience Harvester — Aus dem Denken lernen

Jede SimGrid-Simulation ist eine **Lerngelegenheit**. Der Experience Harvester sammelt automatisch die Ergebnisse und generiert daraus Trainingsdaten.

### 3.1 Automatische Erfahrungserfassung

```python
class ExperienceHarvester:
    """Erntet Lernerfahrungen aus SimGrid-Durchläufen."""

    def harvest(self, simulation_run: SimGridRun) -> ExperiencePacket:
        """
        Konvertiert einen SimGrid-Durchlauf in ein ExperiencePacket:
        - Szenario + alle Varianten
        - Perspektiven-Ergebnisse pro Rolle
        - Konsequenzketten
        - Gewählte Entscheidung + Begründung
        - Optional: Späteres Feedback (war die Entscheidung gut?)
        """
        packet = ExperiencePacket(
            scenario=simulation_run.original_prompt,
            pipeline_type=simulation_run.pipeline_type,
            final_decision=simulation_run.decision.name,
        )

        # Layer-Traces aus der Simulation
        for perspective in simulation_run.perspectives:
            packet.add_layer_trace(LayerTrace(
                layer_name=perspective.role.name,
                phase="simulate",
                decision=perspective.decision,
                confidence=perspective.confidence,
                response=perspective.reasoning,
            ))

        return packet
```

### 3.2 Automatische Test-Case-Generierung

```python
class TestCaseGenerator:
    """Generiert Trainingsdaten aus gesammelten ExperiencePackets."""

    def generate_from_experiences(self,
                                  packets: list[ExperiencePacket],
                                  min_confidence: float = 0.7
                                  ) -> list[TrainingTestCase]:
        """
        Konvertiert ExperiencePackets in Trainings-Test-Cases:
        1. Filtere nach Confidence (nur gesicherte Entscheidungen)
        2. Clustere ähnliche Szenarien
        3. Extrahiere dominante Entscheidungsmuster
        4. Generiere Test-Cases im Format für TrainListGenerator
        """
        # Schritt 1: Filtern
        confident = [p for p in packets if p.overall_confidence >= min_confidence]

        # Schritt 2: Clustern (Embedding-basierte Ähnlichkeit)
        clusters = self._cluster_by_embedding(confident)

        # Schritt 3: Pattern-Extraktion
        test_cases = []
        for cluster in clusters:
            dominant_decision = self._extract_dominant_pattern(cluster)
            representative = self._select_representative(cluster)

            # Schritt 4: Test-Case erzeugen
            for layer in ["ETHIC", "INDIVIDUAL", "SAMT", "LTCLIM"]:
                test_cases.append(TrainingTestCase(
                    scenario=representative.scenario,
                    pipeline="Standard Pipeline",
                    layer=layer,
                    input=representative.scenario,
                    output=dominant_decision.reasoning,
                    decision=dominant_decision.per_layer.get(layer, "GO"),
                    analysis=f"Auto-generated from {len(cluster)} experiences",
                    source="simgrid_harvester",
                    confidence=dominant_decision.confidence,
                ))

        return test_cases
```

---

## Phase 4: Integration in den bestehenden Query-Flow

### Vorher (aktuell)

```
Prompt → process_prompt() → 4 CLIM Layers → Antwort
```

### Nachher (mit Digital Twin SimGrid)

```
Prompt → ComplexityRouter.assess()
           │
           ├─ L0 (Reflexiv)  → process_prompt()          → Antwort
           │
           ├─ L1 (Deliberativ) → process_prompt()
           │                    → SimGrid.quick_check()   → Antwort
           │
           └─ L2 (Simulativ) → SimGrid.full_simulation()
                                ├─ ScenarioGenerator
                                ├─ RoleSimulator
                                ├─ ConsequenceSimulator
                                └─ DecisionSynthesizer   → Antwort
                                         │
                                         ▼
                              ExperienceHarvester → Training
```

### Konkreter Eingriffspunkt: `service.py`

```python
async def process_prompt(self, prompt, context=None, layer=None):
    # NEU: Komplexitäts-Routing
    depth = self._complexity_router.assess(prompt, self._build_context())

    if depth == ThinkingDepth.REFLEXIVE:
        # Wie bisher: direkte CLIM-Pipeline
        return await self._process_prompt_direct(prompt, context, layer)

    elif depth == ThinkingDepth.DELIBERATIVE:
        # CLIM-Pipeline + schnelle SimGrid-Prüfung
        clim_result = await self._process_prompt_direct(prompt, context, layer)
        sim_check = await self._simgrid.quick_check(prompt, clim_result)
        if sim_check.confirms(clim_result):
            return clim_result  # Bestätigt → sofort raus
        else:
            return self._merge_with_simulation(clim_result, sim_check)

    else:  # SIMULATIVE
        # Volle SimGrid-Simulation
        sim_result = await self._simgrid.full_simulation(prompt, context)
        self._experience_harvester.harvest(sim_result)
        return sim_result.to_prompt_response()
```

---

## Datenmodell

### Neue Datenstrukturen

```python
@dataclass
class SimulationScenario:
    id: str                      # UUID
    description: str             # Situationsbeschreibung
    variant_type: str            # "primary", "alternative", "counterfactual"
    role_perspective: str        # Aus welcher Rolle betrachtet
    parameters: dict             # Kontextparameter
    parent_scenario_id: str      # Verweis auf Elternszenario (bei Varianten)

@dataclass
class Role:
    id: str                      # z.B. "self", "counterpart", "future_self"
    name: str                    # Anzeigename
    description: str             # Rollenbeschreibung
    system_prompt: str           # Prompt-Prefix für diese Rolle

@dataclass
class PerspectiveResult:
    role: Role
    scenario_id: str
    decision: str                # GO / STOP / REVIEW / ADJUST / ...
    reasoning: str               # Begründung aus dieser Perspektive
    confidence: float            # 0.0 - 1.0
    ethic_value: float           # Ethik-Score
    layer_decisions: dict        # {layer: decision} pro CLIM-Layer

@dataclass
class ConsequenceNode:
    step: int                    # 1 = sofort, 2 = mittel, 3 = lang
    description: str             # Was passiert
    probability: float           # Wahrscheinlichkeit (0-1)
    ethic_impact: float          # Ethische Bewertung (-10 bis +10)
    children: list               # Folge-Konsequenzen

@dataclass
class SynthesizedDecision:
    chosen_scenario: SimulationScenario
    decision: str
    reasoning: str               # Multi-Perspektiven-Begründung
    confidence: float
    ethic_score: float
    alternatives: list           # Verworfene Alternativen + Gründe
    simulation_trace: dict       # Vollständiger Simulations-Verlauf
    thinking_depth: ThinkingDepth
    thinking_time_ms: float
```

---

## Implementierungsplan

### Sprint 1: Fundament (Complexity Router + SimGrid Core)

| # | Task | Datei(en) | Aufwand |
|---|------|-----------|---------|
| 1.1 | `ThinkingDepth` Enum + `ComplexityRouter` | `simulation/complexity_router` module | S |
| 1.2 | `SimulationScenario` + `Role` Datenmodelle | `simulation/models` module | S |
| 1.3 | Integration in `service.py::process_prompt()` | `api/service` module | M |
| 1.4 | UI: Denktiefe-Indikator im Chat | `training.js` / `chat.js` | S |

### Sprint 2: Rollen-Simulation

| # | Task | Datei(en) | Aufwand |
|---|------|-----------|---------|
| 2.1 | `RoleSimulator` — Perspektivwechsel-Engine | `simulation/role_simulator` module | L |
| 2.2 | Standard-Rollen-Set (5 Basis-Rollen) | `simulation/roles` module | S |
| 2.3 | Rollen-Prompt-Templates | Simulation Prompts Package | M |
| 2.4 | `ScenarioGenerator` — Varianten-Erzeugung | `simulation/scenario_generator` module | M |

### Sprint 3: Konsequenz-Simulation

| # | Task | Datei(en) | Aufwand |
|---|------|-----------|---------|
| 3.1 | `ConsequenceSimulator` — Kausalketten | `simulation/consequence_simulator` module | L |
| 3.2 | `ConsequenceTree` Datenstruktur | `simulation/models` module | S |
| 3.3 | `DecisionSynthesizer` — Gewichtete Synthese | `simulation/decision_synthesizer` module | L |

### Sprint 4: Experience Harvesting + Auto-Training

| # | Task | Datei(en) | Aufwand |
|---|------|-----------|---------|
| 4.1 | `ExperienceHarvester` — Erfahrungen sammeln | `simulation/experience_harvester` module | M |
| 4.2 | `TestCaseGenerator` — Auto-Generierung | `simulation/test_case_generator` module | L |
| 4.3 | Embedding-basiertes Clustering | `simulation/clustering` module | M |
| 4.4 | Integration in Training-Pipeline | `clim/train_list_generator` module | M |

### Sprint 5: UI + API

| # | Task | Datei(en) | Aufwand |
|---|------|-----------|---------|
| 5.1 | API-Endpunkte für SimGrid-Steuerung | `api/app` module | M |
| 5.2 | Chat-UI: Denk-Indikator (L0/L1/L2) | Frontend Chat Component | M |
| 5.3 | SimGrid-Dashboard: Visualisierung | Frontend SimGrid Component | L |
| 5.4 | Perspektiven-Ansicht im Chat | Frontend Chat Component | M |

---

## Bestehende Infrastruktur die wiederverwendet wird

| Bestand | Wird zu | Anpassung nötig |
|---------|---------|-----------------|
| `SimulationsGrid` | `SimGridEngine` (Kern des Digital Twin) | Refactoring: Von Standalone zu integriertem Denkschritt |
| `SimulationTopic` | `SimulationScenario` (erweitert) | `to_dict()`, Rollen-Felder, Varianten-Typ |
| `SimulationCache` | Bleibt (TTL-Cache für wiederholte Szenarien) | Keine Änderung |
| `ExperiencePacket` + `ExperienceStore` | Harvester-Output-Format | Rollen-Traces ergänzen |
| `CLIMStack.process()` | Wird intern vom RoleSimulator aufgerufen | Neuer `type="simulate"` Modus |
| `Decision` Enum | Bleibt | Keine Änderung |
| `Pipelines` | Bleibt (Standard + Emergency) | Keine Änderung |
| `TrainListGenerator` | Akzeptiert auch auto-generierte Cases | `source`-Feld hinzufügen |

---

## Analogie: Wie ein Gehirn denkt

```
Reiz (Prompt)
    │
    ├─ Amygdala-Schnellcheck (L0: Reflexiv)
    │   "Kenne ich das? → Sofort reagieren"
    │
    ├─ Präfrontaler Cortex (L1: Deliberativ)
    │   "Hmm, das muss ich kurz abwägen"
    │
    └─ Mentale Simulation (L2: Simulativ)
        "Das muss ich erst durchspielen..."
        ├─ Perspektivübernahme (Theory of Mind)
        ├─ Zukunftsprojektion (Episodisches Gedächtnis)
        ├─ Konsequenzabschätzung (Dorsaler Pfad)
        └─ Handlungsentscheidung (Exekutive Funktion)
```

Das SimGrid-Konzept bildet diesen neurokognitiven Prozess technisch ab:

| Gehirn | EthosAI SimGrid |
|--------|-----------------|
| Amygdala (reflexiv) | `ComplexityRouter` → L0 |
| Arbeitsgedächtnis (deliberativ) | `ComplexityRouter` → L1 + Quick-Check |
| Mentale Simulation | `SimGridEngine` → L2 |
| Theory of Mind (Perspektivwechsel) | `RoleSimulator` |
| Episodisches Zukunft-Denken | `ConsequenceSimulator` |
| Exekutive Entscheidung | `DecisionSynthesizer` |
| Hippocampus (Lernen) | `ExperienceHarvester` + `TestCaseGenerator` |

---

## Offene Designentscheidungen

1. **VRAM-Budget für L2-Simulation**: Vollständige Multi-Perspektiven-Simulation mit 5 Rollen × 4 Szenarien = 20 CLIM-Durchläufe. Bei ~2s/Durchlauf = ~40s. Akzeptabel für L2, aber GPU-Management nötig.

2. **Rollen-Set dynamisch vs. statisch**: Sollen Rollen aus dem Kontext abgeleitet werden (z.B. bei „Mein Chef will..." → Rolle „Chef" dynamisch erzeugen)?

3. **Feedback-Loop-Zeitraum**: Wie lange warten wir auf reales Feedback (z.B. „Dein Rat war gut/schlecht"), bevor ein ExperiencePacket als „confirmed" gilt?

4. **Parallelisierung**: Sollen Rollen-Simulationen parallel laufen (schneller, mehr VRAM) oder sequentiell (langsamer, weniger VRAM)?

---

## Zusammenfassung

Dieses Konzept erweitert das Originalkonzept (SimGrid → Test Cases) um die **zentrale Erkenntnis**: Das SimGrid ist kein nachgelagertes Testtool, sondern ein **integraler Bestandteil des Denkprozesses**. 

Es implementiert:
- **Alles was im Originalkonzept fehlte** (Schritte 1-6: Analyse, Clustering, Pattern-Extraktion, Varianten-Generierung, automatisierte Test-Case-Erzeugung)
- **Digital Twin Ansatz** mit Multi-Perspektiven-Simulation
- **Drei Denktiefe-Stufen** für effiziente Ressourcennutzung
- **Selbst-lernendes System** das aus seinen eigenen Simulationen Trainingsdaten generiert

> *Das SimGrid ist der innere Raum, in dem das EthosAI-Individuum denkt, bevor es handelt — genau wie ein Mensch, der komplexe Entscheidungen erst im Kopf durchspielt.*
