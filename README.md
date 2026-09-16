# Parano1d Miner

A GPU miner for the **Parano1d (NOID)** proof-of-work, for NVIDIA RTX 30/40/50-series cards. Speaks HTTP and Stratum pools natively, runs on Linux, Windows and HiveOS, one download per platform.

**Current release: v2.17.0**

---

## Download

Grab the archive for your platform from [Releases](../../releases/latest), unpack it, and run it from inside the folder.

| Platform | Archive |
|---|---|
| Linux | `parano1d-miner-2.17.0-linux.tar.gz` |
| Windows | `parano1d-miner-2.17.0-windows.zip` |
| HiveOS | `parano1d-2.17.0.tar.gz` (custom-miner package) |

Every archive carries a `MANIFEST.txt` with the SHA-256 of each file inside it, and `SHA256SUMS` on the release lists the archives themselves.

No CUDA Toolkit is needed to run — only a current NVIDIA driver. The Linux and HiveOS binaries run on any 64-bit Linux with glibc 2.31 or newer (Ubuntu 20.04 and later, current HiveOS images).

## Quick start

**Linux**
```
./gb-parano1d-miner cuda-mine --pool innovlab --coinbase <your-o1-address> --worker rig0 --search-binary ./gb-parano1d-engine
```

**Windows**
```
./gb-parano1d-miner.exe cuda-mine --pool innovlab --coinbase <your-o1-address> --worker rig0 --search-binary gb-parano1d-engine.exe
```

**HiveOS** — Flight Sheet → Custom miner, installation URL pointing at the HiveOS archive. Put your `o1...` address in the wallet field and the pool address in the pool field; everything else is optional (see the README inside the package).

## Pools

`--pool` takes a keyword, or use `--rpc <url>` for any other pool — HTTP and Stratum are detected automatically from the address. InnovLab is the default.

| Keyword | Kind |
|---|---|
| `innovlab` | Stratum (default) |
| `suprnova` | Stratum |
| `parano1d` | HTTP |
| `ariabrain` | HTTP |

A second pool can be given with `--backup-pool` / `--backup-rpc`; the miner switches over on its own if the main pool stops answering, and switches back when it returns.

## Hardware

RTX 30/40/50-series, detected per card, with natively compiled kernels. Other NVIDIA GPUs with compute capability 8.0 or newer run a JIT-compiled fallback kernel (correct, speed not measured, needs a driver with CUDA 13.3 support). **RTX 20-series and older are not supported.** One binary covers all three generations, so a rig with mixed generations needs no separate download.

## Multi-GPU

```
--devices 0,1,2
```

Each card is tuned and driven independently under a single shared pool connection — the pool sees one worker, not one per card. Without `--devices`, the miner uses the visible GPU.

## Autotune

On by default: the miner measures once per card the lowest memory clock that still delivers within 1 % of the best hashrate and locks it while mining (this proof-of-work needs no memory bandwidth, so that is pure power saving — about 15 W on an RTX 5070 Ti). The lock is removed when the miner exits. Needs passwordless `nvidia-smi` on Linux or Administrator rights on Windows; without them the miner says so and mines on at default clocks.

## Power limit

New in 2.17.0. `--powerlimit 270` sets the board power limit of every selected card to 270 W before mining and puts the previous limit back when the miner exits, Ctrl+C included. `270w` works too, and on multi-GPU rigs a comma list gives each `--devices` entry its own value:

```
--devices 0,1 --powerlimit 270,150
```

The miner reads each card's allowed range first and refuses to start with a value outside it. Measured on the release kernels, the last 5 % of board power buy only about 1 % of hashrate, so this is an efficiency setting — fewer watts per share, cooler and quieter cards — not a speed knob. The README in each package lists NVIDIA's reference board power for every supported model from the RTX 3060 to the RTX 5090 with a starting point for efficient mining. On HiveOS the same setting is `POWER_LIMIT=270` in the flight sheet's custom user config.

## Developer fee

This miner mines a disclosed **3 % developer fee** into a separate wallet, interleaved continuously rather than taken as one long block, so the pool never sees either session go idle. The fee is not shown in the terminal and the fee wallet is never displayed — the README is the disclosure.

## Status display

The header at the top of the window updates in place while mining. It reads well live but copies out of a terminal as merged lines — set `PARANO1D_PLAIN_OUTPUT=1` for a plain, append-only log that pastes cleanly into a bug report.

## What's new in 2.17.0

- Power-limit mode: `--powerlimit <W>` (also `270w`, one value or one per GPU), range-checked against the card, restored at exit; HiveOS `POWER_LIMIT=`.
- Linux and HiveOS binaries now built against glibc 2.31, so they start on Ubuntu 20.04/22.04 and current HiveOS images, not only on the newest distributions.
- Per-model board-power table (RTX 3060 … RTX 5090, Ti and Super models included) with efficient starting points in every README.

### 2.15.0

- Developer fee lowered to 3 %.
- Faster kernels on RTX 30/40/50: same hashrate as the fastest competing miner at 5–8 % less board power on an RTX 5070 Ti and RTX 3060 (measured on the pool's own accepted-work counter).
- HTTP pools: no more duplicate-share rejections when the pool re-issues a template at the same height.
- Autotune on by default, with a memory-clock calibration per card that is cached next to the binary.
- Full HiveOS integration: per-GPU temperature, fan and bus data on the dashboard, `%WAL%`/`%WORKER_NAME%` placeholders, wallet.worker form.
- JIT fallback kernel for other NVIDIA GPUs with compute capability 8.0+ (needs a driver with CUDA 13.3 support).

## License

The Parano1d Miner is proprietary software, distributed as a binary release — see `LICENSE`. Reverse engineering, decompilation, disassembly, and analysis by any automated system (including large language models) are prohibited under that license, except where such a prohibition is void under applicable mandatory law. Some proof-of-work components are used under the Apache License 2.0; they are listed in `NOTICE`, with the license text in `LICENSE-THIRD-PARTY`.
