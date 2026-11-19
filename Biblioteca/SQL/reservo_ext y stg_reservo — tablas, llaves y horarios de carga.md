---
type: recurso
temas: [dwh, leads]
autor: "[[Joaquín Herrera]]"
fecha: 2026-11-19
---

# reservo_ext y stg_reservo — tablas, llaves y horarios de carga

**Propósito**: dónde vive la data de Reservo en el DWH de AgendaPro, cómo se cruzan sus tablas y cuándo se actualiza. Vale al 19-nov-2026. Cómo se usa en el Board Book: [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]]. La documentación de Reservo es el repo `reservo-dw-modelo` (privado; sin acceso nuestro al 19-nov) y el STTM de Christopher Lagos.

## Cadena y horarios
1. Base de Reservo (MySQL `reservov2`) → proceso DW de Reservo (19 tablas). `dim_centro` carga a las 06:00 UTC y la foto `fact_clientes_diario` se genera a las 06:15 UTC.
2. Copia a S3 (`s3://ap-dwh-reservo/reservo/`) 4 veces al día: 00:55, 06:55, 12:55 y 18:55 UTC.
3. Carga de AgendaPro a las 04:10 y 23:10 (hora de Chile) → Redshift Spectrum `reservo_ext` (17 tablas externas) → dbt `staging.stg_reservo__*` (17) → `reservo.int_reservo_payments_flattened` y `reservo.reservo_mrr` → Evidence.

- Solo en S3/Athena, no en Redshift: `fact_citas` (pedida a Israel Gutiérrez el 17-nov) y `fact_cobros_online` (no sirve).
- Alertas de carga: desde el 12-nov, llaves únicas, relaciones y llegada de la foto; desde el 18-nov (dbt #509), frescura por tabla, y la carga se detiene si un texto excede el largo de la columna.
- Réplica MySQL de Reservo: acceso de solo lectura desde el 18-nov, por túnel. No se lee desde Redshift ni entra al DWH (datos de salud).

## Desde cuándo hay datos
| Tabla | Desde |
|---|---|
| `fact_cambios_estado` | jul-2012 |
| `fact_pagos` | ago-2014 |
| `fact_lead` | jul-2016 |
| `fact_cambios_mensualidad` | jul-2020 |
| `fact_demos` | dic-2023 |
| `fact_ventas` | abr-2026 |
| `fact_clientes_diario` (foto diaria) | 9-nov-2026 (la del 9-nov se corrió a mano y no se puede regenerar) |

## Llaves
- Cuenta = `stg_reservo__dim_centro.id`. `id_cliente` (entero) es único por centro y es la llave que usa Pipedrive.
- `fact_pagos.id_dim_centro` → `dim_centro.id`. Solo trae pagos con estado 1; el estado 0 no es revenue (Christopher, 18-nov). `es_primer_pago` marca el new merchant. `medio_de_pago` dejó de llenarse en 2018.
- `fact_cambios_estado`: columnas `id`, `id_dim_centro`, `id_dim_tiempo`, `tipo`, `razon_baja`, `fecha_hora` (no existe `tipo_cambio`).
- `fact_lead` (Pipedrive): la llave a la cuenta es `cliente_id` (texto) = `dim_centro.id_cliente`. **No usar** `id_dim_centro` (siempre 9347, «No informado»), `id_dim_rubro` (siempre 8) ni `id_reservo` (casi no cruza). `fact_ventas` apunta al centro −1.

```sql
-- Negocio ganado de Pipedrive → cuenta de Reservo
-- Validado el 19-nov-2026: cruzan 1.589 de 1.640 ganados de 2026 (96,9 %)
select l.id                      as id_lead,
       l.fecha_primer_ganado_utc,
       l.embudo,
       l.canal,
       c.id                      as id_dim_centro
from staging.stg_reservo__fact_lead l
left join staging.stg_reservo__dim_centro c
  on l.cliente_id ~ '^[0-9]+$'
 and c.id_cliente = l.cliente_id::int
where l.fecha_primer_ganado_utc >= '2026-01-01';
-- Para el primer pago: join staging.stg_reservo__fact_pagos p
--   on p.id_dim_centro = c.id and p.es_primer_pago
```

## Cómo leer fact_lead
- Desde abr-2026, ganado = new merchant: el primer pago cae el mismo mes de `fecha_primer_ganado_utc`. Antes de abril, cerca de un tercio se marcaba ganado después de pagar.
- Los ganados pasan del embudo «Ventas» a «Clientes: Onboarding / Vigentes / Riesgo Alto». Para conversión por cohorte hay que sumar esos embudos (lectura sin confirmar con Christopher).
- Las fechas de calificado y demo existen solo desde abr-2026. `canal` siempre viene lleno en los ganados («Nueva sede» = sucursal nueva de una cadena). `cantidad_profesionales` viene en 0 o vacío en la mayoría de los B2B3: no sirve para B2B2.
- Campos con datos personales (url, gclid, razón de pérdida): usarlos solo agregados.
