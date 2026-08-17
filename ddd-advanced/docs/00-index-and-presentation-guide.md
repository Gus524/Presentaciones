# Guía de Presentación e Índice: DDD Avanzado en C# .NET 10

Este conjunto de documentos `.md` ha sido revisado y alineado con los roles y reglas de los agentes **`strategic-ddd-architect`**, **`tactical-ddd-modeler`** y el **`architectural-linter`**. 

Está diseñado específicamente para ser consumido por herramientas de IA CLI (como Sidedev, Slidev, Marp o CLI agents) con el fin de generar una presentación técnica rigurosa sobre **Domain-Driven Design (DDD) Avanzado y Arquitectura Hexagonal**.

Toda la información, patrones, reglas de diseño y ejemplos de código provienen directamente de la plantilla de producción ubicada en [`templates/dotnet`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet).

---

## 🎯 Objetivo de la Presentación

Enseñar a desarrolladores senior y arquitectos de software cómo diseñar sistemas empresariales robustos en .NET 10 combinando:
1. **DDD Estratégico** (Bounded Contexts, Bounded Context Maps, Anti-Corruption Layer - ACL, integración asíncrona por eventos de dominio).
2. **DDD Táctico** (Aggregate Roots, Design by Contract, Strongly Typed IDs con reglas de asignación de memoria `struct` vs `class/record`, Value Objects ricos con algoritmos como Fowler Allocation, Máquina de Estados Polimórfica).
3. **Event-Driven Architecture (EDA)** con **Outbox Pattern** e interceptores de EF Core (`OutboxDomainEventsInterceptor`).
4. **Arquitectura Hexagonal (Puertos y Adaptadores)** y **CQRS Asimétrico** con `Response<T>` pattern.

---

## 📚 Índice de Módulos `.md`

| Archivo | Agente / Rol Principal | Conceptos Clave |
| :--- | :--- | :--- |
| [`01-screaming-architecture-and-bounded-contexts.md`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/docs/advanced-ddd/01-screaming-architecture-and-bounded-contexts.md) | **Strategic DDD Architect** | Screaming Architecture en `Core/`, Bounded Context Boundaries (`IAM`, `Prestamos`, `Productos`), Anti-Corruption Layer (ACL) y SharedKernel sin dependencias. |
| [`02-tactical-ddd-aggregates-entities-vo.md`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/docs/advanced-ddd/02-tactical-ddd-aggregates-entities-vo.md) | **Tactical DDD Modeler** | Design by Contract (`private` constructors + static factories), Strongly Typed IDs (`readonly record struct` vs `record` por tamaño), Fowler Allocation Algorithm en `Dinero.cs`. |
| [`03-domain-state-machine-and-services.md`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/docs/advanced-ddd/03-domain-state-machine-and-services.md) | **Tactical DDD Modeler** | Domain State Pattern con C# Records (`EstadoPrestamo`), eliminación de `switch/if` por transiciones polimórficas y Domain Services puros (`CalcularSaldoService`). |
| [`04-domain-events-and-outbox-pattern.md`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/docs/advanced-ddd/04-domain-events-and-outbox-pattern.md) | **Tactical DDD Modeler** | Estándar `DomainEvent` (hechos pasados), solución al Dual-Write Problem con `OutboxDomainEventsInterceptor` en EF Core y procesamiento asíncrono. |
| [`05-hexagonal-architecture-and-cqrs-pipeline.md`](file:///home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/docs/advanced-ddd/05-hexagonal-architecture-and-cqrs-pipeline.md) | **Architectural Linter** | Aislamiento de Puertos (Interfaces en Dominio, Adaptadores en Infraestructura), `Response<T>` pattern en lugar de excepciones, y CQRS asimétrico. |

---

## 💡 Prompt Recomendado para la IA CLI (Sidedev / Slidev Generator)

```text
Actúa como un Principal Software Architect y experto en DDD. Genera una presentación en diapositivas (formato Sidedev / Marp / Slidev) utilizando la información detallada contenida en los archivos en templates/dotnet/docs/advanced-ddd/*.md.

Requisitos de la presentación:
1. Divide el contenido en 6 secciones principales correspondientes a los archivos .md.
2. Incluye fragmentos de código C# reales extraídos de la documentación (Fowler Allocation en Dinero.cs, EstadoPrestamo State Pattern, OutboxInterceptor de EF Core, Response<T> Pattern).
3. Enfatice la regla de rendimiento en Value Objects (struct para IDs de 1 campo, record de referencia para compuestos).
4. Utiliza diagramas en sintaxis Mermaid para ilustrar las capas de la Arquitectura Hexagonal, el Outbox Pattern y la máquina de estados.
5. Enfócate en el POR QUÉ técnico (trade-offs, prevención de bugs, mantenibilidad a largo plazo).
```
