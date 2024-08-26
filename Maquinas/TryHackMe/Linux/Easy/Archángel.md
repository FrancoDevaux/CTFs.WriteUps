### Tags: `LFI` `PathHijacking` 

# Escaneo (*22,80*)
```css
 nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.60.214 -oN escaneo
```

# Web port80
Observamos que esta el puerto 80 abierto y en las web hay un *dominio*, entonces lo ponemos en `/etc/hosts`

![image](https://github.com/user-attachments/assets/97d385f4-7806-4aae-9364-820e1b24d8ee)

## Gobuster
Con gobuster encontramos este directorio `test.php` medio curioso
![image](https://github.com/user-attachments/assets/81fe8211-1d5c-4b27-99ba-d6eb8052548b)

Vemos que había en la URL un = entonces se podía tartar de un LFI y claramente era vulnerables a LFI ---> Probamos de todo y el que nos funcionó fue poner alternado `.././.././.././.././../`

![image](https://github.com/user-attachments/assets/717f14eb-bc03-43ce-875f-75afedb5b7fc)

### LFI --> access.log
Cuando podes hacer LFI una alternativa para ganar acceso a la máquina es ir al directorio **access.log** que se encuentra en `/var/log/apache2/`.
![image](https://github.com/user-attachments/assets/e3aa32f1-99c5-459f-8e37-ca8b6529107d)

Ahora lo que hacemos es hacer un CURL a la web para ver si se ve el comando que ejecutamos en la web y vemos que si
Entonces lo que hacemos es poner le comando este para poder después hacer una Reverse Shell y listo

![image](https://github.com/user-attachments/assets/fade08b6-2453-4503-a208-3bfce553fd09)

# Esacala (*archangel*)
Vemos que hay un archivo crontab que lo ejecuta el usurario archangel

![image](https://github.com/user-attachments/assets/8c797ff7-f5cb-4a7c-93e2-96bc02bc3963)

Como vi que TODOS tienen permisos para modificarlo lo que hice es poner otra ReverseShell
![image](https://github.com/user-attachments/assets/12651602-5f59-4c8b-9cb3-312d5e29340f)
![image](https://github.com/user-attachments/assets/950871f2-658a-495d-851a-6711a56fc014)

## Escalda ROOT
Vemos que hay un archivo **backup** y le haceos un `strings` y hay un comando cp que siempre eso lo tiene que ejecutar ROOT y por ende podes pensar en PATH Hijacking
![image](https://github.com/user-attachments/assets/6b6b1211-6769-4c70-ae04-706dcedaf699)

Ahora lo que hacemos es hacer un nano cp y dentro ponerle SUID a la bash
Le damos permisos 777 a cp
Hacemos el EXPORT (el punto es para indicar el actual directorio de trabajo)
Ejecutamos el ./backup y después hacemos un bash -p

![image](https://github.com/user-attachments/assets/94804e2f-323e-4a28-bd9c-9411bbf1e4e7)
![image](https://github.com/user-attachments/assets/1bd49c9d-2082-41dc-bec5-dd5593aa4887)

