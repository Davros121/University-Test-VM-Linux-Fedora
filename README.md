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
