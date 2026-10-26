# Nivel 4 · Diagrama de código

> **Proyecto:** FarmaFind  
> **Módulos detallados:** Reservas, Inventario y Pagos  
> **Modelo:** C4 · Nivel 4 de 4 · **Notación:** UML · **Versión:** 1.1

---

## 1. Objetivo

Mostrar **cómo se organizará el código** de uno de los flujos más importantes de FarmaFind: la creación de una reserva temporal de medicamento.

Este nivel presenta el diseño propuesto de clases, interfaces, casos de uso y adaptadores que participarán en el proceso **Crear reserva**, involucrando principalmente los módulos **Reservas** e **Inventario**.

El módulo **Pagos** se mantiene como una extensión futura y opcional, sin formar parte del flujo obligatorio de reservas del MVP.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué clases se proponen dentro de los módulos y cómo colaborarán para cumplir las reglas de negocio? |
| Audiencia | Desarrolladores |
| Notación | UML: diagrama de clases y diagrama de secuencia |
| Por qué estos módulos | Reservas representa el proceso central de FarmaFind; Inventario garantiza la disponibilidad del medicamento y Pagos contempla una funcionalidad complementaria futura |
| Nivel anterior | `Nivel3-Diagrama-de-Componentes.md` |
| Estado | Diseño arquitectónico propuesto, pendiente de implementación |

---

> **Pendiente visual:** los diagramas de clases y secuencia deberán reflejar aceptación física autorizada, plazos separados, importación y conciliación con movimientos_inventario, y alertas Outbox. Los PNG y sus enlaces se conservan; las responsabilidades siguientes constituyen el diseño propuesto.

## 2. Diagrama de clases

### 2.1 Capas de Clean Architecture

| Capa | Contiene | Regla |
|---|---|---|
| **Presentación** | Controladores HTTP | Traduce las solicitudes del navegador a llamadas de casos de uso |
| **Aplicación** | Casos de uso | Coordina el flujo de negocio sin depender de tecnologías concretas |
| **Dominio** | Entidades, objetos de valor, interfaces y reglas | Contiene las reglas principales del negocio y las transiciones válidas de las reservas |
| **Infraestructura** | Repositorios, adaptadores y servicios externos | Implementa los contratos utilizando PostgreSQL y otras tecnologías externas |

> **Regla de dependencia:** las clases externas dependen de las abstracciones internas, pero el dominio no depende de Express, PostgreSQL, Redis ni servicios externos.

---

### 2.2 Clases del módulo Reservas

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `ReservaController` | Controlador | Presentación | Recibe `POST /api/v1/reservas`, valida la solicitud mediante los mecanismos correspondientes y devuelve la respuesta al cliente |
| `CrearReservaUseCase` | Caso de uso | Aplicación | Coordina la creación de la reserva y la operación transaccional con Inventario |
| `ConsultarReservaUseCase` | Caso de uso | Aplicación | Permite consultar el estado de una reserva respetando los permisos del usuario |
| `AceptarReservaUseCase` | Caso de uso | Aplicación | Verificará dueño, administrador o delegación expresa y comprobación física antes de aceptar PENDIENTE vigente; asignará el plazo de recogida o coordinará cancelación por falta física |
| `ConfirmarEntregaUseCase` | Caso de uso | Aplicación | Verifica los permisos, el estado y la vigencia de la reserva antes de confirmar su entrega |
| `ExpirarReservaUseCase` | Caso de uso | Aplicación | Gestiona el vencimiento de reservas y coordina la liberación del stock comprometido |
| `Reserva` | Entidad | Dominio | Representa una reserva temporal y controla sus reglas y transiciones de estado |
| `EstadoReserva` | Enumeración | Dominio | Define los estados PENDIENTE, CONFIRMADA, ENTREGADA, CANCELADA y EXPIRADA |
| `ReservaRepository` | Interfaz | Dominio | Contrato para guardar, consultar y actualizar reservas |
| `PostgresReservaRepository` | Repositorio | Infraestructura | Implementa las operaciones de persistencia utilizando PostgreSQL |

#### Estados del ciclo de vida de una reserva

| Estado | Descripción |
|---|---|
| `PENDIENTE` | La reserva fue creada y el stock quedó comprometido temporalmente; está pendiente de validación por la farmacia |
| `CONFIRMADA` | La farmacia aceptó la reserva y el medicamento está pendiente de recojo |
| `ENTREGADA` | El personal autorizado confirmó la entrega del medicamento al cliente |
| `CANCELADA` | La reserva fue cancelada antes de su entrega y se liberó el stock comprometido |
| `EXPIRADA` | La reserva superó su fecha límite sin entregarse y se liberó el stock comprometido |

#### Transiciones permitidas

| Estado inicial | Estado final | Condición |
|---|---|---|
| PENDIENTE | CONFIRMADA | El dueño, administrador o personal expresamente delegado comprueba físicamente el producto y acepta la reserva vigente |
| PENDIENTE | CANCELADA | Se solicita o ejecuta una cancelación válida |
| PENDIENTE | EXPIRADA | Se agota el plazo de respuesta sin aceptación |
| CONFIRMADA | ENTREGADA | El personal autorizado verifica la reserva vigente y confirma la entrega |
| CONFIRMADA | CANCELADA | Se ejecuta una cancelación válida antes de la entrega |
| CONFIRMADA | EXPIRADA | Se alcanza la fecha de expiración sin entrega |

Los estados `ENTREGADA`, `CANCELADA` y `EXPIRADA` son terminales. Una reserva que alcance alguno de ellos no podrá volver a un estado anterior.

PENDIENTE tendrá un plazo de respuesta configurable de 20 minutos iniciales desde su creación. Al aceptar, su vencimiento se sustituirá por el plazo de recogida de CONFIRMADA desde la aceptación: 24 horas como valor inicial propuesto configurable, sujeto a validación.

Ante falta física se cancelará PENDIENTE con motivo STOCK_FISICO_INSUFICIENTE. Unit of Work guardará cancelación, liberación única del compromiso, incidencia, movimiento y alerta Outbox en una transacción. La corrección física requerirá conciliación respecto de los compromisos restantes; la alerta no modificará existencias.

La cancelación y la expiración liberarán el stock comprometido una sola vez; la entrega descontará tanto el físico registrado como el comprometido.

---

### 2.3 Clases del módulo Inventario

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `InventarioPort` | Interfaz | Dominio | Contrato para verificar, comprometer y liberar stock |
| `InventarioService` | Servicio de aplicación | Aplicación | Aplica las reglas de disponibilidad y coordina las operaciones de inventario |
| `StockMedicamento` | Entidad | Dominio | Representa las existencias de un medicamento en una farmacia |
| `PostgresInventarioRepository` | Repositorio | Infraestructura | Consulta y actualiza el inventario utilizando PostgreSQL |

La disponibilidad digital se calculará como stock físico registrado menos unidades comprometidas. Inventario incluirá registro manual e importación obligatoria .xlsx/.csv: un adaptador leerá archivos y aplicación coordinará plantilla de nombre, concentración, presentación, precio referencial y stock físico, vista previa, validación y mapeo al catálogo general antes de confirmar, sin suponer columnas uniformes.

Los ajustes e importaciones revalidarán compromisos concurrentes, sin sobrescribirlos ni guardar stock físico inferior al comprometido; las discrepancias requerirán conciliación. Para el MVP, la importación será atómica por archivo: si una fila es inválida o no está conciliada, no se guardará ninguna; se presentará un reporte de errores para corregir y reintentar. El archivo completo se guardará en una única transacción. PostgreSQL conservará movimientos_inventario con producto, farmacia, cantidad anterior y nueva, motivo, responsable y fecha. Las búsquedas mostrarán fecha de última revisión por la farmacia y disponibilidad referencial.

La operación de comprometer stock deberá ser atómica y rechazar solicitudes que excedan la disponibilidad digital registrada, sin garantizar existencia física hasta la aceptación.

---

### 2.4 Clases del módulo Pagos

El módulo Pagos se considera **opcional y posterior al MVP**. La creación básica de reservas no dependerá de una pasarela de pago.

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `ProcesarPagoUseCase` | Caso de uso | Aplicación | Coordinará el procesamiento de pagos cuando se incorpore esta funcionalidad |
| `PagoPort` | Interfaz | Dominio | Definirá el contrato para comunicarse con proveedores de pago |
| `ResultadoPago` | DTO | Aplicación | Representará el resultado de una solicitud de cobro |
| `PasarelaPagoAdapter` | Adaptador | Infraestructura | Implementará la integración con una pasarela externa |

La futura incorporación de pagos requerirá definir estados de pago independientes de los estados de reserva, reglas de idempotencia y manejo de fallos externos.

---

### 2.5 Clases de notificaciones

El núcleo de FarmaFind contará con mecanismos para registrar y publicar solicitudes de notificación, mientras que el servicio independiente de Notificaciones será responsable de procesarlas.

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `NotificacionPort` | Interfaz | Aplicación | Define el contrato para solicitar el registro o publicación de eventos de notificación |
| `NotificacionAdapter` | Adaptador | Infraestructura | Implementa la comunicación con el mecanismo de entrega de eventos |
| `RegistrarEventoReservaUseCase` | Caso de uso | Aplicación | Coordina el registro de eventos relacionados con reservas |
| `EventoOutbox` | Modelo de persistencia | Infraestructura | Representa un evento pendiente de publicación fiable |

La arquitectura contempla utilizar el patrón **Transactional Outbox** para registrar los eventos del ciclo de reservas y las alertas de corrección dentro de la misma transacción de PostgreSQL. También se propondrán recordatorios de inventario antiguo persistidos en Outbox con umbral y frecuencia configurables, sin alterar stock.

El procesamiento de notificaciones se realizará de manera independiente, permitiendo reintentos y deduplicación por identificador de evento.

---

### 2.6 Notación UML utilizada

| Símbolo | Significado | Ejemplo |
|---|---|---|
| Línea continua con flecha abierta | Asociación | `CrearReservaUseCase` → `ReservaRepository` |
| Línea discontinua con flecha abierta | Dependencia | `CrearReservaUseCase` utiliza `Reserva` |
| Línea discontinua con triángulo hueco | Realización | `PostgresReservaRepository` implementa `ReservaRepository` |
| Rombo relleno | Composición | Una entidad contiene objetos que forman parte de su ciclo de vida |
| `<<interface>>` | Interfaz | `ReservaRepository`, `InventarioPort`, `NotificacionPort` |
| `+` y `-` | Método público o atributo privado | `+ ejecutar()`, `- estado` |

---

## 3. Diagrama de secuencia: crear reserva

El flujo principal del MVP considera la reserva temporal de medicamentos **sin pago en línea obligatorio**.

La creación de la reserva y la actualización del inventario deberán confirmarse dentro de una única transacción de base de datos.

| Paso | Origen → destino | Acción |
|---|---|---|
| 1 | Cliente web → `ReservaController` | Envía `POST /api/v1/reservas` con medicamento, farmacia y cantidad |
| 2 | `ReservaController` → `CrearReservaUseCase` | Envía los datos validados y la identidad del cliente autenticado |
| 3 | `CrearReservaUseCase` → Gestor transaccional | Inicia una transacción PostgreSQL |
| 4 | `CrearReservaUseCase` → `InventarioPort` | Solicita comprometer la cantidad de stock de forma atómica |
| 5 | `InventarioPort` → `PostgresInventarioRepository` | Verifica y actualiza el inventario dentro de la transacción |
| 6 | `CrearReservaUseCase` → `Reserva` | Crea la entidad con estado PENDIENTE y fecha de expiración |
| 7 | `CrearReservaUseCase` → `ReservaRepository` | Guarda la reserva utilizando la misma transacción |
| 8 | `CrearReservaUseCase` → Persistencia Outbox | Registra el evento de reserva creada dentro de la transacción |
| 9 | Gestor transaccional → PostgreSQL | Confirma la transacción si todas las operaciones fueron exitosas |
| 10 | `ReservaController` → Cliente web | Responde `201 Created` con identificador, estado y fecha de expiración |
| 11 | Publicador Outbox → Servicio de Notificaciones | Publica el evento pendiente de manera independiente |
| 12 | Servicio de Notificaciones → Canal de envío | Procesa el aviso y aplica deduplicación y reintentos cuando corresponda |

#### Flujos alternativos

| Situación | Comportamiento esperado |
|---|---|
| Stock insuficiente | No se crea la reserva y se devuelve `409 Conflict` |
| Solicitud inválida | Se rechaza la solicitud con `400 Bad Request` y no se modifica el inventario |
| Usuario no autenticado | Se rechaza la operación protegida con `401 Unauthorized` |
| Error en la transacción | Se revierten los cambios de inventario, reserva y registro del evento |
| Falla del servicio de Notificaciones | La reserva permanece válida y el evento pendiente podrá procesarse posteriormente |
| Solicitudes concurrentes | La operación transaccional impide comprometer más unidades de las disponibles |

#### Consideraciones de consistencia

La disponibilidad deberá verificarse y actualizarse mediante un mecanismo seguro de concurrencia, como una actualización condicional atómica o un bloqueo transaccional apropiado.

El sistema no deberá dar por creada una reserva PENDIENTE si falla el compromiso de inventario, la persistencia de la reserva o el evento Outbox. Crear PENDIENTE no equivaldrá a la aceptación física CONFIRMADA.

El identificador de la reserva, el registro de inventario y el evento pendiente deberán permanecer consistentes después de confirmar la transacción.

---

## 4. Patrones y principios que se ven en este nivel

| Elemento del diseño | Patrón o principio |
|---|---|
| `ReservaRepository` y `PostgresReservaRepository` | Patrón **Repository** |
| `PagoPort` y `PasarelaPagoAdapter` | Patrón **Adapter**, previsto para pagos futuros |
| `NotificacionPort` y `NotificacionAdapter` | Patrón **Adapter** |
| `CrearReservaUseCase` depende de interfaces | Principio de **inversión de dependencias** |
| Separación entre controladores, casos de uso, entidades y repositorios | Principio de **responsabilidad única** |
| Reservas utiliza los contratos públicos de Inventario | Regla del **monolito modular** |
| Inventario y Reservas participan en una misma transacción | Consistencia transaccional |
| Registro de eventos de notificación en la transacción | Patrón **Transactional Outbox** |
| Procesamiento de eventos mediante identificadores únicos | Idempotencia y deduplicación |
| Las búsquedas pueden usar caché, pero las reservas validan stock transaccional | Separación entre optimización de consultas y consistencia |

---

## 5. Reglas importantes del flujo

| Código | Regla | Motivo |
|---|---|---|
| RN-01 | No se puede crear una reserva si no existe stock disponible suficiente | Evita reservas imposibles de atender |
| RN-02 | La cantidad solicitada debe ser un número entero positivo y no superar el stock disponible | Garantiza cantidades válidas |
| RN-03 | Toda reserva debe tener una fecha de expiración futura definida por la política del sistema | Evita reservas indefinidas |
| RN-04 | Si una reserva expira, su stock comprometido debe liberarse una sola vez | Mantiene un inventario consistente |
| RN-05 | Solo el personal autorizado de la farmacia correspondiente puede confirmar una entrega | Protege las operaciones del sistema |
| RN-06 | Una reserva entregada, cancelada o expirada no puede modificarse hacia un estado anterior | Impide transiciones inválidas |
| RN-07 | La actualización del inventario y la creación de la reserva deben ejecutarse en una única transacción | Evita inconsistencias y sobreasignación de stock |
| RN-08 | La creación de la reserva no debe depender de que el servicio de Notificaciones esté disponible | Mejora la resiliencia |
| RN-09 | La creación de una reserva no requiere pago electrónico durante el MVP | Mantiene independiente el proceso principal |
| RN-10 | Los cambios de estado deben seguir únicamente las transiciones definidas | Garantiza un ciclo de vida uniforme |
| RN-11 | La identidad del cliente debe obtenerse del contexto autenticado y no confiarse únicamente a datos enviados por el navegador | Evita operaciones en nombre de terceros |
| RN-12 | Las reservas concurrentes no deben comprometer cantidades superiores al stock digital disponible registrado | Garantiza consistencia ante solicitudes simultáneas |

---

## 6. Relación con los archivos de diseño interno

| Archivo | Relación |
|---|---|
| `../03-diseño-de-software/diseño-interno/diseño-interno-de-modulos.md` | Describe las capas, clases, contratos y flujo del módulo Reservas |
| `../03-diseño-de-software/diseño-interno/principios-de-diseño.md` | Sustenta la aplicación de los principios SOLID |
| `../03-diseño-de-software/diseño-interno/patrones-de-diseño.md` | Describe los patrones Repository, Adapter y Dependency Injection |
| `../01-analisis-de-sistema/03-requisitos-funcionales.md` | Define los requisitos funcionales relacionados con inventario y reservas |
| `../01-analisis-de-sistema/04-atributos-de-calidad.md` | Establece los atributos de consistencia, seguridad y resiliencia |
| `../01-analisis-de-sistema/07-decisiones-arquitectonicas.md` | Contiene las decisiones arquitectónicas que justifican el diseño |
| `../01-analisis-de-sistema/08-escenarios-atributos-calidad.md` | Establece los escenarios utilizados para validar las decisiones del sistema |
