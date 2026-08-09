# GPS-Style Localization via Nonlinear Least Squares

Estimating an unknown position from noisy distance measurements — comparing five optimization methods on planar trilateration and its GPS extension (unknown clock bias), with a full noise/geometry robustness study.

📄 **[Read the full report](report.pdf)** for the complete mathematical derivations and analysis.

> Group project — built with [Elhabib Touisse](https://github.com/elhabib-touisse) and Hervé Nikiema, supervised by Louis Thiry (Sorbonne Université, M1 Mathematics and Applications — Numerical Optimization and Data Science).

## Problem

Given `m` fixed stations at known positions and noisy distance measurements to an unknown point `x`, recover `x`. In the noiseless case this is a system of quadratic equations; in practice, measurement noise turns it into a nonlinear least-squares problem:

```
min_x  Σ (‖x - aᵢ‖ - dᵢ)²
```

A second, harder variant models GPS-style pseudoranges, where each measurement is also offset by an unknown clock bias `b`:

```
ρᵢ = ‖x - aᵢ‖ + b + εᵢ
```

requiring joint estimation of both `x` and `b`.

## Methods implemented & compared

| Method | Summary |
|---|---|
| **Gauss-Newton** | Linearizes the residuals via their Jacobian (computed with PyTorch autodiff) and solves the normal equations at each step. |
| **Levenberg-Marquardt** | Damped Gauss-Newton (`JᵀJ + λI`), interpolating between Gauss-Newton and gradient descent depending on λ. |
| **Gradient descent — fixed step** | Direct minimization of the cost function with a constant learning rate. |
| **Gradient descent — optimal step** | Step size chosen at each iteration by solving for the minimizer along the descent direction. |
| **Projected gradient (SDP relaxation)** | Reformulates the problem via `X = xxᵀ`, relaxes to `X ⪰ xxᵀ` (a convex PSD constraint via Schur complement), and projects onto the PSD cone after each gradient step. |

## Key Results

- **Levenberg-Marquardt is the most reliable method overall**, and by far the most robust to the initial point in the GPS case — it recovers the correct position and clock bias even from initial guesses far from the true solution.
- **Gauss-Newton** is fast and accurate when initialized close to the solution, but can diverge to nonsensical values on bad initial guesses, especially with the extra clock-bias parameter.
- **Fixed-step gradient descent** is simple and surprisingly robust — more so than the "smarter" optimal-step variant, which suffered from numerical instability in the step-search and occasionally produced very poor estimates.
- **The SDP-projected gradient** performs well on some trilateration geometries (e.g. recovering `(-0.99, 0.01)` for a true position of `(-1, 0)`) but fails to generalize to the GPS/clock-bias case.
- **Station geometry matters as much as the algorithm.** Well-distributed stations make the problem well-conditioned for every method; aligned or clustered stations degrade all of them, with the SDP approach notably more resilient to bad geometry than plain gradient descent.
- Noise robustness was tested across 20 noise levels (σ from 0.01 to 2) on three station layouts (distributed, aligned, clustered), consistently confirming these patterns.

## Tech Stack

- **Language:** Python
- **Differentiable programming:** PyTorch (`torch.autograd.functional.jacobian`) for exact Jacobians in Gauss-Newton / Levenberg-Marquardt
- **Numerical tools:** NumPy, SciPy (`fsolve` for optimal step-size search)
- **Visualization:** Matplotlib (convergence trajectories over cost-function contour/surface plots)

## Repository Contents

- `notebook.ipynb` — full implementation: all five methods, initial-point sensitivity study, noise/geometry robustness study, GPS extension
- `report.pdf` — complete written report (modeling, method-by-method analysis, comparative results, conclusion)
- `requirements.txt` — dependencies to reproduce the analysis

## Getting Started

```bash
git clone https://github.com/<your-username>/gps-localization-optimization.git
cd gps-localization-optimization
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

## Academic Context

This project was submitted as coursework for the "Optimisation numérique et science des données" module, M1 Mathematics and Applications, Sorbonne Université, supervised by Louis Thiry.

## Authors

**Valentin Cherin** — [GitHub](https://github.com/votre-profil)
**Elhabib Touisse** — [GitHub](https://github.com/elhabib-touisse)
**Hervé Nikiema**
