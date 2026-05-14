# Whisper Village

A 2D top-down RPG where every NPC is powered by a locally hosted Large Language Model (Ollama). Receive an AI-generated quest to slay 3 slimes, hunt them using a proximity attack system, and return for an AI-generated reward.

---

## Overview

**Module:** Game Design 3A — Part 2: LLM-Integrated Game  
**Engine:** Unity 2022.3 LTS  
**LLM:** Ollama (llama3) — fully local, no internet required during play

---

## Features

- Top-down 2D player movement (WASD) with smooth camera follow
- 4 interactable NPCs with unique Ollama-generated personalities
- Elder Rodric delivers a slime-killing quest dynamically via Ollama
- Quest only activates after player reads and closes the dialogue
- "Press E to talk" prompt hides during dialogue and returns after close
- Proximity-based slime combat — walk into range then left-click to attack
- 3 slimes take 3 hits each, flash red on hit, wander randomly
- Quest tracker updates instantly on every kill: `Slimes killed: 0 / 3`
- Quest Complete panel on all 3 kills
- Rodric generates unique contextual dialogue on quest completion
- Fully local inference — Ollama runs on your machine

---

## How to Play

| Input | Action |
|---|---|
| WASD / Arrow Keys | Move player |
| E | Talk to nearby NPC |
| ESC | Close dialogue panel |
| Left Mouse Click | Attack slime (must be in range) |

**Quest flow:**
1. Walk to Elder Rodric → press E → Ollama generates quest speech (2–5 sec)
2. Read the dialogue → press ESC to close → quest tracker appears
3. Walk into a slime's range → left-click 3 times to kill it
4. Repeat for all 3 slimes → Quest Complete panel appears
5. Return to Rodric → press E for reward dialogue

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
git clone https://github.com/YOUR_USERNAME/whisper-village.git
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
│   ├── PlayerController.cs     — WASD movement
│   ├── CameraFollow.cs         — Smooth camera follow
│   ├── OllamaManager.cs        — HTTP requests to Ollama API
│   ├── DialogueUI.cs           — Dialogue panel, prompt hide/restore logic
│   ├── NPCInteraction.cs       — Regular NPC E-key dialogue
│   ├── NPCQuestGiver.cs        — Rodric quest delivery, starts on dialogue close
│   ├── QuestManager.cs         — Kill tracker, instant UI update, completion
│   ├── SlimeHealth.cs          — Proximity trigger + left-click attack system
│   └── SlimeMovement.cs        — Random slime wander
├── Scenes/
│   └── VillageScene.unity
├── Art/
│   ├── Tilesets/
│   └── Characters/
└── UI/
docs/
├── high-concept.md
├── ollama-plan.md
├── setup.md
├── refinements-changes.md
├── readme.md
└── prompts-used.md
```

---

## Collider Setup Reference

| GameObject | Collider | Is Trigger | Purpose |
|---|---|---|---|
| Player | Box Collider 2D | FALSE | Solid body |
| NPCs | Box Collider 2D | TRUE | E key proximity detection |
| Slime (body) | Circle Collider 2D | FALSE | Solid body |
| Slime (range) | Circle Collider 2D | TRUE | Attack zone — auto-created by SlimeHealth.cs |

---

## Dependencies

| Dependency | Version | Purpose |
|---|---|---|
| Unity | 2022.3 LTS | Game engine |
| Ollama | Latest | Local LLM inference server |
| llama3 | 8B | NPC dialogue + quest generation |
| TextMeshPro | Bundled with Unity | UI text rendering |

---

## AI Tools Used

| Tool | Purpose |
|---|---|
| Ollama (llama3) | Real-time NPC dialogue and quest delivery at runtime |
| Claude (claude.ai) | Documentation, script architecture, prompt design, debugging |

---

## Videos

- **Technical Demonstration (3–6 min):** Ollama running locally, terminal showing HTTP request/response, NPC dialogue and quest delivery in action.
- **Final Showcase (3–6 min):** Full gameplay — quest given, 3 slimes killed with proximity system, quest complete, Rodric reward dialogue. Design intent and reflection.

---

## Credits

**Developer:** [Your Name]  
**Institution:** [Your Institution]  
**Module:** Game Design 3A  
**Year:** 2026  

---

## References

Game Endeavor (no date) *Mystic Woods — 16x16 pixel art asset pack*. itch.io. Available at: https://game-endeavor.itch.io/mystic-woods (Accessed: 14 May 2026).

GameArt2D.com (no date) *Free fantasy game GUI*. GameArt2D.com. Available at: https://www.gameart2d.com/free-fantasy-game-gui.html (Accessed: 14 May 2026).

Kenmi (no date) *Cute fantasy RPG — 16x16 top down pixel art asset pack*. itch.io. Available at: https://kenmi-art.itch.io/cute-fantasy-rpg (Accessed: 14 May 2026).
