## Arquitectura de Bounded Contexts

## Mapa de Contextos

### Diagrama General: Relaciones Upstream/Downstream

``` mermaid
---

config:

layout: dagre

---

flowchart TB

subgraph CoreSubdomain["CORE: BENEFICIOS"]

direction TB

Ops["⚙️ Operaciones BC"]

Loan["📊 Préstamos BC"]

Savings["💰 Ahorros BC"]

end

subgraph AffiliationSubdomain["AFILIACIÓN"]

Affiliation["Afiliación BC"]

end

subgraph PoliciesSubdomain["POLÍTICAS BENEFICIOS"]

Products["Productos BC"]

end

subgraph AccountingSubdomain["CONTABILIDAD"]

Accounting["Contabilidad BC"]

end

subgraph SecuritySubdomain["GENERIC: SEGURIDAD"]

IAM["IAM BC"]

end

IAM -- Upstream (Identity Provider) --> Affiliation

Affiliation -- Upstream (Supplier) --> Ops & Loan & Savings

Products -- Upstream (Policy Provider) --> Loan & Savings & Ops

Loan -- Upstream (Domain Events) --> Accounting

Savings -- Upstream (Domain Events) --> Accounting

Ops -- Upstream (Domain Events) --> Accounting

Ops -- Downstream (Consults) --> Loan & Savings

  

style Ops fill:#fff,stroke:#d4a017,stroke-width:2px

style Loan fill:#fff,stroke:#d4a017,stroke-width:2px

style Savings fill:#fff,stroke:#d4a017,stroke-width:2px

style CoreSubdomain fill:#fff4dd,stroke:#d4a017,stroke-width:2px

style AffiliationSubdomain fill:#e1f5fe,stroke:#01579b,stroke-dasharray: 5 5

style PoliciesSubdomain fill:#e1f5fe,stroke:#01579b,stroke-dasharray: 5 5

style AccountingSubdomain fill:#e1f5fe,stroke:#01579b,stroke-dasharray: 5 5

style SecuritySubdomain fill:#f5f5f5,stroke:#9e9e9e
```

---

## Dominios y Subdominios

### Core Domains (Núcleo del Negocio)

Los tres contextos Core contienen la lógica diferenciadora del negocio.

#### 🎯 1. Préstamos Context (Core Domain)

**Responsabilidad:** Gestionar la vida financiera del préstamo post-otorgamiento.

**Puntos Relevantes:**
- Cálculo de amortización y saldos insolutos
- Gestión de Intereses Moratorios y devengados
- Control del estado del préstamo (Pendiente, Aprobado, Cancelado, Terminado)

**Domain Events:**
- `PrestamoAprobado`
- `PrestamoLiquidado`

**Dependencias:**
- ✅ DOWNSTREAM: Productos Context (tasas de interés)
- ✅ DOWNSTREAM: Contabilidad Context (registros de movimientos)
- ✅ Recibe notificaciones UPSTREAM: Operaciones Context

---

#### 💰 2. Ahorros Context (Core Domain)

**Responsabilidad:** Gestionar la captación de capital y generación de valor para afiliados.

**Puntos Relevantes:**
- Administración de cuentas de ahorro y capitalización de rendimientos
- Control de saldos (Principal vs. Rendimiento)
- Aplicación de lógica de prelación en retiros (Rendimiento primero, Capital después)

**Domain Events:**
- `CuentaAhorroCreada`
- `ContribucionRecibida`
- `CuentaAhorroCerrada`

**Dependencias:**
- ✅ DOWNSTREAM: Productos Context (configuración de tasas)
- ✅ DOWNSTREAM: Contabilidad Context (registros de movimientos)
- ✅ Recibe notificaciones UPSTREAM: Operaciones Context, Afiliación Context

---

#### ⚙️ 3. Operaciones Context (Core Domain)

**Responsabilidad:** Orquestar procesos complejos que involucran múltiples contextos. Es el "Cerebro" transaccional que formaliza las voluntades del afiliado.

**Puntos Relevantes:**
- Orquestación de **Refinanciamiento** (pago de deuda con crédito nuevo)
- Orquestación de **Ampliación** (aumento de límite)
- Validación de reglas de desvinculación (Socio Fundador, Ahorro tipo dos)
- Cruces de saldos entre Ahorros y Préstamos para liquidaciones automáticas
- Gestión de **Instrucciones** (punto de entrada de voluntades del afiliado)

**Domain Events:**
- `InstruccionCreada`
- `RefinanciamientoSolicitado`
- `AmplificacionSolicitada`
- `AmplificacionAprobada`

**Dependencias:**
- ✅ DOWNSTREAM: Préstamos Context (consulta saldos, validaciones)
- ✅ DOWNSTREAM: Ahorros Context (consulta saldos, validaciones)
- ✅ DOWNSTREAM: Productos Context (obtiene definiciones de operaciones permitidas)
- ✅ UPSTREAM a: Préstamos Context, Ahorros Context
- ✅ Recibe notificaciones UPSTREAM: Afiliación Context

---

### Supporting Subdomains (Apoyo Especializado)

Contextos que proporcionan capacidades de apoyo pero no son diferenciadores del negocio.

#### 📋 4. Contabilidad Context (Supporting Subdomain)

**Responsabilidad:** Gestionar la relación financiera con el mundo exterior (Bancos) y garantizar consistencia de movimientos.

**Puntos Relevantes:**
- Generación de archivos de dispersión (PAG)
- Procesamiento de **Enteros** (carga de pagos y movimientos bancarios)
- Garantiza consistencia entre reportes bancarios y estados de cuenta internos

**Domain Events:**
- `MovimientoPrestamoRegistado`
- `MovimientoAhorroRegistrado`
- `EnteroProcesado`

**Dependencias:**
- ✅ DOWNSTREAM a: Préstamos Context, Ahorros Context, Operaciones Context (registra eventos)

---

#### 📦 5. Productos Context (Supporting Subdomain)

**Responsabilidad:** Definir las "reglas del juego". Catálogo de configuraciones y políticas.

**Puntos Relevantes:**
- Definición de **Accesorios** (comisiones, seguros, cargos extra)
- Configuración de tasas de interés y plazos permitidos
- Plantillas de comportamiento para tipos de Préstamos y Ahorros
- Reglas de validación para operaciones complejas

**Domain Events:**
- `ProductoCreado`
- `ProductoActualizado`
- `AccesorioAgregado`

**Dependencias:**
- ✅ UPSTREAM a: Préstamos Context, Ahorros Context, Operaciones Context (provee definiciones)

---

#### 👥 6. Afiliación Context (Supporting Subdomain)

**Responsabilidad:** Administrar el ciclo de vida del "Maestro" (Afiliado) dentro de la organización.

**Puntos Relevantes:**
- Registro y validación de datos del afiliado
- Gestión de estados (Activo, Inactivo, Fundador)
- Mantenimiento de la jerarquía y roles dentro del sistema de beneficios

**Domain Events:**
- `AfiliacionRegistrada`
- `EstadoAfiliacionActualizada`
- `AfiliadoDadoDeBaja`
- `BeneficioAsignado`

**Dependencias:**
- ✅ UPSTREAM a: Operaciones Context, Préstamos Context, Ahorros Context, IAM Context (notifica cambios de estado)

---

### Generic Subdomains (Infraestructura)

Contextos genéricos que podrían ser reemplazados por soluciones off-the-shelf, pero son críticos para operar.

#### 🔑 7. IAM Context (Generic Subdomain)

**Responsabilidad:** Infraestructura de seguridad y acceso a todos los contextos.

**Puntos Relevantes:**
- Autenticación de usuarios y gestión de credenciales
- Autorización basada en roles (Secretarios, Contadores, Administradores)
- Auditoría de acceso a diferentes módulos

**Dependencias:**
- ✅ Dependencia GENERIC: Todos los contextos (validación transversal)

---

## Relaciones entre Contextos

### Patrón Upstream/Downstream

En DDD, las relaciones se clasifican como:

| Relación | Descripción | Ejemplo |
|:---|:---|:---|
| **DOWNSTREAM** | A depende de B; B es autoridad | Préstamos *depende de* Productos |
| **UPSTREAM** | A notifica a B; A es autoridad | Afiliación *notifica a* Operaciones |
| **GENERIC** | Relación transversal de infraestructura | Todos dependen de IAM |

---

## Patrones de Integración

### 1. **Anti-Corruption Layer (ACL)**

Protege los Core Domains de cambios en Productos Context:

```csharp
// En Préstamos Context
public interface IProductDefinitionService
{
    Task<LoanProductDefinition> GetProductAsync(string productCode);
}

// Implementación en la aplicación (puede usar HTTP, gRPC, etc.)
public class ProductDefinitionAdapter : IProductDefinitionService
{
    public async Task<LoanProductDefinition> GetProductAsync(string productCode)
    {
        // Traduce la respuesta externa al lenguaje ubicuo de Préstamos
        var externalProduct = await _productsClient.GetAsync(productCode);
        return new LoanProductDefinition(
            externalProduct.Code,
            externalProduct.MinRate,
            externalProduct.MaxRate
        );
    }
}
```

### 2. **Event-Driven Integration**

Los Core Domains se comunican vía Domain Events publicados al Outbox:

```csharp
// En Ahorros Context - generar evento
await _repository.AddAsync(savingsAccount);
savingsAccount.AddDomainEvent(new ContributionReceivedEvent(
    affiliateId,
    amount,
    accountId
));

// En Operaciones Context - subscribirse
public class ContributionReceivedEventHandler : IEventHandler<ContributionReceivedEvent>
{
    public async Task HandleAsync(ContributionReceivedEvent @event)
    {
        // Procesar en contexto de Operaciones
        await _operationOrchestrator.ProcessContributionAsync(@event);
    }
}
```

### 3. **Published Language (PL)**

Define contratos claros entre contextos para mensajería y APIs:

```csharp
// Contrato compartido (en SharedKernel)
public record LoanApprovedMessage
{
    public Guid LoanId { get; init; }
    public Guid AffiliateId { get; init; }
    public decimal Amount { get; init; }
    public DateTime ApprovalDate { get; init; }
}
```

### 4. **Conformist Pattern**

Cuando un Supporting Subdomain no puede cambiar, el Consumer se conforma:

```csharp
// Afiliación Context no cambia; Préstamos se adapta
public class AffiliateDisaffiliationHandler
{
    private readonly IRepository<Loan, LoanId> _loanRepository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task HandleAsync(AffiliateDisaffiliatedEvent @event)
    {
        // Nota: GetByAffiliateAsync es una extensión específica del BC de Préstamos
        // Para este ejemplo, asumimos que existe
        var loansToSettle = await _loanRepository
            .GetByAffiliateAsync(@event.AffiliateId);
        
        foreach (var loan in loansToSettle)
        {
            loan.SettleOnDisaffiliation();
            _loanRepository.Update(loan);  // ← Síncrono (cambios en memoria)
        }
        
        await _unitOfWork.SaveChangesAsync();  // ← Persistencia aplazada
    }
}
```

---

## Recomendaciones Arquitectónicas

### ✅ Do's (Recomendado)

1. **Mantener independencia de Core Domains:** Préstamos, Ahorros y Operaciones NO deben conocerse entre sí directamente. Comunícate vía Domain Events.

2. **Usar el Lenguaje Ubicuo:** Los nombres de clases, métodos y eventos deben ser comprensibles para expertos del dominio.

3. **Validar en Bordes:** Las ACLs traducen y validan datos externos antes de entrar al Core Domain.

4. **Documentar Eventos Críticos:** Cada evento publicado debe estar documentado con su contexto de origen y subscribers esperados.

### ❌ Don'ts (Evitar)

1. ❌ **No crear tablas compartidas:** Cada BC maneja sus propias tablas (Multi-Tenancy per tenant).

2. ❌ **No importar entidades entre contextos:** Usa IDs (Aggregate Root IDs) en lugar de referencias de objetos.

3. ❌ **No crear "servicios globales":** Problemas de acoplamiento. Usa eventos en su lugar.

4. ❌ **No ignorar el Lenguaje Ubicuo:** Los nombres técnicos deben reflejar la realidad del negocio.
