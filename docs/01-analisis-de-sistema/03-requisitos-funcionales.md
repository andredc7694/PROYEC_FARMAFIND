# Requisitos funcionales del sistema FarmaFind

## Requisitos funcionales

Los siguientes requisitos describen las funciones que deberá ofrecer FarmaFind para permitir la búsqueda de medicamentos, la gestión de reservas y la administración de farmacias.

| ID | Requisito funcional |
|---|---|
| RF01 | El sistema debe permitir buscar medicamentos por nombre comercial o principio activo, sin requerir iniciar sesión. |
| RF02 | El sistema debe mostrar las farmacias que registren disponibilidad del medicamento consultado. Las búsquedas utilizarán Redis como caché y PostgreSQL como fuente principal de datos. |
| RF03 | El sistema debe mostrar la disponibilidad referencial, su fecha de última actualización, el precio referencial y la dirección o ubicación de las farmacias que ofrecen el medicamento. |
| RF04 | El sistema debe permitir filtrar u ordenar los resultados según disponibilidad, precio o cercanía. Si el servicio de mapas falla, deberá mostrar la dirección textual de la farmacia. |
| RF05 | El sistema debe permitir a un cliente autenticado reservar temporalmente una cantidad disponible de un medicamento en una farmacia, registrando la reserva en estado PENDIENTE con un plazo de respuesta inicialmente de 20 minutos configurable. La disponibilidad física quedará sujeta a comprobación y aceptación de la farmacia. |
| RF06 | El sistema debe verificar y comprometer el stock disponible mediante una operación atómica en PostgreSQL, evitando reservas que superen las existencias. El compromiso de stock digital, el registro de la reserva PENDIENTE y su evento Outbox deberán realizarse en una misma transacción mediante Unit of Work; Redis no validará reservas. |
| RF07 | El sistema debe permitir cancelar reservas vigentes y expirar automáticamente las reservas PENDIENTES o CONFIRMADAS que superen su plazo sin entrega. En ambos casos deberá liberar el stock comprometido una sola vez. |
| RF08 | El sistema debe permitir al cliente consultar el estado y la fecha de expiración de sus reservas, impidiendo el acceso no autorizado a reservas de otros usuarios. |
| RF09 | El sistema debe permitir al dueño o administrador de farmacia, o al personal con delegación expresa, aceptar reservas PENDIENTES vigentes tras comprobar físicamente el producto y pasarlas a CONFIRMADA. Al confirmar se asignará un plazo de recogida configurable, inicialmente propuesto en 24 horas y sujeto a validación. El personal autorizado podrá registrar la entrega de CONFIRMADAS vigentes, pasándolas a ENTREGADA y descontando stock físico y comprometido. Si falta stock físico en PENDIENTE, se cancelará con motivo STOCK_FISICO_INSUFICIENTE, se liberará el compromiso una sola vez y se registrarán incidencia y alerta de corrección mediante Outbox. |
| RF10 | El sistema debe permitir al administrador de farmacia registrar manualmente e importar Excel (.xlsx/.csv) como función obligatoria del MVP para gestionar precios y existencias propios, asociando productos al catálogo general para evitar duplicidades. Se propondrá una plantilla con nombre del medicamento, concentración, presentación, precio referencial y stock físico; se permitirá vista previa, validación y mapeo de columnas antes de confirmar. Los ajustes e importaciones no sobrescribirán compromisos ni dejarán stock físico inferior al comprometido; las discrepancias requerirán conciliación. PostgreSQL conservará movimientos_inventario con producto, farmacia, cantidad anterior y nueva, motivo, responsable y fecha. |
| RF11 | El sistema debe permitir al administrador de FarmaFind registrar, actualizar y desactivar farmacias participantes, impidiendo nuevas reservas en farmacias desactivadas. Su función será supervisora, sin exigirle actualizar el inventario de todos los establecimientos. |
| RF12 | El sistema debe generar y procesar notificaciones relacionadas con la creación, confirmación, entrega, cancelación y vencimiento de reservas, alertas de corrección y recordatorios propuestos por inventario antiguo, mediante un servicio independiente. Los eventos deberán persistirse con Transactional Outbox para que una falla del servicio no impida registrar reservas. Los recordatorios tendrán umbral de antigüedad y frecuencia configurables y no modificarán existencias automáticamente. |
| RF13 | El sistema podrá incorporar, en una futura iteración, el pago electrónico de reservas mediante una pasarela externa. El pago no será obligatorio para realizar reservas durante el MVP. |

### Reglas funcionales generales

- Las reservas utilizarán los estados **PENDIENTE, CONFIRMADA, ENTREGADA, CANCELADA y EXPIRADA**.
- Los estados ENTREGADA, CANCELADA y EXPIRADA serán terminales.
- PostgreSQL será la fuente oficial de inventario y reservas. El stock físico registrado podrá diferir de la existencia observada; stock disponible digital = físico registrado − comprometido. La búsqueda y la creación de PENDIENTE no garantizarán existencia física.
- La fecha de última actualización corresponderá a la revisión del inventario por la farmacia; una consulta, recarga de caché o compromiso de reserva no renovará esa fecha.
- Para el MVP, la importación será atómica por archivo: si una fila es inválida o no está conciliada, no se guardará ninguna; se presentará un reporte de errores para corregir y reintentar. Se revalidarán stock y compromisos concurrentes al guardar el archivo completo en una única transacción.
- La conciliación resolverá compromisos afectados mediante transiciones válidas y liberación única antes de guardar un ajuste físico inferior. Una alerta o recordatorio no alterará el stock por sí mismo.
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
