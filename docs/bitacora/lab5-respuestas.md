\# Lab 5 - Oracle DBA Workflow II



\## 1. Retos de práctica profesional



\### Parte B - Factor de bloqueo real de PEDIDO



He agrupado las filas de PEDIDO por bloque con `DBMS\_ROWID.ROWID\_BLOCK\_NUMBER`:

\- Bloques con filas: 5220.

\- Mínimo de filas por bloque: 2.

\- Media de filas por bloque: 191,57.

\- Máximo de filas por bloque: 200.

El diccionario DBA\_TABLES registra 1000000 filas y 5320 bloques, por lo que el factor de bloqueo calculado es:

bfr = 1000000 / 5320 = 187,97 filas por bloque.

La media real es algo mayor porque solo cuento los bloques con filas. El diccionario cuenta 100 bloques más.

No todos los bloques contienen el mismo número de filas debido a la longitud variable de los registros, el espacio reservado y la existencia de bloques parcialmente llenos.



\### Parte C - Anatomía de UX\_CLI\_EMAIL



Cálculo teórico con 100000 entradas, 30 bytes por entrada, 7996 bytes útiles por hoja y un factor de ramificación máximo de 543:

\- Capacidad por hoja: 266 entradas.

\- Hojas estimadas: 376.

\- Altura teórica: 2.

Tras ejecutar ANALYZE INDEX VALIDATE STRUCTURE y consultar INDEX\_STATS, se obtiene:

\- Altura real: 3.

\- Hojas: 501.

\- Bloques de rama: 3.

\- Bytes medios por entrada de hoja: 35,89.

\- Bytes medios por entrada de rama: 18,17.

La diferencia se debe al mayor tamaño real de las entradas y al espacio libre de las hojas. El mayor tamaño de las entradas de rama reduce también el factor de ramificación máximo estimado a unas 441 entradas.

La radiografía muestra una ocupación media del 89,6 %, con 500 hojas en el intervalo del 80-89 % y una hoja por debajo del 10 %. No hay hojas vacías ni entradas borradas.

A diferencia de PK\_CLIENTE, cuyas claves numéricas crecientes producen hojas muy compactas, UX\_CLI\_EMAIL almacena claves de texto más largas y presenta una menor capacidad por hoja. Su mapa es uniforme y no muestra un patrón cola ni fragmentación significativa.



\### Parte D - Índice cubriente para Q9



No convertiría IX\_PED\_FECHA en un índice cubriente: recuperar los 20 pedidos más recientes cuesta solo 5 lecturas lógicas, mientras que la tabla completa ocupa 41,6 MB y duplicar sus columnas en el índice aumentaria el espacio y el coste de escritura.



\### Parte E - Curva del punto de equilibrio



!\[Curva del punto de equilibrio](lab5/evidencia/config/04-curva-equilibrio.png)

El optimizador abandona el índice de CF malo entre el 0,1 % y el 0,2 %, porque penaliza las lecturas aleatorias frente a las secuenciales.

El índice de CF malo supera las 5232 lecturas del recorrido completo entre el 0,5 % y el 1 %, debido a los continuos saltos entre bloques.

El índice de CF bueno solo supera ese coste entre el 50 % y el 100 %, porque las filas cercanas en el índice también están agrupadas físicamente.



\### Parte F - Revisión de código SQL



La consulta propuesta aplica SUBSTR sobre teléfono y UPPER sobre apellidos, impidiendo las búsquedas directas por los índices convencionales.

Propongo sustituir el predicado por:

&#x20;   WHERE telefono LIKE '600%'

&#x20;      OR apellidos IN ('Gil', 'GIL', 'gil')



Con índices sobre teléfono y apellidos, Oracle puede combinar ambas ramas mediante BITMAP OR.

Resultados medidos:

\- Original: 1487 lecturas lógicas y 24,6 ms.

\- Reescrita: 728 lecturas lógicas y 3 ms.

Ambas recuperan 981 filas.

La reducción de lecturas lógicas es del 51 %. No se necesita un índice funcional para la búsqueda por prefijo del teléfono.

En el banco actual no existen apellidos que satisfagan UPPER(apellidos) = 'GIL'. En producción, la reescritura de apellidos requiere garantizar su normalización o contemplar todas las variantes de mayúsculas y minúsculas.



\### Parte G - Defensa de un índice propio (V006)



Propuesta: IX\_CLI\_TELEFONO sobre CLIENTE(telefono), para optimizar Q5.

Necesidad: Q5 realiza un recorrido completo de CLIENTE sin el índice. Con el nuevo índice, Oracle combina las búsquedas por email y teléfono mediante BITMAP OR.

Viabilidad: la consulta medida recupera una fila de 100000 (0,001 %). Las lecturas lógicas pasan de 1486 a 6 y el tiempo de 28,9 a 0,2 ms.

Escritura: en 2000 inserciones, los bloques modificados por fila pasan de 10,1 a 12,1 y el redo de 1375 a 1628 bytes por fila. El índice ocupa 3 MB.

Redundancia: Ninguno de los cuatro índices existentes sobre CLIENTE tiene teléfono como columna inicial. El nuevo índice no es redundante.

Ciclo de vida: antes de incorporar definitivamente el índice, se debe observar su uso con una carga representativa. Si posteriormente deja de ser necesario, se puede hacer invisible antes de eliminarlo.

Conclusión: el índice ofrece una mejora importante en Q5 con un coste adicional de escritura cuantificado. Dejaría V006 en una rama experimental hasta comprobar cuánto se usa realmente.

Después de la prueba eliminé el índice. Siguen los 100000 clientes y los nueve índices de V005.



\### Parte H - Diagnóstico de IX\_COLA



El índice analizado en H.2 presenta un patrón cola: 286 hojas vacías concentradas a la izquierda, 90277 entradas borradas y una ocupación viva del 41,1 %.

La consulta del siguiente mensaje requiere 288 lecturas lógicas, frente a las 2 de un índice sano.

COALESCE reduce las hojas de 487 a 201 y las lecturas de 288 a 2.

SHRINK SPACE permite además reducir el segmento de 4 MB a 1,69 MB. REBUILD ONLINE no aporta una mejora adicional en las lecturas.

Recomiendo COALESCE como primera medida y SHRINK SPACE si también interesa recuperar espacio.



\### Parte I - MTTR bajo carga



Tras actualizar 20000 filas y ejecutar COMMIT, los bloques de redo pendientes pasan de 3992 a 7655, un incremento del 91,8 %.

El MTTR estimado permanece en 4 segundos. Hay casi el doble de redo pendiente, pero en esta prueba el tiempo estimado no ha cambiado.



\### Parte K - Dimensionamiento del redo



Durante la hora de carga de TiendaSur se registraron 5 cambios de log, con grupos de 200 MB.

Para reducirlos a un máximo orientativo de 4 cambios:

5 \* 200 / 4 = 250 MB.

Pondría 300 MB por grupo para dejar algo de margen. Es una estimación basada en el historial horario, que puede incluir cambios forzados y otras operaciones.

Comandos propuestos (no ejecutados):

```sql

ALTER DATABASE ADD LOGFILE GROUP 4 ('/opt/oracle/oradata/FREE/redo04.log', '/opt/oracle/oradata/FREE/redo04b.log') SIZE 300M;

ALTER DATABASE ADD LOGFILE GROUP 5 ('/opt/oracle/oradata/FREE/redo05.log', '/opt/oracle/oradata/FREE/redo05b.log') SIZE 300M;

ALTER DATABASE ADD LOGFILE GROUP 6 ('/opt/oracle/oradata/FREE/redo06.log', '/opt/oracle/oradata/FREE/redo06b.log') SIZE 300M;

ALTER SYSTEM SWITCH LOGFILE;

ALTER SYSTEM CHECKPOINT;

SELECT group#, status, archived FROM v$log;

```

Repetir los cambios de log y checkpoints hasta que los grupos antiguos estén INACTIVE y archivados.

Y entonces:

```sql

ALTER DATABASE DROP LOGFILE GROUP 1;

ALTER DATABASE DROP LOGFILE GROUP 2;

ALTER DATABASE DROP LOGFILE GROUP 3;

```

Se mantiene la multiplexación de dos miembros por grupo.

He dejado los tres grupos de 200 MB. Omití la prueba 4c para no borrar un grupo y realicé una rotación adicional para inicializar los seis miembros.



\### Parte L - Agotamiento de la FRA



La base ya estaba en ARCHIVELOG antes de comenzar esta parte.



Al intentar configurar la ruta de la FRA sin establecer primero su tamaño, obtuve los errores ORA-02097 y ORA-19802. Después configuré correctamente el límite de 4 GB y la ruta dentro del volumen persistente.



Para simular la falta de espacio, reduje temporalmente el límite a 2 MB, cuando ya había 20,3 MB ocupados.



Al intentar archivar el redo aparecieron los errores ORA-16038 y ORA-19809. El alert log también registró ORA-19815, indicando que no quedaba espacio disponible.



Restablecí el límite de 4096 MB y Oracle pudo continuar archivando. La ocupación final del experimento fue de 29,4 MB.



En producción ampliaría la FRA si hubiera espacio disponible o realizaría una copia de los archivados mediante RMAN con DELETE INPUT. No eliminaría directamente los archivos desde el sistema operativo.



\### Parte M - Coste del DML sobre la FRA



La ocupación de la FRA pasa de 30817792 a 69190144 bytes, aumentando en 36,59 MiB.

Las operaciones generan 11750 KB de redo y sus rollbacks otros 5793 KB, sumando 17,13 MiB.

La FRA crece más porque también se archiva redo anterior a la prueba y de otras operaciones.

El DELETE es la operación más costosa: consume 3416 KB de undo y genera 6622 KB de redo, más otros 4701 KB al deshacerlo.



\### Parte P - Alerta de ocupación de la FRA



Utilizaría un umbral del 85 % de ocupación para detectar el problema antes de que Oracle deje de poder archivar el redo.



&#x20;   SELECT name,

&#x20;          ROUND(100 \* space\_used / space\_limit, 2) AS porcentaje\_usado

&#x20;   FROM v$recovery\_file\_dest

&#x20;   WHERE space\_limit > 0

&#x20;     AND space\_used > space\_limit \* 0.85;



La consulta solo devuelve una fila cuando se supera el umbral.



En mi última ejecución del healthcheck, la FRA ocupaba 117 MB de los 4096 MB disponibles (2,9 %), por lo que no se habría generado ninguna alerta.



\## 2. Preguntas de comprobación



\*\*1. Comparación de las copias lab3-final y lab4-final\*\*



En ambas copias el inventario indica 0 archivos de datos fuera del volumen. En Lab 4 cambié la ubicación de los datafiles dentro del propio volumen, no de un volumen a otro.



\*\*2. Factor de bloqueo de CLIENTE y PEDIDO\*\*



CLIENTE: 100000 / 1517 = 65,92 filas por bloque.



PEDIDO: 1000000 / 5320 = 187,97 filas por bloque.



La diferencia se debe principalmente al tamaño medio de las filas. Las de CLIENTE ocupan 102 bytes, mientras que las de PEDIDO ocupan 32 bytes.



\*\*3. Doble ejecución de las consultas\*\*



La primera ejecución puede tener que cargar bloques en caché. Con la segunda puedo comparar las consultas con la caché ya caliente.



\*\*4. Altura y factor de ramificación de IX\_PED\_CLIENTE\*\*



En mi evidencia 02, el índice tiene una altura de 3, 2216 hojas y 6 bloques de rama.



El factor de ramificación real es:



fo = (2216 + 6 - 1) / 6 = 370,17.



Con mil millones de filas, la altura estimada sería de 4 niveles, por lo que localizar una fila requeriría unas 5 lecturas contando el acceso a la tabla.



\*\*5. ROWID en el volcado de PK\_CLIENTE\*\*



La primera entrada contiene la clave c1 02, correspondiente al cliente 1, y el ROWID:



00 00 03 1b 00 00



Los primeros cuatro bytes identifican el bloque 795 (31B en hexadecimal) y los dos últimos la ranura 0.



No aparece el número de archivo porque el índice utiliza un ROWID corto y la tabla pertenece a un tablespace bigfile, que tiene un único datafile.



\*\*6. Comparación de tres tipos de índice\*\*



\- Índice único: en Q13, las lecturas pasan de 1486 a 3. Además de acelerar la búsqueda por DNI, impide que se introduzcan duplicados.

\- Índice cubriente: en Q7, las lecturas pasan de 15 con el índice original a 7 con el compuesto y finalmente a 3 con el cubriente. Se evita tanto la ordenación como el acceso a la tabla.

\- Bitmap: en Q14, las lecturas pasan de 4073 a 528. Permite combinar filtros de baja cardinalidad mediante operaciones sobre mapas de bits.



\*\*7. Clustering factor\*\*



IX\_PED\_FECHA tiene un CF de 5220 porque los pedidos se encuentran físicamente agrupados siguiendo aproximadamente el orden de sus fechas.



IX\_PED\_CLIENTE tiene un CF de 999041 porque los pedidos de un mismo cliente están dispersos por la tabla.



Un REBUILD no solucionaría el problema, ya que reconstruir el índice no cambia la distribución física de las filas de la tabla.



\*\*8. Punto de equilibrio\*\*



El índice con CF malo deja de compensar en lecturas lógicas entre el 0,5 % y el 1 % de las filas. El de CF bueno lo hace entre el 50 % y el 100 %.



La fórmula teórica para el CF malo da:



1 / bfr = 1 / 188 = 0,53 %.



Si considero que una lectura aleatoria cuesta cinco veces más que una secuencial, el resultado baja al 0,11 %.



Esto explica que Oracle abandone el índice con CF malo entre el 0,1 % y el 0,2 %, incluso antes de que supere al recorrido completo en lecturas lógicas.



\*\*9. Búsqueda mediante Gonz%\*\*



El prefijo fijo permite realizar un INDEX RANGE SCAN, pero las 2056 filas recuperadas están dispersas por la tabla.



La consulta realiza 2024 lecturas lógicas, más que las aproximadamente 1500 que requiere recorrer CLIENTE completamente.



Que una consulta pueda utilizar un índice no significa necesariamente que sea la opción más eficiente.



\*\*10. Coste de escritura y selección de índices\*\*



Los bloques modificados por cada INSERT son:



\- Configuración inicial (4 índices): 10,1.

\- Propuesta del compañero (9 índices): 20,2.

\- V005 (5 índices): 12,2.



La propuesta del compañero prácticamente duplica el coste de inserción.



He rechazado IX\_PED\_CANAL por su baja selectividad y porque no se utilizó durante la prueba. IX\_PED\_FEC\_CLI tampoco se utilizó y solapa su prefijo con IX\_PED\_FECHA.



IX\_PED\_ESTADO se sustituye por el índice parcial de pendientes, e IX\_PED\_CLI\_FEC por su versión cubriente. Sí he conservado la propuesta de indexar el código postal, que reduce las lecturas de Q3 a 106.



\*\*11. Divisiones de hoja y patrón cola\*\*



IX\_AZAR presenta 595 divisiones de hoja, ninguna 90/10, y una ocupación del 75,4 %. El espacio libre se encuentra repartido por el índice.



IX\_CRECIENTE presenta 398 divisiones, todas 90/10, y una ocupación del 99,9 %. Las inserciones se concentran en el extremo derecho.



IX\_COLA presenta 286 hojas vacías concentradas a la izquierda y una ocupación viva del 41,1 %. Se debe a que se eliminan las claves antiguas mientras las nuevas se insertan por el extremo contrario.



\*\*12. Remedio para el patrón cola\*\*



Elegiría COALESCE como primera opción porque permite reorganizar las hojas en línea sin reconstruir el índice entero.



En mi prueba reduce las hojas de 487 a 201 y las lecturas necesarias para obtener el siguiente mensaje de 288 a 2.



Si también necesitara devolver espacio al tablespace, utilizaría SHRINK SPACE. No recurriría a REBUILD ONLINE sin una razón adicional, ya que no ha mejorado las lecturas respecto a COALESCE.



\*\*13. SCN y necesidad de recuperación\*\*



Oracle compara la información de checkpoint del control file con la de las cabeceras de los datafiles para determinar si los archivos son coherentes y necesitan recuperación.



En la prueba se ve que el SCN avanza con los cambios y que el checkpoint avanza al forzarlo. El SCN actual y el SCN del checkpoint no tienen por qué coincidir continuamente, ya que la base sigue realizando operaciones.



\*\*14. Multiplexación del control file\*\*



Tener tres copias del control file protege frente a la pérdida o corrupción individual de uno de los archivos.



Sin embargo, como las tres están en el mismo volumen, no protege frente a la pérdida completa de ese almacenamiento.



En producción distribuiría las copias entre dispositivos de almacenamiento independientes, de forma que el fallo de uno no implique perder todas las copias.



\*\*15. Estados de los grupos de redo\*\*



\- CURRENT: grupo en el que LGWR está escribiendo actualmente.

\- ACTIVE: grupo que ya no recibe escrituras, pero cuyo contenido sigue siendo necesario para recuperar la instancia.

\- INACTIVE: grupo cuyo contenido ya no es necesario para la recuperación de la instancia.



Oracle impide eliminar el grupo CURRENT y los grupos ACTIVE porque todavía son necesarios.



Tampoco permite eliminar un grupo pendiente de archivar cuando la base está en ARCHIVELOG, ni reducir el número de grupos por debajo del mínimo obligatorio de dos.



\*\*16. FRA llena\*\*



ORA-19815 advierte de que la FRA está cerca de alcanzar su límite.



Cuando se agota el espacio, Oracle no puede crear nuevos archivos de redo archivado, apareciendo errores como ORA-19809 y ORA-19804.



ARCn deja de poder archivar los grupos de redo. LGWR continúa escribiendo mientras dispone de grupos reutilizables, pero cuando estos se agotan tampoco puede continuar.



La base termina bloqueando las operaciones que necesitan generar redo.



La solución consiste en liberar espacio de forma controlada mediante RMAN o ampliar la capacidad de la FRA.



\*\*17. Lectores y escritores\*\*



En la prueba de lectura consistente, una sesión modifica una fila sin confirmar el cambio, mientras que otra puede seguir consultándola.



Oracle utiliza el undo para reconstruir la versión de la fila correspondiente al SCN de la consulta.



La sesión lectora no necesita esperar a que termine la transacción de escritura. Después del COMMIT, una nueva consulta puede ver el valor actualizado.



\*\*18. Undo de DELETE y redo del ROLLBACK\*\*



Un INSERT necesita guardar principalmente la información necesaria para eliminar la fila insertada si se deshace la operación.



Un DELETE necesita conservar la información de la fila eliminada para poder reconstruirla, además de los cambios correspondientes en los índices.



En mi prueba, DELETE consume 3416 KB de undo y genera 6622 KB de redo.



El ROLLBACK también genera redo porque deshacer una transacción implica modificar bloques. Oracle debe proteger esos cambios frente a una posible caída durante el propio rollback.



En el caso del DELETE, su rollback genera otros 4701 KB de redo.



\*\*19. Durabilidad después del COMMIT y del checkpoint\*\*



Justo después del COMMIT, la durabilidad está garantizada por el redo que LGWR ha escrito en disco, aunque DBWn todavía no haya escrito los bloques modificados de la tabla.



En la evidencia 13 se compara la actividad de LGWR y DBWn.



Después del checkpoint, los bloques modificados correspondientes se escriben en los datafiles y avanza la información de checkpoint.



Esto reduce la cantidad de redo que Oracle tendría que aplicar durante una recuperación de instancia.



\*\*20. Recuperación de instancia: roll forward y roll back\*\*



En el simulacro A, la modificación de la fila 102 se confirma mediante COMMIT, mientras que la modificación de la fila 103 queda sin confirmar antes de provocar la caída.



Durante el roll forward, Oracle aplica el redo necesario para reconstruir los cambios realizados antes de la caída.



Durante el roll back, utiliza el undo para deshacer los cambios de las transacciones que no llegaron a confirmarse.



El resultado es que la fila 102 conserva su cambio confirmado y la fila 103 recupera su estado anterior.



\*\*21. Comparación de los simulacros C y D\*\*



Simulacro C:



\- RTO: 88 segundos.

\- RPO: ventana de 2 segundos entre la copia y el fallo.



Simulacro D:



\- RTO: 10 segundos.

\- RPO: 0.



En el simulacro C se restaura una copia completa del estado anterior de la base, por lo que los cambios posteriores a esa copia no están garantizados.



En el simulacro D solo se restaura el datafile afectado mediante RMAN y se aplica el redo necesario para recuperarlo.



La prueba demuestra que el cambio confirmado de la fila 104, realizado después de la copia RMAN, sobrevive a la pérdida del archivo.



Aunque Oracle cerró FREEPDB1 al detectar el fallo de medio, fue posible recuperar el archivo y volver a abrir la PDB en 10 segundos.



\*\*22. Clasificación de los errores del runbook\*\*



Efectos provocados por el laboratorio:



\- ORA-01113: necesidad de recuperación de un datafile.

\- ORA-01110: identificación del datafile afectado.

\- ORA-63999: fallo de medio detectado durante el simulacro de pérdida del datafile.

\- ORA-00202: identificación del control file afectado.

\- ORA-00205: error al identificar el control file durante el simulacro de pérdida de una de sus copias.



Ruido conocido del entorno:



\- ORA-00800 (Set Priority Failed): problemas al establecer la prioridad de procesos como LGWR y VKTM en el entorno de contenedores. No han impedido el funcionamiento de la instancia durante las pruebas.



No veo ningún incidente pendiente fuera de los simulacros.



El healthcheck final muestra la instancia abierta, FREEPDB1 en READ WRITE, tres copias del control file, todos los grupos de redo multiplexados y una ocupación de la FRA del 2,9 %. Tampoco se registran errores ORA-01555 ni falta de espacio en undo.



\## 3. Preguntas tipo examen



\### Parte B



\- B1. B. Faltan estadísticas con histograma que reflejen la distribución desigual de los valores.



\### Parte C



\- C1. C. Tres lecturas para recorrer el índice y una para acceder a la tabla.

\- C2. C. Con dos millones de hojas y un fan-out de 500 se necesitan cuatro niveles.



\### Parte D



\- D1. B. Índices bitmap por la baja cardinalidad y la ausencia de escrituras concurrentes durante las consultas.

\- D2. A y C. Un índice cubriente evita acceder a la tabla, pero aumenta el coste de escritura.



\### Parte E



\- E1. B. El clustering factor es demasiado alto y el recorrido completo resulta más conveniente.



\### Parte F



\- F1. B. Oracle aplica una conversión implícita a la columna VARCHAR2.

\- F2. A y C. Ambas expresiones delimitan el día sin aplicar funciones sobre la columna de tipo DATE.



\### Parte G



\- G1. B. Un índice invisible sigue manteniéndose y puede volver a hacerse visible.

\- G2. B. Aproximadamente dos bloques modificados adicionales por inserción, es decir, unos 1000 por segundo.



\### Parte H



\- H1. B. COALESCE permite reutilizar las hojas con espacio libre sin reconstruir todo el índice.

\- H2. A y C. REVERSE distribuye las inserciones, pero impide las búsquedas por rango sobre la clave.



\### Parte I



\- I1. B. Oracle compara el SCN de checkpoint del control file con el de la cabecera del datafile.



\### Parte J



\- J1. B. Hay que sustituir la copia dañada por una sana o retirarla de CONTROL\_FILES.

\- J2. A y C. El control file registra los archivos de la base y el SCN del último checkpoint.



\### Parte K



\- K1. C. El grupo ACTIVE todavía puede ser necesario para recuperar la instancia.



\### Parte L



\- L1. B. Primero hay que configurar DB\_RECOVERY\_FILE\_DEST\_SIZE.

\- L2. A y C. Ampliar la FRA o liberar espacio mediante RMAN.



\### Parte M



\- M1. C. La sesión B obtiene el valor confirmado anterior mediante el undo.

\- M2. C. DELETE necesita guardar la información necesaria para reconstruir las filas eliminadas.

\- M3. B y D. Aumentar la capacidad y retención del undo o garantizar su conservación.



\### Parte N



\- N1. A. Con el COMMIT normal, el redo de la transacción ya está escrito en el redo log online.

\- N2. B y C. DBWn escribe durante los checkpoints y cuando se necesitan buffers libres.



\### Parte O



\- O1. B. Oracle realiza automáticamente la recuperación de instancia.

\- O2. B y D. Se puede sustituir el control file perdido por una copia sana o retirarlo de la configuración.

\- O3. B. OFFLINE, RESTORE, RECOVER y ONLINE.

\- O4. D. RMAN RECOVER TABLE permite recuperar la tabla a un instante anterior al DROP sin restaurar toda la base.



\## 4. Simulacro de examen 1Z0-182



\- S1. B — Parte E. Clustering factor alto y accesos dispersos.

\- S2. A y C — Parte F. Índices funcionales y sargabilidad.

\- S3. B — Parte F. Conversión implícita de tipos.

\- S4. B — Parte D. Índices bitmap y baja cardinalidad.

\- S5. B — Parte D. INDEX SKIP SCAN.

\- S6. B — Parte G. Índices invisibles.

\- S7. A y C — Parte H. COALESCE y SHRINK SPACE.

\- S8. B — Parte H. Índices REVERSE.

\- S9. C — Parte G. Coste de mantenimiento de los índices.

\- S10. B — Parte C. Altura y factor de ramificación.

\- S11. A y C — Parte D. Tablas organizadas como índice.

\- S12. B — Parte I. SCN de checkpoint.

\- S13. C — Parte I. CKPT actualiza el control file y las cabeceras.

\- S14. B y E — Parte O, simulacro B. Estado NOMOUNT y vistas de instancia.

\- S15. B — Parte L. El cambio se hace con la base montada, no abierta; ya estaba activado en mi entorno.

\- S16. B — Parte K. CLEAR LOGFILE permite recrear un grupo inactivo ya archivado (no ejecutado).

\- S17. A y C — Parte M. Lectura consistente y Flashback Query.

\- S18. B — Parte M. Redo generado por el undo.

\- S19. A — Parte N. Durabilidad del COMMIT.

\- S20. A — Parte I. Checkpoints y MTTR.

\- S21. B — Parte L. Agotamiento de la FRA.

\- S22. B — Parte O, simulacro D. Pérdida de un datafile.

\- S23. B — Parte O. SYSTEM requiere recuperar con la base montada (caso teórico).

\- S24. B — Partes L y O. Limitaciones de NOARCHIVELOG.

\- S25. B y D — Parte M. Más espacio y retención ayudan a conservar el undo; RETENTION GUARANTEE evita reutilizarlo antes de tiempo (caso teórico).

\- S26. A — Parte L. Ocupación de la FRA por tipo de archivo.



\## 5. Simulacros de recuperación (Parte O)



| Simulacro | RTO medido | RPO |

| A. Caída de instancia | 7 s | 0 s |

| B. Pérdida de un control file | 6 s | 0 s |

| C. Borrado de tabla con PURGE | 88 s | 2 s |

| D. Pérdida de un datafile | 10 s | 0 s |



En el simulacro C, los 2 segundos de RPO corresponden al intervalo entre la copia y el fallo. No se ha comprobado la pérdida de ninguna transacción de negocio durante ese intervalo.



\### Retrospectiva



\*\*Simulacro A\*\*



Me ha sorprendido lo rápido que se recupera Oracle tras una parada brusca. El cambio confirmado de la fila 102 se conservó y el de la 103 se deshizo.



Como mejora, automatizaría la comprobación de ambas filas y la medición del tiempo desde el inicio de la caída.



\*\*Simulacro B\*\*



Me ha sorprendido que la pérdida de una sola copia impida montar la base aunque existan otras dos copias sanas. Bastó con recuperar la copia perdida y arrancar.



En producción distribuiría las copias del control file entre distintos dispositivos de almacenamiento, ya que las tres copias actuales comparten el mismo volumen.



\*\*Simulacro C\*\*



La diferencia entre recuperar una tabla desde la papelera y restaurar toda la base es considerable. FLASHBACK TABLE recuperó las 60000 matrículas directamente, mientras que la restauración completa tardó 88 segundos.



En un entorno real utilizaría RMAN RECOVER TABLE para recuperar únicamente la tabla borrada con PURGE, siempre que disponga de las copias y del redo necesarios.



\*\*Simulacro D\*\*



Oracle cerró FREEPDB1 automáticamente al detectar el fallo de medio (ORA-63999). Tuve que recuperar el datafile con la PDB montada y abrirla después.



La fila 104 seguía en EGRESADO aunque el cambio se hizo después de la copia RMAN. El redo permitió recuperarlo.



Mantendría la validación previa de la copia RMAN y comprobaría siempre el estado de la PDB antes de comenzar la recuperación.



La recuperación de un solo datafile tardó 10 segundos frente a los 88 segundos de la restauración completa del simulacro C, sin perder los cambios confirmados.



