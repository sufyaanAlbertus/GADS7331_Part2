# refinements-changes.md — Whisper Village

## Continuous Log of Scope Changes & AI-Assisted Decisions

This is a living document. Update it every time a design decision is made, a feature changes, or an AI tool assists development.

---

## Entries

### [Project Start] — Concept Selection
**Change/Decision:** Selected "2D RPG Village with Ollama-powered NPC dialogue" as the game concept.
**Reason:** Satisfies all LLM integration requirements in the brief. Simple enough to complete within timeline. Large supply of free top-down 2D art assets.
**AI Assistance:** Claude (claude.ai) used to brainstorm concepts and confirm alignment with brief.
**Impact:** All documentation and development scoped around this concept.

---

### [Project Start] — Model Selection
**Change/Decision:** Chose llama3 as the primary Ollama model. Mistral as fallback.
**Reason:** llama3 produces high-quality natural-language dialogue and follows character prompts reliably. Mistral is faster on lower-spec hardware.
**AI Assistance:** None — based on Ollama documentation.
**Impact:** OllamaManager.cs has a MODEL constant swappable in one line.

---

### [Refinement 1] — Slime Quest Added
**Change/Decision:** Added a slime-killing quest as the core gameplay mechanic.
**Reason:** The brief requires LLM content that "contributes directly to gameplay." A quest delivered via Ollama and tracked in-game satisfies this more strongly than dialogue alone.
**AI Assistance:** Claude (claude.ai) designed the quest flow and wrote NPCQuestGiver, QuestManager, SlimeHealth, SlimeMovement scripts.
**Impact:** 4 new scripts. Quest tracker UI, quest complete panel, 3 slime GameObjects added.

---

### [Refinement 2] — OllamaManager Extended
**Change/Decision:** Added SendPromptWithTask() method alongside SendPrompt().
**Reason:** Quest delivery needed an explicit goal instruction that standard NPC prompts don't use.
**AI Assistance:** Claude (claude.ai) wrote the updated OllamaManager.
**Impact:** Two public methods: SendPrompt() for regular NPCs, SendPromptWithTask() for Rodric's quest speech.

---

### [Refinement 3] — Quest Starts on Dialogue Close
**Change/Decision:** Quest tracker and slime spawn only trigger after player closes the quest dialogue panel.
**Reason:** Starting quest the moment text arrived felt abrupt — player may not have read it yet.
**AI Assistance:** Claude (claude.ai) added OnDialogueClosed Action event to DialogueUI and pendingQuestStart flag to NPCQuestGiver.
**Impact:** DialogueUI fires OnDialogueClosed callback. QuestManager.StartSlimeQuest() only called then.

---

### [Refinement 4] — Press E Prompt Hides During Dialogue
**Change/Decision:** "Press E to talk" prompt disappears when dialogue opens and returns when closed.
**Reason:** Having the prompt visible while already in dialogue looked wrong and confused testers.
**AI Assistance:** Claude (claude.ai) updated DialogueUI.ShowLoading() to track and restore the prompt.
**Impact:** DialogueUI tracks promptWasVisible. ShowLoading() hides prompt. CloseDialogue() restores it.

---

### [Refinement 5] — Quest Counter Updates Instantly on Kill
**Change/Decision:** "Slimes killed: X / 3" updates immediately when a slime dies.
**Reason:** Early version felt laggy. Immediate feedback is more satisfying.
**AI Assistance:** Claude (claude.ai) moved RefreshProgressUI() before completion check in OnSlimeKilled().
**Impact:** Player sees counter change same frame the slime is destroyed.

---

### [Refinement 6] — Dedicated CameraFollow Script
**Change/Decision:** Removed camera logic from PlayerController.cs, created CameraFollow.cs.
**Reason:** Separation of concerns — camera config should be independent of player movement.
**AI Assistance:** Claude (claude.ai) wrote CameraFollow.cs.
**Impact:** Main Camera has CameraFollow.cs. Smooth follow with configurable speed.

---

### [Refinement 7] — Proximity-Based Slime Attack System
**Change/Decision:** Slime combat changed from clicking sprite to proximity trigger zone + left-click.
**Reason:** OnMouseDown() was unreliable — blocked by UI, required precise clicking on small sprites.
**AI Assistance:** Claude (claude.ai) redesigned SlimeHealth.cs with dual-collider setup.
**Impact:** Each slime has two CircleCollider2D components. Attack only works inside trigger zone.

---

### [Refinement 8] — Null Reference Error Fixes
**Change/Decision:** Added null checks and Debug.LogError messages to all NPC scripts.
**Reason:** NPCInteraction was throwing NullReferenceException when DialogueUI wasn't found.
**AI Assistance:** Claude (claude.ai) diagnosed and fixed the pattern.
**Impact:** All scripts check for null on Start() and log clear error messages.

---

### [Refinement 9] — Slimes Hidden Until Quest Starts
**Change/Decision:** Slimes are hidden on scene load and only activated after player closes quest dialogue.
**Reason:** Slimes appearing before the quest was given broke the narrative flow.
**AI Assistance:** Claude (claude.ai) updated QuestManager with HideSlimes()/ShowSlimes() and Slimes array.
**Impact:** QuestManager.Start() calls HideSlimes(). StartSlimeQuest() calls ShowSlimes().

---

### [Refinement 10] — Story Order System (StoryManager)
**Change/Decision:** Added StoryManager to enforce talk order: Mira → Yara → Finn → Rodric.
**Reason:** Players could skip to Rodric and get the quest without any story context.
**AI Assistance:** Claude (claude.ai) wrote StoryManager.cs, updated NPCInteraction and NPCQuestGiver.
**Impact:** Each NPC checks IsCurrentTarget(). Locked NPCs show brief hint. Rodric locked until all three talked to.

---

### [Refinement 11] — Direction Indicator Arrow
**Change/Decision:** Replaced minimap idea with a simple compass arrow pointing to next objective.
**Reason:** Minimap required world bounds calibration and dot positioning — too complex. Arrow is simpler and more intuitive.
**AI Assistance:** Claude (claude.ai) wrote DirectionIndicator.cs with phase-based target sequence.
**Impact:** Arrow rotates toward next target silently. No text. Hides after Rodric final interaction.

---

### [Refinement 12] — Full Animation System
**Change/Decision:** Added directional animations for player (idle/walk/attack), NPCs (idle), and slimes (idle/move/death).
**Reason:** Static sprites look unprofessional. Animations make the game feel alive.
**AI Assistance:** Claude (claude.ai) wrote PlayerAnimator.cs, SlimeAnimator.cs, provided Animator Controller setup guide.
**Impact:** Player has 9 animation directions covered by 6 sprites using flipX for left. Slimes have death animation before destruction.

---

### [Refinement 13] — Game Manager + Main Menu + Game Over
**Change/Decision:** Added GameManager.cs controlling main menu visibility, game over fade-in, restart and quit. All in one scene.
**Reason:** Single scene is simpler to manage. Main menu hides on Play, game over fades in after final dialogue.
**AI Assistance:** Claude (claude.ai) wrote GameManager.cs and updated NPCQuestGiver final dialogue flow.
**Impact:** MainMenuPanel hidden on Play(). GameOverPanel fades in via CanvasGroup after 2-second delay.

---

### [Refinement 14] — Custom Pixel Art UI Assets
**Change/Decision:** Created custom pixel art main menu background, Play/Quit/Restart buttons, game over screen, and NPC portrait icons as PNG images.
**Reason:** Default Unity UI looks generic. Custom pixel art assets match the game's visual style.
**AI Assistance:** Claude (claude.ai) generated all UI images using the actual game map screenshot as the menu background.
**Impact:** whisper-village-menu.png, button-play.png, button-quit.png, button-restart.png, gameover-bg.png, and 4 NPC icon PNGs created and imported.

---

### [Refinement 15] — NPC Icons in Dialogue Panel
**Change/Decision:** Added NPC portrait icons to the dialogue panel that update automatically per speaker.
**Reason:** Dialogue panel showed only a name — no visual identity for who was speaking. Icons make each NPC instantly recognisable.
**AI Assistance:** Claude (claude.ai) updated DialogueUI.cs with NPCIconEntry array and SetIcon() method. Generated 4 pixel art portrait icons (80x80px) for Mira, Yara, Finn, and Rodric.
**Impact:** DialogueUI now has an npcIcons array mapped by name. Icon updates on ShowLoading() and ShowDialogue(). Custom pixel art portraits created for all 4 NPCs.

---

### [Refinement 16] — Custom Character Sprite Sheets
**Change/Decision:** Created pixel art idle sprite sheets for all 5 characters (Player, Mira, Yara, Finn, Rodric).
**Reason:** No suitable matching sprites were found in the free asset packs for all characters. Custom sprites ensure visual consistency across the whole cast.
**AI Assistance:** Claude (claude.ai) generated all 5 sprite sheets programmatically — 4-frame idle animation per character, 32x32px per frame.
**Impact:** Each character has a unique colour palette, accessory, and idle bob animation. Frame 3 includes an eye blink. Sheets are ready to slice and import into Unity Animator.

---

### [Refinement 17] — Quest Panel and Arrow Hidden on Game Over
**Change/Decision:** GameManager now hides the quest panel, quest complete panel, and direction indicator the moment TriggerGameOver() is called.
**Reason:** When the game over screen faded in, the quest tracker and arrow indicator were still visible underneath, breaking the visual presentation.
**AI Assistance:** Claude (claude.ai) updated GameManager.cs to accept references to these panels and call SetActive(false) before the fade sequence.
**Impact:** Game over screen is now fully clean — no overlapping UI elements from gameplay visible beneath it.

---

### [Refinement 18] — Player Frozen During Dialogue
**Change/Decision:** Player movement and attack are both disabled while the dialogue panel is open.
**Reason:** Player could walk away from NPCs or trigger attack animations while reading dialogue, which broke immersion and caused unintended slime damage.
**AI Assistance:** Claude (claude.ai) added IsDialogueOpen() method to DialogueUI.cs and updated PlayerController.cs and PlayerAnimator.cs to check it.
**Impact:** PlayerController.cs blocks movement input when IsDialogueOpen() returns true. PlayerAnimator.cs blocks both movement and attack animations. Player returns to idle pose during dialogue. All input resumes immediately on dialogue close.

---

## AI Tools Used

| Tool | Purpose |
|---|---|
| Ollama (llama3) | Core game feature — NPC dialogue at runtime |
| Claude (claude.ai) | Documentation, script architecture, debugging, prompt design, UI image generation, sprite generation |

---

## Scope Decision Log

| Feature | Status | Reason |
|---|---|---|
| NPC dialogue via Ollama | ✅ In scope | Core brief requirement |
| Slime-killing quest | ✅ In scope | LLM-driven gameplay loop |
| Quest tracker UI | ✅ In scope | Live feedback on LLM-generated quest |
| Story order enforcement | ✅ In scope | Narrative structure and immersion |
| Direction indicator arrow | ✅ In scope | Simple player guidance system |
| Proximity attack system | ✅ In scope | Reliable combat without UI interference |
| Full animation system | ✅ In scope | Professional presentation |
| Main menu + game over | ✅ In scope | Complete game experience |
| Custom pixel art UI | ✅ In scope | Visual polish matching game style |
| NPC dialogue icons | ✅ In scope | Visual identity per speaker |
| Custom character sprites | ✅ In scope | Visual consistency across cast |
| Player frozen during dialogue | ✅ In scope | Immersion and bug fix |
| Combat animations (swing) | ✅ In scope | Visual Aid |
| Player health / death | ❌ Out of scope | Adds complexity without LLM value |
| Multiple scenes | ❌ Out of scope | One scene is sufficient |
| Inventory system | ❌ Out of scope | Not required by brief |
| Save / load | ❌ Out of scope | Not required by brief |
