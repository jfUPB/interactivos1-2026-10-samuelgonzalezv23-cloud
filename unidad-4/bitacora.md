# Unidad 4

## Bitácora de proceso de aprendizaje

Abriendo gitbash en mi repositorio, una vez con node.js instalado:

npm install             -instalar las dependencias

node bridgeserver.js         -abrir el server

node bridgeserver.js --device microbit       -abrir el server y buscar el dispositivo microbit

## Bitácora de aplicación 

Recibí un programa con una arquitectura que no podía modificar, mi trabajo era crear un adaptador para un dispositivo cuyo código tampoco podía modifcar, dispositivo que enviaría la información en el siguiente formato: 
```
$T:tiempo|X:acel_x|Y:acel_y|A:estado_a|B:estado_b|CHK:checksum\n
```
cada línea comienza con $

Así que escribí código para el micro:bit para que envíe los datos en el formato

```
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()

    aState = 1 if button_a.is_pressed() else 0
    bState = 1 if button_b.is_pressed() else 0

    t = running_time()

    chk = abs(xValue) + abs(yValue) + aState + bState

    data = "$T:{}|X:{}|Y:{}|A:{}|B:{}|CHK:{}\n".format(
        t, xValue, yValue, aState, bState, chk
    )

    uart.write(data)

    sleep(100)  # 10 Hz
```

Luego, cree un nuevo archivo llamado MicrobitAdapterV2 que hereda del adaptador base.

<img width="367" height="513" alt="image" src="https://github.com/user-attachments/assets/2a4fe11e-d115-41bc-8da0-ed34e272b8e0" />

Solo que en lugar de utilizar la función parseCsvLine, utiliza una función nueva llamada parseLine
<img width="1283" height="467" alt="image" src="https://github.com/user-attachments/assets/beb1049d-2a73-4bc9-9a1a-541516c580c5" />

```
function parseLine(line) {

  if (!line.startsWith("$"))
    throw new ParseError("Line must start with $");

  const parts = line.slice(1).split("|");

  const data = {};

  for (let p of parts) {
    const [key, value] = p.split(":");
    data[key] = value;
  }

  const x = Number(data.X);
  const y = Number(data.Y);
  const a = Number(data.A);
  const b = Number(data.B);
  const chk = Number(data.CHK);

  if (!Number.isFinite(x) || !Number.isFinite(y))
    throw new ParseError("Invalid numeric data");

  if (x < -2048 || x > 2047 || y < -2048 || y > 2047)
    throw new ParseError("Out of expected range");

  if (![0,1].includes(a) || ![0,1].includes(b))
    throw new ParseError("Invalid button data");

  const calc = Math.abs(x) + Math.abs(y) + a + b;

  if (calc !== chk)
    throw new ParseError("Checksum mismatch");

  return {
    x: x | 0,
    y: y | 0,
    btnA: a === 1,
    btnB: b === 1
  };
}
```
Al final del adaptador debo hacer que se exporte a si mismo:
<img width="785" height="342" alt="image" src="https://github.com/user-attachments/assets/ebe5de60-2261-4a09-8e49-079e2b9a395a" />

Y en bridgeserver.js, debo hacer que la función createAdapter cree el adaptador que acabo de crear:
<img width="882" height="343" alt="image" src="https://github.com/user-attachments/assets/38b3c1a6-30a2-4574-b15a-8f55266197b3" />






## Bitácora de reflexión
