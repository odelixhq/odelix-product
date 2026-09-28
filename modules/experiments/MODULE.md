# RES-EXP — experiments

**Owner repo:** `odelix-product` · **Activation gate:** RES-1 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/experiments`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Experiment orchestration, protocol и reproducibility manifest; dataset/code/engine pins и job lifecycle. Не второй compute runtime.

## Public ports и контракты

FreezeExperiment, StartTrial, GetTrial, CancelTrial; emits ExperimentFrozen/TrialStarted/TrialCompleted/TrialFailed.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

RES-HYP/FTR/SCOPE; RES-LED append port; MKT-DQ readiness and MKT-RCP compute; PLT quota/rights.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

DRAFT→FROZEN→RUNNING→COMPLETED/FAILED/CANCELLED. Reservation and trial-id assigned before scheduling; external job/outbox idempotent.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Failed/cancelled runs remain searchable; holdout cannot be re-used for silent tuning; cancellation of experiment does not cancel live order.

## Acceptance и review

Crash after enqueue; duplicate StartTrial; expired rights before run; modified dataset digest; quota reservation release; no hidden second trial on retry.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.


## Приёмка r15

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI. См. [SEMANTIC-ASSESSMENT-LIFECYCLE.md](../../docs/SEMANTIC-ASSESSMENT-LIFECYCLE.md) и owner Issue в [общем backlog](../../docs/DEVELOPMENT.md#DEP-b335630551). Новые гарантии требуют tests/evidence; сохранённая спецификация не означает implemented.
