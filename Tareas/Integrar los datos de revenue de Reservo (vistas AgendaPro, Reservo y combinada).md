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
- [ ] Consolidar el revenue de Reservo con el de AgendaPro en las tres vistas — listo cuando estén las tres vistas en `main`: levantar el PR del Board Book con los cambios posteriores al #874 y re-apuntar el #879 del Flash Revenue
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

### Pendientes menores
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
