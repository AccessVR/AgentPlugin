# Generating images, videos, narration and Replica videos

All four spend the organization's tokens. Confirm before every call, restating
what will be made and that it spends tokens. The balance is in
`list-organizations-tool {scope: "mine"}` under `consumables.tokens.balance`,
for people who manage the organization.

Every generate tool takes `organization` and runs with `execute_write_tools`.
The list, get and status tools run with `execute_read_tools`.

## Generate an image or a 360° backdrop

`generate-asset-image-tool {organization?, prompt, panorama?, resolution?, aspect?, count?, source_asset_ids?}`

- `panorama: true` makes a 360° image for a scene backdrop. It always
  renders at 4K, the most expensive tier, and at 2:1, so leave `aspect` out.
- A flat image takes `resolution` `1K` (default and cheapest), `2K` or `4K`,
  and `aspect` `1:1` (default), `2:3`, `3:2`, `3:4`, `4:3`, `9:16`, `16:9`
  or `21:9` (ultra-wide).
- `count` makes 1 to 4 variations, each charged.
- `source_asset_ids` lists up to 5 library images to base the result on. A
  panorama takes at most 1.

Tokens are reserved when you call. State the count, the resolution and
whether it's a panorama, and get a yes. "Not enough tokens" leaves nothing
behind; tell the person and don't retry.

Then `get-asset-generation-status-tool {ids}` with the returned `ids`, every
several seconds, until `all_done`. An image is done once it's `processed`,
which is what publishing needs; `ready` shows in the editor but is still
processing. A failed image releases its tokens. Read `failure_reason` before
trying again.

## Generate a video

A short video made from a prompt: a forklift crossing a warehouse, a dock
door rolling up. It can start from a library image and land on another. It
lands in the library as a flat `video` asset (not 360°) to play on a screen in
a scene, as a media layer, or in a Guide.

`generate-asset-video-tool {organization?, prompt, model?, resolution?, duration?, aspect?, sound?, start_asset_id?, end_asset_id?}`

- `model` is `kling-v3` (default), `kling-v3-turbo` or `kling-v4`.
  `kling-v3` alone can be silent and make `4k`; `kling-v4` alone runs past
  15 seconds, up to 30, and makes `21:9`.
- `resolution` is `720p`, `1080p` (default) or `4k`. `duration` is whole
  seconds, 3 to 15 (30 for `kling-v4`), default 5.
- `aspect` is `16:9` (default), `9:16` or `1:1`. A start frame sets the shape,
  so leave `aspect` out with one.
- `start_asset_id` and `end_asset_id` are library images. An end frame needs
  a start frame, and `kling-v3-turbo` doesn't take one.
- The prompt describes what happens: the subject, the action, how the camera
  moves, the light and the sound.

The price is known before it starts, from the model, resolution, sound and
length. A 10-second 1080p `kling-v3` video with sound is 48 tokens; a
30-second `kling-v4` at 1080p is 202. State the
model, resolution, length, sound and the token cost, and get a yes. If you
don't know the cost, call once the person has agreed to the settings, and
report the `tokens_reserved` it returns.

"AI video generation is turned off" means the organization has switched it
off; an organization admin can turn it back on. Don't retry.

Then `get-asset-generation-status-tool {ids}` with the returned `id`, every
30 seconds or so, until `all_done`. A video takes several minutes, and a long
one more. A failed video releases its tokens; read `failure_reason` before
trying again.

## Generate narration

Narration is a script read aloud in a Voice: a voiceover for a scene, a
narrated intro, spoken instructions. It lands in the library as an `audio`
asset.

1. Pick the Voice with `list-voices-tool {search?}`. Only a ready, active
   Voice is accepted.
2. Agree the script (up to 5,000 characters) and the Voice. Tags such as
   `[whispers]` or `[excited]` direct the reading.
3. It costs about 6 tokens per 1,000 characters of the script, and at least
   1. Restate the Voice and the script, say it spends tokens, and get a yes.
   The tool returns the exact count as `tokens_charged`.
4. `generate-narration-tool {organization?, voice_id, script, name?}`.
5. The tokens are charged once the audio is spoken. "Not enough tokens", or
   a reading that fails, leaves nothing behind and charges nothing; tell the
   person, and don't retry the tokens refusal.
6. The audio is usually `processed` when the tool returns. If `state` is
   still `processing`, poll `get-asset-generation-status-tool {ids}` with the
   returned `id`.

To play it in an Experience, add it to a scene as an audio layer with
`add-layer-tool` (see the build-experience skill).

## Generate a Replica video

1. Pick the Replica with `list-replicas-tool {search?}`, or use the Replica
   on `get-character-tool {id}` to have a Character deliver it. Pick a Voice,
   if needed, with `list-voices-tool {search?}`.
2. Agree the script (up to 5,000 characters), the Replica and the voice. The
   script is read in the Replica's linked Voice when it has one, otherwise in
   its own voice. `voice_id` picks another Voice; `use_replica_voice: true`
   uses the Replica's own voice. To change the Replica's linked Voice for
   every video, its owners can use `update-replica-tool {id, voice_id}`.
   Tags such as `[whispers]` direct a Voice, but the Replica's own voice
   reads them aloud. A Voice's narration must run at least 3 seconds.
3. Tokens are charged by the script's length. Restate the Replica, the voice
   and the script, say it spends tokens, and get a yes.
4. `generate-replica-video-tool {organization?, replica_id, script, name?, voice_id?, use_replica_voice?, transparent_background?}`.
5. Poll `get-asset-generation-status-tool {ids}` with the returned `id` until
   `all_done`. It takes several minutes, and the video is `processed` when it
   finishes.

"Your organization does not have enough tokens available" is refused before
anything is spent. Tell the person, and don't retry. A Replica still being
prepared is refused too.

To use the video as a prebrief, pass its id as `prebriefing_video_asset_id`
to `update-character-tool` or `update-thread-tool`.

