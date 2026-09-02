---
name: experiment-design
description: "Проектирует минимальный интерпретируемый тест для уже сформулированной продуктовой гипотезы: primary metric, guardrails, expected effect, критерии успеха, опровержения и остановки. Не используй для формулирования исходной проблемы или анализа завершённого теста."
---

# Experiment Design

Цель skill — подобрать самый экономный тест, который уменьшает ключевую неопределённость гипотезы, не обещая более сильный вывод, чем допускает дизайн. Сначала должна существовать hypothesis, сформированная через [`../hypothesis-framing/SKILL.md`](../hypothesis-framing/SKILL.md).

Прочитай [`../../references/evidence-policy.md`](../../references/evidence-policy.md), особенно правила для typed evidence links, числовых claims и причинных выводов. Перед присвоением `ready_to_test` выполни [`../../references/pre-output-checklist.md`](../../references/pre-output-checklist.md).

## Начальная проверка

До выбора метода проверь:

- `if`, `then`, `because` разделены;
- указан `target_segment`;
- существует `falsification_condition`;
- assumptions перечислены и имеют ID;
- известны доступный трафик, риски, измеримость и ограничения;
- `primary_metric` может наблюдать `then`.

Если эти условия не выполнены, вернись к framing. Не маскируй неясный claim сложным экспериментом. Для incomplete plan используй `status: blocked`; schema допускает `null` в metric/test blocks.

## Выбор теста

Выбирай метод по наиболее рискованному `assumption_id` и требуемой силе вывода:

| Неопределённость | Подходящий ранний test | Что он не доказывает |
|---|---|---|
| Пользователь понимает и может выполнить действие | `usability_test`, prototype test | Массовый metric lift |
| Пользователь проявит намерение | fake door, smoke test | Долгосрочное использование и retention |
| Пользователь получит ценность при ручной доставке | concierge/pilot | Масштабируемость реализации |
| Изменение вызывает количественный эффект | randomized A/B test | Переносимость за пределы выбранной population |
| Рандомизация невозможна | cohort/quasi-experiment | Полную защиту от confounding |
| Причинный механизм пока неясен | targeted interview или observation | Частоту эффекта и причинность |

Один test должен иметь один главный learning objective. Если требуется проверить независимые assumptions, раздели тесты или явно укажи последовательность.

## Метрики

### `primary_metric`

Выбери ровно одну метрику, непосредственно отражающую `then`. Определи:

- `name` и операционное `definition`;
- `event_or_formula`;
- ожидаемое `direction`;
- `baseline` и `target`, если они известны;
- `unit`, `measurement_window` и `data_source`.

Не подменяй поведенческий outcome удобной vanity metric. Если текущая инструментализация не позволяет измерить метрику, это blocking measurement assumption.

### `guardrails`

Добавь показатели возможного вреда: ошибки, отмены, complaints, latency, revenue dilution, нагрузка, privacy или ухудшение соседнего этапа funnel. Для каждого guardrail определи direction, threshold и `breach_action`. Не придумывай threshold: при отсутствии основания используй `null`, добавь blocking reason и не ставь `ready_to_test`, если method требует порог до запуска.

## `expected_effect`

Задай направление и диапазон эффекта. Для `range.min` и `range.max` разрешены только основания:

- историческое наблюдение в сопоставимом контексте;
- расчёт из baseline и известного ограничения;
- релевантный benchmark с явной оговоркой переносимости;
- числовое допущение, явно принятое для расчёта мощности.

Укажи `basis` и `rationale`, а supporting evidence оформи links с `supports: expected_effect` или `baseline`. Если основания нет, обе границы остаются `null`; это не мешает раннему qualitative test, но блокирует расчёт quantitative experiment.

## План теста

Заполни:

- `method` и проверяемые `riskiest_assumption_ids`;
- `design`, `population`, `control`, `treatment` и `allocation`;
- `duration` и `sample_size` с основанием либо `null`;
- заранее определённые `success_criteria` и `falsification_criteria`;
- `stop_conditions`, включая breach guardrail;
- `analysis_plan` и известные `limitations`.

Для A/B test не указывай sample size на глаз. Нужны baseline rate/variance, minimum detectable effect, alpha, power и ожидаемый traffic. Если их нет, перечисли их в `missing_inputs` и установи test или hypothesis в `blocked` до расчёта.

Качественный тест может опровергнуть usability или comprehension assumption, но сам по себе не подтверждает количественный `then`. Отрази это в limitations и confidence.

## Semantic readiness gate

Заполни `readiness` после проверки, а не до неё. `ready_to_test` требует `readiness.ready: true`, разрешимый registry, определённую instrumentation/population и все параметры, обязательные для выбранного method. Для A/B test это включает control/treatment/allocation, baseline, MDE или effect range, alpha/power, рассчитанные sample size и duration. Для qualitative test нужны recruitment criteria, observable task/outcome и stopping rule.

При любом method-specific пробеле используй `blocked` и конкретный `blocking_reason`. `rejected` hypothesis не требует test object.

## Quality gate

- test проверяет конкретную hypothesis и наиболее рискованный assumption;
- метод соответствует требуемой силе вывода;
- primary metric ровно одна и измеряет `then`;
- guardrails имеют условия breach и действия;
- критерии успеха, опровержения и остановки заданы до запуска;
- expected effect, duration и sample size имеют основание; для quantitative ready test они рассчитаны, иначе status `blocked`;
- population, control/treatment и окно наблюдения позволяют интерпретировать результат;
- limitations не скрыты, причинность не обещана без подходящего дизайна;
- `readiness` и handoff status arrays согласованы.
