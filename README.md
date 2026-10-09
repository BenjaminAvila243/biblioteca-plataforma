# biblioteca-plataforma

Levanta el sistema completo de la biblioteca con Docker Compose: RabbitMQ, PostgreSQL, el worker `biblioteca-eventos`, `libros.mjs`, `prestamos.mjs`, el gateway, el BFF y el Angular.

## Cómo levantarlo

Desde la carpeta `biblioteca-plataforma`:

1. Copiar `.env.example` a `.env` y completar las contraseñas.
2. Levantar todo:

```
docker compose up -d --build --wait
```

3. Ver el estado:

```
docker compose ps
```

## Mensaje envenenado

Se corre desde la carpeta `biblioteca-plataforma`, con el sistema levantado (`docker compose up -d --wait`).

Publicar un evento `prestamo.creado` cuyo cuerpo no es JSON:

```
docker compose run --rm --no-deps eventos node herramientas/emitir.mjs 1 --roto
```

Comprobar dónde quedó:

```
docker compose exec rabbit1 rabbitmqctl list_queues name messages
```

Qué se tiene que ver:

- En los logs del worker (`docker compose logs -f eventos`), dos líneas `ERROR ... rechazado hacia la DLQ`: una de `AuditoriaConsumidor` y otra de `NotificacionesConsumidor`.
- En `list_queues`, `auditoria.dlq` y `notificaciones.dlq` con 1 mensaje cada una, y las colas de trabajo en 0.

Para ver el header `x-death` del mensaje muerto sin sacarlo de la cola:

```
docker compose run --rm --no-deps eventos node herramientas/ver-dlq.mjs auditoria.dlq
```