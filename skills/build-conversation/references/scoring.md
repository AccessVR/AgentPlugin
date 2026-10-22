# Report Types and Measurables

A Report Type is a rubric. Each Measurable is one criterion on it. Every report
also carries a set of always-on Measurables the platform provides; those can't
be edited.

## Report Type fields

`create-report-type-tool` and `update-report-type-tool` take:

| Argument | What it does | Default |
| --- | --- | --- |
| `name`, `description` | Name and gallery card text. | |
| `system_prompt` | Drives the written narrative of the report. Leave it out (or set null) for a measurables-only report. | none |
| `include_perception_analysis` | Feed the analysis of how the learner came across on camera into the report, and show it as a section. | true |
| `exclude_demographics_analysis` | Rewrite that analysis to remove anything revealing race, gender or sexual orientation, keeping mood, attitude and posture. | false |
| `active` | Update only, and only OrchestrateXR support staff can change it. Whether it can be newly attached to a Character or Thread; off also stops it being sold. Content already using it keeps it. | true |

## Measurable fields

`create-measurable-tool` and `update-measurable-tool` take:

| Argument | What it does |
| --- | --- |
| `conversation_report_type_id` | Create only. The Report Type it belongs to. Required. |
| `label` | What the criterion is called in the report. Required on create. |
| `key` | The export column name, in snake_case. Derived from the label when left out. It names the column forever, so don't change it once results exist. |
| `definition` | The criterion the model applies, in plain words. |
| `help_text` | Shown to the person reading the report. Never sent to the model. |
| `additional_guidance` | Extra nuance for the model. It can't override the value type or the need for evidence. |
| `applicability_rule` | When the criterion does not apply. Leave out if it always applies. |
| `source_type` | Where the value comes from. See below. Default `llm_transcript`. |
| `value_type` | The shape of the answer. See below. Default `binary`. |
| `value_config` | Settings for the value type. |
| `computed_metric` | Which built-in metric, when `source_type` is `computed`. |
| `derived_formula` | A formula over other Measurables, when `source_type` is `derived`. |
| `position`, `active` | Update only. Place in the order, and whether it is scored at all. |

### Value types

| `value_type` | Answer | Roll-ups | `value_config` |
| --- | --- | --- | --- |
| `binary` | yes or no | pass rate, distribution | none |
| `ordinal` | a point on a scale | median, distribution, approximate mean | `{"min":1,"max":5}`, which is also the default |
| `count` | a whole number | mean, sum, distribution | none |
| `categorical` | one of a set of labels | distribution only | `{"members":["A","B"]}` |
| `percentage` | a percentage | mean, distribution | none |

Prefer `binary`. Raters agree on yes/no criteria far more often, and they roll up
into clean pass rates. Ordinal scales are approximations, and percentages make
models clump at round numbers. A vague quality ("shows empathy, 1 to 5") is
usually better split into two or three yes/no criteria ("acknowledges the
patient's worry before answering").

### Source types

| `source_type` | Value comes from |
| --- | --- |
| `llm_transcript` | The model reading the transcript. |
| `llm_perception` | The model reading the perception analysis. |
| `computed` | A built-in metric, named in `computed_metric`: `duration_seconds`, `turn_count`, `learner_talk_ratio`, `mean_learner_utterance_length` or `goals_achieved`. |
| `derived` | Other Measurables on the same Report Type, combined by `derived_formula`, for example `{"op":"mean","inputs":["q1","q2"]}`. The operations are `mean`, `median`, `sum`, `rate`, `count_where`, `weighted_mean` and `threshold_map`. |

## Changing a Measurable that has results

Read `has_results` with `get-measurable-tool {id}` first. Once results exist,
the criterion is part of a time series. Rewording the label or help text is
fine. Changing the `definition`, `value_type` or `key` changes what past scores
meant, so take it to the human before making it.
