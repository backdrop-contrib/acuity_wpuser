# Changelog — acuity_wpuser

Dev convention: the `.info` carries no `version` (the release packager adds it);
this changelog is the running version tally. On release, the top heading's
`(unreleased)` becomes the release date.

## 1.x-1.0.0-beta3 (2026-11-15)

Git tag: `1.x-1.0.0-beta3`.

### Added
- Login with WordPress 6.8+ passwords. WordPress 6.8 stores passwords as bcrypt (`$wp$2y$...`), which Backdrop core cannot verify, so imported users with such a password were refused with the right one. A login validator now checks them as WordPress does and re-saves the password as a Backdrop hash; core then logs the user in. Plain `$2y$` bcrypt hashes are handled too. Flood control applies as usual.
- Overview: the number of accounts still holding a WordPress 6.8+ password. Keep the module enabled until it reaches 0.

## 1.x-1.0.0-beta2 (2026-07-22)

### Acuity Migration Centre: relocated + Overview tab (2026-07-21)
- Now a card in the Migration Centre's **WordPress** section. Registration hook
  renamed `hook_acuity_wpmc_group_info()` → `hook_acuity_migration_group_info()`
  with a `category` key. Depends on the new **acuity_migration** hub.
- **Relocated** from `admin/config/acuity-wpmc/users` to
  `admin/config/acuity-migration/wordpress/users`.
- **Canonical tabs Overview → Settings.** The old Import tab became the default
  **Overview**, which now leads with a shared prerequisite checklist (WordPress
  connection set? users imported so far?) via the hub's
  `acuity_migration_checklist()`, plus the Run button (disabled until the
  connection is up). Settings (batch size) unchanged.

### Migration Centre export helper (2026-07-20)
- Declares its `export_tables` (`wp_users`, `wp_usermeta`) via
  `hook_acuity_wpmc_group_info()`, so the dashboard's new "Database export
  helper" includes them in the list of WordPress tables to export.

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
