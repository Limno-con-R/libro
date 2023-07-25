# Correlaciones y regresión múltiple {#regmul}

_**Autora de esta unidad:** Natalia Morandeira_

En esta Unidad vamos a abordar análisis de correlación y regresiones múltiples (modelos lineales). Las correlaciones son análisis útiles para explorar nuestros datos y para evaluar si dos variables están asociadas positivamente o negativamente. Por otro lado, en el caso de una regresión simple, tenemos una variable respuesta (o dependiente) y una variable explicativa (o independiente). Un análisis de regresión múltiple es muy adecuado cuando hemos medido muchas variables, por ejemplo, si tenemos una variable respuesta y muchas posibles variables explicativas. Nos interesa conocer cuál o cuáles variables explican la variación de la variable respuesta, y elegir el mejor modelo. 

En el marco de análisis limnológicos, una situación común es tener una variable respuesta medida en cuerpos de agua y múltiples variables que creemos que pueden explicar su variación. Entre las variables respuestas, podríamos tener (de acuerdo al objetivo de nuestro estudio) la abundancia, biomasa o diversidad de especies de un dado taxón o de grupos funcionales; o la concentración de un agroquímico; o incluso variables físico-químicas del cuerpo de agua que hipotetizamos que dependen de otro factor. Entre las variables respuestas, es posible que tengamos variables físico-químicas, características morfométricas de la laguna, usos de suelo del entorno, características de la cuenca, abundancia de otro taxón que interactúe con nuestra especie de interés, etc. Nos puede interesar en primer lugar analizar si están correlacionadas nuestras potenciales variables explicativas. Luego, podemos intentar ajustar un buen modelo que explique cómo varía nuestra variable respuesta en función de varias variables explicativas. Ese es el camino que recorreremos en esta Unidad.


## Caso de estudio

El sitio de estudio es la turbera de Rancho Hambre (Tierra del Fuego, Argentina), en donde entre octubre de 2009 y abril de 2010 se realizaron muestreos en lagunas (ver [artículo](https://link.springer.com/article/10.1007/s10750-016-2969-2) de Quiroga _et al._ 2017, y la [tesis doctoral](http://hdl.handle.net/20.500.12110/tesis_n5433_Quiroga) de María Victoria Quiroga (FCEN-UBA)). En cinco lagunas (RH1 a RH5), se realizaron muestreos estacionales (octubre, diciembre, febrero y abril), es decir, tenemos un total de 20 observaciones. Contamos con dos tablas de datos:

+ **Datos limnológicos físicoquímicos.** Los datos corresponden a los sitios de muestreo de orilla de los cuerpos de agua. Los parámetros limnológicos fueron estimados según lo descripto en la sección _Sampling regime, physical-chemical analyses_ de Materiales y Métodos en Quiroga _et al._ (2017). Los [datos físicoquímicos están disponibles en el repositorio digital de CONICET](http://hdl.handle.net/11336/201874). Consideraremos a las variables medidas como variables explicativas: pH, conductividad, oxígeno disuelto, dureza total, DOC, DIN, nitrógeno total, DRP, fósforo total, ag(440), SUVA254 e índice MS; todas ellas variables numéricas. Por otro lado tenemos variables categóricas que discutiremos cómo considerar: la laguna (cinco niveles) y la fecha de muestreo (cuatro niveles). 

+ **Biomasa de morfotipos bacterianos**. La biomasa de bacterias heterótrofas se estimó utilizando microscopía de epifluorescencia y análisis de imágenes. Para detalles ver sección _Bacterioplankton morphotypes_ de Materiales y Métodos en Quiroga _et al._ (2017). Los [datos de morfotipos bacterianos están disponibles en el repositorio digital de CONICET](http://hdl.handle.net/11336/201811). En la tabla se cuenta con la biomasa de seis morfotipos bacterianos distintos. Podemos tomar a la biomasa de cada morfotipo como una variable respuesta por separado, o bien considerar para un primer análisis a la biomasa total de bacterias heterótrofas como la variable a modelar.


## Leer y emprolijar los datos

Siempre debemos mirar nuestra tabla de datos antes de empezar: detectar posibles errores o datos faltantes, retocar los nombres de las variables, agrupar filas y columnas si corresponde, etc. En nuestro caso además tenemos por un lado las variables físico-químicas y por el otro lado las variables biológicas, pero para los análisis es conveniente tener todos los datos en un mismo _dataframe_.

Empezamos leyendo las bases de datos en R. Es una buena práctica guardar los archivos en una carpeta llamada _data_, dentro de la carpeta de nuestro proyecto. Tenemos los datos en formato _csv_ en [GitHub Limno-con-R/CILCAL2023](https://github.com/Limno-con-R/CILCAL2023/tree/main/datasets) **RH_abiotic_data.csv** y **RH_morpho_biomass.csv** (son casi iguales a los disponibles en el repositorio CONICET, se cambió un guión bajo por un espacio en una de las tablas). Cargamos los datos indicando que se saltee (_skip_) la primera fila (¿por qué?).


```r
library(readr)

datos_fq <- read_csv("data/RH_abiotic_data.csv", skip = 1)

datos_bacterias <- read_csv("data/RH_morpho_biomass.csv", skip = 1)
```

Ahora vamos a ver los datos. Recordar o anotar cuántas filas y columnas tiene cada _dataframe_.


```r
datos_fq
```

```
## # A tibble: 20 x 16
##    ID    Pool  Site      Date     pH Condu~1 Disso~2 Total~3   DOC   DIN Total~4
##    <chr> <chr> <chr>     <chr> <dbl>   <dbl>   <dbl>   <dbl> <dbl> <dbl>   <dbl>
##  1 1O    RH1   shore     Octo~  4.91    13.4    NA      21.2   5.6  83.6    1870
##  2 2O    RH2   shore     Octo~  5.52     8.7    NA      46.2   5.1  41.3    1980
##  3 3O    RH3   shore     Octo~  4.72    15.2    NA      13.8   2.8  10      1980
##  4 4O    RH4   north sh~ Octo~  5.09    11.8    NA      33.2   5.7  55.3    1650
##  5 5O    RH5   shore     Octo~  4.82     5.5    NA      25.8   3.9  54.4    3410
##  6 1D    RH1   shore     Dece~  6.39    19.1    11.0    38.5   7.2  21.1    1320
##  7 2D    RH2   shore     Dece~  4.75    22.6    11.0    42.1   7.7  12.1    1870
##  8 3D    RH3   shore     Dece~  4.67    26.7    11.7    18.9  13.4  22.6    5720
##  9 4D    RH4   north sh~ Dece~  6.75    25.6    11.7    24.8   5.3  34      7480
## 10 5D    RH5   shore     Dece~  4.65    23      11.5    20.1  11    11.2   10230
## 11 1F    RH1   shore     Febr~  5.9     21.4    10.4    39.5  12.7  53      8030
## 12 2F    RH2   shore     Febr~  5.19    22.6    10.2    34     5.9  30.3   10010
## 13 3F    RH3   shore     Febr~  4.66    25.7    10.6    32    12.5  45.3   11330
## 14 4F    RH4   north sh~ Febr~  6.65    26.8    10.9    36.6   4     0      5610
## 15 5F    RH5   shore     Febr~  4.82    27.2    10.4    24.5   7.1  22.3    8250
## 16 1A    RH1   shore     Apri~  7.1     21.4    10.7    25.3   7.6  23.1    9790
## 17 2A    RH2   shore     Apri~  4.88    24      11.4    20.3   7.5   0     11110
## 18 3A    RH3   shore     Apri~  5.4     28.5    10.4    18.8  12.7 103.     8910
## 19 4A    RH4   north sh~ Apri~  6.2     28.5    11.2    27.7   5.3  43      3630
## 20 5A    RH5   shore     Apri~  5.43    27.1    11.2    24.2  11.6  73.2    3740
## # ... with 5 more variables: DRP <dbl>, `Total phosphorus` <dbl>,
## #   `ag(440)` <dbl>, SUVA254 <dbl>, `MS index` <dbl>, and abbreviated variable
## #   names 1: Conductivity, 2: `Dissolved oxigen`, 3: `Total hardness`,
## #   4: `Total nitrogen`
```


```r
datos_bacterias
```

```
## # A tibble: 20 x 10
##    ID    Pool  Site        Date  Filam~1 Large~2 Vibri~3 Large~4 Small~5 Small~6
##    <chr> <chr> <chr>       <chr>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1 1O    RH1   shore       Octo~       0   16921    5315   12676    1747   25994
##  2 2O    RH2   shore       Octo~    1435    5645    2384    7381     888   12075
##  3 3O    RH3   shore       Octo~   13307    1597     392    8131     708   13222
##  4 4O    RH4   north shore Octo~       0    8085    3499    7534    2028   25353
##  5 5O    RH5   shore       Octo~       0    1921     451   15662     666   31621
##  6 1D    RH1   shore       Dece~       0   12374    9228   21906    4503   38421
##  7 2D    RH2   shore       Dece~       0   22924    5933   34663    6687   77581
##  8 3D    RH3   shore       Dece~       0   12309    3471   46807    8694  139006
##  9 4D    RH4   north shore Dece~   44419   76819   19692   44391   24931  156132
## 10 5D    RH5   shore       Dece~       0   66952   23046  134707    2949  102890
## 11 1F    RH1   shore       Febr~    3177    6414    7533   22218    4013   34495
## 12 2F    RH2   shore       Febr~       0    7263    4879   25886    4098  137049
## 13 3F    RH3   shore       Febr~    3886    9283    1294   17402    2522   93008
## 14 4F    RH4   north shore Febr~    5860   54624   20331   33949   12469   58990
## 15 5F    RH5   shore       Febr~       0   21324   17482   24397    7064  106885
## 16 1A    RH1   shore       Apri~   12331   21098   11615   16083    6592   33478
## 17 2A    RH2   shore       Apri~       0    6950    4028   22525    1395   94037
## 18 3A    RH3   shore       Apri~       0   12394   17281   18608    4678  121090
## 19 4A    RH4   north shore Apri~       0     901    2484    3455     642   50957
## 20 5A    RH5   shore       Apri~       0    7626    3338   17231    2016   60287
## # ... with abbreviated variable names 1: Filaments, 2: Large_rods,
## #   3: Vibrio_shaped, 4: Large_cocci, 5: Small_rods, 6: Small_cocci
```

Para unir estas tablas en una sola, debemos buscar una variable o columna que sea un indicador único de cada dato, y que sea equivalente en ambas tablas. ¿Cuál les parece que es?

Con ese indicador como nexo, haremos una **unión exterior completa** o _full_join_. Esto se debe a que queremos mantener todos los registros de nuestras tablas aunque para alguna laguna --quizás-- no hayamos podido medir los parámetros físicoquímicos o no hayamos podido medir la biomasa bacteriana. Para saber más sobre funciones que permiten relacionar conjuntos de datos, recomendamos consultar el capítulo [13. Datos relacionales](http://es.r4ds.hadley.nz/datos-relacionales.html) (en particular _13.4.1. Entendiendo las uniones_) en el libro traducido a castellano [R para Ciencias de Datos](http://es.r4ds.hadley.nz/), de Hadley Wickham y Garret Grolemund (2017).

Hacemos la unión e inspeccionamos la tabla (¿cuántas filas y columnas tiene?).


```r
library(tidyverse)
```


```r
datos_RH <- full_join(datos_bacterias, datos_fq, by = "ID")
datos_RH 
```

```
## # A tibble: 20 x 25
##    ID    Pool.x Site.x    Date.x Filam~1 Large~2 Vibri~3 Large~4 Small~5 Small~6
##    <chr> <chr>  <chr>     <chr>    <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1 1O    RH1    shore     Octob~       0   16921    5315   12676    1747   25994
##  2 2O    RH2    shore     Octob~    1435    5645    2384    7381     888   12075
##  3 3O    RH3    shore     Octob~   13307    1597     392    8131     708   13222
##  4 4O    RH4    north sh~ Octob~       0    8085    3499    7534    2028   25353
##  5 5O    RH5    shore     Octob~       0    1921     451   15662     666   31621
##  6 1D    RH1    shore     Decem~       0   12374    9228   21906    4503   38421
##  7 2D    RH2    shore     Decem~       0   22924    5933   34663    6687   77581
##  8 3D    RH3    shore     Decem~       0   12309    3471   46807    8694  139006
##  9 4D    RH4    north sh~ Decem~   44419   76819   19692   44391   24931  156132
## 10 5D    RH5    shore     Decem~       0   66952   23046  134707    2949  102890
## 11 1F    RH1    shore     Febru~    3177    6414    7533   22218    4013   34495
## 12 2F    RH2    shore     Febru~       0    7263    4879   25886    4098  137049
## 13 3F    RH3    shore     Febru~    3886    9283    1294   17402    2522   93008
## 14 4F    RH4    north sh~ Febru~    5860   54624   20331   33949   12469   58990
## 15 5F    RH5    shore     Febru~       0   21324   17482   24397    7064  106885
## 16 1A    RH1    shore     April~   12331   21098   11615   16083    6592   33478
## 17 2A    RH2    shore     April~       0    6950    4028   22525    1395   94037
## 18 3A    RH3    shore     April~       0   12394   17281   18608    4678  121090
## 19 4A    RH4    north sh~ April~       0     901    2484    3455     642   50957
## 20 5A    RH5    shore     April~       0    7626    3338   17231    2016   60287
## # ... with 15 more variables: Pool.y <chr>, Site.y <chr>, Date.y <chr>,
## #   pH <dbl>, Conductivity <dbl>, `Dissolved oxigen` <dbl>,
## #   `Total hardness` <dbl>, DOC <dbl>, DIN <dbl>, `Total nitrogen` <dbl>,
## #   DRP <dbl>, `Total phosphorus` <dbl>, `ag(440)` <dbl>, SUVA254 <dbl>,
## #   `MS index` <dbl>, and abbreviated variable names 1: Filaments,
## #   2: Large_rods, 3: Vibrio_shaped, 4: Large_cocci, 5: Small_rods,
## #   6: Small_cocci
```

Las variables Pool, Site y Date se repiten en ambas tablas, por eso aparecen como Pool.x y Pool.y (por ejemplo). Dado que son idénticas, tenemos la opción de eliminar estas variables de alguna de las dos tablas, o bien incluirlas como parte de la información de nexo.


```r
datos_RH <- full_join(datos_bacterias, datos_fq, by = c("ID", "Pool", "Site", "Date"))
datos_RH 
```

```
## # A tibble: 20 x 22
##    ID    Pool  Site  Date  Filam~1 Large~2 Vibri~3 Large~4 Small~5 Small~6    pH
##    <chr> <chr> <chr> <chr>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl> <dbl>
##  1 1O    RH1   shore Octo~       0   16921    5315   12676    1747   25994  4.91
##  2 2O    RH2   shore Octo~    1435    5645    2384    7381     888   12075  5.52
##  3 3O    RH3   shore Octo~   13307    1597     392    8131     708   13222  4.72
##  4 4O    RH4   nort~ Octo~       0    8085    3499    7534    2028   25353  5.09
##  5 5O    RH5   shore Octo~       0    1921     451   15662     666   31621  4.82
##  6 1D    RH1   shore Dece~       0   12374    9228   21906    4503   38421  6.39
##  7 2D    RH2   shore Dece~       0   22924    5933   34663    6687   77581  4.75
##  8 3D    RH3   shore Dece~       0   12309    3471   46807    8694  139006  4.67
##  9 4D    RH4   nort~ Dece~   44419   76819   19692   44391   24931  156132  6.75
## 10 5D    RH5   shore Dece~       0   66952   23046  134707    2949  102890  4.65
## 11 1F    RH1   shore Febr~    3177    6414    7533   22218    4013   34495  5.9 
## 12 2F    RH2   shore Febr~       0    7263    4879   25886    4098  137049  5.19
## 13 3F    RH3   shore Febr~    3886    9283    1294   17402    2522   93008  4.66
## 14 4F    RH4   nort~ Febr~    5860   54624   20331   33949   12469   58990  6.65
## 15 5F    RH5   shore Febr~       0   21324   17482   24397    7064  106885  4.82
## 16 1A    RH1   shore Apri~   12331   21098   11615   16083    6592   33478  7.1 
## 17 2A    RH2   shore Apri~       0    6950    4028   22525    1395   94037  4.88
## 18 3A    RH3   shore Apri~       0   12394   17281   18608    4678  121090  5.4 
## 19 4A    RH4   nort~ Apri~       0     901    2484    3455     642   50957  6.2 
## 20 5A    RH5   shore Apri~       0    7626    3338   17231    2016   60287  5.43
## # ... with 11 more variables: Conductivity <dbl>, `Dissolved oxigen` <dbl>,
## #   `Total hardness` <dbl>, DOC <dbl>, DIN <dbl>, `Total nitrogen` <dbl>,
## #   DRP <dbl>, `Total phosphorus` <dbl>, `ag(440)` <dbl>, SUVA254 <dbl>,
## #   `MS index` <dbl>, and abbreviated variable names 1: Filaments,
## #   2: Large_rods, 3: Vibrio_shaped, 4: Large_cocci, 5: Small_rods,
## #   6: Small_cocci
```

Ahora vamos a mejorar los nombres de las columnas. Una buena práctica es elegir siempre el mismo tipo de nomenclatura. Hay varias convenciones pero las más elegidas en el mundo de R son **_snake_** y **_camel_**. La nomenclatura _snake_ implica usar guiones bajos como separador de palabras y, generalmente, todas las letras en minúscula. La nomenclatura _camel_ implica, en vez de usar guiones, usar a las mayúsculas como indicador de que hay una palabra nueva, y siempre dejar a la primera palabra en minúscula. Si nuestra variable se llama "Fecha de muestreo", la nomenclatura sería _fecha_de_muestreo_ o _fechaDeMuestreo_, de acuerdo a la convención elegida.

En la tabla que tenemos, hay una mezcla de criterios: se usan guiones bajos pero a la vez las variables empiezan con mayúscula. Vamos a ajustar. Vamos a pasar todas las variables al formato _snake_ con una función muy útil, que también sirve para nombres de columnas más desprolijos (por ejemplo, si tenemos otros signos de puntuación en los nombres). 


```r
library(janitor)
```



```r
datos_RH <- clean_names(datos_RH)
datos_RH 
```

```
## # A tibble: 20 x 22
##    id    pool  site  date  filam~1 large~2 vibri~3 large~4 small~5 small~6   p_h
##    <chr> <chr> <chr> <chr>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl> <dbl>
##  1 1O    RH1   shore Octo~       0   16921    5315   12676    1747   25994  4.91
##  2 2O    RH2   shore Octo~    1435    5645    2384    7381     888   12075  5.52
##  3 3O    RH3   shore Octo~   13307    1597     392    8131     708   13222  4.72
##  4 4O    RH4   nort~ Octo~       0    8085    3499    7534    2028   25353  5.09
##  5 5O    RH5   shore Octo~       0    1921     451   15662     666   31621  4.82
##  6 1D    RH1   shore Dece~       0   12374    9228   21906    4503   38421  6.39
##  7 2D    RH2   shore Dece~       0   22924    5933   34663    6687   77581  4.75
##  8 3D    RH3   shore Dece~       0   12309    3471   46807    8694  139006  4.67
##  9 4D    RH4   nort~ Dece~   44419   76819   19692   44391   24931  156132  6.75
## 10 5D    RH5   shore Dece~       0   66952   23046  134707    2949  102890  4.65
## 11 1F    RH1   shore Febr~    3177    6414    7533   22218    4013   34495  5.9 
## 12 2F    RH2   shore Febr~       0    7263    4879   25886    4098  137049  5.19
## 13 3F    RH3   shore Febr~    3886    9283    1294   17402    2522   93008  4.66
## 14 4F    RH4   nort~ Febr~    5860   54624   20331   33949   12469   58990  6.65
## 15 5F    RH5   shore Febr~       0   21324   17482   24397    7064  106885  4.82
## 16 1A    RH1   shore Apri~   12331   21098   11615   16083    6592   33478  7.1 
## 17 2A    RH2   shore Apri~       0    6950    4028   22525    1395   94037  4.88
## 18 3A    RH3   shore Apri~       0   12394   17281   18608    4678  121090  5.4 
## 19 4A    RH4   nort~ Apri~       0     901    2484    3455     642   50957  6.2 
## 20 5A    RH5   shore Apri~       0    7626    3338   17231    2016   60287  5.43
## # ... with 11 more variables: conductivity <dbl>, dissolved_oxigen <dbl>,
## #   total_hardness <dbl>, doc <dbl>, din <dbl>, total_nitrogen <dbl>,
## #   drp <dbl>, total_phosphorus <dbl>, ag_440 <dbl>, suva254 <dbl>,
## #   ms_index <dbl>, and abbreviated variable names 1: filaments, 2: large_rods,
## #   3: vibrio_shaped, 4: large_cocci, 5: small_rods, 6: small_cocci
```

El nombre de la variable pH quedó un poco raro, lo vamos a cambiar. A continuación se pide el nombre de las columnas de datos_RH, luego específicamente el de la columna 11, y luego lo cambiamos.


```r
colnames(datos_RH)
```

```
##  [1] "id"               "pool"             "site"             "date"            
##  [5] "filaments"        "large_rods"       "vibrio_shaped"    "large_cocci"     
##  [9] "small_rods"       "small_cocci"      "p_h"              "conductivity"    
## [13] "dissolved_oxigen" "total_hardness"   "doc"              "din"             
## [17] "total_nitrogen"   "drp"              "total_phosphorus" "ag_440"          
## [21] "suva254"          "ms_index"
```

```r
colnames(datos_RH)[11]
```

```
## [1] "p_h"
```

```r
colnames(datos_RH)[11] <- "ph"
colnames(datos_RH)
```

```
##  [1] "id"               "pool"             "site"             "date"            
##  [5] "filaments"        "large_rods"       "vibrio_shaped"    "large_cocci"     
##  [9] "small_rods"       "small_cocci"      "ph"               "conductivity"    
## [13] "dissolved_oxigen" "total_hardness"   "doc"              "din"             
## [17] "total_nitrogen"   "drp"              "total_phosphorus" "ag_440"          
## [21] "suva254"          "ms_index"
```
Finalmente, vamos a agregar una nueva columna con la biomasa total bacteriana. Existen múltiples métodos para hacer esto, vamos a ver dos, uno con la sintaxis base de R y otro con la sintaxis de _tidyverse_. Para conocer más sobre los tipos de sintaxis de R, recomendamos la hoja de referencia [Comparación de sintaxis de R](https://raw.githubusercontent.com/rstudio/cheatsheets/main/translations/spanish/syntax_es.pdf), por Amelia McNamara (traducida al castellano por RLadies).


```r
#Con la sintaxis base
datos_bio_opcion1 <- datos_RH #duplico el dataframe para no sobreescribirlo y comparar los resultados

datos_bio_opcion1$bio_total <- datos_bio_opcion1$filaments + datos_bio_opcion1$large_rods + datos_bio_opcion1$vibrio_shaped + datos_bio_opcion1$large_cocci + datos_bio_opcion1$small_rods + datos_bio_opcion1$small_cocci

datos_bio_opcion1
```

```
## # A tibble: 20 x 23
##    id    pool  site  date  filam~1 large~2 vibri~3 large~4 small~5 small~6    ph
##    <chr> <chr> <chr> <chr>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl> <dbl>
##  1 1O    RH1   shore Octo~       0   16921    5315   12676    1747   25994  4.91
##  2 2O    RH2   shore Octo~    1435    5645    2384    7381     888   12075  5.52
##  3 3O    RH3   shore Octo~   13307    1597     392    8131     708   13222  4.72
##  4 4O    RH4   nort~ Octo~       0    8085    3499    7534    2028   25353  5.09
##  5 5O    RH5   shore Octo~       0    1921     451   15662     666   31621  4.82
##  6 1D    RH1   shore Dece~       0   12374    9228   21906    4503   38421  6.39
##  7 2D    RH2   shore Dece~       0   22924    5933   34663    6687   77581  4.75
##  8 3D    RH3   shore Dece~       0   12309    3471   46807    8694  139006  4.67
##  9 4D    RH4   nort~ Dece~   44419   76819   19692   44391   24931  156132  6.75
## 10 5D    RH5   shore Dece~       0   66952   23046  134707    2949  102890  4.65
## 11 1F    RH1   shore Febr~    3177    6414    7533   22218    4013   34495  5.9 
## 12 2F    RH2   shore Febr~       0    7263    4879   25886    4098  137049  5.19
## 13 3F    RH3   shore Febr~    3886    9283    1294   17402    2522   93008  4.66
## 14 4F    RH4   nort~ Febr~    5860   54624   20331   33949   12469   58990  6.65
## 15 5F    RH5   shore Febr~       0   21324   17482   24397    7064  106885  4.82
## 16 1A    RH1   shore Apri~   12331   21098   11615   16083    6592   33478  7.1 
## 17 2A    RH2   shore Apri~       0    6950    4028   22525    1395   94037  4.88
## 18 3A    RH3   shore Apri~       0   12394   17281   18608    4678  121090  5.4 
## 19 4A    RH4   nort~ Apri~       0     901    2484    3455     642   50957  6.2 
## 20 5A    RH5   shore Apri~       0    7626    3338   17231    2016   60287  5.43
## # ... with 12 more variables: conductivity <dbl>, dissolved_oxigen <dbl>,
## #   total_hardness <dbl>, doc <dbl>, din <dbl>, total_nitrogen <dbl>,
## #   drp <dbl>, total_phosphorus <dbl>, ag_440 <dbl>, suva254 <dbl>,
## #   ms_index <dbl>, bio_total <dbl>, and abbreviated variable names
## #   1: filaments, 2: large_rods, 3: vibrio_shaped, 4: large_cocci,
## #   5: small_rods, 6: small_cocci
```


```r
#Con la sintaxis tidyverse, la cual usa el operador _pipe_ ("la pipa") %>%

datos_bio_opcion2 <- datos_RH %>%
  mutate(bio_total = filaments + large_rods + vibrio_shaped + large_cocci + small_rods + small_cocci, .before=filaments)

datos_bio_opcion2
```

```
## # A tibble: 20 x 23
##    id    pool  site        date  bio_t~1 filam~2 large~3 vibri~4 large~5 small~6
##    <chr> <chr> <chr>       <chr>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1 1O    RH1   shore       Octo~   62653       0   16921    5315   12676    1747
##  2 2O    RH2   shore       Octo~   29808    1435    5645    2384    7381     888
##  3 3O    RH3   shore       Octo~   37357   13307    1597     392    8131     708
##  4 4O    RH4   north shore Octo~   46499       0    8085    3499    7534    2028
##  5 5O    RH5   shore       Octo~   50321       0    1921     451   15662     666
##  6 1D    RH1   shore       Dece~   86432       0   12374    9228   21906    4503
##  7 2D    RH2   shore       Dece~  147788       0   22924    5933   34663    6687
##  8 3D    RH3   shore       Dece~  210287       0   12309    3471   46807    8694
##  9 4D    RH4   north shore Dece~  366384   44419   76819   19692   44391   24931
## 10 5D    RH5   shore       Dece~  330544       0   66952   23046  134707    2949
## 11 1F    RH1   shore       Febr~   77850    3177    6414    7533   22218    4013
## 12 2F    RH2   shore       Febr~  179175       0    7263    4879   25886    4098
## 13 3F    RH3   shore       Febr~  127395    3886    9283    1294   17402    2522
## 14 4F    RH4   north shore Febr~  186223    5860   54624   20331   33949   12469
## 15 5F    RH5   shore       Febr~  177152       0   21324   17482   24397    7064
## 16 1A    RH1   shore       Apri~  101197   12331   21098   11615   16083    6592
## 17 2A    RH2   shore       Apri~  128935       0    6950    4028   22525    1395
## 18 3A    RH3   shore       Apri~  174051       0   12394   17281   18608    4678
## 19 4A    RH4   north shore Apri~   58439       0     901    2484    3455     642
## 20 5A    RH5   shore       Apri~   90498       0    7626    3338   17231    2016
## # ... with 13 more variables: small_cocci <dbl>, ph <dbl>, conductivity <dbl>,
## #   dissolved_oxigen <dbl>, total_hardness <dbl>, doc <dbl>, din <dbl>,
## #   total_nitrogen <dbl>, drp <dbl>, total_phosphorus <dbl>, ag_440 <dbl>,
## #   suva254 <dbl>, ms_index <dbl>, and abbreviated variable names 1: bio_total,
## #   2: filaments, 3: large_rods, 4: vibrio_shaped, 5: large_cocci,
## #   6: small_rods
```

La ventaja de la segunda opción es que podemos elegir dónde agregar la nueva columna (antes de la columna _filaments_, por ejemplo). Así que nos quedaremos con esta tabla para los siguientes pasos. De paso, borramos los dataframes que ya no necesitamos.


```r
datos_RH <- datos_bio_opcion2

rm(datos_bio_opcion2)
rm(datos_bio_opcion1)
```

Un último paso es informar que las variables pool y date son categóricas (factores). En el caso de date, además es adecuado ordenar los niveles de los factores, tanto para algunos análisis que consideren el orden de los meses (aquí nos exceden) como para mejorar la forma en que resumimos los resultados en una tabla o en un gráfico.


```r
datos_RH$pool <- factor(datos_RH$pool) #en este caso no indicamos los niveles, ya que el orden alfabético es adecuado

datos_RH$date <- factor(datos_RH$date, levels = c("October_2009", "December_2009", "February_2010", "April_2010"))
```

¡Listo! Podemos empezar con los análisis.

## Correlaciones

Primero vamos a realizar análisis rápidos de todos los pares de variables con la librería _GGally_. 


```r
library(GGally)
```


### Entre biomasa de morfotipos bacterianos

Vamos a analizar en primer lugar las variables biológicas, para lo cual generamos un nuevo dataframe que es un subconjunto de los anteriores.


```r
datos_bacterias <- datos_RH[,5:11]
  
ggpairs(datos_bacterias, title="Correlograma entre la biomasa de morfotipos bacterianos") 
```

![](02-RegMult_files/figure-latex/correlo1-1.pdf)<!-- --> 
Obtenemos una matriz de gráficos, en la que cada celda expresa la correlacion de un par de variables. En este caso, todas las variables son continuas, por lo que observamos que los gráficos en la parte inferior de la matriz son de dispersión (puntos).

En las diagonales, observamos histogramas suavizados para cada variable. Por ejemplo, comparemos el histograma de la biomasa total.


```r
hist(datos_bacterias$bio_total, breaks=10, col="lightblue", xlab="Biomasa total de bacterias heterótrofas", ylab="Frecuencia", main="")
```

![](02-RegMult_files/figure-latex/hist-1.pdf)<!-- --> 

En la parte superior, se muestra el coeficiente de correlación para cada par de variables y asteriscos indicando su nivel de significancia. Podemos evaluar el par que nos interese con una función, para tener información numérica más detallada.


```r
cor.test(datos_bacterias$vibrio_shaped, datos_bacterias$large_rods)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  datos_bacterias$vibrio_shaped and datos_bacterias$large_rods
## t = 6.3382, df = 18, p-value = 5.68e-06
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.6144454 0.9311213
## sample estimates:
##       cor 
## 0.8310104
```
Aquí observamos que el coeficiente de la correlación de Pearson es r = 0.8310, igual al que observamos en el correlograma. Además obtenemos un intervalo de confianza y un p-valor. Sin embargo, la correlación de Pearson supone que la distribución de las variables es normal. Evaluemos la normalidad de estas dos variables (deberíamos analizarlo para todas).


```r
shapiro.test(datos_bacterias$vibrio_shaped)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  datos_bacterias$vibrio_shaped
## W = 0.8493, p-value = 0.005189
```

```r
shapiro.test(datos_bacterias$large_rods)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  datos_bacterias$large_rods
## W = 0.71437, p-value = 5.866e-05
```
Dado que se rechaza el supuesto de distribución normal, tenemos dos opciones. La primera es hacer una correlación de Spearman, la cual es adecuada si suponemos que la relación entre las variables es monotónica (las variables siempre crecen o decrecen). La desventaja es que, en la fórmula de cómputo del método, las variables se ordenan en un rango y pasan a ser ordinales. Agregamos a la misma función que el método sea Spearman (por defecto es Pearson, ver la ayuda de la función con _?cor.test_). El rho es de 0.8316.


```r
cor.test(datos_bacterias$vibrio_shaped, datos_bacterias$large_rods, method = "spearman")
```

```
## 
## 	Spearman's rank correlation rho
## 
## data:  datos_bacterias$vibrio_shaped and datos_bacterias$large_rods
## S = 224, p-value = 2.994e-07
## alternative hypothesis: true rho is not equal to 0
## sample estimates:
##       rho 
## 0.8315789
```

La segunda opción es hacer una correlación de Pearson por permutaciones. Indicamos entonces el número de permutaciones a generar o dejamos el valor por defecto que es 999. Puede tardar un poco ya que tiene que permutar. El coeficiente de correlación obtenido es de 0.8310.


```r
library(RVAideMemoire) #Si no encuentran esta librería en las computadoras del curso, pueden probar este paso en sus hogares. En caso de problemas para instalarla, instalar primero la librería mixOmics con estas instrucciones: http://www.bioconductor.org/packages/release/bioc/html/mixOmics.html  
```


```r
perm.cor.test(datos_bacterias$vibrio_shaped, datos_bacterias$large_rods, progress=FALSE)
```

```
## 
## 	Pearson's product-moment correlation - Permutation test
## 
## data:  datos_bacterias$vibrio_shaped and datos_bacterias$large_rods
## 999 permutations
## t = 6.3382, p-value = 0.002
## alternative hypothesis: true correlation is not equal to 0
## sample estimates:
##       cor 
## 0.8310104
```

En el correlograma, vamos a cambiar qué coeficiente se muestra. Dejaremos el de Spearman. Observar que para algunos de los pares de variables hay diferencias entre los coeficientes de Pearson y Spearman.


```r
datos_bacterias <- datos_RH[,5:11]
  
ggpairs(datos_bacterias, title="Correlograma entre la biomasa de morfotipos bacterianos", upper = list(continuous = wrap( "cor", method="spearman"), combo = "box_no_facet", discrete = "count", na =  "na")) 
```

![](02-RegMult_files/figure-latex/correlo2-1.pdf)<!-- --> 

Ahora visualizaremos la correlación como un mapa de calor.


```r
ggcorr(datos_bacterias, method = c("everything", "pearson")) 
```

![](02-RegMult_files/figure-latex/correlo3-1.pdf)<!-- --> 


```r
ggcorr(datos_bacterias, method = c("everything", "spearman")) 
```

![](02-RegMult_files/figure-latex/correlo4-1.pdf)<!-- --> 

### Entre variables abióticas

Ahora analizaremos las variables explicativas. Esto es útil ya que en el siguiente paso de realizar regresiones múltiples evitaremos colocar en el modelo dos variables muy correlacionadas entre sí, ya que son redundantes. Generamos un nuevo dataframe que es un subconjunto de los anteriores.

Podemos repetir el estudio de comparar correlaciones de Pearson, de Pearson por permutaciones y de Spearman. Aquí resumidamente pasaremos a los correlogramas visuales. ¿Qué pasa con el oxígeno disuelto? Tendremos cuidado con el uso de esta variable en los análisis siguientes.


```r
datos_fq <- datos_RH[,12:23]

ggcorr(datos_fq, method = c("everything", "spearman")) 
```

![](02-RegMult_files/figure-latex/correlo5-1.pdf)<!-- --> 
Si queremos tener un resumen numérico de los coeficientes de correlación, podemos generar una matriz.


```
## Warning: package 'Hmisc' was built under R version 4.2.2
```

```
## Loading required package: lattice
```

```
## Loading required package: survival
```

```
## Loading required package: Formula
```

```
## 
## Attaching package: 'Hmisc'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     src, summarize
```

```
## The following objects are masked from 'package:base':
## 
##     format.pval, units
```


```r
rcorr(as.matrix(datos_fq), type="spearman") 
```

```
##                     ph conductivity dissolved_oxigen total_hardness   doc   din
## ph                1.00         0.05            -0.03           0.39 -0.22  0.14
## conductivity      0.05         1.00             0.06          -0.31  0.41 -0.07
## dissolved_oxigen -0.03         0.06             1.00          -0.34 -0.03 -0.29
## total_hardness    0.39        -0.31            -0.34           1.00 -0.16  0.01
## doc              -0.22         0.41            -0.03          -0.16  1.00  0.16
## din               0.14        -0.07            -0.29           0.01  0.16  1.00
## total_nitrogen   -0.16         0.48            -0.24          -0.30  0.45 -0.15
## drp              -0.38        -0.09            -0.15          -0.60  0.24 -0.13
## total_phosphorus  0.19         0.52            -0.22          -0.29  0.13  0.13
## ag_440           -0.01         0.75            -0.32          -0.14  0.49  0.14
## suva254          -0.13         0.36            -0.63          -0.12  0.13 -0.12
## ms_index         -0.25         0.17            -0.27          -0.20  0.41 -0.18
##                  total_nitrogen   drp total_phosphorus ag_440 suva254 ms_index
## ph                        -0.16 -0.38             0.19  -0.01   -0.13    -0.25
## conductivity               0.48 -0.09             0.52   0.75    0.36     0.17
## dissolved_oxigen          -0.24 -0.15            -0.22  -0.32   -0.63    -0.27
## total_hardness            -0.30 -0.60            -0.29  -0.14   -0.12    -0.20
## doc                        0.45  0.24             0.13   0.49    0.13     0.41
## din                       -0.15 -0.13             0.13   0.14   -0.12    -0.18
## total_nitrogen             1.00  0.41             0.27   0.42    0.36     0.10
## drp                        0.41  1.00             0.28   0.00    0.25     0.33
## total_phosphorus           0.27  0.28             1.00   0.66    0.46     0.03
## ag_440                     0.42  0.00             0.66   1.00    0.59     0.29
## suva254                    0.36  0.25             0.46   0.59    1.00     0.27
## ms_index                   0.10  0.33             0.03   0.29    0.27     1.00
## 
## n
##                  ph conductivity dissolved_oxigen total_hardness doc din
## ph               20           20               15             20  20  20
## conductivity     20           20               15             20  20  20
## dissolved_oxigen 15           15               15             15  15  15
## total_hardness   20           20               15             20  20  20
## doc              20           20               15             20  20  20
## din              20           20               15             20  20  20
## total_nitrogen   20           20               15             20  20  20
## drp              20           20               15             20  20  20
## total_phosphorus 20           20               15             20  20  20
## ag_440           20           20               15             20  20  20
## suva254          20           20               15             20  20  20
## ms_index         20           20               15             20  20  20
##                  total_nitrogen drp total_phosphorus ag_440 suva254 ms_index
## ph                           20  20               20     20      20       20
## conductivity                 20  20               20     20      20       20
## dissolved_oxigen             15  15               15     15      15       15
## total_hardness               20  20               20     20      20       20
## doc                          20  20               20     20      20       20
## din                          20  20               20     20      20       20
## total_nitrogen               20  20               20     20      20       20
## drp                          20  20               20     20      20       20
## total_phosphorus             20  20               20     20      20       20
## ag_440                       20  20               20     20      20       20
## suva254                      20  20               20     20      20       20
## ms_index                     20  20               20     20      20       20
## 
## P
##                  ph     conductivity dissolved_oxigen total_hardness doc   
## ph                      0.8450       0.9094           0.0875         0.3417
## conductivity     0.8450              0.8292           0.1788         0.0760
## dissolved_oxigen 0.9094 0.8292                        0.2182         0.9142
## total_hardness   0.0875 0.1788       0.2182                          0.5120
## doc              0.3417 0.0760       0.9142           0.5120               
## din              0.5541 0.7813       0.2979           0.9824         0.4934
## total_nitrogen   0.5036 0.0318       0.3973           0.1926         0.0472
## drp              0.1009 0.6939       0.5901           0.0048         0.3147
## total_phosphorus 0.4266 0.0192       0.4402           0.2116         0.5765
## ag_440           0.9699 0.0001       0.2474           0.5434         0.0271
## suva254          0.5801 0.1200       0.0126           0.6090         0.5735
## ms_index         0.2850 0.4631       0.3369           0.3975         0.0714
##                  din    total_nitrogen drp    total_phosphorus ag_440 suva254
## ph               0.5541 0.5036         0.1009 0.4266           0.9699 0.5801 
## conductivity     0.7813 0.0318         0.6939 0.0192           0.0001 0.1200 
## dissolved_oxigen 0.2979 0.3973         0.5901 0.4402           0.2474 0.0126 
## total_hardness   0.9824 0.1926         0.0048 0.2116           0.5434 0.6090 
## doc              0.4934 0.0472         0.3147 0.5765           0.0271 0.5735 
## din                     0.5348         0.5760 0.5876           0.5560 0.6267 
## total_nitrogen   0.5348                0.0740 0.2464           0.0675 0.1193 
## drp              0.5760 0.0740                0.2333           0.9873 0.2787 
## total_phosphorus 0.5876 0.2464         0.2333                  0.0015 0.0393 
## ag_440           0.5560 0.0675         0.9873 0.0015                  0.0057 
## suva254          0.6267 0.1193         0.2787 0.0393           0.0057        
## ms_index         0.4556 0.6686         0.1489 0.8907           0.2144 0.2494 
##                  ms_index
## ph               0.2850  
## conductivity     0.4631  
## dissolved_oxigen 0.3369  
## total_hardness   0.3975  
## doc              0.0714  
## din              0.4556  
## total_nitrogen   0.6686  
## drp              0.1489  
## total_phosphorus 0.8907  
## ag_440           0.2144  
## suva254          0.2494  
## ms_index
```
  
  




## Regresiones múltiples

Intro

hay que estandarizar?

### Selección automática de variables

#### Upward: del modelo nulo, sumando variables explicativas

#### Forward: del modelo completo, quitando variables explicativas

### Selección manual de variables

#### Modelos univariados
Todos los modelos
AIC

#### Modelos de regresión múltiple
vif


### Evaluación e interpretación del modelo final
Evaluación de supuestos. Si no se cumplen, permutaciones
interpretación


## A modo de conclusión
