# Escenarios de atributos de calidad del sistema FarmaFind

## EQ-01. Rendimiento: búsqueda de medicamentos

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Rendimiento |
| Fuente | Cliente |
| Estímulo | El cliente realiza una búsqueda de medicamentos por nombre comercial o principio activo, aplicando opcionalmente filtros de disponibilidad, precio o cercanía. |
| Condición | El sistema funciona bajo una carga de prueba definida, con un conjunto de datos representativo y Redis disponible como contenedor obligatorio del MVP. El módulo Búsqueda utiliza Cache-Aside para consultar primero Redis y recurrir a PostgreSQL cuando no existe una entrada válida. |
| Respuesta esperada | El sistema muestra medicamentos y farmacias con disponibilidad referencial, fecha de última revisión del inventario por la farmacia, precio referencial y ubicación sin errores inesperados; la existencia física quedará sujeta a aceptación de la farmacia. Las consultas repetidas pueden resolverse mediante Redis, reduciendo accesos innecesarios a PostgreSQL. Los resultados almacenados en caché deben respetar las políticas de TTL e invalidación. |
| Medida | El 95 % de las búsquedas debe responder en 2 segundos o menos (p95 ≤ 2 s), con errores técnicos menores al 1 %. Se medirán también p99, throughput, latencia de Redis, porcentaje de aciertos y fallos de caché (hit rate y miss rate), consultas a PostgreSQL, CPU, memoria y conexiones. Se comparará el rendimiento con caché habilitada y deshabilitada bajo condiciones equivalentes. |
| Verificación | Ejecutar pruebas de carga sobre búsquedas repetidas y variadas, utilizando el mismo conjunto de datos, recursos y carga para las comparaciones. Registrar p95, p99, throughput, errores, hit rate, miss rate y cantidad de consultas a PostgreSQL. Comprobar el comportamiento de Cache-Aside ante aciertos y fallos de caché. |
| Estado | Pendiente de implementación y validación mediante pruebas de rendimiento. |

## EQ-02. Disponibilidad: continuidad del servicio

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Disponibilidad |
| Fuente | Usuario del sistema |
| Estímulo | Una instancia del núcleo FarmaFind deja de funcionar o Redis presenta una interrupción temporal. |
| Condición | El sistema cuenta con dos instancias del núcleo en ejecución, PostgreSQL disponible y Redis configurado como caché. Se realizan pruebas controladas de interrupción de una instancia y de Redis por separado, manteniendo disponibles los demás componentes necesarios. |
| Respuesta esperada | Si una instancia de la API falla, la instancia sana debe continuar atendiendo solicitudes. Si Redis falla, el módulo Búsqueda debe utilizar PostgreSQL como alternativa, sin impedir consultas ni operaciones de reserva. La información transaccional no debe depender de Redis. |
| Medida | Ante la caída de una instancia del núcleo, las solicitudes deberán volver a ser atendidas por la instancia sana en 30 segundos o menos. Durante la interrupción de Redis, las búsquedas deberán continuar mediante PostgreSQL, sin fallos funcionales atribuibles exclusivamente a la indisponibilidad de caché. Se registrarán tiempos de recuperación, errores y degradación del rendimiento. |
| Verificación | Ejecutar dos pruebas separadas: detener una instancia de la API y comprobar la continuidad mediante la otra; detener Redis y comprobar que las búsquedas recurren a PostgreSQL. Verificar que las reservas continúan validando stock mediante operaciones transaccionales. Registrar tiempos, respuestas y errores. |
| Estado | Pendiente de pruebas de disponibilidad y tolerancia a fallos. PostgreSQL único permanece declarado como punto potencial de fallo. |

## EQ-03. Escalabilidad: aumento de carga

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Escalabilidad |
| Fuente | Crecimiento de usuarios |
| Estímulo | Aumenta progresivamente la cantidad de usuarios que realizan búsquedas y reservas en FarmaFind. |
| Condición | Se comparan una y dos instancias de la API utilizando el mismo conjunto de datos y recursos equivalentes de PostgreSQL y Redis. Las pruebas incluirán consultas repetidas y variadas para evaluar el efecto de Cache-Aside, así como operaciones de reserva sobre inventario compartido. |
| Respuesta esperada | El sistema deberá admitir un incremento de carga sin superar inmediatamente los límites de rendimiento definidos en EQ-01. Redis deberá contribuir a reducir consultas repetidas a PostgreSQL, mientras que las operaciones de reserva deberán mantener consistencia independientemente del número de instancias de la API. |
| Medida | Comparar el throughput máximo alcanzado antes de superar p95 ≤ 2 s y errores técnicos menores al 1 % en las búsquedas. Registrar p95, p99, CPU, memoria, conexiones a PostgreSQL, hit rate, miss rate, consumo de recursos Redis y cuellos de botella. Medir por separado el comportamiento de las reservas concurrentes. |
| Verificación | Ejecutar pruebas con carga creciente sobre una y dos instancias. Mantener un conjunto de datos y recursos comparables. Comparar resultados con Redis habilitado y deshabilitado para analizar su contribución al rendimiento. Documentar los límites encontrados y cualquier mejora que no pueda demostrarse. |
| Estado | Pendiente de validación mediante pruebas de carga y escalabilidad. No se presupone una mejora cuantitativa sin mediciones. |

## EQ-04. Seguridad: acceso protegido

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Seguridad |
| Fuente | Usuario autenticado o no autorizado |
| Estímulo | Un usuario intenta acceder a reservas ajenas, modificar inventario de otra farmacia, aceptar una reserva como personal sin delegación expresa o acceder directamente a componentes de infraestructura sin autorización. |
| Condición | FarmaFind dispone de usuarios con roles y permisos definidos. Las operaciones protegidas requieren autenticación y autorización por recurso. PostgreSQL y Redis se encuentran restringidos a componentes autorizados del backend. |
| Respuesta esperada | El sistema rechaza los accesos no autorizados a reservas y funciones administrativas. La identidad del cliente utilizada en una reserva proviene del contexto autenticado. Redis no es accesible directamente desde el navegador y no almacena contraseñas, tokens ni información privada innecesaria en la caché de búsquedas públicas. |
| Medida | El 100 % de los intentos no autorizados incluidos en las pruebas definidas deberá ser rechazado. Ninguna prueba deberá demostrar exposición de contraseñas, tokens, credenciales de PostgreSQL o Redis en respuestas o registros técnicos. Los accesos directos no autorizados a Redis deberán ser rechazados. |
| Verificación | Probar accesos a reservas ajenas, operaciones sobre otras farmacias, manipulación de identificadores enviados por el navegador y accesos no autorizados a Redis. Probar aceptación por dueño o administrador y por personal con y sin delegación expresa, siempre tras comprobación física. Revisar autenticación, autorización, red y registros técnicos. |
| Estado | Pendiente de implementación y pruebas de seguridad. |

## EQ-05. Consistencia: reservas concurrentes

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Consistencia |
| Fuente | Clientes concurrentes |
| Estímulo | Varios clientes intentan reservar simultáneamente unidades de un mismo medicamento en una farmacia. |
| Condición | Existen 10 unidades disponibles y se realizan 100 intentos simultáneos, válidos e independientes, solicitando una unidad cada uno. PostgreSQL es la fuente oficial de inventario y reservas. Redis podrá contener resultados referenciales de búsqueda, pero no participará en la confirmación del stock. |
| Respuesta esperada | El sistema deberá aceptar únicamente las reservas correspondientes al stock disponible. El compromiso de stock, el registro de la reserva con estado PENDIENTE y el evento Outbox deberán ejecutarse dentro de una misma transacción PostgreSQL mediante Unit of Work. Las operaciones fallidas deberán revertirse. |
| Medida | Deberán crearse exactamente 10 reservas distintas en PENDIENTE, sin stock negativo, duplicación de reservas exitosas ni compromiso superior a las existencias. El inventario deberá quedar con 10 unidades comprometidas y 0 unidades disponibles, suponiendo que no existan otras operaciones concurrentes de inventario ni vencimientos durante la prueba. |
| Verificación | Ejecutar 100 solicitudes concurrentes válidas e independientes sobre el mismo registro de inventario. Verificar las respuestas, la cantidad de reservas persistidas, el stock físico, el stock comprometido y la cantidad de eventos Outbox. Probar importaciones y ajustes concurrentes sin sobrescribir compromisos ni guardar stock físico inferior al comprometido; exigir conciliación. Simular falta física al aceptar PENDIENTE: cancelar con STOCK_FISICO_INSUFICIENTE y comprobar liberación única, incidencia, movimiento y alerta Outbox transaccionales. Repetir cancelación, expiración y entrega para verificar liberación o consumo único. Verificar los plazos configurables separados: 20 minutos iniciales de respuesta y, desde la aceptación, 24 horas propuestas de recogida, sujetas a validación. |
| Estado | Pendiente de implementación y validación mediante pruebas de concurrencia y transacciones PostgreSQL. |

## EQ-06. Mantenibilidad: cambio de adaptador

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Mantenibilidad |
| Fuente | Equipo de desarrollo |
| Estímulo | Se necesita reemplazar el adaptador Redis del módulo Búsqueda o sustituir el adaptador de integración con Notificaciones por una implementación simulada. |
| Condición | Los módulos utilizan Clean Architecture, contratos internos e inyección de dependencias. Búsqueda depende de `BusquedaCachePort` y Reservas utiliza contratos para registrar eventos de notificación. Las implementaciones concretas se encuentran en infraestructura. |
| Respuesta esperada | El adaptador Redis podrá sustituirse por uno simulado o por una implementación sin caché sin modificar las entidades de dominio ni los contratos de los casos de uso. De manera equivalente, la integración con Notificaciones podrá sustituirse por una implementación simulada sin alterar las reglas de Reservas. |
| Medida | La sustitución de cada adaptador deberá realizarse sin modificar el dominio ni los casos de uso relacionados. Las pruebas existentes deberán continuar ejecutándose correctamente. Se registrarán los archivos modificados y cualquier cambio necesario en composición o configuración. |
| Verificación | Sustituir `RedisBusquedaCacheAdapter` por un adaptador simulado y ejecutar las pruebas de Búsqueda. Repetir el ejercicio con el adaptador de registro de notificaciones. Comprobar que las modificaciones se limitan a infraestructura, composición y configuración, sin cambios en las reglas de negocio. |
| Estado | Pendiente de implementación y validación mediante pruebas de mantenibilidad. |

## EQ-07. Interoperabilidad: contrato con notificaciones

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Interoperabilidad |
| Fuente | Núcleo FarmaFind |
| Estímulo | El publicador Outbox envía al servicio independiente de Notificaciones un evento generado por una reserva, una alerta de corrección o un recordatorio de inventario antiguo. El mismo evento puede enviarse nuevamente debido a un reintento. |
| Condición | Existe un contrato de eventos definido entre el núcleo y el servicio de Notificaciones. La comunicación utiliza HTTPS/REST interno autenticado. Cada evento contiene un identificador único, tipo de evento, referencia a la reserva o al inventario de la farmacia, según el tipo, y datos necesarios para su procesamiento. El servicio consumidor dispone de persistencia para registrar eventos recibidos. |
| Respuesta esperada | El servicio de Notificaciones valida el evento, registra un único trabajo lógico por identificador y reconoce correctamente los reintentos del mismo evento. El publicador podrá marcar el evento Outbox como publicado después de recibir una confirmación válida. Redis no participa en este proceso. |
| Medida | Al enviar repetidamente un evento con el mismo identificador, deberá existir un único trabajo lógico persistido. Los eventos válidos deberán cumplir el contrato definido y las solicitudes no autorizadas o inválidas deberán rechazarse. La deduplicación de trabajos no se considerará una garantía absoluta de entrega única por parte de proveedores externos. |
| Verificación | Validar el contrato de eventos y la autenticación entre servicios. Enviar el mismo evento varias veces y comprobar que se registre un solo trabajo lógico. Simular una interrupción después de que el consumidor registre el evento y antes de que el publicador reciba la confirmación, y comprobar la deduplicación durante el reintento. Importar archivos .xlsx/.csv con columnas distintas, mapear nombre, concentración, presentación, precio referencial y stock físico, previsualizar y validar antes de confirmar; comprobar asociación al catálogo general y atomicidad por archivo en el MVP: si una fila es inválida o no está conciliada, no se guardará ninguna. Verificar el reporte de errores y que, tras corregir el archivo y reintentar, se guarde completo en una única transacción. |
| Estado | Pendiente de definición final del contrato y pruebas de interoperabilidad e idempotencia. |

## EQ-08. Resiliencia: falla de notificaciones y caché

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Resiliencia |
| Fuente | Falla temporal del servicio de Notificaciones o de Redis |
| Estímulo | El servicio independiente de Notificaciones se interrumpe durante 5 minutos o Redis deja temporalmente de responder solicitudes. |
| Condición | El núcleo FarmaFind y PostgreSQL permanecen disponibles. Las reservas utilizan Transactional Outbox para almacenar eventos pendientes. El módulo Búsqueda utiliza Cache-Aside con Redis y dispone de una ruta alternativa de consulta a PostgreSQL. Las pruebas de ambas interrupciones se ejecutan por separado para identificar sus efectos. |
| Respuesta esperada | Durante la interrupción de Notificaciones, las reservas deberán continuar y los eventos pendientes permanecerán registrados en PostgreSQL. Tras recuperar el servicio, los eventos deberán procesarse mediante reintentos y deduplicación. Durante la interrupción de Redis, las búsquedas deberán continuar consultando PostgreSQL, y las reservas no deberán verse afectadas funcionalmente por la caída de caché. |
| Medida | Después de interrumpir Notificaciones durante 5 minutos, todos los eventos correspondientes a transacciones confirmadas deberán permanecer registrados y ser procesables tras la recuperación, sin pérdida de trabajos lógicos. Se medirá el retraso de procesamiento. Ante el fallo de Redis, las búsquedas deberán continuar mediante PostgreSQL y se medirán errores, latencia y tiempo de utilización del mecanismo alternativo. |
| Verificación | Detener Notificaciones durante 5 minutos, generar reservas y verificar los eventos pendientes en Outbox. Restaurar el servicio y comprobar su procesamiento, los reintentos y la ausencia de trabajos lógicos duplicados. En una prueba independiente, detener Redis, ejecutar búsquedas, verificar el acceso alternativo a PostgreSQL y comprobar la recuperación posterior de la caché. Validar también TTL e invalidación después del commit de importaciones y ajustes. Generar recordatorios por inventario antiguo con umbral y frecuencia configurables; verificar su persistencia Outbox, recuperación y ausencia de cambios automáticos de existencias. |
| Estado | Pendiente de implementación y pruebas de recuperación ante fallos de Notificaciones y Redis. |

## EQ-09. Observabilidad: seguimiento de pruebas

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Observabilidad y pruebas |
| Fuente | Equipo de desarrollo y pruebas |
| Estímulo | Se ejecutan pruebas de búsquedas, reservas concurrentes, uso de Redis, fallos de caché, publicación de eventos y recuperación del servicio de Notificaciones. |
| Condición | El sistema dispone de registros estructurados, identificadores de correlación y mecanismos para recopilar métricas del backend, PostgreSQL, Redis y procesamiento de eventos. Las pruebas utilizan conjuntos de datos y cargas reproducibles. |
| Respuesta esperada | Cada reserva de prueba podrá relacionarse con registros y eventos correspondientes. Importaciones, ajustes e incidencias tendrán trazabilidad en movimientos_inventario con producto, farmacia, cantidad anterior y nueva, motivo, responsable y fecha. Las consultas permitirán identificar si fueron atendidas mediante Redis o PostgreSQL. Los fallos de transacciones, errores de conexión, eventos pendientes y reintentos deberán poder identificarse mediante registros y métricas, sin exponer información sensible. |
| Medida | Registrar p95, p99, throughput, errores técnicos, conflictos de stock, CPU, memoria, conexiones a PostgreSQL, transacciones fallidas y eventos Outbox pendientes. Para Redis, medir hit rate, miss rate, latencia, uso de memoria, errores de conexión, invalidaciones y utilización del mecanismo alternativo. Para Notificaciones, registrar eventos publicados, trabajos pendientes, reintentos, errores y retrasos de procesamiento. |
| Verificación | Ejecutar los escenarios EQ-01 a EQ-08 y revisar sus registros y métricas. Comprobar la correlación entre solicitudes, reservas y eventos. Comparar búsquedas con caché habilitada y deshabilitada. Verificar que los fallos de Redis y Notificaciones sean observables y que las evidencias obtenidas permitan evaluar los criterios de aceptación de cada escenario. |
| Estado | Pendiente de configuración de observabilidad, implementación y validación mediante pruebas. |