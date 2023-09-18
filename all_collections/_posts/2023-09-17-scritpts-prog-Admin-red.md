---
layout: post
title: Scripts de programacion de la admin de red
categories: [ProgAdmin]
---


## Back-Up de un archivo

```bash
#!/usr/bin/env bash

archivoBackUp=$1
dirDestino=$2

if [ "$#" -lt 2 ];then
    echo -e "Debes de ingresar todos los argumentos\nUSO: ./tarea2.sh <archivo> <directorio_destino>"
else
    echo -e "$(cat "$archivoBackUp")\n""$USER" | tee "$dirDestino"/$(basename "$archivoBackUp")-$(date +%Y-%m-%d) 1> /dev/null
fi 
```
 
Resolucion usando ```cp```

```bash
#!/usr/bin/env bash

archivoBackUp=$1
dirDestino=$2
nombre=$(basename "$archivoBackUp")-$(date +%Y-%m-%d)

cp "$archivoBackUp" "$dirDestino"/"$nombre" && echo "$USER" >> "$dirDestino"/"$nombre"
```

## Contar que archivos de tal directorio contiene la palabra recibida por un argumento

```bash
#!/usr/bin/env bash
cadena=$1
directorio=$2

if [ "$#" -lt 2 ];then
	echo "te faltan argumentos"
else
    if [ -d "$directorio" ];then
		for archivo in $(ls "$directorio"); do
			if [ -f "$directorio/$archivo" ];then
				if cat "$directorio/$archivo" | grep -i "$cadena" &> /dev/null; then
					echo $(basename "$archivo")
				fi
			fi
		done
	fi
fi
```

Para evitar el uso de ```if``` anidados se puede usar ```inline if```

```bash
[ -d $directorio ] || { echo "no es un directorio"; exit 1; }
```

Adicionalmente se puede resolver de la siguiente forma:

```bash
grep $cadena $directorio &> /dev/null && echo $archivo
```


## Verificar si hay internet

```bash
#!/usr/bin/env bash

if curl http://google.com &> /dev/null; then
	exit 0
else
	exit 1
fi
```

Resolucion usando inline if

```bash
#!/usr/bin/env bash
curl http://www.google.com/ &> /dev/null && { echo "si"; exit 0; } || { echo "no"; exit 1; }
```

## Contar el numero de directorios dentro un directorio pasado por argumento

```bash
#!/usr/bin/env bash

dir=$1
count=0

[ "$#" -lt 1 ] && { echo "debes de ingresar un argumento"; exit 1; }
! [ -d "$dir" ] && { echo "debe ser un directorio"; exit 1; }

for dirs in $(ls -la "$dir"); do
	if [ -d "$dir/$dirs" ]; then
		let count=count+1
	fi
done
echo "El numero de directorios en "$dir" es: $count"

```

Una alternativa es usar ```find``` con ```-type d``` para indicar que sea un directorio

```bash
find /tmp -type d 2> /dev/null | wc -l
```