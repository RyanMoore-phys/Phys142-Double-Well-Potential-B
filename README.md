# Double Well Potential with Path Integral MCMC

PHYS 142/242 (UCSD) final project, Problem 1.

This repository samples imaginary-time paths of a quantum particle in the double well

$$V(x) = \alpha x^4 - 2x^2 + \frac{1}{\alpha}, \qquad \alpha = 0.4,\quad m = \hbar = 1,$$

with Markov chain Monte Carlo. From the paths alone it recovers the ground-state energy $E_0$, the tunneling gap $\Delta E = E_1 - E_0$, and the position distribution. All results are checked against the eigenstates of the discretized Hamiltonian, the method used in Assignment 2.

All of the work is in a single notebook, [`double_well_mcmc.ipynb`](double_well_mcmc.ipynb), saved with its outputs and figures so it can be read on GitHub without running it.

## Contents

| File | Purpose |
|---|---|
| `double_well_mcmc.ipynb` | Sampler, error analysis, and Problems A, B, and C with figures and discussion |
| `requirements.txt` | Python packages needed to run the notebook |
| `README.md` | This file |

## How to run

Requires Python 3.10 or newer.

```bash
git clone <this repository's URL>
cd double-well-mcmc
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook double_well_mcmc.ipynb
```

Then run all cells in order (Kernel → Restart & Run All). To run the whole notebook from the command line and save the outputs in place:

```bash
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 double_well_mcmc.ipynb
```

The full run takes about 3–4 minutes on an 8-core laptop and peaks at about 2 GB of memory. Each measurement runs 8 independent Markov chains in parallel threads (`N_CHAINS` in the first code cell). Every chain has its own fixed seed, so the numbers are reproducible on the same machine and library versions; the first call also spends a few seconds compiling the Numba kernels.

## Method

* **Benchmark.** The Hamiltonian is discretized with second-order finite differences on $[-6, 6]$ and diagonalized (`scipy.linalg.eigh_tridiagonal`). This reproduces the Assignment 2 reference values of $E_0$ and $E_1$ to about $10^{-6}$.
* **Path sampling.** Closed paths of $N = \beta/\delta\tau$ slices are weighted by $e^{-S_E}$, using the primitive discretized Euclidean action. Each sweep tries a Metropolis update of every slice with a uniform proposal of half-width $h = 2.5\sqrt{\delta\tau}$, which gives about 43% acceptance. An optional whole-path shift $x_i \to x_i + d$ can follow each sweep. The loops are compiled with Numba.
* **Energy.** The virial estimator $\langle V + \tfrac12 xV'\rangle_\beta$ is measured every sweep. $E_0$ is taken at $\beta = 40$, from five values of $\delta\tau$ extrapolated to $\delta\tau \to 0$.
* **Tunneling.** The correlation function $C(\tau) = \langle x(0)x(\tau)\rangle_\beta$ is averaged over all pairs of slices with an FFT. It is fit to the two-state form $|x_{01}|^2\cosh[\Delta E(\tau-\beta/2)]/\cosh(\beta\Delta E/2)$ at $\beta = 20$.
* **Errors.** Errors follow Lab 12. Each chain is cut into blocks much longer than the integrated autocorrelation time, giving 64 block means per measurement. Block means give the errors on averages, and a block jackknife gives the errors on the fitted $\Delta E$ and $|x_{01}|$. Integrated autocorrelation times are reported for the energy and the path center $\bar x$.
* **Finite temperature.** $\beta = 9, 4, 2, 1, 0.5$ are compared with the thermal density $\rho_\beta(x)$ and energy $\langle \hat H\rangle_\beta$ built from 40 eigenstates.

## Results

| Quantity | Path integral MCMC | Eigenstates |
|---|---|---|
| $E_0$ ($\beta = 40$, $\delta\tau \to 0$) | $1.2361 \pm 0.0021$ | $1.2345$ |
| $\Delta E$ (local moves) | $0.111 \pm 0.005$ | $0.1124$ |
| $\Delta E$ (local + shift moves) | $0.106 \pm 0.003$ | $0.1124$ |
| $\lvert\langle\phi_0\lvert x\rvert\phi_1\rangle\rvert$ | $1.349 \pm 0.004$ | $1.3512$ |
| $\langle \hat H\rangle_\beta$ for $\beta = 0.5$ to $9$ | agrees within $1.4\sigma$ at every $\beta$ | |

Main findings:

* $\beta$ must be large compared with $1/\Delta E \approx 8.9$, not just $1/(E_2 - E_0) \approx 0.56$. Otherwise the estimator returns roughly $(E_0+E_1)/2$, the energy of a particle localized in one well.
* The primitive action gives energies that fall linearly in $\delta\tau^2$, by about 5% at $\delta\tau = 0.25$.
* With local moves alone, the chain changes wells only once every $\sim 10^5$ sweeps at $\beta = 20$, and $\tau_{\text{int}}(\bar x) \approx 4.5\times10^4$ sweeps.
* The whole-path shift move cuts $\tau_{\text{int}}(\bar x)$ to about 130 sweeps at $\beta = 20$, because most paths at that $\beta$ contain no kinks. At $\beta = 40$ it helps much less, because most paths contain kinks.
* As $\beta \to 0$, the sampled density approaches the classical Boltzmann distribution $e^{-\beta V(x)}$.

## References

1. K. Banerjee and S. P. Bhatnagar, "Two-well oscillator", Phys. Rev. D **18**, 4767 (1978).
2. S. Mittal, M. J. Westbroek, P. R. King, and D. D. Vvedensky, "Path integral Monte Carlo method for the quantum anharmonic oscillator", Eur. J. Phys. **41**, 055401 (2020).
3. M. Creutz and B. Freedman, "A statistical approach to quantum mechanics", Ann. Phys. **132**, 427 (1981).
4. H. Flyvbjerg and H. G. Petersen, "Error estimates on averages of correlated data", J. Chem. Phys. **91**, 461 (1989).
