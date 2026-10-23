---
type: reunion
fecha: 2026-10-21
titulo: "Definiciones e infraestructura de datos de Reservo (Christopher)"
personas: ["[[Pablo Lucero]]"]
proyectos: []
temas: [definiciones, churn, dwh, coordinacion]
fuente: https://app.notion.com/p/400ab34c233b8118ba67cc0efbc1fd15
verificado: false
---

# Definiciones e infraestructura de datos de Reservo (Christopher)

**Acuerdos** (máx 5)
- Christopher (datos, Reservo) coordina con [[Pablo Lucero]] y los jefes de área de Reservo para documentar las definiciones en conjunto: corrige lo que Pablo dejó en Notion y completa bookings, uso de features, leads y conversión, MRR, churn, segmentación, waterfall y monedas; las de postventa, con Natalia Contreras y Sebastián Cruzat.
- Formato del documento compartido: separar la definición comercial (qué significa el término) de la técnica (tablas y variables).
- Reservo tiene un modelo de fuga RFM calibrado por rubro, plan y país (ROC ~81%; acierta 60–70% de las bajas, ~100 bajas al mes). Terminología: sucursal = merchant = location = venue; compañía = agrupación de sucursales.
- Infraestructura de Reservo: DW en Postgres (obsoleto) y uno nuevo en S3 + Glue + Lambda pensado para migrar a Redshift; ETL comerciales en una VM Windows; postventa consulta la réplica. A mediano plazo su DW sería un data mart conectado al DW de AgendaPro; al principio, definiciones alineadas sin fusionar infraestructura.
- Accesos a la organización de GitHub y al DW de Reservo pendientes: Chris consulta a Sebastián Evia; el AWS de Reservo se normaliza la próxima semana.

**Mis acciones**
- [ ] Separar las definiciones comerciales de las técnicas en el documento compartido

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/400ab34c233b8118ba67cc0efbc1fd15).
