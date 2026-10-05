# Project checklist: pension fund volatility and VaR

**Legend:** `[x]` drafted in this repo and tested locally (19 pytest tests pass, pipeline runs on all 3 funds);
`[ ]` still to do, or needs running on your GitLab/Azure/Databricks. Cert exams are shown so the project
tasks line up with the study plan. Move an exam by 1-3 days if your timed practice score is below ~80%.

## Setup (before Day 1)
- [ ] Create a **private** GitLab project, push this repo, protect `main`
- [ ] Enable GitLab Duo; start a new chat and confirm it follows `AGENTS.md` and `.gitlab/duo/chat-rules.md`
- [ ] Create an Azure service principal; add `ARM_*` and `TF_VAR_subscription_id` as masked CI/CD variables
- [ ] Set an Azure budget alert and tag resources with `cost-owner`
- [ ] Install Terraform locally (needed for `fmt`, `validate`, `plan`)

## Week 1: foundations, stylized facts, ARCH tests

### Day 1: repo, CI skeleton, data ingest
- [x] Repo layout, `pyproject.toml`, `config.yaml`, `.gitignore`
- [x] `ingest.py`: robust CSV loader (BOM, MM/DD/YYYY, trailing commas), log-returns in percent
- [x] Tests that all 3 funds load, are sorted, positive, and returns match `100*ln(P_t/P_{t-1})`
- [ ] First pipeline run on GitLab (`ruff` and `pytest` jobs green)

### Day 2: Terraform base
- [x] `infra/`: resource group, VNet, subnet, NSG, storage (ADLS Gen2), Key Vault, private DNS
- [ ] Run `terraform fmt && terraform init -backend=false && terraform validate`; fix any errors
- [ ] `terraform init` against GitLab-managed state; commit `.terraform.lock.hcl`

### Day 3: CI for infra
- [x] `.gitlab-ci.yml`: ruff, pytest, tf-validate, tf-plan, tf-apply (manual), run-pipeline, pages
- [ ] Open a merge request; confirm tf-validate and tf-plan run and the plan appears in the MR
- [ ] **Exam: Terraform Associate (Day 4)**

### Day 4: stylized facts
- [x] `diagnostics.stylized_facts`: ADF, skewness, kurtosis, Jarque-Bera, Ljung-Box (returns and squared returns)
- [ ] Notebook: price vs return plots, histogram vs normal, ACF of returns and squared returns (stylized facts 1-5)
- [ ] Short write-up: which stylized facts hold in each fund

### Day 5: mean equation
- [x] Constant mean in all models
- [ ] Test an AR(1) mean (`mean="AR", lags=1`) and compare with constant mean by BIC
- [ ] Ljung-Box on returns documented per fund

### Day 6: ARCH-effect tests
- [x] `arch_effect_present`: ARCH-LM and Ljung-Box on squared returns
- [ ] Table in the notebook: test statistics and p-values at lags 1, 6, 12
- [ ] Note why ARCH effects can be weaker at monthly frequency (Hurlin, stylized fact 7)

### Day 7: ARCH and GARCH estimation
- [x] ARCH(1), ARCH(3), GARCH(1,1) with normal, Student-t, skew-t, GED innovations
- [ ] Review parameter estimates and confirm stationarity (`persistence < 1`) for the chosen models
- [ ] Compare GARCH(1,1) against the BIC winner for each fund (set `forced_spec`)

## Week 2: asymmetric models, Azure network, Databricks

### Day 8: private networking
- [x] Private endpoints + private DNS for storage and Key Vault in Terraform
- [ ] Apply to a dev subscription; confirm public access is blocked and private resolution works from the VNet
- [ ] Store any secrets in Key Vault, not CI variables where possible

### Day 9: extensions and model selection
- [x] GJR-GARCH, TGARCH, EGARCH; AIC/BIC ranking table per fund (`model_selection.csv`)
- [ ] Interpret the leverage effect: sign and size of the asymmetry term per fund
- [ ] **Stretch:** GARCH-M (needs a custom likelihood) and a true IGARCH (alpha + beta = 1) restriction

### Day 10: residual diagnostics
- [x] `residual_diagnostics`: Ljung-Box on z and z-squared, ARCH-LM on z
- [ ] Add QQ-plot and ACF of standardized residuals to the notebook
- [ ] Decide for each fund whether the selected model is adequate; document exceptions

### Day 11: baselines
- [x] Random walk with drift, damped Holt-Winters, AR(1) ported to `baselines.py` and tested
- [x] Fidelity fund added to the config (it was not in the original workbook)
- [ ] Reconcile `outputs/*/baseline_forecasts.csv` with `legacy/fund_forecast_calculations.xlsx` for the two original funds
- [ ] **Exam: AZ-104**

### Day 12: Databricks workspace
- [x] `azurerm_databricks_workspace` (premium) in Terraform behind `enable_databricks`
- [ ] Apply; open the workspace; create `raw` and `curated` containers from inside the VNet or Databricks
- [ ] Upload the 3 CSVs to the `raw` zone

### Day 13: Delta tables and a job
- [ ] Bronze (raw CSV) -> silver (clean prices, returns) -> gold (selected model outputs) Delta tables
- [ ] Schedule a Databricks Workflow that runs the pipeline monthly
- [ ] Data-quality checks: duplicates, gaps in month-end dates, non-positive prices

### Day 14: MLflow
- [ ] Log every model fit (spec, distribution, AIC, BIC, persistence, diagnostics p-values)
- [ ] Register the best model per fund; tag with data end date
- [ ] Add `mlflow` via `pip install -e ".[databricks]"`

## Week 3: forecasts, VaR, backtests, AI layer, release

### Day 15: out-of-sample forecasts
- [x] Rolling one-step mean forecasts vs baselines (`mean_forecast_errors.csv`)
- [x] Rolling volatility forecasts: GARCH vs EWMA vs historical, MSE and QLIKE (`volatility_eval.csv`)
- [x] 3-month GARCH risk bands (`garch_risk_bands.csv`, `risk_band.png`)
- [ ] Check bands against realised outcomes in a pseudo-live test (hold out the last 12 months)

### Day 16: VaR
- [x] Parametric GARCH, filtered historical simulation, historical, RiskMetrics at 95% and 99%
- [ ] Add Expected Shortfall (parametric and FHS)
- [ ] Sensitivity: refit frequency (3, 6, 12 months) and minimum training window

### Day 17: backtests
- [x] Kupiec (UC), Christoffersen independence and conditional coverage
- [ ] Note the limited power at 99% with monthly data; add daily iShares prices for a longer test sample
- [ ] **Exam: Databricks ML Associate**

### Day 18: Basel-style reporting
- [x] Binomial traffic-light zones (reproduces Basel 0-4 / 5-9 / 10+ for 250 obs at 1%)
- [x] `report.md` and `index.html` with per-fund tables and charts
- [ ] Add a one-page interpretation per fund: what the model says, where it fails

### Day 19: AI commentary (AI-103)
- [x] `commentary.py`: grounded context from `summary.json`, system prompt that forbids invented numbers
- [ ] Set `enable_openai = true`, apply, and add an `azurerm_cognitive_deployment` for an available model
- [ ] Build an evaluation set: numeric accuracy, groundedness, refusal to give investment advice
- [ ] Turn on content filtering; use managed identity instead of API keys

### Day 20: monitoring and cost
- [ ] Alert when VaR exceptions in the last 12 months exceed the binomial threshold
- [ ] Track parameter drift between refits (alpha, beta, nu)
- [ ] Review Azure costs; `terraform destroy` for anything not needed

### Day 21: release
- [ ] Architecture diagram in the README (data -> Databricks -> MLflow -> VaR -> report -> Pages)
- [ ] Verify GitLab Pages is restricted to project members
- [ ] Tag `v0.1.0`; write release notes with limits and known issues
- [ ] **Exam: AI-103**

## Optional stretch
- [ ] Multivariate GARCH (DCC) across the three funds, if "v-garch" meant vector GARCH
- [ ] Daily data for a longer VaR backtest sample
- [ ] VNet-injected Databricks with Unity Catalog and an external location on the storage account
- [ ] Mean-variance or risk-parity analysis using the fitted volatilities
