# Advanced Time Series Analysis and Forecasting of EUR/USD Exchange Rates

## Data Preparation and Initial Analysis
1. Convert dates to proper format
2. Calculate log returns: $r_t = \log(P_t) - \log(P_{t-1})$

## Rolling Window Forecast
- Window length: $w = 100$
- Number of forecasts: $n = \text{length}(r_t) - w$

## For each window $i = 0$ to $n-1$:

### 1. ARIMA Model Selection
ARIMA (Autoregressive Integrated Moving Average) models are used to capture linear dependencies in the time series. The general form of an ARIMA(p,d,q) model is:

$\phi(B)(1-B)^d X_t = \theta(B)\epsilon_t$

where:
- $B$ is the backshift operator
- $\phi(B) = 1 - \phi_1B - ... - \phi_pB^p$ is the AR polynomial
- $\theta(B) = 1 + \theta_1B + ... + \theta_qB^q$ is the MA polynomial
- $(1-B)^d$ is the differencing operator

In this code, we're using ARIMA(p,0,q) models, assuming the series is already stationary (d=0):

$\phi(B)r_t = \theta(B)\epsilon_t$

We select the best (p,q) combination using the Akaike Information Criterion (AIC):

$AIC = 2k - 2\ln(\hat{L})$

where $k$ is the number of parameters and $\hat{L}$ is the maximum likelihood.

### 2. GARCH-ARIMA Model
While ARIMA models the conditional mean of the series, financial time series often exhibit volatility clustering, where periods of high volatility tend to cluster together. This phenomenon violates the assumption of homoskedasticity in ARIMA models. To address this, we combine ARIMA with a GARCH (Generalized Autoregressive Conditional Heteroskedasticity) model.

The GARCH(1,1)-ARIMA(p,q) model is specified as:

Mean equation:
$r_t = \mu + \sum_{i=1}^p \phi_i r_{t-i} + \sum_{j=1}^q \theta_j \epsilon_{t-j} + \epsilon_t$

Variance equation:
$\epsilon_t = \sigma_t z_t$, where $z_t \sim \text{SGED}(0,1,\nu,\xi)$
$\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2 + \beta \sigma_{t-1}^2$

Where:
- $\sigma_t^2$ is the conditional variance at time t
- $\omega, \alpha, \beta$ are parameters to be estimated
- SGED is the Skewed Generalized Error Distribution with parameters $\nu$ (shape) and $\xi$ (skewness)

The SGED allows for both excess kurtosis (fat tails) and skewness, which are common features in financial returns:

$f(x) = \frac{\nu \exp(-\frac{1}{2}|\frac{x-\mu}{\sigma \xi^{sign(x-\mu)}}|^\nu)}{\sigma \xi^{sign(x-\mu)} 2^{1+1/\nu} \Gamma(1/\nu)}$

### 3. Model Fitting and Forecasting
For each rolling window:
1. Find optimal ARIMA(p,q) order
2. Fit GARCH(1,1)-ARIMA(p,q) model
3. Generate one-step-ahead forecast: 
   $\hat{r}_{t+1|t} = E[r_{t+1}|\mathcal{F}_t]$
   
   Where $\mathcal{F}_t$ is the information set available at time t.
   
4. Predict direction: $d_{t+1} = \text{sign}(\hat{r}_{t+1|t})$

The forecast is based on the mean equation of the GARCH-ARIMA model:

$r_t = \mu + \sum_{i=1}^p \phi_i r_{t-i} + \sum_{j=1}^q \theta_j \epsilon_{t-j} + \epsilon_t$

For a one-step-ahead forecast, this becomes:

$\hat{r}_{t+1|t} = \mu + \sum_{i=1}^p \phi_i r_{t+1-i} + \sum_{j=1}^q \theta_j \epsilon_{t+1-j}$

Where future innovations $\epsilon_{t+1}$ are set to their expected value of zero.

## Rationale for Model Choice
1. ARIMA captures linear dependencies and seasonality in the mean of the series.
2. GARCH captures volatility clustering and time-varying volatility.
3. The combination allows for modeling both the conditional mean and conditional variance.
4. SGED distribution accounts for the non-normality often observed in financial returns.

This approach provides a comprehensive framework for modeling the complex dynamics of financial time series, accounting for both the evolving nature of the series (through rolling windows) and its key statistical properties.
