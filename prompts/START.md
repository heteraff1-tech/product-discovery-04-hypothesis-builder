# START — сформировать hypotheses с нуля

Используй этот prompt, когда GPT3 handoff содержит problem/cause analysis, но продуктовые hypotheses ещё не сформированы.

## Prompt для агента

Прочитай `README.md`, `SYSTEM_PROMPT.md`, `references/gpt3-intake-adapter.md`, `references/evidence-policy.md` и `skills/INDEX.md`. Выбери минимальный набор skills. Перед ответом выполни `references/pre-output-checklist.md`.

Режим: `run_mode: start`.

Ожидаемый upstream input:

```yaml
schema_version: "GPT3 schema version"
generated_at: "GPT3 generation timestamp"
analysis_id: "stable GPT3 analysis ID"
problem_definition:
  problem_id: "stable problem ID"
  statement: "observed problem"
  evidence_for_refs: ["E-..."]
  evidence_against_refs: ["E-..."]
candidate_causes:
  - cause_id: "C-..."
    hypothesis: "upstream causal hypothesis"
    mechanism: "upstream mechanism"
    causal_status: "upstream enum value"
    causal_basis_ref: null
    evidence_for_refs: ["E-..."]
    evidence_against_refs: ["E-..."]
    possible_confounders: []
causality_assessment:
  causal_basis: null
evidence_register: []
evidence_registry_ref: null
decision_handoff:
  problem_id: "same stable problem ID"
provenance:
  problem_id: "same stable problem ID"
  problem_id_origin: input
```

Дополнительно пользователь может передать `desired_outcome`, target segments, metric definitions, constraints, traffic и risk limits.

Сделай следующее:

1. Сохрани обязательные GPT3 fields verbatim в `upstream_intake`; при пробеле установи `adapter_status: incomplete`.
2. Создай `evidence_context` с embedded registry, versioned reference или `unavailable` без выдуманных registry metadata.
3. При необходимости создай evidence-backed JTBD frames и/или Opportunity Solution Tree.
4. Сформируй различающиеся hypotheses, каждая с одним ведущим mechanism.
5. Для evidence используй typed `evidence_links` с `supports`, `stance` и `claim_ref`. Не переноси problem evidence в expected-effect claim.
6. Заполни известные hypothesis/test fields; неизвестные оставь `null` и отрази в `missing_inputs`/`blocking_reasons`.
7. Пропусти каждую hypothesis через method-specific readiness gate. Только полностью исполнимые tests получают `ready_to_test`.
8. Синхронизируй handoff arrays со status всех hypotheses и передай versioned `evidence_registry_ref`.

Не утверждай, что hypothesis верна. Причинный status и basis GPT3 сохраняются без усиления.
