# 🧠 Document Fraud Sentinel — Detección Inteligente de Fraude en Onboarding Bancario

> **"Verifica la identidad, no solo la imagen."**

Este proyecto aborda la detección temprana de **fraude documental (DNI y pasaportes)** en procesos de **onboarding digital bancario**.  
El sistema combina visión computacional, análisis contextual y monitoreo continuo para identificar intentos fraudulentos en la carga de documentos de identidad durante la apertura de cuentas.

---

## 🚀 Objetivo

Clasificar automáticamente si un documento cargado (DNI o pasaporte) es **auténtico o fraudulento**, integrándose de forma transparente al flujo de onboarding digital.

El modelo opera en **tiempo real**, devolviendo:
- Un **score de riesgo (0–1)**.
- Una **decisión operativa**: `ACEPTAR`, `REVISAR_MANUAL` o `RECHAZAR`.
- **Razones explicativas** (Grad-CAM / SHAP) para trazabilidad y auditoría.

---

## 🧩 Arquitectura General

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

### 🔹 Componentes Principales
| Módulo | Descripción |
|--------|--------------|
| **FastAPI** | Servicio REST de inferencia (`/score`, `/metrics`) |
| **Modelo CNN + XGBoost** | Combina señales visuales y contextuales |
| **Feature Store (Feast)** | Gestión de features online/offline |
| **Prometheus** | Monitoreo de métricas (latencia, PSI, errores) |
| **Grafana** | Dashboards y alertas de modelo |
| **Evidently** | Cálculo de drift y calidad de datos |
| **MLflow** | Versionado de experimentos y modelos |
| **Streamlit (opcional)** | Consola de analistas para revisión manual |

---

## 🧠 Enfoque de Modelado

El sistema es **multimodal**:

| Canal | Fuente de datos | Ejemplo de features |
|-------|------------------|---------------------|
| 🖼️ **Visual** | Imágenes frente/dorso del documento | ELA, blur, bordes, pHash, layout, matching con plantilla oficial |
| 📋 **Contextual** | Metadatos de carga | IP, geolocalización, EXIF, tipo de conexión, timestamp, dispositivo |
| 🔍 **OCR / MRZ** | Texto extraído | Inconsistencias entre OCR ↔ formulario ↔ MRZ |

### 🔧 Modelos
- **CNN liviana (MobileNet / EfficientNet)** → `p_fraude_img`
- **XGBoost / CatBoost** → `p_fraude_tab`
- **Fusión tardía (Logistic Regression calibrada)** → `score_riesgo`
- **Detección de anomalías** (`IsolationForest`) para casos out-of-distribution

---

## 🧮 Pipeline de Machine Learning

1. **Ingesta**
   - Recepción del documento y metadatos desde el frontend.
   - Validación inicial (formato, tamaño, EXIF básico).

2. **Preprocesamiento**
   - Deskew, normalización de iluminación, detección de bordes (OpenCV).
   - OCR y validación MRZ (Tesseract / PaddleOCR).
   - Extracción de indicadores forenses (ELA, blur, pHash).

3. **Feature Engineering**
   - Construcción de variables visuales y contextuales.
   - Fuzzy matching OCR ↔ formulario (Levenshtein / Jaro-Winkler).

4. **Entrenamiento**
   - CNN + XGBoost con fusión tardía.
   - Focal loss, class weights, cross-validation temporal.
   - Explicabilidad con Grad-CAM y SHAP.

5. **Serving**
   - FastAPI + ONNX Runtime para baja latencia.
   - Respuesta en segundos, compatible con APIs de onboarding.

6. **Monitoreo y Drift**
   - Evidently calcula PSI, KS y métricas de estabilidad.
   - Prometheus recolecta métricas.
   - Grafana visualiza dashboards y lanza alertas.

---

## ⚙️ Stack Tecnológico

| Categoría | Herramientas |
|------------|---------------|
| **Visión y OCR** | OpenCV · Pillow · Tesseract · PaddleOCR |
| **Machine Learning** | PyTorch · TensorFlow · scikit-learn · XGBoost |
| **Tracking & Orquestación** | MLflow · Airflow/Prefect |
| **Feature Store** | Feast |
| **Monitoreo** | Prometheus · Grafana · Evidently |
| **Infraestructura** | Docker · FastAPI · ONNX Runtime |
| **Explicabilidad** | SHAP · Grad-CAM |
| **Calidad de Datos** | Great Expectations |

---

## 🧱 Estructura de Carpetas

```
document-fraud-sentinel/
├── app/                      # Servicio FastAPI
│   ├── main.py               # Endpoints /score y /metrics
│   ├── preprocessing.py      # Extracción de features forenses
│   ├── models/               # CNN + XGBoost + fusión
│   └── requirements.txt
│
├── notebooks/                # EDA, entrenamiento, validación
├── prometheus/               # Configuración Prometheus
├── grafana/                  # Provisioning de dashboards
├── docker-compose.yml        # Stack completo app + prometheus + grafana
├── README.md
└── LICENSE
```

---

## 🧰 Ejemplo de Scoring API

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

El servicio FastAPI expone métricas nativas para Prometheus:

- `fraude_score_latency_seconds` → latencia por solicitud  
- `fraude_model_score` → distribución de scores de fraude  
- `fraude_feature_psi` → drift por feature (publicado desde Evidently)  
- `fraude_errors_total` → errores en scoring  

### 🚀 Despliegue local
```bash
docker compose up --build
```
- FastAPI → http://localhost:8000/docs  
- Prometheus → http://localhost:9090  
- Grafana → http://localhost:3000 (admin / admin)

---

## ⚖️ Privacidad y Cumplimiento

Cumple **Ley 25.326 (Argentina)** y **GDPR (UE)**:
- Minimización y pseudonimización de datos.
- Cifrado AES-256 en reposo, TLS 1.2+ en tránsito.
- Logs auditables y control RBAC.
- Model Cards y explicaciones auditables (Grad-CAM/SHAP).

---

## 📈 Métricas Clave

| Métrica | Objetivo | Descripción |
|----------|-----------|-------------|
| Recall de fraude | > 85% | Minimizar falsos negativos |
| FPR | < 5% | No rechazar usuarios legítimos |
| Latencia (P95) | < 2s | Respuesta rápida en onboarding |
| PSI (Drift) | < 0.2 | Estabilidad del modelo |
| Costo esperado | ↓ constante | Costo de revisión/errores |

---

## 🔍 Ética y Sesgos

- Evaluación de performance por subgrupos (país, tipo de documento, dispositivo).
- Revisión humana en decisiones “REVISAR_MANUAL”.
- Documentación de fairness y trazabilidad.
- Mecanismo de apelación para clientes legítimos.

---

## 📅 Roadmap

| Etapa | Entregable | Estado |
|--------|-------------|---------|
| 1️⃣ | Pipeline de features (imagen + OCR) | ✅ |
| 2️⃣ | Baseline multimodal (CNN + XGBoost) | ✅ |
| 3️⃣ | API REST + métricas Prometheus | ✅ |
| 4️⃣ | Stack Docker (Prometheus + Grafana) | ✅ |
| 5️⃣ | Evidently Drift + Alertas Grafana | 🔄 En progreso |
| 6️⃣ | Streamlit revisión manual | 🔜 |
| 7️⃣ | Retraining y versionado MLflow | 🔜 |

---

## 🧾 Créditos

Proyecto desarrollado por **Pedro Rubio**  
**Data / ML Analyst & Fraud Detection Developer**  
📧 srdelosdatos@gmail.com · 🌐 [github.com/srdelosdatos](https://github.com/srdelosdatos) · [LinkedIn: srdelosdatos](https://linkedin.com/in/srdelosdatos)

---

### 🧩 Licencia
Este proyecto utiliza datos sintéticos y código abierto bajo licencia **MIT**.  
Se prohíbe el uso con datos reales sin cumplir con la normativa de protección de datos aplicable.
