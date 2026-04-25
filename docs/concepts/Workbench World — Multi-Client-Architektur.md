---
title: "Workbench World — Multi-Client-Architektur & Daten-Synchronisation"
brief: "Architekturkonzept für zwei Kernszenarien: (A) Client-Zugriff auf Kundendaten unabhängig vom Speicherort, (B) Multi-User/Multi-EthosAI Collaboration in gemeinsamen Workbench/World Sessions. Ansatz: Client-seitiges Rendering + Modell-Daten-Synchronisation."
status: draft
version: "0.1.0"
author: Robert Alexander Massinger
date: 2026-03-09
tags: [workbench, architecture, multi-user, collaboration, sync, data-access, websocket]
---
# Workbench World — Multi-Client-Architektur & Daten-Synchronisation

## 0 · Entscheidung

> **Grundsatz:** Rendering geschieht **ausschließlich auf den Clients** (WebGL/Three.js).
> Synchronisation erfolgt über **Modell-Daten** (JSON-Deltas), **nicht** über gerenderte Video-Streams.
> Bei Multi-Client-Sessions gibt es einen **Host-Client**, an den sich Gäste anhängen.

Begründung:
- Kein Server-GPU nötig → läuft auf jeder EthosAI-Instanz (Laptop, VM, Raspberry Pi)
- Bandbreite: JSON-Deltas ~1 KB/s vs. Video-Stream ~5 MB/s
- Jeder Client hat eigene Kamera/Viewport → unabhängige Exploration
- Skaliert auf N Clients ohne Server-Rendering-Bottleneck
- FPV-Feeds bleiben lokal (kein Privacy-Risiko durch Server-seitige Kamerabilder)

---

## 1 · Szenario A: Wo liegen die Daten?

### 1.1 Drei Speicherorte

```
┌─────────────────────────────────────────────────────┐
│  Variante 1: Daten auf dem Client (lokal)           │
│                                                     │
│  Client ←→ server:PORT ←→ customers/             │
│  (EthosAI + Daten auf demselben Rechner)            │
│                                                     │
│  → Trivial: Alles lokal, kein Netzwerk nötig        │
├─────────────────────────────────────────────────────┤
│  Variante 2: Daten auf Netzlaufwerk (NAS/SMB)      │
│                                                     │
│  Client ←→ server:PORT ←→ <network-share>/projects │
│  (EthosAI lokal, Daten remote gemountet)            │
│                                                     │
│  → .env: ETHOSAI_CUSTOMERS_DIR=<network-share>/projects │
│  → Latenz bei STL-Load, Caching empfohlen           │
├─────────────────────────────────────────────────────┤
│  Variante 3: Daten nur auf dem Server               │
│                                                     │
│  Client (Browser) ──HTTP──→ Server:PORT ──→ storage/ │
│  (Kein lokaler Zugriff, alles über API)             │
│                                                     │
│  → Standard-Web-Architektur                         │
│  → STLs werden über GET /file/ gestreamt            │
└─────────────────────────────────────────────────────┘
```

### 1.2 Lösung: Transparente Daten-Pipeline

Der Client braucht STL-Geometrien zum Rendern. Egal wo die Daten physisch liegen, der Client lädt sie immer über **dieselbe API**:

```
GET /api/workbench/projects/{project}/file/{path}
```

**Was sich ändert, ist nur die Server-Konfiguration:**

| Speicherort | Server-Config | Client-Änderung |
|-------------|--------------|-----------------|
| Lokal | `ETHOSAI_CUSTOMERS_DIR=./customers` (Default) | Keine |
| Netzlaufwerk | `ETHOSAI_CUSTOMERS_DIR=<network-share>/projects` | Keine |
| Nur Server | Default (Server hat die Daten) | Keine |

### 1.3 Caching-Schicht

STL-Dateien ändern sich selten. Ein Cache verhindert wiederholtes Laden über langsame Netzlaufwerke:

```
┌──────────┐     ┌─────────────────┐     ┌──────────────────┐
│  Client   │────→│  EthosAI Server │────→│  Daten-Backend   │
│  (Browser)│     │                 │     │  (lokal/NAS/S3)  │
│           │     │  STL-Cache      │     │                  │
│  IndexedDB│     │  (LRU, 500 MB) │     │                  │
│  (Browser)│     │  ETag/304       │     │                  │
└──────────┘     └─────────────────┘     └──────────────────┘
```

**Zwei Cache-Ebenen:**

1. **Server-Cache** (LRU, konfigurierbar)
   - `ETHOSAI_STL_CACHE_SIZE_MB=500`
   - SHA-256 Hash als ETag → `304 Not Modified` bei unveränderter Datei
   - Besonders relevant bei Netzlaufwerk-Variante

2. **Client-Cache** (Browser IndexedDB)
   - STL-Geometrien nach dem Parsen als serialisierte BufferGeometry speichern
   - ETag aus Server-Response als Cache-Key
   - Erspart erneutes Parsen bei Revisit

### 1.4 Lazy Loading für große Projekte

Nicht alle 14 STLs sofort laden, sondern:

1. **Manifest zuerst** — `GET /stl-list` liefert Namen + Größen
2. **LOD-Stufen** — Low-Poly-Hülle (Bounding-Box) sofort, Detailgeometrie on-demand
3. **Priorisierung** — sichtbare Parts zuerst (Frustum-Culling auf Manifest-Ebene)
4. **Progressive Load** — große STLs (>1 MB) chunked streamen

---

## 2 · Szenario B: Multi-User / Multi-EthosAI Sessions

### 2.1 Session-Modell

```
┌────────────────────────────────────────────────────────────────┐
│                    Workbench Session                            │
│                                                                │
│  ┌──────────────────┐   WebSocket    ┌──────────────────────┐  │
│  │  Host-Client     │◄─────────────►│  EthosAI Server       │  │
│  │  (Session-Owner) │   /ws/wb      │  (Session-Broker)     │  │
│  │  ✦ hat die Daten │               │  ✦ leitet Deltas      │  │
│  │  ✦ rendert lokal │               │  ✦ verwaltet State    │  │
│  └──────────────────┘               │  ✦ authentifiziert    │  │
│                                     └──────┬───────────────┘  │
│                                            │                   │
│                          ┌─────────────────┼────────────────┐  │
│                          │                 │                │  │
│                ┌─────────▼──┐    ┌─────────▼──┐   ┌────────▼─┐│
│                │ Guest A    │    │ Guest B    │   │ EthosAI  ││
│                │ (Browser)  │    │ (Browser)  │   │ Agent    ││
│                │ rendert    │    │ rendert    │   │ (API)    ││
│                │ lokal      │    │ lokal      │   │ headless ││
│                └────────────┘    └────────────┘   └──────────┘│
└────────────────────────────────────────────────────────────────┘
```

**Rollen:**
- **Host-Client** — Erstellt die Session, bestimmt das Projekt, hat Master-State
- **Guest-Client** — Tritt bei, bekommt den kompletten State, synchronisiert Deltas
- **EthosAI-Agent** — Kann als headless Teilnehmer beitreten (API-only, kein Rendering)

### 2.2 Synchronisations-Protokoll

Keine Video-Streams. Stattdessen: **JSON-Delta-Messages über WebSocket.**

```
Client → Server → All Clients
```

**Message-Typen:**

```jsonc
// 1. Session Lifecycle
{ "type": "wb:session:create",  "session_id": "...", "project": "...", "host": "P1" }
{ "type": "wb:session:join",    "session_id": "...", "user": "P2" }
{ "type": "wb:session:leave",   "session_id": "...", "user": "P2" }
{ "type": "wb:session:close",   "session_id": "..." }

// 2. State Sync (Initial)
{ "type": "wb:state:full",      "parts": [...], "avatars": [...], "cameras": [...], "annotations": [...] }

// 3. Deltas (laufend)
{ "type": "wb:avatar:move",     "name": "P1", "x": 500, "y": 0, "z": -200, "ts": 1741500000 }
{ "type": "wb:avatar:emote",    "name": "P1", "emote": "point" }
{ "type": "wb:avatar:say",      "name": "P1", "text": "Hier ist das Problem" }
{ "type": "wb:camera:update",   "user": "P2", "pos": [8000,4000,6000], "target": [0,0,0] }
{ "type": "wb:annotation:add",  "id": "A-001", "part": "cockpit", "pos": [100,200,50], "text": "Spalt zu eng" }
{ "type": "wb:part:visibility",  "name": "hull_main", "visible": false }
{ "type": "wb:animation:play",  "preset": "FM Landing", "time": 0.0 }
{ "type": "wb:animation:seek",  "time": 6.5 }
{ "type": "wb:ruler:measure",   "from": [0,0,0], "to": [500,0,0], "value": 500.0 }
{ "type": "wb:clipping:update", "plane": "X", "value": 0.5 }

// 4. FPV (Metadaten, nicht Pixel!)
{ "type": "wb:fpv:activate",    "name": "P1", "active": true }
{ "type": "wb:fpv:capture",     "name": "P1", "filename": "fpv_P1_cockpit.png" }
```

**Bandbreite:** ~50–200 Byte/Message × ~10 Messages/s = **~1–2 KB/s** (vs. ~5 MB/s für 720p Video)

### 2.3 State-Management

```
┌─────────────────────────────────────────────────┐
│  Session State (Server, authoritative)          │
│                                                 │
│  ┌─────────────────────────────────────────────┐│
│  │ project: "customer-a/front-module/evol-1-2" ││
│  │ host: "P1-Commander"                        ││
│  │ participants: ["P1", "P2", "EthosAI-Agent"] ││
│  │ avatars: { P1: {x,y,z,emote}, P2: {...} }  ││
│  │ annotations: [ {id, part, pos, text} ]      ││
│  │ part_visibility: { hull_main: true, ... }   ││
│  │ animation_state: { preset, time, playing }  ││
│  │ clipping_planes: { X: 0.5, Y: null, Z: null}││
│  │ cursor_ghosts: { P2: {pos, target} }        ││
│  └─────────────────────────────────────────────┘│
│                                                 │
│  → Full State wird bei JOIN gesendet            │
│  → Danach nur Deltas                            │
│  → Server = Single Source of Truth              │
│  → Konflikt-Auflösung: Last-Write-Wins + ts    │
└─────────────────────────────────────────────────┘
```

### 2.4 Daten-Verteilung bei Multi-User

Wenn die Daten nur auf dem Host liegen:

```
Option A: Server-Relay (bevorzugt)
─────────────────────────────────
Host hat Daten → Server cached → Guests laden via API

  Host ──upload──→ Server (STL-Cache) ←──GET /file/──→ Guests

  ✦ Host exportiert STLs per Toolbox
  ✦ Server macht sie über /api/workbench/projects/... verfügbar
  ✦ Guests laden dieselben STLs
  ✦ Kein Peer-to-Peer nötig


Option B: Host-Proxy (Fallback)
──────────────────────────────
Host dient als Proxy für seine lokalen Dateien

  Guest ──request──→ Server ──relay──→ Host-Client ──file──→ response
                                       (WebSocket binary frame)

  ✦ Komplex, nur wenn Server keinen Dateizugriff hat
  ✦ Peer-to-Peer über WebRTC DataChannel als alternative
```

**Empfehlung:** Option A (Server-Relay) für v1. Option B nur bei Edge-Cases.

### 2.5 EthosAI-Agent als Session-Teilnehmer

Ein EthosAI-Agent (z.B. Review-Ingenieur) kann headless an einer Session teilnehmen:

```python
# Agent-seitig (Python)
async with websockets.connect("ws://server:PORT/ws/wb") as ws:
    # Session beitreten
    await ws.send(json.dumps({
        "type": "wb:session:join",
        "session_id": session_id,
        "user": "EthosAI-ReviewBot"
    }))

    # State empfangen
    state = json.loads(await ws.recv())

    # Analyse durchführen → Annotation setzen
    await ws.send(json.dumps({
        "type": "wb:annotation:add",
        "id": f"AI-{uuid4().hex[:8]}",
        "part": "cockpit",
        "pos": [120, 350, -50],
        "text": "⚠️ Kopffreiheit 95. Perzentil: 12mm — unter NASA-STD-3001 Minimum (25mm)",
        "severity": "warning",
        "source": "EthosAI-ReviewBot"
    }))
```

---

## 3 · WebSocket-Architektur

### 3.1 Bestehende Infrastruktur

EthosAI hat bereits einen `WebSocketManager` ([websocket_manager.py](api/websocket_manager module)):
- `connect()`, `disconnect()`, `broadcast()`, `send_personal()`
- Aktiver Endpoint: `ws://server:PORT/ws`

### 3.2 Erweiterung: Workbench-WebSocket

Neuer Endpoint für Workbench-Sessions:

```
ws://server:PORT/ws/wb?session={session_id}&user={user_name}
```

**Separater Channel**, damit Workbench-Traffic nicht mit dem Advisor-Chat kollidiert.

```python
# workbench_router.py Erweiterung
@router.websocket("/ws/wb")
async def workbench_ws(ws: WebSocket, session: str = Query(...), user: str = Query(...)):
    await wb_session_manager.connect(session, user, ws)
    try:
        while True:
            data = json.loads(await ws.receive_text())
            await wb_session_manager.handle_message(session, user, data)
    except WebSocketDisconnect:
        wb_session_manager.disconnect(session, user)
```

### 3.3 Session-Manager (Backend)

```python
class WorkbenchSessionManager:
    sessions: dict[str, WorkbenchSession]

    async def create_session(self, session_id, project, host, ws) → WorkbenchSession
    async def join_session(self, session_id, user, ws) → full_state
    async def handle_message(self, session_id, user, data) → broadcast_delta
    async def disconnect(self, session_id, user) → notify_others
    async def close_session(self, session_id) → cleanup
```

---

## 4 · FPV in Multi-User-Kontext

### 4.1 Aktueller Stand

FPV ist **rein client-seitig** (Three.js `WebGLRenderTarget` → Canvas → PNG). Kein Backend-Endpoint.

### 4.2 Multi-User FPV

Im Multi-User-Kontext braucht **kein** FPV-Pixel-Stream übers Netz:

- Jeder Client rendert seinen eigenen FPV-Feed lokal
- Die **Kameraposition des Avatars** wird synchronisiert (JSON-Delta)
- Jeder Client kann den FPV eines anderen Avatars **lokal nachbilden** (gleiche Kamera-Parameter → gleiches Bild)

```jsonc
// Avatar-Eye-Kamera-Sync (nicht Pixel, sondern Parameter!)
{
  "type": "wb:fpv:camera",
  "name": "P1",
  "eye": [120, 1650, -450],      // Augenposition
  "lookAt": [120, 1650, -1000],  // Blickrichtung
  "fov": 90,
  "aspect": 1.333
}
```

**Ergebnis:** Jeder Client rendert das gleiche Bild, weil die Szene + Kameraparameter synchron sind. Null Bandbreite für Pixel.

### 4.3 FPV-Capture als Befund

Wenn ein Benutzer einen FPV-Screenshot als Befund erfasst:

```jsonc
{
  "type": "wb:fpv:capture",
  "name": "P1",
  "filename": "fpv_P1_cockpit_clearance.png",
  "annotation_id": "A-003"
}
```

Jeder Client kann den Screenshot **lokal rendern** (identische Szene + Kamera) oder der auslösende Client lädt das PNG per `POST /api/workbench/session/{id}/findings` hoch.

---

## 5 · Zusammenfassung der Architektur-Entscheidungen

| Entscheidung | Gewählt | Begründung |
|-------------|---------|------------|
| Rendering | Client-seitig (WebGL) | Keine Server-GPU, skaliert, jeder hat eigenen Viewport |
| Synchronisation | JSON-Deltas über WebSocket | ~1 KB/s statt ~5 MB/s Video-Stream |
| Daten-Zugriff | Einheitliche REST-API | Client merkt nicht, ob Daten lokal/NAS/Server liegen |
| Caching | Server-LRU + Client-IndexedDB | Netzlaufwerk-Latenz kompensieren |
| Session-Topologie | Host + Guests über Server-Broker | Einfacher als Peer-to-Peer, NAT-sicher |
| FPV-Sync | Kamera-Parameter, nicht Pixel | Bandbreite = 0 für FPV |
| EthosAI-Agent | Headless WebSocket-Client | Kann Annotationen setzen, Analysen liefern |
| Konflikt-Auflösung | Last-Write-Wins + Timestamp | Einfach, ausreichend für Review-Sessions |

---

## 6 · Implementierungs-Roadmap

| Phase | Arbeitspaket | Aufwand |
|-------|-------------|---------|
| **Phase 1** | `ETHOSAI_CUSTOMERS_DIR` .env-konfigurierbar | 0.5 Sprint |
| **Phase 1** | ETag/304 Caching für `/file/` Endpoint | 0.5 Sprint |
| **Phase 1** | Client-IndexedDB STL-Cache | 1 Sprint |
| **Phase 2** | `ws://server:PORT/ws/wb` Endpoint | 1 Sprint |
| **Phase 2** | `WorkbenchSessionManager` (create/join/leave) | 1 Sprint |
| **Phase 2** | Client-JS: Session-Join + Delta-Sync | 1.5 Sprint |
| **Phase 2** | Avatar/Camera Ghost-Cursors | 0.5 Sprint |
| **Phase 3** | Annotation-Sync + Findings-Upload | 1 Sprint |
| **Phase 3** | EthosAI-Agent als headless Teilnehmer | 1 Sprint |
| **Phase 3** | FPV-Camera-Parameter-Sync | 0.5 Sprint |

**Gesamt: ~8.5 Sprints** (Phase 1: ~2, Phase 2: ~4, Phase 3: ~2.5)
