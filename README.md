# Blanco-Salguero-Tarea23-Instalar-Clientes-para-tres-SGBD

## Comprobaciones de SGBD (Antes de Empezar)

Antes de empezar comprobamos si los SGBD funcionan con los siguientes comandos:
* **Oracle 23ai Free** _(Puerto: 1521)_
````bash
sudo podman ps -a
````
![Oracle estado](/evidencias/oracle_SGBD.png)
_Demostración de estado `healthy` de Oracle._
* **PostgreSQL** _(Puerto: 5432)_
````bash
systemctl status postgresql
````
![Postgre estado](/evidencias/postgreSQL_SGBD.png)
_Demostración de que el servicio `PostgreSQL` está activo._
* **MariaDB** _(Puerto: 3306)_
````bash
systemctl status mariadb
````
![Maria estado](/evidencias/mariaDB_SGBD.png)
_Demostración de que el servicio `MariaDB` está activo._

## Prerrequisitos en la VM

Primero tenemos que tener instalado un JDK y unzip, para ello usaremos los siguientes comandos:

````bash
sudo apt update
sudo apt install default-jdk unzip -y
````

Ahora comprobamos la version del JDK que tenemos con:

````bash
java --version
````

La version de **Java** que tenemos es la siguiente: `openjdk 21.0.12.1`

## SQL Developer

A continuación tenemos que descargar el [SQL Developer](https://www.oracle.com/database/sqldeveloper/technologies/download/).

Una vez descargado lo descomprimimos en `/opt`, tenemos que usar el nombre del archivo que hemos descargado, en mi caso `sqldeveloper-26.2.0.186.2220-x64.zip`, así que mi comando a usar sera el siguiente:

````bash
sudo unzip sqldeveloper-26.2.0.186.2220-x64.zip -d /opt/
````

y para ejecutarlo tenemos que hacerlo con el comando:

````bash
/opt/sqldeveloper/sqldeveloper.sh
````

En caso de querer poner un icono en el menu de inicio para abrirlo cómodamente desde el, primero tenemos que comprobar las rutas del Exec y del Icon, las cuales deben coincidir lo comprobamos con los comandos:

````bash
ls /opt/sqldeveloper/sqldeveloper.sh
ls /opt/sqldeveloper/icon.png
````

Y la ruta debe coincidir.

Después debes poner el siguiente comando para crear el lanzador en el menú:

````bash
sudo tee /usr/share/applications/sqldeveloper.desktop > /dev/null <<'EOT'
[Desktop Entry]
Version=1.0
Name=SQL Developer
Comment=Entorno de desarrollo SQL
Exec=/opt/sqldeveloper/sqldeveloper.sh
Icon=/opt/sqldeveloper/icon.png
Terminal=false
Type=Application
Categories=Development;Database;
Keywords=database;db;sql;query;administration;
EOT
````
### Conexión con `Orauser`

Una vez abierto **SQL Developer** en la interfaz gráfica, en el menú de la izquierda `Conexiones` aparece un `+`, lo pulsamos para crear una nueva conexión y rellenamos los siguientes datos:

* **Usuario:** `orauser`
* **Host:** `localhost` (el cliente y el servidor están en la misma VM)
* **Puerto:** `1521`
* **Service Name:** `FREEPDB1`

Pulsamos `Probar` y comprobamos que el estado es `Correcto`, después `Conectar`.

![Datos de la conexión a Oracle](/evidencias/prueba_conexion_SQL_Developer.png)
_Muestra `Estado: Correcto` en la prueba de conexión a `FREEPDB1` con `orauser` desde `SQL Developer`._

Como se puede apreciar, al darle a `Probar`, en **Estado:** nos ha aparecido `Correcto`

### Cliente gráfico: VS Code y extensión SQL Developer

Para usar VS Code con su extensión de SQL Developer, primero debemos instalar [VS Code](https://code.visualstudio.com/).

Una vez descargado nos dirigimos a la carpeta `Descargas` y lo instalamos:
````bash
cd ~/Descargas
sudo apt install ./code_1.142.0-1792035033_amd64.deb #En mi caso el paquete descargado tiene ese nombre.
````

Una vez instalado lo abrimos, vamos al apartado de extensiones y buscamos la extensión `Oracle SQL Developer`, una vez encontrada la instalamos y configuramos la conexión de la misma manera.

Una vez instalada la extensión, creamos la conexión con los mismos datos que en SQL Developer:

* **Usuario:** `orauser`
* **Host:** `localhost` (el cliente y el servidor están en la misma VM)
* **Puerto:** `1521`
* **Service Name:** `FREEPDB1`

Guardamos la conexión y se establece sin problemas.

![Conexión con la extensión de VS Code a Oracle](/evidencias/prueba_conexion_SQL_Developer_vscode.png)
_Muestra la conexión a `FREEPDB1` con `orauser` desde la extension de VS Code._

### Cliente de Terminal: SQL*Plus

Para Mint vamos a descargar los ZIP de Oracle Instant Client para Linux x86-64: Basic y SQL*Plus de la misma versión (23.26.3.0.0), los descargamos desde [aquí](https://www.oracle.com/database/technologies/instant-client/downloads.html)

Ahora que hemos descargado los zip, creamos la carpeta `oracle-client` y descomprimimos los archivos `.zip` descargados.

````bash
mkdir -p ~/oracle-client
cd ~/Descargas
unzip instantclient-basic-linux.x64-23.26.3.0.0.zip -d ~/oracle-client
unzip instantclient-sqlplus-linux.x64-23.26.3.0.0.zip -d ~/oracle-client
ls ~/oracle-client
````

Esto nos crea la carpeta `instantclient_23_26`. Para poder usar el cliente exportamos las variables de entorno:

````bash
export LD_LIBRARY_PATH=~/oracle-client/instantclient_23_26:$LD_LIBRARY_PATH
export PATH=~/oracle-client/instantclient_23_26:$PATH
````

Ahora comprobamos que funciona y vemos la versión con el comando:
````bash
sqlplus -v
````

Y ahora nos conectamos con el siguiente comando:
````bash
sqlplus orauser@//localhost:1521/FREEPDB1
````

![Prueba_SQLPLUS](/evidencias/prueba_conexion_SQLPlus.png)
_Muestra la conexión a `FREEPDB1` con `orauser` desde el cliente de terminal `SQL*PLUS`_

## Clientes para PostgreSQL

### Cliente de Terminal: psql

El cliente de consola de PostgreSQL es `psql`. Lo instalamos con:

````bash
sudo apt update
sudo apt install postgresql-client
````

Ahora comprobamos la versión que hemos instalado con:

````bash
psql --version
````

En mi caso es la versión `16.15`. Ahora, nos conectaremos a la base de datos con el usuario `pguser`. **IMPORTANTE:** Usar la opción `-W` para no poner la contraseña en el comando.

````bash
psql -h localhost -U pguser -d dbpg -W
````

Para demostrar la conexión una vez ya conectado a la base de datos:

![Postgre_en_Terminal](/evidencias/postgre_terminal.png)
_Muestra la conexión a `dbpg` con pguser desde el cliente de terminal `psql`_

### Cliente Gráfico: pgAdmin 4

pgAdmin 4 no está en los repositorios de Mint, así que añadimos el [repositorio oficial](https://www.pgadmin.org/download/pgadmin-4-apt/). En esa página se ven las versiones admitidas, y mi Mint 22 se basa en Ubuntu 24.04. Por eso ponemos `noble` a mano en el repositorio en vez de usar `$(lsb_release -cs)`:

````bash
sudo apt install curl gnupg lsb-release
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /etc/apt/keyrings/packages-pgadmin-org.gpg
sudo sh -c 'echo "deb [signed-by=/etc/apt/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/noble pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list'
sudo apt update
sudo apt install pgadmin4-desktop
````

A continuación, abrimos pgAdmin 4, le damos a `add new server` y empezamos a rellenar los datos para realizar la conexión:

* **Host**: `localhost`
* **Port**: `5432`
* **Maintenance DB**: `dbpg`
* **Username**: `pguser`
* **Password**: (Su contraseña)

![PostgreSQL_con_pgAdmin4](/evidencias/pgAdmin4.png)
_Muestra la conexión a `dbpg` con `pguser` desde pgAdmin 4_

## Clientes para MariaDB

### Cliente de Terminal: mariadb

El cliente de consola de MariaDB es `mariadb`. Lo instalamos con:

````bash
sudo apt update
sudo apt install mariadb-client
````

A continuación comprobamos la versión con el comando:

````bash
mariadb --version
````

En mi caso es la `10.11.14`. Ahora, nos conectamos a la base de datos `dbm` con el usuario `muser` y usamos la opción `-p` para que pida la contraseña en vez de ponerla en el comando.

````bash
mariadb -h localhost -u muser -p dbm
````

Ahora que estamos dentro usamos la siguiente consulta para demostrar la conexión:

````sql
SELECT CURRENT_USER(), DATABASE();
````

![terminal_conexión_mariadb](/evidencias/terminal_mariadb.png)
_Muestra la conexión a dbm con el usuario muser desde el cliente de terminal mariadb y la consulta que lo demuestra._

### Cliente Web: phpMyAdmin

Para usar phpMyAdmin, primero tenemos que instalar `apache2`, `php` y `phpMyAdmin`, para ello usaremos los comandos:

````bash
sudo apt update
sudo apt install apache2 php php-mysql php-mbstring php-zip php-gd php-curl phpmyadmin
sudo phpenmod mbstring
sudo systemctl restart apache2
````

Durante la instalación, en el asistente seleccionamos `apache2` como servidor web.

> [!NOTE]
> La configuración de Apache no se activó sola: el asistente no llegó a crear el enlace en Apache, así que lo creamos a mano:
>````bash
>sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
>sudo a2enconf phpmyadmin
>sudo systemctl reload apache2
>````

Ahora vamos al navegador e iniciamos sesión con `muser` y su contraseña para comprobar `dbm`

![phpmyadmin_conexion](/evidencias/phpmyadmin_conexion.png)
_Se ve que se ha iniciado sesión en phpMyAdmin_

## Clientes para varios SGBD

### DBeaver Community

Descargamos el `.deb` de [DBeaver](https://dbeaver.io/download/) y lo instalamos con los comandos:

````bash
cd ~/Descargas
sudo apt install ./dbeaver-ce-26.2.3-linux-x86_64.deb
````


Una vez instalado, lo abrimos y seleccionamos el idioma deseado, después comenzamos a realizar las diferentes conexiones con nuestros diferentes SGBD, para ello tendremos que darle a `nueva conexión de base de datos` y seleccionar nuestra base de datos.

Los datos que usamos en cada conexión son:

* **Oracle:** Host `localhost`, puerto `1521`, base de datos `FREEPDB1` y usuario `orauser`
* **PostgreSQL:** Host `localhost`, puerto `5432`, base de datos `dbpg` y usuario `pguser`
* **MariaDB:** Host `localhost`, puerto `3306`, base de datos `dbm` y usuario `muser`

En todas ellas usamos el tipo de autenticación de usuario y contraseña. La primera vez que probamos cada conexión, DBeaver nos pide descargar el driver del SGBD correspondiente, le damos a aceptar y espera a que termine la descarga.

![DBeaver_con_los_tres](/evidencias/DBeaver.png)
_DBeaver conectado a los 3 SGBD_


<!--ERRORES-->
## Incidencias

### Incidencia: SQL Developer no arranca en Linux (`Module javafx.base not found`)

**Cliente:** Oracle SQL Developer 26.2.0 (ZIP para Linux, ejecutando `/opt/sqldeveloper/sqldeveloper.sh`)

**Problema:** Al arrancar fallaba al instante con:
`Error occurred during initialization of boot layer — java.lang.module.FindException: Module javafx.base not found`

**Causa:** SQL Developer necesita JavaFX, que desde JDK 11 no se incluye en el JDK.
Además, el ZIP oficial solo empaqueta los módulos de JavaFX para Windows y macOS
(`modules/javafx/windows-x64` y `osx-aarch64`); la ruta configurada para Linux
en `sqldeveloper/bin/java11.conf` apunta a una carpeta `linux-x64` que no existe.

**Solución:** Instalar OpenJFX del sistema (`sudo apt install openjfx`, JAR en
`/usr/share/openjfx/lib`) y editar la línea de Linux en `java11.conf`:

````bash
sudo sed -i 's|--module-path=../../modules/javafx/linux-x64/lib|--module-path=/usr/share/openjfx/lib|' /opt/sqldeveloper/sqldeveloper/bin/java11.conf
````

#### Muestra del ERROR:
![ERROR_SQL_DEVELOPER](/evidencias/ERROR_SQLDeveloper.png)

#### Muestra de que esta funcionando:

![SOLUCIONADO_SQL_DEVELOPER](/evidencias/Solu_SQLDeveloper.png)

### Incidencia: SQL*Plus no arranca por `libaio.so.1`

**Cliente:** SQL*Plus 23.26 (Instant Client 23.26.3.0.0 en `~/oracle-client/instantclient_23_26`)

**Problema:** Al ejecutar `sqlplus -v` fallaba con:
`sqlplus: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory`

**Causa:** aunque el paquete `libaio1t64` estaba instalado, la librería se renombró a `libaio.so.1t64` en la transición *t64* y no se crea el enlace
`libaio.so.1` que los binarios de Oracle esperan.

**Solución:** crear el enlace simbólico:

````bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libaio.so.1t64 /usr/lib/x86_64-linux-gnu/libaio.so.1
````
#### Muestra del ERROR:
![ERROR_libaio](/evidencias/ERROR_libaio.png)

#### Muestra de que esta funcionando:
![Prueba_SQLPLUS](/evidencias/prueba_conexion_SQLPlus.png)

## Fuentes:

* [Oracle Database / SQL Developer / Instant Client](https://www.oracle.com/database/technologies/)
* [pgAdmin 4](https://www.pgadmin.org/download/pgadmin-4-apt/)
* [phpMyAdmin](https://www.phpmyadmin.net/)
* [DBeaver](https://dbeaver.io/download/)
* [MariaDB](https://mariadb.org/)
* Mistral AI: asistencia para dudas y errores.
