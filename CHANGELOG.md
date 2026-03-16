# Changelog — Haner AI

## [1.0.0] — 2024

### Core
- AIAgent coordinator with subsystem auto-discovery
- AIBlackboard shared key-value store
- AIStateMachine: 14 states with Enter/Execute/Exit
- AIRegistry singleton for scene-wide agent and source tracking

### Behaviour Tree
- BTPrioritySelector: re-evaluates every tick, auto-aborts lower-priority branches
- BTSelector with AbortType.LowerPriority support
- BTSequence with AbortType.Self support
- BTParallel with configurable success/fail policy
- BTGuard decorator: aborts child when condition turns false
- BTCooldown, BTTimeout, BTInverter, BTSucceeder, BTRepeater decorators
- BTAction with OnAbort callback
- BTWait, BTCheckBB, BTSetBB, BTClearBB leaves
- BTVisualizer ASCII tree printer
- Pre-built trees: Infantry, Sniper, Scout

### Perception
- Vision: configurable FOV cone, peripheral zone, multi-height LOS (3 rays)
- Hearing: ISoundSource interface, loudness × range × suppression multiplier
- Scent: IScentSource interface, time-decay, wind direction modifier
- Touch: proximity overlap trigger
- Awareness Score 0–100 with AlertPhase transitions
- Per-target memory with time-decay

### Emotion
- Morale, Fear, Anger — each 0–100, decay to configurable baseline
- Accuracy, fire rate, speed multipliers derived from emotion state
- Panic mode, rage mode, morale break events
- Squad message hooks (AllyDown, EnemyDown, Reinforce, FallBack)

### Squad
- 6 roles: Leader, Assault, Support, Sniper, Scout, Medic
- Dynamic leader election on leader death
- 12 squad message types
- Cohesion radius maintenance
- Flanking position calculation (left/right)
- Pincer movement check

### Combat
- Burst fire with burst/pause cycle
- Suppression fire (spray at LKP)
- Cover evaluation scoring (distance, LOS block, normal angle, quality)
- Peek & shoot while in cover
- Grenade throwing (static target detection)
- Flanking with squad coordination
- Retreat with safe-position calculation
- Spiral search pattern generation

### Movement
- NavMeshMovement: full pathfinding, per-mode speed, obstacle avoidance
- RigidbodyMovement: physics-based, freeze-rotation, per-mode speed

### World Interaction
- IInteractable interface for doors, buttons, ladders
- Vault detection for low obstacles
- Interaction coroutine with duration support

### LOD
- 4 distance tiers: High / Medium / Low / Culled
- NavMesh obstacle avoidance quality scales with LOD
- Player auto-discovery via tag

### Health
- Hit zone multipliers (Head, Body, Limb)
- Armour reduction coefficient
- Regen with delay
- Optional ragdoll on death
- RecentDamageNormalized for cover trigger

### Utility
- AISoundEmitter: ISoundSource implementation, duration, suppressor flag
- AIScentEmitter: IScentSource implementation, time-decay
- CoverPoint: scoring data, occupancy, Gizmo visualization
