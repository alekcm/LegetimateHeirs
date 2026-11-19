# Аудит `FrostkitSampleScenario`

**Источник:** developer sample, добавленный в `master` коммитом `c2e60bc` («Экзампл») и перенесённый в текущую ветку без изменения бинарных assets.
**Метод:** статический разбор путей, asset metadata и читаемых string table/import references из `.uasset`. Это надёжнее догадки, но **не заменяет** открытие в FrostKit: graph pins, runtime semantics и compatibility должны быть подтверждены через Compile/PIE.

## Вердикт

Sample радикально повышает реализуемость версии 0.1. Он содержит нативные, не пустые, связанные друг с другом examples всех основных опор Act I:

- полноценный `PXScenario` со startup/runtime/nation/level/frostland configs;
- `FlowHub`, который уже вызывает cutscene, запускает quests, ждёт quest completion, открывает predefined dialogs и активирует unlock;
- `PXQuest` assets с objective setup, async duration wait и `ImportantMainQuest`;
- `PXScenarioScript` для unlock Frostland;
- Story Arc, который использует `CanBeStarted`, `OnStarted`, `GetCurrentGameTime`, `StartQuestInPlace`, `WaitForStoryBeat`, `FinishQuest`, dialogs and unlock events;
- Variable Storage, Dialogs, Dilemma, Unlocks, Script Packs, levels/cameras.

Это правильная основа для **копирования и перепривязки**, а не для нового сценария с нуля.

## 1. Содержимое

| Блок | Нативный asset из sample | Как используется в Legitimate Heirs |
|---|---|---|
| Scenario | `Configs/DA_FroskitSample_Scenario.uasset` | duplicate как `DA_LH_Scenario`; сохраняет полный набор global config links |
| Runtime/startup | `DA_FrostkitSample_RuntimeSettings`, `DA_FrostkitSample_StartupConfig` | duplicate и подмена собственных start/nation/levels |
| Flow | `BlueprintsSerialized/BP_FrostkitSample_FlowHub.uasset` | основа intro → quest → dialog → next stage |
| Main quest | `Quests/BP_FrostkitSample_Basics.uasset`, `BP_FrostkitSample_Coal.uasset` | duplicate для age-amendment и проверяемых preconditions |
| Scenario script | `ScenarioScripts/BP_FrostkitSample_UnlockFrostland.uasset` | основа разблокировки контекста/сценарных реакций |
| Story Arc | `StoryArks/BP_FrostkitSample_ColdSnops.uasset` | основа week-triggered anniversary/deadline story arc |
| State | `VariableStorage/BP_FrostkitSample_VariableStorage.uasset` | duplicate для campaign flags/int state |
| Dialogs | `Configs/Messages/Dialogs/*.uasset` | duplicate для secret-death, anniversary, succession text |
| Dilemma | `Configs/Dilemmas/DA_FrostkitSample_Generic_StockpileMaterials5.uasset` | доказанный native `PXDilemmaType`; основа promise/first-course choices |
| Unlocks | `Configs/Unlocks/DA_FrostkitSampleGameStart.uasset` | root unlock pattern |
| Script packs | `Configs/ScriptPacks/*.uasset` | phase gates / Act II activation pattern |
| Maps | `Levels/*.umap`, `Levels/Gameplay/*.umap`, `Levels/Evironment/*.umap` | v0.1 должна переиспользовать sample-compatible map, не делать terrain pipeline с нуля |

## 2. Подтверждённые конкретные native types и nodes

### 2.1. Blueprint parent/type imports

| LH потребность | Sample evidence | Что можно утверждать |
|---|---|---|
| Quest | `BP_FrostkitSample_Basics`, `BP_FrostkitSample_Coal` импортируют `/Script/Frostpunk2.PXQuest` | Quest blueprint должен происходить от `PXQuest` |
| Scenario script | `BP_FrostkitSample_UnlockFrostland` импортирует `/Script/Frostpunk2.PXScenarioScript` | Scenario Script blueprint должен происходить от `PXScenarioScript` |
| Variable Storage | sample VariableStorage ссылается на shared `BlueprintsSerialized/VariableStorage/BP_Object` | дублировать sample; реальный editor parent/variable UI проверить при открытии |
| Story Arc | `BP_FrostkitSample_ColdSnops` ссылается на `BP_Standard_StoryArc` | duplicate этого Story Arc безопаснее создания с чистого листа |
| Scenario | `DA_FroskitSample_Scenario` импортирует `/Script/Frostpunk2.PXScenario` | отдельный custom scenario — правильная сущность |
| Runtime settings | `DA_FrostkitSample_RuntimeSettings` импортирует `PXDefaultScenarioRuntimeSettings` | runtime config duplicate имеет подтверждённый parent |
| Startup config | `DA_FrostkitSample_StartupConfig` импортирует `PXScenarioStartupConfig` | custom startup config имеет подтверждённый parent |
| Dilemma | `DA_FrostkitSample_Generic_StockpileMaterials5` импортирует `PXDilemmaType` и `PXDilemmaChoice` | promise/first-course choices не нужно изобретать вне native dilemma model |

### 2.2. Подтверждённые lifecycle / flow operations

В binary graph metadata sample найдены следующие вызовы:

```text
FlowHub:
  StartCameraCutscene
  AsyncAction_WaitForCutsceneEnd
  StartQuest
  AsyncAction_WaitForQuest
  AsyncAction_OpenPredefinedDialog
  SetUnlockActive

Quest:
  ReceiveStartQuest
  AddQuestObjective
  AddDistrictWithTagsQuestObjective
  AsyncAction_WaitForDurationPassed
  EPXQuestType::ImportantMainQuest

Story Arc:
  CanBeStarted
  OnStarted
  GetCurrentGameTime
  StartQuestInPlace
  AsyncAction_WaitForQuest
  AsyncAction_OpenPredefinedDialog
  WaitForStoryBeat
  FinishQuest
  IsAnyQuestOfTypeFinished
  UnlockChangedEvent

Scenario Script:
  SetUnlockActive
```

Это доказывает путь реализации intro, quest chain, controlled dialog, delayed story event и unlock **без написания C++ и без придумывания собственных systems**.

## 3. Что sample ещё не доказывает

Следующие пункты остаются editor/runtime checks:

1. Каким exact node/condition проверить week 52/104: `GetCurrentGameTime` существует, но его тип/units/pins нужно открыть в Editor.
2. Native hard scenario Game Over: sample не показал очевидную end-game path. Quest fail/dialog не следует называть Game Over до PIE-prove.
3. Процент 67%, charter voting и relation-based delegate behaviour: эти поля документированы в policy guide, но sample их не реализует.
4. Создание и UI-регистрация собственных малых communities/factions: sample опирается на base scenario community/faction configs.
5. Гарантированная save/load persistence каждого нового custom variable: доказать в PIE.
6. Cooked build and mod-menu registration.

## 4. Обновлённый практический план v0.1

1. В FrostKit открыть **сам sample из этого репозитория** и first compile/play it unchanged. Это устанавливает compatibility baseline.
2. Duplicate entire sample into `/Game/Scenarios/LegitimateHeirs` через Editor (не OS copy/rename `.uasset`, чтобы Unreal обновил references).
3. Сначала заменить только player-facing texts/Scenario name и доказать boot.
4. Duplicate `FlowHub`, `BP_FrostkitSample_Basics`, `BP_FrostkitSample_ColdSnops`, VariableStorage и dialog assets; собрать prologue → main quest → week trigger → legal succession mock.
5. Заменить mock на настоящие Act I lines только после утверждения точных objectives/debts and promise terms.
6. Сделать the Council / 67% proof отдельным узким экспериментом до полного переноса пакетов законов.

## 5. Граница ответственности

С этим sample **пользователю не придётся вручную разбирать Blueprint**, если рабочая среда предоставляет агенту FrostKit Editor. Агент сможет делать duplicate, links, graph wiring, Compile, PIE и Cook. В текущем Arena Linux checkout самого Editor/Game по-прежнему нет: наличие `.uasset` делает проект подготовленным, но не создаёт возможности редактировать Binary Blueprints без редактора.
