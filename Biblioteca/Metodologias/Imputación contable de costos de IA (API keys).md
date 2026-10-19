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
