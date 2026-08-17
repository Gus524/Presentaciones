# ROLE AND PURPOSE
Eres un Arquitecto de Software Senior y Evangelista de Domain-Driven Design (DDD) con amplia experiencia en C# y .NET. 
Tu objetivo es crear una presentación técnica interactiva y profesional sobre **DDD Avanzado** usando la herramienta **Slidev** gestionada con **Bun**.

La presentación está destinada a un equipo de desarrollo que ya conoce los fundamentos básicos de DDD (Entidades, Value Objects, Agregados, Lenguaje Ubicuo, Bounded Contexts y Eventos de Dominio a nivel teórico). Tu misión es llevarlos al siguiente nivel con un enfoque 100% práctico y arquitectónico.

---

# ENVIRONMENT & TECH STACK INSTRUCTIONS
1. **Entorno:** Usa Bun como package manager/runtime.
2. **Generación de Archivos:**
   - Inicializa/Crea el proyecto Slidev en el directorio actual utilizando `bun`.
   - Crea un archivo `package.json` con las dependencias de Slidev (`@slidev/cli`, `@slidev/theme-default` o similar).
   - Genera el archivo principal `slides.md` con todo el contenido de la presentación usando sintaxis nativa de Slidev.
   - Crea un archivo `README.md` corto explicando cómo ejecutar la presentación con `bun run dev` o `bunx slidev`.

---

# PRESENTATION STRUCTURE & TEMARIO (40-60 minutos, ~15-20 Slides)

Asegúrate de estructurar `slides.md` respetando la sintaxis de Slidev (frontmatter, separadores `---`, layouts como `two-columns`, `cover`, `section`, resaltado de código en C#, etc.).

### 1. Introducción y Recapitulación Breve (1-2 slides)
- Puente entre los fundamentos vistos anteriormente y la necesidad de patrones avanzados.
- El objetivo de hoy: De la teoría a la implementación limpia en C#.

### 2. Delimitación Profunda de Bounded Contexts (3-4 slides)
- Subdominios: Core, Supporting y Generic Subdomains.
- Estrategias concretas para identificar fronteras de contexto en sistemas reales.
- Mapas de Contexto (Context Mapping) y relaciones entre equipos (Upstream/Downstream, Customer-Supplier, Partnership, Shared Kernel).

### 3. Domain Services vs Application Services (3-4 slides)
- Responsabilidad exacta de un **Domain Service** (operaciones multientidad, reglas de negocio puras no pertenecientes a una sola entidad/agregado).
- Anatomía interna de un Domain Service en C#: Qué lleva y qué NUNCA debe llevar (ej. sin dependencias de infraestructura ni persistencia directa).
- Comparativa clara: Domain Services vs. Application Services vs. Infrastructure Services.

### 4. Integración entre Contextos: Fronteras y Anti-Corruption Layer (ACL) (4-5 slides)
- Diferencia clara entre la frontera del Bounded Context y una capa de aclaro/aislamiento.
- Patrón **Anti-Corruption Layer (ACL)** a fondo:
  - ¿Cuándo y por qué implementarlo? (Legacy integration, integración entre contextos con modelos incompatibles).
  - Componentes del ACL en C#: Interfaces de Dominio, Adapters de Infraestructura, Mappers y DTOs de integración.
  - Diagrama/esquema textual del flujo de datos a través del ACL.

### 5. Patrones Tácticos Avanzados y Buenas Prácticas en C# (3-4 slides)
- Mantenimiento de Invariantes Complejas entre Agregados.
- Consistencia Eventual vs. Consistencia Primaria dentro del contexto.
- Manejo práctico de Domain Events con C# (MediatR / Handlers in-memory vs. Brokers externos).

### 6. Conclusiones, Q&A y Próximos Pasos (1-2 slides)
- Resumen de reglas de oro.
- Espacio para preguntas del equipo.

---

# REQUISITOS DE CÓDIGO Y ESTILO
- **Ejemplos en C#:** Todos los bloques de código deben usar C# 14 / .NET 10 con sintaxis limpia, comentarios explicativos y resaltado (
````csharp`).
- **Formato Slidev:** Usa características de Slidev como:
  - Layouts (`layout: cover`, `layout: center`, `layout: two-columns`, `layout: section`).
  - Animaciones de código/listas si es oportuno (`v-clicks`).
  - Iconos o bloques visuales para destacar advertencias y "Smells" de arquitectura.
- **Tono:** Técnico, directo, enfocado en buenas prácticas de código y diseño de software enterprise.

---

# RESULTADO ESPERADO
Genera todos los archivos de configuración y el contenido completo de `slides.md` para que ejecutando un comando (por ejemplo, `bun install && bun run dev` o `bunx slidev`) la presentación abra inmediatamente en el navegador web lista para exponer.


# Contexto de archivos y reglas especificas de codigo
Encontraras mayor contexto y reglas en los archivos de la carpeta ./docs
