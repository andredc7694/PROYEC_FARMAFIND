# Nivel 4 · Diagrama de código

> **Proyecto:** FarmaFind · **Módulos detallados:** Reservas, Inventario y Pagos  
> **Modelo:** C4 · Nivel 4 de 4 · **Notación:** UML · **Versión:** 1.0

---

## 1. Objetivo

Mostrar **cómo está organizado el código** de uno de los flujos más importantes de FarmaFind: la creación de una reserva temporal de medicamento. Este nivel detalla clases, interfaces, casos de uso y adaptadores que participan en el proceso **Crear reserva**, involucrando principalmente los módulos **Reservas**, **Inventario** y **Pagos**.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué clases existen dentro de un módulo y cómo colaboran para cumplir una regla de negocio? |
| Audiencia | Desarrolladores |
| Notación | UML: diagrama de clases y diagrama de secuencia |
| Por qué estos módulos | Reservas es el proceso central de FarmaFind; Inventario asegura disponibilidad real y Pagos permite el cobro opcional |
| Nivel anterior | `Nivel3-Diagrama-de-Componentes.md` |

---

## 2. Diagrama de clases

### 2.1 Capas de Clean Architecture

| Capa | Contiene | Regla |
|---|---|---|
| **Presentación** | Controladores HTTP | Traduce peticiones del navegador a llamadas de casos de uso |
| **Aplicación** | Casos de uso | Coordina el flujo de negocio sin depender de tecnologías concretas |
| **Dominio** | Entidades, objetos de valor, interfaces y reglas | Contiene la lógica principal del negocio |
| **Infraestructura** | Repositorios, adaptadores y servicios externos | Implementa los contratos usando PostgreSQL, Redis o APIs externas |

> **Regla de dependencia:** las clases externas dependen del dominio, pero el dominio no depende de Express, PostgreSQL, Redis ni servicios externos.

---

## 2.2 Clases del módulo Reservas

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `ReservaController` | Controlador | Presentación | Recibe `POST /api/v1/reservas` y devuelve la respuesta al cliente |
| `CrearReservaUseCase` | Caso de uso | Aplicación | Coordina la creación de la reserva, valida stock, solicita pago opcional y guarda la reserva |
| `Reserva` | Entidad | Dominio | Representa una reserva temporal y controla sus estados |
| `EstadoReserva` | Enumeración | Dominio | PENDIENTE, CONFIRMADA, EXPIRADA, CANCELADA, ENTREGADA |
| `ReservaRepository` | Interfaz | Dominio | Contrato para guardar, buscar y actualizar reservas |
| `PostgresReservaRepository` | Repositorio | Infraestructura | Implementa `ReservaRepository` usando PostgreSQL |

---

## 2.3 Clases del módulo Inventario

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `InventarioPort` | Interfaz | Dominio | Contrato para verificar, reservar y liberar stock |
| `InventarioService` | Servicio de aplicación | Aplicación | Aplica reglas de stock disponible y stock reservado |
| `StockMedicamento` | Entidad | Dominio | Representa el stock de un medicamento en una farmacia |
| `PostgresInventarioRepository` | Repositorio | Infraestructura | Consulta y actualiza stock en PostgreSQL |

---

## 2.4 Clases del módulo Pagos

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `ProcesarPagoUseCase` | Caso de uso público | Aplicación | Permite que Reservas solicite un cobro opcional |
| `PagoPort` | Interfaz | Dominio | Contrato para procesar pagos |
| `ResultadoPago` | DTO | Dominio | Indica si el pago fue aprobado, rechazado o quedó pendiente |
| `PasarelaPagoAdapter` | Adaptador | Infraestructura | Implementa `PagoPort` usando una pasarela externa |

---

## 2.5 Clases de notificaciones

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `NotificacionPort` | Interfaz | Dominio | Contrato para enviar avisos de reserva |
| `NotificacionAdapter` | Adaptador | Infraestructura | Envía mensajes por correo, SMS, WhatsApp u otro canal definido |
| `EnviarNotificacionReservaUseCase` | Caso de uso | Aplicación | Envía avisos cuando una reserva se crea, confirma o expira |

---

## 2.6 Notación UML utilizada

| Símbolo | Significado | Ejemplo |
|---|---|---|
| Línea continua con flecha abierta | Asociación | `CrearReservaUseCase` → `ReservaRepository` |
| Línea discontinua con flecha abierta | Dependencia | `CrearReservaUseCase` usa `Reserva` |
| Línea discontinua con triángulo hueco | Realización | `PostgresReservaRepository` implementa `ReservaRepository` |
| Rombo relleno | Composición | `Reserva` contiene datos del medicamento reservado |
| `<<interface>>` | Interfaz | `ReservaRepository`, `InventarioPort`, `PagoPort` |
| `+` y `-` | Método público o atributo privado | `+ ejecutar()`, `- estado` |

---

## 3. Diagrama de secuencia: crear reserva

| Paso | Origen → destino | Acción |
|---|---|---|
| 1 | Cliente web → `ReservaController` | Envía `POST /api/v1/reservas` con medicamento, farmacia, cantidad y datos del usuario |
| 2 | `ReservaController` → `CrearReservaUseCase` | Ejecuta el caso de uso |
| 3 | `CrearReservaUseCase` → `InventarioPort` | Verifica disponibilidad real del medicamento |
| 4 | `InventarioPort` → `PostgresInventarioRepository` | Consulta stock disponible en PostgreSQL |
| 5 | `CrearReservaUseCase` → `InventarioPort` | Reserva temporalmente el stock |
| 6 | `CrearReservaUseCase` → `Reserva` | Crea la entidad Reserva con estado PENDIENTE |
| 7 | `CrearReservaUseCase` → `ProcesarPagoUseCase` | Solicita pago opcional si el flujo lo requiere |
| 8 | `ProcesarPagoUseCase` → `PagoPort` | Envía solicitud de cobro |
| 9 | `PagoPort` → `PasarelaPagoAdapter` | Procesa el pago con la pasarela externa |
| 10 | `CrearReservaUseCase` → `ReservaRepository` | Guarda la reserva en PostgreSQL |
| 11 | `CrearReservaUseCase` → `NotificacionPort` | Solicita aviso de reserva creada |
| 12 | `ReservaController` → Cliente web | Responde `201 Created` con el código de reserva |
| 13 | Alternativa | Si no hay stock, no se crea reserva y se responde `409 Conflict` |
| 14 | Alternativa | Si el pago es rechazado, la reserva puede quedar pendiente o cancelarse según la regla definida |

---

## 4. Patrones y principios que se ven en este nivel

| Elemento del diagrama | Patrón o principio |
|---|---|
| `ReservaRepository` y `PostgresReservaRepository` | Patrón **Repository** |
| `PagoPort` y `PasarelaPagoAdapter` | Patrón **Adapter** |
| `NotificacionPort` y `NotificacionAdapter` | Patrón **Adapter** |
| `CrearReservaUseCase` depende de interfaces | Principio de **inversión de dependencias** |
| `ReservaController`, `CrearReservaUseCase`, `Reserva` y repositorios tienen responsabilidades separadas | Principio de **responsabilidad única** |
| Reservas se comunica con Pagos mediante `ProcesarPagoUseCase` | Regla del **monolito modular** |
| La reserva valida stock antes de guardar | Regla de consistencia del inventario |
| La caché no se usa para confirmar disponibilidad | Seguridad frente a datos desactualizados |

---

## 5. Reglas importantes del flujo

| # | Regla | Motivo |
|---|---|---|
| 1 | No se puede crear una reserva si no existe stock disponible. | Evita sobreventa o reservas imposibles de atender. |
| 2 | El stock debe reservarse temporalmente al crear la reserva. | Impide que dos usuarios reserven el mismo medicamento al mismo tiempo. |
| 3 | Toda reserva debe tener una fecha de expiración. | El stock no debe quedar bloqueado indefinidamente. |
| 4 | Si la reserva expira, el stock reservado debe liberarse. | Permite que otros clientes puedan reservar el medicamento. |
| 5 | El pago es opcional y no debe bloquear el funcionamiento básico de reserva. | FarmaFind puede operar con recojo y pago presencial en farmacia. |
| 6 | Las notificaciones no deben romper la creación de la reserva. | Si falla el aviso, la reserva igual puede quedar registrada. |

---

## 6. Relación con los archivos de diseño interno

| Archivo | Relación |
|---|---|
| `03-diseño-de-software/diseño-interno/01-clean-architecture.md` | Explica la separación por capas usada en este nivel |
| `03-diseño-de-software/diseño-interno/02-hexagonal.md` | Explica puertos y adaptadores como `PagoPort` e `InventarioPort` |
| `03-diseño-de-software/diseño-interno/03-solid-patrones.md` | Sustenta SOLID, Repository y Adapter |
| `03-diseño-de-software/diseño-interno/04-modelo-datos.md` | Relaciona reservas, inventario, medicamentos y farmacias |
| `03-diseño-de-software/diseño-interno/05-api-contratos.md` | Define endpoints como `POST /api/v1/reservas` |

---