# Arquitectura inicial del sistema FarmaFind

![Arquitectura inicial de FarmaFind](../../img/arquitectura-inicial-farmafind.png)

## Resumen de la arquitectura

| Elemento | Descripción |
|---|---|
| Actores | Representa a los usuarios principales del sistema: cliente, personal de farmacia, administrador de farmacia y administrador de plataforma. |
| Presentación | Corresponde a la aplicación web desde donde los usuarios interactúan con FarmaFind mediante una API REST. |
| Lógica de negocio | Contiene el núcleo del sistema, organizado en módulos como usuarios, farmacias, medicamentos, búsqueda, inventario, reservas y administración. |
| Datos | Utiliza PostgreSQL para almacenar la información principal del sistema, como usuarios, farmacias, stock y reservas. |
| Servicio separado | Incluye Notificaciones como servicio independiente para procesar avisos relacionados con las reservas. |
| Sistemas externos | Considera integraciones futuras o complementarias como pasarela de pago, mapas e inventario externo. |