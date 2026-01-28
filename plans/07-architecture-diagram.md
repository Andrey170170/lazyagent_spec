# Architecture diagram

```mermaid
flowchart LR
  User([User]) --> UI[TUI/CLI]
  UI --> Daemon[Local Orchestrator]

  subgraph Core[Core Services]
    VariantMgr[Variants]
    RunExec[Runs]
    SessionMgr[Sessions]
    ContextMgr[Context Packs]
    MemoryMgr[Memory]
    CapabilityMgr[Capabilities]
    MergePlan[Merge Plans]
    PortMgr[Ports]
  end

  subgraph Storage[Local Storage]
    Store[(.agentplane)]
  end

  subgraph Git[Git]
    Worktrees[Worktrees]
    Patches[Patch Bundles]
  end

  subgraph Envs[Environments]
    Native[Native]
    Container[Container + Sidecar]
  end

  subgraph Tools[Agent Tools]
    Tool[OpenCode/Cursor CLI]
  end

  Daemon --> Core
  Core --> Store
  VariantMgr --> Worktrees
  MergePlan --> Patches
  Patches --> Worktrees
  RunExec --> Envs
  SessionMgr --> Container
  Container --> Tool
  PortMgr --> RunExec
  CapabilityMgr --> Worktrees
  ContextMgr --> Tool
  MemoryMgr --> Tool
```

Notes
- The daemon is the primary source of truth; TUI/CLI connect as clients.
- Adapters export into tool-specific configs in the repo (opencode.json, .cursor/rules).
- Context packs and merge plans are first-class artifacts stored locally.
