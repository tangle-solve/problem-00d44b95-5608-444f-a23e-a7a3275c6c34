<!-- solve-problem:00d44b95-5608-444f-a23e-a7a3275c6c34 -->
# AMM privacy versus quote accuracy

A [Tangle Solve problem](https://solve.tangle.tools/problems/00d44b95-5608-444f-a23e-a7a3275c6c34).

## What is the problem?

A constant function market maker (CFMM) such as Uniswap maintains public reserves $(x, y)$ and quotes a marginal price $p(x,y)$ derived from the trading function $\varphi(x,y) = k$. Every trade moves the reserves, so an observer of on-chain state learns information about individual trades. Angeris, Evans and Chitra (2021) formalized this: any ordinary CFMM implementation **cannot** provide privacy, because the post-trade reserves and price fully determine (approximately) the trade that just occurred. Their remedy is to perturb the publicly *quoted* price with noise calibrated for differential privacy (DP): the reported price $\tilde p = p + \eta$ satisfies an $(\varepsilon, \delta)$-DP guarantee against inference about a single trade. But noise degrades quote accuracy, and repeated/adaptive queries compound the leakage.

**The open problem:** derive *tight* three-way tradeoffs among (i) privacy parameters $(\varepsilon,\delta)$, (ii) the accuracy of the quoted price (e.g., $\ell_\infty$/mean-squared error between $\tilde p$ and true $p$), and (iii) **delay** (how long a quote may lag / how often the true state is revealed), when the adversary is *adaptive* — choosing the next query based on all previous noisy outputs. Then exhibit mechanisms (noise-adding schemes, batched/revealed-lag quotes, state-release schedules) achieving the lower bounds, i.e., matching constructions.

Formal setting: mechanism $M$ releases noisy functions of a monotonically updated reserve state; adversary issues $T$ adaptive queries; require $(\varepsilon,\delta)$-DP at the level of a single trade and bound $\sup \, \mathbb{E}\|M(s) - p(s)\|$. "Delay" may be modeled as the staleness window between a trade and its effect on quotes; this is the least standardized part of the formulation — the solver must fix a definition (verify; the census record does not pin it down).

## Why does it matter?

Solving this would tell DEX designers exactly how much privacy is purchasable at what quote-accuracy and latency cost, and give concrete mechanism designs rather than ad hoc noise injection. It also connects the AMM privacy problem to the mature adaptive-data-analysis literature (reusable DP, answer accuracy under adaptivity), potentially importing sharp lower-bound machinery. If it stays open, privacy-preserving AMM designs remain heuristic, with unquantified leakage under persistent blockchain observers.

## What's already known?

- Angeris, Evans and Chitra (arXiv:2103.01193) prove the impossibility for ordinary CFMMs: from public reserves and the trading function one can essentially invert to recover trades; they then define DP for the reported-price mechanism and give a Laplace/Gaussian-style noise mechanism with an accuracy-vs-$\varepsilon$ tradeoff for **non-adaptive** observation. They do not address adaptive probing, delay, or lower-bound tightness.
- From the broader DP literature: adaptive-query lower bounds are known — answering $T$ adaptive statistical queries to accuracy $\alpha$ requires $\varepsilon \gtrsim \sqrt{T}\,\alpha$-type tradeoffs (Dwork, Rothblum, Vadhan 2010; Bassily et al. 2016; Steinke–Ullman 2017). These are the natural template, but the AMM setting differs: the "database" (reserves) is *updated* by the very trades being hidden, prices are nonlinear in state, and the adversary is a continuous blockchain observer, not a query interface.
- What's missing: no matching constructions with adaptive guarantees in the AMM model; no formalization of delay/staleness; no proof that the known tradeoffs are tight in this setting.

## What would count as solving it?

1. **Lower bound:** a theorem showing any mechanism that is $(\varepsilon,\delta)$-DP per trade, answers adaptive price queries within accuracy $\alpha$, and reveals state with staleness at most $d$, must satisfy an explicit inequality among $\varepsilon, \delta, \alpha, d, T$ — against an explicit adaptive adversary.
2. **Matching construction:** a mechanism (noise distribution + quote-staleness schedule) that is $(\varepsilon,\delta)$-DP and achieves accuracy/delay within constants of the lower bound.
3. Together, these constitute the "tight privacy–accuracy–delay tradeoff with matching constructions" demanded by the census record. A referee would want the model defined (state update rule, adversary's observation set, DP adjacency notion = single trade), the lower bound proven, and the mechanism analyzed under the same definitions.

## How to start

1. Read arXiv:2103.01193 end to end — it's short; extract their exact DP adjacency definition and noise mechanism. Pin down the model precisely, especially "delay" (their setting reveals current noisy price; a lagged/batched variant is the natural extension).
2. Read the adaptive-data-analysis lower bounds: Dwork–Rothblum–Vadhan (FOCS 2010) and Bassily et al. (J. Privacy Confidentiality 2016) for the $\sqrt{T}$-style accuracy–privacy tradeoff machinery; check whether the numeric/one-dimensional nature of a price quote (a single real, not a histogram) breaks or preserves those bounds — this is the key modeling tension.
3. Main obstacle: the adversary sees *all* noisy outputs continuously, and trades change the true price, so standard "same dataset, many queries" arguments don't directly apply. Consider modeling the stream of trades as the dataset and the price as a post-processing of it, then apply DP composition.
4. For constructions: Gaussian noise on the quote plus releasing the true price only every $d$ blocks is the natural candidate; analyze its adaptive accuracy via DP amplification/post-processing lemmas.

## Key references

- Guillermo Angeris, Tarun Chitra, Theo Diamandis, Alex Evans, "A Note on Privacy in Constant Function Market Makers" — note: verify author order; the census cites Angeris, Evans and Chitra (2021), arXiv:2103.01193. Proves that public CFMM state reveals trades and constructs DP noise mechanisms for quoted prices, with accuracy tradeoffs in the non-adaptive case. https://arxiv.org/abs/2103.01193
- Cynthia Dwork, Guy Rothblum, Salil Vadhan, "Boosting and Differential Privacy" (FOCS 2010) — introduces reusable DP and the fundamental accuracy–privacy tradeoff under adaptive queries. https://arxiv.org/abs/0801.3961 (verify exact arXiv ID)
- Raef Bassily, Kobbi Nissim, Adam Smith, Thomas Steinke, Uri Stemmer, Jonathan Ullman, "Algorithmic Stability for Adaptive Data Analysis" (2016) — gives tight (up to constants) $\alpha$ vs $\varepsilon$ vs $T$ tradeoffs for adaptive queries. https://arxiv.org/abs/1511.02513
- Thomas Steinke, Jonathan Ullman, "Between Fingerprinting and the Birthday Attack" (2017) — nearly tight lower bounds for adaptive queries under DP. https://arxiv.org/abs/1607.04454

## Warnings

- The 2021 paper is a "note" — informal in places; its DP adjacency definition (what counts as neighboring trades) must be made fully precise before proving anything, and different choices change the answer.
- "Delay" is not defined in the census record or (clearly) in the literature; any solution must define it and note the choice is an interpretation, since it changes the tradeoff surface.
- Don't assume the mechanism in the note achieves its accuracy guarantee adaptively — its analysis is essentially non-adaptive; adaptive recomputation via composition is where the real work lies.
- Continuous blockchain observation is stronger than a $T$-query interface; a lower bound proven only for query-bounded adversaries may not apply to the on-chain setting.

---
*`BCGT-000356` · flagged for review*

## Acceptance guidance

1. **Lower bound:** a theorem showing any mechanism that is $(\varepsilon,\delta)$-DP per trade, answers adaptive price queries within accuracy $\alpha$, and reveals state with staleness at most $d$, must satisfy an explicit inequality among $\varepsilon, \delta, \alpha, d, T$ — against an explicit adaptive adversary.
2. **Matching construction:** a mechanism (noise distribution + quote-staleness schedule) that is $(\varepsilon,\delta)$-DP and achieves accuracy/delay within constants of the lower bound.
3. Together, these constitute the "tight privacy–accuracy–delay tradeoff with matching constructions" demanded by the census record. A referee would want the model defined (state update rule, adversary's observation set, DP adjacency notion = single trade), the lower bound proven, and the mechanism analyzed under the same definitions.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/00d44b95-5608-444f-a23e-a7a3275c6c34?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000356.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000356)
