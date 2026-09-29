---
type: metrica
fuente: skill finance-metric-definitions
temas: [adquisicion]
actualizado: 2026-09-22
---

# CVR / tasa de conversión

- **Definition**: `CVR = New Merchants ÷ Leads del período` — the **same** ratio as the acquisition
  Funnel. Not a new metric.
- **Formula**: `CVR = SUM(new_merchants) / NULLIF(SUM(leads), 0)` — a **same-period** ratio.
- **Grain**: **Aggregate only** — total / country / month. Exclude the current (partial) month.
- **Key rules / filters**:
  - **Valid only in aggregate — never per rep** (two-owner attribution). Show leads and New Merchants as
    separate columns in rep rankings; no per-person CVR.
  - A same-period ratio makes the latest months read low while cohorts are still converting —
    maturation, not a bug.
  - The old HubSpot lifecycle-"customer" / COR / 90-day cohort model is **retired**.
- **Source tables**: New-Merchant sources (numerator) + lead sources (denominator), above.

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.
