# Blog Máquina expendedora

## Descripción detallada de la solución
El controlador de la máquina de café se ha diseñado sobre una arquitectura reactiva y no bloqueante en Arduino Uno. En lugar de utilizar esperas activas (delay()), que congelarían la lectura de sensores y botones, el sistema combina threads, interrupciones hardware, un temporizador Watchdog y una máquina de estados que coordina los tres modos de operación: Arranque, Servicio y Administrador.

### Threads
Se han usado Threads ya que el sistema debe atender múltiples tareas con cadencias distintas al mismo tiempo como por ejemplo que parpadee un LED cada segundo o comprobar pulsaciones largas de un botón. Usar hilos coop permite repartir estas responsabilidades con periodos independientes y sin bloquear el bucle principal.
El ledThread ejecuta el parpadeo de LED1 durante el modo Arranque. Se deshabilita automáticamente al terminar. buttonThread cuenta los ticks que permanece pulsado el botón externo. Distingue entre una pulsación de 2–3 s y una de ≥ 5 s. presenceThread mide la distancia con el sensor de ultrasonidos cuando el sistema está en reposo. Si detecta un usuario a menos de 100 cm, lanza la lectura de temperatura/humedad; en caso contrario mantiene ESPERANDO CLIENTE. tempThread temporiza los 5 segundos (HOLD_TICKS) durante los que se muestra la temperatura y humedad ambiente al detectar un cliente antes de abrir el catálogo de bebidas. joystickThread gestiona la navegación por ejes, el botón del joystick, el progreso de preparación con PWM, el mensaje RETIRE BEBIDA que debe surar 3 segundos y la actualización en tiempo real de las vistas del modo Admin.

### Interrupciones hardware
Se ha utilizado dos interrupciones hardware. Una para el pulsador y otra para el botón del joystick. Al usar attachInterrupt cualquier flanco del los botones queda registrado en ese instante lo que permite una respuesta inmediata sin pérdida de eventos. 

### Watchdog
Si que se ha implementado Watchdog. Al inicio de setup() se ejecuta wdt_disable() para evitar bucles de reinicio tras un reset previo, y una vez inicializados todos los periféricos y el arranque se habilita con wdt_enable(WDTO_4S). Se ha optado por esperar 4 segundos porque al utilizar gran cantidad de sensores y en especial el sensor de infrarrojos para el que se ha usado pulseIn, puede haber un cierto tiempo de espera por lo que poner un Watchdog corto podía implicar un reinicio erróneo de arduino.
