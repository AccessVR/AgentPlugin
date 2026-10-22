---
name: assemble-course
description: Build or change a Course in OrchestrateXR and publish it. Use when someone wants to put Experiences, Characters, Threads, Guides or Assets in order for learners, lock items until the one before is done, add a debrief, set due dates, visibility or self-enrollment, reorder or remove Course content, publish or archive a Course, enroll or withdraw people and Groups, reset a learner's progress, or check who is enrolled and who has finished.
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

On a published Course, most settings reach learners the moment you save
them: `visibility`, `self_enrollment_enabled`, the start and end dates, the
due-date fields and `thumbnail_asset_id`. Confirm those as changes to what
learners get now. The debrief and the learning objectives wait for
publishing, and so does the name on the Course's own screen, though a new
name shows in lists at once.

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

`publish-course-tool {id}` makes the current items, debrief, name and
learning objectives what learners get. Until then, learners keep the
published order, gating and retry settings, so a change to them doesn't lock
or unlock anything for a learner yet. An unpublished Course shows learners
nothing. Publishing a Course with no
items is allowed, but nobody can start it, and the response warns you.

After any change to a published Course, remind the person to publish again.
Check the draft state with `get-course-tool {id}`: `publish.has_unpublished_changes`.

## 5. Enrolling learners

Publish first: learners can't see an unpublished Course, and enrolling still
emails them.

1. Find who to enroll with
   `list-enrollment-targets-tool {id, search?, limit?}` (run with
   `execute_read_tools`). It returns the `people` and `groups` that can be
   enrolled in this Course. Each person's `enrollment_status` is `null` if
   they've never been on it, otherwise `active`, `completed` or `withdrawn`.
   Each Group says whether it's `enrolled`.
2. Name the Course and every person and Group, say that each person newly
   enrolled gets an email, and get a yes.
3. `enroll-in-course-tool {id, user_ids?, group_ids?}` (run with
   `execute_write_tools`). Send at least one of the two lists.

- A Group's current members are enrolled, and so is anyone who joins it
  later, so adding someone to an enrolled Group with
  `update-group-members-tool {id, add_user_ids}` (run with
  `execute_write_tools`) enrolls them too.
- Someone who had been withdrawn comes back with their earlier progress and
  isn't emailed again. Someone already enrolled is left as they are.
- Relay the `warnings`. The response lists who was `enrolled`, `reinstated`
  and `already_enrolled`, and how many members each Group added.

The Course's **Share** button gives the link learners use to find the Course
and enroll themselves. Give the person the Course's `url` from a tool result
to open that screen. Never build a learner link yourself.

### Withdrawing learners or Groups

`withdraw-from-course-tool {id, user_ids?, group_ids?}` (run with
`execute_write_tools`). Name the Course and everyone being withdrawn, and get
a yes first. Nobody is emailed.

- A withdrawn person keeps their progress. Enrolling them again picks up where
  they left off.
- Withdrawing a Group removes only the enrollments that Group made, and stops
  enrolling its new members. People who enrolled themselves or came through
  another Group stay.

### Resetting a learner's progress

`reset-course-progress-tool {id, user_ids}` (run with `execute_write_tools`)
starts enrolled learners over. **It permanently deletes their completion
records for every item and debrief in the Course, and can't be undone.**
Restate the Course's name and each learner's name, say their progress will be
erased for good, and get a yes for that specifically. Their Sessions and
Conversations are kept. If anyone named isn't enrolled, nothing is reset.

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
