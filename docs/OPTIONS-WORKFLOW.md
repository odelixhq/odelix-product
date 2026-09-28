# Options workflow

> **Основа и reuse текущей редакции:** [локальная карта внешних компонентов и адаптеров](DEVELOPMENT.md#implementation-basis). Там указаны источник, режим использования, наши модули/пути, задачи и ограничения. Кандидат не установленная зависимость; этот предметный документ не требует реализации библиотечной механики с нуля.


**Уточнение scope Research: r12.1 · 2026-09-18.** Пользовательские примеры не определяют встроенную стратегию или обязательный dataset; статус реализации не меняется.

**Редакция:** 1.1.1 · **Дата:** 2026-09-28 · **Статус:** PROPOSED / implementation status requires evidence.

## 3. Полный каталог стратегий и конструктор

### 3.1 Каталог — семейства конструкций, а не закрытый список сделок

| Семейство | Основные конструкции | Что обязательно рассчитывать | Особые ограничения |
|---|---|---|---|
| Направленный рост | Long call, bull call debit spread, bull put credit spread | Премия/кредит, breakeven, payoff, Greeks, максимальный убыток при корректной модели | Credit spread требует обеспечения и защиты от разрыва ног |
| Направленное падение | Long put, bear put debit spread, bear call credit spread | Те же величины, риск и доступность выхода | Long put не страхует автоматически весь портфель |
| Защита | Protective put, put spread hedge, collar | Отдельно риск хеджа и остаточный риск базового портфеля | Collar ограничивает upside; put spread покрывает только участок падения |
| Большое движение | Long straddle, long strangle | Два breakeven, премия, theta/vega, сценарии движения и IV | Угаданное направление не гарантирует окупаемость премии |
| Диапазон | Long butterfly, condor, iron butterfly, iron condor | Экстремумы кусочного payoff, совместные fees, обеспечение коротких ног | Нет обещания стабильного дохода; проверка пакетного исполнения |
| Покрытая продажа | Covered call, cash-secured put | Риск всего пакета с underlying/cash, упущенный upside, assignment | Covered call сохраняет риск падения underlying; collateral/settlement должны соответствовать |
| Сроки и относительная IV | Calendar, diagonal, double calendar | Стоимость во времени, поверхности по срокам, ранняя экспирация, margin path | Одной payoff-кривой на общий expiry их не описать |
| Skew и относительная стоимость | Risk reversal, ratio/backspread, broken-wing butterfly | Неограниченные хвосты, ratios, collateral, режимы IV | Некоторые варианты имеют неограниченный убыток; исследование допускается, retail live по умолчанию нет |
| Динамическая волатильность | Delta-hedged options, gamma scalping, volatility carry | Путь хеджа, turnover, funding, fees, latency, inventory | Нужны частые котировки и реалистичный hedge execution; не эквивалент статическому payoff |
| Относительная стоимость между рынками | Basis/volatility spreads, cross-venue combinations | Различия индексов, валют, экспираций, расчётов, funding и доступности капитала | Межплощадочная атомарность не предполагается |
| Событийные/сигнальные | Пользовательские событийные и многофакторные правила, order-flow-triggered конструкции, event hedges | Точное правило события, задержка, выбор инструмента, вход/выход, статистика | Сигнал не подменяет цену опциона или модель исполнения |
| Собственные | Произвольная разрешённая комбинация перечисленных primitives и пользовательских правил | Семантика каждой ноги, временной логики, риска, данных и исполнения | Неподдерживаемая модель возвращает явный отказ, не приблизительное «всё проверено» |

В первом вычислительном релизе покрываются long call/put, четыре vertical spread, protective put, collar, straddle/strangle и стандартные butterfly/condor при единой поддержанной спецификации. Calendar/diagonal и динамические стратегии имеют отдельные valuation/execution модели; их карточки и требования видны в каталоге до допуска к расчёту. Это ограничение зрелости конкретного движка, а не исключение класса из продукта.

### 3.2 Независимые статусы доступности

Для каждой стратегии и версии данных система показывает `definitionSupported`, `valuationSupported`, `historicalTestEligible`, `paperEligible`, `liveEligible`, а также причины отказа. Наличие шаблона в каталоге не даёт пяти разрешений сразу.

- **SYNTHETIC:** проверяется арифметика или UX на созданных данных.
- **MODELLED:** сценарий использует модельные значения; это не наблюдавшиеся сделки.
- **HISTORICAL:** фиксированные реальные данные с раскрытой моделью заполнения.
- **PAPER:** наблюдение и симулированные действия в отдельной среде.
- **LIVE:** фактические операции на конкретном разрешённом маршруте.

Статусы не смешиваются в графике доходности. Даже реалистичный HISTORICAL-бэктест не является историей реальных fills.

### 3.3 Три входа в один конструктор

**Намерение:** пользователь описывает цель, горизонт, актив и ограничения. AI уточняет недостающие параметры и формирует draft. **Шаблон:** пользователь выбирает семейство и меняет параметры. **Профессиональный builder:** редактирует legs, selectors, triggers, sizing и exits напрямую. Все три пути компилируются в один typed `StrategySpec`.

Конструктор состоит из независимых частей: observation universe, signal/features, instrument selector, leg composition, position sizing, entry, exit/roll, execution policy, portfolio constraints. Условия — переиспользуемые predicates с units, параметрами, versioned dependency plan и требованиями данных. Entry и exit редактируются независимо; ни формула, ни порог, ни источник из примера не зашиваются в core.

Пользователь может подключить свои лицензированные данные. Для ряда нужны источник, временная семантика, units, schema, known gaps и права использования. Загруженный CSV без времени доступности не становится PIT-доказательством. Пользовательский код выполняется в изолированном исследовательском runtime с лимитами CPU/RAM/времени, без торговых ключей и с запрещённой сетью по умолчанию. Версия кода и dependencies входят в manifest.

## 4. От собственной идеи до работающей стратегии

### 4.1 Предметные состояния

`IDEA → SPEC_DRAFT → DATA_CHECKED → EXPERIMENT_REGISTERED → TESTED → VALIDATED_VERSION → PAPER_DEPLOYED → LIVE_ELIGIBLE → LIVE_DEPLOYED → PAUSED/RETIRED`.

Переход не обязан завершиться продвижением: `INSUFFICIENT_DATA`, `REJECTED`, `INCONCLUSIVE` и `RESEARCH_ONLY` — полноценные результаты. Live может быть недоступен даже для полезного исследования.

| Шаг | Что делает AI | Что определяет результат | Что сохраняется |
|---|---|---|---|
| Формализация | Выявляет неоднозначности; предлагает проверяемые правила | Пользователь подтверждает экономический смысл; schema validator проверяет полноту | Hypothesis, StrategySpec, список допущений |
| Поиск существующего | Ищет аналогичные templates/features/experiments | Версии и семантика библиотек | Reuse plan, а не копия нового модуля |
| Проверка данных | Вызывает capability query, объясняет пробелы | MKT-DS/DQ/TQ, права и PIT coverage | DataRequirement/CapabilityReport |
| План эксперимента | Предлагает baselines, splits, costs и sensitivities | Заранее фиксированный ExperimentSpec | Hash спецификации и budget поиска |
| Бэктест | Запускает задачу, отслеживает прогресс | Детерминированный replay и simulator | ExperimentResult, trade ledger, skips, provenance |
| Критика | Ищет leakage, переоптимизацию и неверные интерпретации | Автоматические проверки + независимый review | ValidationReport и ограничения |
| Paper | Помогает настроить наблюдение и лимиты | Отдельный PAPER runtime, реальные текущие данные, simulated fills | DeploymentManifest, исполненные/пропущенные события |
| Live | Готовит точную версию и объясняет разрешения | Eligibility, risk, mandate, human approval, route readiness | Immutable version + policy + audit |
| Сопровождение | Объясняет расхождения, предлагает исследовать изменения | Телеметрия, позиции, риск, versioned drift policy | DriftReport, pause или новый experiment |

### 4.2 Контракт стратегии

`StrategySpec` содержит `strategyId/version`, автора и tenant, цель, базовую валюту учёта, полный перечень источников, event-time/available-time policy, формулы и warmup, selectors, legs/ratios, sizing, entry schedule, exits, position overlap, costs, order type, partial-fill policy, expiry behavior, missing-data policy, risk limits, code/config digests и библиографию. Draft допускает нерешённые вопросы; runnable spec — нет.

`ExperimentSpec` отдельно фиксирует universe, период, обучающие и контрольные окна, embargo/purge для перекрывающихся labels, random seeds, варианты, критерии успеха и отказа, вычислительный бюджет. В ledger записываются все trials, включая неудачные, interrupted и отвергнутые. Повторный поиск после просмотра holdout создаёт новую исследовательскую итерацию с новым holdout; старый нельзя вновь назвать независимым.

`ValidatedStrategyVersion` связывает **конкретный** spec/code/data/engine и validation report. Это не награда стратегии навсегда. Доказательство процедуры, статистическая пригодность, разрешение торговли и фактическая прибыль — четыре отдельных утверждения.

### 4.3 Бэктест, который полезно сравнивать с реальностью

Один state/feature/strategy engine используется в replay и online; меняются source/clock/execution adapters. Историческая проверка включает реально существовавшие инструменты, делистинги, spreads, fees, lot/tick size, валюты, задержку и отсутствие котировки. Mid, mark и доступный bid/ask не взаимозаменяемы.

Обязательные результаты: число независимых сигналов и сделок, экспозиция и время в позиции, turnover, чистые денежные потоки, drawdown, хвостовые потери, стоимость исполнения, sensitivity, coverage, причины пропусков, результаты по режимам и интервал неопределённости. Нереализованный P&L на mark и liquidatable value показываются отдельно.

Baseline выбирается по гипотезе: cash/no trade, underlying exposure сопоставимого риска, та же опционная конструкция без сигнала, тот же сигнал без дополнительного фильтра. Сравнивать только с убыточным случайным вариантом недостаточно. Sharpe не вычисляется из редких сделок так, будто они независимые ежедневные наблюдения.

### 4.4 Продвижение и остановка

Для PAPER нужны replay/recovery, отрицательные сценарии, immutable policy и явная модель fills. Для live дополнительно: доступность инструментов, права клиента/площадки, экономические лимиты, защита от повторной отправки и сверка состояния. Новая версия модели сигнала, формулы, источника или выхода требует нового versioned review. AI не может «подправить» работающую стратегию в фоне.

Остановка стратегии прекращает новые sends и инициирует разрешённые отмены. Открытые позиции и незавершённые заявки продолжают учитываться. `run.cancel` отменяет аналитическую работу; `deployment.pause` меняет торговую политику; `order.cancel` имеет собственный жизненный цикл. Эти команды не заменяют друг друга.

## 7. Продуктовые поверхности и непрерывность работы

### 7.1 Один продукт и несколько способов работы

| Поверхность | Для чего нужна | Что остаётся общим |
|---|---|---|
| Connect: MCP/API/SDK/CLI | Пользователь работает в собственном агенте, notebook или приложении | Objects, данные, compute, hosted Skills, permissions, usage |
| Options Desk / начальный Market Workspace | Быстро исследовать выбранный рынок, собрать стратегию и разобрать результат | Scene/selection, Evidence, Thesis, StrategySpec, Experiments |
| Browser Workstation | Ежедневная многооконная работа, совместные исследования и глубокая визуализация | Та же предметная модель и серверные capabilities |
| Desktop Workstation | Несколько окон, hotkeys, локальный cache и desktop integration | Общая web-native клиентская основа и renderer contracts; wrapper выбирается проверкой |
| Pi Market Analyst | Тонкий агентный клиент и пользовательские локальные workflows | Hosted-вызовы private intelligence; открытые tools и личные настройки локально |
| Mobile companion | Просмотр разрешённых share-карточек, существенных уведомлений и follow-up | Стабильные object IDs и deep links; мобильный терминал не обязательный первый релиз |

Private Skills, рубрики Critic и eval corpus не распространяются вместе с клиентом. Одна и та же предметная операция из кнопки, команды, внешнего агента или AI-панели проходит одинаковые серверные проверки.

### 7.2 Что означает «Cursor для трейдера»

| Примитив | Реализация Odelix |
|---|---|
| Контекст открытого проекта | Market State, Workspace/Scene, активные Thesis, стратегии, позиции и разрешённая память |
| Выделение фрагмента | `SelectionRef`: instrument, time/price bounds, layers, object revisions, live/replay cursor и asOf |
| Ask | Короткий Fast Ask или ограниченный Deep Investigation с инструментами и проверкой evidence |
| Edit | Typed ChangeSet: изменить annotations/layout, создать Thesis/Watch, предложить новую версию стратегии |
| Diff и применение | Preview → Apply / Edit / Reject; exact Undo для обратимых изменений среды |
| Проверка | Replay, численные tests, experiment validation, Critic и воспроизводимый trace |
| Продолжение работы | Адресуемые objects, сохранённые layouts, Journal, Missions и Decision Memory |
| Расширения | Опубликованные contracts, SDK/MCP, templates, разрешённые predicates и Skills |

AI действует над структурированными объектами среды. Screenshot может помогать с расположением элементов, но цены, временные границы и риск он получает из typed tools. Undo интерфейса не отменяет финансовую сделку; cancel/close являются отдельными торговыми командами с собственными последствиями.

### 7.3 Workspace, View, Pane, Lens и команды

**Workspace** сохраняет рабочую задачу, bindings, открытые objects, focus и layout. **View** представляет объект или market scope. **Pane** задаёт геометрию и не владеет копией рыночной истины. **Layout Preset** меняет расположение. **Lens** меняет смысловой фокус той же Scene — например Order Flow, Liquidity, Options Context, Risk или Learning — без создания новой истории данных.

Command Bar, клавиатура, мышь и агент вызывают именованные команды одного registry. У каждой команды определены inputs, permissions, side effects и результат. Предлагая открыть панель, выделить уровень или построить overlay, агент указывает target object, reason и Evidence. ChangeSet содержит base revision; конфликт с уже изменённой Scene требует нового preview. Undo восстанавливает предыдущий UI/object state в допустимых границах.

Replay является режимом времени текущего Workspace: сохраняются layout и связанные objects, а data access ограничивается replay cursor. Переход назад не оставляет в контексте незаметно доступные будущие quotes, новости или результаты сделки. Live/paper/replay видны в интерфейсе постоянно.

### 7.4 Terminal и Options: конкретные рабочие возможности

| Область | Содержание и связь с работой |
|---|---|
| Terminal | Underlying chart, tape, depth/DOM, heatmap, footprint, CVD, VWAP и profiles по доступным данным; typed events и evidence |
| Options Overview | Spot/index, IV/RV, term/skew, expected move, ключевые уровни, quality и объяснение того, что изменилось |
| Smart Chain | Bid/ask и размеры, IV/Greeks/OI, spread/liquidity, expiry/delta filters; выбор нескольких контрактов для структуры |
| Strike Inspector | Единая карточка strike/expiry из Chain, Gamma, Flow, Scenario или underlying overlay; история и source timestamps |
| Gamma Map | Profile и time×price view; отдельные модели concentration, signed GEX assumptions, observed flow и estimated inventory |
| Options Flow | Prints, block/RFQ metadata при наличии, aggressor flow, inferred structures с маркированной неопределённостью |
| Volatility | Surface, smile/skew, risk reversal/butterfly, term structure, IV/RV и исторические percentiles |
| Scenarios | Ветки развития, условия подтверждения/отмены, evidence for/against, target/range и отличие от предыдущей версии |
| Strategy Lab | Шаблоны и свои legs/rules; payoff сейчас/на expiry, Greeks, time/spot/IV shocks, сравнение, переход в Quant/Paper |
| Positions | Book P&L, collateral/margin из правильного owner, aggregate Greeks, expiry/strike concentration и full revaluation/stress |
| Options Scalping | Синхронные underlying/option views, strikes in play, доступность ликвидности и gated execution ticket |

Это целевой каталог возможностей, а не перечень одновременно готовых экранов. Начальная поверхность использует минимальные панели для законченного процесса. Глубокий high-rate renderer выносит обработку потока из React; tick path не зависит от LLM.

Options и Terminal связаны временем, инструментами и Thesis. Нажатие на option print переводит underlying cursor к тому же моменту. Gamma-level открывает исходную модель и наблюдавшиеся реакции. Переход из Scenario в Strategy Lab переносит контекст, а не открывает пустую форму. OI/gamma не доказывают точный dealer inventory; оценка режима остаётся моделью. Недоступный quote не заменяется нулём, а cross-venue dispersion при единственном источнике остаётся неопределённой.

### 7.5 Живые Thesis, Watch и Journal

Thesis хранит horizon, claims, evidence for/against, scenarios, confirmation, invalidation, неизвестные и expiry. Новое наблюдение создаёт revision и видимый delta. Пользователь может вернуться к состоянию «что было известно тогда»; последующий исход не переписывает основание старого решения.

Watch показывает формальные predicates, inputs/freshness, semantic question, cooldown, срок действия и правила уведомления. На событии он сначала проверяет данные, затем при необходимости запускает ограниченное исследование. Пропавший feed переводит зависимую проверку в degraded/paused; отсутствие информации не считается подтверждением Thesis.

Journal связывает исходную Scene, действие пользователя, accepted/edited/rejected предложения, позиции и outcome. CSV-импорт собственной истории может дать начальный материал для Decision Memory до подключения exchange keys. Process review различает удачное решение, случайно прибыльный результат, нарушение правил и недостаток данных.

Post-trade companion использует актуальный position/risk snapshot: объясняет P&L по движению underlying, IV, времени, fees/spread и residual approximation error; сравнивает Hold/Close/Roll/Reduce как пересчитанные варианты. Watch или уведомление сами не открывают сделку. Закрытие/отзыв доступа не зависят от доступности AI-чата.

### 7.6 Personal Agent Builder и Agent Console

Пользователь задаёт **Goal → Inputs → Tools → Trigger → Policy → Output → Review**. Примеры: «Проверять мою опционную гипотезу после закрытия 4H», «Сообщать о существенном изменении skew», «Каждое утро собирать brief по моим Thesis». Это конфигурация ограниченной capability, а не разрешение на произвольный shell или кошелёк.

Перед активацией Builder показывает источники, доступные actions, triggers, бюджет, пример output, условия остановки и необходимые approvals. Новые права требуют отдельного действия пользователя. Один AgentDefinition может порождать короткие Runs или долгоживущую Mission; live permissions появляются только после соответствующего этапа готовности.

Console показывает state, текущую задачу, следующий запуск, tool calls, evidence, model/skill versions, стоимость, checkpoint, ошибки, approvals и причину блокировки. Pause Mission, cancel Run, revoke grant, disable deployment и cancel order различаются в UI. Отмена текущего LLM-ответа не гарантирует остановку уже отправленного приказа; состояние сверяется с Execution.

### 7.7 Учебная и профессиональная глубина

Simple / Trader / Quant меняют глубину представления одних данных. Simple объясняет экономический смысл, стоимость и ограничения; Trader открывает chain, Greeks и flow; Quant — формулы, inputs, timestamps, source versions и controls эксперимента. Профессионалу не нужно проходить обязательный обучающий диалог перед обычной операцией.

Learning Workspace объединяет Guided Replay, Live Tutor и Review. Tutor предлагает остановиться на историческом моменте, сформулировать гипотезу, проверить её на доступных тогда данных и сравнить решение с последующим исходом. Обучение использует те же Evidence, расчёты и защиту от будущих данных. Учебный прогресс не выдаёт торговые права.

Calm-by-default означает material updates, debounce, quiet hours, attention budget и группировку связанных событий. Пользователь делегирует наблюдение без обязанности читать каждый tick. Аварийные risk events обрабатываются по отдельной policy и не зависят от доступности AI.

### 7.8 Продолжение, экспорт и распространение

Decision Passport связывает goal, snapshot, alternatives, costs, risk, решение, approvals и outcome. Strategy Passport добавляет точную spec/code/data/engine version, trials, validation и раздельную paper/live историю. Research Package переносит разрешённые данные/refs, methodology и objects в внешнюю среду; доступность экспорта зависит от data rights.

Share-карточка рыночного момента или стратегии может быть открыта, изменена и проверена другим пользователем. Получатель получает новую версию и пересчёт на своём состоянии; чужой анализ не передаёт полномочий на торговлю и не раскрывает частную историю автора. Переносимые пакеты остаются полезны даже без публичного permalink.

WebMCP рассматривается как дополнительный браузерный интерфейс контекста и разрешённых действий. Он использует те же серверные contracts и permissions; поддержка конкретным browser/agent проверяется отдельно. Workstation-first не зависит от этого механизма. [Описание WebMCP от Chrome](https://developer.chrome.com/blog/webmcp-epp).


### 7.9 Полнота Workstation после отказа от Emacs

Переход на React/TypeScript сохраняет не внешний вид редактора, а объектную рабочую модель: независимые views вместо buffers, единые команды вместо mode-specific shortcuts, долговечные Workspace/Layout/Lens, точное выделение и общий agent context. Полный каталог восстановлен в [Workstation specification](DEVELOPMENT.md#DEP-568bec8dbe): **72 исходных типа views, 12 panel roles, 15 workspaces и 12 lenses**. Proposed IDs выведены из исторических modes и не объявлены опубликованным API.

15 workspaces: Morning, Market, Level Investigation, Flow, Options, Quant Lab, Thesis, Missions, Portfolio, Pretrade, Position Guard, Replay, Review, Operations и Custom. Двенадцать lenses: Price, Flow, Liquidity, Perpetuals, Options, Thesis, Position, Execution, Replay, Regime, Quant и Review. Это каталог задач и представлений, а не 72 панели на экране и не столько же microservices. Default остаётся спокойным: Primary Canvas, AI/Context, Command Bar и status strip; остальные views открываются по потребности.

Вернулись также точные поведенческие границы: comparison instruments/venues отдельно от similarity search; synchronized cursor/options/underlying; сохранить layout отдельно от Thesis; Scene snapshot/delta/epoch/resync; backpressure; renderer fallback; восстановление сессии после crash; privacy/export; keyboard access; scope-aware Agent Builder; предложения изменения среды и различие Undo/cancel/close. Полная Workstation расширяется по текущей очереди до PRO_WORKSTATION; FLOW_DESK_LOCAL и FLOW_DESK_PILOT имеют отдельный scope. Полный каталог и первый платёж не являются техническими предпосылками этих экранов.

### 7.10 Техническая приёмка среды

Проверяются object identity UI/API/agent, точность SelectionRef/asOf, один policy path для мыши/клавиатуры/агента, запрет stale ChangeSet, replay future firewall, корректный resync после gap, numerical/model labels в Options и отсутствие private hosted machinery в клиенте. Длительная работа и renderer recovery требуют отдельного runtime soak, не выводятся из наличия этих текстов. Исторические Emacs/Doom/Elisp и wrapper-harness предложения сохранены как reference, но не возвращаются в действующую архитектуру.


## Пользовательская логика отдельно от опционной конструкции

Каталог legs/payoff не ограничивает источники сигналов. Пользователь комбинирует доступные данные, исследует зависимость без позиции либо задаёт независимые entry/exit для выбранной конструкции. Formula/rule editor, dependency view, data readiness, timeline of decisions и version comparison используют общую Research-модель. Параметры и источники примера не становятся defaults; semantic conflicts разрешаются до runnable spec. Подробнее: [Research specification](STRATEGY-RESEARCH-AND-EXECUTION.md).


## Актуализация r15 — рабочий flow и проверяемая семантика

Fastify в apps/product-host. Ранние Scene contracts и grants; SemanticAssessment persistence/disposition/visibleAt. Product не считает market flow и не подменяет evaluator/runtime. Billing позже первого raw UI.

Подробная обязательная спецификация: [SEMANTIC-ASSESSMENT-LIFECYCLE.md](SEMANTIC-ASSESSMENT-LIFECYCLE.md). Старые утверждения о полноте прототипа и исторические оценки сроков не являются runtime evidence.
