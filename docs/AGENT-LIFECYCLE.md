# Agent lifecycle — взаимодействие, размещение, полномочия и история

**r15.11 · 28 сентября 2026 · PROPOSED.** Owner: `odelix-product`. Это спецификация, не работающий сервис. Связанные задачи: PRD-032/033/035/036/037, WKS-022/023/024, HAR-012/017/018/020. Исполняемые текущие среды ограничены read/draft, REPLAY и PAPER; будущие real environments не активированы.

<a id="axes"></a>
## 1. Пять независимых осей

<!-- BEGIN GENERATED AGENT AXES -->
| Ось | Значения и смысл |
|---|---|
| interaction | INTERACTIVE, SCHEDULED, EVENT. Причина запуска, не authority |
| placement | HOSTED, LOCAL, EMBEDDED. HYBRID — композиция отдельных исполнителей, не enum |
| environment | READ_ONLY, REPLAY, PAPER. Среда, не разрешение на действие |
| authority | READ, DRAFT, SIMULATE. USER_MUTATION — отдельная owner command после подтверждения |
| brain | DETERMINISTIC, MODEL, MANAGED, BYOK, EXTERNAL. Источник решения, не риск-авторитет |
<!-- END GENERATED AGENT AXES -->

L0 Observe / L1 Explain / L2 Assist / L3 Watch / L4 Suggest / L5 PAPER — производные UX-профили. L6 обозначает будущую отдельно рассматриваемую возможность, не runtime capability этого выпуска. Схема проверяет заявленные сочетания и egress_mode/brain_execution/private_skill_refs; policy повторно проверяет resolved endpoints, секреты, права и реальные потоки. Схема не доказывает отсутствие сетевой утечки. USER_MUTATION — отдельная подтверждаемая команда владельца объекта, не полномочие MANAGED brain.

MANAGED допускается к чтению, черновикам и разрешённой симуляции. BYOK означает источник модельного ключа, а не обязательный Node. Требование не выпускать данные наружу исключает hosted inference, если именно эти данные ему понадобятся. Измеренное latency-требование и local-data policy могут обосновать размещение, но не освобождают от прав на данные.

## 2. Объекты Product и границы

`AgentSpec` — декларативная конфигурация цели, inputs, brain, interaction, policy refs, approvals, evidence, retention и размещения. Поля цели/комментария могут быть текстовыми; исполняемая политика не выводится из свободного текста без отдельного typed draft и проверки.

`AgentManifest` — неизменяемая разрешённая компиляция Spec с digest схем, runtime, source definitions, policy и dataset bindings. Изменение значимого поля создаёт новый manifest, а не меняет старый.

`AgentRegistration` — владелец, разрешённые tenants, текущий manifest и lifecycle. `DeploymentRecord` — желаемое/наблюдаемое размещение и ревизии. `ApprovalRequest` — точный target digest, субъект, authority, срок, одноразовый идентификатор и outcome. `DecisionRecord` — Product-проекция неизменяемых producer events; это не второй OMS.

Product не вычисляет цены, fills или риск, не планирует шаги Pi и не провизионирует VPS через инженерный harness. Execution предоставляет policy/simulation ports; Market — данные; Harness — интеллектуальную процедуру; Stack — release/infra tooling.

## 3. Две ранние линии и полная Research-интеграция

| Линия | Приёмка | Не требуется |
|---|---|---|
| Импортированная история решений | FLIGHT_RECORDER_READY, STK-020 | Jev, private Skills, Research, Node, broker keys |
| Базовая hosted Mission | HOSTED_MISSION_READY, STK-022 | Полный options/research Critic, Node и PAPER |
| Общая hosted-симуляция | HOSTED_PAPER_READY, STK-021 | Проверенная StrategyVersion, полный MKT-013 |
| Полная Research → PAPER | PAPER_READY/STK-107 и PAPER_RESILIENT/STK-108 | Не закрывается первым тестовым симулятором |

Generic PAPER использует конечный `SimulationScenarioRef`, подготовленные разрешённые данные, одну реализацию OMS и узкий Market SimulationPort. Результат не переименовывается в «проверенную стратегию». Research binding сохраняет ValidationReport/StrategyVersion и поздние численные prerequisites.

## 4. Lifecycle и approvals

DRAFT → VALIDATED → REGISTERED → READY → RUNNING/SLEEPING → PAUSED/COMPLETED/FAILED/REVOKED — проекция событий, а не независимая копия state machine Executor. Факт принятия задачи, её выполнение и наблюдаемый результат различаются.

Пользователь видит цель, входы, размещение, egress, бюджет, разрешённые действия и срок мандата. **Proposal/Plan никогда не является authorization.** Plan не выполняет ни операции, ни сетевую активацию. Подтверждение относится к точному manifest/intent digest. Новый digest или устаревший state требует новой проверки. Истечение TTL — EXPIRED, не default-approve. Повторная доставка не применяется дважды; reconnect не восстанавливает pending как одобренный.

Первые поверхности подтверждения — аутентифицированные Workstation и mobile web через Product. Webhook/email/messenger отправляют уведомление/ссылку, но не являются разрешением по наличию сообщения. Все права перепроверяются при применении, включая owner/tenant, revision, validity и usage budget.

## 5. Durable Watch/Mission без обязательного LLM

Product владеет сохранённым Watch, расписанием/предикатом, outbox и желаемым состоянием. Базовый Watch не требует Thesis/Semantic: PRD-022/023/024. Специфические привязки вводит PRD-037.

На trigger создаётся идентифицированная итерация. Harness использует fenced lease и один bounded Run; повтор trigger/доставки не создаёт второго root Run. Время, токены, tools и стоимость учитываются на root со всеми детьми/retries. SLEEPING не вызывает модель. Stale input — PAUSED/INSUFFICIENT_DATA с причиной; запрет права — POLICY_BLOCKED, не отсутствие рынка. Research review — HAR-020, а не prerequisite простой scheduled observation.

## 6. DecisionRecord: время и полнота

Исходное producer событие не перезаписывается. Outcome, correction и reflection добавляются как новые события с parent/supersedes refs. Поля producer schema: `event_at` (nullable, с `event_at_precision`), `recorded_at`, `known_at`, `outcome_known_at`, `assessment_revision`. Порядок: `(producer_id, stream_id, stream_epoch, sequence)`; sequence — неотрицательное decimal-string, а не JavaScript float. Импортный sequence описывает приём, не выдумывает исходный биржевой порядок. Старое `event_at` у позднего импорта не даёт системе знания в прошлом.

Два запроса: «какую source vintage мы можем восстановить» и «что Odelix действительно знала тогда». Они используют разные ограничения. Memory retrieval применяет consent/rights, knowledge time и доступность outcome; reflection не заменяет outcome. `critic_status=NOT_RUN` требует `critic_result_ref=null`; любой выполненный verdict имеет ссылку на реальный результат, включая ERROR. Отсутствие Critic не означает PASS. Записываются отказ/abstention/недостающие inputs, не только удачные действия.

Уровни полноты: IMPORTED, RECONSTRUCTABLE, VERIFIED_AS_SHOWN с явной областью доказательства. Центральный record с hash-only refs не становится reconstructable. Полный локальный record и ограниченный egress-envelope — разные представления одного источника; raw/derived/narrative права проверяются до передачи и на cache hit.

## 7. Personal collection и разрешённые inputs агента

Deribit — полный публичный universe с поэтапными профилями PERSONAL_USE; FRED/ALFRED — ограниченный перечень личных файлов/рядов/vintages, не mirror или API-store. PERSONAL_USE не разрешает чтение личного корпуса продуктовым кодом, AI или показ другим. Возможность скачать личную копию FRED не равна разрешению хранить её в Odelix-корпусе: STORE требует отдельного подтверждения применимых прав.

Registry хранит ссылки на dataset coverage/version/rights, не копию собственного каталога Market. Недостающий vintage или historic L2 нельзя заменить latest data. OpenBB preview не является полной историей FRED или native orderflow. Отзыв прав блокирует новые операции и инициирует применимую retention-процедуру без подделки прежних событий.

## 8. Agent Validation Profile

Agent Validation Profile — проекция evidence о конкретных software/model/policy/dataset versions, покрытии событий, сценариях отказа и числе решений. Это не гарантия прибыльности и не автоматическое продвижение полномочий. Нулевая активность не проходит проверку за счёт дней uptime. Неизвестный outcome не считается безопасным успехом.

Повторная оценка требуется при значимом изменении версии, источника, режима, модели или политики. Primary admission не требует уже совершённого реального действия. Стратегические real-environment гейты остаются вне текущей реализации.

## 9. Node и продуктовый deploy lifecycle

Portable PAPER Node — условный способ запуска тех же interfaces, не отдельный продуктовый движок. В плане существует заранее; начать существенную реализацию позволяет `NODE_PAPER_DEMAND`.

Предложенный, ещё не подписанный порог: 5 уникальных операторов за 60 дней после hosted PAPER, минимум 2 пилотных участника, подтверждённая причина (local/no-egress/offline/измеренная задержка). Review каждые 14 дней в STK-023; Founder может принять отдельный малый spike. Порог и результат проверки — разные документы/состояния, сборка не объявляет спрос подтверждённым.

Product владеет `odelix` CLI plan/validate, DeploymentRecord, approval и регистрацией. Stack — переносимым шаблоном, CI и подписанными package manifests. Инженерный `./workshop` разрабатывает software и не исполняет пользовательские Missions. Браузер не запускает произвольную локальную команду. Несколько облачных провайдеров не обязательны для первой поставки portable package.

## 10. Текстовые схемы потоков

ONLINE: Selection/DatasetBinding → Product context/rights → bounded Harness Run → draft/evidence → Product preview → user apply.

HOSTED 24/7: durable trigger → lease/budget → observation or typed proposal → DecisionEvents → policy-filtered Product projection → outbox → user returns.

PAPER: approved immutable simulation mandate → durable intent record → existing Execution simulation → outcome events → Product projection. Состояние пишется до моделируемого эффекта.

NODE: Product plan/approval → portable package → same simulation interfaces → local full events → allowed egress records. Нет private hosted Skills в клиентском образе; отказ хоста не приравнивается к подтверждённой остановке.

## 11. Приёмка

Schema fixtures, independently expected event timeline, duplicate/late import, lease fencing, TTL expiry/revoke, wrong tenant, forbidden egress, zero-data/missing-vintage, policy budget, crash points, unavailable node and reconstruction coverage. Contract test != runtime, legal approval или нагрузочная приёмка.

Локальные схемы: [AgentSpec](../contracts/agent.spec.v1.schema.json), [AgentManifest](../contracts/agent.manifest.v1.schema.json), [NodeHeartbeat](../contracts/node.heartbeat.v1.schema.json), [ModelSignal](../contracts/model.signal.v1.schema.json). Полный план — в локальной [выборке задач](../delivery/CONTEXT.json).

<a id="axis-validation"></a>
## Проверка сочетаний и граница MANAGED

MANAGED допустим только для READ / DRAFT / SIMULATE и только в нынешних READ_ONLY / REPLAY / PAPER. Любое расширение на реальные среды требует письменного юридического заключения и отдельного ADR через perimeter review STK-104; ни spec, ни профиль проверок не заменяют этот допуск.

NO_EGRESS требует LOCAL/EMBEDDED + LOCAL_PROCESS и исключает MANAGED/EXTERNAL; локальный MODEL/BYOK допустим только если resolved model/provider действительно локален. LOCAL/EMBEDDED запрещают private_skill_refs. Hosted private Skills не доставляются в Node. `HYBRID` — граф явно описанных компонентов, не лазейка в egress.

DecisionEvent — исходное неизменяемое событие `decision.event.v1`; DecisionRecord — Product-проекция. Старое draft-имя decision.record.v1 заменено до публикации; реализованные внешние клиенты всё равно требуют обычной version/compatibility процедуры. Не превращать согласование документа в утверждение опубликованной схемы.

## Проверяемые сочетания полей

<!-- BEGIN GENERATED AXIS CONSTRAINTS -->
- `NO_EGRESS` ⇒ `LOCAL`/`EMBEDDED`, `brain_execution=LOCAL_PROCESS`, brain не `MANAGED`/`EXTERNAL`.
- `LOCAL`/`EMBEDDED` ⇒ `private_skill_refs` пуст; hosted private Skills не переносятся в клиент.
- `MANAGED` ⇒ `placement=HOSTED`, `brain_execution=HOSTED_SERVICE`, `egress_mode=GRANTED_EGRESS`, authority только `READ`/`DRAFT`/`SIMULATE`.
- `SIMULATE` ⇒ environment только `REPLAY`/`PAPER`; `READ_ONLY` не является средой симуляции.
<!-- END GENERATED AXIS CONSTRAINTS -->

Schema-проверка не заменяет runtime-проверку resolved endpoints, rights и egress. GRANTED_EGRESS — заявленный профиль, не созданное сборкой разрешение.
