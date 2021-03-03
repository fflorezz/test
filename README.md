# Lenga

Aplicacion pricipal de Donar Online...

&nbsp;

## Tabla de Contenidos

- [Prerrequisitos](#prerrequisitos)
- [Instalación](#instalación)
  - [1. Instalar ImageMagick](#1-instalar-imagemagick)
  - [2. Clonar el repo](#2-clonar-el-repo-e-ingresar-al-directorio-lenga)
  - [3. Instalar Bundler 2](#3-instalar-bundler-2-y-ejecutar-el-comando-bundle)
  - [4. Configurar archivos del directorio config/](#4-configurar-archivos-del-directorio-config)
  - [5. Añadir el archivo GeoLite](#5-añadir-el-archivo-geolite)
  - [6. Configurar la base de datos](#6-configurar-la-base-de-datos)
  - [7. Crear la estructura de la base de datos](#7-crear-la-estructura-de-la-base-de-datos)
  - [8. Migrar la ultima versión de la base de datos](#8-migrar-la-ultima-versión-de-la-base-de-datos)
  - [9. Correr el local server](#9-correr-el-local-server)
- [Otras Consideraciones](#otras-consideraciones)
  - [Branching de La base de Datos](#branching-de-la-base-de-datos)
  - [Comentarios en el Código](#comentarios-en-el-código)

&nbsp;

## Prerrequisitos

- Ruby >= 2.4
- PostgreSQL >= 9.6
- Redis >= 3
- ImageMagick 6.x (Hay algunos problemas entre las últimas versiones y El Capitan OS en Mac, por esta razón se recomienda usar la versión 6 en el entorno de desarrollo)
- Node 10.x

&nbsp;

## Instalación

#### 1. Instalar ImageMagick

Descargar el paquete de ImageMagick:

```bash
  wget https://www.imagemagick.org/download/ImageMagick-6.9.12-2.tar.gz
```

Extraer el contenido e ingresar al directorio:

```bash
  tar xvzf ImageMagick.tar.gz
```

```bash
  cd ImageMagick-6.9.12-2/
```

Configurar y compilar ImageMagick con el soporte de modulos:

```bash
  ./configure --with-modules=yes
```

Ejecutar el comando `make` para compilar:

```bash
  make
```

Instalar:

```bash
  sudo make install
```

Verificar la instalación:

```bash
  identify -version
  # Version: ImageMagick 6.9.12-2 Q16 x86_64 2021-02-27 https://imagemagick.org
  # Copyright: (C) 1999-2021 ImageMagick Studio LLC
  # License: https://imagemagick.org/script/license.php
  # Features: Cipher DPC Modules OpenMP(4.5)
  # Delegates (built-in): bzlib djvu fontconfig freetype jbig jng jpeg lcms lqr ltdl lzma openexr png tiff wmf x xml zlib
```

&nbsp;

#### 2. Clonar el repo e ingresar al directorio `lenga`:

```bash
  git clone git@github.com:donaronline/lenga.git
```

```bash
  cd lenga
```

&nbsp;

#### 3. Instalar Bundler 2 y ejecutar el comando `bundle`:

```ruby
  gem install bundler -v '~> 2.0'
```

```bash
  bundle
```

&nbsp;

#### 4. Configurar archivos del directorio `config/`

Hay varios archivos dentro del directorio `config/` con la extensión `.yml.example`:

```bash
  config/
    ╰─ database.yml.example
    ╰─ lenga.yml.example
    ╰─ services.yml.example
```

Duplicar cada uno de los archivos y remover la extensión `.example`:

```bash
  config/
    ╰─ database.yml
    ╰─ database.yml.example
    ╰─ lenga.yml
    ╰─ lenga.yml.example
    ╰─ services.yml
    ╰─ services.yml.example
```

&nbsp;

#### 5. Añadir el archivo `GeoLite`:

Descargar la base de datos mas reciente del siguiente link: [https://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.tar.gz](https://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.tar.gz)

Descomprimir el archivo `GeoLite2-Country.tar.gz` y añadirlo al directorio `tmp/`.
En caso de que no exista el directorio `tmp/` crearlo:

```bash
 lenga/
  ╰─ app/
  ╰─ bin/
  ╰─ config/
  ╰─ tmp/
    ╰─ GeoLite2-Country.mmdb
```

&nbsp;

#### 6. Configurar la base de datos:

&nbsp;

> :warning: Los comandos `rake`, `rails` y otros deben ejecutarse junto al comando `bundle exec` de manera que puedan correr las versiones correctas de las gems y su dependencias.

&nbsp;

Crear la base de datos:

```bash
  bundle exec rake db:create
```

En caso de que sea necesario eliminar la base de datos se puede ejecutar el siguiente comando:

```bash
  bundle exec rake db:drop
```

&nbsp;

#### 7. Crear la estructura de la base de datos:

&nbsp;

> :warning:
> En los siguientes pasos es necesario que los servidores de **PostgreSQL** y **Redis** esten corriendo. Tambien asegurarse que el comando de Postgres corresponda a la version de Postgres instalada en el sistema.

&nbsp;

Para PostgreSQL v10.x

```bash
  bundle exec rake db:structure:load SCHEMA=db/structure.pg10.sql
```

Para PostgreSQL v11.x

```bash
  bundle exec rake db:structure:load SCHEMA=db/structure.pg11.sql
```

Estos comandos crearan todas las tablas y estructuras generales de la base de datos, pero estaran vacias.

&nbsp;

#### 8. Migrar la ultima versión de la base de datos:

```bash
  bundle exec rake db:migrate
```

&nbsp;

#### 9. Correr el local server:

```bash
  bundle exec rails s
```

Rails hace uso de la potente _CLI_ de Ruby, a la cual se puede acceder con el comando:

```bash
  bundle exec rails c
```

&nbsp;

## Otras Consideraciones

#### Branching de la Base de Datos

A veces cuando se trabaja en una nueva funcionalidad es necesario hacer cambios en la estructura de la base de datos. Estos cambios por lo general suelen ser destructivos. Por este motivo es importante en estos casos trabajar en una **branched database**, de manera que si se necesita volver a trabajar en una **branch** todo seguirá funcionando de forma correcta.

Para trabajar en una **branched database** Rails hace uso de la configuración de `git` en las primeras lineas del archivo `database.yml`:

```bash
  <%
    # git config --bool branch.feature.database true
    # http://mislav.uniqpath.com/rails/branching-the-database-along-with-your-code/
    branch = `git symbolic-ref HEAD 2>/dev/null`.chomp.sub('refs/heads/', '')
    suffix = `git config --bool branch.#{branch}.database`.chomp == 'true' ? "_#{branch}" : ""
  %>
```

Par habilitar la **branched database** es necesario ejecutar el siguiente comando en la **branch** donde se va a crear la nueva base de datos.

&nbsp;

> :warning: Se debe remplazar `[BRANCHE_NAME]` con el nombre de la nueva base de datos.

&nbsp;

```bash
  # Remember to replace [BRANCH_NAME] with the actual name of the branch
  git config --bool branch.[BRANCH_NAME].database true
```

Rails buscará la base de datos `do-lenga-dev-db_[BRANCH_NAME]` (si no se cambio el nombre por defecto en `database.yml`).

Ahora solo hace falta crear la nueva base de datos:

```bash
  bundle exec rake db:create
  # ... output of the database creation, you should see the names of the new database in here
```

And lastly you can load data from other database, for example in this case from the `master` branch (replace `[USERNAME]` and `[BRANCH_NAME]` accordly)

```bash
  # Remember to replace [USERNAME] and [BRANCH_NAME] accordly
  pg_dump -U [USERNAME] -h localhost do-lenga-dev-db | psql -U [USERNAME] -d do-lenga-dev-db_[BRANCH_NAME]

  # If you want to copy everything from another database, for instance from the branch staging, then you should run:
  pg_dump -U [USERNAME] -h localhost do-lenga-dev-db_staging | psql -U [USERNAME] -d do-lenga-dev-db_[BRANCH_NAME]
```

Or also you can load a blank or empty structure following the step [6](#6---create-the-database-structure-be-sure-to-run-the-right-one-for-your-postgresql-version) on the [Steps to run in development](#steps-to-run-in-development) section

#### Comentarios en el Código

Let's avoid to comment code, only leave them when is necessary.
Whenever we need to mention that a refactor or another thing is needed in the future,
leave them with a `# TO-DO:` and then describe it
