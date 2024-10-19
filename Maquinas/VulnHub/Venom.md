### Tags:

# Reconocimiento (`Linux`)

Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/5c139b65-c56b-455f-8989-465ee29136b9)

# Escaneo (*21,80,139,445,443*)

```ruby
sudo nmap -p21,80,139,443,445 -sCV 192.168.0.198 -oN escaneo 
```

![image](https://github.com/user-attachments/assets/b87a8b3d-cb46-4bfb-8005-e825a3c1145c)

# Port 80

Poniendo la ip en el navegador obtenemos un **Default Apacha page** y si hacemos `Ctrl + u` encontramos esto:

```txt
<!...<5f2a66f947fa5690c26506f66bde5c23> follow this to get access on somewhere.....-->
```

Como vemos que era un **hash** fuimos a [CrackStation](https://crackstation.net/), pegamos el hash y nos lo descifro : `hostinger`

![image](https://github.com/user-attachments/assets/0ac32878-1a4d-4075-a0f4-fb42f616d136)

# Enum4linux

- Probamos en poner hostinger como ruta en la web pero no funcionó, entonces pensando descubrimos que era una contraseña para conectarnos pero nos faltaba enumerar usuarios 
	- Haciendo uso de `enum4linux <IP>`, nos encontró 2 usuarios `nathan y hostinger`

![image](https://github.com/user-attachments/assets/aaf435dd-384b-4416-9a8c-36f25704e82e)

## FTP

Nos conectamos por ftp **probando combinatorias** y funcionó la de `hostinger:hostinger` y nos trajimos un archivo `hint.txt`

![image](https://github.com/user-attachments/assets/49e9e622-80dc-40b4-9c08-50b76e2eb696)

El archivo  contiene esto:

```txt
        Hey there... 

T0D0 --

* You need to follow the 'hostinger' on WXpOU2FHSnRVbWhqYlZGblpHMXNibHBYTld4amJWVm5XVEpzZDJGSFZuaz0= also aHR0cHM6Ly9jcnlwdGlpLmNvbS9waXBlcy92aWdlbmVyZS1jaXBoZXI=
* some knowledge of cipher is required to decode the dora password..
* try on venom.box
password -- L7f9l8@J#p%Ue+Q1234 -> deocode this you will get the administrator password 


Have fun .. :)

```

Con [CyberChef](https://gchq.github.io/CyberChef/) obtuvimos como un tipo de cifrado `standard vigenere cipher` y este enlace `https://cryptii.com/pipes/vigenere-cipher`

- Lo que hacemos también es poner en él `/etc/hosts` a **venom.box** ya que el mensaje hace referencia a eso
	- Ahora ponemos `http:venom.box` y tenemos una web en la que hay un `panel de login`

![image](https://github.com/user-attachments/assets/f9f06ba4-791f-4643-b0b0-e6a26e70afd5)

Lo que hacemos es probar con el usuario `dora` que dice en el **hint.txt** y la contraseña descifrada en **Cyberchef**  `E7r9t8@Q#h%Hy+M1234`

![image](https://github.com/user-attachments/assets/edb851e4-4bc1-49f3-ad04-4eba38c97e77)

Vemos que funciono con éxito las credenciales `dora:E7r9t8@Q#h%Hy+M1234` ya que estamos dentro.

![image](https://github.com/user-attachments/assets/30fef925-4bdd-41b6-8342-f6404b41bcf3)

# Explotación

Lo que hacemos es apretar acá 

![image](https://github.com/user-attachments/assets/b35b9da6-03d0-47e0-9775-c2ec9f53ff31)

Y nos redirigió a `/panel`

![image](https://github.com/user-attachments/assets/9e68215a-3a84-4f36-afaf-ba2e5eee3bc0)

En la web encontramos es CMS Subrion con su versión `4.2.1`

![image](https://github.com/user-attachments/assets/9c91bfa0-21e0-42ed-9aac-f915ee2834b0)

- Investigando en Google descubrimos que podemos ejecutar código PHP en `/panel/uploads` a través de un archivo .pht o .phar, porque el archivo .htaccess los omite. 
	- Primero lo que hacemos es crearnos un ``script.phar`` de este enlace  [Link](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php) (cambiar la IP y puerto)
	- Una ves creado el `.phar` lo que hacemos es ir subirlo de esta manera:

![image](https://github.com/user-attachments/assets/48ad5194-faea-4b30-a1f1-2207c7bfdb81)

Antes del paso 6 ponernos en escucha con `netcat` y después apretar ahí en el Link del paso 6 y listo

![image](https://github.com/user-attachments/assets/8f82c1fc-15f1-4020-b887-8b4832b260ab)

Hacemos el tratamiento de la `TTY`

```ruby
script /dev/null -c bash
Ctrl + z
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 42 columns 189
```

# Escalada (``hostinger``)

Lo que hicimos es reutilizar la contraseña de **FTP** de ``hostinger:hostinger`` y funcionó

![image](https://github.com/user-attachments/assets/ea2255a5-0e1e-444c-831a-825aaab788a0)

## Escalada (``nathan``)

Navegando encontramos la credencial de `nathan:FzN+f2-rRaBgvALzj*Rk#_JJYfg8XfKhxqB82x_a` dentro de un directorio `/backup`

![image](https://github.com/user-attachments/assets/5d13d67d-cf86-4afd-9092-9da2a7f48b8f)


# Escalada ``ROOT``

La máquina tenia varias escalada para ser `root`, yo utilice la de buscar por privilegios SUID y aprovecharme `/usr/bin/find`, buscando en [GTFObins](https://gtfobins.github.io/gtfobins/find/#suid) copias el comando y sos !root!

![image](https://github.com/user-attachments/assets/b093626b-4126-44ae-9cb6-3a20634df914)
![image](https://github.com/user-attachments/assets/25f6251f-2e7b-47e7-82e8-6b05ce9433e3)








































