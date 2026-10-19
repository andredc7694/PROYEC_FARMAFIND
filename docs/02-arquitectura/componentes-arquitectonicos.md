# Componentes arquitectónicos del sistema FarmaFind


![Componentes arquitectónicos de FarmaFind](../../img/componentes-arquitectonicos-farmafind.png)

## Resumen de componentes

| Componente | Descripción |
|---|---|
| Aplicación web | Interfaz desarrollada con React, desde donde clientes, personal de farmacia y administradores interactúan con FarmaFind. Las búsquedas públicas no requieren autenticación. |
| API REST | Desarrollada con Node.js, Express y TypeScript. Expone las funciones del sistema mediante HTTPS/JSON y aplica autenticación, autorización y validación de solicitudes. |
| Módulos de negocio | Agrupan Usuarios, Farmacias, Medicamentos, Búsqueda, Inventario, Reservas y Administración dentro de un monolito modular con Clean Architecture. |
| Base de datos | PostgreSQL almacena usuarios, farmacias, medicamentos, inventario, reservas y eventos Outbox. Las operaciones críticas de Reservas e Inventario utilizan transacciones para garantizar la consistencia. |
| Caché Redis | Componente obligatorio del MVP que almacena temporalmente resultados de búsquedas mediante Cache-Aside. Utiliza TTL e invalidación de datos. Si Redis falla, las búsquedas continúan consultando PostgreSQL. |
| Notificaciones | Servicio independiente del núcleo FarmaFind, encargado de procesar avisos relacionados con reservas. Recibe eventos registrados mediante Transactional Outbox y publicados por HTTPS/REST interno autenticado, aplicando reintentos y deduplicación. |
| Sistemas externos | Incluyen un proveedor de mapas para consultas de ubicación, con dirección textual como alternativa. Los pagos electrónicos y la sincronización automática con inventarios externos se consideran funcionalidades futuras. |
