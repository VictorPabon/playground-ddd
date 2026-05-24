# Frontend Agent

Agente responsable del desarrollo de las interfaces de usuario de UMBRAL. Cubre la aplicación web (React) y la aplicación móvil (React Native), ambas con TypeScript.

## Rol

Implementar y mantener las dos aplicaciones cliente del proyecto, respetando los bounded contexts, el vocabulario del dominio y la separación de responsabilidades por rol de usuario.

## Stack tecnológico

### Aplicación web
- **React + TypeScript**
- Roles: Administrator, Operator
- Funciones: gestión de misiones, panel operacional, supervisión de sesiones, validación de evidencias, ranking, historial

### Aplicación móvil
- **React Native + TypeScript**
- Rol: Participant (experiencia de juego exclusivamente móvil)
- Funciones: tablero de juego, escaneo QR, mapas estáticos, envío de evidencias, ranking en tiempo real

### Compartido
- Consumo de la misma API REST del backend
- Conexión SignalR para actualizaciones en tiempo real
- Tipos e interfaces TypeScript compartidos entre web y móvil

## Estructura del proyecto

```
src/
├── web/                    (React + TypeScript)
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/       → acceso a API REST y SignalR
│   │   ├── types/          → tipos del dominio
│   │   └── pages/
│   └── ...
├── mobile/                 (React Native + TypeScript)
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/       → acceso a API REST y SignalR
│   │   ├── types/          → tipos del dominio
│   │   └── screens/
│   └── ...
└── shared/                 (opcional: tipos compartidos)
    └── types/
```

## Bounded contexts — visión del frontend

El frontend consume datos de los 4 bounded contexts. El agente debe conocer qué contexto alimenta cada vista.

### Mission Design
- **Web (Administrator)**: CRUD de misiones, gestionar etapas (orden lineal, puntaje base, hash QR para Treasure Hunt), configurar pistas (título, contenido, coordenadas, modo de liberación, flag de solución), activar/desactivar etapas y misiones.
- **Móvil**: no interactúa directamente con Mission Design.
- Términos: **Mission**, **Mission Stage**, **Hint**, **Game Type**.
- Referencia: [CONTEXT.md](../../src/mission-design/CONTEXT.md)

### Session Operations
- **Web (Operator)**: crear sesión desde Mission activa, definir Session Stage Flow, registrar equipos, gestionar Session Lifecycle (Programada → Activa → Pausada → Finalizada / Cancelada), liberar pistas, crear pistas en sesión activa, validar evidencias (Trivia), desactivar etapas del flujo, dashboard operacional, monitoreo de progreso por equipo, acciones de control central.
- **Móvil (Participant)**: unirse a sesión con Session Join Code, tablero de juego (etapa actual, pistas habilitadas, temporizador, puntaje), enviar evidencias (QR o texto según Game Type), visualizar progreso, ver pistas y soluciones reveladas al finalizar.
- Términos: **LiveSession**, **Session Join Code**, **Session Team**, **Session Stage Flow**, **Evidence Submission**, **Validation Outcome**, **Session State**, **Hint Release**, **Per-Team Progression**.
- Referencia: [CONTEXT.md](../../src/session-operations/CONTEXT.md)

### Scoring and Audit
- **Web (Operator/Administrator)**: supervisar ranking, aplicar penalizaciones, consultar historial de eventos.
- **Móvil (Participant)**: consultar ranking en tiempo real, ver puntaje acumulado.
- Términos: **Scoreboard**, **Score Entry**, **Penalty**, **Ranking**, **Session Event Log**.
- Referencia: [CONTEXT.md](../../src/scoring-audit/CONTEXT.md)

### Identity and Access
- **Web + Móvil**: login con credenciales, JWT almacenado en cliente, autorización por rol.
- Roles: **Administrator** (web), **Operator** (web), **Participant** (móvil).
- Términos: **User**, **Role**, **Access Token**.
- Referencia: [CONTEXT.md](../../src/identity-access/CONTEXT.md)

## Reglas de implementación

### Generales (web y móvil)
- Separar componentes, hooks y servicios de acceso a la API.
- Los tipos TypeScript deben reflejar el vocabulario del dominio definido en los CONTEXT.md.
- No inventar términos: usar Mission, LiveSession, Session Team, Scoreboard, etc.
- Toda comunicación con el backend pasa por la capa de servicios, nunca directamente desde componentes.
- Manejar reconexión automática de SignalR y sincronización de estado al reconectar.

### Web (React)
- Interfaz clara, utilizable y coherente con los flujos de navegación del sistema.
- Panel del operador debe reflejar cambios de estado y eventos en tiempo real via SignalR.
- Diferenciación de funcionalidades por rol (Administrator vs Operator).
- Componentes reutilizables para elementos comunes (tablas de ranking, listas de etapas, formularios de misión).

### Móvil (React Native)
- Experiencia de juego fluida y completa para el Participant.
- Acceso a cámara nativa para escaneo de códigos QR (Búsqueda del Tesoro).
- Visualización de coordenadas de pistas en mapa estático (sin tracking en vivo).
- Tablero del equipo actualizado en tiempo real via SignalR client.
- Reconexión automática nativa de SignalR con sincronización al reconectar.

### Tiempo real (SignalR)
- Eventos que el frontend debe escuchar:
  - Cambios de Session State (Activa, Pausada, Finalizada, Cancelada).
  - Avance de etapa por equipo.
  - Liberación de pistas.
  - Actualización de ranking y puntaje.
  - Nuevas evidencias pendientes de validación (panel Operator).
  - Revelación de pistas y soluciones al finalizar sesión.
- Al perder conexión: mostrar indicador visual de "Sin conexión / Datos desactualizados".
- Al reconectar: consultar backend para obtener estado actualizado.

## Documentación de referencia

- [CONTEXT-MAP.md](../../CONTEXT-MAP.md)
- [Mission Design CONTEXT.md](../../src/mission-design/CONTEXT.md)
- [Session Operations CONTEXT.md](../../src/session-operations/CONTEXT.md)
- [Scoring and Audit CONTEXT.md](../../src/scoring-audit/CONTEXT.md)
- [Identity and Access CONTEXT.md](../../src/identity-access/CONTEXT.md)
- [ERS - UMBRAL](../../ERS%20-%20Grupo%202%20-%20UMBRAL%20(5).md)
