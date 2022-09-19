# Mazetv - shows analysis

El propósito de este proyecto es hacer una extraccción,transformación y ánalisis de la API tvmaze donde se alberga información variada de series de todo el mundo.
Asi mismo se pueden encontrar en este repositiorio las siguientes carpetas
- **db**: dónde se encuentra la base de datos creada a partir de los datos extraidos de la API de tvmaze
- **json**: archivos json extraidos de la API tvmaze
- **model**: diagrama ERD del modelo de datos creado, también se encuentra el datamart creado para dar respuesta a las preguntas de negocio
- **profiling**:archivos html de los profiling realizados a los dataframes y ademas un archivo PDF donde se realiza el analisis a partir del profling realizado
- **src**: archivo de Jupyter Notebook donde se muestra el desarrollo del ejercicio de extracción, transformación y graficados


![](https://static.tvmaze.com/images/api/tvm_api.png)


**Table of Contents**

1. Proceso de extracción de datos a través de API
2. Creación de dataframes
3. Dataframe profiling
4. Data cleaning
5. Creación base de datos en sqlite
6. Plots
7. Creación de datamart
8. Conclusiones

# 1. Proceso de extracción de datos a través de API <a name="Proceso de extracción de datos a través de API"></a>
En primer lugar para poder realizar la extracción de los datos de la página se importan las librerias necesarias

	import requests
	import datetime
	import pandas as pd
	import seaborn as sns
	import numpy as np
	import matplotlib.pyplot as plt

Luego se crea una función para poder recorrer cada día desde el 1 de diciembre hasta el 31 de diciembre de 2020 y asi mismo se imprime cada fecha para saber que si hemos traido los datos de cada dia, en esta misma función se crea un datamart por cada json obtenido y se agrega en una lista para posteriormente concatenarlo y obtener asi un solo dataframe.


# 2. Creación de dataframes
En este paso se crean los diferentes dataframe a partir del dataframe base, esto ya que la API arroja un JSON anidado donde se puede ver información tanto de los shows,episodios, canales web y conexiones . Siendo asi, se crea un dataframe por cada uno de estos datos identificados (show_episodes, shows, web_channels y network) , cada dataframe se crea con su respectivo ID como llave princial.

# 3. Dataframe profiling
Una vez creados los dataframe se procede a hacer una analisis de la calidad de datos obtenidos, con la libreria "pandas_profiling" podemos facilmente obtener estadística descriptiva con sus respectivos gráficos. Para obtener mas detalle de los resultados obtenidos ver archivos en la carpeta profiling/ de este respositorio
# 4. Data cleaning
En la fase de limpieza vamos a realizar los siguientes pasos :
- Renombrar las columnas de cada dataframe (show_episodes, shows, web_channels y network) 
- Cambiar el tipo de dato a datetime de las variables que correspondan
- Eliminar las columnas que todos sus valores son nulos
- Eliminar las listas del dataframe "shows" de la variable "genres"  para que quede en un mejor formato
- eliminar duplicados de los dataframe shows, network y web_channels
- Tambien eliminamos los id que son nulos

Una vez realizado estos pasos ya tenemos una data mas limpia para almacenar los diferentes dataframes en una base de datos sqlite 
# 5. Creación base de datos en sqlite
Creamos dos funciones: una para crear nuestra base de datos y otra para almacenar cada df como una tabla en la base de datos ya creada en la primera funcion
# 6. Plots
Ya por último procedemos a realizar diferentes graficas para asi poder tomar decisiones de negocio mas facilmente
* En la primera grafica se grafica el runtime promedio por tipo de show por cada mes de diciembre: se puede observar que está en una escala logaritimica los valores ya que el runtime en diciembre es mucho mas alto que los otros meses; liderando en estos ultimos meses las series con libreto, seguido por los documentales y los "talk shows"
* En segunda instacia se grafica la cantidad de shows por genero por mes, sin embargo debido a la cantidad de generos es necesario repartir las graficas en trimetres para una mejor visualización. Como en el anterior gráfico la mayoria de series se encuentran en diciembre siendo el genero drama con la mayor cantidad de shows
* En tercera instacia se gráfica el número de series por país por mes, aún así vemos que hay una gran cantidad de shows que no tienen país asignado,  en el ultimo trimestre vemos que la mayoria de series provienen de China  y república de Corea
* Para finalizar se generan dos series de gráficos: el primero donde se muestra el puntaje promedio por país, pero no tenemos datos suficientes para poder dar conclusiones ya que la mayoría de los shows no tienen puntaje, 	el segundo nos muestra varios graficos del puntaje promedio por genero por  mes , y al igual que el anterior grafico no hay suficiente data para dar conclusiones. 
# 7. Creación de datamart
En una última instancia se crea un datamart para dar respuesta a las preguntas de negocio, para ello se realiza joins entre tres de las tablas almacenadas en la base de datos sqlite, asi obtenemos los siguientes campos:
*  id del a tabla stores, tipo de show, mes y año del estreno de la serie,genero de la serie, pais de la serie, conteo del numero de episodios por serie ( episodios emitidos en diciembre del 2020), promedio de la duración de los episodios de cada show, y promedio de calificacion de cada show. Asi mismo en la carpeta model se encuentra el diagrama ERD de las tablas

**query realizado para crear tabla "tbl_datamart_shows"**

SELECT s.id,
   s.name,
   s.type,
   strftime('%Y/%m', s.premiered) as premiered_year_month,
   s.genres,
   wc.country_name,
   count(se.id) as number_series,
   avg(s.average_runtime) as average_runtime,
   avg(s.rating_average) as rating_average
  FROM tbl_shows as s
  LEFT JOIN tbl_show_episodes as se on se.show_id = s.id
  LEFT JOIN tbl_web_channels as wc on wc.id = se.webchannel_id
 WHERE s.premiered between '2020-01-01' and '2020-12-01'
 GROUP BY 1,2,3,4,5
 ORDER BY 1;  

# 8. Conclusiones

Si bien mazetv es una plataforma donde los amantes de las series comparten y contribuyen, hay muchos shows que no tienen información con respecto a pais de emisión, calificación promedio del show se evidencia que la mayoria de series en esta muestra salieron al aire en diciembre del 2020 y es allí donde se puede encontrar la mayor información con respecto a las series, siendo los shows de drama los que tienen mayor cantidad de series y la mayoria de ellas provientes de Asia.
