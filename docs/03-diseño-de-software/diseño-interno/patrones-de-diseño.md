# Patrones de diseño

## Objetivo

Documentar los patrones de diseño aplicados en el backend de FarmaFind, indicando dónde se usan, qué problema resuelven y con qué tecnología se implementan, para que el equipo programe de forma uniforme.

## Contexto

- **Estilo:** monolito modular con servicio separado de notificaciones.
- **Backend:** Node.js + Express.
- **Enfoque interno:** Clean Architecture.
- **Base:** diagrama de componentes y diseño interno de módulos.

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Se implementa con | Archivos en el código |
|---|---|---|---|---|
| Adapter | Reservas → Notificaciones | El sistema de notificaciones puede tener su propia API o proveedor externo. | API REST, correo, SMS o servicio externo | `notificacion-port.ts`, `email-notificacion.adapter.ts` |
| Adapter | Reservas/Pagos → Pasarela de pago | La pasarela de pago tiene su propia forma de procesar cobros. | SDK o API REST de la pasarela | `pago-port.ts`, `pasarela-pago.adapter.ts` |
| Adapter | Búsqueda/Farmacias → Mapas | El proveedor de mapas tiene su propia API para ubicación o distancia. | API REST del proveedor de mapas | `mapa-port.ts`, `maps.adapter.ts` |
| Repository | Reservas → Base de datos | La lógica de negocio no debe depender directamente de SQL ni de PostgreSQL. | PostgreSQL con librería `pg` u ORM | `reserva-repository.ts`, `postgres-reserva.repository.ts` |
| Repository | Medicamentos e inventario → Base de datos | Permite consultar y actualizar medicamentos, precios y stock sin acoplar el dominio a la base de datos. | PostgreSQL con librería `pg` u ORM | `medicamento-repository.ts`, `inventario-repository.ts`, `postgres-inventario.repository.ts` |
| Dependency Injection | Composición de módulos | Permite entregar al caso de uso las implementaciones concretas sin que el dominio las cree directamente. | Constructores y archivo de módulo | `reservas.module.ts`, `crear-reserva.use-case.ts` |
| DTO / Command | Entrada de casos de uso | Ordena los datos que llegan desde la presentación hacia la aplicación. | Objetos de entrada tipados | `crear-reserva.command.ts`, `actualizar-stock.command.ts` |

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Adapter | Traduce una operación interna del sistema hacia una API externa. Por ejemplo, FarmaFind solicita `enviarReservaCreada(reservaId)` y el adaptador lo convierte en un correo, SMS o llamada HTTP. |
| Repository | El negocio solicita acciones como `guardar(reserva)` o `buscarPorId(id)`, y el repositorio se encarga de ejecutar la consulta en PostgreSQL. |
| Dependency Injection | El caso de uso recibe sus dependencias desde afuera. Así puede trabajar con interfaces como `ReservaRepository` o `NotificacionPort` sin conocer sus implementaciones concretas. |
| DTO / Command | Agrupa los datos necesarios para ejecutar un caso de uso. Por ejemplo, `CrearReservaCommand` contiene cliente, medicamento, farmacia, cantidad y fecha de expiración. |

## Beneficio

- Cambiar el proveedor de notificaciones, pagos o mapas solo afecta al adaptador correspondiente.
- La lógica de negocio no depende directamente de Express, PostgreSQL ni servicios externos.
- Los casos de uso se pueden probar con implementaciones simuladas.
- El código queda organizado por módulos y capas, siguiendo el enfoque de Clean Architecture.
- El equipo puede programar con una estructura uniforme en reservas, inventario, medicamentos y farmacias.

## Patrones previstos para siguientes iteraciones

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Observer | Reservas → Notificaciones | Cuando se necesite reaccionar automáticamente a eventos como reserva creada, reserva expirada o entrega confirmada. |
| Factory | Reservas | Si la creación de reservas requiere más reglas, validaciones o tipos de reserva. |
| Decorator | Búsqueda de medicamentos → Caché | Si se necesita acelerar búsquedas frecuentes usando Redis sin modificar el repositorio principal. |
| Facade | Comunicación entre módulos | Cuando módulos como reservas, inventario y notificaciones crezcan y necesiten una entrada más simple para coordinar operaciones. |
| Strategy | Ordenamiento de resultados | Si los criterios de búsqueda crecen, por ejemplo ordenar por precio, cercanía, disponibilidad o farmacia recomendada. |