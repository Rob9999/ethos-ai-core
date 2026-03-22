---
title: "EthosAI Security Tiers — Dreistufiges Einsatz-Sicherheitsmodell"
brief: "Konzept für ein dreistufiges Sicherheitsmodell das EthosAI vom Standard-Business-Einsatz bis zum Regierungsbetrieb absichert — aufbauend auf der bestehenden Need-to-Know Architecture."
status: draft
version: "0.1.0"
author: Robert Alexander Massinger & GitHub Copilot (Claude Opus 4.6)
date: 2026-03-07
tags: [security, tiers, dsgvo, behörden, regierung, bsi, anssi, air-gap, encryption, compliance]
target-release: "v1.64.0"
predecessors:
  - "Security Architecture — JWT, RBAC & TLS (v1.2.0)"
  - "Security & Geheimhaltung — Need-to-Know Architecture (v1.5.0)"
history:
  - date: 2026-03-07
    change: "Erstversion — Drei-Tier-Modell, Compliance-Matrizen, Deployment-Varianten, Integration bestehender Module."
---

# EthosAI Security Tiers — Dreistufiges Einsatz-Sicherheitsmodell

## 1. Motivation

EthosAI wird in sehr unterschiedlichen Umgebungen eingesetzt:

- Ein **Solo-Ingenieur** nutzt EthosAI als persönlichen Assistenten — Datenschutz
  nach DSGVO reicht aus.
- Ein **Forschungslabor** einer Universität arbeitet mit Prototypdaten unter
  NDA — der IT-Sicherheitsbeauftragte verlangt BSI-Grundschutz-Konformität.
- Eine **Behörde oder ein Verteidigungsunternehmen** verarbeitet
  eingestufte Informationen (VS-NfD, VS-Vertraulich) — hier gelten
  Geheimschutzvorschriften.

Ein einziges Sicherheitsniveau kann diese Bandbreite nicht abdecken. Daher
führt dieses Konzept ein **dreistufiges Sicherheitsmodell** ein, das
skalierbar, konfigurierbar und auditierbar ist.

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                     EthosAI Security Tiers                        │
  │                                                                   │
  │  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐   │
  │  │   TIER 1        │  │   TIER 2        │  │   TIER 3         │   │
  │  │   Standard      │  │   Erhöht        │  │   Höchste        │   │
  │  │                 │  │                 │  │                  │   │
  │  │  DSGVO          │  │  BSI-Grundschutz│  │  VS-NfD / GEHEIM │   │
  │  │  NDA            │  │  Behörden       │  │  Air-Gap         │   │
  │  │  EU-Recht       │  │  Forschungslabs │  │  HSM-Krypto      │   │
  │  │  TLS + RBAC     │  │  On-Premise     │  │  Kein Internet   │   │
  │  └─────────────────┘  └─────────────────┘  └──────────────────┘   │
  │         ▲                     ▲                     ▲             │
  │         │                     │                     │             │
  │  Bestehende Security   Erweiterung durch      Maximale            │
  │  Architecture (v1.2)   Need-to-Know (v1.5)   Härtung (v1.64+)     │
  └───────────────────────────────────────────────────────────────────┘
```

---

## 2. Bestandsaufnahme — Was existiert bereits?

Vor der Definition der drei Tiers ein Überblick über die implementierten
Security-Module:

| Modul | Pfad | Seit | Funktion |
|-------|------|------|----------|
| **JWT + RBAC** | `ethos_ai/api/auth.py` | v1.2.0 | Authentifizierung, 4 Rollen (admin, advisor, user, readonly) |
| **DSGVO Checks** | BL-062 | v1.2.0 | Grundlegende Datenschutz-Compliance |
| **TLS Enforcement** | Config | v1.2.0 | HTTPS-only für API-Kommunikation |
| **ClassificationLevel** | `ethos_ai/security/classification_level.py` | v1.5.0 | 4-Stufen: PUBLIC → INTERNAL → CONFIDENTIAL → SECRET |
| **ProjectNamespace** | `ethos_ai/security/project_namespace.py` | v1.5.0 | Projekt-Isolation, Chinese-Wall-Prinzip |
| **NDA Registry** | `ethos_ai/security/nda_registry.py` | v1.5.0 | Vertragsverwaltung, Compliance-Check |
| **Information Barriers** | `ethos_ai/security/information_barrier.py` | v1.5.0 | Chinese-Wall-Enforcement + Audit-Trail |
| **Security Service** | `ethos_ai/api/security_service.py` | v1.5.0 | Zentrale Security-API (Facade) |
| **Sandbox** | `ethos_ai/security/sandbox.py` | v1.3.0 | Code-Execution-Isolation |

> **Erkenntnis:** Tier 1 ist bereits weitgehend implementiert. Tier 2 erfordert
> Konfigurationshärtung + Deployment-Anpassungen. Tier 3 erfordert neue Module.

---

## 3. Tier-Definitionen

### 3.1 Tier 1 — Standard-Sicherheit

**Zielgruppe:** Einzelunternehmer, KMU, Ingenieurbüros, Freelancer  
**Regulatorischer Rahmen:** DSGVO, EU-Recht, branchenübliche NDAs  
**Deployments:** Cloud (EU-Region), lokaler Server, Desktop  

#### 3.1.1 Anforderungen

| Bereich | Anforderung | Status | Implementierung |
|---------|------------|--------|----------------|
| **Authentifizierung** | JWT + RBAC (4 Rollen) | ✅ Impl. | `auth.py` |
| **Transportverschlüsselung** | TLS 1.2+ | ✅ Impl. | Config |
| **Datenverschlüsselung** | At-Rest für CONFIDENTIAL+ | ✅ Impl. | `classification_level.py` |
| **Datenschutz** | DSGVO Compliance-Checks | ✅ Impl. | BL-062 |
| **NDA-Management** | NDA Registry + Compliance | ✅ Impl. | `nda_registry.py` |
| **Audit-Trail** | Basis-Logging (who, when, what) | ✅ Impl. | `information_barrier.py` |
| **Projekt-Isolation** | Logische Namespace-Trennung | ✅ Impl. | `project_namespace.py` |
| **Sandbox** | Code-Execution in Sandbox | ✅ Impl. | `sandbox.py` |
| **Backup** | Regelmäßige verschlüsselte Backups | 🔲 Config | Operativ |
| **Updates** | Automatische Security-Patches | 🔲 Config | PyPI/GitHub |

#### 3.1.2 Deployment-Optionen

```
┌──────────────────────────────────────────────┐
│  Tier 1 — Deployment                         │
│                                              │
│  Option A: EU-Cloud (managed)                │
│  ┌────────┐    TLS    ┌─────────┐            │
│  │ Client ├───────────┤ EthosAI │ EU-DC      │
│  └────────┘           │ Server  │            │
│                       └────┬────┘            │
│                            │ TLS             │
│                       ┌────▼────┐            │
│                       │ EU LLM  │ EU-hosted  │
│                       │ Provider│            │
│                       └─────────┘            │
│                                              │
│  Option B: Lokaler Server / Desktop          │
│  ┌────────┐ localhost ┌─────────┐            │
│  │ Client ├───────────┤ EthosAI │ On-Prem    │
│  └────────┘           │ Server  │            │
│                       └────┬────┘            │
│                            │ TLS             │
│                       ┌────▼────┐            │
│                       │ EU LLM  │ API        │
│                       │ Provider│            │
│                       └─────────┘            │
└──────────────────────────────────────────────┘
```

---

### 3.2 Tier 2 — Erhöhte Sicherheit

**Zielgruppe:** Behörden, Forschungslabore, Unternehmen mit Geheimschutzbetreuung  
**Regulatorischer Rahmen:** BSI IT-Grundschutz, ISO 27001, NIS2-Richtlinie  
**Deployments:** On-Premise (Pflicht), Air-Gap optional  

#### 3.2.1 Zusätzliche Anforderungen (über Tier 1 hinaus)

| Bereich | Anforderung | Status | Umsetzung |
|---------|------------|--------|-----------|
| **Authentifizierung** | MFA (TOTP/FIDO2) | 🔲 Neu | `auth.py` erweitern |
| **Zertifikate** | Client-Zertifikate (mTLS) | 🔲 Neu | Reverse-Proxy + Config |
| **Krypto-Standards** | BSI TR-02102-2 (TLS-Cipher-Suites) | 🔲 Config | Config-Hardening |
| **Schlüsselverwaltung** | Dedizierter Key-Store (PKCS#11 oder SoftHSM) | 🔲 Neu | `security/key_store.py` |
| **Audit-Trail** | Signierte, unveränderliche Logs (HMAC) | 🔲 Neu | `information_barrier.py` erweitern |
| **Netzwerk** | VPN-only Access, keine öffentlichen Endpunkte | 🔲 Config | Deployment |
| **LLM-Provider** | Nur self-hosted oder EU-sovereign (kein US-Cloud) | 🔲 Config | `llm_provider_policy.py` |
| **Pentest** | Jährlicher Penetrationstest durch akkreditierte Stelle | 🔲 Prozess | Operativ |
| **Incident Response** | Dokumentierter IR-Plan, max. 72h Meldung (NIS2 Art. 23) | 🔲 Prozess | Operativ |
| **Lösch-Konzept** | Automatisierte Datenlöschung nach Retention-Policy | 🔲 Neu | `security/retention.py` |
| **Log-Aufbewahrung** | Min. 7 Jahre (handelsrechtliche Aufbewahrung) | 🔲 Config | Audit-Trail |
| **Code-Audit** | Nachvollziehbare Code-Herkunft (SBOM, signierte Builds) | 🔲 Prozess | CI/CD |

#### 3.2.2 LLM-Provider-Policy

In Tier 2 dürfen **keine externen US-Cloud-LLMs** verwendet werden.
Erlaubte Varianten:

| Variante | Beschreibung | Datenlage |
|----------|-------------|-----------|
| **Self-Hosted FOSS** | z.B. Llama, Mistral, Qwen (FOSS-Modelle) auf eigener Hardware | Daten verlassen nie das Netz |
| **EU-Sovereign Cloud** | z.B. OVH, Hetzner, IONOS mit EU-LLM-Hosting (Mistral Le Chat, etc.) | Daten bleiben in EU-DC |
| **On-Premise Appliance** | Dedizierte GPU-Box mit vorinstalliertem Modell | Kein Netzzugang nötig |

```python
class LLMProviderPolicy(Protocol):
    """Policy-Interface für erlaubte LLM-Provider pro Tier."""

    def is_allowed(
        self,
        provider_id: str,
        security_tier: SecurityTier,
    ) -> tuple[bool, str]:
        """Return (erlaubt, Begründung)."""
        ...

    def get_allowed_providers(
        self,
        security_tier: SecurityTier,
    ) -> list[str]:
        """Alle für diesen Tier erlaubten Provider-IDs."""
        ...
```

#### 3.2.3 Deployment-Architektur

```
┌──────────────────────────────────────────────────────────────┐
│  Tier 2 — On-Premise Deployment                              │
│                                                              │
│  ┌─────────────────────────────┐         ┌──────────────┐    │
│  │  Behörden-/Firmen-Netz      │         │  DMZ         │    │
│  │  (intern)                   │         │  (optional)  │    │
│  │                             │         │              │    │
│  │  ┌────────┐  mTLS  ┌──────┐ │  VPN    │  ┌─────────┐ │    │
│  │  │ Client ├────────┤Ethos ├─┼─────────┼──┤Reverse  │ │    │
│  │  │ (MFA)  │        │ AI   │ │         │  │Proxy    │ │    │
│  │  └────────┘        └──┬───┘ │         │  │(mTLS)   │ │    │
│  │                       │     │         │  └─────────┘ │    │
│  │                 ┌─────▼───┐ │         └──────────────┘    │
│  │                 │On-Prem  │ │                             │
│  │                 │ LLM     │ │  ← Kein Internet-Zugang     │
│  │                 │(GPU-Box)│ │                             │
│  │                 └─────────┘ │                             │
│  │                             │                             │
│  │  ┌──────────┐ ┌───────────┐ │                             │
│  │  │SoftHSM / │ │ Signierter│ │                             │
│  │  │Key-Store │ │ Audit-Log │ │                             │
│  │  └──────────┘ └───────────┘ │                             │
│  └─────────────────────────────┘                             │
└──────────────────────────────────────────────────────────────┘
```

---

### 3.3 Tier 3 — Höchste Sicherheit

**Zielgruppe:** Nachrichtendienste, Verteidigungsministerien, kritische Infrastruktur (KRITIS)  
**Regulatorischer Rahmen:** VS-NfD / VS-Vertraulich / GEHEIM, BSI Zulassung,
CC EAL4+, nato.int RESTRICTED+  
**Deployments:** Air-Gapped, gehärtete Hardware, Akkreditierungspflicht  

#### 3.3.1 Zusätzliche Anforderungen (über Tier 2 hinaus)

| Bereich | Anforderung | Status | Umsetzung |
|---------|------------|--------|-----------|
| **Netzwerk** | Vollständig air-gapped — kein Internet, kein LAN | 🔲 Arch. | Deployment-Architektur |
| **Krypto** | HSM-basierte Schlüsselverwaltung (FIPS 140-2 Level 3 oder BSI-zugelassen) | 🔲 Neu | `security/hsm_adapter.py` |
| **Authentifizierung** | Smartcard / PKI (BSI TR-03116-4) | 🔲 Neu | `auth.py` erweitern |
| **Betriebssystem** | Gehärtetes OS (BSI SiSyPHuS / DISA STIG) | 🔲 Prozess | Deployment |
| **Audit-Trail** | HSM-signiert, WORM-Storage, extern prüfbar | 🔲 Neu | `security/worm_audit.py` |
| **Code-Herkunft** | Signierte Releases, SBOM, reproduzierbare Builds | 🔲 Prozess | CI/CD |
| **LLM** | Nur lokales Modell auf zertifizierter Hardware | 🔲 Config | Keine API-Calls |
| **Datenträger** | Full-Disk-Encryption (BSI-zugelassen), Löschnachweis | 🔲 Prozess | Operativ |
| **Tempest** | TEMPEST-geschirmte Hardware (NATO SDIP-27/AMSG-788) | 🔲 Hardware | Beschaffung |
| **Zulassung** | BSI-Akkreditierung des Gesamtsystems | 🔲 Prozess | Akkreditierungsverfahren |
| **Daten-Export** | Kein Export ohne Freigabeverfahren (4-Augen-Prinzip) | 🔲 Neu | `security/export_control.py` |
| **Löschung** | Kryptographische Löschung (Key-Destruction) | 🔲 Neu | `security/crypto_shred.py` |
| **Betrieb** | Nur durch sicherheitsüberprüftes Personal (Ü2/Ü3) | 🔲 Prozess | Operativ |

#### 3.3.2 Air-Gap-Deployment

```
┌───────────────────────────────────────────────────────────────────┐
│  Tier 3 — Air-Gapped Deployment                                   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  TEMPEST-geschirmter Raum                                    │ │
│  │                                                              │ │
│  │  ┌───────────┐   mTLS/PKI   ┌──────────┐   LOCAL  ┌───────┐  │ │
│  │  │ Terminal  ├──────────────┤ EthosAI  ├──────────┤ LLM   │  │ │
│  │  │(Smartcard)│              │ Server   │          │(lokal)│  │ │
│  │  └───────────┘              └────┬─────┘          └───────┘  │ │
│  │                                  │                           │ │
│  │                     ┌────────────┼────────────┐              │ │
│  │                     ▼            ▼            ▼              │ │
│  │                 ┌────────┐  ┌─────────┐  ┌──────────┐        │ │
│  │                 │  HSM   │  │  WORM   │  │Encrypted │        │ │
│  │                 │(Krypto)│  │  Audit  │  │  Storage │        │ │
│  │                 └────────┘  └─────────┘  └──────────┘        │ │
│  │                                                              │ │
│  │  ════════════════════════════════════════════════════════    │ │
│  │  ↕ Einziger Datenweg: Diode / Schleusung (4-Augen)           │ │
│  │  ════════════════════════════════════════════════════════    │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  KEIN Netzwerk-Zugang. KEIN USB ohne Freigabe.                    │
│  KEIN Fernzugriff. KEIN Cloud-LLM.                                │
└───────────────────────────────────────────────────────────────────┘
```

#### 3.3.3 Datendiode / Schleusung

Für den Datenimport (z.B. Modell-Updates, Wissenspakete) wird eine
**Datendiode** oder ein **kontrolliertes Schleusungsverfahren** eingesetzt:

```
  Externes Netz                    Air-Gap                 Geheimhaltungsnetz
  ─────────────    ┌──────────┐    ╳╳╳╳╳╳    ┌─────────┐  ────────────────
                   │ Staging  │              │ Prüf-   │
  Modell-Update ──>│ Server   ├── Diode ────>│ station │──> EthosAI (Tier 3)
  Wissenspaket  ──>│ (extern) │  (nur →)     │ (intern)│
                   └──────────┘              └─────────┘
                                              4-Augen
                                              Malware-Scan
                                              Signatur-Check
```

---

## 4. Security-Tier-Konfigurationsmodell

### 4.1 Enum-Definition

```python
from enum import IntEnum

class SecurityTier(IntEnum):
    """Einsatz-Sicherheitsstufe der EthosAI-Instanz."""

    STANDARD = 1   # DSGVO, NDA, EU-Recht
    ELEVATED = 2   # BSI-Grundschutz, Behörden, Forschungslabs
    HIGHEST  = 3   # VS-NfD+, Air-Gap, HSM, Akkreditierung

    @property
    def display_name_de(self) -> str:
        return {1: "Standard", 2: "Erhöht", 3: "Höchste"}[self.value]

    @property
    def requires_on_premise(self) -> bool:
        return self.value >= 2

    @property
    def requires_air_gap(self) -> bool:
        return self.value >= 3

    @property
    def requires_hsm(self) -> bool:
        return self.value >= 3

    @property
    def allows_external_llm(self) -> bool:
        return self.value <= 1

    @property
    def audit_retention_years(self) -> int:
        return {1: 3, 2: 7, 3: 30}[self.value]
```

### 4.2 Tier-Policy-Engine

```python
@dataclass
class TierPolicy:
    """Definiert die konkreten Regeln für einen Security-Tier."""

    tier: SecurityTier

    # Kryptographie
    min_tls_version: str                 # "1.2" | "1.3"
    allowed_cipher_suites: list[str]     # BSI TR-02102-2 konform
    key_storage: str                     # "file" | "softhsm" | "hsm"
    encryption_algorithm: str            # "AES-256-GCM"

    # Authentifizierung
    auth_methods: list[str]              # ["jwt"] | ["jwt", "mfa"] | ["pki", "smartcard"]
    session_timeout_minutes: int         # 480 | 60 | 15
    max_failed_attempts: int             # 10 | 5 | 3

    # Netzwerk
    network_mode: str                    # "cloud" | "vpn" | "air-gap"
    allowed_outbound: list[str]          # ["*"] | ["llm.eu.provider"] | []

    # LLM Provider
    llm_mode: str                        # "any_eu" | "self_hosted" | "local_only"
    llm_data_residency: str              # "EU" | "on-premise" | "air-gap"

    # Audit
    audit_format: str                    # "jsonl" | "signed_jsonl" | "hsm_signed_worm"
    audit_retention_days: int            # 1095 | 2555 | 10950
    audit_tamper_protection: str         # "none" | "hmac" | "hsm"

    # Classification
    max_classification: ClassificationLevel  # CONFIDENTIAL | SECRET | SECRET
    auto_classify_uploads: bool          # True ab Tier 2

    # Export-Kontrolle
    export_approval_required: bool       # False | True | True (4-Augen)
    export_encryption_required: bool     # False | True | True

    # Data Destruction
    deletion_method: str                 # "delete" | "overwrite_3x" | "crypto_shred"
```

### 4.3 Vorkonfigurierte Tier-Policies

```python
TIER_POLICIES: dict[SecurityTier, TierPolicy] = {

    SecurityTier.STANDARD: TierPolicy(
        tier=SecurityTier.STANDARD,
        min_tls_version="1.2",
        allowed_cipher_suites=["TLS_AES_256_GCM_SHA384", "TLS_CHACHA20_POLY1305_SHA256"],
        key_storage="file",
        encryption_algorithm="AES-256-GCM",
        auth_methods=["jwt"],
        session_timeout_minutes=480,
        max_failed_attempts=10,
        network_mode="cloud",
        allowed_outbound=["*"],
        llm_mode="any_eu",
        llm_data_residency="EU",
        audit_format="jsonl",
        audit_retention_days=1095,
        audit_tamper_protection="none",
        max_classification=ClassificationLevel.CONFIDENTIAL,
        auto_classify_uploads=False,
        export_approval_required=False,
        export_encryption_required=False,
        deletion_method="delete",
    ),

    SecurityTier.ELEVATED: TierPolicy(
        tier=SecurityTier.ELEVATED,
        min_tls_version="1.3",
        allowed_cipher_suites=["TLS_AES_256_GCM_SHA384"],
        key_storage="softhsm",
        encryption_algorithm="AES-256-GCM",
        auth_methods=["jwt", "mfa"],
        session_timeout_minutes=60,
        max_failed_attempts=5,
        network_mode="vpn",
        allowed_outbound=["llm.self-hosted"],
        llm_mode="self_hosted",
        llm_data_residency="on-premise",
        audit_format="signed_jsonl",
        audit_retention_days=2555,
        audit_tamper_protection="hmac",
        max_classification=ClassificationLevel.SECRET,
        auto_classify_uploads=True,
        export_approval_required=True,
        export_encryption_required=True,
        deletion_method="overwrite_3x",
    ),

    SecurityTier.HIGHEST: TierPolicy(
        tier=SecurityTier.HIGHEST,
        min_tls_version="1.3",
        allowed_cipher_suites=["TLS_AES_256_GCM_SHA384"],
        key_storage="hsm",
        encryption_algorithm="AES-256-GCM",
        auth_methods=["pki", "smartcard"],
        session_timeout_minutes=15,
        max_failed_attempts=3,
        network_mode="air-gap",
        allowed_outbound=[],
        llm_mode="local_only",
        llm_data_residency="air-gap",
        audit_format="hsm_signed_worm",
        audit_retention_days=10950,
        audit_tamper_protection="hsm",
        max_classification=ClassificationLevel.SECRET,
        auto_classify_uploads=True,
        export_approval_required=True,
        export_encryption_required=True,
        deletion_method="crypto_shred",
    ),
}
```

---

## 5. Integration in bestehende Module

### 5.1 Mapping: ClassificationLevel ↔ SecurityTier

Das bestehende 4-Stufen-Klassifikationsmodell (PUBLIC/INTERNAL/CONFIDENTIAL/SECRET)
bleibt erhalten. Die Security-Tiers **beschränken**, welche Klassifikationsstufen
in welchem Deployment verwendet werden dürfen:

| ClassificationLevel | Tier 1 ✅/❌ | Tier 2 ✅/❌ | Tier 3 ✅/❌ |
|---------------------|-------------|-------------|-------------|
| PUBLIC | ✅ | ✅ | ✅ |
| INTERNAL | ✅ | ✅ | ✅ |
| CONFIDENTIAL | ✅ | ✅ | ✅ |
| SECRET | ❌ | ✅ | ✅ |

> **Regel:** SECRET-Daten dürfen nur in Tier 2+ verarbeitet werden.
> In Tier 1 erzeugt der Versuch, SECRET-Daten zu speichern, einen
> `SecurityTierViolation`-Error.

### 5.2 Need-to-Know ↔ Security-Tiers

| Need-to-Know-Feature | Tier 1 | Tier 2 | Tier 3 |
|----------------------|--------|--------|--------|
| ProjectNamespace Isolation | Logisch (DB-Trennung) | Physisch (separate Volumes) | Air-Gap (separater Rechner) |
| NDA Registry | JSON-basiert | Signiert + verschlüsselt | HSM-signiert |
| Information Barriers | Software-enforced | + Netzwerk-Segmentierung | + physische Trennung |
| Audit Trail | File-basiert | HMAC-signiert, 7 Jahre | HSM-signiert, WORM, 30 Jahre |

### 5.3 Configuration-File-Erweiterung

```yaml
# ethosai.yaml — Security-Tier-Konfiguration
security:
  tier: 1  # 1 = Standard, 2 = Erhöht, 3 = Höchste

  # Tier-spezifische Overrides (optional)
  overrides:
    session_timeout_minutes: 120  # Abweichung von Tier-Default

  # LLM-Provider-Einschränkung
  llm_policy:
    allowed_providers:
      - provider_id: "mistral-self-hosted"
        jurisdiction: "DE"
        data_residency: "on-premise"

  # Audit-Konfiguration
  audit:
    storage_path: "/secure/audit/"
    signing_key: "softhsm:slot0:audit-key"
    retention_days: 2555

  # Export-Kontrolle
  export:
    approval_required: true
    approvers:
      - "admin:security-officer"
      - "admin:project-lead"
```

---

## 6. Compliance-Matrix

### 6.1 DSGVO-Artikel ↔ Tier-Abdeckung

| DSGVO-Artikel | Beschreibung | Tier 1 | Tier 2 | Tier 3 |
|---------------|-------------|--------|--------|--------|
| **Art. 5** | Grundsätze der Verarbeitung | ✅ | ✅ | ✅ |
| **Art. 6** | Rechtsgrundlage | ✅ Config | ✅ | ✅ |
| **Art. 17** | Recht auf Löschung | ✅ Delete | ✅ 3x-Overwrite | ✅ Crypto-Shred |
| **Art. 20** | Datenübertragbarkeit | ✅ Export | ✅ Encrypt-Export | ✅ 4-Augen-Export |
| **Art. 22** | Keine automatisierte Entscheidung | ✅ Human-in-the-Loop | ✅ | ✅ |
| **Art. 25** | Privacy by Design | ✅ | ✅ | ✅ |
| **Art. 30** | Verarbeitungsverzeichnis | ✅ Audit-Log | ✅ Signiert | ✅ HSM-WORM |
| **Art. 32** | Technische Maßnahmen | TLS+RBAC | +mTLS+VPN+MFA | +HSM+Air-Gap |
| **Art. 33** | Meldepflicht (72h) | ✅ Prozess | ✅ NIS2-konform | ✅ + BSI-Meldung |
| **Art. 35** | Datenschutz-Folgenabschätzung | Optional | ✅ Pflicht | ✅ Pflicht |
| **Art. 44–49** | Drittland-Transfer | EU-only Policy | Kein Transfer | Air-Gap |

### 6.2 BSI-Grundschutz-Bausteine ↔ Tier-Mapping

| BSI-Baustein | Beschreibung | Tier 1 | Tier 2 | Tier 3 |
|-------------|-------------|--------|--------|--------|
| **ORP.1** | Organisation | 🔲 | ✅ | ✅ |
| **ORP.4** | Identitäts-/Berechtigungsmanagement | ✅ RBAC | ✅ +MFA/mTLS | ✅ +PKI |
| **CON.1** | Kryptokonzept | ✅ TLS | ✅ +BSI TR-02102 | ✅ +HSM |
| **CON.2** | Datenschutz | ✅ DSGVO | ✅ +DSFA | ✅ +Geheimschutz |
| **OPS.1.1.3** | Patch-Management | 🔲 Manual | ✅ Managed | ✅ +Signiert |
| **OPS.1.1.5** | Protokollierung | ✅ Log | ✅ +SIEM | ✅ +WORM |
| **OPS.1.2.5** | Fernwartung | ✅ SSH/VPN | ✅ VPN-only | ❌ Kein Fernzugriff |
| **SYS.1.1** | Allgemeiner Server | ✅ | ✅ Gehärtet | ✅ +TEMPEST |
| **NET.1.1** | Netzarchitektur | ✅ | ✅ VPN-Segmentiert | ✅ Air-Gap |
| **DER.1** | Detektion | 🔲 | ✅ IDS/SIEM | ✅ + lokales IDS |
| **DER.2.1** | Incident Management | 🔲 | ✅ NIS2-Prozess | ✅ + BSI-gemeldet |

---

## 7. Neue Module (Implementierungs-Roadmap)

### 7.1 Übersicht

```
ethos_ai/security/
├── classification_level.py      # ✅ Existiert (v1.5.0)
├── project_namespace.py         # ✅ Existiert (v1.5.0)
├── nda_registry.py              # ✅ Existiert (v1.5.0)
├── information_barrier.py       # ✅ Existiert (v1.5.0)
├── sandbox.py                   # ✅ Existiert (v1.3.0)
│
├── security_tier.py             # 🔲 NEU — SecurityTier Enum + TierPolicy
├── tier_policy_engine.py        # 🔲 NEU — Policy-Enforcement
├── llm_provider_policy.py       # 🔲 NEU — LLM-Zugangssteuerung pro Tier
├── key_store.py                 # 🔲 NEU — PKCS#11 / SoftHSM Adapter
├── hsm_adapter.py               # 🔲 NEU — HSM-Integration (Tier 3)
├── worm_audit.py                # 🔲 NEU — WORM-Audit-Trail (Tier 3)
├── export_control.py            # 🔲 NEU — 4-Augen-Export-Kontrolle
├── crypto_shred.py              # 🔲 NEU — Kryptographische Löschung
├── retention.py                 # 🔲 NEU — Automatisierte Retention-Policy
└── mfa/                         # 🔲 NEU — Multi-Faktor-Authentifizierung
    ├── __init__.py
    ├── totp.py                  # TOTP (RFC 6238)
    └── fido2.py                 # FIDO2/WebAuthn
```

### 7.2 Priorisierung

| Phase | Module | Tier | Sprint-Ziel |
|-------|--------|------|-------------|
| **Phase 1** | `security_tier.py`, `tier_policy_engine.py` | 1–3 | Tier-Enum + Policy-Engine |
| **Phase 2** | `llm_provider_policy.py`, `retention.py` | 2+ | Provider-Einschränkung + Löschlogik |
| **Phase 3** | `mfa/`, `key_store.py` | 2+ | MFA + SoftHSM-Anbindung |
| **Phase 4** | `export_control.py`, `crypto_shred.py` | 2–3 | Export-Kontrolle + Krypto-Löschung |
| **Phase 5** | `hsm_adapter.py`, `worm_audit.py` | 3 | HSM + WORM (nur für zertifizierte Umgebungen) |

---

## 8. Tier-Erkennung zur Laufzeit

Das aktive Security-Tier wird beim Start aus der Konfiguration gelesen und
als **globaler Singleton** bereitgestellt:

```python
class SecurityTierManager:
    """Zentrale Tier-Verwaltung — Singleton."""

    _instance: ClassVar[SecurityTierManager | None] = None
    _tier: SecurityTier
    _policy: TierPolicy
    _locked: bool = False

    @classmethod
    def initialize(cls, config_path: str) -> SecurityTierManager:
        """Liest den Tier aus ethosai.yaml und setzt die Policy."""
        ...

    @property
    def tier(self) -> SecurityTier:
        return self._tier

    @property
    def policy(self) -> TierPolicy:
        return self._policy

    def enforce(self, operation: str, context: dict) -> None:
        """Prüft ob die Operation im aktuellen Tier erlaubt ist.

        Raises SecurityTierViolation bei Verstoß.
        """
        ...

    def can_process_classification(self, level: ClassificationLevel) -> bool:
        """Prüft ob der aktuelle Tier diese Klassifikation verarbeiten darf."""
        if level == ClassificationLevel.SECRET and self._tier < SecurityTier.ELEVATED:
            return False
        return True
```

---

## 9. Interaktion mit der Internet-Research-Toolbox

Die Security-Tiers bestimmen, wie die Internet-Research-Toolbox operiert:

| Aspekt | Tier 1 | Tier 2 | Tier 3 |
|--------|--------|--------|--------|
| Web-Suche erlaubt? | ✅ Cloud + Self-Host | ✅ Nur Self-Host (VPN) | ❌ Keine Internet-Suche |
| Such-Provider | FOSS oder EU-kommerziell | Nur self-hosted FOSS | N/A (Offline-Wissenspaket) |
| Suchanfragen-Routing | TLS zum Provider | VPN + Tor-Circuit | N/A |
| Ergebnis-Cache | Encrypted at rest | + zugriffsbeschränkt | N/A |
| Offline-Modus | Optional | ✅ Fallback | ✅ Einziger Modus |
| Wissenspaket-Import | Automatisch | Signiert + geprüft | Diode + 4-Augen |

> **Tier 3 Konsequenz:** In der höchsten Sicherheitsstufe gibt es **keine
> Internet-Recherche**. Stattdessen wird über **signierte Wissenspakete**
> recherchiert, die über die Datendiode importiert wurden. Die Bias-Analyse
> erfolgt trotzdem — auf den importierten Inhalten.

---

## 10. Nicht-Ziele (Abgrenzung)

| Nicht-Ziel | Begründung |
|-----------|-----------|
| **BSI-Zertifizierung durchführen** | Dieses Konzept beschreibt die Architektur — die Zertifizierung ist ein separater Organisationsprozess |
| **HSM-Hardware beschaffen** | Hardware-Beschaffung ist Aufgabe des Betreibers; EthosAI stellt die Software-Adapter bereit |
| **TEMPEST-Spezifikation** | Physischer Schutz ist Infrastruktur-Aufgabe, nicht Software-Aufgabe |
| **Geheimschutzbetreuung** | Organisatorische Maßnahme des Betreibers |
| **Eigene PKI betreiben** | EthosAI nutzt eine vorhandene PKI; es wird keine eigene CA betrieben |

---

## 11. Offene Entscheidungen

| # | Frage | Optionen | Empfehlung |
|---|-------|---------|------------|
| OE-1 | Tier-Wechsel zur Laufzeit? | Ja (Downgrade verboten), Nein (Neuinstallation) | Nein — Tier wird bei Installation festgelegt, Upgrade per Migration |
| OE-2 | Tier im CLIM-Modell? | Tier beeinflusst Verhalten, Tier ist nur technisch | Tier als CLIM-Kontext — beeinflusst z.B. Auskunftsbereitschaft |
| OE-3 | SoftHSM für Tier 2 oder echtes HSM? | SoftHSM (kostenfrei), HSM-only | SoftHSM als Default, echtes HSM optional (PKCS#11 kompatibel) |
| OE-4 | Audit-Trail-Format? | JSON Lines, Protocol Buffers, SQLite | Signed JSONL — universell, lesbar, signierbar |
| OE-5 | WORM-Storage für Tier 3? | Software-WORM (immutable files), Hardware-WORM (Spezialdatenträger) | Software-WORM als Default, Hardware-WORM als Betreiber-Option |

---

## 12. Referenzen

| Quelle | Typ | Relevanz |
|--------|-----|----------|
| DSGVO (Verordnung EU 2016/679) | EU-Recht | Datenschutz, alle Tiers |
| NIS2-Richtlinie (EU 2022/2555) | EU-Recht | Cybersicherheit, Tier 2+ |
| EU AI Act (EU 2024/1689) | EU-Recht | Transparenzpflicht |
| BSI IT-Grundschutz-Kompendium | DE-Standard | Bausteine für Tier 2+ |
| BSI TR-02102-2 | DE-Standard | TLS-Cipher-Suites |
| BSI TR-03116-4 | DE-Standard | Smartcard-Auth (Tier 3) |
| BSI SiSyPHuS | DE-Projekt | OS-Härtung |
| VS-Anweisung (VSA) | DE-Verwaltung | Verschlusssachen-Handling (Tier 3) |
| ISO/IEC 27001:2022 | International | ISMS-Standard (Tier 2+) |
| Common Criteria EAL4+ | International | Zertifizierungsniveau (Tier 3) |
| FIPS 140-2 Level 3 | US/International | HSM-Zertifizierung |
| NATO SDIP-27 (AMSG-788) | NATO | TEMPEST-Standards |
| EthosAI Security Architecture | Intern | Vorgänger (v1.2.0) |
| EthosAI Need-to-Know Architecture | Intern | Vorgänger (v1.5.0) |
| EthosAI Internet Research Toolbox — Konzept | Intern | Nutzt Security-Tiers |

---

*Dieses Konzept bildet die Grundlage für alle sicherheitsrelevanten
Architekturentscheidungen in EthosAI. Es wird als lebendes Dokument
gepflegt und vor Implementierung pro Phase reviewt.*
