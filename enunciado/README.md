# Enunciado - Unix Shell #

> **Nota**: Esta practica es una traducción de la práctica [Process Shell](https://github.com/remzi-arpacidusseau/ostep-projects/tree/master/processes-shell) del libro de Remzi. Esta traducción puede tener muchos errores y a veces no ser fiel con lo que el autor quiere transmitir. Si desea leerla en ingles puede encontrar esto en el siguiente [enlace](https://github.com/remzi-arpacidusseau/ostep-projects/tree/master/processes-shell).

En esta practica, usted construirá un simple Unix shell. El shell es el corazon de la interface de linea de comandos, y por lo tanto es el ambiente de programación central de Unix/C. 

Es necesario dominar el uso del shell para volverse competitivo en este mundo; saber cómo se construye el shell en sí es el foco de este proyecto.

## Objetivos ##

Hay tres objetivos especificos para esta actividad:
1. Familiarizarse más con el entorno de programación de Linux.
2. Aprender como los procesos son creados, destruidos y gestionados.
3. Ganar exposición a la funcionalidad necesaria en los shells.

## Overview ##

En esta actividad usted  implementará un command line interpreter (CLI), tambien conocido mas comunmente como shell. El shell funciona basicamente de la siguiente manera: Cuando usted escribe un comando (en respuesta a su prompt), el shell crea un proceso hijo y entonces ejecuta el comando que usted digitó; luego, cuando este comento se ejecuta se retorna al promt a la espera de otro comando.

El shell a implementar será similar pero mas simple al implementado en Unix.

## Especificaciones del programa ##

### Basic Shell: wish ###

Su shell basico será llamadao **wish** (abreviación para Wisconsin Shell, de la cual se tomó esta practica), un shell es basicamente un loop interactivo que: imprime repetidamente el promp ```wish> ``` (nota: despues del signo ```>``` hay un espacio), analiza (parse) el comando ingresado a la entrada, ejecuta dicho comando y espera a que este finalice. Este proceso es repetido hasta que el usuario digite ```exit```. Una vez compile el código, el nombre final de su ejecutable será ```wish```.

El shell puede ser invocado tanto sin  argumentos o con unico argumento; cualquier otra cosa generará un error. A continuación se muestra como sería la ejecución para el caso sin argumento:

```
prompt> ./wish
wish> 
```

En este punto, ```wish``` esta ejecutandose y esperando para aceptar comandos. 

El modo descrito anteriormente es llamado **interactive mode** y permite al usuario digitar comandos directamente. El shell tambien soporta un **batch mode**, el cual lee la entrada desde un **batch file** y ejecuta los comandos contenidos en este. A continuación se muestra como el shell **wish** ejecutaria un batch file llamado **batch.txt**:

```
prompt> ./wish batch.txt
```

Una diferencia entre el los modos batch e interactivo es que en el modo interactivi, un promt es impreso (**wish> **) mientras que en el modo batch el prompt no es impreso. 

Usted deberia estructurar su shell de manera que cree un procesos para cada nuevo comando (con los built-in commands, que se discutirán abajo, como excepción). El shell basico deberpa ser capas de analizar (parser) un comando y correr el programa correspondiente a ese comando. Por ejemplo, si el usuario digita ```ls -la /tmp```, el shell deberia ejecutar el programa ```/bin/ls``` con los argumentos dados ```-la``` y ```/tmp``` (¿Como sabe el shell ejecutar ```/bin/ls```?. Es algo llamado **shell path**; mas de esto abajo).

## Structure ##

### Basic Shell ###

El sel es muy simple (conpeptualmente): Este corre en un ciclo infinito solicitando repetidamente una entrada que dice el comando a ejecutar. Luego ejecuta ese comando. El ciclo continua indefinidamente hasta que el usuario escribe el comando integrado (built-in ) ```exit```, el cual hace que se salga del shell.

Para leer las lineas de entrada, usted podría usar ```getline()```. Esta le permite obtener lineas de entrada arbitrariamente largas con facilidad. Generalmente, el shell correra en **interactive mode** (modo interactivo), donde el usuario digita un comando (uno a la vez) y el shell actua sobre este. Sin embargo, el shell tambien soportara el **batch mode**, en el cual al shell se le da como entrada un archivo de comandos; en este caso, el shell no leerá la entrada de usuario (de **stdin**) sino que la entrada será tomada desde un archivo que contendrá los comandos a ejecutar.

En cada modo, si se encuentra un marcador de fin de archivo **EOF** (end-of-file marker), se deberá invocar la llamada **exit(0)** para salir.

Para hacer el parsing de la linea de entrada en cada una de las piezas constituyentes, usted podria usar ```strsep()```. Lea el manual (cuidadosamente) para mas detalles.

Para ejecutar comandos, examine ```fork()``` ```exec()``` y ```wait()/waitpid()```. Mire los manuales (man pages) de estas funciones, y tambien lea el capitulo del libro relevante a esto para obtener una breve visión general de este. 

Note que hay una variedad de comandos en la familia ```exec```; para este proyecto, deberá usar ```execv```. No use la función ```system()``` para correr un comando. Recuerede que si ```execv``` es exitoso, no retornara; por otro lado, si retorna, es por que hay un error (por ejemplo, **the command does not exist**). La parte más desafiante es obtener los argumentos correctamente especificados.

### Paths ###

En el ejemplo de arriba, el usuario digitó ```ls``` pero el shell sabia que tenia que ejecutar ```/bin/ls```. ¿Como el shell conoce esto?



Resulta que el usuario debe especificar una variable **path** para describir el conjunto de directorios para buscar ejecutables; el conjunto de directorios que componen la ruta se suele llamar **search path**(ruta de búsqueda) del shell. La variable **path** contiene la lista de todos los directorios, en orden, para buscar ejecutables y, es empleada cuando el usuario digite un comando.


**Importante**: Tenga en cuenta que el propio shell no implementa ```ls``` u otros comandos (excepto los integrados (build-in commands)). Todo lo que hace es encontrar esos ejecutables en uno de los directorios especificados por ruta (**path**) y crear un nuevo proceso para ejecutarlos.

Para chequear si un archivo particular existe en un directorio y es ejecutable, considere la llamada a sistema ```access```. Por ejemplo, cuando el usuario digita ```ls``` y la **path** incluye los directorios ```/bin``` y ```/usr/bin```, intente usar ```access("/bin/ls", X_OK)``` para verificar si el ejecutable existe en el directorio ```/bin```. Si la funcion falla, continue evaluandola empleando el el proximo directorio de **path** (```/usr/bin```) (mas exactamente use la función ```access("/usr/bin/ls", X_OK)```). Es esta falla tambien (pues se recorrieron todos los directorios del **path** en este caso), entonces el resultado es un error.

La **shell path** inicial debe contener solo un directorio: ```/bin```.

**Nota**: La mayoría de los shells permiten especificar un binario sin usar una ruta de búsqueda (**search path**), usando rutas absolutas o rutas relativas. Por ejemplo, un usuario podría escribir la ruta absoluta ```/bin/ls``` y ejecutar el binario ```ls``` sin necesidad de una ruta de búsqueda. Un usuario también podría especificar una ruta relativa que comienza con el directorio de trabajo actual y especifica el ejecutable directamente, por ejemplo, ```./main```. En este proyecto, no tiene que preocuparse por estas características.

### Built-in Commands ###

Siempre que el shell acepte un comando, deberá validar si el comando es un **built-in command** o no. Si lo es, no será ejecutado como otros programas. En vez de eso, el shell invocará su implementación del comando integrado (**built-in command**). Por ejemplo, para implementar el comando integrado ```exit```, simplemente basta invocar la función ```exit(0)``` implementada en el codigo fuente del shell **wish** lo cual, permitirá entonces salir del shell. 

En este proyecto, usted deberá implementar ```exit```, ```cd```, y ```path``` como comandos integrados (**build.in commnads**).

* **```exit```**: Cuando el usuario digite ```exit```, el shell debería simplemente invocar la llamada al sistema ```exit``` con ```0``` como parámetro. Es un error pasar argumentos para salir.

* **```cd```**: cd siempre toma solo un argumento (0 o mas de un argumento deben señalarse como un error). Para cambiar de directorio, use la llamada al sistema ```chdir()``` con el argumento proporcionado por el usuario; si ```chdir``` falla, eso también es un error.

* **```path```**: El comando **path** toma **0** o más argumentos, con cada argumento separado de los demás por espacios en blanco. Un uso típico sería el siguiente: ```wish> path /bin /usr/bin```, que agregaría ```/bin``` y ```/usr/bin``` a la ruta de búsqueda (**search path**) del shell. Si el usuario establece que la ruta esté vacía, entonces el shell no debería poder ejecutar ningún programa (excepto los comandos integrados). El comando de ```path``` siempre sobrescribe la ruta anterior con la ruta recién especificada.

### Redirection ###

Muchas veces, un usuario de un shell prefiere enviar la salida de un programa a un archivo en lugar hacerlo a la pantalla. Por lo general, un shell proporciona esta característica con el carácter **```>```**. Formalmente, esto se denomina redirección de salida estándar. Para este proyecto, su shell también debe incluir esta función, pero con un ligero giro (explicado a continuación).


Por ejemplo, si un usuario escribe ```ls -la /tmp > output```, no se debe imprimir nada en la pantalla. En su lugar, la salida estándar del programa ```ls``` debería redireccionarse archivo ```output```. Además, la salida de error estándar del programa debe redirigirse al archivo ```output``` (el giro es que es un poco diferente a la redirección estándar).


Si el archivo ```output``` existe antes de ejecutar su programa, simplemente debe sobrescribirlo (después de truncarlo).

El formato exacto de la redirección es un comando (y posiblemente algunos argumentos) seguido del símbolo de redirección seguido de un nombre de archivo. Varios operadores de redirección o varios archivos a la derecha del signo de redirección son errores.

**Nota**: No se preocupe por la redirección de los comandos integrados (por ejemplo, no se probara lo que sucede cuando se digita el comando ```path /bin > file```).

### Parallel Commands ###

El shell también deberá permitir al usuario ejecutar comandos paralelos. Esto se logra con el operador ampersand (**&**) de la siguiente manera:

```
wish> cmd1 & cmd2 args1 args2 & cmd3 args1
```

En este caso, en lugar de ejecutar ```cmd1``` y luego esperar a que termine, su shell debe ejecutar ```cmd1```, ```cmd2``` y ```cmd3``` (cada uno con los argumentos que el usuario le haya pasado) en paralelo, **antes** de esperar a que se complete alguno de ellos.

Luego, después de iniciar todos estos procesos, asegurese de usar ```wait()``` (o ```waitpid```) para esperar a que se completen. Una vez finalizados todos los procesos, devuelva el control al usuario como de costumbre (o, si está en **batch mode**, pase a la siguiente línea).

### Program Errors ###

The one and only error message. You should print this one and only error message whenever you encounter an error of any type:

**Solo hay un mensaje de error** el cual se debe imprimir cada vez que se encuentre un error de cualquier tipo:

```C
char error_message[30] = "An error has occurred\n";
write(STDERR_FILENO, error_message, strlen(error_message); 
```
El mensaje de error debe ser impreso a **stderr** (standard error), como se mostró arriba.

Después de la mayoría de los errores, el shell simplemente continúa funcionado después de imprimir el único mensaje de error. Sin embargo, si se invoca el shell con más de un archivo, o si se le pasa al shell un **batch file** malo, se debe salir del shell llamando a ```exit(1)```.

### Miscellaneous Hints ###

Recuerde hacer funcionar la funcionalidad básica de su shell antes de preocuparse por todas las condiciones de error y casos finales. Por ejemplo, primero ejecute un solo comando (probablemente primero un comando sin argumentos, como ```ls```).

A continuación, agregue los **built-in commands** (comandos integrados). Luego, intente trabajar en la redirección. Finalmente, piense en los comandos paralelos. Cada uno de estos requiere un poco más de esfuerzo en el análisis, pero no debería ser demasiado difícil de implementar.

En algún momento, debe asegurarse de que su código sea robusto para espacios en blanco de varios tipos, incluidos espacios (``` ```) y tabulaciones (```\t```). En general, el usuario debería poder poner cantidades variables de espacios en blanco antes y después de los comandos, argumentos y varios operadores; sin embargo, los operadores (redirección y comandos paralelos) no requieren espacios en blanco.

Verifique los códigos de retorno de todas las llamadas al sistema desde el comienzo de su trabajo. Esto a menudo detectará errores en la forma en que invoca estas nuevas llamadas al sistema. También es solo un buen sentido de programación.

¡Mejore su propio código! Usted es el mejor (y en este caso, el único) evaluador de este código. Agregue muchas entradas diferentes y asegúrese de que el shell se comporte bien. El buen código se obtiene mediante pruebas; debe ejecutar muchas pruebas diferentes para asegurarse de que todo funcione como se desea. No sea amable -- otros usuarios ciertamente no lo serán.

Finalmente, conserve las versiones de su código. Los programadores más avanzados utilizarán un sistema de control de fuentes como git. Como mínimo, cuando tenga una parte de la funcionalidad funcionando, haga una copia de su archivo .c (quizás un subdirectorio con un número de versión, como v1, v2, etc.). Al mantener las versiones más antiguas y funcionales, puede trabajar cómodamente para agregar nuevas funciones, con la seguridad de saber que siempre puede volver a una versión más antigua y funcional si es necesario.

## Anexo ##

Aunque estos fragmentos de código no estan asociados al libro de Remzi, entenderlos puede ser de utilidad para el desarrollo de la practica. Si desea profundizar un poco mas puede consultar el siguiente documento: [Shell Program](LinuxTutorial2.pdf)


### Esqueleto de un bash ###

El siguiente fragmento de codigo muestra la estructura general de un bash interactivo.

```C

const char *mypath[] = {
  "./",
  "/usr/bin/",
  "/bin/",
  NULL
};

/** Dentro del main del intérprete*/
while (...) {
  /* Wait for input */
  printf ("prompt> ");
  fgets (...);
  /* Parse input */
  while (( ... = strsep (...)) != NULL) {
    ...
  }

  /* If necessary locate executable using mypath array */
  /* Launch executable */
  if (fork () == 0) {
    ...
    execv (...);
    ...
  }
  else
  {
    wait (...);
  }
}

```

### Ejecución de un comando integrado ###

Recuerde que los comandos integrados no son mas que funciones que hacen llamadas de sistema del API Posix cuando son invocadas, por ello no es necesario usar ```fork()``` y ```exec()``` en estos casos:

```C
/** Comandos como funciones */
tipo_retorno orden1 (tipo args, ...);
...
tipo_retorno ordenN(tipo args, ...);


/** Dentro del main del intérprete*/
  ...
  /* Hacer el parsing de la entrada */
  num = separaItems (expresion, &items, &background);
  ...
  /* Obtener comando */
  if(ordenIngresada == orden1) {
     /* Lanzar el ejecutable asociado a la orden 1 */
     // Código...
  }
  ...
  else if(ordenIngresada == ordenN) {
     /* Lanzar el ejecutable asociado a la orden 1 */
     // Codigo: suponiendo que es interna…
     ordenN(parametros); // Como se llame dependerá si tiene o no &
  }
  ...
```

### Ejecución de un externo ###

Cuando el comando ingresado es externo, se hace uso de la pareja ```fork()``` y ```exec()``` para que este sea llamado desde el interprete.

```C
/** Dentro del main del interprete*/
  ...
  /* Hacer el parsing de la entrada */
  num = separaItems (expresion, &items, &background);
  ...
  /* Obtener comando */
  if(ordenIngresada == orden1) {
     /* Lanzar el ejecutable asociado a la orden 1 */
     // Codigo...
  }
  ...
  else if(ordenIngresada == ordenN) {
     /* Lanzar el ejecutable asociado a la orden 1 */
     // Codigo: suponiendo que es externa…
     if (fork () == 0) {
        ...
        execv (...); // Acá va la invocación de la orden externa
        ...
     }
     ...
  }
  ...
```
