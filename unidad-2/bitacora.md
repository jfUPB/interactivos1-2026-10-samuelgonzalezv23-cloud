# Unidad 2

## Bitácora de proceso de aprendizaje

### Actividad 1
Los estados de ese código en partícular son: 
-WaitInOn
-WaitInOff



## Bitácora de aplicación 
<img width="255" height="633" alt="image" src="https://github.com/user-attachments/assets/f7d3a569-1e0a-4727-8eeb-428340c02b3a" />

```
from microbit import *
import utime



class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

class Reloj:
    def __init__(self):
        self.event_queue = [] #crea la cola de eventos
        self.timers = []      #Crea la cola de timers
        
        self.segundos = 20    #tiempo inicial
        self.min_seg = 15     #min
        self.max_seg = 25     #max

        
        self.myTimer = self.createTimer("Timeout",1000)    #Crea un timer que manda timeout y puedo iniciarlo cuando quiera
        self.estado_actual = None
        self.transicion_a(self.estado_setup)           #voy al estado setup
        
    def createTimer(self,event,duration):   #crea un timer con los argumentos "evento" y "duración"
        t = Timer(self, event, duration)
        self.timers.append(t)                #Guarda el timer en una lista
        return t                          #Devuelve el timer creado llamado t al owner del timer

    def post_event(self, ev):                        #Crea unna función para postear evento con los argumentos dueño (siempre self) y evento
        self.event_queue.append(ev)                #añade el evento a la lista

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def dibujar_pixels(self):    #función para dibujar los pixeles
        display.clear()            #limpia los pixeles
        total = self.segundos       

        for i in range(total):
            x = i % 5      #operación Módulo 5
            y = i // 5      #división entera sobre 5
            display.set_pixel(x, y, 9)

    def apagar_siguiente(self):
        total = self.segundos
        index = total - 1

        x = index % 5
        y = index // 5
        display.set_pixel(x, y, 0)

        self.segundos -= 1



    def estado_setup(self, ev):
        if ev == "ENTRY":
            self.segundos = 20
            self.dibujar_pixels()

        if ev == "A":
            if self.segundos < self.max_seg:
                self.segundos += 1
                self.dibujar_pixels()

        if ev == "B":
            if self.segundos > self.min_seg:
                self.segundos -= 1
                self.dibujar_pixels()

        if ev == "S":
            self.transicion_a(self.estado_contando)

    def estado_contando(self, ev):
        if ev == "ENTRY":
            self.myTimer.start(1000)

        if ev == "Timeout":
            if self.segundos > 0:
                self.apagar_siguiente()
                self.myTimer.start(1000)
            else:
                self.myTimer.stop()
                self.transicion_a(self.estado_muerto)
            
    def estado_muerto(self, ev):
        if ev == "ENTRY":
                display.show(Image.SKULL) 
        if ev == "A":
           self.transicion_a(self.estado_setup)
        
            
    
reloj=Reloj()

while True:
    # Aquí generas los eventos de los botones y el gesto
    if button_a.was_pressed():
        reloj.post_event("A")
    if button_b.was_pressed():
        reloj.post_event("B")
    if accelerometer.was_gesture("shake"):
        reloj.post_event("S")
    reloj.update()
    utime.sleep_ms(20)


```


## Bitácora de reflexión


