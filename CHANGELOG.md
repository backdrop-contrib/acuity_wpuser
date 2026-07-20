# Changelog — acuity_wpuser

Dev convention: the `.info` carries no `version` (the release packager adds it);
this changelog is the running version tally. On release, the top heading's
`(unreleased)` becomes the release date.

## 1.x-1.0.0-alpha1 (unreleased)

### Menu (2026-07-20)
- Moved to the renamed Migration Centre root: the User Importer is now at
  `admin/config/acuity-wpmc/users` (was `admin/config/acuity/wpmc/users`), with
  its Import / Settings tabs and `configure` link updated. Requires the matching
  acuity_wpmc change.

### Docs & packaging (2026-07-20)
- Added README.md, covering import, data mapping, and login after migration:
  migrated users sign in with their existing WordPress password, which Backdrop
  core validates (it natively understands `$P$`/`$H$` hashes) and upgrades to a
  secure `$S$` hash on first login — no companion auth module is required.
- Added LICENSE.txt (GPL v2) and CLAUDE.md (developer notes).
- Fixed the `.info` `configure` link, which pointed at a dead path
  (`admin/config/acuity/wpuser` → `admin/config/acuity-wpmc/users`).

### Fixed (blocked bbPress members, 2026-07-12)
- Import now respects bbPress "blocked" status. A blocked member carries
  the `bbp_blocked` capability in `wp_capabilities`, but their
  `wp_users.user_status` stays `0` (active) — so status derived from
  user_status alone imported them as **enabled** accounts. The importer
  now unserializes `wp_capabilities` and forces `status = 0` (blocked in
  Backdrop) when `bbp_blocked` is set. Their forum content still imports
  with full attribution; they simply cannot log in.
  - Note: this reflects the source on every run — re-importing a member
    who is still `bbp_blocked` will re-block them in Backdrop.
  - Source check (2026-07-12): the current dataset has 0 blocked members;
    the guard is for the live import.
