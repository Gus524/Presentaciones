# Módulo 1: Strategic DDD - Screaming Architecture, Monolito Modular y Bounded Contexts

> **Perspectiva del Strategic DDD Architect:** *Un Bounded Context define el límite explícito dentro del cual un modelo de dominio se aplica de forma coherente. Mezclar vocabulario o permitir acoplamiento directo entre contextos destruye la arquitectura.*

---

## 🏛️ 1. Concepto: Arquitectura Gritante (Screaming Architecture)

La **Arquitectura Gritante** establece que al examinar la estructura de un repositorio, la organización debe "gritar" el **dominio de negocio** (los Bounded Contexts) antes que los marcos tecnológicos o capas de infraestructura.

En `templates/dotnet`, aplicamos este principio organizando la solución como un **Monolito Modular** estructurado en Bounded Contexts aislados dentro del proyecto `Core`.

```text
Core/
├── IAM/                         # Bounded Context: Identity & Access Management
│   ├── Application/
│   ├── Domain/
│   └── README.md
├── Prestamos/                   # Bounded Context: Gestión de Préstamos
│   ├── Application/
│   ├── Domain/
│   └── README.md
├── Productos/                   # Bounded Context: Catálogo de Productos
│   ├── Application/
│   ├── Domain/
│   └── README.md
└── SharedKernel/                # Núcleo Compartido (Vocabulario Común)
    ├── Abstractions/
    ├── Behaviors/
    ├── Enums/
    ├── Exceptions/
    ├── Interfaces/
    ├── Ports/
    ├── ValueObjects/
    └── Wrappers/
```

---

## 🗺️ 2. Relaciones entre Bounded Contexts (Strategic Context Mapping)

Para evitar fugas de modelo y acoplamientos rígidos entre los Bounded Contexts (`IAM`, `Prestamos`, `Productos`), el **Strategic DDD Architect** impone las siguientes reglas de integración:

### A. Comunicación Asíncrona mediante Eventos de Dominio
Los Bounded Contexts del Core **NUNCA** mantienen dependencias directas síncronas o referencias a bases de datos entre sí. La comunicación entre contextos se realiza mediante eventos asíncronos o contratos de interfaz expuestos.

```mermaid
graph LR
    subgraph IAM ["Bounded Context: IAM (Upstream)"]
        UserDomain[Usuario / Identity]
    end
    
    subgraph Prestamos ["Bounded Context: Préstamos (Downstream)"]
        ACL[Anti-Corruption Layer - ACL]
        LoanDomain[Préstamo Aggregate]
    end
    
    UserDomain -->|UsuarioCreadoEvent (Async)| ACL
    ACL -->|Mapea a BeneficiarioId| LoanDomain
```

### B. Anti-Corruption Layer (ACL)
Cuando el Bounded Context de `Prestamos` necesita información de `IAM` o `Productos`, **no utiliza directamente las entidades del otro contexto**. Utiliza un **Puerto de Entrada/Salida (ACL)** que traduce los modelos externos al Lenguaje Ubicuos local del contexto consumible.

---

## 🌐 3. El Núcleo Compartido (`SharedKernel`)

El `SharedKernel` representa el **vocabulario técnico y de dominio universal** que todos los Bounded Contexts comparten. 

> [!CAUTION]
> **REGLA DE ORO DEL SHARED KERNEL:**
> El `SharedKernel` **NUNCA** debe depender de librerías de infraestructura (como EF Core, Dapper, ASP.NET Core) ni de ningún Bounded Context específico. Es código C# puro (.NET Standard / BCL).

### Contratos Principales del SharedKernel

#### 1. Wrapper Unificado de Respuesta (`Response<T>`)
Evita excepciones en el flujo de control predecible y estandariza los retornos del sistema:

```csharp
namespace Core.SharedKernel.Wrappers;

public class Response<T>
{
    public bool Succeeded { get; set; }
    public string? Message { get; set; }
    public T? Data { get; set; }
    public Dictionary<string, string[]>? Errors { get; set; }
    
    [JsonIgnore]
    public SuccessType SuccessType { get; set; }
    
    [JsonIgnore]
    public ErrorType ErrorType { get; set; }

    public static Response<T> Success(T data, string? message = null, SuccessType type = SuccessType.Ok) =>
        new() { Succeeded = true, Data = data, Message = message, SuccessType = type };

    public static Response<T> Fail(string message, ErrorType type = ErrorType.BadRequest) =>
        new() { Succeeded = false, Message = message, ErrorType = type };
}
```

---

## 📌 Puntos Clave para la Presentación

1. **Screaming Architecture**: La estructura del código refleja el negocio, no los frameworks.
2. **Context Boundaries & ACL**: Los Bounded Contexts están desacoplados; la integración entre contextos usa eventos asíncronos y capas anticorrupción (ACL).
3. **SharedKernel Puro**: No se contamina con ORMs ni lógica de infraestructura.
