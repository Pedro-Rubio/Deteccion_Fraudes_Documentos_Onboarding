# 🧠 Document Fraud Sentinel — Sistema Inteligente de Detección de Fraude Documental

> **"Verifica la identidad, no solo la imagen."**

Este proyecto desarrolla una solución integral para detectar **fraude documental (DNI y pasaportes)** en procesos de **onboarding digital bancario**.  
Combina visión computacional, análisis contextual y monitoreo continuo para identificar intentos fraudulentos en tiempo real, reduciendo la carga operativa y los falsos positivos.

---

## 🎯 Objetivo General

Clasificar automáticamente si un documento cargado por el cliente es **auténtico o fraudulento**, integrándose de forma transparente al flujo de apertura de cuenta digital.

El modelo entrega:
- Un **score de riesgo (0–1)**.  
- Una **decisión automática**: `ACEPTAR`, `REVISAR_MANUAL` o `RECHAZAR`.  
- **Explicaciones visuales y tabulares** (Grad-CAM / SHAP) para trazabilidad y cumplimiento regulatorio.

---

## 🧩 Arquitectura General del Sistema

```
[Cliente App/Web]
      ↓
 [API Onboarding] → [Servicio ML FastAPI]
                          ↓
                 [Feature Store (Feast)]
                          ↓
             [Prometheus ← Evidently jobs]
                          ↓
                   [Grafana Dashboards]
                          ↓
                [Alertas → Equipo de Fraude]
```

### Componentes Clave
| Módulo | Descripción |
|--------|--------------|
| **FastAPI** | Servicio REST de inferencia (`/score`, `/metrics`) |
| **Modelo CNN + XGBoost** | Fusión de análisis visual y contextual |
| **Feature Store (Feast)** | Gestión de features online/offline |
| **Prometheus** | Recolección de métricas (latencia, drift, errores) |
| **Grafana** | Dashboards y alertas del sistema |
| **Evidently** | Cálculo de drift y calidad de datos |
| **MLflow** | Tracking y versionado de modelos |
| **Streamlit** | Consola de analistas para revisión manual |

---

## 🏗️ Arquitectura de Ingeniería de Datos (Medallion)

El sistema sigue un enfoque de **Data Lakehouse estructurado en capas Medallion (Bronze → Silver → Gold)** para garantizar trazabilidad, calidad y escalabilidad.

### 🥉 **Bronze (Raw / Aterrizaje)**
Almacena los datos tal como llegan del proceso de onboarding:

- Imágenes originales (`dni_front`, `dni_back`, `passport_page`).
- Metadatos de carga (fecha, IP, dispositivo, EXIF, ubicación).
- OCR crudo y validaciones MRZ.
- JSON original de la app.

Ejemplo de tabla: `raw_onboarding_events`
```text
event_id, user_id, doc_type, upload_ts, img_front_path, img_back_path, raw_metadata
```

---

### 🥈 **Silver (Curado / Limpio)**
Estandariza, normaliza y enriquece los datos.

Incluye:
- Parsing de metadatos JSON.  
- Cálculo de indicadores forenses (blur, ELA, pHash).  
- Resultados de OCR/MRZ validados.  
- Reglas contextuales (intentos previos, IPs, dispositivos repetidos).

Tablas principales:
- `doc_capture_enriched_silver`  
- `doc_visual_features_silver`  
- `doc_text_features_silver`  
- `doc_behavior_silver`

---

### 🥇 **Gold (Analítico / ML-Ready)**
Integra todas las fuentes para entrenar y servir modelos.

Tabla principal: `fraud_doc_onboarding_gold`
```text
event_id, user_id, doc_type, upload_ts, laplacian_var, ela_mean,
mrz_valid, ocr_doc_match_form, ip_country, device_type,
attempts_last_24h, doc_reuse_count, ip_reuse_count, is_fraud
```

Esta tabla se usa en los notebooks de entrenamiento y validación (Google Colab).

---

### 🧠 **Feature Store (Feast)**
Gestiona features tanto para entrenamiento (offline) como para scoring (online):

| Feature Set | Descripción |
|--------------|--------------|
| `fs_doc_features_v1` | Variables visuales y OCR (ELA, blur, MRZ, OCR match) |
| `fs_user_risk_v1` | Historial del usuario y dispositivo |
| `fs_device_risk_v1` | Frecuencia y comportamiento de carga |

---

### ⚙️ **Orquestación (Airflow / Prefect)**
- **Job 1:** Landing → Bronze (ingesta continua).  
- **Job 2:** Bronze → Silver (enriquecimiento y cálculo de features).  
- **Job 3:** Silver → Gold (join completo + etiquetado).  
- **Job 4:** Retraining (semanal).  
- **Job 5:** Monitoreo de drift (Evidently + Prometheus).

---

## 🧠 Enfoque de Modelado

| Canal | Fuente | Ejemplos de Features |
|-------|---------|----------------------|
| 🖼️ **Visual** | Imágenes frente/dorso | ELA, blur, phash_diff, aspect_ratio |
| 📋 **Contextual** | Metadatos de carga | IP, país, dispositivo, hora, conexión |
| 🔍 **OCR / MRZ** | Texto extraído | MRZ válida, OCR↔formulario consistente |

### Modelos utilizados
- **CNN liviana** (MobileNet / EfficientNet) → `p_fraude_img`  
- **XGBoost / CatBoost** → `p_fraude_tab`  
- **Fusión tardía** (Logistic Regression calibrada) → `score_riesgo`  
- **Detección de anomalías** (IsolationForest) → casos fuera de distribución

---

## 🧮 Pipeline de Machine Learning

1. **Ingesta y limpieza (Bronze → Silver)**  
   - OCR, MRZ, EXIF, ELA, pHash.
2. **Feature engineering (Silver)**  
   - Normalización, joins, derivación de variables contextuales.
3. **Dataset final (Gold)**  
   - Unión completa, etiquetas confirmadas.
4. **Entrenamiento multimodal**  
   - CNN + XGBoost + fusión.  
   - Métricas: Recall > 0.85, FPR < 0.05.
5. **Servir modelo (FastAPI)**  
   - `/score` → scoring  
   - `/metrics` → Prometheus  
6. **Monitoreo (Evidently + Grafana)**  
   - Drift, latencia, tasa de errores, PSI.

---

## 🧱 Estructura del Repositorio

```
document-fraud-sentinel/
├── data/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── dags/                      # Airflow / Prefect
│   ├── bronze_ingest.py
│   ├── silver_enrich.py
│   └── gold_unify.py
├── notebooks/                 # Google Colab notebooks
│   ├── 01_eda_bronze.ipynb
│   ├── 02_features_silver.ipynb
│   └── 03_model_gold.ipynb
├── app/                       # FastAPI / Streamlit
│   ├── main.py
│   └── requirements.txt
├── monitoring/
│   ├── prometheus.yml
│   └── grafana/
└── README.md
```

---

## 🧰 Stack Tecnológico

| Categoría | Herramientas |
|------------|---------------|
| **Visión y OCR** | OpenCV · Pillow · Tesseract · PaddleOCR |
| **Machine Learning** | PyTorch · TensorFlow · XGBoost · scikit-learn |
| **Feature Store** | Feast |
| **Orquestación** | Airflow · Prefect |
| **Monitoreo** | Prometheus · Grafana · Evidently |
| **Infraestructura** | Docker · FastAPI · ONNX Runtime |
| **UI Operativa** | Streamlit |
| **Versionado** | MLflow |
| **Validación** | Great Expectations |

---

## 🧰 Ejemplo de API de Scoring

**POST /score**
```bash
curl -X POST http://localhost:8000/score   -F "img_front=@dni_frente.jpg"   -F "meta_json={\"ip\":\"200.89.101.1\",\"device\":\"Android\"}"
```

**Respuesta**
```json
{
  "score": 0.72,
  "decision": "RECHAZAR",
  "reasons": [
    ["ela_mean", 0.31],
    ["laplacian_var", 22.8],
    ["deskew_ok", 1]
  ]
}
```

---

## 📊 Monitoreo (Prometheus + Grafana)

El servicio expone métricas de operación:

- `fraude_score_latency_seconds` → Latencia  
- `fraude_model_score` → Distribución de scores  
- `fraude_feature_psi` → Drift por feature  
- `fraude_errors_total` → Errores en scoring  

### Despliegue local
```bash
docker compose up --build
```
- FastAPI → http://localhost:8000/docs  
- Prometheus → http://localhost:9090  
- Grafana → http://localhost:3000 (admin / admin)

---

## ⚖️ Privacidad y Cumplimiento

Cumple con **Ley 25.326 (Argentina)** y **GDPR (UE)**:

- Minimización y pseudonimización de datos.  
- Cifrado en tránsito (TLS 1.2+) y en reposo (AES-256).  
- Logs auditables y control RBAC.  
- Model Cards y explicabilidad por feature.  
- Retención limitada y borrado seguro de imágenes.

---

## 📈 KPIs del Proyecto

| Indicador | Meta | Descripción |
|------------|------|--------------|
| Recall fraude | > 85% | Capacidad para detectar intentos reales |
| FPR | < 5% | Rechazos erróneos mínimos |
| Latencia P95 | < 2s | Tiempo máximo de respuesta |
| PSI (Drift) | < 0.2 | Estabilidad de datos |
| % reducción de revisiones manuales | -30% | Impacto operativo directo |

---

## 🧭 Roadmap de Desarrollo

| Fase | Entregable | Estado |
|------|-------------|--------|
| 1️⃣ | Pipeline Bronze → Silver | ✅ |
| 2️⃣ | Feature Engineering completo | ✅ |
| 3️⃣ | Modelo multimodal (CNN + XGBoost) | ✅ |
| 4️⃣ | API REST y monitoreo básico | ✅ |
| 5️⃣ | Evidently + Grafana (drift + alertas) | 🔄 En curso |
| 6️⃣ | Streamlit revisión manual | 🔜 |
| 7️⃣ | MLflow retraining automatizado | 🔜 |

---

## ✍️ Nota del Coordinador Técnico

> “La prioridad es mantener un equilibrio entre precisión, interpretabilidad y cumplimiento normativo.  
Cada fase del pipeline está diseñada para auditar y reproducir las decisiones del modelo, garantizando confianza tanto técnica como regulatoria.”

**Pedro Rubio**  
_Data & ML Analyst — Coordinador Técnico del Proyecto_  
📧 srdelosdatos@gmail.com · 🌐 [github.com/srdelosdatos](https://github.com/srdelosdatos)

---

### 🧩 Licencia
Este proyecto utiliza datos sintéticos y código abierto bajo licencia **MIT**.  
Se prohíbe el uso con datos reales sin cumplir con la normativa de protección de datos aplicable.
