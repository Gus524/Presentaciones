# Módulo 3: Máquina de Estado Polimórfica de Dominio y Servicios de Dominio

## 🔄 1. Máquina de Estados Polimórfica (Domain State Pattern)

### El Anti-Patrón Habitual: Sentencias `switch` y Enums
En proyectos tradicionales, los estados se manejan con un `enum` y bloques de código repetitivos llenos de `if/switch`:

```csharp
// ❌ ANTI-PATRÓN: Lógica de estados dispersa y propensa a errores
if (prestamo.Estado == EstadoEnum.Pendiente) {
    prestamo.Estado = EstadoEnum.Activo;
} else {
    throw new Exception("Estado inválido");
}
```

### La Solución DDD: Patrón State Polimórfico en C# Records

En `templates/dotnet`, modelamos los estados como una jerarquía de clases de registros (`record`) inmutables. La clase base define el comportamiento por defecto (lanzando `DomainException`) y cada estado concreto sobrescribe **únicamente** las transiciones válidas.

#### Clase Base Abstracción: `EstadoPrestamo.cs`

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo.EstadoPrestamo;

public abstract record EstadoPrestamo
{
    public TipoEstadoPrestamo TipoEstadoPrestamo { get; }
    public virtual bool PuedeOperar => false;
    
    protected EstadoPrestamo(TipoEstadoPrestamo tipoEstadoPrestamo) => TipoEstadoPrestamo = tipoEstadoPrestamo;

    internal virtual EstadoPrestamo AprobarLayout()
        => throw new DomainException($"No se puede aprobar layout en estado {TipoEstadoPrestamo}");

    internal virtual EstadoPrestamo AprobarContabilidad()
        => throw new DomainException($"No se puede aprobar contabilidad en estado {TipoEstadoPrestamo}");

    internal virtual EstadoPrestamo Pausar()
        => throw new DomainException($"No se puede pausar un prestamo en estado {TipoEstadoPrestamo}");

    internal virtual EstadoPrestamo ReactivarPrestamo()
        => throw new DomainException($"No se puede reactivar un prestamo en estado {TipoEstadoPrestamo}");

    internal virtual EstadoPrestamo Cancelar() => new EstadoCancelado();

    internal virtual EstadoPrestamo Finiquitar()
        => throw new DomainException($"No se puede finiquitar un prestamo en estado {TipoEstadoPrestamo}");
}
```

#### Estado Concreto: `EstadoPendiente.cs`

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo.EstadoPrestamo;

public record EstadoPendiente() : EstadoPrestamo(TipoEstadoPrestamo.Pendiente)
{
    internal override EstadoPrestamo AprobarLayout() => new EstadoActivo();
}
```

#### Uso en la Entidad Principal (`Prestamo.cs`)

```csharp
public class Prestamo : AggregateRoot<PrestamoId>
{
    public EstadoPrestamo.EstadoPrestamo Estado { get; private set; } = new EstadoPendiente();

    public void AprobarLayout()
    {
        // Transición delegada a la clase de estado actual
        Estado = Estado.AprobarLayout();
    }
}
```

### Ventajas del Patrón State en DDD:
1. **Sin Enums Inseguros:** No se pueden realizar transiciones ilícitas porque la clase del estado actual simplemente no lo permite.
2. **Encapsulación:** Toda la regla de transición vive en la clase correspondiente al estado.
3. **Extensibilidad:** Agregar un nuevo estado consiste en añadir una nueva clase sin modificar condicionales gigantescos.

---

## ⚡ 2. Servicios de Dominio (Domain Services)

### ¿Cuándo usar un Domain Service?
A veces, una operación de negocio:
- Involucra múltiples Aggregates o Entidades del dominio.
- No pertenece de forma natural a ninguna entidad en particular (evita la sobrecarga de responsabilidades).
- Es un proceso o cálculo puro de negocio sin estado.

> [!NOTE]
> **Diferencia Clave:**
> - **Domain Service:** Contiene reglas de negocio puras (sin dependencias de infraestructura ni base de datos).
> - **Application Service / Handler:** Coordina la infraestructura (repositorios, transacciones, DTOs).

### Ejemplo en la Plantilla: `CalcularSaldoService.cs`

Este servicio de dominio coordina el cálculo financiero entre la entidad `Prestamo` y el objeto de valor `EstadoCuenta`:

```csharp
namespace Core.Prestamos.Domain.Services;

public class CalcularSaldoService
{
    /// <summary>
    /// Flujo 1: Coordina el cálculo pre-autorización de un Pago (Abono)
    /// </summary>
    public static DesgloseProcesado ValidarYCalcularPago(
        Prestamo prestamo, 
        EstadoCuenta estadoCuenta, 
        Dinero monto, 
        Periodo periodo)
    {
        ValidarContratoActivo(prestamo);
        
        return estadoCuenta.CalcularDesglosePago(monto, periodo, prestamo.Condiciones);
    }

    /// <summary>
    /// Flujo 2: Coordina el cálculo pre-autorización de un Periodo Faltante
    /// </summary>
    public static DesgloseProcesado ValidarYCalcularPeriodoFaltante(
        Prestamo prestamo, 
        EstadoCuenta estadoCuenta, 
        CodigoCobranza codigo, 
        Periodo periodo)
    {
        ValidarContratoActivo(prestamo);

        if (codigo.EfectoContable != EfectoContable.Indomiciliado && codigo.EfectoContable != EfectoContable.Neutral)
            throw new DomainException("Este método solo procesa movimientos Indomiciliados o Neutrales.");

        return estadoCuenta.CalcularPeriodoFaltante(periodo, codigo, prestamo.Condiciones);
    }

    private static void ValidarContratoActivo(Prestamo prestamo)
    {
        if (!prestamo.EstaActivo)
            throw new DomainException("Solo se pueden procesar movimientos en préstamos activos.");
    }
}
```

---

## 📌 Puntos Clave para la Presentación

1. **State Pattern Polimórfico**: Reemplaza estructuras `switch/case` por clases inmutables que encapsulan reglas de transición de estado.
2. **Domain Services Puros**: Métodos estáticos o servicios sin estado que operan sobre múltiples objetos de dominio manteniendo la pureza tecnológica.
