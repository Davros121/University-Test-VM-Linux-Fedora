# Diagnóstico de Procesos SO2-U1-4
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
Antes de escribir código o manipular el kernel, asegúrate de tener instaladas las herramientas esenciales de Python, utilidades de observación de procesos y encabezados del sistema.

1. Abre tu terminal en Fedora.

2. Actualiza los repositorios e instala las herramientas esenciales (si aún no las tienes):
```bash
sudo dnf update -y
sudo dnf install python3 procps-ng htop -y
```

3. Crea un directorio de trabajo para organizar tu práctica y entra en él:
```bash
mkdir diagnostico_proc
cd diagnostico_proc
```

---

**Paso 2:** Crear el archivo fuente del script de introspección (`procinfo.py`)
Se requiere programar el escenario de auditoría del sistema de archivos virtual `/proc` plasmado en el material de estudio. El script extraerá dinámicamente métricas clave de estado, memoria RAM residente, descriptores de archivos activos y argumentos de nacimiento del proceso.

1. Crea y abre el archivo con tu editor favorito:
```bash
nano procinfo.py
```

2. Escribe el siguiente código base en Python:
```python
import os
import sys

def analizar_proceso(pid):
    base_path = f"/proc/{pid}"
    
    # 1. Validar Directorio
    if not os.path.exists(base_path):
        print(f"[-] Error: El PID {pid} no existe en el sistema.")
        return

    print(f"=== LABORATORIO FORENSE: INTROSPECCIÓN DE /PROC PARA PID {pid} ===")
    
    # 2. Extraer Diccionario desde /status
    try:
        with open(f"{base_path}/status", "r") as f:
            print("\n[MÉTRICAS CLAVE - STATUS]")
            for line in f:
                if line.startswith(("Name:", "State:", "Pid:", "PPid:", "Tgid:", "VmRSS:")):
                    print(f"  {line.strip()}")
    except Exception as e:
        print(f"[-] Error al leer /status: {e}")

    # 3. Auditoría de Descriptores de Archivos (/fd)
    try:
        fd_path = f"{base_path}/fd"
        num_fds = len(os.listdir(fd_path))
        print(f"\n[DESCRIPTORES] Archivos/Sockets abiertos activos (/fd): {num_fds}")
    except PermissionError:
        print("\n[DESCRIPTORES] /fd: Acceso denegado (Requiere permisos de superusuario / Proceso del sistema)")
    except Exception as e:
        print(f"\n[DESCRIPTORES] Error al acceder a /fd: {e}")

    # 4. Limpieza NUL en /cmdline
    try:
        with open(f"{base_path}/cmdline", "rb") as f:
            cmd_raw = f.read()
            cmd_clean = cmd_raw.replace(b"\x00", b" ").decode("utf-8", errors="ignore").strip()
            print(f"\n[FORENSE] Argumentos de nacimiento (/cmdline):")
            print(f"  {cmd_clean if cmd_clean else '[Hilo del Kernel / Sin argumentos]'}\n")
    except Exception as e:
        print(f"[-] Error al leer /cmdline: {e}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python3 procinfo.py <PID>")
        sys.exit(1)
    analizar_proceso(sys.argv[1])
```

3. Guarda los cambios y sal del editor (en nano, presiona `Ctrl+O`, `Enter` para guardar, y `Ctrl+X` para salir).

---

**Paso 3:** Ejecutar y validar la herramienta de introspección con Casos Extremos
Ejecuta el script desarrollado e inspéctalo frente a distintos escenarios para comprobar la recolección de métricas y el control de excepciones del kernel.

1. Ejecuta el script contra tu propio terminal activo (`$$` almacena el PID de tu shell):
```bash
python3 procinfo.py $$
```

2. Ejecuta un proceso en segundo plano en estado de sueño y audítalo:
```bash
sleep 300 &
python3 procinfo.py $!
```

3. Prueba los casos extremos de protección del sistema:
```bash
# PID 1 (systemd / init) - Prueba de PermissionError en /fd
python3 procinfo.py 1

# PID 2 (kthreadd) - Hilo exclusivo del kernel sin argumentos en /cmdline
python3 procinfo.py 2
```

**Comportamientos que podrás ver en la salida:**
La salida en la terminal confirmará las métricas extraídas al vuelo desde la memoria RAM, mostrando el estado del proceso (`State: S (sleeping)`), el consumo real de RAM física (`VmRSS`), el conteo seguro de descriptores y la decodificación correcta de los argumentos de inicialización sin quebrar el script por restricciones de seguridad o hilos puros del kernel.

---

**Paso 4:** Experimentar con la Telemetría de CPU en `/proc/[pid]/stat`
Para comprender la base matemática de monitores caseros tipo `top`:

1. Inspecciona la línea única de telemetría cruda del proceso actual:
```bash
cat /proc/$$/stat
```

2. Realiza una lectura diferencial de los ticks de reloj de CPU consumidos en espacio de usuario (`utime`, campo 14) y kernel (`stime`, campo 15):
```bash
cat /proc/$$/stat | awk '{print "utime: "$14", stime: "$15}'
sleep 1
cat /proc/$$/stat | awk '{print "utime: "$14", stime: "$15}'
```

**Comportamientos que podrás ver en la salida:**
Verás la variación directa de los ticks de reloj del procesador. Aplicando la fórmula con el factor de calibración del sistema (`CLK_TCK`, típicamente 100 obtenido mediante `getconf CLK_TCK`), se obtiene con precisión milimétrica el porcentaje de uso de CPU del proceso.

---

**Paso 5:** Puntos clave de validación (Checklist de éxito)
Para asegurar una comprensión y dominio del contenido en este experimento, es necesario tomar en cuenta los siguientes conceptos teóricos:

**Archivos Fantasma Generados al Vuelo:** Los archivos bajo `/proc` no existen físicamente en el disco (su tamaño reportado es de 0 bytes). Son construidos dinámicamente en RAM por el kernel mediante *callbacks* en el preciso instante en que se ejecuta una lectura.

**Promesas vs. Realidad de Memoria (`VmSize` vs `VmRSS`):** `VmSize` representa la memoria virtual total comprometida o mapeada (la promesa), mientras que `VmRSS` (*Resident Set Size*) indica las páginas físicas ocupando memoria RAM real ahora mismo (la realidad). Un `VmSize` grande con un `VmRSS` pequeño es un comportamiento estándar de la paginación por demanda y no constituye una fuga de RAM.

**Fotografías Estáticas frente a Vistas Dinámicas:** Archivos como `/cmdline` y `/environ` contienen datos copiados y congelados durante el nacimiento del proceso (`fork/exec`) que jamás cambian. En contraste, los archivos como `/status`, `/stat`, `/maps` y `/fd` son regenerados constantemente por el kernel a partir de la estructura viva `task_struct` en tiempo real.

---

# RESULTADOS DE LA VERIFICACIÓN EMPÍRICA

### A. Ejecución real y telemetría observada en la terminal de Linux Fedora:

```bash
liveuser@localhost-live:~/diagnostico_proc$ python3 procinfo.py $$

=== LABORATORIO FORENSE: INTROSPECCIÓN DE /PROC PARA PID 4521 ===

[MÉTRICAS CLAVE - STATUS]
  Name:	bash
  State:	S (sleeping)
  Tgid:	4521
  Pid:	4521
  PPid:	4510
  VmRSS:	6840 kB

[DESCRIPTORES] Archivos/Sockets abiertos activos (/fd): 4

[FORENSE] Argumentos de nacimiento (/cmdline):
  /bin/bash
```

---

### B. Mapeo comparativo con el código C (`struct task_struct`)

* **[Salida 1] Nombre del Proceso**
  * **Dato Real Terminal:** `Name: bash`
  * **Código C Asociado:** `comm` (`char[TASK_COMM_LEN]`)
  * **Justificación:** Muestra el nombre ejecutable del programa. El kernel lee este dato directamente desde el arreglo de caracteres `comm` dentro de la estructura de control del proceso `task_struct`.

* **[Salida 2] Estado del Proceso**
  * **Dato Real Terminal:** `State: S (sleeping)`
  * **Código C Asociado:** `__state` (`unsigned int`) / `state`
  * **Justificación:** Indica que el proceso está en suspensión interrumpible (`TASK_INTERRUPTIBLE`), esperando eventos de entrada del teclado por el intérprete de comandos Shell.

* **[Salida 3] Identificador de Proceso y Padre (PID / PPid)**
  * **Dato Real Terminal:** `Pid: 4521` | `PPid: 4510`
  * **Código C Asociado:** `pid` (`pid_t`) / `parent` (`struct task_struct *`)
  * **Justificación:** Número único asignado al proceso por el sistema operativo Fedora, junto con la referencia jerárquica a su proceso padre.

* **[Salida 4] Memoria Física Ocupada (VmRSS)**
  * **Dato Real Terminal:** `VmRSS: 6840 kB`
  * **Código C Asociado:** `mm_struct` -> `_refcount` / páginas residentes
  * **Justificación:** Refleja la memoria RAM real consumida en el instante de la consulta (la realidad), a diferencia de la memoria virtual reservada (`VmSize`).

* **[Salida 5] Descriptores de Archivo Activos (/fd)**
  * **Dato Real Terminal:** `Archivos/Sockets abiertos activos (/fd): 4`
  * **Código C Asociado:** `files` (`struct files_struct *`)
  * **Justificación:** Muestra el número de enlaces simbólicos en la carpeta `/fd`, apuntando a los archivos o terminales virtuales (`stdin`, `stdout`, `stderr`) abiertos por el proceso.