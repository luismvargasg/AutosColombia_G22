# Plan de adaptación de la Iteración 1

## Objetivo

Adaptar de forma incremental el proyecto de referencia al alcance acordado para
la gestión de entrada, salida, reasignación y consultas del parqueadero. Cada
etapa debe ser pequeña, explicable y comprobable. Este plan no autoriza todavía
cambios de código, base de datos, dependencias ni frontend.

El contexto funcional que gobierna el plan está en
[`contexto_iteracion_1.md`](contexto_iteracion_1.md).

## Línea base comprobada

| Área | Existe actualmente | Diferencia o trabajo pendiente |
| --- | --- | --- |
| Dominio | `Vehiculo`, `Celda`, `Operador` y `Estancia` | Falta `Cliente` y la relación 1:N con `Vehiculo` |
| Mensualidad | No existe representación | Falta el estado mínimo `ACTIVA`/`INACTIVA` en la ubicación indicada por los diagramas; no se creará entidad `Mensualidad` |
| Vehículos | Modelo y CRUD completo; tipos `MOTO`, `AUTOMOVIL`, `CAMIONETA` | El CRUD administrativo queda fuera del alcance; faltan cliente y validación de mensualidad |
| Celdas | Modelo y CRUD completo; `tipo` es texto libre | Falta vocabulario definitivo, compatibilidad, consulta de disponibles y control del inventario 70/10 |
| Operadores | Modelo y CRUD completo | El CRUD administrativo queda fuera del alcance; se conservan las referencias de entrada y salida |
| Estancias | Entrada, salida y consulta individual por placa | Faltan validaciones de cliente/mensualidad/compatibilidad, reasignación y consultas globales |
| Integridad | FK restrictivas, `CHECK`, transacciones, bloqueo de salida e índices únicos parciales | Deben conservarse y ampliarse solo según el modelo definitivo |
| Ocupación | Se infiere por `estancia.salida IS NULL` | No existe filtro de celdas disponibles ni resumen; no hace falta una bandera duplicada salvo que el modelo definitivo la exija |
| Frontend | Control de acceso y tres pantallas CRUD | El contrato ya incluye el tipo de vehículo, pero la pantalla no lo muestra; tampoco muestra cliente ni mensualidad, carga todas las celdas y no tiene reasignación ni panel global; los CRUD exceden el alcance |
| Migraciones | `0001` y `0002`, lineales | La alineación debe ser una nueva revisión posterior a `0002` |
| Pruebas | Unitarias/API simulada e integración PostgreSQL optativa, incluida concurrencia | Faltan pruebas para todas las reglas nuevas |
| Artefactos de diseño | No se encontraron en el workspace | Deben proporcionarse antes de cambiar el esquema |
| Entorno local | No existen `backend/.venv` ni `backend/.env` en este clon | Preparar el entorno del repositorio antes de ejecutar pruebas, sin reutilizar otro proyecto |

## Elementos que se conservan

- FastAPI, Pydantic, SQLAlchemy, Psycopg, Alembic, PostgreSQL y Angular.
- La separación actual entre modelos, esquemas, rutas y servicios, sin añadir
  capas o interfaces artificiales.
- El contrato de `GET /health`, independiente de PostgreSQL.
- La carga privada por variables de entorno y `backend/.env`, sin credenciales en
  el repositorio.
- Las rutas relativas del frontend y la base de API centralizada.
- La normalización de placas y códigos y los errores públicos sin detalles del
  controlador.
- Las transacciones, el rollback ante fallos, las FK restrictivas y las
  restricciones que impiden estancias activas duplicadas.
- La fecha/hora generada por el servidor y el registro separado del operador de
  entrada y salida, sujetos a los nombres definitivos de los diagramas.
- Las migraciones `0001` y `0002` sin reescribirlas.
- Las pruebas de concurrencia existentes como base para las reglas nuevas.

## Elementos que se modificarán

- El modelo y el esquema físico para incorporar `Cliente`, su relación con
  `Vehiculo` y la representación mínima del estado de mensualidad exactamente
  como aparezcan en los diagramas.
- La representación de tipos de celda y las validaciones de compatibilidad.
- La consulta por placa para incluir la información necesaria de vehículo,
  cliente y mensualidad.
- La entrada para validar registro, cliente, mensualidad, estancia activa,
  compatibilidad y disponibilidad.
- Las consultas de celdas para devolver solo opciones compatibles disponibles en
  el flujo de entrada o reasignación.
- El servicio de estancias para efectuar la reasignación atómica.
- Las consultas para listar vehículos dentro y resumir ocupación/disponibilidad.
- Las pruebas unitarias, de contrato e integración correspondientes.
- La documentación de contratos compartida con el responsable de frontend.

## Elementos que se retirarán del alcance

- La administración completa de clientes, vehículos, operadores, celdas y
  mensualidades.
- Pagos, facturación, tarifas, roles, permisos y vencimientos automáticos.
- Cualquier funcionalidad de novedades u observaciones.
- Las pantallas CRUD administrativas como parte del recorrido entregable.

Los CRUD heredados no se eliminarán al comienzo: el control de acceso actual y
la preparación manual de datos todavía dependen de parte de ellos. Primero se
implementarán contratos operativos y se acordará la precarga de datos. Después,
el responsable de frontend podrá retirar sus rutas y navegación, y el equipo
decidirá si los endpoints de escritura heredados se eliminan o quedan fuera de
la superficie demostrada. Esto evita romper el proyecto antes de tener un
reemplazo y no los convierte en requisitos de la iteración.

## Orden de implementación

### Etapa 0 - Confirmar los diagramas definitivos

**Objetivo:** eliminar ambigüedades antes de crear DDL.

Acciones:

1. Obtener MER, modelo relacional, modelo físico y diagrama de clases definitivos.
2. Completar una matriz atributo por atributo entre diagramas y código actual.
3. Resolver los puntos enumerados en la sección de información pendiente.
4. Actualizar estos documentos si una decisión confirmada cambia el plan.

Archivos afectados:

- `docs/contexto_iteracion_1.md`;
- `docs/plan_adaptacion_iteracion_1.md`; y
- los archivos fuente de los diagramas, si el equipo decide versionarlos.

Criterio de salida: no queda ningún nombre, tipo, longitud, nulabilidad, estado o
restricción física por inventar para la primera migración.

### Etapa 1 - Alinear base de datos y modelos

Esta es la **primera tarea de implementación posterior**.

Acciones:

1. Añadir `Cliente` al ORM con los atributos exactos del modelo físico.
2. Añadir la relación `Cliente` 1:N `Vehiculo` y su FK exacta.
3. Incorporar el estado mínimo de mensualidad donde indiquen los diagramas, sin
   crear una entidad `Mensualidad`.
4. Alinear los atributos y categorías de `Vehiculo`, `Celda`, `Operador` y
   `Estancia` con los diagramas.
5. Conservar o adaptar explícitamente las restricciones de estancia activa única,
   cierre completo, orden de fechas y FK restrictivas.
6. Definir en la migración cómo preservar o transformar filas existentes, sin
   fabricar asociaciones, estados ni categorías no aprobadas.
7. Crear una revisión Alembic lineal cuyo `down_revision` sea `0002`; no editar
   `0001` ni `0002`.
8. Añadir pruebas que comparen ORM, migración y DDL PostgreSQL sin conectarse a la
   base del usuario.
9. Generar y revisar `upgrade head --sql` y el tramo aislado
   `upgrade 0002:<nueva_revision> --sql` con el intérprete de `backend/.venv`.
   Ninguno conecta ni aplica DDL. No aplicar la migración sin autorización
   independiente.

Archivos previstos:

- `backend/app/models.py`;
- `backend/migrations/versions/<nueva_revision>_alinear_modelo_iteracion_1.py`;
- `backend/tests/test_models.py`;
- `backend/tests/test_estancia.py`; y
- `backend/tests/test_modelo_iteracion_1.py` como prueba focalizada nueva si la
  cobertura no queda clara en los archivos existentes.

Los esquemas API no se cambiarán dentro de este bloque salvo que sea necesario
para mantener la importación y las pruebas; sus contratos se adaptarán en la
etapa siguiente.

Criterios de aceptación:

- existen exactamente las cinco entidades de dominio acordadas;
- no existen entidades independientes `Salida` o `Mensualidad`;
- nombres, tipos, longitudes, nulabilidad y relaciones coinciden con los
  diagramas definitivos;
- la nueva revisión continúa desde `0002` y el SQL offline es revisable;
- se preserva la integridad de las estancias y `/health` no cambia; y
- ninguna migración se aplica a la base local sin autorización.

### Etapa 2 - Consulta por placa y validaciones previas

**Objetivo:** obtener la información necesaria antes de autorizar una entrada.

Acciones:

- definir el esquema de lectura de `Cliente` sin construir su CRUD;
- mantener los datos actuales del vehículo, incluido su tipo, y ampliar la
  consulta por placa con los datos acordados de cliente y mensualidad;
- implementar validaciones de vehículo registrado, cliente asociado,
  mensualidad activa y ausencia de estancia activa; y
- mantener mensajes públicos y códigos HTTP documentados.

Archivos previstos:

- `backend/app/schemas/cliente.py` (nuevo, solo contratos necesarios);
- `backend/app/schemas/vehiculo.py`, si el contrato anidado se amplía allí;
- `backend/app/schemas/estancia.py`;
- `backend/app/services/estancia.py`;
- `backend/app/routers/estancias.py`;
- `backend/tests/test_estancias_api.py`; y
- `backend/tests/test_estancias_postgres.py`.

### Etapa 3 - Disponibilidad, compatibilidad y entrada

**Objetivo:** que el operador solo pueda seleccionar una celda válida.

Acciones:

- consultar celdas sin estancia activa y compatibles con el vehículo;
- validar de nuevo compatibilidad y disponibilidad dentro de la transacción de
  entrada, sin confiar en la selección del frontend;
- mantener la garantía de una estancia activa por vehículo y por celda; y
- comprobar el inventario esperado de 70 celdas de carro y 10 de moto según el
  mecanismo de datos que se acuerde.

Archivos previstos:

- `backend/app/schemas/celda.py`;
- `backend/app/services/celda.py`;
- `backend/app/routers/celdas.py`;
- `backend/app/services/estancia.py`;
- `backend/tests/test_celdas_postgres.py`;
- `backend/tests/test_estancias_api.py`; y
- `backend/tests/test_estancias_postgres.py`.

### Etapa 4 - Salida y reasignación atómica

**Objetivo:** conservar la salida existente y añadir el cambio seguro de celda.

Acciones:

- mantener el cierre con fecha/hora del servidor y operador de salida;
- definir el contrato de reasignación una vez confirmado el modelo;
- bloquear las filas necesarias para evitar carreras;
- comprobar estancia activa, celda nueva distinta, compatibilidad y
  disponibilidad dentro de una transacción; y
- confirmar que un fallo conserva la celda anterior y no ocupa la nueva.

Archivos previstos:

- `backend/app/schemas/estancia.py`;
- `backend/app/services/estancia.py`;
- `backend/app/routers/estancias.py`;
- `backend/tests/test_estancia.py`;
- `backend/tests/test_estancias_api.py`; y
- `backend/tests/test_estancias_postgres.py`.

### Etapa 5 - Consultas operativas

**Objetivo:** completar el panel del operador.

Acciones:

- listar vehículos con estancia activa y sus datos necesarios;
- resumir ocupación y disponibilidad total y por categoría de celda; y
- comprobar que las lecturas derivan el estado de una fuente consistente.

Archivos previstos:

- `backend/app/schemas/estancia.py` y/o un esquema de consultas específico si
  evita mezclar contratos;
- `backend/app/services/estancia.py` y/o
  `backend/app/services/consultas.py` si las consultas justifican separarlo;
- `backend/app/routers/estancias.py` y/o un router de consultas coherente;
- `backend/app/main.py` solo si se registra un router nuevo;
- `backend/tests/test_estancias_api.py`; y
- `backend/tests/test_estancias_postgres.py`.

No se creará un módulo vacío únicamente para reproducir el nombre de una capa del
diagrama.

### Etapa 6 - Coordinación del frontend y depuración del alcance visible

Esta etapa corresponde al compañero responsable del frontend. El backend debe
entregar antes los contratos confirmados.

Archivos que probablemente deberá adaptar el responsable de frontend:

- `frontend/src/app/models/vehiculo.ts`;
- `frontend/src/app/models/celda.ts`;
- `frontend/src/app/models/estancia.ts`;
- `frontend/src/app/models/cliente.ts`, si se decide separar ese contrato;
- `frontend/src/app/services/estancias.service.ts`;
- `frontend/src/app/services/celdas.service.ts`;
- `frontend/src/app/control-acceso/control-acceso.ts`;
- `frontend/src/app/control-acceso/control-acceso.html`;
- `frontend/src/app/control-acceso/control-acceso.css`;
- `frontend/src/app/control-acceso/control-acceso.spec.ts`;
- `frontend/src/app/app.routes.ts`;
- `frontend/src/app/app.html`; y
- `frontend/proxy.conf.json` solo si aparece un prefijo API nuevo.

Si el panel se implementa como un componente separado, su nombre y sus archivos
se acordarán con el responsable de frontend en vez de inventarlos en este plan.

Resultados esperados:

- formularios de entrada y salida y panel de consultas alineados;
- selector alimentado por celdas compatibles disponibles;
- visualización de vehículo, cliente y validación de mensualidad;
- reasignación y consultas globales; y
- pantallas CRUD excluidas fuera de la navegación/entrega, después de acordar la
  estrategia de datos.

## Cambios de contrato API que debe conocer el frontend

Los nombres exactos de campos y rutas nuevas no se fijarán antes de revisar los
diagramas. Sí están confirmados estos cambios conceptuales:

| Contrato actual | Cambio requerido |
| --- | --- |
| `GET /estancias/vehiculo/{placa}` devuelve el vehículo, incluido su tipo, y el estado dentro/fuera | La consulta operativa debe incorporar los datos acordados de cliente y mensualidad, y el frontend debe mostrar el tipo. Se confirmará si se amplía esta ruta o se define otra forma contractual |
| `GET /celdas` es un listado administrativo sin filtro | El flujo operativo debe consumir una consulta backend de celdas compatibles disponibles; se definirá si es una variante de esta ruta o un contrato separado |
| `POST /estancias/entrada` recibe placa, celda y operador | Puede conservar esa forma, pero el backend añadirá todas las validaciones de negocio y el contrato final dependerá de los diagramas |
| `POST /estancias/salida` recibe placa y operador | Se conserva conceptualmente; los nombres públicos se contrastarán con el diagrama de clases y el contrato acordado |
| No existe reasignación | Se añadirá un contrato atómico después de confirmar cómo identificar la estancia y la celda nueva |
| No existe listado global | Se añadirá una consulta de vehículos dentro |
| No existe resumen | Se añadirá una consulta de ocupación/disponibilidad total y por tipo |

Los nombres físicos actuales `entrada`, `salida`, `operador_entrada_id` y
`operador_salida_id` deben contrastarse con el modelo físico. Los nombres públicos
de la API se contrastarán con el diagrama de clases y el contrato acordado; un
cambio de columna no obliga por sí solo a romper la API. Cualquier cambio público
incompatible se documentará antes de modificar los tipos TypeScript.

## Estrategia de pruebas

### Sin PostgreSQL real

- correspondencia entre ORM y migraciones;
- compilación de DDL para PostgreSQL y SQL offline de Alembic;
- validación estricta de solicitudes y respuestas Pydantic;
- estados HTTP y cuerpo esperado con sesiones simuladas;
- regresión de `/health`; y
- pruebas unitarias de reglas que no requieran concurrencia real.

### Integración PostgreSQL optativa

Solo con autorización, `backend/.venv`, configuración privada válida y
`RUN_POSTGRES_TESTS=1`:

- FK, nulabilidad, unicidad, `CHECK` e índices;
- cliente 1:N vehículos y estado de mensualidad;
- compatibilidad y disponibilidad de celdas;
- entrada/salida y conflictos;
- reasignación correcta y rollback sin cambios parciales;
- carreras por el mismo vehículo, la misma celda, salida y reasignación; y
- exactitud del listado de vehículos dentro y del resumen de ocupación.

Las pruebas no deben crear tablas ni alterar migraciones para prepararse. Deben
aislar datos y conservar el rollback o la limpieza exacta existente.

### Frontend

A cargo del compañero:

- contratos HTTP simulados para consulta previa, disponibles, entrada, salida,
  reasignación y panel;
- invalidación de respuestas tardías y bloqueo de doble envío;
- estados vacíos y errores de negocio; y
- compilación con `strict` y `strictTemplates` y suite sin modo observación.

## Información necesaria antes de la Etapa 1

No se puede realizar de forma segura la primera modificación del esquema sin:

1. los atributos completos de las cinco entidades;
2. nombres físicos exactos de tablas, columnas, PK y FK;
3. tipos, longitudes, nulabilidad, valores iniciales y restricciones;
4. ubicación, nombre, tipo y valores del estado de mensualidad;
5. literal canónico de cada tipo de vehículo y celda;
6. definición de si la disponibilidad se deriva de la estancia activa;
7. mecanismo previsto para las 80 celdas y sus códigos/datos iniciales;
8. confirmación de los nombres definitivos de los campos de `Estancia`;
9. decisión del diseño sobre si una reasignación sustituye la celda sin guardar
   historial; y
10. firmas o contratos del diagrama de clases que deban reflejarse en el código;
    y
11. estrategia de migración para asociar vehículos existentes a clientes,
    inicializar mensualidades y convertir los valores actuales de tipos de
    vehículo y celda sin inventar datos.

Estos datos deben provenir de los diagramas definitivos, que no se encontraron en
el workspace. Hasta recibirlos no se crearán columnas, longitudes, estados ni
entidades por inferencia.

## Estado de verificación al crear este plan

- Raíz, rama `backend_dev`, remoto y seguimiento `origin/backend_dev`: verificados.
- Árbol de trabajo inicial: limpio.
- Inspección estática de backend, frontend, migraciones y pruebas: completada.
- `backend/.venv`: ausente.
- `backend/.env`: ausente; no se leyó ni mostró ningún secreto.
- Variables `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` y
  `DB_CONNECT_TIMEOUT`: ausentes del proceso inspeccionado; no se consultaron
  valores.
- `RUN_POSTGRES_TESTS`: ausente del proceso inspeccionado.
- `pip check`: no ejecutado porque falta el entorno virtual del proyecto.
- `pytest`: no ejecutado porque falta el entorno virtual del proyecto.
- PostgreSQL: no se intentó conexión y no se aplicaron migraciones.
- SQL offline de Alembic: no generado porque falta el intérprete y las
  dependencias del `backend/.venv` autorizado.
- Build y pruebas de frontend: no ejecutados; esta tarea solo modifica
  documentación e instrucciones.
