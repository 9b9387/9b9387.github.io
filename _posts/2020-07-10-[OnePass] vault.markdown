---
layout: post
title: "[OnePass] vault"
data: 2020-07-10 21:00:00 +0800
category: "OnePass"
---

`vault` is a simple password manager. Given a passphrase and the name of a service, it returns a strong password for that service. You only need to remember your passphrase, which you do not give to anyone, and vault will give a different password for every service you use. The passphrase can be any text you like.

- GitHub: [https://github.com/jcoglan/vault](https://github.com/jcoglan/vault)
- Website: [https://getvau.lt/](https://getvau.lt/)

### Install
```
npm install -g vault
```
### Usage
- `--phrase` or `-p` You will be prompted for your passphrase.
- `--length` or `-l` Set the desired length.
- `--lower [n]` (n==0) no lowercase letter, (n>0) at least n lowercase letter `a`-`z`.
- `--upper [n]` Control uppercase letter `A`-`Z`.
- `--number [n]` Control number`0`-`9`.
- `--space [n]` The space character.
- `--dash [n]` Dashes (`-`) and underscores (`_`).
- `--symbol [n]` All other printable ASCII characters.
- `-r [n]` The maximum number of times the same character can appear in a row.
- `--key` or `-k` Use a value signed using your SSH private key instead of a simple passphrase.
- `--config` or `-c` To save your setting at `~/.vault` file, encrypted with `AES-256`, using your username as the key by default.
```
vault -c --upper 0  // global config
vault -c google --symbol 0 // service config
```
- `--export [file]` Export the settings to `file`.
- `--import [file]` Import the settings from `file`.
- `--delete [service]` or `-x [service]` Removes settings for an individual service.
- `--delete-globals` Removes your global settings.
- `--clear` or `-X` Deletes all saved settings.
- `--notes` or `-n` No longer supported.

### Setting Config Format
```
{
  "global": {
    "phrase": "xxx",
    "upper": 0
  },
  "services": {
    "google": {
      "length": 6,
      "phrase": "xox",
      "upper": 0
    },
    "twitter": {}
  }
}
```