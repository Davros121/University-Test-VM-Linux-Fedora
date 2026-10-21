#IUPSM Hilos y Pthreads SO II#

Esta es una prueba evaluable que sigue pasos numerados para experimentar y comprender la utilidad y el potencial de una máquina virtual (VM).
Integrantes

Jesus Ernesto Ramirez Ruiz

Anthony Alvarado

El siguiente experimento/actividad fue realizado en una maquina virtual empleado el S.O. Linux Fedora por medio de Oracle VirtualBox.
Cabe destacar que algunos "Comandos" difieren dependiendo de que "distro" (Distribución) uses. Un ejemplo de comandos simples de diferentes distros son:

```bash
Ubuntu/ Debian / Linux Mint = sudo apt update 
```
```bash
Arch Linux / Manjaro        = sudo pacman -Syu
```

#Instrucciones para realizar la misión de Laboratorio#

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
mkdir suma_paralela
cd suma_paralela
```

**Paso 2:** Crear el archivo fuente de la Suma Paralela
Vamos a programar un arreglo masivo de 100 millones de elementos dividido en 4 partes (chunks), asignando cada bloque a un hilo independiente mediante una estructura de rangos, y almacenando los resultados de forma aislada en un arreglo de sumas parciales para evitar condiciones de carrera.

1. Crea y abre el archivo con tu editor favorito:

```bash
nano suma_paralela.c
```

2. Escribe el siguiente código base en C. Este programa simula el procesamiento concurrente mediante hilos POSIX:

```bash

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <sys/time.h>

#define MAX_ELEMENTS 100000000
#define NUM_THREADS 4

// Arreglo global de datos y arreglo de resultados parciales seguros
long long int arreglo[MAX_ELEMENTS];
long long int suma_parcial[NUM_THREADS];

// Estructura para definir los límites que procesará cada hilo
typedef struct {
    int id;
    long long int inicio;
    long long int fin;
} Rango;

// Función ejecutada por cada hilo de forma paralela
void *sumar_segmento(void *arg) {
    Rango *r = (Rango *)arg;
    long long int local_suma = 0;

    // Cada hilo procesa exclusivamente su rango asignado
    for (long long int i = r->inicio; i < r->fin; i++) {
        local_suma += arreglo[i];
    }

    // Guarda el resultado en su índice exclusivo (Aislamiento Seguro)
    suma_parcial[r->id] = local_suma;
    pthread_exit(NULL);
}

int main() {
    pthread_t hilos[NUM_THREADS];
    Rango rangos[NUM_THREADS];
    struct timeval t_ini, t_fin;
    long long int suma_total = 0;

    printf("Inicializando arreglo de %d millones de elementos...\n", MAX_ELEMENTS / 1000000);
    for (long long int i = 0; i < MAX_ELEMENTS; i++) {
        arreglo[i] = 1; // Llenamos con 1 para verificar que la suma sea exacta
    }

    long long int chunk_size = MAX_ELEMENTS / NUM_THREADS;

    // Inicio de medición de tiempo
    gettimeofday(&t_ini, NULL);

    // 1. Crear los 4 hilos en paralelo
    for (int i = 0; i < NUM_THREADS; i++) {
        rangos[i].id = i;
        rangos[i].inicio = i * chunk_size;
        rangos[i].fin = (i == NUM_THREADS - 1) ? MAX_ELEMENTS : (i + 1) * chunk_size;
        
        pthread_create(&hilos[i], NULL, sumar_segmento, (void *)&rangos[i]);
    }

    // 2. Sincronizar: esperar a que todos los hilos terminen
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(hilos[i], NULL);
    }

    // 3. Consolidar las sumas parciales en el hilo principal
    for (int i = 0; i < NUM_THREADS; i++) {
        suma_total += suma_parcial[i];
    }

    // Fin de medición de tiempo
    gettimeofday(&t_fin, NULL);

    double tiempo_ejecucion = (t_fin.tv_sec - t_ini.tv_sec) + (t_fin.tv_usec - t_ini.tv_usec) / 1e6;

    // Resultados
    printf("--- RESULTADOS DEL LABORATORIO ---\n");
    printf("Suma Total calculada: %lld\n", suma_total);
    printf("Tiempo de ejecución paralelo (%d hilos): %.6f segundos\n", NUM_THREADS, tiempo_ejecucion);

    return 0;
}
```

3. Guarda los cambios y sal del editor (en nano, presiona Ctrl+O, Enter para guardar, y Ctrl+X para salir).

**Paso 3:** Compilar el programa
Verifica que el código cumpla con la sintaxis de C y genera el ejecutable utilizando gcc enlazando la biblioteca de hilos.

1. Ejecuta el comando de compilación:

```bash
gcc suma_paralela.c -o suma_paralela -pthread
```

2. Asegúrate de que no arroje errores de compilación y que se haya generado el binario suma_paralela.

**Paso 4:** Ejecutar y verificar el comportamiento
Corre el programa en tu terminal de Fedora para observar el flujo de ejecución en tiempo real:

```bash
./suma_paralela
```

Procesos que podrás ver en la salida:

El programa inicializa el arreglo masivo de 100 millones de elementos de manera eficiente.

Los 4 hilos distribuyen la carga de trabajo y procesan los datos de forma simultánea en paralelo.

El hilo principal utiliza pthread_join() para sincronizar y esperar a que todos los hilos terminen antes de consolidar la suma total exacta y el tiempo de ejecución en segundos.

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una compresion y dominio del contenido en este experimento, es necesario tomar en cuenta las siguientes precauciones para evitar bugs o problemas:

**Aislamiento de Memoria:** Notarás que cada hilo lee su propio segmento del arreglo global pero escribe en casillas independientes (suma_parcial[i]), lo que previene por completo las condiciones de carrera (Race Conditions).

**Sincronización Correcta:** El uso obligatorio de pthread_join() garantiza que el hilo principal no imprima resultados parciales o basura antes de que los hilos hijos terminen sus operaciones de cómputo.
