---
type: metrica
fuente: skill finance-metric-definitions
temas: [unit-economics]
actualizado: 2026-09-22
---

# Run-rate

- **Definition**: An annualized/forward projection of a recent observed rate, not a closed actual.
- **Formula**: Two uses — (1) **ARR run-rate** = last-month MRR × 12; (2) **Forecast base** = trailing
  3m/6m average of each driver/cost, held flat and projected forward.
- **Grain**: Monthly; trailing 3m/6m windows.
- **Key rules / filters**: Forecast run-rate is anchored to the last **closed** P&L month.

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.
