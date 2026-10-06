# **Práctica 1: Basic Vacuum Cleaner**

## Objetivos de la práctica:

En esta práctica se nos pide programar una aspiradora de gama baja para que limpie una casa movíendose pseudoaleatoriamente por ella. Se programará un autómata con al menos 3 estados (avanzando, retrocediendo, girando). Se puede incluir un estado haciendo-espiral, que aumenta la eficiencia del barrido. Con él se logra una navegación pseudoaleatoria que estocásticamente cubre toda la casa, limpiándola. La aspiradora, al ser de gama baja no incorpora algoritmos robustos y precisos de autolocalización. La posición estimada por la odometría no se puede usar como posición verdadera, pues acumula ruido. Sí se puede utilizar la orientación para medir giros de modo aproximado. 

El autómata se implementará como un bucle infinito. No se admiten sleeps que rompan la reactividad del bucle principal.

## Desarrollo:

Para empezar la práctica instalamos Docker y el entorno necesario de Unibotics. Una vez hecho esto podemos empezar a programar.

Primero, traté de hacer un algoritmo sencillo. Mi idea era aprovechar las formas rectangulares y cuadradas de todos los objetos de la casa, haciendo que la aspiradora solo avanzase en línea recta siguiendo las direcciones N, S, E y W. Ya que HAL.getPose3d().yaw devuelve la orientación en radianes del robot, podemos gire hasta que llegue a la posición que queremos aproximadamente y vuelva a avanzar. Por tanto mi primera idea fue que siguiera una secuencia E → N → W → S → E.

Pero esta idea tenía dos problemas:  
No es pseudoaleatorio, por tanto no nos sirve. Además si entra en una Habitación cuadrada, se quedaría dando vueltas de esquina a esquina.

Por tanto, había que hacer algo menos sofisticado y más aleatorio, como se pide en el enunciado.

Para ello definí 3 estados para la variable global CURRENT\_STATE:  
MOVING\_FOWARD: El robot avanza en línea recta hasta chocar con algo.  
MOVING\_BACKWARDS: el robot retrocede tras chocar  
TURNING: el robot gira.

El funcionamiento es el siguiente:

 El robot comienza avanzando en línea recta. Si choca con algo cambia al estado de retroceso. Detecta el choque si el láser detecta un objeto a menos de 0.2 de distancia. En la explicación de la práctica de Unibotics comenta que para detectar un choque frontal se use el valor \[90\]  del láser, pero para evitar también que el robot se choque con los laterales busco el valor mínimo en laser.Values.

Si detecta un choque, cambia de estado y retrocede hasta que el objeto esté a 0.3 de distancia, y entonces pasa al estado de girar. El giro detecta la orientación actual del robot y le suma un valor aleatorio entre $\pi {\ }$ y \-$\pi$. Como la orientación se mide en radianes y $\pi$ es igual que $k*\pi \ {\ }$en muchos casos, el robot se queda girando más de una vuelta para llegar a la misma orientación que podría con media. Por tanto normalizo la posición para que recorra el camino más corto posible. Si es positivo, gira hacia la derecha y si es negativo hacia la izquierda. A cada tick() comprueba la orientación actual y la contrasta con la del objetivo. Si la diferencia entre ambas es lo suficientemente pequeña, para de girar.

Una vez para de girar vuelve a ir hacia delante y se repite el proceso.

Adicionalmente, he añadido un timeout de 50 ticks en el retroceso, para que si choca al retroceder, y la distancia con el objeto con el que se había chocado inicialmente no ha superado 0.3, pase a giro desde ahí y evite quedarse retrocediento sin parar.

Este es un vídeo demostrativo del funcionamiento de la práctica:  


https://github.com/user-attachments/assets/db7cf34c-e221-495c-b86f-e194bd6ba666


Si no se detuviera debería poder cubrir el 100, pero al ser pseudoaleatorio es muy difícil que lo haga sin tardar una eternidad.

## Conclusión:

Tras terminar la práctica he concluido que es un algoritmo demasiado simple para una aspiradora, ya que su aleatoriedad podría causar que se quede en una sola habitación durante horas. Lo ideal sería algo más sofisticado, pero sí que me ha servido para entender mejor python y cómo hacer diferentes estados. Además he visto la importancia de no dejar un estado completamente aislado hasta que se cumpla y tener en cuenta variaciones para evitar que el robot se quede pillado.

## Entrega:

Adjunto el archivo python en el aula virtual.

## Hecho por: Marcos García Vila, 3º Ingeniería Robótica Software

