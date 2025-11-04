# 📆 Plan Técnico de Proyecto — Document Fraud Sentinel

> **Coordinación y Dirección Técnica: Pedro Rubio**  
> Proyecto de detección de fraude documental en procesos de onboarding bancario digital.

---

## 🎯 Propósito

Implementar un sistema completo de detección de fraude en documentos de identidad (DNI y pasaportes), combinando análisis visual, metadatos contextuales y monitoreo en tiempo real.  
El objetivo es reducir los intentos de fraude sin afectar la experiencia del cliente legítimo.

Duración estimada: **12 semanas**  
Equipo base: Data Science, Computer Vision, Backend y MLOps.

---

## 🧱 Fase 1 — Análisis y Diseño (Semanas 1–2)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 1 | Definición de requerimientos técnicos y de negocio. Diseño inicial del flujo de onboarding y arquitectura general. | Documento de requerimientos (FRD) + diagrama de arquitectura. |
| 2 | Revisión de fuentes de datos, creación de dataset inicial con información sintética y validación de etiquetas. | Dataset base para pruebas y validaciones iniciales. |

🔹 **Resultado esperado:** arquitectura validada, dataset listo para ingeniería de características.

---

## ⚙️ Fase 2 — Ingesta de Datos y Feature Engineering (Semanas 3–4)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 3 | Desarrollo del pipeline de ingesta: limpieza, OCR, EXIF, validación de formatos. | ETL inicial (`preprocessing.py`) con funciones base. |
| 4 | Extracción de características visuales (blur, ELA, pHash) y metadatos contextuales (IP, geolocalización, dispositivo). | Dataset enriquecido y validado. |

🔹 **Resultado esperado:** pipeline estable y features listos para entrenamiento.

---

## 🧠 Fase 3 — Modelado Multimodal (Semanas 5–6)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 5 | Entrenamiento de modelo CNN (MobileNet / EfficientNet) para features visuales. | Modelo de imagen exportado (`.pt` / `.onnx`). |
| 6 | Entrenamiento de modelo tabular (XGBoost) y fusión de resultados con logistic regression calibrada. | Modelo multimodal calibrado y documentado (MLflow). |

🔹 **Resultado esperado:** modelo final con recall > 0.85 y FPR < 0.05.

---

## 🧩 Fase 4 — API y Despliegue Inicial (Semanas 7–8)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 7 | Desarrollo del servicio FastAPI para inferencia en tiempo real. | Endpoint `/score` y `/metrics` operativo. |
| 8 | Despliegue con Docker Compose (app + Prometheus + Grafana). | API funcional y monitoreada localmente. |

🔹 **Resultado esperado:** API en funcionamiento con métricas expuestas.

---

## 📊 Fase 5 — Monitoreo y Observabilidad (Semanas 9–10)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 9 | Configuración de Prometheus y creación de dashboards en Grafana. | `prometheus.yml` y paneles de monitoreo. |
| 10 | Integración de Evidently para cálculo de drift y métricas de estabilidad. | Alertas activas para PSI y latencia. |

🔹 **Resultado esperado:** sistema de monitoreo activo y alertas automáticas.

---

## 👁️ Fase 6 — Consola de Revisión Manual (Semana 11)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 11 | Desarrollo de una app interna en Streamlit para revisar casos de fraude. | UI interna conectada al endpoint de scoring. |

🔹 **Resultado esperado:** herramienta operativa para analistas de fraude.

---

## 🔄 Fase 7 — MLOps y Entrega Final (Semana 12)

| Semana | Actividades principales | Entregables |
|--------|--------------------------|--------------|
| 12 | Configuración de MLflow para versionado, y pipeline de retraining con Airflow o Prefect. | Reentrenamiento automatizado + documentación final. |

🔹 **Resultado esperado:** sistema completo documentado y versionado.

---

## 📈 KPIs del Proyecto

| Indicador | Meta | Descripción |
|------------|------|--------------|
| Recall de fraude | > 85% | Capacidad de detectar intentos fraudulentos reales. |
| FPR | < 5% | Minimizar rechazos a usuarios legítimos. |
| Latencia promedio | < 2s | Tiempo de respuesta del servicio de scoring. |
| PSI (Drift) | < 0.2 | Estabilidad del modelo en producción. |
| % reducción revisiones manuales | -30% | Impacto operativo. |

---

## 🧭 Coordinación y Comunicación

- **Reuniones diarias breves** para seguimiento técnico (10–15 min).  
- **Revisión semanal de avances** (modelo, métricas, infraestructura).  
- **Retroquincenal** para ajustar backlog y prioridades.  
- **Comité mensual** para evaluar métricas de riesgo y efectividad del sistema.

---

## ✍️ Comentario final

La prioridad es mantener un equilibrio entre **precisión, interpretabilidad y cumplimiento normativo**.  
El modelo debe ser transparente, auditable y compatible con un flujo real de onboarding bancario.  
Cada entrega intermedia se probará con casos sintéticos y reportes claros de resultados.

---

**Pedro Rubio**  
_Data & ML Analyst — Coordinador Técnico del Proyecto_  
📧 srdelosdatos@gmail.com · 🌐 [github.com/srdelosdatos](https://github.com/srdelosdatos)
