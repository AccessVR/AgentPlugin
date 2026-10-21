---
name: assemble-course
description: Build or change a Course in OrchestrateXR and publish it. Use when someone wants to put Experiences, Characters, Threads, Guides or Assets in order for learners, lock items until the one before is done, add a debrief, set due dates, visibility or self-enrollment, reorder or remove Course content, publish or archive a Course, or check who is enrolled and who has finished.
metadata:
  requires: authoring
---

# Assemble a Course

A Course is an ordered list of content that learners work through. Each
learner's enrollment in a Course is an Assignment.

## Before you start

- Org Admins and Training Directors build Courses, in organizations with
  Learning Management turned on. If `create-course-tool` answers
  "Unauthorized", either the person's role can't build Courses there or
  Learning Management is off for that organization. Only OrchestrateXR
  support can turn it on. Offer a support ticket.
- If the person belongs to more than one organization and hasn't said which,
  ask, and pass `organization` when you create the Course. Every item must
  belong to that organization or be shared with it.
- Every write here runs with `execute_write_tools`. Say what will change and
  get a yes first. Restate the Course's name before publishing or deleting.

## 1. Find the content

Resolve each item to an id with its list tool. All of these run with
`execute_read_tools`, and they can go in one batch:

- Experiences: `list-experiences-tool {search, organization?}`
- Characters: `list-characters-tool {search, organization?}`
- Threads: `list-threads-tool {search, organization?}`
- Guides: `list-guides-tool {search, organization?}`
- Assets: `list-assets-tool {search, organization?}`

Each result says whether it has been published. Learners get the published
version of each item, so point out any item that has never been published.

## 2. Create the Course

`create-course-tool {organization?, name, description?, learning_objectives?, visibility?, due_date_mode?, due_date?, due_days_after_enrollment?, self_enrollment_enabled?, debrief_enabled?, debrief_character_id?, passing_grade?}`

- Only `name` is required. Propose the rest from the conversation rather than
  asking field by field.
- `visibility`: `private` (the default), `unlisted` (reachable by link and by
  members, kept out of the Courses gallery), `public`, or `secret`.
- Due dates: `due_date_mode` is `none`, `fixed` (send `due_date`), or
  `relative` (send `due_days_after_enrollment`, at least 1). Use `relative`
  when people join on a rolling basis.
- `self_enrollment_enabled` is on unless you turn it off. Off means only
  administrators enroll learners.
- A debrief needs a Character: set `debrief_enabled` and
  `debrief_character_id` (from `list-characters-tool`). Do this before
  turning on a debrief after an individual item.

Change settings later with
`update-course-tool {id, name?, visibility?, due_date_mode?, due_date?, due_days_after_enrollment?, self_enrollment_enabled?, debrief_enabled?, debrief_character_id?, thumbnail_asset_id?}`.
Only the fields you pass change.

## 3. Set the items

`set-course-items-tool {id, items}` replaces the whole list. Each entry is
`{"type": ..., "id": ..., "gated": ..., "retry_enabled": ..., "retry_limit": ..., "debrief_enabled": ...}`,
with `type` one of `experience`, `character`, `thread`, `guide`, `asset`.
Position comes from the order of the array.

- `gated: true` locks an item until the one before it is complete. Don't gate
  the first item; nothing comes before it.
- `retry_limit` caps retries (at least 1). `null` removes a cap.
- `retry_enabled: false` turns retries off, so the item can be completed
  once. It's the **Allow Retries** switch in the Course editor.
- `debrief_enabled: true` runs the Course's debrief Character right after that
  item.
- List each item once.

A new item gets the defaults: not gated, retries allowed with no limit, no
debrief.

**Changing an existing Course.** Read it first with `get-course-tool {id}`
(run with `execute_read_tools`). Its `items` come back in the same shape this
tool takes. Edit that list and send all of it back.

- An item you leave out is removed.
- An item already in the Course keeps any setting its entry leaves out, so
  `{"type": ..., "id": ...}` alone moves it without changing how it behaves.
  Name a setting only to change it.
- **Removing an item permanently deletes every learner's completion record
  for it.** Before the call, name each item being removed and say that
  learners' progress on it will be lost. Get a yes for that specifically.
  Reordering and changing settings keep learners' progress.
- Relay the `warnings` the response returns.

## 4. Publish

`publish-course-tool {id}` makes the current settings and items what learners
get. Until then, learners keep the published order, gating and retry settings,
so a change to them doesn't lock or unlock anything for a learner yet. An
unpublished Course shows learners nothing. Publishing a Course with no
items is allowed, but nobody can start it, and the response warns you.

After any change to a published Course, remind the person to publish again.
Check the draft state with `get-course-tool {id}`: `publish.has_unpublished_changes`.

## 5. Enrolling learners

No tool enrolls or withdraws learners. Once the Course is published, the
person does that from the Course's screen in OrchestrateXR, with its
**Enroll** button. They can enroll individuals or a Group, and a Group keeps
enrolling its new members, so adding someone to an enrolled Group with
`update-group-members-tool {id, add_user_ids}` (run with
`execute_write_tools`) enrolls them too. Its **Share** button gives the link learners use
to find the Course and enroll themselves. Give them the Course's `url` from a
tool result to open that screen. Never build a learner link yourself.

## 6. Who is enrolled, and who has finished

`list-assignments-tool {course_id, status?, search?, limit?, cursor?}` (run
with `execute_read_tools`) lists enrollments, newest first, with each
learner's name and email, `enrolled_at`, `completed_at`, and
`source_group` when a Group enrolled them. `status` is `active`,
`completed`, or `withdrawn`. Pass `user_id` instead of `course_id` for
everything one person holds.

An Assignment is complete only when every item and every debrief is done. The
tool doesn't report progress item by item. For how far someone got, or how
they scored, read `get-skill-tool {name}` with `review-results`.

## Archiving a Course

`delete-course-tool {id}` archives the Course. It disappears from learners'
lists, and enrollments and progress stay in the archive. OrchestrateXR support
can recover it. Before calling, read `enrollment_count` from
`get-course-tool {id}` and restate the Course's name and that count.

## If you're working in a Project

Link the new Course to the Project as a deliverable with
`link-project-content-tool {project, action, type, id}` (run with
`execute_write_tools`; `action` "link", `type` "course").
