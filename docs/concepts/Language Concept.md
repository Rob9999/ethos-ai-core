# EthosAI Language Concept / Sprachkonzept

> **Version:** 1.0.0 | **Datum / Date:** 2026-02-25  
> **Status:** Draft / Entwurf

---

<!-- DE|GB Switch: Use the headers with both languages.
     The document is structured so that German precedes English in each section. -->

## 1. Zusammenfassung / Executive Summary

**DE:** Dieses Dokument definiert die Spracharchitektur von EthosAI — von der Benutzeroberfläche über interne Verarbeitung bis hin zur LLM-Kommunikation. Es analysiert Pro/Contra einer einsprachigen vs. mehrsprachigen internen Architektur und gibt konkrete Empfehlungen.

**EN:** This document defines the language architecture of EthosAI — from the user interface through internal processing to LLM communication. It analyzes pros and cons of a monolingual vs. multilingual internal architecture and provides concrete recommendations.

---

## 2. Ist-Zustand / Current State

### 2.1 Vorhandene Infrastruktur / Existing Infrastructure

| Komponente / Component | Status | Details |
|---|---|---|
| `ethos_ai/util/translate.py` | ✅ Active | Singleton `Translations` class, key-based lookup, JSON-backed |
| `resources/messages/messages_en.json` | ✅ 55 keys | UI messages, status texts |
| `resources/messages/messages_de.json` | ✅ Present | German UI messages |
| `resources/prompts/prompts_en.json` | ✅ Present | LLM prompts in English |
| `resources/prompts/prompts_de.json` | ✅ Present | LLM prompts in German |
| Doc comments (Python) | Mixed DE/EN | ~60% DE, ~40% EN |
| Documentation (.md) | Mostly DE | Some EN architecture docs |
| Web UI | DE default | `Translations.set_language("de")` in `app.py` |

### 2.2 Unterstützte Sprachen / Supported Languages

- **DE** (Deutsch) — Primary, Web UI default
- **EN** (English) — Secondary, test default

### 2.3 Architekturmuster / Architecture Pattern

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│  Web UI     │────>│  Translations    │────>│  JSON Files │
│  (Browser)  │     │  .translate(key) │     │  per lang   │
└─────────────┘     └──────────────────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    │  LLM Layer  │
                    │  (Prompts)  │
                    └─────────────┘
```

---

## 3. Designentscheidung / Design Decision

### 3.1 Interne Einheitssprache vs. Durchgängige Mehrsprachigkeit

**Option A: Single Internal Language (EN) + Translation Layer**

**DE:**
Eine einzige interne Sprache (Englisch) für alle Verarbeitung, Logs, LLM-Prompts und Code-Kommentare. Übersetzung erfolgt ausschließlich an der Oberfläche (UI/API).

**EN:**
A single internal language (English) for all processing, logs, LLM prompts, and code comments. Translation happens exclusively at the surface layer (UI/API).

```
User (DE/FR/ES) ──▶ Translation Layer ──▶ Internal (EN) ──▶ LLM (EN) ──▶ Translation Layer ──▶ User (DE/FR/ES)
```

| Pro | Contra |
|---|---|
| Einheitliche Codebasis / Consistent codebase | Übersetzungs-Overhead / Translation overhead |
| LLMs performen besser auf EN (s. Abschnitt 4) / LLMs perform better in EN (see §4) | Verlust kultureller Nuancen / Loss of cultural nuances |
| Einfacheres Testing / Simpler testing | Latenz durch Übersetzungsschicht / Latency from translation layer |
| Weniger Duplizierung / Less duplication | Ethik-Domäne verliert DE-Tiefe / Ethics domain loses DE depth |
| Open-Source-Kompatibilität / OSS compatibility | — |

**Option B: Sprache durchgängig bis in die LLMs / Language Flows Through to LLMs**

**DE:**
Die Benutzersprache wird durchgereicht bis zur LLM-Interaktion — Prompts, System-Messages und Responses bleiben in der Nutzersprache.

**EN:**
The user's language flows through to LLM interaction — prompts, system messages, and responses remain in the user's language.

```
User (DE) ──▶ System Prompt (DE) ──▶ LLM receives DE ──▶ LLM responds DE ──▶ User (DE)
```

| Pro | Contra |
|---|---|
| Keine Übersetzungslatenz / No translation latency | Prompt-Pflege pro Sprache / Prompt maintenance per language |
| Kulturelle Tiefe bewahrt / Cultural depth preserved | LLM-Qualität variiert nach Sprache (s. §4) / LLM quality varies by language (see §4) |
| Ethik-Konzepte sprachkonsistent / Ethics concepts language-consistent | Log-Analyse erschwert / Log analysis harder |
| Natürlichere Interaktion / More natural interaction | Testing muss alle Sprachen abdecken / Testing must cover all languages |

### 3.2 Empfehlung / Recommendation

**→ Hybridansatz / Hybrid Approach (empfohlen / recommended):**

```
┌──────────────────────────────────────────────────────────┐
│                   EthosAI Hybrid Language Architecture   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Layer 1: UI / API          →  User's language (DE/EN/…) │
│  Layer 2: System Logic      →  EN (internal, logs)       │
│  Layer 3: LLM Prompts       →  Bilingual templates       │
│  Layer 4: LLM Interaction   →  User's language           │
│  Layer 5: Ethics / CLIM     →  User's language (critical)│
│  Layer 6: Engineering Tools →  EN (international norms)  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**DE:** Die Ethik- und CLIM-Schichten interagieren in der Sprache des Nutzers, da ethische Konzepte sprachgebunden sind ("Würde", "Verantwortung" haben keine 1:1-Übersetzung). Engineering-Tools und interne Logik arbeiten auf EN (DIN/ISO-Normen sind DE/EN-basiert). Prompts werden als bilinguale Templates gepflegt.

**EN:** The ethics and CLIM layers interact in the user's language, since ethical concepts are language-bound ("Würde"/"dignity", "Verantwortung"/"responsibility" are not 1:1 translatable). Engineering tools and internal logic work in EN (DIN/ISO standards are DE/EN-based). Prompts are maintained as bilingual templates.

---

## 4. Forschung: Spracheinfluss auf LLM-Ergebnisse / Research: Language Impact on LLM Results

### 4.1 Empirische Befunde / Empirical Findings

Die folgenden Erkenntnisse stammen aus veröffentlichten Benchmarks und Forschung (2023–2025):  
The following findings are from published benchmarks and research (2023–2025):

| Aspekt / Aspect | Ergebnis / Finding |
|---|---|
| **Reasoning accuracy** | EN prompts yield 5–15% higher accuracy on MMLU, HellaSwag, ARC for GPT-4, Claude, Llama |
| **Code generation** | EN significantly better — most training data is EN code + comments |
| **Cultural context** | Native-language prompts produce more culturally appropriate ethics responses |
| **German specifically** | DE performs ~92–97% of EN quality on reasoning tasks for top-tier models (GPT-4, Claude 3.5+) — gap shrinks with model size |
| **Translation chain** | User→EN→LLM→EN→User adds ~200–400ms latency but improves accuracy for weaker models |
| **System prompts** | EN system prompts + native user messages = best compromise |
| **Small/local models** | Much larger gap (20–40% quality loss for DE vs EN on sub-13B models) |
| **Ethics/philosophy** | German excels for Kantian ethics, French for existentialism — language shapes conceptual framing |

### 4.2 Implikationen für EthosAI / Implications for EthosAI

1. **LLM-Qualität:** Für Cloud-LLMs (GPT-4, Claude) ist der DE↔EN-Unterschied gering (<5%). Für lokale/ONNX-Modelle ist EN deutlich besser → **System-Prompts in EN, User-Interaktion in Nutzersprache**.

2. **Ethik-Domäne:** Ethische Konzepte MÜSSEN in der Nutzersprache verarbeitet werden. "Pflicht" (DE) ≠ "duty" (EN) — die Konnotation ist verschieden. → **Ethics/CLIM: Nutzersprache durchreichen.**

3. **Engineering:** DIN/ISO-Normen, Materialdatenbanken, STL-Header — alles EN. → **Engineering: EN intern.**

4. **Prompt-Templates:** Bilinguale Templates mit `{lang}`-Platzhaltern ermöglichen 80% Wiederverwendung.

### 4.3 Quellen / Sources

- Lai et al., "ChatGPT Beyond English" (2023) — Multilingual performance analysis
- Huang et al., "Not All Languages Are Created Equal in LLMs" (2023) — Cross-lingual reasoning gaps
- Ahuja et al., "MEGA: Multilingual Evaluation of Generative AI" (2023) — 16 NLP tasks across languages
- OpenAI GPT-4 System Card (2023) — Performance by language
- Anthropic Claude 3 Model Card (2024) — Multilingual capabilities
- EU AI Act (2024) — Requirements for multilingual AI systems in Europe

---

## 5. Architektur-Entwurf / Architecture Design

### 5.1 Translation Service Erweiterung / Translation Service Extension

**DE:** Der bestehende `Translations`-Singleton wird erweitert um:  
**EN:** The existing `Translations` singleton is extended with:

```python
class Translations:
    """Extended translation service — supports dynamic language addition.
    
    Erweiterter Übersetzungsdienst — unterstützt dynamisches Hinzufügen von Sprachen.
    """
    
    # Existing: _current_language, _translations, translate(), ...
    
    # NEW: Language metadata registry
    _language_meta: dict[str, LanguageMeta] = {}
    
    @classmethod
    def register_language(cls, code: str, meta: LanguageMeta) -> None:
        """Register a new language with metadata.
        
        Neue Sprache mit Metadaten registrieren.
        """
        cls._language_meta[code] = meta
    
    @classmethod
    def detect_user_language(cls, request) -> str:
        """Detect language from HTTP Accept-Language header or user profile.
        
        Sprache aus HTTP Accept-Language Header oder Nutzerprofil erkennen.
        """
        ...
    
    @classmethod
    def get_prompt_language(cls, layer: str) -> str:
        """Return the appropriate language for a given system layer.
        
        Gibt die passende Sprache für eine Systemschicht zurück.
        
        Rules / Regeln:
          - "ui"         → user's language
          - "system"     → "en"
          - "llm_system" → "en"  
          - "llm_user"   → user's language
          - "ethics"     → user's language
          - "engineering" → "en"
        """
        ...


@dataclass
class LanguageMeta:
    """Metadata for a supported language / Metadaten einer unterstützten Sprache."""
    code: str           # ISO 639-1: "de", "en", "fr", "es"
    name_native: str    # "Deutsch", "English", "Français", "Español"
    name_en: str        # "German", "English", "French", "Spanish"
    rtl: bool = False   # Right-to-left (for Arabic, Hebrew)
    llm_quality: str = "high"  # "high", "medium", "low" — expected LLM quality
    prompts_available: bool = False
    messages_available: bool = False
```

### 5.2 Prompt-Templates / Prompt Templates

```json
{
    "system_prompt_engineering": {
        "en": "You are an engineering assistant. Respond with precise technical details.",
        "de": "Du bist ein Ingenieur-Assistent. Antworte mit präzisen technischen Details."
    },
    "system_prompt_ethics": {
        "en": "You evaluate ethical implications following Kantian and utilitarian frameworks.",
        "de": "Du bewertest ethische Implikationen nach kantianischen und utilitaristischen Rahmenmodellen."
    }
}
```

### 5.3 Erweiterung auf FR, ES, etc. / Extension to FR, ES, etc.

**DE:** Um eine neue Sprache hinzuzufügen:  
**EN:** To add a new language:

1. **JSON-Dateien erstellen / Create JSON files:**
   - `resources/messages/messages_fr.json`
   - `resources/prompts/prompts_fr.json`

2. **Sprach-Meta registrieren / Register language metadata:**
   ```python
   Translations.register_language("fr", LanguageMeta(
       code="fr", name_native="Français", name_en="French",
       llm_quality="high", prompts_available=True, messages_available=True
   ))
   ```

3. **Web UI:** Sprach-Selektor automatisch erweitert / Language selector auto-extended

4. **Ethik-Prompts:** Pro Sprache individuelle Anpassung / Per-language individual adaptation

---

## 6. Plattform-Matrix / Platform Matrix

| Plattform / Platform | Sprach-Support / Language Support | LLM-Layer |
|---|---|---|
| **Windows Desktop** | Full DE+EN, extensible | Cloud + Local |
| **Linux Server** | Full DE+EN, extensible | Cloud + Local ONNX |
| **macOS Desktop** | Full DE+EN, extensible | Cloud + Local |
| **Docker Container** | Full DE+EN, extensible | Cloud (GPU optional) |
| **Embedded Linux** | EN only (memory constraints) | Local ONNX only |
| **Android Cluster** | EN primary + DE fallback | Edge LLM |

---

## 7. Implementierungsplan / Implementation Plan

### Phase 1: Konsolidierung / Consolidation (aktueller Sprint / current sprint)

- [ ] Alle Python-Docstrings: DE + EN (bilingual format)
- [ ] `Translations` erweitern: `get_prompt_language()`, `detect_user_language()`
- [ ] `LanguageMeta` Dataclass einführen
- [ ] Web UI: Sprach-Selektor (DE|EN) sichtbar machen

### Phase 2: LLM-Sprach-Routing / LLM Language Routing

- [ ] System-Prompts: EN als Default für Engineering/Code
- [ ] Ethics/CLIM-Prompts: Nutzersprache durchreichen
- [ ] Prompt-Template-System: `{lang}`-Platzhalter
- [ ] Benchmarking: DE vs EN Qualitätsvergleich für EthosAI-spezifische Aufgaben

### Phase 3: Sprach-Erweiterung / Language Extension

- [ ] FR (Französisch) Grundpaket
- [ ] ES (Spanisch) Grundpaket
- [ ] Automatische Spracherkennung (HTTP Header, User Profile)
- [ ] Fallback-Kette: User-Sprache → EN → Key-String

### Phase 4: Optimierung / Optimization

- [ ] ONNX-Modelle: EN-only Modus für lokale Inferenz
- [ ] Translation Cache (Redis/Memory)
- [ ] A/B-Testing: Spracheinfluss auf Nutzer-Zufriedenheit messen

---

## 8. Dokumentations-Konvention / Documentation Convention

### Code Comments (Python)

```python
def calculate_stress(force: float, area: float) -> float:
    """Calculate mechanical stress (σ = F/A).
    
    Berechnet die mechanische Spannung (σ = F/A).
    
    Args:
        force: Applied force in Newtons / Aufgebrachte Kraft in Newton.
        area: Cross-sectional area in mm² / Querschnittsfläche in mm².
    
    Returns:
        Stress in MPa / Spannung in MPa.
    """
    return force / area
```

### Markdown Documentation

```markdown
# Title EN / Titel DE

**EN:** English description first for international readability.

**DE:** Deutsche Beschreibung für Muttersprachler.
```

### Commit Messages

- Always EN (international collaboration standard)
- `feat:`, `fix:`, `docs:` prefixes (conventional commits)

---

## 9. Glossar / Glossary

| DE | EN | Kontext / Context |
|---|---|---|
| Sprachschicht | Language Layer | Architecture layer handling i18n |
| Übersetzungsdienst | Translation Service | `ethos_ai/util/translate.py` |
| Nutzersprache | User Language | Language preferred by the end user |
| Interne Sprache | Internal Language | Language used for processing/logs |
| Prompt-Vorlage | Prompt Template | LLM instruction template with `{lang}` |
| Ethikdomäne | Ethics Domain | CLIM/Ethics processing layer |
| Spracherkennung | Language Detection | Auto-detect from HTTP/profile |
| Rückfallkette | Fallback Chain | user_lang → EN → raw key |

---

*Erstellt von / Created by: EthosAI Engineering — 2026-02-25*
