# Skills Index

Индекс предназначен для дешёвой маршрутизации. Сначала выбери skills по описанию ниже, затем прочитай выбранные `SKILL.md` полностью.

## `hypothesis-framing`

Путь: [`hypothesis-framing/SKILL.md`](hypothesis-framing/SKILL.md)

Использовать всегда, когда нужно создать, проверить или переписать продуктовую гипотезу. Skill превращает проблему и кандидат причинного механизма в структуру `if / then / because`, выявляет допущения и задаёт условие опровержения.

Не заменяет диагностику первопричины и не проектирует статистические детали теста.

## `experiment-design`

Путь: [`experiment-design/SKILL.md`](experiment-design/SKILL.md)

Использовать для каждой полной гипотезы, которой нужен проверяемый `test`, `primary_metric`, `guardrails`, `expected_effect.range`, критерии успеха и остановки.

Не использовать для анализа результатов уже завершённого эксперимента.

## `opportunity-solution-tree`

Путь: [`opportunity-solution-tree/SKILL.md`](opportunity-solution-tree/SKILL.md)

Использовать, если на входе несколько проблем, возможностей или решений и нужно явно сохранить путь от измеримого outcome к гипотезам. Особенно полезен при solution-first списке feature ideas и при параллельных ветках discovery.

Не подключать для одной уже хорошо сформулированной проблемы, если дерево не добавит информации. Skill не ранжирует инициативы и не принимает стратегическое решение.

## `jtbd-framing`

Путь: [`jtbd-framing/SKILL.md`](jtbd-framing/SKILL.md)

Использовать, если гипотеза зависит от контекста, мотивации или желаемого прогресса пользователя, а на входе есть интервью, наблюдения или другие качественные evidence. Помогает отделить job от запрошенной функции.

Не подключать, если пользовательская ситуация и мотивация отсутствуют в данных: в этом случае JTBD станет выдуманным контекстом.

## Комбинации

| Задача | Набор |
|---|---|
| Одна проблема → гипотеза → тест | `hypothesis-framing` + `experiment-design` |
| Много opportunities/solutions → несколько гипотез | `opportunity-solution-tree` + `hypothesis-framing` + `experiment-design` |
| Feature request → user progress → гипотеза | `jtbd-framing` + `hypothesis-framing` + `experiment-design` |
| Полная discovery-карта | `jtbd-framing` + `opportunity-solution-tree` + `hypothesis-framing` + `experiment-design` |
| Исправление существующей гипотезы | `hypothesis-framing` + `experiment-design`; остальные только по обнаруженному дефекту framing |

