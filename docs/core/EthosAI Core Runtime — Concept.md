# EthosAI Core Runtime — Concept

> Defines the minimal `ethos_ai` subset that any EthosAI product
> (Worlds, future products) needs to run standalone.

| Field       | Value                |
|-------------|----------------------|
| Status      | Living               |
| Since       | v1.82.0              |
| Author      | EthosAI Architecture |
| Date        | 2026-03-22           |
| Backlog     | (internal tracking), (internal tracking) |
| Releases    | v1.93.0 (Phase 1), v1.94.0 (Phase 2) |

---

## 1  Motivation

EthosAI CLIM is a monorepo that contains multiple products:

| Product            | Package              | Nature             |
|--------------------|----------------------|--------------------|
| **EthosAI CLIM**   | `ethos_ai`           | AI-Mitbürger, CLIM Stack, Advisor, Training, Engineering |
| **EthosAI Worlds** | `ethos_ai_worlds`    | 3-D-Welten, Workbench, Marketplace, Standalone Server |
| *(Future)*         | *tbd*                | Further standalone products |

`ethos_ai_worlds` depends on a small subset of `ethos_ai` for
infrastructure concerns: authentication, runtime detection, security
enums, and the FastAPI application factory.  The full `ethos_ai`
package (170+ modules, ML models, LLM Router, SimGrid, Professions,
CLIM Stack) is **never** shipped with Worlds — it is a separate
system, a separate citizen, a separate product.

Today the bundle script (build script (build_worlds_bundle.py)) copies the
required files as a **Core-Subset** into the delivery archive.  This
works, but the subset is defined implicitly (a file list in the build
script) and grows ad-hoc.

**Goal:** Extract a well-defined `ethos_ai-core` runtime layer that
can be maintained, versioned, and delivered independently.

---

## 2  Core Runtime — Scope

The Core Runtime contains **infrastructure only**, no business logic:

### 2.1  Modules

| Sub-Package            | Module                    | Responsibility                     | LOC  |
|------------------------|---------------------------|------------------------------------|------|
| `ethos_ai.api`         | `auth.py`                 | JWT auth, Role enum, UserStore, require_role | ~250 |
| `ethos_ai.api`         | `app_factory.py`          | FastAPI middleware, static-file serving, mount helpers | ~450 |
| `ethos_ai.api`         | `models.py`               | Pydantic models (LoginRequest, TokenResponse, HealthResponse, UserResponse) | ~100 |
| `ethos_ai.runtime`     | `container.py`            | Container detection, RuntimeInfo, EthosAIMode | ~220 |
| `ethos_ai.security`    | `classification_level.py` | ClassificationLevel enum (4-tier data classification) | ~130 |
| `ethos_ai.security`    | `security_tier.py`        | SecurityTier IntEnum, TierPolicy, TIER_POLICIES | ~130 |
| `ethos_ai.security`    | `customer_folder.py`      | get_customers_dir(), ClientMeta, ProjectMeta | ~100 |
| *Total*                |                           |                                    | **~1 380** |

### 2.2  External Dependencies

The Core Runtime requires only lightweight, well-maintained packages:

| Package          | Purpose               |
|------------------|-----------------------|
| `fastapi`        | Web framework         |
| `uvicorn`        | ASGI server           |
| `pydantic`       | Data validation       |
| `python-jose`    | JWT encode/decode     |
| `bcrypt`         | Password hashing      |
| `slowapi`        | Rate limiting         |
| `starlette`      | HTTP middleware        |

No ML frameworks, no numpy, no torch, no transformers.

### 2.3  Additionally Required: Toolboxes Subset

| Sub-Package                     | Module            | Responsibility    | LOC |
|---------------------------------|-------------------|-------------------|-----|
| `toolboxes.world_framework`     | `world_packs.py`  | WorldManager (pack discovery, CRUD) | ~300 |

This is needed by `app_factory.mount_worlds()`.  It has zero
`ethos_ai` imports itself.

---

## 3  Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                  EthosAI CLIM (full)                 │
│  Advisor · Training · Engineering · SimGrid · CLIM   │
│  Security V2 (Router, Policy Engine, WORM, mTLS, …) │
│  Professions · Consciousness · Maturity Model        │
│  ┌───────────────────────────────────────────────┐   │
│  │ ★  ethos_ai-core  (Runtime Layer)             │   │
│  │    auth · app_factory · models                │   │
│  │    runtime/container                          │   │
│  │    security/{tier, classification, customer}  │   │
│  └──────────────────────┬────────────────────────┘   │
└─────────────────────────┼────────────────────────────┘
                          │  imported by
┌─────────────────────────┼────────────────────────────┐
│  EthosAI Worlds         │                            │
│    api/ (standalone_app, routers)                     │
│    ui/  (SPA, Three.js, components)                  │
│    toolbox-packages/ (WorldManager)                  │
└──────────────────────────────────────────────────────┘
```

---

## 4  Current Implementation (v1.82.0)

**Phase 0 — Core-Subset via Bundle Script**

The build script build script (build_worlds_bundle.py) defines
`ETHOSAI_CORE_FILES` — a list of individual files copied into
`app/ethos_ai/` in the delivery archive.  This preserves the import
structure (`from ethos_ai.api.auth import Role`) without any code
changes in `ethos_ai_worlds`.

The `security_router` import in `app_factory.mount_worlds()` is
wrapped in `try/except ImportError` so it gracefully degrades when the
full CLIM is not present.

**Advantages:**
- Zero code duplication in the repository
- Zero import changes in `ethos_ai_worlds`
- Bundle script is the single source of truth for what ships

**Limitations:**
- No independent versioning of the core layer
- File list must be maintained manually when core modules change
- No pip-installable artifact yet

---

## 5  Roadmap — Phases 1–3

### Phase 1 — Formal Package Boundary (Sprint: v1.93.0) ✅

- [x] Create `core/` package sub-package with `__init__.py` that
      re-exports all core symbols
- [x] Add a `core_manifest.json` listing all core modules (machine
      readable), consumed by the bundle script
- [x] Automated test: verify no core module imports from non-core
      `ethos_ai` modules
- [x] Bundle script reads from `core_manifest.json` (single source
      of truth) with hardcoded fallback

### Phase 2 — Pip-Installable Core (Sprint: v1.94.0) ✅

- [x] Separate `pyproject.toml` for `ethos_ai-core` (or namespace
      package `ethos_ai.core`)
- [x] Publish to private PyPI / artifact store
- [x] `ethos_ai_worlds/pyproject.toml` declares
      `dependencies = ["ethos_ai-core >= 1.93"]` (standalone extra)
- [x] Bundle script simplified: `pip install ethos_ai-core` in venv

### Phase 3 — Core Runtime Contract & Public API (Sprint: TBD)

- [ ] Stability guarantee: Core API is versioned with semver
- [ ] Breaking changes require major version bump
- [ ] Core-only CI pipeline (fast, no ML deps)
- [ ] Integration contract tests between `ethos_ai-core` and
      `ethos_ai_worlds`
- [ ] Define **public API surface**: which symbols from
      `ethos_ai.core` are stable/public vs `_internal`
- [ ] Generate `py.typed` marker + stub files for downstream type
      checking
- [ ] Publish release notes per core version (CHANGELOG-core.md)

---

## 6  Design Principles

1. **Infrastructure only** — no business logic, no domain knowledge.
2. **Minimal surface** — expose the smallest possible API.
3. **No ML dependencies** — the core must install in < 10 seconds.
4. **Backward compatible** — all existing `from ethos_ai.api.*`
   imports continue to work unchanged.
5. **Single source of truth** — core code lives in `ethos_ai/`, never
   duplicated into `ethos_ai_worlds/`.

---

## 7  Open Questions (Design)

| # | Question | Status |
|---|----------|--------|
| Q1 | Should `models.py` be trimmed to only the 4 models needed by Worlds? | Open |
| Q2 | Should `world_framework-toolbox/` move into `ethos_ai_worlds/`? | Open |
| Q3 | Namespace package (`ethos_ai.core`) vs. separate top-level (`ethosai_core`)? | **Resolved** — `ethos_ai-core` distribution installs into the `ethos_ai` namespace, preserving all existing import paths.  Build script in build script (build_core_package.py) assembles core files from `core_manifest.json`. |
| Q4 | Does `app_factory.py` (450 LOC) need slimming — e.g. WebSocket code into Worlds? | Open |

---

## 8  Multi-Repo Strategy Integration

The marketing strategy defines a **3 + 1 public-repo split**:

| Public Repository          | License    | Content sourced from Core Runtime? |
|----------------------------|------------|------------------------------------|
| `ethos-ai-concepts`        | CC-BY 4.0  | No — concept docs only             |
| `ethos-ai-world-sdk`       | MPL-2.0    | **Yes** — ABC interfaces, Manifest schema, MCP tool-schemas |
| `ethos-ai-eu-compliance`   | CC-BY 4.0  | No — compliance docs only          |
| *(Private)* `ethos_ai_clim`| MPL-2.0    | Full Core Runtime (proprietary layer) |

### 8.1  SDK Extraction Plan

For `ethos-ai-world-sdk` the following artifacts must be extracted
**from** the Core Runtime:

| Artifact                       | Source in core_manifest.json          | Notes |
|--------------------------------|---------------------------------------|-------|
| ABC interfaces (`WorldBase`, …)| `toolbox-packages/world_framework/world_packs.py` | Abstract base classes only |
| Manifest JSON-Schema           | `core_manifest.json`                  | Machine-readable core module list |
| MCP Tool schema stubs          | `tool/` package                      | Public tool descriptions, no impl |
| Pydantic request/response      | `api/models` module             | LoginRequest, HealthResponse, etc. |

The extraction script (future) reads `core_manifest.json` and produces
a slim `ethos_ai_sdk` wheel with interfaces only — no implementation
code.

### 8.2  Boundary Rule

> **Nothing from the `ethos_ai-core` wheel that is classified as
> 🔴 vertraulich may appear in a public repository.**
>
> The build pipeline must verify this invariant before any public
> release.

---

## 9  IP & Licensing

### 9.1  Classification of Core Components

Per the IP classification scheme:

| Component                        | IP Class             | Distribution Channel    |
|----------------------------------|----------------------|-------------------------|
| Core Runtime (`ethos_ai-core`)   | 🔴 Vertraulich       | Private PyPI / bundle   |
| Core Concept (this document)     | 🟡 Intern-nur-Doku   | Internal only           |
| ABC interfaces (SDK extract)     | 🟢 Öffentlich         | `ethos-ai-world-sdk`    |
| Manifest JSON-Schema             | 🟢 Öffentlich         | `ethos-ai-world-sdk`    |
| Pydantic models (API surface)    | 🔵 Open-Source        | `ethos-ai-world-sdk`    |

### 9.2  Dual-Licensing Prerequisites

Before the Core Runtime can be offered under a commercial license
(Enterprise tier), the following must be in place:

1. **CLA signed** by all contributors (currently sole author)
2. **License headers** in every core source file:
   `SPDX-License-Identifier: MPL-2.0` with commercial-option notice
3. **Trademark notice** in README and wheel metadata:
   `EthosAI™ is a registered trademark of Robert Alexander Massinger`
4. **Bereinigung** (sanitization) of this concept document:
   - Remove internal backlog references ((internal tracking))
   - Remove absolute file paths
   - Remove internal LOC counts from public-facing docs
   - Keep only the public API surface description

### 9.3  Bereinigung Compliance Checklist

| Rule (from IP-Klassifikation §4) | Status in this doc | Action needed |
|----------------------------------|--------------------|---------------|
| No internal ticket IDs in public docs | ⚠ (internal tracking) in §1 table | Remove before public release |
| No absolute file paths           | ✅ Relative paths only | — |
| No internal LOC counts           | ⚠ LOC column in §2.1 | Strip from public version |
| License header in source files   | ⬜ Not yet added   | Phase 3 task |
| CLA on file                      | ⬜ Sole author     | File before first external contributor |

---

## 10  Open Questions (Strategy & IP)

| # | Question | Status |
|---|----------|--------|
| Q5 | When should the SDK extraction script be built — Phase 3 or separate sprint? | Open |
| Q6 | Should SPDX headers be added via automated script or manual pass? | Open |

---

*This document is maintained alongside the codebase.  Updates happen
in the sprint that implements the next phase.*
