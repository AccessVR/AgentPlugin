# Copying a Character, Persona, Thread or Report Type

A copy keeps the work already done. Confirm the source and the new name with
the person before each call, then edit the copy as in the main skill. It
publishes like anything else.

## Characters, Personas and Threads

Find the source with `list-characters-tool {search}`,
`list-personas-tool {search}` or `list-threads-tool {search}`.

- `duplicate-character-tool {organization?, id, name?}` keeps the
  configuration, documents, Guides and Goals. The copy uses the same Persona
  as the source, so editing that Persona changes both. Duplicate the Persona
  too if the copy needs its own behavior, then point the copy at it with
  `update-character-tool {id, persona_id}`.
- `duplicate-persona-tool {organization?, id, name?}` keeps the prompt,
  settings, documents and Guides.
- `duplicate-thread-tool {id, name?}` keeps the lineup and its per-slot
  settings, and makes the copy private. The copy plays the same Characters as
  the source. It has no `organization` and lands in the person's active
  organization, so if they work in more than one, tell them where it will land
  before calling.

What every copy has in common:

- It is an unpublished draft owned by the person.
- **Organization.** Pass `organization` (id or exact name) when the person
  works in more than one. The copy can only land in an organization that owns
  the source or subscribes to it.
- Content another organization shares with yours can be copied only when its
  owner allows subscribers to duplicate it, and only from its published
  version. One never published can't be copied. Such a copy leaves out the
  owner's Guides, and the response's `warnings` name each one. Relay them, and
  attach your own Guides if the copy needs them (see the
  `knowledge-from-sources` skill).

## Report Types

No tool duplicates a Report Type, so rebuild it:

1. `get-report-type-tool {id}` on the source. Note its `system_prompt`, its
   analysis settings, and its Measurables.
2. Skip every Measurable with `always_on` true. Every report carries those
   already.
3. For each of the rest, read the full criterion with `get-measurable-tool {id}`.
4. Confirm the new name and organization, then
   `create-report-type-tool {organization?, name, description?, system_prompt?, include_perception_analysis?, exclude_demographics_analysis?}`.
5. In the original order, recreate each criterion with
   `create-measurable-tool {conversation_report_type_id, label, key?, definition?, help_text?, additional_guidance?, applicability_rule?, source_type?, value_type?, value_config?, computed_metric?, derived_formula?}`.
   Create the Measurables a `derived` one refers to before it.
6. Compare the two with `get-report-type-tool`, then attach and publish the copy
   as in the main skill.
