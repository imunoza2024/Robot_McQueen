## Global Navigation -> TeleTaxi 
El objetivo de esta práctica es aprender el algoritmo de planificación global GPP (Gradient Path Planning) bajo una simulación de TeleTaxi.

<img width="344" height="213" alt="imagen" src="https://github.com/user-attachments/assets/10691129-cca1-467a-ac9a-aeb5946bc7fe" />
<img width="52" height="44" alt="imagen" src="https://github.com/user-attachments/assets/eb71ee27-a260-40fe-acf7-c884a72424af" />
<img width="288" height="217" alt="imagen" src="https://github.com/user-attachments/assets/b4adc175-bf38-4f67-8c26-16c6e29639e7" />
<img width="469" height="302" alt="imagen" src="https://github.com/user-attachments/assets/86e19c5d-2620-4292-bff4-92e54933f05a" />

La idea global de la práctica es darle al coche un destino, con este se calcula una ruta desde su posición inical hasta el destino. Como es un mapa de una ciudad, el coche deberá evitar los edificios, no pueden formar parte de la trayectoria y atravesarlos.

En la simulación se usará un mapa de [400, 400] en blanco (libre) y negro (obstáculo) 

Gradient Path Planning (GPP): técnica que calcula una ruta libre de obstáculos desde un punto inicio hasta un destino usando un mapa global. 
Funciona mediante creación de campos de costes numéricos en el mapa, cada celda tiene un coste propio en relación a la posición del coche y del objetivo. Este valor representa el coste asociado a desplazarse desde dicha posición hasta el destino y con ello se puede decidir hacia donde dirigirse. En un principio, cuanto más barato salga el camino mejor.

El objetivo será un mínimo y los obstáculos tendrá más costes para evitarlos. El coche se desplaza ya que va eligiendo casillas que valen menos que en la que está. Esto se ha conseguido usando dos campos:

- Subcampo de propagación al destino: representa el coste de desplazamiento necesario para alcanzar el objetivo.

- Subcampo de penalización por proximidad a obstáculos: incorpora costes adicionales a las posiciones cercanas a zonas no transitables.

      Campo total = Campo de propagación + Campo de penalización acumulada

Es importante recalcar que el camino que cueste menos no tiene porqué ser el que matemáticamente recorra menos distancia, ya que esa distancia puede llevbar más costes que un camino más largo pero con menor penalización.
El algoritmo también tiene en cuenta la proximidad a los obstáculos, por lo que puede preferir una trayectoria algo más larga si permite mantener una mayor distancia respecto a ellos.

Una de las principales ventajas de este planteamiento es que el campo puede reutilizarse durante toda la navegación. 
El robot no necesita calcular una trayectoria completamente nueva en cada iteración, sino que puede consultar los valores de las celdas cercanas a su posición y utilizar esa información para decidir hacia dónde avanzar.

**Wave Front**

Wave Front expande los costes como si fuera una onda que empieza en el destino hasta llegar al inicio.

Al inicializar las casillas, se hace a infinito, ya que no se conoce el coste asignado a cada una de ellas

La casilla destino debe valer 0

Se evalúan las demás casillas teniendo en cuenta la distancia, las diagonales y los obstáculos cercanos

Cuando encuentra una alternativa de menor coste, actualiza el valor de la celda y la incorpora a la estructura de búsqueda para continuar propagando los costes.

Los movimientos rectos tienen un coste de 1, mientras que los diagonales tienen un coste de 1,4. Esta diferencia permite aproximar la distancia real recorrida en la cuadrícula y evita considerar que desplazarse en diagonal cuesta exactamente lo mismo que hacerlo horizontal o verticalmente.

