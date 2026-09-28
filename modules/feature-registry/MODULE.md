# RES-FTR — feature-registry

**Уточнение scope Research: r12.1 · 2026-09-18.** Пользовательские примеры не определяют встроенную стратегию или обязательный dataset; статус реализации не меняется.

**Owner repo:** `odelix-product` · **Activation gate:** RES-1 · **Design:** 1.0.1 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/feature-registry`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Definitions, units, availability, versions и lineage признаков. Численный расчёт выполняет Market research port, без дублирующего TS indicators engine.

## Public ports и контракты

RegisterFeature, ResolveFeatureVersion, CompileFeatureDependencies → FeaturePlan; results reference definition digest.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

MKT-RCP compute, MKT-TQ data, MKT-DQ quality, RES-SCOPE universe.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Immutable FeatureDefinition с lookback/calendar/smoothing/initialization/missing policy; renamed label не меняет semantic id.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Каждый признак фиксирует clock, параметры, units, smoothing и warmup; dependency graph включает inputs для entry/exit/target. CVD по реальным сделкам не смешивается с candle proxy; availability lag обязательный. Один и тот же runtime вычисляет разные пользовательские определения без именованных strategy branches.

## Acceptance и review

Parameterized warmup; incomplete bar rejection; asynchronous multi-source as-of join; semantic digest changes on source/window/smoothing; proxy labels; invalid dependency/unit/cycle rejection; reference fixture comparison.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.


## Приёмка r15

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI. См. [SEMANTIC-ASSESSMENT-LIFECYCLE.md](../../docs/SEMANTIC-ASSESSMENT-LIFECYCLE.md) и owner Issue в [общем backlog](../../docs/DEVELOPMENT.md#DEP-b335630551). Новые гарантии требуют tests/evidence; сохранённая спецификация не означает implemented.
