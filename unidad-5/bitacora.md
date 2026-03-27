# Unidad 5
## Bitácora de proceso de aprendizaje
#### Actividad 1
MicrobitBinaryAdapter
```
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error {}

function parsePacket(buf) {
  if (buf.length !== 8) throw new ParseError("Invalid packet size");   //si el buffer no mide 8, error

  if (buf[0] !== 0xAA) throw new ParseError("Invalid header");         //si el primer elemento del buffer no es el byte AA, error

  const data = buf.slice(1, 7);                               //extrae los los elementos 2, 3, 4, 5, 6 y 7. Todo excepto el header y el checksum
  const checksum = buf.slice[8];                              //extrae el checksum por su cuenta

  const calc = data.reduce((sum, b) => sum + b, 0) % 256;        //calcula el checksum de nuevo

  if (calc !== checksum) throw new ParseError("Checksum mismatch");  // compara el checksum extraido con el recién calculado y lanza error si no hacen match

  const x = data.readInt16BE(0);                // utiliza el array con los datos extraidos y los guarda por separado
  const y = data.readInt16BE(2);
  const a = data[4];
  const b = data[5];

  return {
    x, 
    y,
    btnA: a === 1,                     //devuelve los datos recién guardados al bridgeserver en el mismo formato que puede juego recibr sketch.js
    btnB: b === 1
  };
}

class MicrobitBinAdapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required");

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open(err => err ? reject(err) : resolve());
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", chunk => this._onChunk(chunk));
    this.port.on("error", err => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close(err => err ? reject(err) : resolve());
      });
    }

    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf = Buffer.concat([this.buf, chunk]); //copia la información llegada a un buffer

    while (this.buf.length >= 8) {    //se asegura de esperar que el buffer tenga un tamaño de al menos ocho
      // Buscar header 0xAA
      const start = this.buf.indexOf(0xAA);         //guarda la posición del primer byte que sea AA

      if (start === -1) {
        this.buf = Buffer.alloc(0);       //Si no encuentra un header, elimina todo el buffer
        return;
      }

      if (this.buf.length < start + 8) break;      //Verifica si hay suficientes bytes para un paquete completo.

      const packet = this.buf.slice(start, start + 8);    //extrae 8 bytes del paquete
      this.buf = this.buf.slice(start + 8);           //Borra el paquete

      try {
        const parsed = parsePacket(packet);       //llama a parsepacket para leerlo
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (this.verbose) console.log("Bad packet:", e.message);    //si recibe un error de parsepacket, envía un error a la consola
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 1024) this.buf = Buffer.alloc(0); //si el buffer se llena demasiado, lo borra
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed (event)");
  }
}

module.exports = MicrobitBinAdapter;
```

Bridgeserver.js
```

//   Uso:
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200

//   WS contract:
//    * bridge To client:
//        {type:"status", state:"ready|connected|disconnected|error", detail:"..."}
//        {type:"microbit", x:int, y:int, btnA:bool, btnB:bool, t:ms}
//    * client To bridge:
//        {cmd:"connect"} | {cmd:"disconnect"}
//        {cmd:"setSimHz", hz:30}
//        {cmd:"setLed", x:2, y:3, value:9}


const { WebSocketServer } = require("ws");
const { SerialPort } = require("serialport");
const SimAdapter = require("./adapters/SimAdapter");
//const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter");
// const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");
const MicrobitAdapterV2 = require("./adapters/MicrobitAdapterV2");
const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter");
const MicrobitBinAdapter = require("./adapters/MicrobitBinaryAdapter");

const log = {
  info: (...args) => console.log(`[${new Date().toISOString()}] [INFO]`, ...args),
  warn: (...args) => console.warn(`[${new Date().toISOString()}] [WARN]`, ...args),
  error: (...args) => console.error(`[${new Date().toISOString()}] [ERROR]`, ...args)
};


function getArg(name, def = null) {
  const i = process.argv.indexOf(`--${name}`);
  if (i >= 0 && i + 1 < process.argv.length) return process.argv[i + 1];
  return def;
}

function hasFlag(name) {
  return process.argv.includes(`--${name}`);
}

function nowMs() { return Date.now(); }

function safeJsonParse(s) {
  try {
    return JSON.parse(s);

  } catch (e) {
    log.warn("Failed to parse JSON: ", s, e);
    return null;
  }
}

function broadcast(wss, obj) {
  const text = JSON.stringify(obj);
  for (const client of wss.clients) {
    if (client.readyState === 1) client.send(text);
  }
}

function status(wss, state, detail = "") {
  broadcast(wss, { type: "status", state, detail, t: nowMs() });
}

const DEVICE = (getArg("device", "sim") || "sim").toLowerCase();
const WS_PORT = parseInt(getArg("wsPort", "8081"), 10);
const SERIAL_PATH = getArg("serialPort", null);
const BAUD = parseInt(getArg("baud", "115200"), 10);
const SIM_HZ = parseInt(getArg("hz", "30"), 10);
const VERBOSE = hasFlag("verbose");

async function findMicrobitPort() {
  const ports = await SerialPort.list();
  const microbit = ports.find(p =>
    p.vendorId && parseInt(p.vendorId, 16) === 0x0D28
  );
  return microbit?.path ?? null;
}

async function createAdapter() {
  if (DEVICE === "microbit") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
    return new MicrobitAsciiAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbit2") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit2 not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit2 found at ${path}`);
    return new MicrobitAdapterV2({ path, baud: BAUD, verbose: VERBOSE });
  }

  if (DEVICE === "microbit-bin") {
     const path = SERIAL_PATH ?? await findMicrobitPort();
     if (!path) {
       log.error("micro:bit not found. Use --serialPort to specify manually.");
       process.exit(1);
     }
     return new MicrobitBinAdapter({ path, baud: BAUD, verbose: VERBOSE });
   }

  return new SimAdapter({ hz: SIM_HZ });
}

async function main() {
  const wss = new WebSocketServer({ port: WS_PORT });
  log.info(`WS listening on ws://127.0.0.1:${WS_PORT} device=${DEVICE}`);

  const adapter = await createAdapter();

  adapter.onConnected = (detail) => {
    log.info(`[ADAPTER] Device Connected: ${detail}`);
    status(wss, "connected", detail);
  };

  adapter.onDisconnected = (detail) => {
    log.warn(`[ADAPTER] Device Disconnected: ${detail}`);
    status(wss, "disconnected", detail);
  };

  adapter.onError = (detail) => {
    log.error(`[ADAPTER] Device Error: ${detail}`);
    status(wss, "error", detail);
  };

  adapter.onData = (d) => {
    broadcast(wss, {
      type: "microbit",
      x: d.x,
      y: d.y,
      btnA: !!d.btnA,
      btnB: !!d.btnB,
      t: nowMs(),
    });
  };

  status(wss, "ready", `bridge up (${DEVICE})`);

  wss.on("connection", (ws, req) => {
    log.info(`[NETWORK] Remote Client connected from ${req.socket.remoteAddress}. Total clients: ${wss.clients.size}`);

    const state = adapter.connected ? "connected" : "ready";

    const detail = adapter.connected
      ? adapter.getConnectionDetail()
      : `bridge (${DEVICE})`;

    ws.send(JSON.stringify({ type: "status", state, detail, t: nowMs() }));

    ws.on("message", async (raw) => {
      const msg = safeJsonParse(raw.toString("utf8"));
      if (!msg) return;

      if (msg.cmd === "connect") {
        log.info(`[NETWORK] Client requested adapter connect`);

        if (adapter.connected) {
          log.info(`[HW-POLICY] Adapter already open. Sending current status to incoming client.`);
          ws.send(JSON.stringify({ type: "status", state: "connected", detail: adapter.getConnectionDetail(), t: nowMs() }));
          return;
        }
        
        try {
          await adapter.connect();
        } catch (e) {
          const detail = `connect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "disconnect") {
        log.info(`[NETWORK] Client requested adapter disconnect`);
        if (wss.clients.size > 1) {
          log.info(`[HW-POLICY] Adapater kept open. Shared with ${wss.clients.size - 1} other active client(s).`);
          ws.send(JSON.stringify({ type: "status", state: "disconnected", detail: "logical disconnect only", t: nowMs() }));
          return;
        }
        
        try {
          await adapter.disconnect();
        } catch (e) {
          const detail = `disconnect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "setSimHz" && adapter instanceof SimAdapter) {
        log.info(`Setting Sim Hz to ${msg.hz}`);
        await adapter.handleCommand(msg);
        status(wss, "connected", `sim hz=${adapter.hz}`);
        return;
      }

      if (msg.cmd === "setLed") {
        try {
          await adapter.handleCommand?.(msg);
        } catch (e) {
          const detail = `command failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }
    });

    ws.on("close", () => {
      log.info(`[NETWORK] Remote Client disconnected. Total clients left: ${wss.clients.size}`);
      if (wss.clients.size === 0) {
        log.info("[HW-POLICY] No more remote clients. Auto-disconnecting adapter device to free resources...");
        adapter.disconnect();
      }
    });
  });

  if (DEVICE === "sim") {
    await adapter.connect();
  }
}

main().catch((e) => {
  log.error("Fatal:", e);
  process.exit(1);
});
```

## Bitácora de aplicación 

### Actividad 2
Adaptador binario.
Igual al adaptador ascii, excepto en las funciones OnChunk y una función nueva que remplaza ParseLine, llamada ParsePacket

<img width="1342" height="755" alt="image" src="https://github.com/user-attachments/assets/a1f79b9b-e40a-4523-ba81-22d0d037785b" />

<img width="1433" height="583" alt="image" src="https://github.com/user-attachments/assets/ea9c46eb-a067-4987-8c9c-22ec094d85d6" />


voy a temporalmente crear un checksum arbitrario para capturar el error en consola
<img width="657" height="157" alt="image" src="https://github.com/user-attachments/assets/61f20310-e204-4313-b77f-c41efda32371" />

Efectivamente, nada es dibujado y la consola registra el error de checksum
<img width="1306" height="726" alt="image" src="https://github.com/user-attachments/assets/d54c4ec7-d4c3-4745-926f-81d3b9c3b815" />


## Bitácora de reflexión
