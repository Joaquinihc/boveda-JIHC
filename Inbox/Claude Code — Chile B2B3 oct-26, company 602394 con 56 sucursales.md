---
type: borrador
origen: Claude Code
fecha: 2026-11-10
temas: [adquisicion, leads]
revisar: true
---

# Claude Code — Chile B2B3 oct-26, company 602394 con 56 sucursales

- **Qué se vio (9-nov)**: en el Board Book, la conversión B2B3 de Chile de octubre 2026 da 23,1 %, contra ~13 % en jul–sep. Leads estables (659); New Merchants B2B3 de Chile 152 contra ~90 los meses anteriores (`dwh.country_new_merchants`, que suma `plan_quantity`).
- **Causa**: una sola compañía, company_id 602394 («Peluquería de Andrés Gil»), entra en octubre con `plan_quantity` = 56. Sin ella, Chile tendría 96 New Merchants y ~14,6 % de conversión.
- **Pendiente**: revisar en Chargebee y en el DWH si son 56 sucursales reales o un error de carga del plan. Si es un error, el New Merchants B2B3 de octubre (y el CAC y el payback de Chile) están inflados.
- **Nota aparte**: los picos de conversión, CAC y payback que se vieron en `localhost:3000` eran de un caché local viejo (leads de octubre parciales); producción no estaba afectada.
