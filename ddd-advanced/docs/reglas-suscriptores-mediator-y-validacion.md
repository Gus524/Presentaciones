# Guía Técnica: Eventos de Dominio, Suscriptores Decoupled, Mediador Personalizado y Validación en .NET

> **Nota de Arquitectura:** En Domain-Driven Design (DDD) avanzado, el acoplamiento transaccional entre Agregados es uno de los errores más severos. Este documento establece las reglas estrictas de nomenclatura, la desacoplación total de los Suscriptores de Eventos, el manejo de validaciones sin excepciones en el Pipeline CQRS y la implementación **completa y funcional de un Mediador Personalizado sin librerías externas (sin MediatR)**.

---

## 🚫 1. Regla de Nomenclatura: Eliminación del Sufijo Redundante `Event`

### El Anti-Patrón
Nombrar clases como `PagoAutorizadoEvent` o `PrestamoCreadoEvent` añade ruido técnico redundante. La interfaz que implementa la clase (`IDomainEvent`) ya declara su naturaleza técnica.

### La Regla DDD
Los Eventos de Dominio representan **hechos inmutables que sucedieron en el pasado del negocio**. Deben nombrarse exclusivamente con sustantivos y verbos en pasado:

```csharp
// ❌ INCORRECTO: Redundancia técnica
public record PagoAutorizadoEvent(...) : DomainEvent;

// ✅ CORRECTO: Expresividad del Lenguaje Ubicuo en tiempo pasado
public record PagoAutorizado(
    PrestamoId PrestamoId,
    MovimientoId MovimientoId,
    Dinero MontoDepositado,
    CodigoCobranza CodigoCobranza,
    Periodo Periodo,
    DesgloseProcesado ResultadoCalculo
) : DomainEvent;
```

---

## 🛡️ 2. Regla Estricta para Event Subscribers (Suscriptores de Eventos)

### La Regla de Oro
> **UN SUSCRIPTOR DE EVENTOS NUNCA INYECTA REPOSISTORIOS (`IRepository`), NI `DbContext`, NI `IUnitOfWork`.**

### ¿Por qué?
Si un Suscriptor de Eventos (`PagoAutorizadoSubscriber`) inyecta `IEstadoCuentaRepository` y modifica el Agregado `EstadoCuenta` dentro del mismo ciclo transaccional del `Prestamo`, se violan dos principios fundamentales:
1. **Límite Transaccional del Agregado**: Solo se debe modificar **1 Agregado por Transacción**.
2. **Consistencia Eventual**: Los Agregados secundarios deben actualizarse de manera asíncrona/eventual a través de intenciones aisladas (Comandos).

### El Patrón Correcto: Evento ➔ Command Mapeo ➔ Mediador
El Suscriptor tiene **UNA SÓLA RESPONSABILIDAD**: recibir el evento de dominio, mapear sus datos hacia un **Comando de Aplicación** y despacharlo mediante el Mediador.

```csharp
namespace Core.Prestamos.Application.Features.EstadoCuenta.EventSubscribers;

using Core.Prestamos.Application.Features.EstadoCuenta.Commands.AplicarMovimiento;
using Core.Prestamos.Domain.Events;
using Core.SharedKernel.Ports.In;

public class PagoAutorizadoSubscriber(
    IMediator mediator // ✅ ÚNICA DEPENDENCIA PERMITIDA
) : IDomainEventSubscriber<PagoAutorizado>
{
    public async Task Handle(PagoAutorizado domainEvent, CancellationToken cancellationToken)
    {
        // 1. Mapeo transparente desde el Evento de Dominio hacia un Comando de Aplicación
        var command = new AplicarMovimientoCommand(
            PrestamoId: domainEvent.PrestamoId.Value,
            MovimientoId: domainEvent.MovimientoId.Value,
            Monto: domainEvent.MontoDepositado.Monto,
            Moneda: domainEvent.MontoDepositado.Moneda,
            Concepto: domainEvent.CodigoCobranza.Descripcion,
            Periodo: domainEvent.Periodo.Valor
        );

        // 2. Despachar el comando. El CommandHandler correspondiente abrirá su propia transacción aislada.
        await mediator.Send(command, cancellationToken);
    }
}
```

---

## ⚡ 3. Pipeline CQRS: Validación sin Excepciones (`ValidationBehavior`)

Lanzar excepciones (`throw new BadHttpRequestException()`) en validaciones de entrada degrada el rendimiento del runtime y rompe el *Result Pattern*. `ValidationBehavior` debe interceptar las fallas y retornar `Response<TResponse>` tipado.

### 📄 [`Core/SharedKernel/Application/Behaviors/ValidationBehavior.cs`](file:///var/home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/Core/SharedKernel/Application/Behaviors/ValidationBehavior.cs)

```csharp
using Core.SharedKernel.Application.Mediator;
using Core.SharedKernel.Application.Wrappers;
using FluentValidation;

namespace Core.SharedKernel.Application.Behaviors;

public class ValidationBehavior<TRequest, TResponse>(
    IEnumerable<IValidator<TRequest>> validators
) : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    public async Task<Response<TResponse>> Handle(
        TRequest request,
        RequestHandlerTransfer<TResponse> next,
        CancellationToken cancellationToken)
    {
        if (!validators.Any()) return await next();

        var context = new ValidationContext<TRequest>(request);
        var failures = validators
            .Select(v => v.Validate(context))
            .SelectMany(result => result.Errors)
            .Where(f => f != null)
            .ToList();

        if (failures.Count != 0)
        {
            var errorsMap = failures
                .GroupBy(e => e.PropertyName)
                .ToDictionary(g => g.Key, g => g.Select(e => e.ErrorMessage).ToArray());

            // Devuelve Response con estado BadState/Unprocessable sin lanzar excepciones
            return Response.UnprocessableEntity<TResponse>(
                "La solicitud contiene errores de validación.", 
                errorsMap
            );
        }

        return await next();
    }
}
```

---

## 🔌 4. Mediador Personalizado Ligero (.NET 10+ Sin MediatR)

A continuación se presenta el **código fuente completo e integral** para implementar un Mediador de alto rendimiento integrado en el `SharedKernel` y la infraestructura del proyecto.

### 4.1 Contratos del Dominio y Aplicación (`Core/SharedKernel`)

#### 📄 `Core/SharedKernel/Application/Mediator/IRequest.cs`
```csharp
using Core.SharedKernel.Application.Wrappers;

namespace Core.SharedKernel.Application.Mediator;

public interface IRequest<TResponse>;

public delegate Task<Response<TResponse>> RequestHandlerTransfer<TResponse>();

public interface IRequestHandler<in TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    Task<Response<TResponse>> Handle(TRequest request, CancellationToken cancellationToken);
}

public interface ICommandHandler<in TCommand, TResponse> : IRequestHandler<TCommand, TResponse>
    where TCommand : IRequest<TResponse>;

public interface IQueryHandler<in TQuery, TResponse> : IRequestHandler<TQuery, TResponse>
    where TQuery : IRequest<TResponse>;

public interface IDomainEventSubscriber<in TDomainEvent>
{
    Task Handle(TDomainEvent domainEvent, CancellationToken cancellationToken);
}
```

#### 📄 `Core/SharedKernel/Ports/In/IMediator.cs`
```csharp
using Core.SharedKernel.Application.Mediator;
using Core.SharedKernel.Application.Wrappers;

namespace Core.SharedKernel.Ports.In;

public interface IMediator
{
    Task<Response<TResponse>> Send<TResponse>(
        IRequest<TResponse> request,
        CancellationToken cancellationToken = default);

    Task Publish<TDomainEvent>(
        TDomainEvent domainEvent,
        CancellationToken cancellationToken = default);
}
```

---

### 4.2 Implementación de Infraestructura (`WebApi/Mediator` / `SharedKernel`)

#### 📄 `WebApi/Mediator/RequestHandlerBase.cs`
```csharp
using Core.SharedKernel.Application.Wrappers;

namespace WebApi.Mediator;

internal abstract class RequestHandlerBase<TResponse>
{
    public abstract Task<Response<TResponse>> Handle(
        object request, 
        IServiceProvider serviceProvider, 
        CancellationToken cancellationToken);
}
```

#### 📄 `WebApi/Mediator/RequestHandlerWrapper.cs`
```csharp
using Core.SharedKernel.Application.Mediator;
using Core.SharedKernel.Application.Wrappers;
using Microsoft.Extensions.DependencyInjection;

namespace WebApi.Mediator;

internal class RequestHandlerWrapper<TRequest, TResponse> : RequestHandlerBase<TResponse>
    where TRequest : IRequest<TResponse>
{
    public override async Task<Response<TResponse>> Handle(
        object request, 
        IServiceProvider serviceProvider, 
        CancellationToken cancellationToken)
    {
        var typedRequest = (TRequest)request;
        var handler = serviceProvider.GetRequiredService<IRequestHandler<TRequest, TResponse>>();

        // Resolver Pipeline Behaviors (Validation, Logging, Transactions) en orden inverso
        var behaviors = serviceProvider.GetServices<IPipelineBehavior<TRequest, TResponse>>()
            .Reverse()
            .ToList();

        RequestHandlerTransfer<TResponse> next = () => handler.Handle(typedRequest, cancellationToken);

        foreach (var behavior in behaviors)
        {
            var currentNext = next;
            next = () => behavior.Handle(typedRequest, currentNext, cancellationToken);
        }

        return await next();
    }
}
```

#### 📄 `WebApi/Mediator/InternalMediator.cs`
```csharp
using System.Collections.Concurrent;
using Core.SharedKernel.Application.Mediator;
using Core.SharedKernel.Application.Wrappers;
using Core.SharedKernel.Ports.In;
using Microsoft.Extensions.DependencyInjection;

namespace WebApi.Mediator;

public class InternalMediator(IServiceProvider serviceProvider) : IMediator
{
    private static readonly ConcurrentDictionary<Type, object> RequestWrapperCache = new();

    public Task<Response<TResponse>> Send<TResponse>(
        IRequest<TResponse> request, 
        CancellationToken cancellationToken = default)
    {
        var requestType = request.GetType();

        var wrapper = (RequestHandlerBase<TResponse>)RequestWrapperCache.GetOrAdd(requestType, type =>
        {
            var wrapperType = typeof(RequestHandlerWrapper<,>).MakeGenericType(type, typeof(TResponse));
            return Activator.CreateInstance(wrapperType) 
                   ?? throw new InvalidOperationException($"No se pudo crear el wrapper para {type}");
        });

        return wrapper.Handle(request, serviceProvider, cancellationToken);
    }

    public async Task Publish<TDomainEvent>(
        TDomainEvent domainEvent, 
        CancellationToken cancellationToken = default)
    {
        if (domainEvent is null) return;

        // Resuelve todos los suscriptores registrados para TDomainEvent en el contenedor IoC
        var subscribers = serviceProvider.GetServices<IDomainEventSubscriber<TDomainEvent>>();

        foreach (var subscriber in subscribers)
        {
            await subscriber.Handle(domainEvent, cancellationToken);
        }
    }
}
```

---

## 📌 Resumen de Reglas para el Equipo de Desarrollo

1. **EVENTOS SIN SUFIJO 'EVENT'**: Usar `PagoAutorizado`, `PrestamoCreado`. NUNCA `PagoAutorizadoEvent`.
2. **SUSCRIPTORES MAGROS (SLIM SUBSCRIBERS)**: `IDomainEventSubscriber<T>` solo inyecta `IMediator` y despacha un `Command`. Cero acceso a BD o Repositorios dentro del Suscriptor.
3. **RESULT PATTERN EN PIPELINE**: Validaciones devuelven `Response.UnprocessableEntity(...)` en lugar de `throw`.
4. **MEDIADOR PROPRIETARIO**: Despacho en memoria ultrarrápido utilizando `InternalMediator`, libre de dependencias de terceros.
