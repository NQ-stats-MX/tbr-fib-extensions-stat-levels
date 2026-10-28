# NY TBR Stat Map [XelMod]

A TradingView **Pine Script v6** overlay indicator for **mean reversion trading off a Time Based Range (TBR)**. It draws a range you define, projects Fibonacci extensions beyond it, and adds historical statistics: how often price actually reaches each extension, and percentile bands for the session's maximum excursion from the open. This lets you judge levels by probability instead of by eye.

Built and tuned for **NQ / MNQ futures**. However, it works on any symbol with regular intraday sessions.

---

## Lineage

This is a **fork of a fork**:

| Version | Author | Notes |
|---|---|---|
| Original | © **Lucymatos** | Time Based Range concept: range box, session open, hi/lo/eq, Fibonacci levels and extensions |
| Mod | **XeL_Arjona** | Pine v6 rewrite, extension hit-rate statistics, session excursion percentile levels, range-scaled (R) mode |
| This fork | **Aidan** | Time-picker session inputs, documentation, mean-reversion workflow |

Licensed under the **Mozilla Public License 2.0**, the same license as the original work. See [License](#license).

---

## The idea

The indicator is built around one workflow:

1. **Pick a Time Based Range.** It is a fixed window of time whose high and low set the reference range (**R**).
2. **Fade the ±0.33 R extensions.** These are the first projections outside the range. Price often probes them and returns into the range. They are the primary mean-reversion entries.
3. **Look for exhaustion in the ±1.33 to ±1.66 R zone.** When price runs past the first extension, this band is where extended moves tend to stall. Look for exhaustion and reversal here rather than chasing the move.

The statistics layers tell you how often each of those levels has actually been touched in recent sessions. You can see whether a fade at +0.33 R is fading something that usually holds, or something that usually breaks.

### Suggested ranges

| Range | Time (NY) | Notes |
|---|---|---|
| **NY AM** | **08:12 – 09:12** | Pre-open range ahead of the cash session |
| **Globex** | **18:00 – 19:00** | First hour of the overnight session |

> ⚠️ **Timeframe alignment.** The statistics are collected on the **History timeframe** (default 30m). Every session boundary must fall on a bar of that timeframe, or the stats will be slightly off.
> - **08:12 – 09:12** needs a History timeframe that divides 12 minutes evenly: **1, 2, 3, 4, 6 or 12 minutes**. **3m** is a good compromise between precision and sample depth.
> - **18:00 – 19:00** works with **15m or 30m**.
>
> Also set the **Extension Window** to start where the range ends (for example, 09:12 for the NY AM range).

---

## What it draws

### 1. Time Based Range
- **Range box** covering the high/low of the range session
- **Session open** line (the open of the range session)
- **Hi / Lo / Eq** (high, low and midpoint of the range)
- Optional **Fibonacci retracements** inside the range (0.236 / 0.382 / 0.618 by default)
- Optional **session anchor**, a vertical line at the range start

### 2. Fibonacci Extensions (the core levels)
Projections beyond the range, measured in multiples of the range size **R**:

- Up: `Range High + k × R`
- Down: `Range Low − k × R`

Defaults are **0.33, 1.33 and 1.66**. Each extension label can show its **historical touch rate**, for example `+0.33 R · 62%`.

Levels extend to the right through the **Extension Window**, then freeze. The last *N* sessions stay on the chart as history.

### 3. Extension Hit Rate (statistics)
For each past session, the indicator records whether price touched each extension level (up and down separately) during the Extension Window. The touch rate is the share of the last *N* sessions in which it happened.

- Only **prior sessions** are used, so historical labels show what you would have seen live. Nothing repaints.
- Rates are shown in the extension labels and in an optional **hit-rate table**.
- Below a minimum sample size, the table shows `n<10` instead of a number.

### 4. Session Excursion Levels (statistics)
A separate layer for the **Excursion Session** (default 09:30 – 16:00 NY). For each past session, it measures the maximum distance price traveled **above** and **below** that session's open. Then it plots percentile bands of that distribution from today's open:

- Four percentile levels per side (default P25 / P50 / P75 / P90) plus the mean
- Labels show either **touch probability** (`75% touch ↑` means about 75% of past sessions reached it) or the raw percentile (`P25 ↑`)
- Levels lock at the session open and don't change intraday

Three scaling modes are available:

| Mode | How it works |
|---|---|
| **Raw** | Percentiles of past log-excursions, applied to today's open as is |
| **Vol-adjusted** | Each past excursion is divided by an EWMA of prior session ranges, and the levels are rescaled by today's EWMA. It adapts to volatility regimes |
| **Range (R)** *(default)* | Each past excursion is divided by the TBR size known at that session's open, and the levels are rescaled by today's TBR. Uses the same R units as the Fib extensions |

---

## Settings

### Session Settings
| Setting | Default | Description |
|---|---|---|
| Range Session | 04:00 – 09:30 | Time window that builds the range. The end is exclusive. |
| Extension Window | 09:30 – 11:30 | How far levels extend right, **and** the window where extension touches are measured. Start it at the range end. |
| Time Zone | America/New_York | Used by the range, the window and the excursion session. |

### Anchor / Session Open / Range Box
Toggles, line styles, colors and widths for the anchor line, the session open line and the range box.

### Fibonacci Settings (inside the range)
| Setting | Default | Description |
|---|---|---|
| Show Fibonacci Levels | off | Retracement levels inside the range |
| Fib Level 1 / 2 / 3 | 0.236 / 0.382 / 0.618 | Retracement ratios |
| Fibonacci Anchor | Low | Measure from the range **Low** or the session **Open** |

### Fibonacci Extension Settings
| Setting | Default | Description |
|---|---|---|
| Show Fibonacci Extensions | on | Projections beyond the range |
| Fib Ext Level 1 / 2 / 3 | 1.33 / 1.66 / 0.33 | Extension multiples of R |
| Style / color / width / labels | — | Appearance |

### Extension Hit Rate
| Setting | Default | Description |
|---|---|---|
| Hit-rate sample (sessions) | 60 | Number of past sessions in the touch-rate calculation |
| Min sessions to show | 10 | Hide the rates until the sample reaches this size |
| Add touch % to extension labels | on | Appends `· xx%` to each extension label |
| Show hit-rate table | on | Summary table of up/down touch rates |
| Table position | Top Right | Corner of the chart |

### Session High/Low Settings
Toggle and style for the range Hi / Lo / Eq lines and labels.

### Display Settings
| Setting | Default | Description |
|---|---|---|
| Sessions to Display | 10 | Past ranges kept on chart (max 15 because of TradingView's 500-object limit) |
| Projection Offset (Bars) | 8 | How many bars levels extend past the current bar |
| Anchor Label Size | Small | — |
| Price Decimal Precision | 2 | Decimals in price labels |

### Range Label Sizes
Separate label sizes for the open, Hi/Lo/Eq, Fib levels and Fib extensions.

### Excursion Session
| Setting | Default | Description |
|---|---|---|
| Excursion Session | 09:30 – 16:00 | Session whose open-to-high and open-to-low excursions are measured |
| History timeframe | 30 | Timeframe used for **all** statistics (excursions and hit rates). A higher timeframe gives a deeper sample. It must align with every session boundary. |
| Sessions in sample | 60 | Past sessions in the excursion distribution |

### Excursion Scaling
| Setting | Default | Description |
|---|---|---|
| Levels based on | Range (R) | Raw / Vol-adjusted / Range (R), described above |
| Vol memory (EWMA span) | 10 | Only for Vol-adjusted. A longer span is smoother and slower to react. |

### Excursion Levels
Four percentile slots (show toggle and percentile value), mean line, open line, label toggle, label mode (touch probability or percentile) and label size.

### Excursion Colors
Colors for the open, the four upper and four lower levels, and the upper and lower means.

---

## Suggested setup

**NY AM range**
```
Range Session      08:12 – 09:12
Extension Window   09:12 – 11:30   (adjust to your trading window)
History timeframe  3
Fib Ext Levels     1.33 / 1.66 / 0.33
Excursion scaling  Range (R)
```

**Globex range**
```
Range Session      18:00 – 19:00
Extension Window   19:00 – <end of your overnight window>
History timeframe  30
```

Run the chart on **1m** for execution and let the History timeframe handle the statistics.

---

## Installation

1. Open TradingView and go to **Pine Editor**.
2. Paste the contents of the `.pine` file.
3. Click **Save**, then **Add to chart**.
4. Open the indicator settings and set your Range Session, Extension Window and History timeframe.

---

## Notes and limitations

- **No repainting of statistics.** Hit rates and excursion percentiles update only when a session completes. Excursion levels lock at the session open.
- **Sample depth depends on the History timeframe.** Lower timeframes give more precise boundaries but fewer sessions of history. If the table stays at `n<10`, raise the timeframe (keeping it aligned) or lower the sample requirement.
- **Truncated first session.** A range that starts on the first loaded bar is ignored, because it may be incomplete.
- **Object limits.** TradingView caps lines and labels at 500, which is why *Sessions to Display* maxes at 15.
- Statistics describe past behavior. They are not a forecast or a trading signal.

---

## License

This work is subject to the terms of the **Mozilla Public License 2.0** (https://mozilla.org/MPL/2.0/).

Original work and code © **Lucymatos**. Modifications by **XeL_Arjona**. Further modifications by **Aidan**.

## Disclaimer

For educational and research purposes only. Not financial advice. Futures trading involves substantial risk of loss.
