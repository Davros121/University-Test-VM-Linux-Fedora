# Diagnostico de Procesos SO2-U1-3
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
Antes de escribir código o manipular el kernel, asegúrate de tener instaladas las herramientas esenciales de compilación (gcc), utilidades de observación de procesos y encabezados del sistema.

1. Abre tu terminal en Fedora.

2. Actualiza los repositorios e instala las herramientas esenciales (si aún no las tienes):

```bash
sudo dnf update -y
sudo dnf install gcc make procps-ng htop -y
```

3. Crea un directorio de trabajo para organizar tu práctica y entra en él:
```bash
mkdir diagnostico_procesos
cd diagnostico_procesos
```

**Paso 2:** Crear el archivo fuente del Proceso Zombie (zombie_maker.c)
Se requiere programar el escenario del Laboratorio Forense 01 plasmado en el material de estudio. Un proceso hijo terminará inmediatamente, pero el proceso padre no invocará la llamada a sistema `wait()`, obligando al kernel a mantener la entrada en la tabla de procesos en estado Zombie (Z).

1. Crea y abre el archivo con tu editor favorito:

```bash
nano zombie_maker.c
```

2. Escribe el siguiente código base en C:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    printf("=== LABORATORIO FORENSE 01: CREANDO UN ZOMBIE ===\n");
    
    pid_t pid = fork();

    if (pid < 0) {
        perror("Error al ejecutar fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        // Código del proceso Hijo
        printf("[HIJO] PID: %d -> Terminando inmediatamente con exit(0)...\n", getpid());
        _exit(0); // El hijo muere pero retiene su entrada hasta que el padre haga wait()
    } else {
        // Código del proceso Padre
        printf("[PADRE] PID: %d -> Hijo creado con PID: %d\n", getpid(), pid);
        printf("[PADRE] Durmiendo por 100 segundos sin invocar wait()...\n");
        printf("[INSTRUCCION] Abre otra terminal y verifica el estado con:\n");
        printf("              cat /proc/%d/status | grep State\n\n", pid);
        
        sleep(100); // Mantiene al padre vivo sin recolectar los restos del hijo
    }

    return 0;
}
```

3. Guarda los cambios y sal del editor (en nano, presiona Ctrl+O, Enter para guardar, y Ctrl+X para salir).

**Paso 3:** Compilar y ejecutar la prueba del Proceso Zombie
Compila el programa en C e inspéctalo desde una segunda terminal para comprobar la existencia del proceso en estado Zombie.

1. Compila el programa con GCC:
```bash
gcc -Wall zombie_maker.c -o zombie_maker
```

2. Ejecuta el binario generado:
```bash
./zombie_maker
```

3. Abre una **segunda terminal** en Fedora y consulta el estado del proceso hijo utilizando el PID mostrado en pantalla:
```bash
cat /proc/<PID_DEL_HIJO>/status | grep State
```
*(También puedes verificarlo usando `ps -o pid,stat,comm -p <PID_DEL_HIJO>`)*

**Comportamientos que podrás ver en la salida:**
La salida en la terminal confirmará el estado: `State: Z (zombie)`. Esto demuestra que el hijo ya ha finalizado su ejecución, pero retiene su PID e historial de salida en la tabla de procesos del kernel a la espera de que el padre ejecute `wait()`.

---

**Paso 4:** Experimentar con el Estado de Sueño Ininterrumpible (Estado D)
Para complementar la práctica (Laboratorio Forense 02), invocaremos operaciones sincrónicas de lectura directa al disco obligando al kernel a colocar al proceso en estado `D` (*TASK_UNINTERRUPTIBLE*).

1. En la terminal, vacía la caché de páginas de memoria del kernel para forzar lectura física en el dispositivo de almacenamiento:
```bash
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
```

2. Ejecuta una lectura masiva sincrónica directamente desde la unidad de almacenamiento:
```bash
dd if=/dev/sda of=/dev/null bs=1M count=2000 iflag=direct &
```
*(Nota: Si usas una unidad NVMe en tu máquina virtual o Fedora, cambia `/dev/sda` por `/dev/nvme0n1` o un archivo pesado de prueba)*.

3. Monitorea las transiciones de estado a alta frecuencia con `ps`:
```bash
watch -n 0.2 'ps -o pid,stat,wchan:32,comm -C dd'
```

**Comportamientos que podrás ver en la salida:**
Verás al proceso `dd` alternar entre los estados **R** (*Running*) y **D** (*Uninterruptible Sleep*). Durante el estado **D**, el campo `WCHAN` mostrará funciones internas de lectura de bloque del kernel como `blkdev_read_iter` o `submit_bio`, demostrando que el proceso está congelado a nivel de hardware y no puede ser interrumpido ni siquiera por señales como `kill -9`.

---

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una comprensión y dominio del contenido en este experimento, es necesario tomar en cuenta los siguientes conceptos teóricos:

**La Anatomía del Estado Zombie (Z):** Un proceso zombie no consume memoria RAM ni ciclos de CPU, pero consume una entrada en la tabla de procesos y agota el límite de PIDs del sistema. La única forma de despejarlo es que el padre ejecute `wait()` o que el padre muera para que el proceso `init` (`systemd`) lo herede y limpie.

**Inmunidad del Estado D (Uninterruptible Sleep):** Las tareas en estado D están procesando llamadas al sistema bloqueantes ligadas directamente al hardware (como discos o sistemas de archivos en red NFS). El kernel bloquea las señales para evitar la corrupción de búferes o datos en memoria.

**Unificación en Linux (Estado R):** A diferencia de la teoría clásica de Sistemas Operativos que separa "Listo" y "En Ejecución", el kernel de Linux unifica ambos bajo el estado **R** (*TASK_RUNNING*), administrados mediante el planificador CFS (*Completely Fair Scheduler*) y sus colas *runqueues* asociadas a cada núcleo de CPU.