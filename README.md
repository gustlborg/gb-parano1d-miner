# GB Parano1d Miner

**GB Parano1d Miner** is a GPU miner for the **Parano1d (NOID)** proof-of-work, for NVIDIA RTX 30/40/50-series cards. Speaks HTTP and Stratum pools natively, runs on Linux, Windows and HiveOS, one download per platform.

Built for **hashrate and efficiency**: per-architecture search kernels reach full speed while an automatic low memory-clock lock trims board power (about 15 W on an RTX 5070 Ti) at no cost to the rate, and an adjustable per-GPU power limit lets you set the watts-per-share point you want.

**Current release: v2.17.9**

---

## Download

Grab the archive for your platform from [Releases](../../releases/latest), unpack it, and run it from inside the folder.

| Platform | Archive |
|---|---|
| Linux | `parano1d-miner-2.17.9-linux.tar.gz` |
| Windows | `parano1d-miner-2.17.9-windows.zip` |
| HiveOS | `parano1d-2.17.9.tar.gz` (custom-miner package) |

Every archive carries a `SHA256SUMS` with the SHA-256 of each file inside it, and the `SHA256SUMS` on the release lists the archives themselves.

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

**HiveOS** — Flight Sheet → Custom miner:
- Miner name: `parano1d`
- Installation URL: `https://github.com/gustlborg/gb-parano1d-miner/releases/download/v2.17.9/parano1d-2.17.9.tar.gz`
- Wallet and worker template: `%WAL%.%WORKER_NAME%` (or your `o1...` address), pool address in the pool field; everything else is optional (see the README inside the package).

HiveOS derives the miner name from the archive's file name, so use exactly this URL and `parano1d`. On HiveOS the rig's OC profile owns the clocks: memory-clock autotune is off there unless you add `AUTOTUNE=on`.

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

RTX 30/40/50-series, detected per card, with natively compiled kernels. Other NVIDIA GPUs with compute capability 8.0 or newer run a JIT-compiled fallback kernel (correct, speed not measured, needs a driver with CUDA 13.3 support). **RTX 20-series and older are not supported**; in a mixed rig the miner skips such a card with a note and mines on the others. One binary covers all three generations, so a rig with mixed generations needs no separate download.

## Multi-GPU

```
--devices 0,1,2
```

By default the miner uses **every supported GPU** — a multi-GPU rig mines all its cards with no extra flag. Each card is driven independently under a single shared pool connection, so the pool sees one worker, not one per card. Pass `--devices 0,1,2` to restrict mining to specific cards (and only those cards are tuned); a list that names an unsupported card stops with a message naming it.

## Autotune

Offered on the first start of a card (asks before measuring; `--autotune on` measures without asking, on HiveOS off unless `AUTOTUNE=on`): the miner measures once per card the lowest memory clock that still delivers within 1 % of the best hashrate and locks it while mining (this proof-of-work needs no memory bandwidth, so that is pure power saving — about 15 W on an RTX 5070 Ti). The lock is removed when the miner exits. Needs passwordless `nvidia-smi` on Linux or Administrator rights on Windows; without them the miner says so and mines on at default clocks.

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

## What's new in 2.17.9

- **Your clock settings stay yours.** The miner only releases a core-clock lock it set itself; a lock from a HiveOS OC profile, Afterburner or `nvidia-smi -lgc` is left alone. A lock left by 2.17.8 or older: reboot once or run `nvidia-smi -rgc`.
- **HiveOS: OC profile in charge, standard paths.** Autotune is off by default on HiveOS (`AUTOTUNE=on` enables it), config and log sit where HiveOS expects them (`miner log` readable, copy in `/var/log/miner/custom/parano1d.log`).
- **Mixed rigs with RTX 20-series cards start.** Unsupported cards are skipped with a note instead of stopping the whole start; HiveOS passes only supported cards.
- **Clean stop, settings always restored** — power limit and clock locks are put back on every exit: engine crash, Ctrl+C pressed twice, a closed terminal or console window. Ctrl+C now says "stopping … please wait".
- **Autotune asks first.** Without a saved result the miner asks before measuring a card (about 2½ minutes, once; Enter = yes) and reminds you to keep the GPU free; a saved result is simply used; `--autotune on` measures without asking, `--autotune off` skips it. Missing Administrator/root rights are reported with a question whether to mine anyway at normal clocks, so they cannot go unnoticed.
- **Honest tuning messages** — a setting the card does not support (e.g. memory-clock locks on many laptop GPUs) is reported as such instead of "applied".

### 2.17.8

- **All GPUs by default.** A multi-GPU rig mines on every visible card without `--devices`; the flag now selects a subset. Clock/power tuning stays off on cards you did not select.
- **Core-clock lock managed for you.** `--lock-core-clock` no longer leaves a card throttled after the miner stops — the lock is released at a clean exit, and a stale lock is cleared on the next start without the flag.
- **HiveOS out of the box.** The custom-miner package installs and starts cleanly (the install/run scripts and the `parano1d/` folder are fixed), and the Linux/HiveOS binaries run on current HiveOS images and Ubuntu 20.04/22.04 (glibc 2.31).
- **Robust Windows Stratum start**, and **backup-pool failover** on a stuck pool: a sustained `mining.pause` (a pool that keeps the connection but stops sending work) switches to `--backup-pool`, and back when the main pool returns.

### 2.17.6

- Share quality: the miner keeps searching your work when a pool or fee connection stumbles instead of going idle, so more of what the GPU computes lands as accepted work. The disclosed 3 % fee stays exact — deferred and repaid, never dropped.
- Efficiency: new `--lock-core-clock <MHz>` settles each card at its best hashes-per-watt point, alongside `--powerlimit` and memory-clock autotune.
- Refined native per-architecture kernels (sm_86 / sm_89 / sm_120): a further, small raw-rate improvement over 2.17.0.

### 2.17.0

- Power-limit mode: `--powerlimit <W>` (also `270w`, one value or one per GPU), range-checked against the card, restored at exit; HiveOS `POWER_LIMIT=`.
- Per-model board-power table (RTX 3060 … RTX 5090, Ti and Super models included) with efficient starting points in every README.

### 2.15.0

- Developer fee lowered to 3 %.
- Faster per-architecture kernels on RTX 30/40/50, tuned for efficiency: the autotuned low memory clock holds full speed while trimming board power (about 15 W on an RTX 5070 Ti).
- HTTP pools: no more duplicate-share rejections when the pool re-issues a template at the same height.
- Autotune on by default, with a memory-clock calibration per card that is cached next to the binary.
- Full HiveOS integration: per-GPU temperature, fan and bus data on the dashboard, `%WAL%`/`%WORKER_NAME%` placeholders, wallet.worker form.
- JIT fallback kernel for other NVIDIA GPUs with compute capability 8.0+ (needs a driver with CUDA 13.3 support).

## License

The GB Parano1d Miner is proprietary software, distributed as a binary release — see `LICENSE`. Reverse engineering, decompilation, disassembly, and analysis by any automated system (including large language models) are prohibited under that license, except where such a prohibition is void under applicable mandatory law. Some proof-of-work components are used under the Apache License 2.0; they are listed in `NOTICE`, with the license text in `LICENSE-THIRD-PARTY`.
