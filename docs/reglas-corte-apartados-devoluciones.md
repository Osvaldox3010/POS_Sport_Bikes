# Reglas finales — Corte de caja, Apartados y Devoluciones/Cancelaciones/Reembolsos

> **✅ Documento cerrado.** D5, D6 y D7 confirmados, y la fórmula de "Diferencia" del corte quedó resuelta como dos indicadores independientes: **Diferencia Efectivo** y **Diferencia Total**. Con esto, la Fase 1 queda completa.

---

## 1. Corte de caja

### 1.1 Reglas
1. Hay **un corte por día** y **una sola caja abierta a la vez**. La del día anterior se cierra antes de abrir otra.
2. Lo abre y lo cierra el vendedor o el admin (sin diferencia de permisos entre ambos para esta operación).
3. Al abrir, se captura el **fondo inicial**.
4. Al cerrar, **primero** se capturan los gastos del día (concepto + monto), **después** se muestra el resumen.
5. En un solo paso, el usuario captura **dos montos contados a mano**: el **efectivo real** en el cajón y el **total de tarjeta + transferencia** que tiene anotado en su control externo (un solo campo que suma ambos métodos, ya que no hay integración con la terminal ni con el banco). El sistema calcula, con ambos, **dos diferencias independientes** (sección 1.3).
6. Un corte **cerrado no se edita**; los montos esperados y ambos montos contados se congelan en ese momento.
7. Se imprime en la impresora de tickets y se genera también en PDF, incluyendo ambas diferencias.
8. Funciona sin conexión.
9. Si un día no se abre caja, no hay corte: los reportes lo tratan como "sin actividad", no como error.

### 1.2 Fórmulas

Con F = fondo inicial, E = cobros en efectivo, T = cobros por terminal, X = cobros por transferencia, R = reembolsos del día (siempre en efectivo, regla D5), G = gastos, **CE = efectivo contado (captura manual)**, **CTT = tarjeta + transferencia contado (captura manual, un solo campo)**:

| Concepto | Fórmula |
|---|---|
| Efectivo esperado | F + E − R − G |
| Terminal | T |
| Transferencia | X |
| Total no efectivo | Terminal + Transferencia |
| Total de ventas (sin fondo) | (E − R) + Total no efectivo |
| Total con fondo | Total de ventas (sin fondo) + F − G |
| **Diferencia Efectivo** | CE − Efectivo esperado |
| **Diferencia Total** | (CE + CTT) − Total con fondo |

### 1.3 Qué detecta cada diferencia (✅ resuelto)

- **Diferencia Efectivo:** compara el efectivo físico contado contra lo que el sistema calcula que debería haber en el cajón. Detecta errores de manejo de efectivo (dar mal el cambio, un billete faltante, etc.). Es exactamente la fórmula que ya existía.
- **Diferencia Total:** compara **todo lo que el vendedor verificó a mano ese día** (efectivo contado + su control externo de tarjeta/transferencia) contra el **Total con fondo** que calcula el sistema a partir de lo registrado en cada venta. Detecta algo que la Diferencia Efectivo no puede ver: que el sistema haya registrado una venta como cobrada por tarjeta o transferencia, pero ese dinero **nunca haya llegado de verdad** (una transferencia que no cayó, un cobro con terminal que se marcó como exitoso por error, etc.) — o al revés, un cobro que sí llegó pero no se registró.
- Ambos montos (CE y CTT) se capturan **juntos, en el mismo paso** del cierre (regla 1.1.5).

**Ejemplo (mismos datos que antes: F=$500, E=$1,200, R=$50, G=$100, T=$800, X=$300 → Efectivo esperado=$1,550, Total con fondo=$2,650):**

| Escenario | CE | CTT | Diferencia Efectivo | Diferencia Total | Lectura |
|---|---|---|---|---|---|
| Todo cuadra | $1,550 | $1,100 | $0 | $0 | Día perfecto |
| Falta un billete | $1,540 | $1,100 | −$10 | −$10 | Error de efectivo, se ve en ambas |
| Una transferencia de $50 nunca llegó | $1,550 | $1,050 | $0 | −$50 | El efectivo cuadra, pero el control externo revela un cobro no efectivo que no se recibió — **solo la Diferencia Total lo detecta** |

### 1.4 Vista de administrador — cortes por período

- El admin consulta cortes por **día, semana, mes, año, fecha o rango personalizado**.
- En períodos de varios días se **suman**: ventas, no efectivo, gastos, reembolsos y **ambas diferencias** por separado.
- El **fondo inicial no se suma** entre días (cada día tiene su propio fondo; sumarlo no tendría significado económico).
- Cada corte del período se puede abrir en detalle desde la vista consolidada.

> **Nota sobre el formato impreso del corte:** la estructura de ticket que aparece en `plan.md` sección 4.13 se actualiza para incluir una línea de "Tarjeta/transferencia contado (a mano)" junto a "Efectivo contado", y las dos diferencias por separado en vez de una sola. El detalle exacto de layout se resuelve al construir el módulo de caja (Fase 10); ver también `docs/wireframes.md`, sección 6, ya actualizada.

---

## 2. Apartados

### 2.1 Reglas
1. Un apartado tiene dos estados posibles: **pagado** y **en proceso de pago**. No hay estados de "trabajo" (recibido/en proceso/entregado): esos aplican a reparaciones, y las reparaciones se venden como servicio, no como apartado.
2. Puede incluir productos y servicios.
3. Se captura **nombre y teléfono** del cliente; no existe una tabla de clientes.
4. **Sin anticipo mínimo.**
5. El **stock se descuenta al crear el apartado** (se aparta desde ese momento, no al liquidar).
6. Recibe **uno o varios abonos**; cada abono guarda fecha, hora y método.
7. Al completarse el pago, pasa a *pagado* y se guarda su **fecha de finalizado**.
8. **No vencen.** El admin ve los días transcurridos desde su creación.
9. Se imprime al crearlo, en cada abono, y al liquidarlo (marcado "LIQUIDADO").
10. Cualquier usuario del sistema puede crearlos y abonarlos, sin restricción de rol.
11. Sin conexión: se pueden crear y abonar; se guardan completos en la PC sin importar su antigüedad (a diferencia de las ventas normales, que solo guardan los últimos 30 días).

### 2.2 Cancelación de apartados
- Solo se cancelan los apartados de **puros productos** (sin ningún servicio).
- Al cancelar: se devuelven **todos los abonos** recibidos hasta ese momento y se devuelve el **stock** apartado (en efectivo, por regla D5, sin importar cómo se abonó originalmente — ver sección 3.5).
- Un apartado con al menos un servicio **no se puede cancelar** — ver la regla D6 (sección 3.3) para el mecanismo de corrección en ese caso.

### 2.3 Regla D7 — devolver productos de un apartado aún no liquidado (**✅ confirmada**)

**Contexto:** un apartado en proceso de pago podría necesitar que el cliente regrese uno de los productos antes de terminar de pagarlo (ej. cambió de talla).

**Regla confirmada:** mientras el apartado **no esté pagado por completo**, no existe una "devolución parcial de apartado" como operación distinta; se maneja igual que una **cancelación** (sección 2.2), es decir, solo aplica a apartados de puros productos. Si el apartado tiene productos y servicios y se necesita quitar solo un producto antes de liquidar, se resuelve como un ajuste (ver UC-26 en `docs/casos-de-uso.md`): el admin registra la operación ligada al apartado, con motivo.

Una vez que el apartado está **pagado** (liquidado), aplican las reglas normales de devolución de la sección 3.

### 2.4 Cómo cuenta un apartado en el corte

- Cada abono cuenta en el corte **del día en que se cobra**, no en el día de creación ni en el de liquidación.
- El ticket del apartado no se cuenta completo ni al abrirse ni al cerrarse; solo los montos de cada abono, el día que se cobran.

**Ejemplo (de `plan.md` 4.11):** apartado de $1,000. Lunes: anticipo $300 efectivo. Miércoles: abono $200 transferencia. Viernes: liquidación $500 con terminal.
- Corte del lunes: +$300 efectivo.
- Corte del miércoles: +$200 transferencia.
- Corte del viernes: +$500 terminal. El apartado queda *pagado*, con fecha de finalizado el viernes.

---

## 3. Cancelaciones, devoluciones y reembolsos

### 3.1 Principio general
Ninguna operación confirmada se borra ni se edita. Todo cambio posterior es una **operación nueva ligada a la venta original**, con usuario, fecha/hora, **motivo obligatorio**, el ticket afectado y su impacto en inventario, caja y pagos. Todo entra a auditoría.

### 3.2 Cancelación de venta
- La puede hacer **cualquier usuario**, en cualquier momento, **sin aprobación ni PIN**.
- Cancela el ticket completo: el stock regresa y el dinero cobrado se reembolsa automáticamente (en efectivo, regla D5).
- El ticket **conserva su folio** y pasa a *cancelado*; el folio nunca se reutiliza.
- **Los tickets con servicios no se cancelan** (ver 3.3).

### 3.3 Regla D6 — corregir un ticket con servicios (**✅ confirmada**)

**Contexto:** un servicio no se devuelve ni se reembolsa, y un ticket que contiene al menos un servicio no se puede cancelar (regla de negocio 18 en `plan.md`). Sin embargo, un ticket con servicios sí puede tener un error (precio mal capturado, servicio equivocado, cobro duplicado).

**Regla confirmada:** el admin (no el vendedor) corrige el error mediante un **ajuste registrado**: una operación nueva, ligada al ticket original, con motivo obligatorio y visible en auditoría, que documenta la corrección sin alterar el ticket original. Reglas del ajuste:
- Si el ticket tiene productos además de servicios, los productos sí se pueden devolver normalmente (regla 4.12: "los productos de un ticket con servicios sí se pueden devolver"); el ajuste solo cubre la parte de servicios.
- El ajuste no mueve inventario (los servicios no tienen stock) y su efecto en caja se resuelve como un reembolso manual en efectivo (regla D5) del monto que corresponda, ligado al ajuste.
- Un "cambio de servicio" (ej. el cliente pidió "Parche" y era "Servicio completo") se resuelve como: ajuste que anula el monto original + una venta nueva con el servicio correcto — sin lógica especial adicional, igual que un cambio de producto (regla de negocio de `plan.md` 4.12).

### 3.4 Devolución de productos
- Se localiza el ticket por folio o QR.
- Se eligen productos y cantidades; puede ser parcial o total.
- No se puede devolver más de lo vendido menos lo ya devuelto.
- El producto **siempre regresa al stock**, sin excepción.
- Los servicios nunca se devuelven; los productos de un ticket con servicios sí.
- El monto devuelto es el precio al que se vendió en el ticket original (menudeo, mayoreo o personalizado) — nunca el precio actual del catálogo.

### 3.5 Reembolso
- Operación independiente, ligada siempre a una devolución o a una cancelación (nunca "suelta").
- **Siempre se entrega en efectivo, sin importar el método con el que se pagó originalmente.** El vendedor no tiene acceso a la cuenta bancaria de la tienda ni a la terminal para revertir un cargo o una transferencia, así que esas opciones no existen para un reembolso.
- Quien lo hace captura el **monto**, sin exceder lo cobrado en la venta/devolución original. No hay campo de "elegir método": el sistema lo fija en EFECTIVO.
- No hay registro de cliente ni saldo a favor: el dinero se entrega en efectivo en el momento.
- Resta siempre del **efectivo esperado** del corte del día en que se hace el reembolso (variable R, sección 1.2); el corte del día original ya cerrado no se toca.
- **Nota operativa:** si se reembolsa en efectivo una venta que se cobró originalmente por transferencia o tarjeta, ese día puede quedar con menos efectivo del esperado si no entró suficiente efectivo por otras ventas. No es un error de caja: es una consecuencia esperada de la regla D5, y queda reflejada en el corte de ese día como cualquier otro reembolso.

### 3.6 Sin conexión
- Se pueden cancelar y devolver tickets guardados en la PC del mostrador: los **últimos 30 días**.
- Tickets más antiguos que la copia local requieren conexión.

---

## 4. Resumen de aprobación

| # | Punto | Estado |
|---|---|---|
| **D5** | Todo reembolso se entrega en efectivo, sin importar el método original de la venta | ✅ Confirmada |
| **D6** | Un ticket con servicios se corrige con un ajuste registrado, ligado al ticket, con motivo y auditoría | ✅ Confirmada |
| **D7** | Mientras un apartado no esté pagado, "devolver" un producto se maneja como cancelación (solo puros productos) | ✅ Confirmada |
| — | Fórmula del corte: dos diferencias — **Diferencia Efectivo** (como antes) y **Diferencia Total** (efectivo contado + tarjeta/transferencia contado a mano, contra el total con fondo), capturadas juntas en el mismo paso | ✅ Confirmada |

**Este documento queda cerrado.**
