---
name: Pattern Hunter
description: Discovers hidden patterns and correlations in complex datasets using adaptive algorithms
version: 2.1.0
author: OpenClaw Analytics Team
maintainer: smouj
tags:
  - pattern-recognition
  - correlations
  - analysis
  - visualization
  - data-science
  - machine-learning
type: analysis
dependencies:
  - python>=3.9
  - numpy>=1.21.0
  - pandas>=1.3.0
  - scikit-learn>=0.24.0
  - matplotlib>=3.4.0
  - seaborn>=0.11.0
  - plotly>=5.1.0
  - scipy>=1.7.0
  - networkx>=2.6.0
  - joblib>=1.0.0
system_requirements:
  min_ram_mb: 4096
  min_disk_gb: 5
  python_packages_install: true
workdir_requires: true
output_formats:
  - json
  - csv
  - plotly_html
  - png
  - pdf
---

# Pattern Hunter Skill

Adaptive pattern discovery and correlation analysis engine for complex datasets. Uses ensemble methods combining statistical tests, clustering, and time-series decomposition to identify non-obvious relationships.

## Purpose

Real-world applications:
- Detect fraud patterns in transaction logs with >85% precision
- Identify seasonal trends in IoT sensor data (temperature, vibration, humidity)
- Find feature correlations in ML training data to improve model selection
- Discover root-cause relationships in incident post-mortems
- Uncover user behavior sequences in product analytics
- Anomaly detection in system metrics with pattern drift tracking

## Scope

### Commands

#### `pattern-hunt analyze`
```bash
openclaw pattern-hunt analyze \
  --dataset /data/sensor_readings.csv \
  --target-column 'failure_flag' \
  --time-column 'timestamp' \
  --method ensemble \
  --confidence 0.85 \
  --output results/sensor_patterns.json \
  --visualize \
  --groupby device_id
```

#### `pattern-hunt correlate`
```bash
openclaw pattern-hunt correlate \
  --dataset /data/financial_transactions.csv \
  --columns amount,merchant_category,location \
  --max-lags 24 \
  --min-correlation 0.6 \
  --output correlations/financial_correlations.csv
```

#### `pattern-hunt sequence`
```bash
openclaw pattern-hunt sequence \
  --dataset /data/user_events.jsonl \
  --event-column 'action' \
  --user-column 'user_id' \
  --time-column 'ts' \
  --min-support 0.1 \
  --max-pattern-length 5 \
  --output sequences/user_journey_patterns.json
```

#### `pattern-hunt drift`
```bash
openclaw pattern-hunt drift \
  --baseline /data/metrics_2025_Q1.csv \
  --current /data/metrics_2025_Q2.csv \
  --key-columns service,instance \
  --value-columns cpu,memory,latency \
  --threshold 0.15 \
  --output drift_report_Q2.csv
```

#### `pattern-hunt visualize`
```bash
openclaw pattern-hunt visualize \
  --results results/sensor_patterns.json \
  --type heatmap \
  --title 'Sensor Failure Correlations' \
  --output plots/sensor_heatmap.html
```

### Environment Variables

```bash
export PATTERN_HUNTER_MAX_WORKERS=4        # Parallel processing workers
export PATTERN_HUNTER_CACHE_DIR=/tmp/ph_cache  # Temporary storage
export PATTERN_HUNTER_MEMORY_LIMIT=0.7    # Memory fraction (0.0-1.0)
export PATTERN_HUNTER_RANDOM_STATE=42     # For reproducibility
export PATTERN_HUNTER_LOGGING=INFO        # DEBUG, INFO, WARNING, ERROR
```

## Work Process

### Phase 1: Data Preprocessing

1. **Ingest dataset** with automatic format detection (CSV, JSONL, Parquet, Excel)
2. **Type inference** for each column (numeric, categorical, datetime, text)
3. **Missing value strategy** selection:
   - Numeric: median imputation (default) or interpolation
   - Categorical: mode imputation or 'missing' placeholder
4. **Outlier detection** using IQR (numeric) or frequency threshold (categorical)
5. **Normalization** options: standard scaler, min-max, or robust scaling
6. **Time-series parsing** with timezone handling and resampling if needed

### Phase 2: Pattern Discovery

#### For `analyze`:
- Run Pearson/Spearman correlations (numeric)
- Cramer's V for categorical associations
- Mutual information scores (non-linear relationships)
- Auto-correlation (ACF/PACF) for time series
- Clustering-based pattern detection (DBSCAN, K-means)
- Principal component analysis for dimensionality reduction

#### For `correlate`:
- Cross-correlation with lag detection
- Granger causality tests
- Dynamic time warping for misaligned sequences
- Cointegration tests (ADF)

#### For `sequence`:
- Sequential pattern mining (PrefixSpan algorithm)
- Markov chain transition matrices
- N-gram frequency analysis
- Longest common subsequence detection

#### For `drift`:
- Kolmogorov-Smirnov test per column
- Population stability index (PSI)
- Wasserstein distance
- Population distribution comparison

### Phase 3: Confidence Scoring

Each pattern receives composite score:
- Statistical significance (p-value < 0.05)
- Effect size (Cohen's d, Cramer's V magnitude)
- Consistency across time windows
- Business logic validation (if rules provided)
- Minimum support threshold (appears in >X% of data)

### Phase 4: Output Generation

Results structured as:
```json
{
  "patterns": [
    {
      "id": "pat_001",
      "type": "correlation",
      "features": ["cpu_usage", "disk_io"],
      "score": 0.87,
      "p_value": 0.002,
      "lag": 0,
      "visualization": "heatmap_cell_[0,1]",
      "description": "High CPU usage precedes disk I/O spikes by 2 minutes"
    }
  ],
  "metadata": {
    "dataset_hash": "sha256:abc123",
    "rows_processed": 145678,
    "runtime_seconds": 42.3,
    "parameters": {...}
  }
}
```

## Golden Rules

1. **ALWAYS validate input data shape** before processing (rows > 100, columns > 1)
2. **NEVER skip timezone conversion** when `time-column` specified (use UTC)
3. **ALWAYS set `--confidence`** threshold; default 0.8 is too permissive for production
4. **LIMIT categorical levels** to 50 distinct values; merge rare categories automatically
5. **NEVER run without `--groupby`** on multi-entity time series (prevents aggregation bugs)
6. **ALWAYS sample** datasets > 1M rows for pattern discovery (use `--sample 100000`)
7. **VERIFY pattern stability** across at least 3 temporal splits before business use
8. **NEVER trust p-values alone**; always inspect effect size and confidence intervals
9. **CACHE intermediate results** for expensive operations (auto-cached when `workdir` set)
10. **ALWAYS log the random seed** for any exploratory analysis

## Examples

### Example 1: Sensor Failure Prediction

**Input prompt:**
```bash
openclaw pattern-hunt analyze \
  --dataset /iot/sensor_data_30d.csv \
  --target-column 'failure_flag' \
  --time-column 'reading_time' \
  --method ensemble \
  --confidence 0.9 \
  --groupby sensor_id \
  --output iot/failure_patterns.json \
  --visualize
```

**Sample output (excerpt):**
```json
{
  "patterns": [
    {
      "id": "pat_sensor_45",
      "type": "multivariate_anomaly",
      "features": ["temperature", "vibration"],
      "score": 0.94,
      "conditions": {
        "temperature__rolling_mean_5m > 75 AND vibration__std_10m > 0.15"
      },
      "lead_time_minutes": 12,
      "false_positive_rate": 0.03,
      "visualization": "sensor_45_anomaly_timeline.html"
    }
  ]
}
```

### Example 2: Financial Transaction Correlation

**Input prompt:**
```bash
openclaw pattern-hunt correlate \
  --dataset /fraud/transactions_q1.csv \
  --columns amount,merchant_category,ip_country,device_type \
  --method cramer_v \
  --min-correlation 0.65 \
  --output fraud/correlations_q1.csv
```

**Sample output:**
```csv
feature_1,feature_2,cramer_v,p_value,interpretation
amount,merchant_category,0.72,0.0001,High-value transactions cluster in luxury retail
ip_country,device_type,0.68,0.0003,Same device rarely used across countries
```

### Example 3: User Journey Pattern Mining

**Input prompt:**
```bash
openclaw pattern-hunt sequence \
  --dataset /app/events_20250101.jsonl \
  --event-column 'event_name' \
  --user-column 'user_uuid' \
  --time-column 'event_ts' \
  --min-support 0.05 \
  --max-pattern-length 4 \
  --output app/common_journeys.json
```

**Sample output:**
```json
{
  "frequent_patterns": [
    {
      "sequence": ["view_product", "add_cart", "checkout_start", "purchase"],
      "support": 0.12,
      "avg_duration_minutes": 8.3,
      "conversion_rate": 0.87
    }
  ]
}
```

### Example 4: Metric Drift Detection

**Input prompt:**
```bash
openclaw pattern-hunt drift \
  --baseline /metrics/prod_2025-01.csv \
  --current /metrics/prod_2025-02.csv \
  --key-columns service_name,region \
  --value-columns p99_latency,error_rate,request_rate \
  --threshold 0.2 \
  --output metrics/drift_february.csv
```

**Sample output:**
```csv
service,region,metric,psi,threshold,status,action
api-gateway,us-east,p99_latency,0.28,0.2,drift,investigate
api-gateway,eu-west,error_rate,0.08,0.2,stable,monitor
```

## Rollback Commands

### Remove generated artifacts
```bash
# Delete all pattern outputs and visualizations
rm -rf results/ plots/ correlations/ sequences/ drift_report_*.csv

# Clean cache (if custom cache dir used)
rm -rf /tmp/ph_cache/*
```

### Restore baseline dataset
```bash
# If analysis accidentally modified original data ( shouldn't happen )
openclaw pattern-hunt verify --dataset /data/original.csv --checksum <original_sha256>
# Expected: "VERIFIED" or "MODIFIED"
```

### Database rollback (if results ingested)
```bash
# If patterns were loaded into database via `--ingest`
openclaw db delete --table pattern_findings \
  --where "created_at > '$(date -d '1 hour ago' +%Y-%m-%d %H:%M:%S)'" \
  --confirm
```

### Cancel running analysis
```bash
# Find and kill
pkill -f "pattern-hunt analyze"
# Or if running via OpenClaw daemon:
openclaw daemon stop pattern-hunt-<timestamp>
```

### Revert configuration
```bash
# Restore environment variables
git checkout ~/.openclaw/config/ext/custom-overrides.json
# Or remove specific export
sed -i '/PATTERN_HUNTER_/d' ~/.bashrc
```

### Full system reset
```bash
# Remove all Pattern Hunter data (destructive)
openclaw skill disable pattern-hunter
rm -rf ~/.openclaw/skills/pattern-hunter/work/*
openclaw skill enable pattern-hunter
```

## Verification Steps

### After `analyze`:
```bash
# 1. Check output exists and valid JSON
jq empty results/sensor_patterns.json

# 2. Count patterns found (should be > 0)
jq '.patterns | length' results/sensor_patterns.json

# 3. Verify high-confidence patterns
jq '.patterns[] | select(.score > 0.85)' results/sensor_patterns.json | wc -l

# 4. If visualizations generated, check HTML renders
python -c "from plotly.io import validate_html; validate_html('plots/sensor_heatmap.html')"
```

### After `correlate`:
```bash
# 1. Validate CSV structure
csvstat correlations/financial_correlations.csv

# 2. Check correlations meet threshold
awk -F, '$3 >= 0.6 {count++} END {print count}' correlations/financial_correlations.csv
```

### After `sequence`:
```bash
# 1. Validate sequences longest pattern <= max-pattern-length
jq '.frequent_patterns[].sequence | length' sequences/user_journey_patterns.json | sort -n

# 2. Ensure support values sum to 1.0 (approximate)
jq '[.frequent_patterns[].support] | add' sequences/user_journey_patterns.json
```

### After `drift`:
```bash
# 1. Check drift flags (should match threshold)
csvgrep -c status -m drift drift_report_Q2.csv | wc -l

# 2. Verify all flagged services have PSI > threshold
csvsql --query "SELECT service, metric, psi FROM drift_report_Q2 WHERE status='drift' AND psi <= 0.15" drift_report_Q2.csv
```

## Troubleshooting

### Issue: "MemoryError: Unable to allocate array"
**Cause:** Dataset too large for memory
**Fix:** Use sampling: `--sample 100000` or increase `PATTERN_HUNTER_MEMORY_LIMIT`

### Issue: "ValueError: No numeric columns found"
**Cause:** All columns detected as categorical/text
**Fix:** Explicitly set column types with `--type-overrides '{"column1":"numeric"}'`

### Issue: "Pattern scores all near 0.5"
**Cause:** Weak/no relationships in data
**Fix:** Check data quality, increase dataset size, or relax `--confidence` threshold

### Issue: "Killed (core dumped)" on large correlation
**Cause:** OOM from dense correlation matrix
**Fix:** Add `--sparse-matrix` flag or reduce number of columns with `--columns`

### Issue: "Time column parsing failed"
**Cause:** Ambiguous datetime format
**Fix:** Specify format: `--time-format '%Y-%m-%d %H:%M:%S%z'`

### Issue: "Visualization not generated"
**Cause:** Plotly backend missing
**Fix:** `pip install plotly kaleido` and set `export PATTERN_HUNTER_VISUALIZATION_ENGINE=plotly`
```