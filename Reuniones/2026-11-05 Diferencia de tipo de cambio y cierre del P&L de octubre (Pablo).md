---
type: reunion
fecha: 2026-11-05
titulo: "Diferencia de tipo de cambio y cierre del P&L de octubre (Pablo)"
personas: ["[[Pablo Lucero]]", "[[Sebastián Cruzat]]"]
proyectos: ["[[_P&L y Centros de Costo]]"]
temas: [pnl, cierre-mensual, mrr, waterfall]
fuente: https://app.notion.com/p/40fab34c233b81edaaf6f0880858180b
verificado: false
---

# Diferencia de tipo de cambio y cierre del P&L de octubre (Pablo)

**Acuerdos** (máx 5)
- Diferencia de tipo de cambio en el P&L Master: de las dos cuentas de tipo de cambio del mapping (una G y una F), la G en realidad es de comisiones bancarias y alguien la renombró. Vuelve a su nombre, se ajustan los registros de diferencia de cambio (aparecen desde mayo; antes probablemente estaban en la F) y se crea una cuenta nueva para separar dos filas: intercompany y externos.
- Prioridad de Pablo: cerrar el mes primero y arreglar el resto después. Con viajes y diferencia de cambio corregidos, se cierra octubre.
- Cash flow de octubre: da 3,1 frente a 2,8 del otro lado; la diferencia es Vambe y otros ítems. Vambe de marzo a octubre sale con ayuda de Seba Cruzat y el acceso a Mercado Pago; enero y febrero quizás requieran dos planillas más.
- Vista solo AgendaPro (Julio la vio «too good to be true»): remuneraciones, costos de servicio y adquisición explicados; corporativos y ventas aún no cuadran. Comisiones: se provisionaron 47 y se pagaron 39; la nueva provisión de 38 puede quedar corta por un error que deja casos fuera del cálculo (+~5K). Development cae −30K (dos servicios descontinuados), payroll baja por 3 finiquitos en tiendas y administración sube por viajes a México y Airbnb/Uber mal catalogados, que van a cada área.
- Promo 3x1: ~40 merchants con MRR 0 no cuentan como activos, pero deberían contar como New Merchants. Requiere cambiar la tabla MRR y la condición de activo (hoy MRR > 0); en el Waterfall, excepción en la query solo para este mes.

**Mis acciones**
- [ ] Devolver a su nombre la cuenta de comisiones bancarias, ajustar los registros de diferencia de tipo de cambio y crear la cuenta nueva (intercompany vs externos)
- [ ] Revisar con Rafa el cierre del cash flow de octubre (5-nov, después de almuerzo)
- [ ] Proponer cómo modificar la tabla MRR y la condición de activo para contar como New Merchants a los ~40 comercios de la promo 3x1
- [ ] Cerrar los ajustes del P&L de octubre: pedir a Nico +~5K de provisión de comisiones y la reclasificación de viajes (Airbnb, Uber) a cada área; revisar Development (−30K) y el alza de administración
- [ ] Pedir las dos planillas de Vambe de enero y febrero, si hacen falta
- [ ] Documentar los cambios hechos este mes

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/40fab34c233b81edaaf6f0880858180b). Asistentes según Notion: Pablo Lucero y Joaquín. La transcripción dice «Bambe» (es Vambe), «Moodle» y «Levita» (no claros) y «Seba Conche» (probablemente Sebastián Concha, que dio el acceso a Mercado Pago; no se linkea). «Nico» no se linkea: hay más de un Nico. Relacionadas: [[Aplicar en el P&L Master las correcciones de diferencia de cambio y cashback de 2026]], [[2026-11-05 Vuelos de TAM — cobro doble y reclasificación de viajes (Álvaro)]].
