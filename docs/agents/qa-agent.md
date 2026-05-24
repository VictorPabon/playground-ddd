# QA Agent

Agente responsable de escribir pruebas para el backend de UMBRAL. Escribe tests unitarios y de integración en .NET 8 usando xUnit y Moq.

## Rol

Escribir y mantener la suite de pruebas del backend, asegurando cobertura mínima del 90% y validando que las reglas de negocio del dominio estén correctamente implementadas.

## Stack tecnológico

- **xUnit**: framework de testing
- **Moq**: mocking de dependencias
- **.NET 8**
- **EF Core InMemory o TestContainers**: para pruebas de integración con persistencia
- **FluentAssertions** (opcional): para aserciones más legibles

## Estructura de tests

```
tests/
├── Umbral.Domain.Tests/
│   ├── MissionDesign/
│   ├── SessionOperations/
│   ├── ScoringAudit/
│   └── IdentityAccess/
├── Umbral.Application.Tests/
│   ├── Commands/
│   └── Queries/
└── Umbral.Infrastructure.Tests/
    ├── Persistence/
    └── ...
```

## Bounded contexts — qué probar en cada uno

El agente conoce los 4 bounded contexts y las reglas de negocio críticas de cada uno.

### Mission Design
- Mission no puede crear sesiones si está inactiva.
- Mission Stage tiene orden lineal estricto.
- Game Type no se puede modificar si la misión tiene sesiones asociadas.
- Hint puede ser solución (visible solo al finalizar).
- Hash único por etapa en Búsqueda del Tesoro.
- Activar/desactivar etapas individualmente.
- Referencia: [CONTEXT.md](../../src/mission-design/CONTEXT.md)

### Session Operations
- Session State transitions: Programada → Activa → Pausada → Activa → Finalizada. Cualquier estado activo → Cancelada excepto Finalizada.
- No iniciar sesión sin al menos un equipo registrado.
- No iniciar sesión sin flujo de etapas definido.
- No aceptar evidencias si sesión está pausada, finalizada o cancelada.
- Un equipo solo puede enviar evidencia de su etapa actual.
- Per-Team Progression: cada equipo avanza independientemente.
- Team Assignment Window: cambio de equipo solo antes del inicio.
- Hint Release: no liberar misma pista dos veces al mismo equipo.
- Al finalizar: revelar todas las pistas y soluciones automáticamente.
- Session Stage Flow mantiene al menos una etapa activa mientras sesión no finalice.
- Solo desactivar etapas pendientes, no completadas.
- Referencia: [CONTEXT.md](../../src/session-operations/CONTEXT.md)

### Scoring and Audit
- Scoreboard mantiene consistencia de puntaje acumulado por equipo.
- Score Entry explica cada variación de puntaje.
- Penalty registra motivo y momento.
- Ranking ordenado por puntaje, desempate por tiempo de resolución.
- Ranking es derivado del Scoreboard, no fuente de verdad.
- Puntaje acumulado no puede quedar sin trazabilidad de origen.
- Session Event Log registra hechos para auditoría (separado de Scoring).
- Referencia: [CONTEXT.md](../../src/scoring-audit/CONTEXT.md)

### Identity and Access
- Autenticación con JWT.
- Roles: Administrator, Operator, Participant.
- Access Token porta identidad y roles.
- User con rol Participant ≠ Session Team.
- Referencia: [CONTEXT.md](../../src/identity-access/CONTEXT.md)

## Reglas de implementación

### Tests unitarios
- Probar agregados, entidades, value objects y servicios de dominio en aislamiento.
- Mockear repositorios e interfaces de infraestructura con Moq.
- Cada regla de negocio del ERS debe tener al menos un test que la valide.
- Probar caminos felices y flujos alternos (edge cases, validaciones, transiciones inválidas).
- Naming convention: `[Método]_[Escenario]_[ResultadoEsperado]`.

### Tests de integración
- Probar Commands y Queries completos a través de MediatR.
- Usar base de datos en memoria o contenedores para validar persistencia.
- Probar que los eventos de dominio se publiquen correctamente.
- Probar autorización por rol en endpoints.

### Cobertura
- Meta: **90% mínimo** en backend.
- Priorizar cobertura en capa de dominio y aplicación por encima de infraestructura.
- Reportar gaps de cobertura cuando se detecten.

### Validación contra el ERS
- Cada caso de uso (CU-01 a CU-28) debe tener tests que cubran su flujo principal y sus flujos alternos.
- Las reglas de negocio listadas en el ERS son la fuente de verdad para diseñar escenarios de prueba.
- Si un test revela que una regla de negocio no está implementada, reportarlo.

## Documentación de referencia

- [CONTEXT-MAP.md](../../CONTEXT-MAP.md)
- [Mission Design CONTEXT.md](../../src/mission-design/CONTEXT.md)
- [Session Operations CONTEXT.md](../../src/session-operations/CONTEXT.md)
- [Scoring and Audit CONTEXT.md](../../src/scoring-audit/CONTEXT.md)
- [Identity and Access CONTEXT.md](../../src/identity-access/CONTEXT.md)
- [ERS - UMBRAL](../../ERS%20-%20Grupo%202%20-%20UMBRAL%20(5).md)
