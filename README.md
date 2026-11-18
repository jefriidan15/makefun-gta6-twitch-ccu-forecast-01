[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/jefriidan15/makefun-gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/jefriidan15/makefun-gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

# GTA VI Twitch CCU forecast — make.fun entry

Forecasts how many people watch Grand Theft Auto VI live on Twitch in each
hour of its release day (2026-11-19 UTC), as five quantiles per hour, from
everything recorded up to the end of the day's first hour.

## Method

A comparable-launch curve model, fitted at run time from the case's own
`comparables` (past big game launches' hourly Twitch category counts from
SullyGnome):

- Each comparable's release day is expressed as log-ratios to its own hour-1
  print, `lr_h = ln(viewers_h / viewers_1)`. Hour 1 anchors the scale;
  the comparables teach the day's *shape* — the early NZ/Australia hours,
  the US-east unlock at 05:00 UTC, the US-west unlock at 08:00 UTC, and
  the US prime-time evening peak.
- Per hour, a weighted local-linear regression of `lr_h` on
  `ln(viewers_1) - ln(p1)` (Gaussian weights, bandwidth 1.5 log units, so
  launches that opened at a similar magnitude count most). The fitted value
  at zero is the median log-ratio; the slope captures that smaller openings
  grow more in relative terms over the day. Quantiles are the fitted median
  plus the weighted quantiles of the regression residuals — the honest
  cross-launch dispersion, not a guessed spread.
- Guards: hours with fewer than 3 usable comparables fall back to
  persistence; the median log-ratio is clamped to [-1.5, 2.5]; quantiles
  are positive, non-decreasing, capped at 1e9.

## Validation

Scored offline against the historical source's own counts for the four
practice launches (which the kit's cases do not carry):

| evaluation | this entry | house baseline |
|---|---|---|
| practice cases (mean pinball loss) | **0.196** | 0.387 |
| leave-one-out, all 38 comparables | **0.555** | 0.930 |
| leave-one-out, 20 launches opening ≥ 50k | **0.209** | 0.251 |

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Python 3.13, standard library only, no network, deterministic.
`main.py` reads every `*.in` in the input directory and writes one
`*.out` per case: `{"hours": [{"hour": h, "q": [q05, q25, q50, q75, q95]}]}`.

## Competition

[gta6-twitch-ccu-forecast-01](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
— $250 escrowed, entries close 2026-11-19T01:00:00Z.
