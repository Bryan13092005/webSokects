# WebSockets Chat (Socket.IO + Express)

Proyecto de chat en **tiempo real** usando **Node.js + Express** para el servidor HTTP y **Socket.IO** para la comunicación bidireccional.

Incluye:
- Registro de usuario (nombre)
- Listado de usuarios conectados (nombre, IP y un ID de conexión)
- Mensajes **globales** (para todos)
- Mensajes **privados** (a un usuario específico)
- Mensajes de sistema al conectar/desconectar

---

## Estructura del proyecto

- **`server.js`**: servidor Express + Socket.IO. Mantiene la lista de usuarios conectados y maneja eventos.
- **`public/index.html`**: interfaz del chat (registro, usuarios, chat global y privado).
- **`package.json`**: dependencias y comandos.

---

## Requisitos

- Node.js instalado (recomendado Node 18+ / 20+)
- NPM (normalmente viene con Node)

---

## Instalación

1. Abre una terminal en la carpeta del proyecto:
   `c:/Users/Usuario/Documents/web/web-sockets`
2. Instala dependencias:

```bash
npm install
```

---

## Cómo ejecutar el proyecto

1. Inicia el servidor:

```bash
node server.js
```

2. Abre el navegador en:

- **http://localhost:3000**

3. Para probar mensajería en tiempo real:
- Abre **la misma URL** en 2 o más pestañas/navegadores.
- Regístrate en cada una con un nombre distinto.
- Envía mensajes globales y privados.

---

## Cómo funciona (flujo de eventos)

### 1) Conexión y registro
Cuando un cliente se conecta:
- Se crea un usuario en memoria con:
  - `id`: el `socket.id`
  - `nombre`: inicialmente `"Sin nombre"`
  - `ip`: `socket.handshake.address`
  - `horaConexion`: hora local
- Se emite `usuarios-conectados` a todos con la lista actual.

Luego el cliente envía:
- **Evento**: `registrar` con el nombre
- El servidor actualiza el nombre del usuario y emite:
  - `usuarios-conectados`
  - `mensaje-sistema` indicando que el usuario se registró

### 2) Mensajes globales
- **Evento (cliente → servidor)**: `mensaje-global` con el texto
- El servidor busca al emisor y emite a todos:
  - `mensaje-global` con `{ de, ip, texto, hora }`

### 3) Mensajes privados
Existen dos manejadores en el servidor, pero la UI del proyecto usa uno:

- **Evento (cliente → servidor)**: `mensaje-privado-ip` con `{ destinoId, texto }`
- El servidor:
  - Busca emisor y destino en la lista de usuarios
  - Envía al destinatario mediante `io.to(destino.id).emit('mensaje-privado', ...)`
  - Envía confirmación al emisor (solo si el destino no es él mismo)

**UI (front-end)**:
- Envía `mensaje-privado-ip` usando el ID seleccionado en el `<select>`.

### 4) Desconexión
Al desconectarse un cliente:
- Se elimina el usuario de la lista
- Se emite `mensaje-sistema` con que ese usuario se desconectó
- Se emite nuevamente `usuarios-conectados`

---

## Interfaz (public/index.html)

La página tiene 4 secciones:

1. **Registrar usuario**
   - Campo: `#nombreInput`
   - Botón: llama a `registrarUsuario()`

2. **Usuarios conectados**
   - Muestra tabla con nombre, IP y un fragmento del ID

3. **Mensaje privado**
   - Selector `#usuarioDestino`
   - Input `#mensajePrivadoInput`
   - Botón: `enviarPrivado()`

4. **Mensaje global**
   - Input `#mensajeGlobalInput`
   - Botón: `enviarGlobal()`

---

## Notas importantes (comportamiento del servidor)

- Los usuarios se almacenan en memoria (arreglo `usuarios`).
  - Si reinicias el servidor, los usuarios se pierden.
- El servidor valida algunos casos para evitar errores:
  - Si no existe el usuario en memoria, ignora el evento.
  - Solo registra nombre si el nombre es real y distinto al actual.

---

## Troubleshooting

- **No carga la página**:
  - Asegúrate de ejecutar `node server.js`
  - Revisa que el puerto `3000` esté libre.

- **No llegan mensajes**:
  - Verifica que varias pestañas estén abiertas apuntando a `http://localhost:3000`
  - Asegúrate de registrar cada pestaña con un nombre.

---

## Personalizaciones rápidas

- Cambiar puerto:
  - En `server.js`, modifica `server.listen(3000, ...)`.
- Cambiar apariencia:
  - Edita el `<style>` dentro de `public/index.html`.
- Añadir nuevos tipos de mensajes:
  - Implementa un nuevo evento en `server.js` y un listener/emisor en el front.

---

## Licencia

Sin licencia específica incluida.

