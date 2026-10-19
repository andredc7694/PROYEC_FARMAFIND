# Principios de diseño (SOLID)

## Objetivo

Documentar cómo se aplicarán los principios SOLID en el backend de FarmaFind, organizado mediante Clean Architecture, para mantener las responsabilidades separadas, facilitar las pruebas y evitar que las reglas de negocio dependan directamente de las tecnologías utilizadas.

El sistema utilizará PostgreSQL para la persistencia, Redis como caché obligatoria de búsquedas y un servicio independiente de Notificaciones.

## Contexto

- **Estilo:** monolito modular con servicio independiente de Notificaciones.
- **Backend:** Node.js + Express + TypeScript.
- **Frontend:** React + Vite + JavaScript (JSX).
- **Enfoque interno:** Clean Architecture: presentación, aplicación, dominio e infraestructura.
- **Base de datos:** PostgreSQL.
- **Caché:** Redis obligatorio en el MVP mediante Cache-Aside.
- **Módulo de referencia:** Reservas, porque integra reglas de negocio, inventario, transacciones PostgreSQL y eventos de notificación.
- **Patrones relacionados:** Repository, Adapter, Dependency Injection, Unit of Work y Transactional Outbox.

## SOLID en el módulo Reservas

![SOLID en el módulo Reservas](../../../img/solid-modulo-reservas.png)

| Principio | Dónde se aplica (capa) | Cómo se cumple en FarmaFind | Archivos |
|---|---|---|---|
| **S** · Responsabilidad única | Todas las capas | Cada componente tendrá una responsabilidad: el controlador recibe solicitudes HTTP, el caso de uso coordina la reserva, la entidad valida sus reglas, el repositorio persiste los datos y Unit of Work coordina las transacciones. | `reservas.controller.ts`, `crear-reserva.use-case.ts`, `reserva.ts`, `postgres-reserva.repository.ts`, `postgres-unidad-trabajo.ts` |
| **O** · Abierto/cerrado | Aplicación e infraestructura | Se podrán agregar nuevos adaptadores de infraestructura sin modificar las reglas de negocio. Por ejemplo, Búsqueda utilizará Redis mediante un puerto de caché y Notificaciones utilizará un adaptador para publicar eventos. | `busqueda-cache.port.ts`, `redis-busqueda-cache.adapter.ts`, `notificacion-port.ts`, `postgres-outbox.adapter.ts` |
| **L** · Sustitución de Liskov | Aplicación e infraestructura | Los adaptadores que implementan un mismo contrato deberán poder sustituirse respetando su comportamiento. Por ejemplo, el adaptador Redis podrá reemplazarse por uno simulado durante las pruebas sin cambiar los casos de uso. | `busqueda-cache.port.ts`, `redis-busqueda-cache.adapter.ts`, `notificacion-port.ts` |
| **I** · Segregación de interfaces | Aplicación y dominio | Cada contrato expondrá únicamente las operaciones necesarias. `ReservaRepository` administrará la persistencia de reservas, `InventarioPort` permitirá comprometer o liberar stock y `NotificacionPort` permitirá registrar eventos para su publicación posterior. | `reserva-repository.ts`, `inventario-port.ts`, `notificacion-port.ts`, `unidad-trabajo.ts` |
| **D** · Inversión de dependencias | Aplicación e infraestructura | Los casos de uso dependerán de interfaces y recibirán sus dependencias mediante inyección. No conocerán directamente Express, PostgreSQL, Redis ni proveedores externos. | `crear-reserva.use-case.ts`, `reservas.module.ts`, `reserva-repository.ts`, `inventario-port.ts` |

> **Nota:** los nombres de archivos TypeScript representan una estructura propuesta. No significa que estos archivos ya estén implementados.

## Qué pasaría sin SOLID en FarmaFind

| Situación real | Sin SOLID | Con SOLID |
|---|---|---|
| Se cambia el proveedor de notificaciones | Habría que modificar directamente el código del proceso de reservas. | Se reemplaza el adaptador correspondiente sin modificar las reglas de negocio. |
| Se modifica la implementación de PostgreSQL | Las consultas SQL podrían quedar mezcladas con las reglas de reserva. | Se modifica el repositorio correspondiente, manteniendo sus contratos. |
| Se desea probar la creación de una reserva | Las pruebas dependerían directamente de PostgreSQL y de Notificaciones. | Se utilizan repositorios y adaptadores simulados para probar los casos de uso. Las pruebas de transacciones reales se realizan por separado. |
| Cambia una regla de reserva | Sería necesario modificar controladores, rutas y consultas SQL. | La regla se modifica en la entidad `Reserva` o en su caso de uso correspondiente. |
| Se agrega un pago opcional | La lógica de pago podría mezclarse con el proceso principal de reserva. | Se incorpora un módulo de Pagos con contratos y adaptadores propios. |
| Redis deja de funcionar | Las búsquedas podrían fallar por depender directamente de la caché. | El módulo Búsqueda utiliza un contrato de caché y continúa consultando PostgreSQL cuando Redis no está disponible. |
| Se necesita garantizar la consistencia del stock | Las actualizaciones de inventario y reservas podrían ejecutarse independientemente. | Unit of Work permite coordinar las operaciones críticas dentro de una misma transacción PostgreSQL. |
| El servicio de Notificaciones falla | Una reserva podría quedar afectada por el fallo del servicio externo. | Transactional Outbox registra el evento dentro de la transacción y permite publicarlo posteriormente. |

## Relación con Clean Architecture y los patrones

| Principio | Lo cumple en el proyecto |
|---|---|
| Responsabilidad única | La separación por capas: presentación, aplicación, dominio e infraestructura. Cada componente tiene responsabilidades definidas. |
| Abierto/cerrado | El uso de contratos y adaptadores para PostgreSQL, Redis, Notificaciones y futuras integraciones de pagos o mapas. |
| Sustitución de Liskov | Los adaptadores que implementan un mismo puerto pueden reemplazarse respetando el contrato establecido. |
| Segregación de interfaces | Los contratos se dividen según su responsabilidad: `ReservaRepository`, `InventarioPort`, `NotificacionPort`, `BusquedaCachePort` y `UnidadTrabajo`. |
| Inversión de dependencias | Los casos de uso dependen de interfaces, no de Express, PostgreSQL, Redis ni proveedores externos. |

Los principios SOLID se complementarán con los siguientes patrones:

- **Repository:** separa la persistencia de las reglas de negocio.
- **Adapter:** permite conectar PostgreSQL, Redis y servicios externos mediante contratos.
- **Dependency Injection:** permite sustituir implementaciones y facilitar las pruebas.
- **Cache-Aside:** optimiza búsquedas utilizando Redis y PostgreSQL.
- **Unit of Work:** coordina las transacciones de Reservas e Inventario.
- **Transactional Outbox:** conserva eventos de Notificaciones para su publicación posterior.

La aplicación de estos patrones deberá respetar los límites de cada módulo y las dependencias establecidas por Clean Architecture.

## Beneficio

- El backend estará organizado y será más fácil de mantener.
- Las reglas de negocio no dependerán directamente de tecnologías externas.
- Los módulos podrán probarse utilizando implementaciones simuladas.
- Redis permitirá optimizar las búsquedas sin reemplazar PostgreSQL.
- Unit of Work permitirá mantener la consistencia entre reservas e inventario.
- Transactional Outbox permitirá conservar eventos cuando Notificaciones falle.
- Se facilitará la incorporación futura de pagos e inventarios externos.
- El equipo podrá seguir criterios uniformes de desarrollo en todos los módulos.

