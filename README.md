# A Mean-Field Game of Limit Order Book Dynamics

**Filippo Antonio Cadorin** · Research project (PRL), École Polytechnique, January to June 2026
Supervised by Prof. Michele Fabi (Télécom Paris, CREST)

📄 **[Read the report (PDF, 33 pages)](MFG_OrderBook_Filippo-Antonio-Cadorin.pdf)**

---

## The question

A limit order book (LOB) is the standing inventory of buy and sell orders at each price, and its *shape* (depth as a function of price) drives execution costs and liquidity. Most microstructure models take that shape as given: they either fit it to data or assume order arrivals follow an exogenous stochastic process (e.g. Cont, Stoikov and Talreja, 2010; Avellaneda and Stoikov, 2008).

This project asks whether the shape can instead be **derived from fundamentals**: if liquidity providers (LPs) differ only in how they value the asset and how patient they are, and compete à la Bertrand for incoming order flow, what book emerges?

## Approach

The model builds on the mean-field game (MFG) formulation of LOB competition of Ma and Noh (2020). A continuum of traders shares a single valuation distribution $F(v)$, which simultaneously determines who sends market orders, each LP's cost basis, and the buyer/seller boundary. LPs also differ in their discount rate $r \sim G(r)$. Each LP controls how quickly to replenish their queue at their chosen price, facing quadratic replenishment costs and rank-dependent execution intensities.

The equilibrium is a fixed point of three nested problems, solved numerically:

| Loop | Problem | Method |
|---|---|---|
| Inner | Each LP type's optimal replenishment policy (stationary HJB equation) | Policy iteration |
| Middle | Consistency between policies and the stationary distribution of queues (Kolmogorov forward equation) | Quantile fixed-point iteration |
| Outer | Each LP type's best price given aggregate depth posted by everyone else (Bertrand competition) | Damped Picard iteration |

No functional form for depth is imposed at any stage: the book is built from strategic primitives alone.

## Results

### Baseline model (the report)

* The solver produces a full two-sided book with a recognisable LOB shape. Depth accumulates away from the touch, because orders far from the mid are rarely filled and so build up.
* LPs **sort by patience**: for a given valuation, patient LPs post deep in the book (low fill probability, high margin) while impatient LPs post at the touch (margin traded for speed of execution).
* The bid-ask spread pins at the no-arbitrage floor. Every LP earns a positive margin even when quoting at the mid, so Bertrand competition drives both touches together. The report identifies this, together with convergence of the outer loop only in a Cesàro (time-averaged) sense, as the main limitations.

### Follow-up: endogenous spread

After the report, I extended the model to address both limitations:

* **Classification-consistent LP populations**, so that each side's marginal LP is defined by the opposite side's best quote. The two touches become a joint fixed point across both sides of the book.
* **Glosten–Milgrom adverse selection**: a share $\eta$ of order flow is informed, so a fill at a given price is partly bad news for the LP.
* **Price-priority fill rates**, a fixed inventory lattice for the state space, exact stationary distributions via a direct linear solve, and a continuous best response, which together let the outer loop converge on a residual criterion without averaging.

With a 1% fee and 5% informed flow, the spread is now determined by competition rather than by the price grid (0.085 on a unit price scale), and it increases with the informed share $\eta$. This is ongoing, exploratory work and goes beyond what is presented in the report.

## Repository contents

| File | Contents |
|---|---|
| [`MFG_OrderBook_Filippo-Antonio-Cadorin.pdf`](MFG_OrderBook_Filippo-Antonio-Cadorin.pdf) | Full write-up: theory, algorithm, results |
| [`LOB_Model.ipynb`](LOB_Model.ipynb) | Baseline solver and figures for the results in the report |
| [`LOB_Model_Fixed.ipynb`](LOB_Model_Fixed.ipynb) | Follow-up model (post-report): endogenous spread, residual-based convergence |
| [`LOB_Extensions.ipynb`](LOB_Extensions.ipynb) | Earlier exploration of two spread mechanisms: adverse selection and inventory risk |
| [`LOB_Model_3_discrete.ipynb`](LOB_Model_3_discrete.ipynb) | Robustness check with discrete valuation and discount-rate distributions |

**Where to start:** the report, then `LOB_Model.ipynb`.

## Reproducing the results

```bash
git clone https://github.com/SFilo18/mfg-orderbook-prl.git
cd mfg-orderbook-prl
pip install numpy scipy matplotlib joblib jupyter
jupyter notebook
```

Approximate run times on a single CPU core:

| Notebook | Run time |
|---|---|
| `LOB_Model.ipynb` | about 3 minutes |
| `LOB_Model_Fixed.ipynb` | about 40 minutes (several full equilibrium solves) |
| `LOB_Extensions.ipynb` | about 10 minutes, plus 30 to 40 minutes for the final parameter sweep |
| `LOB_Model_3_discrete.ipynb` | about 5 minutes |

The baseline code, run top to bottom from a clean kernel (Python 3.12, NumPy 2.x), reproduces the equilibrium reported in Table 5 of the report.

## Possible extensions

* Calibrate $F(v)$ and $G(r)$ to market data and compare the model book with observed snapshots.
* Replace the fixed mid price with a stochastic fundamental, adding mark-to-market risk for LPs.
* Introduce a designated market maker as a leader in a Stackelberg mean-field game (Zeng et al., 2025).

## Key references

* Ma, J. and Noh, E. (2020). Equilibrium model of limit order books: a mean-field game view. arXiv:2002.12857.
* Lasry, J.-M. and Lions, P.-L. (2007). Mean field games. *Japanese Journal of Mathematics*.
* Chan, P. and Sircar, R. (2015). Bertrand and Cournot mean field games. *Applied Mathematics & Optimization*.
* Glosten, L. and Milgrom, P. (1985). Bid, ask and transaction prices in a specialist market with heterogeneously informed traders. *Journal of Financial Economics*.
* Avellaneda, M. and Stoikov, S. (2008). High-frequency trading in a limit order book. *Quantitative Finance*.

The full bibliography is in the report.