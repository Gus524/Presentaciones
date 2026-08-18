# Documentación Técnica: Arquitectura Agéntica y Pipeline SDD

> **Proyecto:** Fideicomisos Migración (.NET 10 + Nx (Angular 22) + GeneXus Legacy ACL)  
> **Patrón Arquitectónico:** Spec-Driven Development (SDD), Deterministic JIT Routing, Hexagonal/Clean Architecture, Human-in-the-Loop (HITL) Mandate.

---

## Índice de Contenidos

Esta suite de documentación especifica de manera formal y rigurosa la arquitectura agéntica implementada en el monorepo. Diseñada para su entrega en **sidev**, libre de alucinaciones y basada 100% en los artefactos físicos y protocolos del proyecto.

1. [`01-principios-y-filosofia.md`](file:///home/gus/Documents/Repositorios/Gitlab/fideicomisosmigracion/docs/arquitectura-agentica/01-principios-y-filosofia.md)
   - Mandato de Cero Alucinación y Determinismo.
   - Carga Perezosa Multi-Nivel (JIT Hydration).
   - Protocolo "Trade-Off First" y Evaluación Arquitectónica.
   - Invariante de Inmutabilidad de Esquemas Legacy (GeneXus).
   - Gestión de Contexto Pass-by-Reference (Claim Check Protocol).
   - Estado Único de la Verdad mediante Markdown Checkboxes.

2. [`02-catalogo-de-agentes-y-skills.md`](file:///home/gus/Documents/Repositorios/Gitlab/fideicomisosmigracion/docs/arquitectura-agentica/02-catalogo-de-agentes-y-skills.md)
   - Catálogo detallado de Agentes Globales, Backend y Frontend.
   - Roles, herramientas permitidas, límites infranqueables y payload de retorno.
   - Matriz de Skills (Globales, Backend DDD/Hexagonal, Frontend Signals/Nx).
   - Mapeo de Conocimiento (*Knowledge*) y Memoria Persistente (*Lessons Learned*).

3. [`03-flujos-sdd-y-protocolos-de-ejecucion.md`](file:///home/gus/Documents/Repositorios/Gitlab/fideicomisosmigracion/docs/arquitectura-agentica/03-flujos-sdd-y-protocolos-de-ejecucion.md)
   - Fórmula del *Deterministic Complexity Score* (DCS) y enrutamiento a Branch A / Branch B.
   - Flujo detallado del Pipeline SDD de 5 Fases.
   - Ciclo de Commit Atómico y Protocolo `git-commit-expert`.
   - Manejo de Fallas de Compilación (*Stash Guard* y HITL no bloqueante).
   - Diagramas de Arquitectura y Secuencia en **MERMAID**.

4. [`04-herramientas-cli-y-beneficios.md`](file:///home/gus/Documents/Repositorios/Gitlab/fideicomisosmigracion/docs/arquitectura-agentica/04-herramientas-cli-y-beneficios.md)
   - Herramientas CLI automatizadas (`bun run next:task`, `mark:task`, `verify:artifact`, `task:stash`).
   - Beneficios clave: Eficiencia de Tokens $O(1)$, Trazabilidad Atómica en Español, Recuperación Inmediata ante CompACT / Interrupciones.
   - Guía de Integración para **sidev**.

---

## Mapa General de la Arquitectura Agéntica

```mermaid
graph TD
    User([Desarrollador / Lead Architect]) <-->|HITL Gates / Trade-off Validation| GO[global-architect-orchestrator]
    
    subgraph Discovery & Routing [Fase 1 & 2: Telemetría y Complejidad]
        GO -->|Invocación Read-Only| SSA[system-state-analyzer]
        SSA -->|Micro Payload: metrics + phase-1-scout.md| GO
    end
    
    subgraph Design Layer [Fase 3: Especificación Técnica]
        GO -->|Diseño Backend| SDA[strategic-ddd-architect]
        GO -->|Diseño Frontend| FUA[frontend-ui-architect]
        SDA -->|Genera backend-design.md & backend-tasks.md| SpecStore[(/specs/<change-id>/)]
        FUA -->|Genera frontend-design.md & frontend-tasks.md| SpecStore
    end
    
    subgraph Tactical Execution [Fase 4: Bucle de Implementación Atómica]
        GO -->|Extracción Metadata Task| CLI_NEXT[bun run next:task]
        GO -->|Dispatch Táctico| TDM[tactical-ddd-modeler]
        GO -->|Dispatch Táctico| ACL[acl-legacy-integration-expert]
        GO -->|Dispatch Táctico| FUA
        
        TDM -->|Payload Claim-Check| GO
        ACL -->|Payload Claim-Check| GO
        
        GO -->|Invocación Sincrónica con Manifest| GCE[git-commit-expert]
        GCE -->|Validar Build & Conventional Commit| GitLog[(Git Repository)]
        GCE -->|Registro Hash| CLI_MARK[bun run mark:task]
    end

    subgraph Quality Assurance [Fase 5: Verificación de Integridad]
        GO -->|Invocación QA| DQTE[domain-quality-testing-expert]
        DQTE -->|Reporte QA: phase-5-qa-report.md| GO
    end
```
