# Sincronizacion y Concurrencia SO2-U1-7
Esta es una prueba evaluable que sigue pasos numerados para experimentar y comprender la utilidad y el potencial de una máquina virtual (VM).

# Integrantes
Jesus Ernesto Ramirez Ruiz 
Anthony Alvarado

# Prueba de Laboratorio
El siguiente experimento/actividad fue realizado en una maquina virtual empleado el S.O. Linux Fedora por medio de Oracle VirtualBox.
Cabe destacar que algunos "Comandos" difieren dependiendo de que "distro" (Distribución) uses. Un ejemplo de comandos simples de diferentes distros son:
```bash
Ubuntu/ Debian / Linux Mint = sudo apt update 
```
```bash
Arch Linux / Manjaro        = sudo pacman -Syu
```

**Instrucciones para realizar la misión de Laboratorio**

**Paso 1:** Preparar el entorno de trabajo en Fedora
Antes de escribir código, asegúrate de tener instaladas las herramientas esenciales de compilación (gcc), la biblioteca de hilos (pthreads) y el editor de texto o entorno que prefieras (nano, vim o code).

1. Abre tu terminal en Fedora.

2. Actualiza los repositorios e instala las herramientas de desarrollo básicas (si aún no las tienes):

```bash
sudo dnf update -y
sudo dnf install gcc make glibc-devel -y
```

3. Crea un directorio de trabajo para organizar tu práctica y entra en él:
```bash
mkdir seccion-critica
cd seccion-critica
```

**Paso 2:** Crear el archivo fuente de la Condición de Carrera (race_counter.c)
SE requiere programar el escenario del problema de la sección crítica plasmado en el archivo (SO2-U1-7 Pag. 4). Cuatro hilos intentarán incrementar una variable global compartida un millón de veces cada uno, demostrando por qué una operación aparentemente simple como contador++ no es atómica a nivel de hardware y genera un resultado no determinista.

1. Crea y abre el archivo con tu editor favorito:

```bash
nano race_counter.c
```

2. Escribe el siguiente código base en C:

```bash
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 4
#define ITER 1000000

// Variable global compartida (Sección Crítica)
static long contador = 0;

// Función ejecutada por cada hilo
void *incrementar(void *arg) {
    for (long i = 0; i < ITER; i++) {
        contador++; /* Operación no atómica susceptible a condición de carrera */
    }
    pthread_exit(NULL);
}

int main() {
    pthread_t hilos[NUM_THREADS];
    long esperado = (long)NUM_THREADS * ITER;

    printf("Iniciando prueba con %d hilos, %d iteraciones cada uno...\n", NUM_THREADS, ITER);
    printf("Valor esperado exacto: %ld\n", esperado);

    // 1. Crear los hilos concurrentes
    for (int i = 0; i < NUM_THREADS; i++) {
        if (pthread_create(&hilos[i], NULL, incrementar, NULL) != 0) {
            perror("Error al crear el hilo");
            exit(EXIT_FAILURE);
        }
    }

    // 2. Esperar a que todos los hilos terminen
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(hilos[i], NULL);
    }

    // 3. Mostrar resultados
    printf("--- RESULTADOS DEL LABORATORIO ---\n");
    printf("Valor Esperado: %ld\n", esperado);
    printf("Valor Obtenido: %ld\n", contador);
    printf("Diferencia (Perdidos): %ld\n", esperado - contador);

    return 0;
}
```

3. Guarda los cambios y sal del editor (en nano, presiona Ctrl+O, Enter para guardar, y Ctrl+X para salir).

**Paso 3:** Compilar el programa y usar ThreadSanitizer (TSan)
Para evidenciar formalmente la colisión en tiempo de ejecución, utilizaremos la herramienta de diagnóstico de GCC, ThreadSanitizer (-fsanitize=thread), junto con la biblioteca de hilos (-pthread).

    Ejecuta el comando de compilación con TSan:

```bash
gcc -Wall -pthread -fsanitize=thread race_counter.c -o race_counter
```

1. Asegúrate de que no arroje errores de compilación y que se haya generado el binario race_counter.

**Paso 4:** Ejecutar y verificar el comportamiento
Corre el programa compilado en tu terminal de Fedora para observar el error numérico y el reporte del analizador:
```bash
./race_counter
```

***Procesos y comportamientos que podrás ver en la salida:***

El programa imprime el valor esperado (4,000,000) frente al valor obtenido real, el cual será menor debido a que múltiples hilos leen, incrementan y escriben simultáneamente la misma posición de memoria sin exclusión mutua.

**ThreadSanitizer (TSan)** emitirá una alerta roja detallando una Data Race (Condición de Carrera) en la variable contador, indicando exactamente dónde ocurre la colisión de lecturas y escrituras concurrentes.

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una compresión y dominio del contenido en este experimento, es necesario tomar en cuenta los siguientes conceptos teóricos:

**La Falacia de la Atomicidad:** Una instrucción en C como contador++ se traduce en lenguaje de máquina en múltiples pasos desacoplados (LOAD del registro, INC y STORE de vuelta), permitiendo que el planificador del sistema operativo mezcle las instrucciones de los hilos de manera impredecible.

**La Necesidad de Sincronización:** Demostrar este fallo justifica la implementación de mecanismos de Exclusión Mutua (como Semáforos y Mutex basados en instrucciones atómicas del hardware) para proteger la Sección Crítica.
