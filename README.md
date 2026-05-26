# Caracterización de fallos en Pull Requests que modifican configuración de CI/CD generados por agentes

---

## 1. Motivación



## 2. Problema

Los estudios recientes sobre PRs agénticos en AIDev abordan calidad de descripciones, intervención humana, esfuerzo de revisión y errores de pruebas, pero ninguno caracteriza los modos de fallo cuando el cambio toca configuración CI/CD. Específicamente, se desconoce: qué proporción de los PRs CI/CD agénticos termina cerrada sin merge, si esa tasa varía sistemáticamente entre agentes, qué señales del proceso de revisión predominan en los rechazos, qué tipos de defectos aparecen con más frecuencia en los archivos CI/CD modificados, y cómo se comparan estos patrones con PRs CI/CD escritos por humanos.

## 3. Diseño metodólogico

### 3.1 Objetivo
Caracterizar empíricamente los modos de fallo presentes en los Pull Requests que modifican configuración CI/CD generados por agentes de IA y que terminan cerrados sin merge, mediante el análisis del dataset AIDev y la construcción de una taxonomía inductiva validada por acuerdo intercodificador.

### 3.2 Conjunto de datos: pasos de construcción
Se utiliza la versión filtrada AIDev-pop (repositorios con ≥ 100 estrellas), por contar con señales de revisión más completas. Las tablas explotadas y sus columnas son:

| Tabla | Columnas explotadas |
|---|---|
| `pull_request` | `id`, `state`, `merged_at`, `closed_at`, `created_at`, `agent`, `repo_url`, `html_url`, `title`, `body` |
| `human_pull_request` | mismas columnas (baseline humano) |
| `pr_commit_details` | `pr_id`, `filename`, `status`, `additions`, `deletions`, `changes`, `message`, `patch` |
| `pr_reviews` | `pr_id`, `state`, `body` |
| `pr_review_comments_v2` | `pr_id`, `body` |
| `pr_comments` | `pr_id`, `body` |
| `pr_timeline` | `pr_id`, `event` |

La construcción del conjunto de trabajo se realiza con la siguiente cadena explícita de pasos. Las reducciones se reportan tras la ejecución; los órdenes de magnitud anticipados sirven de referencia operacional.

| Paso | Filtro | Unidad de salida |
|---|---|---|
| F0 | Universo: `pull_request` (33.6K PRs de agentes en AIDev-pop) | PRs |
| F1 | El PR modifica **al menos un** archivo CI/CD (matching de `pr_commit_details.filename`) | PRs |
| F2 | El PR está rechazado: `state == 'closed' AND merged_at IS NULL` | PRs |
| F3 | El PR es único y sin repeticiones | PRs |


### 3.2.1 Patrones de filename considerados CI/CD

```
.github/workflows/.*\.ya?ml         .gitlab-ci\.ya?ml
.circleci/config\.ya?ml             Jenkinsfile(\..+)?
azure-pipelines\.ya?ml              .travis\.ya?ml
bitbucket-pipelines\.ya?ml          .drone\.ya?ml
buildkite\.ya?ml                    appveyor\.ya?ml
```
### 3.2.3 Definición operacional de estado

- **Aceptado:** `merged_at IS NOT NULL`.
- **Rechazado (cerrado sin merge):** `state == 'closed' AND merged_at IS NULL`.
- **Abierto:** `state == 'open'` (excluido del análisis principal).

Se discute explícitamente en *Amenazas a la validez* que "cerrado sin merge" no equivale a "rechazado por defecto"; puede reflejar abandono, superación por otro PR o reorganización.

## 4. Preguntas



## 5. Resultados
