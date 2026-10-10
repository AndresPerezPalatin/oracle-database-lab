1. ¿Por qué un Issue sin criterios de aceptación es un problema, aunque la descripción general parezca clara?

Porque no queda definido de forma objetiva cuándo puede considerarse terminado. Los criterios de aceptación permiten comprobar si realmente se ha cumplido lo que pedía el Issue.

2. Explica la diferencia entre "Refs #N" y "Closes #N" en un mensaje de commit o en la descripción de un Pull Request.

`Refs #N` relaciona el cambio con el Issue #N, pero no lo cierra. `Closes #N` indica que ese cambio resuelve el Issue y GitHub lo cierra automáticamente cuando se integre en main.

3. ¿Qué ocurre exactamente si intentas hacer git push directamente sobre una branch main protegida? ¿Es un error tuyo o un fallo del sistema?

GitHub rechaza el push porque la regla de protección obliga a hacer los cambios mediante un Pull Request. No es un fallo del sistema, la protección está funcionando como debe.

4. Un compañero te dice: "he aprobado el PR sin mirar los archivos, total ya me fío". ¿Qué riesgo tiene esa forma de revisar?

Que pueden integrarse errores, problemas de seguridad o cambios que no cumplen lo pedido. La revisión pierde su utilidad si se aprueba sin comprobar realmente los cambios.

5. Si el reviewer pide un cambio y tú ya habías hecho push de tu branch, ¿tienes que abrir un Pull Request nuevo? Explica qué ocurre técnicamente con el PR existente cuando haces un nuevo commit.

No. Se hace el cambio en la misma branch y se crea y sube un nuevo commit. El Pull Request ya abierto se actualiza automáticamente con ese commit.

6. Describe con tus palabras la diferencia entre Merge commit, Squash and merge y Rebase and merge. ¿Cuál usarías para una branch con commits "wip", "fix", "fix2", "ok ya"?

Merge commit conserva los commits de la branch y añade un commit de merge. Squash and merge combina todos los commits en uno. Rebase and merge conserva los commits pero los coloca de forma lineal sobre main. Para una branch con commits como "wip", "fix", "fix2" y "ok ya" usaría Squash and merge, porque esos commits por separado aportan poco al historial.

7. ¿Por qué borrar una branch después del merge no elimina el trabajo realizado en ella?

Porque los cambios ya han sido integrados en main. Al borrar la branch solo eliminamos esa referencia, no los cambios que ya forman parte del historial.

8. ¿Qué información debería contener siempre la descripción de un Pull Request, como mínimo?

Un resumen del cambio, qué se ha modificado, cómo se ha comprobado y el Issue relacionado.

9. Un reviewer escribe solo "esto está mal" como comentario. ¿Qué le falta a ese comentario para ser útil? Reescríbelo tú con un ejemplo inventado.

Le falta explicar qué problema ha encontrado, por qué importa y qué debería hacerse para solucionarlo. Ejemplo: `issue (blocking): Esta contraseña está escrita directamente en el archivo. Debería obtenerse de una variable de entorno para evitar exponer credenciales.`

10. ¿Qué diferencia hay entre que main esté protegida y que simplemente el equipo "se ponga de acuerdo" en no hacer push directo?

Con un acuerdo dependemos de que todo el mundo lo respete. Con una protección, GitHub aplica técnicamente la regla y rechaza un push directo aunque alguien intente hacerlo.

11. Explica con un ejemplo propio la diferencia entre un comentario issue: (blocking) y uno nitpick: (if-minor) en formato Conventional Comments.

`issue: (blocking)` señala un problema que debe corregirse antes del merge. Por ejemplo: `issue (blocking): Esta consulta elimina registros sin comprobar el ID.`

`nitpick: (if-minor)` señala una mejora pequeña que no impide aprobar el cambio. Por ejemplo: `nitpick (if-minor): Cambiaría este nombre de variable para que sea más descriptivo.`

12. Si tu próximo commit es feat!: cambia la firma de la función principal de la API, ¿qué tipo de versión SemVer se dispara y por qué?

Una versión major, porque `!` indica un breaking change: el cambio rompe la compatibilidad con quienes utilizaban la firma anterior de la API.

13. ¿Por qué abrir un Draft PR desde el primer commit estructural puede ahorrar tiempo al equipo, aunque parezca más lento para el autor individual?

Porque permite recibir feedback antes de haber invertido mucho tiempo. Si el enfoque inicial es incorrecto, puede corregirse pronto en vez de descubrirlo cuando el trabajo ya está terminado.