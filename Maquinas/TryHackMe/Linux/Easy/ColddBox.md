### Tags: `wordpress` `wpscan` 

# Escaneo (*80, 4512*)
```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.160.226 -oN escaneo
```
![image](https://github.com/user-attachments/assets/5127edd1-af64-4d93-8831-f415f863a771)

# Web port80
En la web tenemos un `wordpress` y también lo podemos visualizar con `wapalyzer` y que utiliza `php`

![image](https://github.com/user-attachments/assets/2c7af5f4-485f-4c57-9b70-67f4a5b2a7bb)

Lanzandole un **script de enumeracion** al puerto 80 con ``nmap`` obtuvimos este directorio `/hidden` muy curioso

![image](https://github.com/user-attachments/assets/09085503-fc16-4df2-bc72-874c2e44de32)

Dentro de ese directorio en la web obtuvimos 3 posibles usuarios `c0ldd`,`hugo`,`philip`

![image](https://github.com/user-attachments/assets/1baa9318-f2a1-4625-b7c2-726b109d3a9b)

# Wpscan (**c0ldd**)
Con la herramienta `wpscan` obtuvimos la contraseña de `c0ldd` para accder al `wp-login.php` del wordpress

```ruby
  wpscan --url http://10.10.160.226/ -U c0ldd -P /usr/share/wordlists/rockyou.txt
```
![image](https://github.com/user-attachments/assets/36530c38-3350-41e1-b333-6909d4e30d24)

### Dentro (**wordpress**)
Una ves que accedimos al wordpress lo que tenemos que hacer es ir a `appearance` > `editor` > `404 Template`

![image](https://github.com/user-attachments/assets/d85edbca-f9ce-4406-a4a6-3c3f4f43e8f9)
![image](https://github.com/user-attachments/assets/d241a95e-971e-49eb-9887-41aba65a8d2d)
![image](https://github.com/user-attachments/assets/32d835cd-c285-4b9c-9120-d7081525e8ba)

Ahora nos copiamos con este script en **php** de [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![image](https://github.com/user-attachments/assets/ef7dd5ec-505a-4c0c-98b6-3fd637a28c42)

Una subido en la *terminal* nos ponemos en escucha por el puerto especificado antes
```css
  nc -nlvp 443
```

Y por último vamos al firefox y ponemos esto y nos da la `reverseShell`

![image](https://github.com/user-attachments/assets/8803b355-ed59-495b-aa52-a524453e66c6)

# Escalada ROOT
Buscando binarios `suid` encontramos este que nos llamó la atención `/usr/bin/find`

```ruby
  find / -perm -4000 2>/dev/null
```
![image](https://github.com/user-attachments/assets/bd51d6ed-4884-416c-808d-cde0812e7b58)

Este binario lo buscamos en [GTFObins](https://gtfobins.github.io/) y listo

![image](https://github.com/user-attachments/assets/c5d0aea8-a0ce-4061-9113-a87546ab803d)


