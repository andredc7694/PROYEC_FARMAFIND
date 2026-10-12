# Diseño interno de módulos

> **Proyecto:** FarmaFind  
> **Backend:** Node.js + Express  
> **Módulo de referencia:** Reservas  
> **Estilo arquitectónico:** Monolito modular organizado por capas  
> **Enfoque arquitectónico:** Clean Architecture dentro de cada módulo  
> **Nivel C4:** 4 - Código  
> **Versión:** 1.0

---

## 1. Propósito

Este documento describe cómo se organiza el código dentro de un módulo del backend de FarmaFind. Se toma como referencia el módulo **Reservas**, porque representa uno de los procesos más importantes del sistema: asegurar temporalmente un medicamento disponible para que el cliente pueda recogerlo en una farmacia.

El objetivo de este diseño interno es:

- mantener una estructura común para todos los módulos;
- separar las reglas de negocio de los detalles técnicos;
- facilitar el mantenimiento del sistema;
- permitir que cada integrante del equipo sepa dónde ubicar cada archivo;
- evitar dependencias directas entre el dominio y tecnologías como Express, PostgreSQL o servicios externos.

---

## 2. Estructura de carpetas

~~~text
src/
└── modules/
    └── reservas/
        ├── dominio/
        │   ├── reserva.ts
        │   ├── estado-reserva.ts
        │   ├── reserva-repository.ts
        │   ├── inventario-port.ts
        │   └── notificacion-port.ts
        ├── aplicacion/
        │   ├── crear-reserva.use-case.ts
        │   ├── consultar-reserva.use-case.ts
        │   ├── confirmar-entrega.use-case.ts
        │   └── crear-reserva.command.ts
        ├── infraestructura/
        │   ├── postgres-reserva.repository.ts
        │   ├── postgres-inventario.adapter.ts
        │   └── email-notificacion.adapter.ts
        ├── presentacion/
        │   ├── reservas.controller.ts
        │   └── reservas.routes.ts
        ├── reservas.module.ts
        └── index.ts
~~~

| Archivo | Capa | Responsabilidad | Importa de |
|---|---|---|---|
| `reserva.ts` | Dominio | Representa una reserva y sus reglas principales. | `estado-reserva.ts` |
| `estado-reserva.ts` | Dominio | Define los estados posibles de una reserva. | Nada |
| `reserva-repository.ts` | Dominio | Declara las operaciones necesarias para guardar y consultar reservas. | `reserva.ts` |
| `inventario-port.ts` | Dominio | Declara operaciones para verificar, reservar y liberar stock. | Nada |
| `notificacion-port.ts` | Dominio | Declara operaciones para enviar avisos relacionados con la reserva. | Nada |
| `crear-reserva.use-case.ts` | Aplicación | Coordina la creación de una reserva. | Dominio |
| `consultar-reserva.use-case.ts` | Aplicación | Consulta el estado de una reserva. | Dominio |
| `confirmar-entrega.use-case.ts` | Aplicación | Confirma que el medicamento reservado fue entregado. | Dominio |
| `crear-reserva.command.ts` | Aplicación | Define los datos de entrada para crear una reserva. | Nada |
| `postgres-reserva.repository.ts` | Infraestructura | Guarda y consulta reservas en PostgreSQL. | Dominio + driver de BD |
| `postgres-inventario.adapter.ts` | Infraestructura | Verifica y actualiza stock en PostgreSQL. | Dominio + driver de BD |
| `email-notificacion.adapter.ts` | Infraestructura | Envía notificaciones mediante correo o servicio externo. | Dominio + proveedor externo |
| `reservas.controller.ts` | Presentación | Recibe peticiones HTTP, invoca casos de uso y devuelve respuestas JSON. | Aplicación |
| `reservas.routes.ts` | Presentación | Define las rutas REST del módulo de reservas. | Presentación |
| `reservas.module.ts` | Composición | Crea instancias concretas e inyecta dependencias. | Todas las capas |
| `index.ts` | Composición | Expone la API pública del módulo. | `reservas.module.ts` |

---

## 3. Capas del módulo y regla de dependencia

El módulo se organiza siguiendo Clean Architecture. La regla principal es que las dependencias siempre deben apuntar hacia el dominio.

![Diagrama de paquetes del módulo Reservas](../../../img/paquetes-modulo-reservas.png)

~~~text
Presentación → Aplicación → Dominio
Infraestructura → Dominio
Composición → Todas las capas
~~~

| Capa | Contiene | Responsabilidad | Puede depender de | No puede depender de |
|---|---|---|---|---|
| Dominio | Entidades, estados e interfaces | Define las reglas principales de la reserva. | Nada externo | Express, PostgreSQL, controladores, rutas o adaptadores. |
| Aplicación | Casos de uso y comandos | Coordina los pasos del proceso de reserva. | Dominio | Infraestructura, presentación, Express o PostgreSQL. |
| Infraestructura | Repositorios y adaptadores | Implementa contratos del dominio usando tecnologías concretas. | Dominio y librerías externas | Presentación. |
| Presentación | Controladores y rutas | Traduce peticiones HTTP hacia casos de uso. | Aplicación | Base de datos directamente. |
| Composición | `reservas.module.ts` | Une casos de uso, repositorios y adaptadores. | Todas las capas | No aplica. |

> **Cómo comprobarlo:** si un archivo dentro de `dominio/` importa Express, PostgreSQL, rutas, controladores o adaptadores, la regla de dependencia se rompió.

---

## 4. Diagrama de clases

![Diagrama de clases del módulo Reservas](../../../img/diagrama-clases-reservas.png)
### 4.1 Notación UML utilizada

| Símbolo | Significado | Ejemplo en el módulo Reservas |
|---|---|---|
| Línea continua con flecha abierta | Asociación entre clases o componentes. | `CrearReservaUseCase` usa `ReservaRepository`. |
| Línea discontinua con flecha abierta | Dependencia temporal o uso puntual. | `ReservasController` crea un `CrearReservaCommand`. |
| Línea discontinua con triángulo hueco | Realización de una interfaz. | `PostgresReservaRepository` implementa `ReservaRepository`. |
| Rombo relleno | Composición. | `Reserva` contiene datos propios de la reserva. |
| `«interface»` | Contrato sin implementación concreta. | `ReservaRepository`, `InventarioPort`, `NotificacionPort`. |
| `+` / `-` | Público / privado. | `+ crear()`, `- estado`. |

---

## 5. Flujo de ejecución: crear reserva

![Diagrama de secuencia de crear reserva](../../../img/diagrama-secuencia-crear-reserva.png)

| Paso | Origen → destino | Acción | Capa |
|---|---|---|---|
| 1 | Cliente web → `ReservasController` | Envía solicitud para reservar un medicamento. | Presentación |
| 2 | `ReservasController` → `CrearReservaUseCase` | Ejecuta el caso de uso con los datos recibidos. | Presentación → Aplicación |
| 3 | `CrearReservaUseCase` → `InventarioPort` | Verifica si existe stock disponible. | Aplicación → Dominio |
| 4 | `PostgresInventarioAdapter` → PostgreSQL | Consulta disponibilidad del medicamento en la farmacia. | Infraestructura → Base de datos |
| 5 | `CrearReservaUseCase` → `InventarioPort` | Reserva temporalmente el stock solicitado. | Aplicación → Dominio |
| 6 | `CrearReservaUseCase` → `Reserva` | Crea la reserva con estado pendiente y fecha de expiración. | Aplicación → Dominio |
| 7 | `CrearReservaUseCase` → `ReservaRepository` | Guarda la reserva creada. | Aplicación → Dominio |
| 8 | `PostgresReservaRepository` → PostgreSQL | Inserta la reserva en la base de datos. | Infraestructura → Base de datos |
| 9 | `CrearReservaUseCase` → `NotificacionPort` | Solicita el envío de una notificación al cliente. | Aplicación → Dominio |
| 10 | `EmailNotificacionAdapter` → Servicio externo | Envía correo, SMS o aviso de reserva creada. | Infraestructura → Servicio externo |
| 11 | `ReservasController` → Cliente web | Responde con el identificador y estado de la reserva. | Presentación |

---

## 6. Reglas internas del módulo Reservas

| Código | Regla |
|---|---|
| RN-01 | Una reserva solo puede crearse si existe stock disponible. |
| RN-02 | La cantidad solicitada no puede ser mayor al stock disponible. |
| RN-03 | Toda reserva debe tener una fecha de expiración. |
| RN-04 | Si una reserva expira, el stock reservado debe liberarse. |
| RN-05 | Solo el personal de farmacia puede confirmar la entrega de una reserva. |
| RN-06 | Una reserva atendida no puede cancelarse ni modificarse. |
| RN-07 | El sistema debe evitar reservas duplicadas que superen el stock real. |

---

## 7. Código de referencia

### 7.1 Dominio · entidad e interfaces

~~~ts
// dominio/estado-reserva.ts
export enum EstadoReserva {
  PENDIENTE = 'PENDIENTE',
  ATENDIDA = 'ATENDIDA',
  CANCELADA = 'CANCELADA',
  EXPIRADA = 'EXPIRADA',
}
~~~

~~~ts
// dominio/reserva.ts
import { EstadoReserva } from './estado-reserva';

export class Reserva {
  private constructor(
    readonly id: string,
    readonly clienteId: string,
    readonly medicamentoId: string,
    readonly farmaciaId: string,
    readonly cantidad: number,
    private estado: EstadoReserva,
    readonly fechaExpiracion: Date,
  ) {}

  static crear(
    id: string,
    clienteId: string,
    medicamentoId: string,
    farmaciaId: string,
    cantidad: number,
    fechaExpiracion: Date,
  ): Reserva {
    if (!clienteId) throw new Error('La reserva debe tener un cliente.');
    if (!medicamentoId) throw new Error('La reserva debe tener un medicamento.');
    if (!farmaciaId) throw new Error('La reserva debe tener una farmacia.');
    if (cantidad <= 0) throw new Error('La cantidad debe ser mayor a cero.');

    return new Reserva(
      id,
      clienteId,
      medicamentoId,
      farmaciaId,
      cantidad,
      EstadoReserva.PENDIENTE,
      fechaExpiracion,
    );
  }

  confirmarEntrega(): void {
    if (this.estado !== EstadoReserva.PENDIENTE) {
      throw new Error('Solo una reserva pendiente puede ser atendida.');
    }

    this.estado = EstadoReserva.ATENDIDA;
  }

  expirar(): void {
    if (this.estado === EstadoReserva.ATENDIDA) {
      throw new Error('Una reserva atendida no puede expirar.');
    }

    this.estado = EstadoReserva.EXPIRADA;
  }
}
~~~

~~~ts
// dominio/reserva-repository.ts
import { Reserva } from './reserva';

export interface ReservaRepository {
  guardar(reserva: Reserva): Promise<void>;
  buscarPorId(id: string): Promise<Reserva | null>;
  actualizar(reserva: Reserva): Promise<void>;
}
~~~

~~~ts
// dominio/inventario-port.ts
export interface InventarioPort {
  verificarDisponibilidad(
    medicamentoId: string,
    farmaciaId: string,
    cantidad: number,
  ): Promise<boolean>;

  reservarStock(
    medicamentoId: string,
    farmaciaId: string,
    cantidad: number,
  ): Promise<void>;

  liberarStock(
    medicamentoId: string,
    farmaciaId: string,
    cantidad: number,
  ): Promise<void>;
}
~~~

~~~ts
// dominio/notificacion-port.ts
export interface NotificacionPort {
  enviarReservaCreada(reservaId: string): Promise<void>;
}
~~~

### 7.2 Aplicación · caso de uso

~~~ts
// aplicacion/crear-reserva.command.ts
export interface CrearReservaCommand {
  clienteId: string;
  medicamentoId: string;
  farmaciaId: string;
  cantidad: number;
  fechaExpiracion: Date;
}
~~~

~~~ts
// aplicacion/crear-reserva.use-case.ts
import { Reserva } from '../dominio/reserva';
import { ReservaRepository } from '../dominio/reserva-repository';
import { InventarioPort } from '../dominio/inventario-port';
import { NotificacionPort } from '../dominio/notificacion-port';
import { CrearReservaCommand } from './crear-reserva.command';

export class CrearReservaUseCase {
  constructor(
    private readonly reservaRepository: ReservaRepository,
    private readonly inventarioPort: InventarioPort,
    private readonly notificacionPort: NotificacionPort,
  ) {}

  async ejecutar(command: CrearReservaCommand): Promise<string> {
    const disponible = await this.inventarioPort.verificarDisponibilidad(
      command.medicamentoId,
      command.farmaciaId,
      command.cantidad,
    );

    if (!disponible) {
      throw new Error('No existe stock suficiente para realizar la reserva.');
    }

    await this.inventarioPort.reservarStock(
      command.medicamentoId,
      command.farmaciaId,
      command.cantidad,
    );

    const reserva = Reserva.crear(
      crypto.randomUUID(),
      command.clienteId,
      command.medicamentoId,
      command.farmaciaId,
      command.cantidad,
      command.fechaExpiracion,
    );

    await this.reservaRepository.guardar(reserva);
    await this.notificacionPort.enviarReservaCreada(reserva.id);

    return reserva.id;
  }
}
~~~

### 7.3 Infraestructura · repositorio

~~~ts
// infraestructura/postgres-reserva.repository.ts
import { Pool } from 'pg';
import { ReservaRepository } from '../dominio/reserva-repository';
import { Reserva } from '../dominio/reserva';

export class PostgresReservaRepository implements ReservaRepository {
  constructor(private readonly pool: Pool) {}

  async guardar(reserva: Reserva): Promise<void> {
    await this.pool.query(
      `INSERT INTO reservas (id, cliente_id, medicamento_id, farmacia_id, cantidad, estado, fecha_expiracion)
       VALUES ($1, $2, $3, $4, $5, $6, $7)`,
      [
        reserva.id,
        reserva.clienteId,
        reserva.medicamentoId,
        reserva.farmaciaId,
        reserva.cantidad,
        'PENDIENTE',
        reserva.fechaExpiracion,
      ],
    );
  }

  async buscarPorId(id: string): Promise<Reserva | null> {
    const resultado = await this.pool.query(
      'SELECT * FROM reservas WHERE id = $1',
      [id],
    );

    if (resultado.rowCount === 0) return null;

    // En una implementación real se reconstruye la entidad desde la fila.
    return null;
  }

  async actualizar(reserva: Reserva): Promise<void> {
    await this.pool.query(
      'UPDATE reservas SET estado = $1 WHERE id = $2',
      ['ACTUALIZADA', reserva.id],
    );
  }
}
~~~

### 7.4 Presentación · controlador y rutas

~~~ts
// presentacion/reservas.controller.ts
import { Request, Response } from 'express';
import { CrearReservaUseCase } from '../aplicacion/crear-reserva.use-case';

export class ReservasController {
  constructor(private readonly crearReservaUseCase: CrearReservaUseCase) {}

  async crear(req: Request, res: Response): Promise<void> {
    const reservaId = await this.crearReservaUseCase.ejecutar({
      clienteId: req.body.clienteId,
      medicamentoId: req.body.medicamentoId,
      farmaciaId: req.body.farmaciaId,
      cantidad: req.body.cantidad,
      fechaExpiracion: new Date(req.body.fechaExpiracion),
    });

    res.status(201).json({
      id: reservaId,
      estado: 'PENDIENTE',
    });
  }
}
~~~

~~~ts
// presentacion/reservas.routes.ts
import { Router } from 'express';
import { ReservasController } from './reservas.controller';

export function crearReservasRoutes(controller: ReservasController): Router {
  const router = Router();

  router.post('/reservas', controller.crear.bind(controller));

  return router;
}
~~~

---

## 8. Módulos principales de FarmaFind

| Módulo | Responsabilidad principal |
|---|---|
| Usuarios | Gestiona clientes, personal de farmacia y administradores. |
| Farmacias | Administra farmacias afiliadas, ubicación, estado y datos de contacto. |
| Medicamentos | Gestiona medicamentos, nombre comercial, principio activo y presentación. |
| Inventario | Controla stock disponible, stock reservado y precios por farmacia. |
| Búsqueda | Permite buscar medicamentos y ordenar resultados por disponibilidad, precio o cercanía. |
| Reservas | Permite crear, consultar, cancelar, expirar y confirmar reservas. |
| Notificaciones | Envía avisos relacionados con creación, confirmación o expiración de reservas. |
| Pagos | Gestiona el pago opcional mediante una pasarela externa. |