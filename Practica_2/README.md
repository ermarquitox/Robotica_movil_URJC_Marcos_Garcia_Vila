# **Práctica 2: Follow Line**

## Objetivos de la práctica:

En esta práctica se nos pide programar un coche de Fórmula1, equipado con una cámara frontal, para que recorra un circuito de carreras que tiene una línea roja en el medio de la pista. Se programará uno o varios controladores PID para gobernar la velocidad lineal del coche (V) y la velocidad angular (W). El coche debe funcionar en varios circuitos, completarlos en el menor tiempo posible y sin alejarse de la línea roja. Cuantas menos oscilaciones tenga mejor, y conviene que sea robusto si en algún momento pierde la línea.

La aplicación robótica se implementará como un bucle infinito. Cada iteración incluirá las instrucciones para materializar el procesamiento de la imagen y para la toma de decisiones sobre los dos actuadores V y W.

## Desarrollo:

Para comenzar la práctica primero hice una prueba de las funciones que se nos dan. Para ello simplemente puse dentro del bucle principal las funciones HAL.getImage() seguido de WebGUI.showImage() para ver qué se ve. Este fue el resultado  
//captura

Antes de hacer que el coche comenzase a moverse, había que crear una máscara que filtrase el color rojo de la línea para así poder seguirla. Para ello usar los colores BGR8 que nos da la función de imagen es más complicado, por tanto para filtrar el color rojo primero pasaremos los colores a HSV. Para ello necesitamos la biblioteca cv2. Con ella, usando la función cv2.inRange(hsv, lower, upper), podemos pasar la imagen a hsv, y al definir los valores upper y lower en el umbral que necesitamos para el rojo, se crea nuestra máscara, dejando en la imagen todo lo que no es rojo de color negro y la línea roja de color blanco.  
//captura

Una vez conseguido esto, ahora necesitamos encontrar el centro de la línea para así poder seguirla y corregir los valores que le pasamos a los actuadores en función de el error que hay entre el centro de nuestra imagen y el centro de la línea. Por comodidad defino la región a partir de donde se ve la línea en pantalla, que es la mitad inferior region \= mapped\_image\[height//2:, :\]

Para encontrar el centro de la línea, he usado la función np.where(region \> 0\) que nos devuelve las filas y las columnas de nuestra región donde los píxeles son blancos, es decir, la línea roja. Ahora que sabemos cuales son y se guardan en nuestra variable columns, con np.mean(columns), sacamos el punto medio.

Ahora que nuestro coche ya sabe dónde está el punto medio de la línea ubicado en la imagen en todo momento, debemos implementar el controlador PID.

Calculamos el error, es decir, la cantidad de píxeles y la dirección que diferencian el centro de nuestra imagen con el centro de la línea: error \= (width//2) \- line\_middle. La variable width es el ancho de la imagen, que hemos obtenido con image.shape.

Cuando tenemos el error aplicamos la fórmula para comenzar con la P de nuestro controlador, por tanto la W sería w \= Kp \* error. Pero esto por sí mismo no es suficiente para evitar que el coche oscile y se desvía, además de que se nos pide explícitamente un PID. Por tanto para añadir esa D necesitamos calcular la derivada del error, que es la diferencia entre el error actual y el anterior. Por eso añadimos al principio del tick que el error anterior sea el actual: previous\_error \= error. Y tras calcular el error en esta nueva iteración, se calcula la derivada: derivative \= error \- previous\_error

Y con esto aplicamos la fórmula: w \= Kp \* error \+ Kd \* derivative, que ya hace mucho más fluido el giro y una vez ajustemos los valores de las ganancias, harán que no oscile mucho y no se desvíe. Pero antes nos falta la I del PID. 

En este caso yo he dejado la ganancia de la I (Ki) en 0, ya que al ser una simulación, el escenario es ideal y no hay pequeños errores. Pero está implementado de tal manera que se acumula el error a lo largo del tiempo y se usa para corregir problemas que la P y la D no solucionan, como que la cámara estuviera algo torcida y el robot fuera descentrado en la línea. La I va sumando el error en cada tick: suma\_errores \= suma\_errores \+ error. Aunque el error sea pequeño (5 píxeles), si es constante, la suma va creciendo.

Y la corrección de la I es Ki \* suma\_errores. Como la suma crece, la corrección crece, hasta que el error residual se elimina. Cumpliendo la fórmula del PID:  
w \= Kp \* error \+ Kd \* derivative \+ Ki \* error\_sum.

Con esto, el coche ya recorre el circuito siguiendo la línea con una velocidad constante y girando en función de dónde esté el centro de la línea. El problema surge si la velocidad es lo suficientemente alta como para que se desvíe en las curvas. Por tanto usamos un control de velocidad adaptativo: en vez de ir siempre a V\_max, el coche modula su velocidad según lo desviado que esté de la línea y lo rápido que esté cambiando esa desviación. La fórmula es: V \= V\_max \- K\_v \* abs(error) \- K\_dv \* abs(derivative). Así, cuanto mayor sea el error y más tarde en centrarse, más lento va, para dar tiempo de reacción en las curvas.

Una vez implementado todo esto, fui probando con velocidades máximas, y diferentes ganancias para ver cuál se ajustaba mejor. Probando ajusté los siguientes valores, que me parece que dan un equilibrio suficiente entre velocidad, seguir con precisión la línea y no oscilar:   
Kp \= 0.0035  
Kd \= 0.05  
Ki \= 0  
V\_max \= 12\.

Por último he implementado que si no se guarde el último error en cada interacción para que si no se encuentran píxeles blancos el coche siga girando en la dirección donde perdió la línea, ya que a menos que el circuito estuviera mal hecho, lo más probable es que pierda la línea al no girar lo suficiente en una curva. Por tanto sigue girando en la dirección de la curva buscando reencontrar la línea.

Aquí están los vídeos de demostración:

Video del primer circuito completo:  
//video1

Video del segundo circuito completo:  
//video2

Vídeo de demostración de robustez en el circuito 3:  
//video 3  
En este último se ve que al haber curvas muy cerradas el coche pierde la línea, pero reduce la velocidad y sigue girando en la misma dirección, por tanto la consigue encontrar y se reincorpora.

## Conclusión:

A pesar de no ser perfecto y oscilar a veces más de lo que gustaría, el diseño cumple con el objetivo y es robusto. He de añadir que al ir a velocidad 12, estamos sacrificando estabilidad a cambio de velocidad, ya que antes de las curvas entra muy velozmente, haciendo que reaccione peor. Si bajásemos la velocidad, a pesar de ser más lento, sería más estable.

## Entrega:

Adjunto el archivo python en el aula virtual.

## Hecho por: Marcos García Vila, 3º Ingeniería Robótica Software

