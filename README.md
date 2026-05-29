# mci506-f1-analysis-ronald

## Descripción

Proyecto de análisis de datos de Fórmula 1 usando FastF1 y Google Cloud Storage.

Permite descargar eventos y sesiones de carreras en formato parquet, almacenar los datos en `data/raw/` y subirlos a un bucket de GCS con organización tipo Hive.

## Estructura del proyecto

- `README.md` — documentación del proyecto.
- `requirements.txt` — dependencias Python necesarias.
- `notebook.ipynb` — análisis exploratorio y visualización.
- `utils.py` — utilidades comunes, rutas, helpers y cliente GCS.
- `scripts/`
  - `extract.py` — descarga datos de FastF1 y genera parquet.
  - `load.py` — sube los parquet de `data/raw/` a Google Cloud Storage.
- `data/`
  - `cache/` — caché de FastF1 y datos descargados.
  - `raw/` — archivos parquet generados por la extracción.
- `mci506-data-eng-496723-5d26bf34def8.json` — posible credencial de servicio (no subir a repositorios públicos).

## Dependencias

El proyecto usa:

- `fastf1`
- `pandas`
- `pyarrow`
- `google-cloud-storage`
- `python-dotenv`

Instalar:

```bash
pip install -r requirements.txt
```

## Configuración

1. Crear y activar un entorno virtual de Python.
2. Instalar dependencias con `pip install -r requirements.txt`.
3. Exportar la credencial de Google Cloud en la variable de entorno `GCP_SA_KEY`.

```powershell
$env:GCP_SA_KEY = Get-Content .\mci506-data-eng-496723-5d26bf34def8.json -Raw
```

> Nota: `load.py` requiere `GCP_SA_KEY` con el JSON completo de la cuenta de servicio.

## Uso

### Extraer datos desde FastF1

El script `scripts/extract.py` descarga:

- `schedule_{year}.parquet`
- `results_{year}_{event}.parquet`
- `laps_{year}_{event}.parquet`

La extracción usa los años y rondas definidos en `scripts/extract.py` y guarda los archivos en `data/raw/`.

Ejecutar:

```bash
python scripts/extract.py
```

### Subir datos a Google Cloud Storage

El script `scripts/load.py` sube todos los archivos `.parquet` que estén en `data/raw/` al bucket definido en `utils.py`.

La ruta en GCS se arma así:

- `raw/schedule/year=<year>/schedule_<year>.parquet`
- `raw/results/year=<year>/eventname=<eventname>/results_<year>_<eventname>.parquet`
- `raw/laps/year=<year>/eventname=<eventname>/laps_<year>_<eventname>.parquet`

Ejecutar:

```bash
python scripts/load.py
```

## Workflows y GitHub Actions

El repositorio define un workflow en `.github/workflows/pipeline.yml` llamado `Extract and Load F1 Data`.

### Triggers

- `push`: se ejecuta en cada push al repositorio.
- `workflow_dispatch`: permite ejecutar el pipeline manualmente desde la interfaz de GitHub.

### Pasos de la acción

1. `actions/checkout@v4` — extrae el código del repositorio.
2. `actions/setup-python@v5` — instala Python 3.13.
3. `actions/cache@v4` — cachea `data/cache` para acelerar descargas de FastF1.
4. `pip install -r requirements.txt` — instala dependencias.
5. `python scripts/extract.py` — extrae datos desde FastF1.
6. `python scripts/load.py` — sube los `.parquet` generados a Google Cloud Storage.

### Variables y secretos usados

- `GCP_SA_KEY` debe estar definido en los Secrets de GitHub para poder subir los datos a GCS.
- `DEV_MODE` se calcula automáticamente en el workflow según la rama: se activa cuando la rama no es `main`.

### Cómo ejecutar manualmente

En GitHub, ir a la pestaña `Actions`, seleccionar `Extract and Load F1 Data` y pulsar `Run workflow`.

## Modo desarrollo

El proyecto detecta `DEV_MODE` si:

- la variable de entorno `DEV_MODE=true` está activa, o
- la rama Git no es `main`.

En modo desarrollo, `extract.py` limita la descarga a filas pequeñas para pruebas más rápidas.

## Notas adicionales

- Los archivos parquet se generan a partir de DataFrames de FastF1 con `to_parquet()` en `utils.py`.
- El helper `slug()` normaliza nombres de eventos para los nombres de archivo.
- `data/cache/` se usa como caché de FastF1 y evita descargas repetidas.

## Siguientes pasos

- Abrir `notebook.ipynb` para análisis y visualización de los datos extraídos.
- Ajustar los años y rondas en `scripts/extract.py` según la temporada que se quiera procesar.
