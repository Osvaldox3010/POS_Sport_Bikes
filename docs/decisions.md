# Decisiones de arquitectura — BikePOS

> Registro de decisiones (estilo ADR ligero). Cada entrada documenta el contexto, la decisión tomada, sus consecuencias y las alternativas consideradas. Se agregan entradas nuevas conforme el proyecto avanza; **ninguna entrada existente se edita o se borra** una vez aceptada — si una decisión cambia, se agrega una entrada nueva que reemplaza a la anterior y se referencia.

Estado de cada decisión: `Aceptada` · `Propuesta` · `Reemplazada`.

---

## ADR 001 — Monorepo (`/frontend`, `/backend`, `/docs`)

**Estado:** Aceptada · Fase 0

**Contexto**
BikePOS tiene dos aplicaciones (frontend React/Vite y backend Spring Boot) más documentación, desarrolladas por una sola persona, con despliegue en dos servicios distintos (Cloudflare Pages y Render) pero con un ciclo de vida y una hoja de ruta compartidos (las fases del plan tocan frontend y backend a la vez).

**Decisión**
Un único repositorio con la estructura:

```text
POSBikeStoreSportBikes/
├── frontend/
├── backend/
├── docs/
├── docker-compose.yml
├── .env.example
└── README.md
```

**Consecuencias**
- Un solo `git clone` levanta todo el proyecto; útil para portafolio y para revisión externa.
- Un solo lugar para issues, PRs y convención de commits.
- `docker-compose.yml` en la raíz puede orquestar frontend + backend + PostgreSQL en un solo comando (Fase 2).
- Los pipelines de CI/CD (Fase 17) deben filtrar por carpeta (`frontend/` vs `backend/`) para no reconstruir ambos servicios en cada cambio.
- El despliegue sigue siendo independiente por servicio (Cloudflare Pages solo lee `frontend/`, Render solo lee `backend/`), el monorepo no acopla el despliegue, solo el código fuente.

**Alternativas consideradas**
- *Multi-repo* (`bikepos-frontend`, `bikepos-backend`): descartado por ser un proyecto de una sola persona y de alcance acotado; habría duplicado configuración (CI, convención de commits, documentación) sin beneficio real a este tamaño.

---

## ADR 002 — Monolito modular (no microservicios)

**Estado:** Aceptada · Fase 0

**Contexto**
El backend debe cubrir autenticación, catálogo, inventario, ventas, pagos, devoluciones, apartados, caja/corte, autorización de precios, reportes, configuración, auditoría y sincronización. El sistema es para **una sola tienda y una sola caja**, con 3–4 usuarios, y debe desplegarse gratis en un solo servicio (Render).

**Decisión**
Un único servicio Spring Boot ("monolito"), organizado internamente por **módulos** con límites claros: `auth`, `users`, `catalog`, `inventory`, `sales`, `payments`, `returns`, `layaway` (apartados), `cash` (caja y corte), `price-approval`, `reports`, `settings`, `audit`, `sync`.

**Consecuencias**
- Una sola transacción de base de datos puede abarcar varios módulos (ej. confirmar venta: valida caja, descuenta stock, crea pago, genera folio) sin necesidad de transacciones distribuidas.
- Un solo despliegue en Render; encaja con el límite de "un solo servicio backend gratuito" (sección 11 del plan).
- La separación por módulos (paquetes/carpetas, no servicios) permite mantener responsabilidades claras y, si en el futuro el proyecto creciera, extraer un módulo a un servicio propio sin rediseñar todo.
- Exige disciplina para no dejar que los módulos se acoplen entre sí de forma desordenada (evitar dependencias circulares entre paquetes).

**Alternativas consideradas**
- *Microservicios*: descartados. Añadirían complejidad operativa (orquestación, comunicación entre servicios, transacciones distribuidas) y costo (cada servicio adicional consume cuota gratuita) sin beneficio para una sola tienda con una sola caja.

---

## ADR 003 — Local-first en el mostrador, desde la Fase 1

**Estado:** Aceptada · Fase 0

**Contexto**
La tienda necesita poder **vender sin internet** desde la PC del mostrador y sincronizar al reconectar (sección 1 y 5 del plan). Si el modo sin conexión se agregara al final del proyecto, probablemente exigiría rediseñar el flujo de venta, el manejo de folios y la capa de datos del frontend ya construidos.

**Decisión**
El frontend de la PC del mostrador se diseña **desde la Fase 1** (planeación) y se implementa desde la Fase 2 (esqueleto) como una aplicación **local-first**: escribe primero en una base local en el navegador (IndexedDB / Dexie.js) — catálogo con precios de venta y stock, tickets recientes, apartados pendientes y una cola de operaciones sin sincronizar — y sincroniza en segundo plano con el backend, que sigue siendo la única fuente de verdad.

**Consecuencias**
- El POS (Fase 6) se construye directamente sobre la capa local-first, en vez de agregarse "modo offline" como una capa posterior.
- El backend siempre revalida stock, precios y permisos al sincronizar, aunque el frontend ya haya validado localmente (el backend es la fuente de verdad, no el navegador).
- El precio de compra y el porcentaje de ganancia **nunca** se replican a la copia local (riesgo de seguridad si se pierde/roba la PC del mostrador).
- Introduce complejidad adicional desde el inicio (cola de sincronización, resolución de conflictos, folios locales) que se paga en la Fase 2 y no se puede posponer.
- Solo la PC del mostrador opera en local-first; el resto de pantallas (admin en otro dispositivo, dashboard, reportes) requieren conexión siempre.

**Alternativas consideradas**
- *Agregar el modo sin conexión al final del proyecto*: descartado explícitamente en el plan porque forzaría a rehacer el flujo de venta y el manejo de folios ya construidos.

---

## ADR 004 — Idempotencia por operación (id único generado en el cliente)

**Estado:** Aceptada · Fase 0

**Contexto**
En el modo local-first (ADR 003), una operación creada sin conexión (una venta, un abono, un ajuste) puede intentar sincronizarse más de una vez — por un reintento de red, por cerrar y volver a abrir el navegador, o por un fallo a mitad de la sincronización. Sin control, esto podría duplicar ventas, duplicar cobros o descontar el inventario dos veces.

**Decisión**
Cada operación creada en el mostrador (venta, abono, ajuste, etc.) lleva un **id único generado en el cliente** (UUID) desde el momento en que se crea localmente. El backend guarda ese id (tabla `sync_operations`) y, si recibe una operación con un id ya procesado, la descarta o devuelve el resultado ya registrado, en vez de volver a aplicarla.

**Consecuencias**
- Un reintento de sincronización nunca duplica una venta, un abono o un movimiento de inventario.
- El backend necesita una tabla/índice de operaciones ya aplicadas (`sync_operations`) y aplicar las operaciones **en orden**.
- El id se genera en el cliente (no lo asigna el servidor), por lo que debe usarse un identificador con garantía práctica de unicidad (UUID v4) y no un consecutivo local ingenuo.
- Es la base técnica que hace segura la sincronización completa de la Fase 12 (jornada completa sin internet que se sincroniza sin duplicados).

**Alternativas consideradas**
- *Deduplicar por heurística en el servidor* (mismo producto + monto + minuto aproximado): descartado por ser propenso a falsos positivos/negativos; un id explícito generado en el cliente es determinístico y no depende de coincidencias de datos.
