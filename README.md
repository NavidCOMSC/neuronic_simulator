# OpenMC Pin-Cell Demonstrator: Monte Carlo Neutronics + pandas

A small, reproducible demonstrator of continuous-energy Monte Carlo neutron transport with the
[OpenMC](https://openmc.org) Python API (v0.16). It models an infinite lattice of PWR UO₂ fuel pins
(Zircaloy-4 cladding, borated light water, reflective boundaries), solves the k-eigenvalue problem, and
analyses flux spectra, reaction rates and uncertainties with pandas.

## Repository structure
```
.
├── README.md
├── environment.yml
├── notebooks/
│   └── pincell_pandas_demo.ipynb
├── run_pincell/            # created at runtime (model.xml, statepoint.*.h5, summary.h5) - git-ignored
└── LICENSE
```

## Installation
1. Install Miniforge/Miniconda. Linux and macOS are supported natively; Windows needs WSL2 or Docker
   (`docker run openmc/openmc:latest`).
2. `conda env create -f environment.yml && conda activate openmc-pincell`
3. Download nuclear data: the ENDF/B-VIII.0 HDF5 library from https://openmc.org/data/, then unpack it.
4. `export OPENMC_CROSS_SECTIONS=/abs/path/to/cross_sections.xml`, or set `openmc.config['cross_sections']`
   in the notebook.

## Running
```bash
jupyter lab notebooks/pincell_pandas_demo.ipynb
```
Run all cells (~1–3 min on a laptop with 10k particles × 150 batches). Expected k_∞ ≈ 1.2–1.3 with σ ≈ 50–100 pcm;
exact values depend on the data library.

## Theory summary
**Transport equation (k-eigenvalue form):**

$$\boldsymbol\Omega\cdot\nabla\psi+\Sigma_t\psi=\int\!\!\int\Sigma_s(E'\to E,\boldsymbol\Omega'\to\boldsymbol\Omega)\psi\,d\Omega'dE'+\frac{\chi(E)}{4\pi k}\int\nu\Sigma_f\phi\,dE'$$

**Macroscopic cross section:** $\Sigma_x = N\sigma_x$. **Free-flight sampling:** $s=-\ln\xi/\Sigma_t$.

**Power iteration (method of successive generations):** $S^{(n+1)}=\frac{1}{k^{(n)}}\mathcal F S^{(n)}$, with
$k^{(n+1)}\approx \frac{\text{fission neutrons in gen } n+1}{\text{source neutrons in gen } n}$.

**Shannon entropy:** $H=-\sum_i S_i\log_2 S_i$, used to choose the number of inactive batches.

**Tally estimators:**
- analog: $R_x=\frac1W\sum_{A}w_i$
- collision: $R_x=\frac1W\sum_{C}w_i\frac{\Sigma_x}{\Sigma_t}$
- track-length: $R_x=\frac1W\sum_{T}w_i\ell_i\Sigma_x$

**Batch statistics:** $\bar x=\frac1n\sum x_i,\quad s_{\bar x}=\sqrt{\frac{\sum(x_i-\bar x)^2}{n(n-1)}}$,
relative error $R=s_{\bar x}/\bar x$, and $\mathrm{FOM}=1/(R^2T)$.

**Four-factor formula (infinite medium):** $k_\infty=\eta f p\varepsilon$.

## API notes (differences from the official pandas-dataframes notebook)
- Uses `openmc.Model` + `Model.run()` instead of separate `export_to_xml()` calls.
- Uses `IndependentSource(..., constraints={'fissionable': True})`. `Box(only_fissionable=True)` has been deprecated since 0.15.0.
- Uses `Universe.plot()` instead of `openmc.Plot`/`plot_inline`. `Plot` has been deprecated since 0.16.0 in favour of `SlicePlot`/`VoxelPlot`.
- Single pin with `openmc.model.RectangularPrism(..., boundary_type='reflective')` instead of a 17×17 lattice with mixed vacuum/reflective planes.
- Adds `c_H_in_H2O` thermal scattering and a He gap, following `openmc/examples/pincell/build_xml.py`.

## References
- OpenMC documentation (stable): https://docs.openmc.org/en/stable/. See the Quick Install Guide, Data Configuration, Theory: Tallies and Eigenvalue Calculations, and the Python API (`openmc.Tally`, `openmc.model.Model`, `openmc.IndependentSource`).
- OpenMC v0.16.0 release notes: https://github.com/openmc-dev/openmc/releases/tag/v0.16.0
- Example notebooks: https://github.com/openmc-dev/openmc-notebooks (pandas-dataframes.ipynb, pincell.ipynb); script example: `openmc/examples/pincell/build_xml.py`
- Nuclear data: https://openmc.org/data/ ; scripts: https://github.com/openmc-dev/data
- P. K. Romano et al., "OpenMC: A state-of-the-art Monte Carlo code for research and development," *Ann. Nucl. Energy* 82, 90–97 (2015), doi:10.1016/j.anucene.2014.07.048
- P. K. Romano, B. Forget, "The OpenMC Monte Carlo particle transport code," *Ann. Nucl. Energy* 51, 274–281 (2013)
- J. Lieberoth, "A Monte Carlo technique to solve the static eigenvalue problem of the Boltzmann transport equation," *Nukleonik* 11, 213–219 (1968)
- F. B. Brown, "On the use of Shannon entropy of the fission distribution for assessing convergence of Monte Carlo criticality calculations," LA-UR-06-3737 (2006)
- J. J. Duderstadt, L. J. Hamilton, *Nuclear Reactor Analysis*, Wiley (1976)
- D. L. Brown et al., "ENDF/B-VIII.0," *Nucl. Data Sheets* 148, 1–142 (2018)
