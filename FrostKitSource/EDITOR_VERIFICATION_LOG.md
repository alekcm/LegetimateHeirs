# FrostKit editor verification log — The Legitimate Heirs

**Policy:** fill this only by observing the installed editor/game. `Unknown` is a valid result. Do not replace an unknown with UE-memory, a web guess or an invented Blueprint node.

## Environment

| Field | Value |
|---|---|
| Date verified | NOT YET VERIFIED |
| FrostKit version | NOT YET VERIFIED |
| Frostpunk 2 version | NOT YET VERIFIED |
| Storefront / legal game path | NOT YET VERIFIED |
| FrostKit Game Directory has no whitespace | NOT YET VERIFIED |
| Mod Cooker Game Directory set to same `Frostpunk2.exe` folder | NOT YET VERIFIED |
| FrostKit content root | NOT YET VERIFIED |
| Sample scenario root / name | Repository path known: `FrostkitSampleScenario/`; Editor compatibility NOT YET VERIFIED |
| Current custom Content root | `/Game/Scenarios/LegitimateHeirs` planned |

## Native asset audit

Static metadata from the repository sample is recorded below; **runtime behaviour still needs Editor/PIE proof**. Full evidence: `SAMPLE_SCENARIO_AUDIT_2026-11-19.md`.

| Need | Exact sample asset path | Parent class / static evidence | Verified operation | Result |
|---|---|---|---|---|
| Scenario variable storage | `FrostkitSampleScenario/BlueprintsSerialized/VariableStorage/BP_FrostkitSample_VariableStorage.uasset` | shared `BP_Object` reference | bool/int read/write and save persistence | METADATA FOUND; PIE NOT YET VERIFIED |
| Scenario Script | `FrostkitSampleScenario/BlueprintsSerialized/ScenarioScripts/BP_FrostkitSample_UnlockFrostland.uasset` | `PXScenarioScript` | scenario-start event, week source | METADATA FOUND; PIE NOT YET VERIFIED |
| FlowHub | `FrostkitSampleScenario/BlueprintsSerialized/BP_FrostkitSample_FlowHub.uasset` | StartQuest / WaitForQuest / dialog / unlock calls found | start/wait/sequence/dialog control | METADATA FOUND; PIE NOT YET VERIFIED |
| Quest | `FrostkitSampleScenario/BlueprintsSerialized/Quests/BP_FrostkitSample_Basics.uasset` | `PXQuest` | objective, complete, fail | METADATA FOUND; PIE NOT YET VERIFIED |
| Story Arc | `FrostkitSampleScenario/BlueprintsSerialized/StoryArks/BP_FrostkitSample_ColdSnops.uasset` | `BP_Standard_StoryArc` reference | start condition, active limit, end | METADATA FOUND; PIE NOT YET VERIFIED |
| Dialog | `FrostkitSampleScenario/Configs/Messages/Dialogs/DA_FrostkitSample_BasicsMessage.uasset` | native dialog data asset | localized text, continue, action | METADATA FOUND; PIE NOT YET VERIFIED |
| Dilemma | `FrostkitSampleScenario/Configs/Dilemmas/DA_FrostkitSample_Generic_StockpileMaterials5.uasset` | `PXDilemmaType`, `PXDilemmaChoice` | choices and state effects | METADATA FOUND; PIE NOT YET VERIFIED |
| Unlock | `FrostkitSampleScenario/Configs/Unlocks/DA_FrostkitSampleGameStart.uasset` | `PXUnlockType` reference | activation and use in requirements | METADATA FOUND; PIE NOT YET VERIFIED |
| Script Pack | — | — | deferred script activation | NOT YET VERIFIED |
| Scenario failure/end | — | — | hard Game Over, not mere message | NOT YET VERIFIED |
| Week statistic/timer | — | — | actual campaign week read incl. fast-forward | NOT YET VERIFIED |
| Custom nation/faction | — | — | relation and council registration | NOT YET VERIFIED |

## Laws and Council proof

| Test | Asset path / test setup | Result | Evidence |
|---|---|---|---|
| `Required Yes Percentage=67` changes real passage threshold | — | NOT YET VERIFIED | — |
| Custom policy appears in own custom scenario | — | NOT YET VERIFIED | — |
| `Is Charter Policy` uses relations in this nation | — | NOT YET VERIFIED | — |
| Custom relation can be set/read by scenario script | — | NOT YET VERIFIED | — |
| Existing policy requirement can be Mark Deprecated without breaking boot | — | NOT YET VERIFIED | — |
| Base law can be gated/replaced for this scenario only | — | NOT YET VERIFIED | — |

## Act I functional tests

| ID | Expected outcome | PIE | Save/load | Cooked build | Notes/evidence |
|---|---|---|---|---|---|
| A1 | Scenario boots to city | NOT RUN | NOT RUN | NOT RUN | — |
| A2 | Secret-death intro only once | NOT RUN | NOT RUN | NOT RUN | — |
| A3 | State survives load without reset | NOT RUN | NOT RUN | NOT RUN | — |
| A4 | Week-52 ultimatum fires once | NOT RUN | NOT RUN | NOT RUN | — |
| A5 | Death remains secret before legal succession | NOT RUN | NOT RUN | NOT RUN | — |
| A6 | Incomplete group line cannot grant promise/votes | NOT RUN | NOT RUN | NOT RUN | — |
| A7 | Promise saves, commits real support | NOT RUN | NOT RUN | NOT RUN | — |
| A8 | Below 67 cannot trigger legal succession | NOT RUN | NOT RUN | NOT RUN | — |
| A9 | 67% passage sets AgeAmendmentPassed | NOT RUN | NOT RUN | NOT RUN | — |
| A10 | Week 104 produces hard scenario failure | NOT RUN | NOT RUN | NOT RUN | — |
| A11 | Deadline-boundary ordering documented | NOT RUN | NOT RUN | NOT RUN | — |
| A12 | Exactly one first-course flag and Act II unlock | NOT RUN | NOT RUN | NOT RUN | — |

## Cook and release test

| Check | Result | Evidence |
|---|---|---|
| Cook Mod completed | NOT YET VERIFIED | — |
| Mod launches outside PIE | NOT YET VERIFIED | — |
| No missing-reference errors | NOT YET VERIFIED | — |
| Russian texts resolve in shipped build | NOT YET VERIFIED | — |
| New save works | NOT YET VERIFIED | — |
| Existing/other-mod coexistence considered | NOT YET VERIFIED | — |
