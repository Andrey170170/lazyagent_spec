# Architecture diagram

```mermaid
flowchart LR
  User([User]) --> UI[TUI/CLI]
  User --> Extension[VS Code/Cursor Extension]
  UI --> Daemon[Local Orchestrator/Daemon]
  Extension --> Daemon

  subgraph Core[Core Services]
    Registry[(Workspace Registry)]
    VariantMgr[Variant Manager]
    SessionMgr[Session Manager]
    ContextMgr[Context Manager]
    CapabilityMgr[Capability Manager]
    RunExec[Run Executor]
    MergePlan[Agentic Merge Planner]
  end

  subgraph Storage[Local Storage]
    MetaStore[(.agentplane metadata)]
    RunLogs[(Run logs & artifacts)]
    ContextStore[(Context packs & facts)]
    SkillStore[(Skills & profiles)]
  end

  subgraph Git[Git Layer]
    Worktrees[git worktrees]
    PatchBundles[Patch bundles]
  end

  subgraph Envs[Runtime Environments]
    Native[Native]
    subgraph Container[Containerized Variant]
      Sidecar[Sidecar Bridge]
      ToolPTY[Tool Session (OpenCode/Cursor CLI)]
    end
    Compose[Docker Compose]
  end

  subgraph Adapters[Tool Adapters]
    OpenCode[OpenCode adapter]
    Cursor[Cursor adapter]
  end

  UI --> Registry
  UI --> ContextMgr
  UI --> MergePlan
  UI --> RunExec
  UI --> CapabilityMgr
  UI --> VariantMgr
  UI --> SessionMgr

  Daemon --> Registry
  Daemon --> VariantMgr
  Daemon --> SessionMgr
  Daemon --> ContextMgr
  Daemon --> CapabilityMgr
  Daemon --> RunExec
  Daemon --> MergePlan

  Registry --> MetaStore
  ContextMgr --> ContextStore
  CapabilityMgr --> SkillStore
  RunExec --> RunLogs

  VariantMgr --> Worktrees
  MergePlan --> PatchBundles
  PatchBundles --> Worktrees

  RunExec --> Native
  RunExec --> Container
  RunExec --> Compose

  SessionMgr --> Sidecar
  Sidecar <--> ToolPTY

  CapabilityMgr --> OpenCode
  CapabilityMgr --> Cursor
  OpenCode --> Worktrees
  Cursor --> Worktrees
```

Notes
- The daemon can be optional for early MVP; CLI can call services directly.
- Adapters export into tool-specific configs in the repo (opencode.json, .cursor/rules).
- Context packs and merge plans are first-class artifacts stored locally.
