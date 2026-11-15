# Entrega final Kaggle · Labo 1 Virtual 2026 · Analista Jr

**Alumno:** <Nombre y apellido>
**Competencia:** [labo-1-virtual-2026-analista-jr](https://www.kaggle.com/competitions/labo-1-virtual-2026-analista-jr)
**Submit final elegido:** `KA9592_ens4_1900.csv` (ensamble de 4 corridas, 1.900 envíos)

---

## Resumen

La entrega es un **ensamble de 4 corridas** del workflow. Cada corrida ya es un semillerío de 50 modelos LightGBM, así que el submit final promedia 200 modelos.

Se parte del workflow oficial `PredFinal/z728_final_junior_power.ipynb`. Del z728 se conservan el semillerío con semillas primas, el armado de `param_final` con `modifyList`, la grilla de hiperparámetros y `training_pct = 1.0` (salvo en la corrida 9530, cambio 6). Sobre esa base se agregan los cambios de la tabla siguiente, apoyados en los Experimentos Colaborativos, en el libro de la asignatura y en corridas propias.

## Modificaciones respecto de `z728_final_junior_power`

| # | Cambio | Oficial | Esta entrega | Fundamento |
|---|---|---|---|---|
| 1 | Meses de pandemia en training y final_train | se usan todos los meses desde 201901 | **sin 202003 a 202007** (corridas 9510 y 9511) | Libro, 6.9.1 (período del Grupo A sin esos meses) y 6.16 ("salvedades pandémicas"). Problema #05, Grupo A: excluir la pandemia restringida dio una media levemente mayor que la base, sin significancia. Por eso se cubre con dos corridas que sí usan todos los meses (cambio 5). |
| 2 | Variables del modelo | incluye `numero_de_cliente` y `foto_mes` | **sin `numero_de_cliente` ni `foto_mes`** | Problema #16: en el Grupo B (clase conocida en 202107) quitar ambos mejoró la ganancia en 10 de 10 semillas (p = 0,006); en el Grupo A (Public de 202109) no la afectó. `foto_mes` no puede extrapolar a un mes no visto; `kmes` se mantiene. |
| 3 | Semillerío | 20 semillas | **50 semillas** | PDF del profesor "Cómo elegir el mejor modelo": con 50 modelos la ganancia media del semillerío se estabiliza y el desvío del Private baja de 0,27 (un modelo) a 0,06. |
| 4 | Cortes de envíos | 1.700 a 2.300 | **1.700 a 2.600** en las corridas; **1.400 a 2.600** en el ensamble | En las corridas exploratorias 9500 y 9501 el máximo del Public cayó en 2.400, el borde superior. En la 9510 y la 9511 el máximo cayó en 1.700, el borde inferior, así que antes de correr el ensamble se extendió su rango hacia abajo. |
| 5 | Predicción final | un workflow | **ensamble de 4 corridas** con distinta semilla primigenia | Promediar corridas con distinta semilla reduce también el ruido de la grilla de hiperparámetros, que el semillerío no reduce. Combina dos corridas sin pandemia y dos con todos los meses: el PDF "Cómo elegir el mejor modelo" recomienda ensamblar modelos distintos entre sí y evitar la endogamia. |
| 6 | `training_pct` en la grilla | 1.0 | **0.25 solo en la corrida 9530** | Las tres primeras corridas correlacionaron entre 0,989 y 0,995 (ver "Correlaciones"), por encima del umbral de 0,98 fijado de antemano: el ensamble tenía poca diversidad. La 9530 busca hiperparámetros con otra semilla y con el 25% de los CONTINUA, para ordenar a los clientes de forma algo distinta. Problemas #9A y #9B: recortar no mostró una pérdida significativa de ganancia. |

### Lo que se evaluó y no se incorporó

| Idea | Motivo |
|---|---|
| Balanceo de clases (`is_unbalance`, `scale_pos_weight`) | Problema #13 (experimento propio, Grupo A): ninguna mejora significativa; el balanceo automático dio menor ganancia. |
| Más lags, tendencias, medias móviles | Problema #04, Grupo A (202109): todas las variantes rindieron menos que lag 1 y 2. Problema #05, Grupo A: los lags 3, 6 y 12 empeoraron el modelo. |
| Undersampling en la grilla como receta principal | Problemas #09 y #13: sin mejora de ganancia; solo ahorra tiempo. Libro 6.16: usar la mayor cantidad de datos. Se usa solo en la 9530, para sumar diversidad al ensamble (cambio 6). |
| Otro boosting (rf, dart) | Problema #14: gbdt superó a ambos en los dos grupos. |

## Corridas

| Experimento | Notebook | Semilla primigenia | Receta |
|---|---|---|---|
| 9510 | `z728_final_junior_9510.ipynb` | 291647 | Cambios 1 a 4 (sin pandemia) |
| 9511 | `z728_final_junior_9511.ipynb` | 348149 | Cambios 1 a 4 (sin pandemia) |
| 9520 | `z728_final_junior_9520.ipynb` | 596927 | Cambios 2 a 4 (con todos los meses) |
| 9530 | `z728_final_junior_9530.ipynb` | 318443 | Cambios 2 a 4 y 6 (con todos los meses, `training_pct = 0.25`) |
| **9592** | **`z728_ensamble_9592.ipynb`** | — | Promedio simple de las probabilidades de las 4 corridas |

Cada corrida graba `/content/buckets/b1/exp/WF<experimento>/prediccion.txt`, con la probabilidad promedio de sus 50 modelos para cada cliente de 202109. El ensamble lee esos 4 archivos, promedia `prob` con el mismo peso para cada corrida (1/4), ordena de mayor a menor y marca con 1 a los primeros N clientes.

Public de cada corrida suelta (solo como referencia; no se usa para elegir):

| Envíos | 1.700 | 1.800 | 1.900 | 2.000 | 2.100 | 2.200 | 2.300 | 2.400 | 2.500 | 2.600 | Media |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 9510 | 86,40 | 84,24 | | | | 86,23 | | | | 82,74 | 83,7 |
| 9511 | 86,81 | 83,90 | | | | 79,75 | | | | 79,34 | 82,3 |
| 9520 | 82,49 | 82,82 | 80,42 | 78,17 | 78,42 | 85,90 | 86,48 | 84,40 | 85,65 | 82,41 | 82,7 |
| 9530 | 83,32 | 83,82 | 81,33 | 79,34 | 83,32 | 86,98 | 87,98 | 85,15 | 82,91 | 80,33 | 83,4 |

<COMPLETAR los cortes vacíos de la 9510 y la 9511.>

## Correlaciones

Ensamble de las 3 primeras corridas:

| | 9510 | 9511 | 9520 |
|---|---|---|---|
| 9510 | 1,000 | 0,995 | 0,993 |
| 9511 | 0,995 | 1,000 | 0,989 |
| 9520 | 0,993 | 0,989 | 1,000 |

Antes de verlas se fijó una regla: si todas superaban 0,98, el ensamble tenía poca diversidad y se sumaba una cuarta corrida (la 9530). Todas la superaron.

Correlación de la 9530 con las otras tres: <COMPLETAR con la salida del notebook de ensamble>.

## Elección del corte

La regla se fijó antes de ver el Public del ensamble. Según el PDF "Cómo elegir el mejor modelo", cuando solo cambia la semilla el Public y el Private son independientes, así que el máximo del Public no se usa por sí solo para elegir:

1. Se miran los cortes del ensamble (la regla original hablaba de 1.700 a 2.600; el rango se extendió a 1.400 antes de correr el ensamble, ver cambio 4).
2. Se descartan los picos aislados (un corte alto con los vecinos bajos).
3. Entre los que quedan se elige el más alto; si dos están a menos de 2 puntos, el de más envíos, porque cerca del óptimo enviar de más cuesta menos que enviar de menos.
4. Si no hay una zona alta clara, se elige 2.200.

Referencia previa: el rango de cortes del z728 (1.700 a 2.300, paso de 100) está centrado en 2.000, igual que el rango del notebook Senior (8.000 a 12.000) está centrado en el óptimo indicado en clase para esa modalidad.

Public del ensamble 9592 de 4 corridas (`KA9592_ens4_*.csv`):

| Envíos | 1.400 | 1.500 | 1.600 | 1.700 | 1.800 | 1.900 | 2.000 | 2.100 | 2.200 | 2.300 | 2.400 | 2.500 | 2.600 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Public | 87,31 | 84,90 | 85,48 | 82,99 | 87,06 | **84,98** | 81,99 | 78,92 | 82,91 | 83,99 | 84,57 | 81,58 | 82,74 |
| Media con sus 2 vecinos | — | 85,9 | 84,5 | 85,2 | 85,0 | 84,7 | 82,0 | 81,3 | 81,9 | 83,8 | 83,4 | 83,0 | — |

Como referencia, el ensamble de las 3 primeras corridas (`KA9592_ens3_*.csv`) dio: 80,50 · 84,82 · 85,98 · 86,23 · 86,56 · 83,99 · 81,83 · 78,84 · 86,48 · 83,74 · 81,08 · 81,74 · 79,09 (de 1.400 a 2.600). No se usa para el envío final.

**Aplicación de la regla:**

- Paso 2: se descartan 1.800 (87,06, entre 82,99 y 84,98) por ser un pico aislado, y 1.400 (87,31), que está en el borde, con un solo vecino más bajo y fuera del rango original de la regla.
- Paso 3: el más alto de los que quedan es 1.600 (85,48). A menos de 2 puntos y con más envíos están 1.900 (84,98), 2.300 (83,99) y 2.400 (84,57).

Leída al pie de la letra, la regla da **2.400**. Se eligió **1.900**, por estos motivos:

- La media de cada corte con sus vecinos ronda 85 entre 1.500 y 1.900, y 83,5 entre 2.300 y 2.400. La diferencia es chica, pero apunta a la zona baja.
- El ensamble de 3 corridas mostraba lo mismo: una meseta en 1.600–1.800 y una caída después de 1.900.
- 1.900 es el corte con más envíos dentro de la zona alta. El paso 3 busca enviar de más cerca del óptimo, no saltar por encima de la caída de 2.000–2.100 a otra zona.
- Queda cerca de la referencia previa de 2.000–2.200.

Esta interpretación se tomó con la curva a la vista, y la diferencia entre 1.900 y 2.400 está dentro del ruido del Public. La caída en 2.000–2.100 aparece en todas las corridas, pero como todas se evalúan contra los mismos clientes del Public, eso no prueba que exista también en el Private.

## Cómo reproducir el submit final

1. Usar la VM `desktop-jr` de la cátedra (8 vCPU, 64 GB de RAM), con el dataset `analistajr_competencia_2026.csv.gz` en `/content/datasets`.
2. Copiar los notebooks de esta carpeta a `/content/buckets/b1/`.
3. Ejecutar completos, en cualquier orden, los 4 notebooks de las corridas (`z728_final_junior_9510`, `_9511`, `_9520` y `_9530`), con el runtime en R. Las tres primeras tardan entre 1,5 y 2 horas cada una; la 9530, cerca de una hora.
   - Si una corrida se interrumpe, se vuelve a ejecutar el mismo notebook: no repite los modelos ya grabados en `modelos/`.
4. Ejecutar `z728_ensamble_9592.ipynb` con `PARAM$ensamble$corridas <- c(9510, 9511, 9520, 9530)`. Genera `/content/buckets/b1/exp/WF9592/kaggle/KA9592_ens4_1900.csv`, que es el submit final (se incluye una copia en esta carpeta para comparar).

## Archivos

- `z728_final_junior_9510.ipynb`, `z728_final_junior_9511.ipynb`, `z728_final_junior_9520.ipynb`, `z728_final_junior_9530.ipynb`: las 4 corridas. Cada cambio está marcado en el código con el comentario `# MODIFICADO [n]`.
- `z728_ensamble_9592.ipynb`: el ensamble y la generación de los archivos para Kaggle.
- `PARAM_9510.yml`, `PARAM_9511.yml`, `PARAM_9520.yml`, `PARAM_9530.yml`, `PARAM_9592.yml`: los parámetros efectivamente usados, grabados por cada notebook.
- `KA9592_ens4_1900.csv`: el archivo subido a Kaggle como entrega final.

## Entorno

Google Cloud, VM `desktop-jr` de la cátedra (Toronto, 8 vCPU, 64 GB de RAM, tarifa estándar). R con `data.table`, `lightgbm`, `primes`, `yaml` y `R.utils`.
