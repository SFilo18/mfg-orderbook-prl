# A Mean-Field Game of Limit Order Book Dynamics

**Filippo Antonio Cadorin** · Research project (PRL), École Polytechnique, January to June 2026
Supervised by Prof. Michele Fabi (Télécom Paris, CREST)

📄 **[Read the report (PDF, 33 pages)](report/MFG_LOB_report_Cadorin_2026.pdf)**

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

### Baseline model (report, Notebook 1)

![Baseline order densities and LOB shape](figures/v1_lob_shape.png)

* The solver produces a full two-sided book with a recognisable LOB shape. Depth accumulates away from the touch, because orders far from the mid are rarely filled and so build up.
* LPs **sort by patience**: for a given valuation, patient LPs post deep in the book (low fill probability, high margin) while impatient LPs post at the touch (margin traded for speed of execution).
* The bid-ask spread pins at the no-arbitrage floor. Every LP earns a positive margin even when quoting at the mid, so Bertrand competition drives both touches together. The report identifies this, together with convergence of the outer loop only in a Cesàro (time-averaged) sense, as the main limitations.

### Follow-up: endogenous spread (Notebook 2)

After the report, I extended the model to address both limitations:

* **Classification-consistent LP populations**, so that each side's marginal LP is defined by the opposite side's best quote. The two touches become a joint fixed point across both sides of the book.
* **Glosten–Milgrom adverse selection**: a share $\eta$ of order flow is informed, so a fill at a given price is partly bad news for the LP.
* **Price-priority fill rates**, a fixed inventory lattice for the state space, exact stationary distributions via a direct linear solve, and a continuous best response, which together let the outer loop converge on a residual criterion without averaging.

![Model v2 LOB shape with endogenous spread](figures/v2_lob_shape_endogenous_spread.png)

With a 1% fee and 5% informed flow, the spread is now determined by competition rather than by the grid (0.085 on a unit price scale), and it increases with the informed share $\eta$. This is ongoing, exploratory work and goes beyond what is presented in the report.

## Repository structure

```
├── report/
│   └── MFG_LOB_report_Cadorin_2026.pdf     Full write-up: theory, algorithm, results
├── notebooks/
│   ├── 01_baseline_model.ipynb             Solver and figures for the results in the report
│   ├── 02_endogenous_spread.ipynb          Model v2 (post-report): endogenous spread, convergence
│   ├── 03_extensions_adverse_selection_inventory.ipynb
│   │                                       Earlier exploration of two spread mechanisms
│   └── 04_discrete_distributions.ipynb     Robustness check with discrete type distributions
├── figures/                                Figures used in this README
└── requirements.txt
```

## Reproducing the results

```bash
git clone https://github.com/SFilo18/PRL_tests.git
cd PRL_tests
pip install -r requirements.txt
jupyter notebook notebooks/
```

Each notebook is self-contained and runs top to bottom. Approximate run times on a single CPU core:

| Notebook | Run time |
|---|---|
| 01 Baseline | about 3 minutes |
| 02 Endogenous spread | about 40 minutes (several full equilibrium solves) |
| 03 Extensions | about 10 minutes (the parameter sweep is off by default) |
| 04 Discrete distributions | about 5 minutes |

Notebook 1 was re-run from a clean kernel (Python 3.12, NumPy 2.x) and reproduces the equilibrium reported in Table 5 of the report.

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
