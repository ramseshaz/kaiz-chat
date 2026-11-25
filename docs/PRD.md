# PRD · <kaiz chat>

> Documento de requisitos del producto (Product Requirements Document).
> Complétalo en el Sprint 0 (historia H0.3) y pide la aprobación del profesor en un Issue.
> Cada requisito funcional (RF) se convierte en un Issue con la plantilla "Historia de usuario".

| Campo | Valor |
| --- | --- |
| Alumno | |
| Proyecto n.º | |
| Sitio de referencia | |
| Compañero revisor | |
| URL desplegada | (se completa en el Sprint 5) |

## 1. Problema

¿Qué problema real resuelve y a quién le duele? 
En grupos de estudio y equipos pequeños, las conversaciones de distintos temas se mezclan en un solo chat desordenado y, cuando la conexión a internet falla de forma intermitente, los mensajes redactados se pierden o no se envían en orden.

## 2. Usuarios

Usuario principal (participante conectado): Persona que inicia sesión desde su computadora o teléfono para conversar en tiempo real en los canales temáticos y recibir notificaciones de mensajes nuevos.

Usuario secundario (participante en movilidad / con red inestable): Persona que accede desde zonas con señal intermitente y necesita redactar mensajes sin conexión para que la cola offline los envíe automáticamente en orden al reconectarse.
## 3. Objetivo y métrica de éxito

Un usuario conectado envía un mensaje en un canal temático y los demás participantes lo reciben en menos de 1 segundo vía WebSocket; si el usuario pierde conexión, el mensaje queda guardado en cola local y se entrega automáticamente en orden al reconectarse sin pérdida de datos.

## 4. Alcance del MVP

1. Registro e inicio de sesión básico de usuarios.
2. Visualización y selección de canales de texto temáticos (estilo Discord: barra lateral de canales y panel central de chat).
3. Envío y lectura de mensajes en tiempo real mediante WebSockets sin recargar la página.
4. Persistencia de historial de mensajes en PostgreSQL ordenado cronológicamente por canal.
5. Cola de mensajes sin conexión (FIFO) en el navegador para redactar offline y sincronizar automáticamente al volver la red.
6. Notificaciones visuales/sonoras y contador de mensajes no leídos por canal.

### Funcionalidades → carpetas de `src/` (screaming architecture)

| Funcionalidad | Carpeta |
| --- | --- |
| Usuarios y sesiones | `src/users` |
| Canales de chat | `src/channels` |
| Mensajes e historial | `src/messages` |
| Conexiones en tiempo real (WebSocket) | `src/websockets` |
| Cola sin conexión y notificaciones | `src/notification


## 5. Fuera de alcance

- Creación de múltiples servidores personalizados ni jerarquías complejas de roles o permisos.
- Llamadas de voz, videollamadas WebRTC o transmisión de pantalla.
- Subida de archivos pesados, imágenes o integración de bots externos.


## 6. Requisitos funcionales
| ID | Requisito | Prioridad (alta, media, baja) |
| --- | --- | --- |
| RF-01 | El usuario puede registrarse e iniciar sesión con su nombre de usuario y contraseña. | Alta |
| RF-02 | El usuario puede navegar entre canales de texto predefinidos y ver el historial de mensajes. | Alta |
| RF-03 | Los mensajes enviados en un canal se transmiten al instante a los usuarios conectados mediante WebSocket. | Alta |
| RF-04 | Si se pierde la conexión WebSocket, los mensajes enviados se guardan en una cola FIFO local y se envían automáticamente al reconectarse. | Alta |
| RF-05 | El sistema muestra un contador de mensajes no leídos en canales inactivos y lanza una notificación del navegador al recibir mensajes nuevos. | Media |

## 7. Requisitos no funcionales

- Responsive, móvil primero (375 px y 1280 px).
- Lighthouse ≥ 90 en Accesibilidad y Buenas prácticas.
- 0 errores en la consola del navegador.
- Funciona con 10.000 registros sin congelarse.

## 8. Modelo de datos

```mermaid
erDiagram
  USERS ||--o{ MESSAGES : envia
  CHANNELS ||--o{ MESSAGES : contiene
  USERS {
    int id PK
    text username UK
    text password_hash
    timestamptz created_at
  }
  CHANNELS {
    int id PK
    text name UK
    text description
    timestamptz created_at
  }
  MESSAGES {
    int id PK
    int channel_id FK
    int user_id FK
    text content
    timestamptz created_at
  }

## 9. Reto técnico distintivo

Chat en tiempo real (WebSocket) + cola de mensajes sin conexión + notificaciones:

- WebSocket: Servidor en Node.js que agrupa usuarios por canal y difunde cada mensaje nuevo en vivo apenas se guarda en PostgreSQL.
- Cola offline (FIFO): Arreglo en el navegador (`localStorage`) que encola mensajes escritos sin red con estado *Pendiente* y los envía en orden al reconectarse.
- Notificaciones: Alertas del navegador (`Notification` API) e insignias numéricas cuando llegan mensajes a canales inactivos.

## 10. Riesgos y preguntas abiertas
- Mensajes duplicados al reconectar: Se evitarán enviando un ID único por mensaje (`client_msg_id`) generado en el cliente.
- Bloqueo de notificaciones del navegador: Si el usuario rechaza el permiso, seguirá viendo el contador visual de mensajes no leídos en la barra de canales.
- Pregunta abierta: ¿Se usarán 3 canales fijos creados desde `db/seed` (`#general`, `#estudio`, `#off-topic`) o se permitirá crear canales nuevos? (Para el MVP se iniciará con canales predefinidos).
