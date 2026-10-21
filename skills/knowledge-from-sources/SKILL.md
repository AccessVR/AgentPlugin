---
name: knowledge-from-sources
description: Turn documents, videos, recordings and websites into knowledge an OrchestrateXR Character or Persona draws on during conversations. Covers making a Guide from an uploaded file, reading a website into a Guide, writing a Guide directly, attaching Guides to a Persona or Character, and checking what the knowledge base now answers. Use when someone wants a Character to know a manual, policy, procedure, product sheet, transcript or website, or asks why a Character doesn't know something.
metadata:
  requires: authoring
---

# Knowledge from sources

A Character answers from **Guides**: pages in the organization's knowledge
base. Make a Guide from a file, a website, or text you write, then attach it to
a Persona or a Character. During a conversation the Character searches the
Guides it can reach and answers from them.

Every `list-`, `get-` and `search-` tool named here runs with
execute_read_tools. Every other tool runs with execute_write_tools.

## Before you start

- **Say where the knowledge will live.** A Guide made from a file or a website
  is stored in the organization's knowledge base, where the organization's
  members can find and read it. Say so when you propose it, before any upload
  or create. If the file is confidential, let the person decide.
- **Organization.** The create tools take `organization` (id or exact name).
  If the person belongs to more than one organization and hasn't said which,
  ask, then pass it on every create. A Guide can be attached only to a Persona
  or Character in the same organization.
- **Confirm every write.** Say which record and what changes, and get a yes.
- **"Wasn't found, or isn't available."** Check the id with the matching list
  tool. Don't repeat the call.
- Guides need the Guides module. If a create is refused for that reason, relay
  it; OrchestrateXR support turns it on.

First check whether a Guide already covers it:
`list-guides-tool {search, organization?}`. The results show each Guide's
`state`.

## 1. From a file

Documents (PDF, Word and similar) become a File guide: OrchestrateXR reads the
file and writes it up as an organized, searchable page with the original
attached. Video and audio become a Media guide written from the transcript.
Images are refused.

1. Get the file into the asset library. Read
   `get-skill-tool {name: "add-assets"}` for uploading, or find one already
   there with `list-assets-tool {search}`.
2. Wait for the file to finish processing. Poll `get-asset-tool {id}` until its
   `state` is `processed` or `ready`. If it failed, follow "When processing
   fails" in the add-assets skill.
3. Confirm the file, and that the Guide will live in the organization. Then
   `create-guide-from-asset-tool {organization?, asset_id}`.
4. The Guide comes back in state `processing`. Call
   `get-guide-tool {id, wait_seconds: 30}`, which waits up to 30 seconds for it
   to finish. Repeat until `state` is `ready` or `failed`. Its title,
   description and keywords are filled in from the file.

## 2. From a website

Confirm the address and how much of the site to read, then call
`create-website-guide-tool {organization?, url, name?, crawl_depth?, crawl_page_limit?, crawl_frequency?}`.

- `crawl_depth`: 0 (the default) reads the pages the site lists about itself.
  Each step beyond follows links one level further. The tool's schema states
  the maximum.
- `crawl_page_limit`: the most pages to keep. The default is also the most an
  author can ask for; a larger site needs OrchestrateXR support.
- `crawl_frequency`: `weekly` or `monthly` re-reads the site on that schedule.
  Leave it out to read the site once.

The crawl follows the site's robots.txt and stops if the site declines AI use.
Addresses on private networks are refused. Wait for it with `get-guide-tool`
as for a file.

## 3. Write one directly

For material the person gives you in the conversation, or that you write with
them: `create-guide-tool {organization?, name, markdown, intro?, keywords?}`.
It is ready at once, and search picks it up shortly after.

## 4. Name it for what it holds

At the start of each conversation the Character is told the names of the
Guides it can search, which helps it recognize when a question calls for a
lookup. "Riverside Clinic Discharge Procedure, 2026" tells it far more than
"Notes". Rename one with `update-guide-tool {id, name}`.

## 5. Attach it

Choose the scope with the person:

- **Persona**: `add-persona-guides-tool {id, guide_ids}`. Every Character on
  that Persona can search them. Adding the first Guide turns on the Persona's
  knowledge search.
- **Character**: `add-character-guides-tool {id, guide_ids}`. Only this
  Character searches them, and only when its Persona's knowledge search is on.
  The response warns when it isn't. Fix that by also adding a Guide to the
  Persona.

`update-persona-tool {id, search_all_guides}` lets a Persona search every Guide
in the organization instead. That is broad, and the Character isn't told the
Guides' names; confirm it with the person.

Attaching and removing Guides reaches the next conversation without publishing.
A Guide doesn't need publishing to be searched; it needs to be `ready`.
Publishing a Guide with `publish-guide-tool` is for showing it in the
organization's docs, and is a separate decision.

Remove with `remove-persona-guides-tool {id, guide_ids}` or
`remove-character-guides-tool {id, guide_ids}`. The Guide itself is untouched.
Removing a Persona's last knowledge source turns its knowledge search off.

Read back what is attached with `get-character-tool {id}` or
`get-persona-tool {id}`, under `knowledge`.

## 6. Check what it can answer

`search-knowledge-base-tool {query, max_results?}` returns the passages that
best match a question, with citations. Ask the way a learner would ("What do I
do if the patient's oxygen drops below 90?").

- It searches the whole knowledge base of the person's active organization,
  not one Character's Guides. A hit shows the Guide is ready and searchable. It
  doesn't show the Character reaches it; check that with `get-character-tool`.
- A Guide in another of the person's organizations won't show here.
- `document_id` narrows the search to one uploaded document. It does not take a
  Guide id.

## Who the Character answers for

A Character answers only from Guides the person in the conversation may read.

- A private Guide reaches members of the organization.
- Someone without an account, running an unlisted Character or Thread, gets
  nothing from private Guides.
- A Guide restricted to certain roles reaches only people with those roles.

Setting a Guide's visibility to public with
`update-guide-tool {id, visibility: "public"}` lets anyone's conversation use
it, and makes the Guide readable by anyone. Propose that only for material
that may be public, and confirm it by name.

## When a Character doesn't know something

Check in order:

1. The Guide is attached to this Character or its Persona (`get-character-tool`).
2. The Persona's knowledge search is on (`get-persona-tool`, `knowledge.search_enabled`).
3. The Guide's `state` is `ready` (`list-guides-tool`).
4. The person in the conversation may read it (see above).
5. `search-knowledge-base-tool` finds the passage for the question as asked.
   If not, the Guide may not say it, or may say it in other words.
