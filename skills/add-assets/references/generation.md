# Generating images and Replica videos

Both spend the organization's tokens. Confirm before every call, restating
what will be made and that it spends tokens. The balance is in
`list-organizations-tool {scope: "mine"}` under `consumables.tokens.balance`,
for people who manage the organization.

Both generate tools take `organization` and run with `execute_write_tools`.
The list, get and status tools run with `execute_read_tools`.

## Generate an image or a 360° backdrop

`generate-asset-image-tool {organization?, prompt, panorama?, resolution?, aspect?, count?, source_asset_ids?}`

- `panorama: true` makes a 360° image for a scene backdrop. It always
  renders at 4K, the most expensive tier.
- A flat image takes `resolution` `1K` (default and cheapest), `2K` or `4K`,
  and `aspect` `1:1` (default), `3:4`, `4:3`, `9:16` or `16:9`.
- `count` makes 1 to 4 variations, each charged.
- `source_asset_ids` lists up to 5 library images to base the result on.

Tokens are reserved when you call. State the count, the resolution and
whether it's a panorama, and get a yes. "Not enough tokens" leaves nothing
behind; tell the person and don't retry.

Then `get-asset-generation-status-tool {ids}` with the returned `ids`, every
several seconds, until `all_done`. A failed image releases its tokens. Read
`failure_reason` before trying again. This tool counts `ready` as done, so
check `get-asset-tool` for `processed` before publishing content that uses it.

## Generate a Replica video

1. Pick the Replica with `list-replicas-tool {search?}`, or use the Replica
   on `get-character-tool {id}` to have a Character deliver it. Pick a Voice,
   if needed, with `list-voices-tool {search?}`.
2. Agree the script (up to 5,000 characters), the Replica and the voice. The
   script is read in the Replica's linked Voice when it has one, otherwise in
   its own voice. `voice_id` picks another Voice; `use_replica_voice: true`
   uses the Replica's own voice. Tags such as `[whispers]` direct a Voice,
   but the Replica's own voice reads them aloud. A Voice's narration must
   run at least 3 seconds.
3. Tokens are charged by the script's length. Restate the Replica, the voice
   and the script, say it spends tokens, and get a yes.
4. `generate-replica-video-tool {organization?, replica_id, script, name?, voice_id?, use_replica_voice?, transparent_background?}`.
5. Poll `get-asset-generation-status-tool {ids}` with the returned `id`. It
   takes several minutes.

"Your organization does not have enough tokens available" is refused before
anything is spent. Tell the person, and don't retry. A Replica still being
prepared is refused too.

To use the video as a prebrief, pass its id as `prebriefing_video_asset_id`
to `update-character-tool` or `update-thread-tool`.

