# Sistema de Gestión MP 2026 — HHHA

Herramienta **local y completa** de gestión de mantención de equipos clínicos del
Hospital Hernán Henríquez Aravena (HHHA). No es un visor de consulta: permite **crear y
operar todo el ciclo** —equipos, correctivo (con sus rutas, línea de compra, cierres y
bajas), preventivo y pendientes— con **todo editable** y **guardado en tu propio
computador**.

Es una sola página autocontenida (`index.html`): se abre con doble clic, funciona sin
servidor y sin internet, y trae los datos reales precargados.

## Cómo usar

1. Abre **`index.html`** en el navegador (recomendado Chrome o Edge para el autoguardado en
   archivo).
2. Trabaja en las cinco vistas desde el menú lateral.
3. Guarda una copia en tu computador desde **Datos y respaldo** (ver más abajo).

## Las cinco vistas

| Vista | Qué hace |
| --- | --- |
| **Panel** | Pulso de un vistazo: operatividad (casos abiertos, en servicio técnico, bajas, pendientes documentales, equipos críticos), avance del programa preventivo y estado de los pendientes. Tabla de equipos críticos y de pendientes próximos/vencidos. |
| **Pendientes** | Cola única de acción. Filtros por estado, situación, prioridad y responsable + búsqueda. Crear, editar, completar/reabrir y eliminar. La **situación y los días de atraso se calculan en vivo** contra la fecha de hoy. |
| **Equipos** | Buscador + lista. Al abrir un equipo, su **Ficha 360**: identidad, estado actual derivado, preventivo mes a mes e historiales de correctivo y pendientes. Crear/editar equipos. |
| **Correctivo** | Lista de casos (OT) por estado. Al abrir uno, su **expediente completo** por folio: OT + reparación en terreno + línea de compra + repuestos + visitas + servicio técnico + baja. Crear OT y operar cada etapa. |
| **Preventivo** | *Resumen* (avance por servicio, métricas E/C2/C3/R/B) y *Registro* (tabla filtrable). Registrar y editar mantenciones. El resumen se **deriva** del registro. |

## Guardado y respaldo (tus datos, en tu computador)

Desde el botón **Datos y respaldo** del menú lateral:

- **Autoguardado en el navegador** (`localStorage`): siempre activo; cada cambio se guarda solo.
- **Guardar en un archivo de mi computador**: conecta un archivo `.json` real en tu disco y
  los cambios se escriben solos en él (File System Access API, en Chrome/Edge).
- **Abrir datos desde un archivo**: continúa el trabajo desde un `.json` guardado.
- **Descargar / Importar respaldo `.json`**: funciona en cualquier navegador (también
  Firefox/Safari).
- **Restablecer a datos semilla**: vuelve a los datos originales.

> Recomendación: al empezar, usa "Guardar en un archivo" para tener tu copia maestra en el
> computador, y descarga un respaldo cada cierto tiempo.

## Datos precargados

La app viene con los datos reales embebidos (provienen de `datos_semilla.json`): **893
equipos, 14 casos correctivos** con sus etapas, **1.242 mantenciones preventivas, 59
pendientes** y los catálogos (ingenieros, empresas, servicios, etc.).

Reglas de negocio implementadas (ver `CLAUDE.md` para el detalle del proceso):

- N° de inventario y N° de serie son **texto** (conservan ceros a la izquierda).
- Situación y días de atraso de pendientes **calculados en vivo**.
- Mapeo preventivo a 6 códigos: `Sí, Si-RA → E`; `C2 → C2`; `C3 → C3`;
  `C1,C4–C8,No,NU,FS → R`; `Baja → B`. **Programadas** = E+C2+C3+R, **% avance** = E/Programadas.
- **Equipos críticos** = equipos presentes a la vez en correctivo, preventivo y pendientes.
- Dos cierres del caso correctivo: **operativo** (OT en SIGEM) y **documental**
  (reporte recibido); el pendiente documental se marca con *¿Reporte recibido? Sí/No*.

## Archivos del repositorio

| Archivo | Para qué |
| --- | --- |
| `index.html` | La aplicación completa (autocontenida, con datos embebidos). |
| `datos_semilla.json` | Datos semilla originales (referencia y para re-precargar). |
| `CLAUDE.md` | Brief del proceso, reglas y modelo de datos del sistema. |
| `modulos/` | Herramientas previas independientes (mantención correctiva y pendientes) que dieron origen a este sistema. Se conservan como referencia. |

## Notas técnicas

- Un solo archivo HTML con CSS y JavaScript *vanilla* embebidos; sin dependencias ni red.
- Probado de forma automatizada: renderizado de las cinco vistas con los datos reales,
  formularios de edición de todas las entidades y verificación de las métricas calculadas
  (avance 72%, 24 pendientes vencidos, 5 equipos críticos) contra la semilla.
- El autoguardado en archivo (escritura directa a disco) requiere un navegador con File
  System Access API (Chrome/Edge); en el resto se usa descargar/importar respaldo.
