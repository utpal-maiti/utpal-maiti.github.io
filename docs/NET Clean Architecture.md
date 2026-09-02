### .NET Clean Architecture — Complete Guide

---

#### 1. What is it?
**.NET Clean Architecture** is an architectural style and set of practices for building maintainable, testable, and loosely coupled .NET applications by separating concerns into distinct layers and enforcing dependency rules so that business rules are independent of frameworks, UI, and infrastructure. 

---

#### 2. Why was it created?
It was created to address the long-term maintainability and testability problems of monolithic, tightly coupled applications by making business logic independent of delivery mechanisms and infrastructure so teams can change UI, database, or frameworks without rewriting core rules. 

---

#### 3. What problem does it solve?
- **Coupling of business logic to frameworks** — prevents framework lock-in.  
- **Poor testability** — isolates domain logic so it can be unit tested without infrastructure.  
- **Hard-to-change code** — enforces clear boundaries so changes in one layer don’t ripple across the system. 

---

#### 4. Who uses it?
- Enterprise teams building long-lived systems.  
- Teams needing high test coverage and frequent change.  
- Architects and developers building microservices, web APIs, and complex domain-driven systems in .NET. 

---

#### 5. Where is it used?
- Web APIs (ASP.NET Core)  
- Background workers and services (Worker Service)  
- Desktop apps (WPF, WinForms) when separation is needed  
- Microservices and modular monoliths in enterprise systems. 

---

#### 6. When should I use it?
Use Clean Architecture when:
- The project is medium-to-large or expected to evolve over years.  
- You need clear separation for testing, multiple UIs, or multiple data stores.  
- You want to avoid framework lock-in and enable parallel team development. 

---

#### 7. Which concepts are most important?
- **Entities / Domain Models** — core business objects.  
- **Use Cases / Application Services** — orchestrate business rules.  
- **Interfaces / Ports** — abstractions for infrastructure (e.g., repositories, email senders).  
- **Adapters / Infrastructure** — concrete implementations (EF Core, SMTP, file system).  
- **Dependency Rule** — inner layers must not depend on outer layers; dependencies point inward.  
- **DTOs / ViewModels** — boundary objects for UI and external systems. 

---

#### 8. How does it work?
- **Layering**: Typical layers are **Domain (Entities, Value Objects)** → **Application (Use Cases, Interfaces)** → **Infrastructure (EF Core, External APIs)** → **Presentation (API, UI)**.  
- **Inversion of Control**: Outer layers implement interfaces defined by inner layers; dependency injection wires concrete implementations at composition root.  
- **Ports and Adapters**: The application defines ports (interfaces); adapters implement them to talk to databases, web, or other systems. 

---

#### 9. How is it implemented?
**Typical .NET solution structure (folders/projects):**

| Project | Responsibility |
|---|---|
| **MyApp.Domain** | Entities, value objects, domain services, domain exceptions |
| **MyApp.Application** | Use cases (commands/queries), interfaces (repositories, services), DTOs |
| **MyApp.Infrastructure** | EF Core DbContext, repository implementations, external API clients |
| **MyApp.API** | ASP.NET Core Web API controllers, DI composition root, middleware |
| **MyApp.Tests** | Unit and integration tests |

**Implementation steps (high level):**
1. Model domain entities and invariants in `Domain`.  
2. Define use case interfaces and DTOs in `Application`.  
3. Implement repositories and external integrations in `Infrastructure`.  
4. Build controllers in `API` that call application services.  
5. Wire dependencies in `Program.cs` / composition root. 

---

#### 10. How is it secured?
- **Authentication & Authorization** at the presentation layer (ASP.NET Core Identity, JWT, OAuth2).  
- **Policy-based authorization** in application services to enforce business rules.  
- **Input validation** at boundaries (FluentValidation).  
- **Secrets management** (Azure Key Vault, user secrets).  
- **Secure infrastructure**: parameterized queries, least-privilege DB accounts, TLS, logging/monitoring.  
- **Defense in depth**: validate at UI, application, and domain levels. 

---

#### 11. How is it scaled?
- **Vertical scaling**: optimize DB, caching (Redis), and app instances.  
- **Horizontal scaling**: stateless API instances behind load balancers; state in distributed caches or databases.  
- **Microservices**: split bounded contexts into separate services following the same Clean Architecture principles.  
- **CQRS and Event Sourcing**: separate read/write workloads for scale and performance.  
- **Asynchronous messaging**: use message brokers (RabbitMQ, Azure Service Bus) for decoupling and resilience. 

---

#### 12. Real-world use cases
- **Banking**: transaction rules in domain, multiple channels (web, mobile).  
- **E-commerce**: product/catalog domain, order processing, payment adapters.  
- **Healthcare**: strict domain rules, auditability, multiple integrations.  
- **SaaS platforms**: multi-tenant logic isolated in domain, pluggable infrastructure. 

---

#### 13. Common mistakes
- **Leaking frameworks into domain** (e.g., EF Core attributes in entities).  
- **Fat controllers** that contain business logic.  
- **Over-abstraction**: too many layers and interfaces for small projects.  
- **Wrong dependency direction**: inner layers referencing infrastructure.  
- **Not testing boundaries**: skipping unit tests for use cases. 

---

#### 14. Interview questions
- What is the Dependency Rule in Clean Architecture?  
- How do you structure a .NET solution for Clean Architecture?  
- How do you prevent EF Core from leaking into domain models?  
- Explain Ports and Adapters with an example.  
- When would Clean Architecture be overkill?  
- How do you implement transactions and unit of work?  
- How do you test application services in isolation?  
- Explain how to implement CQRS in Clean Architecture.  

(See section 21 for expert Q&A with sample answers and code snippets.)

---

#### 15. How would a Solution Architect explain and design it?
A Solution Architect would:
- **Identify bounded contexts** and domain boundaries.  
- **Define domain model** and core use cases.  
- **Map integration points** (databases, external APIs, messaging).  
- **Choose technology stack** (ASP.NET Core, EF Core, Redis, Azure Service Bus).  
- **Design solution layout** into projects (Domain, Application, Infrastructure, API).  
- **Define CI/CD, observability, and security** requirements.  
- **Create a composition root** to wire implementations to interfaces.  
- **Plan for scaling** (stateless services, caching, message-driven workflows). 

---

#### 16. Core Concepts with real examples

##### Domain Entity (C#)
```csharp
public class Order
{
    public Guid Id { get; private set; }
    public DateTime CreatedAt { get; private set; }
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyCollection<OrderLine> Lines => _lines.AsReadOnly();

    public void AddLine(Product product, int qty)
    {
        if(qty <= 0) throw new DomainException("Quantity must be > 0");
        _lines.Add(new OrderLine(product.Id, product.Price, qty));
    }
}
```

##### Application Use Case (C#)
```csharp
public interface IPlaceOrder
{
    Task<Guid> ExecuteAsync(PlaceOrderDto dto);
}

public class PlaceOrderService : IPlaceOrder
{
    private readonly IOrderRepository _orders;
    private readonly IUnitOfWork _uow;

    public async Task<Guid> ExecuteAsync(PlaceOrderDto dto)
    {
        var order = new Order();
        // map dto to domain, apply rules
        _orders.Add(order);
        await _uow.CommitAsync();
        return order.Id;
    }
}
```

##### Infrastructure (EF Core repository)
```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;
    public void Add(Order order) => _db.Orders.Add(order);
}
```
These examples show **domain-first** design and that infrastructure implements interfaces defined by application. 

---

#### 17. Architectural Patterns with real examples
- **Ports and Adapters (Hexagonal)**: Application defines `IEmailSender`; Infrastructure implements `SmtpEmailSender`.  
- **Onion Architecture**: Domain at center, application around it, infrastructure outermost.  
- **CQRS**: `CreateOrderCommand` handled by command handler; `OrderQuery` handled by read model optimized for queries.  
- **Event-Driven**: Domain raises `OrderPlaced` event; integration publishes to message broker for downstream processing. 

---

#### 18. Enterprise Use Cases with real examples
- **Payment Processing**: Domain enforces payment rules; infrastructure plugs in Stripe/PayPal adapters; asynchronous reconciliation via events.  
- **Inventory Management**: Domain models stock rules; read model for fast queries; background worker for restock alerts.  
- **Reporting**: Use event handlers to project domain events into analytics store (e.g., Azure Data Lake). 

---

#### 19. Practical Example (mini project)
**Project: Simple E-commerce Order Service**

**Solution layout**
- `Eshop.Domain` — `Order`, `Product`, `Customer`  
- `Eshop.Application` — `PlaceOrder`, `GetOrder`, DTOs, interfaces `IOrderRepository`, `IUnitOfWork`  
- `Eshop.Infrastructure` — `EfOrderRepository`, `AppDbContext`, `SqlServer` config  
- `Eshop.API` — ASP.NET Core controllers, `Program.cs` DI composition  
- `Eshop.Tests` — unit tests for `PlaceOrderService`, integration tests with in-memory DB

**Flow**
1. `POST /orders` → Controller maps request to `PlaceOrderDto`.  
2. Controller calls `IPlaceOrder.ExecuteAsync`.  
3. `PlaceOrderService` creates `Order` domain entity, applies rules, calls `IOrderRepository.Add`.  
4. `UnitOfWork.CommitAsync` saves via EF Core.  
5. Domain event `OrderPlaced` published to message bus for fulfillment. 

---

#### 20. Quick Gist with real examples
- Keep **domain pure** (no EF attributes).  
- Define **interfaces** in application layer.  
- Implement **infrastructure** outside domain.  
- Use **DI** at composition root.  
- Prefer **small, focused services** (one use case per service).  
- Example: `IOrderRepository` in `Application`, `EfOrderRepository` in `Infrastructure`. 

---

#### 21. Expert-Level Interview Questions & Answers with examples

##### Q1: How do you enforce the Dependency Rule in .NET?
**A:** Put interfaces in the Application layer and have Infrastructure implement them. Use DI in `Program.cs` to bind implementations. Avoid referencing Infrastructure from Domain or Application projects.  
**Code snippet:**
```csharp
// Program.cs
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
```
##### Q2: How do you prevent EF Core from leaking into domain models?
**A:** Use persistence models (EF Core entity configurations) or mapping layers (AutoMapper) and keep domain entities free of EF attributes. Use value objects and private setters.  
##### Q3: How to implement transactions?
**A:** Implement `IUnitOfWork` in Infrastructure that wraps `DbContext.SaveChangesAsync()` and inject it into application services. Use explicit transaction scopes for multi-repository operations.  
##### Q4: How to test application services?
**A:** Mock repository interfaces and unit of work; test use case logic in isolation. For integration tests, use an in-memory or test container DB.  
##### Q5: How to implement CQRS and Event Sourcing with Clean Architecture?
**A:** Keep command handlers in Application; read models in a separate read-side project; persist events in an event store; project events to read models via handlers. Use message bus for eventual consistency. 

---

## Architecture Diagrams (textual)
```
[API / UI]
   ↓
[Application Layer]  <-- interfaces (IOrderRepository, IEmailSender)
   ↓
[Domain Layer]       <-- Entities, Value Objects, Domain Services
   ↑
[Infrastructure Layer] <-- EF Core, SMTP, External APIs (implements interfaces)
```

---

## Best Practices
- Start domain-first: model business rules before tech choices.  
- Keep layers small and focused.  
- Use meaningful interfaces; avoid interface explosion.  
- Favor composition root for wiring dependencies.  
- Write unit tests for use cases and domain invariants.  
- Use integration tests for infrastructure.  
- Keep DTOs separate from domain models.  
- Use logging, metrics, and distributed tracing for observability. 

---

## Sample Project (Git-style README summary)

**Eshop (sample)**

- `Eshop.Domain` — domain models  
- `Eshop.Application` — use cases, interfaces, DTOs  
- `Eshop.Infrastructure` — EF Core, repository implementations  
- `Eshop.API` — ASP.NET Core Web API  
- `Eshop.Tests` — unit & integration tests

**Run**
1. `dotnet restore`  
2. `dotnet ef database update --project Eshop.Infrastructure`  
3. `dotnet run --project Eshop.API`  
4. `POST /orders` with JSON `{ "customerId": "...", "items":[{ "productId":"...", "qty":1 }] }`

---

If you want, I can:
- Provide a **complete starter repository** scaffold (file tree + key code files).  
- Generate **detailed code for each project** (Domain, Application, Infrastructure, API).  
- Produce **sample unit and integration tests** for the `PlaceOrder` use case.