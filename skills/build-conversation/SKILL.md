---
name: build-conversation
description: Set up an AI conversation in OrchestrateXR for training, assessment, sales practice or any role-play. Covers choosing a Replica, writing the Persona, creating the Character, scoring it with a Report Type and Measurables, adding Goals, attachments and a prebrief video, stringing Characters into a Thread, and publishing in the right order. Use when someone wants a new Character or Thread, wants to change how conversations are scored, or wants to copy a Character, Persona, Thread or Report Type as a starting point.
metadata:
  requires: authoring
---

# Build a conversation

A conversation is built from a few records. A **Replica** is the on-screen
person. A **Persona** is how that person thinks and behaves. A **Character**
pairs one Replica with one Persona and adds the scenario. A **Thread** plays
several Characters in order. A **Report Type**, made of **Measurables**, scores
a Character's conversations or a whole Thread run. **Goals** are targets a
learner can meet during a conversation with a Character.

Every `list-`, `get-` and `search-` tool named here runs with
execute_read_tools. Every other tool runs with execute_write_tools.

## Before you start

- **Organization.** The create tools take `organization` (id or exact name).
  If the person belongs to more than one organization and hasn't said which,
  ask, then pass it on every create. Everything a Character or Thread uses must
  belong to the same organization or be stock. A Replica, Persona, Report Type
  or Asset from another organization is refused.
- **Confirm every write.** Say which record and what changes, and get a yes.
  Before publishing or generating a video, restate the record by name.
- **Draft, then publish.** Edits change the draft. Learners keep the published
  version until the matching publish tool runs.
- **Links.** Share only URLs a tool returned: `url` to work on a record,
  `share_url` to send it. Never build one from an id.
- **"Wasn't found, or isn't available."** Check the id with the matching list
  tool. Don't repeat the call.

## 1. Copy or start fresh

Ask whether an existing Character, Persona, Thread or Report Type is close to
what they want. If one is, read `references/copying.md`: it covers the
duplicate tools and what a copy keeps and leaves behind.

## 2. Pick a Replica

`list-replicas-tool {search?, sex?, age_group?, ethnicity?}` lists stock
Replicas and your organization's own. `get-replica-tool {id}` shows its training
status and thumbnails. A Replica still being prepared can be put on a Character,
but the Character can't be published until it's ready.

The Voice a Persona speaks in is set by OrchestrateXR support.
`list-voices-tool {search?}` shows the Voices available. If the person wants a
particular Voice on the Persona, offer to file a ticket with
`create-support-ticket-tool {subject, description}`.

## 3. Write the Persona

Reuse one when it fits (`list-personas-tool {search}`). Otherwise agree the prompt with the human, then call
`create-persona-tool {organization?, name, system_prompt, description?, default_replica_id?}`.

- `system_prompt` says who the person is and how they behave. Leave out the
  platform's guardrails; they are added automatically.
- Put the scenario on the Character's `conversational_context`, not in the
  Persona. One Persona can then back several Characters.
- A new Persona is live from the start. Later edits with
  `update-persona-tool {id, system_prompt?, name?, description?}` stay in the
  draft until `publish-persona-tool {id}`, which reaches every Character on
  that Persona at its next conversation.

## 4. Create the Character

`create-character-tool {organization?, name, replica_id, persona_id, conversational_context, custom_greeting?, custom_greeting_type?, conversation_report_type_id?}`

- `custom_greeting_type` is `literal` (spoken word for word, the default) or
  `prompt` (an instruction to improvise a greeting).
- It starts as a private, unpublished draft.
- Set time limits, recording, visibility, contact collection and surveys with
  `update-character-tool`. Read `references/settings.md` for every setting.
- `update-character-tool` has no `goals` argument. Goals have their own tools
  (step 6).

## 5. Score it

Without a Report Type, conversations produce no report. Decide where it goes:

- On the Character, it scores every conversation with that Character, inside a
  Thread or not.
- On the Thread, it scores the whole run. Without one, the run gets an unscored
  summary of its Characters' own reports.

1. Look for one to reuse: `list-report-types-tool {search?}`, then compare
   criteria with `get-report-type-tool {id}`.
2. `create-report-type-tool {organization?, name, description?, system_prompt?}`.
   `system_prompt` drives the written narrative. Leave it out for a
   measurables-only report.
3. Agree the list of criteria with the human. Add each one with
   `create-measurable-tool {conversation_report_type_id, label, definition, value_type?, applicability_rule?, help_text?}`.
   Prefer a binary value type (the default): yes/no criteria agree best between
   raters and roll up into clean pass rates. Write `definition` so it can be
   checked from the transcript. Read `references/scoring.md` for the other
   value types and fields.
4. Attach it: `update-character-tool {id, conversation_report_type_id}` or
   `update-thread-tool {id, conversation_report_type_id}`, or pass it on create.
5. `publish-report-type-tool {id}`. Until a Report Type is first published, it
   scores with its draft. After that, edits count only once it is published
   again. Publishing changes scoring for everyone using it, so restate its name.
6. Publishing scores new conversations only. To score past ones under the
   changed criteria, offer `run-conversation-report-tool {ids}`: read "Score
   Conversations again" in the `review-results` skill.

Before changing a Measurable with `update-measurable-tool`, read "Changing a
Measurable that has results" in `references/scoring.md`.

## 6. Add Goals

Goals sit on Characters, not Threads. Search first with `list-goals-tool {search}`:
reusing a Goal keeps results comparable across Characters.

- New: `create-goal-tool {name, criteria, character_id?, show_on_goals_list?}`.
  With `character_id` it is assigned in the same call. Write `criteria` as
  something the learner says ("names three vital signs that changed"), never
  "communicates well".
- Existing: `assign-goals-tool {id, goals}`, where `id` is the Character and
  each entry is `{goal_id, show_on_goals_list?, celebration_message?}`.
- Reorder or adjust: `update-goal-assignment-tool {id, goal_id, position?, show_on_goals_list?}`.
  Remove: `unassign-goals-tool {id, goal_ids}`.

A Goal is hidden from the learner's Goals list by default, so they discover it
by meeting it. Learners meet Goals once the Character is published.

## 7. Prebrief video and attachments

Read `references/prebrief-and-attachments.md` before either. A prebrief video
made with `generate-replica-video-tool` spends organization tokens, so restate
the Replica, voice and script and get a yes first. Attachments are links or
files handed to the learner during the conversation; they reach the next
conversation without publishing.

## 8. Build the Thread

`create-thread-tool {organization?, name, description?, conversation_report_type_id?, thumbnail_asset_id?, characters?}`.
`characters` is the ordered lineup; each entry is `{character_id, ...}` with
the slot settings in `references/settings.md`. The same Character can appear
twice.

`thumbnail_asset_id` is the image on the Thread's card. Without one, the card
shows the first Character's Replica. Set or change it later with
`update-thread-tool {id, thumbnail_asset_id}`; pass null to remove it. It must
be an image asset (`list-assets-tool {asset_type: "image"}`, or one made with
the `add-assets` skill once it has finished generating), and it applies at
once, without publishing.

To change the lineup later, read the current one with `get-thread-tool {id}`,
then send the complete new list to `set-thread-characters-tool {id, characters}`.
It replaces the lineup wholesale: any entry left out is removed.

## 9. Publish in order

1. Persona: `publish-persona-tool {id}`, only if it was edited after creation.
2. Report Type: `publish-report-type-tool {id}` once the criteria are agreed.
3. Character: `publish-character-tool {id}`. An incomplete Character is refused
   with a list of problems: a Replica or Persona still being prepared, a
   prebrief with nothing to show, a webhook with no URL. Fix them and try again.
   Relay any warnings.
4. Thread: `publish-thread-tool {id}`. It is refused until every Character in
   the lineup is published. Publishing pins each Character's current published
   version, so after republishing a Character, republish every Thread that
   uses it.

Then give the person the `share_url` from `get-character-tool` or
`get-thread-tool`. A private record is open only to the organization's members.
