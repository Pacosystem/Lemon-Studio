# Lemon Studio on Linux with yabridge

Notes and files for running this fork on Linux (PipeWire, JACK, Windows plugins through yabridge).

## What this fork changes

| Change | Why |
|---|---|
| `RLIMIT_RTTIME` is capped only on the rtkit path (`rt.rs`) | Capping it on the direct `SCHED_FIFO` path also lowered it for yabridge's Wine hosts, which warned *Low RTTIME limit detected*. |
| Windows editors are asked for their size every 250 ms, not every frame (`plugins.rs`) | Through yabridge each `getSize` is an IPC round trip into Wine made while the UI holds the plugin; every frame it kept the plugin past the audio's 2 ms wait and the effect was heard dry (crackles). |
| **Settings ▸ Audio ▸ Direct rendering** (`transport.rs`) | Renders each block in the device callback with no margin ahead, for REAPER-like latency. Off by default; `LEMON_DIRECT=1`/`0` overrides it. |

## Install

```sh
cargo install --path crates/os-app          # → ~/.cargo/bin/lemonstudio
install -m755 contrib/linux/lemonstudio contrib/linux/wine-yabridge ~/.local/bin/
install -m644 contrib/linux/lemonstudio.desktop ~/.local/share/applications/
```

Edit the cases in `wine-yabridge` to match your Wine prefixes and builds.

## Low-latency setup

- **Settings ▸ Audio:** choose the JACK host (`host=jack` in `~/.config/Lemon Studio/audio.txt`) so Lemon runs inside the PipeWire graph at its quantum, instead of the PulseAudio layer with its own buffer.
- **PipeWire quantum:** 128 at 48 kHz is a good start; e.g. a WirePlumber drop-in setting `default.clock.quantum = 128`.
- **Real-time limits:** be in the `realtime` group (rtprio 98, memlock and rttime unlimited).
- **Project PPQ:** a higher PPQ (e.g. 384) means shorter engine ticks, so live notes wait less.
- **Direct rendering:** on for the lowest latency; turn it off if a heavy project crackles.
