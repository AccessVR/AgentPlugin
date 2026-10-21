# Character and Thread settings

All of these change the draft. Learners get them once the Character or Thread
is published, except where noted.

## Character settings

Set with `update-character-tool {id}` plus only the fields that change.
`get-character-tool {id}` reads every one of them back.

| Argument | What it does | Values |
| --- | --- | --- |
| `name`, `description` | Gallery name and card text. | text |
| `conversational_context` | The scenario the Persona is given for every conversation with this Character. | text |
| `custom_greeting` | The opening line. | text, or null for none |
| `custom_greeting_type` | `literal` speaks the greeting word for word. `prompt` treats it as an instruction to improvise a fresh greeting each time. | `literal`, `prompt` |
| `wait_for_participant_to_greet` | Stay silent until the learner speaks first. | true, false |
| `persona_demeanor_key` | Conversational manner. `standard` runs the Persona as configured. `reticent` is guarded, so the learner must draw them out. `assertive` drives hard and resists being derailed. `reflective` asks structured questions and gives thinking time. | `standard`, `reticent`, `assertive`, `reflective` |
| `max_call_duration` | Time limit for one conversation, in seconds. | 60 or more |
| `enable_recording` | Record conversations. | true, false |
| `visibility` | Who can run it. `private` (the default) is the organization's members only. `unlisted` is anyone with the link, including people without an account. | `private`, `public`, `unlisted`, `secret` |
| `is_active` | Whether the Character can be used at all. | true, false |
| `replica_id`, `persona_id` | Swap the Replica or the Persona. | ids |
| `background_asset_id` | An image Asset shown behind the Replica. Null goes back to the Replica's own backdrop. | image Asset id, or null |
| `conversation_report_type_id` | The Report Type that scores each conversation. Null removes it. | Report Type id, or null |
| `document_retrieval_strategy` | Trade-off when searching the Character's knowledge. | `speed`, `balanced`, `quality` |
| `prebrief_enabled`, `prebriefing_content`, `prebriefing_video_asset_id` | A screen shown before the conversation: HTML content, a video Asset, or both. | true/false, HTML, video Asset id |
| `survey_url`, `survey_description`, `survey_button_text` | A link to an outside survey offered after the conversation. | URL, text |
| `allows_subscriber_duplicates` | Let organizations that subscribe to this Character copy it. Takes effect at once, without publishing. | true, false |

### Contact collection for people without an account

Use these on an `unlisted` Character, where many learners have no
OrchestrateXR account.

| Argument | What it does |
| --- | --- |
| `contact_collection_enabled` | Ask for the learner's name and email before starting. |
| `contact_collection_required` | Don't let them start until they enter it. |
| `contact_send_reports` | Email the conversation report to the address they gave. |
| `contact_webhook_enabled`, `contact_webhook_url` | Post the collected email to a URL. A Character with the webhook on and no URL can't be published. |

`participant_absent_timeout` can be read on `get-character-tool` but only
OrchestrateXR support can change it.

## Thread settings

Set with `update-thread-tool {id}` plus the fields that change. The lineup is not set here; use
`set-thread-characters-tool`.

| Argument | What it does |
| --- | --- |
| `name`, `description`, `visibility` | As for a Character. |
| `conversation_report_type_id` | The Report Type that scores the whole run. Null removes it. |
| `prebrief_enabled`, `prebriefing_content`, `prebriefing_video_asset_id` | A prebrief shown before the first conversation. If a Character in the lineup has its own prebrief, both show, the Thread's first. |
| `survey_enabled`, `survey_url`, `survey_description`, `survey_button_text` | The end-of-Thread survey. |
| `contact_collection_enabled`, `contact_collection_required`, `contact_send_reports`, `contact_webhook_enabled`, `contact_webhook_url` | Contact collection, as for a Character. |
| `allows_subscriber_duplicates` | As for a Character. |

The demo settings on `update-thread-tool` are for OrchestrateXR support only.

## Lineup slot settings

Each entry in the `characters` list of `create-thread-tool` or
`set-thread-characters-tool`. Position comes from the order of the list.

| Field | What it does | Default |
| --- | --- | --- |
| `character_id` | The Character in this slot. Required. | |
| `auto_continue` | Move the learner straight to the next Character when this conversation ends. Off shows a Continue button instead. | on |
| `receives_context` | Give this Character the transcripts of the earlier conversations in the Thread. Off starts it fresh. | on |
| `show_prebrief` | Show this Character's own prebrief, if it has one. | on |
| `show_survey` | Offer this Character's own survey. Only the last slot can use it. | off |
| `time_limit_enabled`, `max_call_duration`, `show_countdown` | A time limit for this slot, in seconds, and whether the learner sees a countdown. | |

Slot settings belong to the Thread. Changing them takes `set-thread-characters-tool`
with the whole lineup, then `publish-thread-tool`.
