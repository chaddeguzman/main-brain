# Second Self Wiki Schema

## Invariants

- Raw is pending; Processed is immutable.
- Existing curated notes stay in place.
- Every generated page has valid frontmatter and `verification: derived`.
- Topic, entity, and analysis claims trace to source pages.
- Source pages link to the actual archived or in-place evidence.
- Use relative Markdown links. Preserve `log.md` history.
- Do not create links solely to eliminate an orphan.

## Source pages

Store under `03-wiki/sources/<source-id-prefix>-<slug>.md`.

Required frontmatter:

```yaml
type: wiki-source
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active
verification: derived
source_id: <full sha256 or bundle manifest hash>
source_path: <private-relative final path>
source_sha256: <full sha256 or bundle manifest hash>
source_kind: <format or bundle>
processed_at: <ISO timestamp>
duplicate_of: ""
supersedes: ""
tags: []
projects: []
related: []
```

Use sections: Summary, Key Evidence, Connections, Uncertainties, and Source.
Label visual transcription or interpretation as agent-generated.

## Topic, entity, and analysis pages

Required frontmatter:

```yaml
type: wiki-topic # or wiki-entity / wiki-analysis
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active
verification: derived
source_ids: []
source_count: 0
tags: []
projects: []
related: []
```

Use claim-level source links where practical. Include tensions or uncertainty
instead of flattening disagreement.

## Special files

- `index.md`: retain generated markers; catalog every page with a one-line description and source count.
- `log.md`: append `## [YYYY-MM-DD] ingest|query|lint|refresh|duplicate | Title`.
- `open-questions.md`: record contradictory claims, ambiguous identities, stale material, and missing evidence.

## Broker specification

Use:

```json
{
  "operation": "wiki_process",
  "changes": [{"path": "03-wiki/...", "content": "..."}],
  "moves": [{
    "from": "01-strategy-storage/01 Notes/00 Raw/...",
    "to": "01-strategy-storage/01 Notes/99 Processed/YYYYMMDD_HHMMSS+OriginalName.ext"
  }]
}
```

The broker limits a proposal to ten moved source units, verifies input hashes,
validates generated pages, journals the transaction, and rolls back synchronous
failures.

Keep `99 Processed` flat. Preserve the original source name and extension after
the processing timestamp. Add a minimal numeric suffix only when names collide.
For legacy nested archives, use one `wiki_process` transaction containing the
Processed-to-Processed move and the matching source-page path update. The broker
removes emptied legacy date folders after the transaction succeeds.
