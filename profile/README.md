# Voicetastic

Push-to-talk voice messaging — and full radio configuration — over the
[Meshtastic](https://meshtastic.org/) LoRa mesh. No internet, no cellular,
no central server.

This GitLab group hosts every part of the Voicetastic stack: the Android app,
the Linux desktop client (CLI + GUI + shared core), the zero-install browser
client, the marketing / browser-flasher site, and forks of the upstream
Meshtastic firmware and device UI used to ship Voicetastic-aware builds.

> Voice messaging is still experimental — the wire protocol (v3) is stable and
> implemented end-to-end across Android, desktop, and the browser client, but
> field-testing on real radios is ongoing. Text messaging and radio
> configuration are the supported paths today.

---

## Projects

| Project | What it is | Language |
|---|---|---|
| [voicetastic/android](https://git.cha-sam.re/voicetastic/android) | Android app — PTT voice, text chat, live mesh roster, full radio config UI. | Kotlin / Compose |
| [voicetastic/voicetastic-desktop](https://git.cha-sam.re/voicetastic/voicetastic-desktop) | Linux desktop companion. Workspace of `voicetastic-core` (transport + protocol + voice + codecs), `voicetastic-cli`, `voicetastic-gui` (egui), and `voicetastic-android-bridge` (UniFFI). | Rust |
| [voicetastic/voicetastic-web](https://git.cha-sam.re/voicetastic/voicetastic-web) | Zero-install browser client. Radio plugs into the user's machine over **Web Serial**; `voicetastic-core` compiled to WASM runs the protocol in-page. | Rust → WASM |
| [voicetastic/site](https://git.cha-sam.re/voicetastic/site) | Marketing site + in-browser ESP firmware flasher. Deployed to GitLab Pages on push to `main`. | Astro / TS |
| [voicetastic/firmware](https://git.cha-sam.re/voicetastic/firmware) | Fork of [meshtastic/firmware](https://github.com/meshtastic/firmware) tracking Voicetastic-specific firmware changes. | C++ / PlatformIO |
| [voicetastic/device-ui](https://git.cha-sam.re/voicetastic/device-ui) | Fork of [meshtastic/device-ui](https://github.com/meshtastic/device-ui) for the on-device LVGL UI. | C++ / LVGL |
| [voicetastic-desktop wiki](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/home) | Normative spec and implementer guide for the Voicetastic Voice Protocol v3. | Markdown |

---

## How the pieces fit together

```mermaid
flowchart LR
    A["Android app<br/>(Kotlin / Compose)"]
    C["Desktop CLI<br/>(Rust)"]
    G["Desktop GUI<br/>(Rust / egui)"]
    W["Browser client<br/>(Rust → WASM)"]
    N["Meshtastic node<br/>(Voicetastic firmware + device-ui)"]
    M(("LoRa mesh"))

    A  -- "BLE / USB serial" --> N
    C  -- "BLE / USB serial" --> N
    G  -- "BLE / USB serial" --> N
    W  -- "Web Serial" --> N
    N <--> M
```

Inside that pipe:

| Layer | Detail |
|---|---|
| Transport | BLE service `6ba1b218-15a8-461f-9fa8-5dcae273eafd`, USB serial, or Web Serial — same protobuf framing on all of them. |
| Text | `TEXT_MESSAGE_APP` (port 1). |
| Voice | `PRIVATE_APP` (port 256), 16-byte v3 header. Codecs: AMR-NB (default on Android), Opus, PCM_S16LE, Codec2. The codec is advertised in the header; the browser client currently sends/receives Codec2 only. |
| Config writes | `ADMIN_APP` (port 6). |
| Voice reliability | Reed-Solomon FEC + receiver-driven NACK + sender-side retransmit, implemented once in `voicetastic-core` and re-used by every client — Android through the UniFFI bridge, the browser through the WASM build. |

The normative wire format lives in the
[voicetastic-desktop wiki](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/Voice-Protocol);
the reference implementation is
[`crates/voicetastic-core/src/voice/`](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/tree/main/crates/voicetastic-core/src/voice).
The browser client is the same core compiled to `wasm32` — see
[voicetastic-web's README](https://git.cha-sam.re/voicetastic/voicetastic-web)
for the WASM driver.

---

## Getting started

Pick a path:

* **Just want to try it on a radio?** Flash a Voicetastic firmware build from
  the [site's `/flash` page](https://git.cha-sam.re/voicetastic/site), then
  pick a client:
  * [voicetastic-web](https://git.cha-sam.re/voicetastic/voicetastic-web) —
    nothing to install; plug the radio into your machine and open the page in
    Chrome/Edge or Firefox 151+.
  * [Android app](https://git.cha-sam.re/voicetastic/android) — pairs over BLE.
  * [desktop client](https://git.cha-sam.re/voicetastic/voicetastic-desktop) —
    CLI + egui GUI for Linux, over BLE or USB serial.
* **Building from source?** Each repo has its own README with prerequisites
  and build instructions. The desktop repo pulls
  [meshtastic/protobufs](https://github.com/meshtastic/protobufs) as a
  submodule — clone with `--recurse-submodules`. The web repo depends on
  `voicetastic-core` by relative path, so check it out beside
  `voicetastic-desktop`.
* **Implementing the protocol elsewhere?** Start with the
  [Overview](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/Overview),
  then [Frame Format](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/Frame-Format),
  then the [Sender](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/Sender-Guide)
  and [Receiver](https://git.cha-sam.re/voicetastic/voicetastic-desktop/-/wikis/Receiver-Guide) guides.

---

## Contributing

Voicetastic is a hobby / research project — issues, ideas, and merge requests
are welcome on any of the repos above. If you're changing protocol-affecting
code, please update the wiki in the same MR so the spec keeps tracking the
implementation.

---

## License

All Voicetastic-authored code is distributed under the
[GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html)
(`SPDX-License-Identifier: GPL-3.0-or-later`).

The `firmware` and `device-ui` forks retain their upstream licenses
(GPL-3.0-only, see each repo's `LICENSE`).

---

## Credits

* The [Meshtastic project](https://meshtastic.org/) for the open mesh
  firmware, BLE protocol, and protobuf definitions that everything here
  builds on.
