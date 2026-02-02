# Architecture diagram

```mermaid
flowchart LR
  U[User]
  CLI[CLI primary]
  UI[UI client optional]
  Daemon[Local orchestrator]

  U --> CLI
  U --> UI
  CLI --> Daemon
  UI --> Daemon

  subgraph Core
    CoreHub[Core services]
    VariantMgr[Variants]
    RunExec[Runs]
    SessionMgr[Sessions]
    ContextMgr[Context packs]
    MemoryMgr[Memory]
    CapabilityMgr[Capabilities]
    MergePlan[Merge plans]
    PortMgr[Ports]
  end

  subgraph Storage
    Store[lazyagent store]
  end

  subgraph Git
    Worktrees[Worktrees]
    Patches[Patch bundles]
  end

  subgraph Envs
    Native[Native]
    Container[Container sidecar]
  end

  subgraph Tools
    Tool[Agent tools]
  end

  Daemon --> CoreHub
  CoreHub --> Store
  VariantMgr --> Worktrees
  MergePlan --> Patches
  Patches --> Worktrees
  RunExec --> Native
  RunExec --> Container
  SessionMgr --> Container
  Container --> Tool
  PortMgr --> RunExec
  CapabilityMgr --> Worktrees
  ContextMgr --> Tool
  MemoryMgr --> Tool
```

Notes
- The daemon is the primary source of truth; CLI is the primary client, UI optional later.
- Adapters export into tool-specific configs in the repo (opencode.json, .cursor/rules).
- Context packs and merge plans are first-class artifacts stored locally.
