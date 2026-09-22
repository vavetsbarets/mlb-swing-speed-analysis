# Batter's Swing Speed as an Exogenous and Endogenous Variable

An analysis of MLB Statcast bat-tracking data from the 2024 season, examining how a batter's
swing speed relates to swing outcomes, and what game-state factors drive how fast a batter
swings.

Bat-tracking metrics (bat speed, swing length) became publicly available part-way through the
2024 season, so at the time of this analysis there was essentially no prior methodology for
working with them.

**Author:** Vladimir Averin, PhD candidate, Department of Statistics & Data Science, Yale
University

**Submitted as:**
- Final project for S&DS 625, *Statistical Case Studies*, Yale University (December 2024).
  The report served as the written data-analysis component of the Yale S&DS PhD qualifying
  examination.
- Entry to the [CSAS 2025 Data Challenge](https://statds.org/events/csas2025/challenge.html),
  Connecticut Sports Analytics Symposium, graduate track (January 2025).

---

## The question

The analysis treats swing speed in two opposite ways.

**Swing speed as a cause (exogenous).** Assuming a batter can choose how hard to swing, how
does swing speed change the probability of putting the ball in play? Is there an interior
optimum — a point past which swinging harder starts to hurt?

**Swing speed as an outcome (endogenous).** Treating swing speed as a readout of the batter's
strategy and caution, which game-state and pitch characteristics make batters swing faster or
slower?

---

## Data

Source: MLB Statcast pitch-level data for the 2024 regular season and playoffs
(2 April – 30 October 2024), as released for the CSAS 2025 Data Challenge. The same data are
publicly available from [CSAS 2025 Data Challenge](https://statds.org/events/csas2025/challenge.html).

Raw dimensions: 701,557 pitches × 113 variables — game state, players and base runners, pitch
velocity, trajectory, spin and location, swing and bat-tracking measurements, and outcomes.

### Processing

| Step | Result |
|---|---|
| Raw pitches | 701,557 |
| Remove 286 corrupted records, restore true in-game order | — |
| Keep swings only (~45% of pitches), remove bunts | — |
| Drop each batter's slowest 10% of swings (isolates committed swings from checked ones) | 288,630 |
| Restrict to fastballs | — |
| Group by batter × batting side (handles 55 switch-hitters) | — |
| Require ≥30 committed fastball swings per batter-side | — |
| **Analysis dataset** | **92,903 swings, 509 batters, 2,354 games** |

Outcome base rate: 34% of swings put the ball in play.

---

## Method

The substantive work is in the feature engineering.

**Within-batter normalisation.** The relationship between swing speed and contact reverses
sign between levels: batters who swing faster *on average* put fewer balls in play, but for
any *given* batter, swinging faster than his own average is associated with more balls in
play. Demeaning swing speed by batter separates the between- and within-batter effects and
makes pooled analysis interpretable.

**Strike-zone coordinate normalisation.** Pitch location rescaled so the zone's side edges are
−1 and +1 and its bottom and top are 0 and 5.2, preserving the real aspect ratio, so locations
are comparable across batters of different heights.

**Handedness inversion.** After examining all four pitcher-hand × batter-side combinations
separately, mirroring horizontal location and horizontal movement for left-handed batters makes
the four patterns nearly identical — allowing all combinations to be pooled into one model with
a single same/opposite-hand indicator rather than four separate models.

**Other features.** Piecewise (hinge) terms at zone edges where the slope changes; pitch speed
decomposed into the pitcher's own average versus deviation from it; per-batter demeaned
strike-zone bottom as a proxy for stance depth; count encoding reflecting the non-linear effect
of reaching two strikes.

**Models.** Logistic regression for the exogenous question (19 predictors, 7 interactions), plus
370 per-batter models fitted separately for every batter with more than 100 swings. Linear
regression for the endogenous question, benchmarked against LightGBM to check that a non-linear
model found no additional structure.

---

## Findings

**Swing speed and contact.** Swinging faster than one's own average is associated with a higher
probability of putting the ball in play — logit coefficient 0.085 per mph (p < 2e-16), roughly
9% higher odds per mph above the batter's own average. No interior optimum: the squared term is
insignificant in both the simple (p = 0.50) and full (p = 0.26) models. The effect is stable
across individual batters and stays positive even under the least favourable realistic scenario
(100 mph pitch, far edge of the zone, opposite-handed pitcher, 0–2 count).

**What drives swing speed** (R² = 0.185). Count is the strongest game-state driver: each strike
lowers swing speed by 0.89 mph, each ball raises it by 0.46 mph — batters protect the plate when
behind and swing freely when ahead. Pitch location is the other major driver, with the fastest
swings on pitches middle-horizontal and low in the zone. Faster pitches relative to the pitcher's
own average lower swing speed. Later innings lower swing speed, and away-team batters slow
further from the 7th inning on. Runners on base lower swing speed, increasing from first
(−0.10 mph) to third (−0.19 mph). Outs and run differential show no detectable effect.

**Batter identity alone explains 36% of raw swing-speed variance.**

**Alternative outcomes.** Probability of contact gives the same conclusions. xwOBA was explored
as a target and rejected with documented reasons — its distribution is dominated by zeros with
rare large values and is non-monotonic and non-smooth in exit velocity and launch angle, giving
unstable estimates. One consistent pattern did emerge: xwOBA is effectively capped until swing
speed crosses roughly 66–72 mph, above which high-value outcomes appear quickly.

---

## Limitations

**The exogenous result is not causal.** A batter who reads a pitch as easy is likely both to
swing harder and to make contact, so batter confidence confounds the swing speed–outcome
relationship. The reported association should be read observationally.

**This analysis uses bat speed only, not swing length.** Swing length is available in the same
data and is not used here.

**Fastballs only.** Other pitch types were deliberately excluded rather than pooled, and are
left for separate analysis.

**Most variance in swing speed is unexplained.** Likely candidates: omitted factors and
measurement noise. Natural next steps include swing-speed autocorrelation within an at-bat,
fatigue measures, extreme-value analysis of the fastest swings, integration with swing-decision
models, and extension to other pitch types.

---

## Repository contents

| File | Description |
|---|---|
| `Batters_Swing_Speed_as_Exogenous_and_Endogenous_Variable.Rmd` | Full analysis: data processing, feature engineering, models, figures |
| `Batters_Swing_Speed_as_Exogenous_and_Endogenous_Variable-compressed.pdf` | Knitted report submitted to the CSAS 2025 Data Challenge |


## Running the analysis

Built in R. Main packages: `arrow`, `dplyr`, `ggplot2`, `patchwork`, `pubtheme`, `lightgbm`.

Download the 2024 Statcast data from [here](https://statds.org/events/csas2025/challenge.html), place the CSV in the working directory,
adjust the file path at the top of the `.Rmd`, and knit.
