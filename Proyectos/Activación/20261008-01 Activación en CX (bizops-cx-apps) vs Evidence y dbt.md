---
type: analisis
proyecto: "[[_Activación]]"
estado: activo
fecha: 2026-10-08
temas: [activacion, graduacion, definiciones, evidence, dwh]
metricas: ["[[Activation]]", "[[Activation Velocity]]"]
---

# Activación en CX (bizops-cx-apps) vs Evidence y dbt

**Pregunta**: ¿Cómo define la activación CX en `bizops-cx-apps` y en qué se diferencia de Evidence y de dbt?

## Conclusión
> - **CX tiene cuatro definiciones de activación, no una.** La que más se ve es la del directorio (@M4), en el panel ejecutivo de CX y en el reporte del comité. Las otras tres: la de onboarding y comisiones (umbral por nicho y tamaño), una "activación de calidad" ya retirada de las pantallas, y la graduación.
> - **La del directorio no se compara con la nuestra.** Usa otra ventana (meses calendario 0 a 4 desde el primer pago, unos 120–150 días, contra Semana 9 / Mes 3), cuenta **cuentas** y no sedes, y suma **cobros**: la actividad es el máximo entre reservas efectivas y cobros pagados. Además no filtra cuentas test ni reservas importadas.
> - **Contradicción**: el código de CX dice que activar "la pone Finanzas y vive en `int_company_activation_status`: reservas acumuladas y nada más", pero su activación del directorio no calcula eso.
> - **Error en dbt que afecta a Evidence y a CX**: `booking_count_active` (modelo `bookings_incremental`) usa los estados 1, 2, 3, 7 y 9, pero el 9 no existe y Pendiente es el 8. Quedan fuera ~1% de reservas pendientes en `company_sales_days` (Evidence) y en CX. No afecta al fact dbt, que cuenta todos los estados.

## Método
- Lectura del código de `agendapro/bizops-cx-apps`, rama `main`, commit `d1a7d926` (8-oct-2026). No se midió con datos; las cifras de abajo son las que CX dejó escritas en su código.
- Archivos: `apps/cx-head-app/data_layer.py` (`sql_activation_dates_cte`, cohorte @M4), `_shared/ceo_report/cohorts.py` (`sql_cohort_activation`, `HEADLINE_MONTH_INDEX = 4`), `_shared/onboarding_core.py` (`is_activated`, `umbral_for`), `_shared/booking_pipeline_metrics.py` (`actividad_efectiva`, cobros), `_shared/fitness_bookings.py` (reservas efectivas y clases) y `_shared/graduacion_cx.py`.

## Evidencia
**Activación del directorio (@M4) frente a Evidence y dbt**

| | Evidence | dbt (`int_company_activation_status`) | CX directorio @M4 |
|---|---|---|---|
| Reservas | `company_sales_days` (solo activas) | Fact dbt (todos los estados) | `dwh.bookings` directo: `booking_count_active` sin fake ("efectivas" desde el 23-sep-2026), sin los filtros de pago ni de outliers de `company_sales_days` |
| Cobros | No | No | Sí: máximo entre reservas efectivas y cobros pagados (`staging.stg_agendapro__payments`, `is_payed`), desde el 28-sep-2026. Máximo y no suma, porque una cita cobrada deja los dos registros |
| Clases | Día de la clase, sin canceladas ni no-show | Creación, solo orígenes de demanda | Igual que Evidence: [[Pablo Lucero]] copió en el PR 769 la definición de CX |
| Importadas | Las cuenta | Las excluye (PR 415) | Las cuenta |
| Universo | New Merchants B2B3, sin cuentas test | Todas las empresas | `merchants_segments.segment2 = 'B2B3'` con primer mes pagado en `dwh.mrr`; no filtra cuentas test |
| Día 0 | `start_date_pago_mx` | Toda la vida, trial incluido | `merchants_segments.start_date_pago` |
| Ventana | Semana 9 (63 días) / Mes 3 (90 días) | Sin ventana | Meses calendario 0 a 4 desde el mes del primer pago. Una cohorte cuenta solo cuando cerró su mes 4. Hasta el 28-sep cortaba en el mes 3 |
| Unidad | Sedes | Empresa | Cuentas |
| Umbral | 100 (con selector) | 20 / 40 / 100 por segmento | 100 fijo (el universo es solo B2B3) |

**Otras definiciones de CX**
- **Onboarding y comisiones** (`onboarding_core.is_activated`, "canónica 2026-07"; la usan Onboarding TL, CS Comisiones y las apps de onboarders). Activa quien supera un umbral **por nicho y tamaño**, que sale de la tabla del motor de comisiones (Salón de Belleza 6+ = 175, Beauty 6+ = 555), **y** además tiene 2 ciclos pagados. En planes que se pagan por más de 2 meses, basta con 5 semanas. Las cuentas con 1–2 profesionales activos y 2+ ciclos activan con 70 reservas. La actividad también es el máximo entre reservas y cobros. Las ventanas de 16/25 semanas son de comisión, no de activación.
- **"Activación de calidad"**: 100 reservas + 3 ciclos pagados (`activated_paid`). Salió de las pantallas principales, pero la constante sigue en el código.
- **Graduación de CX** (`graduacion_cx.py`): 100 reservas + 4 ciclos pagados + un mes con reservas en sus 4 semanas, todo a la vez y en cualquier momento. Equivale a la vía objetiva de la graduación v2 en dbt. Sin cobros.

**Cifras que CX dejó en su código** (no las validé)
- Reservas efectivas (23-sep-2026): la activación @M4 de las cohortes ene-2025 → may-2026 bajó de 4.044 a 3.869 cuentas, entre −1,2 y −4,4 pp por cohorte. Según CX, hasta ese día el 18,6% de las reservas de 2026 eran canceladas o no-show.
- Pasar la ventana del mes 3 al mes 4 sube la activación entre 1 y 4 pp.
- Usar la graduación en vez de la activación baja las cohortes entre 5 y 6 pp (medido el 21-ago-2026).
- Universo comparable @M4: 60,9% sobre 2.351 cuentas, contra 53,4% sobre 3.451 si se mezclan cohortes que todavía no maduran.

## Caveats
- Las decisiones del 23-sep y del 28-sep figuran en el código como "decisión de Pablo", sin apellido.
- No se midió el puente Evidence → CX (ventana, unidad, cobros y universo). Queda pendiente si hace falta cuantificar cuánto pesa cada diferencia.
- Las diferencias de ventana y de unidad explicarían al menos parte de por qué los números de CX y de Finanzas no calzan (por ejemplo, la pregunta sobre Chile en el hilo de Otros LATAM), pero no está medido.
- Definiciones y fuentes vigentes: [[Activación — fuentes y definiciones vigentes]].
