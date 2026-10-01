# Basic Vacuum Cleaner

El objetivo de esta práctica es crear un algoritmo de navegación utilizado por un robot aspirador. La idea principal es que se limpie la mayor cantidad de área posible.

Este área sabremos cual es gracias a un porcentaje que la interfaz de la casa nos ofrece.
<img width="630" height="337" alt="imagen" src="https://github.com/user-attachments/assets/0e046201-d318-4491-a406-4f64cefcbf25" />

El propio enunciado nos ofrece dos formas sobre las cuales empezar a idear nuestro programa.

  Espiral: utilizando la fórmula v=R⋅ω, donde v es la velocidad a la que girará, generada por el radio de giro, 
  y w una velocidad angular. A medida que vaya aumentando el radio, la espiral irá creciendo y, consecuentemente, abarcando más terreno el cual limpiar. 
  En el caso de encontrarse con un obstáculo deberá cambiar su trayectoria y seguir haciendo la espiral en otra dirección.

  Movimiento Dash: la aspiradora avanzará en linea recta hasta encontrar un obstáculo. En ese instante deberá cambiar la dirección de su trayectoria, girando, 
  y seguir en línea recta hasta encontrar el siguiente obstáculo. De este modo, irá limpiando de una forma más aleatoria los espacios de la casa.

Para ambos casos se aconseja la idea de usar ángulos aleatorios, es decir, dado un ángulo de giro fijo, 
dejar que el robot gire un tiempo aleatorio. Así cada ángulo generado será diferente a los demás, evitando que entre en un bucle.

Problemas:

  El robot sólo cuenta con un láser, el cual utilizaremos para evitar los obstáculos.

  No posee un mapa mediante el cual puede saber su posición, lo que ha limpiado y lo que le falta.

  La decisión de cuanto girar es completamente aleatoria, eficaz para evitar bucles; pero, a su vez, en cada ejecución será diferente.

Para realizar mi práctica, inicialmente seguí el algoritmo Dash, en donde fijo una dirección para que avance. Observé que era un programa demasiado básico y no cumplía bien con sus funciones. Conseguí que como máximo limpiase un 22% de la casa.

Utilizar únicamente un algoritmo de espiral tampoco era factible ya que muchas veces volvía a empezar el trabajo en una zona parcialmente limpia. Aunque tuvo mejores resultados que el primero, solo limpión el 30%, un porcentaje todavía bastante bajo.

Al final la mejor opción ha sido implementar una mezcla de ambos, donde pueda avanzar, retroceder si se choca, girar, etc. Usando para ello una máquina de estados, además de contabilizar el tiempo en vez de dejarlo dormido para evitar parar el ciclo.

Gracias al láser, evalúo la distancia de los obstáculos y, dependiendo de esta, decido si en cada iteración seguir avanzando o girar.

  El robot dispone de un sensor láser que realiza 180 mediciones correspondientes a diferentes ángulos entre 0° y 180°. 
  Cada medición representa la distancia hasta el obstáculo detectado en esa dirección. La medición situada en el índice 90 corresponde a la dirección frontal del robot.
  <img width="627" height="211" alt="imagen" src="https://github.com/user-attachments/assets/8982f2be-cbe0-4f8e-b5de-ddb4f5c7774a" />

Implementación:

ESPIRAL: El robot ejecuta una trayectoria en espiral incrementando progresivamente la velocidad lineal, manteniendo una velocidad angular constante.

RECTA: Movimiento rectilíneo manteniendo.

PARADA: Al detectar un obstáculo, el robot detiene completamente sus motores.

RETROCESO: Aplica una velocidad lineal negativa 

GIRO: Aplica un giro sobre su propio eje durante un intervalo aleatorio de tiempo.

REAJUSTE: Tras completar el giro, el robot efectúa una breve pausa, reinicia los parámetros de velocidad iniciales.



Resultado: aunque el algoritmo permite recorrer una parte importante del entorno, no garantiza una cobertura completa, ya que se basa en reacciones. 
El robot no mantiene información explícita de las zonas que ya ha visitado, por lo que puede pasar varias veces por una misma zona mientras deja otras sin explorar.



