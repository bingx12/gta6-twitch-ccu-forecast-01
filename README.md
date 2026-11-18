# GTA VI Launch-Day Twitch Forecast

[![make.fun](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/bingx12/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/bingx12/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

Entry for the [make.fun](https://make.fun/bounty/gta6-twitch-ccu-forecast-01) competition
`gta6-twitch-ccu-forecast-01`: forecast Grand Theft Auto VI's launch-day Twitch
concurrent viewers, hour by hour (hours 2–24 of 2026-11-19 UTC), as five quantiles
(5%, 25%, 50%, 75%, 95%) of each hour's mean minute count.

## How it works

`main.py` (Python 3.13, standard library only, no network, deterministic) reads
one case from `/input/<id>.in` and writes `/output/<id>.out`.

The forecast is a **kernel-weighted empirical quantile forecast**, fitted at run
time from the past big launches shipped in the case itself (`comparables`):

- Hour 1 alone is a poor anchor. Across 37 past launches the ratio of an
  hour's viewers to hour 1's ranges from ~0.5x (hype-heavy launches whose
  category was already huge before midnight) to 1000x+ (launches whose category
  barely existed before unlock).
- But the ratio depends strongly and smoothly on hour 1's own level: the bigger
  hour 1, the flatter the day. So for each hour we take the empirical quantiles
  of log(viewers<sub>h</sub> / viewers<sub>1</sub>) over past launches, weighted
  by a Gaussian kernel in log(hour-1 viewers) — bandwidth 1.0 in log units — so
  launches whose opening hour looked like this one dominate.
- The GTA VI category's own pre-release history is not a launch and is
  excluded, as is any comparable missing a full release day.
- If a case carries too few usable comparables, the program falls back to the
  house baseline (flat at hour 1, spread widening like a random walk).

## Local validation

Leave-one-out pinball loss on the 37 usable past launches (log scale, the
contest's own scoring), scored with the shipped code:

| model | all launches | high hour-1 launches (like GTA VI) |
|---|---:|---:|
| this entry (kernel, bw 1.0) | **0.410** | **0.205** |
| house baseline (starter) | 0.930 | 0.251 |
| unconditional per-hour quantiles | 0.714 | 0.362 |

Runs in ~0.06 s per case, byte-identical across runs.

## Run it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

## License

MIT — see [LICENSE](LICENSE).

---

Built for [make.fun](https://make.fun/r/k6eq9xtw) — join with my referral link:
https://make.fun/r/k6eq9xtw
