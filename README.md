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
- [Otras Consideraciones](#otras-consideraciones)
  - [Branching de La base de Datos](#branching-de-la-base-de-datos)

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

```ruby
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

#### 6. Configurar la base de datos

&nbsp;

> :warning: Los comandos `rake`, `rails` y otros deben ejecutarse junto al comando `bundle exec` de manera que puedan correr las versiones correctas de las gems y su dependencias.

&nbsp;

Crear la base de datos:

```ruby
  bundle exec rake db:create
```

En caso de que sea necesario eliminar la base de datos se puede ejecutar el siguiente comando:

```ruby
  bundle exec rake db:drop
```

&nbsp;

#### 7. Crear la estructura de la base de datos:

&nbsp;

> :warning:
> En los siguientes pasos es necesario que los servidores de **PostgreSQL** y **Redis** esten corriendo. Tambien asegurarse que el comando de Postgres corresponda a la version de Postgres instalada en el sistema.

&nbsp;

Para PostgreSQL v10.x

```ruby
  bundle exec rake db:structure:load SCHEMA=db/structure.pg10.sql
```

Para PostgreSQL v11.x

```ruby
  bundle exec rake db:structure:load SCHEMA=db/structure.pg11.sql
```

Estos comandos crearan todas las tablas y estructuras generales de la base de datos, pero estaran vacias.

&nbsp;

#### 7. Migrar la ultima versión de la base de datos:

```ruby
  bundle exec rake db:migrate
```

&nbsp;

#### 8. Correr el local server:

```ruby
  bundle exec rails s
```

Rails hace uso de la poderosa _CLI_ de Ruby y se puede ejecutar con el comando:

```bash
  bundle exec rails c
```

## Otras Consideraciones

#### Branching de la Base de Datos

Sometimes when we are working on features we need to make changes to the **database** structure, and ofently they are destructive, maybe remove a column or rename it. That's why it's important on this cases to work on a **branched database** so if you need to go back to another branch to work on another thing, everything keeps working as expected.

If you duplicate the `database.yml.example` file you will see that in the first few lines there is some _weird_ ruby code. This code allows you to reference to another database if there is a `true` value on a config in the git branch.

So to do it, you need to run the following commands on the branch in which you need to create a new database, you should replace `[BRANCH_NAME]` with the actual name of the branch:

```bash
  # Remember to replace [BRANCH_NAME] with the actual name of the branch
  git config --bool branch.[BRANCH_NAME].database true
```

With this the rails project knows that it will look for a database named: `do-lenga-dev-db_[BRANCH_NAME]` (if you did not change the default name on your `database.yml`)

So now you need to create that new database

> All the `rake`, `rails` and other commands should be runned with the `bundle exec` in order to ensure the use of the right versions of the gems and their dependencies

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

#### Comments in code

Let's avoid to comment code, only leave them when is necessary.
Whenever we need to mention that a refactor or another thing is needed in the future,
leave them with a `# TO-DO:` and then describe it
