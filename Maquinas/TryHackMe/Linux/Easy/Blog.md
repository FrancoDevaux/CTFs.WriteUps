### Tags: `Wordpress` `wpscan` `xmlrpc`

# Escaneo (*22,80,139,445*)
```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.90.146 -oN escaneo
```
![image](https://github.com/user-attachments/assets/a501777e-8bc4-4010-bb6f-477c1062b627)

# Web port80
Vemos que es una web de **WordPress** y tenemos 2 posibles usuarios

![image](https://github.com/user-attachments/assets/ea243af6-a9d4-4cd6-8c5c-4bc1915b94b8)

Tambien tenemos un dominio `blog.thm` que lo añadimos al `/etc/hosts`

![image](https://github.com/user-attachments/assets/066729d6-abfc-42bb-a63a-882112408bc9)

Como es un WordPress siempre hay un ``wp-login.php`` y vemos que si

![image](https://github.com/user-attachments/assets/0cd5cb36-b500-4b2e-82d8-6cbdbf2d8f12)

