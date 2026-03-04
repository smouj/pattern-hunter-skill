name: Pattern Hunter
description: Descubre patrones ocultos y correlaciones en conjuntos de datos complejos utilizando algoritmos adaptativos
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

# Habilidad Pattern Hunter

Motor adaptativo de descubrimiento de patrones y análisis de correlaciones para conjuntos de datos complejos. Utiliza métodos ensemble que combinan pruebas estadísticas, clustering y descomposición de series temporales para identificar relaciones no obvias.

## Propósito

Aplicaciones en el mundo real:
- Detectar patrones de fraude en registros de transacciones con >85% de precisión
- Identificar tendencias estacionales en datos de sensores IoT (temperatura, vibración, humedad)
- Encontrar correlaciones de características en datos de entrenamiento de modelos de ML para mejorar la selección de modelos
- Descubrir relaciones de causa raíz en análisis post-mortem de incidentes
- Descubrir secuencias de comportamiento de usuario en análisis de productos
- Detección de anomalías en métricas del sistema con seguimiento de deriva de patrones

## Alcance

### Comandos

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

### Variables de Entorno

```bash
export PATTERN_HUNTER_MAX_WORKERS=4        # Parallel processing workers
export PATTERN_HUNTER_CACHE_DIR=/tmp/ph_cache  # Temporary storage
export PATTERN_HUNTER_MEMORY_LIMIT=0.7    # Memory fraction (0.0-1.0)
export PATTERN_HUNTER_RANDOM_STATE=42     # For reproducibility
export PATTERN_HUNTER_LOGGING=INFO        # DEBUG, INFO, WARNING, ERROR
```

## Proceso de Trabajo

### Fase 1: Preprocesamiento de Datos

1. **Ingerir conjunto de datos** con detección automática de formato (CSV, JSONL, Parquet, Excel)
2. **Inferencia de tipos** para cada columna (numérico, categórico, fecha/hora, texto)
3. Selección de **estrategia de valores faltantes**:
   - Numérico: imputación por mediana (predeterminado) o interpolación
   - Categórico: imputación por moda o marcador de posición 'missing'
4. **Detección de valores atípicos** usando IQR (numérico) o umbral de frecuencia (categórico)
5. Opciones de **normalización**: escalado estándar, min-max o escalado robusto
6. **Análisis de series temporales** con manejo de zona horaria y remuestreo si es necesario

### Fase 2: Descubrimiento de Patrones

#### Para `analyze`:
- Ejecutar correlaciones Pearson/Spearman (numéricas)
- Cramer's V para asociaciones categóricas
- Puntuaciones de información mutua (relaciones no lineales)
- Auto-correlación (ACF/PACF) para series temporales
- Detección de patrones basada en clustering (DBSCAN, K-means)
- Análisis de componentes principales (PCA) para reducción de dimensionalidad

#### Para `correlate`:
- Cross-correlación con detección de desfase
- Pruebas de causalidad de Granger
- Dynamic time warping para secuencias desalineadas
- Pruebas de cointegración (ADF)

#### Para `sequence`:
- Minería de patrones secuenciales (algoritmo PrefixSpan)
- Matrices de transición de cadenas de Markov
- Análisis de frecuencia de N-gramas
- Detección de subsecuencia común más larga

#### Para `drift`:
- Prueba Kolmogorov-Smirnov por columna
- Índice de estabilidad poblacional (PSI)
- Distancia de Wasserstein
- Comparación de distribuciones poblacionales

### Fase 3: Puntuación de Confianza

Cada patrón recibe una puntuación compuesta:
- Significancia estadística (p-valor < 0.05)
- Tamaño del efecto (d de Cohen, magnitud de Cramer's V)
- Consistencia entre ventanas de tiempo
- Validación de lógica de negocio (si se proporcionan reglas)
- Umbral de soporte mínimo (aparece en >X% de los datos)

### Fase 4: Generación de Salida

Los resultados se estructuran como:
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

## Reglas de Oro

1. **SIEMPRE valide la forma de los datos de entrada** antes del procesamiento (filas > 100, columnas > 1)
2. **NUNCA omita la conversión de zona horaria** cuando se especifique `time-column` (usar UTC)
3. **SIEMPRE establezca el umbral `--confidence`**; el valor predeterminado 0.8 es demasiado permisivo para producción
4. **LIMITE los niveles categóricos** a 50 valores distintos; combine automáticamente categorías raras
5. **NUNCA ejecute sin `--groupby`** en series temporales de múltiples entidades (previene errores de agregación)
6. **SIEMPRE muestree** conjuntos de datos > 1M de filas para descubrimiento de patrones (use `--sample 100000`)
7. **VERIFIQUE la estabilidad de los patrones** en al menos 3 divisiones temporales antes de uso empresarial
8. **NUNCA confíe solo en los p-valores**; inspeccione siempre el tamaño del efecto y los intervalos de confianza
9. **ALMACENE en caché los resultados intermedios** para operaciones costosas (se almacena automáticamente cuando se establece `workdir`)
10. **REGISTRE siempre la semilla aleatoria** para cualquier análisis exploratorio

## Ejemplos

### Ejemplo 1: Predicción de Fallos del Sensor

**Prompt de entrada:**
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

**Salida de muestra (extracto):**
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

### Ejemplo 2: Correlación de Transacciones Financieras

**Prompt de entrada:**
```bash
openclaw pattern-hunt correlate \
  --dataset /fraud/transactions_q1.csv \
  --columns amount,merchant_category,ip_country,device_type \
  --method cramer_v \
  --min-correlation 0.65 \
  --output fraud/correlations_q1.csv
```

**Salida de muestra:**
```csv
feature_1,feature_2,cramer_v,p_value,interpretation
amount,merchant_category,0.72,0.0001,High-value transactions cluster in luxury retail
ip_country,device_type,0.68,0.0003,Same device rarely used across countries
```

### Ejemplo 3: Minería de Patrones de Recorrido de Usuario

**Prompt de entrada:**
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

**Salida de muestra:**
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

### Ejemplo 4: Detección de Deriva de Métricas

**Prompt de entrada:**
```bash
openclaw pattern-hunt drift \
  --baseline /metrics/prod_2025-01.csv \
  --current /metrics/prod_2025-02.csv \
  --key-columns service_name,region \
  --value-columns p99_latency,error_rate,request_rate \
  --threshold 0.2 \
  --output metrics/drift_february.csv
```

**Salida de muestra:**
```csv
service,region,metric,psi,threshold,status,action
api-gateway,us-east,p99_latency,0.28,0.2,drift,investigate
api-gateway,eu-west,error_rate,0.08,0.2,stable,monitor
```

## Comandos de Rollback

### Eliminar artefactos generados
```bash
# Delete all pattern outputs and visualizations
rm -rf results/ plots/ correlations/ sequences/ drift_report_*.csv

# Clean cache (if custom cache dir used)
rm -rf /tmp/ph_cache/*
```

### Restaurar conjunto de datos base
```bash
# If analysis accidentally modified original data ( shouldn't happen )
openclaw pattern-hunt verify --dataset /data/original.csv --checksum <original_sha256>
# Expected: "VERIFIED" or "MODIFIED"
```

### Rollback de base de datos (si los resultados fueron ingeridos)
```bash
# If patterns were loaded into database via `--ingest`
openclaw db delete --table pattern_findings \
  --where "created_at > '$(date -d '1 hour ago' +%Y-%m-%d %H:%M:%S)'" \
  --confirm
```

### Cancelar análisis en ejecución
```bash
# Find and kill
pkill -f "pattern-hunt analyze"
# Or if running via OpenClaw daemon:
openclaw daemon stop pattern-hunt-<timestamp>
```

### Revertir configuración
```bash
# Restore environment variables
git checkout ~/.openclaw/config/ext/custom-overrides.json
# Or remove specific export
sed -i '/PATTERN_HUNTER_/d' ~/.bashrc
```

### Restablecimiento completo del sistema
```bash
# Remove all Pattern Hunter data (destructive)
openclaw skill disable pattern-hunter
rm -rf ~/.openclaw/skills/pattern-hunter/work/*
openclaw skill enable pattern-hunter
```

## Pasos de Verificación

### Después de `analyze`:
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

### Después de `correlate`:
```bash
# 1. Validate CSV structure
csvstat correlations/financial_correlations.csv

# 2. Check correlations meet threshold
awk -F, '$3 >= 0.6 {count++} END {print count}' correlations/financial_correlations.csv
```

### Después de `sequence`:
```bash
# 1. Validate sequences longest pattern <= max-pattern-length
jq '.frequent_patterns[].sequence | length' sequences/user_journey_patterns.json | sort -n

# 2. Ensure support values sum to 1.0 (approximate)
jq '[.frequent_patterns[].support] | add' sequences/user_journey_patterns.json
```

### Después de `drift`:
```bash
# 1. Check drift flags (should match threshold)
csvgrep -c status -m drift drift_report_Q2.csv | wc -l

# 2. Verify all flagged services have PSI > threshold
csvsql --query "SELECT service, metric, psi FROM drift_report_Q2 WHERE status='drift' AND psi <= 0.15" drift_report_Q2.csv
```

## Solución de Problemas

### Problema: "MemoryError: Unable to allocate array"
**Causa:** Conjunto de datos demasiado grande para la memoria
**Solución:** Use muestreo: `--sample 100000` o aumente `PATTERN_HUNTER_MEMORY_LIMIT`

### Problema: "ValueError: No numeric columns found"
**Causa:** Todas las columnas detectadas como categóricas/texto
**Solución:** Establezca tipos de columna explícitamente con `--type-overrides '{"column1":"numeric"}'`

### Problema: "Pattern scores all near 0.5"
**Causa:** Relaciones débiles o inexistentes en los datos
**Solución:** Verifique la calidad de los datos, aumente el tamaño del conjunto de datos o relaje el umbral `--confidence`

### Problema: "Killed (core dumped)" en correlación grande
**Causa:** OOM de matriz de correlación densa
**Solución:** Añada el indicador `--sparse-matrix` o reduzca el número de columnas con `--columns`

### Problema: "Time column parsing failed"
**Causa:** Formato de fecha/hora ambiguo
**Solución:** Especifique el formato: `--time-format '%Y-%m-%d %H:%M:%S%z'`

### Problema: "Visualization not generated"
**Causa:** Backend de Plotly faltante
**Solución:** `pip install plotly kaleido` y establezca `export PATTERN_HUNTER_VISUALIZATION_ENGINE=plotly`
```