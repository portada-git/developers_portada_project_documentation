# Configuración para la extracción de datos
## Manual para desarrolladores: Extracción de información de noticias en el contexto del proyecto portada
Este manual se ha creado básicamente para ayudar a los desarrolladores del proyecto **_PortAda_** a configurar la aplicación *[jportada_boat_fact_extractor](https://github.com/portada-git/jportada_boat_fact_extractor)* para conseguir extraer los datos referentes a las embarcaciones llegadas a los diferentes puertos de estudio referenciadas en los periódicos estudiados.

## Consideraciones previas sobre la biblioteca _jportada_auto_news_extractor_lib_
La aplicación *[jportada_boat_fact_extractor](https://github.com/portada-git/jportada_boat_fact_extractor)*  está basada en la biblioteca *[jportada_auto_news_extractor_lib](https://github.com/portada-git/jportada_auto_news_extractor_lib)*, la cual presenta 3 funcionalidades necesarias para realizar la extracción en el contexto del proyecto portada.

### Funcionalidades de *jportada_auto_news_extractor_lib*
#### Ensamblador de archivos digitales
Esta funcionalidad permite unir, en un solo archivo, los múltiples textos pertenecientes a una misma unidad informativa, que el procesador OCR haya generado a partir de los periódicos digitalizados. Aunque la biblioteca permitiría usar otras metodologías, en el proyecto **_PortAda_** se decidió usar el enfoque basado en el nombre de los archivos generados por el procesador OCR. Durante el proceso de obtención del texto a partir de las imágenes de los periódicos analizados, las imágenes se fraccionan en bloques para facilitar su ordenación y transcripción textual. En el proyecto se ha optado por nombrar cada bloque con un nombre que permita identificar la fecha, el nombre del periódico, la edición del ejemplar (mañana, tarde, ...), así como el tipo de noticia, la página y el número de bloque procesado por el OCR. 

Este formato nos permite determinar que todos los archivos de texto pertenecientes a una misma fecha, periódico, y edición, pueden contener información relacionada de una misma unidad de información y, por tanto, necesitamos unirla en un solo texto para poder disponer de un texto bien ordenado. 

Además, al tiempo que se realiza la unión, se consigue obtener información extra que no se encuentra en la noticia, objeto de nuestro análisis y posterior extracción. No referimos a los datos indicados en el nombre: Fecha del ejemplar, lugar de edición, nombre del periódico, edición del ejemplar, tipo de noticia y páginas de las que se realizará la extracción. Dicha información será incorporada, al resto de datos extraídos. Por ello **es importante que el nombre de los archivos sigan el patrón descrito: AAAA_MM_DD_PUE_NP_E_T_PG_BLOC.txt, donde AAAA corresponde al año de edición, MM al mes y DD al día;  PUE indica el nombre del puerto al que hace referencia la noticia a extraer (BCN, BUE, HAB, MAR); NP se refiere al nombre del periódico (DB por diario de Barcelona, DM por diario de la marina, EN por el nacional, LP por la prensa, SM por le semafore de Marseille, ...); E indica la edición del ejemplar (M por mañana, T por tarde, N por noche o U por única) la posición T define el tipo de noticia y admite los valores: E por entradas de embarcaciones, M por manifiesto de descarga o 0 si no se conoce el contenido; la posiciones PG, contendrán el número de página, con dos dígitos de manera obligatoria, y los últimos 4 dígitos (BLOC) indicarán el número del bloque del que se ha transcrito el texto. Por ejemplo, el archivo 1854_04_25_BUE_EN_U_E_08_005.txt indicaría que se trata del quinto bloque de la página 8 de las noticias referidas a las entradas de buques al puerto de Buenos Aires del periódico _El Nacional_ de la única edición del día 25 de abril de 1854.

#### Cortador de los fragmentos objetivo 
Una vez unidos todos los bloques en un único cabe segregar el texto objetivo donde se encuentra la noticia o sección de la cual extraer la información del resto de texto. Esta funcionalidad de búsqueda y segregación de los fragmentos-objetivo de estudio,  Para ello, en el proyecto PortAda se usará el enfoque basado en expresiones regulares que delimiten el inicio y el final de la sección. La composición de estas expresiones regulares sigue la metodología usada por la biblioteca _jportada_auto_news_extractor_lib_ que se explicará más adelante. El objetivo de esta funcionalidad consiste en reducir las posibilidades de error en el momento de realizar la extracción, eliminando de forma previa los fragmentos de noticias y secciones ajenas a la información que se quiere extraer. De esta forma, el proceso de extracción se hará únicamente con el texto necesario, evitando así, posible ruido que pueda falsear e inducir a error de los datos extraídos.

#### Analizador para extraer la información
La última de las funcionalidades tiene como objetivo extraer los datos contenidos en el texto de las noticias, clasificándolo en categorías predefinidas a las que llamaremos campos. Estos son:

__model_version__: Indica la versión del nombre del campo model.

__publication_date__: Muestra la fecha del periódico

__publication_name__: Muestra el nombre del periódico

__publication_edition__: Indica la edición del periódico en caso de que haya más de una al día: M para mañana, T para tarde o N para noche. En caso de que haya una sola edición el valor será U (única).

__fact_type__: Es el tipo de noticia que se analiza. Puede tomar valores como E para entradas de buques o M para manifiestos de descarga.

__ship_departure_port__: Indica el puerto de salida del buque en este viaje

__ship_arrival_port__: Indica el puerto de llegada (Marsella, Buenos Aires, La Habana o Barcelona) del buque en este viaje. En la mayoría de los casos, esta información no aparece en las noticias y se deduce implícitamente según el periódico

__ship_departure_date__: Indica la fecha de salida del barco del puerto de salida

__ship_arrival_date__: Indica la fecha en la que el barco llegó al puerto de llegada (Marsella, Buenos Aires, La Habana o Barcelona)

__travel_arrival_moment_value__: Indica la hora de llegada al puerto. Puede expresarse como hora de llegada o como un periodo más amplio (mañana, tarde, noche, ...)

__ship_travel_time__: Indica el tiempo que el barco estuvo viajando desde el puerto de salida al puerto de llegada días u horas

__ship_travel_time_unit__: Indica la unidad de tiempo en la que se expresa la duración.

__ship_port_of_call_list__: Indica la lista de puertos (y opcionalmente más información como fechas de llegada o salida) en los que el barco había hecho escala durante su trayecto al puerto de llegada. Si la información de esta lista es solo el nombre de los puertos, la lista estará compuesta por nombres de puertos separados por comas. En caso contrario, cada información de escala se encerrará entre paréntesis cuadrados separados, también por comas, y dentro de cada uno de ellos se especificaran si fuera posible: el nombre del puerto, la fecha de llegada a ese puerto y la fecha de salida (de acuerdo a los 3 campos siguientes). 

__ship_port_of_call_place__: Muestra el nombre del puerto de un elemento de la lista de puertos de escala

__ship_port_of_call_arrival_date__: Muestra la fecha de llegada de un elemento de la lista de puertos de escala

__ship_port_of_call_departure_date__: Muestra la fecha de salida de un elemento de la lista de puertos de escala

__ship_type__: Describe el tipo de barco (bergantín, goleta,  vapor, etc.) que menciona el periódico

__ship_flag__: Hace referencia al nombre del país o región de la bandera del barco descrito por el periódico

__ship_name__: Indica el nombre del barco que normalmente se presenta completo, como se menciona en la fuente del periódico

__ship_tons__: Especifica la capacidad del barco en toneladas presentada como un valor numérico con la unidad de medida. En el caso de los barcos, esto siempre es lo mismo, ya que se refiere al tonelaje del buque. Este dato se da normalmente con abreviaturas como "ton." o "t."

__ship_master_role__: Hace referencia a la categoría de la persona que comanda el buque. Puede ser capitán o patrón, aunque en algunos casos también aparece piloto. Las abreviaturas que se utilizan para designarlos suelen ser “c” y “p”, respectivamente

__ship_master_name__: Es la identificación nominal de la persona que comanda el buque. Puede aparecer de varias formas, al menos lleva el apellido, precedido de su cargo (rol). Indica el apellido del capitán del buque, a menudo precedido de “cap.” o “c.”

__ship_agent__: Esta información puede indicar tanto el agente del buque, es decir, la persona que se encarga de las transacciones y la operación del buque, como el armador, es decir, la persona que es propietaria del buque o de parte del mismo. En ocasiones también puede hacer referencia al armador

__ship_crew__: Es el valor numérico de la tripulación del buque.

__ship_cargo_list__: Es la descripción de la lista con la información relativa a toda la carga transportada por el buque entrante (tipo de carga, cantidad, persona receptora de la carga, si la hay o “a la orden” en caso contrario, etc.). Inicialmente, se mostrará como descripción textual separada por comas, pero en una segunda fase, cada mercancía se descompondrá en los 6 siguientes campos.

__cargo_merchant__: Es la persona a la que iba destinada la carga, muchas veces será el comerciante que la había comprado y que se hizo cargo de ella en el momento de la descarga. Indica el destinatario de la carga, con ocasional mención a “divers” [varios/diversos].
En este caso vemos nombres de personas o empresas. Estos nombres tienen las mismas características y dificultades que el resto de denominaciones. En ocasiones los barcos llegaban a carga completa y estaban destinados a la misma persona, y en otros casos, cada carga tenía su destinatario. También aparece con frecuencia la expresión “a la orden”, que en principio es una carga para ser vendida a su llegada a puerto y que, por el contrario, no tiene un propietario anterior, más allá del propio capitán personalmente o por cuenta de alguien.

__cargo_type__: Expresa los productos o tipos de mercancías que han llegado. Es un valor muy variable, las mercancías más habituales son el carbón o el algodón, pero existe una extraordinaria diversidad de productos que llegan al puerto.

__cargo_value__: Expresión numérica del importe de la carga

__cargo_unit__: Expresa las unidades en las que aparece la carga. Estas pueden ser unidades de peso, volumen, recuentos o unidades relativas al embalaje.

__cargo_origin__: Puerto de origen de la carga

__cargo_destination__: Puerto de destino de la carga

__ship_quarantine__: Información relativa a condiciones especiales de la llegada motivadas por circunstancias sanitarias.

__ship_forced_arrival__: Indicación sobre las causas de la llegada forzosa

__ship_amount__: Este campo aparece únicamente en modelos cuantitativos donde, en lugar de especificar la información de cada buque, se indica el número de buques que han llegado o están a punto de llegar. Normalmente, se trata de un modelo específicamente pensado para el transporte de cabotaje.

__ship_origin_area__: Este campo aparece únicamente en modelos cuantitativos donde, en lugar de especificar la información de cada buque, se utiliza la zona de origen o de transporte. Normalmente, se trata de un modelo específicamente destinado al transporte de cabotaje.

En este caso existirán dos enfoques metodológicos para realizar la extracción, seleccionables mediante un atributo en el fichero de configuración (ver el apartado de configuración).  Uno de ellos estará basado en expresiones regulares (compuestas también siguiendo la metodología usada por la biblioteca _jportada_auto_news_extractor_lib_ que se explicará más adelante). El otro enfoque se basará en el uso de inteligencia artificial generativa (concretamente OpenAI).  

### Diseño de software basado en proxies
La biblioteca *[jportada_auto_news_extractor_lib](https://github.com/portada-git/jportada_auto_news_extractor_lib)* se ha diseñado para permitir diversos enfoques metodológicos en el tratamiento de la extracción. A fin de facilitar el desarrollo de los enfoques actuales y permitir futuras alternativas, la biblioteca dispone de un conjunto de proxies capaces de manejar, de forma transparente, múltiples enfoques que respondan a una funcionalidad determinada, organizada alrededor de una interfaz.

![diagrama UML del patron proxy](media/proxy_annotation_system.jpg)

Se ha implementado 4 patrones diferentes de tipo proxy. Uno para cada funcionalidad (ensamblaje de archivos, segregación de fragmentos-objetivo y analizador de contenido) y un cuarto sistema para gestionar los llamados calculadores de campos. Al iniciarse la ejecución, cada proxy busca entre un conjunto definido de  paquetes, las classes anotadas por una anotación de marcaje asociada al proxy. Al iniciarse la ejecución, el proxy usará su anotación asociada para  encontrar todas las classes que implementen la interficie vinculada al proxy y así gestionar la delegación de las peticiones de los clientes en función de un parámetro de selección (el enfoque)

### Configuración
La biblioteca _jportada_auto_news_extractor_lib_ dispone de diversos sistemas de configuración, los cuales se complementan entre sí. El sistema se inicializa argumentos pasados desde el sistema y también mediante un fichero típico de configuración (ini, properties, ...) el cual contiene en cada línea el nombre de un atributo junto a su valor separado por ":".  Por otro lado, la biblioteca, para aquellos enfoques basados en expresiones regulares, dispone de un conjunto de carpetas y ficheros donde se pueden definir, de forma parcial, expresiones regulares que podrán aprovecharse en la composición de expresiones más complejas. A este tipo de configuración la llamaremos _conjunto de expresiones regulares_ y generalmente se ubicará en el directorio "regex", aunque existe un atributo de configuración inicial que pude asignar otra ubicación. Existe todavía un tercer sistema de configuración específico para definir la extracción. Tiene formato JSON y generalmente se encuentra en un archivo llamado _extractor_config.json_, pero de nuevo, desde la configuración inicial, puede especificarse el nombre y ruta donde se ha ubicado. 

Seguidamente, describiremos con más detalles estos 3 sistemas  de configuración.

#### Inicialización o configuración inicial
 El sistema se inicializa mediante argumentos pasados desde el sistema y también mediante un fichero típico de configuración (ini, properties, ...) el cual contiene en cada línea el nombre de un atributo junto a su valor separado por ":".  Por defecto, la biblioteca busca un fichero llamado _init.properties_ en el directorio de ejecución o en el subdirectorio llamado _config_, pero se le puede pasar otra ubicación usando el argumento -c [RUTA_INIT_PROPERTIES] desde el sistema. Casi todos los atributos permitidos en el fichero se pueden pasar usando el sistema (consola). De esta forma se puede decidir qué argumentos se pasan desde el fichero y qué otros desde el sistema. En caso de que se pasaran argumentos repetidos por fichero y consola, estos últimos tendría siempre prioridad sobre los definidos en el fichero. Desde la consola los atributos a pasar son:

**-h**, **--help** show this help message and exit
 
**-c** *[INIT_CONFIG_FILE]*, **--init_config_file** *[INIT_CONFIG_FILE]*  Camino donde se encuentra el archivo de configuración  (default: config)
 
**-d** *[ORIGIN_DIR]*, **--origin_dir** *[ORIGIN_DIR]*  Directorio de dónde leer los archivos OCR con las  noticias
 
**-o** *[OUTPUT_FILE]*, **--output_file** *[OUTPUT_FILE]*  Camino al archivo de salida. Por ejemplo: -o c:  /directorio/nombre_archivo
 
**-a** *[APPENDOUTPUTFILE]*, **--appendOutputFile** *[APPENDOUTPUTFILE]*  Indica si se desea añadir los barcos extraídos al final del archivo de salida o se crea un nuevo  archivo en cada extracción. Solo acepta los valores  '[s]i', '[y]es', '[c]ert', '[t]rue' como verdaderos. El resto de valores se consideran falsos.
 
**-x** *[FILE_EXTENSION]*, **--file_extension** *[FILE_EXTENSION]*  Indica qué extensión deben tener los archivos en  leer
 
**-r** *[REGEXBASEPATH]*, **--regexBasePath** *[REGEXBASEPATH]*  Directorio donde se encuentran especificadas las  expresiones regulares del análisis 
 
**-f** *[FACT_MODEL]*, **--fact_model** *[FACT_MODEL]* Indica qué tipo de hecho o noticia que se deberá analizar. En el caso del proyecto PorTAda, el tipo de hecho es _boatfacts_.
 
**-n** *[NEWSPAPER]*, **--newspaper** *[NEWSPAPER]* Indica qué nombre del periódico usado para extraer la notícia (db, sm, lp, dm, en, ...).
 
**-oe** *[OCR_ENGINE_MODEL]*, **--ocr_engine_model** *[OCR_ENGINE_MODEL]* Permite indicar qué modelos de expresiones regulares es necesario aplicar, en caso de que el motor OCR se comporte de manera específica.
 
**-p** *[PARSE_MODEL]*, **--parse_model** *[PARSE_MODEL]*  Indica qué modelos de analizador (parser) es necesario usar. Esto es, un nombre identificador del tipo de patrón con el que está escrita la noticia a extraer.
 
**-pcf** *[PARSER_CONFIG_JSON_FILE]*, **--parser_config_json_file** *[PARSER_CONFIG_JSON_FILE]* Indica cuál es el archivo JSON de configuración del exctractor.

**-tfb_pck** *[TARGET_FRAGMENT_BREAKER_PROXY_PACKAGES_TO_SEARCH]*, **--target_fragment_breaker_proxy_packages_to_search** *[TARGET_FRAGMENT_BREAKER_PROXY_PACKAGES_TO_SEARCH]* Indica en qué paquetes el proxy buscará los distintos enfoques de los segregadores de fragmentos-objetivo. 

**-iub_pck** *[INFORMATION_UNIT_BUILDER_PROXY_PACKAGES_TO_SEARCH]*, **--information_unit_builder_proxy_packages_to_search** *[INFORMATION_UNIT_BUILDER_PROXY_PACKAGES_TO_SEARCH]* Indica en qué paquetes el proxy buscará los distintos enfoques de los constructores de unidades de información. 

**-dex_pck** *[DATA_EXTRACT_PROXY_PACKAGES_TO_SEARCH]*, **--data_extract_proxy_packages_to_search** *[DATA_EXTRACT_PROXY_PACKAGES_TO_SEARCH]* Indica en qué paquetes el proxy buscará los distintos enfoques de los extractores de información. 
 
**-decb_pck** *[DATA_EXTRACT_CALCULATOR_BUILDER_PACKAGES_TO_SEARCH]*, **--data_extract_calculator_builder_packages_to_search** *[DATA_EXTRACT_CALCULATOR_BUILDER_PACKAGES_TO_SEARCH]* Indica en qué paquetes el proxy buscará las distintas clases de calculadores de campos. 
 
**-fbapp** *[FRAGMENT_BREAKER_APPROACH]*, **--fragment_breaker_approach** *[FRAGMENT_BREAKER_APPROACH]*  Indica qué enfoque metodológico se usa para separar los fragmentos-objetivo. Actualmente, solo en enfoque "regex" (basado en expresiones regulares) se encuentra implementado.
 
**-exapp** *[EXTRACTOR_APPROACH]*, **--extractor_approach** *[EXTRACTOR_APPROACH]*  Indica qué enfoque metodológico se utiliza para
 hacer la extracción. Actualmente, solo en enfoque "regex" (basado en expresiones regulares) se encuentra implementado.
 
**-rd** *[RUN_FOR_DEBUGGING]*, **--run_for_debugging** *[RUN_FOR_DEBUGGING]* Indica si es necesario ejecutar el proceso en modo  depuración o en modo normal. Los valores: '[s]i', '[y]es', '[c]ert', '[t]rue', '[v]ertader' se toman como valores ciertos, cualquier otro valor se considerará falso.

En el fichero de configuración se admiten todos los argumentos aceptados por consola en su versión larga. Además, dicho fichero acepta otros parámetros relativos al sistema de registro (logs).

#### conjunto de expresiones regulares
A fin de poder generar expresiones regulares complejas, la biblioteca _jportada_auto_news_extractor_lib_ permite definir múltiples ficheros con expresiones regulares parciales que pueden ser usadas para componer nuevas expresiones regulares, las cuales podrían ser usadas de nuevo, recursivamente, como componentes de otras expresiones. 
Una expresión compuesta podría ser: 
```
^(.*{##embarcaciones##} {##llegadas##} .{2,7} {##puerto##}.{8,25})\s+$
``` 
Esta expresión buscaría el contenido del fichero _embarcaciones.regex_ y reemplazaría la posición ocupada por `{##embarcacione##}` con el contenido encontrado en el fichero. También buscaría el fichero _llegadas.regex_ y el fichero _puerto.regex_ realizando la misma operación de reemplazo. 

Imaginemos que el fichero _embarcaciones.regex_ contuviera la expresión:
```
[EA]{2,3}barca.{2,4}nes
``` 
Que el contenido de _llegadas.regex_ fuera:
```
(?:(?:[|i¡l][|i¡l])|(?:[UHN]))[eoa]g[aoeu]d..

``` 
Y el de _puerto.regex_ fuera:
```
p[uo][eo]rt[oe]
``` 

La expresión final conseguida con el ejemplo anterior sería:
```
^(.*[EA]{2,3}barca.{2,4}nes (?:(?:[|i¡l][|i¡l])|(?:[UHN]))[eoa]g[aoeu]d.. .{2,7} p[uo][eo]rt[oe].{8,25})\s+$
``` 



#### Configuración del extractor




## Aplicación _jportada_boat_fact_extractor_


## Repositorio de configuración _portada_boat_fact_extractor_config_data_
