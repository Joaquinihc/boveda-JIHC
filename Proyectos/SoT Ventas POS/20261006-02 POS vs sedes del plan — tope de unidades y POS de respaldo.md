---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-10-06
temas: [pos, ventas, payments]
metricas: []
---

# POS vs sedes del plan — tope de unidades y POS de respaldo

**Pregunta**: ¿Los comercios contratan más o menos POS que sedes? ¿Conviene topar las unidades de POS por las sedes del plan de cada mes, y tratar el exceso como POS de respaldo, porque el revenue de pagos dependería de las sedes con POS y no de los terminales?

## Conclusión
> - **El 92% de los comercios con POS tiene 1 sede**, y ahí casi siempre es 1 sede = 1 POS. Más POS que sedes se da en el 6% de los de 1 sede. Menos POS que sedes, en la mitad o más de los multi-sede.
> - **El respaldo no es ingreso cero.** Cuando un comercio que ya cubría todas sus sedes compra otro POS, su GPV POS sube +13% en la mediana en 3 meses. Si el POS cubre una sede nueva, +27%. La línea base es +2%.
> - **Las sedes cubiertas ordenan el GPV mejor que los terminales** donde difieren (Spearman 0,33 vs 0,20). Lo que mejor lo ordena son las sedes que de verdad transaccionan con POS (0,60).
> - **Topar por sedes del plan afectaría a pocos:** 27 comercios y 34 u (6,1%) en ene–sep 2026, casi todo POS adicional y canal Others.
> - **Decisión de Joaquín (8-oct):** el board book se queda en unidades sin tope. El tope por sedes queda para un reporte aparte de utilización de terminales: [[Reporte de utilización de terminales POS en Evidence (Payments)]].

## Método
- **Unidades:** `dwh.pos_sales_sot` (válidas, sin soft), acumuladas por comercio.
- **Sedes:** activas hoy (`dwh.locations.active`) y sedes del plan por mes (`dwh.mrr.venues`, `is_active`).
- **Uso real:** seriales de terminal en `dwh.haulmer_augmented_transactions` (Chile) y `dwh.oel_augmented_transactions` (México). Traen `pos_serial` en el 100% de las filas; SumUp no aporta.
- **GPV y fee POS:** `dwh.location_sales_months` por comercio y mes, con las sedes con `pos_payment_count > 0`.
- **Estudio de eventos:** cada POS adicional válido entre abr-24 y jun-26. Se compara el GPV de los 3 meses antes con los 3 meses después. La línea base son los comercios con POS sin ventas en esa ventana.
- **Simulación del tope:** POS vigentes = unidades desde la última reactivación (la reactivación reinicia el conteo). Tope = sedes del plan del mes de la venta; si falta, la última o la siguiente conocida. Por venta: cubre sede = mín(POS después, tope) − mín(POS antes, tope); respaldo = el resto.

## Evidencia
Corte: 6 al 8-oct-2026. Comercios con primera venta desde 2024, Chile + México.

**POS acumulados vs sedes activas**

| Sedes | Comercios | Menos POS que sedes | Igual | Más POS que sedes |
|---|---|---|---|---|
| 1 | 1.422 | — | 1.339 | 83 |
| 2 | 90 | 46 | 33 | 11 |
| 3–5 | 34 | 20 | 12 | 2 |

**Terminales usados** (seriales con transacción, últimos 30 días)
- 1 sede: 31 de 576 comercios (5%) usan 2 o más terminales.
- 2 sedes: 47 terminales para 72 sedes.
- 3 o más sedes: 39 terminales para 169 sedes.
- De 29 comercios de 1 sede que contrataron de más, solo 10 usan el terminal extra en 90 días.

**GPV POS, jul–sep 2026** (USD/mes)
- 1 sede · 1 POS (768 comercios): promedio 4.567, GPV por POS 4.567.
- 1 sede · 2+ POS (37): promedio 9.618, GPV por POS 4.237.

**Simulación del tope, ene–sep 2026**

| País | Unidades hoy | Cubre sede | Respaldo | Comercios con respaldo |
|---|---|---|---|---|
| Chile | 409 | 383 | 26 | 20 |
| México | 152 | 144 | 8 | 7 |

- Por tipo de venta: primeras ventas 515 → 509; POS adicional 46 → 18.
- Por canal en Chile, la parte que pasa a respaldo: Others 39%, Stock 4,5%, Inbound 0,5%.

## Caveats
- **Sedes de hoy, no del momento de la venta.** Las sedes activas son las actuales: un comercio pudo abrir sedes después de comprar. Con las sedes del plan del mes de la venta da parecido.
- **Cobertura de seriales.** El conteo de terminales usados cubre solo Haulmer y OEL, no TBK/Klap/Getnet. 56 comercios transaccionan POS ahí sin venta válida en la SoT.
- **Muestras chicas.** El estudio de eventos tiene 28 a 31 eventos por grupo: sirve como dirección, no como precisión. Parte del alza es causalidad inversa (se compra el terminal porque ya se vende más).
- **Stock sobreestimado.** La SoT no registra devoluciones de terminales, solo reactivaciones.
