---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-10-02
temas: [ai, pnl, centros-de-costo, odoo]
metricas: []
---

# Reclasificación contable de costos de API keys de IA

**Pregunta**: ¿De qué cuentas de Odoo hay que descontar el costo de las API keys de IA y a qué cuenta y centro de costo hay que sumarlo, para separar servicio y tecnología por equipo?

## Conclusión
> Corte: exports de Claude, OpenAI y Azure por API key (ene–sep 2026, USD con IVA) y Odoo, asientos publicados de AgendaPro Inc, leído el 1–2 oct 2026.
> - El consumo de API keys de ene–sep suma **120.636,12 USD**. Se descuenta de tres cuentas, según el proveedor:
>
>   | Proveedor | Cuenta | USD |
>   |---|---|---:|
>   | Claude | 5.2.25 G2104 Tech platforms | 51.134,85 |
>   | OpenAI | 5.2.22 G2004 Product platforms | 8.447,71 |
>   | Azure | 5.2.0 G0001 AWS | 61.053,56 |
>
> - Se suma en cuentas nuevas por equipo: **95.847,65 USD**.
>   - Servicio y soporte (G0030–G0035 y G0205): 91.708,28.
>   - Tecnología (G2112–G2116): 4.139,37.
> - Se suma en cuentas existentes de equipos centrales: **24.788,47 USD**.
>   - P&T-TECH en G2104: 7.331,92.
>   - REV-SUPP en G0204: 10.508,43.
>   - REV-SALES en G1104: 4.415,42.
>   - ADM-HQ en G3015/G3017: 2.532,70.
> - **Odoo no separa API keys de suscripciones en Anthropic.** Todo se paga con tarjeta (Brex y Mastercard por persona). G2104 tiene 194.994,10 de Anthropic; después de restar el consumo de API keys quedan 143.859,25, que son suscripciones de Claude (Team, Max, Code).
> - **Hay desfase de facturación.** Odoo registra la factura cuando llega, no en el mes de consumo:
>   - OpenAI: el consumo supera lo contabilizado en ene, feb, jul y sep.
>   - Azure: lo supera de ene a may.
>   - Por eso el asiento conviene hacerlo sobre lo contabilizado de cada mes, repartido con el mix de consumo de ese mes.

## Método
- **Consumo:** registro `finance.ai_api_key_usage_monthly`, armado con la skill `update-ai-api-costs`.
  - Claude: export de costos por key.
  - OpenAI: export de costos más el de uso, que mapea cada key a su proyecto.
  - Azure: vista Costo por recurso, con IVA 1,19.
- **Contabilizado:** Odoo vía XML-RPC (proyecto `mcp-odoo-agendapro`, solo lectura). `account.move.line` publicadas de AgendaPro Inc en las cuentas 5.2.25, 5.2.22 y 5.2.0, por partner y mes.
- **Imputación de cada key:** con las reglas de [[Imputación contable de costos de IA (API keys)]].
- **Sheets de trabajo:**
  - [API Keys Costs 2026](https://docs.google.com/spreadsheets/d/1JHzp6l86pgBbR6rX6scnkc974nlOojlcNB_8X4s5Z0g/edit), con las pestañas Summary, API Keys Costs, Cambios en cuentas en Odoo, CECOs Odoo y Cuentas G nuevas.
  - [API Keys diferencias por cuenta 2026](https://docs.google.com/spreadsheets/d/1He9lzWONzjTofRFbz0L3F5yzHRfDlT69FkwhIOcWihE/edit).
  - [API Keys montos IA en Odoo 2026](https://docs.google.com/spreadsheets/d/15miJb3yy9eDNnah7PM32-3tYMp1zZbPu_1Wjq6yQEtQ/edit).

## Evidencia
Consumo de API keys por mes (USD con IVA):

| Ene | Feb | Mar | Abr | May | Jun | Jul | Ago | Sep |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 2.213,51 | 2.708,67 | 5.206,77 | 15.259,41 | 23.032,20 | 16.710,75 | 24.225,44 | 19.847,39 | 11.431,98 |

Contabilizado en Odoo vs consumo, ene–sep 2026 (USD):

| Cuenta | Proveedor | Contabilizado | Consumo API keys | Queda en la cuenta |
|---|---|---:|---:|---:|
| G2104 | Anthropic | 194.994,10 | 51.134,85 | 143.859,25 |
| G2004 | OpenAI | 11.482,40 | 8.447,71 | 3.034,69 |
| G0001 | Microsoft (Azure) | 60.769,31 | 61.053,56 | −284,25 |

Peso de los proveedores de modelos IA en cada cuenta, ene–sep 2026:

| Cuenta | Total | Proveedores IA | % |
|---|---:|---:|---:|
| G2104 | 317.075,90 | 196.017,21 | 61,8% |
| G2004 | 281.508,38 | 11.482,40 | 4,1% |
| G0001 | 301.587,66 | 60.769,31 | 20,1% |

## Caveats
- **Septiembre seguía abierto en Odoo** al 2 oct: faltaban facturas grandes, como AWS en G0001 y G2004.
- **Cargos IA en G2104 que no están en el registro de API keys:**
  - Google Agents 599,96, ElevenLabs 107,77, Minimax 25 y TypeSafe AI 25, que suman 757,73.
  - OpenAI 217,43 y Microsoft 47,95; en este último la glosa no dice si es Azure.
  - No se reclasifican.
- **Queda fuera un cargo de VPN** de 19,99 (abr) con glosa «para usar Claude desde Nicaragua».
- **Repo:** `agendapro-dbt`, rama `feat/ai-api-key-team-seed`, [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]], commit `5bfed446`.
