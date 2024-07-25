# ARIMA_GARCH_FOREX

# Time Series Analysis and Forecasting of EUR/USD Exchange Rates

## Data Preparation
1. Convert dates to proper format
2. Calculate log returns: $r_t = \log(P_t) - \log(P_{t-1})$

## Rolling Window Forecast
- Window length: $w = 100$
- Number of forecasts: $n = \text{length}(r_t) - w$

## For each window $i = 0$ to $n-1$:

1. Select rolling returns: $R_i = \{r_{i+1}, ..., r_{i+w}\}$

2. ARIMA Model Selection
   - Find optimal $(p,q)$ for $ARIMA(p,0,q)$ using AIC:
     $AIC = 2k - 2\ln(\hat{L})$
   where $k$ is the number of parameters and $\hat{L}$ is the maximum likelihood

3. GARCH-ARIMA Model
   - Specify GARCH(1,1)-ARIMA(p,q) model:
     $r_t = \mu + \sum_{i=1}^p \phi_i r_{t-i} + \sum_{j=1}^q \theta_j \epsilon_{t-j} + \epsilon_t$
     $\epsilon_t = \sigma_t z_t$, $z_t \sim \text{SGED}(0,1,\nu,\xi)$
     $\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2 + \beta \sigma_{t-1}^2$

4. Forecast
   - Generate one-step-ahead forecast: $\hat{r}_{t+1|t}$
   - Predict direction: $d_{t+1} = \text{sign}(\hat{r}_{t+1|t})$

## Output
- Forecasts: $\{\hat{r}_{w+1}, ..., \hat{r}_{T}\}$
- Directions: $\{d_{w+1}, ..., d_{T}\}$
