# Ventas & CxC — Guía de puesta en marcha

Sistema de Ventas y Cuentas por Cobrar (estilo Microsip, sin timbrado fiscal),
separado de tu sistema de inventario. Un solo archivo HTML + Supabase, mismo
patrón que tus otras herramientas.

## 1. Crear el proyecto en Supabase

1. Ve a https://supabase.com → crea un proyecto nuevo (puede ser gratuito).
2. En **SQL Editor**, pega y ejecuta todo el contenido de `schema.sql`.
3. Ese script crea las tablas, la vista de antigüedad de saldos, los triggers
   que mantienen los saldos actualizados automáticamente, y las políticas de
   seguridad (RLS) para que cada vendedor solo vea sus propios clientes.

## 2. Crear usuarios (admin y vendedores)

1. En Supabase → **Authentication → Users**, crea una cuenta por cada persona
   (correo + contraseña). Tú (admin) y cada vendedor de campo.
2. En **Table Editor → usuarios**, agrega una fila por cada persona:
   - `id`: copia el UUID que Supabase generó en Authentication → Users
   - `nombre`: nombre visible
   - `rol`: `admin` o `vendedor`
   - `activo`: true

Sin este paso, la persona puede iniciar sesión pero la app le dirá que no
tiene perfil asignado.

## 3. Conectar la app a tu proyecto

Abre `index.html` y edita estas dos líneas cerca del inicio del `<script>`:

```js
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU_ANON_KEY';
```

Ambos valores están en Supabase → **Settings → API**.

## 4. Desplegar

Igual que tu calendario y boletín: arrastra la carpeta (o solo `index.html`)
a Netlify (netlify.com/drop), o súbelo a un repo de GitHub conectado a Vercel.
No requiere build ni instalación de paquetes.

## 5. Cargar catálogo inicial

Ya dentro de la app (como admin):
1. **Artículos** → da de alta tus productos y servicios (código, precio MXN/USD).
2. **Clientes** → da de alta tus clientes, asigna vendedor, límite y días de crédito.
3. Los vendedores, al entrar con su propia cuenta, solo verán y podrán
   capturar ventas/cobros de los clientes que tú les asignaste.

## Cómo funcionan los Anticipos (saldo a favor)

No hay una pantalla separada de "crear anticipo": es automático.

- Si registras un **cobro** y el cliente no tiene facturas pendientes (o el
  importe es mayor a lo que debe), el sobrante se queda guardado como
  **anticipo disponible** — no se pierde ni hay que hacer nada extra.
- En la pestaña **Anticipos** ves todos los saldos a favor por cliente.
  Cuando ya exista la factura a cubrir, entras y das **"Aplicar a factura"**:
  eliges la venta y el importe, y el saldo de esa factura baja igual que con
  un cobro normal.
- Un mismo anticipo se puede aplicar a varias facturas distintas, en
  distintos momentos, hasta agotar su saldo disponible.

## Cómo funciona Pedidos (con surtido parcial)

1. El vendedor levanta un **Pedido** con las partidas que el cliente quiere.
2. Cuando hay mercancía disponible (aunque sea parte), se abre el pedido y se
   pulsa **"Surtir"**: se captura cuánto se entrega de cada partida.
3. Al confirmar, el sistema automáticamente:
   - Genera una **venta** (factura interna) solo con lo que se está entregando.
   - Descuenta la **existencia** de los artículos que tienen control de inventario.
   - Marca el pedido como **"parcial"** (si aún falta algo) o **"surtido"**
     (cuando ya se entregó todo).
4. Puedes volver a surtir el mismo pedido varias veces hasta completarlo —
   cada entrega genera su propia venta y su propio saldo por cobrar.

## Cómo funciona el saldo automáticamente

- Al crear una **venta**, el saldo inicial = total de la factura.
- Al registrar un **cobro**, la app aplica el importe a las facturas más
  antiguas primero (como Microsip) y el saldo de cada factura baja solo —
  hay un trigger en la base de datos que lo hace, así que nunca se
  desincroniza aunque captures desde varios dispositivos a la vez.
- Una factura pasa a **"pagada"** automáticamente cuando su saldo llega a cero.
- El reporte de **CxC / Antigüedad** clasifica cada factura vigente en
  0-30 / 31-60 / 61-90 / 90+ días según su fecha de vencimiento
  (fecha de la venta + días de crédito).

## Qué falta / próximos pasos sugeridos

- **Estado de cuenta imprimible por cliente** (PDF/WhatsApp) — puedo
  agregarlo en una siguiente iteración, similar al PDF del panel admin
  de tu sitio de la iglesia.
- **Notas de crédito/cargo** desde la interfaz (la tabla y lógica ya están
  en la base de datos, falta el formulario en la app).
- **Tipo de cambio automático** (hoy se captura manual al hacer venta/cobro
  en USD; se puede conectar a una API de tipo de cambio).
- **Edición/cancelación de ventas y pedidos** ya creados.
- **Cancelar pedido** desde la interfaz (la columna de estatus ya lo soporta).
- **Reportes por vendedor** para ti como admin (ranking de cobranza, pedidos
  surtidos vs. pendientes, etc.)

Dime cuál de estos quieres primero y lo agrego.
