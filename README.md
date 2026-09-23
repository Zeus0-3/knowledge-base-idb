# knowledge-base-idb
Base de conocimiento para estudiantes: artículos organizados a partir de clases y PDFs, estructurados con el formato OKF de Google.

# KnowledgeDataBase-IDB — Guía de Configuración y Uso

## Qué es esto

Una **base de conocimiento persistente** para las clases del taller IDB. Un agente LLM lee las transcripciones (`.md`) de los videos, las libretas Jupyter (`.ipynb`) de análisis y **los PDFs de lecturas y fuentes confiables** (artículos científicos, normas, manuales técnicos). Extrae el contenido, lo estructura, genera referencias cruzadas y mantiene el conocimiento organizado a medida que se procesan más materiales.

La wiki sigue el patrón **LLM-Wiki** (Karpathy) y está escrita en el formato **Open Knowledge Format (OKF) v0.2** de Google: cada página es un archivo markdown con frontmatter YAML, lo que la hace legible por humanos, por Obsidian/GitHub y por cualquier agente, sin herramientas especiales.

### Principios clave:
- **La wiki es un artefacto persistente** — no respuestas temporales de chat
- **Las referencias cruzadas se construyen automáticamente** — conceptos, herramientas, técnicas y artículos se conectan entre clases
- **Cada afirmación de un artículo es rastreable** hasta el PDF original
- **El LLM hace el trabajo pesado** (transcribir, estructurar, vincular) — tú controlas la calidad
- **Tú apruebas todo antes de guardar** — human in the loop
- **La confianza es visible** — cada página indica si ya fue revisada por un humano

---

## Tu rol

1. Indicar qué contenido procesar (video, libreta o PDF)
2. Curar las fuentes: solo se ingieren PDFs que cumplan los criterios de confiabilidad
3. Revisar los borradores generados (resúmenes, conceptos, pasos, artículos)
4. Verificar los artículos contra su transcripción antes de marcarlos como revisados
5. Hacer preguntas que se guardan como nuevas páginas
6. Supervisar cómo crece la wiki con el tiempo

### Flujo Human-in-the-Loop:
- **PASO 1:** Indicas al LLM qué contenido ingerir (transcripción `.md`, libreta `.ipynb` o PDF) → el agente localiza el archivo
- **PASO 2:** El LLM procesa el contenido (lee transcripción / lee notebook con figuras / transcribe el PDF a texto crudo), extrae contenido, genera borrador estructurado
- **PASO 3:** El LLM te presenta el borrador para revisión
- **PASO 4:** Apruebas (o pides cambios) → el LLM guarda en `wiki/`
- **PASO 5:** El LLM actualiza el índice, el log y los enlaces cruzados con las clases

---

## Estructura de carpetas

```
KnowledgeDataBase-IDB/
│
├── README.md                      # Esta guía
├── LICENSE                        # CC BY-SA 4.0
│
├── schema/                        # Instrucciones para el agente — NO tocar
│   ├── AGENTS.md                 # Contrato maestro del agente
│   └── INGESTION_GUIDE.md        # Guía paso a paso del flujo de ingesta
│
├── raw/                           # Inmutable: solo AGREGAR contenido, nunca modificar
│   ├── videos/                   # Transcripciones (.md) de las clases en video
│   │   ├── 001 introducción al taller IDB.md
│   │   ├── ...
│   │   └── 008 Shading en ventanas.md
│   ├── notebooks/                # Libretas Jupyter de análisis (.ipynb)
│   │   ├── 001_EDA.ipynb
│   │   └── 002_EDA_EPW.ipynb
│   ├── pdfs/                     # NUEVO — PDFs originales de lecturas y fuentes
│   │   └── AAAA-Autor-TituloBreve.pdf
│   └── pdfs-transcripciones/     # NUEVO — texto crudo de cada PDF, 1 a 1
│       └── AAAA-Autor-TituloBreve.md
│
├── wiki/                          # Bundle OKF v0.2 — todo lo genera el LLM
│   ├── index.md                  # Catálogo de contenido (declara okf_version)
│   ├── log.md                    # Historial cronológico de cambios
│   │
│   ├── classes/                  # type: Clase          — NNN-TituloBreve.md
│   ├── concepts/                 # type: Concepto       — Balances-de-Calor.md
│   ├── tools/                    # type: Herramienta    — EnergyPlus.md
│   ├── procedures/               # type: Procedimiento  — Crear-Simulacion.md
│   └── articles/                 # type: Artículo       — NUEVO: AAAA-Autor-TituloBreve.md
│
├── queries/                       # Respuestas a tus preguntas
│   └── q-YYYY-MM-DD-Titulo.md
│
└── notes/                         # Tus notas personales
    └── ...
```

**Qué es parte del bundle OKF:** solo `wiki/`. La carpeta `raw/` queda fuera a propósito: las transcripciones son una copia fiel de la fuente, no conocimiento procesado, y no necesitan cumplir el formato. Los artículos de la wiki apuntan a ellas mediante el campo `sources`.

---

## Formato de las páginas (OKF v0.2)

Todas las páginas dentro de `wiki/` (excepto `index.md` y `log.md`) tienen dos partes:

1. **Frontmatter YAML** — metadatos estructurados entre `---`
2. **Cuerpo markdown** — el contenido

### Tipos de concepto

| Carpeta | `type` | Qué representa |
|---------|--------|----------------|
| `wiki/classes/` | `Clase` | Resumen de una clase del taller |
| `wiki/concepts/` | `Concepto` | Tema transversal que aparece en varias clases |
| `wiki/tools/` | `Herramienta` | Software o librería (EnergyPlus, OpenStudio, Python) |
| `wiki/procedures/` | `Procedimiento` | Receta paso a paso |
| `wiki/articles/` | `Artículo` | Síntesis de un PDF de fuente confiable, con citas rastreables |

### Campos del frontmatter

| Campo | Obligatorio | Uso en este proyecto |
|-------|-------------|----------------------|
| `type` | **Sí** (único obligatorio en OKF) | Uno de los cinco tipos de la tabla anterior |
| `title` | Recomendado | Nombre legible de la página |
| `description` | Recomendado | Una sola oración; se copia al `index.md` |
| `tags` | Recomendado | Temas transversales: `[confort-termico, aleros]` |
| `status` | Recomendado | `draft` (sin revisar), `stable` (aprobado), `deprecated` (obsoleto) |
| `generated` | Recomendado | Quién escribió el contenido y cuándo: `{ by: ..., at: ... }` |
| `verified` | Al aprobar | Quién lo revisó y cuándo: `{ by: human:<usuario>, at: ... }` |
| `sources` | **Sí para Artículos** | De qué materiales deriva la página |
| `resource` | Opcional | URL canónica del recurso (ej. DOI del artículo) |

Todas las fechas usan formato ISO 8601 con zona horaria: `2026-09-23T18:00:00Z`.

### Convención de actores

Los campos `generated.by` y `verified[].by` identifican quién hizo la acción:

- Agente: `claude/<modelo>` — ej. `claude/opus-5.5`
- Persona: `human:<usuario>` — ej. `human:tu-usuario-github`
- Proceso automático: `process:<nombre>` — ej. `process:lint-semanal`

El prefijo `human:` es importante: es lo que distingue una página revisada por una persona de una generada solo por el agente.

### Niveles de confianza

Se derivan automáticamente del campo `verified`:

| Estado | Condición | Significado para el alumno |
|--------|-----------|----------------------------|
| 🔴 **No verificado** | Sin campo `verified` | Borrador del agente; usar con cautela |
| 🟡 **Confirmado por máquina** | `verified` solo por `process:` o agentes | Pasó chequeos automáticos |
| 🟢 **Revisado por humano** | `verified` incluye un `human:` | Revisado contra la fuente original |

### Enlaces entre páginas

Se usan links markdown estándar con rutas relativas a `wiki/` (empiezan con `/`):

```markdown
Ver [Zona Térmica](/concepts/Zona-Termica.md) y la [Clase 006](/classes/006-DosZonasTermicasVentanasAleros.md).
```

Un link roto no es un error: puede representar conocimiento que todavía no se escribe. El mantenimiento periódico los reporta.

---

## Convención de nombres

### Clases
`NNN-TituloBreveCamelCase.md` — donde `NNN` coincide con el prefijo de la transcripción.

- `001-IntroduccionTallerIDB.md`
- `002-ConceptosBasicosBalancesCalor.md`
- `005-AnalisisSimulacionesPython.md`

### Artículos
`AAAA-ApellidoPrimerAutor-TituloBreveCamelCase` — donde `AAAA` es el año de publicación.

El **mismo nombre base** se usa en las tres ubicaciones, para que la trazabilidad sea obvia:

```
raw/pdfs/2002-DeDear-ConfortAdaptativo.pdf                  ← original
raw/pdfs-transcripciones/2002-DeDear-ConfortAdaptativo.md   ← texto crudo
wiki/articles/2002-DeDear-ConfortAdaptativo.md              ← artículo de la wiki
```

---

## Tipos de contenido ingerible

| Tipo | Ubicación | Formato | Qué se extrae |
|------|-----------|---------|---------------|
| **Transcripción de clase** | `raw/videos/*.md` | Markdown transcrito del video | Resumen, conceptos, procedimientos, herramientas |
| **Libreta Jupyter** | `raw/notebooks/*.ipynb` | `.ipynb` (se lee directamente con celdas y figuras) | Código, flujo de análisis, gráficas, hallazgos |
| **PDF de fuente confiable** | `raw/pdfs/*.pdf` | PDF (texto nativo o escaneado) | Transcripción cruda + artículo con ideas clave, citas y enlaces a clases |

---

## Criterios de confiabilidad para PDFs

Antes de agregar un PDF a `raw/pdfs/`, verifica que sea una de estas fuentes:

| Tipo de fuente | Ejemplos | Aceptado |
|----------------|----------|----------|
| Artículo revisado por pares | Revistas indexadas (Energy and Buildings, Building and Environment) | ✅ |
| Norma o estándar técnico | ASHRAE, ISO, NOM, CTE | ✅ |
| Documentación oficial de software | EnergyPlus Engineering Reference, manuales de OpenStudio | ✅ |
| Libro de texto o académico | Editoriales académicas, textos base del curso | ✅ |
| Tesis de posgrado | Repositorios institucionales | ⚠️ Con aprobación del profesor |
| Blog, presentación sin autor, apuntes de terceros | — | ❌ |

El tipo de fuente se registra en el frontmatter del artículo con el campo `tipo_fuente` (extensión propia del proyecto; OKF permite campos adicionales).

---

## Qué se genera al ingerir una transcripción de clase

Cuando apruebas una ingesta, el agente LLM:

1. **Crea página de clase** en `wiki/classes/NNN-Titulo.md` con `type: Clase`
   - Frontmatter: título, descripción, tags, `status: draft`, `generated`
   - Resumen estructurado del contenido
   - Conceptos clave explicados
   - Pasos prácticos / procedimientos mostrados en la clase
   - Timestamps de secciones importantes (cuando aparecen en la transcripción)
   - Referencias a clases anteriores y siguientes
   - Sección **Lecturas de apoyo** (se llena cuando hay artículos relacionados)

2. **Actualiza `index.md`** — agrega entrada al catálogo de clases

3. **Crea o actualiza páginas de conceptos** según sea necesario:
   - Si "balance de calor" aparece en varias clases → `wiki/concepts/Balances-de-Calor.md` se actualiza
   - Cada página de concepto enlaza a todas las clases relevantes

4. **Crea o actualiza páginas de herramientas** para software mencionado:
   - EnergyPlus, OpenStudio, Python, IDF Editor, etc.
   - Cómo se usa cada herramienta, en qué clases aparece

5. **Crea o actualiza páginas de procedimientos** para flujos paso a paso:
   - "Cómo crear tu primera simulación"
   - "Cómo interpretar mensajes de error"
   - Cada procedimiento referencia la clase donde se enseñó

6. **Busca artículos existentes relacionados** y los enlaza en "Lecturas de apoyo"

7. **Actualiza `log.md`** — entrada cronológica de la acción

---

## Qué se genera al ingerir una libreta Jupyter

Cuando apruebas la ingesta de un `.ipynb`, el agente LLM:

1. **Lee la libreta directamente** — celdas de código, markdown y **figuras incluidas** (las figuras embebidas en el output de las celdas son visibles para el LLM)
2. **Crea página de análisis** en `wiki/procedures/` o `wiki/classes/` según corresponda:
   - Descripción del objetivo del análisis
   - Flujo de trabajo paso a paso (qué hace cada bloque de código)
   - Descripción de las gráficas generadas y qué muestran
   - Librerías y funciones usadas (pandas, matplotlib, ear_tools, etc.)
   - Hallazgos o patrones observados en los datos
3. **Actualiza páginas de herramientas** (`wiki/tools/Python.md`, etc.) con funciones y patrones nuevos
4. **Actualiza `index.md`** y **`log.md`**

### Sobre las figuras

- El LLM puede **ver las figuras** directamente en el `.ipynb` (no se necesita exportarlas)
- Las figuras se **describen textualmente** en la wiki (qué ejes, qué variables, qué patrón se observa)
- **No se necesita convertir a `.py`** — eso pierde los outputs y las gráficas, que es lo más valioso
- Si se necesita un artefacto exportable: `uv run jupyter nbconvert --to markdown notebook.ipynb` genera un `.md` con las figuras como PNGs en una carpeta adjunta

### Ejemplo: ingerir una libreta

```markdown
Ingest: raw/notebooks/001_EDA.ipynb
```

El LLM:
1. Lee todas las celdas (código + outputs + figuras)
2. Genera un borrador con el flujo de análisis, descripción de gráficas y hallazgos
3. Tú revisas y apruebas → se guarda en la wiki

---

## Qué se genera al ingerir un PDF

La ingesta de un PDF tiene **dos fases separadas**, cada una con su propia aprobación. Separarlas evita que un error de transcripción se convierta en un "hecho" dentro de la wiki.

### Fase A — Transcripción cruda (PDF → `raw/pdfs-transcripciones/`)

El agente convierte el PDF a texto markdown **sin interpretar, resumir ni corregir**:

1. Extrae el texto completo en el orden original
2. Marca el inicio de cada página: `<!-- página 3 -->`
3. Conserva títulos, tablas y ecuaciones lo más fiel posible
4. Marca lo que no pudo leer: `[ilegible]`, `[figura: descripción breve]`
5. Si el PDF es escaneado (imagen, sin texto seleccionable), aplica OCR y lo indica en el encabezado
6. Agrega un encabezado mínimo de trazabilidad:

```markdown
<!--
fuente: raw/pdfs/2002-DeDear-ConfortAdaptativo.pdf
paginas: 18
metodo: texto-nativo        # o: ocr
transcrito: 2026-09-23T18:00:00Z
transcrito_por: claude/opus-5.5
-->
```

Tú revisas una muestra de páginas contra el PDF y apruebas. **A partir de aquí la transcripción es inmutable**, igual que el resto de `raw/`.

### Fase B — Artículo de la wiki (transcripción → `wiki/articles/`)

El agente lee la transcripción aprobada y:

1. **Crea el artículo** en `wiki/articles/AAAA-Autor-Titulo.md` con `type: Artículo` y `status: draft`
2. **Registra las fuentes** en `sources`: la transcripción (ruta relativa) y el original (DOI o URL)
3. **Cita cada afirmación importante** con una nota al pie ligada a un `sources[].id`
4. **Enlaza con la wiki existente**: clases, conceptos, herramientas y procedimientos donde el tema aparece
5. **Actualiza las clases relacionadas** — agrega el artículo a su sección "Lecturas de apoyo"
6. **Actualiza los conceptos relacionados** — agrega el artículo a su sección "Fuentes y lecturas"
7. **Señala contradicciones** si el artículo dice algo distinto a lo que ya está en la wiki (no las resuelve solo: te las presenta)
8. **Actualiza `index.md`** y **`log.md`**

### Ejemplo: ingerir un PDF

```markdown
Ingest: raw/pdfs/2002-DeDear-ConfortAdaptativo.pdf
```

Borrador que presenta el agente en la Fase B:

```markdown
---
type: Artículo
title: Modelo de confort térmico adaptativo
description: Síntesis del modelo adaptativo y su aplicación al análisis de zonas térmicas.
tags: [confort-termico, zonas-termicas, simulacion]
tipo_fuente: articulo-revisado-por-pares
resource: https://doi.org/<doi-del-articulo>
status: draft
generated: { by: claude/opus-5.5, at: 2026-09-23T18:30:00Z }
sources:
  - id: transcripcion
    resource: ../../raw/pdfs-transcripciones/2002-DeDear-ConfortAdaptativo.md
    title: Transcripción cruda del PDF
  - id: original
    resource: https://doi.org/<doi-del-articulo>
    title: <Título completo del artículo>
    author: <Autores>
    last_modified: 2002-01-01T00:00:00Z
---

# Resumen
<Idea central del artículo en 3-5 oraciones.>[^transcripcion]

# Ideas clave
- <Idea 1>[^transcripcion]
- <Idea 2>[^transcripcion]

# Relación con el taller
- Complementa [Zona Térmica](/concepts/Zona-Termica.md)
- Lectura de apoyo para la [Clase 002](/classes/002-ConceptosBasicosBalancesCalor.md)
- Útil al interpretar resultados en [Análisis con Python](/classes/005-AnalisisSimulacionesPython.md)

# Preguntas para el alumno
1. <Pregunta de comprensión>
2. <Pregunta que conecta con una simulación del curso>

[^transcripcion]: Transcripción cruda del PDF, páginas <n–m>
[^original]: <Referencia completa en formato APA>

---
¿Es correcto? ¿Algún cambio antes de guardar?
```

### Aprobar un artículo (de borrador a revisado)

Cuando revisas el artículo contra la transcripción y confirmas que es fiel, pides al agente:

```markdown
Verificar: wiki/articles/2002-DeDear-ConfortAdaptativo.md
```

El agente agrega tu verificación y cambia el estado:

```yaml
status: stable
verified: { by: human:<tu-usuario>, at: 2026-09-24T10:00:00Z }
```

Solo una persona puede marcar un artículo como revisado. El agente **nunca** agrega un `verified` con prefijo `human:` por su cuenta.

---

## Cómo ingerir una transcripción de clase

### Ejemplo: primera clase

```markdown
Ingest: raw/videos/001 introducción al taller IDB.md
```

El LLM:
1. Lee la transcripción completa
2. Extrae el contenido estructurado (temas, explicaciones, demostraciones)
3. Genera un borrador:

```markdown
---
type: Clase
title: "001 — Introducción al Taller IDB"
description: Presentación del curso, objetivos y herramientas necesarias.
tags: [presentacion, simulacion-energetica]
status: draft
generated: { by: claude/opus-5.5, at: 2026-09-23T18:00:00Z }
sources:
  - id: video-001
    resource: ../../raw/videos/001 introducción al taller IDB.md
    title: Transcripción de la clase 001
---

# Resumen
[Descripción estructurada del contenido de la clase]

# Conceptos clave
- **IDB:** [qué es, para qué sirve]
- **Simulación energética:** [introducción al concepto] → [Simulación Energética](/concepts/Simulacion-Energetica.md)

# Herramientas mencionadas
- [EnergyPlus](/tools/EnergyPlus.md), [OpenStudio](/tools/OpenStudio.md)

# Lecturas de apoyo
- (se llena al ingerir artículos relacionados)

# Conexiones con otras clases
- → Siguiente: [002 — Conceptos Básicos y Balances de Calor](/classes/002-ConceptosBasicosBalancesCalor.md)

---
¿Es correcto? ¿Algún cambio antes de guardar?
```

Tú revisas y apruebas → el LLM guarda la página, actualiza el índice, crea páginas de conceptos y herramientas, registra en el log.

### Clase siguiente (acumula conocimiento)

```markdown
Ingest: raw/videos/002 Conceptos Basicos y Balances de Calor.md
```

El LLM procesa y detecta que "simulación energética" ya existe como concepto...
- Crea `wiki/classes/002-ConceptosBasicosBalancesCalor.md`
- Actualiza `wiki/concepts/Simulacion-Energetica.md` — ahora referencia ambas clases
- Crea `wiki/concepts/Balances-de-Calor.md` — nuevo concepto
- Revisa si hay artículos en `wiki/articles/` sobre balances de calor y los enlaza
- La página de concepto muestra la progresión entre clases

---

## Archivos especiales: `index.md` y `log.md`

### `wiki/index.md`

Es el único lugar donde un `index.md` lleva frontmatter, y solo para declarar la versión del formato:

```markdown
---
okf_version: "0.2"
---

# Clases

* [001 — Introducción al Taller IDB](classes/001-IntroduccionTallerIDB.md) - Presentación del curso y herramientas

# Conceptos

* [Balances de Calor](concepts/Balances-de-Calor.md) - Flujos de energía en una zona térmica

# Artículos

* [Modelo de confort térmico adaptativo](articles/2002-DeDear-ConfortAdaptativo.md) - 🟢 Revisado · Artículo revisado por pares
```

Cada entrada usa la `description` del frontmatter de la página. En los artículos se agrega el nivel de confianza (🔴 / 🟡 / 🟢).

### `wiki/log.md`

Lista de entradas agrupadas por fecha, **la más reciente primero**:

```markdown
# Historial de la wiki

## 2026-09-24
* **Verificación**: [Modelo de confort adaptativo](/articles/2002-DeDear-ConfortAdaptativo.md) revisado por human:<tu-usuario>.

## 2026-09-23
* **Ingesta**: PDF [Modelo de confort adaptativo](/articles/2002-DeDear-ConfortAdaptativo.md); enlazado a clases 002 y 005.
* **Transcripción**: raw/pdfs-transcripciones/2002-DeDear-ConfortAdaptativo.md (18 páginas, texto nativo).
```

Palabras iniciales usadas: **Ingesta**, **Transcripción**, **Creación**, **Actualización**, **Verificación**, **Deprecación**, **Mantenimiento**.

---

## Consultar la wiki (después de varias clases)

Puedes hacer preguntas como:

```markdown
"¿Qué pasos se siguen para crear una simulación desde cero?"
"¿En qué clases se habla de zonas térmicas?"
"Resumen de todo lo relacionado con shading y aleros"
"¿Qué herramientas de Python se usan para analizar simulaciones?"
"¿Qué dicen las fuentes revisadas sobre el confort adaptativo?"
"¿Qué lecturas de apoyo hay para la clase 007?"
```

El LLM lee primero `wiki/index.md`, encuentra las páginas relevantes, las lee y sintetiza una respuesta con referencias a las clases y citas a los artículos. Indica el nivel de confianza de cada artículo que usa. Si la respuesta es sustancial, ofrece guardarla como página nueva en `queries/`.

---

## Progresión del taller (transcripciones disponibles)

| # | Transcripción | Temas esperados |
|---|---------------|----------------|
| 001 | Introducción al taller IDB | Presentación, objetivos, herramientas |
| 002 | Conceptos Básicos y Balances de Calor | Fundamentos térmicos, balances energéticos |
| 003 | Mi Primera Simulación | Primer modelo, configuración básica |
| 004 | Interpretando mensajes y construction sets | Debugging, conjuntos de construcción |
| 005 | Primer Análisis con Python | Scripts de análisis, visualización de resultados |
| 006 | 2 Zonas Térmicas con Ventanas y Aleros | Multi-zona, elementos de fachada |
| 007 | Caso base y aleros | Caso de referencia, estrategias de sombreado |
| 008 | Shading en ventanas | Dispositivos de control solar |

---

## Entorno de desarrollo — uv

Este proyecto usa **[uv](https://docs.astral.sh/uv/)** como gestor de paquetes y entorno Python. Todo se ejecuta a través de `uv`:

| Acción | Comando |
|--------|---------|
| Agregar una dependencia | `uv add nombre-paquete` |
| Ejecutar un script Python | `uv run script.py` |
| Ejecutar main.py | `uv run main.py` |
| Agregar dependencia de desarrollo | `uv add --dev nombre-paquete` |

**Reglas:**
- **Nunca** usar `pip install` — siempre `uv add`
- **Nunca** usar `python script.py` — siempre `uv run script.py`
- `uv` maneja automáticamente el entorno virtual, las versiones de Python y el lockfile
- Las dependencias quedan registradas en `pyproject.toml`

---

## Qué NO hacer

- No modificar archivos en `wiki/` directamente — solo el agente LLM los actualiza
- No borrar archivos de `wiki/concepts/`, `wiki/tools/`, `wiki/procedures/` o `wiki/articles/` sin razón — si algo quedó obsoleto, se marca `status: deprecated`
- No renombrar ni editar archivos en `raw/` (videos, notebooks, PDFs o transcripciones) — son la fuente inmutable
- No agregar PDFs que no cumplan los criterios de confiabilidad
- No marcar un artículo como revisado (`verified: human:...`) sin haberlo comparado contra su transcripción

## Qué SÍ hacer

- Agregar nuevas transcripciones a `raw/videos/`, libretas a `raw/notebooks/` y PDFs a `raw/pdfs/` (con el nombre `AAAA-Autor-TituloBreve.pdf`)
- Revisar borradores antes de aprobar
- Revisar una muestra de páginas de cada transcripción de PDF contra el original
- Hacer preguntas — las respuestas sustanciales se convierten en páginas
- Pedir mantenimiento periódico de la wiki

---

## Mantenimiento periódico

Cada cierto tiempo, pide al agente:

```markdown
"Revisa la wiki — busca páginas huérfanas, conceptos sin conectar, procedimientos incompletos y artículos sin revisar"
```

El agente:
- Encuentra páginas sin referencias → sugiere consolidar o eliminar
- Identifica conceptos que necesitan páginas propias
- Verifica que los procedimientos estén completos y actualizados
- Sugiere conexiones faltantes entre clases
- Verifica que toda página tenga frontmatter con `type` (requisito de OKF)
- Lista artículos en `status: draft` o sin `verified` humano
- Detecta artículos sin `sources`, o fuentes declaradas que nunca se citan en el cuerpo
- Detecta notas al pie cuyo identificador no existe en `sources`
- Detecta artículos que ninguna clase enlaza en "Lecturas de apoyo"
- Detecta PDFs en `raw/pdfs/` sin transcripción, o transcripciones sin artículo (ingesta pendiente)
- Señala contradicciones entre artículos, o entre un artículo y una página de clase

---

## Migración de páginas existentes

Las páginas creadas antes de adoptar OKF v0.2 se actualizan poco a poco, sin reescribir su contenido:

1. Agregar frontmatter con al menos `type`, `title` y `description`
2. Agregar `status: stable` si ya fueron aprobadas en su momento
3. Convertir los enlaces `[[wikilink]]` a links markdown estándar (`[texto](/ruta.md)`)
4. Agregar la sección "Lecturas de apoyo" a cada clase

Pídelo con: `"Migra wiki/classes/ al formato OKF v0.2"` (una carpeta a la vez, con revisión).

---

## Para empezar ahora

**Siguiente paso:** indica qué contenido quieres procesar:

```markdown
# Ingerir la transcripción de una clase
Ingest: raw/videos/001 introducción al taller IDB.md

# Ingerir una libreta Jupyter
Ingest: raw/notebooks/001_EDA.ipynb

# Ingerir un PDF de fuente confiable
Ingest: raw/pdfs/2002-DeDear-ConfortAdaptativo.pdf
```

El agente procesará el contenido, extraerá y estructurará la información, te presentará un borrador → tú apruebas → todo se guarda.

---

## Referencias

- Andrej Karpathy — [LLM Wiki (patrón)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Google Cloud — [Open Knowledge Format, especificación v0.2](https://github.com/GoogleCloudPlatform/open-knowledge-format)
