# setup.md — Whisper Village Technical Setup Guide

## System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| OS | Windows 10 / macOS 12 / Ubuntu 20.04 | Windows 11 / macOS 14 |
| RAM | 8 GB | 16 GB |
| GPU | Not required (CPU inference works) | NVIDIA 6GB+ VRAM (faster inference) |
| Storage | 10 GB free | 20 GB free |
| Unity | 2022.3 LTS | 2022.3 LTS |

---

## Step 1 — Install Ollama

### Windows / macOS
1. Go to [https://ollama.com](https://ollama.com) and download the installer
2. Run and follow the prompts — Ollama starts as a background service automatically

### Linux
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Verify
```bash
ollama --version
```

---

## Step 2 — Pull the LLM Model

```bash
ollama pull llama3
```
Downloads ~4.7 GB. If llama3 is too slow on your machine:
```bash
ollama pull mistral
```
Then change `MODEL = "llama3"` to `MODEL = "mistral"` in `OllamaManager.cs`.

---

## Step 3 — Confirm the Ollama API

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Speak as a worried medieval village elder. One sentence.",
  "stream": false
}'
```
You should receive a JSON object with a `"response"` field. StatusCode 200 = working.

---

## Step 4 — Create the Unity Project

1. Open Unity Hub → **New Project**
2. Template: **2D (Built-in Render Pipeline)**
3. Name: `WhisperVillage` → **Create Project**
4. **Window > Package Manager** → search `TextMeshPro` → Install → **Import TMP Essentials**

---

## Step 5 — Folder Structure

Create inside `Assets/`:
```
Assets/
├── Scenes/
├── Scripts/
├── Art/
│   ├── Tilesets/
│   ├── Characters/
│   └── UI/
├── Animations/
│   ├── Player/
│   ├── NPC/
│   └── Slime/
└── Prefabs/
```

---

## Step 6 — Copy All Scripts

Place all `.cs` files into `Assets/Scripts/`:

| Script | Purpose |
|---|---|
| `PlayerController.cs` | WASD movement |
| `PlayerAnimator.cs` | Directional animations + flipX for left |
| `CameraFollow.cs` | Smooth camera follow |
| `OllamaManager.cs` | HTTP requests to local Ollama API |
| `DialogueUI.cs` | Dialogue panel — hides prompt, fires OnDialogueClosed |
| `StoryManager.cs` | Enforces NPC talk order, locks NPCs out of sequence |
| `NPCInteraction.cs` | Regular NPCs — locked until their turn |
| `NPCQuestGiver.cs` | Elder Rodric — quest flow, final dialogue triggers game over |
| `QuestManager.cs` | Tracks kills, hides/shows slimes, updates UI |
| `SlimeHealth.cs` | Proximity trigger + left-click attack, 3 hits, death anim |
| `SlimeMovement.cs` | Random wander AI |
| `SlimeAnimator.cs` | Directional idle/move/death animations |
| `DirectionIndicator.cs` | Arrow pointing to next objective silently |
| `GameManager.cs` | Main menu hide/show, game over fade-in, restart/quit |

---

## Step 7 — Scene GameObjects & Components

### Player
| Component | Setting |
|---|---|
| Sprite Renderer | Character sprite, Sorting Layer = Characters |
| Rigidbody 2D | Gravity Scale = 0, Freeze Rotation Z |
| Box Collider 2D | Is Trigger = FALSE |
| PlayerController | Move Speed = 4 |
| PlayerAnimator | Attached — no slots needed |
| Animator | Controller = PlayerAnimator |
| Tag | **Player** (critical) |
| Layer | Player (for Physics 2D matrix) |

### Main Camera
| Component | Setting |
|---|---|
| CameraFollow | Target = Player, Smooth Speed = 5 |
| Projection | Orthographic, Size = 5 |

### NPCs (Mira, Yara, Finn)
| Component | Setting |
|---|---|
| Sprite Renderer | Sorting Layer = Characters |
| Box Collider 2D | Is Trigger = TRUE |
| Animator | Controller = NPCAnimator |
| NPCInteraction | Fill name, personality, secret, locked message |

### Elder Rodric
| Component | Setting |
|---|---|
| Sprite Renderer | Sorting Layer = Characters |
| Box Collider 2D | Is Trigger = TRUE |
| Animator | Controller = NPCAnimator |
| NPCQuestGiver | Fill name, personality, secret |

### Slimes (Slime_1, Slime_2, Slime_3)
| Component | Setting |
|---|---|
| Sprite Renderer | Sorting Layer = Characters |
| Rigidbody 2D | Gravity Scale = 0, Freeze Rotation Z |
| Circle Collider 2D #1 | Is Trigger = FALSE, Radius = 0.4 (solid body) |
| Circle Collider 2D #2 | Is Trigger = TRUE, auto-created by SlimeHealth.cs |
| Animator | Controller = SlimeAnimator |
| SlimeHealth | maxHealth = 3, attackRange = 1.0, deathAnimDuration = match clip |
| SlimeMovement | moveSpeed = 1 |
| SlimeAnimator | Attached |
| Layer | Enemies |

### OllamaManager
- Empty GameObject → `OllamaManager.cs` only

### StoryManager
- Empty GameObject → `StoryManager.cs`
- Drag **HintPanel** and **HintText** into Inspector slots
- Required NPC Names: `Mira the Blacksmith`, `Yara the Herbalist`, `Finn the Guard`

### QuestManager
- Empty GameObject → `QuestManager.cs`
- Drag: QuestPanel, QuestTitleText, QuestProgressText, QuestCompletePanel, QuestCompleteText
- Drag all 3 slime GameObjects into the **Slimes** array

### GameManager
- Empty GameObject → `GameManager.cs`
- Drag: MainMenuPanel, GameOverPanel
- **Game Over Group** → drag `GameOverPanel` (the CanvasGroup sits on it)
- Drag: Player GameObject
- Game Over Delay = 2, Fade Time = 1.5

### DirectionIndicator
- Empty GameObject → `DirectionIndicator.cs`
- Player → Player GameObject
- NPCs In Order → Element 0 = Mira, 1 = Yara, 2 = Finn
- Quest Giver → NPC_Rodric
- Slimes In Order → Element 0 = Slime_1, 1 = Slime_2, 2 = Slime_3
- Arrow Image → ArrowImage RectTransform

---

## Step 8 — UI Setup

### Canvas (Screen Space Overlay)
```
Canvas
├── MainMenuPanel        ← shown on start, hidden on Play
│   ├── MenuBGImage      (set whisper-village-menu.png as Source Image)
│   ├── PlayButton       (Source Image = button-play.png) → GameManager.PlayGame()
│   └── QuitButton       (Source Image = button-quit.png) → GameManager.QuitGame()
│
├── DialoguePanel        ← hidden on start
│   ├── NPCNameText      (TMP)
│   ├── ResponseText     (TMP)
│   └── CloseButton      → DialogueUI.CloseDialogue()
│
├── PromptPanel          ← hidden on start
│   └── PromptText       (TMP — "Press E to talk")
│
├── QuestPanel           ← hidden on start
│   ├── QuestTitleText   (TMP)
│   └── QuestProgressText (TMP)
│
├── QuestCompletePanel   ← hidden on start
│   └── QuestCompleteText (TMP)
│
├── GameOverPanel        ← hidden on start, CanvasGroup alpha = 0
│   ├── GameOverBGImage  (set gameover-bg.png as Source Image)
│   ├── RestartButton    (Source Image = button-restart.png) → GameManager.RestartGame()
│   └── QuitButton       (Source Image = button-quit.png) → GameManager.QuitGame()
│
├── HintPanel            ← hidden on start (StoryManager locked hint)
│   └── HintText         (TMP)
│
└── DirectorPanel        ← always visible
    └── ArrowImage       (Image with arrow sprite, ~56x56, top-right corner)
```

### GameOverPanel — Critical Setup
1. Select `GameOverPanel`
2. Add Component → **Canvas Group**
3. Set Alpha = `0`, Interactable = ✅, Blocks Raycasts = ✅

---

## Step 9 — Physics 2D Layer Setup

Prevents player clipping through slimes while keeping OnMouseDown working:

1. **Edit > Project Settings > Tags and Layers**
2. Add Layer: `Player` and `Enemies`
3. Set Player GameObject layer → `Player`
4. Set Slime_1, Slime_2, Slime_3 layer → `Enemies`
5. **Edit > Project Settings > Physics 2D** → Layer Collision Matrix
6. Untick the box where `Player` and `Enemies` intersect

---

## Step 10 — Running the Game

1. Start Ollama: open terminal → `ollama serve`
2. Open `VillageScene.unity` in Unity Editor
3. Press **Play**
4. Click **Play** on the main menu
5. Arrow appears pointing to Mira — walk to her and press E
6. Continue in order: Yara → Finn → Rodric
7. Read quest dialogue → ESC to close → slimes appear
8. Walk into slime range → left-click 3 times to kill each
9. Return to Rodric → ESC to close final dialogue → Game Over fades in

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Dialogue shows "[Ollama unreachable]" | Run `ollama serve` in terminal |
| Press E does nothing near NPC | NPC Box Collider Is Trigger = TRUE; Player tag = "Player" |
| NPC says "talk to someone else first" | That NPC is locked — talk to the current arrow target first |
| Arrow points wrong direction | Adjust Sprite Offset in DirectionIndicator Inspector (0, 90, -90, or 180) |
| Slimes appear before quest | QuestManager Slimes array must be filled; HideSlimes() runs on Start() |
| Left-click does nothing on slimes | Walk closer — must be inside trigger zone; check Player tag = "Player" |
| Game Over panel pops instantly | Confirm CanvasGroup is on GameOverPanel with alpha = 0 |
| Game Over never shows | Check GameManager.Instance is set; check NPCQuestGiver final dialogue flow |
| Animations wrong direction | Check blend tree Pos X/Y values; check flipX logic in PlayerAnimator |
| Player clips through slimes | Set up Physics 2D Layer Collision Matrix (Step 9) |
| Compile errors | Confirm TextMeshPro Essentials imported |
| Response takes >10 seconds | Switch to mistral in OllamaManager.cs |
