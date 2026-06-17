# CLAUDE.md — Sistema de Gestión MP 2026 (HHHA)

Este documento es el brief para construir la aplicación de gestión de mantención de
equipos clínicos del **Hospital Hernán Henríquez Aravena (HHHA)**. Contiene el proceso real,
las reglas, el modelo de datos y la estructura de la interfaz. Junto a él se entregan:

- `datos_semilla.json` — los datos reales actuales (893 equipos, 14 casos correctivos con
  sus rutas, 1.242 mantenciones preventivas, 59 pendientes, catálogos). Úsalo para precargar.
- `Sistema_MP2026_HHHA.html` — un **prototipo funcional de una sola página** (HTML/JS vanilla)
  que ya implementa toda la estructura descrita aquí. Sirve como referencia de UX y de lógica.

El objetivo del responsable (Cristián Beltrán, encargado de Equipamiento Clínico) no es llenar
planillas: es **saber en todo momento el estado de cada equipo y asegurar su operatividad.**

---

## 1. Qué hay que construir

Evolucionar el prototipo de una sola página a una **aplicación web real** con datos
centralizados, pensada para que la use un equipo (no un solo navegador).

**Stack sugerido** (ajustable): frontend en React + TypeScript; backend con API REST
(Node/Express o Python/FastAPI); base de datos relacional (PostgreSQL o SQLite para empezar);
autenticación con roles. Si se prefiere algo más simple al inicio, vale Next.js full-stack con
una base SQLite. Lo importante: persistencia centralizada y multiusuario.

**Funcionalidades núcleo** (las cinco vistas, ver §5): Panel, Pendientes, Equipos + Ficha,
Correctivo + Expediente, Preventivo. Todas con lectura; las de Pendientes y Preventivo y la
apertura de casos correctivos además con escritura.

**Roles:** un administrador (Cristián Beltrán) con acceso total; ingenieros que registran su
trabajo y ven el estado. Toda acción de cierre/registro queda con autor y fecha.

---

## 2. El proceso de Mantención Correctiva

Cuando un equipo falla, el servicio clínico genera una **Orden de Trabajo (OT)** en SIGEM
(plataforma del hospital). Cada OT tiene un **folio SIGEM** que es la raíz del expediente:
todo cuelga de él. El Jefe de Equipamiento Clínico la asigna a un ingeniero interno, que
**siempre intenta repararla en terreno primero**. Si lo logra, escribe la tarea en SIGEM y
cierra. Si no, el caso toma una de **cuatro rutas**:

1. **Repuestos** — equipo No Operativo → se cotiza y compra la pieza → llega → el ingeniero la
   instala y prueba → cierra.
2. **Visita técnica** — viene un especialista externo al hospital → entrega un **Reporte de
   Servicio** → reparó, o detectó que faltan repuestos (bucle: se cotiza el repuesto y se
   coordina otra visita).
3. **Servicio técnico** — el equipo se envía afuera (estado → "en servicio técnico") → vuelve
   con un reporte → operativo, o se reevalúa (repuestos / visita / reenvío / baja).
4. **Baja** — el equipo no tiene reparación: se genera un documento de baja.

### Línea de compra (compartida por las rutas con gasto)
Cotización → ¿**Trato Directo** o **Compra Ágil**? → Leslie tramita la Orden de Compra (OC) →
Finanzas la emite (registra el N°) → el ingeniero envía la OC al proveedor.

### Reglas clave
- **Trato Directo vs Compra Ágil depende de la empresa, no del monto:** si la empresa es la
  **representante exclusiva de la marca** del equipo → Trato Directo (requiere un Informe
  Técnico con folio). Si no → Compra Ágil (sin informe).
- Solo el **ingeniero interno asignado** cierra la OT en SIGEM (nunca el externo).
- El **Reporte de Servicio** se entrega en toda visita y servicio técnico (repare o no); lo
  archiva Cristián Beltrán en la carpeta física del equipo.

### Los dos cierres (concepto central)
Un caso tiene **dos cierres independientes** que no siempre coinciden en el tiempo:
- **Cierre operativo:** el ingeniero cierra la OT en SIGEM cuando el equipo vuelve a funcionar
  (lo que importa para el servicio clínico; suele ocurrir primero).
- **Cierre documental:** llega el Reporte de Servicio y se archiva (el respaldo administrativo).

Si el equipo ya opera pero el reporte no llega, el caso es un **pendiente documental**:
operativamente cerrado, administrativamente abierto. Las hojas de Visita y Servicio Técnico
llevan la marca **¿Reporte recibido? (Sí/No)**. Sobre el pendiente documental hay dos
responsabilidades: el **responsable de conseguir el reporte** (gestiona con el proveedor; por
defecto el ingeniero de la OT) y el **responsable administrativo** (responde por que se cumpla;
Cristián Beltrán por defecto). Una persona puede ocupar ambos roles.

---

## 3. El proceso de Mantención Preventiva

Programa anual masivo (la base es de ~893 equipos). Cada mantención programada se ejecuta
(interno o externo) y se registra con un **código de resultado**:

| Código | Significado |
| --- | --- |
| Sí | Mantención Preventiva Realizada |
| Si-RA | Mantención de Año Anterior Realizada |
| C1–C8 | Reprogramada (con causal, ver abajo) |
| FS | Fuera de Servicio |
| No | No Realizada |
| NU | No Ubicable |
| Baja | Equipo Dado de Baja |

**Causales de reprogramación (C1–C8):**
C1 imposibilidad de desocupar el equipo del paciente · C2 equipo en servicio técnico ·
C3 equipo no operativo esperando repuestos · C4 equipo en préstamo · C5 sin horas-hombre del
funcionario SEC · C6 sin horas-hombre del servicio externo · C7 ausencia justificada del
funcionario >15 días · C8 contingencia hospitalaria.

### Cruce con el correctivo
**C2, C3 y Baja son los puntos donde el correctivo se refleja en el preventivo:** un equipo en
servicio técnico con MP programada ese mes da **C2**; uno no operativo esperando repuestos da
**C3**; uno dado de baja da **Baja**. El correctivo es la *causa*; el código de reprogramación
es el *resultado* que se oficializa.

### Vista resumen ("carta") y mapeo de códigos
Para los tableros se usa un resumen tipo matriz **equipo × mes** con un catálogo reducido de 6
códigos. El mapeo desde los 14 códigos del registro es:

```
Sí, Si-RA            → E   (ejecutada)
C2                   → C2
C3                   → C3
C1, C4, C5, C6, C7, C8, No, NU, FS → R (reprogramada / no ejecutada)
Baja                 → B
```

Métricas derivadas por equipo: **Programadas** = E + C2 + C3 + R ; **Ejecutadas** = E ;
**Por correctivo** = C2 + C3 ; **% avance** = E / Programadas. El registro detallado es la
**única fuente de verdad**; el resumen se calcula a partir de él.

---

## 4. Modelo de datos

Ver `datos_semilla.json`. Estructura (campos principales):

- **equipos[]** — `inv` (N° inventario, clave, texto, conserva ceros a la izquierda),
  `equipo`, `servicio`, `unidad`, `ubic`, `marca`, `modelo`, `serie`.
- **correctivo.ot[]** — `folio` (clave del expediente), `fecha`, `tecnico`, `req`
  (requerimiento), `equipo`, `inv`, `serie`, `estIni` (Operativo/No Operativo), `servicio`,
  `evento` (Abierto/Cerrado), `cierre` (fecha).
- **correctivo.lineaCompra[]** — por `folio`: `ruta`, `prov`, `nCot`, `monto`, `tipo`
  (Trato Directo/Compra Ágil), `nInf`/`fInf`/`respInf` (informe técnico), `nOC`/`fOC`, `fEnvio`.
- **correctivo.repuestos / reparacionTerreno[]** — por `folio`: `fecha`, `desc`, `estFinal`.
- **correctivo.visitas[]** — por `folio`: `fecha`, `prov`, `ingExt`, `nRep`, `fRep`, `arch`,
  `reparo`, `desc`, `estFinal`, `cierre`, `repRecibido` (Sí/No), `respCons`, `respAdm`.
- **correctivo.servicioTecnico[]** — por `folio`: `nHoja`, `respEnvia`, `fEnvio`, `empresa`,
  `nRep`, `fRep`, `fRetorno`, `guia`, `estRetorno`, `decision`, `repRecibido`, `respCons`, `respAdm`.
- **correctivo.bajas[]** — por `folio`: `folioDoc`, `resp`, `fecha`, `motivo`.
- **preventivo[]** — `inv`, `mes` (Enero…Diciembre), `ejec` (Interno/Externo), `emp`, `res`
  (los 14 códigos), `fec`, `est`, `ing` (ingeniero), `pi`/`pe` (protocolos), `obs`. La identidad
  del equipo (equipo, serie, servicio, marca, modelo) se deriva uniendo por `inv` con `equipos`.
- **pendientes[]** — `inv`, `equipo`, `serie`, `tipo`, `vence`, `situacion`, `estado`
  (Por hacer/Completado), `prioridad` (Alta/Media/Baja), `respEjec`, `respAdm`, `desc`, `tareas`,
  `completado`, `creado`. La **situación y los días de atraso se calculan en vivo** contra la
  fecha actual: Completado → Completado; vence < hoy → Vencido; si no → Vigente.
- **catalogos** — `ingenieros[]` (10 + Personal Externo), `empresas[]`, `servicios[]`,
  `estadoEquipo[]`, `tipoCompra[]`, `resultadoVisita[]`, `estadoEvento[]`, `decisionReev[]`,
  `resultadoMP[]` (14 códigos), `ejecucionMP[]`.
- **criticos[]** — N° inventario de los equipos que están a la vez en correctivo + preventivo +
  pendientes (atención integral).

Notas: N° de inventario y serie son **texto**. Las fechas vienen como `dd-mm-aaaa`.

---

## 5. Estructura de la interfaz (las cinco vistas)

1. **Panel** — pulso de un vistazo, sin editar. Tres bloques de tarjetas: *Operatividad*
   (casos abiertos, equipos no operativos, en servicio técnico, bajas, pendientes documentales),
   *Programa preventivo* (programadas, ejecutadas, % avance, reprogramadas, por correctivo),
   *Pendientes* (por hacer, vencidos, completados). Más una tabla de **equipos críticos** y una
   de **pendientes próximos / vencidos**.
2. **Pendientes** — la cola única de acción (transversal: protocolos, reprogramaciones,
   reportes…). Filtros por estado, situación, prioridad y responsable; búsqueda. Acciones:
   completar/reabrir, editar, agregar, eliminar.
3. **Equipos** — buscador + lista. Al elegir un equipo se abre su **Ficha 360**: identidad,
   estado actual (¿caso abierto?, ¿en servicio técnico?, pendientes), preventivo mes a mes, e
   historiales de correctivo, preventivo y pendientes del equipo.
4. **Correctivo** — lista de casos (OT) con su estado. Al abrir uno, su **expediente completo**:
   datos de la OT + todas las etapas asociadas por folio (reparación en terreno, línea de
   compra, repuestos, visitas, servicio técnico, baja).
5. **Preventivo** — pestaña *Resumen* (avance por mes y por servicio) y pestaña *Registro*
   (tabla filtrable de las mantenciones). Acción: registrar mantención.

### Diseño
Herramienta clínica sobria, no decorativa: el estado se lee por color (**verde** = operativo/
ejecutado, **naranja** = pendiente/reprogramado, **rojo** = crítico/no operativo/vencido,
**azul** = institucional/en proceso). Paleta: azul `#1F3A5F`, verde `#1E8E5A`, naranja
`#B45309`, rojo `#B42318`, fondo `#F4F6FA`. Navegación lateral + contenido. Responsive a móvil.

---

## 6. Personas y roles del proceso

- **Servicio clínico** — reporta la falla y genera la OT en SIGEM.
- **Jefe de Equipamiento Clínico** — asigna la OT a un ingeniero.
- **Ingeniero interno asignado** — repara en terreno, cotiza, genera informe técnico, envía la
  OC, coordina visitas, escribe la tarea en SIGEM y cierra. Los 10 ingenieros: Carlos Bahamondes
  Seguel, Cristián Beltrán Oviedo, Cristina Rozas Urrutia, Daniel Díaz Neira, Ignacio Berner
  Bergara, Macarena Toledo, Marco Ulloa, Matías Soazo Garrido, Ricardo Matus Aroca, Tito
  Millapán Riquelme (+ Personal Externo).
- **Leslie** — tramita la Orden de Compra. **Finanzas** — la emite.
- **Ingeniero externo / proveedor** — realiza visita o servicio técnico y entrega el Reporte.
- **Cristián Beltrán** — archiva los Reportes de Servicio y es el responsable administrativo de
  los pendientes documentales.

---

## 7. Sugerencias de implementación

- Cargar `datos_semilla.json` como datos iniciales (seed) en la base de datos.
- Mantener `inv` y `serie` como `string`. Guardar fechas en ISO en la base y mostrarlas en
  `dd-mm-aaaa`.
- Recalcular en vivo la situación y los días de atraso de los pendientes (no almacenarlos fijos).
- El resumen preventivo (carta/métricas) se deriva del registro, no se mantiene a mano.
- Registrar autor y fecha en cada cierre/registro. Dejar trazabilidad del expediente por folio.
- El prototipo `Sistema_MP2026_HHHA.html` ya implementa toda la lógica de cálculo y la UX:
  conviene leerlo como referencia antes de empezar.
