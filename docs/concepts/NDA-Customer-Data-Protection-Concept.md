---
title: "Non-Disclosure & Customer Data Protection — Module Concept"
brief: >
  Architecture for isolating customer/client project data from the Git
  repository and from external LLM providers (Mistral Cloud, OpenAI, etc.).
  Covers folder structure, .env configuration, encryption, .gitignore,
  and data-routing rules for current & future AI-driven toolboxes.
status: draft
version: "0.1.0"
author: Robert Alexander Massinger
date: 2026-03-02
tags: [non-disclosure, data-protection, customers, encryption, llm-routing, security]
notes: >
  ToDo - check all data exports, all training data generation, all training data pools 
---

# Non-Disclosure & Customer Data Protection — Module Concept

## 1. Motivation

EthosAI processes data on behalf of customers and clients.  Some of
this data is subject to **Non-Disclosure Agreements (NDA)**, trade-secret
protection, or regulatory requirements (e.g. GDPR, EU AI Act).

**Key principles:**

1. Customer/client data **MUST NOT** be committed to the Git repository.
2. NDA-protected data **MUST NOT** be sent to external LLM providers
   (Mistral API, OpenAI, Anthropic, etc.).
3. Each customer/client gets an **isolated folder hierarchy**.
4. The storage root is **configurable via `.env`** and may reside outside
   the project tree (e.g. an encrypted volume).
5. Future toolboxes (including AI-driven ones that generate network
   traffic) must respect these data-routing rules.

---

## 2. Folder Structure

```
<ETHOSAI_CUSTOMERS_DIR>/          # configurable root  (default: ./customers/)
├── .encryption_policy.json       # repo-wide encryption settings
├── <client-slug>/                # one folder per customer / client
│   ├── .client_meta.json         # client metadata (contact, NDA ref, …)
│   ├── <project-slug>/           # one subfolder per project / engagement
│   │   ├── .project_meta.json    # project metadata (NDA level, dates, …)
│   │   ├── input/                # raw input files from the client
│   │   ├── output/               # generated artefacts
│   │   ├── models/               # fine-tuned or client-specific models
│   │   └── logs/                 # audit logs for this project
│   └── …
└── …
```

### Naming convention

| Element | Pattern | Example |
|---------|---------|---------|
| Client slug | `lowercase-kebab` | `acme-corp` |
| Project slug | `lowercase-kebab` | `bridge-inspection-2026` |

---

## 3. `.env` Configuration

```dotenv
# ── Customer Data Location ────────────────────────────────────────
# Absolute or relative path to the customer data root.
# Default: ./customers  (relative to project root)
ETHOSAI_CUSTOMERS_DIR=./customers

# ── Encryption ────────────────────────────────────────────────────
# Require at-rest encryption for all customer data.
# "required" | "optional" | "off"
ETHOSAI_CUSTOMERS_ENCRYPTION=required

# Encryption backend:  "fernet" (symmetric, built-in) |
#                       "age"   (modern PGP-like, requires `age` CLI) |
#                       "gpg"   (GnuPG)
ETHOSAI_CUSTOMERS_ENCRYPTION_BACKEND=fernet

# Key file or key-ring reference (backend-specific).
# For fernet: path to a 32-byte base64-encoded key file.
ETHOSAI_CUSTOMERS_KEY_FILE=./keys/customers.key

# ── LLM Data Routing ─────────────────────────────────────────────
# Comma-separated list of LLM provider IDs that are approved for
# NDA-protected data.  Only LOCAL providers should be listed here.
# Providers: "ollama", "llamacpp", "vllm" (all local).
# Cloud providers like "mistral-api", "openai", "anthropic" are
# NEVER added here by default.
ETHOSAI_NDA_APPROVED_PROVIDERS=ollama

# If true, block ANY outbound request that would carry customer
# data to a non-approved provider.  Acts as a safety net.
ETHOSAI_NDA_STRICT_MODE=true
```

---

## 4. `.gitignore` Integration

The root `.gitignore` already contains:

```gitignore
# Customer data (NDA — construction details MUST NOT be public)
customers/
```

**Additional rules** (to be added if the customers dir is relocated):

```gitignore
# Customer data — catch common alternative locations
customer-files/
client-files/
nda-files/
```

The `.env` file itself is also ignored (`*.env*`), ensuring secrets and
customer-folder paths are not leaked.

---

## 5. LLM Data-Routing Architecture

### 5.1 Routing Matrix

| Data classification | Local LLM (Ollama) | External LLM (Cloud) |
|---------------------|--------------------|-----------------------|
| **Public / Open** | ✅ Allowed | ✅ Allowed |
| **Internal** | ✅ Allowed | ⚠️ Allowed with disclaimer |
| **NDA-protected** | ✅ Allowed | ❌ **BLOCKED** |
| **Classified** | ✅ Allowed (encrypted at rest) | ❌ **BLOCKED** |

### 5.2 Enforcement Points

```
 User / Toolbox
       │
       ▼
 ┌─────────────────────┐
 │   DataClassifier    │  ← tags each request payload with a
 │   (middleware)      │     classification level
 └──────────┬──────────┘
            │
            ▼
 ┌─────────────────────┐
 │   LLMRouterGuard    │  ← checks classification against
 │   (pre-send hook)   │     ETHOSAI_NDA_APPROVED_PROVIDERS
 └──────────┬──────────┘
            │
     ┌──────┴──────┐
     ▼             ▼
  Ollama       Cloud API
  (local)      (blocked if NDA)
```

### 5.3 DataClassifier Rules

1. Any payload that references a file under `ETHOSAI_CUSTOMERS_DIR`
   is classified **NDA-protected** (minimum).
2. Per-project metadata (`.project_meta.json`) can override the level
   to **Classified**.
3. Conversations that include customer data snippets inherit the
   highest classification of their constituent parts.

### 5.4 LLMRouterGuard

- Sits inside the `CLIMProvider` abstraction layer.
- Before any `generate()` / `chat()` call, inspects the payload
  classification.
- If the target provider is NOT in `ETHOSAI_NDA_APPROVED_PROVIDERS`
  and the classification is ≥ NDA-protected → **raise
  `DataRoutingViolation`** and log the event.
- In `ETHOSAI_NDA_STRICT_MODE=true`, the request is hard-blocked.
  In `false` mode, a warning is logged but the user is prompted for
  explicit consent (future UX).

---

## 6. Encryption Requirements

### 6.1 At-Rest Encryption

| Backend | Pros | Cons |
|---------|------|------|
| **Fernet** (cryptography lib) | Built-in, no external tool | Symmetric key management |
| **age** | Modern, simple CLI | Requires `age` binary |
| **GPG** | Industry standard, key-ring | Complex key management |

Default: **Fernet** — zero external dependencies beyond `cryptography`
(already in requirements).

### 6.2 Key Management

- Key file stored in `keys/customers.key` (already git-ignored).
- Key rotation: create new key, re-encrypt files, archive old key.
- Future: integrate with Keystore module (v1.34.0 (internal tracking)).

### 6.3 In-Transit Encryption

- Local LLM (Ollama): loopback only — no network encryption needed.
- If Ollama runs on a remote host: TLS required (`--ssl-*` flags).
- Cloud LLMs: always HTTPS (enforced by provider SDKs).

---

## 7. Future Toolbox Considerations

As EthosAI grows, **third-party and AI-driven toolboxes** will generate
data traffic that may touch customer information:

### 7.1 Toolbox Network Policy

Each toolbox declares in its `toolbox_manifest.json`:

```json
{
  "name": "robotics",
  "version": "1.0.0",
  "network_policy": {
    "outbound_allowed": false,
    "endpoints": [],
    "data_classification_max": "internal"
  }
}
```

- `outbound_allowed`: whether the toolbox may initiate HTTP(S) calls.
- `endpoints`: allow-listed URLs (if outbound is true).
- `data_classification_max`: highest data level the toolbox may handle.
  If a toolbox declares `"internal"`, it is **blocked** from accessing
  restricted customer folders.

### 7.2 AI-Driven Toolbox Guard

Toolboxes that use an LLM internally (e.g. a "Smart Inspection Report
Generator") **must** route through the same `LLMRouterGuard`:

```
Toolbox.execute()
    → CLIMProvider.generate(payload, classification)
        → LLMRouterGuard.check(provider, classification)
            → allow / block
```

No toolbox may bypass the guard by calling an LLM directly.

### 7.3 Audit Log

Every data access and LLM call involving customer data is logged to:

```
<ETHOSAI_CUSTOMERS_DIR>/<client>/<project>/logs/access.jsonl
```

Fields: `timestamp`, `actor` (user / toolbox / agent), `action`,
`classification`, `provider`, `verdict` (allow / block).

---

## 8. Implementation Roadmap

| Phase | Scope | Target |
|-------|-------|--------|
| **Phase 1** | Folder structure, `.env` config, `.gitignore`, DataClassifier skeleton | v1.34.0 |
| **Phase 2** | LLMRouterGuard integration in CLIMProvider, strict-mode enforcement | v1.35.0 |
| **Phase 3** | At-rest encryption (Fernet), key rotation CLI | v1.36.0 |
| **Phase 4** | Toolbox network policy manifest, audit logging | v1.37.0 |
| **Phase 5** | UI for data classification & customer project management | v1.38.0 |

---

## 9. Open Questions

1. Should customer slugs be UUIDs instead of readable names (for extra
   anonymisation)?
2. Do we need per-user ACLs on customer folders (multi-advisor scenario)?
3. Should the encryption key be derived from a passphrase (PBKDF2) or
   stored as a raw key file?

---

*This concept is a living document. Update it as implementation progresses.*
