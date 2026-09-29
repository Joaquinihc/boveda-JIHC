---
type: analisis
proyecto: "[[_Churn y Graduación]]"
estado: activo
temas: [churn, graduacion]
fecha: 2026-09-11
autor: Pablo Santa Inés (CX)
personas: ["[[Pablo Santa Inés]]"]
---

# Correcciones al documento «Cambio definición Graduación compañías»

**Para:** Joaquín Herrera (BizOps) · **De:** Pablo Santa Inés (CX) · **11-sep-2026**
**Documento revisado:** [Cambio definición Graduación compañías](https://app.notion.com/p/3d5ab34c233b80e784e3df78395fe00c) y su subhoja [Retención y graduación](https://app.notion.com/p/3d5ab34c233b806c8914d15145ae46c8)

---

## Resumen

El universo y el churn total del documento son correctos: reproduje las 3.239 sedes B2B3 y las 10.393 totales exactas. Las correcciones son tres y todas están en las dos columnas atribuidas a CX.

1. **La columna «Grad. Pablo» no es la definición de CX.** Le falta el filtro de uso semanal, que sí está en producción desde el 21-ago-2026.
2. **«Grad. Pablo +5/sem» cambia dos cosas a la vez**, y la que explica el 80% del efecto no es el umbral de 5 sino evaluar el uso en un solo mes.
3. **La conclusión de retención se cae al corregir eso.** Con el 4º pago como piso, el filtro de ≥5 por semana no separa nada: 70% de logos a 12 meses contra 70%.

La recomendación del documento —adoptar el filtro de 5 reservas semanales como criterio de consolidación— se apoya en el punto 3 y no se sostiene una vez corregida la ventana.

---

## 1. La definición de CX lleva filtro semanal, y no está en el documento

La tabla «Las definiciones» describe la metodología de CX como «100 reservas desde el primer pago Y 4º pago», con «Filtro de uso post-hito: No». Eso está incompleto.

La definición que corre en producción exige **tres** condiciones cumplidas en un mismo mes:

1. ≥100 reservas acumuladas (`GRADUATION_BOOKINGS`),
2. ≥4 ciclos de suscripción pagados (`GRADUATION_PAID_MONTHS`, `dwh.mrr.is_active_and_paid`),
3. reservas en los **4 bloques de 7 días** de ese mes, ≥1 reserva en cada uno (`GRADUATION_WEEKS`).

La fecha de graduación es el **primer mes** en que las tres se cumplen juntas. Es un piso, no una foto del mes 4.

Fuente única del criterio: `cx-apps/apps/cx-head-app/data_layer.py`, función `graduation_ctes()`. El gemelo que alimenta el reporte del comité es `cx-apps/_shared/ceo_report/reclass.py`; hay tests que comparan los dos.

Los bloques son de días del mes y no semanas ISO a propósito: una semana ISO puede aportar un solo día al mes, y entonces «cuatro semanas» dependería del día en que cayó el 1°.

**Efecto de la omisión (B2B3, sedes, ene–ago 2026):** la columna publicada dice 1.566; la definición real da 1.463. La fila «Grad. Pablo reporte» (1.561) queda pegada a la versión sin filtro, así que probablemente esa fila tampoco lo está aplicando.

---

## 2. El «+5/sem» mezcla el umbral con la ventana, y la ventana es la que pesa

El documento define el escenario como «≥5 reservas en cada una de las 4 semanas **del mes del 4º pago**», con evaluación única. Ese segundo cambio es el que produce casi toda la caída, y no está aislado en ninguna columna.

Una cuenta que consolida en el mes 6 y sostiene el ritmo dos años queda fuera para siempre, porque se la mide una sola vez en el mes 4. El 4º pago es el **desde**, no la foto.

### Descomposición — B2B3, sedes

| Regla (4º pago = piso) | Abr | Ene–ago |
|---|---:|---:|
| Sin filtro semanal (= «Grad. Pablo» del doc: 152 / 1.566) | 143 | 1.548 |
| ≥1 por semana, móvil — **definición real de CX** | 135 | 1.463 |
| ≥3 por semana, móvil | 130 | 1.418 |
| ≥5 por semana, móvil | 124 | 1.372 |
| ≥5 por semana solo en el mes 4 (= «+5/sem» del doc: 95 / 1.169) | 99 | 1.148 |

De las 57 sedes que en abril separan 152 de 95: unas 45 las produce la ventana fija y 11 el umbral. Corregida la ventana, subir de 1 a 5 cuesta 11 sedes en abril y 91 en el año, no 397.

### Serie mensual completa — B2B3, sedes

| Mes | Churn total | Sin filtro | ≥1/sem móvil | ≥5/sem móvil | ≥5 solo mes 4 |
|---|---:|---:|---:|---:|---:|
| Ene | 374 | 193 | 185 | 178 | 149 |
| Feb | 374 | 190 | 178 | 168 | 147 |
| Mar | 372 | 180 | 170 | 161 | 130 |
| Abr | 341 | 143 | 135 | 124 | 99 |
| May | 383 | 170 | 160 | 151 | 119 |
| Jun | 493 | 242 | 226 | 212 | 183 |
| Jul | 465 | 220 | 210 | 199 | 174 |
| Ago | 437 | 210 | 199 | 179 | 147 |
| **Total** | **3.239** | **1.548** | **1.463** | **1.372** | **1.148** |

### Serie mensual completa — todas las compañías, sedes

| Mes | Churn total | Sin filtro | ≥1/sem móvil | ≥5/sem móvil | ≥5 solo mes 4 |
|---|---:|---:|---:|---:|---:|
| Ene | 1.085 | 335 | 320 | 297 | 231 |
| Feb | 1.203 | 342 | 319 | 283 | 228 |
| Mar | 1.324 | 337 | 311 | 277 | 211 |
| Abr | 1.299 | 331 | 310 | 275 | 208 |
| May | 1.402 | 386 | 361 | 321 | 249 |
| Jun | 1.421 | 432 | 402 | 353 | 287 |
| Jul | 1.376 | 410 | 382 | 340 | 267 |
| Ago | 1.284 | 431 | 405 | 347 | 263 |
| **Total** | **10.393** | **3.004** | **2.810** | **2.493** | **1.944** |

### Todas las compañías, MRR USD churneado como graduado

| Mes | ≥1/sem móvil | ≥5/sem móvil | ≥5 solo mes 4 |
|---|---:|---:|---:|
| Ene | 13.714 | 12.918 | 10.390 |
| Feb | 12.462 | 11.450 | 9.448 |
| Mar | 12.415 | 11.436 | 9.262 |
| Abr | 12.074 | 10.897 | 8.215 |
| May | 14.219 | 13.241 | 10.407 |
| Jun | 18.515 | 16.911 | 14.431 |
| Jul | 16.132 | 14.955 | 12.080 |
| Ago | 17.176 | 15.268 | 12.476 |
| **Total** | **116.707** | **107.076** | **86.709** |

---

## 3. Corregida la ventana, el filtro de 5 por semana no separa nada

Este es el punto que cambia la recomendación. El documento sostiene que «el filtro de 5 reservas por semana es el único que separa»: 75% de logos a 12 meses contra 50% de las excluidas. Ese contraste es real, pero mide la ventana fija, no el umbral.

Cohortes de graduación ene-2023 a ago-2025, seguidas 12 meses sobre `dwh.mrr_saas`:

### Todas las compañías

| Definición | Compañías | Logos m12 | GDR m12 |
|---|---:|---:|---:|
| A · ≥1/sem móvil (CX vigente) | 9.408 | 70% | 69% |
| B · ≥5/sem móvil | 8.689 | 70% | 70% |
| C · ≥5/sem solo mes 4 (doc) | 6.981 | 70% | 71% |
| D · excluidas por B (gradúan en A, no en B) | 593 | 20% | 21% |
| E · excluidas por C (gradúan en A, no en C) | 2.427 | 50% | 60% |

### B2B3

| Definición | Compañías | Logos m12 | GDR m12 |
|---|---:|---:|---:|
| A · ≥1/sem móvil (CX vigente) | 4.884 | 70% | 70% |
| B · ≥5/sem móvil | 4.721 | 70% | 71% |
| C · ≥5/sem solo mes 4 (doc) | 4.021 | 70% | 72% |
| D · excluidas por B | 157 | 10% | 15% |
| E · excluidas por C | 863 | 50% | 62% |

Cómo leerlo:

- **Con la ventana corregida, el umbral de 5 saca 6% de las compañías y la retención no se mueve:** 70% contra 70%. Las que saca (grupo D) sí son zombis —20% de logos—, pero son 593 de 9.408 y no alcanzan a mover el promedio.
- **Las 2.427 que saca la versión del documento (grupo E) retienen 50%, no 20%.** No son cuentas muertas: son cuentas que consolidaron después del mes 4 y la evaluación única nunca las volvió a mirar. La mitad sigue viva al año.
- La comparación A contra B está sesgada **a favor de B**: al graduar más tarde, sus cohortes ya sobrevivieron más meses antes de entrar. Aun con ese sesgo a favor, B no retiene mejor.

Conclusión: el poder discriminante que el documento atribuye al umbral de 5 viene de castigar el arranque lento, no de detectar cuentas malas.

---

## Qué cambiar en el documento

1. **Corregir la fila «Pablo» de la tabla «Las definiciones»**: agregar la tercera condición (reservas en los 4 bloques de 7 días de un mismo mes) y cambiar «Filtro de uso post-hito: No» por «Sí, ≥1 reserva por bloque, evaluado en todo mes desde el 4º pago».
2. **Rehacer las columnas «Grad. Pablo» y «Onb. Pablo»** de las cuatro tablas de churn con esa definición. Los valores están en las series de arriba.
3. **Reemplazar «Pablo +5/sem» por la versión móvil**, o marcar explícitamente que el escenario publicado mide una ventana de un mes. Tal como está, la columna no responde «¿sirve exigir 5 por semana?» sino «¿sirve exigir que consoliden exactamente en el mes 4?».
4. **Reescribir el bullet de la sección de retención.** Con la ventana corregida, ninguna de las cuatro definiciones separa por retención: las cuatro dan 70% de logos a 12 meses.
5. **Revisar la fila «Grad. Pablo reporte»** (1.561): coincide con la variante sin filtro semanal, no con la definición de producción, que da 1.463.

Lo que el documento concluye bien y no cambia: la vía objetiva del 4º pago + uso es necesaria, la marca de journey sola no sirve (4% de retención a 12 meses) y el churn total supera el budget en todas las definiciones.

---

## Metodología, supuestos y limitaciones

**Fuentes.** `dwh.revenue_waterfall_saas` (eventos de churn, sedes = `-sum(quantity_impact)`, MRR = `-sum(mrr_change_usd)`), `dwh.mrr` (ciclos pagados), `dwh.company_sales_months` (reservas mensuales), `dwh.cache_company_sales_days` + `dwh.recent_company_sales_days` (reservas diarias), `dwh.mrr_saas` (MRR a FX constante para las cohortes). Motor: Redshift.

**Supuestos.**

- Universo B2B3 = `dwh.companies.first_customer_segment_2 = 'B2B3'`. Es el único corte que reproduce las 3.239 sedes del documento; conviene dejarlo escrito ahí, porque `customer_segment_2` actual da 2.201 y `segment_at_churn` da 2.452.
- «4º pago» = cuarto mes distinto con `is_active_and_paid`, que es lo que usa el código de CX. Si BizOps lo define por número de facturas de Chargebee, los bordes cambian.
- Una cuenta cuenta como graduada si su fecha de graduación es anterior o igual al mes del evento de churn.
- Las ventanas diarias se leen de las dos tablas físicas porque `dwh.company_sales_days` se renombró a `_old` el 3-sep-2026 sin dejar la vista que la reemplaza.

**Limitaciones.**

- Mis cifras quedan entre 3% y 6% por debajo de las publicadas en las columnas equivalentes (1.548 contra 1.566 en B2B3; 3.004 contra 3.097 en el total). La brecha viene de detalles de implementación que el documento no explicita. La dirección y el reparto del efecto no dependen de eso.
- La comparación de retención entre definiciones es condicional a haber graduado, como advierte la propia subhoja: mide «qué pasó después», no capacidad predictiva sin mirar el futuro.
- Las cohortes de 12 meses solo incluyen graduaciones hasta ago-2025.
- No revisé las tablas de MRR por definición del documento ni la subhoja de diagnóstico de la marca de journey.

---

## Borrador de mensaje para Slack

> Joaquín, revisé el documento de graduación. El universo y el churn total están perfectos —reproduje las 3.239 sedes B2B3 exactas—, pero hay tres correcciones en las columnas que me atribuyen.
>
> La definición de CX que corre en producción lleva una tercera condición que no está en el documento: reservas en los 4 bloques de 7 días de un mismo mes. Y el escenario «+5/sem» evalúa el uso solo en el mes del 4º pago, cuando el 4º pago es el piso, no la foto: de las 57 sedes que en abril separan 152 de 95, unas 45 las produce esa ventana fija y solo 11 el umbral de 5.
>
> Eso cambia la recomendación. Con el 4º pago como piso, el filtro de 5 por semana saca 6% de las compañías y la retención queda igual: 70% de logos a 12 meses contra 70%. Las 2.427 que saca tu versión retienen 50%, no son zombis: son cuentas que consolidaron después del mes 4.
>
> Te dejo el detalle con las series corregidas y los supuestos: <link>
