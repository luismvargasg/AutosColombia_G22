# Contexto de la Iteración 1

## Propósito del documento

Este documento conserva el contexto funcional y técnico acordado para la
**S25 - Evidencia de Aprendizaje 1. Diseño Sistema de Información - Iteración 1**
de Autos Colombia. Debe consultarse antes de proponer cambios de alcance,
contratos API, modelos o migraciones.

La adaptación se realiza en el repositorio compartido
`luismvargasg/AutosColombia_G22` y en la rama `backend_dev`. El proyecto de
referencia es una base técnica autorizada, pero sus funcionalidades no definen
por sí solas el alcance académico del grupo.

## Alcance funcional

El único actor de esta iteración es el **Operador**. El sistema debe permitir:

- consultar un vehículo por placa;
- mostrar sus datos y los del cliente asociado;
- comprobar que el vehículo esté registrado;
- permitir la verificación visual de placa, marca, color y tipo;
- validar que la mensualidad del vehículo esté activa;
- impedir una segunda entrada cuando el vehículo ya tenga una estancia activa;
- consultar celdas disponibles y compatibles con el tipo de vehículo;
- asignar una celda y registrar la entrada;
- consultar la estancia activa de un vehículo;
- registrar la salida y dejar disponible la celda;
- reasignar la celda de una estancia activa de forma atómica;
- consultar los vehículos que se encuentran dentro; y
- consultar el estado general y la disponibilidad del parqueadero.

Los datos mínimos necesarios de clientes, vehículos, operadores y estado de
mensualidad pueden precargarse para demostrar los flujos. Esta posibilidad no
autoriza a crear módulos administrativos que no estén en el alcance.

## Funcionalidades excluidas

No pertenecen a esta entrega:

- CRUD completo de clientes;
- CRUD completo de vehículos como módulo administrativo;
- CRUD completo de operadores o usuarios;
- administración completa de celdas;
- pagos, facturación y tarifas;
- roles y permisos;
- administración avanzada de mensualidades;
- vencimientos automáticos de mensualidades; y
- novedades u observaciones del vehículo.

La funcionalidad de novedades fue descartada expresamente. No deben añadirse
campos, métodos, rutas ni pantallas relacionados con ella.

Los CRUD administrativos heredados del proyecto de referencia existen en el
código actual, pero no forman parte del recorrido funcional de esta iteración.
Su tratamiento se hará de forma coordinada y sin retirarlos antes de contar con
los contratos operativos y la estrategia de datos que los sustituyan.

## Reglas de negocio

- El parqueadero funciona únicamente por mensualidad.
- La mensualidad corresponde a un vehículo concreto, no al cliente en general
  ni a una celda fija.
- Un cliente puede tener varios vehículos; cada vehículo requiere su propia
  mensualidad.
- Las únicas categorías de vehículo permitidas son motocicleta, automóvil y
  camioneta. No se permiten camiones.
- La forma canónica que se almacenará para cada categoría aún debe comprobarse
  en los diagramas; en particular, el material disponible usa tanto
  `MOTOCICLETA` como `MOTO` para la primera categoría.
- El parqueadero tiene exactamente 80 celdas: 70 para automóviles/camionetas y
  10 para motocicletas.
- Una motocicleta solo puede ocupar una celda de moto.
- Un automóvil o una camioneta solo puede ocupar una celda de carro.
- Las celdas no se asignan de forma permanente; se eligen según disponibilidad.
- Puede haber varios operadores. El operador que registra la salida puede ser
  distinto del que registró la entrada.
- Para esta iteración solo se necesita el estado `ACTIVA`/`INACTIVA` de la
  mensualidad al decidir el ingreso. Su ubicación y representación física deben
  confirmarse en los diagramas.
- No se debe crear una entidad independiente `Mensualidad`.
- Una estancia activa debe ser única por vehículo y por celda.
- La reasignación debe validar compatibilidad y disponibilidad, liberar la celda
  anterior y ocupar la nueva dentro de una sola transacción. Un fallo no puede
  dejar cambios parciales.
- Las reglas se ejecutan en el backend aunque también se representen en la
  interfaz.

## Flujos acordados

### Entrada

1. El operador consulta la placa.
2. El sistema verifica que el vehículo esté registrado y asociado a un cliente.
3. El sistema muestra los datos del vehículo y del cliente.
4. El operador verifica visualmente placa, marca, color y tipo.
5. El sistema valida la mensualidad activa del vehículo.
6. El sistema comprueba que el vehículo no tenga una estancia activa.
7. El sistema consulta las celdas compatibles disponibles.
8. El operador selecciona una celda.
9. El sistema registra la entrada y la celda queda ocupada por la estancia
   activa.

### Salida

1. El operador consulta la placa.
2. El sistema verifica que exista una estancia activa.
3. El sistema registra fecha/hora y operador de salida.
4. El sistema finaliza la estancia.
5. La celda queda disponible.

### Reasignación

1. El sistema identifica la estancia activa.
2. El operador selecciona una nueva celda.
3. El sistema comprueba compatibilidad y disponibilidad.
4. El sistema actualiza la celda de la estancia.
5. El cambio libera la celda anterior y ocupa la nueva de forma atómica.

## Modelo de dominio vigente

El modelo definitivo del grupo tiene cinco entidades:

1. `Cliente`.
2. `Vehiculo`.
3. `Celda`.
4. `Operador`.
5. `Estancia`.

No se crearán entidades independientes `Salida` ni `Mensualidad`. La salida
forma parte de `Estancia`.

Relaciones confirmadas:

- `Cliente` 1:N `Vehiculo`;
- `Vehiculo` 1:N `Estancia`;
- `Celda` 1:N `Estancia`;
- `Operador` 1:N `Estancia` como operador de entrada; y
- `Operador` 1:N `Estancia` como operador de salida.

En una estancia activa, la fecha/hora de salida y el identificador del operador
de salida admiten `NULL`. La lista completa de atributos, sus nombres físicos y
sus tipos todavía no se considera confirmada.

## Arquitectura acordada y correspondencia sencilla

La planeación identifica las siguientes responsabilidades:

- Interfaz: `FormularioEntrada`, `FormularioSalida` y `PanelConsultas`.
- Lógica: `ControlEstancia`, `ServicioValidacion`, `GestionCeldas` y
  `ServicioConsultas`.
- Acceso a datos: `AccesoDatos` y PostgreSQL.
- Contratos del diseño: `IControlEstancia`, `IValidacion`, `IGestionCeldas`,
  `IConsultas` e `IAccesoDatos`.

Su correspondencia en la base técnica actual será directa:

- Los formularios y el panel son componentes Angular, a cargo del responsable
  de frontend.
- Las rutas y el servicio de estancias coordinan entrada, salida, consulta de la
  estancia activa y futura reasignación.
- Las validaciones de vehículo, cliente, mensualidad, ingreso activo y
  compatibilidad deben permanecer en servicios de backend. Pueden ser funciones
  cohesivas dentro de esos servicios o extraerse si realmente se reutilizan.
- El servicio de celdas resuelve consultas de disponibilidad/compatibilidad; el
  cambio que ocupa o libera una celda se coordina mediante la transacción de la
  estancia.
- Las consultas globales pueden agruparse en un servicio de consultas cuando se
  implementen, evitando duplicar SQL.
- `backend/app/database.py`, la sesión de SQLAlchemy y PostgreSQL cumplen la
  responsabilidad de acceso a datos.
- Las interfaces del diseño expresan contratos y responsabilidades. No es
  obligatorio crear clases abstractas, protocolos o capas vacías en Python si
  no aportan una necesidad concreta.

Las operaciones del diseño pueden usar `snake_case` en Python:
`registrar_entrada`, `registrar_salida`, `consultar_estancia_activa`,
`validar_vehiculo`, `validar_mensualidad`, `validar_ingreso_activo`,
`consultar_disponibles`, `asignar_celda`, `ocupar_celda`, `liberar_celda`,
`reasignar_celda`, `consultar_vehiculos_dentro` y
`consultar_estado_parqueadero`.

## Responsabilidades del equipo

- El responsable de backend trabaja la adaptación en `backend_dev`: modelo,
  migraciones, validaciones, servicios, rutas y pruebas de backend.
- El compañero es responsable del frontend y de otros entregables del documento.
- Los cambios de contrato API deben documentarse y comunicarse antes de que el
  compañero adapte los modelos, servicios, componentes y pruebas Angular.
- No se modificará frontend desde el trabajo de backend salvo acuerdo o solicitud
  expresa.
- No se harán commits, publicaciones, cambios de rama ni operaciones destructivas
  sin una solicitud explícita.

## Decisiones y estado confirmados al preparar la adaptación

- La rama local es `backend_dev`, sigue a `origin/backend_dev` y el remoto
  `origin` corresponde al repositorio compartido.
- El código actual tiene cuatro entidades de dominio: `Vehiculo`, `Celda`,
  `Operador` y `Estancia`. Falta `Cliente`.
- `Estancia` ya reúne entrada y salida; no existe una entidad `Salida`.
- El modelo actual impide dos estancias abiertas para un mismo vehículo o una
  misma celda mediante índices únicos parciales.
- El cierre exige fecha y operador de salida juntos, y una salida no anterior a
  la entrada. Las claves foráneas usan eliminación restringida.
- La ocupación se deriva de una estancia con salida nula; `Celda` no tiene una
  bandera de ocupación.
- Entrada, salida y consulta individual por placa ya existen.
- La salida bloquea la estancia activa y las operaciones de estancia se ejecutan
  en transacciones. Estas garantías deben conservarse.
- Faltan `Cliente`, validación de mensualidad, compatibilidad vehículo/celda,
  consulta de celdas compatibles disponibles, reasignación, listado de vehículos
  dentro y resumen de ocupación.
- Los listados actuales de celdas no filtran disponibilidad ni compatibilidad.
- Los CRUD completos de vehículos, celdas y operadores y sus pantallas Angular
  son heredados y exceden el alcance funcional vigente.
- Las migraciones existentes son `0001` (vehículo, celda y operador) y `0002`
  (estancia). Cualquier alineación futura se hará con una revisión posterior;
  no se reescribirán revisiones aplicadas.
- No hay `autos_colombia.sql` ni otros archivos SQL en el árbol inspeccionado.
- No hay diagramas de MER, modelo relacional, modelo físico o clases en el
  workspace inspeccionado.
- Al preparar este documento no existían `backend/.venv` ni `backend/.env`. La
  plantilla `backend/.env.example` sí existe. No se reutilizará el entorno de
  otro proyecto ni se mostrarán secretos.

## Información pendiente de los diagramas definitivos

Antes de modificar modelos o crear una migración se necesita contrastar, como
mínimo, lo siguiente:

### Definiciones transversales

- nombres exactos de tablas, columnas, atributos y relaciones;
- tipo PostgreSQL/Python de cada atributo y longitud de cada texto;
- nulabilidad, valores predeterminados y campos generados;
- claves primarias y foráneas, unicidad, índices y restricciones `CHECK`;
- comportamiento previsto al actualizar o eliminar referencias; y
- correspondencia exacta entre modelo físico, modelo relacional y clases; y
- estrategia para conservar o transformar datos existentes al añadir relaciones,
  columnas obligatorias o catálogos, sin inventar clientes, estados ni tipos.

### Cliente

- lista completa de atributos;
- identificador y estrategia de generación;
- campos obligatorios, únicos y sus formatos; y
- nombre exacto de la relación y de la clave foránea desde `Vehiculo`.

### Vehiculo y mensualidad

- atributos que se conservan o cambian respecto a `placa`, `color`, `modelo`,
  `marca` y `tipo`;
- nombre, tipo y nulabilidad de la clave foránea a `Cliente`;
- ubicación exacta del estado de mensualidad sin crear una entidad nueva;
- nombre, tipo, valores canónicos y valor inicial de ese estado; y
- valores canónicos del tipo de vehículo, especialmente `MOTO` frente a
  `MOTOCICLETA` y la representación de `AUTOMOVIL`.

### Celda

- atributos definitivos;
- valores canónicos para distinguir celda de moto y celda de carro;
- si la disponibilidad seguirá derivándose exclusivamente de la estancia activa;
- códigos concretos de las 80 celdas y mecanismo de precarga; y
- si la distribución 70/10 es solo un conjunto controlado de datos o también
  requiere alguna restricción física verificable.

### Operador

- atributos definitivos y si `nombre` y `telefono` actuales se conservan;
- restricciones, longitudes y formatos; y
- nombres exactos de las dos relaciones con `Estancia`.

### Estancia y reasignación

- nombres definitivos de sus atributos, incluidos fecha/hora de entrada y salida
  e identificadores de operadores;
- tipos, zona horaria, valores predeterminados y nulabilidad;
- si se conservan los índices únicos parciales y los `CHECK` actuales con esos
  nombres o deben expresarse de otro modo; y
- si la reasignación sustituye la celda de la estancia activa sin conservar un
  historial de celdas. No se añadirá una entidad o atributo de auditoría que no
  aparezca en el diseño acordado.

También se necesitan las firmas o datos definitivos del diagrama de clases que
deban reflejarse en los contratos, sin convertir automáticamente cada interfaz
del diseño en una clase abstracta de Python.

Para una base ya poblada se debe acordar, además, cómo asociar los vehículos
existentes con clientes, cómo inicializar el estado de mensualidad y cómo mapear
los valores actuales de `Vehiculo.tipo` y el texto libre de `Celda.tipo`. Una
migración no asignará valores ficticios por iniciativa propia.
