# Shark Spotting Business (California) · Shark Attacks EDA & Cleaning
**JuanJo Manu Consulting**

## Objetivo del proyecto
El objetivo del proyecto consiste en limpiar el dataset buscando hipótesis con las que explorar negocios potencialmente rentables.

## Contexto del negocio
Somos una consultora (`Manuel-Juanjo Consulting Enterprise`) que entrega un informe a una empresa de *shark spotting* que ya opera en **Florida** y quiere **expandirse al mejor estado posible** minimizando riesgo y maximizando oportunidades de avistamiento.

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
- `mes`: mes extraído desde `date` (`Jan`–`Dec`) o `None` si no se pudo parsear.

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
- ¿Qué estado es mejor candidato para una expansión desde Florida si buscamos **más avistamientos** y **menor riesgo**?
- En California: ¿en qué meses se producen más incidentes?
- En California: ¿qué actividades aparecen como más seguras para un negocio de avistamiento?
- En California: ¿qué localizaciones concentran más incidentes y dónde conviene situar una sede operativa para poder abaratar costes de desplazamiento?

## Proceso de análisis
- EDA inicial: estructura, tipos, nulos, duplicados, variabilidad.
- Limpieza: normalización de categorías (`type/sex/fatal/species/state`), limpieza de strings y reducción de ruido.
- Feature engineering: `age_clean` (numérica) y `mes` (mes desde `date`), recategorización de `activity`.
- Segmentación: comparación inicial California vs Hawaii y focalización final en California.
- Métricas usadas: conteos por mes/estado, aproximación de letalidad por `fatal`, distribución de localizaciones y proporciones.

## Resultados / Insights

### Marketing
- El análisis por edad de nuestro dataframe arroja que la mediana de edad es de `24` años. Esta es la edad que se ha imputado en los datos. Se recomienda un marketing que el target principal sea este grupo de edad.
- Por la conclusión de la hipótesis 2, la campaña publicitaria podría segmentarse en `fishing` y `diving` como reclamo de dos grupos de edades: uno más joven (`diving`) y otro más maduro (`fishing`).

### Sede
- Por el porcentaje de avistamientos o encuentros y cercanía de ciudades en el mismo estado de `California`, `Los Angeles` parece la localización idónea para redirigir el flujo de clientes hacia la costa (`Los Angeles`, `San Diego`, `Santa Barbara` y `Orange County`) que en ese momento por avistamientos o análisis ulteriores señalen mayor probabilidad. Juntas, estas cuatro localizaciones suman casi el 40% de los avistamientos en el estado.

## Recomendaciones de negocio
- **Estado recomendado** para expansión: **California** (comparando con Hawaii, el riesgo observado es menor y el volumen de incidentes es comparable).
- **Ventana temporal**: concentrar operaciones en **junio–octubre** (especialmente **julio, agosto y septiembre**).
- **Propuesta de actividades**: priorizar campañas/experiencias orientadas a **fishing** y **diving** como actividades relativamente “más seguras” según el proxy de letalidad.
- **Sede operativa**: usar **Los Angeles** para redirigir flujo hacia costa y zonas cercanas (p.ej. Los Angeles / San Diego / Santa Barbara / Orange County).

## Limitaciones
- Faltan: proxies y correlaciones para un análisis más en detalle buscando también agregaciones con datasets externos para obetener conclusiones más sólidas y una limpieza más profunda y una búsqueda semántica por localización agregada más extensa.
- La fiabilidad de las fuentes no ha sido corroborada.
- Explorar `Hawaii` como estado destino para segmentación de `extreme diving`.

## Próximos pasos
- Profundizar en correlación `species` ↔ `fatal` y estacionalidad por especie, letalidad y tipo de daño (`injury`).
- Robustecer fechas y métrica de letalidad (tasa, no solo conteo).
- Añadir visualizaciones (mapa/heatmap/scatterplot/gráfico de barras por mes y zona).

## Cómo replicar el proyecto
Instrucciones exactas (entorno, dependencias, y orden recomendado):
- Notebook de exploración inicial: `01_notebook_exploracion_inicial.ipynb`
- Notebook de limpieza: `02_notebook_limpio.ipynb`
- Notebook de hipótesis/insights: `03_notebook_hipotesis.ipynb`
