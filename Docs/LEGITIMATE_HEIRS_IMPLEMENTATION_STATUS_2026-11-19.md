# The Legitimate Heirs — честный статус реализации

**Дата:** 19 ноября 2026 (UTC)
**Основания:** все документы из `Docs/`; техническая база — `FROSTKIT_OFFLINE_WIKI_2026-11-19.md`.

## Итог в одном абзаце

Я подготовил офлайн-вики FrostKit и **FrostKit-ready implementation package** в `FrostKitSource/`: asset manifest, точную схему Act I и пакет исходных текстов/состояний. После исходного аудита в ветку перенесён developer sample `FrostkitSampleScenario/` с 69 нативными assets; его практический разбор записан в `FrostKitSource/SAMPLE_SCENARIO_AUDIT_2026-11-19.md`. В Linux-среде по-прежнему отсутствуют FrostKit, Frostpunk 2, Windows `Frostpunk2ModEditor` и `.uproject`, а нативные Unreal graphs нельзя корректно или безопасно сгенерировать текстом. Поэтому в ветке нет фальшивых `.uasset` и нет заявления, что мод уже запускается.

## Что добавлено сейчас

| Артефакт | Содержимое | Статус |
|---|---|---|
| `Docs/FROSTKIT_OFFLINE_WIKI_2026-11-19.md` | полный конспект 23 публичных guide mod.io, источники, версия, границы достоверности, редакторские проверки | готово |
| `FrostKitSource/LegitimateHeirs.editor-manifest.yaml` | именованный реестр native assets, переменных, locks, quests, dialogs, policies, ideas, локализации | готово как source-of-truth для Editor |
| `FrostKitSource/ACT1_EDITOR_WIRING.md` | порядок создания assets и точная state machine первого акта | готово для переноса в FrostKit |
| `FrostKitSource/Narrative/act1_dialogue_source.csv` | русские исходники подтверждённых сцен Act I и их effects | готово для импорта/локализации после editor audit |
| `FrostKitSource/EDITOR_VERIFICATION_LOG.md` | протокол, который не позволит «додумать» API при переносе | готово, ожидает FrostKit |

## Что именно покрывает подготовленная реализация

### Реализуемая логика Act I

- стартовая тайна смерти отца;
- основной квест «Снятие возрастного ценза»;
- 52-я неделя: закрытая годовщина, письмо-ультиматум, запуск видимого 52-недельного отсчёта;
- жёсткое условие `week 104 && amendment not passed → fail`;
- три линии переговоров: Порубежники, Стражи, Основатели;
- отдельные флаги completed/promise/votes для каждой;
- порог 67% и законная передача власти;
- событие самовольной отмены как fail/узурпация;
- актовый финал: объявление смерти, публичные похороны, выбор первой политической опоры;
- переходные флаги для радикализации Act II.

Это — **структура реализации**, но не утверждение, что каждый exact Blueprint node уже существует: nodes и parent classes специально оставлены на проверку в editor.

### Реализуемые FrostKit data assets

По публично документированной модели в manifest предусмотрены:

- scenario chain `PXScenario` → runtime/startup/level/frostland/nation configs;
- Scenario Variable Storage;
- Quest / Scenario Script / Story Arc / FlowHub assets, дублируемые из sample after audit;
- Dialog / Dilemma data assets;
- Unlock data assets;
- Policy/Law/Group/Category assets для возрастной поправки (67%);
- research assets только для **подтверждённых** поздних ideas, если автор после проверки не отменит их;
- RU source text для Localization Dashboard.

## Почему сейчас нельзя заявить «мод целиком реализован»

### A. Технический блокер среды

На машине нет FrostKit и `Frostpunk2ModEditor`. Developer sample с 69 game assets теперь находится в `FrostkitSampleScenario/`, но `.uasset` остаётся бинарным форматом Unreal, в котором новые graphs и asset references создаёт редактор. Самодельный текстовый файл с расширением `.uasset` был бы не модом, а повреждённым мусором.

Файл `FrostKitSource/EDITOR_VERIFICATION_LOG.md` фиксирует конкретную процедуру вместо догадки. После установки FrostKit следующая техническая работа должна идти прямо там: создать native assets из manifest, compile, PIE, cook, запуск вне PIE.

### B. Документы намеренно запрещают часть контента

Ниже перечислены прямые ограничения, которым нельзя противоречить при слове «целиком»:

1. `FROSTPUNK2_CAMPAIGN_LAW_REDESIGN_v0.1.md`, §1.3: **«ЭТО ЧЕРНОВИК И ВЫПОЛНЯТЬ ЕГО НЕ НУЖНО… ОСТАВЬ ПОКА-ЧТО … ЗАКОНЫ ИЗ БАЗОВОЙ ИГРЫ»**.
2. В том же документе `New Work Model`, `Surplus Heat`, `Workplace Control` и несколько других категорий явно обозначены как тестовые и запрещены к реализации.
3. Перед §3.7 прямо написано: **«ВСЕ КАТЕГОРИИ, ЧТО ОПИСАНЫ НИЖЕ — ПОКА НЕ РЕАЛИЗОВЫВАЙ!»**. Это захватывает детство, молодёжь, родительство, воспроизводство, guard immunity, criminal reparation и emergency mandate.
4. §6 «Минимальный вертикальный срез» одновременно предлагает Childhood и Criminal Reparation, но они входят в запрещённый ниже блок. Это внутреннее противоречие; безопасный приоритет — явный запрет «не реализовывай», а не попытка угадать, что автор хотел отменить.

### C. Не утверждены сведения, без которых нельзя закончить сюжет

Narrative audit сам маркирует следующие вещи открытыми: имя/образ героя, город/регион, имена и происхождение лже-братьев, 6–9 точных долгов отца, длительность/цена бури, ветвления и условия полного набора финалов. Внесение их от себя нарушило бы указанное автором правило «не считать утверждённым каноном».

## Безопасная первая игровая версия после установки FrostKit

1. Новый custom scenario на sample-compatible карте.
2. Intro → тайная смерть → quest age amendment.
3. Одна законченная малая линия (например, Порубежники) и одно обещание, затем **симулированные/временные** два других голоса только для проверки threshold.
4. Week 52 / Week 104 timer, legal pass / fail.
5. Один base-game Law и один data-injection или script effect только после редакторской проверки.
6. Act I finale / first-course selection.
7. PIE + save/load + cooked test.

Только после этого надо переносить две остальные линии, весь Act II и Act III. Такой порядок не урезает замысел; он не позволяет построить 66 непроверенных законов и обнаружить, что базовый сценарий не загружается.

## Следующее действие, когда FrostKit будет доступен

Открыть `FrostKitSource/ACT1_EDITOR_WIRING.md` и выполнять его строго по порядку. Заполнять `FrostKitSource/EDITOR_VERIFICATION_LOG.md` ссылками на реальные asset paths / parent classes / скриншоты / результаты PIE. Любая строка `ПРОВЕРИТЬ` должна стать либо проверенным фактом, либо явным техническим ограничением — никогда не предположением.
