# Product Discovery 04 — Hypothesis Builder

Репозиторий задаёт рабочий контур GPT, который принимает диагностический handoff от GPT3 и превращает подтверждённые проблемы и кандидаты причин в фальсифицируемые продуктовые гипотезы. Агент сохраняет upstream lineage, разделяет evidence по типу claim, проектирует тест и заранее определяет условия опровержения. Гипотеза не становится фактом до релевантного результата.

## Граница роли

```text
GPT3 analysis + evidence registry
                ↓ verbatim intake adapter
problem / cause candidate / confounders
                ↓ optional JTBD or OST framing
falsifiable hypothesis
                ↓ semantic readiness gate
test-ready | candidate | blocked | rejected
```

Агент не должен:

- переименовывать `analysis_id`, `problem_id`, `cause_id` или upstream enum values;
- усиливать `causal_status` или терять `causal_basis`, evidence against и confounders;
- придумывать source/evidence IDs, baseline, expected effect, sample size или duration;
- считать problem evidence доказательством механизма или эффекта решения;
- заполнять test fields ради прохождения schema для `blocked`/`rejected`;
- ранжировать hypotheses методом RICE/ICE или принимать решение вместо GPT5.

## Порядок чтения

1. Прочитать [`SYSTEM_PROMPT.md`](SYSTEM_PROMPT.md).
2. Выбрать режим:
   - новая работа → [`prompts/START.md`](prompts/START.md);
   - аудит/исправление → [`prompts/CORRECT.md`](prompts/CORRECT.md).
3. Применить [`references/gpt3-intake-adapter.md`](references/gpt3-intake-adapter.md).
4. Прочитать [`references/evidence-policy.md`](references/evidence-policy.md).
5. Открыть [`skills/INDEX.md`](skills/INDEX.md) и прочитать только выбранные skills.
6. Перед выдачей пройти [`references/pre-output-checklist.md`](references/pre-output-checklist.md).
7. Вернуть Markdown по [`templates/output.md`](templates/output.md) либо JSON по [`schemas/output.schema.json`](schemas/output.schema.json).

## GPT3 → GPT4 intake

В `upstream_intake` сохраняются verbatim:

- `source_schema_version`, `source_generated_at` и `analysis_id`;
- стабильные `problem_id`, `problem_id_origin` и `cause_id`;
- `causal_status`, `causal_basis_ref` и переданный `causal_basis`;
- `evidence_for_refs` и `evidence_against_refs`;
- `possible_confounders`;
- исходные problem/cause statements и mechanism.

Не заменяй `cause_id` локальным `root_cause_candidate_id`. Если поле отсутствует, используй `adapter_status: incomplete`, перечисли `missing_fields` и не создавай замещающее значение.

Перед продолжением проверь равенство `problem_definition.problem_id`, `decision_handoff.problem_id` и `provenance.problem_id` GPT3.

## Evidence registry

Результат всегда содержит `evidence_context` с одним из режимов:

- `embedded`: registry включён в `evidence_registry`, а versioned `evidence_registry_ref.uri` указывает на него;
- `reference`: registry не дублируется, но передан versioned external `evidence_registry_ref`;
- `unavailable`: оба значения `null`, пробел отражён в `missing_inputs`, все hypotheses остаются неготовыми.

`evidence_registry_ref` также передаётся в `handoff`. Это позволяет следующему агенту разрешить evidence links в конкретной версии, а не искать «похожий» источник.

## Typed claim-evidence links

В локальных artifacts используй `evidence_links`, а не плоский список без семантики. Каждый link содержит:

- `evidence_id`;
- `supports`: `problem`, `mechanism`, `baseline` или `expected_effect`;
- `stance`: `supports`, `contradicts` или `context_only`;
- `claim_ref`: ID или JSON Pointer конкретного claim;
- `note`: почему источник релевантен.

Upstream `evidence_for_refs`/`evidence_against_refs` остаются неизменными внутри `upstream_intake`; typed links являются производным слоем.

## Router

| Ситуация | Обязательные skills | Дополнительный skill |
|---|---|---|
| Problem/cause → hypothesis/test | `hypothesis-framing`, `experiment-design` | — |
| Несколько opportunities и solutions | `hypothesis-framing`, `experiment-design` | `opportunity-solution-tree` |
| Feature request скрывает пользовательский progress | `hypothesis-framing`, `experiment-design` | `jtbd-framing`, только при research evidence |
| Полная карта outcome → opportunity → solution → hypothesis | `hypothesis-framing`, `experiment-design` | `opportunity-solution-tree`, при необходимости `jtbd-framing` |
| Аудит существующей hypothesis | `hypothesis-framing`, `experiment-design` | дополнительные skills только по обнаруженному framing defect |

## Hypothesis contract

Полная hypothesis может включать `if`, `then`, `because`, `assumptions`, `falsification_condition`, `primary_metric`, `guardrails`, `expected_effect`, `test` и `confidence`. Для `blocked`/`rejected` неизвестные или неприменимые блоки остаются `null`/пустыми: schema не требует fabrication.

`ready_to_test` — не редакционная оценка. Он допустим только после semantic readiness gate: evidence разрешается, metric/test определены, method-specific параметры полны, условия успеха/опровержения/остановки заданы, instrumentation и population доступны.

## CORRECT lineage

Clean no-op audit допустим и фиксируется как `audit_outcome: no_change`. Если смысл hypothesis не изменился, ID сохраняется. Если изменились intervention, основной outcome или mechanism, создаётся новый ID и явно записывается lineage `previous_hypothesis_id → resulting_hypothesis_id`.

## Handoff

`handoff` содержит раздельные массивы:

- `ready_hypothesis_ids`;
- `candidate_hypothesis_ids`;
- `blocked_hypothesis_ids`;
- `rejected_hypothesis_ids`;
- versioned `evidence_registry_ref` или `null`;
- `decision_notes` без скрытой приоритизации.

Массивы должны покрывать все hypotheses ровно один раз.
