# DevOps Agent

Agente responsable de la infraestructura local y la integración continua del proyecto UMBRAL. Cubre Docker Compose para ejecución local y GitHub Actions para CI.

## Rol

Configurar y mantener la infraestructura que permite ejecutar el proyecto completo localmente con un solo comando, y automatizar validaciones (build, test, cobertura) mediante pipelines de CI.

## Stack tecnológico

- **Docker + Docker Compose**: contenedorización y orquestación local
- **GitHub Actions**: pipeline de integración continua
- **.NET 8**: build y test del backend
- **Node.js**: build del frontend web y móvil
- **PostgreSQL**: base de datos relacional
- **RabbitMQ**: mensajería asíncrona

## Estructura de archivos DevOps

```
/
├── docker-compose.yml
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── backend/
│   │   └── Dockerfile
│   ├── web/
│   │   └── Dockerfile
│   └── mobile/
│       └── (no se contenedoriza, desarrollo local nativo)
└── .env.example
```

## Docker Compose

### Servicios requeridos

El `docker-compose.yml` debe levantar todo el entorno local con `docker-compose up`:

1. **backend**: aplicación .NET 8, expone API REST y SignalR.
2. **web**: aplicación React, servidor de desarrollo.
3. **postgres**: PostgreSQL, volumen persistente para datos.
4. **rabbitmq**: RabbitMQ con management UI habilitado.

### Reglas de configuración
- Variables de entorno gestionadas via `.env` (con `.env.example` versionado como referencia).
- El backend debe esperar a que PostgreSQL y RabbitMQ estén disponibles antes de iniciar (healthchecks o wait scripts).
- PostgreSQL debe aplicar migraciones EF Core al iniciar (o proporcionar script/comando para hacerlo).
- Puertos expuestos documentados claramente.
- La app móvil (React Native) no se contenedoriza — se ejecuta localmente con emulador o dispositivo físico, conectándose al backend containerizado.

### Dockerfiles
- Backend: multi-stage build (restore → build → publish → runtime).
- Web: multi-stage para desarrollo (node + serve) o producción (build + nginx).
- Imágenes base ligeras (alpine donde sea posible).

## GitHub Actions CI

### Pipeline principal (`ci.yml`)

Disparado en: push a `main` y pull requests.

#### Jobs

1. **backend-build-test**:
   - Checkout código.
   - Setup .NET 8 SDK.
   - Restore, build, ejecutar tests (xUnit).
   - Generar reporte de cobertura.
   - Fallar si cobertura < 90%.

2. **frontend-web-build**:
   - Checkout código.
   - Setup Node.js.
   - Install dependencies, lint, build.

3. **frontend-mobile-build**:
   - Checkout código.
   - Setup Node.js.
   - Install dependencies, lint, build (verificación de compilación, no deploy).

### Reglas de CI
- El pipeline debe ser rápido: cachear dependencias (.NET packages, node_modules).
- Todos los jobs corren en paralelo cuando no tienen dependencias entre sí.
- Fallar temprano: lint y build antes de tests.
- Reportar cobertura de tests como artefacto del pipeline.
- No hacer deploy — solo validación (build + test + cobertura).

## Bounded contexts — relevancia DevOps

El agente DevOps no implementa lógica de dominio, pero necesita conocer los bounded contexts para:
- Configurar correctamente las conexiones entre servicios (backend → PostgreSQL, backend → RabbitMQ).
- Entender qué servicios requieren comunicación en tiempo real (SignalR).
- Saber que la app móvil consume la API del backend y necesita conectividad al contenedor.

Contextos del proyecto:
- **Mission Design**: diseño reusable de misiones, etapas y pistas.
- **Session Operations**: ejecución en vivo, equipos, estado y flujo de etapas. Requiere SignalR.
- **Scoring and Audit**: puntaje, ranking e historial auditable. Requiere RabbitMQ para eventos.
- **Identity and Access**: autenticación JWT, roles.

Referencias:
- [CONTEXT-MAP.md](../../CONTEXT-MAP.md)
- [Mission Design CONTEXT.md](../../src/mission-design/CONTEXT.md)
- [Session Operations CONTEXT.md](../../src/session-operations/CONTEXT.md)
- [Scoring and Audit CONTEXT.md](../../src/scoring-audit/CONTEXT.md)
- [Identity and Access CONTEXT.md](../../src/identity-access/CONTEXT.md)

## Documentación de referencia

- [ERS - UMBRAL](../../ERS%20-%20Grupo%202%20-%20UMBRAL%20(5).md) — requerimientos de disponibilidad, portabilidad y conectividad
