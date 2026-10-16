# Lab 3 - Respuestas de comprobacion


### 1. Que diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.

Una imagen es como una plantilla a partir de la que se crea un contenedor. El contenedor es la instancia que realmente se ejecuta. En G2 trabajamos con la imagen y en G4 creamos y ejecutamos un contenedor a partir de ella.

### 2. En el Ejercicio G5 el archivo nota.txt desaparecio y en el G6 no. Explica por que.

En G5 el archivo estaba guardado dentro del propio contenedor, asi que al eliminarlo tambien se perdio el archivo. En G6 usamos un volumen, por lo que el archivo quedo guardado fuera del contenedor y siguio existiendo aunque este se eliminara.

### 3. Que diferencia hay entre docker ps y docker ps -a, y que significa STATUS = Exited (0)?

docker ps muestra los contenedores que estan ejecutandose, mientras que docker ps -a muestra tambien los que estan parados. Exited (0) significa que el contenedor termino su ejecucion correctamente y sin errores.

### 4. En -p 8181:8181, que numero corresponde a tu equipo y cual al contenedor? Que pasaria con -p 80:8080 en el ejercicio de nginx?

El primer numero corresponde al puerto de nuestro equipo y el segundo al puerto del contenedor. Con -p 80:8080 entrariamos desde nuestro equipo por el puerto 80 y el trafico se enviaria al puerto 8080 del contenedor.

### 5. Por que un contenedor de Oracle se queda en marcha y el de hello-world termina solo?

Porque Oracle ejecuta un servicio que tiene que seguir funcionando para aceptar conexiones. hello-world solamente ejecuta una tarea, muestra su mensaje y termina.

### 6. Que es el digest de una imagen y por que lo registramos si ya sabemos que usamos :latest?

El digest es una huella que identifica de forma exacta una imagen concreta. Lo registramos porque latest puede cambiar y apuntar a otra imagen en el futuro, mientras que el digest permite saber exactamente cual usamos.

### 7. Que comando borraria realmente los datos de Oracle? Por que docker rm oralab-26ai no lo hace?

Los datos se borrarian al eliminar el volumen, por ejemplo con docker volume rm oralab-26ai-data. docker rm oralab-26ai solo elimina el contenedor, mientras que los datos siguen guardados en el volumen.


### 8. Por que este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?

Porque asi todo el trabajo queda versionado y relacionado con el resto del proyecto. El Issue indica lo que hay que hacer, la branch permite trabajar sin tocar main directamente y el Pull Request permite revisar los cambios antes de incorporarlos.

### 9. Que diferencia hay entre source 00-config.sh y bash 00-config.sh? Por que usamos source?

bash 00-config.sh ejecuta el script en otro proceso y sus variables no se quedan en nuestra sesion actual. source lo ejecuta en la propia shell, por lo que las variables quedan disponibles despues. Por eso usamos source para cargar las constantes que necesitamos en los siguientes comandos.

### 10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.

20260915 es la fecha, T091230 es la hora y la Z indica que esta en UTC. 02 es el numero de la evidencia, docker indica de que trata y script.log indica que es un log generado a partir de la salida de terminal.

### 11. Para que sirve .gitattributes y que error evita?

Sirve para indicar a Git como debe tratar determinados archivos. En esta practica lo usamos para mantener finales de linea LF y evitar problemas entre Windows y Linux al ejecutar scripts.

### 12. Por que en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?

Porque cada commit representa una Parte concreta de la practica y queremos conservar ese historial. Con Squash and merge todos esos commits se convertirian en uno solo.


### 13. Describe las cuatro capas de la estrategia de contrasenas de la Parte D y que pasaria si te saltas la primera.

La contrasena real se guarda en config/.env, ese archivo esta excluido mediante .gitignore, los scripts leen las contrasenas desde variables y antes del push comprobamos que los secretos no hayan acabado en Git ni en su historial. Si escribimos directamente una contrasena real en un archivo que se versiona, podria terminar en un commit y las demas medidas llegarian demasiado tarde.

### 14. Por que no escribimos la contrasena directamente en el comando docker run, aunque el script no se suba a Git?

Porque podria quedar expuesta en el historial de la terminal, en los procesos o en otros registros del sistema. Es mas seguro obtenerla desde una variable que escribirla directamente en el comando.

### 15. Si descubres tu contrasena en un commit ya publicado, basta con borrarla en un commit nuevo? Que debes hacer?

No, porque la contrasena seguiria existiendo en el historial anterior. Hay que cambiar esa contrasena y eliminar el secreto del historial de Git. Si hace falta, tambien habria que volver a publicar el historial corregido.


### 16. Por que no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y que hicimos en su lugar?

Porque SQL*Plus se ejecutaba dentro del contenedor y nuestros archivos y carpetas de evidencia estaban fuera, en WSL. En su lugar enviamos el SQL mediante la entrada estandar con docker exec -i y guardamos la salida desde fuera usando tee.

### 17. Que hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y que pasaria sin esa linea?

Hace que SQL*Plus termine con error si falla una sentencia SQL. Sin esa linea podria continuar ejecutando el script despues de un fallo y dar la impresion de que la migracion termino correctamente.

### 18. Que es una migracion y por que V000 y V001 no se deben editar una vez aplicadas?

Una migracion es un script numerado que lleva la base de datos de un estado a otro. Una vez aplicada no deberia modificarse porque dejariamos de tener una referencia fiable de lo que se ejecuto originalmente. Si necesitamos otro cambio, creamos una migracion nueva.

### 19. Por que en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?

Porque nuestros usuarios y esquemas estan dentro de la PDB FREEPDB1. FREE corresponde al contenedor principal y para nuestro trabajo necesitamos conectarnos al servicio de la PDB, no mediante un SID.

### 20. Que aporta SQLcl frente a SQL*Plus, y por que un DBA debe dominar ambas?

SQLcl es una herramienta mas moderna y comoda para trabajar por consola. SQL*Plus es la herramienta clasica de Oracle y sigue utilizandose mucho en administracion y scripts. Un DBA debe conocer ambas porque puede encontrarse cualquiera de ellas dependiendo del entorno.


### 21. Por que el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.

Porque WSL 2 nos da un entorno Linux real y mas parecido al que podemos encontrar en un servidor. Con Git Bash podemos tener problemas con las rutas de Windows y Linux y diferencias en comandos o herramientas que no funcionan exactamente igual. En Ubuntu tambien resulta mas sencillo trabajar con herramientas como Java, SQLcl, Docker o ShellCheck.

### 22. Por que clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? Y por que recomendamos bash frente a zsh para los scripts del curso?

Porque dentro del sistema de archivos de Linux evitamos problemas de rendimiento, permisos y diferencias entre Windows y Linux. Usamos bash porque los scripts del curso estan escritos y probados para bash, mientras que zsh podria introducir diferencias innecesarias.
