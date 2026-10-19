---
type: analisis
proyecto: "[[_Ad hoc]]"
estado: activo
fecha: 2026-10-15
temas: [definiciones, cac, reportes]
metricas: ["[[Total Merchants (EOM stock)]]"]
---

# Total de comercios de Julio Guzmán (19.265 ago-26)

**Pregunta**: ¿Cómo llegó Julio Guzmán a 19.265 comercios a fin de agosto-26 en su gráfico «Total de comercios sube y CAC cae en agosto-26» (comercios EOM + CAC, Informasayuno 70, ~10-sep-2026)? Evidence da otra cifra.

> Borrador desde Claude Code (agendapro-dashboards-evidence, 15-oct-2026). Queda en Inbox hasta que Joaquín decida si va como análisis definitivo.

## Conclusión
> Corte: Redshift al 15-oct-2026.
> - **Definición de Julio**: venues activos de `dwh.mrr` (`is_active`, `SUM(venues)`) **menos las cuentas de Colombia que nunca registraron una reserva**. Es el filtro antifraude aplicado **solo a Colombia**.
> - Reproduce su serie a la unidad: dic-25 16.950, ene-26 17.381 (Julio 17.382), jul-26 19.108. **Ago-26 da hoy 19.271 vs 19.265**: el filtro es «reservó alguna vez», así que una cuenta sale de la exclusión cuando hace su primera reserva. 20 venues colombianos activos en agosto reservaron por primera vez en sep–oct, después de que Julio armó el gráfico.
> - **No es ninguna slide del Informasayuno**: la 10 no filtra (ago-26 19.827) y la 11 exige reserva en todos los países (18.791). Lo de Julio queda entre ambas.
> - **El CAC ($298) no se reprodujo exacto.** Con el mismo filtro de Colombia en los New Merchants calza en ene (252 vs 255) y may (276 vs 277), pero en jun–ago su CAC queda bajo el que da el P&L de hoy (ago: 298 vs 301). Probablemente usó el P&L de adquisición de comienzos de septiembre.

## Método
- Se leyeron los 68 puntos mensuales del gráfico desde los píxeles (error ±30 venues; ago 19.268 vs la etiqueta 19.265) y se compararon contra variantes de `dwh.mrr`.
- Pista inicial: en #new-monitor Julio reporta 1.216 merchants nuevos (venues) en ago-26, igual al New Merchants del waterfall de software. El denominador era el canónico; la diferencia estaba en el stock.
- Variantes descartadas (ago-26): `is_active_and_paid` 19.566 · compañías únicas 17.452 · sin Colombia 17.072 · waterfall de software acumulado 19.762 · `dwh.mrr_saas.merchants` 19.570 · Chargebee `active` sin `non_renewing` 19.215 (calza en agosto, pero mar-26 da 18.993 vs 18.169) · filtro de la planilla DB-KPIs (`first_paying_month` en «Otros») 19.808.
- La serie de Julio es igual a `is_active` hasta 2024. La brecha abre en 2025 y sigue la forma de la ola de fraude en Colombia: −401 en jul-25, −1.508 en mar-26, −559 en ago-26.
- Da lo mismo identificar Colombia por `int_company_dimensions.country_group`, por `currency_code = 'COP'` o por `companies.country_name`.

```sql
-- [Redshift] Total de comercios con la definición de Julio
WITH booked AS (
    SELECT company_id
    FROM dwh.fct_bookings_created_monthly
    GROUP BY 1
    HAVING SUM(bookings_count) > 0
)
SELECT m.date_month::date AS month, SUM(m.venues) AS comercios
FROM dwh.mrr m
JOIN dwh.int_company_dimensions cd ON cd.company_id = m.company_id
LEFT JOIN booked b ON b.company_id = m.company_id
WHERE m.is_active
  AND NOT (cd.country_group = 'Colombia' AND b.company_id IS NULL)
GROUP BY 1
ORDER BY 1;
```

## Evidencia
| Mes | Julio (gráfico) | Definición de Julio | Slide 10 (`is_active`) | Slide 11 (reserva, todos los países) |
|---|---|---|---|---|
| dic-25 | 16.950 | 16.950 | 17.582 | 16.658 |
| ene-26 | 17.382 | 17.381 | 18.194 | 17.080 |
| mar-26 | ~18.169* | 18.043 | 19.677 | 17.716 |
| jul-26 | 19.108 | 19.108 | 19.781 | 18.625 |
| ago-26 | 19.265 | 19.271 | 19.827 | 18.791 |

\* Lectura de píxeles en un tramo empinado de la línea; ahí el error es mayor.

## Caveats
- El filtro de reserva es «alguna vez» (lifetime), no al mes: las cifras de meses cerrados suben a medida que las cuentas reservan por primera vez.
- No se tuvo la planilla de Julio: la definición está inferida de la serie, no confirmada con él.
