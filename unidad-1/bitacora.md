# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad 1
Los sistemas físicos interactvos son sistemas que conectan un entorno físico con la computación, crean un entorno regido por medios digitales, pero responden a la interacción física del ser humano para crear experiencias únicas y fuera de este mundo que idealmente no pueden ser conseguidas de otra manera, ya sea con fines de entretenimiento o practiidad.

En mis intereses, podría usar esto a la hora de crear una experiencia más inmersiva para mis juegos, yo estoy en proceso de diseñar un juego de cartas coleccionables y utilizar sistemas físicos interactivos para generar una arena que reaccione a las jugadas al estilo Yu-Gi-Oh! crearía una experiencia extraordinaria y generaría mucho más interés por el juego siendo jugado.

----

### Actividad 2
El diseño y arte generativos son modos de crear sistemas y arte no creando directamente un resultado final, pero creando y diseñando el método por el cual se consigue. Se crea un set de reglas, métodos o algoritmos y luego se hace pasar un set de inputs o datos y dichos métodos transforman estos inputs para generar el resultado final.

En mis intereses, podría usar esto a la hora de crear música, por ejemplo una banda sonora, que añada y cambie intrumentos y el ritmo en función de lo que la obra requiera.

### Actividad 3
Voy a tratar de comentar cada linea del código como práctica
```
from microbit import *                           (importar la librería)

uart.init(baudrate=115200)          
display.show(Image.BUTTERFLY)                    (El Micro:Bit mostrará inmediatamente una mariposa)

while True:                                      (Un loop que se repetirá indefinidamente)
    if button_a.is_pressed():                    (si el botón A está oprimido)
        uart.write('A')                          (Enviar una "A" al dispositivo conectado)
        sleep(500)                               (Dormir por medio segundo para evitar que se envíe información muy rápido)
    if button_b.is_pressed():                    (si el botón B está oprimido)
        uart.write('B')                          (Enviar una "B" al dispositivo conectado)
        sleep(500)                               (Dormir por medio segundo para evitar que se envíe información muy rápido)
    if accelerometer.was_gesture('shake'):       (si el acelerómetro detecta el gesto shake)
        uart.write('C')                          (Enviar una "B" al dispositivo conectado)
        sleep(500)                               (Dormir por medio segundo para evitar que se envíe información muy rápido)
    if uart.any():                               (si llega cualquier dato del dispositivo conectado)
        data = uart.read(1)                      (leer el primer dato)
        if data:                                 (Si el dato se puede leer)
            if data[0] == ord('h'):              (Si el dato en la primera posición del array [0] es una "H")
                display.show(Image.HEART)        (Mostrar un corazón en el Micro:Bit)
                sleep(500)                       (dormir)
                display.show(Image.HAPPY)        (mostrar una carita feliz en el Micro:Bit) 
```
-----------------------------------------------------------------------------------------------------------------------------
```
let port;
let connectBtn;

function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    let sendBtn = createButton('Send Love');
    sendBtn.position(220, 300);
    sendBtn.mousePressed(sendBtnClick);
    fill('white');
    ellipse(width / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
            fill('red');
        }
        else if(dataRx == 'B'){
            fill('yellow');
        }
        else{
            fill('green');
        }
        background(220);
        ellipse(width / 2, height / 2, 100, 100);
        fill('black');
        text(dataRx, width / 2, height / 2);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}

function sendBtnClick() {
    port.write('h');
}
```

### Actividad 4
-

## Bitácora de aplicación 

### Actividad 5
```
let port;
let connectBtn;

function setup() {
    createCanvas(400, 400);
    background(220);
    x=width/2
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    fill('white');
    ellipse(width / 2, height / 2, 100, 100);
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
            x=x-10;
        }
        else if(dataRx == 'B'){
           x=x+10;
        }
        else{
            x=x
        }
        background(220);
        ellipse(x, height / 2, 100, 100);
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}
```

## Bitácora de reflexión







