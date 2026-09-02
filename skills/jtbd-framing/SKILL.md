---
name: jtbd-framing
description: "Переформулирует evidence-backed пользовательскую ситуацию в Jobs to be Done и связывает её с opportunity и гипотезой. Используй при наличии данных о контексте, мотивации и желаемом прогрессе; не используй для выдумывания jobs из feature request без исследования."
---

# JTBD Framing

Цель skill — понять прогресс, которого пользователь пытается достичь в конкретной ситуации, и использовать этот контекст для более точной hypothesis. JTBD не является красивой заменой persona: каждое содержательное поле должно быть связано с research evidence.

Перед работой прочитай [`../../references/gpt3-intake-adapter.md`](../../references/gpt3-intake-adapter.md) и [`../../references/evidence-policy.md`](../../references/evidence-policy.md). После framing передай результат в [`../hypothesis-framing/SKILL.md`](../hypothesis-framing/SKILL.md).

## Необходимый вход

Ищи в интервью, наблюдениях, тикетах или других материалах:

- ситуацию или trigger, в которой возникает задача;
- текущий способ решения и workaround;
- желаемый функциональный результат;
- выраженные пользователем барьеры, тревоги или критерии выбора;
- разрешимые upstream evidence refs с locators;
- сегмент, определённый обстоятельствами и поведением, а не только демографией.

Если данных хватает только на функциональный job, не достраивай emotional и social dimensions. Если нет ситуации или желаемого progress, добавь `missing_inputs` и не называй frame подтверждённым.

## Формирование frame

1. **Определи performer и circumstance.** Сегмент должен объяснять, кто выполняет job и в каком контексте он возникает.
2. **Сформулируй job statement.** Используй структуру: `When [situation], I want [motivation/action], so I can [desired progress]`.
3. **Отдели уровни.** Зафиксируй `functional_job`; добавь `emotional_job` или `social_job` только при прямом основании.
4. **Опиши current alternatives.** Включай текущий workaround, конкурирующий продукт и non-consumption, только если они встречаются в evidence.
5. **Выдели desired outcomes.** Формулируй результат независимо от конкретной реализации. Feature request может быть evidence неудовлетворённости, но не равен job.
6. **Привяжи источники.** Добавь typed `evidence_links` ко всему frame и не переноси цитату между сегментами без основания. Обычно JTBD link поддерживает `problem` или `mechanism`, но не числовой `expected_effect`.
7. **Оцени статус.** Используй `evidence_backed`, когда ключевые элементы опираются на источники; `assumption`, когда frame нужен как вопрос для проверки.

## Переход к гипотезе

Используй JTBD для уточнения:

- `target_segment` и ситуации в `if`;
- пользовательского поведения или progress в `then`;
- предполагаемого механизма в `because`;
- desirability assumptions и выбора раннего test.

Не делай вывод «этот job существует у рынка» по одному яркому интервью. Масштаб claim должен соответствовать покрытию evidence.

## Выход

Заполни `jtbd_frames`:

- `jtbd_id`;
- `status`;
- `target_segment`;
- `situation`, `motivation`, `desired_progress`;
- `job_statement`;
- `functional_job`, при наличии `emotional_job` и `social_job`;
- `current_alternatives`;
- `evidence_links`;
- `open_questions`.

Свяжи frame с `opportunity_ids` или напрямую с hypothesis через соответствующие ID, но не дублируй evidence текстом без необходимости.

## Quality gate

- job описывает progress, а не продуктовую функцию;
- circumstance достаточно конкретна, чтобы отличить релевантный сегмент;
- emotional/social fields не выведены из стереотипов;
- каждый link разрешается в конкретной версии registry и имеет claim type/stance;
- feature request сохранён как наблюдение, но не принят за решение;
- uncertainty и open questions видимы;
- JTBD уточняет hypothesis, а не делает её автоматически истинной.
