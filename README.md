# Fisher Transform Alert — MQL4 Script

A MetaTrader 4 script that computes the **Fisher Transform** by normalizing the current close price within the period's highest high / lowest low range to `[−1, +1]`, applying the natural logarithm formula `0.5 × ln((1 + value) / (1 − value))` with hard clamp to `[−2.0, +2.0]`, and firing two independent alert types: zero-line sign-change crossovers indicating sharp trend reversals, and absolute value threshold breaches indicating statistically extreme readings — using `PrevFisher` persistent global state to enforce strict first-cross detection on the zero-line.

---

## Overview

The Fisher Transform, developed by John Ehlers, converts price into a Gaussian normal distribution by applying the inverse hyperbolic tangent (natural log form) to a normalized price position. Because natural price data is not normally distributed, the Fisher Transform produces sharper, more decisive turning points than linear oscillators — making zero-line crossovers more reliable as trend-change signals and extreme readings more statistically meaningful as overbought/oversold indicators. This script implements the full transform natively in MQL4: it resolves the period high and low via `iHighest()` / `iLowest()`, normalizes the current close to `[−1, +1]`, applies the log formula with division-by-zero protection via `MathMax(-0.999, MathMin(0.999, value))` clamping, and hard-limits the output to `[−2.0, +2.0]` via `MathMax` / `MathMin`.

---

## Features

- **Native Fisher Transform computation** — `CalculateFisherTransform()` resolves `highestHigh` and `lowestLow` via `iHighest()` / `iLowest()`, normalizes: `value = 2 × ((close − low) / (high − low) − 0.5)`, applies `0.5 × MathLog((1 + value) / (1 − value))`, clamps output to `[−2.0, +2.0]`
- **Zero-line crossover detection** — `PrevFisher != 0` guard skips the first cycle; thereafter: `currentFisher > 0 && PrevFisher < 0` → **Bullish Trend Change**; `currentFisher < 0 && PrevFisher > 0` → **Bearish Trend Change** — strict sign-change requirement prevents mid-zone false triggers
- **Extreme level detection** — `MathAbs(currentFisher) >= ExtremeLevel` fires independently of the crossover condition; both conditions can trigger in the same cycle for simultaneous trend-change + extreme alerts
- **Persistent `PrevFisher` state** — global double retains prior-cycle value across loop iterations for sign-change comparison; initialized to `0.0` to suppress first-cycle crossover false positives
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Alert message includes Fisher value and alert type with symbol and timeframe context

---

## How It Works

1. Every minute, `CalculateFisherTransform()` validates `iBars() >= period`, resolves high/low extremes, normalizes close, computes log transform, and returns clamped result
2. If `PrevFisher != 0`, sign-change crossover conditions are evaluated:
   - `currentFisher > 0 && PrevFisher < 0` → **Bullish Trend Change Detected**
   - `currentFisher < 0 && PrevFisher > 0` → **Bearish Trend Change Detected**
3. Independently: `MathAbs(currentFisher) >= ExtremeLevel` → **Extreme Level Detected**
4. `PrevFisher = currentFisher` updated at cycle end
5. `AlertFisher()` dispatches via all enabled channels with Fisher value and timeframe

---

## Input Parameters

| Parameter       | Type            | Default     | Description                                                          |
|-----------------|-----------------|-------------|----------------------------------------------------------------------|
| `TradeSymbol`   | string          | `EURUSD`    | Symbol for analysis                                                  |
| `Timeframe`     | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for analysis                                               |
| `FisherPeriod`  | int             | `10`        | Lookback bars for highest high / lowest low normalization range      |
| `ExtremeLevel`  | double          | `2.0`       | Absolute Fisher value at or above which an extreme alert is triggered|
| `EnableAlerts`  | bool            | `true`      | Fire an on-screen/sound alert                                        |
| `EnableEmail`   | bool            | `false`     | Send an email notification                                           |
| `EnablePush`    | bool            | `false`     | Send a mobile push notification                                      |

---

## Alert Message Format

```
Bullish Trend Change Detected detected on EURUSD (Timeframe: PERIOD_H1)
Fisher Value: 0.47
```

---

## Installation

1. Copy `Fisher_Transform_Alert_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
