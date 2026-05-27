---
name: landscape-drift
description: Headless landscape drift scan — detects divergence between current bank systems and approved target architecture, producing a prioritized drift report.
menu-code: LDS
---

# Headless: `--headless:landscape-drift-scan`

**Language:** отчёт пиши на `{document_output_language}`.

Периодическое сравнение текущего ландшафта с утверждённой целевой архитектурой. Запускается автоматически (CI/cron) или вручную. Выявляет системы, которые drift'уют от target arch — без диалога, без подтверждений.

## Что Success Looks Like

Отчёт содержит:

- Список всех drift-расхождений с severity (blocker / high / medium / low).
- Каждое расхождение цитирует конкретный раздел target arch и конкретную запись landscape registry.
- Summary-блок с количеством по severity (для CI-парсинга).
- Рекомендации по приоритетам исправления.

## Approach

Загрузи `{agent.landscape_registry_path}` и `{agent.target_architecture_path}` параллельно (единый batch-вызов).

Сравни каждую запись в landscape registry с целевой архитектурой по осям:

1. **Capability owner** — соответствует ли владелец системы тому, что определяет target arch для этой capability area?
2. **Integration patterns** — использует ли система паттерны интеграции (ESB/Event Hub/API Gateway), определённые как стандарт в target arch?
3. **Технологический стек** — не устарел ли стек относительно approved tech list в target arch?
4. **Дублирование capabilities** — нет ли в реестре систем, которые реализуют одну и ту же capability — при том, что target arch предписывает единственного owner'а?
5. **Deprecated entries** — есть ли в реестре системы, помеченные как deprecated, но всё ещё активно используемые?

Каждое выявленное расхождение оценивай по severity:

| Severity | Когда применять |
|---|---|
| **blocker** | Нарушение принципа безопасности, compliance-требования или блокирует утверждённый стратегический проект |
| **high** | Явное отклонение от target arch, которое будет накапливать технический долг и усложнит будущие изменения |
| **medium** | Расхождение, которое не критично сейчас, но потребует миграции в горизонте 1–2 лет |
| **low** | Незначительное несоответствие, на усмотрение команды |

## Output Format

Отчёт: `{agent.assessment_output_path}/landscape-drift-{YYYYMMDD}.md`

```
---
run_date: {YYYY-MM-DD}
mode: landscape-drift-scan
knowledge_status: loaded|partial|missing
summary:
  blockers: N
  drift_items: N
  high: N
  medium: N
  low: N
---

# Landscape Drift Report: {YYYY-MM-DD}

## Сводка

{N} расхождений обнаружено: {N} blocker, {N} high, {N} medium, {N} low.

## Расхождения (по severity)

### [BLOCKER] {Название расхождения}
- **Система:** {registry entry + owner}
- **Ось:** {capability owner / integration pattern / tech stack / дублирование / deprecated}
- **Что не так:** {конкретное расхождение}
- **Источник:** {target arch §M.N + registry REG-{NNN}}
- **Рекомендация:** {что сделать и в каком горизонте}

### [HIGH] ...
### [MEDIUM] ...
### [LOW] ...

## Замечания

{Что выглядит хорошо — системы, соответствующие target arch.}
```

## Degraded Mode

Если `landscape_registry` missing — отчёт невозможен; завершить с сообщением об ошибке в YAML summary и кодом ошибки. Если `target_architecture` missing — аналогично.
