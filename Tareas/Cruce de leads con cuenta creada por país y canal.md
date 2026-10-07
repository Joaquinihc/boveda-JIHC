---
type: tarea
estado: en-curso
prioridad: media
tiempo:
tipo: analisis
origen: reunion
proyecto:
temas:
  - leads
  - adquisicion
fuente: "[[2026-10-01 Leads con cuenta creada y PACE (Matías)]]"
fecha_limite:
creado: 2026-10-01
revisar: false
id: T-085
---

# Cruce de leads con cuenta creada por país y canal

Calcular, por país y canal, el % de leads que tenían cuenta creada al momento de crearse el lead (aislando el efecto del proceso de venta) y si tener cuenta en t0 (o t0 + delta) convierte mejor. Lo hace también Matías Ulloa por su lado, para comparar metodologías. Acción de [[2026-10-01 Leads con cuenta creada y PACE (Matías)]].

## Subtareas
- [x] Calcular por país y canal el % de leads con cuenta en t0 y su conversión (reporte publicado el 6-oct)
- [x] Agregar la tasa de creación de cuenta independiente de t0 (ventanas hasta 1, 7, 30, 90 días y sin límite) — reunión 6-oct
- [ ] Agregar al reporte un filtro por vertical (Beauty, Medspa, Health Doctors, Health Non-doctors) — reunión 6-oct
- [ ] Agregar al gráfico de conversión lead → merchant un filtro con / sin cuenta en t0 (confirmar con Matías si hace falta: en la reunión dijo que el gráfico de 30 días ya lo muestra) — reunión 6-oct
- [ ] Analizar la caída de creación de cuenta en t0 por canal: ¿campaña puntual o problema de producto? Cruzar con el ratio de intentos vs cuentas creadas que revisa Matías con Felipe — reunión 6-oct
- [ ] Estimar el impacto en cierres Q vs Q: cuántos NM habría con el % de cuentas creadas de H1-2026 — reunión 6-oct
- [ ] Revisar el doble conteo de reconversiones en la conversión (un mismo pago contado en T1 y T2) — reunión 6-oct
- [ ] Revisar en Chile y Colombia si Ventas crea una segunda cuenta al cerrar (canales donde los leads sin cuenta convierten más) — reunión 6-oct

## Notas
- Matías Ulloa comparte su reporte como referencia.

### 2026-10-07 · Análisis y reporte
- Análisis: [[20261007-01 Leads con cuenta creada en t0 por país y canal]]. SQL: [[SQL — Leads efectivos × cuenta creada (t0 y ventanas)]]. Reporte interactivo: https://claude.ai/artifact/YAEH1sxBmFakK7QkuSoqhG
- Resumen (corte 5-oct-2026, leads B2B3 efectivos): % con cuenta en t0 de los 4 países core 56,7% → 50,8% (H1 → Q3-2026), sobre todo por más formularios de Paid Social; en Chile cae dentro de cada canal. Con cuenta en t0, Paid Social convierte 16,4% vs 6,3% a 90 días; en los demás canales no hay ventaja. En Chile cae también la conversión de los leads con cuenta (capacidad o proceso).
- Reunión 6-oct con Matías ([transcripción](https://app.notion.com/p/agendapro/Mat-as-Joaqu-n-3f1ab34c233b81039c7bd28ae631167b)): revisó el reporte y llegó a conclusiones parecidas (57% → 51%). La caída de jun–jul la atribuye al error de ruteo de reconversiones, con recuperación en ago–sep, y a errores de creación de cuenta que revisa con Felipe. Ve la oportunidad en llevar a Paid Social (~1/3 de los leads) a crear cuenta. El capítulo 5 le sirvió más; el 7 (gasto) aporta poco a esta discusión. Sus próximos pasos: revisar con Felipe el ratio de intentos vs cuentas creadas y presentar hallazgos y plan con Felipe (Producto), [[Pablo Lucero]] y Ventas (asignación y tasa de conversión).

## Historial
- 2026-10-02 · Creada desde [[2026-10-01 Leads con cuenta creada y PACE (Matías)]] (propuesta)
- 2026-10-07 · Aprobada: en curso, prioridad media (Joaquín en conversación)
- 2026-10-07 · Avance desde Claude Code: análisis y reporte publicados ([[20261007-01 Leads con cuenta creada en t0 por país y canal]]); subtareas de la reunión del 6-oct con Matías
