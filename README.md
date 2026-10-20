# IUPSM For-Exec-Wait SO II
This is a graded test following numbered steps to experiment and comprehend the usefullness and potential of a Virtual Machine (VM)

# Integrantes
Jesus Ernesto Ramirez Ruiz 
Anthony Alvarado

El siguiente experimento/actividad fue realizado en una maquina virtual empleado el S.O. Linux Fedora por medio de Oracle VirtualBox.
Cabe destacar que algunos "Comandos" difieren dependiendo de que "distro" (Distribución) uses. Un ejemplo de comandos simples de diferentes distros son:
```bash
Ubuntu/ Debian / Linux Mint = sudo apt update 
```
```bash
Arch Linux / Manjaro        = sudo pacman -Syu
````
# Instrucciones para realizar la misión de Laboratorio

**Paso 1:** Preparar el entorno de desarrollo en Fedora
Antes de escribir código, asegúrate de tener instaladas las herramientas esenciales de compilación (gcc) y el editor de texto o entorno que prefieras (nano, vim o code).

 1. Abre tu terminal en Fedora.

 2. Actualiza los repositorios e instala las herramientas de desarrollo básicas (si aún no las tienes):

```bash
sudo dnf update -y
sudo dnf groupinstall "Development Tools" -y
Crea un directorio de trabajo para organizar tu práctica y entra en él:
```
```bash
mkdir mini-shell-practica
cd mini-shell-practica
```
**Paso 2:** Crear el archivo fuente de la Mini Shell
Vamos a programar el patrón canónico analizado en el material (crear un proceso, ejecutar un comando externo y esperar a que termine).

 1. Crea y abre el archivo con tu editor favorito:

```bash
nano mini_shell.c
```

 2. Escribe el siguiente código base en C. Este programa simula el ciclo de vida básico de una Shell ejecutando un comando de prueba (como ls -l):

```bash
C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid;
    int status;

    printf("=== MINI SHELL INICIADA (PID: %d) ===\n", getpid());

    // 1. Crear el proceso hijo usando fork()
    pid = fork();

    if (pid < 0) {
        // Error crítico: no se pudo crear el proceso (Bug 1 del documento)
        perror("Error al hacer fork");
        exit(EXIT_FAILURE);
    } 
    else if (pid == 0) {
        // --- PROCESO HIJO ---
        printf("[Hijo] Hola, soy el proceso hijo con PID: %d\n", getpid());

        // Argumentos para el comando a ejecutar
        char *args[] = {"ls", "-l", NULL};

        // 2. Transformar el proceso hijo con execvp()
        execvp(args[0], args);

        // Si execvp retorna, ocurrió un error (ej. comando no encontrado)
        perror("[Hijo] Error en execvp");
        _exit(EXIT_FAILURE); // Usar _exit para evitar vaciar búferes heredados
    } 
    else {
        // --- PROCESO PADRE ---
        printf("[Padre] Esperando a que termine el proceso hijo (PID: %d)...\n", pid);

        // 3. Recolectar al hijo con waitpid() para evitar zombies
        pid_t waited_pid = waitpid(pid, &status, 0);

        if (waited_pid < 0) {
            perror("Error en waitpid");
        } else {
            if (WIFEXITED(status)) {
                printf("[Padre] El hijo terminó correctamente con código: %d\n", WEXITSTATUS(status));
            } else {
                printf("[Padre] El hijo terminó de forma anormal.\n");
            }
        }
    }

    printf("=== MINI SHELL FINALIZADA ===\n");
    return 0;
}
```

 3. Guarda los cambios y sal del editor (en nano, presiona Ctrl+O, Enter para guardar, y Ctrl+X para salir).

**Paso 3:** Compilar el programa
Verifica que el código cumpla con la sintaxis de C y genera el ejecutable utilizando gcc.

 1. Ejecuta el comando de compilación:

```bash
gcc mini_shell.c -o mini_shell
```

 2. Asegúrate de que no arroje errores de compilación y que se haya generado el binario mini_shell.

**Paso 4:** Ejecutar y verificar el comportamiento
Corre el programa en tu terminal de Fedora para observar el flujo de ejecución en tiempo real:

```bash
./mini_shel
```

Procesos que podrás en la salida:

El Padre imprime su mensaje inicial y muestra su PID.

El Padre crea al Hijo mediante fork().

El Hijo toma el control, imprime su PID y ejecuta el comando **ls -l** (listando los archivos del directorio actual).

El Padre, gracias a **waitpid()**, se bloquea ordenadamente hasta que el hijo finaliza, luego lee el código de salida y se cierra de forma limpia sin dejar procesos huérfanos ni zombies.

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una compresion y dominio del contenido en este experimento, es necesario tomar en cuenta las siguientes precauciones para evitar bugs o problemas:

**Validación de fork():** Nunca omitas verificar si el valor retornado por fork() es menor a 0.

**Uso de _exit() en el hijo:** Si execvp falla, usar _exit() en lugar de exit() evita duplicar la limpieza de búferes de memoria heredados del padre.

**Limpieza obligatoria:** El uso de waitpid() garantiza que el kernel libere la entrada de la tabla de procesos del hijo, evitando el temido estado Zombie (Z).
