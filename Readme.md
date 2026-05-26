# 🌦️ WRF-Catalunya Meteorological Dashboard

Benvingut al repositori de presentació de **WRF-Catalunya**.

🔗 **[Visita l'aplicació web aquí: wrf-catalunya.streamlit.app](https://wrf-catalunya.streamlit.app/)**

## 📖 Sobre el projecte
Aquest dashboard interactiu permet visualitzar i analitzar dades meteorològiques d'alta resolució per a Catalunya, processant sortides numèriques del model WRF (Weather Research and Forecasting).

## 🛠️ Stack Tecnològic
* **Frontend:** Streamlit
* **Processament de dades:** Python (Pandas, Xarray, etc.)
* **Modelització:** WRF V4.6.1

## ⚙️ Configuració del Model (WRF)

### 🌍 Dominis i Estratègia de nesting
* **Resolució Espacial:** 3 dominis(27 km ➔ 9 km ➔ 3 km).
* **Execució:** Els dominis **d01 (27 km)** i **d02 (9 km)** s'executen de manera simultània. El domini d'alta resolució **d03** (**3 km**) s'alimenta posteriorment de les sortides del d02 utilitzant la `ndown` (1-way nesting).

### 🔬 Parametritzacions Físiques
| Esquema Físic | Configuració | Aplicació |
| :--- | :--- | :--- |
| **Microfísica** | WDM6 | d01, d02, d03 |
| **Cúmuls** | Grell-Freitas | d01, d02 *(Apagada explícitament en d03)* |
| **Capa Límit (PBL)** | YSU | d01, d02, d03 |
| **Radiació (Ona Llarga y Curta)** | RRTMG scheme | d01, d02, d03 |
| **Física de Superfície** | Unified Noah land-surface model | d01, d02, d03 |
| **Capa Superficial** | Revised MM5 Monin-Obukhov scheme (Jimenez, renamed in v3.6) | d01, d02, d03 |
| **Llamps** | Predicting the potential for lightning activity (based on Yair et al., 2010) | d03 |
### 🔄 Cicle Operatiu i Inicialització
* **Cicle Diari::** El model s'executa diàriament a les **18 UTC**.
* **Condicions Inicials i de Contorno:** S'inicialitza a partir de l'anàlisi de les 12 UTC del model GFS a 0.5 graus** de resolució. Les 6 primeres hores es consideren com a Spinup.

### 🧲 Asimilació de Dades (Grid Nudging)
Per a estabilitzar i guiar la simulació en les seves fases inicials, s'aplica *grid nudging* (FDDA) al domini exterior (**d01**) cap a les condicions del model GFS. Aquest forçament es manté actiu únicament durant les **6 primeres hores** de simulació. A més, s'exclou d'aquest procés a la Capa Límit Planetària (PBL) per a permetre que el model resolgui localment la física de superfície sense interferències.

**Extracte del namelist per a FDDA:**
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
```
### 📊 Diagnòstics Avançats i Temps Sever (Domini d03)

Per a l'extracció de variables d'impacte en el domini de major resolució (3 km), s'han activat els mòduls de diagnòstic de la **AFWA** i la parametrització de descàrregues elèctriques per al càlcul del *Índex de Potencial de Llamps (LPI)**.

**Configuració de Diagnòstics AFWA:**
```fortran
&afwa
   afwa_diag_opt   = 1,
   afwa_buoy_opt   = 1,
   afwa_severe_opt = 1,
   afwa_vis_opt    = 1,
/
```



# 🇺🇸 EN: 🌦️ WRF-Catalunya Meteorological Dashboard

Welcome to the presentation repository for **WRF-Catalunya**.

🔗 **[Visit the web application here: wrf-catalunya.streamlit.app](https://wrf-catalunya.streamlit.app/)**

## 📖 About the Project
This interactive dashboard allows users to visualize and analyze high-resolution meteorological data for Catalonia by processing numerical outputs from the WRF (Weather Research and Forecasting) model.

## 🛠️ Technology Stack
* **Frontend:** Streamlit
* **Data Processing:** Python (Pandas, Xarray, etc.)
* **Modeling:** WRF V4.6.1

## ⚙️ Model Configuration (WRF)

### 🌍 Domains and Nesting Strategy
* **Spatial Resolution:** 3 domains (27 km ➔ 9 km ➔ 3 km).
* **Execution:** Domains **d01 (27 km)** and **d02 (9 km)** run simultaneously. The high-resolution domain **d03 (3 km)** is subsequently driven from d02 outputs using `ndown` (1-way nesting).

### 🔬 Physical Parameterizations

| Physical Scheme | Configuration | Application |
| :--- | :--- | :--- |
| **Microphysics** | WDM6 | d01, d02, d03 |
| **Cumulus** | Grell-Freitas | d01, d02 *(Explicitly disabled in d03)* |
| **Planetary Boundary Layer (PBL)** | YSU | d01, d02, d03 |
| **Radiation (Longwave & Shortwave)** | RRTMG scheme | d01, d02, d03 |
| **Land Surface Physics** | Unified Noah land-surface model | d01, d02, d03 |
| **Surface Layer** | Revised MM5 Monin-Obukhov scheme (Jimenez, renamed in v3.6) | d01, d02, d03 |
| **Lightning** | Predicting the potential for lightning activity (based on Yair et al., 2010) | d03 |

### 🔄 Operational Cycle and Initialization
* **Daily Cycle:** The model runs daily at **18 UTC**.
* **Initial and Boundary Conditions:** Initialized from the 12 UTC analysis of the GFS model output at 0.5° resolution. The first 6 hours are considered spin-up

### 🧲 Data Assimilation (Grid Nudging)
To stabilize and guide the simulation during its initial stages, *grid nudging* (FDDA) is applied to the outer domain (**d01**) toward GFS model conditions. This forcing remains active only during the **first 6 hours** of the simulation. Additionally, the Planetary Boundary Layer (PBL) is excluded from this process to allow the model to locally resolve surface physics without interference.

**FDDA namelist excerpt:**
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
```
### 📊 Advanced Diagnostics and Severe Weather (d03 Domain)

To extract impact-related variables in the highest-resolution domain (3 km), the **AFWA** diagnostic modules and the lightning parameterization for calculating the *Lightning Potential Index (LPI)* have been enabled.

**AFWA Diagnostics Configuration:**
```fortran
&afwa
   afwa_diag_opt   = 1,
   afwa_buoy_opt   = 1,
   afwa_severe_opt = 1,
   afwa_vis_opt    = 1,
/
