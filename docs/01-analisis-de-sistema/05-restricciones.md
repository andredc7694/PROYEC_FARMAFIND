# Restricciones del sistema FarmaFind

| ID | Restricción | Aplicación en FarmaFind |
|---|---|---|
| RC01 | Aplicación web | Acceso mediante navegador; búsqueda pública sin cuenta obligatoria. |
| RC02 | API REST | Contratos documentados con OpenAPI; errores y autorización consistentes. |
| RC03 | Base relacional | PostgreSQL para las operaciones transaccionales del núcleo. |
| RC04 | Control de versiones | Un repositorio Git con código, documentación y especificaciones del mismo proyecto. |
| RC05 | Pago externo | Si se incorpora, se usará un proveedor externo sin almacenar datos sensibles de tarjetas. |
| RC06 | Geolocalización | Mapas o rutas mediante un proveedor externo. Ante fallo, mostrar dirección textual; no bloquear reservas. |
| RC07 | Inventario | Actualización manual en el MVP; puerto para futura API de inventario. Sin prometer sincronización automática todavía. |
| RC08 | SDD con Spec Kit | Obligatoria por el prompt. Antes de cada incremento se elaborarán especificación, clarificaciones, plan y tareas; las pruebas se vincularán a criterios de aceptación. |
| RC09 | Dos servicios | Obligatoria para el laboratorio 10 del sílabo. El núcleo y Notificaciones tendrán responsabilidades y procesos separados, con comunicación verificable. |
| RC10 | Entrega reproducible | Docker y Compose, pipeline básico, observabilidad y evidencia de despliegue real o simulado según el alcance académico. |