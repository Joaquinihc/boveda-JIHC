---
type: tarea
estado: en-curso
prioridad: alta
tiempo:
tipo: definicion
origen: manual
proyecto:
temas:
  - definiciones
  - coordinacion
fuente: https://app.notion.com/p/400ab34c233b81c7847af71fd6cb9808
fecha_limite:
creado: 2026-10-22
revisar: false
id: T-099
---

# Definiciones grupo AgendaPro vs Reservo

Hoja de Notion «Definiciones grupo: AgendaPro vs Reservo» (Finance / FP&A) que [[Pablo Lucero]] armó el 21-oct-2026 tras la reunión FP&A grupo del 20-oct con Reservo. Objetivo: dejar por escrito las definiciones de negocio de cada empresa para empezar a trabajarlas en conjunto y, a largo plazo, converger. Pablo pidió revisar la columna AgendaPro, trabajarla con Christopher Lagos (datos, Reservo) y mandar un preview a #data-reservo.

## Subtareas
- [ ] Mandar preview de la hoja a #data-reservo (Pablo lo pidió el 21-oct, «ideal al final del día»). Listo cuando el mensaje esté publicado con el link.
- [ ] Sesión con Christopher para poblar la columna Reservo (clientes activos y MRR primero, waterfall después, pricing en reunión aparte). Listo cuando las filas de la columna Reservo dejen de decir «por completar» en los pasos 1 y 2.

## Notas

### 2026-10-22 · Reestructura de la columna AgendaPro
- Formato acordado con Christopher el 21-oct: cada celda explica primero el concepto de negocio y cierra con una línea gris «Medimos con:» (tabla y variable). Aplicado en Notion el 22-oct.
- Fuentes usadas para validar: skill `finance-metric-definitions`, modelos dbt en `origin/refactor/postgres-to-redshift` (`agendapro-dbt-redshift`, 19-oct), board book de Evidence (`pages/finance/board-book/index.md`), derivas de la bóveda.
- Cambios estructurales: se eliminan «Clientes activos (stock)» y «Revenue vs MRR»; «MRR» pasa a «MRR y revenue por stream» (revenue = MRR + cargos del mes: Charly en SaaS, setup en AI; Payments = fee POS + web + terminales); secciones nuevas Adquisición (Leads y conversión) y Uso del producto (Bookings, Uso de features); fila nueva Macro nicho antes de Vertical con el orden de fuentes override → IA → HubSpot → sector declarado.
- Decisiones de redacción de Joaquín: «logo» = company (el churn se dice merchant churn, en sucursales); fuente de la verdad de sucursales = `plan_quantity` de Chargebee vía `dwh.mrr.venues`; segmento explicado primero como 1 / 2 / 3+ profesionales y después la regla del primer plan; activación con umbrales por segmento, toda la vida con trial y sin reservas importadas; FX en tres bases (Current, Constant, Budget 2026).
- Leads: el board book cuenta cada contacto una vez por mes en que convierte o reconvierte (`dwh.fct_lead_conversions`), no solo la primera conversión. Ver [[Claude Code — Lead: el board book cuenta reconversiones]].
- Pregunta de Joaquín resuelta: el New Merchants del board book no calza exactamente con el `new` del waterfall software (brecha 1–5 %, se cierra en 2026). Ver [[20261022-01 New Merchants board book vs new del waterfall software]].
- Referencia técnica de `venues`, `is_active` e `is_active_and_paid`: [[dwh.mrr — venues, is_active e is_active_and_paid]].

### Pendientes menores
- 2026-10-22 · Actualizar la card Lead de la skill `finance-metric-definitions`: hoy dice «1 lead = 1 contacto por su primera conversión»; el board book usa reconversiones.
- 2026-10-22 · Corregir en la hoja de Notion «Definición verticales y categorización en Macro Nichos» el párrafo de «Contexto» que pone la IA antes que el override (el modelo dbt aplica override primero).
- 2026-10-22 · «Clientes nuevos» suma el `plan_quantity` vigente de `dwh.merchants_segments`, no el del mes del alta; quedó así a propósito, tenerlo presente cuando Reservo pregunte.
- 2026-10-22 · El checkout local de `refactor/postgres-to-redshift` en `agendapro-dbt-redshift` está al 7-sep; `origin` va al 19-oct. Validar siempre contra `origin`.
- 2026-10-22 · Métricas propuestas para sumar a la hoja más adelante (orden de relevancia): ARR y New MRR, NRR, reactivaciones/upgrades/downgrades, CAC y Payback, activación a Mes 3, GMV, Payments (GPV, take rate, POS vendidos, attachment, activación POS), AI (ARR por vertical, companies únicas, ASP), NPS y Soporte, financieros (P&L, EBITDA, Rule of 40, caja), market share, marketplace.
- 2026-10-29 · El P&L de Reservo en Odoo todavía no es comparable con el de AgendaPro (sin nómina 2026 ni cost center; cuentas cajón). Seguimiento en [[Revisar P&L de Reservo gasto por gasto (cuentas Odoo)]].

## Historial
- 2026-10-22 · Creada desde Claude Code: columna AgendaPro reestructurada y aplicada en Notion; quedan el preview a #data-reservo y la columna Reservo.
- 2026-10-23 · Mencionada de nuevo en [[2026-10-21 Definiciones e infraestructura de datos de Reservo (Christopher)]] (separar definiciones comerciales de técnicas)
- 2026-10-23 · Mencionada de nuevo en [[2026-10-22 Data AgendaPro — arquitectura de datos para Reservo]] (compartir el Notion de definiciones con Reservo)
