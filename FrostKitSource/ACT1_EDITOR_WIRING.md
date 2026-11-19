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
Act = 1
# 67% is native Rule-branch behaviour; do not store a parallel threshold.
```

It must **not** reset promises, votes or Act completion when a saved scenario is loaded. Prove this with save/load.

---

## 3. Build the smallest end-to-end spine before three faction lines

### 3.1. Duplicate existing sample assets; never invent class names

From the current FrostKit sample locate and duplicate one each of:

| Needed LH asset | Exact asset to duplicate | Static evidence / record before editing |
|---|---|---|
| `BP_LH_ScenarioScript_Act1` | `FrostkitSampleScenario/BlueprintsSerialized/ScenarioScripts/BP_FrostkitSample_UnlockFrostland` | confirmed `PXScenarioScript`; record scenario-start event and timer/week access |
| `BP_LH_FlowHub_Act1` | `FrostkitSampleScenario/BlueprintsSerialized/BP_FrostkitSample_FlowHub` | confirmed StartQuest, WaitForQuest, predefined dialog and SetUnlockActive calls; record parent/graph pins |
| `BP_LH_Quest_RemoveAgeCensus` | `FrostkitSampleScenario/BlueprintsSerialized/Quests/BP_FrostkitSample_Basics` | confirmed `PXQuest`; record objective types and completion/failure hooks |
| `BP_LH_StoryArc_AnniversaryUltimatum` | `FrostkitSampleScenario/BlueprintsSerialized/StoryArks/BP_FrostkitSample_ColdSnops` | confirmed `BP_Standard_StoryArc` reference, `GetCurrentGameTime` and delayed quest/dialog pattern |
| Dialog assets | `Configs/Messages/Dialogs/DA_FrostkitSample_BasicsMessage` / `ProvideCoal` / `EndMessage` | record action schema and localization fields |
| Dilemma assets | `Configs/Dilemmas/DA_FrostkitSample_Generic_StockpileMaterials5` | confirmed `PXDilemmaType` / `PXDilemmaChoice`; record choice action schema |
| Unlock assets | `Configs/Unlocks/DA_FrostkitSampleGameStart` | exact activation and requirement model |
| Script Pack | `Configs/ScriptPacks/DA_FrostkitSample_MainSP` | add/activate model |

This is deliberate: current public docs confirm the *systems* but not every graph node. The duplicate is the executable source of truth.

### 3.2. Vertical slice state machine

Implement this first using one temporary debug action which sets all three faction lines/votes complete. It validates the central clock and law without writing unapproved faction debts.

```text
BOOT
  → ACT1_ARRIVAL_01
  → main quest active
  → [wait for CurrentWeek >= 52]
  → ACT1_ANNIVERSARY_52, deadline=104
  → [v0.1 Borderlanders promise unlocks the agenda]
  → open native Rule-branch age-amendment vote (67% is native)
     ├─ passes → LEGAL_SUCCESSION → FIRST_COURSE → ACT 2 UNLOCK
     └─ rejected → quest remains active

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
| Find political support | start | v0.1 Borderlanders line exposes its promise negotiation |
| Place amendment on agenda | after the Borderlanders promise | `AgeAmendmentAgendaUnlocked=true` |
| Council vote | after agenda unlock | native Rule-branch Council result; 67% is not mirrored in custom storage |
| Deadline | only after Act I week 52 scene | `AgeAmendmentPassed`; failure at week 104 |

The author explicitly rejected invented faction vote weights. `Политика после пятидесяти` / `Право голоса после тридцати` must be one Law in the existing vanilla **Rule** branch, where the actual Council requires 67% for all laws. The promise must use the current game's verified standing/agenda mechanism; it is not permission to spoof a vote result.

Quest succeeds only on `AgeAmendmentPassed`, never merely on a promise/agenda gate: the actual Rule Law must pass.

---

## 5. The age-census Law category

Put a new Law into the existing vanilla **Rule** branch. Do not make a second custom Rule category or group:

```text
DA_LH_Policy_PoliticsAfterFifty
DA_LH_Policy_VoteAfterThirty
  → DA_LH_Law_AgeCensus
  → existing base-game Rule Law Group
  → existing base-game Rule Law Category
```

Locate the exact current Rule group/category through `PXPolicyConfig` in the editor, duplicate only the policies/law needed for the LH scenario, and record the paths in the verification log.

### 5.1. Required fields

- Start policy: `Политика после пятидесяти`, `Can be Default=true`.
- Amendment: `Право голоса после тридцати`; it must inherit the **native 67% Rule-branch** Council requirement. Verify the UI and actual vote before exposing it.
- Both policies use **Custom Gameplay Effect Text** to disclose scenario-managed effects; do not pretend age filtering is a native policy modifier.
- The amendment's on-activate action / Scenario Script must set `AgeAmendmentPassed=true` only after the successful vote result is received.

### 5.2. Voting verification

The public guide documents `Is Charter Policy` and relation thresholds, but not age-based delegates or custom vote shares. The author selected the native Rule route, so prove these questions in a throwaway Law in the existing Rule branch:

1. Does every current Rule-law UI/behaviour visibly and mechanically require 67% in this game version?
2. Can the LH law be attached to Rule without changing shared/base categories globally?
3. What native standing/relation/unlock route can make the Borderlanders promise unlock the law's agenda without modifying base global assets?
4. Does a failed proposal remain eligible, and on what cadence?

If a promise can only gate the agenda and cannot alter votes, preserve that truth in the UI. Never create an artificial support percentage or claim that an unlock guarantees a passing Council vote.

### 5.3. Usurpation

The campaign says unilateral repeal is impossible and causes expulsion/failure. Do not expose a misleading direct “repeal” button. If a specific design/UI path intentionally allows this choice, its action sets `UsurpationAttempted=true` and runs the same verified failure/end route. It does **not** pass `VoteAfterThirty`.

---

## 6. Faction lines and promises

Version 0.1 implements only the **Borderlanders** line. The author explicitly allowed the agent to write a provisional v0.1 task chain and promise; it must retain the `LH_V01_PROVISIONAL_` prefix and cannot be silently folded into fixed canon. Guards and Founders stay deferred.

### 6.1. Completion and promise contract

```text
if BorderlandersLineComplete && !BorderlandersPromiseGiven:
    unlock/open DA_LH_Dilemma_PromiseBorderlanders

On player acceptance:
    BorderlandersPromiseGiven = true
    BorderlandersVotesCommitted = true
    AgeAmendmentAgendaUnlocked = true
    apply only the verified native Rule-vote standing/agenda effect

On refusal/defer:
    no promise
    no agenda unlock
```

The promise must retain a persistent future-cost flag for Act II. Its detailed text is provisional v0.1 authoring, not fixed canon. It must never calculate or display fake support percentages.

### 6.2. Required native assets for v0.1

| Group | Completion quest | Promise dilemma | Unlock | Status |
|---|---|---|---|---|
| Порубежники | `BP_LH_Quest_BorderlandersLine` | `DA_LH_Dilemma_PromiseBorderlanders` | `DA_LH_Unlock_BorderlandersNegotiation` | **Implement**; provisional narrative IDs carry `LH_V01_PROVISIONAL_` |
| Стражи | — | — | — | deferred |
| Основатели | — | — | — | deferred |

Do not create custom factions/communities until the editor audit identifies the real faction data model, relationships and UI registration route. The v0.1 promise is a verified Rule-agenda/standing gate, not a false custom vote counter.

---

## 7. Restoration packages and the Twelve Seals

The author clarified the intended implementation:

- all twelve §1.2 seals are **custom start policies** and are active from the first week without a vote;
- a seal belongs to its existing base-game Law question;
- when the player replaces a seal, the replacement choices are existing base-game policies in that same question;
- the draft custom Act-I replacement policies in §1.3 remain forbidden.

Implementation order:

1. For each of the twelve categories, identify the current base `PXLawType` and its start/default policy in the editor.
2. Duplicate the closest policy as `DA_LH_Seal_<Category>`; configure the approved negative modifier and player-facing text.
3. Add that duplicate to the **existing** Law; arrange scenario-specific start state so it is active from week one.
4. Confirm that taking a vanilla alternative replaces the active seal and that effects do not stack.
5. Use the three restoration packages only as narrative/quest milestones in v0.1 unless editor testing proves the exact base-law replacement flow works inside this scenario.
6. Do not create §1.3 draft effects.

The editor table must be added only after inspection, because exact asset paths/names and modifier tags are versioned game content. `Паёк ожидания` cannot be wired until the author resolves its explicitly uncertain Hunger/Food effect.

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
| A6 promise gate | Borderlanders line incomplete | no Borderlanders promise dialog / no Rule agenda unlock |
| A7 commitment | Borderlanders line complete, accept promise | persistent promise + native Rule-agenda gate exactly once |
| A8 insufficient support | native Rule vote below 67 | no legal succession |
| A9 67% Rule law | native Council reaches threshold | actual Rule policy vote succeeds and sets amendment flag |
| A10 deadline | reach week 104 without pass | actual hard scenario failure/end |
| A11 edge ordering | pass at deadline boundary | result follows documented/tested order |
| A12 finale | legal pass | death public, exactly one course selected, Act 2 unlocked |
| A13 cooked build | cook then launch outside editor | same flow and no missing assets |

Do not start Act II/III data authoring until A1–A13 pass.
