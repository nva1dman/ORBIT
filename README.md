# ORBIT
ORBIT - Orbital Reconstruction via Bayesian Iterative Technique

This Python 3 script performs a complete Keplerian radial-velocity fit with an MCMC sampler.

It needs: numpy, pandas, matplotlib, scipy, emcee, corner.

Input: a whitespace-separated text table containing three columns
HJD (heliocentric Julian day), RV (km s-1), and sigma (km s-1).
The path is set in the variable data_file.

Execution flow:

Settings block – choose data_file, decide whether to fit eccentricity (fit_ecc), give initial guesses P0 K0 gamma0 and their allowed ranges, set how many trial periods (n_periods) and MCMC steps (nsteps).

Data loading and cleaning – reads the table with pandas, removes non-finite rows, keeps only measurements with positive sigma, rejects 5-sigma outliers.

Grid search for the period – scans n_periods trial values from P0-dP to P0+dP, fits a simple sine plus offset at each trial, computes chi2, and keeps the period with the minimum chi2. The result is stored in best_P.

Orbital model definition – two functions:
true_anomaly solves Kepler’s equation with a Newton method or ten fixed-point iterations if Newton fails;
rv_model uses true_anomaly to compute the line-of-sight velocity for given P K T0 gamma and eccentricity parametrised by x and y where e = sqrt(x^2 + y^2) and omega = atan2(y,x).

Priors and likelihood – log_prior imposes Gaussian priors on P K gamma and hard limits on P K T0 and eccentricity; log_likelihood is chi2 assuming Gaussian errors; log_posterior is their sum.

MCMC sampling – builds an emcee.EnsembleSampler with nwalkers = 4*ndim+10, initialises walkers around theta0, runs for nsteps. Afterward it discards the first 20 percent as burn-in, thins by 10, flattens the chain, finds the sample with the highest posterior, and prints best-fit values with one-sigma uncertainties.

Fit quality – computes chi2, degrees of freedom (dof = N – k), and reduced chi2.

Mass function – converts sampled P and K to SI units, computes f(m) = (K^3 P)/(2 pi G) (1-e^2)^(-1.5) / M_sun for every posterior sample, prints median and standard deviation.

Plotting – folds times by the best-fit period, draws data with error bars and the best curve, plots normalised residuals underneath, styles ticks and fonts, saves the figure as rv_fit_HR8523_secondary.pdf. An optional corner plot of the posterior can be produced by uncommenting two lines in block 6.

Outputs: console text with best parameters plus/minus one sigma, chi2 and reduced chi2, mass function and its error; a PDF file with the phase-folded fit; optionally a PNG corner plot.

To fit a circular orbit set fit_ecc = False.
To use a different data set change data_file.
For deeper convergence increase nsteps or nwalkers.

No other files are required; all physical constants (G, day2s, solar mass) are defined inside the script.
