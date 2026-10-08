# Follow Line

El objetivo de este ejercicio es implementar un control reactivo PID capaz de seguir la línea roja pintada en el circuito de carreras de Fórmula 1.

<img width="285" height="135" alt="imagen" src="https://github.com/user-attachments/assets/f0e95609-4f7f-4864-8ab1-5d615dc7bb4e" />

En esta práctica se adquieren conocimientos de OpenCV y controladores pid, aprendiendo cada una de las componentes de este (proporcional, derivada e integral) y como influyen al control del programa.

El controlador busca regular el comportamiento del circuito. Al ser un bucle cerrado, se hace una comparación entre el resultado obtenido y el real, y, en base a este, se regulan las acciones.

**Controlador PID:**

P (proporcional): la corrección es proporcional al error actual. Con ganancia alta oscila y con ganancia baja reacciona tarde.

D (derivativo): depende de la velocidad a la que cambia el error. Amortigua la respuesta y reduce el zig-zag.

I (integral): acumula el error a lo largo del tiempo para eliminar errores pequeños y constantes.

<img width="565" height="376" alt="imagen" src="https://github.com/user-attachments/assets/099c2e92-b9c2-40eb-87a8-3229d29bd6b1" />

**Desarrollo**

La cámara proporciona una imagen, a esta le añadimos una máscara HSV para que filtre por color, tono y brillo, y así poder obtener la posición de la línea roja la cual hay que seguir para completar el circuito.

La máscara está sobre la franja de la mitad inferior, ya que es donde realmente necesitamos la información. 

Si es más abajo, será demasiado tarde para reaccionar; mientras que, si es más arriba, nos estamos adelantando al circuito y podemos reaccionar demasiado pronto.

Otras acciones son el cálculo de momentos y error. Error = 0 significa que la línea está centrada, negativo a la izquierda y positivo a la derecha.

P: W = -Kp * error. El signo menos es necesario porque el error y el giro deben tener sentido contrario.

PD: derivada = error - error_anterior. Se guarda error_anterior al final del bucle, para que en la siguiente vuelta contenga el error de la vuelta previa.

PID: integral = integral + error, limitada entre -L y +L con L = 0.5 / Ki. 

    Fórmula  W = -(Kp·error + Kd·derivada + Ki·integral)

    HAL.setV(W)

Además añado una velocidad la cual dependiendo de si el trozo de circuito por el que va es recto o curvo, adquirirá una velocidad u otra, es decir, se adapta al terreno.

    V = V_MAX - (V_MAX - V_MIN) * min(abs(error) / 320, 1)

    HAL.setV(V)

Para encontrar la línea, en el caso de que la pierda o se desvíe, se le da una velocidad de giro baja hasta que sea capaz de volver a ella. 

Además, guarda la posición del último lugar donde la vió, izquierda o derecha, para poder actuar consecuentemente a esto, y así, girar a la derecha o izquierda respectivamente y evitar dar un giro sobre sí mismo innecesario. Si no la ha llegado a ver entonces girará hasta que la encuentre.

**Observaciones a tener en cuenta:**

Al inicio tome como franja unas columnas exactas [213:426], esto provocaba que si el coche se salía de la franja se perdía. La solución fue contar con todas las columnas y la mitad inferior (en horizontal). 

A la hora de pasar del coche holonónmico, donde las 4 ruedas giran a la vez; al de Ackermann, donde solo giran las delantera; tuve que cambiar todos los valores de controlador PID y de las velocidades máximas y mínimas. 

Además con una mínima de 1 para las curvas seguía desviandose en la curva, aunque cuando la acababa conseguía volver sobre la línea roja. Un punto en contra es que al subirle la velocidad se descontrola en seguida, lo cual dificulta bastante hacer el circuito en un tiempo pequeño.

**Vídeos**

- HOLONÓMICO

  [Circuito simple.webm](https://github.com/user-attachments/assets/80a2f7de-4fff-446b-8423-434cfd41b01c)

  [Montreal.webm](https://github.com/user-attachments/assets/a86bf68d-c7b1-4b23-9aa1-47b026fd0df4)

  [Encontrar linea.webm](https://github.com/user-attachments/assets/0d4ff84e-3ea8-455c-bb6d-3fe47e285144)

- ACKERMANN

  https://github.com/user-attachments/assets/4abc366f-11dd-4fea-b528-a8bb3cbe7ae2
  
  [Encontrar Linea Ackermann.webm](https://github.com/user-attachments/assets/538a16b3-8d3a-4659-a5fb-007970a97f6f)
  
  Le toma un tiempo estabilizarse pero lo consigue


**Conclusión**
Controlar el modelo de Ackermann es más difícil debido a sus limitaciones al girar en comparación con el holonómico. Para solucionar esto hay que ir modificando cada una de las componentes de PID y aún así la velocidad aplicada debe ser menor para que consiga hacer un correcto giro en las curvas.



  


  

