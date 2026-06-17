# EQUIPAMIENTOCLINICOHHHA6

Sistema unificado de gestión de **equipamiento clínico** del Hospital HHHA.

La aplicación reúne dos módulos independientes bajo una sola navegación
(`index.html`). Cada módulo conserva su propio funcionamiento y se carga en
su propio documento mediante un `<iframe>`, de modo que sus estilos, su
JavaScript y sus datos están completamente aislados entre sí.

## Módulos

| Módulo | Archivo | Descripción |
| --- | --- | --- |
| **Mantención Correctiva** | `modulos/mantencion-correctiva.html` | Registro de mantención correctiva de equipos (reemplaza la planilla Excel). Incluye órdenes de trabajo, inventario de equipos, cierre de casos e importación/exportación a Excel. |
| **Gestión de Pendientes** | `modulos/pendientes.html` | Seguimiento de pendientes de equipos médicos: estados, prioridades, vencimientos, agrupaciones y exportación. |

## Uso

Abre **`index.html`** en el navegador. La barra superior permite cambiar entre
los dos módulos:

- Pestañas **Mantención Correctiva** / **Pendientes**.
- Atajos de teclado **Alt+1** y **Alt+2**.
- Enlaces directos por URL: `index.html#correctiva` e `index.html#pendientes`.
- Botón **Abrir en pestaña** para ver el módulo actual a pantalla completa.

El módulo activo se recuerda entre sesiones. Cada módulo guarda su propia
información en `localStorage` del navegador, con claves distintas que no
colisionan:

- Mantención Correctiva → `hhha_correctivo_v1`
- Pendientes → `pendientesMP_v2`, `pendientesCols_v2`

## Notas técnicas

- Funciona sin servidor: basta abrir `index.html` (o servir la carpeta de
  forma estática, p. ej. con GitHub Pages). Los tres archivos deben mantenerse
  juntos respetando la carpeta `modulos/`.
- Cada módulo es autocontenido (CSS y JavaScript embebidos). La única
  dependencia externa es la tipografía de Google Fonts del módulo de
  Mantención Correctiva, opcional: la app funciona igual sin conexión.
