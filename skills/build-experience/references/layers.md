# Layers, hotspots, questions and actions

Everything `add-layer-tool` and `update-layer-tool` accept. Both take the
layer's settings inside one `properties` object. A setting passed beside
`properties` instead of inside it is refused.

Write keys in snake_case. The camelCase spelling `get-scene-tool` returns
inside hotspots, questions, answers and actions (`sceneId`, `answerText`,
`backgroundColor`) is accepted too, so a list read from a scene can be sent
back unchanged. A key the tools don't know is refused, at any level, and the
refusal lists the keys they take, so fix the spelling and send it again.

## Layer types

`event_type` picks the layer type. Any other value is refused, and a layer's
type can't be changed later: delete it and add a new one.

| event_type | Layer | Starts with | Default timing | Pauses the scene |
| --- | --- | --- | --- | --- |
| 2 | Text card | empty title and description | 4 seconds | no |
| 3 | Question | no questions | no duration | yes |
| 5 | Hotspots | no hotspots, visible | 30 seconds | yes |
| 6 | Media | no asset | no duration | yes |

A question layer with no questions, or a question with no answers, blocks
publishing. Pass the `questions` or `hotspots` list in the same call.

## Layer settings

| Key | Meaning |
| --- | --- |
| `name` | The layer's name in the editor. Validation problems quote it, so name every layer. |
| `title` | The card's heading. |
| `description` | The card's body, in HTML: `<p>`, `<strong>`, `<ul>`, `<a>`. Plain newlines don't break lines. |
| `button_text` | The card's button. Defaults to "Continue". |
| `start_time` | Seconds into the scene when the layer appears. 0 or more. |
| `duration` / `end_time` | How long it stays, or when it goes. Pass one; the other is computed. Both together must agree. `duration` may be null. |
| `pause_playback` | Hold the scene's timeline while the layer is up. |
| `can_be_skipped` | Let the learner skip a media layer that pauses playback. |
| `mute_audio` | Mute a media layer's own sound (the editor's Media Volume). |
| `hide_captions` | `true` stops audio and video showing the asset's automatic captions. |
| `asset_id` | The Asset a media layer plays or shows. `null` detaches it. |
| `display_type` | Where the layer renders. See the next table. |
| `positioning` | HUD cards only: 0 to 8, read top-left to bottom-right row by row. 4 (centered) is the default. |
| `card_size` | HUD cards only: 0 small, 1 medium (default), 2 large. |
| `backdrop_blur` | Blur the scene behind a HUD card. On by default. |
| `background_color`, `button_color`, `button_label_color` | Card colors, as hex such as `#FF6600`. |
| `position`, `rotation`, `scale` | Worldspace placement. `scale` is a vector `{X, Y, Z}` of multipliers. |
| `sprite_mode` | Worldspace layers face the camera unless this is `false`. `rotation` only applies when it is `false`. |
| `spatial` | Media with `display_type` 2: play the audio from `position`. |
| `hotspots` | Hotspot layers: the full list, each existing hotspot with its `id`. Replaces the existing list. |
| `hotspot_visibility` | Hotspot layers: `visible` or `hidden`. Hidden hotspots must be found by looking around. |
| `always_show_labels` | Hotspot layers: show every hotspot's label. |
| `questions` | Question layers: the full list, each existing question with its `id` (answers are recorded against it). Replaces the existing list. |
| `acknowledge_action` | Question layers only: an action that runs once the learner has answered and moved on (the editor's On Acknowledge). Any type but 7. `null` removes it. |

`display_type` by layer type. Any other combination is refused:

| Layer | 0 | 1 | 2 | 3 |
| --- | --- | --- | --- | --- |
| Text card | HUD card | worldspace | | |
| Question | HUD card | worldspace | | answers as hotspots |
| Media | HUD card | worldspace | hidden (audio only) | |
| Hotspots | always 0 | | | |

### Placing things in the scene

Positions are in meters from the viewer, with uppercase axes `{X, Y, Z}`.
Lowercase axes are converted. -Z is in front of the viewer at the start, +X to
the right, -X to the left, +Z behind. Y is height, with 0 at the viewer's eye
level and positive values above it. Two to three meters away reads well, for
example `{"X": 0, "Y": 0, "Z": -2}`.

Layer `scale` is a vector `{X, Y, Z}`. Hotspot `scale` is a single number.
Each is refused in the other's shape.

## Hotspots

A hotspot is `{name, position, scale, icon, background_color, icon_color,
action}`.

- `position` is required. A hotspot without one is refused.
- `scale` is one number, 1 by default.
- `icon` is `{"id": "<name>"}`, a Font Awesome 6 Free solid icon name such as
  `circle-info`, `door-open`, `arrow-right`, `triangle-exclamation`,
  `fire-extinguisher`, `stethoscope`, `play` or `check`. Other names are
  refused. The default is `question`.
- `action` says what clicking it does. A hotspot whose action type is 0, or
  that has no action, blocks publishing.

## Questions

A question is `{text, question_type, button_text, remediate, assessable,
require_correct_answer, allow_change_answer, hotspot_visibility, show_labels,
answers}`. Only `text` and `answers` are required.

| Key | Default | Meaning |
| --- | --- | --- |
| `text` | | The prompt the learner reads. Required. |
| `question_type` | 0 | 0 = one correct answer. 1 = several correct ("mark all that apply"). Free-text questions don't exist. |
| `button_text` | Submit | The submit button. |
| `remediate` | true | Show the correct answer after the learner responds. |
| `assessable` | true | Count the question toward the score. |
| `require_correct_answer` | false | Keep the learner here until they answer correctly. |
| `allow_change_answer` | true | Let the learner change their answer. |
| `hotspot_visibility` | visible | With `display_type` 3: `hidden` answers must be found by looking around. |
| `show_labels` | true | With `display_type` 3: show the answer labels. |

An answer is `{answer_text, correct_answer, action, icon, position,
background_color, icon_color}`. `answer_text` is required: it is the choice
the learner sees. `correct_answer` defaults to false. An answer's `action`
fires when the learner picks it, except on `question_type` 1, where answer
actions never fire. With `display_type` 3 every answer is drawn as a hotspot,
so every answer needs a `position`.

## Actions

Hotspots, answers, a question layer's `acknowledge_action` and a scene's end
use one action shape: `{type, scene_id, time_stamp, preserve_view, event,
url}`.

| type | Does | Needs |
| --- | --- | --- |
| 0 | Nothing. Fine on an answer; blocks publishing on a hotspot. | |
| 1 | Completes the Experience. | |
| 3 | Goes to another scene. | `scene_id` of a scene in the same Experience. Optional `time_stamp` (seconds to start at) and `preserve_view` (keep the learner facing the way they are instead of the scene's initial view). |
| 4 | Shows a card. | `event`: a nested layer with `event_type` 3 or 6. See below. |
| 6 | Resumes the scene's playback. | |
| 7 | Opens a web page in a new tab while the Experience pauses. | `url`: an http or https address, `mailto:` or `tel:`. An address without a scheme is treated as https. |

The tools also accept 2 and 5, which the editor doesn't offer on hotspots or
answers. Don't use them there.

A scene's end action (`update-scene-tool {id, end_action}`) takes only 0, 1,
3 or 5. 5 replays the scene from the beginning. 0 leaves the learner frozen on
the last frame with no completion recorded. New scenes end with 1.

### Show-card cards

A show-card `event` is a whole layer nested inside the action, at most three
levels deep. The editor offers a media card or a question card there. A text
card, or a nested hotspot layer, is refused because the player can't dismiss
it.

The card takes the same properties as a layer, in the same vocabulary:
`title`, `description`, `asset_id`, `questions` and the rest. It gets the
defaults the editor gives a new card, so set only what differs. A question
card's answers can open further cards, down to the three-level limit.
`get-scene-tool` returns a card deeper than that as `{id, truncated: true}`.
Send that placeholder back unchanged and the stored card is kept, so the rest
of the layer can still be edited. To change the deep card itself, point the
person to the Experience Editor.

Read the scene with `get-scene-tool` afterwards to check the card.

## Recipes

Each is the `arguments` of one call, run with `execute_write_tools`.

A text card in the middle of the view for ten seconds:

```json
{"scene_id": 41, "event_type": 2, "properties": {"name": "Welcome", "title": "Welcome to the ward", "description": "<p>Look around, then press <strong>Continue</strong>.</p>", "start_time": 0, "duration": 10, "pause_playback": true}}
```

Hotspots that move the learner between scenes:

```json
{"scene_id": 41, "event_type": 5, "properties": {"name": "Choose a room", "start_time": 0, "hotspots": [
  {"name": "Nurses' station", "position": {"X": -2, "Y": 0, "Z": -2}, "icon": {"id": "door-open"}, "action": {"type": 3, "scene_id": 42}},
  {"name": "Finish", "position": {"X": 2, "Y": 0, "Z": -2}, "icon": {"id": "check"}, "action": {"type": 1}}
]}}
```

A search task: the same hotspot layer with `"hotspot_visibility": "hidden"`.
Each hotspot still needs an action; type 6 resumes the scene once it's found.

A knowledge check:

```json
{"scene_id": 41, "event_type": 3, "properties": {"name": "Hand hygiene check", "start_time": 20, "questions": [
  {"text": "When must you clean your hands?", "question_type": 1, "answers": [
    {"answer_text": "Before touching a patient", "correct_answer": true},
    {"answer_text": "After touching patient surroundings", "correct_answer": true},
    {"answer_text": "Only when gloves are off", "correct_answer": false}
  ]}
]}}
```

Answers placed in the scene as hotspots: the same question layer with
`"display_type": 3` and a `position` on every answer.

A video panel standing in the scene: `event_type` 6 with
`{"display_type": 1, "asset_id": 77, "position": {"X": 0, "Y": 0, "Z": -3},
"scale": {"X": 1, "Y": 1, "Z": 1}}`.

Sound coming from a point in the scene: `event_type` 6 with
`{"display_type": 2, "spatial": true, "asset_id": 78, "position": {"X": 3,
"Y": 0, "Z": 0}}`, where asset 78 is audio.

A hotspot that opens a page: `"action": {"type": 7, "url":
"https://example.com/policy"}`.

A hotspot that shows a video card:

```json
{"name": "Watch the demo", "position": {"X": 0, "Y": 0.5, "Z": -2}, "icon": {"id": "play"}, "action": {"type": 4, "event": {"event_type": 6, "name": "Demo", "title": "Demo", "description": "<p>Watch, then continue.</p>", "button_text": "Continue", "asset_id": 77}}}
```

## Timing

`start_time` counts from the start of the scene's timeline. An image scene
runs 30 seconds unless its `end_time` was changed, and a video scene runs as
long as the video. When the scene reaches its end, its end action runs, so a
layer that starts after the scene's end never appears. Lengthen the scene
with `update-scene-tool {id, end_time}` when layers need more time. Layers
that pause playback hold the timeline until the learner moves on.
