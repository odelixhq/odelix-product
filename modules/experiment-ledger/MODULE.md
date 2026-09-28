# RES-LED — experiment-ledger

**Уточнение scope Research: r12.1 · 2026-09-18.** Приёмка общего механизма, не именованной стратегии.

**Owner repo:** `odelix-product` · **Activation gate:** RES-1 · **Design:** 1.0.1 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/experiment-ledger`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Append-only учёт всех попыток, search budget и причин failure/exclusion. Не только leaderboard победителей.

## Public ports и контракты

AppendTrialEvent, ListTrials(searchId), SummarizeSearch → attempts/exclusions/selection history.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

RES-EXP lifecycle events; PLT tenant/audit; artifact references, no direct compute mutation.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Immutable events with monotonically checked per-trial revision; correction via new event. Audit and report expose number of tested parameter combinations.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Retry same operation id не новый эксперимент; новый threshold/model input — новый trial. Delete request handled by policy with minimal integrity metadata where permitted.

## Acceptance и review

Dedup events; out-of-order arrival; failed trial persistence; search-budget exhaustion; tenant isolation; report includes every attempted user-configured input/entry/exit variant, including failed and negative trials.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.


## Приёмка r15

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI. См. [SEMANTIC-ASSESSMENT-LIFECYCLE.md](../../docs/SEMANTIC-ASSESSMENT-LIFECYCLE.md) и owner Issue в [общем backlog](../../docs/DEVELOPMENT.md#DEP-b335630551). Новые гарантии требуют tests/evidence; сохранённая спецификация не означает implemented.
