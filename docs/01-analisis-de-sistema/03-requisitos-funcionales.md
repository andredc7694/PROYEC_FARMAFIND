# Requisitos funcionales del sistema FarmaFind

## Requisitos funcionales

| ID | Requisito funcional |
|---|---|
| RF01 | El sistema debe permitir buscar medicamentos por nombre comercial o principio activo. |
| RF02 | El sistema debe mostrar las farmacias que disponen del medicamento consultado. |
| RF03 | El sistema debe mostrar la disponibilidad, precio referencial y ubicación de cada farmacia que cuente con el medicamento. |
| RF04 | El sistema debe permitir filtrar u ordenar los resultados de búsqueda según criterios como disponibilidad, precio o cercanía. |
| RF05 | El sistema debe permitir al usuario realizar una reserva temporal de una cantidad disponible del medicamento. |
| RF06 | El sistema debe verificar la disponibilidad antes de confirmar una reserva, evitando comprometer una cantidad mayor al stock existente. |
| RF07 | El sistema debe liberar automáticamente el stock reservado cuando una reserva expire sin haber sido confirmada o atendida. |
| RF08 | El sistema debe permitir al usuario consultar el estado de sus reservas. |
| RF09 | El sistema debe permitir al personal de farmacia confirmar la entrega de un medicamento reservado. |
| RF10 | El sistema debe permitir al administrador de farmacia registrar y actualizar medicamentos, precios y stock. |
| RF11 | El sistema debe permitir al administrador de FarmaFind registrar, actualizar o desactivar farmacias participantes. |
| RF12 | El sistema debe permitir enviar notificaciones relacionadas con la creación, confirmación o próxima expiración de una reserva. |
| RF13 | El sistema debe permitir, de manera opcional, procesar el pago de una reserva mediante una pasarela de pago externa. |

## Relación entre requisitos funcionales e historias de usuario

| Historia de usuario | Requisitos funcionales relacionados |
|---|---|
| HU01 | RF01, RF02 |
| HU02 | RF03, RF04 |
| HU03 | RF05, RF06, RF07, RF08 |
| HU04 | RF09 |
| HU05 | RF10 |
| HU06 | RF11, RF12, RF13 |