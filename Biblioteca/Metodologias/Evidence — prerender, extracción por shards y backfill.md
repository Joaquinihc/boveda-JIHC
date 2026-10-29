---
type: metodologia
temas: [evidence, reportes]
creado: 2026-10-29
---

# Evidence — prerender, extracción por shards y backfill

Cómo llega un número a una página de `agendapro-dashboards-evidence` y qué cuesta cada etapa. Verificado el 29-oct-2026 en `docs/PRERENDER_POLICY.md`, `.github/workflows/k8s-extract.yml`, `.github/workflows/Deployment.yml`, `docs/extract-shard-timings.md` y `svelte.config.js` del repo.

## Dos capas, dos momentos
1. **Extracción** (`sources/dwh/**/*.sql`, Redshift): corre `evidence sources` como Jobs de Kubernetes dentro de la VPC, escribe un parquet por query y lo publica en `s3://ap-evidence-prod/_parquet-cache/cache/<key>/`. Es la única vez que se toca Redshift.
2. **Páginas** (`pages/**/*.md`, DuckDB): los bloques SQL de cada página corren sobre esos parquet. Con prerender on corren en el build y el HTML queda horneado; con prerender off corren en el navegador del usuario con DuckDB-WASM, sobre los mismos parquet.

Consecuencia: **apagar el prerender ahorra build (tiempo y memoria), no extracción.** No hay forma de ejecutar una query de Redshift al abrir una página: el navegador no llega a Redshift y en producción no hay servidor (S3 + CloudFront + Lambda@Edge solo para autenticación).

## El cron de extracción
- `Deployment.yml`: `0 2,8,14,20 * * *` UTC, 4 corridas al día. Los push a `main` re-extraen solo si cambió algo en `sources/` (la cache key es un hash de `sources/**`); en los cron la key incluye la hora, así que siempre re-extraen.
- 363 queries (29-oct-2026) en 13 shards paralelos empaquetados a ~40 min cada uno por costo medido (`docs/extract-shard-timings.md`); tope de 90 min por shard. Redshift se encola fuerte en horario hábil, por eso no se agregan más shards.
- Cada carpeta de `sources/dwh/` tiene un shard de origen; una query pesada se «pinea» en otro shard con `queries` y se saca de su origen con `exclude`. `scripts/check-shard-coverage.sh` verifica que no haya queries duplicadas ni huérfanas y que las parejas AFINIDAD (numerador y denominador de un mismo bloque) compartan shard. Un cambio solo en la matriz no re-extrae: hay que subir `CACHE_VERSION`.

## Backfill fail-soft (consolidate)
- Si a una query le falta parquet (shard caído, timeout, error), `consolidate` copia el parquet del último caché `_complete` y publica igual, marcando la tabla en `_backfilled` y en el resumen del run. La página sale con dato viejo, no con error.
- Guard: si falta más del 25 % de las queries, no publica (extracción globalmente rota).
- Los parquet rellenados se publican en el caché nuevo, así que persisten de corrida en corrida aunque el prune borre cachés de más de 3 días. Esto es lo que permitiría «enfriar» queries a propósito (sacarlas del cron diario y refrescarlas semanalmente): ver [[Apagar o enfriar reportes poco consultados en Evidence]].
- Una query que devuelve 0 filas no se rellena (es legítimo, por ejemplo el día 1 del mes).

## Prerender
- Por defecto todas las páginas prerenderizan (`entries: ['*']`), 2 rutas en paralelo, cada una con su DuckDB-WASM (runner de 2 vCPU / 7.8 GB). Apagar: `+page.ts` con `prerender = false` **y** `ssr = false` al lado del `index.md`; sin `ssr = false` la página sale en blanco en producción.
- Una página sin prerender no tiene HTML en S3: CloudFront responde el 404 con `200.html` (shell de SvelteKit) y el navegador la arma. Reproducible solo con `npm run build && npm run preview`, nunca en `npm run dev`.
- Inventario de las páginas apagadas y por qué: tabla en `docs/PRERENDER_POLICY.md` (14 el 29-oct-2026).

## Lo que no existe
- **Uso de los reportes**: ninguna fuente tiene pageviews (`docs/AGENTS_AND_SKILLS.md`, deuda 1). `/finance/team-activity` mide PRs, no visitas. Lo más barato: access logs estándar de CloudFront (distribución `E234QLHMMWA26A`) a S3, luego una extracción que cuente hits por URI y mes.

## Cómo detectar extracciones sin lector
```bash
for f in $(find sources/dwh -name "*.sql"); do q=$(basename "$f" .sql)
  grep -rqw "$q" pages/ partials/ components/ || echo "ORPHAN $f"; done
```
Regla del `CLAUDE.md` del repo: una extracción solo se borra con cero referencias en `pages/`.
