---
title: "Internet Research Toolbox — Konzept"
brief: "Anbieter-neutrales Konzept für eine EthosAI-Toolbox zur Internet-Recherche mit Bias-Transparenz, EU-Konformität und dreistufigem Sicherheitsmodell."
status: draft
version: "0.1.0"
author: Robert Alexander Massinger
date: 2026-03-07
tags: [toolbox, internet-research, bias, eu, searx, metager, fact-check, dsgvo, osint, sovereignty]
target-release: "v1.64.0"
history:
  - date: 2026-03-07
    change: "Erstversion — Architektur-Entwurf, Bias-Modell, Anbieter-Anforderungen, Security-Tier-Integration."
---

# Internet Research Toolbox — Konzept

## 1. Vision

EthosAI soll eigenständig im Internet recherchieren können — wie ein
sorgfältiger Journalist, nicht wie ein Suchmaschinen-Nutzer.

**Kernprinzip:** Jede Recherche liefert nicht nur Ergebnisse, sondern auch
eine **Bias-Analyse** — welche Verzerrungen vorliegen, woher sie kommen,
wie stark sie wirken und in welche Richtung sie ziehen.

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                    EthosAI Internet Research                    │
  │                                                                 │
  │  ┌──────────┐   ┌──────────────┐   ┌────────────────────────┐   │
  │  │ Research │──>│ Multi-Source │──>│ Bias Analysis &        │   │
  │  │ Request  │   │ Aggregation  │   │ Credibility Assessment │   │
  │  └──────────┘   └──────┬───────┘   └────────────┬───────────┘   │
  │                        │                        │               │
  │         ┌──────────────┼────────────────────────┘               │
  │         ▼              ▼                                        │
  │  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
  │  │ Fact-Check  │  │ Cross-Ref    │  │ Structured Report      │  │
  │  │ Verification│  │ Validation   │  │ (Findings + Bias Map)  │  │
  │  └─────────────┘  └──────────────┘  └────────────────────────┘  │
  └─────────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │      Search Provider Abstraction      │
          │                                       │
          │  ┌────────┐ ┌────────┐ ┌────────────┐ │
          │  │Provider│ │Provider│ │ Provider N │ │
          │  │  A     │ │  B     │ │            │ │
          │  └────────┘ └────────┘ └────────────┘ │
          └───────────────────────────────────────┘
```

---

## 2. Grundsätze

### 2.1 Anbieter-Neutralität

Die Toolbox ist **nicht an einen bestimmten Suchdienst gebunden**. Stattdessen
definiert sie ein **Provider-Interface**, das jeder Suchdienst implementieren
kann. Die Auswahl des konkreten Providers erfolgt über Konfiguration, nicht
über Code.

```python
class SearchProvider(Protocol):
    """Abstraktes Interface für jeden Suchdienst."""

    @property
    def provider_id(self) -> str: ...

    @property
    def jurisdiction(self) -> str: ...
        """ISO 3166-1 alpha-2 oder 'EU', 'FOSS'."""

    @property
    def data_processing_location(self) -> str: ...
        """Wo werden Suchanfragen verarbeitet?"""

    async def search(
        self,
        query: str,
        *,
        language: str = "de",
        region: str = "EU",
        max_results: int = 20,
        categories: list[str] | None = None,
    ) -> list[SearchResult]: ...

    async def health_check(self) -> ProviderHealth: ...
```

### 2.2 EU-Konformität — Unverzichtbare Anforderungen

| # | Anforderung | Begründung |
|---|------------|------------|
| EU-1 | **Quellcode offen oder auditierbar** | Kein Black-Box-Ranking. FOSS (AGPL/MIT/Apache) bevorzugt; proprietär nur wenn Code-Audit durch EU-Stelle vorliegt |
| EU-2 | **Datenverarbeitung innerhalb EU/EWR** | DSGVO Art. 44–49 — kein Datentransfer an Drittstaaten ohne Angemessenheitsbeschluss |
| EU-3 | **Keine Kontrolle durch Nicht-EU-Entitäten** | Kein US-amerikanisches Mutterunternehmen, kein CLOUD Act, kein FISA 702 Exposure |
| EU-4 | **Kein Nutzer-Tracking / kein Profiling** | DSGVO Art. 22 — keine automatisierte Einzelentscheidung basierend auf Profiling |
| EU-5 | **Kein kommerzielles Ranking-Bias** | Suchergebnisse dürfen nicht durch bezahlte Platzierung verzerrt werden |
| EU-6 | **Self-Hosting möglich** | Für Security-Tier 2+3 (Behörden, Forschung, Verteidigung) muss On-Premise-Betrieb möglich sein |
| EU-7 | **API mit strukturierter Ausgabe** | JSON/XML-Response, keine HTML-Scraping-Abhängigkeit |
| EU-8 | **Audit-Trail** | Jede Suchanfrage muss protokollierbar sein (wer, wann, was, wohin) |

### 2.3 Bias-Transparenz — Das Kernversprechen

EthosAI liefert keine „neutralen" Suchergebnisse — denn **neutrale
Suchergebnisse existieren nicht**. Stattdessen liefert EthosAI **Ergebnisse
mit Bias-Annotation**.

Jedes Rechercheergebnis wird mit folgender Bias-Struktur versehen:

```python
@dataclass
class BiasAnnotation:
    """Bias-Bewertung für ein einzelnes Suchergebnis."""

    # Welche Art von Bias liegt vor?
    bias_kind: BiasKind
    # Enum: POLITICAL_LEFT | POLITICAL_RIGHT | COMMERCIAL |
    #        STATE_PROPAGANDA | IDEOLOGICAL | SCIENTIFIC_MAINSTREAM |
    #        SCIENTIFIC_CONTRARIAN | SELECTION | ALGORITHMIC |
    #        CULTURAL | GEOGRAPHIC | TEMPORAL | NONE_DETECTED

    # Was ist die Motivation hinter dem Bias?
    motivation: str
    # z.B. "Werbefinanziertes Geschäftsmodell bevorzugt Anbieter mit
    #        höherem Ad-Spend im Ranking"

    # Welche Wirkung hat der Bias auf den Leser?
    effect: str
    # z.B. "Leser hält kommerzielles Produkt für objektiv bestes Ergebnis"

    # Wie stark ist der Bias? (0.0 = nicht vorhanden, 1.0 = extrem)
    amplitude: float
    # 0.0–0.2 = minimal, 0.2–0.5 = moderat, 0.5–0.8 = stark, 0.8–1.0 = extrem

    # Woher stammt diese Einschätzung?
    assessment_source: str
    # z.B. "EDMO factcheck database", "MediaBiasFactCheck", "EthosAI heuristic"

    # Konfidenz der Bias-Bewertung (0.0–1.0)
    confidence: float
```

---

## 3. Architektur

### 3.1 Schichtenmodell

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: Research Report Generator                         │
│  → Aggregierter Recherchebericht mit Bias-Map,              │
│    Quellen-Ranking, Faktencheck-Status, Empfehlung          │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Analyse-Engine                                    │
│  → Bias-Detektion, Cross-Referencing, Faktencheck,          │
│    Quellen-Glaubwürdigkeit, Duplikat-Erkennung              │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Aggregation & Normalisierung                      │
│  → Ergebnisse aus N Providern zusammenführen, deduplizieren,│
│    in einheitliches Schema überführen                       │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Provider Abstraction                              │
│  → SearchProvider Interface, Health-Check, Failover,        │
│    Rate-Limiting, Jurisdiktions-Filter                      │
├─────────────────────────────────────────────────────────────┤
│  Layer 0: Security & Compliance                             │
│  → Tier-basierte Routing-Policy, Audit-Trail,               │
│    Encryption-at-Rest/in-Transit, Proxy-Konfiguration       │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Provider-Anforderungsprofil

Für die Auswahl konkreter Suchdienste gelten folgende **funktionale
Anforderungen**, nicht-Anbieter-spezifisch formuliert:

| Kategorie | Anforderung | Pflicht? |
|-----------|------------|----------|
| **Meta-Suche** | Aggregation über ≥5 unabhängige Quellen | ✅ Tier 1–3 |
| **Eigener Index** | Eigener Web-Crawler ohne Drittanbieter-Abhängigkeit | Optional (Tier 3 empfohlen) |
| **Self-Hosting** | Docker/OCI-Image, On-Premise deploybar | ✅ Tier 2–3 |
| **FOSS-Lizenz** | AGPL-3.0, MIT, Apache-2.0 oder vergleichbar | ✅ Tier 2–3, empfohlen Tier 1 |
| **API** | REST/JSON nativ (kein HTML-Scraping) | ✅ Tier 1–3 |
| **Kein Tracking** | Keine Cookies, kein Fingerprinting, kein Session-Tracking | ✅ Tier 1–3 |
| **Quell-Transparenz** | Engine-Attribution (welches Ergebnis kommt von welcher Quelle) | ✅ Tier 1–3 |
| **DSGVO Art. 25** | Privacy by Design & Default | ✅ Tier 1–3 |
| **Ergebnisfreiheit** | Kein bezahltes Ranking, keine Werbe-Insertion | ✅ Tier 1–3 |
| **Tor/Proxy-kompatibel** | Suchanfragen über Tor/VPN/Proxy routbar | Optional Tier 1, ✅ Tier 3 |

### 3.3 Datenmodell

```python
@dataclass
class SearchResult:
    """Normalisiertes Suchergebnis aus beliebigem Provider."""
    url: str
    title: str
    snippet: str
    source_engine: str          # z.B. "duckduckgo", "bing", "google", "wikipedia"
    source_provider: str        # z.B. "searxng-instance-1", "metager"
    timestamp: datetime
    language: str
    relevance_score: float      # Provider-normalisiert 0.0–1.0
    metadata: dict[str, Any]    # Provider-spezifische Zusatzdaten

@dataclass
class AnnotatedResult:
    """Suchergebnis mit Bias- und Glaubwürdigkeits-Annotation."""
    result: SearchResult
    bias: BiasAnnotation
    credibility: CredibilityScore
    fact_check: FactCheckResult | None
    cross_references: list[str]  # URLs anderer Quellen die dasselbe bestätigen

@dataclass
class CredibilityScore:
    """Glaubwürdigkeits-Bewertung einer Quelle."""
    domain: str
    overall_score: float        # 0.0–1.0
    factual_accuracy: float     # 0.0–1.0
    editorial_independence: float  # 0.0–1.0
    transparency: float         # 0.0–1.0
    source_db: str              # Woher stammt das Rating?
    last_updated: date

@dataclass
class FactCheckResult:
    """Ergebnis einer Faktenprüfung."""
    claim: str
    verdict: FactVerdict         # TRUE | FALSE | MIXED | UNVERIFIED | SATIRE
    source: str                  # z.B. "EUvsDisinfo", "EDMO", "correctiv.org"
    url: str
    date: date

@dataclass
class ResearchReport:
    """Vollständiger Recherchebericht."""
    query: str
    timestamp: datetime
    security_tier: SecurityTier
    providers_used: list[str]
    total_results: int
    annotated_results: list[AnnotatedResult]
    bias_summary: BiasSummary
    consensus_assessment: str    # Zusammenfassung: worüber herrscht Konsens?
    controversy_map: list[str]   # Punkte, bei denen Quellen widersprechen
    recommendations: list[str]   # Empfehlungen für Weiterrecherche
    audit_id: str                # UUID für Audit-Trail
```

---

## 4. Werkzeuge (Tools)

Die Toolbox stellt folgende 8 Werkzeuge bereit:

### 4.1 `web_search` — Internet-Suche

Führt eine Suche über den konfigurierten Provider durch.

- **Input:** Query, Sprache, Region, Kategorie (general/news/science/images)
- **Output:** Liste normalisierter `SearchResult`-Objekte
- **Multi-Provider:** Parallel-Query an N konfigurierte Provider
- **Deduplizierung:** URL-basiertes Merging mit Quellenattribution
- **Security:** Anfrage wird gemäß aktivem Security-Tier geroutet

### 4.2 `source_bias_analyzer` — Quellen-Bias-Analyse

Bewertet den Bias einer Quelle (Domain/Artikel).

- **Input:** URL oder Domain
- **Output:** `BiasAnnotation` mit kind, motivation, effect, amplitude
- **Methoden:**
  - Lookup in konfigurierbaren Bias-Datenbanken (z.B. EDMO, EUvsDisinfo)
  - Heuristik: Domänen-Ownership, Finanzierungsmodell, Impressum-Analyse
  - Vergleich: Wie berichtet diese Quelle verglichen mit Konsens aller Quellen?
- **Transparenz:** Jede Bewertung enthält `assessment_source` und `confidence`

### 4.3 `cross_reference` — Quellen-Quervergleich

Vergleicht Aussagen über mehrere unabhängige Quellen hinweg.

- **Input:** Claim (Aussage) + Liste von Suchergebnissen
- **Output:** Bestätigungs-/Widerspruchsmatrix
- **Methoden:**
  - Textuelle Ähnlichkeit der Kernaussage über Quellen
  - Zeitliche Einordnung (Erstveröffentlichung vs. Nachdruck)
  - Quellendiversität-Score (N unabhängige Quellen mit gleicher Aussage)

### 4.4 `fact_check` — Faktenprüfung

Prüft eine Behauptung gegen bekannte Faktencheck-Datenbanken.

- **Input:** Claim (Aussage als Text)
- **Output:** `FactCheckResult` mit Verdict + Quelle
- **Datenbanken (konfigurierbar, keine Vendor-Lock-in):**
  - EU-basierte Faktencheck-Netzwerke (EDMO-Mitglieder)
  - EUvsDisinfo (EU East StratCom Task Force)
  - Nationale Agenturen (z.B. correctiv.org, AFP Faktencheck)
  - Wissenschaftliche Datenbanken (PubMed, Cochrane für med. Claims)

### 4.5 `source_credibility` — Quellen-Glaubwürdigkeits-Score

Bewertet die Gesamtglaubwürdigkeit einer Quelle/Domain.

- **Input:** Domain oder URL
- **Output:** `CredibilityScore` mit mehreren Dimensionen
- **Dimensionen:**
  - Faktentreue (historische Genauigkeit)
  - Redaktionelle Unabhängigkeit (Eigentümerstruktur)
  - Transparenz (Impressum, Finanzierung, Korrekturen)
- **Quellen:** Konfigurierbare Datenbanken, kein hardcodierter Anbieter

### 4.6 `research_report` — Aggregierter Recherchebericht

Führt eine vollständige Recherche durch und erstellt einen strukturierten
Bericht.

- **Input:** Forschungsfrage + Tiefe (quick/standard/deep)
- **Output:** `ResearchReport` als Markdown mit:
  - Konsens-Zusammenfassung
  - Kontroversen-Karte (wo widersprechen sich Quellen?)
  - Bias-Übersicht (Gesamtbias aller verwendeten Quellen)
  - Quellenverzeichnis mit Glaubwürdigkeits-Scores
  - Empfehlungen für Weiterrecherche
- **Reproduzierbarkeit:** Jeder Report hat eine `audit_id` für Nachvollziehbarkeit

### 4.7 `domain_intel` — Domain-Hintergrundrecherche

Recherchiert Hintergrundinformationen über eine Domain/Organisation.

- **Input:** Domain-Name
- **Output:** Strukturierte Domain-Intelligence:
  - WHOIS-Daten (Registrar, Registrierungsdatum, Land)
  - Impressum-Extraktion (falls EU-Domain)
  - Eigentümerstruktur (soweit öffentlich)
  - Finanzierungsmodell (Werbung, Spenden, Staat, Paywall)
  - Zugehörigkeit zu Mediennetzwerken
  - Historische Auffälligkeiten (Factcheck-Hits, Desinformation-Flags)

### 4.8 `audit_trail` — Recherche-Protokollierung

Verwaltet den unveränderlichen Audit-Trail aller Suchanfragen.

- **Input:** Query (Abfrage-ID oder Zeitraum)
- **Output:** Audit-Protokoll mit:
  - Zeitstempel, Benutzer, Security-Tier
  - Verwendete Provider
  - Suchanfrage (ggf. pseudonymisiert bei Tier 3)
  - Ergebnis-Hashes (ohne Inhalt, nur Referenz)
- **Compliance:** DSGVO-konform, löschbar (Art. 17), exportierbar (Art. 20)

---

## 5. Bias-Modell — Theorie & Praxis

### 5.1 Bias-Taxonomie

EthosAI verwendet eine transparente, dokumentierte Bias-Taxonomie:

| Bias-Art | Code | Beschreibung | Typische Motivation | Typische Wirkung |
|----------|------|-------------|---------------------|------------------|
| **Politisch Links** | `POLITICAL_LEFT` | Bevorzugung progressiver/sozialer Positionen | Redaktionelle Linie | Leser übernimmt Framing |
| **Politisch Rechts** | `POLITICAL_RIGHT` | Bevorzugung konservativer/nationaler Positionen | Redaktionelle Linie | Leser übernimmt Framing |
| **Kommerziell** | `COMMERCIAL` | Werbefinanzierte Platzierung oder Affiliate-Links | Umsatz | Produkt erscheint als „beste Wahl" |
| **Staatspropaganda** | `STATE_PROPAGANDA` | Staatlich kontrollierte/finanzierte Narrative | Machterhalt | Leser vertraut staatlicher Darstellung |
| **Ideologisch** | `IDEOLOGICAL` | Weltanschaulich motivierte Darstellung | Überzeugung | Bestätigungsverzerrung wird verstärkt |
| **Wiss. Mainstream** | `SCIENTIFIC_MAINSTREAM` | Konsens-Position der Fachwelt | Peer-Review-Kultur | Außenseiter-Positionen werden ignoriert |
| **Wiss. Konträr** | `SCIENTIFIC_CONTRARIAN` | Gezielt gegen Konsens positioniert | Aufmerksamkeit, Ideologie | Zweifel an etabliertem Wissen |
| **Selektion** | `SELECTION` | Auswahl-Bias durch Such-Algorithmus | Algorithmus-Design | Bestimmte Perspektiven fehlen |
| **Algorithmisch** | `ALGORITHMIC` | Ranking-Verzerrung durch Suchmaschine | Engagement-Optimierung | Sensationelles wird bevorzugt |
| **Kulturell** | `CULTURAL` | Kulturspezifische Perspektive dominiert | Autoren-Hintergrund | Andere Kulturen werden nicht abgebildet |
| **Geografisch** | `GEOGRAPHIC` | Regionale Ergebnisfilterung | Lokalisierung | Globale Perspektive fehlt |
| **Temporal** | `TEMPORAL` | Aktualitäts-Bias (neuere = relevanter) | Freshness-Algorithmus | Historischer Kontext geht verloren |
| **Keiner erkannt** | `NONE_DETECTED` | Kein erkennbarer Bias | — | — |

### 5.2 Bias-Amplitude-Skala

```
0.0 ──────── 0.2 ──────── 0.5 ──────── 0.8 ──────── 1.0
│  Minimal   │  Moderat    │  Stark      │  Extrem     │
│            │             │             │             │
│ Wikipedia  │ Tageszeitung│ Wahlkampf-  │ Propaganda- │
│ (Sachthema)│ (Kommentar) │ Berichterst.│ Sender      │
```

### 5.3 Aggregations-Regel

Wenn N Quellen zum selben Thema vorliegen:

1. **Konsens-Kern:** Aussagen, die ≥70% der Quellen teilen (bias-gewichtet)
2. **Kontroversen-Zone:** Aussagen mit <70% Übereinstimmung
3. **Ausreißer:** Aussagen, die nur 1 Quelle macht
4. **Bias-Korrektur:** Quellen mit hoher Bias-Amplitude werden im
   Konsens-Score heruntergewichtet (nicht gelöscht!)

**Wichtig:** EthosAI **löscht oder filtert niemals Ergebnisse** aufgrund von
Bias. Stattdessen werden alle Ergebnisse angezeigt, aber **transparent
annotiert**. Die Entscheidung, welche Quellen vertrauenswürdig sind, trifft
**der Mensch, nicht die Maschine**.

---

## 6. Provider-Evaluierung — Kriterienmatrix

Die folgende Matrix dient als **Bewertungsschablone** für jeden in Betracht
gezogenen Suchdienst. Sie ist absichtlich anbieter-neutral gehalten.

| Kriterium | Gewicht | Score 0–5 | Beschreibung |
|-----------|---------|-----------|-------------|
| Code-Offenheit | 5 | _ | FOSS mit aktiver Community? |
| EU-Jurisdiktion | 5 | _ | Sitz und Datenverarbeitung in EU/EWR? |
| Keine Nicht-EU-Kontrolle | 5 | _ | Kein US/CN/RU Mutterkonzern? |
| Self-Hosting möglich | 4 | _ | Docker/OCI On-Premise? |
| Eigener Index | 3 | _ | Eigener Crawler vs. Drittanbieter? |
| Meta-Suche-Fähigkeit | 4 | _ | Aggregation über ≥5 Engines? |
| JSON-API | 4 | _ | Strukturierte API nativ? |
| Kein Tracking | 5 | _ | Zero-Knowledge bzgl. Nutzeridentität? |
| Kein Paid Ranking | 4 | _ | Keine bezahlten Ergebnisplätze? |
| Engine-Attribution | 3 | _ | Sichtbar, welche Quelle welches Ergebnis lieferte? |
| Tor/Proxy-Kompatibilität | 2 | _ | Anfragen über anonyme Netze? |
| Verfügbarkeit (SLA) | 3 | _ | 99.x% Uptime oder Self-Host-Garantie? |
| **Gesamtpunktzahl** | — | **/235** | Gewichteter Score |

> **Jeder Suchdienst muss diese Matrix durchlaufen, bevor er als Provider
> konfiguriert wird.** Die Bewertung wird im Audit-Trail gespeichert.

---

## 7. Security-Tier-Integration

Die Internet-Research-Toolbox integriert sich in das dreistufige
Sicherheitsmodell von EthosAI (→ siehe separates Konzept:
*„EthosAI Security Tiers — Konzept"*).

| Aspekt | Tier 1 — Standard | Tier 2 — Erhöht | Tier 3 — Höchste |
|--------|-------------------|-----------------|------------------|
| **Provider-Auswahl** | FOSS oder EU-kommerziell | Nur FOSS, self-hosted | Nur FOSS, air-gapped oder Tor-routed |
| **Suchanfragen** | TLS-verschlüsselt | TLS + VPN zum Provider | Tor-Circuit oder dedizierter Proxy-Chain |
| **Audit-Trail** | Logfile, 90 Tage | Signiertes JSONL, 7 Jahre | HSM-signiert, Langzeitarchiv |
| **Ergebnis-Cache** | Encrypted at rest | Encrypted + zugriffsbeschränkt | Encrypted + Air-Gap + HSM-Key |
| **Metadaten-Schutz** | Keine IP-Leaks | Kein DNS-Leak, VPN enforced | Keine Korrelation Anfrage↔Nutzer |
| **Provider-Audit** | Kriterienmatrix | + Code-Review durch Betreiber | + BSI/ANSSI-Zertifizierung |

---

## 8. Verzeichnisstruktur

```
internet-research-toolbox/
├── __init__.py                 # Modul-Docstring
├── toolbox.yaml                # Manifest (8 Tools, 8 Capabilities)
├── README.md                   # Übersicht + API-Beispiele
│
├── web_search.py               # Tool: Internet-Suche (multi-provider)
├── source_bias_analyzer.py     # Tool: Quellen-Bias-Analyse
├── cross_reference.py          # Tool: Quellen-Quervergleich
├── fact_check.py               # Tool: Faktenprüfung
├── source_credibility.py       # Tool: Glaubwürdigkeits-Score
├── research_report.py          # Tool: Aggregierter Recherchebericht
├── domain_intel.py             # Tool: Domain-Hintergrundrecherche
├── audit_trail.py              # Tool: Recherche-Protokollierung
│
├── models.py                   # Dataclasses: SearchResult, BiasAnnotation, ...
├── providers/                  # Provider-Implementierungen
│   ├── __init__.py
│   ├── base.py                 # SearchProvider Protocol
│   └── ... (pro Provider)     # z.B. searxng.py, metager.py, etc.
├── bias/                       # Bias-Analyse-Engine
│   ├── __init__.py
│   ├── taxonomy.py             # BiasKind Enum + Amplitude-Skala
│   ├── heuristics.py           # Regelbasierte Bias-Erkennung
│   └── databases.py            # Adapter für Bias-Datenbanken
├── compliance/                 # Security & Compliance
│   ├── __init__.py
│   ├── tier_policy.py          # Security-Tier-Router
│   ├── audit.py                # Audit-Trail-Writer
│   └── encryption.py           # At-Rest-Encryption für Cache
│
└── docs/
    ├── concept_de.md           # Dieses Dokument
    ├── concept_en.md           # English version
    ├── provider_evaluation.md  # Provider-Bewertungsprotokoll
    └── bias_taxonomy.md        # Ausführliche Bias-Taxonomie-Doku
```

---

## 9. Toolbox-Manifest (Entwurf)

```yaml
name: internet_research
version: "1.0.0"
display_name: "Internet-Recherche"
description: >-
  Anbieter-neutrale Internet-Recherche mit Multi-Provider-Suche,
  Bias-Transparenz, Faktenprüfung und Quellen-Glaubwürdigkeitsbewertung.
  EU-konform, DSGVO-compliant, dreistufiges Sicherheitsmodell.
domain: INTERNET_RESEARCH
icon: "🔍"
author: "EthosAI"
created: "2026-03-07"

requires: []
pip_requires:
  - "httpx>=0.25"
  - "pydantic>=2.0"

singletons: []
startup: []

tools:
  - name: web_search
    function: web_search.search
    display_name: "Internet-Suche"
    description: "Multi-Provider-Suche mit Deduplizierung und Quellenattribution"
    keywords: [suche, search, web, internet, meta, provider]

  - name: source_bias_analyzer
    function: source_bias_analyzer.analyze_bias
    display_name: "Quellen-Bias-Analyse"
    description: "Bias-Art, Motivation, Wirkung und Amplitude einer Quelle bewerten"
    keywords: [bias, verzerrung, quelle, analyse, motivation, wirkung]

  - name: cross_reference
    function: cross_reference.cross_check
    display_name: "Quellen-Quervergleich"
    description: "Aussagen über mehrere unabhängige Quellen hinweg vergleichen"
    keywords: [quervergleich, cross-reference, validierung, konsens]

  - name: fact_check
    function: fact_check.verify_claim
    display_name: "Faktenprüfung"
    description: "Behauptung gegen EU-Faktencheck-Datenbanken prüfen"
    keywords: [faktencheck, factcheck, verifikation, claim, wahrheit]

  - name: source_credibility
    function: source_credibility.assess_credibility
    display_name: "Quellen-Glaubwürdigkeit"
    description: "Mehrdimensionale Glaubwürdigkeitsbewertung einer Domain"
    keywords: [glaubwürdigkeit, credibility, vertrauen, quelle, domain]

  - name: research_report
    function: research_report.generate_report
    display_name: "Recherchebericht"
    description: "Vollständiger Recherchebericht mit Bias-Map und Konsens-Analyse"
    keywords: [bericht, report, recherche, zusammenfassung, analyse]

  - name: domain_intel
    function: domain_intel.investigate_domain
    display_name: "Domain-Intelligence"
    description: "Hintergrundrecherche über Domain (WHOIS, Eigentümer, Finanzierung)"
    keywords: [domain, whois, eigentümer, impressum, hintergrund]

  - name: audit_trail
    function: audit_trail.query_audit
    display_name: "Recherche-Protokoll"
    description: "Unveränderlicher Audit-Trail aller Suchanfragen"
    keywords: [audit, protokoll, trail, compliance, nachweis]

capabilities:
  - id: internet-search
    name: "Internet-Suche"
    description: "Im Internet nach Informationen suchen (multi-provider, EU-konform)"
    keywords: [suche, internet, web, recherche]
    skill: web_search

  - id: bias-analysis
    name: "Bias-Analyse"
    description: "Verzerrungen in Quellen erkennen und transparent annotieren"
    keywords: [bias, verzerrung, analyse, transparenz]
    skill: source_bias_analyzer

  - id: cross-reference
    name: "Quellen-Quervergleich"
    description: "Aussagen über mehrere Quellen verifizieren"
    keywords: [quervergleich, verifizierung, konsens]
    skill: cross_reference

  - id: fact-checking
    name: "Faktenprüfung"
    description: "Behauptungen gegen Faktencheck-Datenbanken prüfen"
    keywords: [faktencheck, verifikation, wahrheit]
    skill: fact_check

  - id: source-credibility
    name: "Quellen-Glaubwürdigkeit"
    description: "Vertrauenswürdigkeit von Quellen bewerten"
    keywords: [glaubwürdigkeit, vertrauen, bewertung]
    skill: source_credibility

  - id: research-report
    name: "Recherchebericht"
    description: "Strukturierte Recherche mit Bias-Map durchführen"
    keywords: [recherche, bericht, analyse, bias-map]
    skill: research_report

  - id: domain-intelligence
    name: "Domain-Intelligence"
    description: "Hintergrundinformationen über Domains recherchieren"
    keywords: [domain, intelligence, hintergrund, osint]
    skill: domain_intel

  - id: audit-compliance
    name: "Recherche-Audit"
    description: "Compliance-konforme Protokollierung aller Recherchen"
    keywords: [audit, compliance, protokoll, dsgvo]
    skill: audit_trail
```

---

## 10. Nicht-Ziele (Abgrenzung)

| Nicht-Ziel | Begründung |
|-----------|-----------|
| **Eigener Webcrawler** | EthosAI crawlt nicht selbst — zu hoher Infrastrukturaufwand. Stattdessen werden bestehende Indizes über Provider-API abgefragt |
| **Zensur oder Filterung** | EthosAI filtert keine Ergebnisse heraus. Bias wird annotiert, nicht entfernt |
| **Eigene Faktenbewertung** | EthosAI erstellt keine eigenen Faktenprüfungen — es greift auf existierende, anerkannte Datenbanken zu |
| **Dark-Web-Zugang** | .onion-Routing nur für Anfrage-Anonymisierung (Tier 3), nicht für Dark-Web-Inhalts-Suche |
| **Konkurrenz zu Suchmaschinen** | EthosAI ist kein Suchmaschinen-Ersatz, sondern ein Recherche-Werkzeug mit Analyse-Schicht |

---

## 11. Offene Entscheidungen

| # | Frage | Optionen | Empfehlung |
|---|-------|---------|------------|
| OE-1 | Bias-Datenbank-Quelle? | EDMO, EUvsDisinfo, eigene Heuristik, Kombination | Kombination: DB-Lookup + Heuristik-Fallback |
| OE-2 | Cache-Strategie? | Kein Cache, TTL-basiert, semantic dedup | TTL 24h (Tier 1), kein Cache (Tier 3) |
| OE-3 | Maximale Provider-Anzahl? | 1, 3, 5, unbegrenzt | 3 aktive + 2 Fallback |
| OE-4 | Bias-Modell trainierbar? | Statisch, regelbasiert, ML-basiert | Regelbasiert + DB-Lookup (kein ML-Training auf Nutzerdaten) |
| OE-5 | Integration CLIM? | Bias als CLIM-Input, CLIM steuert Recherche-Tiefe | Ja — `research_complexity` als CLIM-Signal |

---

## 12. Referenzen

| Quelle | Typ | Relevanz |
|--------|-----|----------|
| DSGVO (Verordnung EU 2016/679) | EU-Recht | Datenschutz, Art. 17/20/22/25/44–49 |
| EU AI Act (Verordnung EU 2024/1689) | EU-Recht | Transparenzpflicht für KI-Systeme |
| Digital Services Act (DSA) | EU-Recht | Transparenz von Empfehlungssystemen |
| NIS2-Richtlinie (EU 2022/2555) | EU-Recht | Cybersicherheit für kritische Infrastruktur |
| BSI IT-Grundschutz | DE-Standard | Sicherheitsanforderungen Tier 2–3 |
| ANSSI Referenzrahmen | FR-Standard | Sicherheitsanforderungen Tier 3 |
| SearXNG Dokumentation | FOSS | Referenz-Meta-Suchmaschine (AGPL-3.0) |
| MetaGer Dokumentation | Non-Profit | Referenz EU-Meta-Suche (Hannover) |
| EDMO | EU-Projekt | European Digital Media Observatory |
| EUvsDisinfo | EU-Projekt | Desinformations-Datenbank |

---

*Dieses Konzept wird als lebendes Dokument gepflegt und vor der Implementierung
mit dem Security-Tier-Konzept abgestimmt.*
