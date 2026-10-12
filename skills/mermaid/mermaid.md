---
name: Mermaid
description: Write and render Mermaid diagrams in chat. Use whenever the user asks for a diagram, flowchart, sequence diagram, class diagram, architecture sketch, gantt chart, timeline, or any other visual in Mermaid syntax.
---

# Mermaid - write correct, renderable diagrams

Current version: **12.1.0** (October 2026).
Docs (source of truth, one page per type): https://mermaid.ai/open-source/syntax/

Rule for rare or new diagram types: check the docs page first. Never invent syntax.

## OpenCode chat renderer (check this FIRST)

OpenCode's chat does NOT use the real Mermaid library. It uses a custom
terminal renderer: the `@opencode/merman` package (verified in the opencode
repo, `packages/merman/src/detect.ts`, v2.0.27). It draws Unicode art in the
TUI and recognizes EXACTLY these 6 diagram types:

| Renders in chat | Keyword |
|---|---|
| Flowchart | `flowchart` / `graph` |
| Sequence | `sequenceDiagram` |
| State | `stateDiagram` / `stateDiagram-v2` |
| Gantt | `gantt` |
| Git graph | `gitGraph` |
| Timeline | `timeline` |

Everything else (class, ER, pie, quadrant, journey, requirement, C4, mindmap,
sankey, xychart, block, packet, kanban, architecture, radar, venn, ishikawa,
wardley, cynefin, treemap, treeView, eventmodeling, swimlanes, railroad,
usecase, zenuml, ...) shows as PLAIN TEXT in chat. Proven by live test:
a 32-diagram sweep rendered only these 6 types; the other 26 were text.
(The full mermaid bundle inside the opencode binary is NOT the chat path.)

### Chat-safe syntax (verified in merman source, `packages/merman/src/`)

Flowchart:
- Shapes: `A[label]` box, `A(label)` rounded, `A([label])` stadium,
  `A{label}` decision, `A((label))` circle, `A[(label)]` database,
  `A[[label]]` subroutine.
- Exotic shapes (hexagon `{{}}`, trapezoid, parallelogram, double circle)
  are NOT supported and break the whole diagram.
- `subgraph` ... `end` (nestable), `direction TB|TD|BT|LR|RL`.
- Edge labels, `==`/`--` thickness, dashed `-.->`, cross ends `--x`/`-x`.
- `classDef` / `class` / `style` / `linkStyle` are ACCEPTED BUT IGNORED
  (terminal has no colors). Safe to include, no effect.

Sequence:
- `participant` / `actor` with `as`, message operators `->>`, `-->>`, `-->`,
  `->`, `--x`, `-x`, `--)`, `-)`, activation `+`/`-`.
- `alt` / `loop` blocks, `activate` / `deactivate`, `autonumber`.
- `Note right of X: text` rendered fine in a live test.

State:
- `stateDiagram-v2`, composite states `state X { ... }` (optional
  `"Label" as X`), transitions `A --> B : label`, `[*]`,
  `note left/right of X` (inline and block form).

Gantt / GitGraph / Timeline: standard official syntax (title, section,
dateFormat, `after`; commit/branch/checkout/merge; title + eras + events).

### Rules for chat output

1. If the idea fits one of the 6 types, draw it with that type.
2. If the user wants another type (class, ER, pie, ...), say in one line
   that OpenCode chat shows it as text, then provide the real Mermaid code
   (v12 syntax, for mermaid.live / GitHub) clearly separated from chat-safe art.
3. If the user wants the diagram as an image file, offer:
   `npx -y @mermaid-js/mermaid-cli -i in.mmd -o out.svg` (one-time install,
   needs a headless browser).
4. Never put unsupported syntax inside a chat diagram: merman throws
   `MermaidSyntaxError` on unknown lines and the WHOLE diagram falls back
   to text.

## Hard rules for chat output

1. Put every diagram in a ` ```mermaid ` code fence.
2. One statement per physical line. A second statement on the same line is a parse error.
3. Keep node labels short. Quote labels with spaces or special characters: `A["Login flow (v2)"]`.
4. Reserved characters in labels need entity codes: `#quot;` `#39;` `#40;` `#41;` `#91;` `#93;` `#96;`. Backslash never escapes anything.
5. Identifiers are `[A-Za-z0-9_]+`. Use an explicit id + display label when other statements must reference the node: `Login("Sign in")`.
6. If the render target shows an error box, read its message and fix the exact line it points to.

## What changed in v12 (breaking, September 2026)

- **ELK is the default layout** for flowchart, state, class, ER, requirement and use case diagrams. Diagrams look different from v11. Keep the old dagre look with front matter:

  ```
  ---
  config:
    layout: dagre
  ---
  ```

- **Default theme is `redux-color`, default look is `neo`** for flowchart, swimlane, class, ER, requirement, sequence, state, use case and venn. Old look: add `theme: default` and `look: classic` to the same front matter `config`.
- **`defaultRenderer` was removed.** Use the top-level `layout: dagre | elk` in front matter instead.
- A subgraph can pick its own ELK algorithm: `subgraph X @{ algorithm: elk.box }`. Allowed values: `elk.layered`, `elk.box`, `elk.rectpacking`, `elk.stress`, `elk.force`, `elk.mrtree`, `elk.radial`, `elk.sporeOverlap`.
- v12.1: `elk.orientFeedbackEdges` option (on by default); packet diagrams accept `bitOrder: ascending | descending`.
- The error box now shows the real parse error text (wrapped to 4 lines). Use it to self-correct.
- If the user embeds mermaid in a web app: v12 requires ES2024, Safari 17.4+, Node 22.12+.
- Do not declare the same subgraph id twice - it now merges into one subgraph under ELK.

## All diagram types (official v12.1.0 docs) and keywords

| Type | Keyword | Notes |
|---|---|---|
| Flowchart | `flowchart` (or `graph`) | subgraphs, classDef, direction |
| Swimlanes (v11.16.0+) | `swimlane-beta` (optionally + direction: `swimlane-beta LR`) | lanes = top-level subgraphs, nodes/edges use flowchart syntax |
| Sequence | `sequenceDiagram` | participant, actor menu `@{...}`, alt/loop/par/opt |
| Class | `classDiagram` | relations, multiplicity, stereotypes |
| State | `stateDiagram-v2` | composite states, transitions, notes |
| ER | `erDiagram` | relationships, attributes |
| User journey | `journey` | title + sections + tasks |
| Gantt | `gantt` | sections, tasks, `after`/`until` |
| Pie | `pie` | title + `: value` lines |
| Quadrant | `quadrantChart` | x-axis, y-axis, actors, lines |
| Requirement | `requirementDiagram` | requirement/functional/perf/interface, `contains`, `derives`, `satisfies`, `verifies`, `tests` |
| Use case (NEW in v12.0.0) | `usecase-beta` | see snippet below |
| Git graph | `gitGraph` | branch, commit, merge, checkout |
| C4 | `C4Context` `C4Container` `C4Component` `C4Dynamic` `C4Deployment` | person/system/rel, updateLayout, boundary |
| Mindmap | `mindmap` | indentation-based, root first |
| Timeline | `timeline` | Title, eras, events |
| ZenUML | zenuml | official plugin since v12 |
| Sankey (verified) | `sankey` | plain CSV, exactly 3 columns |
| XY chart (verified) | `xychart` or `xychart horizontal` | title, x-axis, y-axis, line/bar |
| Block | `block` | beta |
| Packet | `packet` | bit fields |
| Kanban | `kanban` | sections (columns) + tasks |
| Architecture | `architecture` | groups + services with icons |
| Radar | `radar` | axes + shape |
| Event modeling | `eventmodeling` | check docs page for details |
| Treemap, Ishikawa, Wardley, Cynefin, TreeView | check docs | new types, verify keyword before use |

## Verified snippets

Flowchart:

```
flowchart LR
    A[Start] --> B{Decision}
    B -->|yes| C[Do it]
    B -->|no| D[Stop]
    subgraph Group1
        C --> E[Done]
    end
    classDef hl fill:#f9f,stroke:#333
    class A,E hl
```

Swimlanes (verified keyword `swimlane-beta`, v11.16.0+ - does NOT render in
OpenCode chat, merman has no swimlanes module; use for mermaid.live/GitHub):

```
swimlane-beta LR
    subgraph User
        A[Create order] --> B[Pay]
    end
    subgraph Backend
        C[Verify stock] --> D[Ship]
    end
    B --> C
```

Facts: top-level subgraphs become lanes. Nodes and edges use flowchart syntax. Optional direction after the keyword: TB, TD, BT, LR, RL.

Sequence:

```
sequenceDiagram
    participant U as User
    participant A as API
    participant DB as Database
    U->>A: POST /login
    A->>DB: check user
    DB-->>A: ok
    A-->>U: 200 + token
    Note right of A: session stored
```

Class:

```
classDiagram
    class Animal {
        +string name
        +eat() void
    }
    class Dog {
        +bark() void
    }
    Animal <|-- Dog : inherits
```

State:

```
stateDiagram-v2
    [*] --> Idle
    Idle --> Running : start
    Running --> Paused : pause
    Running --> [*] : finish
```

ER:

```
erDiagram
    USER ||--o{ ORDER : places
    ORDER ||--{| ITEM : contains
    USER {
        string email PK
        string name
    }
```

Gantt:

```
gantt
    title Roadmap
    dateFormat YYYY-MM-DD
    section Phase 1
    Design  :a1, 2026-10-12, 7d
    Build   :a2, after a1, 14d
    section Phase 2
    Ship    :a3, after a2, 3d
```

Pie:

```
pie showData
    title Budget
    "Dev" : 40
    "Ops" : 35
    "Other" : 25
```

Git graph:

```
gitGraph
    commit
    branch feature
    checkout feature
    commit
    checkout main
    merge feature
```

Mindmap:

```
mindmap
  root((Project))
    Frontend
      React
      CSS
    Backend
      API
      DB
```

Journey:

```
journey
    title Checkout
    section Browse
      Find product: 5: me
    section Pay
      Enter card: 3: me
      Confirmed: 5: me
```

Timeline:

```
timeline
    title Project
    2026 Q4 : Kickoff : Design
    2027 Q1 : Build : Beta
```

Quadrant:

```
quadrantChart
    title Priorities
    x-axis Low --> High
    y-axis Low --> High
    "Feature A": [0.3, 0.6]
    "Feature B": [0.8, 0.9]
```

Use case (NEW in v12.0.0, keyword `usecase-beta`). Verified against the official docs source:

```
usecase-beta
    direction LR
    actor Customer
    actor Support
    systemBoundary Storefront
        Browse("Browse catalogue")
        Checkout("Checkout")
    end
    systemBoundary Fulfilment
        Track("Track delivery")
    end
    Customer --> Browse
    Customer --> Checkout
    Customer --> Track
    Support --> Track
    Checkout ..> : include Browse
    note for Checkout "Payment runs after stock check"
```

Facts (from the official v12 docs):
- There is NO `usecase` declaration keyword. Use cases are declared bare: `Login("Sign in")` = ellipse, `Report[Generate report]` = rectangle. Undeclared relationship endpoints become ellipse use cases automatically.
- Actors must be declared with `actor ID` (position never implies an actor).
- `A ..> : include B` and `A ..> : extend B` always need a target. A solid label containing "include"/"extend" is only an ordinary association.
- Generalization: `A --|> B` (specialized to general).
- Notes: `note for <target> "text"`. No placement keywords.
- Boundaries: `systemBoundary Id[Title]` or quoted title only (`systemBoundary "Order system"`), body indented, closed with `end`. One level deep. Optional `@{ type: package }` for a package tab.
- `direction TD | TB | BT | LR | RL` sets layout direction.
- Edge IDs: `Customer opens@-- "label" ---> Checkout` - the id before `@` can be styled/animated later.

Sankey (verified keyword `sankey`, plain CSV with exactly 3 columns: source, target, value):

```
sankey-beta
    Browser, Google, 200
    Browser, Bing, 100
    Google, Site A, 120
    Bing, Site A, 80
```

Note: if `sankey-beta` fails on the user's renderer, try `sankey` (v12 docs use `sankey`).

XY chart (verified keyword `xychart`):

```
xychart
    title "Monthly revenue"
    x-axis [Jan, Feb, Mar, Apr]
    y-axis "k EUR" 0 --> 50
    bar [12, 19, 30, 45]
    line [10, 15, 22, 40]
```

Facts: multi-word text values need quotes. `line "name" [...]` adds a legend entry (v11.17+). `xychart horizontal` flips axes.

Venn (verified keyword `venn-beta`):

```
venn-beta
    set A ["Cats"]
    set B ["Dogs"]
    union A B ["Pets"]
    text "Both" in A
```

Facts: identifiers in a union must be defined by earlier `set` lines. `:N` suffix sets size. `style A fill:#f9f` for colors.

Packet:

```
packet
    0-15 : source port
    16-31 : dest port
    32-63 : length
    64-95 : checksum
```

Kanban:

```
kanban
    section Backlog
        Write docs [assigned, gogo]
    section In Progress
        Build API [active]
    section Done
        Setup repo [done]
```

Architecture:

```
architecture
    group web(cloud)
        service api(api) in web
        service db(database) in web
        api:R -- L:db
```

Facts: icons come from the registered icon set (font awesome by default: `api`, `database`, `cloud`, `server`, `computer`, `security`).

C4:

```
C4Context
    title System context
    Person(user, "User")
    System(app, "Web App")
    Rel(user, app, "Uses")
```

## Common pitfalls

- Flowchart "edge limit exceeded": the v12.1 error box tells you this. Reduce edges or raise `maxEdges` via `mermaid.initialize`.
- State diagrams need `stateDiagram-v2`, not `stateDiagram`.
- gantt: task ids must be unique; referencing an unknown `after`/`until` id now warns (v12.1).
- sequence: whitespace between an actor name and its `@{...}` config is fine (v12.1 fix).
- C4: declare `Rel` only between declared elements; use `updateLayout($c4ShapeInRect="true")` to stop huge boxes.
- If a diagram must look exactly like v11, add the dagre/default/classic front matter block from the v12 section.
