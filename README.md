# Gestión MP 2026 · Mantención Correctiva de Equipamiento Clínico

Aplicación web **de un solo archivo, autocontenida y sin backend** para gestionar
la mantención correctiva de equipamiento clínico hospitalario. Reemplaza el libro
Excel conservando exactamente la misma lógica de datos, pero con validaciones, una
vista consolidada por caso (**Expediente**) y un **Tablero** con indicadores
calculados.

Todo el sistema gira en torno al **Folio SIGEM**: cada caso correctivo es una
**Orden de Trabajo (OT)** identificada por su Folio SIGEM, y ese folio vincula
todas las etapas del expediente (terreno, visitas, compras, repuestos, envíos a
servicio técnico y bajas).

## Cómo usarla

1. Abre **`index.html`** en cualquier navegador moderno (Chrome, Edge, Firefox).
   No requiere instalación, servidor ni conexión a internet.
2. Ve a **Importar / Exportar** y carga el listado maestro de equipos desde un
   archivo `.xlsx` o `.csv`.
3. Crea **Órdenes de Trabajo** y registra las etapas del caso. Revisa el avance en
   el **Tablero** y el detalle en el **Expediente** de cada folio.
4. Respalda periódicamente con **Exportar respaldo (JSON)**.

> Los datos se guardan automáticamente en el `localStorage` del navegador. Son
> locales a ese equipo y navegador: usa el respaldo JSON para moverlos o protegerlos.

## Funcionalidades

- **Tablero** con KPIs (casos abiertos, equipos No Operativos, en servicio técnico,
  dados de baja, cerrados) y tabla de casos abiertos con **días abierto** resaltado
  en rojo cuando supera 30 días.
- **CRUD completo** de Órdenes de Trabajo y de todas las etapas, más Equipos y
  Catálogos.
- **Autocompletado**: al ingresar el N° Inventario en la OT se rellenan Equipo,
  N° Serie y Servicio desde el maestro (solo lectura).
- **Expediente por Folio SIGEM**: línea de tiempo cronológica con la OT y todos sus
  registros vinculados.
- **Búsqueda y filtros** por folio, equipo, servicio, técnico, estado y rango de
  fechas.
- **Validaciones**: Folio SIGEM único y obligatorio, campos clave obligatorios,
  selector de fecha, y preservación de ceros a la izquierda en Inventario y Serie.
- **Importar equipos** (Excel/CSV) y **respaldo/restauración** completa en JSON.
- **Exportar a Excel** (`.xlsx`) cada módulo y el Tablero.
- Interfaz en **español (Chile)**, responsive (escritorio/tablet) y accesible por
  teclado.

## Reglas de negocio implementadas

- **Estado actual** de un caso: si está en Bajas → `Baja`; si tiene un envío a
  servicio técnico sin fecha de retorno → `Servicio Técnico`; en otro caso → el
  `Estado inicial` de la OT.
- El **Tablero** lista solo casos cuyo `Estado del evento` es `Abierto`.
- **Días abierto** = hoy − `Fecha OT` (rojo cuando supera 30 días).
- **Monto cotizado** formateado como moneda chilena (CLP). Fechas en formato
  chileno **dd-mm-aaaa**.

## Decisiones de implementación

- **Un solo archivo, sin build, sin frameworks.** Todo el HTML, CSS y JavaScript
  (vanilla) está embebido en `index.html`. El código está modularizado por
  secciones comentadas en español (utilidades, capa de datos, cálculos, esquemas,
  enrutador, vistas, importación/exportación).
- **Formularios y listas dirigidos por esquema.** Cada entidad se describe de forma
  declarativa (campos, tipos, columnas) en el objeto `ENTIDADES`, de modo que un
  único motor genérico renderiza listas, formularios, validaciones y exportaciones.
  Así se evita duplicar código y se mantiene la consistencia.
- **Lectura de `.xlsx` 100 % offline, sin librerías externas.** Un `.xlsx` es un
  ZIP con XML adentro. La app interpreta la estructura ZIP en JavaScript y
  descomprime cada entrada con la API nativa del navegador
  `DecompressionStream('deflate-raw')`; luego parsea `sharedStrings.xml` y la
  primera hoja. No se usa SheetJS ni ninguna dependencia, por lo que **funciona sin
  conexión**. Si un navegador muy antiguo no soporta `DecompressionStream`, la app
  lo informa y sugiere importar el archivo como CSV.
- **Escritura de `.xlsx` sin dependencias.** La exportación genera un XLSX válido
  construyendo el ZIP a mano con entradas *almacenadas* (sin compresión), por lo que
  solo requiere el cálculo de CRC-32 y tampoco depende de librerías.
- **Inventario y Serie siempre como texto.** Nunca se convierten a número, para
  conservar los ceros a la izquierda (p. ej. `00039`). El parser CSV detecta el
  delimitador (`,`, `;` o tabulación), respeta comillas y elimina el BOM.
- **Integridad referencial.** El Folio SIGEM de una OT es inmutable al editar
  (clave primaria). Al eliminar una OT se eliminan en cascada sus registros
  vinculados, con confirmación previa.
- **Fechas sin corrimiento de zona horaria.** Se almacenan en ISO (`aaaa-mm-dd`) y
  se parsean por componentes para mostrarlas como `dd-mm-aaaa` sin desfases.
- **Servicios clínicos auto-derivados** de los valores únicos del campo `Servicio`
  de los equipos importados, además de permitir agregarlos manualmente.

## Verificación

La lógica fue verificada con pruebas automatizadas (reglas de negocio, fechas,
moneda, parser CSV, lector/escritor XLSX) y con una batería de pruebas
*end-to-end* en un navegador real (Chromium) que cubre importación, autocompletado,
creación de casos, cálculo de estado, filtros del tablero, persistencia tras
recarga y exportaciones.
