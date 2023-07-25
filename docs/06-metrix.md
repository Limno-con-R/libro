---
editor_options:
  markdown:
    wrap: sentence
---

# Paquete *metrix* {#metrix}

**Julieta Capeletti**[^1]

[^1]: [julieta.capeletti\@hotmail.com](mailto:julieta.capeletti@hotmail.com){.email}

Instituto Nacional de Limnología (INALI, UNL-CONICET), Laboratorio de Bentos

**Juan Manuel Cabrera**[^2]

[^2]: [juan.cabrera\@uner.edu.ar](mailto:juan.cabrera@uner.edu.ar){.email}

Instituto de Investigación y Desarrollo en Bioingeniería y Bioinformática (IBB, UNER-CONICET)

## ¿Que es Metrix?

*metrix* es un paquete de R de código abierto diseñado específicamente para evaluar la calidad del agua utilizando datos de densidad de macroinvertebrados [@cabrera2023].
Este paquete nos permite calcular una extensa cantidad de métricas e índices bióticos que evalúan la calidad del ambiente de manera rápida y sencilla.
Admite una resolución taxonómica heterogénea que permite cálculos en varios niveles taxonómicos y es capaz de leer hojas de cálculo biológicas a partir de grandes volúmenes de datos.
Este paquete facilita la manipulación de datos, simplificando el proceso mediante el uso de hojas de cálculo predefinidas en formato .csv.
Esto reduce la necesidad de software o paquetes adicionales, haciéndolo accesible y fácil de usar, incluso para usuarios sin conocimientos de programación.
Las futuras versiones del paquete pueden incluir otros índices métricos y bióticos, mejorando aún más sus capacidades.
Además, se puede desarrollar una interfaz fácil de usar para mejorar la usabilidad de la herramienta para usuarios no científicos, como las agencias gubernamentales responsables de evaluar el impacto ambiental de las actividades relacionadas con los seres humanos.

## Conjunto de datos de ejemplo

Antes de comenzar a probar *metrix*, vamos a descargar el conjunto de datos perteneciente a muestreos en la cuenca Cañada Carrizales localizada en la ecorregión Pampeana de Argentina (32°26'23.53 "S, 61°18'10.69"W), perteneciente al trabajo *Métricas basadas en macroinvertebrados como monitores de ambientes con uso de suelo agrícola: estudio preliminar en una cuenca pampeana* [@capeletti2019].
El conjunto de datos está disponible en la carpeta *data* del repositorio de GitHub de *metrix* <https://github.com/Dvelop-R/metrix>.

## Instalar *metrix*

*metrix* se encuentra disponible en el repositorio CRAN.
Primeramente se debe descargar e instalar utilizando el código `install.packages("metrix")`.
Seguidamente se debe cargar el paquete en el entorno de trabajo:


```r
library(metrix)
```

## ¿Que tipos de datos analiza *metrix*?

*metrix* utiliza un formato específico de tabla de datos.
La misma consiste en 8 columnas que representen la clasificación científica y funcional de los taxones:

-   Las primeras 7 columnas corresponden a *Clase*, *Orden*, *Familia*, *Subfamilia*, *Tribu*, *Género* y *Especie*.

-   La columna 8 indica el *grupo funcional* (FG) de los taxones, que puede ser *recolectores filtradores* (GF), *colectores recolectores* (GC), *depredadores* (P), *raspadores* (SCR), o *trituradores* (SHR).

-   Después de estas columnas, se debe incluir los *datos de densidad (individuos/m2)* de los macroinvertebrados para cada muestra de sitio.

Durante el proceso de carga de datos es muy común que se cometan errores de tipeo que pueden llevar a que no se pueda identificar de manera correcta a los taxa incluidos en el conjunto de datos.
*metrix* posee un sistema de autocorreción que puede asisitir al usuario en la identificación de estas fallas en la carga de datos.
Para ello utiliza una base de datos de nombres cientificos utilizados en la descripción de los taxones que son tenidos en cuenta a la hora de calcular las métricas e índices bióticos calculados por el paquete.
Este sistema de autocorrección informa al usuario sobre posibles errores de carga y, si el usuario lo indica, puede corregirlo de manera automática (con un criterio de similitud de palabra).

## Cargar datos de densidad de macroinvertebrados usando *metrix*

Para cargar los datos en el entorno de trabajo de R utilizamos la función `read_data()`.
Esta función recibe 3 parámatros:

-   *file_name*: dirección/nombre del archivo .csv que contiene los datos de densidad de macroinvertebrados.

-   *correct*: valor lógico que indica si se va a utilizar el sistema de autocorrección de datos.

-   *verbose*: valor lógico que indica si se debe mostrar mensajes descriptivos del proceso de carga.


```r
#IMPORTANTE: revisar la dirección del archivo ecological_data.csv. En este ejemplo estaría dentro del directorio de trabajo por lo que no es necesario indicar la ruta completa al archivo
datos_cargados<-read_data(file_name = "ecological_data.csv", correct = TRUE, verbose = FALSE)
```

```
## The word ampullaridae was not found in internal Family word database. Please check that entry.
```

```
## The autocorrect system replace the word ampullaridae for Ampullariidae.
```

En este ejemplo, los datos ecológicos utilizados contienen un error en la columna de *Familia*.
El sistema de autocorrección notificará al usuario sobre este error y, si el parámetro `correct` está seteado con el valor `TRUE`, reemplazará la palabra presuntamente erronea por otra similar hallada en la base de datos interna.

### ¿QUE PASA SI QUIERO CREAR MI PROPIO CONJUNTO DE DATOS?

*metrix* contiene una función que genera un archivo .CSV con una tabla cuyas columnas están debidamente etiquetadas para que pueda ser utilizada por el paquete.
Para generar este archivo en el espacio de trabajo, se debe correr el siguiente código:


```r
metrix_table_template()
```

```
##     Class   Order       Family    Subfamily       Tribe          Genus Species
## 1 Insecta Diptera Chironomidae Chironominae Tanytarsini Paratanytarsus        
##   FG Sample1 Sample2
## 1 GC       1       0
```

Una vez generado el archivo se puede editar el mismo utilizando un bloc de notas o una editor de hojas de cálculo.
Luego se puede cargar al entorno de trabajo de igual manera que en el paso anterior.

## Cálculo de metricas individuales y ecológicas

Para evaluar la calidad biológica del agua, es necesario examinar valores específicos que midan características clave de la comunidad de macroinvertebrados, las cuales son indicativas de su respuesta a degradaciones ambientales.
*metrix* incluye una amplia gama de cálculos tanto para métricas individuales como ecológicas: \* *Métricas individuales*: abarcan la riqueza, que indica la diversidad del conjunto de macroinvertebrados; las métricas de composición, que calculan la abundancia relativa de taxones específicos dentro de la comunidad en términos de porcentajes; y las métricas de densidad, que sirven como medidas universales utilizadas en todo tipo de estudios biológicos.
\* *Métricas ecológicas*: incluyen métricas tróficas (como la riqueza de depredadores, la riqueza de colectores recolectores, la densidad de trituradores y la densidad de raspadores), que actúan como indicadores de procesos complejos como las interacciones tróficas, la producción y la disponibilidad de fuentes de alimento.
Además, se incluyen medidas de tolerancia para indicar la sensibilidad de la comunidad y especies individuales a varios tipos de perturbaciones.

En este ejemplo vamos a calcular metricas de riqueza de nuestro conjunto de datos previamente cargados.
Para ello vamos a utilizar la función `rich_metrics()`, la cual recibe 4 parámetros:

-   *dataset*: conjunto de datos cargados con la función `read_data()`.
-   *store*: valor lógico que indica si el resultado debe ser almacenado en un archivo.
-   *dec_c*: caracter utilizado como separador de decimales.
-   *verbose*: valor lógico que indica si se debe mostrar mensajes descriptivos del calcúlo realizado.


```r
#Calculo de riqueza
rich_metrics(dataset = datos_cargados, store = FALSE, dec_c = ".", verbose = TRUE)
```

```
## Checking table format for Richness measures calculation...
```

```
##                    P1 P2 P3 P4 P5 P6 P7 P8 P9 P10
## n_taxa              4  7  7  7  6  6  4  6  8   4
## n_fam               4  3  6  5  6  6  4  6  7   3
## n_gen               4  6  4  5  5  6  4  6  6   4
## n_insec_fam         2  1  4  2  3  2  2  2  4   1
## n_non_insec_order   2  2  2  2  3  3  2  3  3   2
## n_dip_fam           1  1  3  2  2  1  1  1  3   1
## n_dip_gen           1  4  1  1  1  1  1  1  2   1
## n_dip_chir_gen      1  4  1  1  1  1  1  1  2   1
## n_chir_tax          1  4  1  1  1  1  1  1  2   1
## n_tany_tax          0  0  0  0  0  0  0  0  0   0
## n_stemp_tax         0  0  0  0  0  0  0  0  0   0
## n_non_chir_dip_tax  0  0  2  1  1  0  0  0  2   0
## n_mol_tax           1  1  1  2  1  2  1  2  1   1
## n_gastr_tax         1  1  1  2  1  2  1  2  1   1
## n_biv_tax           0  0  0  0  0  0  0  0  0   0
## n_crus_tax          0  0  0  0  1  1  1  1  1   0
## n_crusmol           1  1  1  2  2  3  2  3  2   1
## n_oligo_tax         1  2  2  3  1  1  0  1  1   2
## n_ephetrich         0  0  0  0  0  0  0  0  1   0
```

Los sitios P1, P7 y P10 son los que menor riqueza presentan, a diferencia de los demás sitios.
En estos sitios la riqueza se da principalmente por la presencia de los órdenes de Diptera, Mollusca, Crustacea y Oligochaeta.

También podemos calcular métricas de tolerancia utilizando la función `tol_metrics()`.
Esta función recibe 4 parámetros:

-   *dataset*: conjunto de datos cargados con la función `read_data()`.
-   *store*: valor lógico que indica si el resultado debe ser almacenado en un archivo.
-   *dec_c*: caracter utilizado como separador de decimales.
-   *verbose*:valor lógico que indica si se debe mostrar mensajes descriptivos del calcúlo realizado.


```r
#Calculo de metricas de tolerancia
tol_metrics(dataset = datos_cargados, store = FALSE, dec_c = ".", verbose = TRUE)
```

```
## Checking table format for Tolerance measures calculation...
```

```
##                 P1     P2     P3     P4     P5     P6    P7     P8    P9    P10
## r_oligochir 0.0066 0.0054 0.0278 0.6667 0.0357 0.0085 0.000 0.0313 0.100 3.3333
## r_oligoset  0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## r_tanychir  0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_lhoff 0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_bothr 0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_tubi  0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_dero  0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_prist 0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_chiro 0.0000 0.0000 0.0000 0.0000 0.0000 0.0000 0.000 0.0000 0.000 0.0000
## den_t_hele  0.5740 0.5740 0.5740 0.5740 0.5740 0.5740 0.574 0.5740 0.574 0.5740
```

La relación de la densidad de Oligochaeta/Chironomidae nos permitiría diferenciar los sitios de acuerdo a su gradiente de impacto.
Esta métrica disminuye a medida que aumenta ladegradación, por lo tanto los sitios con los valores más altos serían los más contaminados a diferencia de los sitios con los valores más bajo los cuales presentarían condiciones de referencia.
La métrica Heleobia/DT presenta el mismo valor en todos los sitios por lo tanto no nos estaría diferenciando los sitios de acuerdo a su gradiente de impacto antrópico.

## Cálculo de indices bióticos

Los índices bióticos comprenden una combinación de dos o tres métricas individuales o ecológicas, que se condensan en un único valor que resume las características biológicas de todos los organismos presentes.
Para los índices cualitativos, estas métricas incluyen la riqueza de taxones y la sensibilidad a la contaminación.
Para los índices cuantitativos se incorpora la abundancia (absoluta o relativa).
*metrix* incluye cálculos para los siguientes índices bióticos:

-   *BMWP* (Biological Monitoring Working Party, [@armitage1983]), *ASPT* (Average Score per Taxon, [@armitage1983]) y sus adaptaciones: *BMWP'* [@alba1988] y *BMWP''* [@loyola2000].
    Estos índices utilizan la presencia y abundancia de ciertas familias de macroinvertebrados.
    A cada familia se le asigna un valor de puntuación basado en su tolerancia a la contaminación (de 0 a 10).
    Las familias más sensibles a la contaminación reciben puntuaciones más altas, mientras que las familias más tolerantes reciben puntuaciones más bajas.
    Las puntuaciones de todas las familias de organismos presentes en una muestra de agua se suman para obtener el valor total del índice y determinar la calidad del agua.

-   *IMRP* (Índice de Macroinvertebrados en Ríos Pampeanos, [@rodrigues1999]).
    Este índice se basa en la presencia y abundancia de ciertos órdenes y familias de macroinvertebrados para determinar la calidad del agua.
    Al igual que los índices BMWP, a cada grupo se le asigna un valor de puntuación basado en su tolerancia a la contaminación (en este caso, de 0 a 1), siendo el valor final del índice la suma de todos los órdenes o familias presentes en la muestra.

-   *ICBrio* (Índice das Comunidades Benticas em Ríos, [@kuhlmann2012]).
    Este índice utiliza los valores de densidad total de la muestra y se basa en la tolerancia de los organismos a la contaminación y su respuesta a las perturbaciones ambientales.
    El índice asigna valores de puntuación a diferentes grupos taxonómicos (a nivel de orden, familia y género) de macroinvertebrados presentes en las muestras.
    Estas puntuaciones se suman luego para obtener un valor total del índice y determinar la calidad del agua.

En este ejemplo vamos a calcular los indices *BMWP* e *ICBrio*.
Para ello vamos a utilizar las funciones `bmwp_ind()` e `icbrio_ind()`.
Ambas funciones reciben 4 parámetros:

-   *dataset*: conjunto de datos cargados con la función `read_data()`.
-   *store*: valor lógico que indica si el resultado debe ser almacenado en un archivo.
-   *dec_c*: caracter utilizado como separador de decimales.
-   *verbose*:valor lógico que indica si se debe mostrar mensajes descriptivos del calcúlo realizado.


```r
#Calculo de BMWP
bmwp_ind(dataset = datos_cargados, store = FALSE, dec_c = ".", verbose = TRUE)
```

```
## Checking table format for BMWP and ASPT index calculation...
```

```
## $Ibmwp_n
##          P1  P2 P3  P4 P5 P6 P7 P8  P9 P10
## ind_bmwp  9 3.0  9 3.0  9 12  8 12 3.0 3.0
## ind_aspt  3 1.5  3 1.5  3  3  4  3 1.5 1.5
## 
## $Ibmwp_c
##                                P1                 P2                 P3
## ind_bmwp_class Class VII Very Bad Class VII Very Bad Class VII Very Bad
## ind_aspt_class       Class VI Bad Class VII Very Bad       Class VI Bad
##                                P4                 P5                 P6
## ind_bmwp_class Class VII Very Bad Class VII Very Bad Class VII Very Bad
## ind_aspt_class Class VII Very Bad       Class VI Bad       Class VI Bad
##                                P7                 P8                 P9
## ind_bmwp_class Class VII Very Bad Class VII Very Bad Class VII Very Bad
## ind_aspt_class   Class V Moderate       Class VI Bad Class VII Very Bad
##                               P10
## ind_bmwp_class Class VII Very Bad
## ind_aspt_class Class VII Very Bad
```

Los valores de BMWP y ASPT nos indican condiciones de mala calidad en los 10 puntos de muestreo.


```r
#Calculo de ICBrio
icbrio_ind(dataset = datos_cargados, store = FALSE, dec_c = ".", verbose = TRUE)
```

```
## Checking table format for ICbrio index calculation...
```

```
## $Icbrio_n
##                  P1 P2 P3 P4       P5 P6 P7       P8   P9      P10
## ind_icbrio 4.333333  4  4  4 3.666667  4  6 3.666667 2.75 4.333333
## 
## $Icbrio_c
##                   P1  P2  P3  P4  P5  P6  P7  P8      P9 P10
## ind_icbrio_class Bad Bad Bad Bad Bad Bad Bad Bad Regular Bad
```

De igual manera el indice ICBRio nos indica que 9 sitios presentan una condición biológica mala, mientras que 1 sitio (P9) presenta regulares condiciones.

## FUTURO DEL PAQUETE

Las futuras versiones del paquete podrían incluir otras métricas e índices bióticos, mejorando aún más sus capacidades.
Además, puede que se desarrolle una interfaz amigable para el usuario, mejorando la usabilidad de la herramienta para usuarios no científicos, como agencias gubernamentales encargadas de evaluar el impacto ambiental de actividades relacionadas con el ser humano.

## AGRADECIMIENTOS

Agradecemos a la Dra.
Florencia Zilli, Dra.
Mercedes Marchese y Dr. Joaquien Cochero por su ayuda y apoyo en la creación de esta herramienta.
