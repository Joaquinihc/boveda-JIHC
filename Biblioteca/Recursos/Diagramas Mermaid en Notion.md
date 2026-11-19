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

## Lo que no sirve
- `"htmlLabels": false`: deja el diagrama ilegible.
- Rellenar con `&nbsp;` o subir `padding`: el cuadro crece, pero el texto se corta igual.

## Obsidian
Sin probar. Obsidian usa Mermaid estándar, así que la misma sintaxis debería servir y `wrappingWidth` no estorba. Falta verificar si allí se produce el mismo corte.
