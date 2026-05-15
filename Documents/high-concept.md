# High Concept Document — Whisper Village

## Project Overview

| Field | Detail |
|---|---|
| **Title** | Whisper Village |
| **Genre** | 2D Top-Down RPG |
| **Engine** | Unity 2022.3 LTS (2D Built-in Render Pipeline) |
| **LLM Integration** | Ollama — llama3 (local inference) |
| **Module** | Game Design 3A — Part 2: LLM-Integrated Game |

---

## Concept Statement

Whisper Village is a 2D top-down RPG set in a small medieval village threatened by slimes. Every NPC dialogue is powered by a locally hosted Large Language Model (Ollama). Players explore the village, talk to NPCs in a specific story order, and receive dynamically generated responses unique to each character.

The core gameplay loop revolves around a single AI-generated quest: **Elder Rodric** uses Ollama to dramatically ask the player to kill 3 slimes. The player must walk into a slime's attack range and left-click to deal damage. Once all 3 are dead, the player returns to Rodric for an AI-generated reward response, after which the game ends with a fade-in Game Over screen.

---

## Core Gameplay Loop

1. Player spawns in the village centre. Movement: WASD / Arrow Keys.
2. A **direction indicator arrow** points the player to the next objective at all times.
3. Player must talk to **Mira → Yara → Finn** in order — talking out of turn shows a locked hint.
4. After all three are spoken to, **Elder Rodric** unlocks.
5. Player approaches Rodric and presses **E** — the "Press E to talk" prompt hides during dialogue.
6. Ollama generates a dramatic urgent quest speech. Player closes the dialogue (ESC or X).
7. Quest tracker UI appears: *"Slimes killed: 0 / 3"* — slimes spawn in the world.
8. Player walks toward a slime and enters its **attack range trigger zone**.
9. Player **left-clicks** to deal 1 damage — slime flashes red, plays hit animation.
10. 3 clicks kills one slime — death animation plays then slime is destroyed.
11. Quest tracker updates instantly: *"Slimes killed: 1 / 3"*
12. Arrow points to remaining slimes. Repeat for all 3.
13. Quest Complete panel appears. Arrow points back to Rodric.
14. Player returns to Rodric — Ollama generates contextual thank-you dialogue.
15. Player closes dialogue → 2-second delay → **Game Over panel fades in**.
16. Player clicks Restart (reloads scene) or Quit (closes game).

---

## LLM Role in Gameplay

The LLM is **core to the experience** — not a supplementary feature.

| Interaction | LLM Behaviour |
|---|---|
| Quest delivery (Rodric, first visit) | Urgent dramatic speech — instructs player to kill 3 slimes |
| Quest reminder (Rodric, mid-quest) | Contextual update using current kill count |
| Quest completion (Rodric, post-kill) | Gratitude and reward dialogue — triggers game over on close |
| Regular NPCs (Mira, Yara, Finn) | Character-consistent dialogue and village lore |

---

## Story Order System

The game enforces a strict NPC talk sequence via the **StoryManager**:

| Step | Target | Locked Until |
|---|---|---|
| 1 | Mira the Blacksmith | Always available first |
| 2 | Yara the Herbalist | Mira talked to |
| 3 | Finn the Guard | Yara talked to |
| 4 | Elder Rodric | All three talked to |
| 5–7 | Slimes (kill order) | Quest given by Rodric |
| 8 | Return to Rodric | All slimes killed |

If the player tries to talk to a locked NPC they see: *"Maybe I should talk to someone else first..."* for 1.5 seconds.

---

## Direction Indicator

A compass-style arrow UI (top-right corner) always points to the next objective silently — no text, just the arrow rotating toward the target. Sequence:

Mira → Yara → Finn → Rodric → Slime 1 → Slime 2 → Slime 3 → Return to Rodric → hidden

---

## Combat Design

Slime combat uses a **proximity-based attack system**:
- Each slime has two colliders: a solid body (Is Trigger = FALSE) and a larger trigger zone (Is Trigger = TRUE, auto-created by SlimeHealth.cs)
- Player must walk into the trigger zone to be in attack range
- Left-clicking while in range deals 1 damage
- Slimes flash red on hit (HitFlash coroutine) and play a directional move animation while alive
- On death: death animation plays, QuestManager notified, GameObject destroyed

---

## Animation System

| Character | Animations | Notes |
|---|---|---|
| Player | Idle (Up/Down/Left/Right), Walk (Up/Down/Right), Attack (Up/Down/Right) | Left = Right flipped via flipX |
| NPCs | Idle only | Single directional idle loop |
| Slimes | Idle (Up/Down/Right), Move (Up/Down/Right), Death (Right only) | Left = Right flipped; death plays once then destroyed |

---

## UI System

| Panel | When Shown |
|---|---|
| MainMenuPanel | On scene load — hides when Play clicked |
| DialoguePanel | When Ollama response arrives — hides on ESC/X |
| PromptPanel | When player enters NPC trigger — hides during dialogue |
| QuestPanel | After player closes quest dialogue from Rodric |
| QuestCompletePanel | After all 3 slimes killed |
| GameOverPanel | Fades in 2 seconds after final Rodric dialogue closed |
| StoryManager HintPanel | Briefly when player tries locked NPC |

---

## Why a Local Model is Appropriate

- **No internet dependency** — inference runs entirely on the developer's machine.
- **Reproducibility** — same Ollama version produces consistent, assessable behaviour.
- **No API costs or rate limits** — unlimited calls during development and video recording.
- **Assessment compliance** — directly demonstrates local vs cloud inference understanding.

---

## Game World

| Element | Detail |
|---|---|
| Setting | Small medieval village — dirt paths, trees, a pond, houses, fences |
| Tone | Warm but tense — villagers frightened by slime appearance |
| Visual Style | Free top-down 2D RPG assets (Kenney.nl, OpenGameArt, Mystic Woods, Cute Fantasy RPG) |
| NPCs | Elder Rodric (quest giver), Mira (blacksmith), Yara (herbalist), Finn (guard) |
| Enemies | 3 green slimes — wander randomly, proximity trigger, 3-hit death, death animation |

---

## Scope

**In Scope:**
- One village scene with tilemap
- 4 interactable NPCs — story order enforced
- Ollama-powered dialogue for all NPCs
- 1 slime-killing quest with live tracker UI
- 3 slimes with proximity attack, directional animation, death animation
- Direction indicator arrow pointing to next objective
- Full animation system (player, NPCs, slimes)
- Main menu, game over screen with fade-in
- Custom pixel art UI — main menu background, buttons, game over screen

**Out of Scope (intentional — prototype only):**
- Player health or death
- Inventory or item systems
- Multiple scenes
- Save / load system
- Combat animations beyond hit flash and death

---

## Success Criteria

- Player talks to Mira, Yara, Finn in order → Rodric unlocks
- Rodric delivers Ollama quest speech → quest starts on dialogue close
- Arrow points correctly to each objective in sequence
- Player enters slime range → left-clicks → slime takes damage → dies after 3 hits
- Kill counter updates instantly: 0/3 → 1/3 → 2/3 → 3/3 → Quest Complete
- Player returns to Rodric → Ollama generates completion dialogue
- Player closes final dialogue → 2-second delay → Game Over panel fades in
- All 4 NPCs produce unique AI-generated dialogue
- Ollama running locally is visible in Technical Demonstration video
