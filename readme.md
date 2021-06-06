´´´
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
Serial.begin(9600);
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
´´´
##FUNCIONAMIENTO
Empezamos declarando el contador de interrupciones con volatile, lo que evitara que se elimine a
causa de optimizaciones del compilador.
Declaramos un contador para ver cuantas interrupciones han ocurrido desde el principio del
programa.
Para la configuración del timer, primero necesitaremos un puntero.
La última variable que tenemos que declarar es del tipo portMUX_TYPE , esta la usaremos para la
sincronización entre el main loop y la ISR.
Dentro de este void tenemos la inicialización de nuestro timer llamando a la función timerBegin . Esta
función recibe como entrada el número del temporizador que queremos usar , el valor del prescaler  e indicamos si el contador cuenta hacia adelante (true) o si lo hace hacia atrás (false).
Ahora usaremos la función timerAttachInterrupt , esta recibe como entrada un puntero al
temporizador, la dirección a la función onTimer que más tarde especificaremos y sirve para manejar
la interrupción y finalmente el valor true para especificar que la interrupción es de tipo edge.
A continuación usaremos la función timerAlarmWrite , esta también recibe como entrada tres valores:
Como primera entrada recibe el puntero al temporizador, como segunda el valor del contador en el
que se tiene que generar la interrupción y finalmente un indicador de si el temporizador se ha de
recargar automáticamente al generar la interrupción.
Finalmente llamamos a la función timerAlarmEnable , en esta pasamos como entrada nuestra variable
de temporizador.
## IMPRESIÓN SERIE

Este programa va a imprimir por pantalla el número de veces que hay una interrupción.
An intrrupt as occurred. Total number: 1
An intrrupt as occurred. Total number: 2
An intrrupt as occurred. Total number: 3
An intrrupt as occurred. Total number: 4
An intrrupt as occurred. Total number: 5
An intrrupt as occurred. Total number: 6
An intrrupt as occurred. Total number: 7
...
Y así indefinidamente hasta que no se pare manualmente.