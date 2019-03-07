## Práctica 'follow line'.

### Introducción

En esta página se explica la metodología empleada para el desarrollo de la práctica de `follow_line` en la asignatura de Visión en Robótica del Máster Oficial en Visión Artificial.

El objetivo de la práctica es aplicar comportamiento reactivo a un modelo de Fórmula 1 para que siga una línea roja pintada sobre el asfalto de la simulación. Con este comportamiento se pretende optimizar los valores de los actuadores del robot para que complete la vuelta en el menor tiempo posible.


### Preproceso

Para que el robot simulador siga la línea se obtienen imágenes de una cámara situada en el centro del modelo. Estas imágenes pasan primeramente por una función que procesa el contenido capturado. Dado que buena parte de la imagen pertenece al cielo de la escena, se procesa el contenido de la imagen de retorno a partir de la mitad, centrando la atención en la mitad inferior de la imagen que será donde ocurran los eventos.

Con la imagen recortada se procesa la imagen para extraer la línea roja del asfalto, que es la guía por donde el vehículo circula. Para esta extracción se decide pasar la imagen RGB que se obtiene de la cámara al modelo de color HSV por ser más robusto frente a cambios de iluminación y, por tanto, obtener valores más fiables e independientes de otros elementos.

Con este cambio en el modelo de color y fijados los valores del filtro se obtiene la línea roja. Puede verse en la siguiente imagen el resultado de la segmentación.




Con la imagen segmentada se procede a la extracción de 2 líneas (filas). Utilizando la librería `numpy` se busca en estas líneas el punto central que existe entre los valores marcados como 255 (blanco) para extraer el centro de la línea. Este punto es el de referencia para las órdenes que se enviarán a los actuadores. 

### Diseño
Dado que la imagen tiene un ancho de 640 píxeles, el punto centra de la imagen se encuentra en el píxel 320. Las diferencias que existen entre el punto central de la imagen y el valor calculado a través de la extracción del punto centro de la fila será la desviación cometida con respecto al centro.

Esta desviación se clasifica como el **error** y es el parámetro en el que se basa la práctica. 

### Comportamiento

Para corregir el error se diseña un control proporcional y derivativo (PD). 



<iframe width="560" height="315" src="https://www.youtube.com/embed/Tqh14Q2boXY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/igarag/mova-vision-robotica/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
