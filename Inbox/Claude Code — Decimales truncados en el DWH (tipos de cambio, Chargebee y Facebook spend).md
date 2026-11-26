---
type: sugerencia
temas: [dwh, revenue, marketing]
creado: 2026-11-26
revisar: true
---

# Decimales truncados en el DWH (tipos de cambio, Chargebee y Facebook spend)

> Borrador desde Cowork (resumen de un canal de trabajo, 26-nov-2026). Salió al revisar la fuente de tipos de cambio: [[20261126-01 Tipos de cambio — Google Finance vs banco central (ene–oct 2026)]].

- **Tipos de cambio (PR #538, mergeado)**: las tablas de tipo de cambio que vienen de Google Sheets pasaron a `numeric(18,4)`, pero los valores aún tienen 2 decimales reales (MXN oct: 18.7900). Confirmar con Israel que el sync trajo 4 decimales y que corrió dbt. Riesgo: si alguien pega datos en la hoja, se puede perder el formato 0.0000.
- **Chargebee (PR #541, abierto)**: `stg_chargebee__invoices.exchange_rate` con `::decimal` deja EUR en 0 (1.782 facturas 2026) y trunca MXN/PEN. Espera revisión y full-refresh de Israel. Afecta DSO, cuentas por cobrar, cobranza y salud de pagos; no MRR ni Board Book.
- **Facebook spend**: truncado, ~16k USD menos ene–oct 2026 (−5,5 %). No es tipo de cambio. Sin PR; el modelo es incremental y necesita full-refresh. Por qué importa: es gasto de adquisición (CAC).
- Pendientes de Israel: sync de 4 decimales, revisar #541 y full-refresh, Facebook spend.
