---
type: reunion
fecha: 2026-11-26
titulo: "Agente de consolidación y carga semanal del cash flow de Odoo (Rafa)"
personas: []
proyectos: []
temas: [odoo, dwh, evidence]
fuente: https://app.notion.com/p/424ab34c233b81f4bbaaf12d39e9f926
---

# Agente de consolidación y carga semanal del cash flow de Odoo (Rafa)

**Acuerdos** (máx 5)
- El agente de consolidación de Rafael Rodríguez no crea un cash flow consolidado en Odoo: exporta a Excel (igual para P&L y balances). Joaquín lo editará en vez de crear uno nuevo: hoy consolida Grupo Reservo, Grupo AgendaPro y ambos, y falta agregar cada compañía por separado (Evidence necesita consolidado, filtro AgendaPro/Reservo/ambos y cada sociedad).
- El cash flow de Odoo se carga al DWH en paralelo al antiguo (planilla «Cash Flow Causal 2025» de Rafael, base de Evidence), que luego se elimina.
- Un agente en el computador de Joaquín cargará el cash flow de Odoo a Google Sheets (y de ahí al DWH) cada lunes a las 10:30: solo mes actual y anterior; para periodos previos genera una lista de cambios pendientes sin aplicarlos (recargar toda la historia tarda ~35 min y aún se modifican periodos pasados).
- Abierto: qué tipo de cambio usar en DWH y cash flow (AgendaPro y Pablo usan Google Finance; Nico, bancos centrales). Rafael dice que debe resolverse con Pablo y Nico y que movería bastante los números.
- Abiertos: por qué el saldo inicial de ene-2026 de México usa el tipo de cambio del 1 o 2 de enero y no el del 31-dic-2025; y las transferencias del 7-oct en Reservo, «en suspenso» (Joaquín cree que debe explicarlas Pablo).

**Mis acciones**
- [ ] Levantar el PR con los cambios al agente de consolidación (el que quedó con el nombre «Rule 96») para que lo revise Rafael (comprometido para el 26-nov)
- [ ] Enviar por escrito a Rafael Rodríguez el problema del tipo de cambio de México (saldo inicial de ene-2026)
- [ ] Dejar corriendo el agente semanal (lunes 10:30) que carga el cash flow de Odoo a Google Sheets para el DWH

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/424ab34c233b81f4bbaaf12d39e9f926) (título en Notion: «Rafa - Joaco»). Se leyó «UDU» como Odoo y «MCP-UDU» como el repo de Odoo MCP. «Pablo» y «Nico» se nombran sin apellido y no se linkearon; Rafael Rodríguez no tiene ficha.
