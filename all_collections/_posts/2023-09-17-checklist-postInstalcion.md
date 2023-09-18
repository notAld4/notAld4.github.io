---
layout: post
title: Configuracion de criterios Post-instalacion de mysql
categories: [mysql, AdminDB]
---

Antes que nada, localizar el archivo ```my.cnf```:

```bash
find / -name my.cnf 2> /dev/null
```
La ruta por defecto suele ser ```/etc/mysql/my.cnf```

Para ver los permisos puedes ejecutar:

```bash
ls -l /etc/mysql/my.cnf
```

Es posible que solo el usuario sea root pueda editarlo, ten en cuenta eso

## Politicas de contraseñas

Desde la CLI de mysql, instala el plugin ```validate_password```:

```sql
INSTALL COMPONENT 'file://component_validate_password';
```

##### 1. La longitud de las contraseñas debe ser mayor o igual a 25 caracteres para las cuentas de usuario antes de ser creadas

Abrir el archivo ```my.cnf``` e ingresa:

```
validate_password.length = 25
```
Con esto le estas indicando que la longitud se minimo de 25, caracteres, aun que puedes establecerlo como sea necesario

Para corroborar ingresa

```sql
SHOW VARIABLES LIKE 'validate_password%';
```

##### 2. Las contraseñas deben ser cambiadas con regularidad, debido a esto, se debe de establecer una fecha de expiracion cada 15 dias

En el archivo ```my.cnf``` ingresa:

```sql
default_password_lifetime=15
```

Para corroborar ingresa:

```sql
SHOW VARIABLES LIKE 'default_password%';
```

Para crear usuarios ahora se usa al final la sentencia ```PASSWORD EXPIRE;``` de la siguiente forma:

```sql
CREATE USER 'alda'@'localhost' IDENTIFIED BY "password" PASSWORD EXPIRE;
```

##### 3. Las contraseñas para todos los usuarios de la base de datos deben de estar cifradas usando minimo SHA-256

En el archivo ```my.cnf``` ingresa:

```sql
default_authentication_plugin=sha256_password
```

Para corroborar ingresa:

```sql
SHOW VARIABLES LIKE 'default_authentication%';
```

Y debe de salir:

```sql
sha256_password
```

Ahora los usuarios los debes de crear usando ```IDENTIFIED WITH sha256_password```

```sql
CREATE USER 'alda'@'localhost' IDENTIFIED WITH sha256_password BY 'password' PASSWORD EXPIRE;
```

##### 4. Cambiar la contraseña por defecto para el acceso de la cuenta de root, ademas cumplir con los criterios anteriores

Se realizo cuando se hizo la instalacion segura de mysql, en caso de que no, primero ingresar:

```sql
FLUSH PRIVILEGES;
```

Y despues:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH sha256_password BY 'password' PASSWORD EXPIRE;
```

##### 5. Las contraseñas para todos los usuarios deben de tener al menos un simbolo, mayuscula, minuscula y numero

En el archivo ```my.cnf``` ingresa:

```sql
validate_password.mixed_case_count=1
validate_password.number_count=1
validate_password.special_char_count=1
```

- El ```mixed_case_count``` indica el total de mayusculas y minusculas, en este caso 1
- El ```number_count``` indica la cantidad de digitos numericos como minimo
- El ```special_char_count``` indica la cantidad de caracteres especiales como minimo

##### 6. Los usuarios que deseen cambiar su contraseña, necesitan asegurar que conocen su contraseña actual

En el archivo ```my.cnf``` ingresa:

```sql
password_require_current=ON
```

Para corroborar ingresa:

```sql
SHOW VARIABLES LIKE ’password_require%';
```

Al final todo te debe de salir asi:

![](/assets/img/checklist/password.png)

Y la manera final de crear usuarios sera usando esta sintaxis y tomando en cuenta que debe tener una longitud de 25 caracteres, 1 mayuscula, 1 caracter especial, 1 numero, y 1 minuscula

```sql
CREATE USER 'alda'@'localhost' IDENTIFIED WITH sha256_password BY 'password' PASSWORD EXPIRE;
```

Para guardar cambios solo reinica el servicio de mysql y no te debe de dar error

```bash
sudo systemctl restart mysql
```

## Politicas de cuentas de usuario (mysql)

##### 1. Las cuentas de usuario no privilegiado se deben de crear especificando la direccion IP desde donde se conectaran

Ahora la forma de crear usuarios sera asi, y tomando los criterios anteriores

```sql
CREATE USER 'alda'@'192.168.1.11' IDENTIFIED WITH sha256_password BY 'password' PASSWORD EXPIRE;
```

## Permisos de nivel de tablas y columnas

##### 1. Los usuario que NO sean privilegiados, no tendran acceso a la base de datos: information_schema ni a sus tablas

Ingresar la consulta:

```sql
REVOKE ALL ON information_schema.* FROM 'alda'@'192.168.1.11';
```

En caso de que no quieras revocar todos los permisos, en el ```REVOKE``` puedes indicar cuales, ejemplo

```sql
REVOKE SELECT
```
Aqui solo se le quitan los permisos de ```SELECT```.

Para aplicar cambios usa 

```sql
FLUSH PRIVILEGES;
```

Despues cambia a la base de datos e ingresa:

```sql
SHOW GRANTS FOR 'alda'@'192.168.1.11';
```

Y te debe de salir vacio

![](/assets/img/checklist/1.png)

## Politicas a nivel de red

##### 1. Se debe de habilitar SSL (Secure Socket Layer) para todas las conexiones a la base de datos

En el archivo ```my.cnf``` ingresa:

```sql
require_secure_transport=ON
```

##### 2. Se debe de usar ufw para autorizar que direcciones IP seran las permitidas para el acceso al servidor de base de datos

Primero debes de habilitar ufw:

```sudo systemctl start ufw```

Despues en una terminal ingresa:

```bash
sudo ufw allow from 192.168.1.11 to any port 3306
```

Con esto se le estaria indicando que la direccion IP ```192.168.1.11``` podra realizar una conexion hacia cualquier direccion IP por el puerto ```3306```

Si quieres que solo sea hacia una direccion especifica puedes usar:

```bash
sudo ufw allow from 192.168.1.11 to 192.168.1.111 port 3306
```

Esto hara que la direccion IP ```192.168.1.11``` solo podra realizar conexion a la direccion IP ```192.168.1.111``` por el puerto ```3306```

