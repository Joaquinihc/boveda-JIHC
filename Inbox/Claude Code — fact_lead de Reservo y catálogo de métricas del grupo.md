---
type: borrador
origen: Claude Code
fecha: 2026-11-19
temas: [leads, adquisicion, dwh, board]
revisar: true
---

# Claude Code — fact_lead de Reservo y catálogo de métricas del grupo

Hallazgos de las sesiones del 12 al 19-nov para «Métricas en conjunto con Reservo» (Notion: https://app.notion.com/p/416ab34c233b80fc9d85de44a9e687ff). Cifras al 19-nov-2026, desde `staging.stg_reservo__*` en Redshift. Cómo se cruzan las tablas: [[reservo_ext y stg_reservo — tablas, llaves y horarios de carga]].

- **Catálogo**: con lo que hay en Redshift salen 27 de las 33 métricas del Board Book para Reservo. Faltan sobre todo citas y reservas (`fact_citas` aún no está en Redshift), MRR contractual antes de nov-26, IA por cuenta y Payments mes a mes (detalle en la hoja 4 de Notion).
- **Embudo, cohorte de sep-26** (`fact_lead`, sumando los embudos Ventas y Clientes): 2.473 leads → 1.078 calificados (44 %) → 461 demos agendadas (19 %) → 204 demos realizadas (8 %) → 129 ganados (5,2 %). Contando solo `embudo = 'Ventas'`, como Israel Gutiérrez, son 2.347 leads. Las fechas de calificado y demo existen solo desde abr-26.
- **Desde abr-26, ganado = new merchant**: el primer pago cae el mismo mes (oct-26: 130 de 137 ganados con cuenta). New merchants de abr a oct 2026 por canal (1.093): Google Ads 591 (54 %), Instagram 196 (18 %), SEO/ingreso directo 134 (12 %), Facebook 83 (8 %), Nueva sede 30, Alianza con Vambe 19, Referido 14 y otros 26.
- **B2B2 no sale de los leads**: de 762 ganados B2B3 de 2026, 363 tienen `cantidad_profesionales` = 0 y 75 lo traen vacío.
- **Foto diaria** (`fact_clientes_diario`, 11 fotos del 9 al 19-nov sin huecos): 4.137 → 4.151 cuentas activas; salieron 49 (44 con evento de baja) y entraron 63 (56 cuentas nuevas, 4 desbloqueos y 3 activaciones). Al 19-nov: B2B3 2.999, B2C 1.136 y 16 sin plan; 3.722 companies.
- **Vertical al 19-nov** con el mapeo de rubros propuesto (sin aprobar): Health 3.429 (Medicina/Salud + Psicología + Odontología), Medspa 505, Beauty 67 y Other 150.
- **Por qué importa**: abre leads y adquisición por canal para el grupo; el mapeo de vertical y la definición de lead del grupo esperan decisión (hoja 6; la definición de lead es la decisión 8). Relacionada: [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]].
