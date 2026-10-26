
# Patrones de diseño

## Objetivo

Documentar los patrones de diseño y arquitectura seleccionados para el backend de FarmaFind, indicando dónde se utilizarán, qué problemas resolverán y con qué tecnologías se implementarán, para que el equipo de desarrollo mantenga criterios uniformes durante la construcción del sistema.

El diseño utilizará un **monolito modular con Clean Architecture**, PostgreSQL como base de datos transaccional, Redis como caché obligatoria del MVP y un servicio independiente de Notificaciones.

Los patrones seleccionados permitirán:

- Separar las reglas de negocio de los detalles tecnológicos.
- Mantener responsabilidades claras entre los módulos.
- Garantizar consistencia entre las reservas y el inventario.
- Optimizar las búsquedas mediante Redis.
- Evitar que los fallos de Notificaciones interrumpan las reservas.
- Facilitar las pruebas y el mantenimiento.
- Mantener coherencia con las decisiones arquitectónicas y los diagramas C4.

Los patrones descritos representan **decisiones de diseño propuestas**, pendientes de implementación y validación.

---

## Contexto

- **Proyecto:** FarmaFind.
- **Estilo arquitectónico:** monolito modular con servicio independiente de Notificaciones.
- **Backend propuesto:** Node.js + Express + TypeScript.
- **Frontend propuesto:** React + Vite + JavaScript/JSX.
- **Enfoque interno:** Clean Architecture.
- **Base de datos principal:** PostgreSQL.
- **Caché obligatoria del MVP:** Redis.
- **Estrategia de caché:** Cache-Aside.
- **Comunicación con Notificaciones:** HTTPS/REST interno autenticado.
- **Persistencia de eventos:** Transactional Outbox.
- **Coordinación transaccional:** Unit of Work.
- **Base documental:** decisiones arquitectónicas ADR-01 a ADR-12, modelo C4 y diseño interno de módulos.
- **Estado:** diseño arquitectónico, pendiente de implementación.

### Organización arquitectónica

FarmaFind estará organizado en módulos internos de negocio:

- Usuarios.
- Farmacias.
- Medicamentos.
- Búsqueda.
- Inventario.
- Reservas.
- Administración.
- Integración con Notificaciones.

El módulo Pagos se mantendrá como una extensión futura.

Cada módulo seguirá una separación de responsabilidades basada en presentación, aplicación, dominio e infraestructura.

El dominio no dependerá directamente de Express, PostgreSQL, Redis ni proveedores externos.

Los componentes se comunicarán mediante contratos y casos de uso públicos.

### Responsabilidades de las tecnologías

| Tecnología | Responsabilidad |
|---|---|
| Node.js + Express + TypeScript | Implementar la API REST y coordinar los módulos del backend. |
| PostgreSQL | Almacenar información persistente y ejecutar transacciones críticas. |
| Redis | Almacenar temporalmente resultados de búsquedas de medicamentos y farmacias. |
| Servicio de Notificaciones | Procesar y enviar avisos de manera independiente. |
| React | Proporcionar la interfaz web para clientes y administradores. |

PostgreSQL será la fuente oficial de información.

Redis no se utilizará para confirmar reservas ni modificar inventario.

---

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Se implementa con | Archivos propuestos en el código |
|---|---|---|---|---|
| Adapter | Reservas → Integración con Notificaciones | Evita que las reglas de reserva dependan directamente del mecanismo de publicación de eventos. | Contratos internos y adaptadores de infraestructura. | `notificacion-port.ts`, `postgres-outbox.adapter.ts`, `notificacion-evento.adapter.ts` |
| Adapter | Reservas/Pagos → Pasarela de pago futura | Aísla las particularidades de proveedores externos de pago. | API REST o SDK del proveedor. | `pago-port.ts`, `pasarela-pago.adapter.ts` |
| Adapter | Búsqueda/Farmacias → Mapas | Permite cambiar el proveedor de mapas sin modificar las reglas de negocio. | API REST del proveedor de mapas. | `mapa-port.ts`, `maps.adapter.ts` |
| Adapter | Inventario → Archivos locales .xlsx/.csv | Aísla la lectura de formatos sin suponer columnas uniformes. La aplicación coordinará mapeo, vista previa, validación y asociación al catálogo general. | Puerto interno y adaptador de lectura; importación obligatoria del MVP. | Nombres por definir durante la implementación. |
| Adapter | Búsqueda → Redis | Evita que los casos de uso dependan directamente del cliente Redis. | Redis y contrato de caché. | `busqueda-cache.port.ts`, `redis-busqueda-cache.adapter.ts` |
| Repository | Reservas → PostgreSQL | Separa las reglas de reserva de las consultas SQL. | PostgreSQL y repositorios. | `reserva-repository.ts`, `postgres-reserva.repository.ts` |
| Repository | Medicamentos e Inventario → PostgreSQL | Permite consultar y actualizar medicamentos, precios y existencias sin acoplar el dominio a PostgreSQL. | PostgreSQL mediante repositorios. | `medicamento-repository.ts`, `inventario-repository.ts`, `postgres-inventario.repository.ts` |
| Repository | Búsqueda → PostgreSQL | Abstrae las consultas del catálogo, farmacias y disponibilidad. | PostgreSQL. | `busqueda-repository.ts`, `postgres-busqueda.repository.ts` |
| Dependency Injection | Composición de módulos | Evita que los casos de uso creen directamente sus dependencias concretas. | Constructores y archivos de composición. | `reservas.module.ts`, `crear-reserva.use-case.ts` |
| DTO / Command | Entrada de casos de uso | Organiza y tipa los datos necesarios para ejecutar operaciones. | Interfaces y objetos TypeScript. | `crear-reserva.command.ts`, `actualizar-stock.command.ts` |
| Cache-Aside | Búsqueda → Redis y PostgreSQL | Reduce las consultas repetidas a PostgreSQL sin convertir Redis en almacenamiento principal. | Redis, PostgreSQL, puerto y adaptador de caché. | `buscar-medicamentos.use-case.ts`, `busqueda-cache.port.ts`, `redis-busqueda-cache.adapter.ts` |
| Unit of Work | Reservas e Inventario | Permite ejecutar operaciones relacionadas dentro de una misma transacción PostgreSQL. | Contexto transaccional y unidad de trabajo. | `unidad-trabajo.ts`, `transaccion-contexto.ts`, `postgres-unidad-trabajo.ts` |
| Transactional Outbox | Reservas → Notificaciones | Evita la pérdida de eventos cuando el servicio de Notificaciones está temporalmente fuera de servicio. | PostgreSQL, tabla Outbox y publicador de eventos. | `postgres-outbox.adapter.ts`, `outbox.publisher.ts` |
| Idempotent Consumer | Servicio independiente de Notificaciones | Evita registrar trabajos duplicados cuando un mismo evento se recibe varias veces. | Identificadores únicos de evento y registro persistente de procesamiento. | `notificacion-evento.handler.ts`, `evento-procesado.repository.ts` |

### Patrones obligatorios del MVP

Los siguientes patrones se consideran decisiones necesarias para la arquitectura inicial:

| Patrón | Prioridad | Justificación |
|---|---|---|
| Repository | Obligatorio | Desacoplar las reglas de negocio de PostgreSQL. |
| Adapter | Obligatorio | Desacoplar las integraciones tecnológicas. |
| Dependency Injection | Obligatorio | Facilitar pruebas y sustitución de dependencias. |
| DTO / Command | Obligatorio | Definir contratos de entrada entre presentación y aplicación. |
| Cache-Aside | Obligatorio | Integrar Redis como caché del módulo Búsqueda. |
| Unit of Work | Obligatorio | Garantizar consistencia transaccional entre Reservas e Inventario. |
| Transactional Outbox | Obligatorio | Registrar eventos de reserva de manera fiable. |
| Idempotent Consumer | Obligatorio | Procesar eventos repetidos sin crear trabajos duplicados. |

El uso de Redis será obligatorio dentro de la arquitectura del MVP, pero el sistema deberá contemplar un funcionamiento alternativo mediante PostgreSQL cuando la caché no esté disponible.

El uso de Transactional Outbox no implica necesariamente incorporar un broker de mensajes durante el MVP.

La publicación de eventos se realizará mediante HTTPS/REST interno autenticado, de acuerdo con el modelo C4 Nivel 2.

---

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Adapter | Traduce un contrato interno del sistema a las operaciones de una tecnología externa. Por ejemplo, un adaptador permite que Búsqueda utilice Redis sin depender directamente de su cliente. |
| Repository | Permite que las reglas de negocio soliciten operaciones como `guardar(reserva)` o `buscarPorId(id)` sin ejecutar directamente consultas SQL. |
| Dependency Injection | Los casos de uso reciben sus dependencias mediante constructores o mecanismos de composición, en lugar de crearlas directamente. |
| DTO / Command | Agrupa los datos necesarios para ejecutar un caso de uso, evitando acoplar la aplicación a las solicitudes HTTP. |
| Cache-Aside | Primero consulta Redis. Si no existe información válida, consulta PostgreSQL, almacena temporalmente el resultado y lo devuelve. |
| Unit of Work | Coordina varias operaciones dentro de una misma transacción para que se confirmen juntas o se reviertan ante un error. |
| Transactional Outbox | Registra un evento pendiente dentro de la misma transacción de negocio. Posteriormente, un publicador entrega el evento al servicio correspondiente. |
| Idempotent Consumer | Identifica eventos previamente procesados para evitar registrar trabajos duplicados cuando se recibe nuevamente el mismo evento. |

### Cache-Aside en FarmaFind

El módulo Búsqueda utilizará Redis mediante un puerto y un adaptador.

El flujo propuesto será:

1. El cliente solicita una búsqueda de medicamentos.
2. `BuscarMedicamentosUseCase` construye una clave de caché a partir de los parámetros de búsqueda.
3. Se consulta Redis mediante `BusquedaCachePort`.
4. Si existe una entrada válida, se devuelve el resultado.
5. Si no existe, se consulta PostgreSQL mediante `BusquedaRepository`.
6. El resultado se almacena temporalmente en Redis con un TTL.
7. Se devuelve la información al cliente.

**Reglas de caché:**

- PostgreSQL será la fuente oficial de información.
- Redis almacenará resultados temporales de búsqueda.
- Se propone un TTL inicial configurable de 60 segundos para resultados que incluyan precios o disponibilidad.
- Los cambios relevantes de inventario, medicamentos o farmacias deberán invalidar las entradas afectadas.
- La invalidación se ejecutará después de confirmar los cambios en PostgreSQL.
- Si Redis no está disponible, las búsquedas continuarán consultando PostgreSQL.
- Las reservas no confirmarán stock utilizando datos almacenados en Redis.

La invalidación deberá contemplar que un medicamento puede aparecer en distintas búsquedas, filtros y páginas de resultados.

Para reducir inconsistencias, podrán utilizarse versiones de claves o invalidación por grupos.

Cuando una invalidación falle temporalmente, se deberán aplicar mecanismos de recuperación y mantener el TTL como límite adicional de vigencia.

La estrategia concreta de invalidación deberá definirse y validarse durante la implementación.

### Unit of Work en FarmaFind

El patrón Unit of Work coordinará las operaciones críticas de Reservas e Inventario, incluidos ajustes e importaciones confirmadas con historial movimientos_inventario. Para el MVP, la importación será atómica por archivo: si una fila es inválida o no está conciliada, no se guardará ninguna; se presentará un reporte de errores para corregir y reintentar. Se revalidarán compromisos concurrentes sin sobrescribirlos ni guardar stock físico inferior al comprometido; las discrepancias exigirán conciliación.

La creación de una reserva deberá ejecutar:

1. Inicio de transacción PostgreSQL.
2. Compromiso atómico del stock solicitado.
3. Creación de la reserva con estado PENDIENTE.
4. Persistencia de la reserva.
5. Registro del evento de reserva creada en Outbox.
6. Confirmación de la transacción.

Si alguna operación falla, todos los cambios transaccionales deberán revertirse.

Los repositorios y adaptadores participantes utilizarán un mismo contexto transaccional.

Esto permitirá evitar:

- Stock comprometido sin una reserva registrada.
- Reservas registradas sin stock comprometido.
- Eventos de notificación asociados a reservas que no se confirmaron.
- Reservas concurrentes que superen la disponibilidad digital registrada; la existencia física se comprobará antes de aceptar.

La transacción se mantendrá dentro de PostgreSQL y no incluirá llamadas externas a Redis, Notificaciones o proveedores de pago.

### Transactional Outbox en FarmaFind

Transactional Outbox permitirá mantener la consistencia entre las operaciones de negocio y el registro de eventos pendientes.

Los eventos cubrirán el ciclo de reservas, las alertas por STOCK_FISICO_INSUFICIENTE y los recordatorios propuestos por inventario antiguo. La cancelación por falta física guardará estado, liberación única, incidencia, movimiento y alerta en una misma transacción. Los recordatorios tendrán umbral y frecuencia configurables y no alterarán existencias.

Cuando se cree una reserva:

1. Se inicia una transacción PostgreSQL.
2. Se compromete el stock.
3. Se guarda la reserva.
4. Se registra un evento en la tabla Outbox.
5. Se confirma la transacción.
6. Un publicador obtiene posteriormente los eventos pendientes.
7. El publicador envía el evento al servicio independiente de Notificaciones mediante HTTPS/REST autenticado.
8. El servicio consumidor registra o reconoce el evento mediante su identificador único.
9. El publicador marca el evento como publicado después de recibir una confirmación válida.

Si el servicio de Notificaciones falla, el evento permanecerá pendiente para su posterior publicación.

El mecanismo utilizará reintentos controlados y registro de errores.

El publicador deberá tolerar que un evento se envíe más de una vez, porque una interrupción podría ocurrir después de que el consumidor reciba el evento, pero antes de que el publicador registre la confirmación.

### Idempotent Consumer en Notificaciones

El servicio independiente de Notificaciones utilizará el identificador único de cada evento para evitar registrar múltiples trabajos correspondientes al mismo evento lógico.

Cuando reciba un evento:

1. Validará el contrato y la autenticación del emisor.
2. Consultará si el identificador del evento ya fue registrado.
3. Si es nuevo, registrará el trabajo de forma persistente.
4. Si ya existe, reconocerá la recepción sin crear un trabajo duplicado.
5. Procesará el envío mediante el proveedor correspondiente.
6. Registrará el resultado y aplicará reintentos cuando sea necesario.

La deduplicación de trabajos no garantiza por sí sola que un proveedor externo nunca entregue dos mensajes. Esa garantía dependerá de los mecanismos de idempotencia disponibles en el proveedor y del tratamiento de respuestas inciertas.

### Relación entre los patrones

Los patrones se complementarán de la siguiente manera:

- **Repository** permitirá acceder a PostgreSQL mediante contratos.
- **Adapter** permitirá integrar Redis y servicios externos.
- **Dependency Injection** conectará los contratos con sus implementaciones concretas.
- **Cache-Aside** permitirá optimizar las búsquedas.
- **Unit of Work** garantizará la atomicidad de las operaciones de reserva.
- **Transactional Outbox** conservará los eventos pendientes.
- **Idempotent Consumer** evitará trabajos duplicados de notificación.

Cada patrón responderá a una responsabilidad concreta, evitando incorporarlo únicamente por complejidad técnica o por motivos decorativos.

---

## Beneficio

Los patrones seleccionados proporcionarán los siguientes beneficios para FarmaFind:

- **Mantenibilidad:** las responsabilidades estarán distribuidas entre componentes y capas.
- **Desacoplamiento:** el dominio no dependerá directamente de Express, PostgreSQL, Redis ni proveedores externos.
- **Consistencia:** las reservas y el inventario utilizarán transacciones compartidas.
- **Rendimiento:** Redis permitirá almacenar resultados frecuentes y reducir consultas repetidas.
- **Resiliencia:** los eventos pendientes permanecerán registrados cuando Notificaciones falle.
- **Seguridad:** los contratos y casos de uso facilitarán aplicar autorización y validaciones en los lugares correspondientes.
- **Pruebas:** podrán utilizarse repositorios y adaptadores simulados para probar casos de uso.
- **Escalabilidad:** la separación de responsabilidades permitirá analizar futuros cambios sin rediseñar inmediatamente el sistema.
- **Interoperabilidad:** los contratos permitirán integrar proveedores externos de manera controlada.
- **Trazabilidad:** los patrones estarán relacionados con decisiones arquitectónicas y atributos de calidad.

### Relación con las decisiones arquitectónicas

| Patrón | Decisión arquitectónica relacionada |
|---|---|
| Repository | ADR-03, ADR-04 |
| Adapter | ADR-03, ADR-05 |
| Dependency Injection | ADR-02, ADR-03 |
| DTO / Command | ADR-02, ADR-03 |
| Cache-Aside | ADR-07, ADR-08 |
| Unit of Work | ADR-04, ADR-09 |
| Transactional Outbox | ADR-09, ADR-11 |
| Idempotent Consumer | ADR-11 |

### Relación con los atributos de calidad

| Atributo | Patrones relacionados | Beneficio esperado |
|---|---|---|
| AC01 — Rendimiento | Cache-Aside, Adapter | Reducir consultas repetidas y mejorar tiempos de respuesta. |
| AC02 — Disponibilidad | Adapter, Cache-Aside | Permitir que las búsquedas continúen mediante PostgreSQL si Redis falla. |
| AC03 — Escalabilidad | Cache-Aside, Repository | Reducir carga repetitiva y facilitar mediciones de capacidad. |
| AC04 — Seguridad | DTO / Command, Dependency Injection | Facilitar separación de entradas y aplicación de controles de acceso. |
| AC05 — Consistencia | Unit of Work, Repository | Mantener operaciones transaccionales coherentes. |
| AC06 — Mantenibilidad | Adapter, Repository, Dependency Injection | Facilitar sustitución de implementaciones. |
| AC07 — Interoperabilidad | Adapter, Idempotent Consumer | Permitir comunicación mediante contratos y evitar trabajos duplicados. |
| AC08 — Resiliencia | Transactional Outbox, Idempotent Consumer | Mantener eventos pendientes y recuperarse de fallos temporales. |
| AC09 — Observabilidad y pruebas | Dependency Injection, Repository, Adapter | Facilitar pruebas controladas e instrumentación de componentes. |

La incorporación de estos patrones no garantiza automáticamente el cumplimiento de los atributos de calidad.

El proyecto deberá comprobar sus resultados mediante pruebas de rendimiento, concurrencia, recuperación y mantenimiento.

---

## Patrones previstos para siguientes iteraciones

Los siguientes patrones se consideran alternativas de evolución, no requisitos obligatorios del MVP.

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Observer | Eventos internos de módulos | Cuando se requiera reaccionar a eventos adicionales dentro del monolito sin aumentar el acoplamiento. |
| Factory | Reservas | Si la creación de reservas incorpora múltiples variantes con reglas diferentes. |
| Decorator | Búsqueda | Si se necesita envolver de forma transparente un repositorio de consultas con una capa de caché adicional. |
| Facade | Comunicación entre módulos | Cuando los contratos públicos de varios módulos crezcan y se necesite una interfaz coordinadora más simple. |
| Strategy | Ordenamiento de resultados | Si se incorporan múltiples algoritmos intercambiables de ordenamiento o recomendación. |

### Diferencia entre Cache-Aside y Decorator

Cache-Aside será una estrategia obligatoria del MVP porque define cómo se consultará Redis y cuándo se recurrirá a PostgreSQL.

Decorator es un patrón estructural opcional que podría utilizarse para encapsular transparentemente ese comportamiento.

Por tanto, **utilizar Cache-Aside no obliga a implementar Decorator**.

En el MVP se utilizará un caso de uso coordinado con un puerto de caché y un adaptador Redis.

El uso posterior de Decorator solo se justificará si simplifica el diseño sin agregar complejidad innecesaria.

### Evolución del sistema

La incorporación de patrones adicionales deberá responder a necesidades comprobadas durante el desarrollo.

No se introducirán patrones únicamente para aumentar la cantidad de componentes del sistema.

Las futuras decisiones deberán mantener coherencia con:

- El estilo de monolito modular.
- Clean Architecture.
- Los contratos públicos entre módulos.
- PostgreSQL como fuente oficial de datos.
- Redis como caché del módulo Búsqueda.
- Las transacciones compartidas de Reservas e Inventario.
- Transactional Outbox para la publicación de eventos.
- El servicio independiente de Notificaciones.

---

> **Coherencia arquitectónica:** los patrones descritos corresponden a las decisiones ADR-01 a ADR-12 y a los diagramas C4 Nivel 2 y Nivel 3, versión 1.2. El tratamiento transaccional de reservas y el registro de eventos mantienen correspondencia con el C4 Nivel 4 y el diseño interno de módulos, versión 1.1.

> **Estado del documento:** este archivo presenta patrones seleccionados para el diseño de FarmaFind. Su aplicación práctica deberá implementarse y validarse posteriormente.
