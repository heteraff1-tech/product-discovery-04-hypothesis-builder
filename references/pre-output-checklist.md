# Pre-output Checklist

Этот checklist проверяет межссылочные и method-specific инварианты, которые намеренно не зашиты в громоздкую JSON Schema. Выполни его перед финальным ответом.

## 1. Upstream lineage

- `analysis_id`, `problem_id`, `problem_id_origin`, `cause_id` и upstream enum values сохранены verbatim.
- `problem_definition.problem_id`, `decision_handoff.problem_id` и `provenance.problem_id` GPT3 совпадают.
- Embedded registry version совпадает с сохранённым `source_generated_at` либо с явно рассчитанным content hash/version; версия не придумана.
- Каждый `problem_id` в hypothesis существует в `upstream_intake.problems`.
- Каждый `cause_id` существует в `upstream_intake.causes`.
- `evidence_for_refs`, `evidence_against_refs`, `causal_status`, `causal_basis` и `possible_confounders` не потеряны и не усилены редакционно.
- `adapter_status: incomplete` используется при любом обязательном пробеле; значения не додумываются.

## 2. Evidence resolution and meaning

- Каждый `evidence_link.evidence_id` разрешается в embedded registry или в указанной версии external registry.
- В link заполнены `supports`, `stance`, `claim_ref` и объяснение связи.
- `supports: problem` не используется как доказательство `expected_effect`.
- Evidence механизма, baseline и expected effect размечены отдельными links.
- Противоречащие evidence имеют `stance: contradicts`, а не исчезают из результата.

## 3. Hypothesis status

`ready_to_test` допустим только если:

- `if`, `then`, `because`, segment и falsification condition конкретны;
- problem link разрешён; непроверенный mechanism явно остаётся causal assumption;
- выбран ровно один операционно определённый `primary_metric`;
- guardrails имеют measurement rule и breach action;
- выбран method и заполнены все обязательные для него параметры;
- criteria success/falsification/stop заданы до запуска;
- instrumentation и доступ к population подтверждены;
- `readiness.ready` и все readiness checks равны `true`;
- `readiness.blocking_reasons` пуст;
- evidence registry доступен через embedded content или versioned ref.

Для quantitative controlled test дополнительно нужны baseline/variance или rate, MDE/expected range, alpha/power, рассчитанный sample size, duration, allocation, control и treatment. Если их нет — `blocked`, а не числа «на глаз».

Для qualitative/prototype test нужны recruitment criteria, observable task/outcome, заранее заданный stopping rule и предел вывода. Такой test не подтверждает population-level lift.

`blocked` требует минимум одну конкретную `blocking_reason`; незаполненные поля могут быть `null`. `rejected` требует `rejection_reason`, но не требует придумывать metric, guardrail или test. `candidate` может быть содержательно оформлен, но не включается в ready handoff.

## 4. CORRECT and lineage

- Clean audit допустим: `audit_outcome: no_change`, одинаковые `previous_hypothesis_id` и `resulting_hypothesis_id`, пустые `issues_found`/`changes_made`.
- Если изменена только ясность без смены intervention/outcome/mechanism, стабильный ID сохраняется.
- Если сменился механизм (`because`), intervention (`if`) или основной outcome (`then`), создаётся новый `hypothesis_id`; его `previous_hypothesis_id` указывает на старый.
- Revision entry явно показывает `previous_hypothesis_id → resulting_hypothesis_id`.
- Переформулировка не повышает `confidence` без нового evidence.

## 5. Handoff consistency

- `ready_hypothesis_ids`, `candidate_hypothesis_ids`, `blocked_hypothesis_ids` и `rejected_hypothesis_ids` вместе покрывают все hypotheses ровно один раз.
- ID находится в массиве, соответствующем его `status`.
- `evidence_registry_ref` в handoff совпадает с `evidence_context.evidence_registry_ref` либо равен `null`, когда registry unavailable.
- `jtbd_ids`, `opportunity_ids`, assumption IDs и test refs разрешаются внутри результата.
- Handoff не содержит RICE/ICE ranking или финального продуктового решения.

## 6. No fabrication

- Не создано ни одного source/evidence/upstream ID без реального объекта или точного locator.
- Неизвестные значения представлены `null`, `unknown`, `missing_inputs` или `blocking_reasons`.
- `blocked` и `rejected` объекты остаются валидными без искусственного заполнения test fields.
- Язык hypothesis остаётся условным до релевантного результата.
