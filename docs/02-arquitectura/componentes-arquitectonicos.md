# Componentes arquitectónicos del sistema FarmaFind

![Componentes arquitectónicos de FarmaFind](../../img/componentes-arquitectonicos-farmafind.png)

## Resumen de componentes

| Componente | Descripción |
|---|---|
| Aplicación web | Interfaz desde donde interactúan clientes, personal de farmacia y administradores. |
| API REST | Expone las funciones principales del sistema mediante solicitudes HTTPS/JSON. |
| Módulos de negocio | Agrupan usuarios, farmacias, medicamentos, búsqueda, inventario, reservas y administración. |
| Base de datos | PostgreSQL almacena usuarios, farmacias, medicamentos, stock y reservas. |
| Notificaciones | Servicio separado encargado de procesar avisos relacionados con reservas. |
| Sistemas externos | Incluyen pago opcional, mapas e inventario externo futuro. |