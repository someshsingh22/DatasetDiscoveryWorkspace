# Reddit corpus layout

The pre-filtered corpus is available inside the workspace at `data/corpus/`.
Files are UTF-8 JSONL, optionally compressed with Zstandard.

Deleted/removed comment bodies and records from NSFW subreddits were removed
during preprocessing. The activity snapshot covers March–April 2020; subreddit
metadata is from January 2025.

## Directories

```text
data/corpus/
├── comments/
├── submissions/
├── subreddits_meta/
└── indexes/
```

Comment and submission shards use names such as
`RS_2020-03_aa.jsonl.zst`. Shards are not aligned record-for-record.

## Comments

Each record in `comments/` is one Reddit comment.

| Field | Type | Description |
|---|---|---|
| `id` | string | Base-36 comment ID |
| `link_id` | string | Parent post fullname: `t3_<id>` |
| `parent_id` | string | Immediate post or comment parent |
| `body` | string | Comment text |
| `author` | string | Reddit username |
| `author_fullname` | string/null | Internal user ID |
| `author_created_utc` | integer/null | Account creation timestamp |
| `author_flair_text` | string/null | Community-assigned author flair |
| `author_patreon_flair` | boolean | Patreon flair flag |
| `author_premium` | boolean | Reddit Premium flag |
| `subreddit` | string | Community display name |
| `subreddit_id` | string | Community fullname |
| `subreddit_name_prefixed` | string | Display name with `r/` prefix |
| `subreddit_type` | string | Public/restricted/private |
| `score` | integer | Net upvotes |
| `controversiality` | integer | Reddit controversy flag |
| `created_utc` | integer | Unix timestamp |
| `is_submitter` | boolean | Whether commenter authored the post |
| `distinguished` | string/null | Moderator/admin distinction |
| `all_awardings` | array | Award objects |
| `awarders` | array | Awarding users |
| `gilded` | integer | Gild count |
| `gildings` | object | Awards by type |
| `total_awards_received` | integer | Total awards |
| `top_awarded_type` | string/null | Highest award type |
| `associated_award` | object/null | Associated award |

## Submissions

Each record in `submissions/` is one thread root.

| Field | Type | Description |
|---|---|---|
| `id` | string | Base-36 post ID |
| `title` | string | Post title |
| `selftext` | string/null | Text-post body |
| `url` | string | External URL or permalink |
| `is_self` | boolean | Whether this is a text post |
| `author` | string | Submitter username |
| `author_fullname` | string/null | Internal user ID |
| `author_created_utc` | integer/null | Account creation timestamp |
| `author_flair_text` | string/null | Submitter flair |
| `author_patreon_flair` | boolean | Patreon flair flag |
| `author_premium` | boolean | Reddit Premium flag |
| `link_flair_text` | string/null | Post flair |
| `subreddit` | string | Community display name |
| `subreddit_id` | string | Community fullname |
| `subreddit_name_prefixed` | string | Display name with `r/` prefix |
| `subreddit_type` | string | Public/restricted/private |
| `subreddit_subscribers` | integer | Subscriber count |
| `score` | integer | Net upvotes |
| `num_comments` | integer | Reply count |
| `over_18` | boolean | Always false in this corpus |
| `created_utc` | integer | Unix timestamp |
| `distinguished` | string/null | Moderator/admin distinction |
| `all_awardings` | array | Award objects |
| `awarders` | array | Awarding users |
| `gilded` | integer | Gild count |
| `gildings` | object | Awards by type |
| `total_awards_received` | integer | Total awards |

Comments link to submissions with:

```python
post_id = comment["link_id"][3:]
assert comment["link_id"] == "t3_" + submission["id"]
```

## Subreddit metadata

`subreddits_meta/subreddits_2025-01` contains one community per record.

| Field | Type | Description |
|---|---|---|
| `id` | string | Community base-36 ID |
| `display_name` | string | Community name |
| `title` | string | Human-readable title |
| `description` | string | Full description |
| `public_description` | string | Short listing description |
| `submit_text` | string | Submission guidance |
| `lang` | string | Primary language |
| `over18` | boolean | Always false in this corpus |
| `quarantine` | boolean | Quarantine flag |
| `subreddit_type` | string | Community visibility type |
| `subscribers` | integer/null | Subscriber count |
| `accounts_active` | integer/null | Active account count |
| `active_user_count` | integer/null | Alternate active-user count |
| `advertiser_category` | string | Topical category |
| `created_utc` | integer | Creation timestamp |
| `wiki_enabled` | boolean/null | Wiki availability |
| `_meta` | object | Corpus activity statistics |

Use `_meta.num_comments` to estimate corpus availability before an expensive
collection scan.

Additional metadata:

- `subreddits_meta_only_2025-01`: slim ID/name/activity index
- `subreddit_rules_2025-01`: community rules
- `subreddit_wikis_2025-01`: wiki pages

## Reading compressed shards

```python
import io
import json

import zstandard


def iter_records(path):
    with path.open("rb") as file:
        if path.name.endswith(".zst"):
            reader = zstandard.ZstdDecompressor().stream_reader(file)
            stream = io.TextIOWrapper(reader, encoding="utf-8")
        else:
            stream = io.TextIOWrapper(file, encoding="utf-8")
        for line in stream:
            if line.strip():
                yield json.loads(line)
```

Use the collection tools for corpus-scale operations instead of manually
scanning every shard.
