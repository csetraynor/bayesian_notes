
---
output: html_document
editor_options: 
  chunk_output_type: console
---
# Model Checking

## Why check models?

-   In theory, a Bayesian model should include all relevant substantive knowledge and subsume all possible theories.
-   In practice, it won't. We need to check how the model fits data.

The question is not whether a model is "true"; it isn't [@Box1976a].
The question is whether it is good enough for the purposes of the analysis.
The problem is how we can specify "good enough" criteria, and how we can check those criteria.

See @GelmanMengStern1996a, @Gelman2007a, @Gelman2009a, @BDA3 [Ch. 6],
@GelmanShalizi2012a, @Kruschke2013b, @GelmanShalizi2012b, @Gelman2014a for a
more discussion of the motivation and use of posterior predictive checks.

## Posterior Predictive Checks

One method evaluate the fit of a model is to use **posterior predictive checks**

-   Fit the model to the data to get the posterior distribution of the parameters: $p(\theta | D)$
-   Simulate data from the fitted model: $p(\tilde{D} | \theta, D)$
-   Compare the simulated data (or a statistic thereof) to the observed data and a statistic thereof. The comparison between data simulated from the model can be formal or visual.

Within a Stan function, this is done in the `generated quantities` block using a `_rng` distribution functions.

The package **[bayesplot](https://cran.r-project.org/package=bayesplot)** includes multiple functions for posterior predictive checks; see the help for [PPC-overview](https://www.rdocumentation.org/packages/bayesplot//topics/PPC-overview) for a summary of these functions.

### Bayesian p-values

A **posterior predictive p-value** is a the tail posterior probability for a statistic generated from the model compared to the statistic observed in the data.
Let $y = (y_1, \dots, y_n)$ be the observed data.
Suppose the model has been fit and there is a set of simulation $\theta^(s)$, $s = 1, \dots, n_sims$.
In  replicated dataset, $y^{rep(s)$, has been generated from the predictive distribution
of the data, $p(y^{(rep)} | \theta = \theta^{(s)}$.
Then the ensemble of simulated datasets, $(y^{rep(s)}, \dots, y^{rep(nsims)})$, is a sample from the posterior predictive
distribution, $p(y^{(rep)} | y)$

The model can be tested by means of discrepancy statistics, which are some function of the data and parameters, $T(y, \theta)$.
If $\theta$ was known, then compare discrepancy by $T(y^{(rep)}, \theta)$.
The statistical significance is $p = \Pr(T(y^{(rep)}, \theta) > T(y, \theta) | y, \theta)$.
If $\theta$ is unknown, then average over the posterior distribution of $\theta$,
$$
\begin{aligned}[t]
p &= \Pr(T(y^{(rep)}, \theta) > T(y, \theta) | y) \\
&= \int Pr(T(y^{(rep)}, \theta) > T(y, \theta) | y, \theta) p(\theta | y) d\,\theta ,
\end{aligned}
$$
which is easily estimated from the MCMC samples as,
$$
p = \frac{1}{n_{sims}}\sum_{s = 1}^{n_{sims}} 1( T(y^{rep(s)}, \theta(s)) > T(y, \theta(s)))
$$

### Test quantities

The definition of a posterior p-value does not specify a particular test-statistic, $T$, to use.

The best advice is that $T$ depends on the application.

-   @BDA3 [p. 146] Speed of light example uses the 90% interval (61st and 6th order statistics).

-   @BDA3 [p. 147] binomial trial example uses the number of switches (0 to 1, or 1 to 0)
    in order to test independence.

-   @BDA3 [p. 148] hierarchical model for adolescent smoking uses.

    -   percent of adolescents in the sample who never smoked
    -   percentage in the sample who smoked in all waves
    -   percentage of "incident smoker": adolescents who began the study and non-smokers and ended as smokers.

### p-values vs. u-values

A posterior predictive p-value is different than a classical p-value.

-   Posterior predictive $p$-value

    -   distributed uniform if the **model is true**.

-   Classical $p$-value

    -   distributed uniform if the **null hypothesis** ($H_0$) is true.

A $u$-value* is any function of the data that has a $U(0, 1)$ sampling distribution [@BDA3, p. 151]

-   a $u$-value can be averaged over $\theta$, but it is not Bayesian, and is not a probability distribution
-   posterior p-value: probability statement, conditional on model and data, about future observations

### Marginal predictive checks

Compare statistics for each observation.

*Conditional Predictive Ordinate (CPO)*:
The CPO [@Gelfand1995a] is the leave-one-out cross-validation predictive density:
$$
p(y_i | y_{-i}) = \int p(y_i | \theta) p(\theta | y_{-i}) d\,\theta
$$
The pointwise predicted LOO probabilities can be calculated using PSIS-LOO or WAIC in the **loo** package.

<!-- The sum of the logged CPOs can be an estimator of the log marginal likelihood and is called the log pseudo marginal likelihood. The ratio of PsMLs can be used as a surrogate for a Bayes Factor (pseudo Bayes Factor) (LaplaceDemon p. 20) -->

**Predictive Concordance and Predictive Quantiles** @Gelfand1995a classifies any $y_i$ that is outside the central 95% predictive posterior of $y^{rep}_i$ is an outlier.
Let the *predictive quantile* ($PQ_i$) be
$$
PQ_i = p(y_i^{(rep)} > y_i) .
$$
Then the *predictive concordance* be the proportion of $y_i$ that are not
outliers. @Gelfand1995a argues that the predictive concordance should match 95%.
In other words that the posterior predictive distribution should have the correct coverage.

### Outliers

Can be identified by the inverse-CPO.

-   larger than 40 are possible outliers, and those higher than 70 are extreme values [@Ntzoufras2009a, p. 376].
-   @Congdon2014a scales CPO by dividing each by its individual max and considers observations with scaled CPO under 0.01 as outliers.

### Graphical Posterior Predictive Checks

> Visualization can surprise you, but it doesn’t scale well. Modeling scales well, but it can’t surprise you. -- [paraphrase of Hadley Wickham](https://www.johndcook.com/blog/2013/02/07/visualization-modeling-and-surprises/)

Instead of calculating posterior probabilities, plot simulated data and observed data and visually compare them. See @BDA3 [p. 154].

-   plot simulated data and real data [@BDA3, p. 154]. This is similar to
    ideas in @WickhamCookHofmannEtAl2010a.

-   plot summary statistics or inferences

-   residual plots

    -   Bayesian residuals have a distribution
        $r_i^{(s)} = y_i - \E(y_i \theta^{s})$

    -   Bayesian residual graph plots single realization of the residuals,
        or a summary of their posterior distributions

    -   binned plots are best for discrete data [@BDA3, p. 157]

<!--
## Average Predictive Comparisons

From @GelmanHill [Ch 21.4]
Let $u$ be the input of interest, and $v$ be all other inputs, so that $x = (u, v)$.
$$
b_u(u^{(lo)}, u^{(hi)}, v, \theta) = \frac{E(y | u^{(hi)}, v, \theta) - E(y | u^{(lo)}, v, \theta)}{u^{(hi)} - u^{(lo)}}
$$
the the average predictive difference per unit change in $u$ is,
$$
B_{u}(u^{(lo)}, u^{(hi)}) = \frac{1}{n} \sum_{i = 1}^n b_u(u^{(lo)}, u^{(hi)}, v_i, \theta) .
$$
This can be adjusted to use observed (weighted) differences of $u$ for each point.
See the Gelman paper on it.
-->

## References

See @GelmanShalizi2012a, @GelmanShalizi2012b, @Kruschke2013b.
