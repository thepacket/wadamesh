<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/wadamesh-readme-dark.svg">
    <img alt="WADAMESH" src="assets/wadamesh-readme-light.svg" width="440">
  </picture>
</p>

<p align="center"><b>A real touchscreen UI for your mesh radio.</b> &middot; open source &middot; GPL-3.0</p>

Touch-UI [MeshCore](https://github.com/meshcore-dev/MeshCore) companion-radio
firmware for the **LilyGo T-Deck / T-Deck Plus** and **Heltec V4 + TFT**
(ESP32-S3).

An LVGL touch UI — map, chat, contacts, channels, settings — split out of
[meshcomod](https://github.com/ALLFATHER-BV/meshcomod). The app depends on a
MeshCore fork via PlatformIO `lib_deps`.

## This fork (`thepacket/wadamesh`)

A fork of [`ALLFATHER-BV/wadamesh`](https://github.com/ALLFATHER-BV/wadamesh) that
adds an **operator-grade RF-diagnostics toolkit** on top of the base touch UI: apps
for seeing exactly what the radio hears, who is in direct range, and how the mesh is
carrying your traffic. Everything reads the tables the firmware already maintains (no
extra radio load beyond the active Discover scan), lives in the app drawer, and works
on every supported board.

**New apps**

- **Packets** — a live, per-frame list of every frame the radio pulls out of the air
  (type, route, hops, size, RSSI/SNR, colour-graded). Tap a frame to decode it: the
  path with hop hashes resolved to names, per-type payload (channel plaintext where you
  hold the key, advert name / type / position / clock-skew, trace hop SNRs), link
  margin against the SF demod floor, airtime, and raw hex.
- **Floods** — the same capture ring grouped by payload fingerprint, so flood
  rebroadcasts collapse into one row: copies heard, hop spread, best signal. Answers
  *"what is my mesh duplicating?"* — the right altitude for flood traffic.
- **Heard** — one row per node whose advert you've received this session, with signal,
  node type, hop count, age, and distance + bearing. Tap for the contact card.
- **Discover** — an active, zero-hop scan: pings every node type and lists who answers,
  with the SNR they heard *you* at (uplink) and the RSSI you heard *them* at (downlink),
  strongest first. Responders are auto-added to contacts and named from adverts when
  known. Client-side rate-limited so it stays within repeaters' discovery budgets.

**New per-node actions** (contact / repeater action sheet)

- **Neighbours** — ask a repeater what *it* hears at its own antenna
  (`REQ_TYPE_GET_NEIGHBOURS`), as a monospace table of name / uplink SNR / age. Coverage
  from the far end, which no other view shows.
- **Get name** — pull a node's advertised name on demand (`REQ_TYPE_GET_OWNER_INFO`), so
  a hex-placeholder contact gets its real name without waiting for its next advert.

**Fixes** (back-portable to upstream): MeshCore's `path_len` is an *encoded* byte —
hop count in the low 6 bits, hash size in the top 2 — not a length. Misreading it as a
length silently dropped every multi-hop advert whose path used multi-byte hashes, both
from the recent-heard table and the Discovered list. Fixed via the core's own
`Packet::isValidPathLen` / `copyPath`.

**Why this fork:** the base UI is an excellent everyday client; this turns it into a
field diagnostic tool for operators who want to see the RF layer directly — link
quality, coverage, flood redundancy, and who is actually reachable right now.

## Boards

See **[DEVICES.md](DEVICES.md)** for the full support matrix, install paths and
per-board status.

- LilyGo T-Deck / T-Deck Plus — env `LilyGo_TDeck_companion_radio_touch` (stable)
- Heltec V4 + TFT + CHSC6x touch — env `heltec_v4_tft_companion_radio_usb_tcp_touch` (stable)
- Tanmatsu (ESP32-P4) — built from `tanmatsu/` (ESP-IDF), ships via the Tanmatsu app store
- Elecrow ThinkNode M9 — env `ThinkNode_M9_companion_radio_touch` (beta)
- RAK WisMesh Tap V2 (RAK3312) — env `rak_tap_v2_companion_radio_touch` (beta)

## Architecture

This repo holds only the **app**: the `companion_radio` glue, the `ui-touch`
LVGL UI, the two boards' glue/variants, and `platformio.ini`. The **MeshCore
core is not vendored here** — it's pulled as a library via `lib_deps` from the
[`ALLFATHER-BV/meshcomod`](https://github.com/ALLFATHER-BV/meshcomod) monorepo
(the same repo as the non-touch firmware), pinned by a lean source-only `core-*`
git tag. The touch-app files this repo owns (TouchPrefsStore, WifiRuntimeStore,
the transports, …) are dropped from the lib via `-DMC_VENDORED_TOUCH_APP` so they
aren't compiled twice. The build is byte-identical to the original in-tree
meshcomod firmware.

## Build

[PlatformIO](https://platformio.org/) pulls the core fork and all libraries
automatically:

```bash
pio run -e heltec_v4_tft_companion_radio_usb_tcp_touch   # Heltec V4 TFT
pio run -e LilyGo_TDeck_companion_radio_touch            # LilyGo T-Deck
# or just `pio run` to build both
```

Flash with the NVS-preserving 4-component chain (bootloader / partitions /
boot_app0 / firmware at `0x0 / 0x8000 / 0xe000 / 0x10000`) so saved Wi-Fi
credentials survive — not a merged image, which 0xFF-pads and wipes NVS.

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). One topic per
PR; inbound contributions are accepted under the project's GPL-3.0 license.

## License

**GPL-3.0-or-later** — see [LICENSE](LICENSE). wadamesh is copyleft: anyone who
distributes a build or a fork must also make their source available under the GPL.
This keeps the UI open and concentrates community effort instead of fragmenting it
into closed forks.

wadamesh incorporates and depends on
[MeshCore](https://github.com/meshcore-dev/MeshCore) (MIT, © Scott Powell /
rippleradios.com) and other third-party components — see [NOTICE](NOTICE) for the
full list and their licenses. MeshCore-derived files keep their MIT notices; the
combined work is distributed under the GPL (MIT is GPL-compatible). The MeshCore
fork that wadamesh builds against stays **MIT** on purpose, so its Wi-Fi/BLE hooks
remain upstreamable to MeshCore.
