# octra wallet (webcli)
![Version](https://img.shields.io/badge/version-0.04.10--alpha-blue)


a full-fledged web client based on a local server for working with the octra network (compatible with both **DEVNET** and **MAINNET ALPHA**).

you can send txs, encrypt and decrypt balances, conduct stealth txs, and much more

## requirements

- c++17 compiler (GCC/Clang)
- openSSL 3.x
- libpvac (from `pvac/` directory)


### macOS (homebrew)

```
brew install openssl@3
```

### linux (debian or ubuntu)

```
sudo apt install g++ libssl-dev make
```

then (valid for both)

```
chmod +x setup.sh
./setup.sh
./octra_wallet
```

### windows

double click `setup.bat` or run it from command prompt
it will install everything automatically and then:

```
octra_wallet.exe
```

open `http://127.0.0.1:8420` in your browser


### windows (MSYS2)

install [MSYS2](https://www.msys2.org/), open MinGW64 shell:

```
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-openssl make
```

## build

```
make
```

## run

```
./octra_wallet # default port 8420
./octra_wallet 9000 # custom port
```

on windows:

```
octra_wallet.exe
octra_wallet.exe 9000
```

open `http://127.0.0.1:8420` in your browser

## launch

0. after opening the web interface in your browser, import your private key or create a new one directly in the modal window
1. enter a 6 digit PIN code to encrypt (AES 256 GCM) your wallet 
2. encrypted wallet files are stored under `data/`
3. the PIN is required on every startup to unlock

The current client also supports multiple encrypted wallet accounts under
`data/` with a manifest at `data/accounts.json`. Legacy `wallet.json` files can
be imported/migrated through the startup flow.

## network settings

Open the web UI settings page to choose the active network before compiling,
deploying, viewing, or calling contracts.

Presets:

- `devnet`: RPC `http://165.227.225.79:8080`, explorer
  `https://devnet.octrascan.io`
- `mainnet`: RPC `https://octra.network`, explorer `https://octrascan.io`
- `custom`: manually entered RPC, explorer, and bridge signer URL

The settings page posts to `/api/settings`, persists the selected RPC/explorer
inside the encrypted wallet file, updates the active RPC client, and clears the
transaction cache when the RPC changes.

Useful API endpoints for tooling:

- `GET /api/wallet`: current address, public key, RPC, explorer, and bridge
  signer settings.
- `POST /api/wallet/derive-address`: derive the Octra address for a supplied
  private key without switching accounts.
- `POST /api/settings`: update RPC, explorer, and bridge signer values.
- `POST /api/contract/compile-project`: compile an AppliedML project payload.
- `POST /api/contract/deploy`: deploy compiled contract bytecode.
- `POST /api/contract/call`: submit a state-changing contract call.
- `GET /api/contract/view`: run a contract view call.


we adhere to a policy of completely eliminating third-party software where possible, we have zero tolerance for vendor dependencies, we only included well-known libs and point implementations in the build, the rest was completely written from scratch by hand to avoid the use of third-party code for security reasons

## vendor libraries

- [cpp-httplib](https://github.com/yhirose/cpp-httplib) (MIT)
- [nlohmann/json](https://github.com/nlohmann/json) (MIT)
- [TweetNaCl](https://tweetnacl.cr.yp.to/) (public domain)
