<!-- GENERATED_TEMPLATE: publisher=odelix-stack source=templates/MODULE.md; update via bundle rebuild, not separately -->
# MODULE — стандарт описания и проверки бизнес-модуля Odelix

**Редакция:** 1.0.0 · **Дата:** 2026-09-17. Это общий стандарт, а не один модуль всей системы.

Модуль — связная единица предметной ответственности в коде: владеет определёнными решениями/состоянием, публикует небольшой контракт, ограничивает зависимости и доказывает инварианты тестами. Stateless compute владеет алгоритмом и семантикой результата; ему не нужна придуманная таблица. DDD context, repo, crate/package, service и module — разные границы. Один модуль может занимать несколько crates; crate может временно обслуживать несколько явно разделённых ports. Перенос рабочего кода ради симметричных папок не является целью.

## 1. Паспорт и текущий статус

Указать ModuleId, название, bounded context, owner repo, семантического владельца, activation gate, design version. Отдельно implementation status: `SPEC_ONLY`, `PARTIAL`, `IMPLEMENTED_NOT_VERIFIED`, `VERIFIED_AT_COMMIT`. Для последних трёх нужны code paths/commit и конкретные проверки. Дата документа не является датой готовности кода. `PROPOSED` не получает release digest.

## 2. Назначение и граница

Описать одну устойчивую ответственность, outcome для consumer, решения/данные, которыми модуль владеет. Затем явно назвать соседние ответственности и их владельцев. Вопрос перед созданием: какую существующую реализацию нашли, почему её нельзя расширить, и какая независимая семантика требует нового модуля? Результат поиска — paths/символы/tests, а не фраза «дубликатов нет» без проверки.

## 3. Карта кода и слои

Различать `existing_path` и `planned_path`. Domain rules не зависят от сети/БД/модели; application реализует use cases; contracts публикуют types/ports; infrastructure реализует I/O; composition root связывает зависимости. Rust: crate exports/privacy и dependency checks. TypeScript: export maps/import allowlists и module tests. Solidity: согласованная финансовая transaction boundary, роли и upgrade surface. Не заводить интерфейс для каждой функции без причины.

## 4. Публичные операции

Для каждой command/query/event: schema id/version, input/output, обязательные units/currency/times, preconditions, typed errors, side effects, idempotency scope, ordering, auth/tenant policy, deadline/cancel semantics. Указать consumer и semantic fixtures. Декларация метода не означает опубликованную schema. Contract source находится у producer; Stack registry хранит discovery/version/digest.

## 5. Зависимости и запрещённые связи

Перечислить allowed provider ports и причину каждого. Запретить private source imports, cross-module table writes, cycles и hidden I/O в domain calculation. Смежный consumer получает immutable input или port, не доступ к внутреннему ORM. Общие IDs/envelopes не должны превращать contracts-kernel в склад risk/pricing/business logic.

## 6. Состояние, транзакции и воспроизведение

Указать authoritative state, projections/cache, states/guards/transitions, concurrency/version conflicts, transaction boundary, migration/rebuild. Для вычислительного модуля — отсутствие mutable business state и точные versioned inputs. Для ledger — conservation и атомарная граница. Для внешнего действия — `UNKNOWN` и reconciliation до повторной отправки. Event replay должен восстанавливать заявленное состояние при закреплённых inputs/versions.

## 7. Время и численные правила

Определить event/receive/available/asOf times, closed intervals/bar boundaries, clock source, freshness/missing policy, correction/vintage и timezone. Определить quantity/price scales, rounding, native/base currency, multiplier, funding/fee conventions. Money balances — fixed point/integer; численный pricing — documented tolerance. Если сравнение между языками допускает tolerance, её задают до теста.

## 8. Инварианты и отказ

Каждый материальный invariant связан с validation point и meaningful test. Указать fail-open/fail-closed по операции, degraded state, retryable/nonretryable ошибки и recovery. Missing data не превращается в ноль; timeout отправки не означает отказ venue; paused strategy не означает отсутствия позиций. LLM output не заменяет policy authorization.

## 9. Наблюдаемость и безопасность

Метрики latency/cost/queue/error/coverage, correlation ids, audit events, allowed telemetry fields, retention. Tenant isolation и secrets redaction. Для пользовательского кода — CPU/memory/time/network/data bounds. Для tool — side-effect classification и scopes. Для денег — независимый reconciliation и action-specific emergency controls.

## 10. Интегрируемые компоненты

Dependency + exact release/commit при реализации, licence/NOTICE, выбранный integration mode, собственный adapter, ownership внешнего состояния, замена/экспорт, обязательные conformance tests. Не писать «используем библиотеку» без того, какую функцию она выполняет и какое важное решение всё ещё принимает Odelix.

## 11. Acceptance и доказательство границы

Нужны tests поведения: golden/numerical/property, semantic contract, temporal leakage, duplicate/retry/race, fault/recovery — по реальному риску. Проверки структуры папок не заменяют их. В PR видны public exports, allowed dependencies и composition. Записывается точная выполненная команда и результат либо честно `planned, not run`. Reviewer проверяет code reuse/placement, инварианты, consumer impact и документацию. Простые editorial изменения не требуют симулированного полноценного test suite.

## 12. Связи с задачами и документацией

Ссылка на Issue/PR, контракт, invariant, dataset или ADR достаточна; не копировать один отчёт в пять мест. Builder обновляет факты своего модуля; Coordinator проверяет свежесть индекса, roadmap dependencies и следующие задачи. Founder принимает существенные изменения продукта/риска. MODULE не содержит второй глобальный roadmap.

## Основа, внешний код и интеграции

Указать IDs локальных integration bindings; точный внешний package/file/service; режим LIBRARY/SERVICE_ADAPTER/SELECTIVE_PORT/REUSE_OWN/REFERENCE; что разрабатываем сами и где adapter. Записать observed pin отдельно от adopted pin, license/NOTICE, tests/known gaps и связанную Issue. Если внешняя основа не нужна — объяснить, какая существующая Odelix-реализация переиспользуется. Не придумывать upstream adoption.
