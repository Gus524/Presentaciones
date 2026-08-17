# Patrón Factory y Reconstitución de Dominio en DDD (.NET)

> **Nota de Arquitectura:** En Domain-Driven Design (DDD), la forma en que los objetos nacen y resucitan es crítica. Este documento explica el patrón **Static Factory Methods con Constructor Privado** y el patrón de **Reconstitución de Dominio (`Reconstruir`)**, marcando una frontera clara entre la intención del negocio y la hidratación desde la base de datos.

---

## 🔒 1. Constructor Privado Único y Métodos Estáticos de Fábrica

### El Problema de los Constructores Públicos Tradicionales
Exponer constructores `public` en Agregados o Entidades de Dominio acarrea tres graves vicios de diseño:
1. **Modelos Anémicos**: Permiten crear objetos en estados parciales o inválidos (ej. un `Prestamo` sin beneficiario o con saldo negativo).
2. **Ambigüedad semántica**: `new Prestamo(...)` no comunica *por qué* ni *cómo* se está creando el objeto según el Lenguaje Ubicuo.
3. **Acoplamiento con la Infraestructura**: Obliga a mantener constructores vacíos requeridos por ORMs (como EF Core), destruyendo la encapsulación.

### La Solución: Static Factory Methods
Mantenemos el constructor **privado** (o `internal`) como el único canal de instanciación del objeto, y exponemos **métodos estáticos públicos** cuyo nombre refleja intencionalidad de negocio.

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo;

public class Prestamo : AggregateRoot<PrestamoId>
{
    public BeneficiarioId BeneficiarioId { get; private set; }
    public Dinero SaldoPendiente { get; private set; }
    public OrigenApertura Origen { get; private set; }
    public CondicionesOriginales Condiciones { get; private set; }
    public EstadoPrestamo.EstadoPrestamo Estado { get; private set; }

    // 1. CONSTRUCTOR PRIVADO ÚNICO: Garantiza la asignación de estado de manera controlada.
    private Prestamo(
        PrestamoId id, 
        BeneficiarioId beneficiarioId, 
        OrigenApertura origen,
        CondicionesOriginales condiciones, 
        Dinero saldoInicial,
        EstadoPrestamo.EstadoPrestamo estado
    ) : base(id)
    {
        BeneficiarioId = beneficiarioId;
        Origen = origen;
        Condiciones = condiciones;
        SaldoPendiente = saldoInicial;
        Estado = estado;
    }

    // 2. FACTORY METHOD DE NEGOCIO (Creación Nueva)
    public static Prestamo NuevoPrestamo(
        PrestamoId id, 
        BeneficiarioId beneficiarioId,
        CondicionesOriginales condiciones, 
        Dinero capitalInicial
    )
    {
        // Reglas de negocio e invariantes de creación
        if (capitalInicial.Monto <= 0)
            throw new DomainException("El capital inicial de un nuevo préstamo debe ser mayor a cero.");

        var origen = new OrigenApertura("Nuevo préstamo originado por sistema", null);
        var prestamo = new Prestamo(id, beneficiarioId, origen, condiciones, capitalInicial, new EstadoPendiente());

        // Registrar evento de dominio que notifica la creación a otros Bounded Contexts
        prestamo.AddDomainEvent(new PrestamoCreadoEvent(prestamo.Id, beneficiarioId, capitalInicial));

        return prestamo;
    }
}
```

---

## ⚡ 2. Creación Nueva vs. Reconstitución desde Persistencia

Existe una diferencia fundamental entre la **Creación** de una entidad y su **Reconstitución**:

| Criterio | Creación Nueva (`NuevoPrestamo` / `Crear`) | Reconstitución de Dominio (`Reconstruir` / ACL Mapper) |
| :--- | :--- | :--- |
| **Origen del Trigger** | Petición de Usuario / Caso de Uso en Aplicación | Lectura de datos desde la Base de Datos (EF Core DB Entity) |
| **Validación de Reglas** | **SÍ**. Ejecuta invariantes del negocio actual (monto > 0, etc.) | **NO**. Los datos históricos ya fueron validados en el pasado al persistirse |
| **Eventos de Dominio** | **SÍ**. Emite eventos (ej. `PrestamoCreadoEvent`) | **NUNCA**. Disparar eventos al leer de la BD causaría efectos secundarios duplicados |
| **Asignación de Estado** | Inicializa estados por defecto (ej. `EstadoPendiente`) | Restaura el estado histórico exacto traído de la BD (ej. `EstadoActivo`) |

---

## 🛠️ 3. El Método `Reconstruir` a Nivel de Dominio

Para permitir que la Capa Anticorrupción (ACL / Mapper) restaure Agregados sin activar lógica de creación ni emitiendo eventos de dominio, añadimos el método estático de fábrica `Reconstruir`:

```csharp
namespace Core.Prestamos.Domain.Model.Prestamo;

public class Prestamo : AggregateRoot<PrestamoId>
{
    // ... campos y constructor privado ...

    /// <summary>
    /// Método de fábrica exclusivo para la Capa Anticorrupción (ACL) y Repositorios.
    /// Reconstituye el objeto desde la persistencia SIN ejecutar validaciones de creación 
    /// ni emitir Eventos de Dominio.
    /// </summary>
    public static Prestamo Reconstruir(
        PrestamoId id,
        BeneficiarioId beneficiarioId,
        OrigenApertura origen,
        CondicionesOriginales condiciones,
        Dinero saldoPendiente,
        EstadoPrestamo.EstadoPrestamo estado
    )
    {
        // Se llama directamente al constructor privado con los datos exactos de la BD.
        // No se invocan métodos como AddDomainEvent(...).
        return new Prestamo(
            id, 
            beneficiarioId, 
            origen, 
            condiciones, 
            saldoPendiente, 
            estado
        );
    }
}
```

---

## 🛡️ 4. Uso del Método `Reconstruir` en el Mapeador ACL (`IMapper`)

En la Capa de Persistencia (`Persistence/Mappers`), el mapeador ACL consume la función `Reconstruir` para transformar el POCO de BD (`PrestamoDbEntity`) al Agregado Puro de Dominio (`Prestamo`):

### 📄 [`Persistence/Mappers/PrestamoMapper.cs`](file:///var/home/goodgus/Documents/Github/DDD-Templates/templates/dotnet/Persistence/Mappers/PrestamoMapper.cs)

```csharp
using Core.Prestamos.Domain.Model.Beneficiario;
using Core.Prestamos.Domain.Model.Prestamo;
using Core.Prestamos.Domain.Model.Prestamo.EstadoPrestamo;
using Core.Prestamos.Domain.ValueObjects;
using Core.SharedKernel.ValueObjects;
using Persistence.Entities;

namespace Persistence.Mappers;

public class PrestamoMapper : IMapper<Prestamo, PrestamoDbEntity>
{
    public Prestamo MapToDomain(PrestamoDbEntity persistence)
    {
        // 1. Mapear primitivos relacionales a Value Objects del Dominio
        var id = new PrestamoId(persistence.Id);
        var beneficiarioId = new BeneficiarioId(persistence.BeneficiarioId);
        var origen = new OrigenApertura(persistence.OrigenConcepto, persistence.OrigenFolio);
        var condiciones = new CondicionesOriginales(
            (Periodicidad)persistence.PeriodicidadId,
            new TasaInteres(persistence.TasaInteres),
            new PlazoMeses(persistence.PlazoMeses)
        );
        var saldoPendiente = new Dinero(persistence.SaldoPendiente, persistence.Moneda);
        var estado = MapearEstadoDesdeCodigo(persistence.EstadoCodigo);

        // 2. RECONSTITUCIÓN PURA: Llama a Prestamo.Reconstruir(...) 
        // No dispara validaciones de "Nuevo Prestamo" ni emite PrestamoCreadoEvent
        return Prestamo.Reconstruir(
            id, 
            beneficiarioId, 
            origen, 
            condiciones, 
            saldoPendiente, 
            estado
        );
    }

    public PrestamoDbEntity MapToPersistence(Prestamo domain)
    {
        var entity = new PrestamoDbEntity();
        MapToExistingPersistence(domain, entity);
        return entity;
    }

    public void MapToExistingPersistence(Prestamo domain, PrestamoDbEntity persistence)
    {
        persistence.Id = domain.Id.Value;
        persistence.BeneficiarioId = domain.BeneficiarioId.Value;
        persistence.SaldoPendiente = domain.SaldoPendiente.Monto;
        persistence.Moneda = domain.SaldoPendiente.Moneda;
        persistence.OrigenConcepto = domain.Origen.Concepto;
        persistence.OrigenFolio = domain.Origen.Folio;
        persistence.PeriodicidadId = (int)domain.Condiciones.Periodicidad;
        persistence.TasaInteres = domain.Condiciones.TasaInteres.Valor;
        persistence.PlazoMeses = domain.Condiciones.PlazoMeses.Valor;
        persistence.EstadoCodigo = domain.Estado.GetType().Name;
    }

    private static EstadoPrestamo MapearEstadoDesdeCodigo(string codigo) => codigo switch
    {
        nameof(EstadoActivo) => new EstadoActivo(),
        nameof(EstadoPendiente) => new EstadoPendiente(),
        nameof(EstadoFiniquitado) => new EstadoFiniquitado(),
        _ => new EstadoPendiente()
    };
}
```

---

## 📌 Reglas de Oro para el Arquitecto

1. **PROHIBIDO Constructores Públicos en Agregados**: El estado de un Agregado o Entidad NUNCA debe ser directamente alterable desde fuera mediante `new MyEntity()`.
2. **MÉTODOS DE FÁBRICA EXPRESIVOS**: Usar nombres como `NuevoPrestamo`, `RegistrarUsuario`, `CrearCuenta` para nuevas entidades en lugar de constructores genéricos.
3. **SEPARACIÓN DE RESPONSABILIDADES DE INSTANCIACIÓN**:
   - `NuevoPrestamo(...)`: Para **Nuevas Entidades** (Valida invariantes actuales y emite Domain Events).
   - `Reconstruir(...)`: Para **Lectura desde BD** (Acepta el estado tal como fue persistido, sin validar nuevamente ni re-emitir eventos).
