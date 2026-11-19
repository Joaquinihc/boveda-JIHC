---
type: analisis
proyecto:
estado: activo
fecha: 2026-11-19
temas: [odoo, evidence, reportes, definiciones]
metricas: []
---

# Cash flow Odoo v2 vs Evidence — comparación ene–oct 2026 y auditoría

**Pregunta**: ¿qué cambió [[Pablo Lucero]] en el cash flow de Odoo (versión 2, en producción desde el 17-nov-2026), en qué difiere del cash flow que publica Evidence (Board Book 5.4 y Financial Performance, que leen la planilla de tesorería) y está bien lo que quedó en Odoo?

Tarea: [[Cash flow en Odoo]]. Detalle completo en Notion: [Cash Flow en Odoo](https://app.notion.com/p/41dab34c233b80fba44fc2690deecf01) (página principal y 7 subhojas). Método reutilizable: [[Cash flow en Odoo (v2) — estructura, mapa a Evidence y cómo reproducir]].

## Conclusión
> **Foto**: Odoo leído el 19-nov-2026 entre 11:51 y 12:45; `finance.cash_flow` el 19-nov a las 11:52. USD a la tasa de la planilla.
> 1. **El reporte de Odoo está bien construido.** El export es igual a la pantalla, los saldos cuadran con la contabilidad y ninguna cuenta tiene dos etiquetas. Lo que falla son datos contables que el reporte ahora deja a la vista.
> 2. **El Free cash flow no es el mismo número.** Ene–oct 2026: Evidence −2.359 mil USD, Odoo −1.830 mil. De los 529 mil de diferencia, 409 son definición, 225 netos son datos por corregir (109 por confirmar) y −105 es el resto (perímetro, corte, clasificación y 58 sin explicar).
> 3. **Mes a mes Odoo salta entre −670 y +399 mil USD; Evidence se mueve entre −154 y −292.** Tres causas: dos nóminas de Chile que caen al mes siguiente, el asiento mensual «Ajuste saldos dlocal» de Chile en el primer semestre y la suspense sin conciliar de SC3 en octubre.
> 4. **La caja de cierre se parece.** 31-oct: Evidence 10.428 mil USD; Odoo 10.212 operacional + 131 Fintech = 10.343.
> 5. **AgendaPro Inc no tiene tasas de cambio en Odoo antes del 9-jul-2026**: el consolidado en USD de Odoo no sirve para ene–jun.
> 6. **Evidence sigue leyendo la planilla.** Si deja de actualizarse, el cash flow del Board Book se congela en octubre.
>
> Nada está decidido: las definiciones (D1–D6 de la hoja 2 de Notion) esperan a Pablo y las correcciones contables (C1–C19 de la hoja 7) son de Contabilidad. Al 19-nov no se les ha enviado.

## Método
- **De abajo hacia arriba**: cada sociedad en moneda local, línea por línea, y encima el consolidado en USD con un puente. Es el método acordado el 26 y 29-oct y el que usó Pablo para Chile jul–oct.
- **Odoo**: export de Rule 91 (`agent_cash_flow_export`) por sociedad, mes, línea y cuenta, más lecturas directas por XML-RPC para auditar el export. Solo lectura.
- **Evidence**: `finance.cash_flow` con las consultas de extracción de `origin/main` y el bloque `fp_cash_flow_table` ejecutado tal cual en DuckDB. Reproduce octubre: caja 10.428.232 y FCF −154.247.
- **Causa de cada diferencia**: asignada por regla (sociedad, línea, mes) a definición, perímetro, corte, clasificación, dato o sin explicar. 64 celdas quedaron «por confirmar».
- **Umbral**: una línea calza si difiere menos de 0,1 M en moneda local o 1 mil USD en el mes.
- Revisión independiente con contexto fresco sobre unas 750 cifras; 9 no calzaban y se corrigieron las importantes.
- Archivos: `~/Documents/GitHub/_analisis/2026-11-cash-flow-odoo-v2/` (`diseno.md`, `plan.md`, `progress.md`, `scripts/`, `datos/`, `salidas/`).

## Evidencia
**Puente del Free cash flow, ene–oct 2026** (miles USD; `salidas/puente_fcf.csv`):

| Paso | Monto |
|---|---:|
| FCF Evidence (Δ caja − Loan − ESOP − M&A − caja adquirida) | −2.359 |
| Fuera del FCF en Odoo: financing repayment 255, otras líneas de Finance 64, tipo de cambio 12 | +331 |
| Variación del saldo en pasarelas | −38 |
| Cobro bruto al entrar a la pasarela y otras diferencias de definición | +116 |
| Dato por corregir (SC3 +452, Chile −277, México +107, Inc −53, Colombia −5) | +225 |
| Perímetro | −59 |
| Corte | +24 |
| Clasificación | +11 |
| Sin explicar | −58 |
| Bajo el umbral | −23 |
| **FCF Odoo v2 (Cash from Operations)** | **−1.830** |

**FCF mensual** (miles USD):

| | Ene | Feb | Mar | Abr | May | Jun | Jul | Ago | Sep | Oct |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Evidence | −254 | −258 | −284 | −190 | −208 | −276 | −195 | −248 | −292 | −154 |
| Odoo v2 | −554 | −139 | −670 | −137 | −132 | 392 | −508 | −260 | −221 | 399 |

**Caja operacional al 31-oct** (miles USD; Odoo menos Evidence = −216):

| Partida | Dif. |
|---|---:|
| Working Capital Propay (pseudo-cuenta de la planilla; en Odoo es caja Fintech) | −134,2 |
| dLocal Colombia (171,6 M COP en ambos lados; no es caja operacional en Odoo) | −54,3 |
| Fondo y CDS de Chase (sin cartola no se sabe qué lado está bien) | −51,8 |
| dLocal CLP Chile (60,84 M en Odoo vs 27,96 M; falta el ajuste de octubre de 32,9 M) | +33,1 |
| Stripe y Treasury de Inc | −5,3 |
| dLocal USD (XB) | −4,4 |
| FCI Argentina | +1,8 |
| Fondo BBVA Colombia contado dos veces en Evidence | −0,8 |
| Santander 95853544 | +0,5 |
| SC3 | −0,4 |

**Chile, primer semestre** (M CLP):
- Asientos «Ajuste saldos dlocal» contra la cuenta 1.0.01.08, con el signo del reporte: +257,4 ene, +225,2 feb, +55,8 mar, −1,7 abr, −32,4 may, −319,1 jun. Dejan los cobros de clientes de ene–feb fuera de Software (línea Software: −35,2 en enero, +22,5 en febrero) y 185,3 netos en Working capital – Propay.
- Nóminas que caen al mes siguiente: febrero → marzo 303,3 y junio → julio 291,6. Por eso «julio a octubre cuadra» no es cierto para julio.
- Intercompany feb → mar 148; fondeo Propay de agosto 92,3.

**SC3, octubre** (M CLP): los abonos de la cuenta suspense sin conciliar inflan Software en 440,7 y Other queda en −452,5; la nómina (77,9) está sin conciliar. 554 líneas de cartola sin conciliar en el año (403 de tarjetas). SC3 y Reservo México no tienen fecha de bloqueo.

**Controles de la auditoría**:
- Export = pantalla: 450 de 450 celdas en 6 sociedades y 405 de 405 en México (marzo sin respuesta de Odoo).
- 0 cuentas con dos etiquetas `CFM:`; reportes 47 y 48 alineados con el YAML de `main`.
- Other fuera de tolerancia: Chile abril 0,4 M (cuenta 105.01.088), Colombia ene–feb −34,7 M COP (37101001) y SC3 en los diez meses (−489,4 M).
- Intercompany: residuo de −80 mil USD que no se compensa.
- De los 14 pendientes del documento de Pablo, solo PayPal está resuelto.
- Octubre a tasa de Odoo: 10.210.734 operacional y 10.341.634 total. Pablo documentó 10.224.069 y 10.354.989; la diferencia es el arreglo de PayPal (16,25 → 3,0 M CLP).

**Evidence con Reservo**: 474 filas de saldo repetidas en 2026 (filtradas en la extracción), fondo BBVA Colombia contado dos veces (773 USD), el modelo dbt `cash_flow.sql` no publica la sociedad, Capex siempre en 0 y POS Purchase en Cash from Finance.

## Caveats
- **Odoo cambia a diario** ([[Nicolás Astudillo]] y Rafael siguen ajustando). Antes de reutilizar una cifra, volver a sacar la foto.
- No se revisaron cartolas ni glosas: lo que depende de una cartola quedó como pendiente de Contabilidad.
- Marzo de México no se comprobó contra la pantalla (Odoo no respondió en 40 minutos).
- 64 causas «por confirmar» (109 mil USD dentro de «dato») y 58 mil USD sin explicar; los proveedores de SC3 de octubre están entre lo no explicado.
- SC3 y Reservo México se compararon solo en sep–oct, los únicos meses que tiene la planilla.
- Las cuentas en USD de Chile, Argentina, México y SC3 se llevaron a moneda local con la tasa implícita del mes de la planilla.
- 2025 no se comparó: los bancos de Chile y Argentina se llevaron con asientos resumen hasta nov-2025.
- Detalles menores sin corregir en Notion: anotados al final de `progress.md`.
