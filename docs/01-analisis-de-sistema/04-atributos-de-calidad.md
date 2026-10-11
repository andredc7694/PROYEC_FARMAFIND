# Atributos de calidad del sistema FarmaFind

| ID | Atributo | Escenario y medida propuesta |
|---|---|---|
| AC01 | Rendimiento | Bajo la carga definida, p95 de búsquedas ≤ 2 s y errores técnicos < 1 %. Reportar también p99, throughput y recursos; separar conflictos de stock de errores técnicos. |
| AC02 | Disponibilidad  | Con dos instancias del núcleo, detener una y verificar que se reanuda el servicio por la instancia sana en ≤ 30 s. La base única seguirá siendo un punto de fallo declarado. |
| AC03 | Escalabilidad  | Comparar una y dos instancias con igual dataset y recursos de base de datos, elevando la carga. Medir capacidad antes de superar AC01; documentar el cuello de botella y cualquier mejora no lograda. |
| AC04 | Seguridad  | Todas las pruebas de acceso a reservas ajenas y administración de otra farmacia deberán ser rechazadas; los logs no deben exponer contraseñas ni tokens. |
| AC05 | Consistencia  | Con 100 intentos simultáneos de reservar una unidad sobre un stock de 10, aceptar exactamente 10 solicitudes distintas y no producir valores negativos ni duplicados. |
| AC06 | Mantenibilidad  | Sustituir el adaptador de notificaciones por uno simulado sin modificar dominio ni casos de uso; ejecutar pruebas de estos componentes. |
| AC07 | Interoperabilidad  | Validar el contrato entre el núcleo y Notificaciones y repetir un evento: el consumidor debe registrar un único trabajo por identificador. |
| AC08 | Resiliencia  | Interrumpir Notificaciones durante 5 minutos: las reservas deben continuar y los avisos pendientes deben persistir. Tras recuperar el servicio, deben procesarse sin pérdida de trabajos y medirse el retraso. |
| AC09 | Observabilidad y pruebas  | Relacionar cada reserva de prueba con logs y correlación; medir latencia, errores, CPU, memoria, conexiones y trabajos pendientes. Derivar pruebas de los criterios de aceptación. |