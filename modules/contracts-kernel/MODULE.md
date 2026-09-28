# PLT-CK — contracts-kernel

**Owner repo:** `odelix-product` · **Activation gate:** R0/G0 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/contracts-kernel`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Domain-neutral IDs, refs, revisions и command/event envelope conventions. Не единая огромная модель всего бизнеса.

## Public ports и контракты

Published neutral types, schema composition helpers and common error envelope; producers extend with domain types in their own packages.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

Minimal serialization/schema primitives; no auth database, market pricer, exchange or harness imports.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

No mutable business state. Versioned pure schema/validation semantics; exact released artifacts pinned by consumer.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Amounts preserve scale/precision; IDs do not imply access; optional envelope fields cannot conceal required domain preconditions.

## Acceptance и review

Large integer serialization TS/Rust; version compatibility; unknown enum behavior; deterministic ref/hash fixtures; no cross-domain dependency cycles.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.


## Приёмка r15

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI. См. [SEMANTIC-ASSESSMENT-LIFECYCLE.md](../../docs/SEMANTIC-ASSESSMENT-LIFECYCLE.md) и owner Issue в [общем backlog](../../docs/DEVELOPMENT.md#DEP-b335630551). Новые гарантии требуют tests/evidence; сохранённая спецификация не означает implemented.
<!-- R159 agent-contracts -->
## Agent и decision contracts

Product owns agent.spec/agent.manifest/node registration/model.signal schemas; Execution publishes producer decision.event.v1/decision.egress.v1, policy.bundle и simulation.scenario. Shared value objects не дают Product права переписать чужую policy. Потребитель pin-ит artifact/schema и fixtures; соседний checkout не prerequisite.

Конкретные схемы drafts размещены в локальном contracts/. Publication и runtime conformance остаются невыполненными.
