---
type: tarea
estado: propuesta
prioridad: alta
tiempo:
tipo: reporte
origen: reunion
proyecto:
temas:
  - revenue
  - dwh
fuente: "[[2026-10-27 FP&A Weekly]]"
fecha_limite:
creado: 2026-10-27
revisar: false
id: T-111
---

# Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)

Prioridad #1 del equipo en la FP&A Weekly del 27-oct: traer los datos de revenue de Reservo y consolidarlos con los de AgendaPro en tres vistas (solo AgendaPro, solo Reservo y combinado). Meta dicha en la reunión: tenerlos al fin de semana del 27-oct. Acción de [[2026-10-27 FP&A Weekly]].

## Subtareas
- [ ] Seguir con Christopher e Israel el acceso al DW de Reservo — listo cuando haya acceso a los datos de revenue. Mientras, preguntarle a Christopher por las 3 filas pagadas sin `fecha_pago` ni aplanado (ids de pago 142881, 149611, 162250) y fijar el criterio
- [ ] Consolidar el revenue de Reservo con el de AgendaPro en las tres vistas — listo cuando estén las tres vistas en `main`: PR #884 (`feat/flash-revenue-reservo-main` → `main`, Board Book posterior al #874 + Flash Revenue, CI verde, reviewer nicofrem-agpro) abierto el 5-nov; el #879 quedó cerrado. El agente revisor pidió cambios el 5-nov (no presentar el revenue de Reservo como MRR; sacar el comentario de migración): aplicados en `2e2c893c` y re-pedida la revisión. Falta que Pablo o Nicolás lo mergeen
- [ ] Cerrar el revenue de octubre de Reservo en la planilla de conciliación de Julio (Vambe real y Mercado Pago) — listo cuando `Reservo!S3` tenga la factura de octubre de Vambe (foto al 01-11, neta de IVA) y `Reservo!M3` el revenue share de Mercado Pago con el criterio bruto/neto decidido
- [x] Corregir los New Merchants de Reservo cruzando primer pago y sucursales de la cadena (ex T-115; cerrada el 3-nov: regla por primer pago por cuenta, cuadra con Pablo Saintard)
    - [x] Pedir a Christopher el detalle completo de pagos (incluidas las filas con estado de pago 0) y cómo calculó los otros meses — el archivo 7 ya las trae
    - [x] Rehacer el cruce con primer pago — agosto da 173
    - [x] Calcular los New Merchants de octubre con ese criterio — 136, igual que la tabla de Pablo

## Notas
- La planilla de definiciones que dejó Christopher es el punto de partida; relacionada: [[Definiciones grupo AgendaPro vs Reservo]] (T-099).

### 2026-11-03 · Estado en Evidence y New Merchants cerrados
- PR #874 (`feat/board-book-reservo`) se mergeó a `main` el 30-oct. Después la rama recibió 20 commits sin PR (hasta `a8592d5a`, 2-nov): merchants = clientes activos vigentes, nuevos = primer pago por cuenta, consolidación desde oct-26, ARR del grupo (slide 5.1). El PR #879 del Flash Revenue (`feat/flash-revenue-reservo`, https://github.com/agendapro/agendapro-dashboards-evidence/pull/879) apunta a esa rama ya mergeada: falta el PR del Board Book a `main` y re-apuntar el #879.
- Decisión 3-nov: los gráficos de grupo suman Reservo solo desde octubre de 2026 (adquisición); la historia de Reservo va pro forma en gráficos propios; tablas de crecimiento con filas reported (like-for-like) y consolidated. Método completo y cómo regenerar la semilla: [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]].
- New Merchants (ex T-115): regla cerrada y cuadrada id a id con Pablo Saintard para los diez meses de 2026; la unidad de abril era un pago fuera de serie del cliente 8601 (sin `fecha_pago`). Detalle y preguntas a Christopher: [[20261103-01 New Merchants y Merchants de Reservo — definiciones finales y cuadre con Pablo Saintard]].
- DW de Reservo: Christopher dijo el 2-nov que queda «entre esta semana y la próxima»; después Israel conecta al DWH.

### 2026-11-05 · Vambe y Mercado Pago en la planilla de conciliación
- Planilla «DB - MRR & fees - DWH - Conciliación tabla», hoja Reservo, fila 202610: software cargado por moneda de pago (G3:K3, 4-nov); Vambe en S3 con el neto de septiembre (14.853.228 CLP, factura 129780) como proxy porque la factura de octubre no existe aún; Mercado Pago en M3 sigue en cero.
- Método y cuadre: el 25 % de los planes activos al día 1 del mes siguiente (consultas de Sebastián Cruzat) reproduce las facturas de Pablo Saintard al centavo en 6 de 7 meses; el 25 % trae el IVA dentro. Mercado Pago es solo Chile y se aproxima por la variación del saldo total de la cuenta (sep→oct: 3.568.795 CLP). Detalle: [[20261105-01 Revenue share de Vambe y comisiones de Mercado Pago en el cierre de octubre de Reservo]]; regla en [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]].

### Pendientes menores
- 2026-11-05 · Reemplazar `Reservo!S3` con la factura de octubre de Vambe cuando Cruzat o Pablo Saintard manden la consulta «Activa al 01-11» o la factura (mediados de noviembre).
- 2026-11-05 · Confirmar el IVA de Vambe con los dos abonos de VAMBE SpA de octubre en Odoo (BICE 24-00071-0): si son 16.181.784 y 16.774.250, el 25 % incluye IVA y S3 está bien.
- 2026-11-05 · Decidir si el revenue share de Mercado Pago va bruto (3.568.795) o neto de IVA (2.998.988) antes de cargar M3; preguntar a Sebastián Concha si el fee se cobra con IVA.
- 2026-11-05 · Pedir las cartas de saldo de Mercado Pago de los meses anteriores (o usar el módulo de reportes) para la historia pro forma; en Drive falta la consulta de Vambe de febrero.
- 2026-11-03 · Decidir si «cuenta nueva sin pago» cuenta como merchant (Reservo las cuenta; Joaquín duda).
- 2026-11-03 · Pedir a Christopher marca + monto del revenue share de Vambe (4–5 cuentas al mes, factura aparte) y la foto mensual de suscripciones para tener MRR real.
- 2026-11-03 · Borradores en Slack sin enviar: DM a Christopher (hallazgos y pedidos), DM a Pablo Lucero (estado, versión corta; borrar la larga), DM a Pablo Saintard (cuadre de abril).
- 2026-11-03 · Rama `feat/fx-bridge-current-month` (slides 7c/7d/7e muestran el mes en curso como MTD) lista sin PR, a la espera de autorización.
- 2026-11-02 · (de T-115) Acceso a la réplica de Reservo pendiente (permisos sobre fichas clínicas); con acceso, el cruce se hace directo sobre `pago cliente mes`.

## Historial
- 2026-10-30 · Creada desde [[2026-10-27 FP&A Weekly]] (propuesta)
- 2026-11-03 · Mencionada de nuevo en [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]] (serie mensual de revenue con BAMBE y comisiones de Mercado Pago)
- 2026-11-03 · Subtarea agregada: «Corregir los New Merchants de Reservo cruzando primer pago y sucursales de la cadena» (de [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]], T-115), ya cumplida (Joaquín en conversación con Claude Code)
- 2026-11-03 · Avance desde Claude Code: Board Book y Flash Revenue con Reservo en ramas; #874 mergeado el 30-oct con 20 commits posteriores sin PR; subtareas 1 y 2 ampliadas (filas sin fecha; PR a main y re-apuntar #879)
- 2026-11-05 · Avance desde Claude Code: revenue share de Vambe y Mercado Pago en la planilla de conciliación (proxy de septiembre en S3, cuadre de consultas de Cruzat vs facturas, cartas de saldo); subtarea agregada «Cerrar el revenue de octubre de Reservo en la planilla de conciliación» (3/3 abiertas, con OK de Joaquín)
- 2026-11-05 · Avance desde Claude Code: PR #884 a `main` con los 24 commits posteriores al #874 rebaseados (https://github.com/agendapro/agendapro-dashboards-evidence/pull/884), tests 423/423 y CI verde; #879 cerrado como reemplazado. Subtarea 2 actualizada
- 2026-11-05 · Avance desde Claude Code: revisión del PR #884 por el agente revisor (changes requested) atendida en `2e2c893c`: filas y título del bloque MRR renombrados a «AgendaPro MRR + Reservo revenue» (las de ARR no se tocaron), comentario de migración eliminado, tests 423/423; respuestas en los dos hilos y re-request a nicofrem-agpro
