# The Legitimate Heirs / «Легитимные Наследники»

Сюжетный сценарий для **Frostpunk 2**, разрабатываемый в FrostKit.

## Текущий статус

В репозитории подготовлены:

- [полная офлайн-вики FrostKit, собранная 19 ноября 2026](Docs/FROSTKIT_OFFLINE_WIKI_2026-11-19.md);
- [честный статус и границы текущей реализации](Docs/LEGITIMATE_HEIRS_IMPLEMENTATION_STATUS_2026-11-19.md);
- [FrostKit editor source package](FrostKitSource/README.md): manifest native assets, схема Act I, исходники диалогов и verification log;
- [аудит нативного developer sample](FrostKitSource/SAMPLE_SCENARIO_AUDIT_2026-11-19.md), находящегося в `FrostkitSampleScenario/`.

> В checkout есть developer sample с бинарными Unreal `.uasset`, но нет установленного FrostKit/Frostpunk 2. Поэтому это **не cooked/playable build**. Нативные assets и Blueprint graphs должны быть созданы и проверены в `Frostpunk2ModEditor` по `FrostKitSource/ACT1_EDITOR_WIRING.md`, после чего обязательны PIE, save/load и Cook Mod проверки.

Исходный дизайн кампании находится в [`Docs/`](Docs/).