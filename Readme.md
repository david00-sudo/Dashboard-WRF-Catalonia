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
| **Radiación (Onda Larga y Corta)** | RRTMG scheme | d01, d02, d03 |
| **Física de Superficie** | Unified Noah land-surface model | d01, d02, d03 |
| **Capa Superficial** | Revised MM5 Monin-Obukhov scheme (Jimenez, renamed in v3.6) | d01, d02, d03 |
| **Rayos** | Predicting the potential for lightning activity (based on Yair et al., 2010) | d03 |
### 🔄 Ciclo Operativo e Inicialización
* **Ciclo Diario:** El modelo se ejecuta diariamente a las **18 UTC**.
* **Condiciones Iniciales y de Contorno:** Se inicializa a partir de la hora +6 de pronóstico de la salida de las **12 UTC del modelo GFS a 0.5 grados** de resolución (coincidiendo así con el inicio de la simulación a las 18 UTC).

### 🧲 Asimilación de Datos (Grid Nudging)
Para estabilizar y guiar la simulación en sus fases iniciales, se aplica *grid nudging* (FDDA) al dominio exterior (**d01**) hacia las condiciones del modelo GFS. Este forzamiento se mantiene activo únicamente durante las **6 primeras horas** de simulación. Además, se excluye de este proceso a la Capa Límite Planetaria (PBL) para permitir que el modelo resuelva localmente la física de superficie sin interferencias.

**Extracto del namelist para FDDA:**
```fortran
&fdda 
   grid_fdda            = 1 
   gfdda_inname         = "wrffdda_d<domain>" 
   gfdda_interval_m     = 180 
   gfdda_end_h          = 6 
   io_form_gfdda        = 2 
   fgdt                 = 0 
   if_no_pbl_nudging_t  = 1 
   if_no_pbl_nudging_q  = 1 
   if_no_pbl_nudging_uv = 1 
/
---

### 📊 Diagnósticos Avanzados y Tiempo Severo (Dominio d03)

Para la extracción de variables de impacto en el dominio de mayor resolución (3 km), se han activado los módulos de diagnóstico de la **AFWA** y la parametrización de descargas eléctricas para el cálculo del **Índice de Potencial de Rayos (LPI)**.

**Configuración de Diagnósticos AFWA:**
```fortran
&afwa
   afwa_diag_opt   = 1,
   afwa_buoy_opt   = 1,
   afwa_severe_opt = 1,
   afwa_vis_opt    = 1,
/
