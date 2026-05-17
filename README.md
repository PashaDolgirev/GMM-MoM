# GMM-MoM

Precomputed Gaussian-mixture-model (GMM) approximations of the equilibrium Bose–Einstein
and Fermi–Dirac distribution functions, together with the MATLAB optimization scripts
that produced them.

These fits are the ingredient that lets the method-of-moments (MoM) approach of the
companion paper evaluate Boltzmann collision integrals in closed form: once
the distribution function is written as a sum of Gaussians, every collision integral
reduces to elementary Gaussian integrals.

## Reference

> P. E. Dolgirev, K. Seetharam, M. Kanász-Nagy, C. Robens, Z. Z. Yan, M. Zwierlein,
> E. Demler, *Accelerating analysis of Boltzmann equations using Gaussian mixture
> models: Application to quantum Bose–Fermi mixtures*,
> Phys. Rev. Research **6**, 033017 (2024).
> [DOI](https://doi.org/10.1103/PhysRevResearch.6.033017) ·
> [arXiv:2304.09911](https://arxiv.org/abs/2304.09911)

The GMM part of the paper (the only part this repository covers) is described in
Sec. III. 

## What is in here

The repo is split into two self-contained MATLAB sub-projects, one per regime
of the fugacity. Each sub-project bundles (i) the precomputed fit tables, (ii) the
optimization routines that generated them, and (iii) a small analysis script
showing how to load and evaluate a fit.

| Folder | Regime | Gaussian parameters |
| --- | --- | --- |
| `Library_fits/` | dilute bosons (`z_B < 1`) and dilute fermions (`z_F < 1`) | **real** amplitudes and widths |
| `Library_fits_complex/` | dense fermions (`z_F > 1`) | **complex** amplitudes and widths (kept in conjugate pairs) |

### `Library_fits/` — real GMM, dilute regime

```
Library_fits/
├── Dilute_bosons/         precomputed fit vectors for z_B ∈ {0.3, 0.5, 0.9, 0.99}
├── Dilute_fermions/       precomputed fit vectors for z_F ∈ {0.3, 0.5, 1, 2}
├── Functions/             optimization helpers + the parametrization
│   └── poly_Gauss_approx.m   ← how a fit vector maps to f(x); also returns the Jacobian
├── bosons_analysis.m      minimal example: load a table, evaluate, plot
├── fermions_analysis.m    same, for fermions
├── dilute_bosons_full.m   re-derives the boson tables from scratch (lsqcurvefit)
└── dilute_fermions_full.m re-derives the fermion tables from scratch
```

Fit-vector layout for `M` Gaussians: `vec = [a_1 … a_M; σ_1 … σ_M]` (length `2M`).
The fit is
```
f(x) ≈ Σ_{s=1..M}  a_s · exp( − x² / (2 σ_s²) )
```

Each table is stored as `vec_<species>_<z>_<M>.txt`, alongside an
`Error_list_<z>.txt` that records the max-norm fit error as `M` grows.
A separate Taylor-seeded version of each table is kept as `vec_<species>_TS_…`.

### `Library_fits_complex/` — complex GMM, dense fermion regime

```
Library_fits_complex/
├── DATA_1000/             tables for z_F up to 10^3
├── DATA_10000/            tables for z_F up to 10^4
├── DATA_100000/           tables for z_F up to 10^5
├── Functions_fit/         optimization helpers + the parametrization
│   └── poly_Gauss_approx.m   ← complex version (uses 2·Re of complex Gaussians)
├── analysis.m             minimal example: load a table, evaluate, plot
├── Single_run.m           re-derives a table from scratch for a given z_F
└── Post_processing.m      diagnostics + Gaussian-reduction experiments
```

Fit-vector layout for `M` complex Gaussians is `vec` of length `4M`, packed as
`[Re(a); Im(a); Re(σ); Im(σ)]`; `convert_from_vec.m` / `convert_to_vec.m` go
back and forth. The fit is
```
f(x) ≈ Σ_{s=1..M}  2 · Re [ a_s · exp( − x² / (2 σ_s²) ) ]
```
i.e. the complex Gaussians are kept in conjugate pairs so `f` stays real.

Each table is stored as `vec_best_<M>.txt`; the corresponding fit error is in
`error_best_<M>.txt`, and the per-run error trajectory in `Error_list.txt`.

## Using a precomputed fit

Both sub-projects follow the same pattern: load the table, evaluate via
`poly_Gauss_approx`, and (optionally) rescale to a smaller fugacity using a
closed-form transformation of the amplitudes — see `bosons_analysis.m`,
`fermions_analysis.m`, and `analysis.m` for working examples. These three
scripts are the recommended entry points if you only want to *use* the
library, not regenerate it.

## Regenerating a fit

To rederive the tables, run `dilute_bosons_full.m` / `dilute_fermions_full.m`
(real case) or `Single_run.m` (complex case). All three are MATLAB scripts
that rely on `lsqcurvefit` from the Optimization Toolbox and write their
output back into the corresponding `DATA_*` / `Dilute_*` folder, skipping
any file that already exists.

## Requirements

MATLAB with the Optimization Toolbox (`lsqcurvefit`, Levenberg–Marquardt).
No other dependencies.

## License

MIT — see `LICENSE`.
