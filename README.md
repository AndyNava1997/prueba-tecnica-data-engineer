# Prueba técnica – Data Engineer

Solución de la prueba práctica de Data Engineer: una sección de **SQL (SQLite)** y otra de **PySpark**, desarrolladas y ejecutadas en Google Colab.

**Autor:** Freddy Navarrete

## Estructura del repositorio

```
prueba-tecnica-data-engineer/
├── Prueba_Tecnica_DE_Data_Engineer_Banco_Pichincha_Freddy_Navarrete.ipynb   # notebook con la solución y la evidencia de ejecución
├── data/
│   └── CENTROS_EDUCATIVOS_MADRID.json                                       # insumo de la Pregunta 5
├── output/
│   └── jugadores_estrella.json                                              # salida de la Pregunta 4 (pedido 3.4)
└── README.md
```

## Cómo ejecutarlo

1. Abrir el notebook en [Google Colab](https://colab.research.google.com/).
2. Ejecutar la celda de configuración del ambiente (Spark + SQLite) y luego la celda de reinicio, como indica el notebook.
3. Subir `data/CENTROS_EDUCATIVOS_MADRID.json` a la ruta `/content`.
4. Ejecutar el resto de celdas en orden. La Pregunta 6 reutiliza los DataFrames de la Pregunta 5.

Cada pregunta se resuelve en una sola celda de código y los enunciados se mantienen sin cambios.

## Resumen de la solución

### Sección 1 – SQL (SQLite)

| Pregunta | Enfoque | Resultado |
|---|---|---|
| **1. Puntos y gol diferencia** | Se separan equipos (`'Local vs. Visitante'`) y marcador (`'gl,gv'`) con `SUBSTR`/`INSTR`, se despivotea cada partido en dos filas (una por equipo) y se agrega con `CASE` (3/1/0) y `SUM(gf - gc)`. | 24 equipos. El resultado coincide al 100% con la tabla de referencia `Resultados_Qatar` (control automático en la misma celda). |
| **2. Clasificados y sede** | `ROW_NUMBER()` por grupo (puntos DESC, gol diferencia DESC). La tabla `Clasificados` solo trae llaves tipo `1º del grupo A vs. 2º del grupo B`: se separan en (posición, grupo) y se cruzan con los clasificados, también para obtener el rival. La ciudad sale de la sede y la fecha/hora se estandariza a ISO 24 h. | 12 equipos con rival, fecha/hora, ciudad y estadio. |
| **3. Equipos élite (quintil superior)** | `NTILE(5)` sobre los puntos, más una regla de corte: clasifican todos los equipos con puntos ≥ al mínimo del quintil 1. | 6 equipos. `NTILE` corta por número de filas y dejaba fuera a uno de los dos equipos empatados a 6 puntos (Corea del Sur y Alemania); la regla de corte evita esa exclusión arbitraria. |

**Calidad de datos detectada en `Partidos`:** el mismo equipo aparece escrito de distintas formas (`Irán`/`Iran`, `España`/`Espana`, `Arabia Saudi`/`Arabia Saudí`, `Perú`/`Peru`) y con punto final (`Portugal.`, `Uruguay.`, `Costa Rica-Nueva Zelanda.`). Se normaliza quitando tildes y ñ, puntos finales y espacios, y pasando a mayúsculas; como `UPPER` de SQLite solo convierte ASCII, primero se reemplazan los acentos. También hay fechas con año 2021 en lugar de 2022, aunque no afectan los cálculos.

### Sección 2 – PySpark

| Pregunta | Enfoque | Resultado |
|---|---|---|
| **4. Ranking de goleadores** | Esquema explícito y tipado de columnas numéricas; limpieza de nombres de país; `row_number()` por país para marcar al mejor jugador. Nuevo indicador: **goles por partido** (efectividad), con desempate por goles totales. Exportación a JSON anidado (`collect_list` + `struct` + `array_sort`). | Sí cambia el resultado: 48 de 50 jugadores cambian de posición, el primer lugar pasa de Cristiano Ronaldo a Sándor Kocsis y cambia el mejor jugador de Kuwait y de Tailandia. |
| **5. Centro educativo más céntrico** | Lectura `multiLine`, exclusión de 4 centros sin coordenadas, promedio de X/Y por `centro_titularidad`, **UDF** de distancia euclidiana (validada contra funciones nativas, diferencia 0) y `row_number()` por grupo. | Un centro de reunión por cada uno de los 4 tipos de titularidad. Las coordenadas son UTM (m), así que la distancia queda en metros. |
| **6. Exportación y formato** | Exportación a Parquet, JSON y CSV (`sep='\|'`, con cabecera) en `/tmp`, con relectura para validar el número de registros y comparación de tamaños. | **Parquet / Delta Lake** como formato recomendado. Con el dataset completo, Parquet ocupó ~264 KB frente a ~794 KB en CSV y ~2,2 MB en JSON. |

**Calidad de datos detectada en la Pregunta 4:** los nombres de país traen espacios de no separación (` `), que `trim()` no elimina; se reemplazan antes de limpiar.

## Tecnologías

Python 3 · SQLite · Apache Spark (PySpark) · Google Colab
