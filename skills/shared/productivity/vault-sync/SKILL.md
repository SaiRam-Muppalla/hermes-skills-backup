---
name: vault-sync
description: Use when syncing a local vault across devices.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, macos, linux]
metadata:
  hermes:
    tags: [vault, sync, markdown, obsidian, syncthing, mobile]
---

# Vault Synchronization

## When to Use

Use this skill when a user wants an Obsidian or local Markdown vault available on multiple computers or mobile devices.

Use this skill for syncing Obsidian or another local Markdown vault between a computer and a phone or tablet.

## Always-on rules

- Resolve the actual vault path before configuring synchronization; do not assume the default Documents path because desktop vaults may live under OneDrive, iCloud, or another synchronized parent.
- Preserve the vault as a normal local folder and sync the whole vault, including `.obsidian`, unless the user explicitly wants settings excluded; excluding it can omit plugins, themes, and workspace configuration.
- Prefer the official sync service for iOS, and prefer Syncthing for Android when the user wants a free, local-first setup; check phone platform and cost/privacy preference first.
- Add the sync folder on the computer before pairing the phone, then verify the indexed file count and transferred state before claiming setup is complete.
- Do not expose a local sync administration interface to the network without authentication; the interface can control access to synced files.
- Give the user only the remaining phone-side steps after computer-side setup, including the exact device ID or pairing token when required; when QR scanning is requested, show a scannable QR artifact encoding the device ID rather than only repeating the text.
- Never claim synchronization is complete after configuring only one device; pairing, folder sharing, and a file-transfer check are separate acceptance gates.
- Treat a disconnected or stale local sync UI as a service-state problem first: verify the service process, local web endpoint, configured folder, and device list, then restart only the sync service if needed and re-check the UI before changing the vault configuration.
- Keep the local administration interface bound to localhost and configure GUI authentication before making any interface reachable from another device; the GUI can control synced files.

## Computer-side procedure

1. Identify the installed note application and resolve the vault path from its configuration or the environment. On Windows, inspect `%APPDATA%\\obsidian\\obsidian.json` when `OBSIDIAN_VAULT_PATH` is unset.
2. Check whether the selected sync tool is installed. For Windows, install Syncthing with `winget.exe install --id Syncthing.Syncthing --exact --source winget --accept-source-agreements --accept-package-agreements --silent` when appropriate.
3. Start the sync service and verify its local administration page responds before configuring folders. If the UI shows an unknown device, zero listeners, or no version after a restart, inspect the service process and local endpoint; do not overwrite the existing vault configuration.
4. Add the resolved vault as a clearly named folder, such as `Obsidian Vault`. Use the exact native Windows path; do not substitute a shell-style or guessed path.
5. Wait for the service to index the folder. Verify that the UI reports the expected local file count and size.
6. Display the computer's device ID, provide it to the user, and, if requested, generate a QR code containing exactly that ID. Wait for the phone device to be added. Accept the phone request only after confirming its identity.
7. Share the vault folder with the paired phone and verify the remote device reports connected and synchronized. If the phone reports disconnected, check both devices are running, on the same Wi‑Fi for the first transfer, not paused, and that the PC UI still lists the phone before retrying the pairing.
8. On the phone, install the platform-appropriate sync client, choose a local destination, receive the shared folder, and open that destination in Obsidian using **Open folder as vault**.
9. Test bidirectional sync with a harmless small note or test edit, then confirm the change appears on both devices.

## Pitfalls

- Check the desktop application's registered vault list before creating a new vault; a fallback path can exist without being the active vault.
- Treat the first sync as a local-network transfer when possible; it is faster and avoids troubleshooting relay or firewall behavior before basic pairing works.
- Do not edit the same note simultaneously on both devices during initial setup; file-based synchronization can create conflict files when changes overlap.
- Keep `.obsidian` synchronized only between trusted devices; it may contain plugin data and configuration that should not be shared with untrusted users.

## Deeper guidance

See `references/platform-selection.md` for the platform/method decision table and phone-side recipes.
