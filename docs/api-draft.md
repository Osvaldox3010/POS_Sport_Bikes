# Borrador de API — BikePOS

> Fase 1. **Borrador**: los nombres exactos de rutas, verbos y payloads pueden ajustarse durante la construcción de cada módulo (Fases 3 a 15); lo que no debería cambiar es el conjunto de operaciones que cada módulo necesita cubrir. Todas las rutas van bajo el prefijo `/api`. Todas requieren JWT salvo `POST /api/auth/login`. La columna **Roles** indica quién puede llamar el endpoint (el backend valida esto en cada endpoint, no solo el frontend — `plan.md` sección 3).

Convención de nombres: `kebab-case` en rutas, sustantivos en plural para colecciones, verbos solo para acciones que no son CRUD puro (`/cancel`, `/verify-code`, etc.).

---

## auth

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/auth/login` | Público | Login con usuario/contraseña. Devuelve JWT + indica si requiere cambio de contraseña. |
| POST | `/api/auth/change-password` | Autenticado | Cambia la contraseña (obligatorio en el primer login). |
| GET | `/api/auth/me` | Autenticado | Datos del usuario en sesión (nombre, rol, debe cambiar contraseña). |

## users

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/users` | Admin, Súper Admin | Lista usuarios (filtra por rol activo/inactivo). |
| POST | `/api/users` | Admin (crea vendedores) · Súper Admin (crea admins) | Alta de usuario. Genera usuario + contraseña inicial según `plan.md` 4.1. |
| PATCH | `/api/users/{id}` | Admin, Súper Admin (según rol del usuario objetivo) | Editar datos del usuario. |
| POST | `/api/users/{id}/deactivate` | Admin, Súper Admin | Baja lógica (no se borra). |
| POST | `/api/users/{id}/reset-password` | Admin, Súper Admin | Asigna una contraseña temporal (ver propuesta D1). |

## catalog — categorías y marcas

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/categories` | Todos | Lista categorías activas. |
| POST | `/api/categories` | Admin | Alta de categoría. |
| PATCH | `/api/categories/{id}` | Admin | Renombrar / activar / desactivar. |
| GET | `/api/brands` | Todos | Lista marcas activas. |
| POST | `/api/brands` | Admin | Alta de marca. |
| PATCH | `/api/brands/{id}` | Admin | Renombrar / activar / desactivar. |

## catalog — productos

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/products/search?q=` | Todos | Búsqueda por ID o nombre (usa `unaccent`/`pg_trgm`, sección 4.3). |
| GET | `/api/products/{id}` | Todos | Detalle de un producto. El campo de precio de compra y % se omiten en la respuesta si el rol es VENDEDOR. |
| POST | `/api/products` | Admin | Alta de producto (genera ID automático, sección 4.2). |
| PUT | `/api/products/{id}` | Admin | Edición completa (el ID nunca cambia). |
| POST | `/api/products/{id}/deactivate` | Admin | Baja lógica. |

## inventory

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/inventory/movements` | Admin | Alta o baja manual de stock (baja requiere `reason`). |
| GET | `/api/inventory/movements` | Admin | Historial, filtrable por producto/fecha/tipo. |
| GET | `/api/inventory/low-stock` | Admin | Productos con stock por debajo del mínimo definido. |

## sales

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/sales` | Vendedor, Admin | Confirma una venta (transacción atómica: valida caja/stock/pagos, descuenta inventario, asigna folio). |
| GET | `/api/sales` | Todos | Historial por día/semana/mes (sección 4.10). |
| GET | `/api/sales/{folio}` | Todos | Detalle de un ticket por folio (o resuelto desde el QR). |
| POST | `/api/sales/{id}/cancel` | Todos | Cancelación (bloqueada si el ticket tiene servicios). |
| POST | `/api/sales/{id}/reprint` | Todos | Registra la reimpresión en auditoría y regresa los datos del ticket para volver a imprimir. |
| POST | `/api/sales/{id}/adjustments` | Admin | Ajuste registrado sobre un ticket con servicios (ver D6). |

## returns / refunds

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/sales/{id}/returns` | Todos | Devuelve una o varias partidas (parcial o total); regresa stock. |
| GET | `/api/sales/{id}/returns` | Todos | Devoluciones ya hechas sobre ese ticket (para validar el máximo devolvible). |
| POST | `/api/refunds` | Todos | Registra un reembolso ligado a una devolución o cancelación. **El método siempre es EFECTIVO (regla D5)**; el endpoint solo recibe el monto — no hay parámetro de método. |

## layaway (apartados)

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/layaways` | Todos | Crea un apartado (aparta stock de inmediato). |
| GET | `/api/layaways` | Todos | Lista de apartados, con filtro por estado. |
| GET | `/api/layaways/{id}` | Todos | Detalle: partidas, abonos, saldo, días transcurridos. |
| POST | `/api/layaways/{id}/payments` | Todos | Registra un abono (fecha/hora/método); liquida si completa el saldo. |
| POST | `/api/layaways/{id}/cancel` | Todos | Cancela (solo si es puros productos); devuelve abonos (en efectivo, D5) y stock. |

## cash (caja y corte)

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/cash-sessions/open` | Todos | Abre la caja del día con el fondo inicial. |
| GET | `/api/cash-sessions/current` | Todos | Sesión de caja abierta actual (si existe). |
| POST | `/api/cash-sessions/{id}/expenses` | Todos | Agrega un gasto del día. |
| POST | `/api/cash-sessions/{id}/close` | Todos | Cierra la caja. Recibe **dos montos capturados juntos**: `countedCash` (efectivo contado) y `countedCardTransfer` (tarjeta + transferencia contado a mano, un solo campo). El backend congela `expectedCash` y `expectedTotal`, y calcula `cashDifference` y `totalDifference` (ver `docs/reglas-corte-apartados-devoluciones.md` sección 1). |
| GET | `/api/cash-sessions` | Admin | Consulta por día/semana/mes/año/rango (consolidado + detalle), incluyendo ambas diferencias. |
| GET | `/api/cash-sessions/{id}/pdf` | Todos | PDF del corte, con ambas diferencias (se genera en el navegador para funcionar sin conexión, ver `plan.md` 4.16). |

## price-approval

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/price-overrides` | Vendedor | Solicita precio personalizado; dispara el correo con el código. |
| POST | `/api/price-overrides/{id}/verify-code` | Vendedor | Valida el código de un solo uso recibido por correo. |
| POST | `/api/price-overrides/{id}/mark-pending-review` | Vendedor | Opción C: vender ahora, revisar después. |
| POST | `/api/price-overrides/emergency-codes` | Admin | Genera un lote de códigos de emergencia (se muestran una sola vez). |
| POST | `/api/price-overrides/{id}/use-emergency-code` | Vendedor | Opción D: usa un código de emergencia. |
| POST | `/api/price-overrides/{id}/mark-reviewed` | Admin | Marca como revisado, con nota. |

## reports / dashboard

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/dashboard/summary?range=` | Admin | Todas las métricas de `plan.md` 4.15 para el período indicado, con comparación al período anterior. |
| GET | `/api/reports/pdf?period=` | Admin | Genera el reporte PDF mensual/anual. |

## settings

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET / PUT | `/api/settings/store` | Admin | Datos de la tienda (nombre, dirección, teléfono, contacto). |
| GET / PUT | `/api/settings/ticket-template` | Admin | Formato del ticket, con vista previa en el frontend. |
| GET / PUT | `/api/settings/authorization-contact` | Admin | Correo (y SMS si aplica) para códigos de precio y para avisos de stock negativo al sincronizar. |
| GET / PUT | `/api/settings/price-approval` | Admin | Parámetros: espera antes de habilitar C, caducidad del código (ver D8). |

## audit

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/audit` | Admin, Súper Admin | Filtros por fecha, usuario, tipo de evento, ticket. Solo lectura — nunca `PUT`/`DELETE`. |
| GET | `/api/audit/review-queue` | Admin | Pendientes de revisión (precios vía C, ventas con stock negativo, conflictos de sync). |
| POST | `/api/audit/review-queue/{id}/resolve` | Admin | Marca un pendiente como resuelto, con nota. |

## sync

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| POST | `/api/sync/operations` | Vendedor, Admin (desde la PC del mostrador) | Envía un lote de operaciones pendientes, cada una con su `client_operation_id`; el backend las aplica en orden y de forma idempotente. Si alguna deja stock negativo, además de marcarla en `review_queue`, dispara el correo de aviso al admin. |
| GET | `/api/sync/catalog-snapshot` | Vendedor, Admin | Copia del catálogo (sin precio de compra ni %) para refrescar la base local. |
| GET | `/api/sync/folio-counter?date=` | Vendedor, Admin | Consecutivo de folio vigente en el servidor, para reconciliar el contador local al reconectar. |

## backup

| Método | Endpoint | Roles | Descripción |
|---|---|---|---|
| GET | `/api/backup/csv` | Admin, Súper Admin | Genera y descarga el ZIP con un CSV por tabla (sin contraseñas ni códigos). |
| GET | `/api/backup/status` | Admin, Súper Admin | Fecha del último respaldo, para el aviso de "hace X días". |

---

## Notas generales

- Todos los endpoints de escritura (`POST`/`PUT`/`PATCH`) que afectan inventario, caja, ventas o precios se ejecutan en una **transacción** de base de datos (sección 19 de `plan.md`/`Design.md`).
- El backend **siempre revalida** lo que ya validó el frontend (stock, permisos, sumas de pago) — nunca confía en el cliente, ni siquiera en el flujo local-first.
- Las respuestas de error siguen un formato centralizado, sin exponer detalles internos (ver `plan.md` sección 10 y el ejemplo de `Design.md` sección 18).
- La documentación formal (Swagger/OpenAPI) se genera a partir de la Fase 3, cuando el backend empieza a existir; este documento es la referencia previa para diseñar los DTOs y los controladores.
