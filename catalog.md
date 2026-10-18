# sql_v1 acquisition contract

Six registered relations: comments, submissions, author_events, subreddits,
flairs, links. `query_tables("SELECT * FROM comments LIMIT 1")` returns typed
schema and a bounded preview. Fact relations hide physical bucket columns.
Comments have original id, body, link_id (t3_ submission), parent_id (t1_ comment
or t3_ submission), author, subreddit, created_utc. Submissions retain id, title,
selftext independently. Dimensions and author_events support joins/filtering.
No historical macros are loaded. Only one SELECT is allowed; table functions,
external readers and unreviewed functions are rejected. CTEs and joins work.

`materialize_query(sql, observation_type, include_parent=false)` selects exactly
one string column named id. materialize_query accepts selected IDs that must
resolve to canonical source entities. Canonical records are hydrated from the
bound source tables; query expressions cannot supply replacement source text or
relationships. Types:
- comment: selected comments alone; optional actual parent is context.
- submission: selected submissions, including zero-comment submissions.
- pair: selected child comment and its real observed parent, both primary.
- thread: selected submission IDs plus all available comments in that snapshot.
Author filtering is ordinary SQL, e.g. `SELECT id FROM comments WHERE author = 'example_author'`.
Selecting a comment never selects siblings. Missing optional parent is recorded;
a pair with a missing parent fails. Full-thread scope is the retained snapshot.

Handles are opaque IDs. Files are under output/artifacts/HANDLE. Each immutable
SQLite data.sqlite contains entities(key, source_table, source_id, record JSON),
observations(key, kind, metadata JSON, labels JSON), and
memberships(observation, entity, role). Read in batches with local Python sqlite3.
Never alter these files; every transformation creates a new artifact.

For derive_artifact, write a workspace-relative JSONL proposal with one row per
selected observation: {"observation_key":"comment:example_id","labels":{"category":"chosen"}}.
Omitted observations are removed; empty file selects zero. Omit labels to retain
current labels. Provenance JSON requires description; script_path optionally
archives exact code bytes and hash. New, removed or changed labels require
label_provenance with method and definitions
mapping new/changed current label names to their meaning; record model/prompt identity there when used.
Omitted or identical labels inherit parent labeling provenance; filter-only
transformations need their own description, not fabricated label provenance.
Exact proposal/provenance/script bytes are archived under child evidence/ and
remain auditable after scratch is deleted. Dependencies such as external models,
packages and services must be recorded separately; scripts do not archive them.
Submit proposal_path, provenance_path and parent handle. No source fields may be
supplied. Further source context needs a new materialize_query acquisition.

compose_artifacts(handles=[...], provenance_path=...) explicitly combines at least
two complete parents in this workspace. Supply a composition description in
provenance JSON. Only requested parent observations and their referenced entities
are copied. Metadata, labels, roles and canonical records are retained. Identical
duplicates deduplicate; conflicts fail. All parent IDs and hashes are recorded.
There are no source reads or new context lookups. This creates a new immutable child.

finalize_artifact(handle, methodology) publishes exactly the selected artifact.
No implicit union, source reads, graph expansion or evaluator conversion occurs.
query_tables returns at most 200 rows and 256 KiB including metadata. Preview
truncation is marked. Materialization streams batches of about 10,000 rows, with
5M selector/entity/observation rows, 10M memberships, 5 GiB SQLite data and
30-minute safety ceilings. IDs are capped at 4096 UTF-8 bytes each and 256 MiB
cumulatively including repeats. Archived evidence and manifest provenance each
have a 64 MiB ceiling. Derivation/composition enforce artifact limits too.
A ceiling hit fails acquisition; an explicit SQL LIMIT completes its selection.
These are operational ceilings, never experiment budgets. The attempt journal
records repeat reads and failures separately from historical thread accounting.

Staging binds the configured resolved 2025_2026 root, six table names, relative
Parquet filenames, sizes and schemas; each SQL worker checks exact inventory
agreement before model SQL. This binds workspace operations to a cheap file/schema
inventory. It is NOT a cryptographic full-corpus content proof.
The journal tolerates only a torn final JSON line, reports it as unresolved, and
still verifies earlier complete operations; middle corruption fails.
DuckDB engine limits are not a hard process RSS guarantee or OS resource sandbox. Python/bash share the OS filesystem:
profile guards and read-only files are cooperative controls, not a sandbox.

SQL/artifact tools are serialized per workspace. Prefer issuing tool calls
one at a time and wait for each result before launching the next.

Bounded previews teach corpus structure and help design selectors. A handful of
examples does not satisfy a configured final-observation target. Once useful
criteria are identified, prefer set-based materialize_query SQL to scale those
criteria over a tiny hand-picked VALUES list of IDs. Continue acquisition and
curation until the target can be finalized, within max_acquired_observations.
Do not duplicate IDs merely to satisfy scale.

Labels are optional unless the research task requires annotations. Do not create
labels merely because derive_artifact supports them; selection/filter-only
proposals need no invented labels. If any label key/value is added, removed, or
changed, provenance must include label_provenance.method and
label_provenance.definitions mapping supplied/changed label names to definitions.

run_policy.json binds experiment observation counts at staging. Task text and
artifact tool results report target_final_observations, acquired_observations,
and remaining_acquisition_observations (null means unbounded). Zero disables each
experiment control. Successful materialize_query observations consume budget,
including repeated acquisitions of the same IDs. Derive, compose and finalize
consume no new acquisition budget. Failed attempts remain in the journal.
Finalization requires at least target_final_observations in the selected artifact.
These experiment controls are separate from operational safety ceilings.
DuckDB uses memory_limit=8GB, max_temp_directory_size=64GB, threads=2 and
preserve_insertion_order=false, with spill files under workspace scratch.
These engine settings are not hard OS/RSS quotas and cannot be changed by model SQL.

## Six-table schemas and join keys

The source schema is also returned by bounded `query_tables` previews. Physical
bucket columns are hidden. IDs are base36 strings, not integers.

### `comments`

```text
id, link_id, parent_id, subreddit, author, body, score, created_utc,
is_submitter, author_flair_text, controversiality, has_media
```

### `submissions`

```text
id, title, selftext, is_self, author, author_flair_text, subreddit, score,
upvote_ratio, num_comments, link_flair_text, created_utc, media_type
```

### `author_events`

```text
author            string   -- sort key 1
created_utc       int64    -- sort key 2
kind              string   -- 'comment' | 'post'
subreddit         string
link_id           string   -- for posts: 't3_' || own id
id                string
score             int32
is_submitter      boolean  -- true for posts
author_flair_text string   -- point-in-time; makes identity-claim queries single-table
```

### `subreddits`

```text
subreddit          string   -- display_name, key
title              string
description        string   -- public + sidebar description
subscribers        int64
retained           boolean
drop_reason        string   -- null | 'activity' | (nsfw subs are absent entirely)
-- window activity (from the filter pass; replaces stale 2020 index counts)
n_comments, n_posts, n_authors, n_threads          int64
-- flair statistics (both kinds)
pct_comments_author_flaired   double  -- comments with non-empty author flair
pct_authors_flaired           double  -- distinct commenting authors with ≥1 flair
pct_posts_link_flaired        double
n_distinct_author_flairs      int32
n_distinct_link_flairs        int32
flair_cardinality_author      double  -- distinct values / flaired holders
flair_cardinality_link        double
-- community self-documentation (instrument discovery surface)
rules  list<struct{short_name, description, violation_reason, created_utc}>
wiki   list<struct{path, content, revision_date}>   -- retained subs only, capped
```

### `flairs`

```text
subreddit   string
kind        string   -- 'author' | 'link'
flair_text  string
n_holders   int64    -- distinct authors (author) / posts (link)
n_comments  int64    -- comment volume under this flair (author kind; 0 for link)
retained    boolean
```

### `links`

```text
post_id          string
subreddit        string   -- where posted (destination audience)
created_utc      int64    -- edge timestamp (post time)
score            int32
upvote_ratio     double   -- reception/controversy at the destination
retained         boolean
target_kind      string   -- 'post' | 'external'
target_post_id   string   -- crossposts: origin post id
target_subreddit string   -- crossposts: origin sub
url_norm         string   -- external: normalized URL
domain           string   -- external: registrable domain
```

Comments and submissions use bare `id` values. Join a comment to its thread
with `comments.link_id = 't3_' || submissions.id`. A comment parent uses
`parent_id = 't1_' || parent_comment.id`; a submission parent uses
`parent_id = 't3_' || submissions.id`. Never equate prefixed IDs with bare IDs.

`author_events.kind = 'comment'` joins `id` to `comments.id`; `kind = 'post'`
joins `id` to `submissions.id`. Include the kind predicate: IDs are scoped to
entity type. For posts, `link_id = 't3_' || id`. Author events have no body text.

Join community metadata on `subreddit`. Join flair vocabulary on `subreddit`,
`flair_text = author_flair_text` and `kind = 'author'`, or on
`flair_text = link_flair_text` and `kind = 'link'` for submission link flairs.
Flairs are point-in-time claims, not stable user labels.

`links.post_id = submissions.id` uses a bare post ID. For crossposts,
`target_kind = 'post'` and `target_post_id = submissions.id` identifies the
origin. External targets use `target_kind = 'external'`, `url_norm` and `domain`.
Dimensions and links can describe non-retained communities/posts; a matching
fact row is not guaranteed. Likewise, deleted or unavailable parents need not
exist in the snapshot. Thread scope means all available retained snapshot rows.
