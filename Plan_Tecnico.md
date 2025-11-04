# 📆 Plan Técnico de Proyecto — Document Fraud Sentinel (Versión Medallion)

> **Coordinación y Dirección Técnica:** Pedro Rubio  
> Proyecto integral de detección de fraude documental en procesos de onboarding bancario digital.

---

## 🎯 Propósito

Desarrollar un sistema completo de detección de fraude en documentos de identidad (DNI y pasaportes), estructurado bajo un enfoque **Data Lakehouse con arquitectura Medallion (Bronze → Silver → Gold)**.  
El proyecto integra visión computacional, machine learning multimodal, y monitoreo operativo continuo.

Duración estimada: **12 semanas**  
Equipo base: Data Engineering, Data Science, MLOps y Backend.

---

## 🏗️ Estructura General del Proyecto

**Capas principales:**
- 🥉 **Bronze:** ingesta cruda y almacenamiento original.
- 🥈 **Silver:** datos curados, enriquecidos y con features forenses.
- 🥇 **Gold:** dataset analítico y ML-ready.
- 🧠 **Feature Store:** vistas online/offline para scoring en tiempo real.

**Infraestructura base:**  
Google Colab (desarrollo exploratorio) + Airflow (orquestación) + FastAPI (serving) + Prometheus/Grafana (monitoreo).

---

## 🧱 Fase 1 — Análisis y Diseño (Semanas 1–2)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 1 | Definición de requerimientos técnicos y de negocio. Diseño del flujo de onboarding y arquitectura Medallion. | Documento de diseño y diagrama de arquitectura. |
| 2 | Identificación de fuentes de datos, definición de esquemas para Bronze y validación de dataset sintético. | Esquemas de tablas Bronze + dataset inicial (sintético). |

🔹 **Resultado esperado:** arquitectura validada, estructura inicial del Lakehouse lista.

---

## 🥉 Fase 2 — Capa Bronze: Ingesta de Datos (Semanas 3–4)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 3 | Desarrollo de jobs de ingesta (JSON, imágenes, OCR, metadatos). | Job Airflow `bronze_ingest.py` con persistencia en storage. |
| 3 | Configuración de almacenamiento (ruta de imágenes y logs). | Estructura `/data/bronze/` organizada. |
| 4 | Validación y control de calidad inicial (schema + logs). | Data dictionary y scripts de validación Great Expectations. |

🔹 **Resultado esperado:** capa Bronze funcional con histórico completo de eventos.

---

## 🥈 Fase 3 — Capa Silver: Curado y Enriquecimiento (Semanas 5–6)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 5 | Limpieza y normalización de campos. | Script `silver_enrich.py` (parseo JSON, normalización). |
| 5 | Cálculo de features visuales (ELA, blur, pHash) y contextuales. | Job de extracción forense y contextual. |
| 6 | Integración con OCR y MRZ, detección de inconsistencias. | Tabla `doc_text_features_silver`. |
| 6 | Consolidación de todas las fuentes en tablas Silver unificadas. | DataFrame unificado `doc_enriched_silver.parquet`. |

🔹 **Resultado esperado:** capa Silver estable y lista para consumo analítico y modelado.

---

## 🥇 Fase 4 — Capa Gold: Dataset Analítico y ML-Ready (Semanas 7–8)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 7 | Integración de todas las tablas Silver + etiquetas. | Tabla `fraud_doc_onboarding_gold`. |
| 7 | Generación de variables derivadas y feature importance inicial. | Dataset final curado (`features_merged.csv`). |
| 8 | Validación de integridad (joins, nulls, duplicados). | Scripts Great Expectations. |
| 8 | Publicación en Feature Store (Feast). | `fs_doc_features_v1`, `fs_user_risk_v1`, `fs_device_risk_v1`. |

🔹 **Resultado esperado:** dataset Gold completo y sincronizado con la Feature Store.

---

## 🧠 Fase 5 — Modelado Multimodal (Semanas 9–10)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 9 | Entrenamiento de modelo CNN (imágenes) y XGBoost (tabular). | Modelos `.onnx` y `.pkl` registrados en MLflow. |
| 9 | Implementación de fusión tardía (logistic regression calibrada). | Modelo final `fraud_fusion_v1`. |
| 10 | Evaluación y explicabilidad (Grad-CAM + SHAP). | Reporte de interpretabilidad y métricas. |
| 10 | Validación del modelo en dataset Gold. | Recall > 85%, FPR < 5%. |

🔹 **Resultado esperado:** modelo final validado y registrado en MLflow.

---

## ⚙️ Fase 6 — Despliegue y API de Scoring (Semana 11)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 11 | Desarrollo de API FastAPI con endpoints `/score` y `/metrics`. | `app/main.py` operativo. |
| 11 | Integración del modelo ONNX en inferencia. | Pipeline completo de scoring con Feature Store. |
| 11 | Despliegue local con Docker Compose. | Stack `FastAPI + Prometheus + Grafana`. |
| 11 | Pruebas unitarias e integración (Pytest). | Tests funcionales aprobados. |

🔹 **Resultado esperado:** servicio en funcionamiento con métricas expuestas.

---

## 📊 Fase 7 — Monitoreo, Drift y Observabilidad (Semana 12)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 12 | Configuración de Prometheus y dashboards Grafana. | Paneles de latencia, drift y score. |
| 12 | Integración de Evidently para monitoreo de drift y PSI. | Jobs `monitor_drift.py` conectados a Prometheus. |
| 12 | Configuración de alertas automáticas (Grafana/Slack). | Alertas “latencia > 2s” y “PSI > 0.2”. |

🔹 **Resultado esperado:** monitoreo continuo de modelo y datos en producción.

---

## 🧩 Fase 8 — Streamlit y Validación Operativa (Post 12 semanas)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 13 | Desarrollo de app Streamlit para revisión manual. | `app/streamlit_review.py`. |
| 13 | Integración con endpoint `/score`. | Consola de analistas con Grad-CAM y SHAP visibles. |
| 14 | Pruebas con casos reales/sintéticos y feedback del equipo de fraude. | Informe de usabilidad y performance. |

🔹 **Resultado esperado:** interfaz operativa interna para revisión de casos sospechosos.

---

## 📈 KPIs Técnicos y de Negocio

| Indicador | Meta | Descripción |
|------------|------|-------------|
| Recall (fraude) | > 85% | Detección efectiva de fraude documental |
| FPR | < 5% | Mínimo rechazo a clientes legítimos |
| Latencia (P95) | < 2s | Respuesta rápida para onboarding |
| PSI (drift) | < 0.2 | Estabilidad del modelo |
| % reducción revisiones manuales | -30% | Eficiencia operativa |
| Disponibilidad del servicio | 99.5% | Uptime del scoring API |

---

## 🧭 Gobernanza y Coordinación

| Frecuencia | Reunión | Objetivo |
|-------------|----------|----------|
| Diario | Daily standup (10–15 min) | Seguimiento técnico |
| Semanal | Sprint review | Validar avances e hitos |
| Quincenal | Retro + planificación | Ajuste de backlog y prioridades |
| Mensual | Comité de riesgo | Evaluar métricas del modelo |

---

## 🧾 Comentario final

El proyecto combina **ingeniería de datos, machine learning y monitoreo operativo** dentro de un mismo flujo controlado.  
El uso de la arquitectura **Medallion** permite mantener trazabilidad completa, facilitando auditorías y retraining continuo sin perder consistencia.

---

**Pedro Rubio**  
_Data & ML Analyst — Coordinador Técnico del Proyecto_  
📧 srdelosdatos@gmail.com · 🌐 [github.com/srdelosdatos](https://github.com/srdelosdatos)
