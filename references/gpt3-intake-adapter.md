# GPT3 → GPT4 Intake Adapter

Этот adapter принимает handoff от `product-discovery-03-root-cause-analyst` и сохраняет аналитическую lineage до построения гипотез. Он не «улучшает» upstream payload и не переводит значения в локальные синонимы.

## Неприкосновенные поля

Следующие значения переносятся verbatim:

- `analysis_id`;
- `problem_id`;
- `problem_id_origin`;
- `cause_id`;
- `causal_status`;
- `causal_basis_ref` и полный `causal_basis`, если он передан;
- `evidence_for_refs`;
- `evidence_against_refs`;
- `possible_confounders`;
- upstream `schema_version` как `source_schema_version`.
- upstream `generated_at` как `source_generated_at`.

Не заменяй `cause_id` на `root_cause_candidate_id`, не меняй регистр/префикс ID и не переводь enum values. Если локальная модель не знает новое значение `causal_status`, сохрани строку как получена и добавь note, а не подставляй ближайшее известное значение.

## Выход adapter

Заполни верхнеуровневый `upstream_intake`:

```yaml
upstream_intake:
  source_agent: product-discovery-03-root-cause-analyst
  source_schema_version: "upstream value"
  source_generated_at: "upstream value"
  adapter_status: complete
  analysis_id: "upstream value"
  problems:
    - problem_id: "upstream value"
      problem_id_origin: "input or generated_local"
      statement: "upstream statement"
      evidence_for_refs: ["upstream refs"]
      evidence_against_refs: ["upstream refs"]
  causes:
    - cause_id: "upstream value"
      hypothesis: "upstream value"
      mechanism: "upstream value"
      causal_status: "upstream value"
      causal_basis_ref: null
      evidence_for_refs: ["upstream refs"]
      evidence_against_refs: ["upstream refs"]
      possible_confounders: ["upstream values"]
  causal_basis: null
  missing_fields: []
  adaptation_notes: []
```

`statement`, `hypothesis` и `mechanism` также сохраняй без смыслового переписывания внутри intake snapshot. Более удобное framing создаётся дальше как новый объект и не заменяет upstream record.

## Точные GPT3 paths

- `analysis_id` ← `#/analysis_id`;
- `source_generated_at` ← `#/generated_at`;
- `problem_id` ← `#/problem_definition/problem_id`;
- problem evidence ← `#/problem_definition/evidence_for_refs` и `evidence_against_refs`;
- causes ← `#/candidate_causes[]` с теми же field names;
- `causal_basis` ← `#/causality_assessment/causal_basis`;
- `problem_id_origin` ← `#/provenance/problem_id_origin`.

Проверь, что `problem_definition.problem_id`, `decision_handoff.problem_id` и `provenance.problem_id` совпадают. Расхождение означает `adapter_status: incomplete`; не выбирай один ID молча.

Если `problem_id_origin: generated_local`, GPT3 формирует ID как `P-{normalized analysis_id}-01`, где неподдерживаемые символы заменены на `-`, а крайние `.`, `_`, `-` удалены. В GPT4 только проверь правило; не пересоздавай и не «исправляй» уже переданный ID.

## Evidence registry

Перенеси `evidence_register` GPT3 в `evidence_context.evidence_registry`, сохраняя `evidence_id`, observation, locator и source refs. Для embedded registry используй детерминированные local metadata: `registry_id: ER-{analysis_id}`, `version: {source_generated_at}` и `uri: #/evidence_context/evidence_registry`. Это ID нового registry artifact, а не замена upstream evidence IDs.

Если registry хранится отдельно, используй `mode: reference` и передай versioned external `evidence_registry_ref`. Ссылка обязана содержать минимум `registry_id`, `version`, `analysis_id` и `uri`.

Если registry отсутствует или `source_generated_at`/другая проверяемая version неизвестна, используй `mode: unavailable`, оставь registry/ref `null`, перечисли missing fields и не помечай ни одну hypothesis как `ready_to_test`.

## Derived claim-evidence links

После сохранения snapshot можно построить локальные `evidence_links`. Это производное представление, поэтому оно не изменяет upstream arrays:

- элемент `evidence_for_refs` становится link с `stance: supports`;
- элемент `evidence_against_refs` становится link с `stance: contradicts`;
- `supports` показывает тип claim: `problem`, `mechanism`, `baseline` или `expected_effect`;
- `claim_ref` указывает конкретный ID или JSON Pointer.

Нельзя автоматически переносить problem evidence в `supports: expected_effect`. Evidence существования проблемы не доказывает эффект будущего intervention.

## Поведение при неполном handoff

Если нет `analysis_id`, стабильного `problem_id`, нужного `cause_id`, evidence refs, causal status/basis или confounders:

1. Не создавай замещающий upstream ID или значение.
2. Установи `adapter_status: incomplete`.
3. Добавь точные JSON fields в `missing_fields` и `missing_inputs`.
4. Можно сохранить и исправить переданную hypothesis со `status: blocked` или `rejected`.
5. Нельзя выдавать результат как `ready_to_test`, пока links не разрешаются в registry/ref.
