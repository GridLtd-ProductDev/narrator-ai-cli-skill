# Resources — Materials, BGM, Dubbing, Templates

Detailed list commands, response formats, and field mappings for the four resource types selected before any task. The selection rules (never auto-select, present 5–8 options, etc.) live in **SKILL.md § Resource Selection Protocol**.

---

## 1. Source Files (Video + SRT)

> ⚠️ **Agent behavior**: Use `material list --json --page 1 --size 100` to fetch pre-built materials. Check the `total` field — if `total > 100`, fetch additional pages until all items are retrieved. **Search programmatically using `grep -i` or `python3 -c` piped from the JSON output — do NOT rely on the terminal display, which may be truncated and miss items.** Present **all matching results** (usually ≤ 3) — show title, year, genre, summary. Wait for the user to pick one. If the user wants to upload their own files, guide them through `file upload` for both video and SRT. Do NOT proceed to writing until `video_file_id` and `srt_file_id` are confirmed.

### Option A: Pre-built materials (recommended)

```bash
narrator-ai-cli material list --json --page 1 --size 100
# If total > 100, fetch more pages: --page 2 --size 100, etc.
```

**Response structure:**

```json
{
  "total": 101,
  "page": 1,
  "size": 100,
  "items": [
    {
      "id": "<material_id>",
      "name": "极限职业",
      "title": "Extreme Job",
      "year": "2019",
      "type": "喜剧片",
      "story_info": "...",
      "character_name": "[柳承龙 (Ryu Seung-ryong), 李荷妮 (Lee Ha-nee), ...]",
      "cover": "https://...",
      "video_file_id": "<video_file_id>",
      "srt_file_id": "<srt_file_id>"
    }
  ]
}
```

**Programmatic search** (case-insensitive):

```bash
narrator-ai-cli material list --json --page 1 --size 100 | grep -i "飞驰人生"

narrator-ai-cli material list --json --page 1 --size 100 \
  | python3 -c "import json, sys; items = json.load(sys.stdin).get('items', []); \
[print(json.dumps(i, ensure_ascii=False)) for i in items if '飞驰' in i.get('name','') or '飞驰' in i.get('title','')]"
```

**Material → `confirmed_movie_json` field mapping** (construct locally — no `search-movie` needed when the material is found):

| Material field | `confirmed_movie_json` field | Notes |
|---|---|---|
| `name` | `local_title` | Chinese title |
| `title` | `title` | English title |
| `year` | `year` | |
| `type` | `genre` | e.g. `喜剧片` |
| `story_info` | `summary` | |
| `character_name` | `stars` | Parse JSON array string |
| (not in material) | `director` | Omit if unavailable |

### Option B: Upload your own

```bash
narrator-ai-cli file upload ./movie.mp4 --json           # returns file_id
narrator-ai-cli file upload ./subtitles.srt --json
narrator-ai-cli file transfer --link "<url>" --json       # transfer by HTTP/Baidu/PikPak link
```

Supported formats: `.mp4`, `.mkv`, `.mov`, `.mp3`, `.m4a`, `.wav`, `.srt`, `.jpg`, `.jpeg`, `.png`.

Full file ops (list, info, download, storage, delete) live in `operations.md`.

---

## 2. BGM (Background Music)

> ⚠️ **Agent behavior**: Infer the mood/genre from context, then use `bgm list --search "<keyword>"` to pre-filter. Present **5–8 tracks** (agent decides which fields best represent each — typically name + style). If the user has no preference, recommend **3 tracks** with a one-line reason for each (e.g., "matches the film's fast-paced action tone"). Wait for confirmation. Do NOT use a `bgm` ID in any task until the user confirms.

```bash
narrator-ai-cli bgm list --json                          # all tracks (146 currently)
narrator-ai-cli bgm list --search "单车" --json
```

The `id` field is what you pass as the `bgm` param when creating clip-data / fast-clip-data tasks.

---

## 3. Dubbing Voice

> ⚠️ **Agent behavior**: Infer the target language from context; if ambiguous, **ask the user** before listing. Run `dubbing list --lang <language>` to filter, then present **all matching voices** (typically < 15 per language) — include name and tags. If the user has no preference, recommend **3 voices** with reasoning (e.g., "neutral tone fits documentary narration"). Wait for confirmation. Do NOT use a dubbing `id` or `dubbing_type` in any task until the user confirms both.

```bash
narrator-ai-cli dubbing list --json                      # 63 voices, 11 languages
narrator-ai-cli dubbing list --lang 普通话 --json
narrator-ai-cli dubbing list --tag 喜剧 --json
narrator-ai-cli dubbing languages --json
narrator-ai-cli dubbing tags --json
```

Each voice exposes `id` (= `dubbing` param) and `type` (= `dubbing_type` param) — both are required when creating clip tasks.

**Languages available**: 普通话 (39), English (4), 日语 (3), 韩语 (2), Spanish (3), Portuguese (2), German (2), French (2), Arabic (2), Thai (2), Indonesian (2).

> ⚠️ **Language linkage** (recap from SKILL.md): If the chosen voice is non-Chinese, the writing task's `language` param **must** be set to the matching language, and magic-video template text params must also be in the matching language. See `magic-video.md` for the template-text rules.

---

## 4. Narration Style Templates (90+, 12 genres)

> ⚠️ **Agent behavior**: Infer the content genre from context and run `task narration-styles --genre <genre>` to pre-filter. Present **3–5 templates** (agent decides best representative fields). Also share the visual preview link to help the user browse:
> https://ceex7z9m67.feishu.cn/wiki/WLPnwBysairenFkZDbicZOfKnbc
> If the user has no preference, recommend **3 templates** with a brief style description and reasoning. Wait for confirmation. Do NOT use a `learning_model_id` until the user confirms.

```bash
narrator-ai-cli task narration-styles --json
narrator-ai-cli task narration-styles --genre 爆笑喜剧 --json
```

**Genres**: 热血动作, 烧脑悬疑, 励志成长, 爆笑喜剧, 灾难求生, 悬疑惊悚, 惊悚恐怖, 东方奇谈, 家庭伦理, 情感人生, 奇幻科幻, 传奇人物.

Use the template's `id` directly as `learning_model_id` — **no `popular-learning` step needed**.
