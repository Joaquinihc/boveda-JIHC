---
type: tarea
estado: pendiente
prioridad: media
tiempo:
tipo: gestion
origen: manual
proyecto:
temas:
  - evidence
  - reportes
fuente:
fecha_limite:
creado: 2026-10-29
revisar: false
id: T-110
---

# Apagar o enfriar reportes poco consultados en Evidence

Sacar del catálogo de `agendapro-dashboards-evidence` los reportes que nadie consulta y bajar el costo de extracción de los que se consultan poco. Aplica a todo Evidence, no solo a Payments.

Conclusión que ordena el trabajo (29-oct, verificado en el repo): **prerender off no cambia la extracción**. Una página sin prerender no consulta Redshift al abrirse: el navegador corre DuckDB-WASM sobre los mismos parquet que produce el cron. Redshift no es alcanzable desde el navegador y en producción solo hay S3 + CloudFront. Así que «ejecutar las queries al renderizar» no es una opción; la única palanca de costo es cuándo corre cada query en el cron. Cómo funciona la plataforma: [[Evidence — prerender, extracción por shards y backfill]].

Evaluación completa en el repo: `docs/superpowers/specs/2026-10-28-pos-sot-resto-evidence-design.md`, §7 «Frente D» (rama `feat/pos-sot-country-pos-attachment`, commit 424e0bd0).

## Subtareas
- [ ] 1. Pedir a DevOps los access logs estándar de CloudFront (distribución `E234QLHMMWA26A`) hacia S3. Sin tráfico medido, toda lista de «reportes que no se consultan» es una suposición. Listo cuando los logs estén cayendo en un bucket.
- [ ] 2. Borrar en un PR chico las 3 extracciones sin ningún lector: `product/activation_weekly/registration_funnel_daily.sql`, `revenue/okr_ai_attachment.sql`, `revenue/okr_kr_budget.sql`. Listo cuando esté mergeado.
- [ ] 3. Con 4 semanas de logs, clasificar el catálogo por sección (caliente / frío / borrar) con Pablo Lucero, Miguel Cárdenas e Ignacio Embry. Listo cuando cada página tenga su nivel decidido.

## Notas

### 2026-10-29 · Método propuesto (Claude Code, pendiente de que Joaquín lo piense)
Tres niveles por página:

| Nivel | Página | Sus extracciones exclusivas |
|---|---|---|
| Caliente | Sin cambio | Cron de 4 veces al día, como hoy |
| Frío | Publicada, prerender off | Pasan a un shard `cold` que corre solo en un cron semanal o a mano; el resto de las corridas las rellena `consolidate` desde el caché anterior |
| Borrar | Se elimina con su entrada en `components/dashboards.js`, la tarjeta del hub y las referencias en docs | Se borran si no les queda lector (`grep -r` en cero) |

Cambios para el nivel frío (todos en el repo, sin DevOps): (1) shard `cold` en la matriz de `.github/workflows/k8s-extract.yml` con `queries` explícitas y `exclude` en los shards de origen, bump de `CACHE_VERSION`; (2) condición de corrida: `schedule` de un día fijo o `workflow_dispatch` con input; (3) el guard del 25 % del backfill no cuenta las frías y las lista aparte de las «stale por fallo»; (4) una query fría nunca comparte bloque de página con una caliente (regla AFINIDAD de `scripts/check-shard-coverage.sh`: ratio con numerador fresco y denominador viejo es un número inventado); (5) columna `extracted_at` en cada query fría y «Datos al <fecha>» en la página; (6) filas nuevas en `docs/PRERENDER_POLICY.md` y `docs/extract-shard-timings.md`.

Secuencia completa: D0 borrar MBR (va en el PR A de [[Migrar el resto de Evidence a pos_sales_sot]]) · D1 huérfanas (subtarea 2) · D2 logs (subtarea 1) · D3 extracción sobre los logs + página interna `/general/usage/` con hits por reporte y mes · D4 clasificar (subtarea 3) · D5 shard frío · D6 PRs de borrado por sección, cada uno con la tabla de extracciones que se van.

Candidatos iniciales a frío por estructura, no por medición: lo que sobreviva de `pages/finance/draft/` (4 de 6 sin cambios desde junio de 2026), `fintech/pos-analytics` (11 queries `propay_*` propias), `fintech/payments-retention` (4 propias), `finance/mx-gateway` (7 propias). Para Fintech conviene clasificar después del PR C de la migración a la SoT: enfriar o borrar una página que todavía lee `pos_sales2` es trabajo que se tira.

Alternativa sin shard nuevo: bajar el cron global de 4 a 2 corridas diarias. Afecta a todos los reportes por igual; es decisión de negocio.

### Pendientes menores
- 2026-10-29 · D3 (página de uso) y D5 (shard frío) son pasos para después de la subtarea 3; se suben a subtareas cuando la clasificación esté decidida.
- 2026-10-29 · Las 14 páginas con prerender off de hoy (board book, OKR, card-on-file, downgrades, drafts…) se consultan o son pesadas para el build; apagarles el prerender no les cambió la extracción.

## Historial
- 2026-10-29 · Creada desde Claude Code (pendiente por pedido de Joaquín) tras evaluar el frente en el repo de Evidence.
