---
type: analisis
proyecto:
estado: activo
fecha: 2026-11-03
temas: [definiciones, ventas, revenue, board, evidence]
metricas: ["[[New Merchants (nuevas altas)]]", "[[Total Merchants (EOM stock)]]"]
---

# New Merchants y Merchants de Reservo — definiciones finales y cuadre con Pablo Saintard

**Pregunta**: ¿con qué regla contamos los merchants y los new merchants de Reservo en el Board Book y el Flash Revenue, y por qué el conteo de abril-2026 difería en una unidad del de Pablo Saintard (Reservo)?

## Conclusión
> **Merchants de Reservo = clientes activos vigentes de Reservo** (su propia lista mensual, menos las cuentas que se están yendo). **New merchants = cuentas cuyo primer pago pagado cae en el mes** (mínimo de `fecha_pago` por `cliente_id` con `estado_pago = 1`, solo filas aplanadas). Con esa regla los diez meses de 2026 dan lo mismo que la tabla dinámica de Pablo, incluido agosto (173, el récord). La unidad de abril no era una diferencia de regla: la lista de 129 ids es idéntica; el 130 de la foto que mandé el 2-nov salía de rellenar `fecha_pago` vacía con `mes_pago` en un pago fuera de la serie del cliente 8601. Fuente: `pagos_mensuales_aplanados_7.xlsx` (corrida de Reservo del 2-nov-2026, datos al 3-nov).

## Método
Cada corrida nueva del archivo de Christopher Lagos (Datos, Reservo) agregó hojas y cambió la regla posible. Evolución, con el corte de sep-2026 / oct-2026:

| Regla de merchants | Sep-26 | Oct-26 | Qué la descartó |
|---|---|---|---|
| Cuentas con pago propio que cubre el mes | 3.366 | — | no cuenta sucursales en cero de cadenas que pagan |
| Sucursales de cadenas con pago (`id_cadena`, historial de estados) | 3.734 | 3.641 (parcial) | Joaquín esperaba ~4.100; el flag `activo` no se reproduce desde el historial |
| Clientes activos de Reservo (hoja `clientes_activos`: no demo, cuenta lista, activada antes del cierre, sin baja, actividad en 12 meses o < 90 días) | 4.169 | 4.233 | incluía cuentas que se están dando de baja |
| **Clientes activos vigentes** (`activo_vigente`, excluye bajas en curso, bloqueadas, apagadas sin pago) | **4.067** | **4.119** (114 excluidas) | regla vigente |

| Regla de new merchants | Sep-26 | Oct-26 | Qué la descartó |
|---|---|---|---|
| Activaciones del mes (`fecha_activacion`) | 228 | 215 | muy por sobre la referencia de Reservo (~170–180) |
| Sucursales de la cadena en el mes del primer pago de la cadena | 151 | 134 | agosto daba 163 vs 173: −6 expansiones de cadenas que ya pagaban, −4 no vigentes |
| **Primer pago pagado por cuenta** (regla de Reservo, hoja 1 y 2 de Pablo) | **157** | **136** | regla vigente |

Serie 2026 con la regla vigente (ene → oct): 142 · 147 · 123 · 129 · 114 · 167 · 160 · 173 · 157 · 136. Es la misma tabla que Pablo pegó en el DM del 3-nov. Noviembre parcial (al 3-nov): 4.086 merchants, 2 nuevos.

Implementación: `scripts/seed-reservo.py` del repo `agendapro-dashboards-evidence` (rama `feat/board-book-reservo`, commit `48b4f780`) escribe `sources/reservo_seed/reservo_monthly_clients.csv`; la regla salta las filas sin `n_meses_cobertura` (fuera de la serie aplanada). Cómo se regenera: [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]].

## Evidencia
**Abril-2026, cuadre id a id.** Pablo mandó 129 `cliente_id`; la regla vigente sobre el archivo 7 devuelve exactamente esos 129 (ninguno solo en una lista). Mi verificación previa sobre el archivo 6 daba 130 en abril y 159 en julio porque, cuando `fecha_pago` venía vacía, usaba `mes_pago`. El único afectado es el cliente 8601 (México):

| id pago | mes_pago | fecha_pago | monto | tipo_pago / n_meses | estado_pago |
|---|---|---|---|---|---|
| 142881 | abr-26 | vacía | 2.100 MXN | vacíos | 1 |
| 150570 | jul-26 | 20-jul-26 | 22.680 MXN | anual / 12 | 1 |

La tabla dinámica de Pablo toma el mínimo de `fecha_pago`, ignora la fila vacía y pone al cliente en julio. El seed hace lo mismo (exige `n_meses_cobertura`). En la hoja `clientes_activos` el 8601 figura en abril y mayo como «cuenta nueva sin pago» y desde julio como «pago propio».

**Filas fuera de la serie aplanada** (sin `tipo_pago` ni `n_meses_cobertura`): 45 de 79.186 en `detalle_pagos`. 40 tienen `estado_pago ≠ 1` y quedan bien fuera. Las otras 5 tienen `estado_pago = 1` y ninguna trae `fecha_pago`; dos son de 2024 con monto cero y marcadas como pago previo a la serie, y tres tienen monto y no están en el revenue de nadie:

| id pago | cliente | mes_pago | monto |
|---|---|---|---|
| 142881 | 8601 | abr-26 | 2.100 MXN |
| 149611 | 2133 | jun-26 | 45.000 CLP |
| 162250 | 4522 | oct-26 | 150.000 CLP |

Los tres clientes tienen otros pagos aplanados, así que son pagos sueltos, no cuentas perdidas.

**Preguntas abiertas para Christopher** (pendientes al 3-nov):
1. ¿Son pagos reales esas tres filas? Si lo son, fecha y cobertura para aplanarlas como el resto.
2. Con `fecha_pago` vacía, ¿se ignora la fila (como la tabla dinámica de Pablo y el seed) o manda `mes_pago`? Es la única diferencia posible entre 129 y 130.
3. Que el DW de Reservo traiga un flag explícito de «fuera de la serie» con el motivo, en vez de detectar columnas vacías.

## Caveats
- La cifra oficial de Reservo suma 4–5 acuerdos de revenue share (Vambe) al mes cuyo pago entra por otra factura: cuentan como merchants, su revenue no está en los pagos. Pedido a Christopher: marca + monto.
- «Cuenta nueva sin pago» (208 en oct-26) está dentro de los merchants porque Reservo las cuenta; Joaquín duda de contarlas. Decisión abierta.
- Los últimos meses de new merchants suben cuando llegan primeros pagos tardíos; el mes en curso se publica como parcial.
- Sigue sin acceso directo al DW de Reservo (fichas clínicas); todo sale del xlsx que Christopher corre a mano. Reuniones de origen: [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]] y [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]].
