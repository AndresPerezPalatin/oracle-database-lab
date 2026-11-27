# Pérdida de un datafile

## Diagnóstico

- ORA-01116: no se puede abrir el datafile.
- ORA-01110: identifica el archivo afectado.
- ORA-27041: el sistema operativo no puede abrir el archivo.
- ORA-63999: fallo de medio que puede provocar el cierre de la PDB.

## Comprobaciones previas

Desde SQL*Plus como SYSDBA:

    SELECT log_mode FROM v$database;
    SELECT file#, name FROM v$datafile;
    SELECT name, open_mode FROM v$pdbs;

Comprobar que la base está en ARCHIVELOG y que existe una copia válida. Desde RMAN:

    LIST BACKUP OF DATAFILE 16;
    RESTORE DATAFILE 16 VALIDATE;

## Recuperación

El número 16 corresponde al datafile de TBS_ACADEMIA en nuestro entorno.

Si FREEPDB1 sigue abierta, ejecutar en SQL*Plus:

    ALTER SESSION SET CONTAINER = FREEPDB1;
    ALTER DATABASE DATAFILE 16 OFFLINE;

Si Oracle ha cerrado la PDB, esperar a que quede en MOUNTED.

Desde RMAN conectado a la raíz:

    RESTORE DATAFILE 16;
    RECOVER DATAFILE 16;

Desde SQL*Plus:

    ALTER SESSION SET CONTAINER = FREEPDB1;
    ALTER DATABASE DATAFILE 16 ONLINE;

Si la PDB estaba cerrada, volver a abrirla desde la raíz:

    ALTER SESSION SET CONTAINER = CDB$ROOT;
    ALTER PLUGGABLE DATABASE FREEPDB1 OPEN;

## Verificación

Comprobar que FREEPDB1 está en READ WRITE y que los datos son accesibles.

    ALTER SESSION SET CONTAINER = CDB$ROOT;
    SELECT name, open_mode FROM v$pdbs;
    ALTER SESSION SET CONTAINER = FREEPDB1;
    SELECT estudiante_id, estado
    FROM admin_academia.aca_estudiantes
    WHERE estudiante_id = 104;

En el simulacro, la fila debe conservar el estado EGRESADO antes de devolverla a su valor original.

Avisar al responsable de bases de datos y al equipo afectado, registrar el tiempo de recuperación y comunicar el restablecimiento del servicio.
