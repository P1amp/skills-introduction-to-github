# Hologram fans: Raspberry Pi / Arduino integration without HDMI

Research date: 2026-10-19. Several vendor and manual domains (manuals.plus, holofanco.com, giwox.com, virtualongroup.com, community.flic.io, atomic14.com) could not be fetched from the research environment. Claims drawn from those sites come from search-result snippets only and are marked as such. GitHub pages were fetched and read directly.

## Are there GitHub projects or blog posts documenting the WiFi protocol or file format of cheap hologram fans? Which controller/app ecosystems do they cover?

### Takeaway
Yes. A small number of credible open-source projects exist, all from 2024–2026. They cover at least three separate controller families: (1) the "3D_42CM_xxxx" AP-mode fans at 192.168.4.1, with TCP port 20320 (the Windows "V13.0" / Android app protocol) and an older port 5233 variant; (2) the "EE37"-header .bin format used by Missyou fans (MS3/F1); (3) a "third-party control" TCP API on port 50200 at 10.10.10.1, reported for fans that use the "3D Magic" app. Playback control (next/prev/play-by-ID/brightness/power) is reverse-engineered. Uploading files over WiFi is **not** working in any open project found.

### Cited Findings
**Michdo93/hologram-fan-projector (Python, MIT). The most complete protocol work found.**
- Targets a POV fan whose AP SSID looks like `3D_42CM_51ABDC`. The protocol was reverse-engineered from the Windows companion app "电脑软件 V13.0 / Windows App V13.0" using binary analysis and Wireshark captures — [GitHub: Michdo93/hologram-fan-projector](https://github.com/Michdo93/hologram-fan-projector)
- The device is a TCP server at `192.168.4.1:20320`. Messages are ASCII frames: fixed header, then three check bytes that depend only on payload length, then payload, then fixed footer. Sending header plus footer alone returns the animation list, and the device also pushes the list periodically — [same](https://github.com/Michdo93/hologram-fan-projector)
- The connection has to stay open ("warm"). Commands sent on a connection that closes straight away are ignored, so the tool performs a handshake, sends keepalives, and often repeats commands. Some firmware may accept only one client at a time, so the official app must be closed — [same](https://github.com/Michdo93/hologram-fan-projector)
- Implemented commands: on/off, play/pause, next, previous, single-loop, list-loop, brightness up/down, rotation CW/CCW, per-video duration (5–30 s), clock settings, list, selftest, raw button and raw payload. Not implemented: BIN upload (no capture of a successful transfer exists) and delete. "Format Disk" and "Clear Cache" are unverified and deliberately not wired in — [same](https://github.com/Michdo93/hologram-fan-projector)
- .BIN format, as reverse-engineered: a sequence of 129,024-byte frames, each 512 angles × 252 bytes (6 groups × 42 radius values). Groups 0–3 carry luminance, group 4 R+G and group 5 G+B, a YUV-like scheme. The app keeps every second frame, so 24 fps becomes 12 fps. Some files have a 20-byte trailer. The Python encoder is not byte-identical to the vendor app's output: it lacks the app's saturation curve and dithering — [same](https://github.com/Michdo93/hologram-fan-projector)
- The device-control script uses only the Python standard library, so it would run on a Raspberry Pi as-is (inference from "device tool needs only the standard library") — [same](https://github.com/Michdo93/hologram-fan-projector)
- A companion repo, Michdo93/hologram-fan, contains only `hologram_fan.py` — [GitHub search result](https://github.com/Michdo93/hologram-fan)

**buzzibaer/FanPlugin.Wrapper (Windows DLL, for virtual-pinball "topper" fans)**
- Controls a generic AliExpress fan over its open WiFi AP: fan at `192.168.4.1`, host given a static 192.168.4.2/24 with no gateway, on a separate USB WiFi dongle — [GitHub: buzzibaer/FanPlugin.Wrapper](https://github.com/buzzibaer/FanPlugin.Wrapper)
- Supports protocol variants by "hardware version": v2 at `192.168.4.1:5233`, v3 also on 5233 (unverified), and "20320" (the Android-app protocol on port 20320). It selects a video by numeric ID with `playVideoWithId(n)`. Media upload is not implemented — [same](https://github.com/Buzzibaer/FanPlugin.Wrapper)
- SD-card files are named with six digits, e.g. `000001.bin`. Leading zeros are needed because the fan sorts filenames bytewise, and that order sets the play IDs. IDs run 0–99 — [same](https://github.com/buzzibaer/FanPlugin.Wrapper)

**zackmcmurrin-dev/holo-fan-linux (Python CLI). The open .bin converter for "EE37" fans.**
- An unofficial offline converter plus SD-card staging tool. It accepts anything FFmpeg can decode (MP4, MOV, GIF) and needs neither the phone app nor a network — [GitHub: zackmcmurrin-dev/holo-fan-linux](https://github.com/zackmcmurrin-dev/holo-fan-linux)
- Format: an 812-byte "EE 37" header, then frames of 137,280 bytes, each followed by a 1,600-byte silent audio slot. The display profile is 116 radial LEDs × 520 lines × 6 colour bits, packed as six bit-planes in R/B/G byte order, sampled from 20 fps input. Files are padded to a multiple of 28,800 bytes — [same](https://github.com/zackmcmurrin-dev/holo-fan-linux)
- Confirmed on one unit only, a Missyou MS3 (F1) 32 cm fan. The README warns that other models and firmware may differ. The `stage` command copies files to the root of a FAT/exFAT volume and verifies them with SHA-256 — [same](https://github.com/zackmcmurrin-dev/holo-fan-linux)

**Port 50200 "third-party control" (3D Magic app ecosystem). Community post, not verified.**
- A Flic community forum post, read via search snippets only, quotes a vendor note: after "third-party control" is enabled in settings and the fan restarted, a TCP client can connect on port 50200, either to the fan's hotspot at `10.10.10.1` or over the LAN. The post gives example byte strings for power on and stop. The poster was hoping to drive the fan from a Flic button and was looking for a developer, so there is no confirmation it works — [Flic community: Remote for hologram fan](https://community.flic.io/topic/18371/remote-for-hologram-fan)
- A HOLOFAN 31.5" product listing advertises a "3rd party control" API (search snippet only; page not fetchable) — [holofanco.com](https://holofanco.com/products/holofan-31inch)

**Apps and ecosystems named in the sources**
- "3D Magic" (App Store id1459519725; Google Play `com.Coeus3D.Coeus3DMagic`; also an APK `com.lailai.middle`, "3D Magic Holographic Fan Display"). The phone joins the fan's hotspot, then uploads, edits and deletes videos and sets play mode and brightness — [App Store](https://apps.apple.com/us/app/3d-magic/id1459519725); [Google Play](https://play.google.com/store/apps/details?id=com.Coeus3D.Coeus3DMagic&hl=en-US); [APKCombo](https://apkcombo.com/3d-magic-holographic-fan-display/com.lailai.middle/); vendor download page [holocloud.online](http://down.holocloud.online/index_en.html)
- "3D LED FAN" (App Store id1436836113). Works over the device hotspot, or after the device joins a router in a "network version" — [App Store](https://apps.apple.com/us/app/3d-led-fan/id1436836113)
- "Showcase Hologram" app. The phone connects to a WiFi network starting with "Showcase" or "3D" — [App Store](https://apps.apple.com/us/app/-/id6472708352)
- Holocircle app (imports MP4) — [holocircle.com](https://holocircle.com/en/app/)

**Low-quality or unreliable source (flagged)**
- ruslanmv/Streaming-Real-Time-3D-Images-to-LED-Holographic-Fans claims to stream frames to Missyou/GIWOX fans with HTTP POST to `http://<fan-ip>/upload_frame`. The endpoint is a generic placeholder ("Most holographic fans have an API endpoint"), several code samples reference undefined variables, and nothing is shown working on real hardware. Treat it as speculative or AI-generated, **not** a real protocol — [GitHub: ruslanmv](https://github.com/ruslanmv/Streaming-Real-Time-3D-Images-to-LED-Holographic-Fans)
- An OpenCV forum "gigs" post seeks someone to convert video to .bin and upload to a hologram fan from iOS, which suggests no ready-made public solution existed when it was posted — [OpenCV forum](https://forum.opencv.org/t/conversion-of-video-files-to-bin-files-upload-to-a-hologram-fan-for-ios/16235)

**Chipsets and controller boards**
- An Atomic14 teardown of a spinning-LED hologram fan (search snippet only) describes one PCB that carries the MCU and all the LEDs with their driver chips, a BLE SoC, a PT2432 brushless DC motor driver, wireless (inductive) power transfer to the rotor, and remote commands relayed by an MCU through a transistor and an LED on the opposite side of the board — [Atomic14 teardown](https://www.atomic14.com/videos/posts/bT716nyK0AY)
- The Michdo93 and holo-fan-linux geometries differ: 42 radius values × 512 angles in the 3D_42CM BIN, against 116 radial lights × 520 lines in the EE37 Missyou format. Different board families therefore use incompatible .bin layouts — [Michdo93](https://github.com/Michdo93/hologram-fan-projector); [holo-fan-linux](https://github.com/zackmcmurrin-dev/holo-fan-linux)
- lparakhin/3D_Hologram_Fan holds a sample `.BIN` and a `224_LEDs` folder, which suggests a 224-LED variant. The repo has no documentation — [GitHub: lparakhin](https://github.com/lparakhin/3D_Hologram_Fan)
- DIY open firmware exists if a fully controllable fan is wanted instead of a commercial one: grimimirg/3d-holographic-fan-display (ESP32, WiFi upload) and LLED-Fans/LED-Fan (ESP32 firmware on the blade) — [grimimirg](https://github.com/grimimirg/3d-holographic-fan-display); [LLED-Fans](https://github.com/LLED-Fans/LED-Fan)

### Inferences
- Two AP-address conventions show up: `192.168.4.1` (typical of ESP8266/ESP32 SoftAP defaults), used by the 3D_42CM / port 20320 / port 5233 family, and `10.10.10.1`, used by the port 50200 "3D Magic third-party control" family. The 192.168.4.1 default hints at an Espressif WiFi module in that family, but no teardown confirms it.
- A Pi with a second WiFi interface (USB dongle) on the fan's AP could use Michdo93's `hologram_fan.py` today for next/previous/play-by-ID/power/brightness on a 3D_42CM-type fan. That is the most direct "Pi controls the fan over WiFi" route documented anywhere.
- Changing which files are on the fan from a Pi over WiFi is not solved in open source. New content still means swapping or re-writing the TF card, or using the vendor app.

### Gaps
- I found no capture or documentation of the WiFi **upload** transfer for any ecosystem.
- Could not read the full Flic post or the HOLOFAN 3rd-party API docs (domains unreachable), so the exact port 50200 command bytes are unconfirmed.
- No teardown names the exact MCU or WiFi chips in the 192.168.4.1 or 10.10.10.1 families. Apps named in the brief such as "HoloFan" and "DSee" were not linked to any protocol documentation.
- No Hackaday or Reddit write-up specific to hologram-fan protocols turned up in searches.

## Which fans play standard MP4 directly from USB/TF without conversion? Does USB "PC mode" expose mass storage a Pi could write to?

### Takeaway
Most budget fans (Missyou, GIWOX-style, 3D_42CM, and many unbranded AliExpress units) need a proprietary .bin produced by vendor PC software or the phone app. The app converts MP4 on the phone before uploading, so "supports MP4" in listings often means the app accepts MP4. Some sellers (INNAYA, some Newegg/LightS listings) claim direct MP4/AVI/MKV playback, but no independent source verifies this. I found no source confirming that any fan enumerates as USB mass storage to a host. The documented workflow is to pull the TF card and write it with a card reader.

### Cited Findings
- Virtual On's FAQ (search snippet only) says its hologram device exclusively processes .bin files and other formats cannot be read directly — [Virtual On FAQ](https://virtualongroup.com/faq-items/type-video-file-plays/)
- Manuals describe converting video and images to .bin with Windows software, then copying the .bin to the TF card with a card reader. Some note that filenames must use only English letters and numbers — [SuperbHolo SH-30 manual](https://www.superbholo.com/wp-content/uploads/2020/01/SH-30-User-manual.pdf); [SuperbHolo SH-4270100 manual](https://www.superbholo.com/wp-content/uploads/2020/01/SH-4270100-User-manual.pdf); [Holofex Mac instructions](https://holofex.com/en-us/pages/instructions-for-mac-users)
- A Missyou manual describes PC software that converts MP4/JPG/GIF/MOV into .bin (snippet) — [manuals.plus Missyou](https://manuals.plus/asin/B08MWQQHCD)
- YouTube vendor video: "how to transfer MP4 to BIN file by the 3D hologram fan controller APP" — [YouTube](https://www.youtube.com/watch?v=9l7K7Idqf_4)
- Seller claims of direct playback: INNAYA says MP4 plays on every INNAYA fan with no conversion, and a Newegg listing says it "plays MP4, AVI, MOV, RMVB, JPG, GIF, JEPG, MKV format video directly" with no .bin conversion. Both are vendor claims only — [INNAYA guide](https://innaya.store/pages/hologram-fan-software); [Newegg listing](https://www.newegg.com/p/0FC-065A-001Y9)
- A product spec lists "mp4, avi, rmvb, mkv, gif, jpg, png" as supported. This may refer to app or PC-software input formats rather than on-card playback — [scccltd listing](https://scccltd.com/products/wifi-3d-fan-hologram-projector-25cm)
- A blog says some modern fans allow direct MP4 playback through a mobile app, but still recommends .bin — [mangalsenandsons blog](https://mangalsenandsons.com/blog/convert-video-to-3d-hologram/)
- USB cable control: one supplier (Holodseelab) sells a "USB hologram fan" that needs a dedicated USB control cable to the PC for "video upload, content switching". It is a sales claim, and the page doesn't say whether this is mass storage or a proprietary protocol — [holodseelab.com](https://holodseelab.com/usb-hologram-fan/)
- A generic HDFocus guide says USB-port fans auto-detect and loop content at power-on, and recommends a FAT32 drive with files in the root folder under simple names — [HDFocus guide](https://www.hd-focus.com/info/how-to-work-a-3d-hologram-fan-a-beginner-s-st-103458601.html)
- A reviewer says app (WiFi) transfer is slow and the memory card is more stable (search snippet of Newegg reviews) — [Newegg](https://www.newegg.com/p/0FC-065A-005Z2)
- The open holo-fan-linux converter lets a Pi or any Linux machine produce Missyou EE37 .bin files from MP4 with FFmpeg — [holo-fan-linux](https://github.com/zackmcmurrin-dev/holo-fan-linux)

### Inferences
- When a listing says "supports MP4", assume the app or PC software accepts MP4 and converts it, unless the seller explicitly says the TF/USB card plays raw MP4. Larger "advertising" fans with Android-class controllers (e.g. LightS with web/mini-program control) are more likely to decode MP4 natively, but this is unverified.
- For a Pi to put content on the fan without the vendor app, the realistic options are: write the TF card on the Pi (convert with holo-fan-linux for Missyou/EE37, or the Michdo93 encoder for 3D_42CM, each with fidelity caveats) and move it physically, or pre-load every clip once and then only *select* clips at runtime.
- A USB SD-card mux or "SD card extender/emulator" could in theory let a Pi rewrite the card in place. I found no hologram-fan-specific report of this, so it is speculative.

### Gaps
- I found no hands-on test confirming any specific consumer fan plays raw MP4 from a TF card or USB stick.
- No source confirms USB mass-storage enumeration ("PC mode"). It is unknown whether USB-port fans present as a USB device or only act as a USB host for a flash drive.

## What is the most reliable way a Raspberry Pi or Arduino can change what the fan shows (schedule or button press)?

### Takeaway
In descending order of reliability for a hobbyist:
1. **IR remote emulation** (record the fan's remote with a receiver, replay it with an IR LED via LIRC/ir-keytable on a Pi or IRremote on Arduino). This uses a standard, well-documented technique, works for any fan with an IR remote, and needs no network. The limit is that you can only step "next/prev", not jump to a clip.
2. **WiFi TCP control** with Michdo93's `hologram_fan.py`, or the FanPlugin port 5233/20320 protocol. This allows direct play-by-ID and works on 3D_42CM-type fans at 192.168.4.1. It needs a dedicated WiFi interface and a persistent connection, and may lock out the phone app.
3. **Relay or smart-plug power switching**. Fans auto-loop content at power-on, so this reliably turns the display on or off on a schedule. It cannot select content, and the power-on clip is undocumented.

Pre-loading all clips on the TF card and selecting them at runtime is more robust than trying to push content.

### Cited Findings
- **IR:** Arduino IRremote can capture codes and replay them with `sendRaw` raw timing arrays. `begin()` must be given the IR LED pin — [Arduino Forum](https://forum.arduino.cc/t/duplicating-fan-remote-with-ir-transmitter/979177); decoding remote hex codes on Arduino — [Hackster.io](https://www.hackster.io/sainisagar7294/arduino-based-ir-remote-decoder-ac67c4); [Arduino Project Hub](https://projecthub.arduino.cc/fanele24/remote-controlled-fan-147ad7)
- LIRC on a Pi or Linux: record with `irrecord`, send with `irsend SEND_ONCE` — [partofthething.com](https://partofthething.com/thoughts/adding-a-ir-send-and-receive-capabilities-to-my-home-assistant-based-smart-home/). Ken Shirriff's library supports NEC, Sony, RC5/RC6 and raw — [righto.com](http://www.righto.com/2009/08/multi-protocol-infrared-remote-library.html)
- In the Atomic14 teardown, the rotor MCU retransmits remote-control signals optically, which implies a standard IR remote path on that fan (snippet) — [Atomic14](https://www.atomic14.com/videos/posts/bT716nyK0AY)
- Retail listings and manuals commonly list a remote control alongside WiFi, app and PC control, e.g. the HoloSpin X5 manual lists WiFi, Bluetooth, app, PC and remote options (snippet) — [manuals.plus HoloSpin X5](https://manuals.plus/asin/B0DK5HLL4Q)
- **WiFi:** Michdo93's commands include next, previous, on/off, brightness, single and list loop, and per-video duration. Clips are selected from the device's animation list — [Michdo93](https://github.com/Michdo93/hologram-fan-projector). FanPlugin.Wrapper uses `playVideoWithId(n)` with fan-sorted `000001.bin`-style names — [FanPlugin.Wrapper](https://github.com/buzzibaer/FanPlugin.Wrapper)
- Some 3D LED FAN-app fans can join a router ("network version"), which would let a Pi on the same LAN reach them without a second WiFi interface — [App Store 3D LED FAN](https://apps.apple.com/us/app/3d-led-fan/id1436836113)
- **Power:** fans auto-detect and loop content at power-on — [HDFocus guide](https://www.hd-focus.com/info/how-to-work-a-3d-hologram-fan-a-beginner-s-st-103458601.html). Power-cycling (unplugging for about 30 s) is a standard reset — [HDFocus troubleshooting](https://www.hd-focus.com/info/why-is-my-hologram-fan-not-working-103081660.html)
- Hacking the commercial fan's internals (e.g. the BLE SoC) is not documented anywhere I found. Fully open alternatives are DIY ESP32 POV fans — [grimimirg](https://github.com/grimimirg/3d-holographic-fan-display); [LLED-Fans](https://github.com/LLED-Fans/LED-Fan)

### Inferences
- Recommended architecture for scheduled or button-driven content: load N clips onto the TF card, named `000001.bin` … so their order is known. Then either (a) on WiFi-protocol fans, run a small Python service on the Pi (cron or GPIO button) that calls play-by-ID or next over TCP 20320; or (b) on any fan with an IR remote, have the Pi or an Arduino send "power on" plus k × "next" IR codes after a known reset state. Add a relay or smart plug as a hard reset to a known state (clip 1 at boot), which makes IR stepping deterministic.
- Set the fan to single-loop mode so it stays on the selected clip rather than advancing on its own.
- Power-cycling with relays carries a mechanical risk: hard stops of a spinning rotor and repeated inrush. Use it sparingly, for daily on/off rather than per-clip switching. This is general engineering judgement, not sourced.

### Gaps
- No source documents whether fans resume the last clip or restart at clip 1 after a power loss. This matters for relay/IR state tracking and should be tested on the specific unit.
- No published IR code sets for hologram fan remotes were found. Each owner has to capture their own; NEC is likely for cheap remotes, but that is unverified.
- Whether the port 50200 third-party API supports play-by-ID is unknown.
