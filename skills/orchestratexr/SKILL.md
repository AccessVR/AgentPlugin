---
name: orchestratexr
description: Use whenever someone asks to do anything in OrchestrateXR, such as building or changing an Experience, Character, Thread, Course, Asset or Guide, finding a record, getting a link to share, or checking how learners did. Explains how the product's records fit together, how to find and run its tools, which organization to work in, what to confirm before a change, how to read a refusal, and which guide covers each job.
---

# Working in OrchestrateXR

Every call runs as the signed-in person, and their memberships and roles
decide what it reaches. Read this once, then the guide for the job.

## The records and how they fit together

- **Experience**: an immersive lesson made of **scenes**, each a 360° image or
  video. A scene holds **layers**: text cards, questions, media, and
  **hotspots** that move the learner to another scene, show a card, or open a
  link.
- **Character**: what a learner talks to. It pairs a **Persona** (how it
  thinks and behaves, speaking in a **Voice**) with a **Replica** (the face a
  learner sees), and adds its own scenario and greeting.
- **Thread**: an ordered lineup of Characters a participant moves through.
- **Report Type**: how conversations are scored, built from **Measurables**
  (one criterion each). On a Character it scores each conversation with that
  Character. On a Thread it scores the whole run.
- **Goals**: targets a learner can meet during a conversation. They live in a
  library and are assigned to Characters.
- **Course**: an ordered list of Experiences, Characters, Threads, Guides and
  Assets that learners are enrolled in. A learner's enrollment is an
  Assignment.
- **Assets**: images, 360° media, video, audio and documents.
- **Guides**: written pages for learners to read and for Personas and
  Characters to draw on as knowledge.
- **Sessions** and **Conversations**: what learners did. A Session is one run
  of an Experience, Character or Thread, with its score and reports. A
  Conversation holds the transcript and the Goals reached.
- **Projects**: where an organization plans content, with a brief and links.

## Finding and running tools

- For a job that takes several tools, read its guide first with
  `get-skill-tool {name}`. Call it directly, the same way you call
  `search-docs-tool {q}` and `get-doc-tool {slug}` for how-to questions about
  the product.
- Otherwise call `search_tools` with the job in plain words ("publish a
  course", "who is enrolled"). Each result gives the tool's exact name, its
  arguments, and `run_with`.
- Run catalog tools through the executor `run_with` names:
  `execute_read_tools` for read-only tools, `execute_write_tools` for the
  rest. Pass exact names:
  `{"calls":[{"name":"list-courses-tool","arguments":{"search":"Onboarding"}}]}`.
  When a guide names a tool, skip the search and call the executor.
- One executor call runs up to 10 calls in order and stops at the first
  error. A batch holding a tool that belongs to the other executor is refused
  before anything in it runs. When a call needs an id from an earlier result,
  make a second executor call.
- A tool the search can't find isn't available to this account. Don't guess
  names, and pass only the arguments a tool declares.
- Every account gets 60 tool calls a minute. Page long lists with `limit`
  and `cursor`.

## Picking the organization

- `list-organizations-tool {scope}` with `scope` "mine" lists the person's
  organizations (run with `execute_read_tools`). Each result's
  `access.can_author_content` says whether they can create content there.
- Create tools take `organization` (id or exact name). If the person belongs to
  more than one organization and hasn't said which, ask, then pass it on every
  create in the job. Without it, the create is refused.
- List tools span every organization the person belongs to unless you pass
  `organization`.

## Before changing anything

- Read-only tools need no confirmation. Before any `execute_write_tools`
  call, say which record and what will change, and get a yes.
- For publishing, deleting, granting, spending tokens, or sending anything,
  restate the record by name and get an explicit yes.
- Some tools replace a whole list, such as a Course's items or a Thread's
  lineup. Whatever you leave out is removed. Read the current list first.
- Relay any warnings a write returns. If a record is locked by another editor,
  say who and don't retry.

## Draft, then publish

Edits change the draft. Learners keep the published version until the
matching publish tool runs: `publish-experience-tool`,
`publish-character-tool`, `publish-persona-tool`, `publish-thread-tool`,
`publish-report-type-tool`, `publish-course-tool`, or `publish-guide-tool`.
After editing something learners already use, remind the person to publish.

## Reading refusals

- **"wasn't found, or isn't available to you"**: the id is wrong or out of
  reach. Check it with the list tool the message names. Don't repeat the call.
- **"Unauthorized"**: the person can't use that tool or act on that record.
  Stop and explain. Don't retry or look for a way around it.
- **"Authoring tools are not enabled for this account"**: creating and
  changing content takes Authoring Tools, which OrchestrateXR support turns on
  per person. Support tickets, Projects and sign-in codes don't need them.
- **"Content from another organization cannot be referenced here"**:
  OrchestrateXR support can subscribe the organization to that content.
  Offer a ticket with
  `create-support-ticket-tool {subject, description}` (run with
  `execute_write_tools`), reading both back to the person first.

## Links

Share only URLs a tool returned. Never build one from an id.

- `url` opens the record's screen, for the people who work on it.
  `share_url` is the link to send to learners. Unless it's clear, ask which
  one the person wants.
- When `share_url` is null, give the reason the tool gives. Never offer `url`
  in its place: the people it's sent to can't open it.

## Guides for each job

| Job | Guide |
| --- | --- |
| Build or change an Experience: scenes, layers, hotspots, questions | `build-experience` |
| Set up a Character, its Persona and Replica, its Goals, and the Thread and Report Type around it | `build-conversation` |
| Upload, import or generate images, video, audio and documents | `add-assets` |
| Turn documents and websites into Guides a Persona can draw on | `knowledge-from-sources` |
| Put content in a Course, publish it, and see who is enrolled | `assemble-course` |
| See how learners did: Sessions, transcripts, scores, Goals, Course progress | `review-results` |

Read one with `get-skill-tool {name}`. When a guide points to a reference
file, read it with `get-skill-tool {name, file}`. Guides that create or change
content need Authoring Tools and are hidden without them.
