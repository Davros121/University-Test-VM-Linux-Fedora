# University-Test-VM-Linux-Fedora
This is a graded test following numbered steps to experiment and comprehend the usefullness and potential of a Virtual Machine (VM)

# Integrantes
Jesus Ernesto Ramirez Ruiz 
Anthony Alvarado

**Instrucciones para realizar la misión de Laboratorio**
**Paso 1:** Preparar el entorno de trabajo y verificación inicial en Fedora
Antes de comenzar con la exploración de arquitecturas del núcleo y llamadas al sistema, asegúrate de tener configurado tu entorno de terminal y las herramientas de compilación esenciales (gcc).

1. Abre tu terminal en Fedora.

2. Actualiza los repositorios e instala las herramientas básicas de desarrollo y utilidades del sistema:

```bash
sudo dnf update -y
sudo dnf install gcc make glibc-devel -y
```
Crea un directorio de trabajo para organizar tu reactivación de sistemas y entra en él:

```bash
mkdir reactivacion-so
cd reactivacion-so
```

**Paso 2:** Crear el programa de diagnóstico del sistema (info_sistema.c)
Vamos a programar un script en C que interactúe con el sistema operativo utilizando llamadas al sistema (syscalls) básicas (como uname e información del kernel) para entender cómo el espacio de usuario obtiene datos privilegiados de forma controlada.

1. Crea y abre el archivo con tu editor favorito:
```bash
nano info_sistema.
```

2. Escribe el siguiente código base en C:

```bash
#include <stdio.h>
#include <sys/utsname.h>
#include <unistd.h>

int main() {
    struct utsname datos_sistema;

    printf("=== REACTIVACIÓN DE SISTEMAS: KERNEL Y ENTORNO ===\n");

    // Obtener información del kernel usando la syscall uname()
    if (uname(&datos_sistema) < 0) {
        perror("Error al obtener información del sistema");
        return 1;
    }

    // Mostrar detalles de la arquitectura y el sistema operativo
    printf("Sistema Operativo : %s\n", datos_sistema.sysname);
    printf("Nombre de Nodo    : %s\n", datos_sistema.nodename);
    printf("Versión del Kernel: %s\n", datos_sistema.release);
    printf("Versión de SO     : %s\n", datos_sistema.version);
    printf("Arquitectura (CPU): %s\n", datos_sistema.machine);

    printf("PID del Proceso   : %d\n", getpid());
    printf("=== INICIALIZACIÓN COMPLETADA CON ÉXITO ===\n");

    return 0;
  
}
```
3. Guarda los cambios y sal del editor (en nano, presiona Ctrl+O, Enter para guardar, y Ctrl+X para salir).

**Paso 3:** Compilar el programa
Verifica que el código cumpla con la sintaxis de C y genera el ejecutable utilizando gcc.

1. Ejecuta el comando de compilación:

```bash
gcc info_sistema.c -o info_sistema
```

2. Asegúrate de que no arroje errores de compilación y que se haya generado el binario info_sistema.

**Paso 4:** Ejecutar y verificar el comportamiento
Corre el programa compilado en tu terminal de Fedora para observar la interacción con el núcleo:

```bash
./info_sistema
```

***Procesos y comportamientos que podrás ver en la salida:***

El programa invoca la estructura utsname a nivel del núcleo para extraer dinámicamente los datos de la distribución, arquitectura del procesador (x86-64) y versión exacta del kernel de Linux en ejecución.

Muestra el ID del proceso actual (PID), ilustrando cómo se identifica cada tarea administrada por el planificador del sistema operativo.

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una compresión y dominio del contenido en este experimento, es necesario tomar en cuenta los siguientes conceptos teóricos:

**El Modo Dual y Aislamiento:** Las aplicaciones operan en Espacio de Usuario (Ring 3) con privilegios restringidos, mientras que las llamadas al sistema (syscalls) como uname permiten cruzar legalmente hacia el Espacio de Kernel (Ring 0) a través de la instrucción syscall.

**La Máquina Extendida:** El sistema operativo actúa como una barrera inteligente que abstrae el hardware físico subyacente, proveyendo una interfaz limpia de procesos y archivos para el desarrollo de software.
