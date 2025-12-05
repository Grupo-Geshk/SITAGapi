# SITAGapi - Sistema de Trazabilidad Ganadera

API REST para la gestión integral de fincas ganaderas, animales, eventos sanitarios, servicios, inventario de insumos y trabajadores.

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Características Principales](#características-principales)
- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Migraciones de Base de Datos](#migraciones-de-base-de-datos)
- [Ejecución](#ejecución)
- [Endpoints Principales](#endpoints-principales)
- [Autenticación y Autorización](#autenticación-y-autorización)
- [Modelos de Dominio](#modelos-de-dominio)
- [Reglas de Negocio](#reglas-de-negocio)
- [Testing](#testing)
- [Deployment](#deployment)
- [Stack Tecnológico](#stack-tecnológico)
- [Contribución](#contribución)
- [Licencia](#licencia)

---

## 📖 Descripción

**SITAGapi** es el backend del Sistema de Trazabilidad Ganadera (SiTaG), diseñado para productores ganaderos en Chiriquí, Panamá. Permite llevar un registro completo del historial de animales, gestionar fincas, movimientos entre ubicaciones, eventos sanitarios, servicios veterinarios, inventario de insumos y trabajadores.

### Objetivos del Proyecto

- Proporcionar trazabilidad completa por animal (origen, ubicación actual, movimientos, peso, salud, producción)
- Facilitar la gestión operativa de fincas (inventario, entradas/salidas, ventas, muertes, traslados)
- Ofrecer consultas rápidas para toma de decisiones (qué animales vender, cuáles están en engorde, etc.)
- Crear una base técnica escalable para futuras integraciones (subastas, mataderos, certificaciones)
- Proveer KPIs para simplificar la toma de decisiones del productor

---

## ✨ Características Principales

- **Multi-tenant**: Separación estricta de datos por productor
- **Gestión de Fincas**: Soporte para fincas propias y no poseídas (arrendadas/cedidas)
- **Trazabilidad de Animales**: Historial completo desde nacimiento hasta venta/muerte
- **Eventos de Vida**: Registro de nacimientos, muertes, compras, ventas, partos, enfermedades
- **Movimientos**: Transferencia de animales entre fincas y divisiones del mismo productor
- **Servicios**: Aplicación de servicios (vacunación, medicación, pesaje, etc.) a uno o varios animales
- **Inventario de Insumos**: Control de stock con alertas de bajo inventario (< 30%)
- **Gestión de Trabajadores**: Asignación de responsables por finca
- **Reportes y KPIs**: Métricas por finca (animales activos, nacimientos/muertes últimos 30 días, servicios recientes)
- **Auditoría**: Registro automático de quién creó/modificó cada registro
- **Soft Delete**: Preservación de históricos sin eliminación física

---

## 🏗️ Arquitectura

El proyecto sigue una **arquitectura por capas** (Layered Architecture) inspirada en Clean Architecture:

```
SITAGapi/
│
├── SITAGapi.Domain/           # Entidades, Value Objects, Enums, Lógica de Dominio
├── SITAGapi.Application/      # Casos de Uso, Servicios, DTOs, Validaciones
├── SITAGapi.Infrastructure/   # EF Core, Repositorios, Persistencia, Migraciones
└── SITAGapi.API/              # Controladores REST, Middleware, Configuración
```

### Principios de Diseño

- **Separación de responsabilidades**: Cada capa tiene un propósito claro
- **Independencia del framework**: La lógica de negocio no depende de EF Core o ASP.NET
- **Testabilidad**: Servicios desacoplados mediante interfaces
- **Multi-tenant by design**: Filtrado automático por `ProducerId`

---

## 📦 Requisitos Previos

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) o superior
- [PostgreSQL 14+](https://www.postgresql.org/download/) (local o remoto)
- [Git](https://git-scm.com/)
- Editor recomendado: [Visual Studio 2022](https://visualstudio.microsoft.com/) o [VS Code](https://code.visualstudio.com/) con extensión C#

---

## 🚀 Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/Grupo-Geshk/SITAGapi.git
cd SITAGapi
```

### 2. Restaurar dependencias

```bash
dotnet restore
```

### 3. Verificar que la solución compile

```bash
dotnet build
```

---

## ⚙️ Configuración

### Variables de Entorno

Crea un archivo `.env` en la raíz del proyecto **SITAGapi.API** con las siguientes variables:

```env
# Base de Datos
DATABASE_CONNECTION_STRING=Host=localhost;Database=sitag_dev;Username=tu_usuario;Password=tu_password

# JWT Configuration
JWT_KEY=tu-clave-secreta-super-segura-de-al-menos-32-caracteres
JWT_ISSUER=SITAGapi
JWT_AUDIENCE=SITAGclient
JWT_EXPIRES_IN_MINUTES=480

# Entorno
ASPNETCORE_ENVIRONMENT=Development
```

> ⚠️ **Importante**: El archivo `.env` NO debe incluirse en el repositorio. Ya está en `.gitignore`.

### Configuración de `appsettings.json`

El archivo `appsettings.json` usa las variables de entorno:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "${DATABASE_CONNECTION_STRING}"
  },
  "Jwt": {
    "Key": "${JWT_KEY}",
    "Issuer": "${JWT_ISSUER}",
    "Audience": "${JWT_AUDIENCE}",
    "ExpiresInMinutes": "${JWT_EXPIRES_IN_MINUTES}"
  }
}
```

---

## 📂 Estructura del Proyecto

```
SITAGapi/
│
├── SITAGapi.Domain/
│   ├── Entities/              # Producer, Farm, Animal, AnimalEvent, etc.
│   ├── ValueObjects/          # AnimalIdentifier, Location, StockQuantity
│   ├── Enums/                 # FarmType, AnimalStatus, EventType, etc.
│   └── Interfaces/            # Contratos de repositorios (opcional)
│
├── SITAGapi.Application/
│   ├── Services/              # AuthService, AnimalService, FarmService, etc.
│   ├── DTOs/                  # Request/Response models
│   ├── Validators/            # FluentValidation rules
│   ├── Mappings/              # Mapster configuration
│   └── Interfaces/            # Contratos de servicios
│
├── SITAGapi.Infrastructure/
│   ├── Data/
│   │   ├── SitagDbContext.cs  # DbContext principal
│   │   └── Configurations/    # Fluent API entity configurations
│   ├── Repositories/          # Implementaciones de repositorios
│   ├── Migrations/            # EF Core migrations
│   └── Services/              # Servicios de infraestructura (logging, etc.)
│
└── SITAGapi.API/
    ├── Controllers/           # Endpoints REST
    ├── Middleware/            # Exception handling, logging
    ├── Extensions/            # ServiceCollection extensions
    ├── appsettings.json       # Configuración base
    └── Program.cs             # Entry point
```

---

## 🗄️ Migraciones de Base de Datos

### Crear una nueva migración

```bash
dotnet ef migrations add NombreDeLaMigracion --project SITAGapi.Infrastructure --startup-project SITAGapi.API
```

### Aplicar migraciones

```bash
dotnet ef database update --project SITAGapi.Infrastructure --startup-project SITAGapi.API
```

### Revertir última migración

```bash
dotnet ef migrations remove --project SITAGapi.Infrastructure --startup-project SITAGapi.API
```

### Ver lista de migraciones

```bash
dotnet ef migrations list --project SITAGapi.Infrastructure --startup-project SITAGapi.API
```

---

## ▶️ Ejecución

### Modo Development

```bash
cd SITAGapi.API
dotnet run
```

La API estará disponible en:
- **HTTP**: `http://localhost:5000`
- **HTTPS**: `https://localhost:5001`
- **Swagger UI**: `https://localhost:5001/swagger`

### Modo Watch (Hot Reload)

```bash
dotnet watch run --project SITAGapi.API
```

---

## 🔌 Endpoints Principales

### Autenticación

```
POST   /api/auth/login          # Iniciar sesión y obtener JWT
POST   /api/auth/register       # Registrar nuevo usuario (AdminSistema)
```

### Productores (Solo AdminSistema)

```
GET    /api/producers           # Listar todos los productores
POST   /api/producers           # Crear productor
PUT    /api/producers/{id}      # Actualizar productor
DELETE /api/producers/{id}      # Desactivar productor
```

### Fincas

```
GET    /api/farms               # Listar fincas del productor autenticado
POST   /api/farms               # Crear finca
PUT    /api/farms/{id}          # Actualizar finca
DELETE /api/farms/{id}          # Desactivar finca
```

### Divisiones

```
GET    /api/divisions?farmId={farmId}  # Listar divisiones de una finca
POST   /api/divisions                  # Crear división
PUT    /api/divisions/{id}             # Actualizar división
DELETE /api/divisions/{id}             # Desactivar división
```

### Animales

```
GET    /api/animals?farmId={farmId}&divisionId={divisionId}&status={status}&page={page}&pageSize={pageSize}
POST   /api/animals             # Registrar nuevo animal
PUT    /api/animals/{id}        # Actualizar datos del animal
GET    /api/animals/{id}        # Detalle del animal + historial completo
```

### Movimientos

```
POST   /api/movements           # Mover animal entre fincas/divisiones
GET    /api/movements/{animalId} # Historial de movimientos de un animal
```

### Eventos

```
GET    /api/animal-events?animalId={animalId}  # Timeline de eventos
POST   /api/animal-events                      # Registrar evento
```

### Servicios

```
GET    /api/services?farmId={farmId}           # Listar servicios
POST   /api/services                           # Aplicar servicio a uno o varios animales
GET    /api/services/{id}                      # Detalle del servicio
```

### Insumos

```
GET    /api/supplies?farmId={farmId}           # Inventario de insumos
POST   /api/supplies                           # Registrar insumo
PUT    /api/supplies/{id}                      # Actualizar stock
GET    /api/supplies/alerts                    # Alertas de bajo stock (< 30%)
```

### Trabajadores

```
GET    /api/workers?farmId={farmId}            # Listar trabajadores
POST   /api/workers                            # Registrar trabajador
PUT    /api/workers/{id}                       # Actualizar trabajador
DELETE /api/workers/{id}                       # Desactivar trabajador
```

### Reportes

```
GET    /api/reports/farm-summary?farmId={farmId}  # KPIs de finca
GET    /api/reports/animal-timeline/{animalId}    # Timeline completo de animal
```

### Backoffice (Solo AdminSistema)

```
GET    /api/admin/producers                    # Vista global de productores
GET    /api/admin/farms                        # Vista global de fincas
GET    /api/admin/animals?producerId={producerId}  # Animales por productor
```

> 📘 **Documentación completa**: Accede a `/swagger` para ver todos los endpoints, modelos y ejemplos.

---

## 🔐 Autenticación y Autorización

### Flujo de Autenticación

1. El cliente envía credenciales a `POST /api/auth/login`
2. El servidor valida las credenciales
3. Si son correctas, devuelve un JWT con:
   - `sub`: ID del usuario
   - `role`: `AdminSistema` o `Productor`
   - `producerId`: ID del productor (si el rol es `Productor`)
   - `exp`: Fecha de expiración

### Uso del Token

Incluir el token JWT en el header `Authorization` de cada request:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Roles y Permisos

| Rol            | Permisos                                                        |
|----------------|-----------------------------------------------------------------|
| **AdminSistema** | Acceso total: CRUD productores, ver todas las fincas y animales |
| **Productor**    | Acceso solo a sus propias fincas, animales, servicios, etc.    |

### Política de Multi-Tenancy

- Todas las consultas y operaciones se filtran automáticamente por `ProducerId`
- Un productor **nunca** puede ver ni modificar datos de otro productor
- El `ProducerId` se extrae del JWT en cada request

---

## 📊 Modelos de Dominio

### Entidades Principales

#### Producer
```csharp
public class Producer : AuditableEntity
{
    public string Name { get; set; }
    public string Phone { get; set; }
    public string? Email { get; set; }
    public ICollection<Farm> Farms { get; set; }
}
```

#### Farm
```csharp
public class Farm : AuditableEntity
{
    public string Name { get; set; }
    public string Location { get; set; }
    public FarmType Type { get; set; }  // Leche, Carne, Mixto
    public FarmTenancyType Tenancy { get; set; }  // Propia, NoPoseida
    public int ProducerId { get; set; }
    public Producer Producer { get; set; }
    public ICollection<Division> Divisions { get; set; }
}
```

#### Animal
```csharp
public class Animal : AuditableEntity
{
    public string Identifier { get; set; }  // Arete único por productor
    public string? Name { get; set; }
    public string Sex { get; set; }
    public string Breed { get; set; }
    public decimal BirthWeight { get; set; }
    public decimal CurrentWeight { get; set; }
    public DateTime BirthDate { get; set; }
    public AnimalStatus Status { get; set; }  // Activo, Vendido, Muerto, Perdido
    public int FarmId { get; set; }
    public Farm Farm { get; set; }
    public int DivisionId { get; set; }
    public Division Division { get; set; }
    public int? FatherId { get; set; }
    public int? MotherId { get; set; }
    public ICollection<AnimalEvent> Events { get; set; }
    public ICollection<Movement> Movements { get; set; }
}
```

#### AnimalEvent
```csharp
public class AnimalEvent : AuditableEntity
{
    public EventType Type { get; set; }  // Nacimiento, Muerte, Compra, Venta, etc.
    public DateTime EventDate { get; set; }
    public string? Notes { get; set; }
    public string? DiseaseType { get; set; }  // Para eventos de enfermedad
    public string? DiseaseCategory { get; set; }
    public int AnimalId { get; set; }
    public Animal Animal { get; set; }
}
```

#### Service
```csharp
public class Service : AuditableEntity
{
    public ServiceType Type { get; set; }  // Vacunacion, Medicacion, etc.
    public DateTime ServiceDate { get; set; }
    public string? Description { get; set; }
    public int FarmId { get; set; }
    public Farm Farm { get; set; }
    public int? WorkerId { get; set; }
    public Worker? Worker { get; set; }
    public ICollection<ServiceAnimal> ServiceAnimals { get; set; }
    public ICollection<SupplyConsumption> SupplyConsumptions { get; set; }
}
```

#### Supply
```csharp
public class Supply : AuditableEntity
{
    public string Name { get; set; }
    public decimal CurrentQuantity { get; set; }
    public decimal MaxStockReference { get; set; }  // Para calcular alertas
    public UnitType Unit { get; set; }  // Unidad, Litro, Mililitro, Gramo
    public int FarmId { get; set; }
    public Farm Farm { get; set; }
}
```

### Enums

```csharp
public enum FarmType { Leche, Carne, Mixto }
public enum FarmTenancyType { Propia, NoPoseida }
public enum AnimalStatus { Activo, Vendido, Muerto, Perdido }
public enum EventType { Nacimiento, Muerte, Compra, Venta, Perdida, Encuentro, Parto, Enfermedad }
public enum ServiceType { Herrado, Vacunacion, Medicacion, Sangria, RevisionMedica, Pesaje, Otros }
public enum UnitType { Unidad, Litro, Mililitro, Gramo }
```

---

## 📏 Reglas de Negocio

### Productores y Fincas

- Un productor puede tener **múltiples fincas**
- Una finca pertenece a **un solo productor**
- Las fincas pueden ser **Propias** o **No Poseídas** (arrendadas/cedidas)
- Las fincas tienen **divisiones internas** (potreros/lotes)

### Animales

- Cada animal tiene un **identificador único** por productor (no global)
- Un animal siempre debe tener una **finca** y **división** actual
- Los animales pueden estar en estados: `Activo`, `Vendido`, `Muerto`, `Perdido`
- Los animales **no activos** se mantienen con **soft delete** (se preserva su historial)
- Los animales no activos **no aparecen** en listados estándar (solo con filtros)

### Movimientos

- Un movimiento transfiere un animal **entre fincas del mismo productor**
- Al mover un animal, se debe especificar la **nueva finca** y **nueva división**
- El movimiento actualiza automáticamente la ubicación actual del animal
- Los movimientos quedan registrados en el historial

### Eventos

- Los eventos alimentan el **timeline del animal**
- Eventos de enfermedad requieren **tipo** y **categoría**
- Todos los eventos quedan asociados al animal en orden cronológico

### Servicios

- Un servicio puede aplicarse a **uno o varios animales**
- Cuando se aplica a varios, se crea un **registro individual por animal**
- Los servicios pueden consumir **insumos** (opcional)
- Los servicios pueden tener un **trabajador responsable** (opcional)

### Insumos

- Los insumos pertenecen a una **finca específica**
- El stock se **descuenta automáticamente** al registrar consumos en servicios
- Se genera una **alerta** cuando el stock cae por debajo del **30% del máximo**
- Soporta **cantidades decimales** (ej: 2.75 frascos)

### Trabajadores

- Los trabajadores pertenecen a una **finca específica**
- Pueden ser asignados como **responsables de servicios**

### Multi-Tenancy

- Todas las operaciones se filtran automáticamente por `ProducerId`
- Un productor **nunca** puede acceder a datos de otro productor
- `AdminSistema` puede ver **todos** los datos (solo lectura)

### Auditoría

- Todas las entidades registran automáticamente:
  - Quién las creó (`CreatedBy`, `CreatedAt`)
  - Quién las modificó (`ModifiedBy`, `ModifiedAt`)
- La auditoría se implementa en el `SaveChangesAsync` del `DbContext`

---

## 🧪 Testing

### Ejecutar tests unitarios

```bash
dotnet test
```

### Ejecutar tests con cobertura

```bash
dotnet test /p:CollectCoverage=true
```

### Estructura de Tests

```
SITAGapi.Tests/
├── Domain.Tests/          # Tests de entidades y value objects
├── Application.Tests/     # Tests de servicios y casos de uso
├── Infrastructure.Tests/  # Tests de repositorios y DbContext
└── API.Tests/             # Tests de integración de endpoints
```

---

## 🚢 Deployment

### Railway (Recomendado)

1. Crear un proyecto en [Railway](https://railway.app/)
2. Agregar un servicio PostgreSQL
3. Agregar un servicio .NET
4. Configurar variables de entorno desde el panel de Railway
5. Conectar el repositorio de GitHub
6. Railway desplegará automáticamente en cada push a `main`

### Docker

```bash
# Build
docker build -t sitagapi:latest .

# Run
docker run -d -p 5000:80 \
  -e DATABASE_CONNECTION_STRING="..." \
  -e JWT_KEY="..." \
  sitagapi:latest
```

### Dockerfile (ejemplo)

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["SITAGapi.API/SITAGapi.API.csproj", "SITAGapi.API/"]
COPY ["SITAGapi.Application/SITAGapi.Application.csproj", "SITAGapi.Application/"]
COPY ["SITAGapi.Domain/SITAGapi.Domain.csproj", "SITAGapi.Domain/"]
COPY ["SITAGapi.Infrastructure/SITAGapi.Infrastructure.csproj", "SITAGapi.Infrastructure/"]
RUN dotnet restore "SITAGapi.API/SITAGapi.API.csproj"
COPY . .
WORKDIR "/src/SITAGapi.API"
RUN dotnet build "SITAGapi.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "SITAGapi.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "SITAGapi.API.dll"]
```

---

## 🛠️ Stack Tecnológico

| Componente                | Tecnología                                      |
|---------------------------|-------------------------------------------------|
| **Framework**             | .NET 8                                          |
| **Lenguaje**              | C# 12                                           |
| **Web API**               | ASP.NET Core Web API                            |
| **Base de Datos**         | PostgreSQL 14+                                  |
| **ORM**                   | Entity Framework Core 8                         |
| **Autenticación**         | JWT (JSON Web Tokens)                           |
| **Validación**            | FluentValidation                                |
| **Mapeo de Objetos**      | Mapster                                         |
| **Documentación API**     | Swagger / OpenAPI (Swashbuckle)                 |
| **Logging**               | Serilog                                         |
| **Testing**               | xUnit + Moq + FluentAssertions                  |
| **Hosting**               | Railway / Docker                                |
| **Control de Versiones**  | Git + GitHub                                    |

---

## 🤝 Contribución

### Flujo de Trabajo

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/nueva-funcionalidad`)
3. Haz commit de tus cambios (`git commit -m 'feat: agregar nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abre un Pull Request

### Convenciones de Código

- **Nomenclatura**: PascalCase para clases, métodos y propiedades
- **Idioma**: Código en inglés, comentarios en español
- **Commits**: Seguir [Conventional Commits](https://www.conventionalcommits.org/)
  - `feat:` Nueva funcionalidad
  - `fix:` Corrección de bug
  - `docs:` Documentación
  - `refactor:` Refactorización
  - `test:` Tests
  - `chore:` Tareas de mantenimiento

### Code Review

- Todo código debe pasar revisión antes de merge
- Asegurar que los tests pasen
- Verificar que el código siga los principios SOLID

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver el archivo [LICENSE](LICENSE) para más detalles.

---

## 📧 Contacto

**Grupo GESHK**  
- GitHub: [@Grupo-Geshk](https://github.com/Grupo-Geshk)
- Email: contacto@geshk.com

---

## 🗺️ Roadmap

### v1.0 - MVP (En Desarrollo)
- [x] Autenticación JWT
- [x] CRUD de Productores y Fincas
- [x] Gestión de Animales
- [x] Movimientos y Eventos
- [x] Servicios e Insumos
- [x] Trabajadores
- [x] Reportes básicos

### v1.1 - Mejoras
- [ ] Notificaciones por email/SMS
- [ ] Exportación de reportes (PDF, Excel)
- [ ] Gráficos y dashboards avanzados
- [ ] Gestión de Hatos como entidad independiente

### v2.0 - Integraciones
- [ ] Integración con subastas ganaderas
- [ ] Integración con mataderos
- [ ] Certificaciones sanitarias digitales
- [ ] API pública para terceros

### v3.0 - Mobile
- [ ] App móvil nativa (Flutter/React Native)
- [ ] Modo offline
- [ ] Escaneo de aretes con cámara

---

## 📚 Recursos Adicionales

- [Documentación de .NET 8](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-8)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Web API](https://learn.microsoft.com/en-us/aspnet/core/web-api/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)

---

**¡Gracias por contribuir a SITAGapi! 🚜🐄**
