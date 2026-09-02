# Evidence Policy

Политика сохраняет проверяемую цепочку `GPT3 analysis → evidence → problem/cause → hypothesis → test` и запрещает смешивать наблюдения, выводы и ожидания.

## Claim types

| Тип | Значение |
|---|---|
| `evidence` | Доступное наблюдение, цитата, событие или рассчитанная метрика с locator |
| `inference` | Интерпретация, совместимая с evidence, но не наблюдавшаяся напрямую |
| `cause_candidate` | Возможный mechanism проблемы с сохранённым upstream `causal_status` |
| `assumption` | Непроверенная предпосылка hypothesis/test |
| `hypothesis` | Фальсифицируемое ожидание будущего эффекта |
| `result` | Результат завершённого test в пределах его design |

Риторика не повышает claim type. Cause candidate не становится установленной причиной, а hypothesis — фактом без релевантного result.

## Upstream preservation

Перед применением этой политики выполни [`gpt3-intake-adapter.md`](gpt3-intake-adapter.md). Upstream `analysis_id`, `problem_id`, `cause_id`, `causal_status`, `causal_basis`, `evidence_for_refs`, `evidence_against_refs` и `possible_confounders` должны оставаться verbatim.

Локальное framing создаёт новые объекты и links, но не редактирует intake snapshot. Любое несоответствие между новым claim и upstream cause показывается явно.

## Evidence registry

Evidence считается разрешимым, если выполняется одно из условий:

- объект с тем же `evidence_id` присутствует в `evidence_context.evidence_registry`;
- versioned `evidence_registry_ref` указывает на доступный registry, содержащий этот ID.

Registry entry должен сохранять как минимум `evidence_id`, observation, source refs и locator. Embedded registry получает `registry_id`, `version`, `analysis_id` и внутренний URI. External reference получает те же identity fields и URI. Если этих данных нет, используй `mode: unavailable`, а не выдуманный ref.

Стабильные GPT3 evidence IDs не переименовываются. Новый локальный evidence ID допустим только для реально предоставленного материала с точным locator; он добавляется в новую версию registry и не маскируется под upstream evidence.

## Typed `evidence_links`

Каждый локальный link содержит:

- `evidence_id`: разрешимый ID;
- `supports`: тип связанного claim — `problem`, `mechanism`, `baseline`, `expected_effect`;
- `stance`: `supports`, `contradicts`, `context_only`;
- `claim_ref`: stable ID или JSON Pointer;
- `note`: краткое объяснение релевантности и ограничения.

Примеры различий:

- падение activation подтверждает `problem`, но не будущий `expected_effect`;
- интервью может поддерживать возможный `mechanism`, но обычно не causal magnitude;
- историческая метрика поддерживает `baseline`, если scope совпадает;
- завершённый сопоставимый experiment может поддерживать `expected_effect`, но только с оговоркой переносимости.

Не копируй один link во все claim types. Если evidence действительно относится к нескольким claims, создай отдельные links с отдельными notes.

Upstream `evidence_for_refs`/`evidence_against_refs` остаются исходными arrays. При производном mapping `for` обычно получает `stance: supports`, `against` — `stance: contradicts`, но `supports` определяется смыслом конкретного claim, а не названием upstream массива.

## Traceability

```text
registry/version
    → evidence_id
    → typed evidence_link
    → problem_id | cause_id | hypothesis claim pointer
    → assumption_id
    → test_id
```

Каждая неотклонённая hypothesis должна ссылаться минимум на evidence существования исходной проблемы. Отсутствие evidence механизма допустимо как явно обозначенное causal assumption, но снижает confidence. Отсутствие разрешимого problem evidence исключает `ready_to_test`.

## Confidence

Оценивай `confidence.level` по прямоте, независимости, покрытию scope, качеству измерения, противоречиям и силе design:

- `low`: evidence косвенное/узкое, registry неполон или mechanism в основном assumed;
- `medium`: несколько согласованных сигналов с понятными ограничениями, причинность/переносимость не установлены;
- `high`: evidence прямо соответствует claim и получено подходящим воспроизводимым методом. Для expected effect это обычно требует валидного experiment.

Всегда добавляй rationale. Число links и качество формулировки сами по себе confidence не повышают.

## Counterevidence and confounders

- Сохраняй links со `stance: contradicts`.
- Не усредняй конфликтующие segments; сузь scope или раздели hypotheses.
- Переноси `possible_confounders` из GPT3 без удаления.
- Добавляй alternative explanations, если test не различает их.
- Отсутствие найденного возражения не равно evidence отсутствия возражения.

## Numeric claims

Baseline, target, effect range, sample size и duration могут иметь только явное basis: observed data, historical data, benchmark с оговоркой, calculation, user input или named assumption.

Если basis отсутствует:

- значение остаётся `null`;
- пробел появляется в `missing_inputs` или `blocking_reasons`;
- количественный controlled test остаётся `blocked`, если без значения нельзя определить MDE/power/duration;
- для `rejected` не создаются metric/test objects только ради полноты.

Unit должна различать percentage points и relative percent.

## Method limits

- Interview/usability evidence поддерживает claims о понимании, мотивации или friction, но не population lift.
- Observational analysis показывает association, если confounding не устранён.
- Randomized test поддерживает causal claim только для своей population, treatment, окна и корректного исполнения.
- `ready_to_test` означает готовность плана, а не истинность hypothesis.

## Prohibited transformations

- создавать или «улучшать» цитаты;
- генерировать upstream/registry IDs без реальных объектов;
- заменять точный locator общей ссылкой;
- терять evidence against или confounders;
- использовать problem evidence как expected-effect evidence;
- заполнять неизвестные числа типовыми значениями;
- требовать test/metric у rejected hypothesis;
- повышать causal status/confidence после редакционной правки.

Финальные межссылочные и readiness-проверки находятся в [`pre-output-checklist.md`](pre-output-checklist.md).

