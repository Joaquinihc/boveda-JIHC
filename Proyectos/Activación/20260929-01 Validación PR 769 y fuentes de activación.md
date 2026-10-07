---
type: analisis
proyecto: "[[_Activación]]"
estado: activo
fecha: 2026-09-29
temas: [activacion, definiciones, evidence, dwh, fitness]
metricas: ["[[Activation Velocity]]", "[[Activation]]"]
---

# Validación del PR 769 y fuentes de activación en Evidence

**Pregunta**: ¿Están bien los cambios que hizo [[Pablo Lucero]] a Activation Velocity (PR 769 de `agendapro-dashboards-evidence`)? ¿Mis cambios para sumar enrollments a la activación no se veían porque Evidence consulta `company_sales_days`?

## Conclusión
> - **El PR 769 está bien.** El universo nuevo cuadra exacto con New Merchants B2B3 (`new_merchants_b2b3.sql`) en ene–ago 2026 y las cifras de Pablo se replican exactas. Casi toda el alza viene de sacar las empresas que nunca pagaron (6–18 por mes en B2B3); las clases suman entre 0,0 y 0,6 pp.
> - **Sí, por eso no se veían.** Desde may-2026, Evidence calcula Activation Velocity directo sobre `dwh.company_sales_days`, que sale de `dwh.bookings` y no trae enrollments. Mis enrollments viven en dbt (PRs #370, #363, #371 y #372), en modelos que esa página no lee.
> - **Quedaron dos definiciones de enrollments** (ver tabla), y las tablas dbt de velocity estaban congeladas al 7-sep porque ningún flow las corría (lo arregló el PR 412).

## Método
- Diff del PR 769 y de los PRs de dbt; queries contra Redshift el 29-sep-2026.
- Réplica de la Semana 9, B2B3, umbral 100, ponderado por sedes, separando el efecto del universo y el de las clases.

## Evidencia
**Réplica de las cifras del PR 769** (B2B3, Semana 9, % de sedes, corte 29-sep):

| Cohorte | Antes | Solo universo | + clases (PR 769) |
|---|---|---|---|
| ene-26 | 51,7 | 53,6 | 53,6 |
| abr-26 | 46,2 | 47,7 | 47,8 |
| jul-26 | 52,7 | 54,2 | 54,8 |

**Qué lee cada página (29-sep)**
- Activation Velocity, OKR y board book (`monthly_weekly`, `monthly_monthly`, `by_onboarder`): `company_sales_days` + clases desde el PR 769. De `dwh.activation_velocity_weekly_report` solo toma `initial_plan_type`.
- Onboarder Capacity: `dwh.activation_velocity_weekly_report` (dbt, con mis enrollments).
- Graduation: `dwh.int_company_activation_status` (dbt, con mis enrollments).

**Dos definiciones de enrollments**

| | PR 769 (Evidence) | dbt (`enrollments_count_demand`) |
|---|---|---|
| Fecha | día de la clase | creación |
| Estados | sin canceladas ni no-show | todos |
| Orígenes | todos, migraciones incluidas | solo demanda (marketplace, recurring, caja) |

En Semana 9 dan casi lo mismo (≤0,2 pp), pero el volumen por empresa difiere mucho: la 530939 tiene 2.905 con la de Evidence y ~6.950 con la de dbt, porque dbt cuenta clases recurrentes futuras y canceladas.

**Observaciones al PR 769**
- El comentario del SQL tiene los estados invertidos: en plt-classes `2 = no_show` y `3 = cancelled`. El filtro igual es correcto, porque excluye los dos.
- Cuenta los orígenes de migración (`membership_migration`, `migracion_ikigai`). La 562909 (B2C, cohorte ago-26) cruza el umbral con 113 enrollments migrados.
- No hay duplicación bookings/enrollments: las empresas *migran* de agenda a clases (los bookings caen a 0 cuando empiezan las clases); solo se solapan 1–2 meses en empresas antiguas.
- El filtro de estados es coherente con los bookings: `booking_count_active` excluye cancelado (5) y no-show (6).

## Caveats
- Fotografía al 29-sep. Después de esto, Pablo extendió las mismas reglas a Retention, R1 y PMF (PR 772) y agregó la curva de PMF Fitness (PR 774). Ver [[Slack — Activation Velocity y tableros de activación con universo New Merchants (PR 769 y 772)]].
- Seguimiento en [[Arreglar definición de activación]].
- Definiciones y fuentes vigentes: [[Activación — fuentes y definiciones vigentes]].
