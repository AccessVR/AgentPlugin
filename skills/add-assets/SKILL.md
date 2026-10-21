---
name: add-assets
description: Get images, 360° panoramas, video, audio and documents into the OrchestrateXR asset library. Covers uploading a file from the chat or computer, importing one from a public link, generating an AI image or a 360° skybox, generating narration (text-to-speech voiceover audio) from a script, and generating a Replica video from a script. Use when someone needs a scene backdrop, a voiceover or any media file, wants to find an asset, check that it finished processing, share it, see where it is used, or recycle it.
metadata:
  requires: authoring
---

# Add assets

An Asset is one file in an organization's library. Its `asset_type` is one
of `image`, `video`, `audio` or `file` (documents and anything else). A video
also has a `video_type` saying how it is shot: `360_video`, `360_3D_video`,
`360_3D_video_sbs`, `180_video`, `180_3D_video`, `180_3D_video_tab`,
`2D_video`, `3D_video_sbs` or `3D_video_tab`.

Every `list-` and `get-` tool here runs with `execute_read_tools`. Every other
tool runs with `execute_write_tools`.

## Before you start

- **Organization.** The tools that add an asset take `organization` (id or
  exact name). If the person belongs to more than one organization and hasn't
  said which, ask, then pass it on every one. The asset can only be used in
  that organization's content.
- **Confirm every write.** Say what will be added or changed and get a yes.
  Restate the asset by name before recycling it, and before anything that
  spends tokens.
- **Links.** Share only URLs a tool returned. Never build one from an id.
- **"Wasn't found, or isn't available."** Check the id with
  `list-assets-tool`. Don't repeat the call.

## Find an existing asset first

`list-assets-tool {search?, asset_type?, video_type?, state?, organization?, limit?, cursor?}`

- `search` matches part of the name, or an exact id.
- `asset_type` takes `image`, `video`, `audio` or `file`. A 360° or 180°
  video is still `video`; filter by its shape with `video_type`, such as
  `360_video`.
- `state: processed` keeps to assets ready to use.
- Results come 25 at a time, most recently changed first. Pass `next_cursor`
  back as `cursor` for more.

A 360° image is an ordinary `image`. `get-asset-tool {id}` gives its `width`
and `height`; a 360° panorama is twice as wide as it is tall.

## Pick a way in

| The file | Do this |
| --- | --- |
| Is at a public web address | Import it from the link. |
| Is attached to the chat or on the machine you run on, and you can make HTTP requests | Upload it in three steps. |
| Is attached, but you can't make HTTP requests | Ask for a public link to import, or ask the person to upload it on the Assets screen, then find it with `list-assets-tool`. |
| Doesn't exist yet: an image or a 360° backdrop | Generate it. |
| Is a script read aloud: narration or a voiceover | Generate narration. |
| Is a person speaking a script to camera | Generate a Replica video. |

You can't make HTTP requests when you have no shell, code sandbox or fetch
tool that can reach the upload address, as in a plain chat on claude.ai.
Never pass a file's contents through a tool argument.

## Import from a link

`create-asset-from-url-tool {organization?, url, name?, asset_type?, video_type?}`

- The address must be public http or https on a standard port, and must
  download the file itself, not a page about it.
- The type comes from the address's extension: `.jpg .jpeg .png .gif .heic
  .svg` are images, `.mp4 .mov .3gp .3g2` are video, `.mp3 .wav .m4a` are
  audio, and anything else is a file. Pass `asset_type` when the address has
  no extension.
- A video needs `video_type`. Ask how it was shot if it isn't clear.
- The asset comes back in state `uploading`. Wait for it (below).

## Upload a file

1. `start-asset-upload-tool {organization?, file_name, name?, asset_type?, video_type?}`
   returns `asset_id`, `upload_url`, `headers` and `expires_at`.
   `file_name` needs its extension. A video needs `video_type`.
2. Send the file's raw bytes to `upload_url` in an HTTP PUT with exactly the
   returned headers. No form encoding and no sign-in header:

   ```
   curl -X PUT --upload-file ./ward-360.jpg "<upload_url>"
   ```

   Add `-H "Name: value"` for each returned header. The address expires
   after 60 minutes; start again after that.
3. `finish-asset-upload-tool {id}`, passing the `asset_id` as `id`. It has no
   `asset_id` argument. It refuses until the bytes have arrived.
4. Wait for processing (below).

Until step 3 the asset is hidden: list and get tools don't find it.

## Wait for processing

Check `get-asset-tool {id}` every several seconds. Images usually finish in
under a minute, videos in several minutes. Tell the person it's processing
rather than polling for a long time in silence.

| `state` | Meaning |
| --- | --- |
| `uploading`, `storing`, `processing`, `generating` | Still working. Wait. |
| `ready` | Usable in the editor, but processing continues. |
| `processed` | Done. Publishing an Experience needs every asset it uses here. |
| `failed` | Stopped. `state_message` says why. Offer the person a support ticket. |

## Generate an image, a 360° backdrop, narration or a Replica video

`generate-asset-image-tool` makes flat images and 360° panoramas for scene
backdrops. `generate-narration-tool` reads a script aloud in a Voice and
saves it as an audio asset. `generate-replica-video-tool` makes a
talking-head video of a Replica reading a script. All three spend tokens and
are checked with `get-asset-generation-status-tool`. Read
`references/generation.md` before calling any of them.

## Details, sharing and links

`get-asset-tool {id}` returns the state, type, size, dimensions, duration,
tags, owning organizations and links. `url` opens the asset's page for people
who can see it. `share_url` is the link to send, given only to people who may
share the asset. When it is null, say only someone who can share the asset
can get it, and never offer `url` instead. No tool returns a download link
for an asset's file.

`update-asset-tool {id, name?, description?, tags?, visibility?, add_secret_emails?, remove_secret_emails?}`
renames, describes, retags or shares an asset. `tags` replaces the whole
list. `visibility` is `private`, `public`, `unlisted` or `secret`. Sharing
changes apply at once, and only after processing has finished. An asset
still uploading, processing or generating can't be edited.

## Where an asset is used

`list-experiences-tool {asset_id}` finds the Experiences whose draft uses the
asset in a layer, as a scene backdrop, or as the preview image. An asset
removed from the draft but still in the published version isn't matched.

## Recycle and restore

`delete-asset-tool {id, force?}` checks Experiences, Characters, Threads and
Courses first and refuses with the list of places still using the asset.
Places the person can't see are counted, not named. Pass `force: true` only
after restating that list and getting a yes: those places lose the media.
`restore-asset-tool {id}` brings a recycled asset back.
