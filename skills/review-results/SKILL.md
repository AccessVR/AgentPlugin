---
name: review-results
description: Find out how learners did in OrchestrateXR. Use when someone asks how a person or a group performed, who finished an Experience, Thread or Course, what someone said in a Conversation, how a conversation was scored and against what criteria, which Goals a learner reached, or wants a report or a summary of results. Covers going from a person's name to their Sessions, Conversations and Course enrollments.
---

# Review results

Everything here only reads, so every tool runs with `execute_read_tools` and
needs no confirmation, except changing Course enrollments at the end of
section 7. What comes back depends on who is asking:

- Learners see their own Sessions and Conversations.
- Org Admins and Training Directors see their organizations' Sessions and
  Conversations, including people who took part without signing in.
- Course enrollments are visible to the people who build Courses.

If a list comes back empty or "Unauthorized", the person may only be able to
see their own results. Say so instead of searching further.

## Where results live

- **Session**: one run of an Experience, Character or Thread, with the
  learner, `attempt_number`, `score`, `started_at`, `completed_at` and
  `exited_at`. Its detail holds the written reports, a PDF link, and the
  Goals reached.
- **Conversation**: the transcript of one conversation with a Character, and
  the Goals reached in it.
- **Assignment**: one learner's enrollment in a Course, with its status and
  completion date.
- **Report Type** and **Measurables**: the criteria a conversation was scored
  against.

## 1. From a name to a person

1. `list-users-tool {search}` matches a partial name, exact email or exact
   username and returns the person's `id`. It only answers for people who
   manage users in an organization.
2. If it refuses, `list-assignments-tool {search}` matches a learner's exact
   email or partial name among Course enrollments, and each row carries
   `learner.id`.
3. Session rows also carry `learner.name`, so you can find someone by reading
   the runs of the content they took.

If several people match, show the names and emails and ask which one.

People who took part without signing in all share one anonymous account, so
their id matches everyone else who did the same. Find them by
`participant_name` (and `participant_email` on Conversations) instead, after
filtering by the content they took.

## 2. Find their runs

`list-sessions-tool {learner_id?, experience_id?, thread_id?, character_id?, status?, started_after?, started_before?, organization?, limit?, cursor?}`

- Resolve the content first with `list-experiences-tool {search}`,
  `list-threads-tool {search}` or `list-characters-tool {search}`.
- `status` is `completed`, `in-progress` or `exited` (left before finishing).
- `learner_id` takes one id or a comma-separated list.
- Results come newest first, up to 100 a page. Follow `next_cursor` while
  `has_more` is true.
- Leave out runs where `is_preview` is true. Those are authors testing their
  own content.

## 3. Read a run in detail

`get-session-tool {id}` adds:

- `reports`: each has a `kind` (`conversation` for a single conversation,
  `thread_rubric` when a Report Type scores the whole Thread run,
  `thread_summary` when none does), a `status` (`pending`, `completed` or
  `failed`), and its written assessment in `content`. A `pending` report is
  still being written: say so and offer to check again shortly.
- `pdf_download_url`, on the Session and on each finished report. These links
  expire after about an hour. Fetch the Session again for a fresh one rather
  than reusing an old link.
- `goals`: for each conversation in the run that had Goals, which were
  reached, when, and in the learner's own words.

A Session the person can't reach answers "Session not found". Check the id
with `list-sessions-tool`.

## 4. Read a Conversation

`list-conversations-tool {character_id?, creator_id?, status?, search?, organization?, limit?, cursor?}`
finds Conversations, newest first. `creator_id` is the person who took part
(their user id). `status` is `active`, `ended` or `failed`. Then
`get-conversation-tool {id}` returns the `transcript`, `duration`,
`participant_name`, and `goals`: the Goals reached, the ones still on the
learner's list, and a count of hidden Goals not yet reached.

A Session records the Conversation that ended its run. When that run had
Goals, `get-session-tool {id}` names each step's Conversation as
`conversation_id` in its `goals`. Otherwise, list the person's Conversations
with that Character and match the time to the Session's `started_at`.

## 5. Explain a score

1. Find which Report Type scored it: `get-character-tool {id}` for a
   Character's conversations, `get-thread-tool {id}` for a whole Thread run.
   Both return `conversation_report_type`. With none, a Character's
   conversations get no report of their own, and a Thread run gets an
   unscored summary.
2. `get-report-type-tool {id}` lists every Measurable it scores with.
3. `get-measurable-tool {id}` gives one criterion's definition and guidance.

Quote the report's own words when explaining a result. Don't invent a score
or a criterion the report doesn't contain.

## 6. Goals

`list-goals-tool {character_id}` lists the Goals assigned to a Character, in
order, and whether each shows on the learner's list. What a learner reached
is in the `goals` of `get-conversation-tool` and `get-session-tool`.

## 7. Course progress

`list-assignments-tool {course_id?, user_id?, status?, search?, limit?, cursor?}`

- Resolve the Course with `list-courses-tool {search}`.
- `status` is `active`, `completed` or `withdrawn`. Each row has
  `enrolled_at`, `completed_at`, and `source_group` when a Group enrolled the
  learner.
- An Assignment is complete only when every item and debrief is done.
- It doesn't show progress item by item. For that, read the Course's items with
  `get-course-tool {id}`, then look up the learner's Sessions for each
  Experience, Character or Thread in it. Guides and Assets leave no Sessions.

### Changing enrollments

People who can edit a Course can also change who is on it. These aren't
reads: they run with `execute_write_tools`, and each needs a yes first.

- `list-enrollment-targets-tool {id, search?}` (a read) turns names into the
  people and Group ids below, with each person's `enrollment_status`.
- `enroll-in-course-tool {id, user_ids?, group_ids?}` enrolls people or
  Groups. Each person newly enrolled is emailed; say so before calling.
- `withdraw-from-course-tool {id, user_ids?, group_ids?}` takes them off the
  Course. Their progress is kept for if they come back.
- `reset-course-progress-tool {id, user_ids}` starts enrolled learners over.
  It permanently deletes their completion records for that Course and can't
  be undone, so restate the Course and each learner by name and say so before
  calling. Sessions and Conversations are kept.

## Summaries and reports

No tool here builds a spreadsheet or a report file. For a summary, page
through the lists above and tally the results yourself. Say how many runs you
counted, what date range they cover, and that preview runs were left out. For
the full PDF of one run, share its `pdf_download_url`. If someone needs a data
export beyond that, offer to file a support ticket with OrchestrateXR support.

Share only the links a tool returned, and only with people the person says
should see them. Results name learners and quote what they said.
