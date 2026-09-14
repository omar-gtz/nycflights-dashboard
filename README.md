# Publicar un flexdashboard de R con GitHub Actions y GitHub Pages

Esta guía explica cómo crear un proyecto local en **R/RStudio**, convertirlo en un repositorio **Git**, conectarlo con **GitHub**, configurar la autenticación y publicar automáticamente un `flexdashboard` estático mediante **GitHub Actions** y **GitHub Pages**.

El flujo completo es:

```text
dashboard.Rmd
      │
      │ git add / commit / push
      ▼
GitHub
      │
      │ GitHub Actions
      ▼
Runner con Ubuntu + R + Pandoc
      │
      │ rmarkdown::render()
      ▼
_site/index.html
      │
      │ deploy
      ▼
GitHub Pages
```

Recordemos que **Git guarda el historial del código, GitHub aloja el repositorio, GitHub Actions ejecuta R y GitHub Pages publica el HTML generado.**

---

# 1. Requisitos previos

Suponemos que al iniciar hemos creado un archivo dashboard.Rmd en local que funciona con flexdashboard. 

También necesitamos Git instalado y una cuenta de GitHub con un token activo. 

---

# 2. Crear el proyecto local

En RStudio:

```text
File → New Project → From Existing Directory → Seleccionar la carpeta donde está el dashboard.Rmd -> escribir un nombre para el proyecto, por ejemplo nycflights-dashboard
```

La carpeta tendrá inicialmente una estructura similar a:

```text
nycflights-dashboard/
│
├── dashboard.Rmd
└── nycflights-dashboard.Rproj
```

El archivo `.Rproj` contiene configuración de RStudio.

---

# 3. Comprobar que el dashboard funciona localmente

Para comprobar que el dashboard funciona localmente, podemos ejecutar:

```r
rmarkdown::render("dashboard.Rmd", output_file = "index.html", output_dir= "_site/")
```

Esto generará una carpeta `_site/` y dentro un archivo `index.html`. La estructura de la carpeta del proyecto ahora será:

```text
nycflights-dashboard/
│
├── dashboard.Rmd
├── nycflights-dashboard.Rproj
│
└── _site/
    └── index.html
```

Abrir:

```text
_site/index.html
```

y comprobar que el dashboard funciona correctamente.

OBSERVACIÓN 1: Necesitamos un `index.html` porque GitHub Pages publica páginas web estáticas. Cuando visitamos una dirección como:

```text
https://usuario.github.io/nycflights-dashboard/
```

el servidor busca normalmente un archivo llamado:

```text
index.html
```

OBSERVACIÓN 2: La carpeta `_site` puede entenderse como una **carpeta de salida o build**.

Vamos a separar el código fuente del sitio web generado. El código fuente será:

```text
dashboard.Rmd
```

y el resultado publicado será:

```text
_site/
└── index.html
```

El nombre `_site` es una convención habitual en proyectos de R Markdown, aunque técnicamente podríamos utilizar otro nombre como:

```text
build/
dist/
public/
```

---

# 4. Configurar nuestra identidad de Git

Antes de empezar a crear commits debemos indicar a Git quién está realizando los cambios.

Abrir la pestaña:

```text
Terminal
```

de RStudio y ejecutar:

```bash
git config --global user.name "Tu Nombre"
```

después:

```bash
git config --global user.email "correo@example.com"
```

usando el mismo correo asociado a la cuenta de GitHub. Podemos comprobar la configuración con:

```bash
git config --global user.name
git config --global user.email
```

Estas configuraciones no son nuestras credenciales ni nuestra contraseña de GitHub. sirven para indicar quién está creando los commits.

---

# 5. Crear un Personal Access Token en GitHub

Si trabajamos con GitHub mediante HTTPS, necesitamos autenticarnos. GitHub no utiliza la contraseña normal de la cuenta para hacer operaciones como `git push`. Podemos usar un **Personal Access Token** (**PAT**).

En GitHub ir a:

```text
Foto de perfil → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token → Generate new token (classic)
```

Asignar un nombre descriptivo, elegir una fecha de expiración. Para trabajar con repositorios normalmente necesitaremos seleccionar el scope:

```text
repo
```

Finalmente:

```text
Generate token
```

GitHub mostrará una cadena similar a:

```text
ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Copiar el token inmediatamente.

OBSERVACIÓN: El token funciona como una contraseña. Nunca debe aparecer en ningún archivo y nunca debe subirse al repositorio.

---

# 6. Guardar el token mediante `gitcreds`

En la consola de R, instalar `gitcreds` si todavía no está instalado:

```r
install.packages("gitcreds")
```

Después ejecutar:

```r
gitcreds::gitcreds_set()
```

R preguntará:

```text
Enter new password or token:
```

Pegar el token generado en GitHub y presionar Enter. 

Podemos comprobar que existe una credencial con la función:

```r
gitcreds::gitcreds_get()
```
R debería mostrar algo similar a lo siguiente:

```text
<gitcreds>
  protocol: https
  host    : github.com
  username: PersonalAccessToken
  password: <-- hidden -->
```
---

# 7. Inicializar Git en nuestra carpeta local

Desde la Terminal de RStudio y estando dentro de la carpeta del proyecto:

```bash
git init
```

Esto transforma nuestra carpeta normal en un repositorio Git.

Git crea internamente una carpeta oculta llamada:

```text
.git/
```

La estructura de la carpeta es:

```text
nycflights-dashboard/
│
├── .git/
├── dashboard.Rmd
└── nycflights-dashboard.Rproj
└── _site/
    └── index.html
```

Nunca debemos modificar manualmente el contenido de `.git`.

---

# 8. Crear `.gitignore`

Vamos a indicar qué archivos no queremos almacenar en Git, para eso creamos un archivo llamado:

```text
.gitignore
```

Estando en la carpeta del proyecto en R ejecutamos en la consola:

```r
file.create(".gitignore")
```

para crear el archivo. Posteriormente ejecutamos:

```r
file.edit(".gitignore")
```

para escribir en el archivo:

```gitignore
.Rproj.user/
.Rhistory
.RData
.Ruserdata
_site/
```

Finalmente, guarda el archivo y cierra el editor.

OBSERVACIÓN: No queremos almacenar en GitHub la carpeta `_site` ya que GitHub Actions será quien genere `_site` cada vez que publiquemos.

---

# 9. Crear el primer commit

Desde la Terminal de RStudio y estando dentro de la carpeta del proyecto:

Primero añadimos los archivos:

```bash
git add .
```

El punto significa:

> añadir los cambios de esta carpeta y sus subcarpetas, excepto los archivos ignorados.

Después comprobar:

```bash
git status
```

Te confirma en letras verdes qué archivos están listos para guardarse.

Finalmente, crear el commit:

```bash
git commit -m "Initial flexdashboard project"
```

OBSERVACIÓN: Todavía no hemos enviado nada a GitHub.

---

# 10. Crear el repositorio en GitHub

En GitHub seleccionar:

```text
+ → New repository
```

Asignar un nombre, por ejemplo:

```text
nycflights-dashboard
```

Mantener el repositorio público.

Como ya tenemos el proyecto creado localmente, es preferible NO seleccionar:

```text
Add a README file
Add .gitignore
Choose a license
```

porque ya tenemos esos archivos localmente.

Seleccionar:

```text
Create repository
```

---

# 11. Conectar el repositorio local con GitHub

Copiar desde GitHub la URL HTTPS:

```text
https://github.com/usuario/nycflights-dashboard.git
```

Después ejecutar en Terminal:

```bash
git remote add origin https://github.com/usuario/nycflights-dashboard.git
```

`origin` es el nombre convencional que Git utiliza para referirse al repositorio remoto principal.

Comprobar:

```bash
git remote -v
```

---

# 12. Utilizar `main` como rama principal

En la Terminal ejecutar:

```bash
git branch -M main
```

Para cambiar el nombre de la rama master. Después enviar el código a GitHub:

```bash
git push -u origin main
```

La primera vez Git utilizará las credenciales HTTPS que hemos guardado mediante `gitcreds`.

**Comprobar en GitHub que están nuestros archivos.**

A partir de ahora normalmente bastará con:

```bash
git push
```

---

# 13. GitHub Actions (Explicación)

Ahora queremos que GitHub ejecute automáticamente nuestro dashboard. GitHub Actions utiliza archivos YAML almacenados en:

```text
.github/workflows/
```

Nuestra estructura será:

```text
nycflights-dashboard/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── .gitignore
├── dashboard.Rmd
├── README.md
└── nycflights-dashboard.Rproj
```

Lo que haremos será usar un runner (una maquina virtual) que se levanta para poder ejecutar el .Rmd, generar el HTML y publicarlo.

---

# 14. Activar GitHub Pages

En GitHub ve a tu repositorio y selecciona:

```text
Settings → Pages
```

En:

```text
Build and deployment
```

buscar:

```text
Source
```

y seleccionar:

```text
GitHub Actions
```

GitHub debería mostrar una cinta azul con la leyenda "GitHub Pages source saved."

---

# 15. Crear el workflow desde GitHub

En el repositorio ir a:

```text
Actions
```

Seleccionar:

```text
set up a workflow yourself
```

GitHub creará un archivo dentro de:

```text
.github/workflows/
```

Podemos llamarlo:

```text
deploy.yml
```

---

# 16. Workflow para publicar el flexdashboard

Utilizar el archivo yaml de este repositorio en 

```text
.github/workflows/
```

con dos jobs:

```text
BUILD
que crear el sitio

DEPLOY
que publicar el sitio
```

Nuestro proceso será:

```text
1. Descargar repositorio
2. Preparar R
3. Preparar Pandoc
4. Instalar paquetes
5. Ejecutar dashboard.Rmd
6. Preparar GitHub Pages
7. Subir _site
```

OBSERVACIÓN: R Markdown utiliza Pandoc para convertir documentos a HTML. De manera simplificada:

```text
dashboard.Rmd
      │
      │ R + knitr
      ▼
Markdown
      │
      │ Pandoc
      ▼
HTML
```

Por eso necesitamos tener pandoc en el runner.

---

# 17. Comprobar GitHub Actions

Al terminar de escribir el yaml, dar click `Commit changes` y manten las especifícaciones por defecto.

Después en tu repositorio ir a:

```text
Actions
```

Deberíamos ver una ejecución similar a:

```text
Deploy flexdashboard to GitHub Pages
```

Dentro aparecerán los pasos:

```text
build

✓ Checkout repository
✓ Setup R
✓ Setup Pandoc
✓ Install R packages
✓ Render dashboard
✓ Setup GitHub Pages
✓ Upload website

deploy

✓ Deploy to GitHub Pages
```

Cuando todo aparezca con:

```text
✓
```

el sitio debería estar publicado.

---

# 18. Dirección del sitio

Normalmente será algo como:

```text
https://usuario.github.io/nycflights-dashboard/
```

y aparecerá cuando el job deploy ejecute correctamente. 

Para tenerlo siempre a la mano podemos ir a la pagina principal del repo 

```text
About → Engrane → seleccionar Use your GitHub Pages website → Save changes
```

y la dirección aparecerá siempre debajo de `About`.

---

# 19. Flujo normal después de configurar todo

Una vez configurado el proyecto, ya no necesitamos publicar manualmente.

Trabajaremos así:

```bash
git status
git add .
git commit -m "Add airline delay chart"
git push
```

Y GitHub hará automáticamente:

```text
git push
    ↓
GitHub Actions
    ↓
instalar R
    ↓
instalar paquetes
    ↓
renderizar dashboard
    ↓
crear _site/index.html
    ↓
GitHub Pages
```

---

# 20. Estructura final del repositorio

El repositorio debería quedar aproximadamente así:

```text
nycflights-dashboard/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── .gitignore
├── dashboard.Rmd
└── nycflights-dashboard.Rproj
```

Localmente también podemos tener:

```text
_site/
└── index.html
```

pero Git lo ignorará.

---

# 21. Errores frecuentes

## Error: `there is no package called ...`

Por ejemplo:

```text
Error in library(plotly):
there is no package called 'plotly'
```

Significa que el dashboard utiliza un paquete que no hemos instalado en GitHub Actions.

Añadirlo a:

```yaml
Install R packages
```

---

## Error de autenticación al hacer `git push`

Primero comprobar:

```bash
git remote -v
```

Si aparece:

```text
git@github.com:...
```

estamos utilizando SSH.

Si queremos utilizar PAT + `gitcreds`, cambiar a HTTPS:

```bash
git remote set-url origin https://github.com/USUARIO/REPOSITORIO.git
```

Después:

```r
gitcreds::gitcreds_set()
```

y volver a probar:

```bash
git push
```

---

## Error al hacer `git push`

Por ejemplo:

```text
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/...'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally ...
```

Significa que creaste archivos directamente en GitHub (como un `README.md` o una licencia) o trabajaste desde otra computadora, y esos cambios no existen en tu versión local. Git bloquea el envío para evitar pérdida de información.

Para solucionarlo y sincronizar tu proyecto, ejecuta en la terminal:

```bash
git pull origin main --rebase
```

Una vez que termine sin errores, vuelve a intentar el envío:

```bash
git push origin main
```

---

## Error: `remote origin already exists`

Significa que ya existe un remoto llamado `origin`.

Comprobar:

```bash
git remote -v
```

No volver a ejecutar:

```bash
git remote add origin ...
```

Si necesitamos cambiar la dirección utilizar:

```bash
git remote set-url origin https://github.com/USUARIO/REPOSITORIO.git
```

---

## Error: GitHub Pages no aparece

Comprobar:

```text
Settings → Pages → Source → GitHub Actions
```

Después comprobar la pestaña:

```text
Actions
```

y verificar que los jobs:

```text
build
deploy
```

terminaron correctamente.

---

## Error: el dashboard funciona en mi computadora pero falla en Actions

Normalmente significa que nuestra computadora tiene algo que el runner no tiene.

Por ejemplo:

- un paquete de R
- un archivo local
- una ruta absoluta
- una variable de entorno
- datos almacenados solamente en nuestro equipo

Esto es precisamente uno de los beneficios de GitHub Actions: nos obliga a comprobar que el proyecto es reproducible.

---

# 22. Seguridad

Nunca subir a GitHub:

- Personal Access Tokens
- contraseñas
- API keys
- secretos
- credenciales
- información confidencial

El PAT utilizado para hacer `git push` debe permanecer almacenado de forma segura en la computadora mediante el sistema de credenciales.

Nuestro workflow de GitHub Actions no necesita conocer ese PAT personal para publicar GitHub Pages.

GitHub Actions utiliza sus propias credenciales y permisos durante la ejecución.

---
