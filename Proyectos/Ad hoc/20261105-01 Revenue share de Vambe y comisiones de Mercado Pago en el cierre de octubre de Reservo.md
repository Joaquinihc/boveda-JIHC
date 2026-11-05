---
type: analisis
proyecto:
estado: activo
fecha: 2026-11-05
temas: [revenue, cierre-mensual, payments, definiciones]
metricas: []
---

# Revenue share de Vambe y comisiones de Mercado Pago en el cierre de octubre de Reservo

**Pregunta**: ¿cómo se reconocen en el P&L los dos flujos de revenue de Reservo que no salen de los pagos de suscripción (el revenue share de la alianza con Vambe y las comisiones de la pasarela Mercado Pago), y qué número va en la planilla de conciliación «DB - MRR & fees - DWH - Conciliación tabla» (hoja Reservo, fila 202610) para el cierre de octubre?

## Conclusión
> **Vambe**: Reservo le factura a Vambe cada mes el 25 % de la suma de los planes Vambe de los clientes de la alianza activos al día 1 del mes siguiente. La factura sale de SC3 SpA en CLP (USD × dólar del día), el 25 % es el total con IVA incluido y el neto es dividir por 1,19. Se reconoce por mes de servicio («Mes al que corresponde la factura»). Las consultas mensuales de Vambe que administra [[Sebastián Cruzat]] reproducen al centavo las facturas de la planilla de [[Pablo Saintard]] en 6 de 7 meses con archivo. **No existe aún la factura de octubre** (sale a mediados de noviembre con la foto al 01-11): la fila 202610 quedó con el neto de septiembre, 14.853.228 CLP (factura 129780, 17.981,75 USD a 982,96), como proxy y marcado para reemplazar. Subestima octubre en un 3–8 %.
> **Mercado Pago**: solo Chile, fee por transacción y venta de bono Fonasa, 100 % plata de Reservo que nunca se retiró de la cuenta (saldo a retirar en cero). La variación del saldo total entre las cartas de saldo del 30-sep y del 31-oct es **3.568.795 CLP**, aproximación de caja del revenue de pagos de octubre. Queda abierto si ese monto es bruto o neto de IVA; la columna M3 sigue en cero hasta decidirlo.

## Método
**Vambe.** Fuentes: planilla «Clientes Reservo Vambe» de Pablo Saintard (hoja `facturas`: número, fecha, mes de servicio, USD, dólar, monto CLP, monto sin IVA, fecha de pago; hoja `clientes`: foto actual sin historia) y la carpeta de Drive de Sebastián Cruzat «Query Vambe - Reservo (Mes)» (una consulta por mes con ID Vambe, email, primer pago, plan, estado al día 1, churn y UUID de la cuenta Reservo). Regla de lectura: un archivo con estado «Activa al 01-MM» alimenta la factura del mes MM−1. Cada archivo lleva un encabezado «MRR Activo»; en el «Cierre Septiembre» el encabezado quedó viejo (67.891) y lo que cuadra es la suma de filas activas (70.073).

**Mercado Pago.** Cartas de saldo de la cuenta de SC3 SpA (CUST_ID 2657785720) al cierre de cada mes. Se usa el **saldo total** (incluye el «a liberar», que son ventas ya hechas), no el disponible. Es caja, no devengo: mezcla fees de octubre con liberaciones de septiembre. [[Sebastián Concha]] confirmó el 5-nov por DM que es solo Chile, sin fee en otros países, y que falta emitir la factura a cada cliente por lo cobrado.

**Planilla de conciliación.** Criterio único: la columna de moneda local es la moneda en la que Reservo recibe el dinero. Software por moneda de pago (G3:K3, cargado el 4-nov desde `aportes_por_moneda`); Vambe 100 % CLP (S3) aunque haya clientes mexicanos, argentinos, colombianos y españoles en la alianza, porque Vambe paga una sola factura chilena; Mercado Pago en CLP (M3). La hoja mapea Vambe a la columna «AI» del bloque USD.

## Evidencia
**Cruce consultas de Cruzat vs facturas** (25 % de la suma de planes activos vs USD de la factura del mes de servicio):

| Archivo | Estado al | Activas | Planes (USD) | 25 % | Factura (mes) | USD factura | Dif. |
|---|---|---|---|---|---|---|---|
| Query Febrero | 01-02 | 70 | 33.185 | 8.296,25 | 114497 (ene) | 8.439,75 | −143,50 |
| Query Abril | 01-04 | 81 | 42.298 | 10.574,50 | 119369 (mar) | 10.905,00 | −330,50 |
| Query Mayo | 01-05 | 92 | 50.047 | 12.511,75 | 119922 (abr) | 12.511,75 | 0 |
| Query Junio | 01-06 | 94 | 54.906 + 2.870 adicional | 14.444,00 | 119984 (may) | 14.444,00 | 0 |
| Query Julio | 01-07 | 94 | 60.713 | 15.178,25 | 120517 (jun) | 15.178,25 | 0 |
| Query Agosto | 01-08 | 96 | 69.905 | 17.476,25 | 128781 (jul) | 17.476,25 | 0 |
| Cierre Septiembre | 01-09 | 97 | 70.073 | 17.518,25 | 129393 (ago) | 17.518,25 | 0 |
| Query Septiembre | 01-10 | 102 | 71.927 | 17.981,75 | 129780 (sep) | 17.981,75 | 0 |

Las diferencias de enero y marzo (574 y 1.322 USD de planes) parecen ajustes del tipo «adicional mes anterior», explícito solo en junio. Faltan archivos para febrero y octubre. La planilla de enero («Clientes Activos Reservo», 24.722 USD) es otra consulta y no cuadra con ninguna factura.

**IVA.** El USD de la factura es exactamente el 25 % de los planes y Vambe pagó exactamente ese monto convertido (columna «Monto pagado» = «monto», p. ej. 16.181.784 en la de julio). Si el IVA fuera encima, el pago sería un 19 % mayor. Pablo Saintard dijo el 4-nov que la factura incluye IVA. Se cierra con los dos abonos de VAMBE SpA de octubre en Odoo (SC3 SpA, BICE 24-00071-0): si son 16.181.784 y 16.774.250, el 25 % incluye IVA y S3 está bien; si son un 19 % mayores, el neto de septiembre sería 17.675.341.

**Serie de facturas 2026 (mes de servicio, USD, CLP con IVA, neto)**: ene 8.439,75 / 7.462.511 / 6.271.018 · feb 9.212,50 / 8.509.310 / 7.150.681 · mar 10.905 / 9.556.270 / 8.030.479 · abr 12.511,75 / 11.176.121 / 9.391.698 · may 14.444 / 12.805.039 / 10.760.537 · jun 15.178,25 / 14.346.027 / 12.055.485 · jul 17.476,25 / 16.181.784 / 13.598.138 · ago 17.518,25 / 16.774.250 / 14.096.008 · sep 17.981,75 / 17.675.341 / 14.853.228. Los pagos llegan con uno a tres meses de rezago (enero se pagó el 3-jun; julio y agosto cayeron en octubre), lo que explica los meses sin abono en Odoo.

**Proxy de octubre.** Foto de Cruzat al 01-10: 71.927 USD de planes. Foto actual de la hoja `clientes` de Pablo (5-nov): 192 activos en Vambe de 256, 77.660 USD de planes, 25 % = 19.415 USD. La factura de octubre debería quedar entre 18.500 y 19.400 USD brutos.

**Mercado Pago, cartas de saldo SC3 SpA (CLP)**:

| | 30-sep-26 | 31-oct-26 | Variación |
|---|---|---|---|
| Saldo total | 15.430.639,59 | 18.999.434,59 | 3.568.795,00 |
| Saldo disponible | 14.190.543,59 | 17.939.294,59 | 3.748.751,00 |
| Saldo a liberar | 1.240.096,00 | 1.059.491,00 | −180.605,00 |
| Saldo comprometido | 0 | 649,00 | 649,00 |

Al dólar de la hoja (980,17) son 3.641 USD brutos; si el fee lleva IVA dentro, 2.998.988 CLP netos. Pablo Saintard hablaba de «unos 20 M» acumulados; al 31-oct son 19,0 M.

**Clientes de la alianza en los pagos de Reservo** (cruce por nombre de cadena entre la hoja `clientes` de Pablo y `clientes_activos` del libro `pagos_mensuales_aplanados (6).xlsx`; 176 de 256 filas calzan con 148 cadenas, 204 cuentas; las 80 sin match son sucursales con otro nombre):

| Activo en Vambe | Pagos a Reservo | Cadenas | Plan activo (USD/mes) |
|---|---|---|---|
| Sí | pagan en 2026 | 95 | 43.527 |
| Sí | nunca pagaron | 16 | 10.954 |
| Sí | pagaron antes, nada en 2026 | 4 | 1.813 |
| No | pagan en 2026 | 25 | 0 |
| No | pagaron antes o nunca | 8 | 0 |

Confirma los dos casos que describió Pablo Saintard: venta Reservo más revenue share (mayoría) y cliente derivado de Vambe que paga cero a Reservo (16 cadenas, ~2.700 USD/mes de share, 14 % del total). El revenue share no está en `detalle_pagos`, así que cargarlo aparte no duplica. Las 148 cadenas pagaron 150.476 USD en 2026, ~6 % del software de Reservo.

**País de los clientes de la alianza** (inferido del formato del RUT/RFC/CUIT/NIT; no hay columna de país): Chile 232 clientes (173 activos, 17.204 USD de share), México 15 (11, 1.281), Argentina 4 (4, 617), Colombia 3 (3, 225), España 2 (1, 89). Dos dudas: las once sedes de «Vivir en Pareja» calzan con una cadena con país «Otro» en Reservo, y «Beauty Massage» tiene RUT colombiano pero cadena chilena.

## Caveats
- El libro de Reservo no marca quién viene de Vambe ni trae RUT, correo o UUID: el cruce es por nombre y es imperfecto. Pedido a Christopher: flag de alianza y plan Vambe por `cliente_id` en el DW.
- La «foto al día 1» no es el promedio del mes: un cliente que entra el día 2 no paga share ese mes.
- La columna «Plan 3 Mes» de la hoja `clientes` no se usa (promoción antigua, confirmó Cruzat el 5-nov). Los 192 activos de Pablo son cuentas Reservo; los 102 de Cruzat son clientes Vambe (una por empresa).
- Mercado Pago: solo hay cartas de sep y oct; para la historia pro forma hacen falta las anteriores o el módulo de reportes al que Sebastián Concha invitó a Joaquín el 5-nov.
- Fuentes de conversación: #data-reservo (4 y 5-nov, hilo de Pablo Saintard del 5-nov 09:55 con Sebastián Cruzat) y DM con Sebastián Concha (5-nov). Carpeta Drive de Cruzat: `drive.google.com/drive/folders/1Ssc9goOLcCaxcLRWo06UTTHTmmjjZZAK` (solo lectura). Planilla de conciliación: Google Sheet de Julio Guzmán `1YopV0kc8t5-GT9shGZrIYdy6EQHI-pvbvKYdbiN1BzQ`, hoja Reservo.
- Relacionado: [[20261103-01 New Merchants y Merchants de Reservo — definiciones finales y cuadre con Pablo Saintard]] y [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]].
