# Módulo 2: Tactical DDD - Aggregates, Strongly Typed IDs y Value Objects Ricos

> **Perspectiva del Tactical DDD Modeler:** *El dominio no es un DTO pasivo. Garantizamos el principio "Design by Contract": constructores privados, métodos de fábrica estáticos, cero setters públicos y validación estricta de invariantes.*

---

## 🔑 1. Strongly Typed IDs y la Regla de Asignación de Memoria

El uso de tipos primitivos (`Guid`, `int`, `string`) conduce a la **Primitive Obsession**, permitiendo que un desarrollador pase accidentalmente un `UserId` en lugar de un `PrestamoId`.

### Regla Técnica de Rendimiento (Allocation Awareness):
- **IDs de un solo campo / Ligeros ($\le 16$–$32$ bytes):** Se definen como `readonly record struct` para garantizar **cero asignación en Heap** (Stack allocation).
- **IDs Compuestos o Value Objects Grandes ($> 16$–$32$ bytes / múltiples campos):** Se definen como `readonly record` (Reference type) para evitar el costo de copiar grandes estructuras por valor en el stack frame del CPU.

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo;

// ID Ligero (<= 16 bytes) -> Struct para cero asignación en Heap
public readonly record struct PrestamoId(Guid Value)
{
    public static PrestamoId New() => new(Guid.NewGuid());
    public static PrestamoId From(Guid value) => new(value);
    public override string ToString() => Value.ToString();
}

// ID Compuesto (> 16-32 bytes) -> Reference record para evitar copiar datos en el stack
public readonly record CompositeTenantEntityId(Guid TenantId, Guid EntityId, int Sequence);
```

---

## 🧱 2. Aggregate Root y Design by Contract

El **Aggregate Root** es la única raíz de entrada para modificar cualquier entidad de su grupo. Acumula eventos de dominio e impone un **constructor privado** y **métodos de fábrica estáticos** (`Create` / `NuevoPrestamo`).

```csharp
namespace Core.SharedKernel.Domain;

public interface IAggregateRoot
{
    IReadOnlyCollection<IDomainEvent> DomainEvents { get; }
    void ClearDomainEvents();
}

public abstract class AggregateRoot<TId>(TId id) : Entity<TId>(id), IAggregateRoot 
    where TId : notnull
{
    private readonly List<IDomainEvent> _domainEvents = [];
    public IReadOnlyCollection<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    protected void AddDomainEvent(IDomainEvent domainEvent) => _domainEvents.Add(domainEvent);
    public void ClearDomainEvents() => _domainEvents.Clear();
}
```

### Implementación Rica en la Entidad Principal (`Prestamo.cs`)

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo;

public class Prestamo : AggregateRoot<PrestamoId>
{
    public BeneficiarioId BeneficiarioId { get; private set; }
    public Dinero SaldoPendiente { get; private set; }
    public OrigenApertura Origen { get; private set; }
    public CondicionesOriginales Condiciones { get; private set; }
    public EstadoPrestamo.EstadoPrestamo Estado { get; private set; } = new EstadoPendiente();

    public bool EstaActivo => Estado is EstadoActivo;

    // Constructor Privado: Único inicializador del estado interno
    private Prestamo(PrestamoId id, BeneficiarioId beneficiarioId, OrigenApertura origen,
        CondicionesOriginales condiciones, Dinero saldoInicial) : base(id)
    {
        BeneficiarioId = beneficiarioId;
        Origen = origen;
        Condiciones = condiciones;
        SaldoPendiente = saldoInicial;
    }

    // Static Factory Method: Garantiza la creación con invariantes válidas
    public static Prestamo NuevoPrestamo(PrestamoId id, BeneficiarioId beneficiarioId,
        CondicionesOriginales condiciones, Dinero capitalInicial)
    {
        if (capitalInicial.EnCeros())
            throw new DomainException("El préstamo no puede aperturarse con capital en ceros.");

        var origen = new OrigenApertura("Nuevo prestamo", null);
        return new Prestamo(id, beneficiarioId, origen, condiciones, capitalInicial);
    }

    public void AutorizarPago(MovimientoId movimientoId, Dinero montoDepositado, CodigoCobranza codigoCobranza, Periodo periodo, DesgloseProcesado resultadoCalculo)
    {
        ValidarPrestamoActivo();

        AddDomainEvent(new PagoAutorizadoEvent(
            Id, movimientoId, montoDepositado, codigoCobranza, periodo, resultadoCalculo
        ));
    }

    private void ValidarPrestamoActivo()
    {
        if (!EstaActivo)
            throw new DomainException("Solo se pueden procesar pagos en préstamos activos.");
    }
}
```

---

## 💎 3. Value Objects Matemáticos Complejos (`Dinero.cs`)

Un **Value Object** se define por sus atributos, es inmutable y valida sus reglas de negocio al instanciarse. 

### Algoritmo de Reparto Exacto (Fowler Allocation Algorithm)

En operaciones financieras, el redondeo simple provoca pérdida de centavos. `Dinero.cs` implementa la distribución de Fowler (piso en centavos + reparto del remanente):

```csharp
namespace Core.SharedKernel.ValueObjects;

public readonly record struct Dinero
{
    private const int CentavosPorUnidad = 100;

    public decimal Monto { get; }
    public string Moneda { get; }

    public Dinero(decimal monto, string moneda = "MXN")
    {
        if (monto < 0)
            throw new DomainException("El monto no debe ser menor que 0.");
        
        Monto = Math.Round(monto, 2, MidpointRounding.AwayFromZero);
        Moneda = moneda;
    }

    public Dinero Sumar(Dinero otro)
    {
        if (Moneda != otro.Moneda) throw new DomainException("No se puede sumar dinero de distintas monedas.");
        return new Dinero(Monto + otro.Monto, Moneda);
    }

    /// <summary>
    /// Reparte el monto proporcionalmente garantizando que la suma final
    /// sea EXACTAMENTE igual al monto original (Fowler Allocation).
    /// </summary>
    public Dinero[] Repartir(Proporciones proporciones)
    {
        if (proporciones.Valores.Count == 0)
            throw new DomainException("Las proporciones deben contener al menos un valor.");

        var totalCentavos = Convert.ToInt64(Monto * CentavosPorUnidad);
        var baseCentavos = new long[proporciones.Valores.Count];
        long sumaBase = 0;

        for (var i = 0; i < proporciones.Valores.Count; i++)
        {
            var asignacion = totalCentavos * proporciones.Valores[i];
            var piso = (long)Math.Floor(asignacion);
            baseCentavos[i] = piso;
            sumaBase += piso;
        }

        var remanente = totalCentavos - sumaBase;
        for (var i = 0; remanente > 0; i = (i + 1) % baseCentavos.Length)
        {
            baseCentavos[i] += 1;
            remanente--;
        }

        var resultado = new Dinero[baseCentavos.Length];
        for (var i = 0; i < baseCentavos.Length; i++)
            resultado[i] = new Dinero(baseCentavos[i] / (decimal)CentavosPorUnidad, Moneda);

        return resultado;
    }
}
```

---

## 📌 Puntos Clave para la Presentación

1. **Design by Contract**: Constructores privados + Static Factory Methods (`NuevoPrestamo`).
2. **Value Object Performance Rules**: Structs para IDs de 1 solo campo; Records de referencia para Value Objects compuestos.
3. **Encapsulación Cero Setter**: Ninguna propiedad del dominio expone `public set`.
