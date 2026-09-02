# CORRECT — аудит и исправление hypotheses

Используй этот prompt для существующих hypotheses, включая clean audit, исправление, блокировку или отклонение.

## Prompt для агента

Прочитай `README.md`, `SYSTEM_PROMPT.md`, `references/gpt3-intake-adapter.md`, `references/evidence-policy.md` и `skills/INDEX.md`. Используй `hypothesis-framing` и `experiment-design`; дополнительные skills — только по реальному framing defect. Перед ответом выполни `references/pre-output-checklist.md`.

Режим: `run_mode: correct`.

На входе ожидаются `existing_hypotheses`, GPT3 analysis/handoff, evidence registry или versioned registry ref, metric context и test constraints.

## Audit

Проверь для каждой hypothesis:

- сохранены ли `analysis_id`, `problem_id`, `cause_id`, causal status/basis, evidence for/against и confounders;
- разделены ли `if`, `then`, `because` и возможно ли наблюдаемое опровержение;
- не выдан ли mechanism за факт;
- разрешается ли каждый evidence ref и размечен ли он через `supports`/`stance`;
- не используется ли problem evidence как expected-effect evidence;
- вынесены ли assumptions;
- измеряет ли единственный `primary_metric` поле `then`;
- определены ли guardrails и breach actions;
- имеют ли expected effect, baseline, sample size и duration реальное основание;
- полны ли method-specific параметры test;
- соответствует ли status фактической readiness.

## Результат audit

- Если дефектов нет: `audit_outcome: no_change`, previous/resulting IDs одинаковы, `issues_found` и `changes_made` пусты.
- Если смысл не изменился: сохрани `hypothesis_id`, зафиксируй `audit_outcome: changed` и объясни правку.
- Если изменился intervention, основной outcome или mechanism: создай новый `hypothesis_id`, заполни `previous_hypothesis_id`, а revision entry покажет `previous_hypothesis_id → resulting_hypothesis_id`.
- Если данных недостаточно: `audit_outcome: blocked`, `status: blocked`, конкретные missing inputs/reasons; metric/test могут быть `null`.
- Если claim опровергнут, дублирует нерелевантную hypothesis или не должен тестироваться: `audit_outcome: rejected`, `status: rejected`, `rejection_reason`; не создавай искусственный test.

Не повышай `confidence` из-за переписывания. Обнови typed evidence links, readiness object и все handoff arrays. Верни Markdown по template либо JSON по schema.

