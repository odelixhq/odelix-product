# RES-STR — strategy-registry

**Owner repo:** `odelix-product` · **Activation gate:** RES-1/2 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/strategy-registry`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Immutable strategy versions, compiler/result pins, validation provenance и promotion state. Не владеет текущими orders/positions.

## Public ports и контракты

RegisterStrategyVersion, AttachValidation, ProposePromotion, GetStrategyPassport; consumers receive exact version/digests.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

RES-HYP/EXP/VAL; MKT-OSC compiled artifacts; CAP-APP approvals and EXE runner port when activated.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

IDEA/SPEC/EXPERIMENT/VALIDATED/PAPER_ELIGIBLE/LIVE_ELIGIBLE distinct from deployment RUNNING/PAUSED. New inputs produce new version.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Changing code/data/model invalidates prior applicability until reviewed; live eligibility binds environment/account/policy. A failed alpha hypothesis remains a preserved strategy record.

## Acceptance и review

Revoke promotion; stale validation digest; exact compiled version used by paper; unauthorized live request; no mutation of running deployment by draft edits.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.


## Приёмка r15

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI. См. [SEMANTIC-ASSESSMENT-LIFECYCLE.md](../../docs/SEMANTIC-ASSESSMENT-LIFECYCLE.md) и owner Issue в [общем backlog](../../docs/DEVELOPMENT.md#DEP-b335630551). Новые гарантии требуют tests/evidence; сохранённая спецификация не означает implemented.
<!-- R159 strategy-binding -->
## Binding к AgentManifest

AgentSpec может ссылаться на immutable StrategyVersion. Смена стратегии/данных/policy меняет binding digest и требует проверки применимости approval. Generic SimulationScenarioRef не преобразуется автоматически в validated strategy и не получает выдуманный ValidationReport. Registry хранит версии, не перезаписывает ранее применённые определения.
