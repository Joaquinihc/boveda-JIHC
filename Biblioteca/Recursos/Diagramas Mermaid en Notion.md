---
type: recurso
temas: []
fuente: sesión de Claude Code del 2026-11-18 (páginas de Notion de T-056)
fecha: 2026-11-18
---

# Diagramas Mermaid en Notion

Notion corta el texto de los cuadros de un diagrama Mermaid: por defecto da a cada cuadro un ancho de texto fijo de unos 100 px (13 a 15 caracteres) y recorta lo que sobra.

## Receta que funciona
- Primera línea del bloque:

```
%%{init: {"flowchart": {"padding": 16, "nodeSpacing": 40, "rankSpacing": 70, "wrappingWidth": 260}}}%%
```

- Cortar cada etiqueta a mano con `<br/>`, con líneas de 26 caracteres como máximo. Sin `wrappingWidth`, el máximo es 14.
- `flowchart` clásico, etiquetas entre comillas y títulos de `subgraph` cortos, en una línea.
- Para ordenar, agrupar con `subgraph` los cuadros de un mismo tema (por ejemplo, uno por vertical).
- Después de pegar el código hay que pasar el bloque a «Vista previa» a mano.
- 2026-11-19 · Un bloque nuevo, o una hoja reescrita completa por la API, queda en modo código. Si se edita el bloque que ya existe (buscar y reemplazar, línea por línea), conserva la vista previa. El método completo para documentar en Notion está en la skill `documentar-en-notion` de Claude Code.

## 2026-11-19 · Validar antes de pegar
Lo aprendido al armar 15 diagramas para «Métricas en conjunto con Reservo»:
- **Largo de línea**: un script recorre cada etiqueta entre comillas, la corta por `<br/>` y avisa si un trozo pasa de 26 caracteres; también revisa que la primera línea sea el `init`.
- **Sintaxis**: `mermaid.parse` (mermaid 10.9.1 en Node, con jsdom para `window` y `document`) encuentra los errores antes de pegar en Notion.
- **Render local**: Chrome headless sobre un HTML con `mermaid.min.js` sirve para ver la forma general, pero **no reproduce el corte de texto de Notion**: el visto bueno final se da en Notion.
- **Líneas de tiempo**: `flowchart TB` con `direction LR` dentro de cada `subgraph` y `A ~~~ B` entre subgraphs; sin eso, los periodos se apilan en vertical.
- **Mapas grandes**: muchos subgraphs apilados se enredan. Funciona mejor agrupar los orígenes en pocos nodos que apuntan a pocos destinos.
- **Paleta** (la misma en todos los diagramas; cada clase significa lo mismo en todas las hojas):

```
classDef disp fill:#dbeafe,stroke:#1d4ed8,color:#0b1f4d
classDef sale fill:#dcfce7,stroke:#15803d,color:#052e16
classDef limite fill:#fef9c3,stroke:#a16207,color:#422006
classDef nosale fill:#fee2e2,stroke:#b91c1c,color:#450a0a
classDef pend fill:#f3f4f6,stroke:#6b7280,color:#1f2937,stroke-dasharray:4 3
classDef neutro fill:#f3f4f6,stroke:#6b7280,color:#1f2937
```

`disp` = dato disponible · `sale` = sale · `limite` = sale con límites · `nosale` = no sale · `pend` = pendiente de otro · `neutro` = contexto.

## Lo que no sirve
- `"htmlLabels": false`: deja el diagrama ilegible.
- Rellenar con `&nbsp;` o subir `padding`: el cuadro crece, pero el texto se corta igual.

## Obsidian
Sin probar. Obsidian usa Mermaid estándar, así que la misma sintaxis debería servir y `wrappingWidth` no estorba. Falta verificar si allí se produce el mismo corte.
