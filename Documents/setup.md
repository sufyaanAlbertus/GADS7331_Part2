# setup.md — Whisper Village Technical Setup Guide

**Module:** Game Design 3A — Part 2: LLM-Integrated Game
**Engine:** Unity 2022.3 LTS
**LLM:** Ollama — llama3 (local inference)
**Group:** ST10436103 Sufyaan Albertus | ST10262263 Sandile Duba

---

## 1. System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| OS | Windows 10 / macOS 12 / Ubuntu 20.04 | Windows 11 / macOS 14 |
| RAM | 8 GB | 16 GB |
| GPU | Not required (CPU inference works) | NVIDIA 6GB+ VRAM (faster inference) |
| Storage | 10 GB free | 20 GB free |
| Unity | 2022.3 LTS | 2022.3 LTS |

---

## 2. Install Ollama

### Windows / macOS
1. Go to [https://ollama.com](https://ollama.com) and download the installer
2. Run and follow the prompts
3. Ollama starts as a background service automatically after install

### Linux
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Verify Installation
```bash
ollama --version
```

> You should see a version number such as `ollama version 0.1.x` — if not, restart your terminal and try again.

---

## 3. Pull the LLM Model

Pull the primary model (downloads approximately 4.7 GB):
```bash
ollama pull llama3
```

If llama3 is too slow on your machine, use mistral as a fallback:
```bash
ollama pull mistral
```

> If switching to mistral, open `OllamaManager.cs` and change the MODEL constant from `"llama3"` to `"mistral"` — one line change.

### Test the Model
```bash
ollama run llama3
```
Type a message and confirm a response is generated. Type `/bye` to exit.

---

## 4. Confirm the Ollama API is Running

Run this command in PowerShell to test the API endpoint:

```powershell
Invoke-WebRequest -Uri http://localhost:11434/api/generate `
  -Method POST `
  -ContentType 'application/json' `
  -Body '{"model":"llama3","prompt":"Speak as a worried medieval village elder. One sentence.","stream":false}'
```

A successful response looks like this:

| Field | Expected Value |
|---|---|
| StatusCode | 200 |
| StatusDescription | OK |
| Content | JSON object containing a `response` field with generated text |
| Content-Type | application/json; charset=utf-8 |

> **Warning:** If you see a connection error, Ollama is not running. Open a terminal and run `ollama serve` then try again.

---

## 5. Create the Unity Project

1. Open Unity Hub and click **New Project**
2. Select template: **2D (Built-in Render Pipeline)** — NOT URP or HDRP
3. Name the project: `WhisperVillage`
4. Choose a save location and click **Create Project**
5. Once open: **Window > Package Manager**
6. Change dropdown to **Unity Registry** — search `TextMeshPro` — click **Install**
7. When prompted click **Import TMP Essentials** — this is required for all UI text

> **Warning:** TextMeshPro Essentials must be imported or the scripts will throw compile errors. If you miss the prompt, go to `Window > TextMeshPro > Import TMP Essential Resources`.

---

## 6. Folder Structure

Create the following folders inside `Assets/` in the Project window:

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

## 7. Scripts — Copy to Assets/Scripts/

| Script | Purpose |
|---|---|
| `PlayerController.cs` | WASD movement — freezes input while dialogue is open |
| `PlayerAnimator.cs` | Directional idle/walk/attack animations + flipX for left |
| `CameraFollow.cs` | Smooth camera follow with configurable speed |
| `OllamaManager.cs` | HTTP POST to local Ollama API — SendPrompt() and SendPromptWithTask() |
| `DialogueUI.cs` | Dialogue panel — hides E prompt, fires OnDialogueClosed, exposes IsDialogueOpen() |
| `StoryManager.cs` | Enforces NPC talk order Mira → Yara → Finn — locks NPCs out of sequence |
| `NPCInteraction.cs` | Regular NPCs — only responds when it is their turn in the story sequence |
| `NPCQuestGiver.cs` | Elder Rodric — quest delivery, progress reminder, final dialogue triggers game over |
| `QuestManager.cs` | Kill tracker, instant UI update, slime hide/show on quest start, completion panel |
| `SlimeHealth.cs` | Proximity trigger zone + left-click attack, 3 hits to kill, death animation trigger |
| `SlimeMovement.cs` | Random wander AI with configurable speed and direction variance |
| `SlimeAnimator.cs` | Directional idle/move/death animations with flipX for left movement |
| `DirectionIndicator.cs` | Arrow rotates silently toward next objective — phase-based sequence |
| `GameManager.cs` | Main menu show/hide, game over CanvasGroup fade, hides quest/arrow panels on end |

---

## 8. Scene GameObjects and Components

### 8a. Player

| Component | Setting |
|---|---|
| Sprite Renderer | Character sprite — Sorting Layer = Characters |
| Rigidbody 2D | Gravity Scale = 0 — Freeze Rotation Z = ticked |
| Box Collider 2D | Is Trigger = FALSE — sized to sprite |
| PlayerController | Move Speed = 4 |
| PlayerAnimator | Attached — no Inspector slots needed |
| Animator | Controller = PlayerAnimator controller |
| Tag | **Player** (CRITICAL — NPCInteraction checks for this tag) |
| Layer | Player (for Physics 2D collision matrix) |

---

### 8b. Main Camera

| Component | Setting |
|---|---|
| CameraFollow | Target = Player GameObject — Smooth Speed = 5 |
| Projection | Orthographic — Size = 5 |
| Background | Dark colour of your choice |

---

### 8c. Regular NPCs (NPC_Mira, NPC_Yara, NPC_Finn)

| Component | Setting |
|---|---|
| Sprite Renderer | Character sprite — Sorting Layer = Characters |
| Box Collider 2D | Is Trigger = **TRUE** (critical — must be ticked) |
| Animator | Controller = NPCAnimator |
| NPCInteraction | Fill: npcName, npcPersonality, npcSecret, lockedMessage |

NPC name fields — must match StoryManager exactly:

| NPC | npcName field value |
|---|---|
| Mira | `Mira the Blacksmith` |
| Yara | `Yara the Herbalist` |
| Finn | `Finn the Guard` |

> **Warning:** The npcName field must match EXACTLY what is in StoryManager's `requiredNPCNames` array. Case-sensitive.

---

### 8d. Elder Rodric (Quest Giver)

| Component | Setting |
|---|---|
| Sprite Renderer | Character sprite — Sorting Layer = Characters |
| Box Collider 2D | Is Trigger = TRUE |
| Animator | Controller = NPCAnimator |
| NPCQuestGiver | npcName = `Elder Rodric` — fill personality and secret |

---

### 8e. Slimes (Slime_1, Slime_2, Slime_3)

| Component | Setting |
|---|---|
| Sprite Renderer | Slime sprite — Sorting Layer = Characters |
| Rigidbody 2D | Gravity Scale = 0 — Freeze Rotation Z = ticked |
| Circle Collider 2D #1 | Is Trigger = **FALSE** — Radius = 0.4 (solid body) |
| Circle Collider 2D #2 | Is Trigger = **TRUE** — auto-created by SlimeHealth.cs at runtime |
| Animator | Controller = SlimeAnimator |
| SlimeHealth | maxHealth = 3 — attackRange = 1.0 — deathAnimDuration = match clip length |
| SlimeMovement | moveSpeed = 1 |
| SlimeAnimator | Attached — no Inspector slots needed |
| Layer | Enemies (for Physics 2D collision matrix) |

---

### 8f. Manager GameObjects

| GameObject | Setup |
|---|---|
| OllamaManager | Empty GameObject — attach `OllamaManager.cs` only |
| StoryManager | Empty GameObject — attach `StoryManager.cs` — drag HintPanel and HintText in |
| QuestManager | Empty GameObject — attach `QuestManager.cs` — drag all UI slots and 3 slimes in |
| GameManager | Empty GameObject — attach `GameManager.cs` — drag MainMenuPanel, GameOverPanel, CanvasGroup, Player, quest panels, director panel in |
| DirectionIndicator | Empty GameObject — attach `DirectionIndicator.cs` — drag Player, NPCs in order, Rodric, Slimes in order, ArrowImage in |

---

## 9. UI Setup — Canvas Hierarchy

All UI elements sit in one Canvas with **Render Mode = Screen Space Overlay**:

```
Canvas  (Screen Space Overlay)
├── MainMenuPanel          ← shown on start, hidden when Play clicked
│   ├── MenuBGImage        — Source Image: whisper-village-menu.png
│   ├── PlayButton         — OnClick: GameManager.PlayGame()
│   └── QuitButton         — OnClick: GameManager.QuitGame()
│
├── DialoguePanel          ← hidden on start
│   ├── NPCIconImage       — Image, 80x80, left side of panel
│   ├── NPCNameText        — TextMeshPro, NPC name bold
│   ├── ResponseText       — TextMeshPro, Ollama response, wrapping enabled
│   └── CloseButton        — OnClick: DialogueUI.CloseDialogue()
│
├── PromptPanel            ← hidden on start
│   └── PromptText         — TextMeshPro: "Press E to talk"
│
├── QuestPanel             ← hidden on start, shown after quest dialogue closed
│   ├── QuestTitleText     — TextMeshPro
│   └── QuestProgressText  — TextMeshPro
│
├── QuestCompletePanel     ← hidden on start
│   └── QuestCompleteText  — TextMeshPro
│
├── GameOverPanel          ← hidden on start, CanvasGroup alpha = 0, fades in at end
│   ├── GameOverBGImage    — Source Image: gameover-bg.png
│   ├── RestartButton      — OnClick: GameManager.RestartGame()
│   └── QuitButton         — OnClick: GameManager.QuitGame()
│
├── HintPanel              ← hidden on start (StoryManager locked hint)
│   └── HintText           — TextMeshPro
│
└── DirectorPanel          ← always visible
    └── ArrowImage         — Image with arrow sprite, ~56x56, top-right corner
```

### GameOverPanel — Critical Setup

> **Warning:** GameOverPanel MUST have a CanvasGroup component with Alpha = 0. Without it the fade will not work.

1. Select `GameOverPanel`
2. Add Component → **Canvas Group**
3. Set the following:

| CanvasGroup Setting | Value |
|---|---|
| Alpha | `0` (starts invisible) |
| Interactable | ✅ ticked |
| Blocks Raycasts | ✅ ticked |
| Ignore Parent Groups | ❌ unticked |

---

## 10. Physics 2D Layer Setup

Prevents player clipping through slimes while keeping the attack trigger zone working:

1. **Edit > Project Settings > Tags and Layers**
2. Scroll to Sorting Layers — add in order: `Background`, `Ground`, `Objects`, `Characters`, `UI`
3. Scroll to Layers — add layer: `Player`
4. Add another layer: `Enemies`
5. Select **Player** GameObject — set Layer to `Player`
6. Select **Slime_1, Slime_2, Slime_3** — set Layer to `Enemies`
7. **Edit > Project Settings > Physics 2D**
8. Scroll down to **Layer Collision Matrix**
9. Find where the `Player` row meets the `Enemies` column — **untick that box**

> Player and Enemies layers no longer physically collide — slimes pass through the player — but the trigger zone still detects the player for attack range. This is the intended behaviour.

---

## 11. Animator Controllers Setup

### Player Animator Controller

**Parameters:**

| Parameter | Type |
|---|---|
| `moveX` | Float |
| `moveY` | Float |
| `idleX` | Float |
| `isMoving` | Bool |
| `isAttacking` | Trigger |
| `attackX` | Float |
| `attackY` | Float |

**Blend Trees:**

| State | Type | Parameters |
|---|---|---|
| `Idle_BlendTree` | 2D Simple Directional | idleX, moveY |
| `Walk_BlendTree` | 2D Simple Directional | moveX, moveY |
| `Attack_BlendTree` | 2D Simple Directional | attackX, attackY |

**Transitions:**

| Transition | Condition | Has Exit Time | Duration |
|---|---|---|---|
| Entry → Idle_BlendTree | Automatic (default state) | — | — |
| Idle → Walk | isMoving = true | ❌ No | 0 |
| Walk → Idle | isMoving = false | ❌ No | 0 |
| Any State → Attack | isAttacking trigger | ❌ No | 0 |
| Attack → Idle | None | ✅ Yes (Exit Time = 1) | 0 |

---

### Slime Animator Controller

**Parameters:**

| Parameter | Type |
|---|---|
| `moveX` | Float |
| `moveY` | Float |
| `idleX` | Float |
| `isMoving` | Bool |
| `isDead` | Trigger |

**Transitions:**

| Transition | Condition | Has Exit Time |
|---|---|---|
| Entry → Idle_BlendTree | Automatic (default state) | — |
| Idle → Move | isMoving = true | ❌ No |
| Move → Idle | isMoving = false | ❌ No |
| Any State → Slime_Death | isDead trigger | ❌ No |

---

## 12. Running the Game

### Before Pressing Play

1. Open a terminal and run: `ollama serve`
2. Confirm the terminal shows: `Listening on 127.0.0.1:11434`
3. Leave the terminal open — do not close it while playing

### Full Gameplay Flow

| Step | Action |
|---|---|
| 1 | Open `VillageScene.unity` in Unity Editor — press **Play** |
| 2 | Click **Play** button on the main menu |
| 3 | Arrow points to Mira — walk to her — press E — read dialogue — press ESC |
| 4 | Arrow points to Yara — repeat step 3 |
| 5 | Arrow points to Finn — repeat step 3 |
| 6 | Arrow points to Rodric — press E — Ollama generates quest speech — press ESC |
| 7 | Quest tracker appears: `Slimes killed: 0 / 3` — 3 slimes spawn in the world |
| 8 | Walk into a slime's trigger range — left-click 3 times to kill it |
| 9 | Quest tracker updates: `1 / 3` — repeat for all 3 slimes |
| 10 | Quest Complete panel appears — arrow points back to Rodric |
| 11 | Walk to Rodric — press E — Ollama generates thank-you dialogue — press ESC |
| 12 | 2-second pause — Game Over screen fades in |
| 13 | Click **Restart** to reload or **Quit** to close |

---

## 13. Troubleshooting

| Problem | Fix |
|---|---|
| Dialogue shows `[Ollama unreachable]` | Run `ollama serve` in a terminal; confirm port 11434 is not blocked |
| Press E does nothing near NPC | Check: NPC Box Collider Is Trigger = TRUE; Player Tag = `Player` |
| NPC says "talk to someone else first" | Talk to the NPC the arrow is currently pointing to — story order is enforced |
| Arrow points wrong direction | Adjust Sprite Offset in DirectionIndicator Inspector: try 0, 90, -90, or 180 |
| Slimes appear before quest | Fill QuestManager Slimes array with all 3 slime GameObjects |
| Left-click does nothing on slimes | Walk closer — must be inside trigger zone first; check Player tag = `Player` |
| Player can move during dialogue | Confirm latest `PlayerController.cs` is in use — checks `IsDialogueOpen()` |
| Player can attack during dialogue | Confirm latest `PlayerAnimator.cs` is in use — checks `IsDialogueOpen()` |
| Game Over pops in instantly | CanvasGroup on GameOverPanel must have Alpha = 0 before pressing Play |
| Game Over never shows | Check `GameManager.Instance` is set in `Awake()`; check NPCQuestGiver final dialogue flow |
| Quest panel visible on Game Over | Fill GameManager `questPanel` and `directorPanel` slots in Inspector |
| Animation plays wrong direction | Check Blend Tree Pos X/Y values match the setup table exactly |
| Sprites look blurry | Set Filter Mode = **Point (no filter)** on all character PNGs |
| Player clips through slimes | Set up Physics 2D Layer Collision Matrix — untick Player vs Enemies |
| Compile errors on import | Confirm TextMeshPro Essentials were imported (`Window > TextMeshPro`) |
| Response takes over 10 seconds | Switch MODEL to `mistral` in `OllamaManager.cs` |
