# Easy Visual Novel Engine

A modular visual novel engine for Unity, built for writers and artists rather
than programmers. Stories are data you edit on a graph, every menu is generated
for you, and the look of the whole game comes from five colours on one asset.

Verified against **Unity 6000.6.0f1**, URP, **Input System (new)**, and
TextMeshPro (which ships inside `com.unity.ugui` 2.x). 54 scripts, no assembly
definitions — everything lands in `Assembly-CSharp`, so your own code can use it
with no extra setup.

---

## Contents

1. [Getting started](#getting-started)
2. [How the system fits together](#how-the-system-fits-together)
3. [The Story Graph](#the-story-graph)
4. [The data model](#the-data-model)
5. [Characters and expressions](#characters-and-expressions)
6. [Backgrounds](#backgrounds)
7. [Variables, conditions and branching](#variables-conditions-and-branching)
8. [Text variables](#text-variables)
9. [Screen effects](#screen-effects)
10. [Audio](#audio)
11. [Theming](#theming)
12. [Menus](#menus)
13. [Player controls](#player-controls)
14. [Save and load](#save-and-load)
15. [Preview, validation and search](#preview-validation-and-search)
16. [Importing from a spreadsheet](#importing-from-a-spreadsheet)
17. [Scripting reference](#scripting-reference)
18. [Troubleshooting](#troubleshooting)

---

## Getting started

1. `Tools > Visual Novel > Build VN Scene Setup` — builds the scene **and every
   menu**: title, system, save, load, settings, backlog, confirm, quick bar.
2. `Tools > Visual Novel > Create Sample Story` — a working branching story.
3. Drag the story onto **Story Player** on `VN Manager`.
4. Press Play.

> The first run may ask to import **TMP Essential Resources**. Accept, wait for
> the import to finish, then run the wizard again.

Then open `Tools > Visual Novel > Story Graph` and start writing.

### Every menu command

| Menu | What it does |
| --- | --- |
| `Build VN Scene Setup` | Creates and wires the entire scene |
| `Create Sample Story` | A three-branch demo with a variable and a locked choice |
| `Create Theme` | A new theme asset |
| `Story Graph` | The visual story editor |
| `Story Preview` | Step through a story without Play mode |
| `Find in Story` | Search and replace across every line |
| `Import Dialogue (CSV or JSON)` | Bulk-load from a spreadsheet |
| `Validate Selected Story` | Report problems to the Console |

### Assets you create

`Assets > Create > Visual Novel >` **Story**, **Dialogue Node**, **Choice Node**,
**Character**, **Theme**. In practice you only create Stories, Characters and
Themes by hand — nodes and choices are made for you in the graph.

---

## How the system fits together

Three layers that never reach into each other:

```
     DATA                    LOGIC                       PRESENTATION
  StoryAsset                                          VNUIController
  DialogueNode   ──────▶  DialogueParser   ──────▶      ├─ BackgroundUI
  ChoiceNode              (walks the story)             ├─ CharacterDisplayUI
  CharacterAsset                 │                      ├─ DialogueBoxUI
  VNTheme                        │                      └─ ChoiceMenuUI
                                 ▼
                            VNManager
              variables · audio · saves · transitions · theme
                                 │
                            VNCommands
                    (keyboard and every on-screen button)
```

- **Data** knows nothing about Unity scenes or UI. A `DialogueNode` is pure
  story: who speaks, what they say, what changes, where it goes.
- **`DialogueParser`** is the only thing that walks a story. It reads nodes,
  drives the typewriter, waits for input, evaluates conditions and picks the next
  node.
- **`VNUIController`** is the only thing the parser talks to for display. Replace
  the entire interface without touching story data or flow logic.
- **`VNManager`** owns everything global and survives scene loads: variables,
  audio, save files, screen transitions, the active theme.
- **`VNCommands`** is the single entry point for player actions. Hotkeys and
  on-screen buttons call the same methods, so they can never disagree.

### What a single line of dialogue actually does

1. `DialogueParser` enters the node and pushes a rollback snapshot.
2. The node's **On Enter Effects** change variables.
3. The line is added to the backlog and `NodeEntered` fires.
4. `VNUIController` applies the visuals: background, character sprite, name
   plate, speaker highlight, and any shake or flash.
5. Audio plays: music action, sound effect, voice line.
6. `{variables}` in the text are substituted, then the typewriter runs.
7. The node is marked read — this is what Skip uses later.
8. Flow resolves: choices → conditional routes → next node → end.

---

## The Story Graph

`Tools > Visual Novel > Story Graph`, or the button on any Story asset. This is
where you write.

| Action | How |
| --- | --- |
| Create a linked node | Drag from a node's right-hand dot into empty space |
| Link two nodes | Drag from a dot onto another node |
| Add a node | Right-click the canvas |
| Add a choice or route | Right-click a node |
| Set the start node | Right-click a node |
| Duplicate a node | Right-click a node |
| Move nodes | Drag (multi-select with Ctrl or Shift) |
| Pan | Middle-drag, or Alt + drag |
| Zoom | Scroll wheel |
| Frame everything | `F`, or **Frame All** |
| Tidy the layout | **Auto Layout** — columns by distance from the start |
| Delete | `Delete`, or right-click > Delete |

Ports are colour-coded to match their links: **grey = next**, **blue = choice**,
**orange = conditional route**. The start node is green, and each node shows a
thumbnail of the expression it will display.

**Nodes never need dragging into the story.** Anything created in the graph is
registered on the story automatically, as a sub-asset of the story file — so a
whole novel stays one asset in the Project window instead of a hundred loose
files. Deleting a node scrubs every link that pointed at it.

**Watching a playthrough.** Enter Play mode with the graph open: the node the
reader is on glows gold and everywhere they have been is marked. You can watch a
player move through your branches in real time.

### Inherit look

The **Inherit look** toggle in the graph toolbar (also on every node's Inspector,
and remembered between sessions) is on by default.

A new node copies the **background, character, pose tag, side and music track**
of the node it came from — the fields that are identical on almost every line of
a scene. Text, choices, links and variable effects are never copied; those are
what make a node itself.

Two deliberate exceptions:

- **Music action** resets to *No Change*, so a carried-over track does not
  restart on every line.
- **Character Sprite** is never copied. It is a manual override that beats the
  pose tag, so inheriting it would freeze every downstream node on one
  expression.

---

## The data model

| Asset | What it is |
| --- | --- |
| **StoryAsset** | One story: start node, every node in it, starting variables |
| **DialogueNode** | One beat: speaker, text, visuals, audio, effects, links |
| **ChoiceNode** | One button: text, target, availability conditions, effects |
| **CharacterAsset** | A cast member: name, colour, default side, named poses |
| **VNTheme** | The look of the entire game |

### DialogueNode fields

**Who is talking** — `character` (optional Character asset), `speakerName`
(overrides it; both empty means narration), `speakerSide`,
`useCharacterDefaultSide`.

**What they say** — `dialogueText`, with TextMeshPro rich text and `{variables}`.

**Visuals** — `emotionPose` (the tag), `characterSprite` (manual override),
`hideCharacter`, `clearOtherCharacters`, `backgroundSprite`, `hideDialogueBox`.

**Audio** — `musicAction` (No Change / Play / Stop), `music`, `sfx`, `voice`.

**Screen effects** — `shakeScreen` with strength and duration, `flashScreen`
with colour and duration.

**Logic** — `onEnterEffects`, `conditionalRoutes`.

**Flow** — `nextNode`, `choices`.

**Timing** — `autoAdvanceDelay` (0 = wait for the player).

**Notes** — never shown to the player.

### Flow priority when a line finishes

1. **Choices** — if any are available, the menu opens and the player decides.
2. **Conditional routes** — the first route whose conditions pass wins.
3. **Next Node** — plain linear continuation.
4. Nothing — the story ends.

A node with choices *and* a Next Node uses Next Node only as the fallback for
when every choice is hidden by its conditions, so a dead end is impossible.

### Node and story identity

Every node and choice carries a hidden GUID assigned once and stored in the
asset. Renaming or moving a node never breaks an existing save. Stories carry a
`storyId` for the same reason.

---

## Characters and expressions

### Adding a character

1. Art in `Assets/VisualNovel/Art/Characters/<Name>/`, named `<Name>_<Pose>.png`.
2. Select the sprites, set **Texture Type = Sprite (2D and UI)**.
3. `Create > Visual Novel > Character` in that folder.
4. Set Display Name and Default Side. Their name uses the theme's speaker colour
   unless you tick **Use Custom Name Color**.
5. Press **Auto-Fill Poses From Folder** — `Aria_Happy.png` becomes the tag
   `happy`. Or add rows by hand.

That folder layout only matters for the auto-fill button. Poses are matched by
**tag**, so you can organise your art however you like.

### Picking an expression

Select a Dialogue Node and click a face in the **thumbnail strip** at the top of
the Inspector. That sets the pose tag and clears any sprite override — no typing,
so the whole class of typo mistakes disappears. The chosen face also appears on
the node in the Story Graph.

### How the sprite is resolved

Only one sprite is ever shown:

1. If `characterSprite` is set, **it wins** and the pose tag is ignored.
2. Otherwise `emotionPose` is looked up on the Character asset.
3. A tag that matches nothing falls back to `neutral`, then to the first pose.

Leave `characterSprite` empty and use pose tags. Both failure modes are reported
rather than silent: the Inspector warns when an override is hiding a tag and
offers a one-click **Use pose instead**, a console warning names an unmatched tag
and lists the real ones, and Validate flags both.

### Three character slots

Left, centre and right. A node's `speakerSide` — or the character's Default Side
— decides which slot is used, and that also drives which side the name plate sits
on. A node that specifies no sprite leaves the slots untouched, so narration does
not wipe the actors off the stage; use `hideCharacter` or `clearOtherCharacters`
when you actually want someone gone. Non-speaking characters are dimmed.

---

## Backgrounds

Drop the image in `Assets/VisualNovel/Art/Backgrounds/`, set Texture Type =
Sprite (2D and UI), drag it into a node's **Background Sprite**.

Leaving it empty keeps whatever is already showing, so you only set it on the
node where the scene actually changes. Changes crossfade; setting the background
that is already on screen costs nothing.

---

## Variables, conditions and branching

Variables are created on demand, are case-insensitive, and reading one that was
never set returns a default instead of throwing. They hold a **bool**, a
**number** or **text**.

- **Starting values** — on the Story asset, `Starting Variables`.
- **Changing them** — a node's `On Enter Effects`, or a choice's `Effects`.
  Operations: Set, Add, Subtract, Toggle.
- **Reading them** — a choice's `Show Conditions`, or a node's
  `Conditional Routes`.

Conditions compare with `==`, `!=`, `>`, `>=`, `<`, `<=`, and a condition set can
require **all** or **any** of its conditions. An empty set always passes.

### Two kinds of branching

**Choices** are player-facing: a menu opens and they decide. A choice whose
conditions fail can either disappear (`Hide`) or show greyed out with alternate
text (`Show Disabled` + `Unavailable Text`, e.g. *"(You need a key)"*).
`Only Once` retires a choice after it is picked.

**Conditional routes** are invisible: the world simply reacts. Use them for
"if the player has the lantern, go to the lit-hallway node instead."

---

## Text variables

Write `{variableName}` anywhere in dialogue, a choice, or a speaker name:

```
Aria: Good to meet you, {playerName}. Affection is at {affection}.
```

It is substituted when the line is shown, so the typewriter counts the characters
the reader will actually see. An unknown name is left exactly as written, so rich
text and stray braces survive. Use `{{` for a literal brace.

---

## Screen effects

Two checkboxes on any Dialogue Node, under **Screen effects**:

- **Shake Screen** — jolts the background, characters and dialogue box, decaying
  to a stop. Menus are deliberately left alone so a shake cannot make one
  unreadable. Strength and duration are per node.
- **Flash Screen** — blinks the screen a colour and back. Never blocks input, so
  it cannot swallow a click the player was already making. Colour and duration
  are per node.

---

## Audio

Three channels — **Music**, **SFX**, **Voice** — working with no setup at all.
Music crossfades between tracks; a new voice line interrupts the previous one.

For real mixing:

1. `Create > Audio Mixer`, name it `VNMixer`.
2. Groups under Master: **Music**, **SFX**, **Voice**.
3. Right-click each group's Volume → *Expose to script*.
4. Rename the exposed parameters `MusicVolume`, `SFXVolume`, `VoiceVolume`.
5. Drag the mixer and groups onto **VN Audio Manager**.

Volumes are linear 0–1 and converted to decibels for you. Levels persist in
PlayerPrefs and are driven by the Settings menu. A character can also carry a
**typing blip** played while their text types out.

---

## Theming

One asset controls the look of the whole game:
`Assets/VisualNovel/Themes/DefaultTheme.asset`.

### The colours you author

| Colour | Used for |
| --- | --- |
| `background` | Screen backdrop, and the tint behind modal menus |
| `panel` | Dialogue box and menu panels |
| `accent` | Selection, slider fills, primary buttons |
| `outline` | Borders |
| `textPrimary` | Main text |
| `speakerNameColor` | The name plate, kept separate so a name never blends into the dialogue below it |

Everything else is **derived**: hover, pressed, selected, disabled, raised
panels, dividers, muted text, and readable text on the accent colour. That is
what keeps a deep-blue panel with a white outline looking deliberate everywhere
it appears instead of drifting into a different blue in the save menu. The
Inspector shows a live strip of every derived colour.

Individual characters override the speaker colour by ticking **Use Custom Name
Color**; otherwise everyone follows the theme.

### Everything else on the theme

- **Shape** — corner radius (0 = hard edges), outline width, pill buttons.
  Rounded corners are generated in memory and nine-sliced; you never import or
  draw a UI sprite.
- **Motion** — how menus enter (fade, fade-scale, slide up or down), open and
  close durations, button hover scale, press scale, **the colour a button flashes
  when clicked** and how long the flash lasts.
- **Type** — font asset and five sizes: title, body, button, small, speaker.
- **Sound** — hover, click and back clips for the whole interface.

Five presets ship in the Inspector: Midnight, Parchment, Charcoal, Rose Noir,
Forest Ink. Edits apply live in the Scene view — no Play mode.

### Theming your own UI

Add **Themed Graphic**, **Themed Text** or **Themed Button** and pick a role:

`Background · Panel · PanelRaised · Overlay · Outline · Divider · ButtonSurface ·
ButtonOutline · Accent · TextPrimary · TextMuted · TextOnAccent · TextTitle ·
SpeakerName`

A graphic declares what it *is*, never what colour it is. Themed components also
repaint themselves whenever they become active, because the generated corner
sprites do not survive a domain reload — without that, a menu page that happened
to be closed at the time would come back with square buttons. If anything ever
does look wrong, **Repaint Scene Now** on the theme asset fixes it.

---

## Menus

All generated by the wizard, all themed, all working:

- **Title** — New Game, Load, Settings, Quit
- **System** (Escape) — Resume, Save, Load, Backlog, Settings, Title, Quit
- **Save / Load** — 100 slots across 17 pages of six, plus dedicated Quick Save
  and Auto Save cards. Each card shows the slot, the date and the line it stopped
  on. Only the visible page is read from disk.
- **Settings** — Music, Sound and Voice volume, text speed, auto-advance pause,
  skip-unread, fullscreen, clear read history
- **Backlog** — every line so far, with a replay button on voiced lines
- **Confirm** — asks before overwriting a save, abandoning progress, or quitting

Escape opens the system menu when you are in the story, and backs out one layer
at a time when a page is open. Menu panels size themselves to their contents via
`VNMenuPanelFitter`, so adding an eighth button to a seven-button menu grows the
panel instead of pushing the last button out underneath the Back button.

### Adding a page of your own

Add a `VNMenuScreen` under `Menus` with an id, and open it with a
`VNMenuButtonRelay` set to **Open Screen**. Escape stacking, theming, animation
and sizing all come for free.

`VNMenuButtonRelay` wires a button to one action by name rather than through a
UnityEvent: `CloseTop`, `CloseAll`, `OpenScreen`, `OpenSave`, `OpenLoad`,
`OpenSettings`, `OpenBacklog`, `OpenMenu`, `NewGame`, `ReturnToTitle`, `Quit`,
`Advance`, `Back`, `ToggleSkip`, `ToggleAuto`, `QuickSave`, `QuickLoad`.

---

## Player controls

Every action has a key **and** an on-screen button in the quick bar.

| Action | Key | Button |
| --- | --- | --- |
| Advance | Click / tap, Space, Enter, gamepad A | click anywhere |
| Back (rollback) | Mouse wheel up, Backspace, PgUp, gamepad LB | `Back` |
| Menu | Escape, right click, gamepad Start | `Menu` |
| Skip | Hold Ctrl or RB, Tab to latch | `Skip` |
| Auto-advance | `A` | `Auto` |
| Backlog | `L` | `Log` |
| Hide interface | `H` | — |
| Quick save | `F5` | `QS` |
| Quick load | `F9` | `QL` |
| Save / Load pages | — | `Save` / `Load` |

**Skip** races through text the reader has already seen and **stops dead at the
first unread line**. What counts as read is remembered across playthroughs in
`vn_read.json` next to the saves. A setting lets readers skip unread text anyway.
Skip always stops at a choice, and mutes sound effects and voice while running.

**Back** restores the world exactly as it was, variables included, by
snapshotting state before each node rather than trying to undo effects. The
reader can rewind up to 200 beats to re-read something or take a different
branch. The backlog rewinds with it.

**Auto** waits for the voice line to finish, then for the reader's chosen pause.

**Hide interface** leaves the art on screen; any advance input brings it back.

Skip and Auto light up in the quick bar while running; Back greys out when there
is nothing to rewind to.

Left click and touch are handled by a full-screen transparent Advance Area button
rather than raw input, which is what stops a click on a choice from also
advancing the line underneath it.

---

## Save and load

100 manual slots (0–99), plus a reserved **Quick Save** slot and an **Auto Save**
slot written every time the player makes a choice — so a crash never costs more
than one decision.

Saves are JSON in `Application.persistentDataPath`, storing the story id, the
exact node, every variable, visited nodes, used choices and playtime. Loading
restores the saved scene first if it differs from the current one.

**For loading to find a story**, it must be in the VN Manager's `Registered
Stories` list or inside a `Resources` folder. Anything played through
`DialogueParser.Play` registers itself automatically.

---

## Preview, validation and search

### Story Preview

`Tools > Visual Novel > Story Preview` — step through a story without entering
Play mode. It runs the real rules: effects applied, conditions evaluated,
unavailable choices filtered. Edit variables live to jump straight to a branch.
**Back** rewinds by replaying the recorded route from the start, so every effect
stays applied exactly once. In Play mode, **Jump in Game** sends the running game
to the node you are looking at.

### Validate

On the Story asset, in the preview window, or
`Tools > Visual Novel > Validate Selected Story`. Reports:

- a missing start node
- dead ends and endings
- choices with no target node
- nodes nothing links to
- duplicate ids, which would break saves
- pose tags that match nothing
- sprite overrides that are silently hiding a pose tag
- nodes with no dialogue text

### Find in Story

`Tools > Visual Novel > Find in Story` searches every line, speaker name, choice
and note in a story, shows each match in context, and replaces across all of them
in one undoable step — for renaming a character or fixing a tic you used forty
times.

---

## Importing from a spreadsheet

`Tools > Visual Novel > Import Dialogue (CSV or JSON)`. Write in Google Sheets or
Excel, export CSV, press Import. Column names are case-insensitive and order does
not matter; only `id` and `text` are required.

```
id,speaker,character,pose,side,text,background,music,sfx,voice,next,set,auto,
choice1,choice1target,choice1if,choice1set,choice2,...
```

| Column | Meaning |
| --- | --- |
| `next` | id of the following row (blank ends the story) |
| `set` | `playerHasKey=true; affection+=1` |
| `choiceNif` | `trust>=2; playerHasKey==true` |
| `choiceNset` | effects applied when that choice is picked |
| `side` | `left` / `center` / `right` |
| `auto` | seconds to auto-advance |
| `background`, `music`, `sfx`, `voice`, `character` | asset names, found anywhere in the project |

Up to six choices per row. Re-importing the same file updates the existing node
assets in place rather than duplicating them, so a writer can keep editing the
sheet. JSON works too:

```json
{ "nodes": [
  { "id": "n1", "text": "Hello", "next": "n2",
    "choices": [ { "text": "Yes", "target": "n3" } ] }
] }
```

---

## Scripting reference

Nothing below is needed for a basic story. It is here for when you want to hook
your own systems in.

### VNCommands — every player action

All parameterless and public, so they drop straight onto a Button's OnClick list:

`Advance` · `Back` · `ToggleSkip` · `ToggleAuto` · `QuickSave` · `QuickLoad` ·
`OpenSave` · `OpenLoad` · `OpenMenu` · `CloseMenu` · `CloseAllMenus` ·
`OpenSettings` · `OpenBacklog` · `ToggleHideUI` · `NewGame` · `ReturnToTitle` ·
`QuitGame` · `ShowToast(string)`

### VNManager

```csharp
VNManager.Instance.Variables            // the global variable store
VNManager.Instance.Audio                // music / sfx / voice
VNManager.Instance.Theme                // assigning repaints everything live

VNManager.Instance.SaveGame(0);         // manual slots 0..99
VNManager.Instance.QuickSave();
VNManager.Instance.AutoSave();
VNManager.Instance.LoadGame(0);
VNManager.Instance.HasSave(0);
VNManager.Instance.PeekSave(0);
VNManager.Instance.DeleteSave(0);

VNManager.Instance.LoadScene("Chapter2", fadeDuration: 0.4f);
VNManager.Instance.Flash(Color.white, 0.3f);
VNManager.Instance.SetFadeColor(Color.black);
VNManager.Instance.ResetPlaythrough();
VNManager.Instance.RegisterStory(story);

VNManager.TextSpeedMultiplier = 1.5f;   // player settings, persisted
VNManager.AutoAdvanceDelay = 2f;
VNManager.SkipUnreadText = false;

VNManager.FindExisting();               // never creates one; safe in edit mode
```

### DialogueParser

```csharp
DialogueParser.Active.Play(story);
DialogueParser.Active.PlayFrom(story, node, resetVariables: false);
DialogueParser.Active.JumpTo(node);
DialogueParser.Active.Back();
DialogueParser.Active.Stop();

DialogueParser.Active.SkipMode = true;
DialogueParser.Active.AutoMode = true;
DialogueParser.Active.Backlog;          // every line so far
DialogueParser.Active.CanRollback;
DialogueParser.Active.HasVisited(node);

DialogueParser.Active.NodeEntered    += node   => { };
DialogueParser.Active.ChoicePicked   += choice => { };
DialogueParser.Active.StoryFinished  += story  => { };
DialogueParser.Active.ModeChanged    += ()     => { };  // skip/auto toggled
DialogueParser.Active.BacklogChanged += ()     => { };
```

### Other useful pieces

```csharp
VNSaveSystem.ReadPage(2);               // one page of slots for a menu
VNSaveSystem.MostRecent();              // for a Continue button
VNSaveSystem.ListAll();

VNText.Resolve("Hello {playerName}");   // substitute variables
VNText.StripRichText(line);             // plain text, for previews

VNReadHistory.IsRead(nodeId);           // what Skip uses
VNReadHistory.Clear();

story.Validate();                       // the same list the tools show
story.CollectReachableNodes();
```

---

## Troubleshooting

**A character's expression never changes.** Almost always one of two things, and
both are now reported. Either **Character Sprite** is set on the node — it beats
the pose tag, and the Inspector offers a one-click *Use pose instead* — or the
pose tag matches nothing and the fallback face is showing, in which case the
Console names the tag and lists the real ones. Run Validate to find every case at
once.

**Text does not render, or everything is pink.** TMP Essential Resources are not
imported: `Window > TextMeshPro > Import TMP Essential Resources`.

**A button looks square instead of rounded.** The generated corner sprites did
not survive a reload. Components repair themselves when they become active; if
one does not, press **Repaint Scene Now** on the theme asset.

**Loading a save says the story was not found.** Add the story to VN Manager's
`Registered Stories`, or move it into a `Resources` folder.

**Scene changes do nothing.** The target scene must be in
`File > Build Profiles > Scene List`.

**A menu button is hidden behind the Back button.** Add a **Menu Panel Fitter**
to the panel, with the panel in `Panel` and its `Body` child in `Content`.

**Changes to the generated scene did not appear.** Hierarchy and layout are
decided when the wizard runs. Delete `VN Manager` and `VN Canvas` and run
**Build VN Scene Setup** again — your stories, characters and theme live in the
Project and are not touched.

---

## Folder layout

```
Assets/VisualNovel/
  Runtime/
    Data/   Nodes, choices, stories, characters, theme, variables, conditions
    Core/   VNManager, DialogueParser, audio, saves, read history, input, text
    UI/     Dialogue box, characters, choices, menus, quick bar, theming
  Editor/   Setup wizard, story graph, preview, importer, find, inspectors
  Themes/   Your theme assets
  Samples/  The generated sample story
```
