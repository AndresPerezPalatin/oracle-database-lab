## 1. ¿Cuál es la diferencia entre Working Directory, Staging Area y Local Repository? Da un ejemplo de un archivo pasando por las tres.

El Working Directory contiene los archivos con los que se está trabajando actualmente. La Staging Area contiene los cambios seleccionados con `git add` para incluirlos en el siguiente commit. El Local Repository contiene los cambios que ya han sido guardados en el historial mediante un commit. Por ejemplo, al crear `docs/customer-schema.md`, primero estaba únicamente en el Working Directory, con `git add docs/customer-schema.md` pasó a la Staging Area y, al ejecutar `git commit`, quedó almacenado en el Local Repository.

## 2. Si modificas un archivo pero no haces `git add`, ¿aparece ese cambio en tu próximo commit? Explica por qué.

No, el cambio permanece en el Working Directory, pero no pasa a la Staging Area. `git commit` confirma los cambios que previamente se han preparado para ese commit, de modo que hay que ejecutar `git add` sobre el archivo modificado si se quiere incluir la modificación.

## 3. ¿Por qué `git status` no mostraba las carpetas vacías que creaste en la Parte C? ¿Qué truco usamos para solucionarlo?

Porque Git versiona archivos, no carpetas vacías. Por eso las carpetas que habíamos creado no aparecían por sí solas en `git status`. Para solucionarlo añadimos dentro de ellas archivos vacíos llamados `.gitkeep`, permitiendo así que Git pudiera registrar y conservar esa estructura de directorios.

## 4. Explica con tus palabras qué es HEAD.

HEAD es el puntero que indica en qué commit o branch se está actualmente, y, al cambiar de branch, HEAD pasa a apuntar a la nueva branch.

## 5. ¿Qué diferencia hay entre crear una branch con `git switch -c` y crear una carpeta nueva con `mkdir`? ¿Cómo lo comprobamos en la Parte G?

`git switch -c` crea una nueva branch, es decir, una nueva línea del historial de Git, mientras que `mkdir` crea físicamente una carpeta en el sistema de archivos. En la Parte G lo comprobamos creando la branch `feature/customer-search`. Al cambiar entre esta branch y `main`, no apareció ninguna carpeta nueva que representase a la branch, y en cambio, el contenido del Working Directory cambiaba: `docs/customer-search.md` aparecía al estar en `feature/customer-search` y desaparecía al volver a `main`, porque ese archivo pertenecía a un commit que solo existía en aquella branch.

## 6. Durante el conflicto de la Parte H, ¿qué representaba el contenido entre `<<<<<<< HEAD` y `=======`? ¿Y entre `=======` y `>>>>>>>`?

El contenido situado entre `<<<<<<< HEAD` y `=======` representaba la versión que ya existía en la branch en la que estábamos situados al realizar el merge. El contenido entre `=======` y `>>>>>>>` representaba la versión procedente de la branch que estábamos intentando fusionar.

## 7. ¿Por qué NO se debe hacer `git commit --amend` sobre un commit que ya se subió con `git push`?

Porque `git commit --amend` reescribe el último commit y crea uno nuevo con un hash diferente. Si el commit original ya se ha subido al repositorio remoto, otras personas pueden haberlo descargado y estar trabajando sobre ese historial. Modificarlo después puede provocar divergencias entre los historiales local y remoto. `--amend` debe utilizarse sobre commits que todavía sean únicamente locales, y si el commit ya se ha subido, es preferible corregirlo mediante un nuevo commit.

## 8. Si borras por accidente la carpeta `.git` de tu proyecto, ¿qué se pierde exactamente? ¿Se pierde también el código fuente que está en el disco?

Al borrar la carpeta `.git` se pierde la información interna que convierte esa carpeta en un repositorio Git: el historial local de commits, las branches, referencias y la configuración local del repositorio, pero no se eliminan los archivos del Working Directory; el código fuente y los demás archivos que se encuentran físicamente en la carpeta del proyecto siguen estando en el disco.

## 9. Explica con tus propias palabras la diferencia entre Git y GitHub, sin usar la palabra "nube".

Git es un sistema de control de versiones que funciona en nuestro propio ordenador y permite registrar cambios, crear commits, branches, realizar merges y consultar el historial incluso sin conexión a Internet, pero GitHub, en cambio, es una plataforma web que permite alojar repositorios Git en un servidor y compartirlos con otras personas, además de proporcionar herramientas útiles de colaboración.

## 10. ¿Por qué no se debe subir un archivo `.env` con contraseñas reales a un repositorio, aunque el repositorio sea privado?

Porque un repositorio privado no garantiza que las credenciales nunca puedan quedar expuestas: otras personas con acceso pueden verlas, el repositorio puede compartirse posteriormente y las contraseñas permanecen además en el historial de Git aunque el archivo se elimine en un commit posterior. Los archivos con credenciales reales no deben versionarse; deben excluirse, por ejemplo, mediante `.gitignore`.

## 11. Un compañero te dice: "hice push y ahora GitHub me rechaza el segundo push con 'non-fast-forward'". ¿Qué ha ocurrido probablemente y qué comando ejecutarías primero?

Probablemente el repositorio remoto contiene commits que el branch local todavía no tiene, por lo que Git rechaza el push para evitar sobrescribir esos cambios. Ejecutaría primero `git pull` para traer e integrar los cambios remotos y resolvería cualquier conflicto antes de volver a hacer `git push`.

## 12. ¿Qué tipo de Conventional Commit (`feat`, `fix`, `docs`, `test`…) usarías para: añadir un índice de rendimiento a una tabla, corregir una restricción mal definida, y actualizar el README?

Para añadir un índice de rendimiento utilizaría `feat:` si se considera una nueva mejora funcional del esquema; para corregir una restricción mal definida utilizaría `fix:`; y para actualizar el README utilizaría `docs:`.