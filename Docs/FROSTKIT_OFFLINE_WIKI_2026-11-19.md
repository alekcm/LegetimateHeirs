# FrostKit / Frostpunk 2 — офлайн-вики для проекта

**Снимок источников:** 19 ноября 2026 (UTC)
**Назначение:** техническая база для `The Legitimate Heirs`; не замена FrostKit и не список выдуманных API.
**Правило достоверности:** здесь фиксируются только сведения, найденные в перечисленных источниках. Если возможность или имя типа не подтверждены, они явно помечены `ПРОВЕРИТЬ В EDITOR` и не считаются фактом.

---

## 1. Что именно было проверено и что означает «полнота»

Полностью сохранить *все существующие и будущие знания* о закрытом Unreal-проекте невозможно: часть информации живёт только в установленном FrostKit, бинарных `.uasset`, контекстной справке редактора и будущих обновлениях. Обещание обратного было бы галлюцинацией.

Зато этот файл — **полный конспект всей публичной технической документации Frostpunk 2 на странице Guides mod.io, доступной во время аудита**: индекс показывал **23 guides**. Все 23 прочитаны; конкурсные условия отмечены как нетехнический источник. В документ включены инструкции, пути, имена классов, ограничения и предупреждения, нужные для работы. Картинки не копируются, но их действия описаны текстом.

### 1.1. Категории источников

| Метка | Что это означает |
|---|---|
| **Официальный** | сообщение 11 bit studios или tutorial от `bitmods` / сотрудника 11 bit на mod.io |
| **Сообщество** | полезное руководство автора mod.io; особенно ранние (2024) пути и названия могут устареть |
| **Старый / не использовать как истину** | инструкция относится к pre-alpha/ранней версии или сама говорит о версии и проблемах |
| **ПРОВЕРИТЬ В EDITOR** | отсутствует подтверждённая публичная документация для конкретной возможности либо версия FrostKit может отличаться |

### 1.2. Главные выводы для этого репозитория

1. FrostKit — официальный Unreal modkit Frostpunk 2; 11 bit заявляет создание карт, моделей и custom scenarios. [Официальное объявление Steam](https://steamcommunity.com/games/1601580/announcements/detail/4572929880423149850)
2. С мая 2025 FrostKit 1.0 вышел из beta; публично заявлены **Play in Editor**, кнопка **Cook Mod**, улучшенная загрузка, tutorials и sample mods. [Официальный анонс/сводка патча](https://store.steampowered.com/news/app/1601580/view/498320947205373969?l=english) / [сводка](https://www.gamerant.com/frostpunk-2-free-major-content-update/)
3. Сюжетный мод надо делать как **собственный scenario**: `PXScenario` + startup/runtime/level/nation configuration + serialized Blueprints/scripts/quests/dialog data. Не как ручную подмену `.pak` базовой игры.
4. Для изменения существующих значений, где подходит модель Data Injection, надо предпочесть **Data Injection** прямому редактированию base assets: система additive и рассчитана на переживание патчей, пока разные моды не меняют один и тот же asset. [Официальное руководство](https://mod.io/g/frostpunk2/r/how-to-use-the-data-injection-system)
5. Нативные `.uasset` и Blueprint graph нельзя достоверно создать или связать текстовым редактором. Их создаёт FrostKit/Unreal Editor. В этом репозитории на момент аудита **нет FrostKit, `Frostpunk2ModEditor`, `.uproject` или стартовых `.uasset`**; поэтому текстовый план не выдаётся за собранный мод.

---

## 2. Установка, тестирование и публикация

### 2.1. Play in Editor (PIE) — официальный текущий маршрут

Источник: [Play-in-editor in Frostkit](https://mod.io/g/frostpunk2/r/play-in-editor-in-frostkit), 26 февраля 2026.

1. Нужны FrostKit и **легально приобретённый Frostpunk 2** на одной машине.
2. Найти каталог, содержащий именно `Frostpunk2.exe`.
3. В пути **не должно быть пробельных символов** (пример плохого пути из руководства: `Program Files (x86)`). Если FrostKit не распознаёт путь, перенести установленную игру в путь без пробелов способом своего магазина.
4. В FrostKit открыть **Editor Settings / Editor Preferences**, найти `Game Directory` и указать точный каталог `Frostpunk2.exe`.
5. Открыть **Mod Cooker** и указать тот же `Game Directory` также там.
6. После этого запуск через зелёную кнопку **Play** тестирует мод в editor без предварительного cook.

Не использовать старые сообщения о том, что PIE выключен: они относятся к 2024 beta и противоречат текущему руководству 2026/релизу 1.0.

### 2.2. Debug

Источник: [Debug options and you](https://mod.io/g/frostpunk2/r/debug-options-and-you), официальный tutorial.

- В игре, запущенной из editor (не в main menu/loading/settings), открыть developer debug menu: `Ctrl+Backspace`; затем `Backspace` открывает/закрывает его.
- `WBP_DebugMainPanel` — виджет панели; клавиши меняются в `IMC_DebugInputContext`.
- Панель умеет, среди прочего: добавлять/удалять city groups, extra fast-forward, открывать research/frostland, менять stats/resources, смотреть barks/lenses/notifications, frostbreak карты и проигрывать sounds.
- Локальный тестовый крючок — `WBP_Debug_CustomLogic_Button` → Graph.
- `F8` ejects camera: можно выбирать runtime actors и инспектировать/временно менять компоненты. Для нормальной картинки нужно подобрать корректный `Visible in View`.
- `~` открывает UE console; Space показывает команды. `HideUI` — приведённый пример.
- Это developer tooling: оно может быть неудобным, багованным или меняться; не включать debug-рецепты в пользовательский релиз.

### 2.3. Cook / совместимость / публикация

- На FrostKit 1.0 есть кнопка **Cook Mod** (публично заявлено в changelog). Никакого подтверждённого CLI-рецепта упаковки в доступных гайдах нет.
- Не копировать вручную pak в игровую папку как штатный путь: ранняя community-дискуссия фиксирует проблемы такого подхода, но не содержит поддержанного решения.
- Для публикации на mod.io конкурсные условия 2025 требовали original work, видимость на mod.io, совместимость с latest game и соблюдение EULA; сами условия не являются техническим how-to. [Terms](https://mod.io/g/frostpunk2/r/fp2-mod-contest-terms-conditions)
- Перед каждым релизом проверять mod в PIE, затем cooked build, затем чистый запуск игры с активным модом. Конкретный экран загрузки/upload flow в публичных гайдах не описан — `ПРОВЕРИТЬ В EDITOR`.

---

## 3. Каноническая минимальная структура custom scenario

Источник: [How to create a basic Scenario Configuration](https://mod.io/g/frostpunk2/r/how-to-create-a-basic-scenario-configuration), официальный tutorial. Это основной рецепт для **The Legitimate Heirs**.

### 3.1. Обязательные уровни

Нужно не менее трёх Level assets:

1. **World level** — содержит базовые sublevels (HoD, IdeaTree, Cameras и прочие); их можно перетащить из `Content/Scenarios/Shared/Levels/Base`.
2. **Gameplay level** — содержит минимум custom layout, чтобы игра не падала при boot; в нём как минимум один `BP_CityArea` и один `BP_Territory` actor.
3. **Environment level** — terrain/landscape, на котором показывается мир.

### 3.2. Цепочка configuration assets

Создавать и связывать так:

```text
Gameplay level + Environment level
  → PXFrostlandTileType
  → PXFrostlandScenario
  → PXFrostlandRandomizationConfig (+ World level, grid size)
  → PXLevelConfig
  → PXScenarioStartupConfig
  → PXDefaultScenarioRuntimeSettings
  → PXScenario
  ↖ runtime settings также получает ссылку на PXScenario
```

Подробности:

- `PXFrostlandTileType`: получает gameplay и landscape levels.
- `PXFrostlandScenario`: получает один или несколько tile types; tiles на world map должны иметь **уникальные X/Y coordinates**, иначе они наложатся.
- `PXFrostlandRandomizationConfig`: получает FrostlandScenario и World level. Grid size измеряется tiles и не может быть меньше числа добавленных tiles.
- `PXLevelConfig`: получает `PXFrostlandRandomizationConfig`.
- `PXScenarioStartupConfig`: получает LevelConfig и стартовый Frostland tile, который открывает первоначальный fog of war.
- Создаются/настраиваются `PXNation Type`, `PXNation Config`, `PXCityType`, `PXCityArea Type`; Nation/City/CityArea assets подключаются в StartupConfig.
- `PXDefaultScenarioRuntimeSettings`: получает StartupConfig, GameplayGMS и World level; здесь задаются difficulty, Blueprint randomization community type на старте и возможность начинать с уже сформированной faction.
- `PXScenario` получает RuntimeSettings.

### 3.3. Поля `PXScenario`, важные для кампании

Подтверждённые поля из tutorial:

- Name/Description для scenario selection UI;
- Main Menu flag (только для собственного main menu);
- Scenario Variable/Constants Storage (библиотеки bool/int/float для scenario flow);
- Scenario Phases (milestones: stats/timeline reset, level switch и т. п.);
- Scenario Tags;
- Override Frame ID (start camera/zoom);
- Start Game Unlock;
- Scenario Scripts (Blueprint scripts, стартующие с scenario: story flow, quest chain и т. п.);
- Script Packs (список script, активируемых позже);
- Maximum Number of Story Arcs (лишние active `BP_Story Arc Scripts` queue-ятся);
- Available Story Arcs (construct на старте и затем проверки «can be started»);
- Game Level State (Wwise states);
- Chapters (набор глав и точек старта; саму логику ведут Blueprint scripts);
- Configuration (базовые global configs).

**Критическое следствие:** предвыборная кампания, 52/104-недельный таймер и переходы актов должны быть отнесены к Variable Storage + Scenario Script / FlowHub / Quest / Story Arc, а не выданы за обычную механику Law.

### 3.4. Utopia Builder map (не основной путь для этого мода)

Источник: [Utopia Builder – maps setup](https://mod.io/g/frostpunk2/r/utopia-builder-maps-setup).

Базовая цепочка близка к scenario выше. Для появления map в sandbox guide предлагает добавить StartupConfig в `WBP_Menu_Sandbox_MapSelection` → `StaticScenarioStartupConfigs`.

Randomization использует `PXFrostlandGridType`, Frostland Randomization Pass Config, Frostland Grid Setup и `PXSiteRandomizationPass`; tags соединяют тип tile с site/outpost/dilemma/community conditions и mapping сложности. Для сюжетной кампании «Легитимные Наследники» это не требуется, пока не утверждён отдельный Utopia режим.

---

## 4. Laws / policies — точная модель, а не «UE-обычная» догадка

Источник: [Policy creation guide](https://mod.io/g/frostpunk2/r/policy-creation-guide).

### 4.1. Иерархия

```text
PXPolicyType (отдельный вариант)
  → PXLawType (вопрос/закон; содержит варианты)
  → PXLawGroupType
  → PXLawCategoryType
  → PXNationType.Initial Main Categories (если категория доступна в начале)
```

Базовое расположение данных: `Content/Scenarios/Shared/Configs/Policies/`; `PXNationType` — `Content/Scenarios/Shared/Configs/Nations/NationTypes`.

### 4.2. `PXPolicyType`: подтверждённые поля

- Ingame Name, Description;
- Can be Default;
- Extreme Mode: Per Affinities / Not Extreme / Extreme;
- Affinity Changes — сообщества/factions с тем же worldview предлагают/support policy; пустое поле означает поддержку всеми;
- Modifiers и Modifiers with Requirements;
- Requirements:
  - `PXResearchOutcome Requirement`;
  - `PXPlayer Affinity Requirement` (min/max City Direction/Zeitgeist);
  - `PXUnlock Requirement` (narrative unlock или extremity);
  - `PXComposite Requirement` (all/any/none и набор условий);
  - **Mark Deprecated** на *существующем* requirement заставляет игру игнорировать его. Руководство специально говорит: для base-game policies не удалять requirement, иначе можно сломать игру;
- one-time Actions on Activate/Deactivate;
- Fallback Group + Default Fallback Group Policy: policies одной группы можно swap-нуть Blueprint node `Set Fallback Policy`; default нужен при старте;
- Required Yes Percentage;
- Voting Behaviour Set Override (`PXVotingBehaviourSet`);
- Is Charter Policy: vote определяется relations communities/factions, а не affinities;
- Charter Aye/Hesitant Standing Threshold;
- Custom (Other) Gameplay Effect Text;
- For/Against Policy Barks.

### 4.3. Применение к возрастной поправке

Подтверждённая безопасная часть реализации:

- создать новый `PXPolicyType` «Право голоса после тридцати»;
- задать `Required Yes Percentage = 67%`;
- создать Law/Group/Category согласно иерархии;
- если голосование должно зависеть от отношений трёх групп, использовать `Is Charter Policy` и threshold после проверки реального голосового поведения в editor;
- подключить категорию в Nation Type, но только если она должна быть видна с первого дня.

**Не подтверждено документацией:** выбор делегатов по возрасту; 100% против на старте; суммирование специально обещанных голосов; блокировка короны; принудительный Game Over. Всё это `ПРОВЕРИТЬ В EDITOR` и проектировать в Scenario Script/FlowHub, а не добавлять вымышленные fields в policy data asset.

### 4.4. Политики оригинальной игры

Для campaign docs важны два строгих правила:

1. Если нужна правка существующей policy requirement, используйте `Mark Deprecated`, не удаление.
2. Документ кампании прямо предписывает: черновик раздела 1.3 **не реализовывать**; временно оставить законы базовой игры. Поэтому до нового авторского решения не создавать custom Act-I replacement policies из этого черновика.

---

## 5. Research / ideas

Источник: [How to add a research idea](https://mod.io/g/frostpunk2/r/how-to-add-a-research-idea-in-frostpunk-2).

### 5.1. Иерархия и размещение

```text
PXResearchTopic
  → PXResearchOutcome (один или несколько)
  → ResearchSubCategory.Topics
  + actor-node в IdeaTree_Nodes level
```

- Data: `Content/Scenarios/Shared/Configs/Research/ResearchTopics/`.
- Visual tree: `Content/Scenarios/Shared/Levels/Base/IdeaTree_Nodes`.
- Создать `PXResearchTopic` Data Asset, затем outcomes (`PXResearchOutcome`) в подпапке `Outcomes`.

### 5.2. Topic — поля и условия

Подтверждены: Ingame Name, Question Text, Research Time, Type (нужен для корректного research panel/icon), Stockpile Cost (base uses `DA_Core_Heatnotes`, но можно иной stockpiled resource), Requirements, Outcomes, Affinity Bucket (Mild/Extreme), Adjustment Slots Count, Problems.

- `Cost` помечен в guide как no longer used.
- Topic/locked topic icons помечены как unused.
- Custom Visibility in Scenario «не работает правильно на данный момент» — не опираться на неё для gating кампании.
- Requirements: `PXResearchTopic`, `PXPlayerAffinity`, `PXUnlock`, `PXComposite`; для base items deprecate, не удалять.
- Outcome может иметь Custom Effect Text, solved Statistics Problem, Affinity, Modifier/Gameplay Effects и Visibility Requirements.
- Outcome может быть requirement для building, hub, district или Policy.

### 5.3. Добавление на дерево

1. Добавить topic в соответствующий `ResearchSubCategory`.
2. Открыть `IdeaTree_Nodes`, найти `ResearchNodesActor`.
3. Выбрать subcategory → `Load Selected Subcategory`.
4. Дублировать node, позиционировать, в Details подменить Research Topic.
5. Data requirements должны соответствовать визуальным предшественникам; корректно сформированная связь рисует yellow line.
6. Pathfinding line ограничен несколькими intersections purple network: не разносить связные nodes слишком далеко.

Для **Coal Prosthetics**, oil gate и иных будущих ideas: сначала делать `PXResearchTopic` + Outcome и gate policy/building через его Outcome; не подменять исследование «законом».

---

## 6. Buildings, Urbos и art — только когда дизайн действительно требует объектов

### 6.1. Playable building config

Источник: [How to add playable buildings](https://mod.io/g/frostpunk2/r/how-to-add-playable-buildings), официальный tutorial.

- Главный asset: `PXBuildingType`; base configs: `Content/Scenarios/Shared/Configs/Buildings`.
- Игровые buildings строятся **только внутри districts**.
- Создать новый `PXBuildingType` или duplicate близкого; затем добавить его в `PXDistrictType` → Gameplay → Building Holder Setup/Buildings. Добавлять также во все нужные upgrade levels district.
- Именование base convention: `DA_[E|P|H|L|M|X]_[Name]_[Affinity|Base]`, где E Extraction, P Production/Industrial, H Housing, L Logistics, M Main, X any district.
- Browser: `Content/UtilitiesTools/EditorUtilities/EditorTools/BuildingsDistrictsBrowser/ET_BuildingsDistrictsConfig.uasset` → RMB → Run Editor Utility Widget.

#### Важные signed values

- Needs Produced: **minus** делает need менее удовлетворённой (то есть operation cost); пример coal mine `-60 maintenance`.
- Statistics Change: positive повышает проблему, negative снижает.
- Workforce required: **negative**.
- Economy resources: positive produces, negative requires. Partial resources есть, но guide советует осторожность: экономике обычно нужны integers.
- Extraction: base amount `0` означает extraction равный output; guide рекомендует оставлять 0, если не нужна особая логика.

#### Базовые sections

- General: name/description, visibility requirements (например `PXResearch Outcome Requirement`), tags (`Building.[district]`, `HasEfficiency`).
- Gameplay: `BP_BaseGameplayBuilding`, needs, statistics, affinity (конфигурационные имена: Solidarity=Equality, Competition=Merit, Change=Reason, Tradition=Tradition, Symbiosis=Adaptation, Dominion=Progress), workforce/economy/modifiers/extraction.
- Construction: base recommendation — 2 weeks, Heatnotes/Prefabs/(Cores advanced), workforce такой же, как operation; `DA_LowPriorityCategory`, subpriority 0; не использовать непроверенные options вроде Infinite Duration.
- Deconstruction: base recommendation 2 weeks, refund около 75% construction stockpile costs; `DA_IndestructibleBlockage` делает object несносимым.
- Ability Group задаётся на **district**, не building; building ability делается отдельной ability с `PXBuilding Requirement`.
- Optional Unique Group не даёт строить больше одного объекта из группы в district.
- Visual: можно duplicate близкий Custom Building Blueprint; прописать icon и solved-problem icon.
- UI: `PXBuildingVariantGroup` объединяет affinity variants; Custom Building Gameplay Effect Text нужен для эффекта, который UI не выводит сам.

### 6.2. Asset art

Источник: [Adding a New Building Asset to the Game](https://mod.io/g/frostpunk2/r/frostkit-first-aidkit), официальный author.

- Не генерировать Light UV автоматически при import, если нужны до трёх UV.
- Обычно используются 3 materials: large surfaces, details, windows.
- RMA map: Red=roughness, Green=metallic (binary), Blue=ambient occlusion.
- Dirt mask uses UV1; construction animation uses vertex paint green channel; избегать резких различий на больших triangles.
- Делать LODs; duplicate существующий Urbos Blueprint предпочтительнее создания с нуля.
- В generator priority — порядок spawn; weight — относительная вероятность в одном priority; amount limit `-1` — без лимита.

### 6.3. Urbos

Источники: [Getting Started](https://mod.io/g/frostpunk2/r/getting-started-with-urbos-configuration), [District and Road Generation](https://mod.io/g/frostpunk2/r/ubros-district-and-road-generation), официальные tutorials.

- Urbos — procedural district generator: L-system → major/minor roads → lots → Urbos tiles; hub занимает центр, special building может заменить tiles.
- Не создавать новый Nation/Urbos config с нуля без необходимости: official guide рекомендует duplicate New London config, потому что отсутствие элемента может crash game.
- District size: `PXDistrictType.Required City Cells Count`; active list — `DA_CityStartupRulesConfig.Supported Districts`.
- Shape: District Type → View Pattern → `PXLotGeneratorData`. Organic: безопасно менять Min Lot Size/Max Base Lot Size/Expected Lots Size; Planned: Wall Aligned sizes, Road Lots Layers, Move Lot Size Factor. Tile set должен соответствовать lots.
- Road knobs: Internal Start Road Points Count, Internal Road Points Count; больше roads = меньше места tile. `DA_CityRoad` + `DA_RoadsOrganic` в Radial Rule включает organic roads.
- `PXUrbosTile`: bounds должны покрывать mesh; big tile закрывает несколько lots. Нужны visibility components для City (main/construction/terraforming) и Frostland (low-poly), appropriate instancing batches; main/construction batches используют `DA_HudiniAnimCustomData`.
- Hub: main/proxy mesh, square/zoom story area, visibility, construction, instancing, `PXUrbosVolume`. Для hub guide задаёт cylinder, radius около 800, height 50000, origin -25000, Works for Flags (HideLSystem blocks roads, HidePipeFlag blocks ZTM), Works for Urbos Tiles и Ignore Every Tile enabled.

**Для Legitimate Heirs:** новых buildings/districts/3D models не создавать в первой реализации, пока не утверждены «distributed furnaces», prosthetics workshop, school и их баланс. Сюжет и policies не требуют подделывать art pipeline.

---

## 7. Terrain / map

Источник: [Creating and Importing Custom Terrain Maps](https://mod.io/g/frostpunk2/r/creating-and-importing-custom-terrain-maps), официальный tutorial.

### 7.1. Terrain tile

- Автор использует World Machine; допускаются Terrain editors с erosion (например TerreSculptor 2.0, Houdini/Blender workflows).
- Default heightmap: 16-bit PNG, **2017×2017**.
- Splatmap: RGB — R snow/rock, G extra detail/flow/wetness, B ice (можно переопределить назначение).
- Import splat/normal в Materials, выключить Virtual Texture Streaming на обоих; для splatmap compression = `Masks (no sRGB)`.
- Duplicate landscape material instance, назначить normalmap/splatmap; при default 2017×2017 отключить options для low-resolution landscapes.
- Landscape import example: location `(0, 0, 51652.5)`, scale `(200, 200, 200)`, material instance; **Landscape Collision = LandscapeMeshes**.
- Для filler tile: configs в `Content/Scenarios/Configs/FrostlandRandomization/Tiles`; duplicate `_Inner`/`_Outer`, set biome, landscape/gameplay levels, optionally Is Unique; добавить в `FRP_SandboxMain` → `PDA_FRP_SandboxMain7_FillRest` → Allowed Tile Types.

### 7.2. City tile

City требует extra pipeline: Terrain Inside (VHM bounding area) + Terrain Outside (proxy mesh), centering/slicing through FP2 Terrain Mesh Toolkit, proxy material with Mesh UVs, import sliced distant meshes. Этот pipeline — отдельная продакшн-задача; не заменять его предположением, что обычный UE Landscape достаточно.

Для кампании выбрана рабочая стратегия: сначала duplicate/sample-compatible map, затем новая карта только после вертикального среза. Конкретная sample/map, доступная в установленной версии, `ПРОВЕРИТЬ В EDITOR`.

---

## 8. Narrative, dialogs, scripts, zoom stories

### 8.1. Подтверждённые опоры

- Scenario Scripts и Available Story Arcs — официально подтверждённые поля `PXScenario` (см. §3).
- Ранний открытый sample scenario, изученный как пример структуры, содержит serialized `FlowHub`, `Quests`, `ScenarioScripts`, `StoryArks`, `VariableStorage`, configs dialogs/messages/dilemmas/unlocks/script packs. Это **community example, не API contract**.
- Локализация: ответ сотрудника в official building guide подтверждает стандартный Unreal **Localization Dashboard** для нескольких языков. Добавление localized sound тогда отмечено как недоступное; не считать это обещанием для нынешней версии.

### 8.2. Zoom Stories — официальный tutorial

Источник: [Zoom Stories: setup guide](https://mod.io/g/frostpunk2/r/zoom-stories-setup-guide).

Иерархия:

```text
PXZoomStoryRoot
  → DA_ZoomStory config(s)
  → Zoom Story Blueprint scene
  → optional Sub Zoom Stories
```

- Root — spawner на plaza, main-quarter hub, one-cell hub, city/environment ZS.
- Enviro route: duplicate existing `BP_EnviroZS_WinterhomeCliff`; сохранить Box collider/PXInteraction и `PXZoomStoryRoot.Is Selectable=true`; place на level.
- Создать `DA_ZoomStory` duplicating existing config, set Spawn BP, добавить в `PXZoomStoryRoot.Pools`. Pools умеют weight, priority, pool requirements; higher priority всегда побеждает lower при выполнении обоих.
- Requirements создаются duplicating existing Blueprint requirements; guide приводит `BP_HubActive_Requirement`, но для environment example ему требуется script logic, потому что он не привязан к hub.
- Все mesh/people должны попадать в `PXVisibleInView`, иначе не render. Пример scale: adult 0.6, child 0.45.
- `PXFollowSpline` + Spline позволяют движение; требуется назначить spline, movable components, speed curve.
- District ZS: child `BP_Shared_ZS`, smaller scenes в Sub Zoom Stories, назначить scenes A–E, включить Avoid ZSUrbosVolume.

**Вывод:** похороны отца можно делать простым Dialog/Dilemma на первом MVP. Добавлять Zoom Story только после того, как сам сценарный флаг и quest работают; иначе арт-сцена не решает core campaign logic.

### 8.3. Как не придумывать несуществующую narrative API

Публичных инструкций, описывающих все nodes FlowHub, Quest, Story Arc и Game Over, на дату снимка нет. Поэтому:

- не называть ни один Blueprint node «фактом», если он не виден в editor/sample;
- открыть FrostKit sample, duplicate именно соответствующий Quest/Scenario Script/FlowHub asset;
- записать exact base class, fields, nodes и compile result в `Docs/FROSTKIT_EDITOR_VERIFICATION_LOG.md`;
- только после этого создавать asset implementation, а не текстовый псевдо-`.uasset`.

---

## 9. UI, icons, contexts

### 9.1. Main menu / custom panel

Источник: [How to add a button to main menu](https://mod.io/g/frostpunk2/r/how-to-add-a-button-to-main-menu), официальный tutorial.

- Duplicate `StoryButton` в `WBP_MainMenuLobbyPanel`, add `ButtonClicked` event.
- Custom panel должен derive from `PXPanel`; tutorial кладёт его в `Content/UI/UiWidgets/MOD_Widgets`.
- При open panel назначить `PXScenario`; start function должна разобрать selected ambitions/challenges/communities/difficulty/settings, а переменные очищаются после opening scenario.
- Input: отдельные Input Action/IMC; добавить mapping context в OnOpen, удалить в OnClose. Свои settings добавить нельзя, но общий Escape mapping можно добавить в `DA_UserSettingsConfig.KeybindSettings`.

**Не нужно** добавлять main-menu button для кампании, пока не доказано, что обычный scenario selection не может отобразить PXScenario. Это уменьшает риск конфликтного UI override.

### 9.2. Contexts / UI Groups

Источник: [Controlling the UI](https://mod.io/g/frostpunk2/r/controlling-the-ui-contexts-and-ui-groups), community.

- Simple Context — basic view state.
- Composite Context: All Of, Any Of, None Of; `DeactivateOnCompositeDeactivate`, `CheckInactive`.
- Major exclusive screens (Idea Tree / End Log / Frostland) используют общий `PXContextGroupType` ExclusiveGroup.
- UI Group привязывается к simple/composite context; options: Open Panels on Activation, Exclusive, Try to Keep Open Panels.
- `PXPanel` может pause, block deposit tooltips и т. д.
- Context Config relationships: Unlock → SimpleContext, AvailabilityTypeGroup with satisfied requirements → SimpleContext. Mapping 1:1 и только SimpleContext.

### 9.3. Icons and old UI notes

- [Sprite sheet guide](https://mod.io/g/frostpunk2/r/how-to-download-a-sprite-sheet): UI frames at `Content/UI/Assets/Textures/AtlasedTextures/Icons/Frames`; Asset Actions → Export; reimport; Sprite Actions → Create Sprite; x/y/width/height crop source; duplicate nearby sprite as fast path.
- [GFX & UI](https://mod.io/g/frostpunk2/r/gfx-ui) — old community locations: UI root, main menu special textures, faction CommunityPanel atlas. Use only as discovery hint and check paths in current editor.

---

## 10. Characters / delegates

Источник: [Anatomy of a delegate](https://mod.io/g/frostpunk2/r/anatomy-of-a-delegate-introduction-to-character-modding-in-fp2), community.

- Delegate is persistent after creation; changing rules/assets does not retrofit an existing save.
- Instance class: `BP_ParliamentCharacter`, assembled from Head, Headwear, Upper body, Lower body, Lamp sockets; every socket used once, including empty fake mesh where nothing should show.
- Group mesh lists live in Data Tables; a mesh needs Main Tag for its socket. Shared `DT_Heads`, `DT_Hats`, `DT_Lamps`, `DT_Lowers`, `DT_Uppers` are additive.
- New delegate skeletal meshes must skin to `SKEL_UniSkeleton`; use Material **Instances**, not a new master material unless necessary.
- Face/hair material slot data assets manage available materials and tags.

**Campaign impact:** don't promise unique visual delegates for three new factions in v0.1. The design documents require political groups and relations, not a custom character art pipeline; base visual assets need a verified legal/technical reuse plan.

---

## 11. Older community guides: audit trail and safe usage

All were in the 23-guide index and read, but they are not stronger than current official guides.

| Guide | Safe retained fact / warning |
|---|---|
| [Creating a Blueprint](https://mod.io/g/frostpunk2/r/creating-a-blueprint) | For world actors prefer `PXActor`, not generic Actor. `PXVisibility` + `PXVisibleInView` are needed for an asset to appear; compile and save. |
| [Things I wish I knew](https://mod.io/g/frostpunk2/r/things-i-wish-i-knew) | Explicitly based on old custom UE 5.2.1 and says later-engine assets may not show. Keep only as historical troubleshooting; do not alter engine config/VRAM settings in project based on it. |
| [Create a Drone](https://mod.io/g/frostpunk2/r/create-a-drone-for-testing-intermeiate-level) | Historical optional visual inspection recipe; current F8 eject/debug guide is safer. Do not modify global gameplay main-screen for release just to get a drone. |
| [Locating Maps](https://mod.io/g/frostpunk2/r/locating-maps-for-editing) | Old paths/names, explicitly pre-alpha names. Use as search hint only. |

---

## 12. Verified design-to-system mapping for The Legitimate Heirs

| Campaign requirement from `Docs/` | Correct FrostKit system | Evidence / limit |
|---|---|---|
| New self-contained narrative campaign | `PXScenario` chain + own levels/nation/runtime settings | §3 official scenario guide |
| State flags, 52/104-week countdown, promises, support | Scenario Variable/Constants Storage + Scenario Script/Quest/Story Arc | Storage/scripts are confirmed; exact nodes need editor audit |
| Main quest and faction task lines | Quest + Scenario Script / Story Arc | Scenario supports scripts/arcs; exact asset class needs sample audit |
| 67% age-law vote | `PXPolicyType.Required Yes Percentage=67`; Law hierarchy | official policy guide |
| Relation-based politics | Is Charter Policy + Charter thresholds, or script | official policy guide; age filtering unsupported |
| Original Act-I laws temporarily retained | do not remove base requirements; use `Mark Deprecated` only after inspection | official policy guide + project constraint |
| New research (prosthetics, etc.) | PXResearchTopic → Outcomes → Idea Tree node | official research guide |
| Law gated by research | PXResearchOutcome Requirement | official policy/research guides |
| New objects (only if approved) | PXBuildingType + add to districts + optional art/Urbos | official building guides |
| Interactive funeral visual | Dialog/Dilemma first; optional Zoom Story after logic works | official zoom-story guide |
| Patch-safe base value change | Data Injection assets | official data injection guide |

---

## 13. Required editor-verification checklist before claiming a playable build

1. Record exact FrostKit version, FP2 version and sample scenario path.
2. Create/duplicate a new `PXScenario`; boot in PIE.
3. Verify VariableStorage Blueprint class, available bool/int/float setters/getters and persistence over save/load.
4. Inspect sample Quest/Story Arc/Scenario Script/FlowHub. Record exact parent class and nodes for: start quest, complete/fail quest, dialog/dilemma choice, schedule timer/week check, add/remove resource, unlock, Game Over/end.
5. Confirm how a custom faction/community can be created and relation values read/written; do not assume base-group assets are editable.
6. Confirm custom Law Category visibility, policy fallback, original-law disabling/deprecation and vote behavior.
7. Prove 67% `Required Yes Percentage` in a tiny throwaway scenario.
8. Prove one 52-week trigger and one failure branch in PIE, save/load, cooked build.
9. Verify Russian localization workflow in current editor; generate loc assets via Localization Dashboard.
10. Cook and start the mod outside PIE; inspect logs, save size, menu registration and mod coexistence.

Until items 1–10 are completed, describe the repository only as **FrostKit-ready source/implementation plan**, never as a compiled or playable Frostpunk 2 mod.

---

## 14. Source register — all 23 public mod.io Guides audited

1. [Play-in-editor in Frostkit](https://mod.io/g/frostpunk2/r/play-in-editor-in-frostkit) — official/current.
2. [FP2 Mod Contest Terms & Conditions](https://mod.io/g/frostpunk2/r/fp2-mod-contest-terms-conditions) — legal/contest, not technical.
3. [Zoom Stories: setup guide](https://mod.io/g/frostpunk2/r/zoom-stories-setup-guide) — official.
4. [Debug options and you](https://mod.io/g/frostpunk2/r/debug-options-and-you) — official.
5. [Anatomy of a delegate](https://mod.io/g/frostpunk2/r/anatomy-of-a-delegate-introduction-to-character-modding-in-fp2) — community.
6. [How to add a button to main menu](https://mod.io/g/frostpunk2/r/how-to-add-a-button-to-main-menu) — official.
7. [How to create a basic Scenario Configuration](https://mod.io/g/frostpunk2/r/how-to-create-a-basic-scenario-configuration) — official.
8. [Urbos: District and Road Generation](https://mod.io/g/frostpunk2/r/ubros-district-and-road-generation) — official.
9. [Creating and Importing Custom Terrain Maps](https://mod.io/g/frostpunk2/r/creating-and-importing-custom-terrain-maps) — official.
10. [How to Use the Data Injection System](https://mod.io/g/frostpunk2/r/how-to-use-the-data-injection-system) — official.
11. [Controlling the UI: Contexts and UI Groups](https://mod.io/g/frostpunk2/r/controlling-the-ui-contexts-and-ui-groups) — community.
12. [Adding a New Building Asset to the Game](https://mod.io/g/frostpunk2/r/frostkit-first-aidkit) — 11 bit author.
13. [Getting Started with Urbos Configuration](https://mod.io/g/frostpunk2/r/getting-started-with-urbos-configuration) — official.
14. [How to add playable buildings](https://mod.io/g/frostpunk2/r/how-to-add-playable-buildings) — official.
15. [Utopia Builder – maps setup](https://mod.io/g/frostpunk2/r/utopia-builder-maps-setup) — community.
16. [Policy creation guide](https://mod.io/g/frostpunk2/r/policy-creation-guide) — community.
17. [How to add a research idea](https://mod.io/g/frostpunk2/r/how-to-add-a-research-idea-in-frostpunk-2) — community.
18. [how to download a sprite sheet](https://mod.io/g/frostpunk2/r/how-to-download-a-sprite-sheet) — community.
19. [Things I wish I knew](https://mod.io/g/frostpunk2/r/things-i-wish-i-knew) — old community.
20. [GFX & UI](https://mod.io/g/frostpunk2/r/gfx-ui) — old community.
21. [Create a Drone for Testing](https://mod.io/g/frostpunk2/r/create-a-drone-for-testing-intermeiate-level) — old community.
22. [Creating a Blueprint](https://mod.io/g/frostpunk2/r/creating-a-blueprint) — old community.
23. [Locating Maps for Editing](https://mod.io/g/frostpunk2/r/locating-maps-for-editing) — old community.

Additional authoritative context: [11 bit Steam announcement](https://steamcommunity.com/games/1601580/announcements/detail/4572929880423149850) and [FrostKit 1.0 update](https://store.steampowered.com/news/app/1601580/view/498320947205373969?l=english).
