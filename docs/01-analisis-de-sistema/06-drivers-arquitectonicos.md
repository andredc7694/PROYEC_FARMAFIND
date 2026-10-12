# Drivers arquitectónicos del sistema FarmaFind

# Drivers arquitectónicos

| ID | Driver arquitectónico | Origen | Influencia |
|---|---|---|---|
| DA01 | El sistema debe mantener actualizada la disponibilidad de medicamentos en las farmacias. | RF02, RF03, RF10, RC07 | Influye en el manejo del inventario, la fecha de actualización y la futura integración con sistemas externos. |
| DA02 | El sistema debe evitar errores cuando varios clientes intentan reservar el mismo medicamento al mismo tiempo. | RF05, RF06, RF07, AC05 | Influye en el uso de transacciones, control de concurrencia y actualización segura del stock. |
| DA03 | El sistema debe responder rápidamente a las búsquedas de medicamentos y farmacias. | RF01, RF02, RF03, RF04, AC01 | Influye en el uso de índices, paginación, límites de consulta y pruebas de rendimiento. |
| DA04 | El sistema debe proteger las reservas, usuarios y operaciones administrativas. | RF08, RF09, RF10, RF11, AC04 | Influye en la autenticación, autorización por rol, validación de permisos y auditoría. |
| DA05 | El sistema debe integrarse con servicios externos como notificaciones, pagos, mapas o inventario futuro. | AC07, RC05, RC06, RC07, RC09 | Influye en el uso de API REST, contratos OpenAPI, puertos y adaptadores. |
| DA06 | El sistema debe permitir modificar funcionalidades sin afectar innecesariamente otros módulos. | AC06 | Influye en la separación de responsabilidades, modularidad y pruebas independientes. |
| DA07 | El sistema debe poder crecer y mantenerse disponible ante mayor carga de usuarios. | AC02, AC03 | Influye en el balanceo, uso de instancias sin estado local indispensable y comparación de capacidad. |
| DA08 | El sistema debe continuar funcionando aunque el servicio de notificaciones falle temporalmente. | AC08, RF07, RF12 | Influye en el uso de trabajos persistentes, reintentos, deduplicación y recuperación ante fallos. |
| DA09 | El sistema debe permitir evidenciar el avance del proyecto y relacionar requisitos, pruebas y decisiones. | RC04, RC08, RC10, AC09 | Influye en el uso de Git, Spec Kit, ADR, pruebas, observabilidad, Docker y evidencias por incremento. |