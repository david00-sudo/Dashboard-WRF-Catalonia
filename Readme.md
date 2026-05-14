# 🌦️ WRF-Catalunya Meteorological Dashboard

Bienvenido al repositorio de presentación de **WRF-Catalunya**.

🔗 **[Visita la aplicación web aquí: wrf-catalunya.streamlit.app](https://wrf-catalunya.streamlit.app/)**

## 📖 Sobre el proyecto
Este dashboard interactivo permite visualizar y analizar datos meteorológicos de alta resolución para Catalunya, procesando salidas numéricas del modelo WRF (Weather Research and Forecasting). 


## 🛠️ Stack Tecnológico
* **Frontend:** Streamlit
* **Procesamiento de datos:** Python (Pandas, Xarray, etc.)
* **Modelización:** WRF V4.6.1

## ⚙️ Configuración del Modelo (WRF)

### 🌍 Dominios y Estrategia de Anidamiento
* **Resolución Espacial:** 3 dominios anidados (27 km ➔ 9 km ➔ 3 km).
* **Ejecución:** Los dominios **d01 (27 km)** y **d02 (9 km)** se ejecutan de forma simultánea. El dominio de alta resolución **d03 (3 km)** se alimenta posteriormente de las salidas del d02 utilizando la `ndown` (1-way nesting).

### 🔬 Parametrizaciones Físicas Principales
| Esquema Físico | Configuración | Aplicación |
| :--- | :--- | :--- |
| **Microfísica** | WDM6 | d01, d02, d03 |
| **Cúmulos** | Grell-Freitas | d01, d02 *(Apagado explícitamente en d03)* |
| **Capa Límite (PBL)** | YSU | d01, d02, d03 |

---

### 📊 Diagnósticos Avanzados y Tiempo Severo (Dominio d03)

Para la extracción de variables de impacto en el dominio de mayor resolución (3 km), se han activado los módulos de diagnóstico de la **AFWA** y la parametrización de descargas eléctricas para el cálculo del **Índice de Potencial de Rayos (LPI)**.

**Configuración de Diagnósticos AFWA:**
```
&afwa
   afwa_diag_opt   = 1,
   afwa_buoy_opt   = 1,
   afwa_severe_opt = 1,
   afwa_vis_opt    = 1,
/

