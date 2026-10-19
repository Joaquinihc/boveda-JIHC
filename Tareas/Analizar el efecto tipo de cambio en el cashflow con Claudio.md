---
type: tarea
estado: propuesta
prioridad:
tiempo:
tipo: analisis
origen: reunion
proyecto:
temas:
  - cierre-mensual
fuente: "[[2026-10-06 Revisión cierre de septiembre — cashflow y headcount]]"
fecha_limite:
creado: 2026-10-06
revisar: true
id: T-093
---

# Analizar el efecto tipo de cambio en el cashflow con Claude

Revisar en detalle cómo se calcula el efecto tipo de cambio del cashflow (posiblemente solo considera la variación de saldos iniciales) y cómo impacta entre países según la proporción de ingresos y costos en cada moneda. Acción de [[2026-10-06 Revisión cierre de septiembre — cashflow y headcount]].

## Subtareas
- [x] Revisar con Claude cómo se calcula hoy el efecto tipo de cambio
- [ ] Cruzar el cashflow con la planilla de headcount para entender los movimientos en moneda
- [ ] Explicarle a Pablo cómo impacta el tipo de cambio entre países según la mezcla de ingresos y costos por moneda

## Notas
### 2026-10-08 · Cómo se calcula hoy el efecto tipo de cambio
- La sospecha de Pablo es correcta: el efecto de TC del cash flow es la **revalorización de los saldos bancarios de apertura**. No mide efecto de TC sobre los flujos del mes, que tesorería ya convierte al TC de cierre.
- Formato Causal: cuenta de tesorería "Efecto tipo de cambio en Saldo Inicial", dentro de Cash from Finance (y contada dos veces, porque el Initial ya viene reconvertido).
- Estructura nueva (Evidence, PR #767): fila FX rate effect = Final − Initial − Ops − Finance. Iguala la cuenta de tesorería cuando no hay descuadres (sep-26: −15.892 en ambos).
- Limitación: solo reconvierte bancos; el TC sobre inversiones y pasarelas queda en Other o en Payment gateways (ene-24: −29,4k del DAP BICE en Other).
- Del lado del P&L: en sep-26 vs ago-26 el TC aportó +6,4k al EBITDA (revenue −21,8k, costos +28,2k), sin abrir por moneda. Ver [[20261007-01 Varianza P&L septiembre vs agosto 2026]].
- Detalle y descuadres de jul–ago: [[Cash flow — estructura nueva vs formato Causal]].

### Pendientes menores
- 2026-10-06 · La planilla mezcla valores en porcentaje y en monto: uniformar.
- Duda: la transcripción no identifica hablantes; se asume que el encargo («dedicarte a analizar… con Claude») es para Joaquín.
  - 2026-10-19 · Resuelta: según el calendario (evento «Pablo - Joaquín CF», 6-oct 16:00), la reunión fue con [[Pablo Lucero]] y el encargo es de Joaquín. «Claudio» en la transcripción es Claude.
- 2026-10-08 · Pedir a tesorería que extienda la fórmula de Amount USD hasta la última fila del Dataset: las filas sin fórmula llegan al DWH en 0 USD (sep-26: cuenta de efecto de TC en −10.215 en el DWH vs −15.892 real).
- 2026-10-08 · Pedir a tesorería reclasificar el rescate del DAP BICE de ene-24 (hoy "Financiamiento"): Evidence lo excluye del FCF como préstamo (ene-24 −676,7k vs −115,2k real).

## Historial
- 2026-10-14 · Creada desde [[2026-10-06 Revisión cierre de septiembre — cashflow y headcount]] (propuesta)
- 2026-10-19 · Avance desde Claude Code: cálculo del efecto de TC explicado (subtarea 1 hecha); nueva subtarea sobre el impacto por país; duda de hablantes resuelta; menciones a «Claudio» cambiadas a «Claude» (Joaquín en conversación)
