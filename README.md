# Hermes skills backup

Private backup of the installed Hermes skill directories present when this snapshot was made.

## Layout

- `skills/shared/` — shared installed skills
- `skills/profiles/<profile>/` — profile-specific copies, kept separate so differences are not lost
- `manifest.json` — source mapping and inventory

This export includes each skill directory and its bundled references, templates, scripts, and assets. It intentionally excludes Hermes index/cache data, VCS/build/runtime caches, Python bytecode, and `.log` files. Imported skills may retain their original licenses and attribution; check each skill's own files before redistribution. Keep this repository private unless you have reviewed the contents and their licenses for public release.

Snapshot inventory: **1223 SKILL.md files** across **6 source trees**, **6544 files** (73.8 MiB).
