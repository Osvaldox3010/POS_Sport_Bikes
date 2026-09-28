# 🚲 BikePOS

Sistema web de punto de venta (POS) para una tienda de bicicletas: una sola caja, uso real en tienda, con modo sin conexión en el mostrador. Proyecto de portafolio Full Stack (frontend, backend, base de datos, seguridad, arquitectura, testing, despliegue y documentación).

> Este README es un **esqueleto** (Fase 0). Se irá completando conforme avancen las fases descritas en `docs/decisions.md` y en el plan de desarrollo del proyecto.

---

## Estado del proyecto

🚧 **En construcción — Fase 2: Esqueleto, base offline y prueba de impresión.**

- [x] Fase 0 — Fundación (repo, `.gitignore`, `.env.example`, README, `docs/decisions.md`)
- [x] Fase 1 — Planeación y diseño offline (casos de uso, reglas de negocio, diagrama ER, borrador de API, wireframes, diseño offline — todo aprobado)
- [ ] Fase 2 — Esqueleto, base offline y prueba de impresión *(pendiente tu visita a la tienda: H1–H3)*
- [ ] Fase 3 — Autenticación, usuarios y auditoría (base) — primer despliegue público
- [ ] Fase 4 — Categorías, marcas y productos
- [ ] Fase 5 — Búsqueda e inventario
- [ ] Fase 6 — POS núcleo (local-first)
- [ ] Fase 7 — Impresión, ticket e historial
- [ ] Fase 8 — Cancelaciones, devoluciones y reembolsos
- [ ] Fase 9 — Apartados
- [ ] Fase 10 — Caja y corte
- [ ] Fase 11 — Precio personalizado
- [ ] Fase 12 — Sincronización completa
- [ ] Fase 13 — Configuración y formato de ticket
- [ ] Fase 14 — Auditoría (pantallas) y dashboard
- [ ] Fase 15 — Reporte PDF y respaldo
- [ ] Fase 16 — Endurecimiento
- [ ] Fase 17 — Despliegue definitivo y CI/CD
- [ ] Fase 18 — Documentación y portafolio

---

## Descripción

BikePOS administra el punto de venta de una tienda de bicicletas: catálogo, inventario, ventas, apartados, caja/corte, precios personalizados con autorización, auditoría y un dashboard para el administrador. Es una aplicación **web privada** (requiere usuario y contraseña), pensada para **una sola tienda y una sola caja**, con capacidad de **vender sin internet** desde la PC del mostrador.

Fuera de alcance (por decisión de producto): tienda en línea, facturación fiscal, múltiples sucursales, registro de clientes, crédito de tienda, WhatsApp, app nativa e inteligencia artificial.

---

## Arquitectura

```text
PC del mostrador (navegador)               Admin: PC / tablet / celular
      │  copia local: catálogo, ventas, cola de sincronización
      │  HTTPS
      ▼
┌─────────────────────────────────┐
│ FRONTEND (PWA)                  │  React + TypeScript + Vite
│ Cloudflare Pages                │  Solo UI y datos locales. Nunca toca la BD.
└───────────────┬─────────────────┘
                │  REST / JSON + JWT
                ▼
┌─────────────────────────────────┐        ┌──────────────────────────┐
│ BACKEND                         │───────▶│ CORREO (y SMS solo si    │
│ Spring Boot, monolito modular   │        │ hay opción gratuita)     │
│ Render (Docker)                 │        │                          │
└───────────────┬─────────────────┘        └──────────────────────────┘
                │  JDBC / JPA
                ▼
┌─────────────────────────────────┐
│ PostgreSQL — Neon               │  Solo el backend accede.
└─────────────────────────────────┘
```

Las decisiones de arquitectura (monorepo, monolito modular, local-first, idempotencia por operación) están registradas y justificadas en [`docs/decisions.md`](./docs/decisions.md).

---

## Stack tecnológico

| Categoría | Tecnología |
|---|---|
| Frontend | React · TypeScript · Vite · React Router · TanStack Query · MUI · Day.js · Recharts/ECharts · Axios |
| PWA / sin conexión | Service worker · IndexedDB (Dexie.js) |
| Backend | Java · Spring Boot · Spring Web · Spring Data JPA · Hibernate · Spring Security · JWT · Bean Validation · Maven |
| Base de datos | PostgreSQL (Neon), extensiones `unaccent` y `pg_trgm` |
| Documentación de API | Swagger / OpenAPI |
| Testing backend | JUnit · Mockito · Spring Boot Test · Testcontainers |
| Testing frontend | Vitest · React Testing Library · E2E sin red |
| DevOps | GitHub Actions · Cloudflare Pages · Render · Neon · Docker |

---

## Estructura del monorepo

```text
bikepos/
├── frontend/          # React + TypeScript + Vite (PWA, POS, dashboard)
├── backend/           # Spring Boot — monolito modular
├── docs/              # Documentación técnica y decisiones
│   ├── decisions.md
│   ├── casos-de-uso.md
│   ├── reglas-corte-apartados-devoluciones.md
│   ├── diagrama-er.md
│   ├── api-draft.md
│   ├── wireframes.md
│   └── diseno-offline.md
├── docker-compose.yml # (Fase 2) frontend + backend + PostgreSQL local
├── .env.example
├── .gitignore
└── README.md
```

---

## Instalación y ejecución local

> Se completará en la Fase 2, cuando exista `docker-compose.yml` y el esqueleto de frontend/backend.

```bash
# Clonar el repositorio
git clone <url-del-repositorio>
cd bikepos

# Copiar variables de entorno
cp .env.example .env

# (Fase 2 en adelante)
docker compose up
```

---

## Variables de entorno

Ver [`.env.example`](./.env.example) para la lista completa (base de datos, JWT, CORS, súper admin inicial, correo, `VITE_API_URL`).

---

## API

Borrador completo de endpoints por módulo en [`docs/api-draft.md`](./docs/api-draft.md). La documentación formal (Swagger/OpenAPI) se agregará a partir de la Fase 3, cuando exista el primer despliegue público.

## Base de datos

Modelo entidad-relación preliminar, aprobado en la Fase 1, en [`docs/diagrama-er.md`](./docs/diagrama-er.md). Las migraciones (Flyway) se documentarán en `docs/database.md` a partir de la Fase 4.

## Testing

La estrategia de pruebas (backend y frontend) se documentará en `docs/decisions.md` y se ampliará conforme cada módulo se construya, según la prioridad por riesgo definida en el plan.

## Despliegue

- **Frontend:** Cloudflare Pages (subdominio gratuito).
- **Backend:** Render (Docker).
- **Base de datos:** Neon (PostgreSQL).

Detalles y límites gratuitos vigentes se documentarán en `docs/decisions.md` al llegar a la Fase 3 y se revisan de nuevo en la Fase 17.

---

## Convención de commits

Este proyecto sigue una convención tipo *Conventional Commits*, con mensajes descriptivos:

```text
feat: add product management
feat: implement JWT authentication
feat: add inventory movements
fix: prevent sale with insufficient stock
refactor: separate sale business logic
test: add sale service tests
docs: update API documentation
chore: configure CI pipeline
```

Prefijos usados: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `style`.

---

## Ramas

- `main` — código estable, desplegable.
- `develop` — integración de trabajo en curso.
- `feature/<módulo>` — una rama por módulo (ej. `feature/auth`, `feature/pos`, `feature/inventario`).

---

## Documentación adicional

- [`docs/decisions.md`](./docs/decisions.md) — decisiones de arquitectura (Fase 0): monorepo, monolito modular, local-first, idempotencia por operación.
- [`docs/casos-de-uso.md`](./docs/casos-de-uso.md) — 38 casos de uso, con flujo detallado de los de mayor riesgo.
- [`docs/reglas-corte-apartados-devoluciones.md`](./docs/reglas-corte-apartados-devoluciones.md) — reglas finales de corte de caja, apartados y devoluciones/cancelaciones/reembolsos.
- [`docs/diagrama-er.md`](./docs/diagrama-er.md) — modelo entidad-relación (Mermaid).
- [`docs/api-draft.md`](./docs/api-draft.md) — borrador de la API REST por módulo.
- [`docs/wireframes.md`](./docs/wireframes.md) — wireframes de baja fidelidad de las pantallas clave.
- [`docs/diseno-offline.md`](./docs/diseno-offline.md) — diseño del modo sin conexión (copia local, cola de sincronización, folio local).
- Próximamente: `docs/architecture.md`, `docs/database.md`, `docs/security.md`.

---

## Autor

Proyecto de portafolio desarrollado por Osvaldo como demostración de desarrollo Full Stack (React + TypeScript, Java + Spring Boot, PostgreSQL).
