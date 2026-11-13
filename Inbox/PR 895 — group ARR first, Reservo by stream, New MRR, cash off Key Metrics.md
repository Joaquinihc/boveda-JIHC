---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 895
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/895
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-10
temas: [board, revenue, mrr, reportes, evidence]
creado: 2026-11-13
revisar: true
---

# PR 895 — Board Book y Flash Revenue: ARR del grupo primero, Reservo por stream, New MRR y caja fuera de Key Metrics

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/board-book-flash-group-arr` → `main`. Creado y mergeado el 10-nov (aprobado por nicofrem-agpro, «con observaciones»). Continúa el PR 892.

**Proyecto**: sin link (el PR no nombra proyecto). Tarea candidata por afinidad, sin mención explícita: [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111) — por eso `revisar: true`.

## Qué cambió
- **Board Book abre con el ARR del grupo**: lámina «Total ARR — AgendaPro + Reservo» después de Key Metrics, con ARR por stream (Reservo partido en software, Mercado Pago y Vambe), tabla mensual y tablas de crecimiento MoM y YoY para AgendaPro, Reservo y el grupo (as reported). 7c pasa a ser el puente de ARR AgendaPro + Reservo (como en Flash Revenue); 7d junta el total y los tres streams en 2 × 2 (absorbe la 7e).
- **Mercado Pago y Vambe de Reservo** salen de una extracción nueva, `reservo_revenue_shares_monthly` (P&L de Reservo en moneda local por país), convertidos con la misma tabla de FX. Flash Revenue los suma a todas las cifras de Reservo y lo parte por stream. Mientras el P&L de Reservo no cierre un mes, esas porciones van en blanco (Reservo = software) y los puentes de ARR las dejan fuera de la ventana para que el mes no se lea como caída.
- **Caja fuera de Key Metrics** (books mensual y trimestral): la tarjeta Cash pasa a **Net MRR Change (Constant FX)** —cambio de MRR de AgendaPro a FX constante contra el cambio del budget—; la 5.5 saca Cash on Bank y Runway. El saldo de caja queda solo en la 5.4 (Cash Flow & Runway). Cash Change (FCF sin la línea de M&A) se mantiene.
- **Lámina nueva 7b2 — New MRR**: MRR del primer mes de los merchants nuevos, solo software, AgendaPro + Reservo por compañía (Reservo desde oct-26) y cada compañía por país. AgendaPro = `new` del waterfall SaaS; Reservo = MRR del primer mes de clientes con primer pago en el mes (`reservo_new_merchants_monthly` suma columnas `new_mrr_usd*`). El subtítulo avisa que la promo 3x1 de Reservo entra a un tercio de la cuota.
- Menores: etiquetas «AgendaPro» en títulos de gráficos solo AgendaPro de Flash Revenue; prop `darkLabels` en `FxArrBridge`; etiquetas de Reservo sin «× 12».
- `docs/METRIC_DEFINITIONS.md`: entradas nuevas Net MRR Change (Constant FX) y New MRR — AgendaPro + Reservo. Tests `npm run test:sql` 532/532.
- Aviso del PR para el primer deploy: vigilar el shard de `reservo_revenue_shares_monthly` (nueva) y `reservo_new_merchants_monthly` (columnas nuevas); si se rellenan desde caché, la 7b2 o las porciones de Reservo fallan hasta la siguiente extracción.

## Por qué
- Que el Board Book parta por el ARR consolidado del grupo, con Reservo completo (no solo software), y sacar la caja de las láminas de KPIs.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/895

## Biblioteca que podría quedar desactualizada
- [[Definiciones KPIs del Board (DB KPIs)]] — la tarjeta Cash pasa a Net MRR Change (Constant FX) y la 5.5 ya no muestra Cash on Bank ni Runway.
- [[MRR - ARR (MRR Total, ARR Total)]] — ARR del grupo con Reservo por stream (software + Mercado Pago + Vambe).
- [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]] — dice que Flash Revenue suma solo software de Reservo; desde este PR suma también Mercado Pago y Vambe.
- [[New Merchants (nuevas altas)]] — New MRR de Reservo (primer mes, promo 3x1 a un tercio).
