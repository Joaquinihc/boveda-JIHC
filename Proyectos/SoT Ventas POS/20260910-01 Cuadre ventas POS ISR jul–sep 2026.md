---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-09-10
temas: [pos, ventas, reactivaciones]
metricas: []
---

# Cuadre ventas POS ISR jul–sep 2026

**Pregunta**: ¿Por qué el board book registra más ventas POS ISR / New en Chile (jul–sep 2026) que el listado de sedes con POS = Sí?

Tarea: [[Locations POS y reactivaciones]] (T-053). Referencia técnica del número: [[SQL — SoT Ventas POS#Dónde vive el número de ventas POS del Board Book]].

## Conclusión
> El +9 no es un error de suma. Son **+13 unidades que el board book cuenta de más** y **−4 que cuenta de menos**, cada una con causa identificada. Los company_id que coinciden calzan también en unidades.

## Hallazgo: el cuadre jul–sep 2026 (Chile, ISR / New), corte 10-sep-2026

Comparado contra el listado de sedes con POS = Sí. Cruce hecho company_id por company_id, no por totales.

| Mes | Listado (Σ Sedes) | Board book (SoT) | Dif |
|---|---|---|---|
| jul-2026 | 27 | 31 | +4 |
| ago-2026 | 44 | 47 | +3 |
| sep-2026 (parcial) | 7 | 9 | +2 |
| **Total** | **78** | **87** | **+9** |

El +9 no es un error de suma. Son **+13 unidades que el board book cuenta de más** y **−4 que cuenta de menos**, cada una con causa identificada. Los company_id que coinciden calzan también en unidades.

**Suman (+13)**

| Causa | u | company_id |
|---|---|---|
| Conversión arriendo → cuotas (mismo terminal) | 5 | 494549 (jul); 373415, 453116, 511752, 520667 (ago) |
| Venta real a cliente con POS previo | 3 | 416679 ago (2 u, acumulado de terminales 2→4); 362121 ago (1 u) |
| Venta revertida con factura pagada | 2 | 562659, 563154 (ago) — en el listado están como NO |
| Fila con Sedes = 0 y POS = Sí | 1 | 529200 (jul) — mismo patrón en 537052 y 533993, sin efecto neto |
| Segunda factura del mismo evento | 1 | 553259 (sep), emitida el 1-sep tras quitarse el addon el 25-ago |
| Venta posterior al corte del listado | 1 | 569917 (sep), 7-sep, pagada el 9-sep |

Las 5 conversiones son el hallazgo más limpio: Chargebee registra el mismo día `added smart-pos-cuotas` + `removed smart-pos-arriendo`. Es cambio de forma de pago, no equipo nuevo.

**Restan (−4)**

- **504406** y **535145** — el SoT los manda a Stock/KAM y son ISR. Son ventas **solo de setup** (`setup-smart-pos_2025-cl`, sin addon de arriendo ni cuotas), y en esas filas `pos_sales2` trae los tres campos de agente nulos. Sin agente y sin deal en el pipeline *Seguimiento POS Expansión*, la regla de canal agota los 4 niveles y cae en la ventana de recencia: 153 y 76 días desde la creación de la empresa, ambos sobre el umbral de 30 → KAM. La factura en `pos_sales` sí trae vendedor (Bárbara Sánchez) pero el SoT no lee ese campo, y HubSpot los marca *Ganado con POS* con barbara@.
- **569393** — agente mal atribuido. `pos_sales2` rellenó `pos_sale_agent` con alvaro@ vía `hubspot_owner_fallback`, que es el `kam_pos_owner` del contacto, no quien cerró. El deal ganado el día de la venta es pipeline **Sales ISR**, owner monika@, 31-ago. Como alvaro@ está en el roster KAM hardcodeado, cae en Stock. El chequeo de deals del SoT solo mira *Seguimiento POS Expansión*, así que nunca ve el Sales ISR.
- **524852** — no existe la venta. Cero filas en `pos_sales2`, cero facturas, cero addons POS en todo el histórico, cero en la planilla KAM. Único rastro: deal *Oportunidad propuesta* (abierto), naily@, cierre proyectado 27-sep.

**Corrimientos de mes (netos 0 en el trimestre)**

El SoT fecha por el cambio de addon, el listado por la factura: 554050 y 554347 (listado ago → SoT jul), 569925 (ago → sep), 570598 (sep → ago).

## Contraste con `dwh.pos_sales_sot` (24-sep-2026)

La tabla nueva de Ignacio ya existe en Redshift y resuelve parte del gap por diseño:

- **Separa `purchase_type`**: `new` / `pos_additional` / `reactivation`. Las 5 conversiones y 416679 caen en `pos_additional` — exactamente el bucket más grande de mi exceso.
- **Filtra con `is_valid_purchase` + `invalid_reason`**: la 2ª factura de 553259 sale como `invoice_not_paid;form_not_sent`.
- **Corrige 569393** → ISR.

Lo que **no** resuelve todavía:

- `channel_attribution` deja **1.338 de 2.288 filas en `Others`** (58%). 504406 y 535145 siguen sin vendedor, ahora como `Others` en vez de KAM. Para comparar contra el listado hay que sumar `ISR + Others`, no solo ISR.
- **Cobertura desde el 12-jun-2026** y 433 filas con `effective_sale_date` nulo. No reproduce historia, así que no sirve para reconstruir el board book hacia atrás.
- Las ventas revertidas (562659, 563154) siguen como `new` válidas.

Chile, unidades válidas por mes (`country='CL'`, ojo: es `CL`, no `Chile`):

| Mes | ISR new | Others new | ISR+Others new | Listado |
|---|---|---|---|---|
| jul-2026 | 23 | 4 | 27 | 27 |
| ago-2026 | 40 | 6 | 46 | 44 |
| sep-2026 | 18 | 2 | 20 | 7 (parcial) |

Julio calza exacto sumando `Others`. Es la pista más fuerte de que el listado y esta tabla ya hablan el mismo idioma, y que lo que falta es terminar la atribución de canal.

## Arreglos propuestos (sobre `country_pos_sot.sql`)

1. **Leer el agente desde `pos_sales.sales_user`** cuando `pos_sales2` lo trae nulo. Recupera 504406 y 535145 y cubre toda venta de solo setup a futuro.
2. **Ampliar el chequeo de deals al pipeline `Sales ISR`**, hoy solo mira *Seguimiento POS Expansión*. Corrige 569393.
3. **Definir con Ignacio si arriendo → cuotas cuenta como venta.** Son 5 de las 13 unidades de exceso. Se detecta buscando `added cuotas` + `removed arriendo` el mismo día para la misma empresa. Equivale a adoptar su `purchase_type = 'pos_additional'`.
4. **Resolver las facturas pagadas con addon removido** (562659, 563154) con Payments antes de tocar la query: puede ser que la venta ocurrió y lo que falla es el registro del addon.

A mediano plazo, lo 1–3 se reemplaza por consumir `dwh.pos_sales_sot` directo, cuando `channel_attribution` baje el `Others` y la tabla cubra historia.

## Corrección (24-sep-2026, perfil de las 17:31)
Dos datos de la sección «Contraste con `dwh.pos_sales_sot`» quedaron mal (el texto de arriba se conserva tal cual):
- La tabla **cubre toda la historia**: su primera fecha efectiva es el 12-jun-**2023**, no 2026. Sí sirve para reconstruir hacia atrás.
- `Others` es el **67% de todas las filas y el 28% en 2026**, no el 58%.

Julio sigue calzando en 27 unidades sumando ISR + Others. Detalle y perfil completo en [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]].

