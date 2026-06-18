# Japanese Migration Manifest Example

This is a synthetic Phase 0 example. It illustrates classification, not real private data.

```yaml
source_vault: source-japanese-vault
target_vault: target-japanese-vault
preserve_data:
  - category: review_cards
    source: Learning/Review
    target: Learning/Review
    compare: frontmatter_and_body
  - category: media
    source: Media
    target: Media
    compare: hash
  - category: source_artifacts
    source: Learning/Sources
    target: Learning/Sources
    compare: links_and_hashes
recreate_from_pack:
  - vault_context
  - path_config
  - templates
  - default_views
  - japanese_pack_skills
transform_with_map:
  - source: OldConfig/custom-view.base
    target: Config/views/custom-view.base
    reason: reviewed_target_config_location
    dry_run: pass
    accepted_by_user: true
remove_after_cutover:
  - embedded_public_repo
  - old_jp_entrypoints
  - temporary_migration_adapters
conflicts:
  - path: Learning/Review/example-conflict.md
    status: blocked
    reason: target_path_collision
verification_report:
  preserved_count: 3
  recreated_count: 5
  transformed_count: 1
  removed_count: 3
  unresolved_conflicts: 1
```
