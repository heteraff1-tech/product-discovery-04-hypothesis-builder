---
name: opportunity-solution-tree
description: "Строит evidence-backed связь desired outcome → opportunities → solution candidates → hypotheses. Используй при нескольких ветках discovery или solution-first списке; не используй для одной готовой гипотезы, скоринга RICE/ICE или окончательного выбора стратегии."
---

# Opportunity Solution Tree

Цель skill — не потерять логическую связь между измеримым outcome, потребностью пользователя, возможным решением и проверяемой гипотезой. Дерево структурирует пространство discovery, но не доказывает, что ветка верна или приоритетна.

Перед построением прочитай [`../../references/gpt3-intake-adapter.md`](../../references/gpt3-intake-adapter.md) и [`../../references/evidence-policy.md`](../../references/evidence-policy.md). Для каждой hypothesis затем используй [`../hypothesis-framing/SKILL.md`](../hypothesis-framing/SKILL.md) и [`../experiment-design/SKILL.md`](../experiment-design/SKILL.md).

## Когда дерево добавляет ценность

Используй skill, если выполняется хотя бы одно условие:

- у одного `desired_outcome` несколько подтверждённых opportunities;
- передан список feature ideas без явной связи с проблемами;
- разные решения смешаны с разными пользовательскими потребностями;
- команде нужна прослеживаемая карта нескольких hypotheses;
- требуется показать, какие ветки основаны на evidence, а какие остаются assumptions.

Не строй декоративное дерево для одной уже ограниченной проблемы. Не используй расположение веток как рейтинг.

## Типы узлов

- `desired_outcome`: измеримый продуктовый или пользовательский результат, на который команда способна влиять.
- `opportunity`: наблюдаемая потребность, препятствие или желаемый прогресс пользователя. Это не feature и не внутренний проект.
- `solution_candidate`: возможный способ воздействовать на opportunity. Название решения не означает одобрение.
- `hypothesis`: проверяемая связь между intervention, outcome и предполагаемым механизмом.
- `test`: минимальная проверка наиболее рискованного допущения.

Каждый узел получает стабильный ID. Каждая opportunity содержит `status`, `jtbd_ids` и typed `evidence_links`; если evidence отсутствует, используй `status: assumption` или вынеси узел в `missing_inputs`.

## Построение

1. **Уточни outcome.** Запиши `desired_outcome` и `outcome_metric`. Если outcome не измерим или не находится в зоне влияния продукта, обозначь пробел до построения веток.
2. **Нормализуй opportunities.** Переформулируй feature requests как пользовательские препятствия или желаемый прогресс только в пределах evidence. Upstream refs не меняй; в локальных links укажи `supports`, `stance` и `claim_ref`.
3. **Раздели независимые потребности.** Не объединяй разные сегменты, ситуации и jobs в один широкий узел ради компактности.
4. **Добавь варианты решений.** Для подтверждённой opportunity допустимы несколько `solution_candidates`. Решения являются идеями, а не evidence.
5. **Свяжи hypotheses.** Для каждого решения укажи `hypothesis_ids`, проверяющие конкретный механизм. Одна hypothesis может относиться к одной ясной ветке; если она покрывает несколько несвязанных веток, раздели её.
6. **Выдели риск.** Свяжи решение с `assumption_ids` и укажи, какой assumption проверяется первым. Это порядок обучения, не бизнес-приоритет.
7. **Покажи пробелы.** Отдельно перечисли unsupported opportunities, отсутствующие source links, противоречия и альтернативные ветки.

## Использование JTBD

Если opportunity зависит от ситуации и мотивации пользователя, подключи [`../jtbd-framing/SKILL.md`](../jtbd-framing/SKILL.md). Связывай `opportunity.jtbd_ids` только с frames, построенными на доступных evidence. Не создавай emotional или social jobs по умолчанию.

## Выход

Заполни `opportunity_solution_tree` по JSON Schema:

- `desired_outcome` и `outcome_metric`;
- `opportunities` с `opportunity_id`, `status`, `statement`, `evidence_links` и `jtbd_ids`;
- `solution_candidates` с `solution_candidate_id`, `statement`, `assumption_ids` и `hypothesis_ids`.

Верни полные hypothesis cards отдельно: дерево даёт навигацию, но не заменяет контракт гипотезы.

## Quality gate

- outcome измерим и отделён от решения;
- opportunities сформулированы как user need, friction или progress;
- у каждой подтверждённой opportunity есть typed links, разрешимые в registry/ref;
- solution candidates не представлены как доказанные ответы;
- каждая hypothesis прослеживается до одной понятной ветки;
- дерево не содержит скрытого RICE/ICE ranking;
- unsupported ветки явно отмечены, а не смешаны с evidence-backed узлами.
