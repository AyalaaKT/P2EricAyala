# P2EricAyala

# PR2_PD_EricAyala

## PRACTICA 2: INTERRUPCIONES ⏸️

**PROGRAMA:**

```cpp
#include <Arduino.h>

int LED_BUILTIN = 23;
int DELAY = 500;
int count = 0;

void IRAM_ATTR isr() {
  Serial.println("Interrupted");
  count = 0;
}

void setup() {
  pinMode (LED_BUILTIN, INPUT_PULLUP);
  Serial.begin(115200);
  attachInterrupt(LED_BUILTIN, isr, FALLING);
}

void loop () {
for (count; count < 1000; count++){
  Serial.println(count);
  delay(DELAY);
}
}
```

En lugar de hacerlo por boton, como indicaba el enunciado, en este caso al on disponer de botones a nuestro alcance hemos decidido crear este código, el cual inicia un contador de 0 hasta 1000, el cúal se reiniciará cada vez que en nuestro circuito llevemos el cable conectado a nuestro led, en este caso el 23, a tierra.

Para esto hemos utilizado la funcion `attachInterrupt(LED_BUILTIN, isr, FALLING),`, Está función toma 3 parámetros:

- LED_BUILTIN: le indica a la ESP32 que patilla debe monitorear, en este caso el led "23".

- ISR: es el nombre de la funcion que se llamará cada vez que se disparé la interrupción, en este caso nuestra función escribe por el serial el mensaje "interrupted", y además inicializará el contador a 0 (count = 0).

- Falling: define cuando se debe disparar la interrupción. Son 5 constantes que están predefinidas como valores válidos, en este caso hemos usado `falling`, lo que indica que se realizará cuando el cable conecte a tierra. Los valores válidos son:

    - LOW: los disparados interrumpen cuando el pin está en LOW
    - HIGH : los disparadores interrumpen cuando el pin está en HIGH
    - CHANGE : los disparadores interrumpen cuando el pin cambiad de valor, de HIGH a LOW, o de LOW a HIGH
    - FALLING : los disparadores interrumpen cuando el pin va de HIGH a LOW
    - RISING : los disparadores interrumpen cuando el pin va de LOW a HIGH


### Practica B interrupción por TIMER

**PROGRAMA:**

``` cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
 portENTER_CRITICAL_ISR(&timerMux);
 interruptCounter++;
 portEXIT_CRITICAL_ISR(&timerMux);
}
void setup() {

 Serial.begin(115200);
 
 timer = timerBegin(0, 80, true);
 timerAttachInterrupt(timer, &onTimer, true);
 timerAlarmWrite(timer, 1000000, true);
 timerAlarmEnable(timer);
}
void loop() {
 if (interruptCounter > 0) {
  portENTER_CRITICAL(&timerMux);
  interruptCounter--;
  portEXIT_CRITICAL(&timerMux);
  
  totalInterruptCounter++;
  
  Serial.print("An interrupt as occurred. Total number: ");
  Serial.println(totalInterruptCounter);
 }
}
```
 Ahora usamos interrupciones de tipo temporizador, en lugar de GPIO.

Este programa utiliza interrupciones por timer en un microcontrolador. Cuando se ejecuta, inicia un contador que aumenta cada vez que ocurre una interrupción de timer, la interrupción se dispara a intervalos regulares definidos por `timerAlarmWrite()`. 

La función `timerAlarmWrite()` toma tres argumentos:
- Timer: el temporizador que se está utilizando. En este caso, se refiere al temporizador configurado anteriormente.

- Period: el valor de comparación, es decir, el valor que el contador del temporizador comparará con el valor actual del temporizador. Cuando estos valores sean iguales, se generará una interrupción. En el ejemplo proporcionado, este valor es de 1000000, lo que podría representar un intervalo de tiempo en microsegundos.

- Autoreload: un booleano que indica si el temporizador debe recargarse automáticamente después de generar una interrupción. Si se establece en true, el temporizador se reiniciará automáticamente después de cada interrupción. En el código dado, se ha establecido en true.

Cuando el contador de interrupción es mayor que cero, el programa registra el evento de interrupción y actualiza el contador total de interrupciones. Esto permite un seguimiento del número total de interrupciones ocurridas.

Finalmente, podemos ver que en el fondo, en este programa seguimos usando las mismas funciones que en el primer ejemplo, pero en lugar de ser con GPIO, son temporales:`(timerAttachInterrupt, timerAlarmWrite, timerAlarmEnable, etc)`.
