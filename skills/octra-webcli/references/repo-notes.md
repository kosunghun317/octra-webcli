# Octra WebCLI Repo Notes

Last updated: 2026-05-18.

## Current Fork State

Repo path:

```text
/Users/kosunghun/Developer/Projects/octra-webcli
```

Remotes:

```text
origin   https://github.com/kosunghun317/octra-webcli.git
upstream https://github.com/octra-labs/webcli.git
```

Current important refs after the latest sync:

```text
main                         85514dd docs: document wallet network settings
origin/fix/amm-main-contract 85514dd docs: document wallet network settings
upstream/main                f9c73e1 mimi updates
origin/main                  0c1303c added history for bridge, recovery, and search for lost txs
```

At that point, local `main` was identical to `origin/fix/amm-main-contract`, ahead of `upstream/main` by 8 fork commits, and ahead of `origin/main` by 9 commits.

Fork commits on top of `upstream/main`:

```text
85514dd docs: document wallet network settings
05d27a7 Tame PVAC registration retries
c2bca8f Update AMM template and OCT formatting
83d2d68 Expose private key address derivation
6b4f2da Lower contract deploy fee fallback
9b8b068 Rename wallet preset to devnet
1d8fafa Add wallet network presets
cbcf6fa new AMM code w/ modularized invariant calculation
```

## Custom Features To Preserve

Network settings UI:

- `static/index.html`
- Settings view has `select#settings-network` with `devnet`, `mainnet`, and `custom`.
- RPC/explorer/relayer fields are `settings-rpc`, `settings-explorer`, and `settings-bridge-signer`.

Network preset logic:

- `static/wallet.js`
- `NETWORK_PRESETS` contains:
  - `devnet`: RPC `http://165.227.225.79:8080`, explorer `https://devnet.octrascan.io`
  - `mainnet`: RPC `https://octra.network`, explorer `https://octrascan.io`
- Keep `detectNetworkPreset`, `applyNetworkPresetFields`, `onNetworkPresetChange`, and `setNetworkFieldLock`.
- `doSaveSettings` posts to `/settings`, updates RPC/explorer state, locks preset fields unless custom, and clears caches when RPC changes.

Backend settings:

- `wallet.hpp` stores wallet RPC/explorer/bridge signer settings.
- `main.cpp` exposes settings and wallet endpoints and resets RPC-related runtime state.
- Useful endpoints documented in `README.md` include:
  - `GET /api/wallet`
  - `POST /api/keys/private`: requires a build with `-DOCTRA_WEBCLI_ENABLE_KEY_EXPORT`; body must include `pin` and `confirm: "I_UNDERSTAND_KEY_EXPORT_RISK"`.
  - `POST /api/wallet/derive-address`
  - `POST /api/settings`
  - contract compile/deploy/call/view endpoints.

AMM utilities:

- `static/templates/amm/main.aml`
- `static/templates/amm/interfaces/IOCS01.aml`
- Related commits update the AMM template and OCT formatting.

## Upstream Merge Rules

Before syncing:

```sh
git status --short --branch
git fetch origin --prune
git fetch upstream --prune
git rev-list --left-right --count main...upstream/main
git rev-list --left-right --count main...origin/fix/amm-main-contract
```

Prefer fast-forward-only when possible:

```sh
git merge --ff-only upstream/main
git merge --ff-only origin/fix/amm-main-contract
```

If upstream has new commits and conflicts with fork utilities, resolve toward preserving:

- The network preset selector and behavior.
- The devnet/mainnet/custom preset values.
- Settings persistence through `/api/settings`.
- Cache invalidation on RPC change.
- The AMM template and related interface files.
- Private-key address derivation endpoint.

Use these checks after conflict resolution:

```sh
rg -n "settings-network|NETWORK_PRESETS|onNetworkPresetChange|doSaveSettings|derive-address|bridge_signer" static/index.html static/wallet.js README.md wallet.hpp main.cpp
git status --short --branch
```

## Build Notes

Build command:

```sh
make
```

Build with private key export enabled:

```sh
make clean
make CXX='g++ -DOCTRA_WEBCLI_ENABLE_KEY_EXPORT'
```

Do not use `make CXXFLAGS+=' -DOCTRA_WEBCLI_ENABLE_KEY_EXPORT'` in this Makefile because it drops the existing include/optimization flags.

Non-installing verification command:

```sh
make OCTRA_SKIP_AUTOSETUP=1
```

Dependencies:

- C++17 compiler
- OpenSSL 3
- LevelDB
- PVAC library built from `pvac/`

Known local build result on 2026-05-18:

```text
fatal error: 'leveldb/db.h' file not found
```

`brew --prefix leveldb` returned `/opt/homebrew/opt/leveldb`, but that path did not exist, so LevelDB likely needs installation before a full compile check can pass.

Ignored generated files include:

```text
octra_wallet
octra_wallet.exe
*.o
data/
pvac/build/
```
