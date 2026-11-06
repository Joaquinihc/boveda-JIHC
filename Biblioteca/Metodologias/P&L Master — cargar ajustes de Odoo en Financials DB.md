---
type: metodologia
temas: [pnl, odoo, cierre-mensual]
proyecto: "[[_P&L y Centros de Costo]]"
fuente: corrección del P&L Master 2026 del 5-nov-2026 (Claude Code con Joaquín)
fecha: 2026-11-05
---

# P&L Master — cargar ajustes de Odoo en Financials DB

Cómo modificar o agregar filas en la hoja Financials DB del P&L Master sin romper fórmulas ni contar nada dos veces. Sale de la corrección de diferencia de cambio, IMSS y Brex del 5-nov-2026: [[20261105-01 Diferencia de cambio y cashback en el P&L Master 2026 (F1003, F1004 y G4004)]] y [[20261105-02 IMSS y nómina de México en el P&L Master 2026]].

## Estructura de Financials DB
- **Columnas que se escriben (A a I):**
  - A Country, B Cost Center, C Code, D Detail.
  - E Amount, en moneda local, con el gasto en negativo. F Currency.
  - G Date: primer día del mes, formato «mmm yyyy».
  - H Comment («Monthly») e I Type («Real» o «Reservo»).
- **Columnas con fórmula (J en adelante):**
  - J: tasa de cambio, desde la pestaña «Exchange rates», que la trae por IMPORTRANGE.
  - K: monto en USD (E / J).
  - U y V: nombre de la cuenta y categoría del P&L, desde Mappings.
  - Estas columnas no se escriben a mano: se copian de una fila existente.
- **Cómo suman las hojas del P&L:** con SUMIFS sobre K, por categoría (V), fecha (G) y Type (I). P&L (current FX) toma «Real» y P&L (Reservo) toma «Reservo». P&L (AP+Reservo) también filtra solo «Real» (pendiente de revisar).
- **Códigos nuevos:** la categoría sale de Mappings. Un código nuevo (por ejemplo, F1004) necesita su fila en Mappings antes de cargar filas con él. Si no la tiene, la fila no cae en ninguna línea del P&L.

## Pasos
1. **Respaldar:** hacer una copia del archivo en Drive antes de tocar nada. La copia muestra los montos en USD solo después de dar «Permitir acceso» al IMPORTRANGE de «Exchange rates».
2. **Mappings primero,** si hay códigos nuevos o que cambian de categoría.
3. **Ir por etapas:** un país o un bloque por vez. Antes y después de cada etapa, medir en P&L (current FX) el EBITDA, Financial expenses, Taxes y Net income por mes. El cambio tiene que ser el esperado, con ±1 USD de redondeo, y 2021–2025 no se puede mover.
4. **Filas nuevas al final de la hoja:** escribir A a I y copiar formato y fórmulas desde una fila modelo **visible**.
   - Si hay un filtro activo (por ejemplo, el de Code) y la fila modelo está oculta, la copia falla: «no se admite en un intervalo con una fila filtrada».
   - Por API, el lote es atómico: si falla, no se aplica nada.
5. **Montos exactos:** el export de `agent_pl_export` redondea a entero los montos menores a 1.000. Para cuadrar al peso, tomar el monto de Odoo, no el del CSV.
6. **Ubicar las filas por país, detalle, fecha y monto, no por número.** Los números cambian cuando alguien inserta o borra filas.

## Antes de agregar líneas de Odoo: comparar el total por área
Comparar cuenta por cuenta engaña cuando el Master tiene una estructura de cuentas distinta a la de Odoo.

- **Ejemplo (México, julio 2026):** la nómina original trae las cargas del empleador dentro de las líneas de remuneraciones (601.01.05x). Por eso la 601.26.001 (IMSS) del Master parecía corta frente a Odoo, y agregar el IMSS por área lo contó dos veces.
- **Regla:** antes de agregar, sumar todas las cuentas de nómina por área y mes, en el Master y en Odoo. Solo se agrega lo que falta en el total.

## Revisar la categoría antes de afirmar el efecto
Antes de decir si un código mueve el EBITDA o solo el resultado, hay que mirar su categoría en Mappings.

- G4009 (Impuesto sobre Nómina) está en Taxes, debajo del EBITDA, y mueve el Net income.
- F1003 y F1004 (diferencia de cambio) van en Financial expenses desde el 5-nov-2026.
- G4004 (cashback) y G4011 (otros ingresos) están los dos en Other Corporate, dentro del EBITDA. Mover algo entre ellos no cambia el EBITDA.
