# **Práctica 3: Navegación local esquivando obstáculos con algoritmo VFF**

## Objetivos de la práctica:

En esta práctica se nos pide programar un coche de Fórmula1, equipado con un sensor laser y un sensor GPS, para que recorra un circuito de carreras con valla en los laterales y con otros coches como obstáculos a lo largo del recorrido. Se programará el algoritmo de navegación local VFF con fuerzas virtuales para gobernar la velocidad lineal del coche (V) y la velocidad angular (W) en cada iteración. El coche debe completar el circuito en el menor tiempo posible y siempre manteniendo la seguridad de que no choca con ningún obstáculo. El sistema de navegación global ha planificado una secuencia de subobjetivos de modo que si se van alcanzando consecutivamente garantiza que se completa la vuelta al circuito. Se considerará un subobjetivo alcanzado cuando la posición del coche está razonablemente de la posición del subobjetivo. En ese caso se pasará a navegar localmente hacia el siguiente subobjetivo.

La aplicación robótica se implementará como un bucle infinito. Cada iteración incluirá las instrucciones para materializar el procesamiento del sensor laser, las fuerzas repulsivas, la fuerza atractiva, su combinación y su traducción a decisiones sobre los dos actuadores V y W.

## Desarrollo:

Para comenzar, probé como se visualizaba el sensor láser y los vectores con las funciones HAL.getLaserData() y WebGUI.showForces(vec1, vec2, vec3).

Una vez hecho eso, antes de hacer que el coche se moviera, me encargué de sacar los vectores a partir de los datos, para luego aplicarlos al movimiento. Primero obtenemos el vector de atracción, asignando el primer objetivo fuera del bucle con current\_target \= WebGUI.getNextTarget(), y una vez en el bucle obtenemos la posición actual del coche con    car\_x \= HAL.getPose3d().x, car\_y \= HAL.getPose3d().y, y la orientación con car\_yaw \= HAL.getPose3d().yaw.

Podemos obtener la distancia al objetivo viendo la diferencia entre las coordenadas del objetivo y del coche, pero esa distancia es global, y nos da un vector que no tiene en cuenta la orientación del coche, por tanto necesitamos aplicar la rotación con la orientación del coche, para que en vez de tomar el eje de coordenadas global del mapa, utilice el centro y la orientación del coche.

Una vez hecho esto, con pitágoras sacamos el valor de la distancia al objetivo y con una condición if, si el coche se acerca lo suficiente (0.5) se registra el objetivo como visitado y se pasa al siguiente.

Ahora que tenemos el vector de atracción necesitamos el de repulsión. Para ello con un bucle for recorremos los 180 valores de laser\_data y a partir de la distancia y el ángulo construimos 180 vectores con sus componentes x e y, y hacemos un sumatorio. Lo hacemos en negativo para que así el vector no vaya del coche al obstáculo si no del obstáculo al coche, ya que es de repulsión. Después dividimos entre 180 los sumatorios de x e y, para que sean magnitudes equiparables a la atracción, ya que son las repulsiones de 180 lecturas.

Por último hallamos el vector dirección sumando la atracción y la repulsión.  Lo convertimos en movimiento pasando el vector direccion y, multiplicado por una ganancia K\_W \= 3 como velocidad angular, y regulamos el movimiento lineal multiplicando una velocidad máxima por el vector dirección\_x. Como están normalizados sus valores están entre \-1 y 1\.

Esto producía un problema. La suma de todos los componentes x de los 180 vectores, siempre resultan en una repulsión en dirección contraria al movimiento, ya que todos los obstáculos detectados se encuentran por delante del coche. En la teoría de la práctica solucionan esto ignorando el componente x del vector dirección por completo y solo utilizando el componente y para girar. Esto funciona pero no da posibilidad de regular la velocidad lineal basándonos en los objetos que hay hacia delante, por tanto a menos que se vaya muy despacio, el coche no tiene tiempo para reaccionar ante los objetos que tiene enfrente.

Para solucionarlo, apliqué una condición dentro del bucle for que obtiene los vectores, y es que solo suma los componentes x de los vectores que se encuentran en el rango (75-105), es decir, los que están en la parte delantera, no en los laterales. Por tanto si se detecta un obstáculo delante a partir de cierta distancia, se suma la repulsión hacia atrás de ese objeto, para poder regular la velocidad lineal. Para que la magnitud de este componente esté a la par con el de y, tengo una variable contador para los componentes x que sumemos, así, luego dividimos el sumatorio entre el contador.

Y para asegurar que el coche no pare y nunca vaya hacia atrás, si la multiplicación de la velocidad máxima por el vector dirección en x es menor que 1, la velocidad lineal se mantiene en 1\. 

Esto permite que el coche reduzca su velocidad lineal al detectar obstáculos próximos en su trayectoria y que, mediante la componente lateral de la repulsión, modifique su orientación para intentar esquivarlos. He usado ganancias para la repulsión de x y de y, las cuales he ido probando hasta obtener valores que funcionaban correctamente.

Aquí está el vídeo de demostración:  
//Video  
Recorre el circuito en aproximadamente 2 minutos y medio. A pesar de que la variable se llame así, la velocidad máxima no es de 6, sino de 6x1.5, ya que el vector de atracción está multiplicado por esa ganancia, y el máximo valor de la velocidad es 6 multiplicado por el valor de x del vector dirección, que al no tener obstáculos delante, es igual al vector atracción. El nombre de la variable lo puse antes de poner la ganancia, por tanto sí que era la velocidad máxima en ese momento ya que el valor máximo del vector era 1\. Solo es un apunte, ya que a pesar de no haberlo cambiado en el código, un nombre más correcto sería factor de velocidad, por ejemplo.

## Conclusión:

Si bien no es perfecto, y al evitar que el coche frene o retroceda, no podemos asegurar que con obstáculos más complicados no se pudiera quedar atrapado, el algoritmo cumple con el objetivo y demuestra las debilidades del diseño de vff, siendo la anulación de fuerzas y la repulsión en sentido contrario, y me ha servido para entender cómo lidiar con ello y cómo desarrollar una navegación local a partir de una navegación global dada, lo cual es interesante porque podríamos desarrollar otro tipo de navegación local, manteniendo la global sin tocar.

## Entrega:

Adjunto el archivo python en el aula virtual.

## Hecho por: Marcos García Vila, 3º Ingeniería Robótica Software

