# Into The Abyss

> A first-person, movement-based FPS shoot-'em-up built in Unity 2022.3 LTS, with procedurally generated infinite levels, state-machine-driven enemy AI, and a boss fight. Targets WebGL via a CI/CD pipeline.

![Glitch Shader](Images/GlitchShader.gif)

---

## Overview

**Into The Abyss** is a team project built by **AlphaBeta** as part of COMP30019 at the University of Melbourne. The player descends through a glitching, infinite office space, fighting hostile NPCs before reaching a boss arena. The game leans into an eerie CRT-era atmosphere (fisheye distortion, chromatic aberration, film grain, and a sickly colour grade) while keeping the moment-to-moment FPS feel snappy and responsive.

- **Engine:** Unity 2022.3.51f1 (LTS)
- **Render Pipeline:** Universal Render Pipeline (URP)
- **Target:** WebGL, deployed via GitHub Actions
- **Team Size:** 4

## Gameplay

| Infinite Office (Level 1) | Boss Arena (Level 2) |
|---|---|
| ![Infinite Office](Images/InfiniteOffice.png) | ![Boss Level](Images/BossLevel.png) |

---

## My Contributions: Alistair Cheah Wern Hao

My contributions cover the **core gameplay loop**: player movement, the weapon system, enemy AI, the boss fight, and the HUD/menu stack. Highlights below, each linking to the relevant source.

### Player Movement & Camera

- Built a **direct-transform FPS controller** in [MovementV2.cs](Assets/Scripts/Player/MovementV2.cs) that delivers a snappy, modern-shooter feel. Chose direct transform writes over rigidbody velocity to eliminate the "slippery" controls the team's prototype suffered from, then layered slope detection and selective physics interactions on top so knockback and collisions still behave correctly.
- Wired the camera through [PlayerRotation.cs](Assets/Scripts/Player/PlayerRotation.cs) using **Cinemachine's virtual camera** for responsive mouse-look. This required routing the in-game settings (sensitivity, FOV) through the Cinemachine rig rather than `Camera.main`, which was a non-obvious gotcha, captured in [SettingManage.cs](Assets/Scripts/UI/SettingManage.cs).
- Added a **stamina-driven sprint system** with a fading UI slider ([SprintSlider.cs](Assets/Scripts/UI/SprintSlider.cs)).
- Implemented the **player health & damage system** with heart-based UI in [HealthSystem.cs](Assets/Scripts/UI/HealthSystem.cs).

### Weapon System

![Weapons](Images/Weapons.png)

- Designed a slot-based **weapon controller** ([PlayerAttackController.cs](Assets/Scripts/Gun/PlayerAttackController.cs)) that manages three distinct weapon types (melee, pistol, and shotgun) and supports **runtime pickup/drop unlocking**.
- Implemented each weapon with its own behaviour:
  - **Pistol**: [Pistol.cs](Assets/Scripts/Gun/Pistol.cs) with recoil and per-shot audio variation.
  - **Shotgun**: [Shotgun.cs](Assets/Scripts/Gun/Shotgun.cs) with spread mechanics.
  - **Melee**: [Melee.cs](Assets/Scripts/Melee/Melee.cs) with a basic swing animation.
- Synced weapon models to vertical head-bob via [GunRotationSync.cs](Assets/Scripts/Gun/GunRotationSync.cs) and [MeleeRotationSync.cs](Assets/Scripts/Melee/MeleeRotationSync.cs) so weapons feel attached to the player, not floating in front.
- Built **bullet projectiles** for both sides of combat: [BulletBehaviourPlayer.cs](Assets/Scripts/Gun/BulletBehaviourPlayer.cs) and [BulletBehaviourEnemy.cs](Assets/Scripts/Gun/BulletBehaviourEnemy.cs).
- Layered in a **full audio cue set** per weapon (firing, reloading, arming, empty-mag alerts) with randomised variations for realism, plus damage grunts for both player and enemies.

### Enemy AI

- Designed an **enemy AI state machine** in [EnemyAI.cs](Assets/Scripts/Enemy/EnemyAI.cs) that transitions cleanly between **Patrolling → Chasing → Attacking** based on player proximity and line-of-sight triggers.
- Shipped **two enemy archetypes** off the same state machine via a serialized enum:
  - **Shooting enemy**: charges a projectile before releasing, forcing the player to break cover.
  - **Charging enemy**: closes the gap, damages on collision, then retreats to telegraph the next strike.
- Integrated with the runtime-baked NavMesh so enemies pathfind correctly through procedurally generated office tiles.
- Added **spatial audio for player and enemy footsteps** ([FootstepsAudio.cs](Assets/Scripts/Player/FootstepsAudio.cs), [EnemyFootstepsAudio.cs](Assets/Scripts/Enemy/EnemyFootstepsAudio.cs)) so the player can hear enemies approaching from off-screen.

### Boss Fight

- Built the boss controller ([BossEnemy.cs](Assets/Scripts/Enemy/BossEnemy.cs)) with waypoint-driven movement and a **dropping-explosive** attack pattern that forces the player to keep moving through the arena.
- Implemented the **boss health system** ([BossHealthSystem.cs](Assets/Scripts/Enemy/BossHealthSystem.cs)) and on-screen boss health bar ([BossHP.cs](Assets/Scripts/Enemy/BossHP.cs)).
- Re-purposed the object spawner to drop the boss into the second-level arena and **baked the static NavMesh** for the church scene (Level 1 uses runtime baking; the boss arena doesn't, so it needed a one-time bake).

### UI / UX

- Reworked the **pause manager** ([PauseManagerScript.cs](Assets/Scripts/UI/PauseManagerScript.cs)) to handle clean transitions between gameplay, pause, settings, main menu, and instructions.
- Built the **HUD** ([PlayerHUD.cs](Assets/Scripts/UI/PlayerHUD.cs)) including crosshair, ammo display, heart-based health, fading sprint slider, and the **10-minute countdown timer** ([CountdownTimer.cs](Assets/Scripts/UI/CountdownTimer.cs)) that drives the game's tension.
- Redesigned the **settings screen** so mouse sensitivity and FOV correctly drive the Cinemachine rig (previously they were silently no-ops because they targeted `Camera.main`).
- Authored an **instructions screen flow** ([InstructionScreenManager.cs](Assets/Scripts/UI/InstructionScreenManager.cs)) to onboard new players.
- Remodelled the level-exit elevator from a flat platform into a triggered open elevator with a fade-to-ceiling transition between scenes.

---

## Technical Highlights

These are the standout visual and systems pieces of the project. I worked closely with the team on integration; direct authorship is called out per item.

### Glitch Shader *(by William Spongberg)*

![Glitch Shader](Images/GlitchShader.gif)

A URP HLSL shader simulating CRT-era visual artifacts on NPCs: per-channel RGB offsets over time, randomised x-vertex displacement, probabilistic pixel discards for a "broken signal" flicker, and dithered black-bar overlay. GPU-instanced for cheap multi-enemy use.

### Boss Shader *(by William Spongberg)*

![Boss Shader](Images/BossShader.gif)

A screen-space "portal" shader for the boss; the texture is locked to screen position regardless of camera, with chromatic aberration, world-position-based distortion, and a red pulsating glow. Paired with a runtime texture swap script for added chaos.

### Blood Particle System *(by William Spongberg)*

![Blood Particle System](Images/BloodParticleSystem.gif)

Drives the enemy hit feedback: bursts of fast, randomly sized particles that darken and shrink over time, with trailing sub-emitters for a dripping effect.

### Post-Processing Stack *(by William Spongberg, integrated across both my scenes)*

Each scene runs a tailored URP Global Volume with Panini projection, film grain, vignette, chromatic aberration, bloom, lift-gamma-gain, lens distortion, depth of field, motion blur, ACES tone mapping, and reduced saturation, all tuned to land somewhere between "old CRT" and "off-brand horror".

### Procedural World Generation *(by William Spongberg)*

Perlin-noise-based infinite office generation with a dynamic grid that spawns and destroys room prefabs around the player and re-bakes the NavMesh on the fly, which is the system my enemy AI runs on.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Engine | Unity 2022.3.51f1 (LTS) |
| Language | C# |
| Rendering | Universal Render Pipeline (URP) 14.0.11 |
| Camera | Cinemachine 2.10.1 |
| AI / Navigation | Unity AI Navigation 1.1.5 (runtime + static NavMesh bake) |
| Input | Unity Input System 1.11.0 |
| UI | TextMeshPro 3.0.9 |
| Shaders | HLSL (URP shader graph + handwritten) |
| Build / Deploy | GitHub Actions → WebGL |

---

## Team

**AlphaBeta**, COMP30019 Project 2, The University of Melbourne.

| Member | Focus |
|---|---|
| **Alistair Cheah Wern Hao** | Movement, Combat, Enemy AI, Boss, UI |
| William Spongberg | Team Lead, World Generation, Shaders, VFX, Post-Processing |
| Ananya Agarwal | Elevator System, Pause Menu, Audio, User Evaluation |
| David Kee Siong Chin | Story, Boss design support |

---

## Credits & Assets

- [Synty Studios Student Bundle](https://assetstore.unity.com/student-plan-pack1): low-poly 3D models, textures, and materials.
- [Unity Starter Assets](https://assetstore.unity.com/packages/essentials/starter-assets-thirdperson-updates-in-new-charactercontroller-pa-196526): NPC movement animations and environment materials.
- Pseudo-random and dithering helper functions in the shaders are sourced from the Unity documentation and are not original work.
