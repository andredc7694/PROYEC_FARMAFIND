# Estilo arquitectónico del sistema FarmaFind

![Estilo arquitectónico de FarmaFind](../../img/estilo-arquitectonico-farmafind.png)

## Resumen del estilo arquitectónico

| Elemento | Descripción |
|---|---|
| Estilo principal | Cliente-servidor con monolito modular organizado por capas. |
| Cliente-servidor | Los usuarios acceden al sistema mediante una aplicación web que se comunica con el backend por HTTPS y JSON. |
| Monolito modular | El backend se mantiene como una sola aplicación, pero dividido en módulos como usuarios, farmacias, medicamentos, inventario, reservas y pagos. |
| Capas | Cada módulo separa rutas/controladores, servicios y repositorios. |
| Base de datos | PostgreSQL centraliza la información principal del sistema. |
| Servicios externos | Pagos, mapas, notificaciones e inventario externo futuro se conectan mediante adaptadores. |