# 📊 Dashboard Ejecutivo de Análisis de Ventas

> Pipeline ETL completo · Modelo estrella · DAX avanzado · Inteligencia de tiempo

---

## 🛠️ Stack Tecnológico

| Herramienta | Uso |
|-------------|-----|
| **Power BI Desktop** | Visualización y dashboard interactivo |
| **Power Query (M)** | ETL y transformación de datos |
| **DAX** | Medidas, KPIs e inteligencia de tiempo |
| **Excel** | Fuente de datos origin |

---

## 🎯 Objetivo

Desarrollo de un **dashboard ejecutivo interactivo** a partir de un dataset con múltiples inconsistencias de calidad de datos, aplicando un flujo completo de ETL, modelado dimensional y análisis con DAX.

---

## ⚙️ Proceso ETL & Modelado

### 🔄 Limpieza de Datos — Power Query

Se procesaron **5 tablas** con distintos tipos de inconsistencias:

| Tabla | Transformaciones aplicadas |
|-------|---------------------------|
| `FactVentas` | • Corrección de valores con símbolos de moneda<br>• Normalización de fechas con formatos mixtos<br>• Normalización de texto y categorías |              
| `Cliente` | Normalización de texto y categorías |
| `Producto` | Estandarización de categorías |
| `Canal` | Limpieza de registros duplicados |
| `MetasVentas` | Estandarización de categorías |

### 🌟 Modelo de Datos

- Arquitectura **modelo estrella** con tabla calendario personalizada
- Relaciones 1:N entre tabla de hechos y dimensiones
- Tabla calendario generada con rango dinámico de fechas

### 📐 Medidas DAX Destacadas

- Inteligencia de tiempo (YTD, MoM, variaciones)
- KPIs de cumplimiento de metas por vendedor y canal
- **Insight dinámico** con texto narrativo automático generado por DAX

---

## 🖥️ Dashboard

Navegación entre páginas mediante **botones interactivos**.

### Página 1 — Resumen Ejecutivo
<img src="screenshots/pagina1_resumen_ejecutivo.png" width="700"/>

### Página 2 — Desempeño
<img src="screenshots/pagina2_desempeno.png" width="700"/>

### Página 3 — Detalle y Ranking
<img src="screenshots/pagina3_detalle_ranking.png" width="700"/>

---

## 📈 Conclusiones & Hallazgos Clave

| Métrica | Resultado |
|---------|-----------|
| 🏆 Cumplimiento anual | **120.29%** — superó la meta |
| 🥇 Canal líder | **Online** con el 37.71% de las ventas |
| 💻 Producto estrella | **Laptop Pro 14** |
| 📉 Período más débil | **Q1 2024** |
| 🚀 Pico máximo | **Junio — $177K** |

---

## 📁 Estructura del Repositorio

```
📦 dashboard-ventas-powerbi
├── 📊 Dashboard_Ventas.pbix
├── 📂 screenshots/
│   ├── pagina1_resumen_ejecutivo.png
│   ├── pagina2_desempeno.png
│   └── pagina3_detalle_ranking.png
└── 📄 README.md
```

---

*Proyecto desarrollado como parte del portafolio de análisis de datos.*
