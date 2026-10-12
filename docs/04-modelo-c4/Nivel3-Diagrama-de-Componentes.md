# Nivel 3 · Diagrama de componentes

> **Proyecto:** FarmaFind · **Contenedor detallado:** API FarmaFind  
> **Modelo:** C4 · Nivel 3 de 4 · **Versión:** 1.0

---

## 1. Objetivo

Abrir el contenedor **API FarmaFind** del nivel 2 y mostrar sus **componentes**: los módulos internos del monolito modular, la responsabilidad de cada uno y cómo se relacionan entre sí, con la base de datos, la caché y los sistemas externos.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Cómo se divide el backend por dentro y qué hace cada parte? |
| Audiencia | Arquitectos, desarrolladores y equipo del proyecto |
| Qué es un componente | Un grupo de funcionalidades con una responsabilidad clara. En FarmaFind, cada **módulo** del backend representa un componente |
| Nivel anterior | `Nivel2-DiagramadeContenedores.md` |
| Siguiente nivel | `Nivel4-DiagramadeCodigo.md` |

---

## 2. Diagrama

![Diagrama de componentes de la API FarmaFind](../../img/nivel3-componentes-api-farmafind.png)

> Archivo editable: `img/fuentes/nivel3-componentes-farmafind.png`

---

## 3. Componentes

| Componente | Responsabilidad | Funcionalidades principales | Carpeta en el código |
|---|---|---|---|
| **API REST** | Punto de entrada HTTP del backend | Rutas `/api/v1`, autenticación JWT, validación de entrada y manejo de errores | `src/api/` |
| **Usuarios** | Identidad y acceso | Registro, inicio de sesión, roles de cliente, personal de farmacia, administrador de farmacia y administrador de plataforma | `src/modules/usuarios/` |
| **Farmacias** | Gestión de farmacias afiliadas | Registro, actualización, ubicación, estado y datos de contacto de farmacias | `src/modules/farmacias/` |
| **Medicamentos** | Gestión del catálogo de medicamentos | Registro de medicamentos, nombre comercial, principio activo, presentación y datos generales | `src/modules/medicamentos/` |
| **Inventario** | Control de stock y precios por farmacia | Stock disponible, stock reservado, precios y actualización manual del inventario | `src/modules/inventario/` |
| **Búsqueda** | Consulta de medicamentos disponibles | Buscar por nombre comercial o principio activo, filtrar por disponibilidad, precio o cercanía | `src/modules/busqueda/` |
| **Reservas** | Proceso principal de reserva temporal | Crear reserva, verificar stock, consultar estado, liberar reserva expirada y confirmar entrega | `src/modules/reservas/` |
| **Pagos** | Pago opcional de una reserva | Solicitar cobro mediante pasarela externa y registrar resultado del pago | `src/modules/pagos/` |
| **Notificaciones** | Comunicación con el cliente | Enviar avisos sobre reserva creada, confirmada, expirada o próxima a vencer | `src/modules/notificaciones/` |
| **Administración** | Operación general de la plataforma | Gestión de usuarios, farmacias afiliadas y configuración general del sistema | `src/modules/administracion/` |

---

## 4. Relaciones entre componentes

| Origen | Destino | Descripción | Tipo de comunicación |
|---|---|---|---|
| Aplicación web | API REST | Llama a la API para ejecutar operaciones del sistema. | HTTPS/JSON |
| API REST | Módulos de negocio | Invoca los casos de uso según la ruta solicitada. | Llamada en memoria |
| Búsqueda | Medicamentos | Consulta datos generales del medicamento buscado. | Llamada en memoria |
| Búsqueda | Farmacias | Consulta farmacias disponibles y ubicación. | Llamada en memoria |
| Búsqueda | Inventario | Consulta stock, precio y disponibilidad por farmacia. | Llamada en memoria |
| Reservas | Inventario | Verifica disponibilidad, reserva stock y libera stock cuando una reserva expira. | Llamada en memoria |
| Reservas | Medicamentos | Valida la existencia del medicamento reservado. | Llamada en memoria |
| Reservas | Farmacias | Valida la farmacia donde se realizará la reserva. | Llamada en memoria |
| Reservas | Pagos | Solicita el pago opcional de una reserva. | Llamada en memoria |
| Reservas | Notificaciones | Solicita avisos de reserva creada, confirmada o expirada. | Evento o llamada en memoria |
| Administración | Usuarios | Administra cuentas y roles del sistema. | Llamada en memoria |
| Administración | Farmacias | Registra, actualiza o desactiva farmacias afiliadas. | Llamada en memoria |

---

## 5. Relaciones con contenedores y sistemas externos

| Componente | Destino | Descripción | Protocolo | Patrón aplicado |
|---|---|---|---|---|
| Usuarios | Base de datos PostgreSQL | Guarda usuarios, credenciales protegidas y roles. | SQL | Repository |
| Farmacias | Base de datos PostgreSQL | Guarda datos de farmacias, ubicación y estado. | SQL | Repository |
| Medicamentos | Base de datos PostgreSQL | Guarda información general de medicamentos. | SQL | Repository |
| Inventario | Base de datos PostgreSQL | Guarda stock disponible, stock reservado y precios por farmacia. | SQL | Repository |
| Reservas | Base de datos PostgreSQL | Guarda reservas, estados, fechas de expiración y confirmaciones. | SQL | Repository |
| Búsqueda | Caché Redis | Guarda consultas frecuentes de medicamentos y farmacias. | TCP | Decorator |
| Pagos | Pasarela de pago | Procesa pagos opcionales de reservas. | HTTPS/REST | Adapter |
| Farmacias/Búsqueda | Servicio de mapas | Consulta ubicación o distancia referencial de farmacias. | HTTPS/REST | Adapter |
| Notificaciones | Servicio de notificaciones | Envía avisos al cliente o farmacia. | HTTPS/REST o cola interna | Adapter |
| Inventario | Inventario externo futuro | Sincroniza stock con sistemas propios de farmacias en futuras iteraciones. | HTTPS/REST | Adapter |

---

## 6. Reglas del monolito modular

| # | Regla | Motivo |
|---|---|---|
| 1 | Todos los componentes principales se despliegan juntos dentro de la API FarmaFind. | Es un monolito modular: un solo proceso con módulos internos separados. |
| 2 | Un módulo usa a otro solo mediante su API pública o casos de uso expuestos. | Evita depender de detalles internos de otro módulo. |
| 3 | Un módulo no debe modificar directamente las tablas internas de otro módulo. | Mantiene independencia lógica y reduce errores entre módulos. |
| 4 | Cada integración externa debe estar encapsulada en un adaptador. | Permite cambiar proveedores sin afectar las reglas de negocio. |
| 5 | Cada módulo se organiza internamente con Clean Architecture. | Las reglas de negocio no dependen de Express, PostgreSQL, Redis ni servicios externos. |
| 6 | Las reservas deben coordinarse con inventario usando transacciones. | Evita reservar más medicamentos que el stock disponible. |
| 7 | Las búsquedas pueden usar caché, pero la reserva siempre debe validar stock real. | Mejora rendimiento sin comprometer la consistencia del inventario. |

---