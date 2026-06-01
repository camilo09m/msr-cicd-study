# Caracterización de fallos en Pull Requests que modifican configuración de CI/CD generados por agentes de IA

---

## 1. Motivación

La creciente adopción de agentes de IA en flujos de desarrollo real hace que esta pregunta tenga consecuencias prácticas inmediatas. Los equipos que integran estos agentes no cuentan con evidencia empírica para decidir qué agente delegar para tareas CI/CD, qué aspectos de esos cambios revisar con mayor cuidado, ni cómo orientar al agente para producir configuraciones aceptables. A su vez, los mantenedores de proyectos se enfrentan a PRs CI/CD generados automáticamente sin disponer de patrones documentados que faciliten su revisión. Caracterizar los fallos en estos PRs es un primer paso necesario para que tanto equipos como revisores puedan tomar decisiones informadas.

## 2. Problema

Cuando un agente de IA propone un cambio sobre un archivo de configuración CI/CD —un workflow de GitHub Actions, un `.gitlab-ci.yml`, un Jenkinsfile— ese cambio puede contener defectos de distinta naturaleza: una sintaxis YAML inválida, una referencia a una acción deprecada, la eliminación injustificada de un job de pruebas, un secret expuesto en un step, un trigger que rompe la convención del proyecto. Estos defectos son detectados durante la revisión del Pull Request, por un revisor humano o por un check automatizado del propio repositorio, y se traducen en que el PR termina cerrado sin merge. Este estudio se ocupa de esa clase de fallo: defectos en el cambio propuesto al archivo CI/CD, observables en el diff y en la discusión del PR, que llevan al rechazo antes de que el cambio se integre. Quedan fuera de alcance los fallos en tiempo de ejecución del pipeline tras el merge, porque AIDev no incluye logs de ejecución de los sistemas CI/CD.

Sobre estos fallos de revisión existe poca evidencia empírica en el contexto de PRs generados por agentes. Específicamente, se desconoce qué tipos de defectos aparecen, con qué frecuencia relativa, si esa distribución varía entre los cinco agentes del dataset (Claude Code, Cursor, GitHub Copilot, OpenAI Codex, Devin) y si el perfil de fallos difiere del que producen los autores humanos sobre el mismo dominio.

## 3. Diseño metodológico

### 3.1 Objetivo

Caracterizar empíricamente los modos de fallo presentes en los Pull Requests que modifican configuración CI/CD generados por agentes de IA y que terminan cerrados sin merge, mediante el análisis del dataset AIDev y la construcción de una taxonomía inductiva validada por acuerdo intercodificador.

### 3.2 Conjunto de datos

Se utiliza la variante AIDev-pop (repositorios con ≥ 100 estrellas), por contar con señales de revisión más completas. Las tablas explotadas y sus columnas relevantes son:

| Tabla | Columnas explotadas |
|---|---|
| `pull_request` | `id`, `state`, `merged_at`, `closed_at`, `created_at`, `agent`, `repo_url`, `html_url`, `title`, `body` |
| `human_pull_request` | mismas columnas (baseline humano) |
| `pr_commit_details` | `pr_id`, `filename`, `status`, `additions`, `deletions`, `changes`, `message`, `patch` |
| `pr_reviews` | `pr_id`, `state`, `body` |
| `pr_review_comments_v2` | `pr_id`, `body` |
| `pr_comments` | `pr_id`, `body` |
| `pr_timeline` | `pr_id`, `event` |

#### 3.2.1 Cadena de filtros

El conjunto de trabajo se construye aplicando los siguientes pasos en orden. Los volúmenes indicados son de referencia operacional; los valores definitivos se reportan tras la ejecución.

| Paso | Filtro | Resultado esperado |
|---|---|---|
| F0 | Universo: tabla `pull_request` (AIDev-pop) | ~33 600 PRs |
| F1 | El PR modifica al menos un archivo CI/CD (`pr_commit_details.filename`) | ~2 376 PRs |
| F2 | PR rechazado: `state == 'closed' AND merged_at IS NULL` | ~454 PRs |
| F3 | Deduplicación: un registro por PR | ~454 PRs únicos |
| F4 | El PR modifica exactamente un archivo CI/CD (`ci_n_files == 1`) | ~269 PRs |

#### 3.2.2 Patrones de filename considerados CI/CD

```
.github/workflows/.*\.ya?ml         .gitlab-ci\.ya?ml
.circleci/config\.ya?ml             Jenkinsfile(\..+)?
azure-pipelines\.ya?ml              .travis\.ya?ml
bitbucket-pipelines\.ya?ml          .drone\.ya?ml
buildkite\.ya?ml                    appveyor\.ya?ml
```

#### 3.2.3 Definición operacional de estado

- **Aceptado:** `merged_at IS NOT NULL`.
- **Rechazado (cerrado sin merge):** `state == 'closed' AND merged_at IS NULL`.
- **Abierto:** `state == 'open'` (excluido del análisis principal).

Se discute explícitamente en *Amenazas a la validez* que "cerrado sin merge" no equivale a "rechazado por mala calidad"; puede reflejar abandono, superación por otro PR o reorganización.

## 4. Preguntas

A partir del conjunto filtrado se formulan cinco preguntas:

PI1. ¿Cuál es la tasa de rechazo de los PRs que incluyen modificaciones CI/CD generados por agentes y cómo varía entre los cinco agentes del dataset?

PI2. ¿Qué señales del proceso de revisión (pr_reviews.state, eventos de pr_timeline, volumen de pr_comments y pr_review_comments_v2, tamaño del cambio en los archivos CI/CD) caracterizan a los PRs CI/CD rechazados frente a los aceptados?

PI3. ¿Qué categorías de fallo emergen al aplicar card sorting sobre los archivos CI/CD modificados en PRs rechazados, y cómo se distribuyen entre agentes?

PI4. ¿Cómo se comparan estos patrones con los PRs CI/CD escritos por humanos (human_pull_request)?

PI5. ¿Qué tan generalizable es la taxonomía obtenida más allá de la muestra analizada?

## 5. Resultados
