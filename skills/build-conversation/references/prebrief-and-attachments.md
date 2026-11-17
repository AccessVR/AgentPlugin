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

- Add: `add-character-attachment-tool {id, name, type, link?, asset_id?, description?, show_when?, show_for_seconds?}`.
  `type` is `link` (pass `link`) or `file` (pass the `asset_id` of an image,
  video or file Asset that belongs to the Character's organization; another
  organization's Asset is refused, even one for sale). To upload a file
  first, read `get-skill-tool {name: "add-assets"}`.
- Change: `update-character-attachment-tool {id, attachment_id, name?, link?, asset_id?, description?, show_when?, show_for_seconds?}`.
  A link can't become a file or the reverse; remove it and add a new one.
- Remove: `remove-character-attachment-tool {id, attachment_id}`. The Asset
  stays in the library.

### Showing an attachment at the right moment

`show_when` lets the Character show an attachment itself, without the learner
opening it. Write it as the moment to watch for, with "you" meaning the
Character: "the learner asks you to show how you'll put in your eye drops".
Say whose it is when it could be either person's. When that moment comes, the
attachment appears beside the Character (over its video on a phone) while the
conversation carries on, so the Character can talk about it. A video plays
with its sound off and closes when it ends. Anything else closes after
`show_for_seconds`: 30 unless set, from 5 to 600, or `null` to keep it up until
the learner closes it. Pass `show_when: null` to stop the Character showing it.

- Suits a video, an image or a PDF. A link the browser can't show in a page
  only offers to open in a new tab.
- Only the web player does this. Ask the human to try the moment in a preview
  conversation: whether the Character spots it depends on how plainly the
  moment is described.

Attachment ids come from `get-character-tool`. Attachment changes reach the
next conversation straight away, with nothing to publish, so confirm each one
by name first.
