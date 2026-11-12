## Global Navigation -> TeleTaxi 
El objetivo de esta práctica es aprender el algoritmo de planificación global GPP (Gradient Path Planning) bajo una simulación de TeleTaxi.

<img width="344" height="213" alt="imagen" src="https://github.com/user-attachments/assets/10691129-cca1-467a-ac9a-aeb5946bc7fe" />
<img width="52" height="44" alt="imagen" src="https://github.com/user-attachments/assets/eb71ee27-a260-40fe-acf7-c884a72424af" />
<img width="288" height="217" alt="imagen" src="https://github.com/user-attachments/assets/b4adc175-bf38-4f67-8c26-16c6e29639e7" />
<img width="469" height="302" alt="imagen" src="https://github.com/user-attachments/assets/86e19c5d-2620-4292-bff4-92e54933f05a" />

La idea global de la práctica es darle al coche un destino, con este se calcula una ruta desde su posición inical hasta el destino. Como es un mapa de una ciudad, el coche deberá evitar los edificios, no pueden formar parte de la trayectoria y atravesarlos.

En la simulación se usará un mapa de [400, 400] en blanco (libre) y negro (obstáculo). 

El GPP es un plan como recurso. En lugar de calcular una única ruta fija, se calcula un campo escalar sobre todo el mapa donde a cada celda se le asigna un valor que indica el coste para llegar desde ella al destino. El robot puede consultar ese campo en cualquier momento y, mirando a sus vecinas, saber hacia dónde moverse e ir hacia la vecina con menor valor, es cecir sigue el gradiente de forma descendente. El destino vale 0.

La ventaja frente a una ruta fija es que, si el robot se desvía un poco, el campo sigue siendo válido desde su nueva posición y no hace falta replanificar, ya que el campo se calcula una vez por destino.


El objetivo será un mínimo y, los obstáculos, tendrán más costes para evitarlos. El coche se desplaza ya que va eligiendo casillas que valen menos que en la que está. Esto se ha conseguido usando dos campos:

- Subcampo de propagación al destino: representa el coste de desplazamiento necesario para alcanzar el objetivo, es decir, la acumulación de costes. Cuanto más lejos del destino, más cuesta

- Subcampo de penalización por proximidad a obstáculos: incorpora costes adicionales a las posiciones cercanas a zonas no transitables.

      Campo total = Campo de propagación + Campo de penalización acumulada

Es importante recalcar que el camino que cueste menos no tiene porqué ser el que matemáticamente recorra menos distancia. Esa distancia puede conllevar más costes que un camino más largo.
El algoritmo también tiene en cuenta la proximidad a los obstáculos, por lo que puede preferir una trayectoria algo más larga si permite mantener una mayor distancia respecto a ellos.

Una de las principales ventajas de este planteamiento es que el campo puede reutilizarse durante toda la navegación. 
El robot no necesita calcular una trayectoria completamente nueva en cada iteración, sino que puede consultar los valores de las celdas cercanas a su posición y utilizar esa información para decidir hacia dónde avanzar.

**Wave Front**

El campo se calcula con un frente de onda (wavefront) donde los costes se expanden como una onda que empieza en el destino y llega hasta el resto del mapa. Lo he implementado con el algoritmo de Dijkstra lanzado desde el destino y no desde el robot. Así, al terminar, cada celda contiene el coste mínimo para llegar al destino desde ella y el campo sirve para cualquier posición del coche.
Usando BFS solo, no se cumplirían los requisitos de la práctica, ya que el coste es 1 en recta, 1,4 en diagonal (aprox. raíz de 2) y además se suma la penalización por obstáculos. Por eso se usa una cola de prioridad que extrae siempre la celda con menor coste total.

**Implementación**

Lo primero, una vez cargado el mapa, se valúa y se crea una máscara donde se va comprobando si dicha celda es camino o no. También es necesario obtener las coordenadas del coche y del destino, verificando que están dentro del mapa en zonas no prohibidas.

Campo de penalización: para saber cuanto debo penalizar una casilla, debo saber la distancia entre esta y el obstáculo más cercano que encuentre. En este cálculo utilizo una BFS donde todas las celdas de obstáculo entran en una cola con distancia 0 y la distancia se propaga a sus vecinas. La propagación del campo se corta a 8 celdas.

La penalización de cada celda es (DISTANCIA_SEGURIDAD - distancia)² si está a menos de 8 celdas de un obstáculo, y 0 en el resto. Al ser cuadrática crece muy deprisa al acercarse al borde y es casi nula lejos de él, lo que empuja al coche hacia el centro de la carretera.

Al inicializar las casillas, se hace a infinito, ya que no se conoce el coste asignado a cada una de ellas.

La casilla destino debe valer 0

Se evalúan las demás casillas teniendo en cuenta la distancia, las diagonales y los obstáculos cercanos. Para ello implemento una cola de prioridad, cuya idea es expandir primero aquellas casillas más baratas.
Con cada extracción, se evalúan sus 8 celdas vecinas, pero no de igual manera. Las rectas valen 1, mientras que las diagonales es 1,4 ~ raíz(2). Se descartan aquellas que estén fuera de los límites del mapa o sean obstáculos y para las diagonales es necesario que dos de sus casillas que estén al lado sean transitables, para que no acabe en unsa esquina.

El coste de propagación en cada iteración se calcula sumándole el nuevo coste al ya obtenido en iteraciones anteriores. 
Cuando encuentra la alternativa de menor coste, actualiza el valor de la celda y la incorpora a la estructura de búsqueda para continuar propagando los costes.

El resultado es un campo en el que, desde cualquier celda alcanzable, bajar por el gradiente conduce al destino.

Para extraer la ruta basandonos en el gradiente he implementado dos funciones

- Sacar camino: va desde la posición inicial del robot hasta el objetivo recorriendo las celdas que cuesten menos y devuelve una lista de estas. Se hace para verificar que el destino sea alcanzable una única vez al inicio.

- Seguir gradiente: desde una celda avanza un número concreto de pasos, siempre hacia la vecina de menor coste, y devuelve la celda a la que llega. Se usa en cada iteración del control para obtener puntos de referencia por delante del coche y detectar con antelación curvas y esquinas.

El campo no se calcula por cada iteración, sino que se hace una vez por destino y después se va consultando poco a poco en cada iteración.

Para tener una velocidad adaptativa el foco está puesto en dos puntos de la función que sigue el gradiente, uno a 3 pasos y el otro a 8. El más cercan, irá guiando la dirección del coche y el lejano anticipará de curvas, esquinas, etc.

Evaluando sus errores angulares modelo las velicidades:

            Angular: W = 2 · error_cercano
            
            Lineal V = VELOCIDAD_MAXIMA * factor_giro * factor_curva
El factor de giro reduce la velocidad si el coche está muy desalineado con el punto cercano, y el factor de curva la reduce si el punto lejano indica una curva próxima. Ambos tienen un mínimo para que el coche no se pare. Así el coche va rápido en las rectas y frena antes de las curvas..

¿Cómo se cuando he llegado? Cuando la distancia entre el coche y el destino es menor que un umbral, se dará por alcanzado y pedirá otro destino. Si se marca uno nuevo en mitad del trayecto, el coche se para, replanifica desde su posición actual y continúa.

**Conclusiones**

Con GPP podemos separa planificación, calculada una vez por destino, y pilotaje, que se revisa en cada iteración

Combinar propagación y penalización da trayectorias más seguras, aunque no sean las más cortas

Dijkstra con cola de prioridad es lo que permite calcular el campo completo

[TeleTaxi.webm](https://github.com/user-attachments/assets/c121f817-ed82-4851-8681-5427b5453cc6)


