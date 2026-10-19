# Requisitos funcionales del sistema FarmaFind

## Requisitos funcionales

Los siguientes requisitos describen las funciones que deberá ofrecer FarmaFind para permitir la búsqueda de medicamentos, la gestión de reservas y la administración de farmacias.

| ID | Requisito funcional |
|---|---|
| RF01 | El sistema debe permitir buscar medicamentos por nombre comercial o principio activo, sin requerir iniciar sesión. |
| RF02 | El sistema debe mostrar las farmacias que registren disponibilidad del medicamento consultado. Las búsquedas utilizarán Redis como caché y PostgreSQL como fuente principal de datos. |
| RF03 | El sistema debe mostrar la disponibilidad, el precio referencial y la dirección o ubicación de las farmacias que ofrecen el medicamento. |
| RF04 | El sistema debe permitir filtrar u ordenar los resultados según disponibilidad, precio o cercanía. Si el servicio de mapas falla, deberá mostrar la dirección textual de la farmacia. |
| RF05 | El sistema debe permitir a un cliente autenticado reservar temporalmente una cantidad disponible de un medicamento en una farmacia, registrando la reserva en estado PENDIENTE y asignándole una fecha de expiración. |
| RF06 | El sistema debe verificar y comprometer el stock disponible mediante una operación atómica en PostgreSQL, evitando reservas que superen las existencias. El compromiso de stock y el registro de la reserva deberán realizarse dentro de una misma transacción. |
| RF07 | El sistema debe permitir cancelar reservas vigentes y expirar automáticamente las reservas PENDIENTES o CONFIRMADAS que superen su plazo sin entrega. En ambos casos deberá liberar el stock comprometido una sola vez. |
| RF08 | El sistema debe permitir al cliente consultar el estado y la fecha de expiración de sus reservas, impidiendo el acceso no autorizado a reservas de otros usuarios. |
| RF09 | El sistema debe permitir al personal autorizado de farmacia aceptar reservas PENDIENTES, cambiándolas a CONFIRMADA, y registrar la entrega de reservas CONFIRMADAS, cambiándolas a ENTREGADA y actualizando el inventario. |
| RF10 | El sistema debe permitir al administrador de farmacia registrar y actualizar medicamentos, precios y existencias de su establecimiento, sin generar cantidades inválidas ni reducir el stock físico por debajo del comprometido. |
| RF11 | El sistema debe permitir al administrador de FarmaFind registrar, actualizar y desactivar farmacias participantes, impidiendo nuevas reservas en farmacias desactivadas. |
| RF12 | El sistema debe generar y procesar notificaciones relacionadas con la creación, confirmación y vencimiento de reservas, mediante un servicio independiente. Los eventos deberán persistirse con Transactional Outbox para que una falla del servicio no impida registrar reservas. |
| RF13 | El sistema podrá incorporar, en una futura iteración, el pago electrónico de reservas mediante una pasarela externa. El pago no será obligatorio para realizar reservas durante el MVP. |

### Reglas funcionales generales

- Las reservas utilizarán los estados **PENDIENTE, CONFIRMADA, ENTREGADA, CANCELADA y EXPIRADA**.
- Los estados ENTREGADA, CANCELADA y EXPIRADA serán terminales.
- PostgreSQL será la fuente oficial de inventario y reservas.
- Redis será obligatorio en el MVP para optimizar búsquedas mediante Cache-Aside. Si Redis falla, las búsquedas continuarán utilizando PostgreSQL.
- Las modificaciones relevantes de medicamentos, precios, farmacias e inventario deberán reflejarse en las búsquedas mediante invalidación de caché.
- Las operaciones críticas de reservas e inventario utilizarán transacciones PostgreSQL coordinadas mediante Unit of Work.
- El servicio independiente de Notificaciones procesará eventos registrados mediante Transactional Outbox.
- Los pagos electrónicos y la sincronización automática con inventarios externos quedarán previstos para futuras iteraciones.

---

## Relación entre requisitos funcionales e historias de usuario

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU01 - Buscar medicamentos y farmacias | RF01, RF02 |
| HU02 - Consultar y ordenar resultados | RF03, RF04 |
| HU03 - Reservar medicamentos | RF05, RF06, RF07, RF08 |
| HU04 - Gestionar y entregar reservas | RF09 |
| HU05 - Administrar inventario | RF10 |
| HU06 - Administrar la plataforma y notificaciones | RF11, RF12, RF13 |
