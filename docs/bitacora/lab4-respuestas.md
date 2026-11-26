# Lab 4 - Preguntas de comprobación



## Respaldo



1. Es consistente porque se hizo con la base parada. Si estuviera abierta, los archivos podrían copiarse en distintos estados.

   Evidencia: 20261125T201109Z_00-backup-lab3.script.log.



2. El manifiesto contiene información de la copia y SHA256SUMS sirve para comprobar su integridad. Antes de restaurar se comprueban los hashes.



3. El RPO es la pérdida de datos que podemos admitir y el RTO el tiempo que podemos tardar en recuperar el servicio. La copia es del 25/11/2026 a las 20:13:35 UTC, por lo que se perderían los cambios posteriores a esa fecha. Todavía no he medido el tiempo de restauración.

   Evidencia: manifiesto.txt del backup lab3-final.



## Diagnóstico y correcciones



4. Los creó fuera del volumen porque no se especificó la ruta adecuada. Se podría haber evitado configurando db_create_file_dest.

   Evidencia: 20261125T202759Z_01-diagnostico.script.log.



5. Porque RENAME DATAFILE cambia la referencia al archivo, pero no lo mueve físicamente.



6. Porque V000 ya estaba aplicada y no conviene modificar el historial de migraciones. Borrar una contraseña del archivo no hace que deje de funcionar.



7. Demuestra que la base sobrevive al borrado del contenedor. Se hace después de mover los archivos para que estos no se pierdan.

   Evidencia: 20261125T213027Z_03-prueba-persistencia.script.log. Se mantienen los cinco tablespaces y las tablas.



## Instancia y arranque



8. De la instancia: estado OPEN, versión 23.26.3.0.0 y fecha de arranque (25/11/2026 a las 21:30:34). De la base: DBID 1509902213, nombre FREE y modo ARCHIVELOG.

   Evidencia: 20261125T230044Z_05-instancia-y-base.spool.log.



9. NOMOUNT lee los parámetros y arranca la instancia, MOUNT abre los control files y OPEN abre los datafiles y redo logs. ORA-01507 aparece porque la base no está montada y ORA-01219 porque todavía no está abierta.

   Evidencia: 20261125T231221Z_06-arranque-por-etapas.script.log.



10. Porque tiene guardado su estado con SAVE STATE. Si no se abriera, la abriría manualmente y guardaría el estado otra vez.

    Evidencia: 20261125T231221Z_06-arranque-por-etapas.script.log, FREEPDB1 en READ WRITE.



11. Ambos son errores conocidos en este entorno. ORA-00800 tiene que ver con los permisos de prioridad de los procesos y ORA-20013 con la lectura del inventario de parches.

    Evidencia: 20261125T231938Z_06-alert-log.script.log. Aparece ORA-00800, pero no he encontrado ORA-20013 en los logs consultados.



## Parámetros



12. v$parameter muestra los valores efectivos y v$spparameter los guardados en el SPFILE. Durante la prueba db_files tenía 200 en memoria y 300 en el SPFILE.

    Evidencia: 20261125T234834Z_07-parametros-cambios.script.log.



13. BOTH cambia el parámetro tanto en memoria como en el SPFILE. Es el alcance por defecto si se ha arrancado con SPFILE.



14. Porque PROCESSES necesita un reinicio para cambiar, aunque aparezca como IMMEDIATE. En la prueba devuelve ORA-02095.

    Evidencia: 20261125T234834Z_07-parametros-cambios.script.log.



15. Sirve para guardar un historial de la configuración y detectar cambios. Configuration drift es cuando la configuración real se va alejando de la prevista.

    Evidencia: initFREE_lab4.ora e initFREE_lab4_memoria.ora.



## Memoria y procesos



16. ASMM, ya que memory_target esta a 0, sga_target a 1536M y pga_aggregate_target a 512M.

    Evidencia: 20261126T000014Z_08-memoria.spool.log.



17. En frío hubo 413 lecturas lógicas y 253 físicas, aproximadamente un 39% de aciertos. En caliente hubo 247 lógicas y ninguna física, un 100% de aciertos. La segunda vez los bloques ya estaban en caché.

    Evidencia: 20261126T000454Z_08-experimento-cache.script.log.



18. Porque los procesos comparten parte de la memoria, como la SGA, y estaríamos contando esa memoria varias veces.



19. LGWR tiene SPID 7361 y aparece con ese mismo PID en /proc. Docker top muestra los PID del host en lugar de los internos del contenedor.

    Evidencias: 20261126T002548Z_09-procesos.spool.log y 20261126T002811Z_09-procesos-so.script.log.



20. PMON participa en la limpieza de procesos que fallan, por ejemplo cuando una sesión termina inesperadamente. SMON se ocupa de la recuperación de la instancia, por ejemplo después de una caída.



## Seguridad y trabajo profesional



21. Porque para observar la base no necesitamos todos los permisos de SYSTEM. C##OPS_RO puede consultar información, pero no modificar las tablas de negocio.



22. Las contraseñas que quedaron en el historial de Git y las que podían aparecer en los argumentos de los procesos.



# Preguntas tipo examen



P1. A y D. La copia es consistente y se puede abrir directamente tras restaurarla.



P2. B. Hay que poner el tablespace offline, copiar el archivo, cambiar su referencia y volver a ponerlo online.



P3. A y C. La fecha de arranque pertenece a la instancia y la de creación a la base.



P4. A y C. En NOMOUNT podemos consultar los parámetros y crear el control file.



P5. B. La base tiene que estar montada pero no abierta.



P6. B. Con SPFILE el alcance por defecto es BOTH.



P7. C. PROCESSES no se puede modificar desde una PDB.



P8. A y C. Podemos arrancar con un PFILE correcto y recrear el SPFILE.



P9. B. ASMM reparte la memoria de la SGA respetando ese mínimo del buffer cache.



P10. A y D. El área de ordenación y la memoria de sesión pertenecen a la PGA en servidor dedicado.



P11. C. LGWR escribe el redo necesario para confirmar la transacción.



P12. B y D. PMON y SMON son procesos obligatorios.

