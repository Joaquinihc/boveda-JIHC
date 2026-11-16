---
type: borrador
origen: Claude Code
fecha: 2026-11-16
temas: [ai, revenue, mrr]
revisar: true
---

# Claude Code — Conciliación AI revenue Beauty y reasignación de verticales

- **Diferencia con Sabrina (13-nov)**:
  - Su back office reportaba 505,9k de ARR de IA de Beauty (507,4k al 16-nov). El DWH da 341,8k facturado (MRR 28,5k); `dwh.mrr_ai` da 341,1k.
  - La diferencia es contratado contra facturado: 162,5k de ventas de KAM sin facturar (sale de su planilla de ventas, con menos de 60 días desde el fee) y 3,1k del pack Pro AI de Argentina.
  - Finance cuenta solo lo facturado y pagado.
- **Las 82 ventas «por facturar»** de Sabrina suman 13.543 USD al mes. Lo sólido son 57 de Beauty por 9.985 USD (~120k de ARR). Detalle en `_analisis/2026-11-beauty-ventas-por-facturar/detalle_por_compania.csv`.
- **15 compañías que atiende Beauty y el DWH asigna a otra vertical** (Medspa 11, Health 3, Other 1). Sabrina pidió pasarlas a Beauty. Impacto en AI Revenue (MRR + setup, `dwh.mrr_ai` al 16-nov):
  - Octubre: Beauty +1.115, Medspa −657, Health −389, Other −69.
  - Noviembre: Beauty +1.806 (670 recurrentes, ~8k de ARR), Medspa −1.648 (−7,6%), Health −89, Other −69.
  - Antes de octubre no cambia.
  - En revenue total (SaaS + IA + Payments), Beauty gana entre 0,9k y 3,5k al mes desde enero; es menos del 1% por vertical.
- **Quién fija hoy su macro nicho**:
  - El clasificador IA decide en 9. En pH2 y Hollywood, HubSpot dice Beauty, pero gana la IA con Spa.
  - HubSpot decide en las otras 6.
  - `dwh.company_vertical` ya dice Beauty para 14 de las 15.
- **12 compañías en Other con IA desde septiembre** (1.069 USD en noviembre): se preguntó en el canal qué vertical las atiende. Santo Glamour (111970) pasó sola de Other a Beauty el 16-nov.
- **Fee de implementación cobrado con el addon recurrente**:
  - JAGA, Oh My Glam! y Glow Girl tuvieron `agente_AI` recurrente solo en octubre. Eso infla el MRR un mes y genera churn falso al siguiente.
  - En Beauty fueron 353 USD en octubre. En septiembre hubo 4 comercios de Medspa por 1.299 USD con el mismo patrón, aunque algunos pueden ser bajas reales.
- **Pack `proaibeauty2026q4-ar`**: hoy cuenta 100% como SaaS en el DWH. Sabrina dice 1/3 software y 2/3 IA; por componentes, la IA es 43% de la línea del plan y 75% de cada profesional extra.
- Frente: [[Clasificación de verticales por compañía]].
