# University-Test-VM-Linux-Fedora
This is a graded test following numbered steps to experiment and comprehend the usefullness and potential of a Virtual Machine (VM)

# Integrantes
Jesus Ernesto Ramirez Ruiz 
Anthony Alvarado

#Instrucciones para realizar la misión de Laboratorio

#1. Instalar Fuente
"sudo apt installl linux-source"

#2. Extraer y localizar
"cd /usr /src && sudo tar xf linux-source-*.tar.vz2"

#3. Navegación Estructural
Abrir en VSCode. Usar Intellisense (F12) o búsqueda de símbolos para encontrar la definición de 
"struct task_struct"

#4. Análisis 
Extraer 10 subcampos en una tabal detallando: nombre, tipo C. subsistema y proposito
Ejm. mm, fs, signal, real_parent, cred

#5. Veruficacion Empirica
Ejecutar "cat /proc/self/status" en su maquina local. Mapear 4 salidas con el codigo C encontrado
Realizar un Push al repositorio de Github de su equipo

Los pasos anteriores son esctructurados para un Sistema de Linux Denby, en mi caso use un sistema Linux Fedora, para facilitar su practica los pasos son los siguientes:

# Guía de Trabajo: Análisis del Código Fuente de Linux (Fedora)

Este documento detalla los pasos secuenciales para clonar el repositorio, preparar el entorno de desarrollo en Fedora, localizar las estructuras esenciales del kernel de Linux y realizar la verificación empírica.

---

## 1. Migración de GitHub a Carpeta Personal (Clonación)

Para comenzar a trabajar, primero se debe traer el repositorio digital de GitHub a su máquina local dentro de su espacio de trabajo personal.

1. Abra la terminal de Linux Fedora.
2. Descargue el repositorio clonándolo con su **Token de Acceso Personal (PAT)** para autenticar la sesión de forma segura:
   ```bash
   git clone https://Nombre_de_Usuario:TU_TOKEN_AQUÍ@github.com
   ```
3. Acceda a la carpeta del proyecto recién creada:
   ```bash
   cd University-Test-VM-Linux-Fedora
   ```

---
**Como generas un Token de Acceso personal?**
Dentro de Github ir a tu cuenta y proceder con los siguientes pasos: Usuario/ Opciones/ Opciones de Desarrollador/ Tokens de Acceso Personal/ Tokens [Clasico]/ Habilitar primera opcion REPO. Hecho esto tendras habilitado un codigo temporal que facilita el acceso para la migracion del repositorio

## 2. Instalar Código Fuente y Cabeceras del Kernel

A diferencia de otras distribuciones, Fedora organiza las cabeceras directamente en el directorio de desarrollo del sistema sin necesidad de realizar una extracción manual de un archivo comprimido.

1. Diríjase a la ruta del kernel dinámico de su máquina:
   ```bash
   cd /usr/src/kernels/$(uname -r)/
   ```
2. El archivo objetivo que debe examinar se encuentra en la siguiente ruta relativa:
   * `include/linux/sched.h`

---


## 3. Localizar Archivos de Cabecera

na vez posicionado dentro del directorio del kernel en la terminal, abra el archivo de cabecera utilizando su editor de código preferido:

* **Para abrir en VS Code:**
  ```bash
  code include/linux/sched.h
  ```
* **Para visualizar directamente en la terminal (modo lectura):**
  ```bash
  less include/linux/sched.h
  ```

Dentro del archivo, utilice la función de búsqueda (`Ctrl + F`) o la búsqueda de símbolos para encontrar el inicio de la definición de la estructura principal:

* **`struct task_struct {`**

## 4. Navegación Estructural en el Editor

1. Abra la carpeta del kernel o el archivo específico en su editor de código (por ejemplo, VS Code).
2. Utilice la función IntelliSense (**F12**) o la búsqueda de símbolos (`Ctrl + P` y escriba `@`) para encontrar la definición de la estructura principal:
   * **`struct task_struct`**

---

## 5. Análisis (Entregable)

Identifique y extraiga **10 subcampos** de la estructura `struct task_struct`. Organice la información en la siguiente tabla de su informe:

| Nombre del Campo | Tipo de Datos C    | Subsistema Asociado | Propósito / Función               |
| :---             | :---               | :---                | :---                              |
| *Ej: pid*        | *pid_t*            | *Process Core*      | *Identificador único del proceso* |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |
|                  |                    |                     |                                   |

*Sugerencia de subsistemas a buscar:* `mm` (memoria), `fs` (sistema de archivos), `signal` (señales), `real_parent` (vínculos familiares de procesos), `cred` (credenciales y seguridad).

---

## 6. Verificación Empírica y Entrega

1. Inspeccione el estado del proceso actual en su máquina local ejecutando en la terminal:
   ```bash
   cat /proc/self/status
   ```
2. Realice un mapeo comparativo asociando al menos **4 de las salidas** visualizadas en la terminal con los campos C correspondientes que analizó dentro de `task_struct`.
3. Guarde el archivo, registre sus cambios en el historial de Git y suba el trabajo final al repositorio remoto:
   ```bash
   git add .
   git commit -m "Análisis de task_struct y verificación empírica completada"
   git push origin main
   ```


