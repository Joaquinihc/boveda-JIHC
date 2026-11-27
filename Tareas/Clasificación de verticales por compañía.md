---
type: tarea
estado: pendiente
prioridad: media
tiempo:
tipo: tabla-dwh
origen: manual
proyecto:
temas:
  - ai
  - revenue
  - dwh
  - definiciones
fuente:
fecha_limite:
creado: 2026-11-16
revisar: false
alcance: "Vertical y macro nicho por compañía: un solo método (company_attribute_macro_niche), overrides por lote acordados con las verticales (Beauty, Medspa, Health), retiro de dwh.company_vertical y clasificador IA de n8n"
id: T-125
---

# Clasificación de verticales por compañía

Dejar un solo método para asignar vertical y macro nicho a cada compañía: la cadena `company_attribute_macro_niche` → `company_attribute_vertical` → `company_attributes`. Es la que leen `dwh.mrr_ai`, `dwh.mrr_saas`, `dwh.payments_revenue` y el Board Book. Las compañías que una vertical atiende y el DWH asigna a otra se corrigen por lotes con filas en `macro_niche_override`. Incluye retirar `dwh.company_vertical`, la clasificación paralela que construyó Jesús Alcántara.

## Subtareas
- [ ] PR dbt con el primer lote de overrides: las 15 compañías de Sabrina más las de Other que confirmen en el hilo. Listo cuando la rama esté pusheada y se haya pedido autorización para abrir el PR.
- [ ] Enviar los DM a Jesús Alcántara (cambio de la fase 3 en company-management, con el párrafo del clasificador IA corregido) y a Israel (estado del workflow de n8n). Listo cuando estén enviados.

## Notas

### 2026-11-16 · Primer lote: 15 compañías de Sabrina y 12 en Other
- Hilo en #vertical-business-units-ai: https://agendapro.slack.com/archives/C0APRT9SPLZ/p1794844098105129. Joaquín pidió revisión a Sabrina, Vale Lopez y Fernanda Santibáñez. En el hilo van las 15 compañías, las 12 en Other y qué tabla usar.
- Hallazgos y cifras: [[Claude Code — Conciliación AI revenue Beauty y reasignación de verticales]].
- Las 15 van a `macro_niche_override` con `macro_niche = 'Beauty'`:
  - Hoy en Medspa: 145405, 80394, 223172, 451897, 400514, 581855, 551286, 552764, 484875, 42275, 348779.
  - Hoy en Health: 174637, 343595, 429284.
  - Hoy en Other: 37433.
- Query de impacto: `_analisis/2026-11-beauty-ventas-por-facturar/impacto_reclasificacion_15.sql`.
- Plan de 4 fases (macro nicho único): `_analisis/2026-11-macro-nicho-unico/resumen-macro-nicho-unico.md`.

### Pendientes menores
- 2026-11-16 · Sabrina revisa con los KAM los 6 comercios sin setup en Chargebee. También confirma si el fee de implementación se cobró con el addon `agente_AI` recurrente (JAGA 210369, Oh My Glam! 332134, Glow Girl 195663).
- 2026-11-16 · Pedir a KAMs y facturación que el fee de implementación se cobre con el addon one-time. Si va en el recurrente, infla el MRR del mes de la venta y genera churn falso al mes siguiente.
- 2026-11-16 · Budget de IA de Beauty de octubre: 35,5k oficial (`dwh.budget_2026_revenue`) contra 40k de Sabrina. El borrador viejo en el DM con Sabrina dice «2/3 IA» para el pack Pro AI, que no calza (por componentes: 43% de la línea del plan, 75% por profesional extra). No enviarlo tal cual.
- 2026-11-16 · Reparar el clasificador IA de n8n (dueño Nicolás Cofré; Israel puede saber el estado). No carga desde el 26-mar-2026 (fase 1).
- 2026-11-16 · Avisar a [[Pablo Lucero]] antes de cada lote: la vertical reescribe el histórico de MRR por vertical y el Board Book. Confirmar si para el primer lote basta con el mensaje en el canal.

## Historial
- 2026-11-16 · Creada desde Claude Code (conversación con Joaquín): conciliación del AI revenue de Beauty con Sabrina y primer lote de overrides.
- 2026-11-17 · Mencionada de nuevo en [[2026-11-13 Preparación de la presentación de resultados de octubre (Pablo)]]: «hablar con Sabri por la diferencia de Beauty en septiembre-octubre» (ya la cubre la conciliación del 13 al 16-nov) (Reuniones)
- 2026-11-20 · Mencionada de nuevo en [[2026-11-17 FP&A Weekly]]: «sobreescribir con un PR en dbt la vertical de las compañías que pidió Sabri, una vez que confirme Vale López» (subtarea del primer lote). Acuerdo: se mantiene la definición de macro nichos, los cambios van por PR en dbt con fecha, quién y de qué a qué, y Company Management queda solo de lectura (Reuniones)
- 2026-11-26 · Pasada a Notion: [Clasificación de verticales por compañía](https://app.notion.com/p/424ab34c233b81068559eb77f4d904e2). Desde hoy se sigue allá; esta nota queda como archivo (Joaquín en conversación)
