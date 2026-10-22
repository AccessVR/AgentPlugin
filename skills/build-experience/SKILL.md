---
name: build-experience
description: Build or change an OrchestrateXR Experience. Covers creating the draft, giving each scene a 360° backdrop, adding text cards, questions, hotspots and media, linking scenes, checking what blocks publishing, and publishing. Use when someone wants a new Experience or virtual tour, wants to add or reorder scenes, add a quiz, hotspots or a video to a scene, fix a failed publish, copy an Experience, or roll one back to an earlier version.
metadata:
  requires: authoring
---

# Build an Experience

An Experience is a set of scenes. Each scene is a 360° image or video, its
skybox, with layers on a timeline: text cards, questions, hotspots and media.
Read `references/layers.md` before adding or changing a layer. It holds every
layer setting, the hotspot, question and action shapes, and worked recipes.

Every `list-`, `get-` and `validate-` tool here runs with
`execute_read_tools`. Every other tool runs with `execute_write_tools`.

## Before you start

- **Organization.** `create-experience-tool` takes `organization` (id or
  exact name). If the person belongs to more than one organization and hasn't
  said which, ask, then pass it. Every Asset the Experience uses must belong
  to that organization or be shared with it.
- **Confirm every write.** Say which Experience, scene or layer changes and
  how, and get a yes. Restate the Experience's name before publishing,
  restoring a version or deleting.
- **Draft, then publish.** Every edit changes the draft. Learners keep the
  published version until `publish-experience-tool` runs.
- **Links.** Share only URLs a tool returned. `url` opens the Experience to
  work on it, `preview_url` plays the current draft without signing in, and
  `share_url` is the link for learners, null until published. Treat
  `preview_url` as a secret.
- **"Wasn't found, or isn't available."** Check the id with
  `list-experiences-tool`, or a scene id with `get-experience-tool`. Don't
  repeat the call.

## Read before you change

1. `list-experiences-tool {search?, organization?, creator_id?, asset_id?}`
   finds an Experience by partial name, exact id or guid.
2. `get-experience-tool {id}` returns its settings, its scenes in order
   (`id`, `name`, `is_initial`, `skybox`, `layer_count`), its `publish`
   state and any edit `lock`.
3. `get-scene-tool {id}` returns one scene's timing, end action and every
   layer with its `id`. You need the layer id to change or delete a layer.
   Hotspots, questions, answers and actions come back with camelCase keys,
   which the write tools accept as they are.

## Build a new Experience

1. **Agree the plan.** Settle the name, the scenes and the backdrop for each,
   what happens in each scene, how learners move between scenes, and how the
   Experience ends.
2. **Create the draft.**
   `create-experience-tool {organization?, name, description?, visibility?, show_scene_list?, allow_skipping?, complete_message?, thumbnail_asset_id?}`.
   It starts private, unpublished and empty. `visibility` is `private`,
   `public`, `protected`, `unlisted` or `secret`. `thumbnail_asset_id` is
   the Preview Image shown on the Experience's card and in share previews.
   It must be an image. Without one, the card shows the first scene's
   backdrop.
3. **Find a skybox for each scene.**
   `list-assets-tool {search?, asset_type?, video_type?, state?, organization?}`.
   `asset_type` takes `image`, `video`, `audio` or `file`. A 360° video is
   `video` with `video_type` `360_video` (or `180_video` for 180°). Use a
   processed 360° image or video. When nothing fits, follow the `add-assets`
   skill (`get-skill-tool {name: "add-assets"}`) to generate a panorama,
   import one from a link, or upload one.
4. **Add the scenes.**
   `create-scene-tool {experience_id, skybox_asset_id, name, end_time?}`.
   This tool names the Experience `experience_id`. The first scene becomes
   the initial scene, where learners start. An image scene runs 30 seconds
   and a video scene runs as long as the video, and each ends by completing
   the Experience. Create every scene before linking them, because a
   go-to-scene action needs the target scene's id.
5. **Add the layers.**
   `add-layer-tool {scene_id, event_type, properties}`. Put every setting
   inside `properties`, and pass a question layer's questions or a hotspot
   layer's hotspots in the same call. Every hotspot needs a position and an
   action.
6. **Link the scenes.** Use go-to-scene actions on hotspots or answers, or a
   scene's end action:
   `update-scene-tool {id, end_action}` with `{"type": 3, "scene_id": 42}`.
7. **Validate.** `validate-experience-tool {id}`, where `id` is the
   Experience's id. It has no `experience_id` argument.
8. **Explain the problems** by scene and layer name, then fix them (see
   below) and validate again until it returns `valid: true`.
9. **Publish.** Restate the Experience's name, confirm, then
   `publish-experience-tool {id}`. It returns the published version number and
   a `preview_url` for that version. Get the learner link from
   `get-experience-tool {id}`'s `share_url`.

## Fixing what blocks publishing

The problems quote editor buttons. Fix them with these tools instead:

| The problem says | Fix |
| --- | --- |
| The Experience is empty | Add a scene with `create-scene-tool`. |
| It is missing an Initial Scene | `update-experience-tool {id, initial_scene_id}` |
| A scene needs its initial view reset | `update-scene-tool {id, initial_view}` with `{"quaternion": {"X": 0, "Y": 0, "Z": 0, "W": 1}, "longitude": -90, "latitude": 0}`, which faces forward. Ask the person to set a different view in the editor if they want one. |
| A hotspot has no action | Send the layer's complete hotspot list with `update-layer-tool`, giving that hotspot an action. |
| Open URL is set but the address is missing | Give the action a `url`. |
| A question layer has no questions, or a question has no answers | Add them with `update-layer-tool`, or delete the layer. |
| An asset is not ready for publishing | Wait for it to process (`get-asset-tool {id}` until `state` is `processed`). If it failed, retry it with `retry-asset-processing-tool {id}`, or swap in another asset. |

## Change an existing Experience

- **Check its state first.** In `get-experience-tool`, `is_published` says
  whether learners have a version, and `publish.has_unpublished_changes`
  says whether the draft differs from it. Say so before editing a live
  Experience, and remind the person to publish when you're done.
- **Change a layer.**
  `update-layer-tool {scene_id, layer_id, properties}`. `scene_id` is
  required. Only the keys in `properties` change, but `hotspots` and
  `questions` replace the whole list: read the scene, edit the list, and
  send all of it back. A layer's `event_type` can't change; delete it and
  add a new one.
- **Remove a layer.** `delete-layer-tool {scene_id, layer_id}`. Restate the
  layer's name first.
- **Change a scene.**
  `update-scene-tool {id, name?, skybox_asset_id?, end_time?, end_action?, description?, show_in_scene_list?, allow_skipping?, mute_audio?}`.
  It never touches layers.
- **Reorder scenes or change where learners start.**
  `update-experience-tool {id, scene_order?, initial_scene_id?}`.
  `scene_order` must list every scene id exactly once.
- **Remove a scene.** `delete-scene-tool {id}`. The initial scene can't be
  removed while other scenes remain: set another `initial_scene_id` first.
- **Settings and sharing.**
  `update-experience-tool {id, name?, description?, visibility?, show_scene_list?, allow_skipping?, complete_message?, add_secret_emails?, remove_secret_emails?}`.
  Secret-access list changes apply at once, without publishing.
- **Preview Image.** `update-experience-tool {id, thumbnail_asset_id}` with
  an image asset. Find one with `list-assets-tool {asset_type: "image"}`, or
  make one with the `add-assets` skill and wait until it has finished
  generating. It applies at once, without publishing. Pass null to go back
  to the first scene's backdrop.
- **Copy one.** `duplicate-experience-tool {organization?, id, new_name?}`
  makes a private, unpublished copy of every scene and layer. Pass
  `organization` when the person works in more than one. The copy can only
  land in an organization that owns the Experience or subscribes to it.
- **Roll back.** `list-experience-versions-tool {id}` lists every publish and
  restore, newest first.
  `restore-experience-version-tool {id, version_number}` replaces the draft
  with that version and saves a backup of the current draft first. Learners
  see the change only after the next publish.
- **Archive.** `delete-experience-tool {id}` removes it for learners at once,
  published version included. Only OrchestrateXR support can bring it back.
  Restate the name and confirm.

## Edit locks

Every write takes the Experience's edit lock for two minutes.

- "*Name* is currently editing the Experience" means someone else has it
  open. Tell the person who it is and wait. Don't retry in a loop.
- A warning that the Experience is open in the editor means the person
  themselves has it open. The editor saves whole scenes from memory and can
  overwrite your changes. Ask them to close or reload the editor before you
  continue.
