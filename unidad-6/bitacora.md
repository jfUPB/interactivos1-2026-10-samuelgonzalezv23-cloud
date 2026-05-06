# Unidad 6

## Bitácora de proceso de aprendizaje

#### Actividad 1

##### Diferencia entre recibir y ejecutar mensaje. 

##### Razones para timestam. Está en la palabra, es un sistema audiovisual. El audio y el video son dos componentes muy diferentes regidos por distinta lógica que deben estar al unísono, el timestamp le dice a ambos sistemas cuando deben mostrar sus actualizaciones para no perder la sincronización.

##### La clase maestra de los adapters, el bridge ahora solo tiene que cambiar ligeramente para acomodar el nuevo adaptador, el cliente y las visuales ahora tienen que recibir un tipo de dato nuevo.

## Bitácora de aplicación 

##### configuración de emisión de eventos strudel. 

```
setcps(0.5)
const pat = s("bd hh sd hh bd*2 hh [sd bd] hh").bank("tr909")

$: stack(
  pat.gain('0.5'),
  pat.osc()
)
```
Tengo mi patrón definido como pat y al final del código está la función pat.osc(), esto hace que cada vez que pat reproduzca un sonido, envíe cada mensaje también en el formato de Open Stage Control al puerto 8080 por defecto, donde pueden ser leidos

##### Estructura elegida.

```
timestamp: msg.timestamp, 
        sound: msg.sound,
        delta: msg.delta || 0.25,
        params: msg.params
```
El paquete de datos llea con un timestamp, el código del sonido según su banco, un delta y las otras cosas. El bridge lo dota de un tipo "Osc" para procesar luego, pero esto es lo que envía strudel.



## Bitácora de reflexión
