# Enfoque arquitectónico del sistema FarmaFind

![Enfoque arquitectónico Clean Architecture de FarmaFind](assets/enfoque-arquitectonico-farmafind.png)

## Resumen del enfoque

| Capa | Descripción |
|---|---|
| Presentación | Contiene los componentes web usados por el cliente, personal de farmacia, administrador de farmacia y administrador de FarmaFind. |
| Aplicación | Agrupa los casos de uso principales: buscar medicamentos, consultar disponibilidad, crear reservas, confirmar entregas y gestionar stock. |
| Dominio | Contiene las entidades principales, reglas de negocio y contratos del sistema. |
| Infraestructura | Implementa los repositorios y adaptadores que conectan el núcleo con la base de datos y servicios externos. |
| Base de datos | PostgreSQL almacena usuarios, farmacias, medicamentos, inventario y reservas. |
| Servicios externos | Incluyen pasarela de pago opcional, servicio de notificaciones y servicio de mapas. |