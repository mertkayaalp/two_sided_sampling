# Signal Recovery from Time and Frequency Samples — Numerical Experiments

The code for the paper:

> M. Kayaalp and O. Szehr, **"Signal Recovery from Time and Frequency Samples,"**
> arXiv:2603.16242 [eess.SP].

The paper studies **two-sided sampling**: reconstructing a signal $f$ from samples of
$f$ (time domain) *and* of its Fourier transform $\hat f$ (frequency domain), under a
fixed total sample budget. The experiments here compare the conditioning of one-sided
vs. two-sided sampling matrices in Hermite- and sinc-based finite-dimensional spaces,
and demonstrate a spectrum-monitoring application where undersampled time data are
supplemented with a few DFT bins.

## Requirements

- Python ≥ 3.10
- `numpy`, `matplotlib`, `jupyter`
- `scipy` ≥ 1.16 (the streaming notebook uses the `rtol` keyword of
  `scipy.sparse.linalg.cg`; older SciPy called it `tol`)

```
pip install numpy scipy matplotlib jupyter

```

Please note that the code is not generally perfected for performance. It is rather meant to illustrate the results from the paper. The code is provided as-is without guarantees.

## Repository layout

| Path | Contents |
|---|---|
| `finite_hermite.ipynb` | Invertibility heatmap of the 3×3 mixed sampling matrix in the Hermite space $\mathcal H_2$ (1 time sample at $t_0=0$, 2 frequency samples swept over $(\omega_0,\omega_1)$). |
| `hermite_sampling_condition.ipynb` | Condition numbers $\kappa_2$ of pure-time vs. mixed time–frequency sampling matrices for Hermite function spaces. Includes regular/random sampling, $d$-dependent intervals, and the DFT-stabilizer experiment. |
| `sinc_sampling_condition.ipynb` | Same condition-number study for integer-shifted sinc (bandlimited) bases, including $d$-dependent shift ranges. |
| `streaming_recon.ipynb` | Spectrum-monitoring demo: reconstruct a $W=1024$ block from every-other time sample plus the top-$K$ DFT bins (matrix-free CG, ring-buffer streaming engine, parameter sweep). |
| `paper.pdf` | Paper corresponding to the numerical experiments. |
| `results/` | Generated artifacts, grouped by notebook. This directory is created automatically when notebooks are run. |

Generated artifacts are organized as follows:

| Directory | Producer |
|---|---|
| `results/finite_hermite/` | `finite_hermite.ipynb` |
| `results/hermite_sampling_condition/` | `hermite_sampling_condition.ipynb` |
| `results/sinc_sampling_condition/` | `sinc_sampling_condition.ipynb` |
| `results/streaming_recon/` | `streaming_recon.ipynb` |

## Notebook → paper figure map

| Paper figure | Notebook | Configuration | Output file |
|---|---|---|---|
| Fig. 1 | `finite_hermite.ipynb` | $d=3$, $t_0=0$, singularity threshold $1.85\times10^{-5}$ | `results/finite_hermite/mixed_sampling.png` |
| Fig. 2 | `hermite_sampling_condition.ipynb` | `TIME_INTERVAL=(1,2)`, `FREQ_INTERVAL=(-1,0)`, `TIME_RATIO=0.5` (repo default) | `results/hermite_sampling_condition/hermite_condition_number_comparison.png` |
| Fig. 3 | `hermite_sampling_condition.ipynb` | **Set `TIME_INTERVAL=(-1,1)`, `FREQ_INTERVAL=(-1,1)` and re-run** | `results/hermite_sampling_condition/hermite_condition_number_all_comparison.png` |
| Fig. 4 | `hermite_sampling_condition.ipynb` | DFT-stabilizer experiment (Experiment 3), default intervals | `results/hermite_sampling_condition/hermite_condition_number_dft_regular.png` |
| Fig. 5 | `sinc_sampling_condition.ipynb` | Experiment 2b: $d$-dependent shifts/time interval, `FREQ_INTERVAL=(-3,3)`, random sampling | `results/sinc_sampling_condition/sinc_condition_number_ddep_random.png` |
| Fig. 6 | `streaming_recon.ipynb` | "Three-way reconstruction comparison" cell: $W=1024$, $N_u=2$, $K\in\{2,4\}$, SNR 16 dB, `np.random.seed(55)` | `results/streaming_recon/reconstruction_comparison_three_plots.png` |

## Conventions

- **Fourier transform:** unitary convention,
  $\hat f(\omega) = \frac{1}{\sqrt{2\pi}}\int f(t)\,e^{-i\omega t}\,dt$.
- **Hermite functions:** evaluated by the stable three-term recurrence;
  eigenfunctions of the Fourier transform, $\hat\varphi_n = (-i)^n \varphi_n$,
  so the frequency block of the mixed matrix is $B_\omega\,\mathrm{diag}((-i)^n)$.
- **Sinc basis:** $s_n(t)=\mathrm{sinc}(t-n)$ with
  $\hat s_n(\omega)=\frac{1}{\sqrt{2\pi}}e^{-i\omega n}\,\mathbb 1_{|\omega|\le\pi}$.
- **Mixed matrix split:** $k=\lceil \texttt{TIME\_RATIO}\cdot d\rceil$ time rows,
  $m=d-k$ frequency rows (default ratio 0.5).
- **Condition number:** $\kappa_2=\sigma_{\max}/\sigma_{\min}$, full SVD for
  $d\le 2000$, iterative `svds` above; values reported as `inf` when
  $\sigma_{\min}$ underflows.
- **Streaming reconstruction:** Method A (time-only) is a diagonal solve;
  Method B (time+frequency) is solved matrix-free with preconditioned CG via FFTs,
  scaling to large $W$ without forming any matrix.

## Running

Open any notebook in Jupyter and run all cells top to bottom:

```
jupyter lab
```

Each notebook has a clearly marked **configuration cell near the top**
(intervals, ratios, dimensions, trial counts, sweep grids). Long sweeps checkpoint
their JSON results incrementally, so partial runs are not lost. Random experiments
use fixed seeds and report means over multiple trials (10 for Hermite/sinc random
sampling, 1000 for the sinc $d$-dependent experiment).

All PNG, JSON, and CSV artifacts are written below the notebook-specific directories
in `results/`; no generated result files are written to the repository root.