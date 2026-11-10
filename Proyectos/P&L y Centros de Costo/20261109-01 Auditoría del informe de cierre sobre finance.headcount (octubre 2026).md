---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-09
temas: [headcount, dwh, cierre-mensual]
metricas: []
---

# Auditoría del informe de cierre sobre finance.headcount (octubre 2026)

**Pregunta**: El proceso de cierre de octubre (agente de revisión de nómina de [[Nicolás Astudillo]]) levantó nueve correcciones a `finance.headcount`. ¿Cuáles eran reales, cuáles ya estaban resueltas y cuáles estaban mal diagnosticadas?

## Conclusión
> Tres de los nueve puntos eran bugs vivos en producción y se corrigieron el 9-nov con el PR #481 de dbt (`agendapro/agendapro-dbt`, contra `refactor/postgres-to-redshift`): la jerarquía concatenada de octubre, la persona duplicada entre Chile y Colombia y, como causa de fondo, que todos los tests del modelo eran `warn`. Dos puntos eran reales pero son datos que la planilla no exporta (identificador estable y sociedad pagadora). Tres puntos eran falsos o ya resueltos (TEC-TEC-HQ, cost_center vacío de enero a junio, tildes). El noveno era de nombre: la columna de empresa se llama `employer`, no `company`.

## Método
- Cada afirmación del informe se contrastó contra `finance.raw__headcount`, `finance.headcount`, el seed `headcount_cost_center_mapping` y el modelo `models/intermediate/finance/headcount.sql` en Redshift (9-nov-2026, foto de octubre).
- La corrección se validó corriendo el SQL compilado del modelo nuevo contra Redshift antes de abrir el PR: filas por mes, áreas vacías, valores de `business` fuera de lista, duplicados por clave y checksums por año del histórico anterior a octubre.

## Evidencia
| # | Hallazgo del informe | Veredicto | Detalle |
|---|---|---|---|
| 1 | Jerarquía concatenada, 162 de 204 filas | Real y grave | En octubre las pestañas de Chile, Argentina y USA pasaron a escribir `Div,Area,Subarea` (coma sin espacio). El modelo separaba por `, `, así que `division` quedó con el string entero, `area` y `subarea` vacías y `business` heredó el string: 33 valores inexistentes en el board book (p. ej. «Revenue,Customer Experience,Customer Support» con 11 personas). Colombia y México, con guion, no se afectaron. |
| 2 | Normalizar tildes | Ya resuelto | «Beauty Expansión», «Retención» y «Health non doctors» ya se traducían; no se aplicó porque la separación falló antes. |
| 3 | Cofré duplicado Chile y Colombia | Real | Colombia y Argentina escriben «Apellidos Nombres» sin coma; la clave de persona solo reordenaba el formato con coma de México, así que «Cofré Méndez Nicolás Alexander» y «Nicolás Alexander Cofré Méndez» no coincidían. People registra a cada jefe además en el país de sus reportes, así que iba a repetirse. |
| 4 | Sin identificador estable | Real, no exportado | Consolidated data tiene el RUT en la columna C; «2026 DWH Loading» no lo exporta. |
| 5 | País de residencia vs sociedad pagadora | Real, no exportado | La moneda (columna B) es el proxy de sociedad y tampoco se exporta. Birello y Aucan Sierra vienen con país por el override de «US mapping». |
| 6 | cost_center vacío ene–jun | Por diseño | El seed `headcount_classification` cubre esos meses; las 189 filas de enero clasifican por seed. Documentado en el modelo. |
| 7 | TEC-TEC-HQ sin mapeo | Falso | Está en el seed (Development, HQ); López y Durán en julio clasifican por `sheet`. «Technology,Technology,Technology» en octubre era otra vez el punto 1. |
| 8 | area vacía en ene (31), feb y mar (3) | Real, histórico | Argentina y USA escribieron solo la división; no se reconstruye sin inventar y no afecta la clasificación. |
| 9 | Columna `company` | Nombre | La columna es `employer`; `company` está reservado para clientes. |

Por qué nadie lo vio: todos los tests del modelo tenían severity `warn`, incluido el `accepted_values` de `business` que debió alertar en el cierre.

**Fix (PR #481, mergeado 9-nov):** toda coma de la jerarquía se normaliza a `, ` antes de separar; la clave de persona pasa a ser los tokens del nombre ordenados alfabéticamente (verificado sobre las 7.526 filas y 432 nombres del histórico: solo junta las copias conocidas); `business` accepted_values y un `area` not_null desde 2026-07 pasan a `error`. Resultado: octubre de 204 a 203 filas, 0 áreas vacías, 0 valores de business fuera de lista; los meses anteriores a octubre quedaron idénticos (conteos y checksums por año iguales).

**Puntos 4 y 5:** se pueden cerrar en un próximo cambio del stream de Airbyte exportando el RUT hasheado (`person_id`) y la moneda (`payroll_entity`) junto con `employer`.

## Caveats
- La fila de Cofré que sobrevive es la de Chile (Bizops Product); la de Colombia decía Bizops Tech. Si People quiere la subárea de Colombia, hay que corregirla en la pestaña de Chile.
- El informe venía con `company` porque a Nicolás se le avisó con ese nombre el 3-nov; falta decirle que es `employer`.
