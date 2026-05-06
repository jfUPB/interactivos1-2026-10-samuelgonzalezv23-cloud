# Unidad 7

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

#### Actividad 2

##### Configuración de Open stage control

Se configuró el envío de datos al puerto 9000

##### Elección de widgets.

Elegí tres widgets sencillos. un Botón para realizar una acción binaria mediante un bool, y un knob y un fader para enviar dos datos continuos dentro de un rango.

##### Estructura de Mensaje 
```
{
  "type": "osc",
  "payload": { "address": "/knob_1", "args": [0.75] }
}
```

##### Integración

bridgeClient.js Recibe el mensaje del WebSocket y lo emite por un callback onData sin conocer la lógica de la FSM.
FSMTask recibe el mensaje del cliente y lo pone en una cola interna (postEvent). En cada frame, despacha los eventos: si es OSC, actualiza configuraciones globales; si es musical, lo envía a la lógica de tiempo.
updateLogic gestiona la Cola Temporal. Calcula cuándo debe ocurrir el evento visual sumando el timestamp del mensaje y lo ordena cronológicamente.
drawRunning no toma decisiones; solo consulta la cola, activa animaciones si el tiempo actual coincide, y dibuja basado en globalConfig.

##### Separación de Responsabilidades

El Adapter convierte el buffer de OSC a un número flotante
Cola permite compensar la latencia de red, ordenando los eventos antes de que lleguen al render.
el renderizado es agnóstico a la red. Si el Bridge se cae, el render sigue funcionando con los últimos datos conocidos en globalConfig.

##### 5. Pruebas de Sincronización y Problemas Solucionados

Problema de Dependencias: El error MODULE_NOT_FOUND se solucionó instalando manualmente osc y ws en el entorno de Node.js.
Error de Referencia (postEvent): Se detectó que el BridgeClient intentaba llamar a la FSM antes de que esta existiera. Se solucionó mediante el uso de callbacks y asegurando que la FSM se inicializara primero en el setup() de p5.js.
Mapeo de Datos: Los controles de OSC llegaban en rango $0.0-1.0$. Se implementó map() para convertirlos a rangos de p5.js ($0-255$ para opacidad, $100-1000$ para tamaños).
Verificación: Se usó console.log(Date.now() - msg.timestamp) para medir el jitter de la red y ajustar la variable LATENCY_CORRECTION.

## Bitácora de reflexión
