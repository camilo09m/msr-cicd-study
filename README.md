# Caracterización de fallos en Pull Requests de CI/CD generados por agentes mediante análisis de Pull Requests cerrados sin merge

---

## Descripción contexto

El contexto de este estudio corresponde al creciente uso de agentes de inteligencia artificial como asistentes o contribuidores dentro de flujos colaborativos de desarrollo de software. En particular, los agentes están comenzando a generar Pull Requests relacionados con configuraciones de integración y despliegue continuo (CI/CD), incluyendo workflows y archivos YAML utilizados en automatización de pipelines.

---

## Problema

El problema es que actualmente existe poca comprensión sobre los tipos de errores o fallos que presentan los Pull Requests relacionados con CI/CD generados por agentes. Aunque algunos PRs son aceptados, otros son rechazados por los maintainers, lo que sugiere problemas en configuraciones, sintaxis, estructura o compatibilidad de pipelines. Sin embargo, no existe una caracterización clara de estos fallos ni de los patrones presentes en los PRs rechazados.

---

## Propuesta

Para abordar el problema, se propone analizar Pull Requests asociados a CI/CD utilizando el dataset AIDev del Mining Challenge MSR. Primero, se realizará un filtrado de PRs relacionados con CI/CD mediante la identificación de archivos YAML y workflows modificados en los cambios del PR. Posteriormente, los PRs serán clasificados según su estado (aceptados o rechazados), enfocando el análisis principalmente en aquellos rechazados.

Finalmente, se aplicará una técnica de card sorting sobre los PRs rechazados para identificar y agrupar patrones de fallos recurrentes presentes en configuraciones de CI/CD generadas por agentes. Esto permitirá construir una caracterización empírica de los errores más comunes asociados a este tipo de tareas.

---

## Metodología

El estudio seguirá un enfoque empírico exploratorio con análisis cuantitativo y cualitativo.

### 1. Recolección y preparación de datos

Se utilizará el dataset AIDev del Mining Challenge MSR. Las tablas principales consideradas serán:

* `pull_request`
* `pr_commit_details`
* `pr_comments`
* `pr_reviews`
* `pr_timeline`

### 2. Filtrado de PRs relacionados con CI/CD

Se considerarán PRs relacionados con CI/CD aquellos que modifiquen archivos asociados a automatización, workflows y pipelines, incluyendo archivos

* `.yml`
* `.yaml`
* `.github/workflows/*`

La identificación se realizará utilizando los nombres de archivos disponibles en `pr_commit_details.filename`.

### 3. Clasificación de PRs

Los PRs serán clasificados según su estado:

* aceptados (`merged`)
* PRs cerrados sin merge (closed y merged_at = null)

En este estudio, dichos PRs serán utilizados como aproximación operacional de PRs rechazados.

### 4. Extracción de información

Para cada PR se analizarán variables observables del dataset, incluyendo:

* additions
* deletions
* changes
* archivos modificados
* comentarios
* reviews
* eventos asociados al PR

### 5. Card Sorting

Los PRs rechazados serán revisados manualmente mediante card sorting abierto realizado por los investigadores. El objetivo será identificar categorías emergentes de fallos relacionados con CI/CD.

Posibles categorías esperadas:

* errores de sintaxis YAML
* configuraciones incompletas
* dependencias incorrectas
* errores en workflows
* problemas de compatibilidad

Cada categoría identificada será documentada mediante una definición explícita y criterios de inclusión para reducir ambigüedades durante el proceso de clasificación.

### 6. Piloto

Antes del análisis completo, se realizará una revisión piloto sobre un subconjunto inicial de PRs para validar criterios de clasificación y unificar interpretaciones entre los investigadores.

### 7. Análisis de resultados

Finalmente, se analizarán las categorías identificadas, su frecuencia y los patrones presentes en los PRs rechazados para construir una caracterización de fallos en PRs de CI/CD generados por agentes.
