# Copying a Character, Persona, Thread or Report Type

A copy keeps the work already done. Confirm the source and the new name with
the person before each call, then edit the copy as in the main skill. It
publishes like anything else.

## Characters, Personas and Threads

Find the source with `list-characters-tool {search}`,
`list-personas-tool {search}` or `list-threads-tool {search}`.

- `duplicate-character-tool {organization?, id, name?}` keeps the
  configuration, attachments, documents, Guides and Goals. The copy uses the same Persona
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
  version. One never published can't be copied. Such a copy keeps the settings
  your organization chose for the original. It leaves out the owner's
  attachments, so add your own if the copy needs them. It also leaves out the
  owner's Guides, and the response's `warnings` name each one. Relay them, and
  attach your own Guides if the copy needs them (see the
  `knowledge-from-sources` skill).

## Report Types

Find the source with `list-report-types-tool {search}`.

- `duplicate-report-type-tool {organization?, id, name?}` keeps the narrative
  prompt, the analysis settings and every Measurable, in order, with all of
  its fields. Derived Measurables still combine the right inputs. The copy is
  attached to no Character or Thread. Compare it with the source using
  `get-report-type-tool`, then attach and publish it as in the main skill.
- Only someone who can edit the Report Type can copy it. One another
  organization shares with yours can be attached as it is, but not copied.
