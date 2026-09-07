# FastVisualNovel
A Visual Novel tool that let's you make a whole system via a single click, and then just focus on the story.


# Visual Novel Engine

A lightweight, modular visual novel engine for Unity, built for writers and
artists. Stories are data, menus are generated, and the look of the entire game
comes from five colours on one asset.

Built and verified against **Unity 6000.6.0f1**, URP, **Input System (new)**, and
TextMeshPro (which ships inside `com.unity.ugui` 2.x).

---

## 60-second start

1. `Tools > Visual Novel > Build VN Scene Setup` — builds the scene *and every
   menu*: title, system, save, load, settings, backlog, confirm, quick bar.
2. `Tools > Visual Novel > Create Sample Story`
3. Drag the story onto **Story Player** on `VN Manager`.
4. Press Play.

> The first run may ask to import **TMP Essential Resources**. Accept, wait for
> the import, then run the wizard again.

Then open `Tools > Visual Novel > Story Graph` and write.

---

## The Story Graph

`Tools > Visual Novel > Story Graph`, or the button on any Story asset.

This is where you write. The whole novel is one map.

| Action | How |
| --- | --- |
| Create a linked node | Drag from a node's right-hand dot into empty space |
| Link two nodes | Drag from a dot onto another node |
| Add a node | Right-click the canvas |
| Add a choice / route | Right-click a node |
| Move nodes | Drag them (multi-select with Ctrl or Shift) |
| Pan | Middle-drag, or Alt + drag |
| Zoom | Scroll wheel |
| Frame everything | `F`, or the Frame All button |
| Tidy the layout | Auto Layout — columns by distance from the start |
| Delete | `Delete` key, or right-click > Delete |

Ports are colour-coded to match the links: **grey = next**, **blue = choice**,
**orange = conditional route**. The start node is green.

**Nodes never need dragging into the story.** Anything created in the graph is
added to the story's node list automatically, as a sub-asset of the story file —
so a whole novel stays one asset in the Project window instead of a hundred loose
files. Deleting a node scrubs every link that pointed at it.

**Watching a playthrough.** Enter Play mode with the graph open: the node the
reader is on glows gold, and every node they have been through gets a marker. You
can see the path a player actually took through your branches.

### Inherit look

The **Inherit look** toggle in the graph toolbar (also on every node's Inspector)
is on by default. With it on, a new node copies the background, character, pose,
side and music track of the node it came from — the fields that are the same on
almost every line of a scene. Text, choices, links and variable effects are never
copied; those are what make a node itself.

Music is carried but its action is reset to *No Change*, so a repeated track does
not restart on every line.

---

## Theming

One asset controls the look of the whole game: `Assets/VisualNovel/Themes/DefaultTheme.asset`.

You author **five colours**:

| Colour | Used for |
| --- | --- |
| `background` | Screen backdrop, and the tint behind modal menus |
| `panel` | Dialogue box and menu panels |
| `accent` | Selection, slider fills, primary buttons |
| `outline` | Borders |
| `textPrimary` | Main text |

Plus a **speaker name colour**, kept separate so a character's name never blends
into the dialogue underneath it. Individual characters can override it by ticking
**Use Custom Name Color**; otherwise everyone follows the theme.

Everything else is **derived**: hover, pressed, selected, disabled, raised panels,
dividers, muted text, and readable text on the accent colour. That is what keeps
a deep-blue panel with a white outline looking deliberate everywhere it appears
instead of drifting into a different blue in the save menu. The Inspector shows a
live strip of every derived colour so you can see what your five choices produce.

Also on the theme:

- **Shape** — corner radius (0 = hard edges), outline width, pill-shaped buttons.
  Rounded corners are generated in memory and nine-sliced; you never import or
  draw a UI sprite.
- **Motion** — how menus enter (fade, fade-scale, slide up/down), open and close
  durations, button hover scale, press scale, and **the colour a button flashes
  when clicked** plus how long the flash lasts.
- **Type** — font asset and five sizes (title, body, button, small, speaker).
- **Sound** — hover, click and back clips for the whole interface.

Five presets ship in the inspector: Midnight, Parchment, Charcoal, Rose Noir,
Forest Ink. Edits apply live in the Scene view — no Play mode.

To theme something you built yourself, add **Themed Graphic**, **Themed Text** or
**Themed Button** to it and pick a role. It will follow the theme from then on.

Themed components repaint themselves whenever they become active. Rounded-corner
sprites are generated at runtime and do not survive a domain reload, so without
that a menu page which happened to be closed at the time would come back with
square buttons. If anything ever does look wrong, **Repaint Scene Now** on the
theme asset fixes it.

---

## Player controls

Every action has a key **and** an on-screen button in the quick bar.

| Action | Key | Button |
| --- | --- | --- |
| Advance | Click / tap, Space, Enter, gamepad A | click anywhere |
| Back (rollback) | Mouse wheel up, Backspace, PgUp, gamepad LB | `Back` |
| Menu | Escape, right click, gamepad Start | `Menu` |
| Skip | Hold Ctrl, or Tab to latch | `Skip` |
| Auto-advance | `A` | `Auto` |
| Backlog | `L` | `Log` |
| Hide interface | `H` | — |
| Quick save | `F5` | `QS` |
| Quick load | `F9` | `QL` |
| Save / Load pages | — | `Save` / `Load` |

**Skip** races through text the reader has already seen and **stops dead at the
first unread line**. What counts as read is remembered across playthroughs in
`vn_read.json`, next to the saves. A setting lets readers who insist skip unread
text too. Skip always stops at a choice.

**Back** restores the world exactly as it was — variables included — by
snapshotting state before each node rather than trying to undo effects. The
reader can rewind up to 200 beats, re-read something, or take a different branch.
The backlog rewinds with it.

**Auto** waits for the voice line to finish, then for the reader's chosen pause.

Skip and Auto light up in the quick bar while running; Back greys out when there
is nothing to rewind to.

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

An **auto-save** is written every time the player makes a choice, so a crash never
costs more than one decision.

Menu panels size themselves to their contents via `VNMenuPanelFitter`, so adding
an eighth button to a seven-button menu grows the panel instead of pushing the
last button out under the Back button.

To add a page of your own: add a `VNMenuScreen` under `Menus` with an id, and
open it with a `VNMenuButtonRelay` set to Open Screen. Escape stacking, theming,
animation and sizing come for free.

---

## The data model

| Asset | What it is |
| --- | --- |
| **StoryAsset** | One story: start node, every node in it, starting variables |
| **DialogueNode** | One beat: speaker, text, sprites, pose, audio, effects, links |
| **ChoiceNode** | One button: text, target, availability conditions, effects |
| **CharacterAsset** | A cast member: name, colour, default side, named poses |
| **VNTheme** | The look of the entire game |

### Flow priority when a line finishes

1. **Choices** — if any are available, the menu opens.
2. **Conditional Routes** — the first route whose conditions pass wins.
3. **Next Node** — plain linear continuation.
4. Nothing — the story ends.

A node with choices *and* a Next Node uses Next Node only as the fallback for when
every choice is hidden by its conditions, so dead ends are impossible.

### Expressions

Pick expressions from the **thumbnail strip** at the top of a Dialogue Node's
Inspector. Clicking a face sets the pose tag and clears any sprite override, so
the whole class of typo mistakes goes away. The chosen face also appears on the
node in the Story Graph.

Only one sprite is ever shown. `characterSprite` is a manual **override that
beats the pose tag**; otherwise `emotionPose` is looked up on the Character asset.
Leave the override empty and use pose tags.

If an expression ever looks stuck, it is almost always one of these two, and both
are now reported rather than silent:

- **Character Sprite is set.** It wins over the pose tag. The Inspector shows a
  warning with a one-click "Use pose instead", and Validate flags it.
- **The pose tag does not match.** The character falls back to `neutral`, then to
  their first pose, and a console warning names the tag and lists the real ones.

"Inherit look" deliberately does **not** copy `characterSprite` — inheriting an
override would freeze every downstream node on one expression.

---

## Text variables

Write `{variableName}` anywhere in dialogue, a choice, or a speaker name and it is
replaced with the current value when the line is shown:

```
Aria: Good to meet you, {playerName}. Affection is at {affection}.
```

An unknown name is left exactly as written, so rich-text markup and stray braces
survive. Use `{{` for a literal brace.

---

## Screen effects

Two checkboxes on any Dialogue Node, under **Screen effects**:

- **Shake Screen** — jolts the background, characters and dialogue box, decaying
  to a stop. Menus are left alone. Strength and duration are per node.
- **Flash Screen** — blinks the screen a colour and back. Never blocks input, so
  it cannot swallow a click. Colour and duration are per node.

---

## Global variables

Created on demand, case-insensitive, and reading one that was never set returns a
default rather than throwing.

- **Starting values** — on the Story asset, `Starting Variables`
- **Changing them** — a node's `On Enter Effects`, or a choice's `Effects`
  (Set / Add / Subtract / Toggle)
- **Reading them** — a choice's `Show Conditions`, or a node's `Conditional Routes`

A choice whose conditions fail can disappear (`Hide`) or show greyed out with
alternate text (`Show Disabled` + `Unavailable Text`, e.g. *"(You need a key)"*).

---

## Adding a character

1. Art in `Assets/VisualNovel/Art/Characters/<Name>/`, named `<Name>_<Pose>.png`.
2. Set **Texture Type = Sprite (2D and UI)**.
3. `Create > Visual Novel > Character` in that folder.
4. Set Display Name and Default Side. Their name uses the theme's speaker
   colour unless you tick **Use Custom Name Color**.
5. Press **Auto-Fill Poses From Folder** — `Aria_Happy.png` becomes the tag `happy`.
6. On a node: drag in the Character, type a pose tag.

## Adding a background

Drop the image in `Assets/VisualNovel/Art/Backgrounds/`, set Texture Type =
Sprite (2D and UI), drag it into a node's **Background Sprite**. Leaving it empty
keeps whatever is already showing, so you only set it where the scene changes.

---

## Audio

Music, SFX and Voice work with no setup. For real mixing:

1. `Create > Audio Mixer`, name it `VNMixer`.
2. Groups under Master: **Music**, **SFX**, **Voice**.
3. Right-click each group's Volume → *Expose to script*.
4. Rename the exposed parameters `MusicVolume`, `SFXVolume`, `VoiceVolume`.
5. Drag the mixer and groups onto VN Audio Manager.

Volumes are linear 0–1 and converted to decibels for you; levels persist in
PlayerPrefs and are driven by the Settings menu.

---

## Save / load

```csharp
VNManager.Instance.SaveGame(0);     // manual slots 0..99
VNManager.Instance.QuickSave();     // F5 slot
VNManager.Instance.AutoSave();      // written on every choice
VNSaveSystem.ReadPage(2);           // one page for a menu
VNSaveSystem.MostRecent();          // for a Continue button
```

JSON in `Application.persistentDataPath`, storing the story id, exact node, every
variable, visited nodes, used choices and playtime. Loading restores the scene
first if the save was made in a different one.

**For loading to find a story**, it must be in the VN Manager's `Registered
Stories` list or inside a `Resources` folder. Anything played through
`DialogueParser.Play` registers itself.

---

## Preview Mode

`Tools > Visual Novel > Story Preview` — step through a story without Play mode.
Runs the real rules: effects applied, conditions evaluated, unavailable choices
filtered. Edit variables live to jump to a branch; **Back** rewinds by replaying
the route so effects stay exact.

**Validate** reports dead ends, choices with no target, unreachable nodes,
duplicate ids, missing pose tags and sprite overrides that are hiding a pose.

## Find and replace

`Tools > Visual Novel > Find in Story` searches every line, speaker name, choice
and note in a story, and replaces across all of them in one undoable step — for
renaming a character or fixing a tic you used forty times.

---

## Importing dialogue from a spreadsheet

`Tools > Visual Novel > Import Dialogue (CSV or JSON)`. Column names are
case-insensitive and order does not matter; only `id` and `text` are required.

```
id,speaker,character,pose,side,text,background,music,sfx,voice,next,set,auto,
choice1,choice1target,choice1if,choice1set,choice2,...
```

| Column | Meaning |
| --- | --- |
| `next` | id of the following row (blank = ends the story) |
| `set` | `playerHasKey=true; affection+=1` |
| `choice1if` | `trust>=2; playerHasKey==true` |
| `side` | `left` / `center` / `right` |
| `auto` | seconds to auto-advance |
| `background`, `music`, `sfx`, `voice`, `character` | asset names, found anywhere in the project |

Re-importing updates existing nodes in place rather than duplicating them.

---

## Folder layout

```
Assets/VisualNovel/
  Runtime/
    Data/     Nodes, choices, stories, characters, theme, variables, conditions
    Core/     VNManager, DialogueParser, audio, save system, read history, input
    UI/       Dialogue box, characters, choices, menus, quick bar, theming
  Editor/     Setup wizard, story graph, preview, importer, inspectors
```

No assembly definitions — everything lands in `Assembly-CSharp`, so your own
scripts can use it with no extra setup.

---

## Extending it

The layers are deliberately separate:

- **Data** knows nothing about UI or scenes.
- **`VNUIController`** is the only thing `DialogueParser` talks to.
- **`VNCommands`** is the only thing the keyboard and buttons talk to — every
  method is public, parameterless and Inspector-wireable.
- **`VNManager`** owns everything global: variables, audio, transitions, saves, theme.

```csharp
DialogueParser.Active.NodeEntered   += node   => { };
DialogueParser.Active.ChoicePicked  += choice => { };
DialogueParser.Active.StoryFinished += story  => { };
DialogueParser.Active.ModeChanged   += ()     => { };  // skip/auto toggled
DialogueParser.Active.BacklogChanged += ()    => { };
```
