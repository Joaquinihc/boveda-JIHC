---
type: analisis
proyecto: ad hoc
estado: cerrado
temas: [leads, hubspot]
fecha: 2026-08-06
metricas: ["[[Lead]]"]
---

# Leads B2B3+ sin owner — Agosto 2026 (1–6 ago)

**Fuente:** `dwh.int_hubspot_leads_flat` (universo del tablero Evidence "Asignación de Leads" de Pablo Lucero: `effective_owner_id IS NULL`, B2B3+ por `numemployees`, mes por `recent_conversion_date`). Emails en texto plano resueltos vía `dwh.hubspot_contacts`. Corte: 2026-08-06.

**Veredicto general:** 13 contactos sin owner en los 4 países core. **8 son pruebas internas confirmadas**, 2 creados hoy (dentro de la ventana normal de asignación), 2 sin resolver por lag del sync DWH, y **solo 1 es un lead real accionable → Meli Rosales (Argentina)**, enviado a #leads-sin-asignar.

---

## 🇨🇱 Chile (6)

| Contact ID | Email | Nombre | Rubro | Empleados | Source | Creado | Veredicto |
|---|---|---|---|---|---|---|---|
| 239432051805 | diego.romero.v13+agendapro@gmail.com | Diego Romero | barbería | 3-5 | Direct | 08-02 | 🧪 Prueba |
| 228563967373 | *(email sin sincronizar en DWH)* | — | centro de estética | 3-5 | Direct | 08-03 | ❓ Sin resolver (patrón = prueba) |
| 239657708693 | fernandoherrera@agendapro.com | Fernando Herrera | salón de belleza | 6-15 | Paid Search | 08-03 | 🧪 Prueba |
| 239646289465 | andressureda@agendapro.com | Andrés Sureda | barbería | 3-5 | Paid Search | 08-04 | 🧪 Prueba |
| 240091929144 | emilio+col1@agendapro.com | Emilio Latorre | barbería | 3-5 | Direct | 08-05 | 🧪 Prueba |
| 226120111782 | emilio+ar2@agendapro.com | Emilio Latorre | barbería | 3-5 | Organic | 06-03 (reconv. 08-05) | 🧪 Prueba |

## 🇲🇽 México (3)

| Contact ID | Email | Nombre | Rubro | Empleados | Source | Creado | Veredicto |
|---|---|---|---|---|---|---|---|
| 239704092270 | alanortiz+lppilates@agendapro.com | Alan Ortiz | pilates | 3-5 | Paid Search | 08-03 | 🧪 Prueba (LP pilates) |
| 239916882720 | *(email sin sincronizar en DWH)* | — | centro de estética | 3-5 | Paid Search | 08-04 | ❓ Sin resolver |
| 240335089547 | *(email sin sincronizar en DWH)* | — | barbería | 6-15 | Paid Search | 08-06 | ⏳ Creado hoy — ventana normal de asignación |

Nota: acá no aparece rubro "otro" porque el universo del tablero ya lo excluye. Los 10 rubro-otro sin owner que vemos en `fct_lead_conversions` son un grupo distinto que por regla no se asigna.

## 🇨🇴 Colombia (1)

| Contact ID | Email | Nombre | Rubro | Empleados | Source | Creado | Veredicto |
|---|---|---|---|---|---|---|---|
| 240122750646 | emilio+col2@agendapro.com | Emilio Latorre | barbería | 3-5 | Direct | 08-05 | 🧪 Prueba |

## 🇦🇷 Argentina (3)

| Contact ID | Email | Nombre | Rubro | Empleados | Source | Creado | Veredicto |
|---|---|---|---|---|---|---|---|
| 239543610993 | meliiirosales16@gmail.com | Meli Rosales | salón de belleza | **6-15** | Social Media | 08-03 | ⚠️ **LEAD REAL — asignar** (3 días en lifecycle `lead`, nunca llegó a MQL) |
| 240091978361 | emilio+ar3@agendapro.com | Emilio Latorre | barbería | 3-5 | Direct | 08-05 | 🧪 Prueba |
| 240300389385 | *(email sin sincronizar en DWH)* | — | barbería | 6-15 | Direct | 08-06 | ⏳ Creado hoy — ventana normal de asignación |

---

## Contexto (por qué el tablero muestra ~9,8% de "desasignación")

1. **Emails internos de prueba entran al universo** del Evidence (los tests del equipo post-incidente del workflow del 1–3 ago). En julio Chile: 30 de los 41 "sin dueño" eran pruebas.
2. **Otros LATAM** (42 sin owner el 1–5 ago) no rutea al equipo ISR core.
3. El **incidente real del workflow** (cambio del viernes 31-jul, 164 leads sin asignar el 03-ago) fue revertido por Alex el mismo día y los leads se reasignaron — el snapshot del Evidence aún lo tenía fotografiado.

**Desasignación real de agosto en MX/CL/AR/CO: 1 lead (Meli Rosales, AR).**

HubSpot: https://app.hubspot.com/contacts/2356021/record/0-1/239543610993/
