# Wireframes — BikePOS

> Fase 1. Estos son wireframes de **baja fidelidad**: definen contenido y disposición (layout), no colores, tipografía ni estilo — eso ya está resuelto en `Design.md` y se aplica al construir cada pantalla real (Fase 6 en adelante). El layout de dos columnas (contenido + panel lateral) sigue la estructura de `Design.md` sección 4 y 5.7.

---

## 1. Punto de venta (POS)

```
┌──────────────────────────────────────────────────────────────────────┐
│ [Logo] BikePOS      Inicio  Ventas  Apartados  Inventario  Reportes  │ ← navbar, ítems según rol
├──────────────────────────────────────────┬───────────────────────────┤
│  BUSCAR PRODUCTO O ID          [🔍_____]  │  TICKET ACTUAL            │
│                                            │  Vendedor: Juan           │
│  Resultados:                              │  ────────────────         │
│  ┌──────────────────────────────────────┐ │  Casco MTB negro   x1     │
│  │ PROD-CASC-0002  Casco MTB negro       │ │   $450.00 (menudeo)       │
│  │ Menudeo $450 · Mayoreo $400 · Stock 12│ │   [menudeo ▾]  [🗑]       │
│  │                        [+ Agregar]    │ │  Cadena SRAM       x2     │
│  └──────────────────────────────────────┘ │   $280.00 (mayoreo)       │
│  ...                                      │   [mayoreo ▾]  [🗑]       │
│                                            │  ────────────────         │
│  [Pasar todo el ticket a mayoreo]         │  [+ Agregar servicio]     │
│  [+ Precio personalizado] (en la partida) │  ────────────────         │
│                                            │  Cant. productos/serv.: 3 │
│                                            │  TOTAL         $810.00    │
│                                            │  [ Apartar ]  [ COBRAR ]  │
└──────────────────────────────────────────┴───────────────────────────┘
```

**Modal — Cobrar**
```
┌─────────────────────────────────────┐
│  Cobrar — Total $810.00              │
│  ○ Efectivo  ○ Tarjeta               │
│  ○ Transferencia  ○ Dividir pago     │
│                                       │
│  (si Efectivo o Dividir)             │
│  Efectivo recibido: [$__________]    │
│  Cambio:            $190.00          │
│                                       │
│  [ Cancelar ]         [ Confirmar ]  │
└─────────────────────────────────────┘
```

---

## 2. Modal de servicio (reparación)

```
┌───────────────────────────────────┐
│  Agregar servicio                   │
│  Descripción: [_________________]   │  ej. "Parche", "Servicio completo"
│  Precio:      [$_______________]    │
│  Cantidad: 1 (fijo, no editable)     │
│                                     │
│  [ Cancelar ]        [ Agregar ]   │
└───────────────────────────────────┘
```

---

## 3. Modal de precio personalizado

```
┌───────────────────────────────────────────┐
│  Precio personalizado                       │
│  Producto: Casco MTB negro                  │
│  Precio de menudeo actual: $450.00          │
│  Precio solicitado:  [$__________]          │
│  Motivo:             [______________]       │
│                                             │
│  [ Enviar solicitud al admin (correo) ]    │
│  Código recibido:  [______]   [ Validar ]  │
│                                             │
│  ── Si el admin no contesta ──             │
│  [ Vender ahora, revisar después ]         │
│  [ Usar código de emergencia ]             │
│                                             │
│  ⚠ No disponible sin conexión               │
└───────────────────────────────────────────┘
```

---

## 4. Devoluciones

```
┌──────────────────────────────────────────────────────────────────────┐
│  Devoluciones — Buscar ticket             [ Folio o escanear QR ___ ]│
├────────────────────────────────────────────┬─────────────────────────┤
│  Ticket 24-09-2026-01 · Juan Vendedor       │  Resumen de devolución   │
│  ┌────────────────────────────────────────┐ │  Cadena SRAM  x1  $140  │
│  │ [ ] Casco MTB negro   x1    $450.00     │ │  ─────────────────      │
│  │ [x] Cadena SRAM       x2    $280.00     │ │  Total a devolver: $140 │
│  │      cantidad a devolver: [1 ▾]         │ │  Se reembolsa en:       │
│  │ Servicio: Parche  (no se puede devolver)│ │   EFECTIVO (fijo)       │
│  └────────────────────────────────────────┘ │                         │
│  Motivo (obligatorio): [________________]    │  [ Confirmar devolución]│
└──────────────────────────────────────────────┴─────────────────────────┘
```

*(La pantalla de Cancelación reutiliza el mismo buscador; si el ticket tiene servicios, el botón "Cancelar venta" aparece deshabilitado con el mensaje "Este ticket tiene servicios; usa un ajuste". El reembolso siempre es en efectivo — regla D5 — así que ya no se elige método, solo se confirma el monto.)*

---

## 5. Apartados

```
┌──────────────────────────────────────────────────────────────────────┐
│  Apartados                                       [ + Nuevo apartado ]│
├───────────────────────────────────────────────┬──────────────────────┤
│ FOLIO      CLIENTE            PROD/SERV ESTADO SALDO  TOTAL│ Panel:   │
│ 24-09-...01 Ana · 55-1234-5678   3     [En proceso] $700 $1000│Folio  │
│                                                        │24-09-...01   │
│                                                        │Ana / 5551234 │
│                                                        │Días: 4        │
│                                                        │─────────────  │
│                                                        │Abonos:        │
│                                                        │ Lun $300 Efec.│
│                                                        │ Mié $200 Transf│
│                                                        │─────────────  │
│                                                        │Saldo: $500     │
│                                                        │[Registrar abono]│
│                                                        │[Cancelar apart.]│
└───────────────────────────────────────────────┴──────────────────────┘
```

---

## 6. Corte de caja

**Abrir caja**
```
┌─────────────────────────────────────┐
│  Abrir caja — 24-09-2026              │
│  Fondo inicial: [$_____________]      │
│  [ Abrir caja ]                       │
└─────────────────────────────────────┘
```

**Cerrar caja — paso 1: gastos**
```
┌─────────────────────────────────────┐
│  Cerrar caja — Gastos del día          │
│  Concepto             Monto            │
│  [________________]   [$______] [+]   │
│  [________________]   [$______] [+]   │
│  [ Continuar ]                         │
└─────────────────────────────────────┘
```

**Cerrar caja — paso 2: conteo manual (los dos montos juntos)**
```
┌─────────────────────────────────────────┐
│  Conteo del día                            │
│  Efectivo contado:            [$_______]   │
│  Tarjeta + transferencia                   │
│  (según tu control):          [$_______]   │
│  [ Continuar ]                             │
└─────────────────────────────────────────┘
```

**Cerrar caja — paso 3: resumen final**
```
┌─────────────────────────────────────────────┐
│  CORTE DE CAJA — 24-09-2026                    │
│  Abrió: Juan   Cerró: Juan   Fondo: $500        │
│                                                 │
│  Tickets del día (n)              [tabla]      │
│  Abonos a apartados cobrados hoy  [tabla]      │
│  Devoluciones/cancelaciones del día [tabla]    │
│                                                 │
│  Gastos del día:                  $100          │
│  Efectivo esperado:               $1,550        │
│  Efectivo contado:                $1,550        │
│  Tarjeta/transferencia contado:   $1,100        │
│  Total no efectivo (sistema):     $1,100        │
│  Total de ventas (sin fondo):     $2,250        │
│  Total con fondo:                 $2,650        │
│  ─────────────────────────────                  │
│  Diferencia Efectivo:             $0             │
│  Diferencia Total:                $0             │
│                                                 │
│  [ Cerrar e imprimir ]   [ Descargar PDF ]     │
└─────────────────────────────────────────────┘
```

---

## 7. Dashboard (Admin)

```
┌──────────────────────────────────────────────────────────────────────┐
│  Dashboard              Filtro: [Día ▾]  [Semana][Mes][Año][Rango...] │
├────────────┬────────────┬────────────┬────────────┬──────────────────┤
│ Ventas     │ N° tickets │ Ticket     │ Utilidad   │ Apartados         │
│ totales    │            │ promedio   │ bruta est. │ pendientes        │
│ $__ ▲6.46% │  __        │ $__        │ $__        │ __ / $__          │
├────────────┴────────────┴────────────┴────────────┴──────────────────┤
│  [Línea: ventas en el tiempo]         [Dona: métodos de pago]         │
│  [Barras: top 10 productos]           [Dona: productos vs. servicios] │
│  [Barras: ventas por vendedor]        [Dona: menudeo vs. mayoreo]     │
│  [Tabla: productos con stock bajo]    [Tabla: pendientes de revisión] │
│  [KPI: cancelaciones/devoluciones/reembolsos]  [KPI: precios pers.]  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 8. Editor de formato de ticket (Configuración)

```
┌───────────────────────────────────────────┬─────────────────────────┐
│  Formato del ticket                         │  Vista previa           │
│  Nombre de la tienda: [________________]    │  [Nombre de la tienda]  │
│  Dirección:           [________________]    │  [Dirección] · [Tel.]   │
│  Teléfono:            [________________]    │  ───────────────────    │
│  Logo:                [ Subir imagen ]      │  Folio: 24-09-2026-01   │
│  Mostrar QR:          [x]                   │  Fecha: 24/09/2026 14:32│
│  Leyenda del QR:      [Buscar como: {folio}]│  Vendedor: Juan         │
│                                              │  ───────────────────    │
│  [ Guardar ]                                │  ... partidas ...       │
│                                              │  Total          $810.00│
│                                              │  [QR]  Buscar como: ... │
└───────────────────────────────────────────┴─────────────────────────┘
```

---

## Notas

- Todas las pantallas usan el layout de dos columnas de `Design.md` (contenido ≈70% + panel lateral 320–380px) cuando hay una lista con detalle.
- Los botones primarios (Cobrar, Apartar, Confirmar, Guardar) van con el estilo `--brand-cta-bg` de `Design.md`; los destructivos (Cancelar venta) con el estilo `--danger`.
- El cierre de caja (sección 6) quedó como un flujo de **tres pasos**: gastos → conteo manual (efectivo + tarjeta/transferencia juntos) → resumen final con las dos diferencias. Ver `docs/reglas-corte-apartados-devoluciones.md` sección 1 para el porqué de cada diferencia.
- Estos wireframes no cubren estados de carga, vacíos ni de error — se agregan al construir cada pantalla (siguen el patrón de `Design.md` sección 5.9 para toasts).
