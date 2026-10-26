# Historias de usuario del sistema FarmaFind


## Historias de usuario

Las siguientes historias describen las principales funcionalidades de FarmaFind desde la perspectiva de los usuarios. Cada historia incluye criterios de aceptación para comprobar su cumplimiento.

| Código | Actor | Historia de usuario | Criterios de aceptación | Requisitos relacionados | Alcance |
|---|---|---|---|---|---|
| HU01 | Cliente | Como cliente, quiero buscar medicamentos por nombre comercial o principio activo y ver las farmacias que los tienen disponibles, para encontrar rápidamente dónde conseguirlos. | 1. Buscar por nombre o principio activo.<br>2. Mostrar las farmacias con disponibilidad registrada.<br>3. Permitir búsquedas sin iniciar sesión.<br>4. Mostrar un mensaje cuando no existan resultados.<br>5. Mantener las búsquedas operativas si Redis falla. | RF01, RF02 | MVP |
| HU02 | Cliente | Como cliente, quiero consultar la disponibilidad, precio referencial y ubicación de los medicamentos, y ordenar los resultados por cercanía, precio o disponibilidad, para elegir la farmacia más conveniente. | 1. Mostrar precio referencial, disponibilidad referencial, fecha de última actualización y dirección de cada farmacia.<br>2. Permitir filtrar u ordenar resultados.<br>3. Mostrar la dirección textual si el servicio de mapas falla.<br>4. Presentar información referencial con TTL e invalidación, sin garantizar existencia física hasta la aceptación de la farmacia. | RF03, RF04 | MVP |
| HU03 | Cliente | Como cliente, quiero reservar temporalmente una cantidad disponible de un medicamento y consultar el estado de mis reservas, para solicitar el producto y esperar su confirmación física antes de ir a la farmacia. | 1. Requerir autenticación para reservar.<br>2. Validar stock digital en PostgreSQL; comprometer unidades, crear PENDIENTE y registrar Outbox en una transacción mediante Unit of Work.<br>3. Crear la reserva en estado PENDIENTE.<br>4. Mostrar estado y vencimiento: respuesta PENDIENTE de 20 minutos configurable; al confirmar, plazo de recogida configurable de 24 horas inicialmente propuesto, sujeto a validación.<br>5. Liberar el stock si la reserva se cancela o expira.<br>6. Evitar reservas que superen el stock disponible.<br>7. No requerir pago electrónico durante el MVP. | RF05, RF06, RF07, RF08 | MVP |
| HU04 | Personal de farmacia | Como personal de farmacia, quiero revisar las reservas de mi establecimiento, aceptar las pendientes si cuento con delegación expresa y confirmar la entrega de los medicamentos, para registrar correctamente su atención. | 1. Mostrar solo reservas de farmacias autorizadas.<br>2. Permitir al dueño, administrador o personal expresamente delegado cambiar PENDIENTE vigente a CONFIRMADA tras comprobar existencias físicas.<br>3. Confirmar la entrega únicamente de reservas CONFIRMADAS vigentes.<br>4. Cambiar el estado a ENTREGADA y actualizar el inventario.<br>5. Impedir entregar reservas canceladas, expiradas o ya entregadas.<br>6. Ante falta física, cancelar PENDIENTE con motivo STOCK_FISICO_INSUFICIENTE, liberar compromisos una sola vez y registrar incidencia y alerta de corrección vía Outbox. | RF09 | MVP |
| HU05 | Administrador de farmacia | Como administrador de farmacia, quiero registrar y actualizar medicamentos, precios y stock, para mantener correcta la información de mi establecimiento. | 1. Registrar manualmente e importar obligatoriamente Excel (.xlsx/.csv), asociando productos al catálogo general sin duplicarlos.<br>2. Actualizar precios y existencias propios con plantilla de nombre, concentración, presentación, precio referencial y stock físico; permitir vista previa, validación y mapeo de columnas antes de confirmar. La importación será atómica por archivo: si una fila es inválida o no está conciliada, no se guardará ninguna y se presentará un reporte de errores para corregir y reintentar.<br>3. Impedir cantidades inválidas, sobrescribir compromisos o registrar stock físico inferior al comprometido; exigir conciliación y registrar movimientos_inventario.<br>4. Permitir modificar únicamente el inventario autorizado.<br>5. Reflejar los cambios en las búsquedas mediante invalidación de caché. | RF10 | MVP |
| HU06 | Administrador de FarmaFind | Como administrador de FarmaFind, quiero gestionar farmacias participantes y supervisar las notificaciones, para administrar las funciones complementarias de la plataforma. | 1. Registrar, actualizar y desactivar farmacias.<br>2. Impedir nuevas reservas en farmacias desactivadas.<br>3. Generar eventos para notificaciones relacionadas con reservas.<br>4. Conservar eventos pendientes si Notificaciones falla; proponer recordatorios por inventario antiguo sin alterar existencias, con umbral y frecuencia configurables.<br>5. Mantener los pagos electrónicos como funcionalidad opcional futura. | RF11, RF12, RF13 | Administración y notificaciones: MVP. Pagos: futuro. |

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

---

## Consideraciones arquitectónicas

- **Redis:** será obligatorio en el MVP para optimizar las búsquedas mediante Cache-Aside. Si falla, las consultas continuarán utilizando PostgreSQL.
- **PostgreSQL:** almacenará los datos oficiales y garantizará la consistencia entre inventario y reservas mediante transacciones.
- **Reservas:** utilizarán los estados PENDIENTE, CONFIRMADA, ENTREGADA, CANCELADA y EXPIRADA.
- **Notificaciones:** funcionará como un servicio independiente y recibirá eventos persistidos mediante Transactional Outbox.
- **Pagos:** no serán obligatorios durante el MVP y se consideran una funcionalidad futura.
