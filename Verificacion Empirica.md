# Datos reales de la prueba de laboratorio
En la practica de laboratorio de "Race Counter" se observaron los siguientes resultados:
Iniciando prueba con 4 hilos, 1000000 iteraciones cada uno...
Valor esperado exacto: 4000000
```bash
==================
WARNING: ThreadSanitizer: data race (pid=34553)
  Read of size 8 at 0x000000403070 by thread T2:
    #0 incrementar <null> (race_counter+0x400542) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Previous write of size 8 at 0x000000403070 by thread T1:
    #0 incrementar <null> (race_counter+0x400557) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Location is global 'contador' of size 8 at 0x000000403070 (race_counter+0x403070)

  Thread T2 (tid=34556, running) created by main thread at:
    #0 pthread_create <null> (libtsan.so.2+0x1fd86) (BuildId: 0cdc18bb66a294b4b8e1e35fb289c44e387e985e)
    #1 main <null> (race_counter+0x4005f4) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Thread T1 (tid=34555, running) created by main thread at:
    #0 pthread_create <null> (libtsan.so.2+0x1fd86) (BuildId: 0cdc18bb66a294b4b8e1e35fb289c44e387e985e)
    #1 main <null> (race_counter+0x4005f4) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

SUMMARY: ThreadSanitizer: data race (/home/jesusramirez/seccion_critica/race_counter+0x400542) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07) in incrementar
==================
==================
WARNING: ThreadSanitizer: data race (pid=34553)
  Write of size 8 at 0x000000403070 by thread T2:
    #0 incrementar <null> (race_counter+0x400557) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Previous write of size 8 at 0x000000403070 by thread T1:
    #0 incrementar <null> (race_counter+0x400557) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Location is global 'contador' of size 8 at 0x000000403070 (race_counter+0x403070)

  Thread T2 (tid=34556, running) created by main thread at:
    #0 pthread_create <null> (libtsan.so.2+0x1fd86) (BuildId: 0cdc18bb66a294b4b8e1e35fb289c44e387e985e)
    #1 main <null> (race_counter+0x4005f4) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

  Thread T1 (tid=34555, running) created by main thread at:
    #0 pthread_create <null> (libtsan.so.2+0x1fd86) (BuildId: 0cdc18bb66a294b4b8e1e35fb289c44e387e985e)
    #1 main <null> (race_counter+0x4005f4) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07)

SUMMARY: ThreadSanitizer: data race (/home/jesusramirez/seccion_critica/race_counter+0x400557) (BuildId: 990ef6f1665be90498c963b50ce1b43ab9dd9f07) in incrementar
==================
--- RESULTADOS DEL LABORATORIO ---
Valor Esperado: 4000000
Valor Obtenido: 3490925
Diferencia (Perdidos): 509075
ThreadSanitizer: reported 2 warnings
```
# Analisis
Los resultados obtenidos en la ejecución de la terminal confirman de manera empírica y matemática los problemas de concurrencia analizados en la teoría de la Unidad 2:

**Discrepancia Numérica:** 

 **Valor Esperado:** 4,000,000

 **Valor Obtenido:** 3,490,925

 **Diferencia (Pérdida de datos):** 509,075 incrementos perdidos debido a que múltiples hilos sobrescribieron simultáneamente los cambios de sus compañeros sin exclusión mutua.

    Evidencia del Analizador (ThreadSanitizer - TSan):

        El reporte muestra con precisión quirúrgica una Data Race (Condición de Carrera) sobre la variable global contador de 8 bytes ubicada en 0x000000403070.

        Se identifican colisiones directas tanto en operaciones de Lectura (Read) como de Escritura (Write) entre los hilos T1 y T2 dentro de la función incrementar.
