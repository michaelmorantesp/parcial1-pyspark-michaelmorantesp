# Análisis Exploratorio de Datos: Hotel Bookings con PySpark

## 📋 Descripción del Proyecto

Este proyecto realiza un **análisis exploratorio de datos riguroso** sobre un dataset de 119,390 reservas hoteleras (2015-2017) de dos propiedades: Hotel Resort y Hotel Ciudad. El objetivo es identificar patrones de comportamiento de booking, factores predictivos de cancelación, y generar insights operacionales para gestión de ingresos (Revenue Management).

**Dataset**: Hotel Bookings (Kaggle)  
**Período**: 2015-2017  
**Registros**: 119,390 reservas  
**Variables**: 32 (20 numéricas + 12 categóricas)  

---

## 🎯 Preguntas de Investigación

1. **¿Cuáles son los perfiles de comportamiento de booking distintos y cómo se relacionan con la cancelación?**
   - Seg mentación por lead_time, cambios en booking, solicitudes especiales
   - Identificación de patrones reflexivos vs. impulsivos

2. **¿Existe una relación significativa entre el canal de distribución y la cancelación?**
   - Análisis comparativo: Direct vs. TA/TO vs. Corporate
   - Heterogeneidad entre tipos de hotel

3. **¿Cuál es la relación conjunta entre duración, composición del grupo, ADR y cancelación?**
   - Efectos de interacción (familias vs. parejas)
   - Segmentación por duración y tipo de grupo

---

## 🏗️ Estructura del Proyecto

```
parcial1-pyspark-[usuario]/
├── docker-compose.yml          # Configuración del contenedor
├── data/
│   └── hotel_bookings.csv      # Dataset (119.3K registros)
├── notebooks/
│   └── analisis_exploratorio.ipynb   # Análisis completo (Jupyter)
└── README.md                   # Este archivo
```

---

## 🐳 Docker Compose Explicado (Línea por Línea)

### `version: '3.8'`
Define la versión del formato de Docker Compose. La versión 3.8 es estable y soporta características esenciales como volúmenes nombrados y redes. Versiones anteriores carecen de ciertas features; versiones muy nuevas pueden tener breaking changes.

**¿Para qué sirve?**: Asegurar compatibilidad con el daemon de Docker. Versión incorrecta → error de parseo.

---

### `services:`
Sección que declara los servicios (contenedores) que compondrán la infraestructura. Puede haber múltiples servicios (ej: Jupyter + Postgres + Redis), cada uno con su propia imagen y configuración. En este caso, solo hay uno: `pyspark-jupyter`.

**¿Para qué sirve?**: Agrupar definiciones de contenedores bajo un único `docker-compose.yml`.

---

### `pyspark-jupyter:`
Nombre del servicio (identificador dentro de docker-compose). Será usado para:
- Referenciar el contenedor en comandos (`docker-compose exec pyspark-jupyter`)
- Resolver DNS dentro de la red de Docker (host: `pyspark-jupyter`)
- Logs y monitoreo

**¿Para qué sirve?**: Organizar múltiples servicios en un proyecto complejo.

---

### `image: jupyter/pyspark-notebook:latest`
Especifica la imagen de Docker a utilizar. `jupyter/pyspark-notebook` es imagen oficial de Jupyter que preinstala:
- **Python 3.x** con Conda
- **Apache Spark** (versión variable según `latest`)
- **PySpark** (bindings Python-Scala)
- **Jupyter Lab + Notebook** (interfaces web)
- Librerías científicas: pandas, scikit-learn, matplotlib, numpy

**¿Para qué sirve?**: Evitar compilar/instalar manualmente. La imagen es reproducible y versionada.

**Nota de reproducibilidad**: `latest` siempre apunta a versión más reciente. Para máxima reproducibilidad, usar tag específico: `jupyter/pyspark-notebook:2023-12-01`.

---

### `container_name: pyspark_hotel_bookings`
Nombre del contenedor cuando corre. Si no se especifica, Docker genera nombre aleatorio (`bold_jones`, etc.). Usar nombre específico facilita:
- Acceso posterior: `docker exec -it pyspark_hotel_bookings bash`
- Identificación en `docker ps`
- Logs específicos: `docker logs pyspark_hotel_bookings`

**¿Para qué sirve?**: Control y referencia del contenedor en ejecución.

---

### `ports:`
Mapeo de puertos host:contenedor. Sintaxis: `"puerto_host:puerto_contenedor"`

```yaml
- "8888:8888"
```

- **Contenedor corre Jupyter en puerto 8888** (por defecto)
- **Host accede vía puerto 8888**
- Cuando abres `http://localhost:8888` en navegador, tráfico se redirige al contenedor

**¿Para qué sirve?**: Exposición de servicios web. Sin este mapeo, Jupyter sería inaccesible desde fuera del contenedor.

**Ejemplo alterno**: `"9999:8888"` → host usa puerto 9999, contenedor usa 8888. Útil si puerto 8888 está ocupado.

---

### `volumes:`
Mapeo de directorios host:contenedor. Sintaxis: `"./ruta_host:/ruta_contenedor"`

```yaml
- ./notebooks:/home/jovyan/work/notebooks
- ./data:/home/jovyan/work/data
```

**Volumen 1: Código**
- **Host**: `./notebooks/` (directorio local en tu máquina)
- **Contenedor**: `/home/jovyan/work/notebooks/` (dentro del contenedor)
- **Efecto**: Los notebooks (.ipynb) que creastes en el contenedor se guardan automáticamente en tu máquina
- **Ventaja**: Si el contenedor se elimina, tus trabajos persisten

**Volumen 2: Datos**
- **Host**: `./data/` (dataset en tu máquina)
- **Contenedor**: `/home/jovyan/work/data/` (dentro del contenedor)
- **Efecto**: `hotel_bookings.csv` es accesible dentro de Jupyter sin copiar
- **Ventaja**: Evita duplicación de archivos grandes

**¿Para qué sirve?**: Persistencia y acceso bidireccional. Sin volúmenes, datos dentro del contenedor se pierden al eliminar el contenedor.


---

### `environment:`
Variables de entorno que se inyectan en el contenedor.

```yaml
- JUPYTER_ENABLE_LAB=yes
- JUPYTER_TOKEN=parcial1
```

**Variable 1: JUPYTER_ENABLE_LAB**
- **Valor**: `yes`
- **Efecto**: Activa Jupyter Lab (interfaz moderna) en lugar de Notebook clásico
- **Acceso**: `http://localhost:8888/lab`
- **¿Para qué?**: Lab tiene mejor editor de código, terminal integrada, debugger

**Variable 2: JUPYTER_TOKEN**
- **Valor**: `parcial1`
- **Efecto**: Requiere token en URL: `http://localhost:8888?token=parcial1`
- **¿Para qué?**: Seguridad básica (evita acceso anónimo). En desarrollo es suficiente. En producción, usar contraseñas con hash.

---

### `working_dir: /home/jovyan/work`
Directorio de trabajo predeterminado cuando el contenedor inicia. Equivalente a `cd /home/jovyan/work`.

**¿Para qué sirve?**: Conveniencia. Los comandos se ejecutan desde este directorio. Sin esta línea, es necesario `cd` manualmente.

---

## 🚀 Cómo Ejecutar el Proyecto

### Requisitos Previos
- **Docker Desktop** instalado y corriendo (Windows, Mac) o **Docker Engine + Docker Compose** (Linux)
- **Dataset** (`hotel_bookings.csv`) en carpeta `data/`

### Pasos

#### 1. Clonar/Descargar el repositorio
```bash
git clone https://github.com/[usuario]/parcial1-pyspark-[usuario].git
cd parcial1-pyspark-[usuario]
```

#### 2. Verificar estructura
```bash
ls -la
# Debe listar:
# - docker-compose.yml
# - data/hotel_bookings.csv
# - notebooks/analisis_exploratorio.ipynb
# - README.md
```

#### 3. Iniciar el contenedor
```bash
docker-compose up
```

**¿Qué sucede?**
1. Docker descarga imagen `jupyter/pyspark-notebook:latest` (~3GB, una sola vez)
2. Crea contenedor con nombre `pyspark_hotel_bookings`
3. Mapea puertos, volúmenes, variables de entorno
4. Inicia Jupyter Lab

**Salida esperada**:
```
pyspark_hotel_bookings  | ... To access the notebook, open this file in a browser:
pyspark_hotel_bookings  |     file:///home/jovyan/.local/share/jupyter/runtime/nbserver_xxxxx.json
pyspark_hotel_bookings  | Or copy and paste one of these URLs:
pyspark_hotel_bookings  |     http://...:8888/?token=parcial1
```

#### 4. Acceder a Jupyter
- Abre navegador: `http://localhost:8888`
- Token: `parcial1` (si es requerido)
- Navega a: `work/notebooks/analisis_exploratorio.ipynb`

#### 5. Ejecutar el análisis
- Kernel: Python 3 (preseleccionado)
- Celda por celda: `Shift + Enter`
- O: Menú Cell → Run All Cells

#### 6. Detener el contenedor
```bash
# En otra terminal:
docker-compose down

# O presiona Ctrl+C en terminal donde corre docker-compose up
```

---

## 📊 Contenido del Notebook

### Secciones:

1. **Introducción y Contexto** (10%)
   - Descripción del dataset (fuente, significado, contexto)
   - Justificación estadística (por qué es relevante)
   - 3 preguntas de investigación específicas
   - Hipótesis preliminares

2. **Carga y Exploración Inicial** (15%)
   - `spark.read.csv()` para lectura
   - `printSchema()` con interpretación
   - Dimensiones y valores nulos
   - Limpieza de datos (reasoning)

3. **Estadística Descriptiva** (25%)
   - **Variables numéricas**: media, mediana, desv.est., IQR, skewness, kurtosis
   - Detección de outliers (criterio IQR)
   - **Variables categóricas**: frecuencias, modas, cardinalidad

4. **Spark SQL** (20%)
   - Consulta 1: Filtro WHERE + agregación
   - Consulta 2: GROUP BY complejo
   - Consulta 3: Funciones MIN/MAX/AVG
   - Explicación línea por línea de cada consulta
   - Interpretación contextualizada

5. **Correlaciones** (15%)
   - Matriz de correlación Pearson
   - Identificación de 3 relaciones más fuertes
   - Interpretación estadística
   - **Nota importante**: Correlación ≠ Causalidad (ejemplo contextualizado)

6. **Conclusiones** (15%)
   - Respuesta fundamentada a cada pregunta
   - Evaluación de hipótesis (confirmadas/refutadas)
   - Limitaciones del análisis (reales, no teóricas)
   - Propuestas de análisis futuro (nivel profesional)

---

## 📈 Hallazgos Principales

### Tasa de Cancelación: **36.8%**
- 44,224 cancelaciones de 119,390 reservas
- Crítico para planificación operacional

### Factores Clave de Cancelación:

| Factor | Efecto | Rango | Peso |
|--------|--------|-------|------|
| Tipo de Depósito | Muy Alto | 16%-40% | ★★★★★ |
| Canal de Distribución | Alto | 24%-41% | ★★★★ |
| Hotel Type | Alto | 30%-40% | ★★★★ |
| Composición de Grupo | Medio | 20%-45% | ★★★ |
| Lead Time | Débil (confundido) | 37%-39% | ★★ |
| Duración de Estancia | Muy Débil | -0.09 corr | ★ |

---

## 🔧 Troubleshooting

### Error: "Port 8888 is already allocated"
**Solución**: Cambiar puerto en `docker-compose.yml`
```yaml
ports:
  - "9999:8888"  # Usar puerto 9999 en host
```

### Error: "Cannot find dataset hotel_bookings.csv"
**Solución**: Verificar estructura
```bash
mkdir -p data
# Copiar hotel_bookings.csv a data/
ls data/hotel_bookings.csv
```

### Error: "Module pyspark not found"
**Solución**: Imagen puede estar desactualizada
```bash
docker-compose down
docker image rm jupyter/pyspark-notebook:latest
docker-compose up  # Descargará versión fresca
```

### Jupyter muy lento
**Solución**: Aumentar memoria en Docker Desktop
- Configuración → Resources → Memory: 4GB → 8GB

---

## 📚 Fuentes y Referencias

- **Dataset**: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand
- **PySpark Docs**: https://spark.apache.org/docs/latest/api/python/
- **Jupyter Docker**: https://jupyter-docker-stacks.readthedocs.io/
- **Docker Compose**: https://docs.docker.com/compose/

---

## 📝 Notas de Implementación

### Por qué Spark en lugar de Pandas?
- Dataset: 119K registros (manejable con pandas, pero)
- Preparación para escala: En producción, millones de registros
- Aprendizaje: Competencia laboral (Data Science requiere Spark)
- Distribuido: Permite procesamiento paralelo en multi-nodos

### Por qué Docker?
- **Reproducibilidad**: Mismo entorno en laptop, servidor, CI/CD
- **Portabilidad**: No afecta sistema host (versiones Python, librerías)
- **Colaboración**: Compañeros ejecutan exactamente tu código sin "en mi máquina funciona"
- **Scaling**: Mismo contenedor en 1 máquina o 1000 nodos Kubernetes

### Decisiones Metodológicas del Análisis

1. **No excluir outliers**: Son datos reales (reservas de larga duración, precios premium). Usar análisis robustos.
2. **Estratificar por hotel**: Resort y City tienen dinámicas tan diferentes que análisis agregado es engañoso.
3. **Enfoque multidimensional**: No solo cancelación. Entender patrones comportamentales y segmentación.
4. **Rigor estadístico**: Skewness, kurtosis, IQR, no solo media/desv.est. que fallan con datos asimétricos.

---

## 👨‍💻 Autor

**Estudiante**: Michael Morantes
**Curso**: Machine Learning con Pyspark y docker
**Universidad**: Universidad Santo Tomas  
**Fecha de Entrega**: Martes 17 de marzo, 2026  

---

## 📞 Contacto / Revisión por Pares

**Revisor compañero**: Daniela Murcia
- GitHub Issue/Comentario: Link
- Feedback: Resumen

---

## 📄 Licencia

Este proyecto es académico. Para propósitos educativos únicamente.

---

**Última actualización**: Martes 17/03/2026  
**Estado**: Completo y funcional ✓
