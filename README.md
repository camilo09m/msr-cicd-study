# Caracterización de rechazos en Pull Requests que modifican configuración de CI/CD generados por agentes de IA

---

## 1. Motivación

La creciente adopción de agentes de IA en flujos de desarrollo real hace que esta pregunta tenga consecuencias prácticas inmediatas. Los equipos que integran estos agentes no cuentan con evidencia empírica para decidir qué agente delegar para tareas CI/CD, qué aspectos de esos cambios revisar con mayor cuidado, ni cómo orientar al agente para producir configuraciones aceptables. A su vez, los mantenedores de proyectos se enfrentan a PRs CI/CD generados automáticamente sin disponer de patrones documentados que faciliten su revisión. Caracterizar los rechazos en estos PRs es un primer paso necesario para que tanto equipos como revisores puedan tomar decisiones informadas.

## 2. Problema

Hoy varios agentes de IA —Claude Code, Cursor, GitHub Copilot, OpenAI Codex y Devin— abren Pull Requests por su cuenta en proyectos reales. Una parte de esos PRs no toca el código del producto, sino los archivos que controlan la integración y el despliegue continuo: los workflows de GitHub Actions, el `.gitlab-ci.yml`, el Jenkinsfile. Son los archivos que deciden cómo se compila, cómo se prueba y cómo se despliega el proyecto.

Muchos de esos PRs no llegan a integrarse: el mantenedor revisa el cambio propuesto y lo cierra sin mergear. Es importante notar qué se puede y qué no se puede afirmar sobre esos cierres. No se puede afirmar que el cambio estuviera "roto" en sentido técnico, porque verificarlo exigiría ejecutar el pipeline con el cambio aplicado, y AIDev no incluye logs de ejecución de CI/CD. Lo que sí se puede observar es el diff del archivo CI/CD, los comentarios del revisor, el veredicto del review y el hecho de que el PR terminó cerrado sin merge. El objeto de estudio, entonces, no son fallos de ejecución, sino cambios rechazados durante la revisión: propuestas del agente que el mantenedor del proyecto decidió no aceptar.

Sobre esos rechazos existe poca evidencia empírica. No sabemos qué tipos de cambios rechazan los mantenedores con más frecuencia cuando provienen de agentes, si todos los agentes son rechazados por motivos parecidos o si cada uno tiene su propio perfil, ni si los humanos que tocan estos mismos archivos enfrentan los mismos motivos de rechazo o no.

## 3. Diseño metodológico

### 3.1 Objetivo

Caracterizar empíricamente los motivos por los cuales los Pull Requests que modifican configuración CI/CD generados por agentes de IA son cerrados sin merge, mediante el análisis del dataset AIDev y la construcción de una taxonomía inductiva validada por acuerdo intercodificador.

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

PI3. ¿Qué categorías de motivos de rechazo emergen al aplicar card sorting sobre los archivos CI/CD modificados en PRs rechazados, y cómo se distribuyen entre agentes?

PI4. ¿Cómo se comparan estos patrones con los PRs CI/CD escritos por humanos (human_pull_request)?

PI5. ¿Qué tan generalizable es la taxonomía obtenida más allá de la muestra analizada?

## 5. Resultados
