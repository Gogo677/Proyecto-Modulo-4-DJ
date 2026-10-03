# Datos del proyecto

## Organización

| Carpeta | Contenido | Quién lo genera |
|---|---|---|
| `crudos/` | Archivos originales de INEGI, tal como se descargan | Descarga manual o automática desde el notebook |
| `procesados/` | Resultados intermedios del proceso ETL | Notebook, sección 2.3 |
| `analiticos/` | Tabla del modelo, predicciones y validación externa | Notebook, secciones 2.3 y 3.5 |
| `diccionario_datos.csv` | Diccionario de la tabla analítica | Notebook, sección 2.1 |

Los archivos de `procesados/` y `analiticos/` se regeneran al ejecutar el notebook completo. Todos los CSV
están en UTF-8 con BOM, para que Excel muestre bien los acentos.

---

## Datos crudos

Fuente: Instituto Nacional de Estadística y Geografía (INEGI). Licencia:
[Términos de Libre Uso de la Información del INEGI](https://www.inegi.org.mx/inegi/terminos.html).
Descargados el 19 de septiembre de 2026.

| Archivo | Tamaño | Contenido | URL |
|---|---|---|---|
| `denue_00_72_1_csv.zip` | 47.1 MB | DENUE 05/2026, sector 72, parte 1 (520,000 establecimientos) | https://www.inegi.org.mx/contenidos/masiva/denue/denue_00_72_1_csv.zip |
| `denue_00_72_2_csv.zip` | 24.9 MB | DENUE 05/2026, sector 72, parte 2 (286,496 establecimientos) | https://www.inegi.org.mx/contenidos/masiva/denue/denue_00_72_2_csv.zip |
| `iter_00_cpv2020_csv.zip` | 34.9 MB | Censo 2020, principales resultados por localidad (ITER) | https://www.inegi.org.mx/contenidos/programas/ccpv/2020/datosabiertos/iter/iter_00_cpv2020_csv.zip |

Huellas SHA-256. El notebook las verifica al inicio: si INEGI publica otra edición, lo avisa.

```
909dd5c7d4447f2905c890ec1336f65e10ad5ca47467d89276236b48c8701b49  denue_00_72_1_csv.zip
cf12fc06171e06726427db15fd9e29fa0cfb44b9b984edf48fb382a69fcce441  denue_00_72_2_csv.zip
9342fdbd45bda5897f2b827a12904843b6f8fb85d4e72da299e09e4d2422ab60  iter_00_cpv2020_csv.zip
```

Cada ZIP incluye el conjunto de datos en CSV, el diccionario oficial y un archivo de metadatos.

**DENUE**

- 806,496 establecimientos y 42 columnas; el análisis usa 17.
- Un establecimiento aparece dos veces: es la última fila de la parte 1 y la primera de la parte 2, un
  artefacto del corte del archivo.
- Codificación latin-1.
- El sector 72 agrupa hoteles (7211), bares (7224) y restaurantes y otros servicios de preparación de
  alimentos (7225), entre otras ramas.

**ITER**

- 195,662 registros y 286 columnas; el análisis usa 19.
- Codificación UTF-8.
- Mezcla cinco niveles geográficos: nacional, estatal, municipal (`LOC = 0000`), localidades y agrupados de
  localidades de una o dos viviendas (`LOC = 9998/9999`).
- Los datos confidenciales de las localidades aparecen como `*`.

---

## Datos procesados

| Archivo | Filas | Descripción |
|---|---|---|
| `sucursales_mcdonalds.csv` | 417 | Sucursales de McDonald's identificadas en el DENUE: nombre, razón social, clase SCIAN, estrato de personal, municipio 2020, plaza comercial, coordenadas y si se identificó por operador |
| `establecimientos_municipio.csv` | 2,424 | Conteos del DENUE por municipio, **sin** registros de McDonald's: restaurantes, restaurantes con 11 o más empleados, empleo estimado, hoteles, plazas comerciales y sucursales y marcas de 8 cadenas rivales |
| `censo2020_municipio.csv` | 2,469 | Indicadores del censo por municipio: población, porcentajes de jóvenes, vivienda con auto e internet, participación económica, seguridad social, urbanización y coordenadas de la localidad más poblada |
| `reasignacion_municipios.csv` | 8 | Municipios creados después de 2020 y el municipio de 2020 al que se asignan sus establecimientos |

## Datos analíticos

| Archivo | Filas | Descripción |
|---|---|---|
| `tabla_modelo.csv` | 2,469 | Tabla del modelo: identificación, 15 variables explicativas, número de sucursales y variable objetivo `tiene_mcdonalds` |
| `predicciones_municipios.csv` | 2,469 | Probabilidad fuera de pliegue de cada municipio, su rango entre los municipios sin McDonald's, el tipo de oportunidad y la distancia a la sucursal más cercana |
| `validacion_externa_top20.csv` | 20 | Resultado de contrastar los 20 primeros candidatos con el sitio oficial de McDonald's: estado, evidencia, fuente y fecha de consulta |

La clave que une todas las tablas es `clave_mun`: la clave geoestadística de 5 dígitos (2 de la entidad y 3
del municipio) del Marco Geoestadístico 2020. Se guarda como texto para conservar los ceros iniciales.
