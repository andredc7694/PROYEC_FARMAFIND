
# Nivel 2 · Diagrama de contenedores

> **Proyecto:** FarmaFind  
> **Modelo:** C4 · Nivel 2 de 4 · **Versión:** 1.2  
> **Estado:** Diseño arquitectónico propuesto, pendiente de implementación

---

## 1. Objetivo

Abrir la caja «FarmaFind» del nivel 1 y mostrar los **contenedores** que forman el sistema: las aplicaciones, servicios y almacenes de datos que se ejecutarán de manera independiente, la tecnología propuesta para cada uno y cómo se comunicarán.

Este nivel permite identificar las responsabilidades principales de cada contenedor, sus límites de ejecución y sus relaciones, manteniendo coherencia con el monolito modular y Clean Architecture adoptados para FarmaFind.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿De qué piezas ejecutables se compone el sistema y con qué tecnología? |
| Audiencia | Arquitectos, desarrolladores, operaciones y equipo del proyecto |
| Qué es un contenedor | Una aplicación, servicio o almacén de datos que se ejecuta o administra como una unidad independiente |
| Nivel anterior | `Nivel1-DiagramadeContextodelSistema.md` |
| Siguiente nivel | `Nivel3-Diagrama-de-Componentes.md` |
| Alcance | Contenedores obligatorios del MVP y extensiones futuras |

---

## 2. Diagrama

![Diagrama de contenedores de FarmaFind](../../img/nivel2-contenedores-farmafind.png)

> **Nota de actualización:** la imagen deberá representar los cinco contenedores obligatorios de esta versión 1.2: Aplicación web, API FarmaFind, PostgreSQL, Redis y Servicio de Notificaciones. También quedan pendientes la importación de Excel en web/API, el historial de inventario y los avisos de corrección y actualización. Se conservará el PNG actual hasta una revisión visual posterior.

### Descripción general

FarmaFind utilizará una arquitectura cliente-servidor con un backend organizado como monolito modular.

La aplicación web se comunicará con la API FarmaFind mediante HTTPS y mensajes JSON.

El backend concentrará las reglas de negocio relacionadas con usuarios, farmacias, medicamentos, inventario, búsqueda y reservas.

PostgreSQL será el sistema de persistencia principal y proporcionará las transacciones necesarias para mantener la consistencia del inventario y las reservas.

Redis será el contenedor de caché obligatorio del MVP y permitirá almacenar temporalmente resultados de búsquedas frecuentes.

El servicio de Notificaciones se ejecutará como un proceso independiente y procesará eventos generados por el núcleo FarmaFind.

Para garantizar la persistencia de eventos de notificación ante fallos temporales, se utilizará el patrón Transactional Outbox.

### Límites del sistema

Formarán parte del MVP:

- Aplicación web React.
- API FarmaFind con Node.js, Express y TypeScript.
- Base de datos PostgreSQL.
- Caché Redis.
- Servicio independiente de Notificaciones.

Se consideran integraciones externas o extensiones futuras:

- Pasarela de pago.
- Proveedor de mapas, cuya selección está pendiente.
- Sistemas externos de inventario de farmacias.

Redis no será utilizado como fuente oficial de inventario ni como mecanismo de confirmación de reservas.

---

## 3. Contenedores

| Contenedor | Tecnología | Responsabilidad | Datos que maneja |
|---|---|---|---|
| **Aplicación web** | SPA · React + Vite + JavaScript/JSX | Interfaz para clientes, personal de farmacia, administradores de farmacia y administrador de plataforma. | Estado temporal de pantalla, formularios, filtros y sesión del usuario. |
| **API FarmaFind** | Node.js + Express + TypeScript | Expone la API REST, aplica las reglas de negocio y coordina los módulos internos del monolito modular. | Procesa solicitudes y utiliza PostgreSQL, Redis e integraciones externas. |
| **Servicio de Notificaciones** | Node.js + worker/API interna | Procesa eventos de reserva, administra avisos, reintentos y deduplicación. | Identificadores de eventos, trabajos de notificación y resultados de envío. |
| **Base de datos** | PostgreSQL | Almacena la información persistente y permite operaciones transaccionales. | Usuarios, farmacias, medicamentos, inventario, reservas, roles, auditoría básica y eventos Outbox. |
| **Caché** | Redis | Almacena temporalmente resultados de búsqueda de medicamentos y farmacias para reducir consultas repetidas a PostgreSQL. | Resultados de búsqueda, filtros y referencias de catálogo con tiempo de expiración. |

### 3.1 Aplicación web

La aplicación web será el punto de interacción de los usuarios con FarmaFind.

Permitirá:

- Buscar medicamentos por nombre comercial o principio activo.
- Consultar farmacias, precios y disponibilidad referenciales con fecha de última revisión del inventario; la existencia física se comprobará al aceptar la reserva.
- Filtrar resultados por precio, disponibilidad o cercanía.
- Crear y consultar reservas.
- Gestionar reservas desde perfiles autorizados.
- Administrar precios e inventario propios con registro manual e importación obligatoria .xlsx/.csv: plantilla propuesta de nombre, concentración, presentación, precio referencial y stock físico; vista previa, validación y mapeo al catálogo general antes de confirmar.
- Acceder a funciones administrativas según el rol.

Las búsquedas públicas podrán realizarse sin iniciar sesión.

Las operaciones protegidas requerirán autenticación y autorización.

### 3.2 API FarmaFind

La API será el contenedor principal del backend.

Utilizará un monolito modular con Clean Architecture dentro de cada módulo.

Los módulos previstos son:

- Usuarios.
- Farmacias.
- Medicamentos.
- Búsqueda.
- Inventario.
- Reservas.
- Administración.
- Integración con Notificaciones.

El módulo Pagos se mantendrá como extensión futura.

El módulo Búsqueda utilizará Redis mediante un adaptador de caché, manteniendo PostgreSQL como fuente principal de los datos.

Las operaciones críticas de Reservas e Inventario utilizarán PostgreSQL y transacciones compartidas.

### 3.3 Servicio de Notificaciones

El servicio de Notificaciones se ejecutará independientemente del núcleo FarmaFind.

Será responsable de:

- Recibir eventos del ciclo de reservas, alertas de corrección y recordatorios propuestos por inventario antiguo; estos últimos tendrán umbral y frecuencia configurables y no modificarán existencias.
- Registrar trabajos de notificación.
- Procesar avisos.
- Administrar estados de procesamiento.
- Aplicar reintentos controlados.
- Evitar trabajos duplicados mediante identificadores únicos.
- Comunicarse con proveedores externos de mensajes.

La creación de reservas no dependerá de la disponibilidad inmediata de este servicio.

El núcleo almacenará eventos pendientes mediante Transactional Outbox.

### 3.4 Base de datos PostgreSQL

PostgreSQL será la fuente oficial de información del sistema.

Almacenará usuarios, farmacias, catálogo general, inventario por farmacia, movimientos_inventario, reservas y eventos pendientes. El historial incluirá producto, farmacia, cantidad anterior y nueva, motivo, responsable y fecha. Ajustes e importaciones no sobrescribirán compromisos ni guardarán stock físico inferior al comprometido; exigirán conciliación y revalidación de concurrencia antes del commit. Para el MVP, la importación será atómica por archivo: si una fila es inválida o no está conciliada, no se guardará ninguna; se presentará un reporte de errores para corregir y reintentar.

La creación de una reserva deberá confirmar conjuntamente:

1. El compromiso atómico del stock solicitado.
2. La creación de la reserva con estado PENDIENTE.
3. El registro del evento de reserva creada en Outbox.

Las operaciones deberán ejecutarse en una misma transacción mediante Unit of Work. PENDIENTE tendrá 20 minutos configurables de respuesta. Al aceptar tras comprobación física, se asignará a CONFIRMADA un plazo de recogida configurable de 24 horas propuestas, sujeto a validación. Ante falta física se cancelará PENDIENTE con STOCK_FISICO_INSUFICIENTE, guardando liberación única, incidencia, movimiento y alerta Outbox transaccionalmente.

Redis no participará en estas transacciones.

### 3.5 Caché Redis

Redis será un contenedor obligatorio del MVP dedicado a mejorar el rendimiento de las búsquedas.

Se utilizará principalmente para:

- Almacenar resultados de consultas frecuentes.
- Evitar consultas repetitivas a PostgreSQL.
- Reducir los tiempos de respuesta del módulo Búsqueda.
- Apoyar las pruebas del atributo de calidad AC01 — Rendimiento.

#### Estrategia de caché

Se utilizará el patrón **Cache-Aside**:

1. El módulo Búsqueda recibe una consulta.
2. Consulta si existe un resultado válido en Redis.
3. Si existe, devuelve el resultado almacenado.
4. Si no existe, consulta PostgreSQL.
5. Almacena temporalmente el resultado en Redis.
6. Devuelve la información al cliente.

La integración con Redis se encapsulará mediante un adaptador, permitiendo desacoplar el módulo Búsqueda de la tecnología concreta.

#### Expiración e invalidación

Los resultados almacenados tendrán un tiempo de vida limitado, conocido como TTL.

Como valor inicial de diseño se propone un TTL configurable de 60 segundos para consultas que incluyan disponibilidad o precios. Este valor deberá validarse mediante pruebas y podrá ajustarse según las necesidades del sistema.

Cuando se modifiquen datos relevantes de medicamentos, farmacias, precios o inventario, el sistema deberá invalidar o actualizar las entradas de caché relacionadas.

Debido a que una búsqueda puede combinar varios filtros, la estrategia de invalidación deberá contemplar las consultas afectadas. Si no es posible identificarlas individualmente, podrá utilizarse una estrategia de versionado de claves o invalidación por grupos.

La expiración por TTL actuará como mecanismo adicional para limitar la antigüedad de los resultados.

#### Manejo de fallos

Si Redis no está disponible, el módulo Búsqueda deberá consultar PostgreSQL directamente.

La caída de Redis podrá disminuir el rendimiento, pero no deberá impedir las búsquedas ni las operaciones transaccionales de reserva.

La creación de reservas siempre validará y comprometerá stock mediante PostgreSQL, independientemente de lo mostrado previamente en una búsqueda.

---

## 4. Relaciones

| Origen | Destino | Descripción | Protocolo |
|---|---|---|---|
| Cliente, personal de farmacia y administradores | Aplicación web | Utilizan la aplicación desde el navegador. | HTTPS |
| Aplicación web | API FarmaFind | Envía solicitudes de búsqueda, reservas y administración. | HTTPS/JSON |
| API FarmaFind | PostgreSQL | Consulta y modifica información persistente. | SQL sobre TCP |
| API FarmaFind | Redis | Consulta, almacena e invalida resultados temporales de búsqueda. | Protocolo Redis sobre TCP |
| API FarmaFind | Registro Outbox en PostgreSQL | Guarda eventos pendientes dentro de transacciones de negocio. | SQL sobre TCP |
| Publicador Outbox del núcleo | Servicio de Notificaciones | Publica eventos pendientes mediante una comunicación autenticada. | HTTPS/REST |
| Servicio de Notificaciones | PostgreSQL | Registra trabajos y estados de procesamiento según el diseño de persistencia compartida. | SQL sobre TCP |
| Servicio de Notificaciones | Proveedor de mensajes | Solicita el envío de avisos relacionados con reservas. | HTTPS/API del proveedor |
| API FarmaFind | Pasarela de pago futura | Solicitará el procesamiento de pagos opcionales. | HTTPS/REST |
| API FarmaFind | Servicio de mapas | Consultará ubicación o distancia referencial de farmacias. | HTTPS/API del proveedor |
| API FarmaFind | Inventario externo futuro | Permitirá consultar o sincronizar información de stock. | HTTPS/REST |

### 4.1 Comunicación entre Reservas e Inventario

Reservas e Inventario serán módulos internos del mismo backend.

Su comunicación se realizará mediante contratos públicos, sin solicitudes HTTP entre ambos.

La creación de una reserva requerirá:

1. Validar los datos y la identidad del cliente.
2. Iniciar una transacción PostgreSQL.
3. Comprometer stock mediante una operación atómica.
4. Crear la reserva con estado PENDIENTE.
5. Registrar el evento de reserva creada en Outbox.
6. Confirmar la transacción si todas las operaciones fueron exitosas.

Si ocurre un error, se revertirá la transacción.

La caché Redis no se utilizará para confirmar la disponibilidad de una reserva.

### 4.2 Comunicación entre Búsqueda, Redis y PostgreSQL

El módulo Búsqueda aplicará una estrategia Cache-Aside.

Primero consultará Redis y, si no encuentra información válida, consultará PostgreSQL.

Los resultados obtenidos de PostgreSQL podrán almacenarse temporalmente para acelerar búsquedas posteriores.

Las claves de caché deberán considerar los parámetros de búsqueda, filtros, ordenamiento y paginación, evitando devolver resultados correspondientes a consultas diferentes.

Las modificaciones relevantes deberán activar mecanismos de invalidación.

El sistema deberá considerar que los resultados de búsqueda representan disponibilidad referencial, sujeta a validación digital en PostgreSQL al solicitar PENDIENTE y comprobación física por la farmacia antes de CONFIRMADA.

### 4.3 Comunicación con Notificaciones

El núcleo FarmaFind utilizará Transactional Outbox.

El registro de eventos se realizará dentro de la misma transacción que modifica la reserva.

Posteriormente, un publicador enviará los eventos al servicio independiente de Notificaciones.

El servicio consumidor deberá aplicar deduplicación por identificador de evento.

Si se produce una falla, los eventos pendientes permanecerán almacenados para ser procesados posteriormente.

### 4.4 Comunicación con sistemas externos

Los servicios externos se conectarán mediante adaptadores.

La API principal será responsable de proteger las credenciales correspondientes.

Si falla el servicio de mapas, podrá mostrarse una dirección textual sin bloquear la reserva.

Los pagos electrónicos y la integración con inventarios externos seguirán siendo funcionalidades futuras.

---

## 5. Decisiones arquitectónicas reflejadas

| Decisión | Alternativa descartada o postergada | Motivo |
|---|---|---|
| Backend principal como monolito modular | Microservicios para todos los módulos | Reduce la complejidad inicial y facilita las transacciones entre Reservas e Inventario. |
| Servicio de Notificaciones separado | Procesar avisos externos directamente durante las reservas | Separa responsabilidades y permite reintentos independientes. |
| React como tecnología propuesta | Mantener varias alternativas sin una referencia definida | Permite establecer una tecnología base para el diseño del frontend. |
| PostgreSQL como base principal | Base NoSQL como almacenamiento transaccional principal | Proporciona relaciones, restricciones y transacciones para inventario y reservas. |
| Redis como caché obligatoria del MVP | Consultar siempre PostgreSQL | Permite optimizar búsquedas frecuentes y evaluar el atributo de rendimiento. |
| Cache-Aside para búsquedas | Acoplar directamente la lógica de búsqueda a Redis | Permite utilizar caché sin convertirla en la fuente oficial de datos. |
| TTL e invalidación de resultados | Mantener indefinidamente resultados almacenados | Reduce la exposición de información desactualizada. |
| Funcionamiento degradado sin Redis | Hacer depender todas las búsquedas de la caché | Mejora la tolerancia a fallos del sistema. |
| Transactional Outbox | Enviar notificaciones directamente dentro de la transacción | Permite conservar eventos pendientes ante fallos externos. |
| HTTPS/REST interno para Notificaciones | Mantener sin definir el mecanismo de comunicación | Establece un contrato verificable con el servicio separado. |
| Pago electrónico fuera del MVP | Hacer obligatoria la pasarela de pago | Permite priorizar las funciones de búsqueda y reserva. |
| Integraciones externas mediante adaptadores | Acoplamiento directo con proveedores externos | Facilita el reemplazo de proveedores y las pruebas. |

### 5.1 Justificación de Redis

La búsqueda de medicamentos es una de las funcionalidades centrales de FarmaFind.

Varias consultas pueden solicitar información similar sobre medicamentos, farmacias o disponibilidad.

Redis permitirá almacenar temporalmente los resultados de estas consultas para reducir el trabajo repetitivo de PostgreSQL.

Su incorporación será evaluada mediante pruebas comparativas de rendimiento con caché habilitada y deshabilitada, utilizando el mismo conjunto de datos y una carga reproducible.

No se presupone una mejora cuantitativa hasta obtener mediciones.

### 5.2 Justificación del monolito modular

El monolito modular permitirá conservar una separación lógica de responsabilidades dentro de una misma aplicación.

Las operaciones críticas de Reservas e Inventario podrán compartir un contexto transaccional.

Redis se utilizará como un contenedor adicional de infraestructura sin alterar los límites internos de los módulos.

### 5.3 Justificación de Transactional Outbox

La creación de reservas no deberá depender de la disponibilidad inmediata de Notificaciones.

La persistencia de eventos dentro de PostgreSQL permitirá mantener los trabajos pendientes aunque falle temporalmente el servicio consumidor.

### 5.4 Decisiones pendientes de validación

| Tema | Consideración |
|---|---|
| TTL de Redis | Valor inicial propuesto de 60 segundos, sujeto a pruebas. |
| Política de invalidación | Definir mecanismos concretos según consultas y eventos de actualización. |
| Tamaño de caché | Definir límites de memoria y política de expulsión. |
| Proveedor de mapas | Pendiente de selección. |
| Canal de notificaciones | Pendiente de selección. |
| Política de expiración de reservas | Validar plazos configurables: respuesta PENDIENTE de 20 minutos iniciales y recogida CONFIRMADA de 24 horas propuestas desde la aceptación. Definir periodicidad configurable del proceso de expiración. |
| Importación e inventario antiguo | Validar columnas reales; la importación será atómica por archivo con reporte de errores; definir umbral y frecuencia configurables de recordatorios. |
| Pasarela de pago | Funcionalidad futura. |
| Integración de inventario externo | Funcionalidad futura. |

---

## 6. Seguridad entre contenedores

| Comunicación | Medida |
|---|---|
| Navegador → Aplicación web y API | Tráfico cifrado mediante HTTPS. |
| Aplicación web → API FarmaFind | Autenticación en operaciones protegidas y autorización por recurso. |
| API FarmaFind → PostgreSQL | Credenciales protegidas y acceso restringido desde la red correspondiente. |
| API FarmaFind → Redis | Acceso restringido, autenticación y configuración segura de red. |
| API FarmaFind → Registro Outbox | Escritura dentro de la transacción de negocio. |
| Publicador Outbox → Servicio de Notificaciones | Comunicación HTTPS autenticada entre servicios. |
| Servicio de Notificaciones → PostgreSQL | Credenciales de acceso con privilegios limitados a sus responsabilidades. |
| Servicio de Notificaciones → Proveedores externos | Protección de credenciales y manejo seguro de respuestas. |
| API FarmaFind → Sistemas externos | Credenciales almacenadas en configuración segura del servidor. |
| Operaciones administrativas | Validación de identidad, rol y permisos sobre cada recurso. |

### 6.1 Seguridad de Redis

Redis será accesible únicamente desde los componentes autorizados del backend.

No deberá exponerse directamente a los navegadores ni a redes públicas sin controles de acceso.

Las credenciales deberán mantenerse fuera del código fuente.

Se limitará la información almacenada en caché a los datos necesarios para las consultas autorizadas.

No deberán almacenarse contraseñas, tokens de autenticación ni información privada de reservas en entradas de caché de búsqueda pública.

La configuración deberá considerar límites de memoria, políticas de expiración y mecanismos de monitoreo.

### 6.2 Seguridad de las reservas

Las operaciones de reservas requerirán autenticación.

La identidad del cliente provendrá del contexto autenticado y no únicamente del cuerpo de la solicitud.

Se verificarán permisos por farmacia para consultar y entregar. La aceptación de PENDIENTE vigente corresponderá al dueño o administrador o al personal con delegación expresa, tras comprobar existencias físicas. FarmaFind supervisará sin actualizar el inventario de todos los establecimientos.

Las transacciones PostgreSQL deberán impedir comprometer más unidades que las disponibles.

### 6.3 Disponibilidad y tolerancia a fallos

Si Redis deja de estar disponible, las búsquedas utilizarán PostgreSQL como alternativa.

La falla de Redis no deberá afectar la consistencia de las reservas.

Si Notificaciones falla, los eventos pendientes permanecerán registrados en PostgreSQL.

Se contempla ejecutar varias instancias de la API para realizar pruebas de disponibilidad.

PostgreSQL seguirá siendo inicialmente una base de datos única y un punto potencial de fallo del sistema.

### 6.4 Observabilidad

El sistema deberá permitir medir:

- Latencia de búsquedas.
- Porcentaje de aciertos y fallos de caché.
- Cantidad de consultas a PostgreSQL.
- Uso de memoria de Redis.
- Errores de conexión con Redis.
- Tiempo de respuesta con caché habilitada y deshabilitada.
- Errores de reservas.
- Eventos pendientes y retrasos de Notificaciones.

Las métricas permitirán determinar si Redis cumple su objetivo de mejorar el rendimiento sin comprometer la consistencia.

---

> **Nota de coherencia arquitectónica:** Redis forma parte obligatoria del MVP como contenedor de caché para el módulo Búsqueda. PostgreSQL continúa siendo la fuente oficial de datos y el único almacenamiento utilizado para confirmar inventario y reservas. El flujo transaccional y Transactional Outbox mantienen coherencia con el C4 Nivel 4, versión 1.1.

> **Estado del documento:** los contenedores, contratos y mecanismos descritos representan decisiones de arquitectura propuestas. Su implementación y validación se realizarán en las siguientes etapas.
