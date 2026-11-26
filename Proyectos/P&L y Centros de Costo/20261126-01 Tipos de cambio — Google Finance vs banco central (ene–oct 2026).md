---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-26
temas: [pnl, odoo, dwh, cierre-mensual]
metricas: []
---

# Tipos de cambio — Google Finance vs banco central (ene–oct 2026)

**Pregunta**: ¿Cuánto cambian las cifras si el DWH y Evidence pasan de Google Finance a tasas oficiales de banco central, como ya hizo Odoo?

## Conclusión
> - **El cambio es chico.** Google vs banco central, ene–oct 2026: promedio mensual 0,1–0,2 %, tasa de cierre 0,3–0,7 %; peor mes ARS de enero (−3,7 %).
> - **P&L a tasa oficial**: revenue −6,9k USD (−0,07 %); EBITDA prácticamente igual.
> - **Ojo con la tabla de Nico en el PR #1087** (−2 % a −5 %): compara promedio mensual vs tasa de cierre, no Google vs banco central.

## Estado de la fuente (26-nov)
- **Odoo**: [[Nicolás Astudillo]] dejó las 7 sociedades en manual con tasas oficiales (PR #1086, abierto).
- **DWH/Evidence**: siguen con Google Finance (hoja «Exchange Rates»). [[Pablo Lucero]] dijo «de acuerdo» a pasar a tasas oficiales, pero falta definir quién cambia la hoja y qué fuente usar para EUR, PEN, CRC, PYG y UYU (decisión D8).
- Argentina (NIC 29, todo a cierre) quedó fuera del mensaje al grupo.

## Método
- Comparación mes a mes de tasas Google Finance (hoja «Exchange Rates») vs banco central de cada país, promedio mensual y cierre, ene–oct 2026; P&L recalculado a tasa oficial.

## Caveats
- Fuente: resumen del canal de trabajo del 26-nov-2026; las queries y planillas de la comparación no quedaron enlazadas.
- La precisión de las tablas de tipo de cambio del DWH es otro tema: [[Claude Code — Decimales truncados en el DWH (tipos de cambio, Chargebee y Facebook spend)]].
