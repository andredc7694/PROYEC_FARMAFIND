# Nivel 1 · Diagrama de contexto del sistema


## 1. Objetivo

Mostrar a **FarmaFind como una sola caja**: quiénes utilizan el sistema y con qué sistemas externos se comunica.

Es la vista de mayor nivel del modelo C4 y permite comprender el alcance general sin mostrar todavía los módulos, servicios internos, tecnologías o bases de datos.

FarmaFind permitirá buscar medicamentos disponibles en farmacias, consultar precios y ubicaciones, realizar reservas temporales y administrar inventarios.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué es FarmaFind, quiénes lo utilizan y con qué sistemas externos se integra? |
| Audiencia | Docente, equipo de desarrollo, usuarios, administración y operación |
| Qué **no** muestra | Tecnologías internas, servidores, bases de datos, caché, módulos ni componentes del backend |
| Siguiente nivel | `Nivel2-DiagramadeContenedores.md` |

---

## 2. Diagrama

![Diagrama de contexto de FarmaFind](../../img/nivel1-contexto-farmafind.png)


## 3. Personas (usuarios)

| Persona | Descripción | Qué hace en el sistema |
|---|---|---|
| **Cliente** | Persona que necesita encontrar medicamentos disponibles en una farmacia. | Busca medicamentos, consulta disponibilidad, precio referencial y ubicación, realiza reservas temporales y consulta su estado. |
| **Personal de farmacia** | Trabajador encargado de atender las reservas de su establecimiento. | Consulta reservas, acepta solicitudes pendientes y confirma la entrega de medicamentos reservados. |
| **Administrador de farmacia** | Responsable de mantener actualizada la información de una farmacia afiliada. | Registra medicamentos, actualiza precios y existencias, y administra el inventario de su farmacia. |
| **Administrador de plataforma** | Responsable de la administración general de FarmaFind. | Gestiona farmacias participantes, administra usuarios y supervisa el funcionamiento del sistema. |

---

## 4. Sistemas

| Sistema | Tipo | Responsabilidad | Quién lo construye |
|---|---|---|---|
| **FarmaFind** | Sistema en alcance | Permite buscar medicamentos y farmacias, consultar disponibilidad, precios y ubicaciones, realizar reservas temporales, administrar inventarios y procesar notificaciones relacionadas con reservas. Incluye el núcleo principal y el servicio independiente de Notificaciones. | Equipo del proyecto |
| **Pasarela de pago** | Sistema externo futuro | Permitirá procesar pagos electrónicos cuando se incorpore esta funcionalidad. No será necesaria para realizar reservas durante el MVP. | Proveedor de pagos |
| **Servicio de mapas** | Sistema externo complementario | Permitirá consultar ubicaciones y distancias referenciales entre los usuarios y las farmacias. | Proveedor de mapas |
| **Inventario externo futuro** | Sistema externo previsto | Permitirá sincronizar existencias con sistemas propios de las farmacias en una futura iteración. | Sistema de cada farmacia o proveedor externo |
| **Proveedor de mensajes** | Sistema externo complementario | Permitirá enviar avisos mediante correo electrónico, SMS u otro canal cuando se defina la integración correspondiente. | Proveedor de mensajería |

### Aclaración del alcance

El servicio de Notificaciones será desarrollado como parte de FarmaFind y funcionará independientemente del núcleo principal.

Por esta razón, no se clasifica como un sistema externo en el diagrama de contexto.

PostgreSQL y Redis también forman parte de la infraestructura interna de FarmaFind.

Redis será obligatorio en el MVP para optimizar búsquedas mediante Cache-Aside, mientras que PostgreSQL será la fuente oficial de información.

Estos componentes se mostrarán en el C4 Nivel 2 porque corresponden a la estructura interna del sistema.

---

## 5. Relaciones

| Origen | Destino | Descripción | Protocolo |
|---|---|---|---|
| Cliente | FarmaFind | Busca medicamentos, consulta precios y disponibilidad, realiza reservas temporales y consulta su estado. | — |
| Personal de farmacia | FarmaFind | Consulta reservas, acepta solicitudes pendientes y confirma entregas. | — |
| Administrador de farmacia | FarmaFind | Registra y actualiza medicamentos, precios y existencias de su farmacia. | — |
| Administrador de plataforma | FarmaFind | Gestiona usuarios, farmacias afiliadas y funciones administrativas. | — |
| FarmaFind | Pasarela de pago | Solicitará el procesamiento de pagos electrónicos cuando se incorpore esta funcionalidad. | HTTPS/REST |
| FarmaFind | Servicio de mapas | Consulta información de ubicación o distancia referencial de farmacias. | HTTPS/REST |
| FarmaFind | Inventario externo futuro | Permitirá consultar o sincronizar existencias cuando se habiliten integraciones externas. | HTTPS/REST |
| FarmaFind | Proveedor de mensajes | Solicita el envío de avisos relacionados con reservas mediante el canal configurado. | API del proveedor |

> En el nivel 1 no se detalla el protocolo entre personas y sistema. Esa información se especifica en el nivel 2 mediante las aplicaciones y sus canales de comunicación.

### Comunicación interna

FarmaFind contará con dos servicios de aplicación principales:

- **Núcleo FarmaFind:** responsable de usuarios, farmacias, medicamentos, búsquedas, inventario, reservas y administración.
- **Notificaciones:** servicio independiente encargado de procesar los avisos relacionados con reservas.

La comunicación entre ambos servicios utilizará eventos registrados mediante Transactional Outbox y publicados por HTTPS/REST interno autenticado.

Esta comunicación es interna y se representará en el C4 Nivel 2, no como una relación entre sistemas externos en este nivel.

---

## 6. Decisiones y supuestos

| # | Decisión o supuesto | Motivo |
|---|---|---|
| 1 | FarmaFind se presenta como un único sistema en el nivel de contexto. | El C4 Nivel 1 explica el alcance general sin detallar sus contenedores internos. |
| 2 | Las integraciones externas se realizarán mediante contratos y adaptadores. | Permite desacoplar FarmaFind de los proveedores externos y facilitar su sustitución. |
| 3 | El pago electrónico no será obligatorio durante el MVP. | El sistema permitirá reservar medicamentos y recogerlos en farmacia sin depender de pagos en línea. |
| 4 | FarmaFind no almacenará datos sensibles de tarjetas. | Los datos de pago serán responsabilidad de una pasarela externa cuando se incorpore esta funcionalidad. |
| 5 | La actualización de inventario será manual durante el MVP. | Permite administrar las existencias sin depender inicialmente de los sistemas propios de cada farmacia. |
| 6 | La integración automática con inventarios externos se realizará en una futura iteración. | Su incorporación dependerá de los sistemas disponibles en las farmacias afiliadas. |
| 7 | Notificaciones pertenecerá a FarmaFind como servicio independiente del núcleo. | Permite separar responsabilidades y cumplir la restricción académica de dos servicios de aplicación. |
| 8 | PostgreSQL será la fuente oficial de información y Redis será obligatorio como caché de búsquedas. | Permite mantener consistencia transaccional y optimizar las consultas frecuentes sin depender de datos temporales para confirmar reservas. |
| 9 | Las reservas y los eventos de notificación se registrarán mediante transacciones PostgreSQL y Transactional Outbox. | Permite mantener la consistencia y conservar eventos pendientes cuando Notificaciones falle temporalmente. |

### Pendientes por definir

| Tema | Detalle |
|---|---|
| Proveedor de mapas | Definir si se utilizará Google Maps, OpenStreetMap u otro servicio de mapas. |
| Canal de notificaciones | Confirmar si los avisos se enviarán mediante correo electrónico, SMS u otro canal. |
| Proveedor de mensajes | Seleccionar el proveedor adecuado para el canal de notificaciones definido. |
| Pago opcional | Definir las condiciones para incorporar pagos electrónicos en una futura iteración. |
| Integración de inventario externo | Confirmar qué farmacias disponen de sistemas propios y qué información podría sincronizarse. |
