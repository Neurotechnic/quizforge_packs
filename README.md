# QuizForge pack format

This app loads quiz packs from a `pack.json` file plus optional media files. Use this guide to author your own packs.

## Folder layout

```
├─ pack.json              # required
└─ media/                 # optional, referenced by question.media
   ├─ image1.png
   └─ ...
```

Zip the folder to import it in the app (“Add custom pack (.zip)”).

## pack.json (top-level)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `schemaVersion` | int | yes | Current app expects the latest version used in sample packs. |
| `id` | string | yes | Unique pack ID. |
| `title` | string | yes | Pack name shown in UI. |
| `description` | string | no | Short description. |
| `language` | string | no | Language code/name, used for filtering. |
| `tags` | array\<string> | no | Tags for filtering (case-sensitive match). |
| `timeLimitMinutes` | int | no | Whole-pack time limit (minutes). App can auto-finish or keep going after timeout. |
| `groups` | array\<Group> | yes | See below. |
| `questions` | array\<Question> | yes | See below. |

### Groups

```
{
  "id": "net",
  "title": "Networking",
  "questionIds": ["q1", "q2"]
}
```

Groups define subsets of questions (the app lets users pick groups). Order of `questionIds` is preserved unless “Shuffle questions” is enabled in the app.

### Questions (common fields)

```
{
  "id": "q1",
  "type": "singleChoice" | "multiChoice" | "textInput" | "numberInput" | "order",
  "prompt": { "text": "Your question text" },
  "media": "media/image1.png",      # optional, path relative to pack.json
  "score": { "max": 1.0 },          # defaults to 1.0
  "data": { ... }                   # type-specific payload
}
```

You can add an explanation string with `data.explain` or `data.explanation` (shown after answering).

### Question types

**singleChoice**
```json
{
  "options": [
    { "id": "a", "text": "Option A", "explain": "Why A is right" },
    { "id": "b", "text": "Option B" }
  ],
  "correctOptionId": "a",
  "shuffleOptions": true   // optional, shuffles options; alias: "shuffle": true
}
```

**multiChoice**
```json
{
  "options": [
    { "id": "a", "text": "Option A" },
    { "id": "b", "text": "Option B" },
    { "id": "c", "text": "Option C" }
  ],
  "correctOptionIds": ["a", "c"],
  "scoring": { "penalizeWrong": true },  // optional, default true
  "shuffleOptions": true                 // optional, alias: "shuffle": true
}
```

**textInput**
```json
{
  "accepted": ["TCP", "tcp"],  // array of accepted answers
  "trim": true,                // optional, default true
  "caseSensitive": false       // optional, default false
}
```

**numberInput**
```json
{
  "correct": 42,   // number
  "tolerance": 0   // optional, default 0 (exact match); set >0 to allow range
}
```

**order**
```json
{
  "items": [
    { "id": "dns", "text": "DNS" },
    { "id": "http", "text": "HTTP" },
    { "id": "tcp", "text": "TCP" }
  ],
  "correctOrder": ["dns", "tcp", "http"],
  "shuffle": true   // optional, shuffles items initially
}
```

## Shuffling behavior
- App toggle “Shuffle questions” randomizes question order (within selected groups) per session.
- Per-question shuffling:
  - `shuffle` or `shuffleOptions` true shuffles options.
  - `shuffle` true shuffles `items` for order questions.

## Time limits
- `timeLimitMinutes` in `pack.json` sets a limit for the whole pack.
- In the app, users choose whether the session auto-finishes at timeout or continues (timer turns red and results mark “Time limit exceeded”).

## Minimal example

```json
{
  "schemaVersion": 1,
  "id": "demo_pack",
  "title": "Demo Pack",
  "description": "Small sample quiz",
  "language": "en",
  "tags": ["demo", "networking"],
  "timeLimitMinutes": 10,
  "groups": [
    { "id": "all", "title": "All Questions", "questionIds": ["q1", "q2"] }
  ],
  "questions": [
    {
      "id": "q1",
      "type": "singleChoice",
      "prompt": { "text": "What does TCP stand for?" },
      "data": {
        "options": [
          { "id": "a", "text": "Transmission Control Protocol" },
          { "id": "b", "text": "Transfer Cable Package" }
        ],
        "correctOptionId": "a",
        "shuffleOptions": true
      }
    },
    {
      "id": "q2",
      "type": "textInput",
      "prompt": { "text": "Default HTTPS port?" },
      "data": { "accepted": ["443"], "trim": true }
    }
  ]
}
```

Happy quiz building! For more sample packs and schemas, see https://github.com/Neurotechnic/quizforge_packs.
