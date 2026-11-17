---
type: metodologia
temas: [okrs, dwh, evidence]
proyecto: "[[_OKRs Finanzas]]"
fuente: skill okr-manual-values-monthly-close (agendapro-dashboards-evidence)
fecha: 2026-11-17
---

# Planilla OKR — dónde se edita y cómo llega al DWH

> Fuente ejecutable: skill `okr-manual-values-monthly-close` en `agendapro-dashboards-evidence` (`.claude/skills/okr-manual-values-monthly-close/`, sobre todo `references/known-cases.md`). Si esta nota y la skill difieren, manda la skill.

## La cadena
1. **«Avance OKr»** (Google Sheets, `1s9h4l6Z0-NzXbatLpGHF0dh24O1fX11x_qF-8u4VeRw`), pestaña **«KR Mensual 2026 »** (con un espacio al final). Una fila por KR y mes. Columnas: A Mes · B Objetivo · C ID KR · G Target · H Target mensual YTD · I Target final · L Evaluación · N Owner · **P Valor actual**. **Aquí se edita.**
2. **«2026 DWH Loading»**, pestaña **«OKR's»**: es un `IMPORTRANGE` de la anterior, en la celda A1. **No se escribe aquí**: escribir una celda rompe la importación.
3. Airbyte lee el **texto visible** de cada celda → `finance.raw__okr` (todo texto) → dbt `okr.sql` limpia el número → `finance.okr`.
4. Evidence (`/general/okr`) lee `finance.okr` después de la corrida de dbt y del deploy.

## Reglas de formato (manda lo que muestra la celda)
- Coma decimal y sin separador de miles (`0,1025`, `17800000`). Con el signo `%`, dbt deja el valor en NULL, y un NULL se ve igual que no haberlo cargado.
- Cada KR tiene su escala: unos van en fracción (0,80 = 80%) y otros en puntos (NPS 60; 4,7 en tiendas). Antes de tipear, mirar el target del mismo KR.
- Si la celda muestra menos decimales que el valor, el valor llega recortado.
- Para verificar, consultar `finance.okr` del mes después de la corrida. Un NULL ahí es un problema de formato, no un olvido.
- Se pueden borrar las filas de un KR completo sin romper nada: ni Evidence ni dbt nombran los KRs de People (revisado el 17-nov con KR10.4 y KR12.5).
- La columna L (Evaluación) es solo una etiqueta: el dashboard decide en qué meses se mide un KR según si tiene target ese mes y según la columna H.

## La planilla de People
- «Avance OKr» → pestaña **«OKR's People»** es un `IMPORTRANGE` de **«OKR People - reporte Finanzas»** (`1_vNBoFyawJ06gH8b2Zup7IFbPj-XDbL2lb1ExQXoKV0`, pestaña `OKR`). El archivo es de Joaquín y lo edita Javiera Correa. Arriba van las metas; abajo, en el bloque «Valor Real», los valores.
- **No alimenta la columna P**: los valores se traspasan a mano, desde la escala 0–100 a la del KR (75 → 0,75).
- Su pestaña «Backup 11/08/2026» guarda la lista original de KRs de People.
- Desde el 17-nov, las metas de People las manda esta planilla.
- Javiera tiene otra planilla en su Drive («OKR People», `1W2-fal4YRy3co5eOoZjqBC9fvmjIu1DW1J0cfEO1MAM`) que dejó de actualizar: no usarla.

## KRs que no se le piden a nadie
- **O2 KR2.1** (rating de la app de clientes en las tiendas): lo saca Finanzas con `scripts/store_ratings.py` de la skill (promedio de App Store y Google Play ponderado por número de calificaciones; es la foto del día en que se corre).
- **O3 KR5** (Product Stability): se calcula del registro Error Health de Core Experience en Notion, detenido en la Week 30 (20-jul) al 17-nov.
- Los que la página calcula en vivo aunque la planilla diga «Manual»: la lista está en la skill.
