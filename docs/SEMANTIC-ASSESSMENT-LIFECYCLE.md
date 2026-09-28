# Product: episodes, assessment revisions и прогнозы

> **Текущая редакция документа: r15.11, 28 сентября 2026.** Добавлен ранний external-data scope; исходные исторические срезы ниже датированы отдельно. Изменение спецификации не является выполнением runtime/источниковой приёмки.

> **Основа и reuse текущей редакции:** [локальная карта внешних компонентов и адаптеров](DEVELOPMENT.md#implementation-basis). Там указаны источник, режим использования, наши модули/пути, задачи и ограничения. Кандидат не установленная зависимость; этот предметный документ не требует реализации библиотечной механики с нуля.


**1.0.0 · 2026-09-28 · PROPOSED.** Runtime: `odelix-product/apps/product-host/` Fastify; предметные modules — `packages/investigation/`, `packages/research/`, adapters — `packages/adapters/`.

## Объекты

Market owns числа и source provenance; Harness owns semantic protocol и evaluator; Product owns доступ, сохранение, revision, presentation disposition и связь с Scene/Evidence/Thesis. WKS не хранит отдельную authoritative копию inference history.

Новые предлагаемые структуры в существующем Investigation: `EpisodeRef`, `AssessmentRevision`, `AssessmentDisposition`, `AsShownTimeline`. Не новый bounded context на каждое имя. Prediction/ForecastSpec и promotion принадлежат Research, а не Agent pane.

## Lifecycle

`requested → running → received → validated → visible | stale | rejected | abstained | error`.
Состояние `visible` требует совпадения request/input/revision/scope, пригодного evidence и актуальных permissions. Late receive сохраняется с исходным snapshot и reason; active bar/new selection не меняются. Повтор request ID идемпотентен. Provider unknown outcome сначала reconciled; не дублировать charges вслепую.

Хранить `eventAt`, `availableAt`, `requestedAt`, `receivedAt`, `visibleAt`, version/model/question set, input digest, evidence refs, actual output и quality. Любая переоценка создаёт новый объект, не меняет исходно показанный. As-shown playback не вызывает provider; reanalysis явно маркирована новой моделью/временем.

## Быстрый endpoint

Предлагаемый read-analysis use case: `requestSemanticAssessment(episodeRef, contextRef)`. Auth/tenant/data rights/consent/budget перед вызовом injected Harness port. Итог доставляется через существующие streams/outbox. Это не buy/sell operation и не пользовательский код, исполняемый в Fastify.

Watch draft, semantic annotation и executable condition — разные сущности. Запрос «сделать условие» создаёт Research draft с независимыми entry/exit, не запускает position. Semantic confidence не может открыть execution permission.

## Forecast

ForecastSpec содержит outcome definition, horizon, instrument/venue/universe, label availability, sample selection, dataset/train/validation/holdout cuts, metric и promotion policy. ForecastResult обязан иметь model/version, timestamp, calibration evidence и applicability. Никакого default «вероятность вверх» без горизонта. Числа не заполняются Jev confidence или similarity score. Negative/inconclusive — корректное завершение.

## Persistence и права

Использовать module-owned Postgres schemas и artifact refs существующей Platform. Retention/delete/export учитывают права и consent; immutable audit означает traceable amendment, не запрет законного удаления персональных данных. Секреты не включаются в JSON/trace. Shared cache не пересекает приватные user contexts.

## Приёмка

ODX-PRD-027: минимальные Scene/Selection contracts раньше полного Workspace.
ODX-PRD-028: append-only revisions и as-shown read.
ODX-PRD-029: event admission/cache/budgets и no-network fallback.
ODX-PRD-030: independent forecast promotion, не release requirement для raw/semantic UI.

Тесты: две параллельные оценки; поздний ответ; другая session/tenant; replay до visibleAt; смена model pin; пропавшее evidence; ретрай без double settlement; невалидный provider payload. Все являются требованиями, не проведёнными проверками.

<a id="external-observations"></a>
## Внешние observations в as-shown истории

External context фиксируется по layer DataBinding: actual publisher, intermediary, receipt/dataset/revision, transforms, known time и quality. Сохранённый OpenBB response не делает temporal guarantee равной native journal replay; historical availableAt может быть unknown.

Assessment хранит ссылки на разрешённые artifacts, normalizer version, exact context digest и egress-policy decision; не загружает актуальные данные при reopen. Новый provider pull и новая нормализация создают явную новую revision. Право display не означает право LLM egress, cross-user cache или бессрочную retention. Если data-rights отозваны, producer может сделать artifact unavailable; прошлый inference не переименовывается в новый факт.

Получив candles/current chain без microstructure, система не заявляет CVD/absorption/options tape из отсутствующих данных. Успех внешнего транспорта не валидирует semantics/forecast.
<!-- R159 generic-decision-store -->
## Общий event primitive и независимый аудит

SemanticAssessment ссылается на базовую immutable history и DatasetBinding; generic DecisionRecord storage не зависит от HAR-014, Jev или этой специализации. Поздняя semantic revision добавляется отдельно и не переписывает as-shown. Audit retention, пользовательская память, eval и model training — разные операции с разными правами; запрет distillation не означает автоматически запрет любого audit-результата.

Blocked provider не мешает читать разрешённые imported decision records. Source/cut/knownAt/units остаются у inputs; ссылка без доступных bytes не запускает скрытый refetch.
