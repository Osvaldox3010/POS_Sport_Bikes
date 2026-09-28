# Casos de uso — BikePOS

> Fase 1. Cubre los tres actores humanos (Vendedor, Admin, Súper Admin) y el actor "Sistema" (sincronización y jobs internos). Los casos marcados con ⚠️ son los de mayor riesgo (coinciden con la prioridad de pruebas de `plan.md`, sección 9) y llevan flujo detallado más abajo.

## Actores

| Actor | Descripción |
|---|---|
| **Vendedor** | Usuario de mostrador. Vende, busca, consulta precios, hace apartados, cancela/devuelve, abre/cierra caja. No ve precio de compra ni administra catálogo/usuarios. |
| **Admin** | Todo lo del vendedor, más catálogo, inventario, usuarios (vendedores), dashboard, auditoría, configuración, reportes y respaldo. |
| **Súper Admin** | Todo lo del admin, más alta/baja/contraseñas de administradores. Es único, lo crea el sistema. |
| **Sistema** | Procesos automáticos: sincronización de la PC del mostrador, recálculo de corte al sincronizar, recordatorio de respaldo. |

---

## Tabla general de casos de uso

| ID | Actor(es) | Caso de uso | Módulo |
|---|---|---|---|
| UC-01 | Todos | Iniciar sesión (con conexión) | auth |
| UC-02 | Vendedor / Admin | Iniciar sesión (sin conexión, solo PC del mostrador) | auth |
| UC-03 | Todos | Cambiar contraseña obligatoria en el primer inicio de sesión | auth |
| UC-04 | Admin | Alta de vendedor | users |
| UC-05 | Súper Admin | Alta de administrador | users |
| UC-06 | Admin / Súper Admin | Dar de baja un usuario | users |
| UC-07 | Admin | Alta de producto | catalog |
| UC-08 | Admin | Editar producto | catalog |
| UC-09 | Admin | Alta de stock (sin motivo) | inventory |
| UC-10 | Admin | Baja de stock (con motivo obligatorio) | inventory |
| UC-11 | Vendedor / Admin | Buscar producto / consultar precio | catalog |
| UC-12 | Vendedor / Admin | ⚠️ Realizar venta con pago único | sales |
| UC-13 | Vendedor / Admin | ⚠️ Realizar venta con pago dividido | sales, payments |
| UC-14 | Vendedor / Admin | Agregar servicio a la venta | sales |
| UC-15 | Vendedor | ⚠️ Solicitar precio personalizado (flujo normal, código por correo) | price-approval |
| UC-16 | Vendedor | ⚠️ Vender con precio pendiente de revisión (opción C) | price-approval |
| UC-17 | Vendedor | ⚠️ Usar código de emergencia (opción D) | price-approval |
| UC-18 | Admin | Generar lote de códigos de emergencia | price-approval |
| UC-19 | Todos | ⚠️ Crear apartado | layaway |
| UC-20 | Todos | ⚠️ Registrar abono a apartado | layaway |
| UC-21 | Todos | Liquidar apartado | layaway |
| UC-22 | Todos | Cancelar apartado (solo productos) | layaway |
| UC-23 | Todos | ⚠️ Cancelar venta | returns |
| UC-24 | Todos | ⚠️ Devolver productos de una venta | returns |
| UC-25 | Todos | ⚠️ Reembolsar | returns, payments |
| UC-26 | Admin | Corregir error en ticket con servicios (ajuste) | returns, audit |
| UC-27 | Todos | Abrir caja | cash |
| UC-28 | Todos | ⚠️ Cerrar caja (corte) | cash |
| UC-29 | Admin | Consultar cortes por período o rango | cash |
| UC-30 | Admin / Súper Admin | Consultar auditoría | audit |
| UC-31 | Admin | Ver dashboard | reports |
| UC-32 | Admin | Generar reporte PDF mensual/anual | reports |
| UC-33 | Admin | Descargar respaldo CSV | settings |
| UC-34 | Admin | Configurar datos de tienda y formato de ticket | settings |
| UC-35 | Todos | Reimprimir ticket | sales, audit |
| UC-36 | Vendedor / Admin (PC mostrador) | ⚠️ Vender sin conexión | sales, sync |
| UC-37 | Sistema | ⚠️ Sincronizar operaciones al reconectar | sync |
| UC-38 | Admin | Revisar pendientes de revisión | audit, sync |

---

## Casos de uso detallados

### UC-12 / UC-13 — Realizar venta (pago único / dividido)

**Actor:** Vendedor, Admin (solo desde la PC del mostrador).

**Precondiciones**
- Hay una sesión de caja abierta.
- El usuario tiene sesión iniciada.

**Flujo principal**
1. El usuario busca y agrega productos y/o servicios al ticket.
2. Por cada partida, elige menudeo o mayoreo (o usa el botón "pasar todo a mayoreo").
3. El sistema muestra el total y la cantidad de productos/servicios.
4. El usuario pulsa "Cobrar".
5. Elige uno o varios métodos de pago. Si es efectivo, captura el monto entregado.
6. El sistema valida: caja abierta, al menos una partida, stock suficiente por partida, suma de pagos = total.
7. El sistema crea la venta, descuenta inventario, asigna folio (`DD-MM-AAAA-NN`), calcula cambio si aplica, y todo ocurre en una sola transacción.
8. El ticket se imprime.

**Flujos alternos**
- 6a. Suma de pagos ≠ total → el sistema no permite confirmar, indica la diferencia.
- 6b. Stock insuficiente en alguna partida → el sistema bloquea esa partida y lo indica (ver D3 en `plan.md`).
- 6c. Sin caja abierta → el sistema bloquea la venta y sugiere abrir caja.

**Postcondiciones**
- Venta con estado *pagado*, folio asignado, stock descontado, pago(s) registrados y ligados a la sesión de caja actual.

---

### UC-15 / UC-16 / UC-17 — Precio personalizado (normal, opción C, opción D)

**Actor:** Vendedor (con conexión). Admin/Súper Admin no requieren este flujo (ponen precio personalizado sin aprobación).

**Precondiciones**
- Hay conexión a internet (este flujo **no existe sin conexión**).
- El contacto de autorizaciones está configurado.

**Flujo principal (normal)**
1. El vendedor captura un precio personalizado en una partida y un motivo breve.
2. El sistema calcula un código de un solo uso, lo liga a producto + precio + ticket + vendedor, y lo envía por correo (y SMS si hay opción gratuita) al contacto de autorizaciones.
3. El vendedor ingresa el código recibido.
4. El sistema valida el código (hash, un solo uso, no caducado, dentro del máximo de intentos) y autoriza el precio.

**Flujos alternos**
- 3a. **El admin no contesta y el vendedor elige "Vender ahora, revisar después" (opción C):** la venta continúa, el ticket queda *pendiente de revisión*; se valida igualmente que el precio no esté por debajo del precio de compra.
- 3b. **El vendedor usa un código de emergencia (opción D):** el admin ya generó un lote de códigos con hash (UC-18); el vendedor ingresa uno, el sistema lo valida (un solo uso) y registra quién, en qué partida y a qué precio.
- 3c. **El precio (por C o D) queda por debajo del precio de compra:** el sistema rechaza la operación; el vendedor solo ve el aviso "precio no permitido", nunca el precio de compra.
- Sin conexión: la pantalla indica "solo disponible con conexión" y no se ofrecen C ni D.

**Postcondiciones**
- Precio autorizado y aplicado a la partida, con entrada en auditoría indicando el mecanismo (código normal, C o D) y quién autorizó (si aplica).

---

### UC-19 / UC-20 / UC-21 / UC-22 — Apartados

**Actor:** Cualquier usuario con sesión.

**Precondiciones (crear)**
- Hay al menos una partida (producto y/o servicio).
- Se captura nombre y teléfono del cliente (no hay registro de clientes).

**Flujo principal — Crear**
1. Se arma el ticket igual que una venta.
2. Al confirmar "Apartar" en vez de "Cobrar", el sistema descuenta el stock de inmediato (se aparta), asigna folio, marca el ticket como *en proceso de pago* y opcionalmente registra un anticipo (sin mínimo).
3. Se imprime el comprobante del apartado.

**Flujo principal — Abonar**
1. Se busca el apartado (folio o QR).
2. Se captura el monto y el método del abono.
3. El sistema registra el abono con fecha, hora y método, y lo liga a la sesión de caja del día en que se cobra.
4. Se imprime el comprobante del abono.
5. Si el abono completa el total, el ticket pasa a *pagado* y se guarda su fecha de finalizado (liquidación); el ticket impreso se marca "LIQUIDADO".

**Flujo alterno — Cancelar**
- Solo aplica si el apartado es **puros productos** (sin servicios).
- Se devuelven todos los abonos registrados (en efectivo, regla D5) y el stock apartado.
- Un apartado con servicios no se puede cancelar (ver UC-26).

**Postcondiciones**
- Cada abono queda contabilizado en el corte del día en que se cobró (no en el día de creación ni en el de liquidación).

---

### UC-23 / UC-24 / UC-25 — Cancelación, devolución y reembolso

**Actor:** Cualquier usuario, sin aprobación ni PIN, en cualquier momento.

**Precondiciones**
- El ticket existe y es localizable por folio o QR.
- (Cancelación) el ticket no contiene servicios.

**Flujo principal — Cancelación**
1. Se localiza el ticket.
2. El usuario captura el motivo (obligatorio).
3. El sistema genera los movimientos inversos: el stock regresa, se crea un reembolso en efectivo por el monto cobrado (regla D5), el ticket pasa a *cancelado* conservando su folio.

**Flujo principal — Devolución (parcial o total)**
1. Se localiza el ticket por folio o QR.
2. Se eligen los productos y cantidades a devolver (los servicios no se devuelven).
3. El sistema valida que no se devuelva más de lo vendido menos lo ya devuelto.
4. El producto regresa al stock al precio en que se vendió (menudeo, mayoreo o personalizado).
5. Se captura el motivo (obligatorio).

**Flujo principal — Reembolso**
1. Ligado a una devolución o cancelación ya creada.
2. El usuario captura el **monto**, sin exceder lo cobrado. El método siempre es **efectivo** (regla D5: el vendedor no tiene acceso a la cuenta bancaria ni a la terminal de la tienda para revertir un cargo o una transferencia), sin importar cómo se pagó originalmente.
3. El sistema resta el monto del **efectivo esperado** en el corte del día en que se hace el reembolso (no se toca el corte del día original).

**Postcondiciones**
- Todo queda en auditoría: usuario, fecha/hora, ticket afectado, motivo, impacto en inventario/caja/pagos.

---

### UC-26 — Corregir error en ticket con servicios (ajuste)

**Actor:** Admin.

**Contexto:** un ticket con servicios no se puede cancelar ni el servicio se puede devolver (regla 4.6/4.12). Este caso de uso cubre la regla D6 (confirmada).

**Flujo principal**
1. El admin localiza el ticket.
2. Registra un **ajuste**: una operación nueva ligada al ticket original, con motivo obligatorio.
3. El sistema registra el ajuste en auditoría, sin alterar el ticket original ni sus montos históricos.
4. Si el ajuste implica devolver dinero, se entrega en efectivo (regla D5) y/o se genera una venta nueva, según corresponda.

---

### UC-27 / UC-28 — Caja y corte

**Actor:** Vendedor o Admin.

**Precondiciones (abrir)**
- No hay otra caja abierta ese día.

**Flujo principal — Abrir**
1. El usuario captura el fondo inicial.
2. El sistema crea la sesión de caja del día.

**Flujo principal — Cerrar (corte)**
1. El usuario captura los gastos del día (concepto + monto), uno o varios.
2. El sistema muestra el resumen de tickets, abonos cobrados hoy y devoluciones/cancelaciones del día.
3. El usuario captura, **en un mismo paso**, dos montos verificados a mano: el **efectivo contado** y el **total de tarjeta + transferencia** según su control externo (un solo campo que suma ambos métodos).
4. El sistema calcula y **congela** los montos esperados (`Efectivo esperado`, `Total con fondo`) y ambas diferencias: **Diferencia Efectivo** (efectivo contado − efectivo esperado) y **Diferencia Total** ((efectivo contado + tarjeta/transferencia contado) − total con fondo). Un corte cerrado no se edita.
5. Se imprime el corte (con ambas diferencias) y se genera su PDF.

**Postcondiciones**
- La caja queda cerrada; no se puede vender hasta abrir una nueva (al día siguiente).
- Ver `docs/reglas-corte-apartados-devoluciones.md` sección 1 para el detalle de qué detecta cada diferencia.

---

### UC-36 / UC-37 — Venta sin conexión y sincronización

**Actor:** Vendedor/Admin (crea la venta) y Sistema (sincroniza).

**Precondiciones**
- La PC del mostrador ya tiene copia local del catálogo y un usuario que inició sesión antes.

**Flujo principal — Vender sin conexión**
1. El flujo es idéntico a UC-12/13, pero toda validación de stock usa la copia local.
2. La operación se guarda en la cola local con un **id único generado en el cliente** (ver ADR-004 en `docs/decisions.md`) y el folio se genera con el contador local del día.
3. El ticket se imprime igual que en línea.

**Flujo principal — Sincronizar (Sistema)**
1. Al recuperar conexión, la PC envía las operaciones pendientes **en el orden en que se crearon**.
2. El backend aplica cada operación usando su id de cliente para no duplicarla si ya fue aplicada antes.
3. Si una venta sincronizada deja stock negativo (porque el admin ajustó el stock mientras no había conexión y la copia local no se había enterado), el backend **acepta la venta**, la marca *pendiente de revisión* **y envía un correo al contacto de autorizaciones** avisando qué venta se aplicó y qué producto quedó en negativo.
4. Si una operación ya no es válida por otra razón, no se descarta: se registra en `review_queue` para que el admin la resuelva.
5. El backend recalcula el corte del día afectado y marca cualquier diferencia.
6. La PC actualiza su contador de folios con el valor confirmado por el servidor.

**Postcondiciones**
- Todas las operaciones quedan aplicadas exactamente una vez; las que generaron conflicto quedan visibles en el dashboard del admin (UC-38) y, en el caso de stock negativo, también por correo.

---

## Trazabilidad

Cada caso de uso corresponde a un módulo de `plan.md` sección 2 y a una fase de construcción de la sección 8. El detalle línea por línea de reglas de negocio para corte, apartados y devoluciones se documenta por separado en `docs/reglas-corte-apartados-devoluciones.md`.
