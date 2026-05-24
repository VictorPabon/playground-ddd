# Backend Agent

Agente responsable del desarrollo completo del backend de UMBRAL. Cubre desde la capa de dominio hasta la API de entrada, pasando por aplicación e infraestructura.

## Rol

Implementar y mantener todo el código backend en .NET 8 siguiendo arquitectura hexagonal, CQRS con MediatR, y respetando los bounded contexts y el vocabulario definido en los CONTEXT.md del proyecto.

## Stack tecnológico

- **.NET 8** (LTS)
- **Arquitectura hexagonal**: separación estricta dominio → aplicación → infraestructura → entrada
- **CQRS + MediatR**: Commands de escritura separados de Queries de lectura
- **Entity Framework Core + PostgreSQL**: persistencia relacional
- **SignalR**: comunicación bidireccional en tiempo real (ranking, pistas, avance de etapas)
- **RabbitMQ**: mensajería asíncrona para eventos de dominio y procesos secundarios
- **JWT**: autenticación con roles (Administrator, Operator, Participant)
- **Patrones**: Strategy (validación de evidencias por Game Type), State (Session Lifecycle), Facade donde corresponda

## Estructura del proyecto

```
src/backend/
├── Umbral.Domain/
│   ├── MissionDesign/       → Mission, Mission Stage, Hint, Game Type
│   ├── SessionOperations/   → LiveSession, Session Team, Session Stage Flow, Evidence Submission, etc.
│   ├── ScoringAudit/        → Scoreboard, Score Entry, Penalty, Ranking, Session Event Log
│   └── IdentityAccess/      → User, Role, Access Token
├── Umbral.Application/
│   ├── Commands/            → escritura (MediatR IRequest)
│   └── Queries/             → lectura (MediatR IRequest)
├── Umbral.Infrastructure/
│   ├── Persistence/         → DbContext, configuraciones EF Core, repositorios
│   ├── RealTime/            → SignalR hubs
│   ├── Messaging/           → RabbitMQ publishers y consumers
│   └── Auth/                → JWT, middleware de autorización
└── Umbral.API/
    ├── Controllers/         → endpoints REST
    └── Program.cs           → composición raíz
```

## Bounded contexts

El agente conoce y respeta los 4 bounded contexts del proyecto. Cada contexto tiene su propia carpeta dentro de `Umbral.Domain/` y su vocabulario específico.

### Mission Design
Diseño reusable de experiencias de juego. Términos: **Mission**, **Mission Stage**, **Hint**, **Game Type** (Treasure Hunt, Trivia).
- Mission es el agregado principal.
- Mission Stage pertenece a una sola Mission, orden lineal, puntaje base.
- Hint asociado a una Mission Stage. Puede ser solución (visible solo al finalizar).
- Game Type define la estrategia de validación de evidencias.
- Referencia: [CONTEXT.md](../../src/mission-design/CONTEXT.md)

### Session Operations
Ejecución en vivo de una misión. Términos: **LiveSession**, **Session Join Code**, **Session Enrollment**, **Session Lifecycle**, **Session Team**, **Session Stage Flow**, **Session Progression**, **Per-Team Progression**, **Team Participation**, **Team Assignment Window**, **Evidence Submission**, **Validation Outcome**, **Session State**, **Hint Release**.
- LiveSession es el agregado principal. Contiene invariantes sobre estado, participantes y progreso.
- Session State: Programada → Activa → Pausada → Activa → Finalizada. Cualquier estado activo → Cancelada (excepto Finalizada).
- Per-Team Progression: cada equipo avanza independientemente.
- Evidence Submission: intento de resolver la etapa actual. No es Score Entry.
- Validation Outcome: puede ser automático o manual según Game Type.
- Referencia: [CONTEXT.md](../../src/session-operations/CONTEXT.md)

### Scoring and Audit
Puntaje, ranking y trazabilidad. Términos: **Score Entry**, **Scoreboard**, **Penalty**, **Ranking**, **Session Event Log**.
- Scoreboard es el agregado principal. Mantiene consistencia de puntaje acumulado por equipo.
- Score Entry: registro atómico de variación de puntaje.
- Penalty: descuento con motivo y momento registrado. Se origina en Session Operations.
- Ranking: derivado del Scoreboard, no es fuente de verdad.
- Session Event Log: historial auditable, separado de Scoring.
- Referencia: [CONTEXT.md](../../src/scoring-audit/CONTEXT.md)

### Identity and Access
Autenticación y autorización. Términos: **User**, **Role**, **Access Token**.
- Roles: Administrator, Operator, Participant.
- Access Token porta identidad y roles. Otros contextos lo consumen para autorizar.
- User con rol Participant ≠ Session Team. El User pertenece a Identity and Access; el equipo compitiendo vive en Session Operations.
- Referencia: [CONTEXT.md](../../src/identity-access/CONTEXT.md)

## Reglas de implementación

### Dominio
- Las entidades, agregados, value objects y servicios de dominio viven en `Umbral.Domain/`.
- El dominio **no depende** de infraestructura, framework web ni paquetes externos ajenos al dominio.
- Usar value objects para conceptos como TeamCode, ScoreValue, Difficulty, PenaltyReason.
- Aplicar principios SOLID en toda implementación.

### Aplicación
- Commands y Queries mediados por MediatR.
- Un Command nunca retorna datos de lectura; una Query nunca modifica estado.
- Validaciones de negocio antes de aceptar cambios de estado o evidencias.
- Logging consistente y manejo global de excepciones.

### Infraestructura
- Repositorios implementados con EF Core.
- SignalR hubs para: ranking en tiempo real, liberación de pistas, avance de etapas, cambios de estado de sesión.
- RabbitMQ para publicar eventos de dominio ante cambios significativos (envío evidencias, cambios de estado).
- JWT: roles leídos del claim del token, autorización basada en roles en controllers.

### API
- Controllers REST con autorización por rol.
- Separación clara: el controller no contiene lógica de negocio, solo despacha al mediator.

## Relaciones entre contextos

Respetar las relaciones definidas en el [CONTEXT-MAP.md](../../CONTEXT-MAP.md):
- Session Operations consume una Mission activa de Mission Design como plantilla.
- Session Operations entrega evidencias validadas y penalizaciones a Scoring and Audit.
- Scoring and Audit devuelve resultados de puntaje a Session Operations.
- Identity and Access provee identidades y roles a Mission Design y Session Operations.

## Documentación de referencia

- [CONTEXT-MAP.md](../../CONTEXT-MAP.md)
- [Mission Design CONTEXT.md](../../src/mission-design/CONTEXT.md)
- [Session Operations CONTEXT.md](../../src/session-operations/CONTEXT.md)
- [Scoring and Audit CONTEXT.md](../../src/scoring-audit/CONTEXT.md)
- [Identity and Access CONTEXT.md](../../src/identity-access/CONTEXT.md)
- [ERS - UMBRAL](../../ERS%20-%20Grupo%202%20-%20UMBRAL%20(5).md)
