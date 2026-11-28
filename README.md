# Simulación del sistema de pago de parqueaderos — Supercentro

El estudio evalúa si los **3 cajeros por punto de salida** del centro comercial son suficientes. El informe recomienda **5 cajeros**, con una espera media reportada de **4,01 minutos**, frente a **14,63 minutos** con 3 (reducción aproximada del 73%). Son resultados documentados, no reproducidos desde este repositorio: falta el módulo de simulación.

## Documentos

- [Cuaderno de simulación](Parqueaderos_modulo_4.ipynb): explicación del experimento y celdas para ejecutar y visualizar resultados. No contiene salidas de ejecución guardadas. Depende de `simulacion_supercentro.py` (no incluido).
- [Cuaderno del transitorio](transitorio.ipynb): análisis autocontenido de la eliminación del estado transitorio mediante la técnica de Welch. Incluye todo el código inline, gráficas generadas y resultados con outputs. Se puede ejecutar sin dependencias externas más allá de NumPy, Matplotlib y SciPy.

## Modelo e inputs

Simulación por eventos discretos del **pago**, no de la ocupación de plazas. Las fuentes describen cada cajero como una cola **M/M/1 independiente**, con atención **FCFS** (primero en llegar, primero en ser atendido), sin cambio de fila ni deserción.

Los parámetros provienen del enunciado de la actividad; el informe describe las medias de servicio y entre llegadas como exponenciales:

| Tipo de usuario | Proporción | Servicio medio (min) | Intervalo medio entre llegadas (min) |
|---|---:|---:|---:|
| Rápido | 25% | 1 | 3 |
| Normal | 20% | 3 | 3 |
| Lento | 27,5% | 4 | 5 |
| Muy lento | 27,5% | 6 | 7 |

## Experimento

- **Horizonte:** 6.000 minutos por réplica.
- **Transitorio:** técnica de Welch con 120 réplicas, muestreo de longitud de cola cada 10 minutos y suavizado centrado de 15 muestras. Se descartan los primeros 450 minutos.
- **Estadísticas finales:** 40 réplicas independientes; ventana efectiva de 5.550 minutos e intervalos de confianza (IC) del 95% con t de Student (39 grados de libertad).
- **Escenarios:** 3, 4, 5 y 6 cajeros con demanda total constante. Según el informe, el intervalo entre llegadas a cada cajero se escala por N/3.
- **Decisión:** elegir el menor número de cajeros cuyo límite superior del IC del 95% de la espera media sea ≤ 5 minutos y cuyo semiancho relativo sea ≤ 10%.

## Resultados y recomendación

Valores de la tabla 5 del informe; las esperas de 3, 4 y 5 cajeros también aparecen en el texto del cuaderno:

| Cajeros | Espera media (min) | IC 95% (min) | Semiancho relativo | Cumple |
|---|---:|---|---:|---|
| 3 | 14,63 | 13,93–15,33 | 4,81% | No |
| 4 | 6,31 | 6,10–6,51 | 3,23% | No |
| **5** | **4,01** | **3,91–4,12** | **2,60%** | **Sí** |
| 6 | 2,98 | 2,90–3,06 | 2,66% | Sí |

Se propone **añadir 2 cajeros por punto de salida**. Cinco es la configuración mínima que satisface el criterio; seis también cumple, pero requiere una inversión adicional. La decisión final debe considerar compra, instalación y mantenimiento: no se presenta un análisis económico.

En la configuración base, la utilización reportada ronda el 77%. El cajero 1 registra el menor servicio medio (3,5654 min) y el 3 el mayor (3,5811 min); las fuentes no atribuyen esta pequeña diferencia a una ventaja real entre cajeros. El informe registra unas 3.587 llegadas por réplica durante la ventana de observación, sumando los tres cajeros.

## Ejecución y estado de reproducibilidad

**La ejecución del cuaderno principal está bloqueada en la versión actual.** `Parqueaderos_modulo_4.ipynb` importa `run` desde `simulacion_supercentro`, pero no se incluye `simulacion_supercentro.py`. Tampoco se incluyen la carpeta `figuras/`, los resultados JSON anunciados ni un archivo `requirements.txt`. Para consultar las conclusiones no es necesario ejecutar el cuaderno.

**El cuaderno del transitorio sí es ejecutable.** `transitorio.ipynb` contiene todo el código inline y se puede ejecutar directamente con:

```
pip install numpy matplotlib scipy
jupyter notebook transitorio.ipynb
```

Este cuaderno reproduce de forma independiente el análisis de Welch (120 réplicas), la comparación antes/después de eliminar el transitorio, el análisis de sensibilidad al periodo de calentamiento y la convergencia con 40 réplicas.

## Limitaciones y discrepancias por resolver

- **Calibración y validación externa pendientes:** los parámetros no se ajustaron con observaciones reales del centro comercial. Las fuentes declaran verificaciones internas (FCFS, orden de eventos, conservación y utilización), pero no pueden auditarse aquí sin el código y sus salidas.
- **Tasa e intervalo de llegada:** el informe presenta una tasa global de 0,645 usuarios/min y la llama equivalente a un intervalo de 4,65 min. No son recíprocos: 1/0,645 ≈ 1,55 min; 4,65 min sería compatible con el intervalo por cajero al repartir el flujo entre tres. El cuaderno no resuelve esta ambigüedad.
- **Supuesto M/M/1:** ambas fuentes usan esa denominación, pero las medias exponenciales distintas por clase requieren aclarar cómo se generan y combinan los flujos y servicios; no puede verificarse la distribución agregada sin el módulo.
- **Conservación y significancia:** llegadas y salidas casi iguales en una ventana finita no prueban por sí solas que todos los usuarios hayan sido atendidos; el informe registra 3.587,1 llegadas y 3.588,2 salidas en esa ventana. Asimismo, el solapamiento de IC que citan las fuentes no sustituye una prueba formal de diferencias entre cajeros.

La recomendación debe interpretarse dentro de estos supuestos y del criterio de servicio elegido, no como una garantía de desempeño real.
