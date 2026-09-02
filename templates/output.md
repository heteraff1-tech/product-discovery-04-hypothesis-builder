# Hypothesis Builder Output

Шаблон синхронизирован с `schemas/output.schema.json` v2.0.0. Для `null` и пустых arrays явно покажи значение; не скрывай обязательное поле. Если JTBD/OST не применялись, используй `jtbd_frames: []` и `opportunity_solution_tree: null`.

## Run

- `schema_version`: `2.0.0`
- `run_mode`: `start` | `correct`
- `skills_used`: []
- `summary`:

## `upstream_intake`

- `source_agent`: `product-discovery-03-root-cause-analyst`
- `source_schema_version`:
- `source_generated_at`:
- `adapter_status`: `complete` | `incomplete`
- `analysis_id`:
- `missing_fields`: []
- `adaptation_notes`: []

### `problems`

Для каждой upstream problem:

- `problem_id`:
- `problem_id_origin`:
- `statement`:
- `evidence_for_refs`: []
- `evidence_against_refs`: []

### `causes`

Для каждой upstream cause:

- `cause_id`:
- `hypothesis`:
- `mechanism`:
- `cause_level`:
- `causal_status`:
- `causal_basis_ref`:
- `evidence_for_refs`: []
- `evidence_against_refs`: []
- `possible_confounders`: []
- `assumptions`: []
- `confidence`: object | `null`
  - `level`:
  - `rationale`:

### `causal_basis`

Передай upstream object verbatim либо `null`.

## `evidence_context`

- `mode`: `embedded` | `reference` | `unavailable`
- `registry_id`:
- `version`:
- `analysis_id`:

### `evidence_registry`

Embedded registry либо `null`. Для каждого entry:

- `evidence_id`:
- `kind`:
- `observation`:
- `source_refs`: []
- `locator`:
- `scope`: object | `null`
- `reliability`:
- `limitations`: []

### `evidence_registry_ref`

Versioned ref либо `null`:

- `registry_id`:
- `version`:
- `analysis_id`:
- `uri`:
- `content_hash`:

## `missing_inputs`

Для каждого пробела:

- `field`:
- `reason`:
- `impact`:
- `resolution`:

## `jtbd_frames`

Для каждого frame:

- `jtbd_id`:
- `status`: `evidence_backed` | `assumption`
- `target_segment`:
- `situation`:
- `motivation`:
- `desired_progress`:
- `job_statement`:
- `functional_job`:
- `emotional_job`:
- `social_job`:
- `current_alternatives`: []
- `evidence_links`: см. typed link format ниже
- `open_questions`: []

## `opportunity_solution_tree`

Object либо `null`:

- `desired_outcome`:
- `outcome_metric`:
- `opportunities`: []

Для каждой `opportunity`:

- `opportunity_id`:
- `status`: `evidence_backed` | `assumption`
- `statement`:
- `evidence_links`: []
- `jtbd_ids`: []
- `solution_candidates`:
  - `solution_candidate_id`:
  - `statement`:
  - `assumption_ids`: []
  - `hypothesis_ids`: []

## Typed `evidence_link`

Используй одинаковую структуру во всех `evidence_links`:

- `evidence_id`:
- `supports`: `problem` | `mechanism` | `baseline` | `expected_effect`
- `stance`: `supports` | `contradicts` | `context_only`
- `claim_ref`:
- `note`:

Не используй problem link как expected-effect link без отдельного основания.

## `hypotheses`

### `hypothesis_id`

- `previous_hypothesis_id`: ID | `null`
- `status`: `candidate` | `ready_to_test` | `blocked` | `rejected`
- `problem_ids`: []
- `cause_ids`: []
- `opportunity_ids`: []
- `jtbd_ids`: []
- `target_segment`:
- `if`:
- `then`:
- `because`:
- `evidence_links`: []
- `alternative_explanations`: []

#### `assumptions`

Для каждого assumption:

- `assumption_id`:
- `statement`:
- `type`: `desirability` | `usability` | `feasibility` | `viability` | `measurement` | `causal`
- `risk_level`: `low` | `medium` | `high`
- `validation_status`: `untested` | `partially_supported` | `supported` | `refuted`
- `evidence_links`: []

#### `falsification_condition`

Строка либо `null` для incomplete/rejected claim.

#### `primary_metric`

Object либо `null`:

- `name`:
- `definition`:
- `event_or_formula`:
- `direction`: `increase` | `decrease` | `maintain` | `null`
- `baseline`:
- `target`:
- `unit`:
- `measurement_window`:
- `data_source`:

#### `guardrails`

Массив, который может быть пустым для incomplete/rejected claim. Для каждого guardrail:

- `name`:
- `definition`:
- `direction`: `not_increase` | `not_decrease` | `stay_within` | `null`
- `threshold`:
- `unit`:
- `measurement_window`:
- `breach_action`:

#### `expected_effect`

Object либо `null`:

- `direction`: `increase` | `decrease` | `maintain` | `null`
- `range.min`:
- `range.max`:
- `range.unit`:
- `basis`: `observed_baseline` | `historical_data` | `benchmark` | `calculation` | `user_input` | `assumption` | `unknown`
- `rationale`:

При `basis: unknown` обе границы равны `null`.

#### `test`

Object либо `null`:

- `test_id`:
- `method`:
- `riskiest_assumption_ids`: []
- `design`:
- `population`:
- `recruitment_criteria`:
- `control`:
- `treatment`:
- `allocation`:
- `instrumentation`:
- `duration.value`:
- `duration.unit`:
- `duration.rationale`:
- `sample_size.total`:
- `sample_size.per_variant`:
- `sample_size.basis`:
- `statistical_parameters.alpha`:
- `statistical_parameters.power`:
- `statistical_parameters.minimum_detectable_effect`:
- `success_criteria`: []
- `falsification_criteria`: []
- `stop_conditions`: []
- `analysis_plan`:
- `limitations`: []

`duration`, `sample_size` и `statistical_parameters` по отдельности могут быть `null`, если неприменимы или неизвестны. Для quantitative `ready_to_test` они должны удовлетворять method-specific checklist.

#### `confidence`

Object либо `null`:

- `level`: `low` | `medium` | `high` | `unknown`
- `rationale`:

#### `readiness`

- `ready`: true | false
- `checks.lineage_complete`: `pass` | `fail` | `not_assessed`
- `checks.evidence_resolvable`: `pass` | `fail` | `not_assessed`
- `checks.framing_complete`: `pass` | `fail` | `not_assessed`
- `checks.primary_metric_defined`: `pass` | `fail` | `not_assessed`
- `checks.guardrails_defined`: `pass` | `fail` | `not_assessed`
- `checks.method_requirements_met`: `pass` | `fail` | `not_assessed`
- `checks.instrumentation_available`: `pass` | `fail` | `not_assessed`
- `checks.population_access_confirmed`: `pass` | `fail` | `not_assessed`
- `blocking_reasons`: []

Для `ready_to_test` все checks равны `pass`, `ready: true`, reasons пусты. Для `blocked` — `ready: false` и минимум одна причина.

#### `rejection_reason`

Строка для `rejected`, иначе `null`.

## `revision_log`

В `start` используй `[]`. В `correct` добавь entry для каждой проверенной hypothesis, включая clean no-op:

- `audit_outcome`: `no_change` | `changed` | `blocked` | `rejected` | `superseded`
- `previous_hypothesis_id`:
- `resulting_hypothesis_id`:
- `issues_found`: []
- `changes_made`: []
- `reason`:
- `preserved_evidence_refs`: []

При смене mechanism/intervention/outcome previous и resulting IDs различаются.

## `limitations`

Массив ограничений evidence, scope и test design.

## `handoff`

- `analysis_id`:
- `evidence_registry_ref`: полный versioned object | `null`
  - `registry_id`:
  - `version`:
  - `analysis_id`:
  - `uri`:
  - `content_hash`:
- `ready_hypothesis_ids`: []
- `candidate_hypothesis_ids`: []
- `blocked_hypothesis_ids`: []
- `rejected_hypothesis_ids`: []
- `decision_notes`: []

Четыре status arrays должны покрывать все hypotheses ровно один раз.
