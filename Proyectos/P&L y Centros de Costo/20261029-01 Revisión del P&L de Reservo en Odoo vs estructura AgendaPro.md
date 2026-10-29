---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-10-29
temas: [pnl, odoo, centros-de-costo]
metricas: []
---

# Revisión del P&L de Reservo en Odoo vs estructura AgendaPro

**Pregunta**: ¿Los gastos de las dos sociedades nuevas de Reservo en Odoo (SC3 SpA, Chile; SOFTWARE RESERVO, México) están clasificados en las mismas cuentas y con el mismo criterio que las sociedades de AgendaPro, de modo que se puedan sumar al P&L Master?

## Conclusión
> La estructura de cuentas está bien: ambos planes son clones de AgendaPro Chile y México con los nombres G-code (184/217 y 182/186 cuentas P&L idénticas) y el 100 % de las facturas de compra 2026 lleva analítica G-code del plan «Centros», el mismo de AgendaPro. Lo que no está listo es la cobertura y el criterio: (1) SC3 no tiene remuneraciones 2026 (en 2025 fueron ~90 MM CLP/mes, más que todas las facturas); (2) SOFTWARE RESERVO solo registra contador y abogado, la operación México se paga desde SC3; (3) no hay plan de cost center HQ/país; (4) Kushki y Transbank son comisiones de recaudar las suscripciones propias y están en G0003/G0015 (Payment Processing) sin revenue I1000; (5) varias cuentas cajón (G3104, G3102, G3009, G2104, G1106, G1009, G3007) mezclan cosas que AgendaPro lleva a activo, P0004, G4006, G2004, G1104 o G1004. Nada de esto es grave en monto salvo la nómina, pero todo hay que resolverlo antes de sumar Reservo al P&L.

## Método
- Fuente: Odoo producción vía XML-RPC con `scripts/lib/odoo_connection.py` del repo `mcp-odoo-agendapro` (sin cambios de código). Corte **2026-10-26**, solo asientos `posted`.
- Sociedades: **SC3 SpA** (`company_id` 7, CLP) y **SOFTWARE RESERVO** (`company_id` 8, MXN). Creadas el 19-oct-2026; asientos cargados por [[Nicolás Astudillo]] el 22-oct (SC3, ~7.050 asientos) y 23-oct (MX, ~230).
- Comparación: (a) diff de `account.account` por código y nombre contra AgendaPro Chile (id 1) y AgendaPro México (id 4); (b) `read_group` de `account.move.line` por cuenta P&L, partner y etiqueta; (c) para cada proveedor relevante, en qué cuenta lo lleva el grupo AgendaPro (ids 1 a 5) en 2026; (d) mapeo G-code → P&L Category de la hoja Mappings de «P&L AgendaPro - Master 2026».

## Evidencia

### Cómo quedaron cargadas

| | SC3 SpA (Chile) | SOFTWARE RESERVO (México) |
|---|---|---|
| Cuentas P&L idénticas a la sociedad AgendaPro del país | 184 de 217 | 182 de 186 |
| Facturas de compra 2026 | 911, todas con analítica G-code | 11, todas con analítica |
| Ingresos 2026 | Centralización mensual MISC, 185 a 230 MM CLP/mes en I0000, más 4.0.00.09 «Ventas de Exportación» desde may-26 (cuenta nueva del 22-oct) | 198 facturas VSAT, ~50k MXN/mes en I0000 |
| Remuneraciones 2026 | Ninguna (diarios REM y SLR vacíos) | Ninguna |
| Cost center HQ/país | No (solo analítica G-code) | No |

- Apertura al 31-12-2025 con el P&L 2025 como bolsones: ventas 1.794 MM CLP en I0000; remuneraciones 1.079 MM en 5.5.00.01 Sueldo Base sin analítica; 601 MM en G3008 Otros Gastos Administración («Gastos generales 2025» 265 MM y «Gastos internacionales 2025» 336 MM). Cualquier comparación 2025 desde Odoo sale con todo el opex en Administración.
- Encima quedó el plan default de Odoo (310xxx/410xxx en Chile; 401.01.01, 601.84.01 y similares en México) sin uso.
- SC3 activó 28,6 MM CLP (muebles 12,1; notebooks y celulares 14,7; monitor 0,4; televisores 1,4) sin ningún asiento de depreciación.
- Coinciden con AgendaPro: AWS G0001, Facebook/Google Ads G1005/G1006, arriendo G3100, BUK G3016, Slack G3018, DTEmite G0007, comisiones bancarias G4003.

### Gastos clasificados distinto a AgendaPro (SC3, ene–oct 2026, CLP)

| Cuenta en SC3 | Qué hay adentro | Criterio AgendaPro | Monto |
|---|---|---|---|
| G0015 Transbank 6,4 MM y G0003 Kushki 20,5 MM | Comisiones por cobrar suscripciones propias (asientos «Comisión pasarela contra 1.0.01.07») | Recaudación propia es Platform (G0004 Payment gateways); G0015 es Payment Processing y Reservo no tiene I1000 | 26,8 MM |
| G3104 Otros Gastos Oficina | 374 líneas: MercadoLibre 6,8; PCFIX 3,8; Alcaplus 2,8; Verona Home 2,7; Iturra Climatización 1,9; Armónicos 1,0; Cencosud 1,0; IKEA, Easy, Rosen | Muebles y habilitación van a activo 1.1.03.00 o a P0004 Proyecto Oficina (AgendaPro lo hace con EDGE); supermercado a G3009 | 30,3 MM |
| G3102 Materiales de Oficina | Notebooks Lenovo 1,7; Ingeniería Informática 1,6; PC Factory 0,9; MacOnline 0,4 | Inconsistente con la propia SC3, que capitaliza otros notebooks del mismo proveedor en 1.1.03.01; AgendaPro Chile lleva equipos a G0312 | 6,2 MM |
| G3009 Beneficios | Betterfly 1,7 (ok); Guris Eventos 1,5; Andina 1,4; Fresa Party, Cielos Americanos, Comedy Producciones | Eventos de equipo van a G4006 Eventos Corporativos (SC3 ya tiene la cuenta) | 7,1 MM |
| G3004 Gastos Legales | Carey 21,0 MM concentrado en ago y sep 2026 | Probable costo de la transacción: marcar como one-off | 23,3 MM |
| G2104 Tech platforms | Anthropic 1,9; Atlassian 1,7; Apple Services 0,4; OpenAI 0,3; Lovable 0,2 | AgendaPro Inc lleva Atlassian, OpenAI y Lovable a G2004 Product platforms; Anthropic de servicio a 5.2.73 Platform IA; App Store a G0004 | 4,6 MM |
| G1106 Otros Gastos Ventas | Salesforce 1,1; asesoría comercial y referidos 0,6 | CRM va a G1104 Sales platforms (como HubSpot); honorarios comerciales a G1101 | 1,7 MM |
| G1009 Otros Marketing | Sendinblue 0,5; TikTok 1,0; PR Prenseable 1,0; LinkedIn | Email marketing va a G1004 Marketing platforms; TikTok en G1009 coincide con México | 2,6 MM |
| G3007 Viajes | Duty Free 0,9 | No es viaje: probablemente regalos, G3009 o G3012 | 0,9 MM |
| 6.0.00.02 Diferencias de Cambio | Venta de dólares, 4 asientos | Igual que AgendaPro Chile, pero la cuenta no tiene G-code y queda fuera del mapeo Master | 1,0 MM |

- México: solo BHSC Contadores (G3005, 14,1k MXN/mes) y Von Wobeser (G3004, 20,4k) como gasto 2026. Dos asientos de prueba: factura de cliente en borrador por 1.183 MXN (N&Q Rehabilitación Oral, 26-oct) en la cuenta default 401.01.01 en vez de la I0000 401.01.001, y dos líneas posteadas en cero a «Usuario Final».
- Preguntas para Reservo: DTEmite cuesta 12 a 14 MM CLP/mes (3,6x AgendaPro Chile), lo que sugiere reventa de boleta electrónica con el ingreso mezclado en I0000; Tecnologías Saludando (0,6 MM en G2105) es un proveedor desconocido.

## Caveats
- Corte 26-oct-2026: octubre está incompleto (37 facturas de compra en SC3).
- Los montos son suma de `balance` de líneas de producto; no se revisó factura por factura, solo por cuenta y proveedor.
- La comparación «criterio AgendaPro» sale de dónde lleva cada proveedor el grupo en 2026, no de una guía escrita; donde AgendaPro no tiene al proveedor se usó la guía `docs/cost-center-classification-guide.md` del repo.
- Tarea de seguimiento: [[Revisar P&L de Reservo gasto por gasto (cuentas Odoo)]].
