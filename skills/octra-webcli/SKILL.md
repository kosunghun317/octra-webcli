---
name: octra-webcli-maintainer
description: Use when working in the octra-webcli repo, especially syncing the fork against octra-labs/webcli, preserving fork utilities such as devnet/mainnet network presets, AMM template changes, wallet settings, bridge signer settings, and local build/run workflows.
---

# Octra WebCLI Maintainer

Use this skill when the current repo is `octra-webcli` or the task mentions the Octra web client fork.

## First Checks

Run these before changing code or syncing branches:

```sh
git status --short --branch
git remote -v
git branch --all --verbose --no-abbrev
```

Expected remotes:

- `origin`: `https://github.com/kosunghun317/octra-webcli.git`
- `upstream`: `https://github.com/octra-labs/webcli.git`

If `upstream` is missing, add it:

```sh
git remote add upstream https://github.com/octra-labs/webcli.git
git fetch upstream --prune
```

## Sync Policy

The local fork contains custom utility work. Preserve it when syncing upstream.

Important custom branch:

- `origin/fix/amm-main-contract`

When this branch is a clean descendant of `main`, fast-forward local `main` to it:

```sh
git fetch origin --prune
git fetch upstream --prune
git merge --ff-only origin/fix/amm-main-contract
```

When merging newer `upstream/main`, keep fork-added utilities if conflicts happen:

- Keep devnet/mainnet/custom network preset selector in settings.
- Keep preset RPC/explorer/bridge signer behavior.
- Keep AMM template updates and utility API additions.
- Keep wallet settings persistence and cache clearing on RPC changes.

After any merge, check:

```sh
rg -n "settings-network|NETWORK_PRESETS|devnet|mainnet|bridge_signer|derive-address" static/index.html static/wallet.js README.md wallet.hpp main.cpp
git status --short --branch
```

## Validation

For a non-installing compile check:

```sh
make OCTRA_SKIP_AUTOSETUP=1
```

Known local caveat as of 2026-05-18: this may fail on macOS if Homebrew `leveldb` is not installed. The failure is `fatal error: 'leveldb/db.h' file not found`.

For key export testing, rebuild with:

```sh
make clean
make CXX='g++ -DOCTRA_WEBCLI_ENABLE_KEY_EXPORT'
```

The UI reveal flow must send `confirm: "I_UNDERSTAND_KEY_EXPORT_RISK"` to `/api/keys/private`.

For detailed repo notes, read `references/repo-notes.md`.
