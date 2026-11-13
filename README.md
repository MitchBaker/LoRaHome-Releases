# LoRaHome firmware releases

Firmware for **LoRaHome**: a LoRa base station that connects battery-powered
sensor nodes to Home Assistant over MQTT. This repository only holds the
released firmware; the files are attached to each
[release](https://github.com/MitchBaker/LoRaHome-Releases/releases).

| Firmware | Boards (build targets) |
|---|---|
| LoRaHome Base | Heltec WiFi LoRa 32 V4 (`heltec-v4`) |
| LoRaHome Node | Heltec WiFi LoRa 32 V4 (`heltec-v4`), V3 (`heltec-v3`) |

The latest release is always at
`https://github.com/MitchBaker/LoRaHome-Releases/releases/latest`, and its
manifest at
`https://github.com/MitchBaker/LoRaHome-Releases/releases/latest/download/manifest.json`.

## Files in a release

For each firmware and board, `<name>-<version>-<target>`:

| File | Use |
|---|---|
| `….bin` | Full image for a blank board over USB, flashed at `0x0`. Erases saved settings. |
| `…-app.bin` | The firmware alone: flash at `0x10000` over USB, or upload in the base console. Keeps settings. |
| `…-ota.zip` | Node only: the app plus a delta, for the base console's firmware upload. |
| `…-delta-from-<old>.bin` | Node only: a small update for nodes running exactly `<old>`. The base sends it automatically when it fits and the full image otherwise. |
| `*.z` | zlib-compressed copies of the app and delta, the form the base stores. |
| `manifest.json` | Versions, boards, sizes, SHA-256 hashes and download URLs of the latest Base and Node. |
| `manifest.json.sig` | Signature over `manifest.json`. |
| `SHA256SUMS` | SHA-256 hashes of every file in the release. |

## First install

1. Flash the full image over USB, for example:
   ```
   esptool --chip esp32s3 --port /dev/ttyUSB0 write-flash 0x0 LoRaHome-Base-<version>-heltec-v4.bin
   ```
   On the V4 use its USB port (`/dev/ttyACM0` on Linux, `COMx` on Windows).
   If the board doesn't connect, hold PRG, tap RST, then release PRG.
2. **Base:** no WiFi or passwords are built in. On first start the base opens
   a WiFi hotspot called `LoRaHome-<ID>`, and its screen shows a QR code and a
   password for it. Join it with a phone and the setup page opens. Pick your
   WiFi network and set the console login; Home Assistant (MQTT) and the time
   zone are optional there. The base then shows its address: open
   `https://<address>/console/` and accept the self-signed certificate once.
3. **Nodes:** start pairing on the base (PRG menu → Start pairing, or
   `SET LHBASE ADOPT_NODES` in the console), power the node, and approve the
   pairing code it shows.

Released builds use a public default adoption key. It only protects the
adoption exchange, which also needs the node's pairing code approved on the
base, so it can be public; after adoption each node uses its own key. A base
and its nodes must use the same adoption key.

## Updating

Upload the new `-app.bin` (Base) or `-ota.zip` (Node) in the base console's
firmware box. For nodes, then use `SET LHBASE OTA_NODE <node>`,
`SET LHBASE OTA_BULK`, a schedule (`OTA_SCHEDULE`), or the node's
**Firmware → Install** button in Home Assistant. In the console, `help ota`
explains each.

## Checking a download

Each file's hash is in `SHA256SUMS`:
```
sha256sum -c SHA256SUMS --ignore-missing
```
`manifest.json` is signed with the release key, an ECDSA P-256 key whose
public half is [`release-signing.pub.pem`](release-signing.pub.pem) in this
repository:
```
openssl dgst -sha256 -verify release-signing.pub.pem -signature manifest.json.sig manifest.json
```
Expect `Verified OK`. The manifest lists every file's SHA-256, so a verified
manifest vouches for the whole release.
