# Prebrief videos and attachments

## Prebrief video

A prebrief is a screen shown before the conversation starts: HTML content, a
video, or both. A Replica can deliver the video. Generating it spends
organization tokens by the length of the script.

1. To have the Character itself deliver it, get its Replica id from
   `get-character-tool {id}`. Any Replica from `list-replicas-tool` works too.
2. Agree the script (up to 5000 characters), the Replica and the voice:
   - By default the script is spoken in the Replica's linked Voice, or in the
     Replica's own voice when it has none.
   - `voice_id` picks another Voice (`list-voices-tool`).
     `use_replica_voice` forces the Replica's own voice.
   - Delivery tags such as `[whispers]` or `[excited]` direct a Voice. The
     Replica's own voice reads them aloud, so leave them out then. A Voice's
     narration must run at least 3 seconds.
3. Restate the Replica, the voice and the script, say it spends tokens, and get
   a yes. Then call
   `generate-replica-video-tool {organization?, replica_id, script, name?, voice_id?, use_replica_voice?, transparent_background?}`.
   It returns an asset `id` in state `generating`. A Replica still being
   prepared, or an organization without enough tokens, is refused before
   anything is spent.
4. Poll `get-asset-generation-status-tool {ids}` several seconds apart until
   `all_done` is true, which means the video is `processed`. Videos take
   several minutes. On failure, read `failure_reason` before retrying; the
   reserved tokens come back.
5. Attach it to the Character
   `update-character-tool {id, prebrief_enabled, prebriefing_video_asset_id, prebriefing_content?}`
   or to the Thread
   `update-thread-tool {id, prebrief_enabled, prebriefing_video_asset_id, prebriefing_content?}`.
   Any video Asset works here, not only a generated one.

A Character with the prebrief on and neither a video nor content can't be
published. When both a Thread and a Character in it have a prebrief, both show,
the Thread's first; turn the Character's off in that slot with `show_prebrief`.

## Attachments

Links or files handed to the learner during the conversation.

- Add: `add-character-attachment-tool {id, name, type, link?, asset_id?, description?}`.
  `type` is `link` (pass `link`) or `file` (pass the `asset_id` of an image or
  file Asset that belongs to the Character's organization; videos are
  refused, and so is another organization's Asset, even one for sale). To
  upload a file first, read
  `get-skill-tool {name: "add-assets"}`.
- Change: `update-character-attachment-tool {id, attachment_id, name?, link?, asset_id?, description?}`.
  A link can't become a file or the reverse; remove it and add a new one.
- Remove: `remove-character-attachment-tool {id, attachment_id}`. The Asset
  stays in the library.

Attachment ids come from `get-character-tool`. Attachment changes reach the
next conversation straight away, with nothing to publish, so confirm each one
by name first.
