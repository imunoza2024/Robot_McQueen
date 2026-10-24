## Obstacle Detection
**Objetivo:** implementar un algoritmo de navegación VFF que guie al coche por todo el circuito sin chocar con los demás coches y que consiga completarlo. 

<img width="427" height="773" alt="Captura desde 2026-10-22 21-15-50" src="https://github.com/user-attachments/assets/0fd275ea-4638-4b61-8f40-63e974f4ecc8" />


Esto implica crear:
- Un vector atractivo desde el coche a los waypoints por los que tiene que ir pasando a lo largo de su recorrido

- Un vector repulsivo de los obstáculos hacia el coche, para evitar que se choque

  **Detalles a tener en cuenta**
  
  Se necesita obtener la posición del coche en todo momento, la de las coordenadas del siguiente objetivo y tambien la de los obstaculos para crear esos vectores de fuerza. Además contamos con la ayuda del láser el cual nos proporciona 180 mediciones.
  
  Una vez obtenidos ambos vectores hay que combinarlos para obtener la fuerza resultante que será la dirección mediante la cual el coche podrá llegar al objetivo evitando por el camino chocar con los obstáculos.

  No en todas las partes del circuito hay el mismo número de obstáculos ni estos están situados a la misma distancia. Por ello se puede usar una velocidad adaptativa que frene o acelere dependiendo de la situación, tanto lineal como angular; y unos pesos más altos cuando el láser detecta objetos de frente y más bajos cuando estos son visualizados por los extremos, evitando así mucha oscuilación.

  No hay que olvidarse de marcar cada objetivo como completado una vez que se supere.
  
  En la simulación el eje en x positivo esta hacia delante y el y hacia la izquierda, además el origen es la posición del coche.
  
Se pretende conseguir una navegación autónoma que combine el seguimiento de objetivos con la evitación local de obstáculos.

La navegación conlleva todas las actividades internas que hay que tener en cuenta para llevar el coche de un punto A -> B de forma autónoma sin que choque. 

- La navegación global crea una ruta desde el inicio hasta el destino utilizando el entor, mapas, etc.
  
- La navegación local divide la ruta de la navegación global en objetivos más pequeños donde determinar cómo debe moverse el robot en cada instante teniendo en cuenta su posición actual, el siguiente punto de paso y los obstáculos detectados en su entorno inmediato.

  **Implementación**

  Utilizo las funciones absolute2relative y parse_laser_data, proporcionadas en el enunciado, para transformar las coordenadas absolutas del objetivo a coordenadas relativas para el coche; y transformar las medidas del sensor láser en coordenadas cartesianas y polares que son las que utilizaremos para hacer los cálculos posteriores.

  En el bucle primcipal se toman las coordenadas del siguiente objetivo, las cuales con la función absolute2relative tranformaremos a relativas, y las coordenadas del coche en este instante de tiempo.

  Cuando el objetivo esté lo suficientemente cerca se marcará como completado y el coche se pondrá rumbo al siguiente waypoint. Esto se sabe ya que calculamos la distancia gracias a la transformación de las coordenadas del objetivo en relativas porque el centro de referencia es el coche así que con Pitágoras sabremos cómo de lejos está.

  Después se procesan las medidas del láser las cuales mandaremos a la función parse_laser_data para obtener coordenadas que podamos manejar.

  El láser toma 180 mediciones de las cuales obtiene la distancia en cada una de ellas. Podrían darse varios casos:
  - Distacias negativas
    
  - Distancias infinatas
    
  - Distancias finitas
  Las que tenemos en cuenta son las finitas ya que nos advierten de un próximo obstáculo, y el resto se descartan por ser inválidas para los calculos de las fuerzas.

El vector de repulsión lo forman la suma de los datos del laser y usaremos el signo negativo para que apunte en la dirección opuesta al obstáculo. La repulsión es inversamente proporcional a la distancia, es decir, a menor distancia, mayor repulsión. Además pondero los pesos según si el obstáculo está de frente o a los lados, ya que un obstáculo como la pared es de menos importancia que uno enfrente

El vector atractivo esta compuesto por las coordenadas relativas del objetivo

El vector resultante es la suma de ambos:

    avg_x = car_x + K_obs * obs_x
    
    avg_y = car_y + K_obs * obs_y

El ángulo que determinamos se calcula con la tangente de las coordenadas del vector resultante, así en cada momento sabremos el valor exacto que hay que girar según el entorno al que nos enfretemos. La velocidad angular aumenta proporcionalmente con el error angular.

-Si el ángulo es positivo, la dirección resultante se encuentra hacia la izquierda.

-Si el ángulo es negativo, la dirección resultante se encuentra hacia la derecha.

-Si el ángulo es próximo a cero, el vector apunta aproximadamente hacia delante.

La velocidad lineal también es adaptativa. Cuando la dirección resultante se encuentra muy desviada del eje frontal, la velocidad lineal se reduce. Así el coche disminuye su velocidad mientras debe realizar un giro pronunciado.





https://github.com/user-attachments/assets/2643595a-e671-4906-af58-e079cb24f9f1




https://github.com/user-attachments/assets/aa6afa28-e8b5-4d36-8a8a-35d107401db6
