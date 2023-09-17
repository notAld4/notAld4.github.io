---
title: Instalacion de mysql en Debian-based distros
tags: [mysql, AdminDB]
---

## Instalar paquetes necesarios

Antes de todo, ejecuta:

```bash
sudo apt update && sudo apt upgrade
```

#### wget
Para verificar si tienes instalado ```wget``` ejecuta esta comando:

```bash
wget --version &> /dev/null && echo "instalado" || echo "no instalado"
```

En caso de no tener ```wget``` instalado, ejecuta:

```bash
sudo apt install wget
```

#### libssl1

Verifica si en tu ``` /etc/apt/sources.list``` tienes este mirror: ```deb http://security.debian.org/debian-security bullseye-security main```, en caso de que no, agregalo

Para descargar ```libssl1``` ejecuta:

```bash
wget http://security.debian.org/debian-security/pool/updates/main/o/openssl/libssl1.1_1.1.1n-0+deb11u5_amd64.deb
```

En tu directorios actual debe de salir un archivo con el nombre ```libssl1.1_1.1.1n-0+deb11u5_amd64.deb```, para instalarlo, ejecuta:

```bash
sudo dpkg -i libssl1.1_1.1.1n-0+deb11u5_amd64.deb
```

## Instalacion de mysql

```bash
https://dev.mysql.com/get/mysql-apt-config_0.8.26-1_all.deb
```

En tu directorio actual debe de salir el ```mysql-apt-config_0.8.26-1_all.deb```, si es asi, ejecuta:

```bash
sudo dpkg -i mysql-apt-config_0.8.26-1_all.deb
```

Actualizar los repositorios con:

```bash
sudo apt update
```

Instala ```mysql server```:

```bash
sudo apt install mysql-server
```

Para verificar que le servicio este corriendo, ejecuta:

```bash
sudo systemctl status mysql
```

La salida debe ser:

- Active: active (running)
- Loaded: mysql.service; enabled

En caso de que no, ejecuta:

```bash
sudo systemctl start mysql && sudo systemctl enable mysql
```

Para terminar la instalacion realiza una instalacion de forma segura, esto te pedira la nueva contraseña de root y habilitar ```validate password plugin```, puedes activarlo o no:

```bash
mysql_secure_installation
```

Para corrobar que se instalo bien, ejecuta e ingresa la contraseña de root:

```bash
mysql -u root -p
```

