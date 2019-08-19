# Visión en Robótica

## Práctica Reconstrucción 3D

Esta segunda práctica de la asignatura de Visión en Robótica consiste en reconstruir una escena 3D a partir de las imágenes que recoge un robot a través de las cámaras de un par estéreo canónico.  Utilizando las propiedades vistas durante el transcurso de la asignatura se pretende obtener correspondencias entre una imagen y la otra para obtener una recontrucción del punto en un espacio tridimensional ofrecido por la plataforma Unibotics.

Este proceso de reconstrucción, así como el código desarrollado, siguen una serie de etapas que se detallan a continuación:

### 1. Preproceso

Partiendo como imagen fija la cámara izquierda, se buscan **puntos de interés** sobre la esta cámara. Estos puntos son obtenidos utilizando el filtro de Canny para obtener bordes. Inicialmente se parte de un umbral donde se detectan pocos puntos para que sea más cómodo trabajar en las primeras iteraciones con el programa. Posteriormente se incrementará el número de puntos como se verá más adelante.

Una vez se tienen estos puntos de interés representada por una nueva imagen con los píxeles con valor 255 donde existe un borde,  comienza la siguiente etapa.

### 2. Retroproyección

Utilizando el [API ofrecido por la plataforma](https://github.com/JdeRobot/RoboticsAcademy/blob/master/exercises/3d_reconstruction/pyProgeo/progeo.py), se toma un punto de los almacenados como puntos de interés para, pasarlo de coordenadas gráficas (píxeles) a coordenadas ópticas y retroproyectarlo. 

```python
pointInOpt = self.camLeftP.graficToOptical(point)
point1_3D_Left = self.camLeftP.backproject(pointInOpt)
```

El resultados de esta operación es un punto (no definido) en el espacio. Este punto, junto con el centro óptico de la cámara, unidos, forman el rayo de retroproyección. Será en este rayo donde se encuentre el punto correspondiente en la otra cámara, el cual se buscará a través de la **línea epipolar**.

### 3. Epipolar

Para reducir el tiempo de búsqueda del punto correspondiente en la cámara derecha,  se limita la búsqueda a una línea, la **línea epipolar**. Ese tiempo de búsqueda de la correspondencia se reduce aún más si se limita a unos pocos puntos de esa línea. Dado que la distancia entre las cámaras es pequeña, no será necesario buscar más allá de unos pocos píxeles. Para esta práctica, esa limitación está en 15 píxeles hacia la izquierda (como se busca en la cámara derecha, la mejor correspondencia estará hacia la izquierda).

<img src="./boxes/reconstruccion/img/epipolar.jpg" width="100%" height="60%">

### 4. Matching

Para saber cómo de buena es la correspondencia se utiliza la función de openCV  `matchTemplate` con la métrica `CCORR_NORMED` que devuelve una medida de correlación. Esta medida se realiza entre dos **recortes** tomados, uno de cada imagen. Fijando un recorte de la cámara izquierda, se extrae, en el mismo punto, uno en la imagen derecha y se mide el parentesco. Esto se repite desplazándose hacia la izquierda en busca la mejor medida de correspondencia. De los 15 píxeles de desplazamiento máximos se retornará de la función `matching` el que mayor correlación tenga. Ese será el recorte que mejor parentesco tiene con el correspondiente de la cámara izquierda y el centro del recorte será la coordenada del píxel que se retroproyecta en la siguiente etapa.

### 5. Nueva retroproyección

Con la coordenada de la cámara derecha del mejor píxel obtenido tras la correlación se repiten los pasos para obtener el punto en 3D que, de igual manera, unido con el centro óptico, se obtiene el nuevo rayo de retroproyección. Esta vez de la cámara derecha.

En el siguiente vídeo puede verse qué ocurre si representamos en el espacio los puntos de retroproyección en ambas cámaras. En color negro están los puntos retroproyectados de la cámara derecha y en blanco los de la cámara izquierda. Con esta disparidad  se calcularán las interrecciones. Estas íntersecciones estarán más próximas a punto desde donde se está visualizando esta escena.

<video width="600px" height="400px" controls preload> 
    <source src="./boxes/reconstruccion/img/reconstruccion_trazas.mp4"></source> 
</video>

### 6. Intersección entre los rayos, triangulación.

Idealemente ambos rayos deberían coincidir en un punto común que coincidiría con el punto tridimensional. En la realidad, eso no ocurre, por lo que se tiene que buscar el punto donde la distancia entre dos rectas sea más pequeña. Para esta relación se utiliza las ecuaciones para obtener la distancia mínima entre dos rectas en [este enlace](http://paulbourke.net/geometry/pointlineplane/). Donde cada uno de los puntos representa la posición del centro óptico de la cámara izquierda, derecha así como los puntos retroproyectados. El punto medio entre las distancias será el punto final que se utilizará en la siguiente etapa.

### 7. Resultados

Una vez se completan los pasos anteriores, la última etapa es la representación en el Canvas 3D de la plataforma. Para ello se obtiene (usando las cooredenadas de la imagen izquierda) el color del pixel para que el resultado se obtenga en color. Pasados aproximadamente 90 segundos se obtiene el resultado que se puede ver en la figura.

<img src="./img/reconstruccion_3d/solucion.png" width="100%" height="60%">

Puede verse el proceso de la reconstrucción en el siguiente vídeo:

<video width="600px" height="400px" controls preload> 
    <source src="./boxes/reconstruccion/img/reconstruccion_solucion.mp4"></source> 
</video>

Las zonas huecas que se ven en la solución (interior del patito de goma) ocurre porque no se han sacado puntos de interés en ese área (caen como si se trataran de fondo) y generan el hueco.

Si lo visualizamos desde otro ángulo, podemos ver los planos formados a distinta profundidad. Debido a la resolución de las imágenes que se obtienen desde las cámaras, se ven claramente los distintos planos de profundidad. Esto es debido a que, objetos más cercanos tienen más disparidad que los lejanos. Dado que la resolución de las imágenes es de 320x240 existen unas *limitaciones* a la hora de obtener correspondencias. A medida que la resolución de la cámara aumenta es posible obtener más planos en la disparidad, lo que se traduciría en mayor número de *planos de imagen* de los que se ven en la siguiente imagen.

<img src="./boxes/reconstruccion/img/plano_cenital.png" width="100%" height="60%">



El **número de puntos** que se han representado son, aproximadamente, 23580 en 1:30 minutos.

### 8. Conclusiones

Se ha solucionado la práctica de reconstrucción 3D de una escena empleando dos cámaras en un par estéreo y midiendo, en la línea epipolar, el parentesco entre parches de las imágenes con el fin de obtener la triangulación y representarse en el espacio 3D.

Una vía para mejorar esta reconstrucción sería obtener más puntos ya que se obtendrían todos los píxeles de los objetos de la escena. Dado que sería muy posible que en este proceso se incluyera más ruido también sería necesaria una comprobación que descartara posibles puntos que no pertenecen a los objetos.

Como conlusión personal, tanto esta como la anterior, han sido prácticas muy interesantes de realizar que han permitido explotar la plataforma mostrando las ventajas de programar en un entorno web.



















## Práctica 'follow line'

### Introducción

<div>
	<img align="left" width="30%" height="30%" src="./img/puntos_interes.png">
</div>

En esta página se explica la metodología empleada para el desarrollo de la práctica de `follow_line` en la asignatura de Visión en Robótica del Máster Oficial en Visión Artificial.

El objetivo de la práctica es aplicar comportamiento reactivo a un modelo de Fórmula 1 para que siga una línea roja pintada sobre el asfalto de la simulación. Con este comportamiento se pretende optimizar los valores de los actuadores del robot para que complete la vuelta en el menor tiempo posible.


### Preproceso

Para que el robot simulador siga la línea se obtienen imágenes de una **cámara situada en el centro** del modelo. Estas imágenes pasan primeramente por una función que procesa el contenido capturado. Dado que buena parte de la imagen pertenece al cielo de la escena, se procesa el contenido de la imagen de retorno a partir de la **mitad**, centrando la atención en la **mitad inferior** de la imagen que será donde ocurran los eventos.

Con la imagen recortada se procesa la imagen para extraer la línea roja del asfalto, que es la guía por donde el vehículo circula. Para esta extracción se decide pasar la imagen RGB que se obtiene de la cámara al modelo de color HSV por ser más robusto frente a cambios de iluminación y, por tanto, obtener valores más fiables e independientes de otros elementos.

Con este cambio en el modelo de color y fijados los valores del filtro se obtiene la línea roja. Puede verse en la siguiente imagen el resultado de la segmentación.

<p align="center">
  <img width="60%" height="60%" src="./img/imagen_segmentada.png">
</p>

Con la imagen segmentada se procede a la extracción de 2 líneas (filas). Utilizando la librería `numpy` se busca en estas líneas el punto central que existe entre los valores marcados como 255 (blanco) para extraer el centro de la línea. Este punto es el de referencia para las órdenes que se enviarán a los actuadores. 

### Diseño
Dado que la imagen tiene un ancho de 640 píxeles, el punto central de la imagen se encuentra en el píxel 320. Las **diferencias** que existen **entre el punto central** de la imagen y el **valor calculado** a través de la extracción del punto centro de la fila **será la desviación cometida** con respecto al centro.

Esta desviación se clasifica como el **error** y es el parámetro en el que se basa la práctica. 


### Comportamiento

Para corregir este error se diseña un **control proporcional** y derivativo (PD). Con este mecanismo de control se **corrige el error** proporcionado por la diferencia entre el centro de la imagen y el centro de la línea de manera proporcional (`Kp`), es decir, tanta corrección como desviación sufra el vehículo. Para **compensar las oscilaciones** proporcionadas por este controlador se añade la **componente derivativa** (`Kd`) que **suaviza** estos **cambios bruscos** calculando **diferencias entre el error actual y el anterior** para conocer si el error **aumenta** y, por tanto, hay que **corregir más** o **disminuye** y la componente proporcional tiene que **corregir menos** cantidad. 

La ecuación que sigue este controlador es la siguiente:

```python
correccion = kp * desviacion + kd * (desviacion - desviacion_anterior)
```

El ajuste de las constantes `Kp` y `Kd` serán las que permitan un funcionamiento más ajustado a la línea y suavidad en la corrección. Son valores que **se calculan de manera experimental** por lo que llevan tiempo adecuar estos valores al problema concreto.



### Control Basado en casos.

Para poder ajustar la velocidad y aumentar el control sobre el vehículo **se construye otro** controlador PD muy similar al anterior. De esta manera y como se haría intuitivamente en un vehículo real, se **acelera** en los **tramos rectos** y se **frena** en las curvas o situaciones donde se desvía mucho de la línea. Para este control se implementa un **control basado en casos** que discrimine entre **Recta** o **Curva**. 

Para detectar la **recta** se ajusta un **rango de valores** de desviación con respecto al centro de +/- 15 píxeles por lo que si el punto central de la línea se encuentra a esa distancia con respecto al centro de la imagen el vehículo se encuentra en una recta y por tanto acelera. 

Para el otro caso, se detectará **curva si no cumple** con el rango de valores estimado en el paso anterior, por lo que **reduce la velocidad**. Aplicando lo conocido para un control PD no disminuye la velocidad de manera constante ni brusca si no en función del segundo punto de estudio de la imagen, **la pared**. 

<p align="center">
  <img width="60%" height="60%" src="./img/telemetria.png">
</p>


El **estudio** del nivel de intensidad **de las paredes** proporciona información de cómo de cerca está el coche de una de ellas. En este caso se asume que un **valor 0** de nivel de intensidad en el punto corresponde a una **pared muy próxima** y por lo tanto se tiene que **reducir la velocidad**. Es aquí donde entra en juego el segundo controlador PD. Las **diferencias entre niveles de intensidad** de la pared harán incrementar la velocidad del fórmula 1 hasta alcanzar la máxima fijada. Este estudio de la pared está representado en el GUI mediante un **punto amarillo** (*wall* en la telemetría).

En el siguiente vídeo puede verse el resultado del algoritmo que lleva a realizar la vuelta al circuito en aproximadamente 47 segundos.

<video width="600px" height="400px" controls preload> 
    <source src="./img/follow_line_solution.mp4"></source> 
</video>


### Conclusiones

El resultado de la práctica ha ido siendo satisfactorio cada pequeño reto que se iba afrontando, desde la segmentación de la imagen, extracción de las líneas de interés, tratamiento de los puntos de interés,  actuación en función de los casos que se planteaban y el ajuste de parámetros. Dado que no he utilizado nunca el control PD el resultado final deja claro que este tipo de control permite llevar más allá un algoritmo que pueda tener limitaciones ya que los cambios son progresivos y permite mejor regulación.

El punto de  mejorar el tiempo marcado con una configuración de parámetros en cada vuelta daba ese toque *adictivo* al desarrollo de la práctica.

