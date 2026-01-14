# 1. Resumen del Proyecto

La mayoría de los sistemas de trading algorítmico fracasan no por falta de indicadores, sino por el "Backtest Overfitting". Este proyecto aborda el desafío de la minería de estrategias en mercados financieros (Forex) desde una perspectiva rigurosa, diseñada para diferenciar la señal estadística del ruido aleatorio.

## El núcleo:

Para mitigar el sesgo de selección y el p-hacking en la búsqueda de estrategias, el pipeline se apoya en tres pilares de computación de alto rendimiento y estadística avanzada:

### Motores de Ejecución Ultra-Rápidos (Numba JIT):
El cálculo de indicadores y el backtesting de miles de combinaciones se realiza mediante funciones decoradas con `@njit`. Esto permite procesar millones de velas en segundos, traduciendo la lógica de Python a código máquina eficiente, esencial para la minería masiva de parámetros.

### Validación Estadística Rigurosa (DSR & FDR):
En lugar de confiar en un simple Sharpe Ratio, el sistema implementa:
- **Deflated Sharpe Ratio (DSR):** Ajusta el rendimiento esperado en función del número de pruebas realizadas, corrigiendo la inflación del Sharpe por minería masiva.
- **False Discovery Rate (FDR):** Aplica el ajuste de Benjamini-Hochberg a los p-values para controlar la probabilidad de aceptar estrategias que son fruto del azar.

### Combinatorial Purged Cross-Validation (CPCV):
A diferencia del K-Fold tradicional, la CPCV permite simular miles de caminos alternativos de la historia del mercado, manteniendo la integridad temporal y eliminando el "leakage" de datos, proporcionando una estimación realista del rendimiento fuera de la muestra (OOS).

## Evaluación

Para este proyecto, el valor reside en el pipeline de validación: tener una infraestructura que rechace sistemáticamente lo que no es robusto es más valioso que un backtest optimista. El sistema busca identificar un "Alpha" real, aunque sea una tarea de baja probabilidad (DSR extremadamente bajos en la mayoría de las pruebas).


# 2. Requisitos, Ejecución y Artefactos

## Datos necesarios (inputs)

Para ejecutar el proyecto se requieren:

- Todos los archivos de bid y ask de los 10 pares analizados (ubicados en la carpeta `data/`)
- Datos de precio horarios por separado para cada par (AUDJPY, EURUSD, etc.) con columnas `Local time`, `Open`, `High`, `Low`, `Close`.

> Nota: Datos estan extraidos de Dukascopy.

## Artefactos generados (outputs)

Al ejecutar el pipeline, se generan automáticamente archivos de soporte, bases de datos de resultados y análisis estadísticos:

### Scripts de soporte
- `utils4alphas_new.py`
- `utils4alphas_old.py`

### Resultados de minería (Dumps)
- `all_raw_new.csv`
- `all_raw_old.csv`
- *Si se encontrasen alphas, se crearían otros dos archivos adicionales con los alphas encontrados.*

### Análisis de indicadores
- `Summary_New_Indicators.csv`
- `Summary_Old_Indicators.csv`

### Otros
- Visualizaciones de Correlación: Plots que analizan la relación entre la performance del alpha y su correlación con el activo objetivo (Target Pair).


# 3. Experimentos (Estrategias "Old" vs. "New")

El framework permite comparar dos paradigmas de análisis técnico para determinar cuál ofrece una ventaja competitiva real:

## Paradigma Clásico (Old)

Se evaluaron indicadores de momentum y osciladores tradicionales:
- **Indicadores:** RSI, Williams %R, Estocástico, MACD, PPO y Bandas de Bollinger entre otros.
- **Enfoque:** Reversión a la media en niveles fijos, cruces de medias móviles seguimiento de tendencia etc.

## Paradigma de Complejidad y Entropía (New)

Se implementaron algoritmos basados en la microestructura del precio y análisis fractal:
- **Hurst Exponent (DFA):** Para medir la persistencia o anti-persistencia de la tendencia.
- **Variance Ratio (VR-MR):** Para detectar regímenes de mean-reversion estadística.
- **Empirical Trend Detection (ETD):** Basado en la descomposición de modos intrínsecos para separar tendencia de ruido.
- **Directional Probability Volatility (DPV):** Para capturar asimetrías en la volatilidad de los retornos.


# 4. Conclusiones y Futuro

El proyecto concluye que encontrar un "Alpha" real es una tarea de baja probabilidad. Sin embargo, el valor reside en el pipeline de validación: tener una infraestructura que rechace sistemáticamente lo que no es robusto es más valioso que un backtest optimista.

## Próximos pasos:

- **Evolución a ICEEMDAN:** Sustituir el método EMD tradicional por **ICEEMDAN** (Improved Complete Ensemble EMD with Adaptive Noise) para el motor de **DynETD**, logrando una descomposición de señal más estable y con menor ruido residual.
- **Sinergia de Indicadores Híbridos:** Implementar un sistema de decisión compuesto, por ejemplo; donde el **VR-MR** determine el régimen de mercado (tendencia vs. reversión) y el **DPV** actúe como disparador para capturar el momentum de la tendencia detectada.
- **Determinación de Tendencia Multi-Escala:** Integrar filtros **LOWESS**, EMD y **regresiones polinómicas** aplicadas a periodos grandes y pequeños de forma simultánea. El modelo operará exclusivamente a favor del trend cuando exista una correlación clara entre las diferentes temporalidades analizadas.
