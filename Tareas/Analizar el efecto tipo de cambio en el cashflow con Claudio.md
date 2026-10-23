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
- [ ] Decidir si los reportes de cash flow muestran la cuenta de tesorería como FX rate effect y, si sí, cambiar las 3 páginas de Evidence con el control |residuo − cuenta| > US$1

## Notas
### 2026-10-22 · La cuenta de tesorería ya calza con el residuo
- Con la fórmula de Amount USD extendida (sep-26) y Shinkansen (MX) agregada a `Auxiliar` por Rafael Rodríguez (ago-26), la cuenta de reevaluación de tesorería y el FX rate effect de Evidence difieren a lo más US$0,10 por mes en feb–sep 2026 (`finance.cash_flow`, 22-oct). Los centavos son redondeo por fila en dbt.
- Ago-26 no era un TC redondeado a 17,34 (diagnóstico del 7-oct, corregido el 20-oct): la cuenta Shinkansen (MX) tenía efecto 0 porque `Auxiliar` no la incluía (+US$33,50).
- Detalle, implicancias y dónde cambiar: [[Cash flow — estructura nueva vs formato Causal]].

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
  - 2026-10-21 · Hecho: Joaquín extendió la fórmula el 20-oct; sep-26 en el DWH quedó en −15.891,56.
- 2026-10-08 · Pedir a tesorería reclasificar el rescate del DAP BICE de ene-24 (hoy "Financiamiento"): Evidence lo excluye del FCF como préstamo (ene-24 −676,7k vs −115,2k real).
- 2026-10-22 · Corregir la explicación de ago-26 («TC redondeado a 17,34») en el Slack a [[Pablo Lucero]] del 7-oct y en la hoja Notes de `2. Cash Flow Statement - nueva estructura.xlsx`: la causa era Shinkansen (MX) fuera de `Auxiliar`.
- 2026-10-22 · Preguntar a tesorería por los TC del CLP de 2026 en la hoja `Tasa de cambio` (930,232558; 934,579439; 970,873786 = 1/0,001075, 1/0,00107, 1/0,00103): parecen USD/CLP con pocos decimales invertidos; sep-26 queda en 970,87 vs 946,7 del P&L.
- 2026-10-22 · Solo si se quiere calce al centavo en Evidence: subir la precisión de `amount` y `amount_usd` en `models/intermediate/finance/cash_flow.sql` (hoy `decimal(18,2)`) y confirmar que el conector lee valores sin formato.

## Historial
- 2026-10-14 · Creada desde [[2026-10-06 Revisión cierre de septiembre — cashflow y headcount]] (propuesta)
- 2026-10-19 · Avance desde Claude Code: cálculo del efecto de TC explicado (subtarea 1 hecha); nueva subtarea sobre el impacto por país; duda de hablantes resuelta; menciones a «Claudio» cambiadas a «Claude» (Joaquín en conversación)
- 2026-10-23 · Avance desde Claude Code: cuenta de tesorería = residuo tras los arreglos de ago y sep-26; nueva subtarea sobre la presentación del FX rate effect (3/3 abiertas); 3 pendientes menores (Joaquín en conversación)
