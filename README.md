# Caracterización de rechazos en Pull Requests que modifican configuración de CI/CD generados por agentes de IA

---

## 1. Motivación

La creciente adopción de agentes de IA en flujos de desarrollo real hace que esta pregunta tenga consecuencias prácticas inmediatas. Los equipos que integran estos agentes no cuentan con evidencia empírica para decidir qué agente delegar para tareas CI/CD, qué aspectos de esos cambios revisar con mayor cuidado, ni cómo orientar al agente para producir configuraciones aceptables. A su vez, los mantenedores de proyectos se enfrentan a PRs CI/CD generados automáticamente sin disponer de patrones documentados que faciliten su revisión. Caracterizar los rechazos en estos PRs es un primer paso necesario para que tanto equipos como revisores puedan tomar decisiones informadas.

## 2. Problema

Los agentes de IA generan cambios sobre configuración CI/CD que son cerrados sin merge. Sin embargo, no existe una razon de los motivos de rechazo ni evidencia de si esos motivos difieren entre agentes.

## 3. Objetivo de la Investigación

### 3.1 Objetivo General

 Caracterizar empíricamente los motivos por los cuales los PRs que modifican configuración CI/CD generados por agentes son cerrados sin merge. 


## 4. Pregunta de Investigación
- ¿Qué categorías de motivos de rechazo emergen al aplicar card sorting sobre los archivos CI/CD modificados en PRs rechazados, y cómo se distribuyen entre agentes?

## 5. Diseño Metodológico

### 5.1 Conjunto de Datos

#### 5.1.1 Fuente de datos

Se utiliza la variante AIDev-pop (repositorios con ≥ 100 estrellas), por contar con señales de revisión más completas.

#### 5.1.2 Tablas utilizadas

| Tabla | Columnas explotadas |
|---|---|
| `pull_request` | `id`, `state`, `merged_at`, `closed_at`, `created_at`, `agent`, `repo_url`, `html_url`, `title`, `body` |
| `human_pull_request` | mismas columnas (baseline humano) |
| `pr_commit_details` | `pr_id`, `filename`, `status`, `additions`, `deletions`, `changes`, `message`, `patch` |
| `pr_reviews` | `pr_id`, `state`, `body` |
| `pr_review_comments_v2` | `pr_id`, `body` |
| `pr_comments` | `pr_id`, `body` |
| `pr_timeline` | `pr_id`, `event` |

#### 5.1.3 Cadena de filtros

El conjunto de trabajo se construye aplicando los siguientes pasos en orden.
| Paso | Filtro | Resultado | % del universo (F0) |
|---|---|---|---|
| F0 | Universo: tabla `pull_request` (AIDev-pop) | ~33 600 PRs | 100 % |
| F1 | El PR modifica al menos un archivo CI/CD (`pr_commit_details.filename`) | ~2 376 PRs | ~7,1 % |
| F2 | PR rechazado: `state == 'closed' AND merged_at IS NULL` | ~454 PRs | ~1,4 % |
| F3 | Deduplicación: un registro por PR | ~454 PRs únicos | ~1,4 % |
| F4 | El PR modifica exactamente un archivo CI/CD (`ci_n_files == 1`) | ~269 PRs | ~0,8 % |

#### 5.1.4 Patrones de filename considerados CI/CD

```text
.github/workflows/.*\.ya?ml
.gitlab-ci\.ya?ml
.circleci/config\.ya?ml
Jenkinsfile(\..+)?
azure-pipelines\.ya?ml
.travis\.ya?ml
bitbucket-pipelines\.ya?ml
.drone\.ya?ml
buildkite\.ya?ml
appveyor\.ya?ml
```

#### 5.1.5 Definición operacional de estado

- **Aceptado:** `merged_at IS NOT NULL`.
- **Rechazado (cerrado sin merge):** `state == 'closed' AND merged_at IS NULL`.
- **Abierto:** `state == 'open'` (excluido del análisis principal).

Se discute explícitamente en *Amenazas a la Validez* que "cerrado sin merge" no equivale necesariamente a "rechazado por mala calidad"; puede reflejar abandono, superación por otro PR o reorganización del proyecto.

### 5.2 Metodología para la Construcción de la Taxonomía

#### 5.2.1 Fundamentación metodológica

La pregunta de investigación se aborda mediante *card sorting*, una técnica cualitativa para derivar temas y taxonomías a partir de texto, ampliamente usada en la comunidad de Ingeniería de Software para crear modelos mentales y derivar taxonomías a partir de datos [4].

El *card sorting* propiamente tal se fundamenta en Zimmermann [4] y en su aplicación por Begel y Zimmermann, quienes agrupan 679 preguntas abiertas en 12 categorías y destilan un catálogo de 145 preguntas [3]. De forma complementaria se incorporan procedimientos de técnicas cualitativas adyacentes —no de *card sorting* en sentido estricto—: la construcción de taxonomías mediante *etiquetado manual* con doble codificación de Wen et al. [1] y la *codificación abierta* consolidada en un *codebook* de Liang et al. [2].

#### 5.2.2 Material por tarjeta (*card*)

Siguiendo el principio de "un pensamiento por tarjeta" [4], cada tarjeta corresponde a un PR rechazado y reúne únicamente la evidencia necesaria para juzgar el motivo del rechazo.

- **Evidencia primaria (cambio CI/CD):** `ci_filenames`, `ci_patches`, `ci_commit_messages`. Se analiza el *diff* real y no solo el mensaje, tal como Wen et al. inspeccionan el código de cada commit y su predecesor en lugar de limitarse al mensaje del commit [1].
- **Contexto del PR:** `title`, `body`.
- **Señales de revisión:** `pr_reviews.body`, `pr_review_comments_v2.body`, `pr_comments.body` y eventos de `pr_timeline`.
- **Identificador y *demographics*:** `pr_id`, `agent`, `repo_url`. El identificador permite vincular posteriormente cada tarjeta con su agente para el análisis cuantitativo descriptivo.

#### 5.2.3 Tipo de card sort y protocolo

Se emplea un *card sort* híbrido [4]: abierto en una primera fase para permitir que emerjan categorías desde los datos y cerrado en una segunda fase para clasificar el resto de las tarjetas con una taxonomía consolidada.

El proceso contempla tres etapas:

1. **Preparación.** Se generan las tarjetas a partir de `ci_cd_pr_filtered.csv`, donde cada registro corresponde a un PR e incluye la evidencia descrita anteriormente.
2. **Ejecución.**
   - **Card sorting abierto:** dos codificadores clasifican de forma independiente la primera mitad de la muestra sin categorías predefinidas. Cada tarjeta puede iniciar un grupo nuevo o asignarse a uno existente, permitiendo fusionar, dividir o redefinir categorías durante el proceso. Las tarjetas sin sentido o fuera de alcance se asignan al grupo *Discard* [4].
   - **Consolidación del esquema:** los codificadores concilian sus grupos mediante discusión abierta y construyen una taxonomía cerrada compartida, fusionando categorías conceptualmente similares en un único libro de códigos (*codebook*), siguiendo una estrategia similar a la utilizada por Liang et al. [2].
   - **Card sorting cerrado:** ambos codificadores clasifican la segunda mitad de la muestra utilizando exclusivamente la taxonomía consolidada.
3. **Análisis.** Se revisa la consistencia interna de los grupos obtenidos y, mediante diagramas de afinidad, se derivan categorías raíz a partir de las categorías de detalle, siguiendo estrategias similares a las reportadas por Begel y Zimmermann [3] y Wen et al. [1].

#### 5.2.4 Acuerdo intercodificador

Cada tarjeta es clasificada por dos codificadores de forma independiente y las discrepancias se resuelven mediante un tercer codificador, replicando el procedimiento utilizado por Wen et al. [1].

Como criterio cuantitativo de acuerdo se exige **Cohen's κ ≥ 0,6**. Si κ < 0,6, se revisará el esquema de codificación antes de continuar.

Este umbral corresponde a una decisión metodológica propia del estudio. Ninguna de las referencias utilizadas reporta Cohen's κ: Wen et al. informan únicamente porcentaje de desacuerdo [1], Liang et al. no reportan acuerdo intercodificador [2], y Begel y Zimmermann realizan clasificación conjunta hasta alcanzar consenso [3].

#### 5.2.5 Categorías semilla

Para el card sort cerrado se considera inicialmente el siguiente conjunto de categorías derivadas de la literatura y del dominio CI/CD:

- Errores de sintaxis YAML.
- Referencias a *actions*, imágenes o *runners* inexistentes.
- Manejo incorrecto de *secrets* o *permissions*.
- Problemas de matriz de versiones.
- Cambios en *triggers* (`on:`).
- Duplicación o eliminación de workflows.
- Cambios fuera de alcance.
- Configuraciones no portables.

Estas categorías funcionan únicamente como punto de partida y podrán modificarse durante la fase abierta.

#### 5.2.6 Estrategia de muestreo

El archivo de trabajo exportado, `ci_cd_pr_filtered.csv`, contiene actualmente 269 PRs rechazados con exactamente un archivo CI/CD (salida del filtro F4), distribuidos por agente de la siguiente forma:

| Agente | Cantidad |
|---|---|
| Devin | 107 |
| OpenAI Codex | 83 |
| Copilot | 58 |
| Cursor | 14 |
| Claude Code | 7 |

Esta distribución impone dos restricciones respecto de los objetivos de muestreo:

- El total de 269 tarjetas queda por debajo del mínimo de 300 tarjetas definido para el estudio.
- Cursor (14) y Claude Code (7) no alcanzan el mínimo de 30 PRs por agente.

Por esta razón, el card sorting se realizará sobre la población F3 (PRs rechazados con uno o más archivos CI/CD; aproximadamente 454 casos según la cadena de filtros), utilizando F4 únicamente como subconjunto auxiliar de menor ruido.

Las tarjetas se barajarán antes de la codificación para reducir efectos de orden [2].

#### 5.2.7 Consideraciones metodológicas

La distribución de categorías por agente se reportará de forma descriptiva.

Se evita sobrecuantificar datos cualitativos: la ausencia de una categoría en un PR no implica necesariamente la ausencia del fenómeno, sino únicamente que dicho motivo no se manifestó explícitamente en ese caso [4].

## 6. Estado de Avance

### 6.1 Fase completada: Card Sorting Abierto 

Se codificaron de forma independiente 50 PRs (Copilot:  distribuidos entre dos codificadores. Esta fase corresponde al card sort abierto descrito en §5.2.3: ambos codificadores asignaron etiquetas libremente sin categorías predefinidas.

| Métrica | Valor |
|---|---|
| Tarjetas codificadas | 50 |
| Codificadores | 2 (Camilo, Gonzalo) |
| PRs en acuerdo | 30 (60,0 %) |
| PRs en desacuerdo | 20 (40,0 %) |
| Acuerdo observado (P₀) | 0,60 |


### 6.2 Codebook consolidado (v1.0)

A partir del card sorting abierto, se construyó un codebook de 17 categorías mediante discusión directa entre codificadores, siguiendo la estrategia de Liang et al. [2].

#### Familia 1 — Fallas de CI/CD

| Código | Nombre | Definición breve |
|---|---|---|
| **C01** | CI Failure General | El pipeline falló por errores introducidos o no anticipados por el PR (runner inválido, rebase incorrecto, fallo no resuelto sin causa clara). |
| **C02** | CI Failure — Cobertura Insuficiente | El pipeline bloqueó el merge porque la cobertura de pruebas no alcanza el umbral mínimo configurado (coverage gate). |
| **C03** | CI Failure — Fallos Masivos en Validaciones | El pipeline presenta fallos generalizados en múltiples pasos de validación (lint, tests, build, deploy) sin que una causa única sea dominante. |
| **C04** | CI Failure — Nuevo Workflow sin Auto-validar | El PR introduce un nuevo workflow de CI/CD que falla en su propia ejecución inicial, sin correcciones antes del cierre. |
| **C05** | CI Failure — Migración / Compatibilidad de Build | El pipeline se rompe durante una actualización de herramientas de build, versiones de dependencias o estrategia CI multi-módulo. |
| **C06** | CI Failure — Governance Gate | El cierre se debe a que el PR no completó un proceso de gobernanza obligatorio: aprobación de revisores, PRLint, firmas de autoría, etc. |
| **C07** | CI Failure — Optimización Incorrecta del Pipeline | El PR modifica la estrategia de ejecución de pruebas (paralelismo, matriz, runners) de forma incorrecta, causando fallo del pipeline. |

#### Familia 2 — Rechazo en Revisión

| Código | Nombre | Definición breve |
|---|---|---|
| **C08** | Rechazo por Diseño o Calidad del Código | El mantenedor rechaza el PR porque la solución no satisface criterios de calidad, diseño funcional o decisión arquitectónica. |
| **C09** | Cambios Demasiado Grandes o Difíciles de Revisar | El PR es cerrado porque el diff generado es excesivamente grande, difícil de leer o está mal estructurado para revisión humana. |
| **C10** | Desalineación entre Solución y Causa Raíz | La solución propuesta no aborda correctamente el problema real, o el agente careció de contexto clave para proponer la solución correcta. |
| **C11** | Cambios Obsoletos por Evolución Arquitectónica | El PR fue cerrado porque la arquitectura del proyecto cambió mientras estaba abierto, dejando los cambios desactualizados o irrelevantes. |

#### Familia 3 — Proceso y Cierre

| Código | Nombre | Definición breve |
|---|---|---|
| **C12** | Cierre Silencioso sin Explicación | El PR fue cerrado sin ningún comentario explicativo; no hay evidencia del motivo. |
| **C13** | Cierre Voluntario / Deliberado | El autor o el mantenedor cierra deliberadamente el PR (experimento concluido, rama eliminada, decisión propia). |
| **C14** | Pospuesto o Scope Reorientado | El PR fue cerrado con trabajo incompleto (WIP permanente) o el alcance fue reorientado para reimplementar más adelante. |
| **C15** | Reemplazado por Otro Pull Request | El PR fue cerrado porque fue sustituido por otro PR (del mantenedor, del propio agente, o abierto en repositorio correcto). |
| **C16** | Configuración Incorrecta de Entorno | El PR falla porque el entorno de CI/CD no tiene los componentes, variables o configuración necesarios para ejecutar el pipeline propuesto. |

#### Sin categoría

| Código | Nombre | Definición breve |
|---|---|---|
| **C00** | No Clasificable / Datos Insuficientes | No hay suficiente información para asignar categoría (página no disponible, idioma no comprensible, sin artefactos accesibles). |

### 6.3  Análisis de desacuerdos entre codificadores

De los 20 PRs en desacuerdo, se identificaron cuatro tipos de discrepancia:

| Tipo de desacuerdo | n | Descripción |
|---|---|---|
| **Desacuerdo Conceptual** | 9 | Los codificadores difieren en la interpretación semántica de la evidencia (ej. C12 vs C13: ¿silencio o voluntad explícita?). |
| **Dato Faltante** | 5 | Uno de los codificadores no pudo acceder al artefacto (página 404, idioma no analizable), mientras el otro sí extrajo evidencia. |
| **Desacuerdo Causa vs Mecanismo de Cierre** | 5 | Un codificador prioriza la causa técnica del fallo (ej. C02 cobertura) y el otro el mecanismo de cierre (ej. C14 scope reorientado). |
| **Desacuerdo de Granularidad CI** | 1 | Diferencia entre una categoría CI general (C01) y una específica (C04 nuevo workflow). |

Los pares de códigos más frecuentes en desacuerdo fueron:

| Gonzalo → Camilo | n | Tipo |
|---|---|---|
| C12 → C13 | 4 | Conceptual (¿silencio o decisión voluntaria?) |
| C00 → C12 / C04 / C14 | 4 | Dato faltante (Gonzalo sin acceso, Camilo con evidencia) |
| C05/C06/C07 → C12/C02/C15 | 3 | Causa vs mecanismo de cierre |

Estos desacuerdos orientan las prioridades de la sesión de consolidación antes del card sort cerrado (§6.5).

## 7. Referencias

Las siguientes referencias fundamentan la metodología cualitativa (*card sorting* y técnicas de codificación adyacentes) empleada en la PI3.

[1] F. Wen, C. Nagy, M. Lanza y G. Bavota, «An Empirical Study of Quick Remedy Commits», en *Proc. 28th Int. Conf. on Program Comprehension (ICPC '20)*, Seúl, Corea del Sur, 2020, pp. 1–12. doi: 10.1145/3387904.3389266.

[2] J. T. Liang, C. Yang y B. A. Myers, «A Large-Scale Survey on the Usability of AI Programming Assistants: Successes and Challenges», en *Proc. IEEE/ACM 46th Int. Conf. on Software Engineering (ICSE '24)*, Lisboa, Portugal, 2024. doi: 10.1145/3597503.3608128.

[3] A. Begel y T. Zimmermann, «Analyze This! 145 Questions for Data Scientists in Software Engineering», en *Proc. 36th Int. Conf. on Software Engineering (ICSE '14)*, Hyderabad, India, 2014, pp. 12–23. doi: 10.1145/2568225.2568233.

[4] T. Zimmermann, «Card-sorting: From Text to Themes», en *Perspectives on Data Science for Software Engineering*, Morgan Kaufmann, 2016, pp. 137–141.
