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
    "to": "01-strategy-storage/01 Notes/99 Processed/YYYY/YYYY-MM-DD/<hash>-..."
  }]
}
```

The broker limits a proposal to ten moved source units, verifies input hashes,
validates generated pages, journals the transaction, and rolls back synchronous
failures.

## Post-archive reference relocation

`wiki_process` always moves sources from Raw to `01 Notes/99 Processed` first.
If the user wants a finished source retained as a reference, ask whether it
should remain in Processed or move to a specific `04 References` subfolder.
Use a separate protected `move` proposal for that relocation; do not bypass the
Raw-to-Processed archive step. After the move is applied, update the affected
wiki source page's `source_path` through a follow-up reviewed `wiki_process`
proposal. Keep the source ID and hash unchanged.
