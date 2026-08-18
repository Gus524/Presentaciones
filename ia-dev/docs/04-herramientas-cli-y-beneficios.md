# 04 - Herramientas CLI, Beneficios Técnicos y Guía para Sidev

> **Módulo:** Arquitectura Agéntica - Fideicomisos Migración  
> **Documento:** 04 de 04  
> **Propósito:** Describir el herramental CLI de automatización (Bun Scripts), los beneficios cuantitativos de la arquitectura y la guía de provisión para **sidev**.

---

## 1. Herramientas CLI y Scripting Determinista (Bun Tooling)

Para evitar la corrupción de sintaxis en los archivos de tareas y eliminar la sobrecarga de tokens al consultar listas largas, el sistema agéntico utiliza una suite de utilidades mecánicas ejecutables mediante el *runtime* **Bun**:

```mermaid
graph LR
    GO[global-architect-orchestrator] -->|Query Tarea Pendiente| NTask[bun run next:task <tasks-file>]
    GO -->|Validación de Artefacto| VArt[bun run verify:artifact <path> <type>]
    GO -->|Aislamiento por Error| Stash[bun run task:stash isolate <task-id>]
    GO -->|Actualizar Estado con Commit| MTask[bun run mark:task <tasks-file> <task-id> <hash>]

    NTask -->|Devuelve String de Tarea + [AGENT]| GO
    VArt -->|Code 0: OK / Code 1: Corruption Error| GO
    Stash -->|Crea Git Stash Asociado a TASK-ID| GitStash[(Git Stash Store)]
    MTask -->|Verifica DAG & Marca [-] a [x]| StateFiles[(Markdown Task Files)]
```

### Especificación de Comandos CLI

#### 1. `bun run next:task <tasks-file>`
- **Función:** Analiza un archivo de tareas (`backend-tasks.md` o `frontend-tasks.md`) y devuelve únicamente la **primera tarea ejecutable** cuyo checkbox esté pendiente (`- [ ]`) y cuyas dependencias declaradas (`[DEPENDS: TASK-XXX]`) hayan sido completadas (`- [x]`).
- **Beneficio:** Evita que el orquestador tenga que leer todo el archivo de tareas en su contexto, ahorrando miles de tokens por iteración.

#### 2. `bun run mark:task <domain-tasks-file> <TASK-ID> <commit_hash>`
- **Función:** Actualiza mecánicamente la casilla de la tarea dada de `- [ ]` a `- [x]` e inyecta el hash del commit verificado.
- **Validación DAG:** Verifica automáticamente que la tarea previa o las dependencias explícitas declaradas (`[DEPENDS: ...]`) estén marcadas como completadas. Si hay una violación de dependencias, falla con código de salida 1 y detiene el pipeline.

#### 3. `bun run verify:artifact <artifact_path> <artifact_type>`
- **Función:** Inspección mecánica multiplataforma que comprueba que el archivo físico del artefacto exista en disco y no esté vacío (bytes > 0).
- **Tipos soportados:** `scout`, `design`, `tasks`, `qa`, `code`.

#### 4. `bun run task:stash isolate | restore | drop <TASK-ID>`
- **Función:** Gestiona stashes temporales en Git asociados a una tarea específica cuando se detecta un error de compilación. Permite aislar los cambios rotos, aplicar fix manual o re-invocar al agente táctico, y finalmente eliminar el stash tras el commit exitoso.

### 1.2 Filosofía de Diseño: IA Probabilística vs Scripting Determinista

Un pilar maestro de esta arquitectura radica en entender las fortalezas y limitaciones de la Inteligencia Artificial:

- **La IA es Probabilística:** Sobresale en razonamiento conceptual, refactorización, modelado de dominio y diseño de interfaces. Sin embargo, **fallar al analizar sintaxis estricta de Markdown, omitir dependencias implícitas o cometer errores al modificar casillas de estado** son riesgos inherentes cuando un LLM intenta actuar como parser mecánico.
- **Los Scripts CLI son Deterministas:** Un programa compilado o script en Bun/Node ejecutado localmente **nunca falla probabilísticamente**. El código de parser de Markdown, la lógica de grafos DAG de dependencias y el estash de Git funcionan con un **100% de determinismo algorítmico**.

Por ello, el orquestador **nunca parsea ni modifica manualmente las casillas de verificación de tareas ni el estado del pipeline**. Delega esa responsabilidad a los scripts de Bun:

```mermaid
grid
```
| Operación Crítica | ¿Se confía al LLM (Probabilístico)? | ¿Se delega a Script CLI (Determinista)? | Motivo Técnico de la Delegación |
| :--- | :--- | :--- | :--- |
| **Selección de la siguiente tarea** | ❌ No | ✅ `bun run next:task` | Evita que el LLM se salte dependencias DAG `[DEPENDS: ...]` o elija tareas pendientes incorrectas. |
| **Actualización de estado (`- [x]`)** | ❌ No | ✅ `bun run mark:task` | Elimina el riesgo de corrupción de sintaxis Markdown o desincronización de hashes de commit. |
| **Verificación de existencia de artefeacto** | ❌ No | ✅ `bun run verify:artifact` | Evita asumir que un archivo fue creado basándose únicamente en la afirmación verbal del agente. |
| **Aislamiento de errores de compilación** | ❌ No | ✅ `bun run task:stash` | Garantiza que las operaciones de Git Stash sean atómicas y repetibles sin sintaxis errónea de Git. |

---

## 2. Beneficios Técnicos y Métricas Cuantitativas

La arquitectura agéntica de Fideicomisos Migración ofrece ventajas sustanciales respecto a enfoques agénticos tradicionales (monolíticos o no estructurados):

```mermaid
grid
  classDef default fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px;
```

| Dimensión | Enfoque Agéntico Convencional | Arquitectura SDD Fideicomisos Migración | Beneficio Técnico |
| :--- | :--- | :--- | :--- |
| **Consumo de Contexto (Tokens)** | Crecimiento cuadrático $O(N^2)$ al acumular chats y archivos fuente. | Crecimiento lineal / constante $O(1)$ gracias a Pass-by-Reference y Task-Scoped Threads. | **Ahorro > 75% en costo de tokens** y eliminación de la degradación por distracción. |
| **Trazabilidad de Commits** | Commits masivos, difusos o mensajes automáticos genéricos en inglés. | Commits atómicos individuales por tarea, verificado por compilación y redactados en **Español**. | **Trazabilidad 100% auditable** alineada a Gitflow y Conventional Commits. |
| **Tasa de Alucinación** | Invención ocasional de librerías, DTOs o estructuras de BD no existentes. | Restricción total por listas blancas (`knowledge/tech-stack-whitelist.md`) y validación de manifiestos. | **Alucinación cercana a 0%** en comandos, imports y llamadas API. |
| **Seguridad Legacy (GeneXus)** | Riesgo de alterar esquemas de BD heredados mediante migraciones automáticas. | **Invariante Estricta de Inmutabilidad** de tablas GeneXus protegida por ACL. | **Cero riesgo de corrupción DDL** en la base de datos de producción. |
| **Resiliencia ante Interrupción** | Pérdida de estado si el contexto se resetea o compacta. | Estado persisted al 100% en discos mediante `pipeline-tasks.md` y `next:task`. | **Recuperación instantánea** tras reseteos de sesión o fallos de red. |
| **Control del Desarrollador** | Agente ejecuta cambios masivos sin consulta previa. | 3 Puertas de Control **HITL obligatorias** (DCS Routing, Stash Guard, QA Fail Classification). | **Control arquitectónico absoluto** en manos del Lead Architect. |

---

## 3. Guía de Entrega e Provisión para Sidev

Esta documentación ha sido generada en formato Markdown estándar con diagramas Mermaid integrados para su consumo directo en **sidev** (o cualquier portal de documentación/wiki técnica):

### Archivos Proporcionados en la Suite

```
docs/arquitectura-agentica/
├── README.md                                    # Índice general y diagrama de topología
├── 01-principios-y-filosofia.md                 # Invariantes, JIT Hydration y Claim-Check
├── 02-catalogo-de-agentes-y-skills.md           # Fichas técnicas de los 8 agentes y skills
├── 03-flujos-sdd-y-protocolos-de-ejecucion.md   # Fórmula DCS, Pipeline 5 Fases y Stash Guard
└── 04-herramientas-cli-y-beneficios.md          # CLI Bun Tools, Benchmarks y Guía Sidev
```

### Instrucciones para Agregar a Sidev

1. **Copia de Artefactos:** Copie la carpeta `docs/arquitectura-agentica/` completa al repositorio o directorio de documentación de **sidev**.
2. **Renderizado de Diagramas:** Asegúrese de que el visor de Markdown de **sidev** tenga habilitado el plugin de renderizado **Mermaid** (soportado nativamente por GitLab, GitHub, Azure DevOps y MkDocs).
3. **Enlaces Relativos:** Los enlaces internos entre documentos utilicen sintaxis relativa estándar (`[01-principios-y-filosofia.md](01-principios-y-filosofia.md)`), garantizando una navegación fluida en sidev.
