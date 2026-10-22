---
name: plan-a-project
description: Plan training or other immersive content in an OrchestrateXR Project. Use when someone wants to start a project, work on its brief (Outcomes, Audience, Requirements), add the people it answers to and invite or notify them, share it or keep it to its people, save the SOPs, manuals and websites behind it, answer questions from those sources, plan and link the content it will produce, or move it to its next stage. Covers finding a project, reading it, rewriting a section without overwriting someone else's edit, and the order the work usually goes in.
---

# Plan a Project

A Project is where an organization plans a piece of content before and while
it's built. It holds:

- a **brief**: Outcomes, Audience and Requirements, plus any sections the
  authors add
- its **people**: the sponsors, experts, reviewers and learner
  representatives it answers to
- its **resources**: Guides holding the SOPs, manuals and websites behind it
- its **deliverables**: the Experiences, Characters, Threads, Courses, Assets
  and Guides it produces, linked once they exist and planned before they do

What goes in the brief is a design question. Read
`get-skill-tool {name}` with `design-training` before drafting a section, and
follow it for what to ask and how to push back.

## Before you start

- Projects don't need Authoring Tools. Building the deliverables does.
- If a tool answers that the organization "doesn't have Projects turned on",
  OrchestrateXR support can turn it on. Offer a support ticket.
- Every write here runs with `execute_write_tools`; reads run with
  `execute_read_tools`. Say what will change and get a yes first, with the one
  exception under "Name it" below.

## 1. Find or start the project

- **Find one**: `list-projects-tool {organization?, status?, query?}`, where
  `query` matches the name and `status` is `exploring`, `designing`,
  `building` or `delivered`. Then read it (step 2).
- **Start one** as a conversation, not a form. Don't ask for a name, an
  organization or the sections. Ask what they want to work on, and work
  through the problem with `design-training`. Once you understand it, propose
  a short name and a first draft of the sections the conversation covered,
  and create it when they agree:
  `create-project-tool {organization?, name, summary?, outcomes?, audience?, requirements?}`.
  The sections are Markdown. Leave out any you haven't discussed; they start
  empty.
- Use the person's active organization. Ask which one only when they belong
  to more than one and nothing they've said tells you.
- Called from a Paloma chat that isn't in a project yet, the chat moves into
  the new project, so the conversation so far stays with it.

The result includes the project's `url`. Share that, never one you build.

## 2. Read it

`get-project-tool {project}` returns everything: the brief in full, each
section with its `id`, `kind` and `version`, the people, the deliverables,
the resources and recent activity.

- Rely on a section's `kind` (`outcomes`, `audience`, `requirements` or
  `custom`), not its title. Authors can rename sections.
- Read the project again before any write if time has passed. Other authors
  work in it too.

## 3. Write the brief

`update-project-section-tool {project, section, version, body}` replaces the
whole section with `body`.

- Start from the section's current text, make your change in it, and send all
  of it back as Markdown. Anything you leave out is deleted.
- Pass the `version` you read. If someone changed the section since, the
  write is refused: read it again, re-apply your change to what it says now,
  and show the person the new text before trying again.
- Show the person the new text and get a yes before every write.
- **Add a section** at the end by leaving out `section` and passing a title:
  `update-project-section-tool {project, title, body}`.
- `design-training` step 8 says what belongs in each section.

## 4. Name it, and keep its details current

`update-project-tool {project, name?, summary?, status?, launch_date?}`
changes only what you pass.

- **Name it**: while a project is still called "Untitled project", give it a
  short name of three to six words as soon as you understand what it's for,
  without asking. Mention the new name in a sentence, and that they can
  rename it by clicking the name. This is the one change you make without
  confirming.
- **Stage** (`status`) records how far along it is:
  - `exploring`: working out the problem and whether training is the answer
  - `designing`: settling the behaviors, objectives and what to build
  - `building`: making the deliverables
  - `delivered`: in learners' hands

  Suggest the next stage when the work has reached it, and confirm first.
- `launch_date` is when the training launches, as a date.
- **Sharing** (`visibility`) is who can open it:
  - `private`, the default: every author in its organization.
  - `unlisted`: the same, plus a read-only link to the brief that anyone
    holding it can read without an account. It shows the brief, never the
    people, resources or chats. The result's `read_only_url` is the link.
    Share it only with the people the person names.
  - `secret`: only its creator, the organization's Org Admins, and its people
    (step 5). Whoever makes it secret is added to its people if they'd
    otherwise lose access.

  Confirm by name before changing it. Going from `unlisted` to anything else
  turns the link off.

## 5. People

`manage-project-stakeholders-tool {project, action, stakeholder?, name?, email?, title?, role?, notes?, invite_as?}`

- `action` is `add`, `update`, `remove`, `invite` or `notify`. Every action
  but `add` takes the stakeholder's `id` as `stakeholder`, from
  `get-project-tool`.
- `role` is `sponsor`, `sme`, `reviewer`, `learner_rep` or `other`. A person
  needs only a name and a role. Many have no OrchestrateXR account.
- **Who is emailed.** Adding someone whose email reaches an account that can
  open the project notifies them by email and in the app. Nobody else is sent
  anything. Before adding anyone with an email, name the person, the address
  and the role, and say whether they'll be notified.
- **Inviting.** The result's `account` says where they stand: `member`,
  `invited`, or null for no access. When it's null and `can_invite_as` isn't
  empty, offer to invite them at one of those roles. Pass `invite_as` with
  `add` to do it in the same step, or use `invite` later. An invitation is an
  email too, so confirm it by name.
- `notify` sends the notice again, for someone who missed it.
- **Notes** are visible to everyone who can open the project, including the
  person they're about. Write them accordingly.
- The project's creator (`is_creator`) is always listed. Their details can
  change, but they can't be removed.
- In a `secret` project the people are its guest list: adding someone with an
  account in the organization lets them open it, and removing them shuts them
  out.

## 6. Resources

Resources are the source material: SOPs, manuals, slide decks and websites,
saved as Guides.

1. Make the Guide from the file or website with `knowledge-from-sources`.
2. Link it as a resource:
   `link-project-content-tool {project, action, type, id, purpose}` with
   `action` "link", `type` "guide" and `purpose` "resource".
3. A resource that's still being read returns nothing until it's ready.

Before answering a question about the project's subject matter, search its
resources with `search-project-resources-tool {project, query, max_results?}`,
and cite the source and page of anything you use. It searches only that
project's resources.

## 7. Deliverables

- **Plan** what the project will produce before it exists, so the plan is
  visible to everyone:
  `link-project-content-tool {project, action, type, title}` with `action`
  "plan".
- **Build** it with the guide for that kind of content: `build-experience`,
  `build-conversation`, `assemble-course` or `add-assets`.
- **Link** it as a deliverable straight after creating it:
  `link-project-content-tool {project, action, type, id}` with `action`
  "link". `purpose` defaults to "deliverable". Use "reference" for existing
  content the project draws on rather than produces.
- `type` is `experience`, `character`, `thread`, `course`, `asset` or `guide`.
  Content has to belong to the project's organization.
- **Take something off** the project with
  `link-project-content-tool {project, action, item}`, `action` "unlink",
  passing the project item's id from `get-project-tool`. The content itself is
  untouched. Confirm first.

## The usual order

1. Work out the problem, the behaviors and the barriers (`design-training`),
   and draft Outcomes and Audience as you go. Stage: `exploring`.
2. Add the people it answers to, and the resources behind it.
3. Settle the objectives and what to build, and write Requirements. Plan the
   deliverables. Stage: `designing`.
4. Build and link each deliverable. Stage: `building`.
5. Publish, enroll learners, and check results against the measure in
   Outcomes (`review-results`). Stage: `delivered`.

People skip around, and that's fine. Follow where they are, and suggest the
next step when they ask what's left.
