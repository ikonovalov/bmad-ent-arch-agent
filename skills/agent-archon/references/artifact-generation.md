---
name: artifact-generation
description: Guided drafting of ADRs, Solution Designs, and ARB Presentation Packages — discovery dialog, bank-context anchoring, and ARB-ready output with explicit compliance flags and alternative analysis.
menu-code: AG
---

# Draft Architectural Artifact

**Language:** общайся на `{communication_language}`, артефакт пиши на `{document_output_language}`.

Пользователь хочет создать новый ADR, Solution Design или ARB-пакет — либо с нуля, либо по ядру мысли, которое нужно оформить в стандарте банка. Твоя работа — вести его через процесс discovery и выдать артефакт, **готовый к передаче в adversarial-review или ARB**, а не болванку с заглушками.

## What Success Looks Like

К концу сессии пользователь получает **конкретный, цитируемый артефакт**, в котором:

- Контекст и силы (forces) задокументированы на уровне, понятном любому члену ARB без пояснений автора.
- Принятое решение (или предлагаемое решение) явно обоснован через ссылки на `{agent.target_architecture_path}`, `{agent.architecture_principles_path}` и `{agent.landscape_registry_path}`.
- Отклонённые альтернативы расписаны с причинами — «мы рассматривали, но» — не просто перечислены.
- Compliance-touchpoints отмечены (не аудит — флажки; для аудита есть capability `compliance-check`).
- Статус и следующие шаги определены.

## Supported Artifact Types

### ADR (Architecture Decision Record)

Используется для фиксации одного архитектурного решения. Область — одна capability area, один сервис, один паттерн интеграции.

**Scope:** «Мы решили X, потому что Y, и отклонили Z, потому что W.»

### Solution Design

Используется для описания целостного решения, затрагивающего несколько систем или capability areas. Больший контекст, больше компонентов, явная integration view.

**Scope:** инициатива от идеи до High-Level Design — достаточно, чтобы команда начала Low-Level Design и спринт-планирование.

### ARB Presentation Package

Структурированный пакет для вынесения на Architecture Review Board: executive summary, архитектурное решение, alignment matrix с target arch, risk & compliance summary, вопросы для ARB. Может генерироваться как из готового ADR/SD (headless-режим), так и интерактивно с нуля.

## Degraded Mode

Если `landscape_registry` missing — раздел «Текущий ландшафт (As-Is)» и все ссылки на системы из registry помечай `[NEEDS INPUT: landscape registry not loaded]`. Не заполняй от себя. Если `target_architecture` missing — раздел «Alignment с целевой архитектурой» получает статус `needs-clarification` по всем строкам. Если `architecture_principles` missing — в артефакте не ссылайся на принципы по номеру, используй `[NEEDS INPUT: principles not loaded — add principle reference before submission]`. Блок Knowledge Disclosure обязателен в начале discovery-диалога и в заголовке финального артефакта.

## Approach

### Discovery Dialog

Не начинай с шаблона. Задай уточняющие вопросы сначала — потом пиши.

Минимальный набор для старта:

1. **Что именно решаем?** Инициатива / задача / проблема одним предложением.
2. **Бизнес-драйвер.** Почему это важно сейчас — дедлайн, регуляторика, стратегическая программа?
3. **Домен данных.** Персональные данные, карточные, KYC, телеметрия — что попадает в scope?
4. **Какие системы затронуты?** Пользователь называет — ты сверяешь с `{agent.landscape_registry_path}`.
5. **Какие варианты уже рассматривались** и почему идём именно этим путём?

Если пользователь торопится и уже всё знает — попроси хотя бы ответы на п.1 и п.5: без них артефакт получится пустым.

### Anchoring в Bank Context

После discovery — до первого драфта. Все три knowledge base уже в контексте сессии (загружены при активации) — сверься с ними параллельно:

- В `{agent.landscape_registry_path}`: найди системы в scope, уточни owner'ов и статусы.
- В `{agent.target_architecture_path}`: какую capability area затрагивает решение, что target arch говорит о том, как она должна выглядеть.
- В `{agent.architecture_principles_path}`: какие принципы напрямую применимы.

Все ссылки, которые найдёшь, **войдут в артефакт явно** — не как фоновое знание, а как цитаты.

### Draft & Iterate

Пиши артефакт в один проход, потом уточняй с пользователем. Не проси подтверждение на каждый раздел — это создаёт friction. Выдай полный черновик, предложи правки блоками.

Следи за пробелами: если нет данных для заполнения раздела — **напиши `[NEEDS INPUT: {точный вопрос}]`** вместо заглушки. Пользователь видит, что нужно уточнить, а не гадает, почему раздел пустой.

### Handoff

После финализации артефакта предложи:

- Immediate adversarial-review — proceed to `adversarial-review` с только что созданным артефактом.
- Compliance-deep-dive — proceed to `compliance-check`, если есть неразрешённые compliance-флажки.
- Сохранить и выйти — запись в `{agent.assessment_output_path}`, выполнение `{agent.on_assessment_complete}`.

## Anti-Patterns

- **Шаблон-в-лоб.** Выдать структуру с заглушками без discovery — значит переложить работу на пользователя. Задай вопросы или заполни из контекста сессии.
- **Ссылки понарошку.** «Соответствует принципам банка» без конкретного принципа — не ссылка. Либо называй пункт, либо не пиши.
- **Альтернативы-для-галочки.** «Рассматривали Kafka, но выбрали API» — не объяснение. Напиши, почему именно Kafka не подошла в данном контексте.
- **Compliance как последний раздел с прочерком.** Домен данных определён на шаге discovery — если там есть ПДн или карточные данные, флажки должны появиться в артефакте.
- **Статус без следующего шага.** «Proposed» без «следующий шаг — передача в ARB до {дата}» — неполный артефакт.

## Completion Criteria

Артефакт считается завершённым, когда:

- Нет незаполненных разделов — только явные `[NEEDS INPUT: ...]` с точными вопросами.
- Каждое утверждение о соответствии target arch или принципам сопровождается ссылкой.
- Отклонённые альтернативы объяснены, не перечислены.
- Compliance-флажки расставлены.
- Статус и next steps определены.

## Output Formats

### ADR

Файл: `{agent.assessment_output_path}/adr-{id-or-slug}-{YYYYMMDD}.md`

Загрузи шаблон `references/templates/adr-template.md` и заполни все секции. Если `{agent.adr_index_path}` задан — прочитай файл для определения следующего номера ADR-{NNN}; иначе — укажи `[NEEDS INPUT: ADR index not configured]`.

---

### Solution Design

Файл: `{agent.assessment_output_path}/sd-{slug}-{YYYYMMDD}.md`

Загрузи шаблон `references/templates/solution-design-template.md` и заполни все секции.

---

### ARB Presentation Package

Файл: `{agent.assessment_output_path}/arb-package-{slug}-{YYYYMMDD}.md`

Загрузи шаблон `references/templates/arb-package-template.md` и заполни все секции. Дату ARB: используй `{agent.arb_next_session_date}` если задан; иначе — TBD.

---

## Headless: `--headless:arb-prep`

Запускается с указанием артефакта: `--headless:arb-prep {path-to-adr-or-sd}`.

Загрузи переданный артефакт, `{agent.target_architecture_path}`, `{agent.architecture_principles_path}` и `{agent.landscape_registry_path}` параллельно (единый batch-вызов). Сгенерируй ARB Presentation Package по шаблону `references/templates/arb-package-template.md`. Дата ARB: используй `{agent.arb_next_session_date}` если задан; иначе — TBD. Все `[NEEDS INPUT: ...]` заполняй из доступного контекста; что не удаётся вывести — помечай явно. Сохрани в `{agent.assessment_output_path}/arb-package-{slug}-{YYYYMMDD}.md` и выполни `{agent.on_assessment_complete}`.
