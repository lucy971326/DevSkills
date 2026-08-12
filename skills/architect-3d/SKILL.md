---
name: architect-3d
description: >-
  Generate an interactive 3D architecture cognitive map from a codebase. NOT a
  dependency graph — a BIM-like spatial model showing functional domains,
  sub-modules, and multi-path data flows. Use when the user wants to visualize
  how a system works, understand data flow, or build a mental model of an
  unfamiliar codebase. Triggers on: "3D architecture", "架构认知地图",
  "可视化架构", "系统心智模型", "BIM 模型", "see the architecture in 3D".
---

# 3D Architecture Cognitive Map

Generate one HTML file that renders the system as an interactive 3D cognitive map. It loads Three.js from a CDN at runtime; serve it over HTTP. Think **BIM building model**, not UML diagram.

## Philosophy

The user wants to **see how the system works**, not a file dependency graph. Every node must carry semantic meaning. The 3D space is for spatial reasoning — proximity = functional affinity, vertical arrangement = nothing more than visual clarity.

**Core anti-patterns to avoid:**
- Nodes that represent files or packages (no "internal/webapi/agentrun.go" as a node)
- Single-node-per-package granularity (too coarse to understand)
- Textbook layered-architecture terminology forced onto a codebase that isn't layered
- Auto-flying camera on click (let the user control their own view)
- Automatic animation loops (let the user step through at their own pace)

## Methodology

### Phase 1: Read the Codebase Thoroughly

Read enough to understand the **runtime behavior**, not just the file structure. Focus on:

1. The entry point — how does the system start?
2. The main data path — what happens when a request comes in?
3. Service initialization — what gets created at startup?
4. External dependencies — what does the system call out to?
5. Persistence — where does data live?

Use any available tools (codegraph, grep, glob, read) to trace the full runtime flow. Do NOT guess — verify by reading the actual code.

Before modeling, collect a source anchor for each applicable category: entry point, initialization, primary flow, persistence, and external boundary. Record file paths and symbols; do not create a node or flow step that lacks an anchor.

### Phase 2: Build the Mental Model

Before writing any code, articulate:

- **System purpose**: What problem does this solve? One sentence.
- **Core flow**: What is the primary business process? Trace it end-to-end.
- **Functional domains**: Group related capabilities. NOT architectural layers — functional clusters based on what things DO.
- **Key design decisions**: What trade-offs were made? Where are the surprises?

Share this mental model and its source anchors with the user before generating the 3D output. Get confirmation when the task is exploratory or the model contains material uncertainty; otherwise state the assumptions and proceed.

### Phase 3: Identify Functional Domains

A functional domain is a group of things that work together, named by what they DO (not where they sit in a layer stack).

Good domain names:
- "HTTP 接口" (not "表示层")
- "运行调度" (not "编排协调层")
- "用户密钥库" (not "领域服务层")
- "沙箱执行" (not "基础设施层")

Aim for **4–6 domains**. Too many = fragmented. Too few = each domain too vague.

### Phase 4: Decompose into Rooms and Sub-modules

Each domain contains **rooms** (major capability clusters). Each room contains **sub-modules** (specific responsibilities within that capability).

Rules for sub-modules:
- Each must have a clear, one-sentence responsibility
- Named by WHAT it does, not HOW (e.g., "解码校验", not "json.NewDecoder")
- Group 3–8 sub-modules per room — fewer means the room is too granular, more means it should be split
- Sub-module spacing: at least 1.5 units apart in the 3D layout to avoid visual clutter

### Phase 5: Define Flow Paths

Each selected API endpoint or business process becomes an independent flow path. Each flow has numbered steps connecting specific sub-modules.

Rules for flow paths:
- Select flows that answer the user's question and expose distinct behavior. Prefer 3–8 flows; include more only when they add new decisions or data paths.
- Steps connect sub-modules (high precision), not rooms (too vague)
- Shared segments between flows are automatically deduplicated
- Each step has a short, descriptive label that tells the user what's happening
- Target 3–19 steps per flow

### Phase 6: Generate the 3D HTML

Output a single HTML file using Three.js from CDN (jsdelivr, version 0.160.0).

#### Data Structure

```javascript
const DOMAINS = [
  { id:'D1', name:'域名称', y:5, color:'#HEX' },
  // y = vertical position (spacing ~4 units between domains)
];

const rooms = [
  {
    id:'room-id', domain:'D1', cx:-6, cz:0, color:'#HEX',
    name:'房间名称', tag:'简短标签',
    summary:'一句话描述这个房间做什么',
    subs: [
      { id:'sub-id', rx:-2.8, rz:-1.6, name:'子模块名', desc:'一句话职责' },
      // rx, rz = position relative to room center cx, cz
    ],
    code:'实际代码位置 (用于详情卡片)'
  },
];

const externals = [
  { id:'bff', name:'外部系统名', desc:'描述', pos:[x,y,z], color:'#EF5350' },
];

const flows = {
  'flow-key': {
    name:'链路名',
    desc:'链路描述',
    steps: [
      { from:'sub-id', to:'sub-id', label:'1. 这一步做什么' },
    ],
  },
};
```

#### Visual Design

- **Room enclosures**: Semi-transparent boxes with thin border lines. The enclosure visually groups sub-modules.
- **Sub-modules**: Small cubes (~1.5×1.0×1.3 units) with emissive material matching the room color, plus edge lines.
- **Room labels**: CSS2D labels floating above the enclosure.
- **Sub-module labels**: Smaller CSS2D labels above each cube.
- **Domain floors**: Large transparent planes with subtle grid lines. Domain name labels at the corner.
- **Flow lines**: Tube geometry (radius ~0.055) with slight upward arc (QuadraticBezierCurve3). Default state: very dim (opacity 0.09). Selected flow: slightly visible (opacity 0.14). Current step: full brightness (opacity 1.0, white).
- **Particles**: Small spheres traveling along the current step's curve.
- **Externals**: Spheres with ring meshes at the top of the scene (Y ~15-19).

#### Interaction Design

1. **Free orbit** — OrbitControls with damping. No forced camera movements on click. Min/max distance for comfortable zoom range.
2. **Flow selector** — Tab buttons at the bottom-right. Click to switch active flow. Highlights the flow's lines, dims others.
3. **Step controls** — ◀/▶ buttons + step counter + current step label. Also keyboard: → next, ← prev, R reset, 1-8 switch flows.
4. **Click sub-module** — Show detail card (right side) with: sub-module description, parent room info, code location. Do NOT fly the camera.
5. **Hover sub-module** — Increase emissive intensity slightly. No tooltip (detail card on click is sufficient).
6. **Legend** — Bottom-left, shows domain colors with names.

#### Color Palette

Use distinct, saturated colors for domains so they're easily distinguishable in 3D:
- Blue family: `#4FC3F7` — HTTP/API surface
- Amber family: `#FFB74D` — Orchestration/coordination
- Green family: `#81C784` — Business capabilities
- Purple family: `#BA68C8` — Execution/runtime
- Gray family: `#90A4AE` — Data/infrastructure
- Red family: `#EF5350` — External systems

#### Three.js Setup Checklist

- Version 0.160.0 from jsdelivr CDN via import map
- CSS2DRenderer for labels (crisp at any zoom level)
- PCFSoftShadowMap for soft shadows
- ACESFilmicToneMapping for better color rendering
- DirectionalLight + AmbientLight + PointLight (fill)
- Scene fog for depth perception
- Starfield background (Points geometry) for spatial reference
- Responsive resize handler

## Design Principles (Discovered Through Iteration)

1. **Nodes represent business concepts, never files.** "用户密钥库", not "userconfig/store.go".
2. **Two-level granularity.** Rooms (big capability) + sub-modules (specific responsibilities). This is the sweet spot.
3. **Multiple independent flow paths.** Do not show only one "main flow"; select the few endpoints or processes that reveal distinct runtime behavior.
4. **Space encodes affinity, not hierarchy.** 3D position + color tell you "these things are related." Don't use vertical position to imply importance.
5. **Deduplicate shared connections.** Multiple flows often share the same segment — render it once, tag it as belonging to multiple flows.
6. **Never move the user's camera.** Free orbit, no auto-fly. Click shows info, doesn't reposition.
7. **Manual stepping, not auto-play.** Each flow step is triggered by the user pressing ▶. This gives them time to understand each transition.
8. **Each step has a narrative label.** "3. models.Lookup(modelID) → 验证 Vision", not "Step 3".
9. **Code locations in detail cards.** Every sub-module knows its source file. Clicking reveals the path.
10. **Evidence before rendering.** Every room, sub-module, and flow step maps to a source anchor. Share the mental model for confirmation when ambiguity makes it valuable.

## File Output

Write to a path like `<project-root>/architecture-3d.html`. Keep application data and UI in one HTML file; it loads Three.js modules from jsDelivr, so network access and an HTTP server are required. Typical file size: 400–600 lines of HTML.
