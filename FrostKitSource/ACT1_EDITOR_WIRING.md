# Act I — wiring in FrostKit Editor

**Goal:** convert the manifest into native FrostKit assets without inventing Blueprint API.
**Canonical campaign input:** `../Docs/FROSTPUNK2_CAMPAIGN_NARRATIVE_AUDIT_v0.1.md`.
**Technical input:** `../Docs/FROSTKIT_OFFLINE_WIKI_2026-11-19.md`.

> **Stop condition:** if an exact Blueprint base class, node or property differs from this document, stop and record the real editor fact in `EDITOR_VERIFICATION_LOG.md`. Do not approximate it from stock Unreal knowledge.

---

## 0. Preconditions

- FrostKit and a legal Frostpunk 2 install are on the same Windows machine.
- Both Editor Settings and Mod Cooker `Game Directory` point to the folder containing `Frostpunk2.exe`; path has no whitespace (official PIE guide).
- The version of FrostKit, game version, sample scenario asset root and the mod Content root have been recorded in the verification log.
- Create all new assets under one stable root, intended in this package as:

```text
/Game/Scenarios/LegitimateHeirs
```

Do not edit shared/base content in place. Duplicate a sample asset into this root, rename it, then update only the duplicate. For a compatible base-value modification, use Data Injection rather than direct edit.

---

## 1. Boot before narrative

Create the `native_scenario_chain` from `LegitimateHeirs.editor-manifest.yaml` in this order:

1. World, Gameplay and Environment levels.
2. Add to the World level the basic shared sublevels required by the current FrostKit sample (HoD, IdeaTree, Cameras and any currently required companions). Record exact paths.
3. Place at least one `BP_CityArea` and at least one `BP_Territory` in the Gameplay level.
4. Create `PXFrostlandTileType`, then FrostlandScenario, RandomizationConfig, LevelConfig, Nation Type/Config, City Type/Area Type, StartupConfig, RuntimeSettings, PXScenario.
5. Point RuntimeSettings back to the PXScenario as the official scenario guide specifies.
6. Save all, run PIE and reach a playable city **before** adding a dialog, law or quest.

**Pass:** map boots, city is visible, no crash on start.
**Fail:** fix only the scenario chain; do not bury a boot error under Story Arc work.

---

## 2. Create and prove state storage

1. In the current FrostKit sample, locate its VariableStorage Blueprint. Duplicate it as `BP_LH_VariableStorage_Campaign`.
2. Record its parent class and how it defines/reads/writes bools and ints.
3. Add all booleans/integers from `state_model` in the manifest.
4. Attach this storage to `DA_LH_Scenario` through its Scenario Variable/Constants Storage field.
5. Make a **throwaway diagnostic Scenario Script** that sets a visible harmless test state, logs it/uses existing debug inspector if available, save-loads, and confirms persistence. Remove the diagnostic action afterwards.

### 2.1. Fresh-run initialization rule

Run initialization only when a newly created scenario has no campaign-start marker. It must set:

```text
FatherDeathSecret = true
AnniversaryUltimatumActive = false
AgeAmendmentPassed = false
LegalSuccessionComplete = false
Act1Complete = false
CurrentWeek = 1                    # only if current-week value must be mirrored
AnniversaryWeek = 52
UltimatumDeadlineWeek = 104
AmendmentRequiredPercent = 67
Act = 1
```

It must **not** reset promises, votes or Act completion when a saved scenario is loaded. Prove this with save/load.

---

## 3. Build the smallest end-to-end spine before three faction lines

### 3.1. Duplicate existing sample assets; never invent class names

From the current FrostKit sample locate and duplicate one each of:

| Needed LH asset | Copy from | Record before editing |
|---|---|---|
| `BP_LH_ScenarioScript_Act1` | sample Scenario Script | parent class, scenario-start event, timer/week access |
| `BP_LH_FlowHub_Act1` | sample FlowHub | parent class, sequencing/wait/dialog nodes |
| `BP_LH_Quest_RemoveAgeCensus` | sample Quest | parent/objective types and completion/failure hooks |
| `BP_LH_StoryArc_AnniversaryUltimatum` | sample Story Arc | start condition and end condition hooks |
| Dialog/Dilemma assets | closest sample dialog/dilemma | choice/action schema, localization fields |
| Unlock assets | sample Unlock | exact activation and requirement model |
| Script Pack | sample Script Pack | add/activate model |

This is deliberate: current public docs confirm the *systems* but not every graph node. The duplicate is the executable source of truth.

### 3.2. Vertical slice state machine

Implement this first using one temporary debug action which sets all three faction lines/votes complete. It validates the central clock and law without writing unapproved faction debts.

```text
BOOT
  → ACT1_ARRIVAL_01
  → main quest active
  → [wait for CurrentWeek >= 52]
  → ACT1_ANNIVERSARY_52, deadline=104
  → [support reaches >=67]
  → open age-amendment policy vote
     ├─ passes → LEGAL_SUCCESSION → FIRST_COURSE → ACT 2 UNLOCK
     └─ rejected/insufficient → quest remains active

At every week tick after week 52:
  if CurrentWeek >= 104 and not AgeAmendmentPassed:
      MISSED_ULTIMATUM → verified scenario failure/end
```

The original design says the first 52 weeks do **not** show a visible countdown; the ultimatum becomes visible at week 52 and lasts 52 weeks. Do not start a 104-week visible timer at boot.

### 3.3. Week source

First try to **read an existing scenario week/statistic**. A manually incremented `CurrentWeek` is fallback only if the current FrostKit sample exposes no safe read. Record the source and whether tick fires when fast-forwarding/loading. The weekend/time unit must be actual campaign weeks; do not change game speed or use construction duration as a proxy.

### 3.4. Game over

The design requires hard Game Over. Public guidance does not document an end-game node. Find the current sample/base scenario’s verified fail/end implementation and duplicate its route. If current editor exposes only quest-failed UI and no scenario end, record that limitation; do not label a message box as Game Over.

---

## 4. Main quest

Create `BP_LH_Quest_RemoveAgeCensus` by duplicating a current sample quest.

**Player-facing title:** `Снятие возрастного ценза`.

**Begin:** after `LH_ACT1_ARRIVAL_01` concludes.

**Stages:**

| Stage | Visible timing | Completion signal |
|---|---|---|
| Find political support | start | any completed line exposes its promise negotiation |
| Build support | after one or more promises | `AmendmentSupportPercent` reaches 67 |
| Deadline | only after Act I week 52 scene | `AgeAmendmentPassed`; failure at week 104 |

Do not require the player to complete all three lines if verified faction vote weights make 67% achievable with fewer. The design says the player must collect enough agreements, but it does not assign numerical weights; those weights must be authored/approved and test-balanced, not invented here.

Quest succeeds only on `AgeAmendmentPassed`, not merely `support >=67`: the actual Law must pass.

---

## 5. The age-census Law category

Follow the official policy hierarchy:

```text
DA_LH_Policy_PoliticsAfterFifty
DA_LH_Policy_VoteAfterThirty
  → DA_LH_Law_AgeCensus
  → DA_LH_LawGroup_AgeCensus
  → DA_LH_LawCategory_AgeCensus
  → DA_LH_NationType Initial Main Categories (only if visibility test passes)
```

### 5.1. Required fields

- Start policy: `Политика после пятидесяти`, `Can be Default=true`.
- Amendment: `Право голоса после тридцати`, **Required Yes Percentage = 67**.
- Both policies use **Custom Gameplay Effect Text** to disclose scenario-managed effects; do not pretend age filtering is a native policy modifier.
- The amendment's on-activate action / Scenario Script must set `AgeAmendmentPassed=true` only after the successful vote result is received.

### 5.2. Voting verification

The public guide documents `Is Charter Policy` and relation thresholds, but not age-based delegates or custom vote shares. Prove these questions in a throwaway law:

1. Does `Required Yes Percentage=67` visibly and mechanically require 67% in the current game?
2. Does `Is Charter Policy` cause relation-based voting for the current scenario nation?
3. What vote input can be changed by a custom group/promise without modifying base global assets?
4. Does a failed proposal remain eligible, and on what cadence?

Only then connect promise state to support. If no native hook exists, preserve the law as UI/council vote and use a documented scenario-script gate that only puts it on agenda after commitments. Never falsify the policy's vote count in text.

### 5.3. Usurpation

The campaign says unilateral repeal is impossible and causes expulsion/failure. Do not expose a misleading direct “repeal” button. If a specific design/UI path intentionally allows this choice, its action sets `UsurpationAttempted=true` and runs the same verified failure/end route. It does **not** pass `VoteAfterThirty`.

---

## 6. Faction lines and promises

The known parts of all three lines are structurally identical; their **specific debts/tasks are explicitly open canon**. Build the shared state and negotiation gates now, but do not invent 2–3 tasks per group.

### 6.1. Completion and promise contract

```text
if <Group>LineComplete && !<Group>PromiseGiven:
    unlock/open DA_LH_Dilemma_Promise<Group>

On player acceptance:
    <Group>PromiseGiven = true
    <Group>VotesCommitted = true
    recalculate AmendmentSupportPercent via verified group-vote rule

On refusal/defer:
    no commitment
    do not silently award votes
```

Each promise must have a persistent future-cost record for Act II. The exact content is open canon; create a placeholder comment/field `Promise_<Group>_Terms_TBD` rather than fake text.

### 6.2. Required native assets

| Group | Completion quest | Promise dilemma | Unlock |
|---|---|---|---|
| Порубежники | `BP_LH_Quest_BorderlandersLine` | `DA_LH_Dilemma_PromiseBorderlanders` | `DA_LH_Unlock_BorderlandersNegotiation` |
| Стражи | `BP_LH_Quest_GuardsLine` | `DA_LH_Dilemma_PromiseGuards` | `DA_LH_Unlock_GuardsNegotiation` |
| Основатели | `BP_LH_Quest_FoundersLine` | `DA_LH_Dilemma_PromiseFounders` | `DA_LH_Unlock_FoundersNegotiation` |

Do not create custom factions/communities until the editor audit identifies the real faction data model, relationships and UI registration route. State flags and dialogs are sufficient to prove Act I campaign flow.

---

## 7. Restoration packages and the Twelve Seals

The written design contains two intentional constraints that conflict with a naïve complete law implementation:

- It defines twelve start seals and three four-law restoration packages.
- It explicitly says **do not implement** the draft custom Act-I laws in §1.3 and use base-game laws instead.

Implementing twelve custom native policies plus twelve custom replacements now would violate the second instruction. Therefore the only safe implementation order is:

1. Keep base-game laws as the current Act-I laws.
2. Make a restoration package a Scenario Dialog/Dilemma/Quest milestone only after checking whether the current engine permits selecting/replacing the exact base policies in a custom scenario.
3. If it does, map every approved wave to four exact existing assets in an editor table; if it does not, model package completion as story state and do **not** claim it changed four policies.
4. Do not create §1.3 draft effects until author direction changes.

The editor table must be added only after inspection, because asset paths/names are versioned game content.

---

## 8. Week 52 and week 104

### 8.1. Anniversary event

`BP_LH_StoryArc_AnniversaryUltimatum` starts exactly once when all are true:

```text
CurrentWeek >= 52
AnniversaryUltimatumActive == false
Act1Complete == false
```

It opens `DA_LH_Dialog_AnniversaryUltimatum`; on completion:

```text
AnniversaryUltimatumActive = true
UltimatumDeadlineWeek = 104
activate/update main-quest visible deadline
```

The event is a private meeting. It must **not** set `FatherDeathSecret=false` or announce the death publicly.

### 8.2. Deadline

At every verified week-update after ultimatum:

```text
if CurrentWeek >= 104 && !AgeAmendmentPassed:
    fail BP_LH_Quest_RemoveAgeCensus
    run verified scenario fail/end
```

If the vote passes in the same update/threshold boundary, define and test ordering; desired design outcome is pass before the deadline succeeds, after deadline fails. Log the tested boundary (`week 103.99 / week 104`) in verification log.

---

## 9. Legal succession and Act I final choice

On a successful amendment result:

1. `AgeAmendmentPassed=true`.
2. Complete the main quest.
3. Run `BP_LH_StoryArc_LegalSuccession`.
4. Set `FatherDeathSecret=false` only here.
5. Open `DA_LH_Dialog_LegalSuccession` (public announcement/political transfer).
6. Open `DA_LH_Dilemma_FirstCourse`.

`FirstCourse` needs exactly three mutually exclusive branches:

| Choice | Persistent flag | Act II intended party |
|---|---|---|
| Порубежники | `CourseBorderlanders=true` | Закалённые |
| Стражи | `CourseGuards=true` | Дети Генератора |
| Основатели | `CourseFounders=true` | Столпы |

Enforce invariant:

```text
count(CourseBorderlanders, CourseGuards, CourseFounders) == 1
```

Then set `LegalSuccessionComplete=true`, `Act1Complete=true`, `Act=2`, activate `DA_LH_Unlock_Act2`.

The docs state the chosen group will be fanatically loyal and offer a unique bonus/cost in Act II, but do not define the three bonuses/costs. Do not manufacture them yet.

---

## 10. Test matrix for Act I

| Test | Setup | Expected result |
|---|---|---|
| A1 boot | new scenario | world, city, scenario load |
| A2 intro | fresh game | secret death intro exactly once; main quest active |
| A3 persistence | save after intro, reload | no state reset |
| A4 anniversary | time to week 52 | private event once; visible deadline starts |
| A5 no early public reveal | reach week 52 without amendment | secret remains true |
| A6 promise gate | line incomplete | no promise dialog/votes |
| A7 commitment | line complete, accept promise | persistent promise + votes state only once |
| A8 insufficient support | submit/attempt before 67 | no legal succession |
| A9 67% law | verified support ≥67 | actual policy vote uses 67% and success sets amendment flag |
| A10 deadline | reach week 104 without pass | actual hard scenario failure/end |
| A11 edge ordering | pass at deadline boundary | result follows documented/tested order |
| A12 finale | legal pass | death public, exactly one course selected, Act 2 unlocked |
| A13 cooked build | cook then launch outside editor | same flow and no missing assets |

Do not start Act II/III data authoring until A1–A13 pass.
