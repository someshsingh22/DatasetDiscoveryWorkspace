# Reddit corpus layout

The pre-filtered corpus is available inside the workspace at `data/corpus/`.
Files are UTF-8 JSONL compressed with Zstandard (`.zst`).

Records from NSFW subreddits were removed during preprocessing. Two kinds of
data are present:

- Per-subreddit full history for 14 target subreddits (files prefixed
  `FULL_`).
- A broad Reddit comment activity snapshot for April–May 2020 (files prefixed
  `RS_2020-04` and `RS_2020-05`).

Subreddit metadata is from January 2025.

## Directories

```text
data/corpus/
├── comments/
│   ├── FULL_<subreddit>_comments.jsonl.zst      # full history, 14 subreddits
│   ├── RS_2020-04_*.jsonl.zst                    # Apr 2020 activity snapshot
│   ├── RS_2020-05_*.jsonl.zst                    # May 2020 activity snapshot
│   └── .subreddit_shard_index.json              # subreddit -> shard file map
├── submissions/
│   └── FULL_<subreddit>_submissions.jsonl.zst   # full history, 14 subreddits
└── subreddits_meta/
    ├── subreddits_2025-01.zst                    # full community metadata
    ├── subreddits_meta_only_2025-01.zst          # slim id/name/activity index
    ├── subreddit_rules_2025-01.zst               # community rules
    └── subreddit_wikis_2025-01.zst               # wiki pages
```

The 14 target subreddits are: `badscience`, `changemyview`, `charts`,
`coolguides`, `design_critiques`, `graphic_design`, `Infographics`,
`LifeProTips`, `PhotoshopTutorials`, `propaganda`, `PropagandaPosters`,
`tutorials`, `UsefulCharts`, `visualization`.

The `RS_2020-*` comment shards (roughly 1,900 files) are sharded by an opaque
suffix (`RS_2020-04_aa`, `RS_2020-04_ab`, ...) and are not aligned
record-for-record with any other file. Prefer the shard index below over
scanning every shard.

## Shard index

`comments/.subreddit_shard_index.json` maps a lowercased subreddit
`display_name` to the list of comment shard files that contain records for that
subreddit:

```json
{"infographics": ["FULL_Infographics_comments.jsonl.zst", "RS_2020-04_ae.jsonl.zst", "..."]}
```

Use it to find the shards for a subreddit before reading, instead of opening
all files in `comments/`.

## Comments

Each record in `comments/` is one Reddit comment. The two file families carry
different field sets.

`FULL_<subreddit>_comments.jsonl.zst` (full per-subreddit history) contains a
reduced schema:

| Field | Type | Description |
|---|---|---|
| `id` | string | Base-36 comment ID |
| `link_id` | string | Parent post fullname: `t3_<id>` |
| `parent_id` | string | Immediate post or comment parent |
| `body` | string | Comment text |
| `author` | string | Reddit username |
| `author_flair_text` | string/null | Community-assigned author flair |
| `subreddit` | string | Community display name |
| `subreddit_id` | string | Community fullname |
| `score` | integer | Net upvotes |
| `controversiality` | integer | Reddit controversy flag |
| `created_utc` | integer | Unix timestamp |
| `distinguished` | string/null | Moderator/admin distinction |
| `gilded` | integer | Gild count |

`RS_2020-*_*.jsonl.zst` (activity snapshot) adds account, community, and award
fields on top of the reduced schema:

| Field | Type | Description |
|---|---|---|
| `author_fullname` | string/null | Internal user ID |
| `author_created_utc` | integer/null | Account creation timestamp |
| `author_patreon_flair` | boolean | Patreon flair flag |
| `author_premium` | boolean | Reddit Premium flag |
| `is_submitter` | boolean | Whether commenter authored the post |
| `subreddit_name_prefixed` | string | Display name with `r/` prefix |
| `subreddit_type` | string | Public/restricted/private |
| `all_awardings` | array | Award objects |
| `awarders` | array | Awarding users |
| `gildings` | object | Awards by type |
| `total_awards_received` | integer | Total awards |
| `top_awarded_type` | string/null | Highest award type |
| `associated_award` | object/null | Associated award |

## Submissions

Each record in `submissions/` is one thread root. Only `FULL_` per-subreddit
files exist (there is no submission activity snapshot).

| Field | Type | Description |
|---|---|---|
| `id` | string | Base-36 post ID |
| `title` | string | Post title |
| `selftext` | string/null | Text-post body |
| `url` | string | External URL or permalink |
| `is_self` | boolean | Whether this is a text post |
| `author` | string | Submitter username |
| `author_flair_text` | string/null | Submitter flair |
| `link_flair_text` | string/null | Post flair |
| `subreddit` | string | Community display name |
| `subreddit_id` | string | Community fullname |
| `subreddit_name_prefixed` | string | Display name with `r/` prefix |
| `subreddit_type` | string | Public/restricted/private |
| `score` | integer | Net upvotes |
| `num_comments` | integer | Reply count |
| `over_18` | boolean | Always false in this corpus |
| `created_utc` | integer | Unix timestamp |
| `distinguished` | string/null | Moderator/admin distinction |
| `gilded` | integer | Gild count |

Within a target subreddit, comments link to submissions with:

```python
post_id = comment["link_id"][3:]
assert comment["link_id"] == "t3_" + submission["id"]
```

Submissions are only available for the 14 target subreddits, so many
`RS_2020-*` snapshot comments reference posts that are not present in
`submissions/`.

## Subreddit metadata

`subreddits_2025-01.zst` holds one community per record with the full Reddit
`about` payload (roughly 100 fields, including many UI/config flags). The most
useful fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Community base-36 ID |
| `display_name` | string | Community name |
| `display_name_prefixed` | string | Name with `r/` prefix |
| `title` | string | Human-readable title |
| `description` | string | Full description |
| `public_description` | string | Short listing description |
| `submit_text` | string | Submission guidance |
| `lang` | string | Primary language |
| `over18` | boolean | Always false in this corpus |
| `quarantine` | boolean | Quarantine flag |
| `subreddit_type` | string | Community visibility type |
| `submission_type` | string | Allowed submission kinds |
| `subscribers` | integer/null | Subscriber count |
| `accounts_active` | integer/null | Active account count |
| `active_user_count` | integer/null | Alternate active-user count |
| `advertiser_category` | string | Topical category |
| `created_utc` | integer | Creation timestamp |
| `wiki_enabled` | boolean/null | Wiki availability |
| `url` | string | Community path |
| `_meta` | object | Corpus activity statistics |

For `subreddits_2025-01.zst`, `_meta` contains `num_comments`, `num_posts`,
`earliest_comment_at`, `earliest_post_at`, `num_comments_updated_at`, and
`num_posts_updated_at`. Use `_meta.num_comments` to estimate corpus
availability before an expensive collection scan.

Additional metadata files:

- `subreddits_meta_only_2025-01.zst`: slim index with `id`, `display_name`, and
  a `_meta` object (`num_comments`, `num_posts`, `earliest_comment`,
  `earliest_post`, and their `*_updated_at` timestamps).
- `subreddit_rules_2025-01.zst`: `subreddit`, `rules` (array), `retrieved_on`.
- `subreddit_wikis_2025-01.zst`: `path`, `content`, `retrieved_on`,
  `revision_author`, `revision_author_id`, `revision_date`, `revision_reason`.

## Reading compressed shards

```python
import io
import json

import zstandard


def iter_records(path):
    # subreddits_2025-01.zst is multi-gigabyte and needs a large window.
    decompressor = zstandard.ZstdDecompressor(max_window_size=2**31)
    with path.open("rb") as file:
        reader = decompressor.stream_reader(file)
        stream = io.TextIOWrapper(reader, encoding="utf-8")
        for line in stream:
            if line.strip():
                yield json.loads(line)
```

Use the collection tools for corpus-scale operations instead of manually
scanning every shard.
