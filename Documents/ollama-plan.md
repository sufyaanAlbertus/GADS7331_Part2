# ollama-plan.md — Whisper Village

## Model Choice

| Setting | Value |
|---|---|
| **Primary model** | llama3 (8B parameters) |
| **Fallback model** | mistral (7B) — swap MODEL constant in OllamaManager.cs |
| **Inference server** | Ollama running locally on localhost:11434 |

**Why llama3:** Strong instruction-following, natural conversational output, handles character-voice prompts reliably. Produces responses in 2–5 seconds on consumer hardware. Mistral is faster on lower-spec machines.

---

## When Inference Occurs

All inference is **on-demand at runtime** — triggered only by player pressing E near an NPC.

| Trigger | When | Script |
|---|---|---|
| Player presses E near Rodric (first visit) | Runtime — after StoryManager confirms all NPCs talked to | NPCQuestGiver.cs |
| Player presses E near Rodric (quest active) | Runtime | NPCQuestGiver.cs |
| Player presses E near Rodric (quest complete) | Runtime — final dialogue that triggers game over | NPCQuestGiver.cs |
| Player presses E near Mira | Runtime — only when it is Mira's turn | NPCInteraction.cs |
| Player presses E near Yara | Runtime — only when it is Yara's turn | NPCInteraction.cs |
| Player presses E near Finn | Runtime — only when it is Finn's turn | NPCInteraction.cs |

**Inference does NOT occur during:**
- Scene load or startup
- Main menu display
- Player movement or combat
- Slime movement, animation, or death
- Quest tracker UI updates
- Direction indicator updates
- Game over fade sequence

---

## Data Flow

```
[Player presses E near NPC]
          |
          v
[StoryManager — IsCurrentTarget() / CanAccessQuestGiver()]
          |
          | Locked → show hint, return
          | Unlocked → continue
          v
[NPCQuestGiver.cs OR NPCInteraction.cs]
          |
          | Builds prompt from NPC identity + player context
          v
[OllamaManager.cs — SendPrompt() or SendPromptWithTask()]
          |
          | HTTP POST → http://localhost:11434/api/generate
          | JSON: { "model": "llama3", "prompt": "...", "stream": false }
          v
[Ollama Local Server — llama3 inference on CPU/GPU]
          |
          | Returns JSON: { "response": "generated text..." }
          v
[OllamaManager.cs — parses response field, fires callback]
          |
          v
[DialogueUI.cs — ShowDialogue(npcName, reply)]
          |
          v
[Player reads dialogue → closes panel (ESC or X)]
          |
          v
[OnDialogueClosed callback fires]
          |
          ├── NPCInteraction: RegisterTalked() → StoryManager advances sequence
          ├── NPCQuestGiver (quest dialogue): QuestManager.StartSlimeQuest() → slimes spawn
          └── NPCQuestGiver (final dialogue): GameManager.TriggerGameOver() → fade in
```

---

## Prompt Templates

### PROMPT-01 — Regular NPC (NPCInteraction.cs)

Used for: Mira, Yara, Finn

```
You are {npcName}, a character living in Whisper Village.
Your personality: {personality}
Your secret (reveal hints if pressed): {secret}
Reply in 3-4 sentences. Stay in character. Do not mention AI or game mechanics.
Player says: Hello, what can you tell me about the village?
```

---

### PROMPT-02 — Quest Delivery: Rodric First Visit (SendPromptWithTask)

```
You are Elder Rodric, a character living in Whisper Village.
Your personality: Wise, deeply worried, and desperate for a hero.
Your secret: He accidentally knocked over a magical artefact that spawned the slimes,
             but is too ashamed to admit it.
Current situation: Three dangerous slimes have appeared at the edge of the village
                   and are threatening the villagers.
Your goal: Urgently beg the player to kill all 3 slimes. Be dramatic and desperate.
           Describe the threat vividly. End with: 'Will you save us?'
Reply in 4-5 sentences. Stay in character. Do not mention AI or game mechanics.
```

---

### PROMPT-03 — Quest Reminder: Rodric Mid-Quest (SendPrompt)

```
You are Elder Rodric...
Player says: I have killed {X} slimes so far. There are {Y} left.
```

---

### PROMPT-04 — Quest Completion: Rodric Post-Kill (SendPrompt)

```
You are Elder Rodric...
Player says: I have slain all three slimes. The village is safe now.
```

---

## Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Ollama response too slow (>8s) | Medium | Switch MODEL to "mistral" in OllamaManager.cs; loading dots shown during wait |
| Ollama not running at game start | Medium | Console shows error; dialogue displays "[Ollama unreachable]" |
| Model breaks character | Low | Prompt includes "Do not mention AI or game mechanics" |
| JSON parse fails | Low | OllamaManager fallback: "[Error: could not parse Ollama response.]" |
| Player can't attack slimes | Low | Must enter trigger zone first; check Player tag = "Player" |
| Quest starts before player reads | N/A | Fixed — StartSlimeQuest() only called in OnDialogueClosed callback |
| Game over triggers before final dialogue | N/A | Fixed — TriggerGameOver() only called in OnFinalDialogueClosed callback |
| CanvasGroup fade not working | Low | Confirm CanvasGroup component is on GameOverPanel; check alpha starts at 0 |
