# Escenarios de atributos de calidad del sistema FarmaFind

## EQ-01. Rendimiento: búsqueda de medicamentos

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Rendimiento |
| Fuente | Cliente |
| Estímulo | El cliente busca medicamentos por nombre comercial o principio activo. |
| Condición | El sistema funciona con la carga definida y datos representativos. |
| Respuesta esperada | El sistema muestra las farmacias con disponibilidad, precio y ubicación sin errores inesperados. |
| Medida | El 95 % de las búsquedas debe responder en 2 segundos o menos, con errores técnicos menores al 1 %. |
| Verificación | Ejecutar pruebas de carga y registrar p95, p99, throughput, errores y uso de recursos. |
| Estado | Pendiente de validación durante las pruebas del proyecto. |

## EQ-02. Disponibilidad: continuidad del servicio

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Disponibilidad |
| Fuente | Usuario del sistema |
| Estímulo | Una instancia del núcleo deja de funcionar. |
| Condición | El sistema cuenta con dos instancias del núcleo en ejecución. |
| Respuesta esperada | El servicio continúa funcionando mediante la instancia sana. |
| Medida | El servicio debe reanudarse en 30 segundos o menos. |
| Verificación | Detener una instancia y comprobar que la otra atiende las solicitudes. |
| Estado | La base de datos única queda declarada como punto de fallo. |

## EQ-03. Escalabilidad: aumento de carga

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Escalabilidad |
| Fuente | Crecimiento de usuarios |
| Estímulo | Aumenta la cantidad de búsquedas y reservas en el sistema. |
| Condición | Se comparan una y dos instancias con el mismo conjunto de datos y recursos de base de datos. |
| Respuesta esperada | El sistema debe aumentar su capacidad antes de superar los límites de rendimiento definidos. |
| Medida | Comparar capacidad, latencia y errores antes de superar el escenario EQ-01. |
| Verificación | Ejecutar pruebas con carga creciente y documentar el cuello de botella. |
| Estado | Pendiente de validación con pruebas de carga. |

## EQ-04. Seguridad: acceso protegido

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Seguridad |
| Fuente | Usuario autenticado o no autorizado |
| Estímulo | Un usuario intenta acceder a reservas ajenas o administrar otra farmacia. |
| Condición | El sistema tiene usuarios con roles y permisos definidos. |
| Respuesta esperada | El sistema rechaza el acceso no autorizado. |
| Medida | El 100 % de intentos no autorizados debe ser rechazado. |
| Verificación | Probar accesos indebidos a reservas, farmacias y funciones administrativas. |
| Estado | Pendiente de pruebas de autorización y revisión de logs. |

## EQ-05. Consistencia: reservas concurrentes

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Consistencia |
| Fuente | Clientes concurrentes |
| Estímulo | Varios clientes intentan reservar el mismo medicamento al mismo tiempo. |
| Condición | Existen 10 unidades disponibles y se realizan 100 intentos simultáneos de reserva. |
| Respuesta esperada | El sistema acepta solo las reservas que correspondan al stock real. |
| Medida | Deben aceptarse exactamente 10 reservas distintas, sin stock negativo ni duplicados. |
| Verificación | Ejecutar prueba concurrente sobre una unidad limitada de inventario. |
| Estado | Pendiente de validación con transacciones de base de datos. |

## EQ-06. Mantenibilidad: cambio de adaptador

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Mantenibilidad |
| Fuente | Equipo de desarrollo |
| Estímulo | Se reemplaza el adaptador de notificaciones por uno simulado. |
| Condición | El dominio y los casos de uso ya están definidos. |
| Respuesta esperada | El cambio se realiza sin modificar dominio ni casos de uso. |
| Medida | El reemplazo debe afectar solo la infraestructura o adaptador correspondiente. |
| Verificación | Sustituir el adaptador y ejecutar pruebas de los componentes relacionados. |
| Estado | Pendiente de implementación y pruebas. |

## EQ-07. Interoperabilidad: contrato con notificaciones

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Interoperabilidad |
| Fuente | Núcleo FarmaFind |
| Estímulo | El núcleo envía un evento al servicio de Notificaciones. |
| Condición | Existe un contrato definido entre el núcleo y el servicio de Notificaciones. |
| Respuesta esperada | El servicio consumidor registra un único trabajo por identificador. |
| Medida | Si se repite el evento, no debe generarse un trabajo duplicado. |
| Verificación | Validar el contrato y repetir un evento con el mismo identificador. |
| Estado | Pendiente de validación del contrato. |

## EQ-08. Resiliencia: falla de notificaciones

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Resiliencia |
| Fuente | Falla temporal del servicio de Notificaciones |
| Estímulo | El servicio de Notificaciones se interrumpe durante 5 minutos. |
| Condición | El núcleo de reservas continúa funcionando. |
| Respuesta esperada | Las reservas continúan y los avisos pendientes quedan persistidos. |
| Medida | Al recuperar Notificaciones, los trabajos pendientes deben procesarse sin pérdida. |
| Verificación | Interrumpir Notificaciones, generar reservas y medir el retraso al recuperar el servicio. |
| Estado | Pendiente de prueba de recuperación. |

## EQ-09. Observabilidad: seguimiento de pruebas

| Elemento | Descripción |
|---|---|
| Atributo de calidad | Observabilidad y pruebas |
| Fuente | Equipo de desarrollo y pruebas |
| Estímulo | Se ejecutan pruebas de reservas, búsquedas y notificaciones. |
| Condición | El sistema cuenta con logs, métricas y correlación de eventos. |
| Respuesta esperada | Cada reserva de prueba puede relacionarse con registros y métricas del sistema. |
| Medida | Registrar latencia, errores, CPU, memoria, conexiones y trabajos pendientes. |
| Verificación | Revisar logs y métricas después de cada prueba definida por criterios de aceptación. |
| Estado | Pendiente de configuración de observabilidad. |