# Parano1d Miner

CUDA miner for Parano1d (NOID), Poseidon2b proof-of-work.

## Supported hardware

NVIDIA RTX 30/40/50 series, Linux.

## Installation

Download the latest release from the [Releases page](../../releases) and
unpack it.

## Usage

```bash
./probe cuda-mine --coinbase <your-o1-address> --worker <name> --search-binary ./live_search
```

## Troubleshooting

`template temporarily unavailable: ... the template just expired, try again
in a moment` is a normal, self-recovering message during mining, not an
error to worry about — the pool briefly had no fresh work ready between
requests. The miner keeps retrying and resumes on its own; the GPU stays
busy throughout. It only matters if it repeats for minutes without the GPU
resuming work.

## Developer fee

The miner directs 3% of mining time to a fixed developer wallet, in short,
disclosed intervals (`dev fee: entering/leaving …` in the console output) —
not configurable, applies equally to every user.

## License

Closed source, see [LICENSE](LICENSE). No redistribution, no reverse
engineering without permission.

## No affiliation

Independent, unofficial mining client — not affiliated with the Parano1d
project or any pool it connects to.
