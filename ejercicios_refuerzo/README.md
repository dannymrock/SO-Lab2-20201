# Ejercicios de programación #

A continuación se colocan unos ejercicios de analisis y programación relacionados con el API de procesos con el objetivo de evaluar la comprensión y uso de dicho API en la solución de problemas de programación en C.

## Analisis de código ##

1. Dado el siguiente programa, explique cual es la salida de la linea comentada como ```LINE A```

```C
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int value = 5;
int main() {
    pid t pid;
    pid = fork();
    if (pid == 0) { /* child process */
        value += 15;
        return 0;
    }
    else if (pid > 0) { /* parent process */
        wait(NULL);
        printf("PARENT: value = %d",value); /* LINE A */
        return 0;
    }
}
```

2. Incluyendo el proceso padre inicial. ¿Cuantos procesos  son creados por el programa mostrado a continuación?

```C
#include <stdio.h>
#include <unistd.h>
int main() {
    /* fork a child process */
    fork();
    /* fork another child process */
    fork();
    /* and fork another */
    fork();
    return 0;
}
```

3. Incluyendo el proceso padre inicial. ¿Cuantos procesos  son creados por el programa mostrado a continuación?

```C
#include <stdio.h>
#include <unistd.h>

int main() {
    int i;
    for (i = 0; i < 4; i++)
        fork();
    return 0;
}
```

4. En el siguiente código ¿La linea de codigo marcada como ```printf("LINE J")```  imprimira el mensaje?

```C
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    pid t pid;
    /* fork a child process */
    pid = fork();
    if (pid < 0) { /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    }
    else if (pid == 0) { /* child process */
        execlp("/bin/ls","ls",NULL);
        printf("LINE J");
    }
    else { /* parent process */
        /* parent will wait for the child to complete */
        wait(NULL);
        printf("Child Complete");
    }
    return 0;
}
```

5. Asumiendo que se tienen como pids para el padre y para el hijo los valores de 2600 y 2603 respectivamente. Identifique los valores del pid en las lines comentadas como ```A```, ```B```, ```C``` y ```D```

```C
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    pid t pid, pid1;
    /* fork a child process */
    pid = fork();
    if (pid < 0) { /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    }
    else if (pid == 0) { /* child process */
        pid1 = getpid();
        printf("child: pid = %d",pid); /* A */
        printf("child: pid1 = %d",pid1); /* B */
    }
    else { /* parent process */
        pid1 = getpid();
        printf("parent: pid = %d",pid); /* C */
        printf("parent: pid1 = %d",pid1); /* D */
        wait(NULL);
    }
    return 0;
}
```

6. Dado el siguiente programa, ¿Cual sería la salida desplegada en las lineas comentadas como ```LINE X``` y como ```LINE Y```?

```C
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#define SIZE 5
int nums[SIZE] = {0,1,2,3,4};

int main() {
    int i;
    pid t pid;
    pid = fork();
    if (pid == 0) {
        for (i = 0; i < SIZE; i++) {
            nums[i] *= -i;
            printf("CHILD: %d ",nums[i]); /* LINE X */
        }
    }
    else if (pid > 0) {
        wait(NULL);
        for (i = 0; i < SIZE; i++)
            printf("PARENT: %d ",nums[i]); /* LINE Y */
    }
    return 0;
}
```

## Problemas de programación ##

1. Escriba un programa que abra un archivo (con la llamada ```open()```) y entonces llame a ```fork()```. Nota: El siguiente [enlace](https://www.geeksforgeeks.org/input-output-system-calls-c-create-open-close-read-write/) puede ser de utilidad para entender la llamada open().
   * ¿Pueden el padre y el hijo acceder al file descriptor retornado por ```open()```?
   * ¿Qué pasa si ellos empiezan a escribir el archivo de manera concurrente, es decir, a la misma vez?

2. Escriba un programa usando fork(). El proceso hijo imprimirá ```"Hello"```; el proceso padre imprimirá ```"goodbye"```. Usted deberá asegurar que el proceso hijo imprima en primer lugar.

3. Escriba un programa que llame ```fork()``` y entonces llame alguna forma de exec() para correr el programa ```/bin/ls```. Intente probar todas las variaciones de la familia de funciones ```exec()``` incluyendo (en linux) ```execl()```, ```execle()```, ```execlp()```, ```execv()```, ```execvp()``` y ```execvpe()```. ¿Por qué piensa usted que existen tantas variaciones para la misma llamada básica?
   
4. Escriba ahora un programa que use ```wait()``` para esperar que el proceso hijo finalice su ejecución. ¿Cuál es el valor de retorno de la función ```wait()```?, ¿Qué pasa si usted usa la función ```wait``` en el hijo?

5. Haga un programa, como el del ejercicio anterior, con una breve modificación, la cual consiste en usar ```waitpid()``` en lugar de ```wait()```. ¿Cuándo podría ser ```waitpid()``` útil?

6. Escriba un programa que cree dos hijos y conecte la salida estándar de un hijo a la entrada estándar del otro usando la llamada a sistema ```pipe()```.

7. Escriba un programa en C llamadao **time.c** que determine la cantidad de tiempo necesaria para correr un comando desde la linea de comandos. Este programa será ejecutado como "```time <command>```" y mostrará la cantidad de tiempo gastada para ejecutar el comando especificado. Para resolver el problema haga  uso de ```fork()``` y ```exec()```, así como de la función ```gettimeofday()``` para determinar el tiempo transcurrido. 
   
   La estrategia general es hacer un fork para crear un proceso hijo el cual ejecutara el comando especificado. Sin embargo, antes de que el proceso hijo ejecute el comando espeficado, debera almacenar el tiempo actual (**starting time**). El padre invocará el wait para esperar por la culminación del proceso hijo. Luego, una vez que el proceso hijo culmine, el padre almacenara el tiempo actual en este punto (**ending time**). La diferencia entre los tiempos **inicial** y **final** (**starting** y **endind**) representará el tiempo gastado para ejecutar el comando. Por ejemplo la salida en pantalla de abajo muestra la cantidad de tiempo para correr el comando ```ls```:

```
./time ls
time.c
time

Elapsed time: 0.25422
```

