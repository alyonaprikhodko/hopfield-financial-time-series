# Hopfield Associative Memory for Financial Time Series

This repository contains the implementation and experimental results for the research project:

**Application of the Hopfield Network Associative Memory for the Reconstruction and Forecasting of Stock Market Time Series**

The study investigates whether recurring local structures in financial time series can be stored and exploited using associative memory. The main focus is the classical binary Hopfield network, with an additional asymmetric temporal associative model for one-step directional forecasting.

---

## Research Objective

The project evaluates whether historical financial patterns can be used as associative memories for:

1. recalling corrupted patterns already stored in memory;
2. reconstructing corrupted previously unseen financial windows;
3. retrieving historical analogues for one-step return-direction forecasting;
4. investigating how the results depend on temporal window length.

The central hypothesis is that recurring local structures in financial time series may allow a Hopfield network trained on historical patterns to improve corrupted unseen fragments and potentially extract information useful for forecasting the sign of the next return.

---

## Data

The experiments use daily historical data for:

- **SPY** — SPDR S&P 500 ETF Trust;
- **AAPL** — Apple Inc.;
- **TSLA** — Tesla Inc.

The investigated period is **January 1, 2020 – December 31, 2025**.

Historical market data are obtained through the Yahoo Finance interface.

Logarithmic returns are calculated as:

`r_t = ln(P_t / P_{t-1})`

where `P_t` is the adjusted closing price at time `t`.

For the classical Hopfield network, returns are converted into bipolar states:

- `x_t = +1` if `r_t >= 0`;
- `x_t = -1` if `r_t < 0`.

A temporal pattern of length `L` is represented as:

`X_t = [x_{t-L+1}, ..., x_t]`

The investigated temporal window lengths are:

`L = {10, 30, 50, 70, 100, 200}`

In addition to the sign-only representation, experiments include a binary volatility channel based on the median absolute return estimated from the training data.

---

## Experimental Pipeline

### 1. Data Exploration and Preprocessing

The notebook calculates descriptive statistics for the financial series, including:

- mean logarithmic return;
- standard deviation;
- positive-return ratio;
- skewness;
- kurtosis;
- Hurst exponent.

The Hurst exponent is treated as a descriptive indicator of scale-dependent behaviour rather than direct evidence of return predictability.

### 2. Classical Hopfield Network

The classical Hopfield network stores bipolar temporal patterns using Hebbian learning:

`w_ij = sum_mu xi_i^mu * xi_j^mu`, for `i != j`

with zero diagonal weights.

The memory load is defined as:

`alpha = P / N`

where:

- `P` is the number of stored patterns;
- `N` is the number of neurons.

The implementation uses asynchronous neuron updates and monitors the Hopfield energy during validation.

### 3. Implementation Validation

Before evaluating financial generalization, the implementation is tested on synthetic bipolar patterns.

The validation checks:

- symmetry of the weight matrix;
- zero diagonal weights;
- recovery of a corrupted stored pattern;
- non-increasing Hopfield energy.

This separates implementation correctness from limitations observed later on real financial data.

### 4. Stored-Pattern Recall

Financial patterns already stored in memory are artificially corrupted and reconstructed by the network.

The experiment evaluates how recall depends on:

- memory load;
- temporal window length;
- corruption level;
- financial asset.

Stored-pattern recall acts as a control experiment for the classical associative-memory mechanism. It verifies that the network can perform the task for which the classical Hopfield architecture was designed before reconstruction of previously unseen financial states is evaluated.

### 5. Reconstruction of Previously Unseen Patterns

The financial series are split chronologically into:

- **80% training data**;
- **20% test data**.

The split is performed **before constructing sliding windows** to prevent overlapping train/test windows across the temporal boundary.

Only training windows can be stored in Hopfield memory. Previously unseen test windows are artificially corrupted and passed through the trained network.

The main reconstruction metric is:

`Delta A = A(X_restored, X) - A(X_noisy, X)`

where `X` is the original clean unseen pattern.

Therefore:

- `Delta A > 0` — Hopfield dynamics corrected the corrupted pattern;
- `Delta A = 0` — reconstruction produced no net improvement;
- `Delta A < 0` — the network introduced more errors than it corrected.

Hamming distance, exact recovery, convergence, and the number of update sweeps are also recorded as diagnostic measures.

Two memory-selection approaches are investigated:

- **uniform historical sampling**;
- **prototype-based selection using k-means clustering**.

Prototype selection retains actual historical patterns closest to the cluster centres rather than storing the continuous cluster centres themselves.

The main noisy-unseen experiment uses multiple test patterns distributed across each test interval and repeated independent corruptions. Paired corruptions are used when comparing memory-selection strategies so that the same damaged inputs are evaluated by both methods.

### 6. Historical-Analogue Forecasting

The forecasting task predicts the sign of the next daily logarithmic return.

For a historical context:

`X_t = [x_{t-L+1}, ..., x_t]`

the target is:

`y_{t+1} = x_{t+1}`

Historical training contexts are paired with their observed next-step continuations.

The forecasting experiments compare associative retrieval with several baselines:

- majority class;
- persistence;
- first-order Markov model;
- nearest historical analogue;
- k-nearest neighbours;
- logistic regression.

The primary evaluation metric is **balanced accuracy**.

The k-nearest-neighbour and logistic-regression baselines operate on continuous return windows rather than the Hopfield-specific bipolar representation. Their preprocessing parameters are estimated using training data only.

For classical Hopfield forecasting, two variants are compared.

**Direct retrieval:** the nearest historical pattern is retrieved directly from the limited Hopfield memory.

**Hopfield-assisted retrieval:** the current context is first transformed by Hopfield dynamics, after which the nearest stored historical pattern is retrieved.

In both cases, the actual historical continuation associated with the retrieved pattern is used as the forecast.

This comparison isolates the contribution of Hopfield dynamics from the effect of restricting retrieval to the same historical memory.

### 7. Asymmetric Temporal Associative Model

The classical Hopfield network stores static attractors and therefore does not directly represent temporal direction.

An additional asymmetric temporal model is evaluated using transitions:

`X_t -> X_{t+1}`

The temporal association matrix is:

`W_temp = sum_t X_{t+1} X_t^T`

A single matrix update is used to estimate the next state, and the final element of this state is interpreted as the predicted next return sign.

This experiment is treated separately from the classical symmetric Hopfield network because the asymmetric model does not have the same classical Hopfield energy interpretation.

### 8. Multi-Scale Analysis

Predictions from three representative temporal scales are aligned:

`L = {10, 50, 200}`

The experiment tests whether agreement between short-, medium-, and long-window temporal models identifies more reliable forecasts.

This analysis investigates scale-dependent behaviour but is **not** interpreted as a formal test of financial multifractality.

---

## Main Results

### Stored-Pattern Recall

The classical Hopfield network successfully recalls stored financial patterns at low memory loads.

Recall deteriorates as the number of stored patterns increases, demonstrating increasing interference between correlated financial memories.

This confirms that the implementation performs the associative-memory task for which the classical Hopfield model was designed.

### Unseen-Pattern Reconstruction

The successful recall of stored memories does **not** generalize to previously unseen financial windows.

Clean unseen test windows are generally not stable states of the trained network. When artificial corruption is introduced, the change in reconstruction accuracy is predominantly negative:

`Delta A < 0`

Hopfield dynamics frequently moves an unseen state toward patterns supported by historical memory, but this movement does not generally recover the original clean unseen financial state.

The main reconstruction hypothesis is therefore **not supported** for the classical binary Hopfield model in the investigated setting.

### Coverage–Interference Trade-Off

The experiments reveal a central limitation of classical Hopfield memory on financial windows:

- small memories preserve stable attractors but cover only a limited part of historical pattern diversity;
- larger memories improve historical coverage but increase interference between stored patterns.

Prototype-based memory selection improves historical coverage in some configurations but does not eliminate this trade-off or produce systematic successful reconstruction of unseen patterns.

### Forecasting

Historical similarity does not provide a stable cross-asset forecasting advantage.

Classical Hopfield processing also does not systematically improve direct historical retrieval from the same limited memory.

The conventional forecasting baselines show heterogeneous results across assets and temporal windows. Their performance is generally close to the balanced-accuracy reference level, although individual configurations perform better or worse.

The results indicate that similarity between historical contexts does not necessarily imply similarity between their next-day return directions.

### Temporal Association

The asymmetric temporal associative model produces stronger results in several individual configurations.

Local improvements are observed for **SPY** and **TSLA**, while **AAPL** remains below the balanced-accuracy reference level across the investigated window lengths.

However, the effect is asset- and scale-dependent and does not establish a universally predictive temporal associative model.

### Temporal Scale

Changing the temporal window affects both reconstruction and forecasting behaviour, but no single window length is consistently superior across all assets.

Agreement between predictions at `L = 10`, `L = 50`, and `L = 200` also does not improve forecasting reliability.

The results therefore support scale-dependent model behaviour but do not demonstrate a universally useful forecasting scale or constitute evidence of financial multifractality.

---

## Main Conclusion

The experiments demonstrate a clear distinction between **associative recall** and **generalization**.

The classical Hopfield network is effective at storing and recovering financial patterns that are already present in memory. However, it does not reliably reconstruct previously unseen financial fragments through association with historical states.

The principal limitation is a trade-off between historical coverage and interference among stored memories. Small memories remain stable but represent too little of the historical pattern space, while larger memories improve coverage at the cost of attractor degradation.

For one-step directional forecasting, classical Hopfield retrieval does not provide a stable advantage over direct historical retrieval or conventional baselines.

The asymmetric temporal experiment shows that explicitly encoding temporal transitions can change the behaviour of the associative model and produce local improvements, but these effects are not consistent across assets and temporal scales.

Overall, the project establishes a classical associative-memory baseline for financial time series and identifies the limitations that more expressive Hopfield architectures would need to address.

---

## Repository Structure

- `README.md` — project overview, methodology, and main conclusions;
- `notebook/hopfield_financial_time_series.ipynb` — complete experimental pipeline;
- `paper/hopfield_financial_time_series.pdf` — research preprint;
- `figures/` — figures generated from the experimental results.

The main notebook contains the complete experimental pipeline, including data preprocessing, Hopfield implementation, reconstruction experiments, forecasting experiments, diagnostic analyses, and visualizations.

---

## Reproducibility

The experiments can be reproduced by running the notebook from top to bottom.

The notebook downloads the required historical market data and performs the complete preprocessing and experimental pipeline.

The main Python dependencies are:

- NumPy;
- pandas;
- Matplotlib;
- scikit-learn;
- yfinance.

A chronological train/test split is used throughout the unseen reconstruction and forecasting experiments to avoid leakage from future observations into training data.

Randomized procedures use fixed random seeds where required by the experimental design.

---

## Limitations

The study is intentionally limited in scope:

- only three US financial assets are investigated;
- daily observations from a single six-year period are used;
- evaluation uses a single chronological holdout rather than full walk-forward validation;
- the classical Hopfield representation strongly compresses financial information;
- the asymmetric temporal experiment is a simple associative extension rather than a modern forecasting architecture.

These limitations restrict the generality of the empirical conclusions but also define clear directions for further investigation.

---

## Future Work

Several extensions follow directly from the experimental results:

1. investigate continuous-valued and modern Hopfield networks to reduce information loss caused by bipolar encoding;
2. evaluate the models using walk-forward validation across multiple market regimes;
3. extend the analysis to additional assets and sampling frequencies;
4. investigate more sophisticated financial motif and representative-memory selection;
5. jointly model short-, medium-, and long-scale representations rather than comparing independently trained window lengths;
6. compare the classical associative-memory baseline with modern Hopfield architectures designed for time-series modelling.

---

## Research Paper

The accompanying research preprint provides the complete methodology, experimental results, interpretation, limitations, and discussion.

**Application of the Hopfield Network Associative Memory for the Reconstruction and Forecasting of Stock Market Time Series**

**Author:** Alena Prikhodko  
**Affiliation:** ITMO University, Saint Petersburg, Russian Federation
