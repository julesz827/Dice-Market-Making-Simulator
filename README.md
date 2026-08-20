# Dice Market-Making Simulator

A Monte Carlo simulation of a market maker quoting prices on a simple dice game,
exploring the core trading tradeoff between capturing spread and controlling
inventory risk.

## The question

A market maker earns money from the *spread* — buying slightly below fair value
and selling slightly above it — but every unmatched trade leaves them holding
inventory, which is risk. This project asks: **can a market maker stay profitable
while keeping inventory under control, and what does that tradeoff cost?**

## The model

The "asset" is the sum of two fair six-sided dice, which has a known expected
value of 7. The simulation has three parts:

- **A market maker** that quotes a bid and an ask centered on fair value, buys at
  its bid, and sells at its ask, tracking inventory and cash on every trade.
- **Players** who respond to price: the further the maker's quote sits from fair
  value, the more likely a player is to take the advantageous side.
- **Inventory skewing** — the key mechanism. When the maker's inventory drifts
  away from zero, it shifts its quotes to attract the trades that pull the
  position back, trading a little profit for lower risk.

Each configuration is run over 1,000 rounds and averaged across 50 trials for a
stable estimate.

## The finding

Sweeping the skew strength from 0 (no inventory control) upward produced a clear
tradeoff curve:

| Skew strength | Avg P&L | Avg inventory risk |
|:---:|:---:|:---:|
| 0.000 | 500.0 | 24.2 |
| 0.005 | 497.7 | 7.5 |
| 0.010 | 495.3 | 5.5 |
| 0.020 | 490.2 | 3.5 |
| 0.030 | 485.2 | 3.3 |
| 0.050 | 475.3 | 2.2 |

The two quantities fall at very different rates. A small amount of skew is
**nearly free risk reduction**: moving from 0 to 0.005 cuts average inventory
risk by about 70% (24.2 to 7.5) at a P&L cost of only 0.5% (500 to 497.7).

Beyond roughly 0.01, the curve shows **diminishing returns** — inventory risk is
already near its floor, so additional skew buys little further risk reduction
while costing steadily more P&L. The most efficient setting is a low skew around
0.005 to 0.01, which captures most of the risk benefit for minimal profit
sacrifice.

## The chart

`Chart.png` visualizes the tradeoff, with inventory risk and P&L on separate axes
against skew strength. Risk drops steeply then flattens; P&L eases gently
downward — the opposition between the two curves is the tradeoff itself.

## Limitations

The player model is intentionally simple and responds smoothly to price, which
makes P&L more stable across trials than a real market would be. A more realistic
extension would add noise to player behavior and allow the maker's fair-value
estimate to be imperfect — the scenario where inventory control matters most.

## Running it

```
python dice_market_maker.py
```

Requires `matplotlib`. All other logic uses the Python standard library.

## What I learned

- Why a market maker profits from the spread rather than from predicting price.
- How a feedback loop between quotes and order flow can either stabilize or
  destabilize a position depending on the sign of the correction.
- How to read a parameter sweep as a tradeoff curve and identify where
  diminishing returns begin, rather than optimizing a single number.
