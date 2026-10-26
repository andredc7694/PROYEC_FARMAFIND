# Enfoque arquitectónico del sistema FarmaFind


![Enfoque arquitectónico Clean Architecture de FarmaFind](../../img/enfoque-arquitectonico-farmafind.png)

> **Estado:** diseño propuesto, pendiente de implementación y validación.

## Resumen del enfoque

| Capa | Descripción |
|---|---|
| Presentación | Contiene la aplicación web propuesta con React + Vite + JavaScript/JSX y los controladores de la API REST. Permite que clientes, personal de farmacia y administradores interactúen con FarmaFind. Las búsquedas públicas no requieren autenticación. |
| Aplicación | Agrupa los casos de uso principales: buscar medicamentos, consultar disponibilidad, crear y cancelar reservas, confirmar reservas y entregas, y gestionar inventario mediante registro manual e importación obligatoria .xlsx/.csv con vista previa, validación y mapeo al catálogo general. Coordina las operaciones mediante contratos, incluyendo Cache-Aside para búsquedas y Unit of Work para transacciones de reservas. |
| Dominio | Contiene las entidades, reglas de negocio y contratos del sistema. Define los estados de reserva PENDIENTE, CONFIRMADA, ENTREGADA, CANCELADA y EXPIRADA, así como disponibilidad referencial, comprobación física para aceptar por dueño, administrador o personal expresamente delegado, y conciliación sin sobrescribir compromisos. Los plazos serán configurables: respuesta PENDIENTE de 20 minutos iniciales y recogida CONFIRMADA de 24 horas propuestas, sujetas a validación. No depende directamente de tecnologías externas. |
| Infraestructura | Implementa los repositorios y adaptadores que conectan los casos de uso con PostgreSQL, Redis y servicios externos. Incluye el adaptador de caché Redis y los mecanismos de registro y publicación de eventos mediante Transactional Outbox. |
| Base de datos | PostgreSQL almacena usuarios, farmacias, medicamentos, inventario, movimientos_inventario, reservas y eventos Outbox. Es la fuente oficial de información y permite garantizar la consistencia mediante transacciones. |
| Caché | Redis es obligatorio en el MVP para almacenar temporalmente resultados de búsqueda mediante Cache-Aside. Utiliza TTL e invalidación de datos relevantes. Si Redis falla, las búsquedas continúan consultando PostgreSQL. |
| Servicio independiente | Notificaciones funciona como un servicio separado del núcleo FarmaFind. Recibe eventos registrados mediante Transactional Outbox, procesará avisos de reservas, alertas de corrección y recordatorios propuestos por inventario antiguo sin alterar stock y aplica reintentos y deduplicación. |
| Servicios externos | Incluyen el proveedor de mapas para consultar ubicaciones, con dirección textual como alternativa ante fallos. Los pagos electrónicos y la sincronización automática con inventarios externos quedan previstos para futuras iteraciones. |

