---
type: metodologia
temas: [ai, pnl, centros-de-costo, odoo]
proyecto: "[[_P&L y Centros de Costo]]"
fuente: skill update-ai-api-costs (agendapro-dbt, PR #407)
fecha: 2026-10-02
---

# Imputación contable de costos de IA (API keys)

> Fuente ejecutable: skill `update-ai-api-costs` en `agendapro-dbt` (`.claude/skills/update-ai-api-costs/SKILL.md`, tabla `ASSIGNMENT_RULES`). Si esta nota y la skill difieren, manda la skill. Registro en el DWH: `finance.ai_api_key_usage_monthly`, una fila por key y mes.

## Aprobación
**La imputación de cada key nueva, o el cambio de una existente, la aprueba Joaquín. Claude propone; nunca decide.**
- La propuesta lleva todos los campos con su motivo: owner, tipo, uso, p_l_category, business, BU, centro de costo, cuenta a descontar y cuenta a sumar.
- No vale como aprobación el silencio, un ok a otra pregunta ni lo aprobado en un mes anterior.
- Las keys que ya tienen historia heredan la imputación del último mes y no se vuelven a preguntar.

## Cuenta a descontar: según el proveedor
Es donde Odoo registra hoy la factura (AgendaPro Inc):

| Proveedor | Cuenta |
|---|---|
| Claude | 5.2.25 G2104 Tech platforms |
| OpenAI | 5.2.22 G2004 Product platforms |
| Azure | 5.2.0 G0001 AWS |

## Centro de costo y cuenta a sumar: según p_l_category + business + BU
| p_l_category | Business | BU | Centro de costo | Cuenta a sumar | Bloque | Uso |
|---|---|---|---|---|---|---|
| Platform | Beauty | HQ | BTY-PLAT-HQ | G0030 AI models Beauty Service | 2- Service Costs | Servicio |
| Platform | Health | HQ | HLT-PLAT-HQ | G0031 AI models Health non-doctors Service | 2- Service Costs | Servicio |
| Platform | Medspa | HQ | MSP-PLAT-HQ | G0032 AI models Medspa Service | 2- Service Costs | Servicio |
| Platform | Marketplace | HQ | MKP-PLAT-HQ | G0033 AI models Marketplace Service | 2- Service Costs | Servicio |
| Platform | Payments | HQ | PAY-PLAT-HQ | G0034 AI models Payments Service | 2- Service Costs | Servicio |
| Platform | Product & Tech | HQ | P&T-PLAT-HQ | G0035 AI models Product & Tech Service | 2- Service Costs | Servicio |
| Customer Support | Marketplace | HQ | MKP-SUPP-HQ | G0205 AI models Marketplace Support | 2- Service Costs | Servicio |
| Customer Support | Revenue | HQ | REV-SUPP-HQ | G0204 Other expenses Support | 2- Service Costs | Servicio |
| Sales | Revenue | HQ | REV-SALES-HQ | G1104 Sales platforms | 3- Acquisition Costs | Servicio |
| Development | Beauty | HQ | BTY-TECH-HQ | G2112 AI models Beauty Tech | 4- Corporate Costs | Desarrollo |
| Development | Health | HQ | HLT-TECH-HQ | G2113 AI models Health non-doctors Tech | 4- Corporate Costs | Desarrollo |
| Development | Medspa | HQ | MSP-TECH-HQ | G2114 AI models Medspa Tech | 4- Corporate Costs | Desarrollo |
| Development | Marketplace | HQ | MKP-TECH-HQ | G2115 AI models Marketplace Tech | 4- Corporate Costs | Desarrollo |
| Development | Payments | HQ | PAY-TECH-HQ | G2116 AI models Payments Tech | 4- Corporate Costs | Desarrollo |
| Development | Product & Tech | HQ | P&T-TECH-HQ | G2104 Tech platforms | 4- Corporate Costs | Desarrollo |
| Administration | Executive Office | HQ | ADM-HQ | G3015 CEO platforms | 4- Corporate Costs | Desarrollo |
| Administration | People & Finance | HQ | ADM-HQ | G3017 Finance platforms | 4- Corporate Costs | Desarrollo |

## Criterios
1. **Uso → bloque del P&L.**
   - Servicio va a Platform, Customer Support o Sales.
   - Desarrollo va a Development o Administration.
2. **Keys verticales de servicio.** Una key vertical de servicio (Beauty, Health, Medspa) va a Platform de su propio vertical.
3. **Centro de costo.** Usa la estructura del headcount, `<ORG>-<FUNC>-<BU>`, con códigos vigentes de `finance.raw__headcount`; nada de códigos antiguos como PTECH. Ver [[Guía clasificación de personas (headcount)]].
4. **La línea del P&L la define el G-code, no el centro de costo.**
   - Los equipos verticales tienen un par de cuentas: servicio en el bloque G003x y tecnología en G21xx.
   - Los equipos centrales usan la cuenta genérica de su línea.
5. **G-code nuevo.**
   - Se toma del rango de su línea: G00xx Platform, G02xx Customer Support y G21xx Development.
   - Hay que verificar que esté libre en la Master, en el plan de cuentas de Odoo (las cinco sociedades), en las analíticas y en `finance.profit_and_loss`. G0020 ya existe como analítica.
   - El nombre en Odoo parte con el G-code. Hay que darlo de alta en la Master con su p_l_category y su bloque.
   - G0030 en adelante quedó como bloque propio para modelos IA de servicio.
6. **Analíticas en Odoo.** El plan «Centros» nombra el centro de costo sin el sufijo de BU (BTY-PLAT, no BTY-PLAT-HQ), salvo ADM-HQ y MKT-HQ.
7. **Una combinación que no está en la tabla** requiere que Joaquín decida el centro de costo y la cuenta. Después se agrega a la skill, con un PR.

## Asiento mensual en Odoo
- **Resumen del mes:** la skill entrega `odoo_reclass`, el monto por cuenta a descontar → cuenta a sumar × centro de costo.
- **Cuenta igual en origen y destino** (G2104 → G2104): solo cambia la analítica.
- **Montos de consumo, no de factura.** Odoo registra la factura cuando llega. Conviene reclasificar lo contabilizado del mes y repartirlo con el mix de consumo.
- **Margen por compañía:** cuando G0030–G0035 tengan movimiento en el P&L, hay que agregar reglas en la macro de pools de `company_margin_*`. Si no, el costo de modelos IA se cuenta dos veces.

## 2026-11-02 · Cómo queda en Odoo y cómo se prepara el asiento
- **Los G-codes de IA no son cuentas en Odoo.** G0030–G0035, G0205 y G2112–G2116 son analíticas del plan «Centros» (id 26). El nombre es el CECO sin BU (BTY-PLAT) o el G-code (G2113); la referencia lleva el G-code («G0030 - AI models Beauty Service»). En la Master sí están como códigos, en Mappings.
- **Cuentas reales en AgendaPro Inc:** Platform → 5.2.73 Platform IA · Customer Support → 5.2.8 G0204 · Sales → 5.2.16 G1104 · Development → 5.2.25 G2104 · Administration → 5.2.57 G3015 o 5.2.59 G3017.
- **Analítica de la línea de débito, según el centro de costo:**

| Centro de costo | Analítica |
|---|---|
| BTY-, HLT-, MSP-, MKP-, PAY- y P&T-PLAT-HQ | BTY-PLAT, HLT-PLAT, MSP-PLAT, MKP-PLAT, PAY-PLAT y P&T-PLAT |
| MKP-SUPP-HQ | MKP-SUPP |
| REV-SUPP-HQ · REV-SALES-HQ | G0204 · G1104 |
| BTY-TECH-HQ | G2112 (por crear al 2-nov) |
| HLT-TECH-HQ | G2113 |
| MSP-TECH-HQ | G2114 (por crear al 2-nov) |
| MKP-TECH-HQ | G2115 |
| PAY-TECH-HQ | PAY-TECH |
| P&T-TECH-HQ | G2104 |
| ADM-HQ | G3015 o G3017, la misma de su cuenta |

- **No usar las analíticas con nombre de CECO de personas** (HLT-TECH, MKP-TECH, MSP-TECH, BTY-TECH, REV-SUPP, REV-SALES, ADM-HQ, P&T-TECH): su referencia es el G-code de remuneraciones.
- **Servicio y desarrollo separados desde oct-2026** (decisión de Joaquín, 2-nov). En sep-2026 (asientos MISC/2026/09/0008–0010) todo el desarrollo quedó con la analítica G2104, y no se corrige.
- **El asiento:** uno por proveedor, con fecha del último día del mes y ref «API Key Cost Reclass». El monto es el consumo del mes con IVA (así se hizo en sep y oct), no lo contabilizado. Lo contabilizado sirve de control, para ver que alcance para la resta: Claude = partners Anthropic y Claude en 5.2.25 (sin ChatGPT, que va con partner OPENAI); OpenAI en 5.2.22; Microsoft en 5.2.0. La línea Claude de P&T-TECH (G2104 → G2104, con analítica G2104) no se mueve.
- **Plazo:** el cierre automático de períodos corre el día 3 de cada mes a las 9:50. Si el asiento se registra después, hay que desbloquear el mes.
- **Quién lo prepara:** la skill `prepare-ai-api-costs-reclass` (`~/.claude/skills/`, nivel usuario), la segunda parte después de `update-ai-api-costs`. Agrega el mes a la planilla «API Keys Costs 2026», la alinea con Odoo y deja en borrador el DM a [[Nicolás Astudillo]]. Necesita los conectores de claude.ai (Google Sheets, Slack y DWH) y leer `~/Documents/GitHub/mcp-odoo-agendapro` (su `.venv` y su `.env`); por eso conviene abrir la sesión en `~/Documents/GitHub`. No necesita el repo dbt, porque el mes sale de `finance.ai_api_key_usage_monthly`. Odoo es solo lectura y Slack, solo borradores.
- **Desde la corrida de noviembre,** la columna del mes de la planilla se calcula sobre la pestaña «Registro DWH» con SUMPRODUCT+EXACT, porque SUMIFS no distingue «Default project» de «Default Project» (sumaba 122,70 dos veces).
