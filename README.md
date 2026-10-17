# multistage-investment-model

Modelo de inversión multietapa con datos de mercado, resuelto con L-shaped anidado.

**Práctica 2 – Optimización Estocástica** 
Profesor: Diego Fonseca
Integrantes: Alejandro Vásquez y Manuela Valencia

## Descripción

Una fundación dispone de USD 100 000 a finales de diciembre de 2025 y necesita USD 106 000 a finales de septiembre de 2026 para comprar equipos de laboratorio. Puede invertir en tres ETF (SPY, LQD y GLD) y reorganizar el portafolio al terminar marzo y junio, cuando ya conoce cómo se comportó el trimestre anterior. Cada dólar de déficit frente a la meta se penaliza con 4 unidades y cada dólar de exceso se premia con 1.

El proyecto construye un árbol de escenarios con datos trimestrales de 2010 a 2025, lo resuelve con L-shaped anidado, compara la política obtenida con la del modelo de valor esperado y, por último, aplica ambas a lo que realmente ocurrió entre enero y septiembre de 2026.

## Estructura del repositorio

```
multistage-investment-model/
├── README.md
├── requirements.txt              # dependencias (rangos de versiones)
├── requirements-lock.txt         # versiones exactas usadas en la ejecución final
├── notebooks/
│   ├── 00_data_download.ipynb
│   ├── 01_calibration_states.ipynb
│   ├── 02_scenario_tree.ipynb
│   ├── 03_modelo_y_equivalente_reducido.ipynb
│   ├── 04_lshaped_anidado.ipynb
│   ├── 05_politica_ev_vss.ipynb
│   └── 06_validacion_2026.ipynb
├── data/
│   ├── raw/                      # descargas diarias crudas (respaldo; los notebooks no las leen)
│   ├── calibration/              # cierres trimestrales 2009–2025 y su metadata
│   └── validation/               # cierres trimestrales dic-2025 a sep-2026 y su metadata
└── results/
    ├── parameters/               # probabilidades y factores de calibración
    ├── tree/                     # los 585 nodos del árbol
    ├── reduced/                  # instancia reducida y su equivalente determinista
    ├── full_de/                  # equivalente determinista completo (solo comparación)
    ├── lshaped/                  # convergencia, cortes y resumen del algoritmo
    ├── policy/                   # política estocástica y regla EV definitivas
    ├── tables/                   # tablas para el informe
    ├── figures/                  # figuras para el informe
    └── validation/               # resultados de la aplicación a 2026
```

## Qué hace cada notebook

| Notebook | Contenido |
|---|---|
| `00_data_download` | Descarga de Yahoo Finance los cierres ajustados diarios de SPY, LQD y GLD, conserva el último cierre de cada trimestre y verifica las fechas contra el calendario oficial de la bolsa de Nueva York. |
| `01_calibration_states` | Calcula los 64 rendimientos trimestrales, sus estadísticas, los ocho estados conjuntos con sus probabilidades y factores representativos, y guarda los parámetros de calibración. |
| `02_scenario_tree` | Construye el árbol de 1, 8, 64 y 512 nodos con sus historias, padres, hijos y probabilidades. |
| `03_modelo_y_equivalente_reducido` | Plantea el modelo nodal, muestra las matrices de enlace y el problema terminal con su dual, y resuelve el equivalente determinista de la instancia reducida de dos estados. |
| `04_lshaped_anidado` | Implementa L-shaped anidado, lo verifica en la instancia reducida, resuelve el caso completo y guarda la política óptima. |
| `05_politica_ev_vss` | Analiza la política obtenida, resuelve el modelo de valor esperado y calcula EEV y VSS. |
| `06_validacion_2026` | Aplica ambas políticas, trimestre a trimestre, a la trayectoria real de 2026. |

## Instalación

Se requiere Python 3.11 o superior. La ejecución final de los notebooks se hizo con Python 3.13.16. Desde la raíz del repositorio:

**Mac / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name multistage-investment-model --display-name "Python (multistage-investment-model)"
```

**Windows (PowerShell)**

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name multistage-investment-model --display-name "Python (multistage-investment-model)"
```

Si PowerShell bloquea la activación del entorno, ejecuten antes `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

Para reproducir exactamente las versiones usadas en el trabajo, instalen `requirements-lock.txt` en lugar de `requirements.txt`.

## Ejecución

Abran los notebooks con el kernel **Python (multistage-investment-model)** y ejecútenlos **en orden, del 00 al 06**, cada uno con el kernel reiniciado. Cada notebook lee los archivos que generó el anterior y se detiene si alguno falta o fue modificado.

**Componentes aleatorios:** no hay. Ningún paso usa números aleatorios ni muestreo, así que no se requiere semilla: con los mismos archivos de datos, la ejecución produce los mismos resultados (salvo los tiempos de cómputo y diferencias de redondeo del solver del orden de 1e-8).


## Datos

**Fuente:** Yahoo Finance, campo *Adj Close* (cierre ajustado), frecuencia diaria, descargado con la librería `yfinance` usando `auto_adjust=False` y `actions=False`. Así se conserva el cierre ajustado tal como lo publica Yahoo y no se suman de nuevo los dividendos y cupones que ya incluye.

**Ventanas solicitadas:** Calibración del 1 de diciembre de 2009 al 31 de diciembre de 2025 y validación del 1 de diciembre de 2025 al 30 de septiembre de 2026. Como la herramienta excluye la fecha final, se pidió hasta el 1 de enero de 2026 y el 1 de octubre de 2026, respectivamente.

**Archivos que usan los notebooks:** los cierres trimestrales.

- `data/calibration/quarterly_adjclose_2009_2025.csv`: 65 cierres por activo, de diciembre de 2009 a diciembre de 2025.
- `data/validation/quarterly_adjclose_2025_2026.csv`: 4 cierres por activo, de diciembre de 2025 a septiembre de 2026.

Junto a cada archivo, `metadata.json` registra la fuente, la configuración de descarga, la fecha y hora de consulta y su huella digital (SHA-256). La carpeta `data/raw/` conserva, como respaldo, los precios diarios descargados de los que se extrajeron esos cierres; los notebooks no la leen.

**Fecha de consulta:** ambos bloques se descargaron el 4 de octubre de 2026, después del cierre definitivo de septiembre. Calibración: 2026-10-04 16:26:02 (hora de Bogotá). Validación: 2026-10-04 16:26:16 (hora de Bogotá). La descarga se hizo con Python 3.14.6 y yfinance 1.7.0, como registran los `metadata.json`.

## Reproducibilidad

**Los datos no se vuelven a descargar:** Yahoo recalcula toda la serie de cierres ajustados cada vez que un fondo hace un pago, de modo que una descarga posterior puede dar niveles distintos. Por eso el notebook 00 reutiliza los archivos trimestrales ya guardados y solo los verifica. Para descargarlos de nuevo hay que poner `FORCE_DOWNLOAD = True` en su configuración.

**Separación temporal:** El modelo se construye como si la decisión se tomara a finales de diciembre de 2025. Los notebooks 01 a 05 usan únicamente datos de calibración. Tanto la política estocástica como la regla del modelo de valor esperado se guardan con su huella digital antes de la validación, y el notebook 06 comprueba esas huellas antes de abrir los datos de 2026. `results/validation/validation_summary.json` registra las huellas verificadas, que coinciden con las de `results/policy/`. Ningún parámetro se reestima ni se vuelve a optimizar con información de 2026.

## Solver

Todos los problemas lineales se resuelven con HiGHS a través de `scipy.optimize.linprog(method="highs")`. Los multiplicadores duales se obtienen de `res.eqlin.marginals`, y su signo se verifica en los notebooks 03 y 04.
