---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-10-06
temas: [pos, ventas, board, evidence]
metricas: []
---

# Board book sobre pos_sales_sot — impacto en ventas, attachment y payback

**Pregunta**: ¿Qué cambia en el board book si las ventas POS, el attachment B2B3 y el payback pasan a leer `dwh.pos_sales_sot` (v9 de [[Ignacio Embry]]) y cuánto se mueven los números?

## Conclusión
> - El total de unidades casi no cambia: ene–sep 2026, Chile 418 → 408 y México 160 → 152. Cambia la composición: Inbound y Stock bajan, y aparece **Others** (Chile 46 u, México 31 u), que son ventas válidas sin deal POS en HubSpot.
> - Attachment B2B3 Chile baja 1–9 pp casi todos los meses (sep-26: 46,0% → 43,7%). México no cambia desde jul-26.
> - Payback de Key Metrics, sep-26: total 12,1 → 12,1; Chile 7,3 → 7,0; México 14,4 → 15,0. Por país y mes se mueve entre −1,4 y +0,8 meses.
> - Al 19-oct el cambio está en el worktree, sin commit ni PR, a la espera del visto bueno de Pablo Lucero.

## Método
- **Dónde**: repo `agendapro-dashboards-evidence`, worktree `~/Documents/GitHub/agendapro-worktrees/pos-sales-sot-dbt`, rama `feat/board-book-pos-sales-sot` (base `d1f87f63`, 5-oct). Archivos: `sources/dwh/payments/country_pos_sot.sql` (reescrito), `pages/finance/board-book/index.md` y `quarterly/index.md`, y el documento `docs/metrics/payments/pos-sales-sot-dbt-migration.md`.
- **Contrato**: `country_pos_sot` mantiene sus columnas (`date, country_group, category, pos_units, pos_merchants`); solo agrega la categoría `Others`. `country_pos` no se tocó: lo siguen leyendo MBR, OKR, CEO dashboard y payments-revenue-monthly.
- **Reglas adoptadas**:
  - Venta = `is_valid_purchase` (factura pagada; formulario desde el 1-sep-2026).
  - Mes = `effective_sale_date`.
  - Canal por deal: ISR → New, KAM → Stock, sin deal → Others; `purchase_type = 'reactivation'` → Reactivacion.
  - Excluye `pos_additional_soft`; Tabata deja de excluirse.
  - Key Metrics «New POS (units)» y el término Payments del payback = todo menos Reactivacion. Antes el payback leía `country_pos`, una tercera población.
  - Numerador del attachment = New (ISR).
- **Slide 8**: Others como serie gris en los dos gráficos. El segundo pasó de compañías a unidades el 8-oct, a pedido de Joaquín; las soft suman 0.
- **Comparación**: las dos versiones de `country_pos_sot.sql` se extrajeron el 5-oct a la misma hora. Los bloques de la página se corrieron en DuckDB sobre el mismo caché. Regresión: los 32 bloques que dependen de `country_pos`, `country_pos_sot` o `pos_attachment_segment` corren en las dos páginas con las mismas filas.

## Evidencia
Corte: extracción del 5-oct-2026.

**Unidades por canal, ene–sep 2026** (sin reactivación salvo la última columna)

| País | Inbound | Stock | Others | Total | Reactivación (u) |
|---|---|---|---|---|---|
| Chile | 232 → 207 | 186 → 155 | 46 | 418 → 408 | 35 → 24 |
| México | 54 → 37 | 106 → 84 | 31 | 160 → 152 | 3 → 6 |

- Por mes, los totales se mueven entre −6 y +5 unidades por país.
- Contra el budget (ISR + KAM, ene–sep), Chile pasa de 166,5% a 162,5% y México de 67,2% a 63,9%.
- Reactivaciones en compañías: Chile 33 → 33, México 3 → 8. En unidades bajan porque las soft suman 0.

**Others, ene–sep 2026**

| Tipo | Chile | México |
|---|---|---|
| Primera venta sin deal | 16 ventas · 17 u | 18 ventas · 19 u |
| POS adicional detectado por factura | 18 ventas · 29 u | 8 ventas · 12 u |

- En México, 11 de las 18 primeras ventas son de comercios con 30 días o menos de alta, y en 8 el POS lo cargó el equipo AE/ISR. Parecen ventas ISR sin deal; no se marcó «¿Contrató POS?».
- En Chile, 12 de las 16 primeras ventas son de comercios con más de 90 días. Tienen perfil Stock.

**El canal de un POS adicional no se hereda.** Lo decide el deal de ese evento. De 31 POS adicionales de ene–sep, 5 tienen deal KAM y 26 caen en Others, aunque la primera venta haya sido ISR (15) o KAM (4).

**Unidades: manda el deal, no la factura.** `pos_quantity` = `cantidad_de_terminales` del deal (1 si viene vacío); sin deal, el máximo facturado. Hay 8 ventas con deal donde no coinciden, y en 7 el deal dice menos. Contradice lo dicho en [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]].

**Attachment B2B3, antes → después**

| Mes | Chile | México |
|---|---|---|
| oct-25 | 21,7% → 17,1% | 17,2% → 11,8% |
| ene-26 | 23,4% → 16,9% | 3,4% → 1,7% |
| jul-26 | 39,7% → 34,6% | 1,3% → 1,3% |
| ago-26 | 49,5% → 46,3% | 4,5% → 4,5% |
| sep-26 | 46,0% → 43,7% | 7,4% → 7,4% |

**Payback**: POS que entran al término Payments, ene–ago 2026: Chile 334 → 341, México 123 → 136. El payback variable total de sep-26 queda igual (7,6). Argentina, Colombia y Other no cambian.

## Caveats
- **Los meses cerrados pueden subir.** Una venta se valida cuando se completa el formulario, sin plazo. Al 5-oct quedaban 15 ventas de septiembre (15 u) pendientes: 7 por formulario, 4 impagas y 4 sin factura.
- **Las impagas dejan de contar**, lo que revierte el criterio del 7-sep (emisión, cualquier estado).
- **Cobertura:** la SoT solo cubre Chile y México.
- **No migradas:** activación y retención (2.4 / 2.5) siguen en `pos_sales`.
- **Las cifras se mueven:** las tablas de Ignacio se recrean en cada corrida de dbt. Este análisis es al 5-oct.
- La definición completa, con diagramas, está en Notion: [Definición venta de POS Ignacio Embry](https://app.notion.com/p/3f1ab34c233b80a0b057fa52491c7b36).
