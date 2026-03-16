# 🧠 Haner AI — Advanced Game AI System for Unity

[![Unity](https://img.shields.io/badge/Unity-2021.3%2B-black?logo=unity)](https://unity.com)
[![License](https://img.shields.io/badge/License-Commercial-red)](#license)
[![Asset Store](https://img.shields.io/badge/Asset%20Store-$59.99-orange)](https://assetstore.unity.com)
[![Publisher](https://img.shields.io/badge/Publisher-Haner%20Games-purple)](https://hanergames.com.tr)

> Production-ready, genre-agnostic AI framework for Unity. State machine + Behaviour Tree, 4-sense perception (vision/hearing/scent/touch), emotional states (morale/fear/anger), squad tactics, cover system, world interaction, LOD optimisation — zero third-party dependencies.

---

## 🏗 Architecture

```
AIAgent  (coordinator)
 ├── AIBlackboard          shared key-value data store
 ├── AIStateMachine        14-state FSM
 ├── BehaviourTree         optional BT layer (PrioritySelector + full abort)
 ├── AIPerception          vision · hearing · scent · touch
 ├── AIEmotionSystem       morale · fear · anger → tactical modifiers
 ├── AISquadController     roles · leader election · flanking · pincer
 ├── AICombatController    burst fire · cover scoring · grenade · search
 ├── IAIMovement           NavMesh or Rigidbody (switchable per-agent)
 ├── AIWorldInteraction    door open · vault · interactables
 └── AILODController       distance-based tick rate + NavMesh quality
```

---

## ✨ Feature Overview

### 🧠 State Machine — 14 States
`Idle` → `Patrol` → `Wander` → `Investigate` → `Alert` → `Combat` → `TakeCover` → `Suppress` → `Flank` → `Advance` → `Retreat` → `Search` → `Interact` → `Stunned` → `Dead`

Transitions are data-driven from a single evaluation function. No spaghetti state-to-state calls.

### 🌳 Behaviour Tree — Full Abort System

| Node | Description |
|------|-------------|
| `BTPrioritySelector` | Re-evaluates all children every tick — higher-priority branch preempts running lower one |
| `BTSelector` | Classic OR-selector with `AbortType.LowerPriority` support |
| `BTSequence` | Classic AND-sequence with `AbortType.Self` support |
| `BTParallel` | All children run simultaneously, configurable success/fail policy |
| `BTGuard` | Condition decorator — aborts child if condition turns false mid-execution |
| `BTCooldown` | Returns Failure if on cooldown |
| `BTTimeout` | Aborts child if it runs too long |
| `BTInverter` / `BTSucceeder` / `BTRepeater` | Standard decorators |

**Abort types:**
- `Self` — If a condition inside this composite turns false, abort own running child
- `LowerPriority` — If a higher-priority branch becomes valid, abort lower-priority running branches
- `Both` — Both of the above

**Example scenario:** Agent is in cover (`TakeCover` branch running). Player becomes visible → `BTPrioritySelector` re-evaluates on next tick → `Combat` branch condition passes and has higher priority → `TakeCover` branch is automatically aborted → agent switches to `Combat`. Zero manual interrupt code needed.

### 👁 4-Sense Perception

| Sense | Mechanism |
|-------|-----------|
| Vision | FOV cone + peripheral zone, multi-height LOS (3 rays), motion bonus, crouch penalty |
| Hearing | `ISoundSource` interface, range × loudness × suppression, sound tag filtering |
| Scent | `IScentSource` interface, time-decay intensity, wind direction modifier |
| Touch | Proximity overlap, instant trigger |

**Awareness Score** — Each stimulus contributes to a 0–100 accumulator. Score drives `AlertPhase`: `Unaware → Suspicious → Alerted → Combat`. Awareness decays over time when no stimulus is present.

### 😤 Emotional System

| Emotion | Range | Effect |
|---------|-------|--------|
| Morale  | 0–100 | Low → prefer cover/retreat. High → advance/flank. |
| Fear    | 0–100 | High → accuracy penalty, panic fire, seek cover |
| Anger   | 0–100 | High → accuracy bonus, ignore cover, aggressive advance |

Emotions respond to: taking damage, ally death, enemy kill, grenade explosion, leader death, reinforcements arriving, prolonged combat. Each emotion decays back to a configurable baseline.

### 🗣 Squad System — 6 Roles

| Role | Behaviour |
|------|-----------|
| Leader | Elected dynamically when previous leader dies |
| Assault | Close-range aggression |
| Support | Suppression, ally cover |
| Sniper | Long-range, defensive positioning |
| Scout | Fast, flanking-focused |
| Medic | Slower movement, stays near allies |

Messages: `EnemySpotted`, `EnemyDown`, `AllyDown`, `NeedSupport`, `FallBack`, `Reinforce`, `HoldPosition`, `AdvanceOn`, `FlankLeft`, `FlankRight`, `TakeCover`, `ClearArea`

### 🛡 Cover System
- `CoverPoint` component — place anywhere in scene
- Scored by: distance, threat distance, cover normal vs threat direction, LOS blocking quality
- Peek & shoot — expose at intervals, fire burst, return to cover
- Occupancy tracking — enemies don't stack on same cover point

### ⚡ LOD Controller

| Distance | Tick Rate | NavMesh Quality |
|----------|-----------|-----------------|
| < 20 m   | 10 Hz     | High quality obstacle avoidance |
| < 45 m   | 5 Hz      | Standard |
| < 80 m   | 2 Hz      | No obstacle avoidance |
| > 80 m   | 0.5 Hz    | Minimal |

### 🚪 World Interaction
- Door open/close via `IInteractable` interface
- Vault detection — low obstacle ahead → trigger vault animation
- Button, ladder, any world object via `InteractionTag`

### 🏃 Dual Movement Backend
- `NavMeshMovement` (default) — full NavMesh pathfinding
- `RigidbodyMovement` — physics-based for arcade/platformer projects
- Switch per-agent via Inspector `movementBackend` field

---

## 🚀 Quick Start

### 1. Import
Copy `Assets/HanerAI/` into your project.

### 2. Build Demo Scene
```
Haner AI → 🤖 Build Demo Scene
```

### 3. Bake NavMesh
```
Window → AI → Navigation → Bake
```

### 4. Play ▶

---

## 📋 API Reference

### AIAgent
```csharp
void  EngageTarget(GameObject target)       // Force combat engagement
void  LoseTarget()                          // Drop target, retain LKP
void  ForceState(AIState state)             // Jump to any state
void  SetStance(Stance stance)              // Stand / Crouch / Prone
void  Stun(float duration)                  // Temporary stun
void  Teleport(Vector3 pos, Quaternion? rot)
void  ReceiveSquadMessage(SquadMessage, Vector3, GameObject)

// Properties
AIState    CurrentState
AlertPhase AlertPhase
Stance     CurrentStance
bool       IsAlive / IsDead
GameObject PrimaryTarget    // from Blackboard
```

### AIPerception
```csharp
bool      CanSeeTarget(GameObject target)
Vector3?  GetLastKnownPosition(GameObject target)
ThreatEntry? GetHighestThreat()
void      NotifySound(Vector3 pos, float loudness, string tag,
                      bool suppressed, GameObject source)

// Events
Action<StimulusEvent>          OnStimulus
Action<AlertPhase, AlertPhase> OnAlertPhaseChanged
Action<ThreatEntry>            OnNewThreatDetected
Action<ThreatEntry>            OnThreatLost
```

### AIEmotionSystem
```csharp
float Morale / Fear / Anger   // 0–100
bool  IsInPanic / IsEnraged / IsDemoralized / IsMoralHigh
float AccuracyMultiplier      // 0.45 – 1.45
float FireRateMultiplier
float SpeedMultiplier

void OnKilledEnemy()
void OnLeaderDied()
void OnReinforced()
```

### BehaviourTree
```csharp
tree.SetRoot(BehaviourTree.BuildInfantryTree());
tree.SetRoot(BehaviourTree.BuildSniperTree());
tree.SetRoot(BehaviourTree.BuildScoutTree());
tree.active   = true;    // BT takes over from FSM
tree.debugLog = true;    // Console logging

BTVisualizer.Print(tree.Root);  // ASCII tree dump
```

### AIRegistry (scene-wide)
```csharp
AIRegistry.Instance.GetAllAgents()
AIRegistry.Instance.GetEnemiesOf(agent)
AIRegistry.Instance.GetNearestEnemy(agent)
AIRegistry.Instance.NotifyAllAgentsOfSound(pos, loudness, tag)
```

---

## 🗂 Package Structure

```
Assets/HanerAI/
├── Scripts/
│   ├── Core/
│   │   ├── AICore.cs             interfaces, enums, Blackboard, StimulusEvent
│   │   ├── AIAgent.cs            coordinator
│   │   ├── AIStateMachine.cs     14-state FSM
│   │   └── AISystems.cs          NavMeshMovement, RigidbodyMovement, AIHealth,
│   │                             AIWorldInteraction, AILODController, AIRegistry,
│   │                             CoverPoint, AISoundEmitter, AIScentEmitter
│   ├── Perception/
│   │   └── AIPerception.cs       4-sense perception + awareness + memory
│   ├── Emotion/
│   │   └── AIEmotionSystem.cs    morale/fear/anger
│   ├── Squad/
│   │   └── AISquadController.cs  roles, leader, tactics, cohesion
│   ├── Combat/
│   │   └── AICombatController.cs burst fire, cover, flank, grenade, search
│   └── BehaviourTree/
│       └── BehaviourTree.cs      full BT with abort system
├── Editor/
│   └── HanerAIBuilder.cs         one-click demo scene
└── Documentation/
    └── README.md
```

---

## ⚙️ Inspector Presets

### Standard Infantry
```
Faction: Enemy    Squad: Alpha    Role: Assault
Vision Angle: 110°    Vision Range: 28m
Hearing Range: 22m    Scent Range: 10m
Base Morale: 70    Burst Duration: 1.0s
```

### Sniper
```
Role: Sniper    Vision Angle: 60°    Vision Range: 60m
Hearing Range: 30m    Preferred Range: 40m
Base Morale: 65    Burst Duration: 0.3s    Base Accuracy: 0.92
```

### Heavy / Tank
```
Role: Support    Base Health: 250    Armour: 0.3
Preferred Range: 8m    Cover Hold Time: 6s
Burst Duration: 2s    Grenade Enabled: false
```

---

## 📦 Technical Specs

| | |
|---|---|
| Unity | 2021.3 LTS+ |
| Render Pipeline | Built-in · URP · HDRP |
| NavMesh | Unity built-in AI |
| Dependencies | **None** |
| Scripts | 10 C# files |
| Lines of code | ~3,500 |
| FSM States | 14 |
| BT Node types | 14 (composites + decorators + leaves) |
| Perception senses | 4 (vision, hearing, scent, touch) |
| Abort types | 3 (Self, LowerPriority, Both) |
| Platforms | Win · macOS · Linux · iOS · Android |

---

## 🛒 Purchase

**[Unity Asset Store — $59.99 →](https://assetstore.unity.com)**

Source code is distributed exclusively through the Unity Asset Store.
This repository contains documentation and issue tracking only.

---

## 📬 Support & Contact

- **Bug reports & feature requests:** [Open an Issue](https://github.com/karasakal/haner-ai/issues)
- **Documentation:** Included in package + this README
- **Email:** info@hanergames.com.tr
- **Publisher:** [hanergames.com.tr](https://hanergames.com.tr)

When reporting a bug, please include: Unity version, render pipeline, movement backend (NavMesh/Rigidbody), and steps to reproduce.

---

## 📄 License

Haner AI is a **commercial asset**. Source code available only to Asset Store purchasers.

- ✅ Unlimited personal and commercial projects
- ✅ Modify for your own project needs
- ❌ Redistribute, resell, or include in other asset packages
- ❌ Share source code publicly

© 2024 Haner Games — hanergames.com.tr
