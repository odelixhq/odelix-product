# odelix-product — руководство разработчика

**r15.11 · 2026-09-28 · PROPOSED / не статус runtime.** В этом файле объединены локальная архитектура, контракты, карта модулей, тесты и выпуск. Подробные предметные спецификации сохранены отдельно.

[Вход в repo](../AGENTS.md) · [Локальная выборка задач](../delivery/CONTEXT.json)


<!-- BEGIN GENERATED IMPLEMENTATION BASIS -->
<a id="implementation-basis"></a>
## Основа реализации: что берём, что пишем и где интегрируем

**r15.11. Основа — Fastify + TypeScript, PostgreSQL/pg и typed domain modules. Identity/billing/storage являются provider adapters; бизнес-состояние и права принадлежат Odelix. External-data policy и credentials остаются PLT-DG; local preview не зависит от полного OIDC, public доступ требует действующих grants.**

Это локальная генерируемая выборка общего решения, а не отдельный редактируемый реестр. Полные metadata — `delivery/CONTEXT.json → implementation_blueprint`. Изменения предлагает владелец repo через Coordinator; генератор обновляет общий и локальные виды вместе. Конкретный upstream/pin и supply-chain проверяются перед включением, не по наличию названия в таблице.

**Три разных зависимости:** библиотека/внешний код; внешний сервис/данные; контракт другого Odelix repo. Последний не разрешает копировать чужую реализацию. Ниже сами архитектурные bindings; реестр документальных источников в конце файла — другая сущность.

| Источник | Режим | Модули этого repo | Задачи |
|---|---|---|---|
| [OWN](#reuse-BND-013) | REUSE_OWN | AREA-PRD-INTEGRATIONS | [ODX-PRD-005](#issue-ODX-PRD-005) |
| [FASTIFY](#reuse-BND-014) | LIBRARY | AREA-PRD-HOST, PLT-CP, PLT-RUN | [ODX-PRD-001](#issue-ODX-PRD-001), [ODX-PRD-008](#issue-ODX-PRD-008), [ODX-PRD-014](#issue-ODX-PRD-014) |
| [TEST](#reuse-BND-015) | LIBRARY | AREA-PRD-HOST, INV-CHG | [ODX-PRD-001](#issue-ODX-PRD-001), [ODX-PRD-011](#issue-ODX-PRD-011) |
| [PG](#reuse-BND-016) | LIBRARY_SERVICE | AREA-PRD-NOTIFICATIONS, AREA-PRD-PERSISTENCE, CAP-EXE, INV-CTX, INV-MEM, INV-THS, INV-WAT, INV-WSP, PLT-BIL, PLT-CP, PLT-RUN, RES-EXP, RES-FTR, RES-STR | [ODX-PRD-002](#issue-ODX-PRD-002), [ODX-PRD-004](#issue-ODX-PRD-004), [ODX-PRD-006](#issue-ODX-PRD-006), [ODX-PRD-009](#issue-ODX-PRD-009), [ODX-PRD-010](#issue-ODX-PRD-010), [ODX-PRD-013](#issue-ODX-PRD-013), [ODX-PRD-014](#issue-ODX-PRD-014), [ODX-PRD-016](#issue-ODX-PRD-016), [ODX-PRD-017](#issue-ODX-PRD-017), [ODX-PRD-019](#issue-ODX-PRD-019), [ODX-PRD-020](#issue-ODX-PRD-020), [ODX-PRD-021](#issue-ODX-PRD-021), [ODX-PRD-022](#issue-ODX-PRD-022), [ODX-PRD-024](#issue-ODX-PRD-024), [ODX-PRD-026](#issue-ODX-PRD-026) |
| [OIDC](#reuse-BND-017) | SERVICE_ADAPTER | PLT-CP, PLT-TA | [ODX-PRD-003](#issue-ODX-PRD-003), [ODX-PRD-008](#issue-ODX-PRD-008), [ODX-PRD-025](#issue-ODX-PRD-025) |
| [PI](#reuse-BND-018) | CONSUME_OWN_RELEASE | PLT-RUN | [ODX-PRD-007](#issue-ODX-PRD-007), [ODX-PRD-014](#issue-ODX-PRD-014) |
| [ARROW](#reuse-BND-019) | LIBRARY | RES-EXP | [ODX-PRD-018](#issue-ODX-PRD-018) |
| [MCP](#reuse-BND-020) | LIBRARY | PLT-CP | [ODX-PRD-008](#issue-ODX-PRD-008) |
| [CEL](#reuse-BND-021) | CONDITIONAL_REFERENCE | INV-THS, INV-WAT | [ODX-PRD-010](#issue-ODX-PRD-010), [ODX-PRD-022](#issue-ODX-PRD-022) |
| [TA](#reuse-BND-022) | SELECTIVE_PORT | INV-MEM | [ODX-PRD-026](#issue-ODX-PRD-026) |
| [STRIPE](#reuse-BND-023) | SERVICE_ADAPTER | PLT-BIL | [ODX-PRD-012](#issue-ODX-PRD-012), [ODX-PRD-013](#issue-ODX-PRD-013) |
| [DAILY](#reuse-BND-024) | SELECTIVE_PORT | INV-WAT | [ODX-PRD-023](#issue-ODX-PRD-023) |
| [AIHF](#reuse-BND-025) | CONDITIONAL_REFERENCE | RES-EXP | [ODX-PRD-090](#issue-ODX-PRD-090), [ODX-PRD-034](#issue-ODX-PRD-034) |
| [OPENBB](#reuse-BND-049) | CONTRACT_CONSUMER_NOT_OPENBB_IMPORT | PLT-DG, INV-CTX | [ODX-PRD-031](#issue-ODX-PRD-031) |

<a id="reuse-BND-013"></a>
### OWN: REUSE_OWN

**Берём:** Наш опубликованный Market/compute contract и fixtures; существующее Rust-ядро остаётся в Market.

**Пишем сами:** Локальный consumer adapter и независимая интеграционная проверка; для Execution — согласованный simulation port.

**Не переносим / граница:** Не копировать book/journal/sim в этот repo и не использовать чужие приватные пути.

**Источник:** `https://github.com/a3ka/hft-platform/tree/4ddd392e14e1dbd4c0b2511e50ddc9e48b8707e0`. **Source pin:** `4ddd392e14e1dbd4c0b2511e50ddc9e48b8707e0`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-005](#issue-ODX-PRD-005) / `AREA-PRD-INTEGRATIONS` | `packages/adapters/market-client/src/`; `packages/adapters/market-client/tests/conformance.test.ts`; `packages/platform/data-rights/` |

<a id="reuse-BND-014"></a>
### FASTIFY: LIBRARY

**Берём:** fastify, fastify-plugin и необходимые совместимые plugins; framework/plugin lifecycle.

**Пишем сами:** Наш buildApp/config/routes, domain composition и безопасный shutdown.

**Не переносим / граница:** Fastify не реализует агентное планирование, domain state или расчёт рынка.

**Источник:** `https://fastify.dev/docs/latest/Reference/Plugins/`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-001](#issue-ODX-PRD-001) / `AREA-PRD-HOST` | `apps/product-host/src/app.ts`; `apps/product-host/src/server.ts`; `apps/product-host/src/config.ts`; `apps/product-host/tests/host.test.ts`; `pnpm-workspace.yaml` |
| [ODX-PRD-008](#issue-ODX-PRD-008) / `PLT-CP` | `apps/product-host/src/plugins/connect.ts`; `apps/product-host/src/routes/oauth-resource.ts`; `tests/mcp-authorization.test.ts`; `tests/mcp-streaming.test.ts` |
| [ODX-PRD-014](#issue-ODX-PRD-014) / `PLT-RUN` | `packages/platform/run-control/`; `packages/adapters/harness-store/`; `tests/run-crash-recovery.test.ts`; `tests/events-reconnect.test.ts` |

<a id="reuse-BND-015"></a>
### TEST: LIBRARY

**Берём:** Vitest, fast-check, Playwright — test/property/browser harness по выбранным package pins.

**Пишем сами:** Независимые semantic oracles, negative fixtures, host/consumer integration и отчёт о выполнении.

**Не переносим / граница:** Наличие test framework или чужой зелёный CI не доказывает наш acceptance.

**Источник:** `https://vitest.dev/`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-001](#issue-ODX-PRD-001) / `AREA-PRD-HOST` | `apps/product-host/src/app.ts`; `apps/product-host/src/server.ts`; `apps/product-host/src/config.ts`; `apps/product-host/tests/host.test.ts`; `pnpm-workspace.yaml` |
| [ODX-PRD-011](#issue-ODX-PRD-011) / `INV-CHG` | `packages/investigation/changes/`; `apps/product-host/src/routes/commands.ts`; `tests/changeset.test.ts` |

<a id="reuse-BND-016"></a>
### PG: LIBRARY_SERVICE

**Берём:** PostgreSQL + node-postgres (pg): соединение/транзакции и storage.

**Пишем сами:** Module-owned schemas, migrations, tenant boundaries и adapters бизнес-объектов.

**Не переносим / граница:** SQL engine не заменяет наш журнал рынка; таблица не становится вторым owner чужого domain.

**Источник:** `https://node-postgres.com/`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-002](#issue-ODX-PRD-002) / `AREA-PRD-PERSISTENCE` | `packages/adapters/postgres/src/`; `packages/platform/run-control/adapters/postgres/`; `packages/platform/tenant/adapters/postgres/`; `migrations/`; `tests/postgres-isolation.test.ts` |
| [ODX-PRD-004](#issue-ODX-PRD-004) / `PLT-CP` | `packages/platform/capabilities/`; `packages/platform/data-rights/`; `packages/platform/run-control/`; `apps/product-host/src/routes/runs.ts`; `tests/run-admission.test.ts` |
| [ODX-PRD-006](#issue-ODX-PRD-006) / `INV-CTX` | `packages/investigation/context/`; `packages/investigation/evidence/`; `apps/product-host/src/routes/contexts.ts`; `tests/context-compiler.test.ts` |
| [ODX-PRD-009](#issue-ODX-PRD-009) / `INV-WSP` | `packages/investigation/workspace/`; `packages/investigation/evidence/`; `apps/product-host/src/routes/workspaces.ts`; `tests/workspace-evidence.test.ts` |
| [ODX-PRD-010](#issue-ODX-PRD-010) / `INV-THS` | `packages/investigation/thesis/`; `packages/investigation/watch/`; `tests/thesis-watch-draft.test.ts` |
| [ODX-PRD-013](#issue-ODX-PRD-013) / `PLT-BIL` | `packages/platform/billing/src/usage/`; `packages/platform/run-control/`; `tests/usage-reservations.test.ts` |
| [ODX-PRD-014](#issue-ODX-PRD-014) / `PLT-RUN` | `packages/platform/run-control/`; `packages/adapters/harness-store/`; `tests/run-crash-recovery.test.ts`; `tests/events-reconnect.test.ts` |
| [ODX-PRD-016](#issue-ODX-PRD-016) / `RES-FTR` | `packages/research/feature-registry/`; `packages/research/hypothesis/`; `apps/product-host/src/routes/research-features.ts`; `tests/feature-registry.test.ts` |
| [ODX-PRD-017](#issue-ODX-PRD-017) / `RES-EXP` | `packages/research/experiments/`; `packages/research/experiment-ledger/`; `apps/product-host/src/routes/experiments.ts`; `tests/experiment-ledger.test.ts` |
| [ODX-PRD-019](#issue-ODX-PRD-019) / `RES-STR` | `packages/research/statistical-validator/`; `packages/research/strategy-registry/`; `tests/strategy-promotion.test.ts` |
| [ODX-PRD-020](#issue-ODX-PRD-020) / `RES-STR` | `packages/research/deployments/`; `packages/platform/approvals/`; `tests/paper-deployment.test.ts` |
| [ODX-PRD-021](#issue-ODX-PRD-021) / `CAP-EXE` | `packages/adapters/execution-client/`; `packages/capital/execution-projection/`; `tests/execution-projection.test.ts` |
| [ODX-PRD-022](#issue-ODX-PRD-022) / `INV-WAT` | `packages/investigation/watch/`; `packages/investigation/missions/`; `packages/adapters/watch-predicate/`; `tests/watch-mission-state.test.ts` |
| [ODX-PRD-024](#issue-ODX-PRD-024) / `AREA-PRD-NOTIFICATIONS` | `packages/platform/notifications/src/outbox/`; `packages/adapters/notification-webhook/`; `tests/notification-recovery.test.ts` |
| [ODX-PRD-026](#issue-ODX-PRD-026) / `INV-MEM` | `packages/investigation/memory/`; `packages/investigation/memory/NOTICES.md`; `tests/memory-known-at.test.ts` |

<a id="reuse-BND-017"></a>
### OIDC: SERVICE_ADAPTER

**Берём:** Managed OIDC, Auth0 — прежний кандидат; jose для верификации JWT/JWKS.

**Пишем сами:** IdentityPort, issuer/audience/expiry, session boundary; отдельные Odelix entitlements и grants.

**Не переносим / граница:** IdP не выдаёт право читать любой инструмент и не вычисляет Odelix billing permissions.

**Источник:** `https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-003](#issue-ODX-PRD-003) / `PLT-TA` | `packages/platform/tenant/`; `packages/adapters/identity-auth0/`; `apps/product-host/src/plugins/auth.ts`; `tests/identity.test.ts` |
| [ODX-PRD-008](#issue-ODX-PRD-008) / `PLT-CP` | `apps/product-host/src/plugins/connect.ts`; `apps/product-host/src/routes/oauth-resource.ts`; `tests/mcp-authorization.test.ts`; `tests/mcp-streaming.test.ts` |
| [ODX-PRD-025](#issue-ODX-PRD-025) / `PLT-CP` | `packages/platform/capabilities/src/market-subscription.ts`; `apps/product-host/src/routes/market-grants.ts`; `tests/market-subscription-grant.test.ts` |

<a id="reuse-BND-018"></a>
### PI: CONSUME_OWN_RELEASE

**Берём:** Публичный Odelix Harness package/events, построенный владельцем поверх Pi; не второй Pi fork в этом repo.

**Пишем сами:** Наш host binding или typed client/session adapter; Product размещает Run, Workstation отображает его события.

**Не переносим / граница:** Не импортировать приватные Skills в клиент; не форкать Pi повторно; не строить новый agent loop.

**Источник:** `https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sdk.md`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-007](#issue-ODX-PRD-007) / `PLT-RUN` | `apps/product-host/src/plugins/harness.ts`; `packages/adapters/harness-store/`; `packages/adapters/harness-tools/`; `tests/hosted-run.test.ts` |
| [ODX-PRD-014](#issue-ODX-PRD-014) / `PLT-RUN` | `packages/platform/run-control/`; `packages/adapters/harness-store/`; `tests/run-crash-recovery.test.ts`; `tests/events-reconnect.test.ts` |

<a id="reuse-BND-019"></a>
### ARROW: LIBRARY

**Берём:** arrow-rs/Parquet для bounded batch import/export и артефактов.

**Пишем сами:** Mapping к нашим units, time/quality/source manifests; согласованный compute boundary.

**Не переносим / граница:** Не заменять HFTJRN02 и не считать float64-экспорт побитово точным по определению.

**Источник:** `https://github.com/apache/arrow-rs`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-018](#issue-ODX-PRD-018) / `RES-EXP` | `packages/adapters/research-compute/`; `packages/adapters/artifact-store/`; `packages/research/experiments/ports/`; `tests/research-job-adapter.test.ts` |

<a id="reuse-BND-020"></a>
### MCP: LIBRARY

**Берём:** Официальный MCP TypeScript SDK для protocol/schema/transport mechanics.

**Пишем сами:** Наш каталог разрешённых capabilities и mapping к единым Product/Harness use cases.

**Не переносим / граница:** MCP не бизнес-ядро; протокол не заменяет auth, grants, cost admission или agent runtime.

**Источник:** `https://ts.sdk.modelcontextprotocol.io/v2/`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-008](#issue-ODX-PRD-008) / `PLT-CP` | `apps/product-host/src/plugins/connect.ts`; `apps/product-host/src/routes/oauth-resource.ts`; `tests/mcp-authorization.test.ts`; `tests/mcp-streaming.test.ts` |

<a id="reuse-BND-021"></a>
### CEL: CONDITIONAL_REFERENCE

**Берём:** cel-expr/cel-spec: семантика ограниченных predicate expressions; конкретный evaluator ещё проверить.

**Пишем сами:** Наш typed predicate adapter, limits, missing/freshness/time semantics.

**Не переносим / граница:** Не произвольный eval/JavaScript и не обязательная установка нового runtime по ссылке на spec.

**Источник:** `https://github.com/cel-expr/cel-spec`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-010](#issue-ODX-PRD-010) / `INV-THS` | `packages/investigation/thesis/`; `packages/investigation/watch/`; `tests/thesis-watch-draft.test.ts` |
| [ODX-PRD-022](#issue-ODX-PRD-022) / `INV-WAT` | `packages/investigation/watch/`; `packages/investigation/missions/`; `packages/adapters/watch-predicate/`; `tests/watch-mission-state.test.ts` |

<a id="reuse-BND-022"></a>
### TA: SELECTIVE_PORT

**Берём:** TradingAgents tradingagents/agents/utils/memory.py: правило as-of lessons только по известным исходам.

**Пишем сами:** Product-owned persistent Memory, consent, tenant isolation и точное время разрешённого retrieval; Harness вызывает port.

**Не переносим / граница:** Не переносить файловый journal как multi-tenant storage и не создавать агентный runtime в Product.

**Источник:** `https://github.com/TauricResearch/TradingAgents/tree/2d17df8da1536c121e4d7395ac5a5dcec9e96d6f`. **Source pin:** `2d17df8da1536c121e4d7395ac5a5dcec9e96d6f`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-026](#issue-ODX-PRD-026) / `INV-MEM` | `packages/investigation/memory/`; `packages/investigation/memory/NOTICES.md`; `tests/memory-known-at.test.ts` |

<a id="reuse-BND-023"></a>
### STRIPE: SERVICE_ADAPTER

**Берём:** Stripe Billing/Checkout/portal и проверка webhook через SDK.

**Пишем сами:** BillingProviderPort, idempotent event projection, usage ledger/outbox и entitlements.

**Не переносим / граница:** Не доверять success URL; не вводить синхронную зависимость каждой операции от Stripe.

**Источник:** `https://docs.stripe.com/billing/subscriptions/webhooks`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-012](#issue-ODX-PRD-012) / `PLT-BIL` | `packages/platform/billing/`; `packages/adapters/billing-stripe/`; `apps/product-host/src/routes/billing-webhook.ts`; `tests/billing-webhooks.test.ts` |
| [ODX-PRD-013](#issue-ODX-PRD-013) / `PLT-BIL` | `packages/platform/billing/src/usage/`; `packages/platform/run-control/`; `tests/usage-reservations.test.ts` |

<a id="reuse-BND-024"></a>
### DAILY: SELECTIVE_PORT

**Берём:** daily_stock_analysis src/notification_noise.py: quiet-hours/severity/timezone helpers и соответствующие тесты.

**Пишем сами:** Tenant policy и durable dedup/outbox delivery в Product.

**Не переносим / граница:** Не process-local dictionary как распределённая доставка; не stock-analysis app и не встроенная стратегия.

**Источник:** `https://github.com/ZhuLinsen/daily_stock_analysis/tree/1168e316269baa38752a8901331a4d7aa8b1fd07`. **Source pin:** `1168e316269baa38752a8901331a4d7aa8b1fd07`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-023](#issue-ODX-PRD-023) / `INV-WAT` | `packages/platform/notifications/src/policy/quiet-hours.ts`; `packages/platform/notifications/src/policy/severity.ts`; `packages/platform/notifications/NOTICES.md`; `tests/notification-policy.test.ts` |

<a id="reuse-BND-025"></a>
### AIHF: CONDITIONAL_REFERENCE

**Берём:** ai-hedge-fund hedge_fund/signals/base.py: AlphaModel.predict/Signal interface как reference.

**Пишем сами:** Наш model adapter к существующему Research sandbox/compute contract.

**Не переносим / граница:** Не их portfolio/simulator, не defaults missing→neutral и не готовые стратегии как обязательный каталог.

**Источник:** `https://github.com/virattt/ai-hedge-fund/tree/154a8b2f46dca0f40764d814e4e747b0ad71f4c4`. **Source pin:** `154a8b2f46dca0f40764d814e4e747b0ad71f4c4`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-090](#issue-ODX-PRD-090) / `RES-EXP` | `packages/adapters/research-model/`; `tests/research-model-bridge.test.ts` |
| [ODX-PRD-034](#issue-ODX-PRD-034) / `RES-STR` | `contracts/model.signal.v1.schema.json`; `packages/adapters/research-model/`; `tests/model-signal.test.ts` |

<a id="reuse-BND-049"></a>
### OPENBB: CONTRACT_CONSUMER_NOT_OPENBB_IMPORT

**Берём:** OpenBB ODP pinned separate service for early Deribit bars/chain and ECB dated reference snapshots. FRED API profile disabled: bounded personal copies are separate, not ODP/Pi/product data.

**Пишем сами:** Rights/credentials partition, DataBinding and protected commercial access. No FRED fallback; service permissions cannot promote PERSONAL_USE corpus into product namespace.

**Не переносим / граница:** Не ODP Workspace/AI/recorder; no broad proxy, no vendor keys in client, no AGPL copy into private core. HTTP boundary не юридический safe harbor.

**Источник:** `https://github.com/OpenBB-finance/OpenBB/tree/3e071fcc2cd9f891cac6040ae60296dba76dab46`. **Source pin:** `3e071fcc2cd9f891cac6040ae60296dba76dab46`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `SOURCE_RECHECKED_2026_09_27_RUNTIME_NOT_RUN`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-PRD-031](#issue-ODX-PRD-031) / `PLT-DG` | `packages/platform/data-rights/external-source-policy.ts`; `packages/investigation/context/external-bindings.ts`; `tests/external-data-rights.test.ts` |

### Сервисы и datasets: прямые, общие и отложенные подключения

Только direct Issue означает scope конкретного подключения. Related/generic — класс работ, не выбранный provider. Future/deferred строки не надо реализовывать автоматически; они сохранены, чтобы отдельный repo не потерял архитектурный замысел.

| Источник | Покрытие | Прямые задачи | Связанный общий scope | Решение/условие |
|---|---|---|---|---|
| Auth0 / managed OIDC | EXPLICIT | `ODX-PRD-003`, `ODX-PRD-004`, `ODX-PRD-025` |  | Первый кандидат IdP; собственные IdentityPort, tenant и grants. Договор и production-настройки не подтверждены.; 1 |
| Stripe Billing | EXPLICIT | `ODX-PRD-012`, `ODX-PRD-013`, `ODX-WEB-004` |  | Checkout/webhooks/entitlements через отдельный adapter; eligibility и разрешение на подключение отдельно.; 6 |
| S3-compatible object storage / существующий offsite | GENERIC_SCOPE |  | `ODX-STK-003`, `ODX-STK-004`, `ODX-PRD-018`, `ODX-MKT-032` | Класс хранения и adapters предусмотрены; конкретный поставщик не назначен этой документацией. Сначала existing backup.; 0, 1, 4 |
| Первый канал уведомлений: test sink/webhook | EXPLICIT | `ODX-PRD-024` |  | Есть конкретная задача durable outbox и webhook adapter. Поставщики почты/SMS/push не выбраны; не считать их подключение выполненным.; 8 |
| TimescaleDB / отдельная time-series SQL БД | DEFERRED |  |  | Не заменяет журнал. В исходнике отложено; dedicated Issue отсутствует.; После workload evidence |
| viem/wagmi; Privy при подтверждённом friction | FUTURE_NO_ISSUE |  |  | Сначала standard external wallet, Privy условен. Read-only Research не требует кошелька.; Execution users / V1 |
| Envio HyperIndex; The Graph/Goldsky резерв | FUTURE_NO_ISSUE |  |  | Managed indexing кандидат, а не источник полномочий collateral. Нет текущей implementation Issue.; При собственных onchain contracts |

### Унаследованные варианты — не дополнительные зависимости

Связь по source group не доказывает выбор каждого пакета. Ниже сохранены reference-кандидаты, связанные с локальными sources. Более старые решения могут расходиться с текущим scope; не разрешать их молча.

| Компонент | Исторический статус | Роль/граница |
|---|---|---|
| [Apache Arrow / arrow-rs / Parquet](https://github.com/apache/arrow-rs) | `ADOPT` | Sealed canonical/raw artifacts и portable columnar packages; Только compaction/export adapters; semantic digest остаётся нашим |
| [PostgreSQL](https://www.postgresql.org/) | `ADOPT` | Domain state и rebuildable projections; Общая physical DB допустима; schemas/migrations принадлежат business modules |
| [CEL specification](https://github.com/cel-expr/cel-spec) | `ADOPT` semantics / `SPIKE` implementation | Безопасный, mutation-free, non-Turing-complete Watch predicate language; `WatchPredicatePort`; typed allowlist context, no I/O/network/secrets |
| [Pi](https://github.com/earendil-works/pi) | `FORK / OWNED FOUNDATION` | Весь Odelix Harness: runtime, skills/hooks/tools/teams/rules, Critic, memory, Missions, traces and evals; Exact upstream commit + patch digest; no external wrapper; Product/Market domain schemas remain producer-owned |
| [Official MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) | `ADOPT-LIMITED` | Transport/protocol projection для builder surface; MCP schemas map к тем же application commands/queries/policy; не в domain |
| Pi provider packages / direct provider SDKs | `FORK FOUNDATION / ADOPT-LIMITED` | Model/provider access внутри Pi fork; HarnessVersion pin, Product secret binding, one provider path per run |
| [fast-check](https://github.com/dubzzz/fast-check) | `ADOPT` | Property/model-based tests TypeScript domain/contract paths; Pinned seeds сохраняются как reproducible fixtures |

### Ранний внешний data layer (генерируемый scope)

OpenBB ODP SELECTED_FOR_IMPLEMENTATION, runtime NOT_RUN; в этом repo применяются только его обязанности. Native flow/replay и права не подменяются внешним snapshot. Полный normative contract принадлежит Market; consumer получает producer artifact.

| Profile | Mode | Доказанная граница / ограничения | Issues |
|---|---|---|---|
| ODP-DERIBIT-BARS | HISTORICAL_POLL | explicit date range; no full-history default; source precision preserved, not exact exchange reconstruction; no aggression/price-bin data | `ODX-MKT-091` |
| ODP-ECB-REFERENCE | DATED_CURRENT_SNAPSHOT | daily.xml, not historical series; not executable intraday FX price; observation date not exact publication timestamp | `ODX-MKT-091` |
| ODP-DERIBIT-CHAIN | COMPOSITE_CURRENT_SNAPSHOT | one connection per expiry in upstream, require measured fanout cap; 2s per expiry receive timeout; partial/exceptions not complete proof; BTC/ETH price conversion to USD and round(2); IV percent divided by 100; New York aware row timestamps normalized to UTC preserving instant; contract_size=1 and today-derived DTE are not verified terms; date-only expiry and current universe do not supply historical PIT | `ODX-MKT-037`, `ODX-WKS-021` |
| ODP-FRED-REVISED-PREVIEW | DISABLED_API_PROFILE_NOT_PERSONAL_FILE_ROUTE | Not installed/selected as active ODP fallback; FRED general restrictions apply beyond API: personal download not corpus storage; Use separate bounded personal file policy; no automated ODP/ML route | `ODX-MKT-090`, `ODX-MKT-042`, `ODX-MKT-043`, `ODX-MKT-044` |

Cache/replay policy: source+model+canonical instrument+query+normalization version+rights credential partition; reauthorize on hit; no silent provider fallback. stored OpenBB response plus versioned deterministic normalization, not upstream event replay; no vendor call during saved replay. Public activation: requires actual Product grants, source/AGPL/data rights and operational acceptance; local preview is not public authorization.

**Приёмка внешнего компонента:** existing-code check → точный package/commit и license/NOTICE/dependencies → adapter test с недоступностью/ошибками/качеством → фактический pin и результат. Не создавать второй runtime/численный engine под видом ускорения. Текст этого блока не устанавливает packages, не покупает сервисы и не выдаёт production rights.
<!-- END GENERATED IMPLEMENTATION BASIS -->

<a id="architecture"></a>
## Архитектура


<a id="уточнение-r151-scalable-read--не-поздняя-оптимизация"></a>
### Scalable read — не поздняя оптимизация

**21 сентября 2026 · PROPOSED; source/runtime status не перепроверен этой сборкой.** При изменении порядка ранних работ действует [план live-read](#DEP-13674f89c9) и [Phase-A requirements](#DEP-686a4a4cec). Рынок не принадлежит подписке: P0 truthfulness + cold guard, затем S1a contracts, S2 shared producer/local fan-out, S3 instrument core и S4 blocks. Ранний F — ограниченный pilot после P0/S2; STK-013 отдельно принимает workload-specific scale. Ни F, ни каталог файлов не доказывают 100k users. Данные/footprint/depth не переписывать заново; WKS продолжает fixture/runtime integration по готовым границам, не ждёт всего распределённого deployment.

Окна пользователя не key вычисления; 1s — только исторические aggregates, не запрет внутрисекундной книги/ленты. NoChange/Replace(empty)/Patch/Invalidated различаются. Историческая ликвидность сохраняется, текущая удаляется по выбранной семантике. Стабильный digest и state/meaning/wire versions с golden tests; converter optional, controlled prewarm обязателен. Все live updates и rollback остаются без публичного full replay. E/семантика потребляют общие instrument features; Product выдаёт grants, не становится прокси всех ticks.


One TypeScript/pnpm workspace, one Fastify composition host initially, one
PostgreSQL cluster with module-owned schemas/tables. Modules are packages with
`domain`, `application`, `contracts`, `ports`, `adapters` and tests where needed.

```text
apps/product-host
  -> packages/platform/*
  -> packages/investigation/*
  -> packages/research/* (RES-1/RES-2; independent numerical jobs via Market ports)
  -> packages/capital/* (DORMANT until its gate)
  -> packages/adapters/*
  -> exact @odelix/harness-runtime package (composition only)
```

Dependency direction: transport/adapters → application → domain; application
depends on required ports, never concrete vendor clients. Cross-module workflow
uses stable refs and a process manager. Transactions do not cross aggregate/module
boundaries; durable async/outbox appears only at a real reliability need.

Market access has two forms: short-lived direct subscription grant for clients,
and coarse PIT/feature queries through the generated Market client. Rust journal
or Product-internal market truth are forbidden.

CONNECT_COMMERCIAL path: OAuth/MCP request → tenant/entitlement/data-right decision → usage
reservation → durable Run intake/placement → hosted Pi → Product Context/Evidence
and Market tools → result/usage settlement. S0 embeds Pi package; S1/S2 use the
same contract in fenced workers. Product knows when/where a Run executes, not
how Skill agents/steps are selected.

Later Workstation path adds Scene/Selection and Product draft/apply, but reuses
the same hosted Skill. Product does not implement agent loop, Skills, teams,
Critic, memory policy, Missions, traces or evals; those live in
`odelix-harness`.


### Options и research integration (PROPOSED)

Use INV-CTX/EVD/THS for objective, context and evidence, not a parallel options backend. Market owns numerical candidates; CAP-APP owns exact approval when enabled; exchange EXC-CLR is not Product CAP-LED.

Detailed scope: [OPTIONS-WORKFLOW.md](OPTIONS-WORKFLOW.md).


### Research ownership

RES-HYP/EXP/LED/VAL/STR own spec/protocol/trials/verdict/version state. Market executes numerical jobs through market.research-compute.v1. INV-MEM owns durable consent-governed user memory; HAR-MEM proposes memory candidates and governs agent use. HAR-CTX owns prompt/context budgets; INV-CTX owns evidence meaning. These pairs are not duplicate databases or runtimes.


### Research/strategy capability requirements

StrategySpec, experiment/trial registry, validation и Strategy Passport. Основные owners: RES-SCOPE/FTR/HYP/EXP/LED/VAL/STR.

Новая поверхность использует эти же contracts и semantic fixtures; численные/торговые engines не копируются в UI или prompts. MODULE описывает actual/planned paths раздельно.


### Workstation-first r15: текущая приёмка

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI.

Каноническая детализация: [SEMANTIC-ASSESSMENT-LIFECYCLE.md](SEMANTIC-ASSESSMENT-LIFECYCLE.md); [current delivery](#DEP-b335630551).


<a id="contracts"></a>
## Контракты

**Статус этой сборки:** перечисленные семейства — спецификация. Заголовок «Published» в унаследованном тексте означает целевую поверхность публикации, не доказательство существующего release. Реальный pin/digest и conformance требуются до integration acceptance.


<a id="уточнение-r151-scalable-read--не-поздняя-оптимизация-1"></a>
### Scalable read — не поздняя оптимизация

**21 сентября 2026 · PROPOSED; source/runtime status не перепроверен этой сборкой.** При изменении порядка ранних работ действует [план live-read](#DEP-13674f89c9) и [Phase-A requirements](#DEP-686a4a4cec). Рынок не принадлежит подписке: P0 truthfulness + cold guard, затем S1a contracts, S2 shared producer/local fan-out, S3 instrument core и S4 blocks. Ранний F — ограниченный pilot после P0/S2; STK-013 отдельно принимает workload-specific scale. Ни F, ни каталог файлов не доказывают 100k users. Данные/footprint/depth не переписывать заново; WKS продолжает fixture/runtime integration по готовым границам, не ждёт всего распределённого deployment.


Public surfaces: Product commands/queries, object representations, direct-market
subscription grants, Context/Evidence/Thesis/Watch schemas and MCP resources/tools.
CONNECT_COMMERCIAL additionally publishes OAuth resource metadata, `EntitlementSnapshot`,
`UsageReservation/Event/Settlement`, `RunLease`, checkpoint reference and
worker-assignment contracts.
OpenAPI/JSON Schema is generated from semantic source and published with a TS
client and golden fixtures. Mutation commands require actor/tenant/capability,
idempotency key, expected revision and trace. MCP cannot bypass application policy.

Consumed Market contracts are pinned generated clients. Product never copies
Market DTOs as a second semantic source. Cross-repo procedure is defined in
`DEP-55b72d67f4` (см. локальный реестр зависимостей).

Product Host also consumes an exact `@odelix/harness-runtime` package and
`RunRequest/HarnessEvent/TerminalOutcome` schemas from
`odelix-harness`. Product supplies tool bindings generated from Product
contracts; it never copies Skills, agent rules or Pi internals.

Raw tool results and managed Skill results have distinct typed classifications.
Product cannot mint `ODELIX_PROCEDURE_VERIFIED`; it only transports a Harness-signed
verification artifact after policy/manifest validation.

Before the relevant freeze gate, `DRAFT` consumers pin exact schema/artifact
digest and run conformance fixtures; semver compatibility ranges are deferred.


### Options и research integration (PROPOSED)

Consume typed objective/candidate/scenario results. Analytical selection is not SignedStrategyOrder. Product usage settlement, trading fee settlement and expiry fixing are distinct contracts.

Detailed scope: [OPTIONS-WORKFLOW.md](OPTIONS-WORKFLOW.md).


### Research/strategy capability requirements


Общие StrategySpec/ExperimentSpec — Product; calculation/data manifests — Market; agent handoff — Harness; order/policy — Execution. Schema producer owns source, Stack registry discovers it. DRAFT entries не считаются published; copy/paste бизнес-типов между repos запрещён.


<a id="modules"></a>
## Модули


| ID | Module | Owns | Gate |
|---|---|---|---|
| PLT-CK | contracts-kernel | neutral IDs/refs/revisions/envelopes | FOUNDATION_GUARDS |
| PLT-TA | tenant-access | tenant/actor isolation decisions | FOUNDATION_GUARDS |
| PLT-CP | capability-policy | deny-by-default capability decisions | FOUNDATION_GUARDS |
| PLT-CB | credential-broker | BYOK/provider secret refs/revocation/redaction | FOUNDATION_GUARDS / current scope |
| PLT-DG | data-governance | rights/retention/export/operator wall | CURRENT_PLAN |
| PLT-IG | invariant-governance | ICR/ControlPart/release evidence | FOUNDATION_GUARDS |
| PLT-BIL | usage-billing | reservation, idempotent usage/settlement, hard caps, billing outbox | CONNECT_COMMERCIAL |
| PLT-RUN | run-control | durable intake, placement, fenced lease/checkpoint refs, worker health; no orchestration | CONNECT_COMMERCIAL |
| INV-WSP | workspace | server-owned workspace identity/bindings | CURRENT_PLAN |
| INV-SCN | scene | selection/context refs and object placement meaning | CURRENT_PLAN |
| INV-LNS | lens | versioned depth/view policy | CURRENT_PLAN |
| INV-CTX | context-compiler | PIT Research/Context Package | CURRENT_PLAN |
| INV-EVD | evidence-claims | Evidence/Claim/taxonomy/materiality | CURRENT_PLAN |
| INV-THS | thesis | Thesis/Scenario lifecycle | CURRENT_PLAN |
| INV-WAT | watch | visible predicates/freshness/PAUSED/trigger | WATCH_READY subset |
| INV-CHG | changeset | preview/apply/edit/reject/undo audit | CURRENT_PLAN |
| INV-MEM | personal-memory | inspectable/exportable/deletable memory | PRO_WORKSTATION minimal |
| INV-RPL | replay | episodes/simulated clock/query gateway | CURRENT_PLAN |
| CAP-* | Personal Capital OS | portfolio/sleeves/ledger/allocation/attribution | LATER |

Agent runtime modules are not Product modules. `HAR-RUN/CTX/SKL/CAP/MDL/CRT/
TRC/EVL/CONNECT/MEM/MSN` live inside the maintained Pi fork
`odelix-harness`. Product publishes typed tools and owns domain mutation.

MODULE specifications may describe planned boundaries; implementation status must remain explicit until code and checks exist.


### Актуальные подробные спецификации

| ID | Gate | MODULE |
|---|---|---|
| RES-SCOPE | RES-1 | [RES-SCOPE](../modules/research-scope/MODULE.md) |
| RES-FTR | RES-1 | [RES-FTR](../modules/feature-registry/MODULE.md) |
| RES-HYP | RES-1 | [RES-HYP](../modules/hypothesis/MODULE.md) |
| RES-EXP | RES-1 | [RES-EXP](../modules/experiments/MODULE.md) |
| RES-LED | RES-1 | [RES-LED](../modules/experiment-ledger/MODULE.md) |
| RES-VAL | RES-2 | [RES-VAL](../modules/statistical-validator/MODULE.md) |
| RES-STR | RES-1/2 | [RES-STR](../modules/strategy-registry/MODULE.md) |
| PLT-CK | R0/G0 | [PLT-CK](../modules/contracts-kernel/MODULE.md) |


<a id="testing"></a>
## Проверка


- domain transition/property tests per module;
- PostgreSQL repository/migration tests with module ownership;
- API/MCP parity and policy adversarial tests;
- tenant/operator-wall/BYOK canary and redaction tests;
- idempotency/concurrency/expected-revision tests;
- Context point-in-time and unsupported-claim fixtures;
- Product tool-contract conformance against pinned Pi Harness fixtures;
- `AGT-016` forbidden agentic routing in Product/control-plane schemas;
- `AGT-017` fenced lease, crash/retry, cross-tenant checkpoint and no-double-charge tests;
- OAuth/scope/entitlement parity and usage reservation→settlement reconciliation;
- result-laundering test: Product cannot mint `ODELIX_PROCEDURE_VERIFIED`;
- draft-only mutation and exact Preview/Apply/Edit/Reject boundary;
- contract fixtures against pinned Market/Product client versions.


### Options и research integration (PROPOSED)

Test budget-versus-total-loss ambiguity, tenant isolation, stale-quote reapproval, research-versus-execution segregation, UI/MCP parity and no authorization from subscriptions or verification labels.

Detailed scope: [OPTIONS-WORKFLOW.md](OPTIONS-WORKFLOW.md).


### Research/strategy capability requirements


Meaningful acceptance scenarios: all trials including failures, frozen protocol, duplicate scheduling, holdout leakage, version promotion and consent. Здесь перечислены planned tests; executed results должны ссылаться на commit/CI.


<a id="runbook"></a>
## Выпуск и эксплуатация


Cover Product Host/PostgreSQL availability, migrations, backups/restore, secret
broker health, model-provider degradation, budget/COGS alerts, policy-denial and
operator-wall audit streams. Market degradation must surface as typed Product
state; do not invent or cache past its contract.

CONNECT_COMMERCIAL operations also cover Run intake/queue depth, lease expiry/fencing,
checkpoint age, usage reservation/settlement reconciliation, entitlement cache
age and OAuth incidents. Queue outage never produces a false accepted Run;
billing outage never widens authorization.


### Options и research integration (PROPOSED)

Product outage does not rewrite chain state. Run cancel affects AI work, not a finalized trade. Show finality/uncertainty from Execution; no inferred success from timeout or billing event.

Detailed scope: [OPTIONS-WORKFLOW.md](OPTIONS-WORKFLOW.md).


### Research/strategy capability requirements


Начинать с inventory существующего кода/Issues. Глобальный порядок и A0 находятся в `DEP-9359295523` (см. локальный реестр зависимостей); Project metadata включает нормализованный Module согласно r15.3; дополнительные поля не добавляются автоматически. Не добавлять ручной второй статус-журнал.


---
<a id="fastify-placement"></a>
## Fastify: точное место и границы


**Целевой owner:** `odelix-product`. **Composition host:** `apps/product-host/`.

Это уже принятое в r13 архитектурное размещение, не новый восьмой repository. Доступ к реальному Product repo в этой работе не подтвердился; ниже — план файлов, а не утверждение об их существовании.

```text
odelix-product/
  apps/product-host/src/
    app.ts                  # buildApp; создание Fastify и composition
    server.ts               # listen, signals, shutdown
    config.ts               # валидируемая конфигурация, без секретов в logs
    plugins/                # auth, harness, connect transport wiring
    routes/                 # HTTP controllers: run/object/research/billing
  packages/platform/        # tenant, capabilities, rights, runs, billing, notifications
  packages/investigation/   # workspace, context, evidence, thesis, watch, changes, memory
  packages/research/        # features, hypotheses, experiments, trial ledger, validation
  packages/capital/         # не активируется до собственной задачи Capital
  packages/adapters/
    postgres/               # persistence ports
    identity-auth0/         # managed IdP -> our identity contract
    market-client/          # generated Market client -> query port
    harness-store/          # Product storage/leases <-> Pi checkpoint protocol
    harness-tools/          # allowlisted application operations -> Pi tools
    billing-stripe/         # Stripe -> our subscription projection/outbox
    research-compute/      # Product job lifecycle -> existing Market numerical jobs
    artifact-store/        # manifests/results, tenant-bound download/export
    execution-client/      # PAPER owner events -> Product projections
    watch-predicate/       # chosen CEL implementation; no I/O
    notification-webhook/  # reviewed delivery channel, bounded endpoints
```

### Путь запроса

```text
Web / Desktop / внешний агент
 -> Fastify product-host
 -> schema + identity + tenant + capability + data-right + budget checks
 -> Product application use case
       -> MarketClient -> Rust facts / deterministic compute
       -> exact @odelix/harness-runtime -> тот же Pi loop
            -> разрешённые Product/Market tools
 -> durable result / evidence / usage
 -> HTTP / MCP / stream / Web projection
```

**Fastify не сам исследовательский AI.** Product решает, принят ли Run, где он исполняется и что хранить. Pi выбирает разрешённые аналитические шаги. Market рассчитывает рыночные числа. Execution владеет фактическим PAPER state. UI не дублирует этих владельцев.

В S0 Pi package исполняется в Product host. Выделение worker/process позже требует измеримой необходимости/изоляции, но не создаёт второй harness. Пользовательский код всегда изолируется отдельно с момента появления такого capability.

Высокочастотная графика не заставляет Fastify пересылать каждый tick через свою бизнес-логику. PRD-025 выдаёт короткоживущий подписанный subscription grant, MKT-017 проверяет его в gateway, WKS-007 принимает ограниченный поток. Coarse queries и все mutations остаются на соответствующих authorized ports.

### Конкретные карточки Fastify / Product

PRD-001 — Fastify host и tests; PRD-002 — Postgres adapter; PRD-003 — IdP/jose; PRD-004 — capabilities и durable Run intake; PRD-005 — MarketClient; PRD-006 — Context/Evidence; PRD-007 — Pi package/store binding; PRD-008 — MCP mount; PRD-009–011 — Workspace/Thesis/ChangeSet; PRD-012–014 — billing/usage/recovery; PRD-015–019 — Research; PRD-020–021 — paper deployment/projections; PRD-022–024 — durable Watch/delivery; PRD-025–026 — stream grant и пользовательская память.

**Где пишем платформу:** Fastify — транспорт/composition; собственная бизнес-платформа — `packages/platform`, `packages/investigation`, `packages/research` и позднее `packages/capital` в том же Product repo. Это не готовая внешняя платформа, которую можно целиком скопировать из TradingAgents/OpenBB.



<a id="early-external-data"></a>
<a id="ранняя-data-ветка-r158-реализация-и-владельцы"></a>
## Ранняя data-ветка: реализация и владельцы

PRD-031 реализует pure source/data-rights policy и DataBinding extension: display/store/export/AI/redistribute независимы, UNKNOWN=deny соответствующего действия. Локальная policy проверяется fixtures до полного OIDC/Fastify; hosted доступ требует actual PRD-004/025 bindings и source agreements.

Workspace/Selection/Evidence хранят per-layer source/profile/receipt/revision и time quality. Market хранит response artifacts; Product — ссылки и права, не второй OHLC store. Credential scopes отделены; secrets не попадают в browser/trace. [Lifecycle оценок](SEMANTIC-ASSESSMENT-LIFECYCLE.md#external-observations) сохраняет as-shown данные без рефетча.

Биллинг и временное личное размещение GitHub не являются prerequisite local preview. Ни одна local fixture не открывает публичный data endpoint.
<!-- R159 agent-product-ownership -->
## Agent Registry, DecisionRecord и approvals

Product owns AgentSpec/Manifest Registry (PRD-033), bitemporal projection (PRD-032), generic ApprovalPort/PAPER lifecycle (PRD-035) и условный deployment lifecycle (PRD-036). [AGENT-LIFECYCLE](AGENT-LIFECYCLE.md) — единственная подробная спецификация пяти осей и переходов. Fastify связывает ports, не считает fills/risk и не реализует второй Pi loop.

PRD-022/023/024 — generic durable Watch/outbox без обязательного Thesis/Semantic. Специфическая связь — PRD-037. Research/CAP binding PRD-020/021 использует общую реализацию, но требует StrategyVersion и своего gate. Credentials/rights принадлежат Product policy; источники Deribit/FRED выбраны, но каждый dataset use проверяется отдельно, включая derived text и cache hit.

<a id="personal-data-firewall"></a>
## Rights namespace и resolved permissions

Проверять grants до чтения, до model call и при cache hit. PERSONAL_USE не даёт Product прав на один и тот же subject автоматически. Schema AgentSpec проверяет заявленные оси; authorization проверяет фактические маршруты, частные Skills, срок и аудиторию. GRANTS catalog экспортируется в локальный CONTEXT как неизменяемая выборка; реальные подписи не выдаются генератором.

<!-- BEGIN GENERATED REPO CONTEXT -->

<a id="modules-index"></a>
## Каталог модулей этой области

Один primary owner в каждой задаче; affected modules отдельно. AREA — служебная классификация. Идентификаторы сохранены. Activation — scope текущего плана, не доказательство исполнения; независимый implementation_status в JSON.

| ID | Вид | Activation | Источник в этом repo |
|---|---|---|---|
| `AREA-PRD-HOST` | DELIVERY_AREA | ACTIVE | [docs/DEVELOPMENT.md](#architecture) |
| `AREA-PRD-INTEGRATIONS` | DELIVERY_AREA | ACTIVE | [docs/DEVELOPMENT.md](#architecture) |
| `AREA-PRD-NOTIFICATIONS` | DELIVERY_AREA | ACTIVE | [docs/DEVELOPMENT.md](#architecture) |
| `AREA-PRD-PERSISTENCE` | DELIVERY_AREA | ACTIVE | [docs/DEVELOPMENT.md](#architecture) |
| `CAP-APP` | MODULE | ACTIVE | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-ATTR` | MODULE | STRATEGIC_BRANCH | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-EXE` | MODULE | ACTIVE | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-LED` | MODULE | STRATEGIC_BRANCH | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-PORT` | MODULE | STRATEGIC_BRANCH | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-REC` | MODULE | ACTIVE | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-RISK` | MODULE | STRATEGIC_BRANCH | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `CAP-SAFE` | MODULE | STRATEGIC_BRANCH | [docs/CAPITAL.md](CAPITAL.md#capital-scope) |
| `INV-CHG` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-CTX` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-EVD` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-LNS` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-MEM` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-RPL` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-SCN` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-THS` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-WAT` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `INV-WSP` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-BIL` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-CB` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-CK` | MODULE | ACTIVE | [modules/contracts-kernel/MODULE.md](../modules/contracts-kernel/MODULE.md) |
| `PLT-CP` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-DG` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-IG` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-RUN` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-TA` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `RES-EXP` | MODULE | ACTIVE | [modules/experiments/MODULE.md](../modules/experiments/MODULE.md) |
| `RES-FTR` | MODULE | ACTIVE | [modules/feature-registry/MODULE.md](../modules/feature-registry/MODULE.md) |
| `RES-HYP` | MODULE | ACTIVE | [modules/hypothesis/MODULE.md](../modules/hypothesis/MODULE.md) |
| `RES-LED` | MODULE | ACTIVE | [modules/experiment-ledger/MODULE.md](../modules/experiment-ledger/MODULE.md) |
| `RES-SCOPE` | MODULE | ACTIVE | [modules/research-scope/MODULE.md](../modules/research-scope/MODULE.md) |
| `RES-STR` | MODULE | ACTIVE | [modules/strategy-registry/MODULE.md](../modules/strategy-registry/MODULE.md) |
| `RES-VAL` | MODULE | ACTIVE | [modules/statistical-validator/MODULE.md](../modules/statistical-validator/MODULE.md) |
| `PLT-AGR` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `PLT-DEP` | MODULE | STRATEGIC_BRANCH | [docs/DEVELOPMENT.md](#modules) |

<a id="work-queue"></a>
## Локальная очередь и следующий шаг

Полный текст каждой задачи — в [delivery/CONTEXT.json](../delivery/CONTEXT.json), это автоматически полученная выборка, не второй редактируемый backlog. Найдите объект по `id`, прочитайте `work`, `acceptance`, `paths`, `depends_on`, `primary_module_id`.

Сначала действующий Issue, actual commit/permissions, затем следующее допустимое действие. Planned wave не статус и не мандат. Внешняя зависимость должна предоставить артефакт/fixture; соседний checkout не предполагается.

| ID | Модуль | Волна | Результат | Зависимости |
|---|---|---|---|---|
| <a id="issue-ODX-PRD-001"></a>`ODX-PRD-001` | `AREA-PRD-HOST` | 1 | Создать Fastify product-host и модульный TypeScript workspace | ODX-STK-008 |
| <a id="issue-ODX-PRD-002"></a>`ODX-PRD-002` | `AREA-PRD-PERSISTENCE` | 1 | Подключить PostgreSQL и module-owned persistence | ODX-PRD-001 |
| <a id="issue-ODX-PRD-003"></a>`ODX-PRD-003` | `PLT-TA` | 1 | Реализовать IdentityPort и managed-OIDC/JWKS адаптер | ODX-PRD-001, ODX-PRD-002 |
| <a id="issue-ODX-PRD-004"></a>`ODX-PRD-004` | `PLT-CP` | 1 | Реализовать capabilities, data-rights и durable Run intake | ODX-PRD-003 |
| <a id="issue-ODX-PRD-005"></a>`ODX-PRD-005` | `AREA-PRD-INTEGRATIONS` | 2 | Написать MarketClient adapter и сервисные границы | ODX-MKT-005, ODX-MKT-006, ODX-PRD-004 |
| <a id="issue-ODX-PRD-006"></a>`ODX-PRD-006` | `INV-CTX` | 2 | Реализовать Context Compiler и immutable Evidence package | ODX-PRD-005 |
| <a id="issue-ODX-PRD-007"></a>`ODX-PRD-007` | `PLT-RUN` | 2 | Встроить private Harness package в Fastify и связать persistence | ODX-PRD-006, ODX-HAR-003 |
| <a id="issue-ODX-PRD-008"></a>`ODX-PRD-008` | `PLT-CP` | 6 | Смонтировать /mcp с OAuth, tenant isolation и общими policies | ODX-PRD-007, ODX-HAR-004 |
| <a id="issue-ODX-PRD-009"></a>`ODX-PRD-009` | `INV-WSP` | 3 | Реализовать Workspace и EvidenceGraph objects | ODX-PRD-006 |
| <a id="issue-ODX-PRD-010"></a>`ODX-PRD-010` | `INV-THS` | 3 | Реализовать Thesis lifecycle и draft WatchSpec | ODX-PRD-009 |
| <a id="issue-ODX-PRD-011"></a>`ODX-PRD-011` | `INV-CHG` | 3 | Написать ChangeSet Preview/Apply/Reject/Undo команды | ODX-PRD-010 |
| <a id="issue-ODX-PRD-012"></a>`ODX-PRD-012` | `PLT-BIL` | 6 | Подключить Stripe Billing через отдельный provider adapter | ODX-PRD-004 |
| <a id="issue-ODX-PRD-013"></a>`ODX-PRD-013` | `PLT-BIL` | 6 | Завершить usage reservation/settlement и коммерческие budgets | ODX-PRD-012, ODX-PRD-007 |
| <a id="issue-ODX-PRD-014"></a>`ODX-PRD-014` | `PLT-RUN` | 6 | Проверить durable recovery, fencing и transport reconnect | ODX-PRD-013 |
| <a id="issue-ODX-PRD-015"></a>`ODX-PRD-015` | `RES-SCOPE` | 4 | Зафиксировать Feature/Target/Experiment/Strategy contracts и fixtures | ODX-PRD-011, ODX-MKT-006 |
| <a id="issue-ODX-PRD-016"></a>`ODX-PRD-016` | `RES-FTR` | 4 | Написать Feature Registry и Hypothesis lifecycle | ODX-PRD-015 |
| <a id="issue-ODX-PRD-017"></a>`ODX-PRD-017` | `RES-EXP` | 4 | Реализовать Experiment/Trial ledger и асинхронные jobs | ODX-PRD-016 |
| <a id="issue-ODX-PRD-018"></a>`ODX-PRD-018` | `RES-EXP` | 4 | Подключить ResearchComputePort и ArtifactStore adapters | ODX-PRD-017, ODX-MKT-008, ODX-STK-010 |
| <a id="issue-ODX-PRD-019"></a>`ODX-PRD-019` | `RES-STR` | 5 | Сохранить ValidationReport и immutable StrategyVersion | ODX-PRD-018, ODX-MKT-014 |
| <a id="issue-ODX-PRD-020"></a>`ODX-PRD-020` | `RES-STR` | 5 | Реализовать PAPER deployment lifecycle и approvals | ODX-PRD-019, ODX-EXE-001, ODX-PRD-033, ODX-PRD-035 |
| <a id="issue-ODX-PRD-021"></a>`ODX-PRD-021` | `CAP-EXE` | 5 | Написать ExecutionClient и read-only paper projections | ODX-PRD-020, ODX-EXE-003 |
| <a id="issue-ODX-PRD-022"></a>`ODX-PRD-022` | `INV-WAT` | 2 | Реализовать durable Watch/Mission objects и predicate engine adapter | ODX-PRD-004, ODX-MKT-006 |
| <a id="issue-ODX-PRD-023"></a>`ODX-PRD-023` | `INV-WAT` | 2 | Перенести quiet-hours/severity helpers в tenant notification policy | ODX-PRD-022 |
| <a id="issue-ODX-PRD-024"></a>`ODX-PRD-024` | `AREA-PRD-NOTIFICATIONS` | 2 | Написать durable notification outbox и первый delivery adapter | ODX-PRD-023 |
| <a id="issue-ODX-PRD-025"></a>`ODX-PRD-025` | `PLT-CP` | 1 | Выдать short-lived MarketSubscriptionGrant без proxy всех ticks | ODX-PRD-004, ODX-MKT-005 |
| <a id="issue-ODX-PRD-026"></a>`ODX-PRD-026` | `INV-MEM` | 8 | Реализовать consent-governed Memory objects и as-of retrieval | ODX-PRD-028, ODX-PRD-019 |
| <a id="issue-ODX-PRD-090"></a>`ODX-PRD-090` | `RES-EXP` | OPT | Оценить AlphaModel adapter для пользовательской модели | ODX-PRD-019, ODX-STK-010 |
| <a id="issue-ODX-PRD-027"></a>`ODX-PRD-027` | `INV-SCN` | 1 | Опубликовать минимальные Scene/Selection/context contracts для раннего WKS | ODX-STK-001 |
| <a id="issue-ODX-PRD-028"></a>`ODX-PRD-028` | `INV-EVD` | 2 | Хранить SemanticAssessment revisions и as-shown history | ODX-PRD-004, ODX-HAR-014 |
| <a id="issue-ODX-PRD-029"></a>`ODX-PRD-029` | `PLT-RUN` | 2 | Подключить быстрый semantic path к Fastify с admission/cache/budgets | ODX-PRD-028, ODX-HAR-014, ODX-PRD-007 |
| <a id="issue-ODX-PRD-030"></a>`ODX-PRD-030` | `RES-VAL` | 5 | Ввести отдельную приёмку ForecastSpec/CalibrationReport | ODX-PRD-019, ODX-MKT-014 |
| <a id="issue-ODX-PRD-031"></a>`ODX-PRD-031` | `PLT-DG` | 0 | Определить external-data rights и интегрировать разрешения в Product | ODX-MKT-090 |
| <a id="issue-ODX-PRD-032"></a>`ODX-PRD-032` | `INV-EVD` | 2 | Сохранить битемпоральную DecisionRecord-проекцию независимо от семантики | ODX-EXE-007, ODX-PRD-004 |
| <a id="issue-ODX-PRD-033"></a>`ODX-PRD-033` | `PLT-AGR` | 3 | Реализовать AgentSpec/Manifest Registry по независимым осям | ODX-PRD-004, ODX-PRD-032, ODX-EXE-009 |
| <a id="issue-ODX-PRD-035"></a>`ODX-PRD-035` | `INV-CHG` | 3 | Реализовать approval inbox и generic PAPER lifecycle | ODX-PRD-033, ODX-EXE-013, ODX-PRD-024 |
| <a id="issue-ODX-PRD-037"></a>`ODX-PRD-037` | `INV-WAT` | 3 | Подключить Thesis и semantic triggers к общему Watch lifecycle | ODX-PRD-022, ODX-PRD-010, ODX-PRD-029, ODX-MKT-019 |
| <a id="issue-ODX-PRD-034"></a>`ODX-PRD-034` | `RES-STR` | 5 | Подключить ModelSignalPort для research/PAPER sandbox | ODX-PRD-015, ODX-STK-010, ODX-EXE-009 |
| <a id="issue-ODX-PRD-036"></a>`ODX-PRD-036` | `PLT-DEP` | 5 | Реализовать portable PAPER deployment lifecycle и product CLI plan | ODX-PRD-033, ODX-EXE-009, ODX-STK-023 |

<a id="dependencies"></a>
## Внешние зависимости и источники

**Реестр ниже не является очередью обязательного чтения.** Большинство записей — происхождение решений; артефакт запрашивается только для конкретной необходимой зависимости задачи. **Независимость checkout не означает отсутствие зависимостей продукта.** Здесь записаны логические координаты издателя, а не относительные переходы в соседнюю папку. `source_sha256` в локальном JSON удостоверяет только исходный документ r15.3. Ни одна строка не является доказательством выпуска schema/SDK.

Для конкретной задачи получить через TaskPacket или разрешённый артефактный канал: producer + contract ID, точный release/commit, schema/package digest, fixtures, consumer-conformance и разрешённые операции. Записать фактический локальный путь после получения; пока artifact не предоставлен, зависимая runtime-работа не готова. Локальные fixture/design-задачи возможны по своему мандату.

Не заменять чужую схему её ручной копией. Обновление версии — producer release → consumer pin → conformance → интеграция. Разрешение читать артефакт не даёт права изменять другой repo.

| Ref | Publisher | Логический источник | Статус исходника |
|---|---|---|---|
| <a id="DEP-13674f89c9"></a>`DEP-13674f89c9` | `odelix-market` | `docs/PHASE-A-REQUIREMENTS.md#scalable-read-programme` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-143d61eff7"></a>`DEP-143d61eff7` | `odelix-stack` | `reference/OSS-REVIEW-RU.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-52ea262d35"></a>`DEP-52ea262d35` | `odelix-stack` | `docs/INTEGRATIONS.md#code-reuse` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-55b72d67f4"></a>`DEP-55b72d67f4` | `odelix-stack` | `docs/05-CONTRACTS-AND-INTEGRATION.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-568bec8dbe"></a>`DEP-568bec8dbe` | `odelix-workstation` | `docs/WORKSTATION-SPEC.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-686a4a4cec"></a>`DEP-686a4a4cec` | `odelix-market` | `docs/PHASE-A-REQUIREMENTS.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-9359295523"></a>`DEP-9359295523` | `odelix-stack` | `docs/DELIVERY-RUNBOOK.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-b335630551"></a>`DEP-b335630551` | `odelix-stack` | `AGENTS.md` | DOCUMENT_SNAPSHOT_ONLY |

`UNRESOLVED_SOURCE_REFERENCE` — отсутствующий документ/якорь исходного пакета явно зарегистрирован; содержание не придумано. Для historical source его можно оставить архивной ссылкой, для обязательной зависимости — запросить источник. Public URLs в предметных документах сохранены как датированные ссылки и не проверялись онлайн этой сборкой.
