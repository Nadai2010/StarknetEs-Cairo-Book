# Instalación

El primer paso es instalar Cairo. Descargaremos Cairo manualmente, utilizando el repositorio de Cairo o un script de instalación. Necesitará una conexión a Internet para la descarga.

### Requisitos previos

Primero deberá tener Rust y Git instalados.

```bash
# Install stable Rust
rustup override set stable && rustup update
```

Instalar [Git](https://git-scm.com/).

## Instalando Cairo con un Script ([Instalador](https://github.com/franalgaba/cairo-installer) por [Fran](https://github.com/franalgaba))

### Instalación

Si deseas instalar una versión específica de Cairo en lugar de la última versión, debes establecer la variable de entorno `CAIRO_GIT_TAG` (por ejemplo,`export CAIRO_GIT_TAG=v1.0.0-alpha.6`).

```bash 
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```

Tras la instalación, sigue [estas instrucciones](#set-up-your-shell-environment-for-cairo) para configurar tu entorno shell.

### Actualización

```
rm -fr ~/.cairo
curl -L https://github.com/franalgaba/cairo-installer/raw/main/bin/cairo-installer | bash
```

### Desinstalación

Cairo se instala dentro de `$CAIRO_ROOT` (por defecto: ~/.cairo). Para desinstalarlo, basta con eliminarlo:

```bash
rm -fr ~/.cairo
```

luego elimina estas tres líneas de .bashrc:

```bash
export PATH="$HOME/.cairo/target/release:$PATH"
```

y, por último, reinicia tu shell:

```bash
exec $SHELL
```

### Configure su entorno shell para Cairo

* Define la variable de entorno `CAIRO_ROOT` para apuntar a la ruta donde
  Cairo almacenará sus datos. Por defecto es `$HOME/.cairo`.
  Si instaló Cairo a través de Git checkout, recomendamos
  la misma ubicación donde lo clonaste.
* Añade los ejecutables de `cairo-*` a tu `PATH` si no están ya allí.

La siguiente configuración debería funcionar para la gran mayoría de usuarios en casos de uso comunes.

  - Para **bash**:
  
  Los archivos de inicio de Bash predeterminados varían ampliamente entre las distribuciones, en cuanto a la fuente de ellos, las circunstancias, el orden y la configuración adicional que realizan. 
  
  Como tal, la forma más confiable de obtener Cairo en todos los entornos es agregar los comandos de configuración de Cairo tanto en `.bashrc` (para shells interactivos) como en el archivo de perfil que Bash utilizaría (para shells de inicio de sesión).

En primer lugar, agrega los comandos a `~/.bashrc` ejecutando lo siguiente en tu terminal:

    ~~~ bash
    echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bashrc
    echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bashrc
    ~~~

    Entonces, si tiene `~/.profile`, `~/.bash_profile` o `~/.bash_login`, añada también allí los comandos.
    Si no tienes ninguno de estos, añádelos a `~/.profile`.

    * Para añadir a `~/.profile`:
      ~~~ bash
      echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.profile
      echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.profile
      ~~~

    * Para añadir a `~/.bash_profile`:
      ~~~ bash
      echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.bash_profile
      echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.bash_profile
      ~~~

  - Para **Zsh**:
    ~~~ zsh
    echo 'export CAIRO_ROOT="$HOME/.cairo"' >> ~/.zshrc
    echo 'command -v cairo-compile >/dev/null || export PATH="$CAIRO_ROOT/target/release:$PATH"' >> ~/.zshrc
    ~~~

    Si también desea obtener Cairo en shells de inicio de sesión no interactivos, añada también los comandos a `~/.zprofile` or `~/.zlogin`.

  - Para **Fish shell**:

    Si tiene Fish 3.2.0 o posterior, ejecútelo interactivamente:

    ~~~ fish
    set -Ux CAIRO_ROOT $HOME/.cairo
    fish_add_path $CAIRO_ROOT/target/release
    ~~~

    De lo contrario, ejecute el siguiente fragmento:

    ~~~ fish
    set -Ux CAIRO_ROOT $HOME/.cairo
    set -U fish_user_paths $CAIRO_ROOT/target/release $fish_user_paths
    ~~~

   En MacOS, es posible que también desee instalar [Fig](https://fig.io/) que proporciona complementos de shell alternativos para muchas herramientas de línea de comandos con una interfaz emergente similar a IDE en la ventana de terminal.
(Tenga en cuenta que sus complementos son independientes del código base de Cairo
por lo que pueden estar ligeramente desincronizadas para cambios de interfaz de última generación).

### Reinicia tu shell

 para que los cambios en `PATH` surtan efecto.

  ```sh
  exec "$SHELL"
  ```

## Instalando Cairo Manualmente ([Guía](https://github.com/auditless/cairo-template) por [Abdel](https://github.com/abdelhamidbakhta))

### Paso 1: Instalar Cairo 1.0

Si utiliza un sistema Linux x86 y puede utilizar el binario de lanzamiento, descargue Cairo aquí:
<https://github.com/starkware-libs/cairo/releases>.

Para todos los demás, recomendamos compilar Cairo desde el código fuente de la siguiente manera:

```bash
# Start by defining environment variable CAIRO_ROOT
export CAIRO_ROOT="${HOME}/.cairo"

# Create .cairo folder if it doesn't exist yet
mkdir $CAIRO_ROOT

# Clone the Cairo compiler in $CAIRO_ROOT (default root)
cd $CAIRO_ROOT && git clone git@github.com:starkware-libs/cairo.git .

# OPTIONAL/RECOMMENDED: If you want to install a specific version of the compiler
# Fetch all tags (versions)
git fetch --all --tags
# View tags (you can also do this in the cairo compiler repository)
git describe --tags `git rev-list --tags`
# Checkout the version you want
git checkout tags/v1.0.0-alpha.6

# Generate release binaries
cargo build --all --release
```

. 

**NOTA: Mantener Cairo actualizado**

Ahora que tu compilador Cairo está en un repositorio clonado, todo lo que necesitas hacer
es extraer los últimos cambios y reconstruir como sigue:

```bash
cd $CAIRO_ROOT && git fetch && git pull && cargo build --all --release
```

### Paso 2: Añade los ejecutables de Cairo 1.0 a tu ruta

```bash
export PATH="$CAIRO_ROOT/target/release:$PATH"
```

**NOTA: Si instala desde un binario Linux, adapte la ruta de destino en consecuencia.

### Paso 3: Configurar el servidor de idiomas

#### Extensión VS Code

- Deshabilite la extensión anterior Cairo 0.x
- Instale la extensión Cairo 1 para un correcto resaltado de sintaxis y navegación por el código.
Simplemente siga los pasos indicados [aquí](https://github.com/starkware-libs/cairo/blob/main/vscode-cairo/README.md).

#### Servidor de Lenguaje de Cairo

Desde [Paso 1](#step-1-install-cairo-10-guide-by-abdel), el binario `cairo-language-server` debe ser construido y ejecutando este comando se copiará su ruta en el portapapeles.

```bash
which cairo-language-server | pbcopy
```

Actualiza el `languageServerPath` de la extensión Cairo 1.0 pegando la ruta.