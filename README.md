# Closed-Form Optimal Window Width for wNAF Elliptic Curve Scalar Multiplication

[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-blue.svg)](https://www.python.org/)
[![Curves](https://img.shields.io/badge/Curves-secp256k1%20%7C%20NIST%20P--256-green.svg)]()
[![Measurements](https://img.shields.io/badge/Benchmarks-177%2C000%20samples-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-purple.svg)]()

This repository contains the official benchmark suite, mathematical framework, and reproducibility dataset for the research paper:
> **"Closed-Form Optimal Window Width for wNAF Elliptic Curve Scalar Multiplication under Memory and Table-Reuse Constraints: Theory and Statistical Validation"**  
> *Carried out within project AL-9624115223 at the National University of Uzbekistan.*

---

## 🔬 Overview & Key Contributions

Scalar multiplication $Q = [k]P$ is the core building block of Elliptic Curve Cryptography (ECDSA, EdDSA, ECDH). While width-$w$ Non-Adjacent Form ($w$NAF) windowing reduces the additions in the main loop, choosing $w$ has traditionally relied on exhaustive trial-and-error profiling or rough heuristics.

This project introduces a **closed-form theoretical decision rule** and validates it on **177,000 empirical timing measurements**:

1. **Closed-Form Theorem ($w^*$):** We prove that expected cost is strictly unimodal in $w$, yielding:
   $$w^* = \min \left\{ w \ge 2 : 2^{w-2}(w+1)(w+2) \ge \kappa \cdot q \cdot t \right\}$$
   where $t$ is scalar bit-length, $q$ is table-reuse frequency, and $\kappa = c_A / c_T \approx 0.468$ (ratio of mixed addition to marginal table point generation).
2. **Renewal Theory Distribution:** We prove that non-zero digit count has asymptotic variance $\frac{2t}{(w+1)^3}$ and converges to $\mathcal{N}(0, 1)$.
3. **Memory-Constrained Optimality:** Under budget $B$ bytes, optimal window width is given strictly by $\min(w^*, w_B)$ with $w_B = 2 + \lfloor \log_2(1 + B/m_P) \rfloor$.
4. **Endomorphism (GLV) & Comb Extensions:** Closed-form bounds extend directly to GLV sub-scalar lengths $t_{\text{GLV}}$ and Lim–Lee comb break-even points.
5. **Execution Time Model:** A 3-parameter field-operation regression predicts scalar execution time on unseen scalars with $R^2 \ge 0.993$.

---

## 📊 Summary of Benchmark Results

All timings measured using `time.perf_counter_ns` across 50 random scalars $\times$ 20 repetitions (paired Wilcoxon tests with Holm adjustment, 95% bootstrap CIs):

### 1. Speedup over Binary ($t=256$, Variable Base, $q=1$)
| Curve | Method | Mean (ms) | 95% CI (ms) | Speedup vs Binary |
| :--- | :--- | :---: | :---: | :---: |
| **secp256k1** | Binary | 2.403 | [2.385, 2.423] | 1.00x |
| | NAF ($w=2$) | 2.174 | [2.165, 2.183] | 1.11x |
| | $w$NAF ($w=5$) | 1.973 | [1.967, 1.980] | **1.22x** |
| | GLV + $w$NAF ($w=5$) | 1.318 | [1.313, 1.323] | **1.82x** |
| | OpenSSL 4.0.3 | 1.036 | [1.033, 1.039] | 2.32x |
| | libsecp256k1 (coincurve) | 0.232 | [0.231, 0.234] | 10.34x |
| **NIST P-256** | Binary | 2.346 | [2.335, 2.357] | 1.00x |
| | NAF ($w=2$) | 2.126 | [2.117, 2.135] | 1.10x |
| | $w$NAF ($w=5$) | 1.924 | [1.919, 1.927] | **1.22x** |
| | Fixed-Window ($w=4$) | 2.007 | [2.004, 2.010] | 1.17x |
| | OpenSSL 4.0.3 | 0.212 | [0.211, 0.213] | 11.08x |

### 2. Theorem 1 Accuracy ($\kappa_{\text{theory}} = 0.468$)
* **Exact matches:** 49 / 60 parameter test cases (81.7%).
* **Within $\pm 1$ step:** 60 / 60 cases (**100.0%**).
* **Mean empirical penalty:** Only **0.13%** (maximum penalty of 1.70% at boundary near-ties).
* **Memory budget predictions:** Exact in 124 / 128 constrained evaluations.

---

## 🛠 Repository Structure

```text
├── ecbench_tajriba.ipynb       # All-in-one runnable Jupyter Notebook
├── ec_core.py                  # Elliptic curve arithmetic (Jacobian, EFD formulas, Fp)
├── algorithms.py               # Binary, wNAF, Fixed-Window, Ladder, GLV, and Comb
├── bench.py                    # High-precision multi-run benchmarking runner
├── analyze.py                  # Statistical analysis (Bootstrap CI, Wilcoxon, Regression)
├── requirements.txt            # Python dependencies
└── results/                    # Generated datasets and figures
    ├── summary.md              # Complete formatted scientific result tables
    ├── S1_times.csv            # Raw timing statistics & noise CV
    ├── S2_speedup.csv          # Speedups, bootstrap intervals & Holm p-values
    ├── S3_op_costs.csv         # Measured group operation durations (tau) & kappa
    ├── S4_density.csv          # Nonzero digit density vs t/(w+1) theoretical limit
    ├── S5_optimal_w.csv        # Empirical vs Theorem-predicted optimal window sizes
    ├── S6_memory.csv           # Budget-constrained window selection validation
    ├── S7_prediction.csv       # Linear regression execution time cross-validation
    └── *.png                   # Evaluation plots (speedups, optimal w, prediction)
