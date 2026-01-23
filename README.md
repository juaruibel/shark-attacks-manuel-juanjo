# Shark Spotting Business (California) · Shark Attacks EDA & Cleaning
**JuanJo Manu Consulting**

## Objetivo del proyecto
El objetivo del proyecto consiste en limpiar el dataset buscando hipótesis con las que explorar negocios potencialmente rentables.

## Contexto del negocio
Somos una consultora (`Manuel-Juanjo Consulting Enterprise`) que entrega un informe a una empresa de *shark spotting* que ya opera en **Florida** y quiere **expandirse al mejor estado posible** minimizando riesgo.

## Dataset
**Fuente**
- Global Shark Attack File (GSAF) — `GSAF5.xls` (descargado vía URL con `pandas.read_excel`).

**Scope**
- Incidentes: cada fila es un incidente/registro de ataque que puede ser `fatal` (Y/N/UNKNOWN)
- Scope final del análisis: **California** (se compara con Hawaii en el planteamiento inicial, y se decide focalizar en California debido a la menor `letalidad`).

**Tamaños**
- Dataset original: `7065` filas × `23` columnas.
- Dataset limpio: `2576` filas × `12` columnas.
- Dataset final (California): `328` filas × `12` columnas.

**Diccionario breve (dataset final)**
- `type`: tipo de incidente (p.ej. `Unprovoked`, `Provoked`, `Watercraft`, `Sea Disaster`, `Invalid`, `Questionable`).
- `country`: país (en nuestro análisis final: `USA`).
- `state`: estado (scope final: `California`).
- `location`: localización (texto).
- `activity`: actividad recategorizada (p.ej. `surfing`, `swimming`, `fishing`, `diving`, `snorkel`, `hunting`, `questionable`, etc.).
- `name`: nombre (con limpieza y anonimización parcial).
- `sex`: sexo (`M`, `F`, `Unknown`).
- `injury`: descripción de lesiones (texto).
- `fatal`: indicador de letalidad (`Y`, `N`, `UNKNOWN`).
- `species`: especie recategorizada (p.ej. `White Shark`, `Tiger Shark`, `Bull Shark`, `Hammerhead Shark`, `Other Shark`, `Unknown`).
- `age_clean`: edad numérica (derivada desde `age`).
- `mes`: mes extraído desde `date` (`Jan`–`Dec`) o `None`.

## Notas sobre calidad del dato
- Se eliminaron **columnas referenciales** que no eran necesarias para el análisis de incidencias/riesgo (p.ej. columnas tipo `pdf`, `href`, `case number`, `unnamed`, etc.).
- Se detectaron **3 duplicados exactos** y se eliminaron.
- Se imputó `age_clean` usando la **mediana** (estrategia robusta frente a extremos).

**Normalizaciones principales (resumen)**
- `fatal`:
```python
def clean_fatal(valor):
    if valor == "Y":
        return "Y"
    elif valor == "N":
        return "N"
    else:
        return "UNKNOWN"
```

- `species`:
```python
def clean_species(valor):
    valor = valor.strip().lower()

    if "white" in valor:
        return "White Shark"
    elif "tiger" in valor:
        return "Tiger Shark"
    elif "bull" in valor:
        return "Bull Shark"
    elif "hammer" in valor:
        return "Hammerhead Shark"
    elif "shark" in valor:
        return "Other Shark"
    else:
        return "Unknown"
```

- `type` (mapping):
```python
type_mapping = {
    "Unprovoked": "Unprovoked",
    "Provoked": "Provoked",
    "Invalid": "Invalid",
    "Watercraft": "Watercraft",
    "Sea Disaster": "Sea Disaster",
    "Questionable": "Questionable",
    "Boat": "Watercraft",
    " Provoked": "Provoked",
    "unprovoked": "Unprovoked",
    "?": "Questionable",
    "Unconfirmed": "Questionable",
    "Unverified": "Questionable",
    "Under investigation": "Questionable",
}
```

- `sex` (mapping):
```python
sex_mapping = {
    "M": "M",
    "F": "F",
    "Questionable": "Unknown",
    "N": "Unknown",
    "m": "M",
    "lli": "Unknown",
    "M x 2": "Unknown",
    ".": "Unknown",
}
```

- `mes` desde `date`:
```python
def mes_map(date):
    meses = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun','Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
    date = str(date).title()
    for mes in meses:
        if mes in date:
            return mes
    return None
```

- `activity` (recategorización por keywords):
```python
def activity_maping(activity):
    activity = activity.strip().lower()
    if "surf" in activity:
        return "surfing"
    if "swim" in activity:
        return "swimming"
    if "fishing" in activity:
        return "fishing"
    if "diving" in activity:
        return "diving"
    if "snorkel" in activity:
        return "snorkel"
    if "hunt" in activity:
        return "hunting"
    if "question" in activity:
        return "questionable"
    else:
        return activity.strip().lower()
```

- `state` (normalización):
```python
def state_format(state):
    state = state.strip().lower()
    if "flor" in state:
        return "Florida"
    if "hawai" in state:
        return "Hawaii"
    if "cali" in state:
        return "California"
    if "baha" in state:
        return "Bahamas"
    else:
        return state.strip().title()
```

## Preguntas clave
Preguntas que guían el análisis (mapeadas en `notebook_limpio.ipynb` y `notebook_hipotesis.ipynb`):
- ¿Qué estado es mejor candidato para una expansión desde Florida si buscamos **menor riesgo**?
- En California: ¿en qué meses se producen más incidentes?
- En California: ¿qué actividades aparecen como más seguras según el proxy de letalidad del dataset?
- En California: ¿qué localizaciones concentran más incidentes y qué área podría servir como sede operativa orientada a cobertura logística?

## Proceso de análisis
- EDA inicial: estructura, tipos, nulos, duplicados, variabilidad.
- Limpieza: normalización de categorías (`type/sex/fatal/species/state`), limpieza de strings y reducción de ruido.
- Feature engineering: `age_clean` (numérica) y `mes` (mes desde `date`), recategorización de `activity`.
- Segmentación: comparación inicial California vs Hawaii y focalización final en California.
- Métricas usadas: conteos por mes/estado, aproximación de letalidad por fatal (pendiente de definir tratamiento de `UNKNOWN`), distribución de localizaciones y proporciones.

## Resultados / Insights

### Marketing
- Debido al alto porcentaje de valores faltantes en `age`, la imputación con la mediana facilita el análisis pero puede sesgar la distribución, por lo que evitamos usarla como recomendación directa de marketing. Como observación preliminar, la edad observada sugiere una concentración en rangos adultos jóvenes, pero se requiere validación con datos completos.
- Como hipótesis de marketing operativa, proponemos orientar campañas a `fishing` y `diving` como dos líneas de producto distintas (perfiles potencialmente diferentes). La segmentación por edad se plantea como siguiente paso, calculando distribuciones y tasas por actividad usando edades no imputadas o imputación controlada (marcando `imputed`).

### Sede
- Dado que `Los Angeles`, `San Diego`, `Santa Barbara` y `Orange County` concentran aproximadamente el 40% de los incidentes registrados en `California` tras la agregación de `location`, proponemos `Los Angeles` como sede operativa para maximizar cobertura de las zonas con mayor concentración de incidentes y reducir fricción logística hacia los principales puntos de interés. Esta decisión debe validarse en una siguiente iteración con denominadores de exposición (afluencia a playas/actividad acuática) y señales de demanda (turismo/operadores/condiciones), ya que el dataset mide incidentes, no avistamientos.

## Recomendaciones de negocio
- **Estado recomendado** para expansión: **California** (en este análisis exploratorio, el riesgo observado es menor que en Hawaii y el volumen de incidentes es comparable; requiere validación tras corregir errores y definir tasa con `UNKNOWN`).
- **Ventana temporal (incidentes registrados)**: mayor concentración en junio–octubre (especialmente julio, agosto y septiembre).
- **Propuesta de actividades**:`fishing` y `diving` se plantean como hipótesis de menor letalidad observada; requiere cálculo formal de tasas por actividad y control de tamaños muestrales.
- **Sede operativa (cobertura de incidentes)**: `Los Angeles` como hipótesis logística para cubrir áreas con mayor concentración de incidentes; requiere validación con exposición/demanda.

## Limitaciones
- Faltan: proxies y correlaciones para un análisis más en detalle buscando también agregaciones con datasets externos para obetener conclusiones más sólidas y una limpieza más profunda y una búsqueda semántica por localización agregada más extensa.
- La fiabilidad de las fuentes no ha sido corroborada.
- Explorar `Hawaii` como estado destino para segmentación de `extreme diving`.

### Errores

- En el notebook `02_notebook_limpio.ipynb` hay dos decisiones que contaminan el análisis:
    * En la celda 9:
        ```python
        # Aplicamos `.map()`
        df_shark_attacks['type'] = df_shark_attacks['type'].map(type_mapping)
        # Rellenamos `NaN` con `Questionable``
        df_shark_attacks.fillna("Questionable", inplace=True)
        ```
        Estas líneas de código rellenan con `Questionable` todos los nulos a nivel global.
    * En la celda 11:
        ```python
        # Debido  gran cantidad de `Questionable` y de valores en rangos creamos columna `age_clean`
        # Guardamos el resto de valores por trazabilidad pero los análisis los haremos sobre `age_clean`
        df_shark_attacks['age_clean'] = pd.to_numeric(df_shark_attacks['age'], errors='coerce')
        # Comprobamos valores nulos por imputar
        print(df_shark_attacks['age_clean'].isnull().sum())
        # Observamos estadísticas para decidir cuál usar para imputar resultados a `NaN``
        print(df_shark_attacks["age_clean"].describe().T)
        # Usaremos la mediana para imputar: las edades están dispersas y el máximo es de `86`
        age_mediana = df_shark_attacks['age_clean'].median()
        # Rellenamos los nulos con la mediana
        df_shark_attacks['age_clean'] = df_shark_attacks['age_clean'].fillna(age_mediana)
        ````
        Estas líneas imputan con la mediana una gran parte de la columna de `age`; esto afecta especialmente a cualquier conclusión basada en `age_clean` (p.ej., segmentación por edad)
- Estas decisiones afectan especialmente a la integridad de valores faltantes por el `fillna` global y a cualquier observación basado en `age_clean` por imputación masiva.

## Próximos pasos
- Profundizar en correlación `species` ↔ `fatal` y estacionalidad por especie, letalidad y tipo de daño (`injury`).
- Robustecer fechas y métrica de letalidad (tasa, no solo conteo).
- Añadir visualizaciones (mapa/heatmap/scatterplot/gráfico de barras por mes y zona).
- Corregir errores detectados en el proceso de limpieza y recalcular métricas clave

## Cómo replicar el proyecto
Instrucciones exactas (entorno, dependencias, y orden recomendado):
- Notebook de exploración inicial: `01_notebook_exploracion_inicial.ipynb`
- Notebook de limpieza: `02_notebook_limpio.ipynb`
- Notebook de hipótesis/insights: `03_notebook_hipotesis.ipynb`
