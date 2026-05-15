# High Concept Document — Whisper Village

---

## Concept Statement

*Whisper Village* is a top-down 2D interactive RPG set in a medieval village threatened by slimes. Instead of relying on pre-written, scripted dialogue trees, all NPC conversation is replaced by AI-powered dialogue that generates dynamic responses aware of both conversational and environmental context. Players navigate the village, talk to NPCs, and receive responses that are uniquely generated per character.

---

## The Role of the LLM

The LLM is **core to the experience** — not a supplementary feature.

| Interaction | Behaviour |
|---|---|
| Quest delivery (Rodric, first visit) | Urgent dramatic speech — instructs player to kill 3 slimes. |
| Quest reminder (Rodric, mid-quest) | Contextual update using the current kill count. |
| Quest completion (Rodric, post-kill) | Gratitude and reward flavour dialogue. |
| Regular NPCs (Mira, Yara, Finn) | Character-consistent dialogue and village lore. |

---

## Combat Design

Slime combat uses a **proximity-based attack system**:

- Each slime has two colliders: a solid body collider and a larger trigger zone (attack range).
- The player must walk into the trigger zone to enter attack range.
- Left-clicking while in range defeats the slime; clicking outside range does nothing.
- This creates intentional engagement — the player must commit to fighting a slime.

---

## Why a Local Model is Appropriate

- **No internet dependency:** Inference runs entirely on the local machine.
- **Reproducibility:** The same Ollama version produces consistent, assessable behaviour.
- **No API costs or rate limits:** Unlimited calls during development and video recording.
- **Assessment compliance:** Directly demonstrates an understanding of local vs. cloud inference.

---

## Game World

| Element | Detail |
|---|---|
| **Setting** | Small medieval village with a well, blacksmith, inn, and open fields. |
| **Tone** | Warm but tense. Villagers are frightened by the sudden slime appearances. |
| **Visual Style** | Free top-down 2D RPG assets (Kenney.nl, OpenGameArt LPC sprites). |
| **NPCs** | Elder Rodric (quest giver), Mira (blacksmith), Yara (herbalist), Finn (guard). |
| **Enemies** | 3 green slimes — wander randomly; proximity trigger + left-click to defeat. |
