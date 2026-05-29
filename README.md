# bmad-ent-arch-agent

Агент корпоративного архитектора банка на платформе [BMad](https://github.com/bmadcode/bmad-method). Помогает продуктовым командам и архитекторам вписывать новые инициативы в утверждённую целевую архитектуру банка, находить готовые решения для переиспользования и готовить материалы для ARB.

## Агент

**Archon** — седой стратег корпоративной архитектуры, двадцать лет на стыке бизнеса и систем банка. Думает в горизонтах 3–5 лет. Знает TOGAF, ArchiMate, BIAN, IT4IT, DDD, Cloud-Native patterns. Различает текущий ландшафт и утверждённую целевую картину — и всегда видит, где между ними дрейф.

Ключевой принцип: **каждое утверждение либо ссылается на конкретный документ, принцип или элемент реестра — либо помечается как мнение.**

## Возможности

| Режим | Когда использовать |
|---|---|
| **Архитектурная консультация** | Новая инициатива — нужно понять, как вписать в ландшафт, получить 2–3 опции с trade-off'ами |
| **Adversarial-ревью** | Готовый ADR / Solution Design / proposal — жёсткий разбор по 8 осям с severity |
| **Проверка compliance и target arch** | Структурированный аудит соответствия 152-ФЗ, 115-ФЗ, PCI DSS, GDPR и target arch |
| **Поиск переиспользования** | Найти в ландшафте банка то, что уже решает задачу, до начала нового строительства |
| **Создание архитектурного артефакта** | Создать ADR, Solution Design или ARB-пакет — с нуля или по идее |

### Автоматические (headless) режимы

```
--headless:landscape-drift-scan   Сканировать ландшафт на дрейф от target arch
--headless:compliance-audit       Audit compliance gaps по всему банку
--headless:arb-prep {path}        Сгенерировать ARB Presentation Package из ADR / SD
```

## Структура проекта

```
bmad-ent-arch-agent/
├── skills/
│   └── agent-archon/             # Агент Archon
│       ├── SKILL.md              # Основной файл агента
│       ├── customize.toml        # Точки кастомизации
│       └── references/           # Промпты возможностей
│           ├── architectural-consultation.md
│           ├── adversarial-review.md
│           ├── compliance-check.md
│           ├── reuse-discovery.md
│           └── artifact-generation.md
├── _bmad/
│   ├── config.toml               # Конфиг проекта (генерируется установщиком)
│   ├── config.user.toml          # Персональный конфиг (генерируется установщиком)
│   ├── custom/                   # Ваши переопределения (не трогаются установщиком)
│   │   ├── config.toml           # Командные настройки
│   │   ├── config.user.toml      # Личные настройки
│   │   └── agent-archon.toml     # Кастомизация Archon для команды/лично
│   └── scripts/
│       └── resolve_customization.py
└── docs/                         # База знаний банка (настраивается)
    ├── landscape/
    │   └── registry.yaml         # Реестр систем банка
    ├── architecture/
    │   ├── target.md             # Утверждённая целевая архитектура
    │   └── principles.md         # Принципы и стандарты банка
    ├── compliance/               # Регуляторный корпус
    └── strategy.md               # Стратегия развития банка
```

## Настройка

### 1. База знаний банка

Archon читает четыре источника при запуске. Пути настраиваются в `_bmad/custom/agent-archon.toml`:

```toml
[agent]
landscape_registry_path    = "{project-root}/docs/landscape/registry.yaml"
target_architecture_path   = "{project-root}/docs/architecture/target.md"
architecture_principles_path = "{project-root}/docs/architecture/principles.md"
compliance_corpus_path     = "{project-root}/docs/compliance/"
strategy_path              = "{project-root}/docs/strategy.md"
assessment_output_path     = "{project-root}/_bmad-output/archon/"
```

Отсутствующие файлы вызывают предупреждение, но не прерывают работу.

### 2. Язык и пользователь

В `_bmad/custom/config.user.toml`:

```toml
[core]
user_name = "Ваше имя"
communication_language = "Russian"
document_output_language = "Russian"
```

### 3. Постоянные факты сессии

В `_bmad/custom/agent-archon.toml` можно добавить факты, которые Archon будет помнить во всех сессиях:

```toml
[agent]
persistent_facts = [
  "file:{project-root}/**/project-context.md",
  "Наш банк — только on-prem для систем Tier-1.",
  "ARB собирается каждую первую среду месяца.",
]
```

### 4. Хук после завершения оценки

```toml
[agent]
on_assessment_complete = "python3 scripts/notify_arb.py --report {output}"
```

## Использование

Активируйте агента в Claude Code:

```
/agent-archon
```

Или с флагом для автоматического режима:

```
/agent-archon --headless:arb-prep docs/decisions/adr-042-event-hub.md
```

После активации Archon предложит выбрать режим работы. При наличии готового артефакта сразу укажите, что именно нужно: консультация, разбор ADR, compliance-аудит и т.д.

## Выходные артефакты

Все отчёты сохраняются в `_bmad-output/archon/` с именованием:

| Тип | Имя файла |
|---|---|
| Архитектурная консультация | `consultation-{slug}-{YYYYMMDD}.md` |
| Adversarial review | `review-{adr-id}-{YYYYMMDD}.md` |
| Compliance check | `compliance-{slug}-{YYYYMMDD}.md` |
| Reuse map | `reuse-{slug}-{YYYYMMDD}.md` |
| ADR | `adr-{NNN}-{YYYYMMDD}.md` |
| Solution Design | `sd-{slug}-{YYYYMMDD}.md` |
| ARB Package | `arb-package-{slug}-{YYYYMMDD}.md` |

## Требования

- [Claude Code](https://claude.ai/code) с поддержкой BMad skills
- Python 3 (для скрипта `resolve_customization.py`)
- Файлы базы знаний банка (без них Archon работает с предупреждениями)
