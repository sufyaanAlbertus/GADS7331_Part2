# Whisper Village

A 2D top-down RPG where every NPC is powered by a locally hosted Large Language Model (Ollama). Talk to villagers in story order, receive an AI-generated quest to slay 3 slimes, hunt them down, and return to the Elder for an AI-generated reward — then watch the game end with a cinematic fade.

---

## Overview

**Module:** Game Design 3A — Part 2: LLM-Integrated Game
**Engine:** Unity 2022.3 LTS
**LLM:** Ollama (llama3) — fully local, no internet required during play

---

## Group Members

| Student Number | Name |
|---|---|
| ST10436103 | Sufyaan Albertus |
| ST10262263 | Sandile Duba |

---

## Features

- Top-down 2D player movement (WASD) with smooth camera follow
- **Story order system** — talk to Mira → Yara → Finn before Rodric unlocks
- **Direction indicator arrow** — silently points to next objective at all times
- 4 interactable NPCs with unique Ollama-generated personalities
- Elder Rodric delivers a slime-killing quest dynamically via Ollama
- Quest only activates after player reads and closes the dialogue
- "Press E to talk" prompt hides during dialogue and returns after close
- **Proximity-based slime combat** — walk into range then left-click to attack
- 3 slimes — wander randomly, take 3 hits, flash red on hit, play death animation
- Slimes hidden until quest starts, spawned when dialogue closes
- Quest tracker updates instantly on every kill: `Slimes killed: 0 / 3`
- Rodric generates unique contextual dialogue on quest completion
- **Game Over screen** fades in after final dialogue closes
- Full directional animation system — player, NPCs, slimes
- Custom pixel art UI — menu background, buttons, game over screen
- Fully local inference — Ollama runs on your machine

---

## How to Play

| Input | Action |
|---|---|
| WASD / Arrow Keys | Move player |
| E | Talk to nearby NPC (if it is their turn) |
| ESC | Close dialogue panel |
| Left Mouse Click | Attack slime (must be in range) |

**Story flow:**
1. Click **Play** on the main menu
2. Arrow points to Mira → walk to her → press E → read dialogue → ESC
3. Arrow points to Yara → repeat
4. Arrow points to Finn → repeat
5. Arrow points to Rodric → press E → Ollama generates quest speech → ESC
6. Quest tracker appears — 3 slimes spawn in the world
7. Walk into a slime's range → left-click 3 times to kill it
8. Repeat for all 3 slimes → Quest Complete panel appears
9. Arrow points back to Rodric → press E → reward dialogue → ESC
10. 2-second pause → **Game Over** fades in
11. Restart or Quit

---

## Requirements

- Unity 2022.3 LTS
- Ollama installed and running (`ollama serve`)
- llama3 model pulled (`ollama pull llama3`)
- Windows 10 / macOS 12 / Ubuntu 20.04 or later
- Minimum 8 GB RAM

---

## Installation

### 1. Clone the repository
```bash
git clone https://github.com/sufyaanAlbertus/GADS7331_Part2.git
cd whisper-village
```

### 2. Install Ollama
Download from [https://ollama.com](https://ollama.com)

### 3. Pull the model
```bash
ollama pull llama3
```

### 4. Open in Unity
- Unity Hub → Open → Add project from disk → select `whisper-village`
- Open `Assets/Scenes/VillageScene.unity`
- Press **Play**

---

## Project Structure

```
Assets/
├── Scripts/
│   ├── PlayerController.cs      — WASD movement
│   ├── PlayerAnimator.cs        — Directional animations + flipX
│   ├── CameraFollow.cs          — Smooth camera follow
│   ├── OllamaManager.cs         — HTTP requests to Ollama API
│   ├── DialogueUI.cs            — Dialogue panel, prompt logic, OnDialogueClosed
│   ├── StoryManager.cs          — NPC talk order enforcement
│   ├── NPCInteraction.cs        — Regular NPCs — locked until their turn
│   ├── NPCQuestGiver.cs         — Rodric — quest + game over trigger
│   ├── QuestManager.cs          — Kill tracker, slime hide/show, completion
│   ├── SlimeHealth.cs           — Proximity attack, 3 hits, death animation
│   ├── SlimeMovement.cs         — Random wander
│   ├── SlimeAnimator.cs         — Directional idle/move/death animations
│   ├── DirectionIndicator.cs    — Arrow pointing to next objective
│   └── GameManager.cs           — Main menu, game over fade, restart/quit
├── Scenes/
│   └── VillageScene.unity
├── Art/
│   ├── Tilesets/
│   ├── Characters/
│   └── UI/
│       ├── whisper-village-menu.png
│       ├── button-play.png
│       ├── button-quit.png
│       ├── button-restart.png
│       └── gameover-bg.png
├── Animations/
│   ├── Player/
│   ├── NPC/
│   └── Slime/
└── Prefabs/
docs/
├── high-concept.md
├── ollama-plan.md
├── setup.md
├── refinements-changes.md
├── readme.md
└── prompts-used.md
```

---

## Collider Reference

| GameObject | Collider | Is Trigger | Purpose |
|---|---|---|---|
| Player | Box Collider 2D | FALSE | Solid body |
| NPCs | Box Collider 2D | TRUE | E key proximity |
| Slime (body) | Circle Collider 2D | FALSE | Solid body |
| Slime (range) | Circle Collider 2D | TRUE | Attack zone — auto-created |

---

## Dependencies

| Dependency | Version | Purpose |
|---|---|---|
| Unity | 2022.3 LTS | Game engine |
| Ollama | Latest | Local LLM inference server |
| llama3 | 8B | NPC dialogue + quest generation |
| TextMeshPro | Bundled with Unity | UI text rendering |

---

## Free Assets Used

| Asset | Source | Licence |
|---|---|---|
| Mystic Woods tileset | [game-endeavor.itch.io](https://game-endeavor.itch.io/mystic-woods) | Free for personal/commercial use |
| Cute Fantasy RPG sprites | [kenmi-art.itch.io](https://kenmi-art.itch.io/cute-fantasy-rpg) | Free for personal/commercial use |
| Fantasy Game GUI | [gameart2d.com](https://www.gameart2d.com/free-fantasy-game-gui.html) | Free to use |

---

## AI Tools Used

| Tool | Purpose |
|---|---|
| Ollama (llama3) | Real-time NPC dialogue and quest delivery at runtime |
| Claude (claude.ai) | Documentation, script architecture, prompt design, debugging, UI image generation |

---

## References

Game Endeavor (no date) *Mystic Woods — 16x16 pixel art asset pack*. itch.io. Available at: https://game-endeavor.itch.io/mystic-woods (Accessed: 14 May 2026).

GameArt2D.com (no date) *Free fantasy game GUI*. GameArt2D.com. Available at: https://www.gameart2d.com/free-fantasy-game-gui.html (Accessed: 14 May 2026).

Kenmi (no date) *Cute fantasy RPG — 16x16 top down pixel art asset pack*. itch.io. Available at: https://kenmi-art.itch.io/cute-fantasy-rpg (Accessed: 14 May 2026).

---

## Videos

- **Technical Demonstration (3–6 min):** Ollama running locally, terminal showing HTTP request/response, NPC dialogue and quest delivery in action.
- **Final Showcase (3–6 min):** Full gameplay — story order, quest given, slimes killed, quest complete, Rodric reward, game over fade. Design intent and reflection.

---

## Credits

**Group Members:**

| Student Number | Name |
|---|---|
| ST10436103 | Sufyaan Albertus |
| ST10262263 | Sandile Duba |

**Institution:** IIE — The Independent Institute of Education
**Module:** Game Design 3A
**Year:** 2026
