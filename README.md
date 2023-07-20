# express-chokidar-livereload
## Server-Side:
I used this modules:

```
npm i express ws chokidar
```

The code was:
```ts
import express from 'express';
import http from 'http';
import { WebSocket } from 'ws';
import path from 'path';
import chokidar from 'chokidar';

const app = express();
const server = http.createServer(app);
const ws = new WebSocket.Server({ server });

ws.on('connection', _ => {});
chokidar
  .watch(__dirname, { persistent: true })
  .on('change', filepath => {
    if (filepath.extname === '.ejs') {
      ws.clients.forEach(c => c.send("reload"))
    }
  })
  
server.listen(80, _ => console.log(`server listening on: http://localhost:80`));
```

What this code do, is when any file with the extension .ejs is updated a websocket server send to the websocket clients a message saying `reload`, then...

## Client-Side:

I added this script tag inside the head:
```html
<script>
  const wsclient = new WebSocket(window.location.origin.replace("http://", "ws://"));
  
  wsclient.addEventListener("message", msg => {
    if (msg.data === "reload") {
      window.location.href=window.location.href;
    }
  });

  wsclient.addEventListener("close", _ => {
    setTimeout(_ => window.location.href=window.location.href, 1000)
  });
</script>
```

What that script do is, 
1. connect to the ws socket server, and get a client.
2. add event listener for messages, if the message says reload, reloads the page.
3. add event listener for web socket server to detect when the server disconnect when that happens it add a timeout of 1 second and try to reload the page.

In summary, what the thing does is:

1. Create an ExpressJS server and a WebSocket using the "express", "http", and "ws" modules.
2. Create a WebSocket.Server object to manage incoming WebSocket connections.
3. Use the "chokidar" module to observe changes in the current directory and its subdirectories.
4. If a change is detected in a file with the extension ".ejs", a "reload" message is sent to all connected WebSocket clients.
5. On the client-side, establish a WebSocket connection with the server and add an event handler to receive "reload" messages.
6. If a "reload" message is received, reload the current page.
7. Additionally, add an event handler to detect when the WebSocket server disconnects, and in that case, attempt to reload the page after a brief delay.
