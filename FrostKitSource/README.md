# The Legitimate Heirs — FrostKit editor source package

Этот каталог — **source-of-truth для переноса в FrostKit**, а не каталог готовых Unreal assets.

Почему здесь нет `.uasset`:

- FrostKit / `Frostpunk2ModEditor` и Frostpunk 2 отсутствуют в checkout/среде;
- `.uasset` — бинарные Unreal assets с editor-generated references и Blueprint graphs;
- создавать их текстом или «угадывать» сериализацию нельзя: это даст повреждённый мод, а не заготовку.

## Как использовать после установки FrostKit

1. Прочитать `../Docs/FROSTKIT_OFFLINE_WIKI_2026-11-19.md` (§2–5, §8, §13).
2. Открыть sample scenario в **той же** версии FrostKit.
3. Заполнить `EDITOR_VERIFICATION_LOG.md`: не переписывать `TBD` по памяти.
4. Создать все native assets из `LegitimateHeirs.editor-manifest.yaml` в собственной постоянной Content folder.
5. Выполнить `ACT1_EDITOR_WIRING.md`, после каждого маленького шага — Compile/Save/PIE.
6. Ввести тексты из `Narrative/act1_dialogue_source.csv` в реальные Dialog/Dilemma assets и собрать RU localization через Localization Dashboard.
7. Проверить save/load, затем Cook Mod и запуск не из PIE.

## Scope guard

Нельзя реализовывать assets, которые campaign-law document прямо помечает «НЕ РЕАЛИЗОВЫВАЙ», без отдельного обновления документов автором. См. `../Docs/LEGITIMATE_HEIRS_IMPLEMENTATION_STATUS_2026-11-19.md`.
