# Escaneo (22,80)
Lanzamos un nmap para descubrir puertos abiertos y los servicios que corren en la maquina
```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.123.253
```
![image](https://github.com/user-attachments/assets/59ea6117-802f-48c6-905a-a74579ad2199)
![image](https://github.com/user-attachments/assets/771c3ab0-3d7b-4987-a3cc-8051882d1068)

# Web port80
Esta es la web
![image](https://github.com/user-attachments/assets/58385192-83ae-49da-ad5e-8c6f8d6b36ca)

En la web encontramos posibles usuraios para hacer **fuerza bruta** y conectranos por `SSH` (pero no es por ahí la via)
![image](https://github.com/user-attachments/assets/f9cbe102-7605-451a-b767-651f7c883c55)

# Gobuster
Utlizamos gobuster para enumerar directorios y encontramos este que nos llamó la atención `assets`
```ruby
gobuster dir -u http://10.10.123.253  -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,html,php --no-error
```
![image](https://github.com/user-attachments/assets/4e10a5e0-fab2-44ef-87aa-a67fbad038ab)

Pero vemos que esta vacío por ende vamos a fuzear otra ves pero con el subdirectorio *assets*
![image](https://github.com/user-attachments/assets/282dcbf6-307a-46f5-9f9c-2759cedcf05f)

```ruby
gobuster dir -u http://10.10.123.253/assets  -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php --no-error
```
Encontramos un `index.php`

![image](https://github.com/user-attachments/assets/753d3a94-41f5-4576-aa26-6ec4defe33af)

Vemos que tenemos inyección de comandos con el parámetro `cmd`

![image](https://github.com/user-attachments/assets/625391f5-4fed-4b1d-b379-11f632c460e7)

El texto de abajo está en **base 64** por ende si lo decodificamos esto nos dice lo siguiente
![image](https://github.com/user-attachments/assets/55ba61a5-f545-4c83-b2de-32feac2fa75e)

Ahora le hacemo un `cat` al /etc/passwd, lo decodificamos y vemos este posible usuario `deku` (trate de listar su *id_rsa* pero no pude)
![image](https://github.com/user-attachments/assets/b6926774-e7c0-48b7-a698-525e12a5e69c)

### Reverse Shell
Ahora nos entablamos una shell con php de [Shell](https://www.revshells.com/)
![image](https://github.com/user-attachments/assets/8c00f4e9-0130-4e08-9502-c46610776468)
![image](https://github.com/user-attachments/assets/3c81d897-5985-4257-a899-86259dfa7e50)

# Escalada (*Deku*)
En el directorio `/var/www` vemos este otro directorio `Hidden_Content` que había este archivo.txt
![image](https://github.com/user-attachments/assets/7b63816d-2a42-452c-b843-dacdd6e12296)

Lo decodificamos como hicimos anteriormente y dice esto:
![image](https://github.com/user-attachments/assets/9f6ee051-e491-400d-8ade-1dec1256ec5c)

Ahora encontramos este archivo **.jpg** la cual es curioso y por lo tanto lo traemos a nustra máquina de atacante
![image](https://github.com/user-attachments/assets/6aecd4dc-f79b-4947-ab8f-5bb01b318e62)

Ahora jugamos con `steghide`
```ruby
steghide extract -sf oneforall.jpg
```
y cuando nos pida el salvaconducto ponemos esta `AllmightForEver!!!` que conseguimos antes
![image](https://github.com/user-attachments/assets/d12a8664-e68e-47d2-8ab1-2eba665ff76a)
![image](https://github.com/user-attachments/assets/e967c76d-10d4-4a47-bf88-98fe3b2ee54b)

Obtuvimos un archivo llamado **creds.txt** la cual contiene la contraseña de `deku`
![image](https://github.com/user-attachments/assets/c060f667-a047-4eb8-8a9c-62340046f921)
![image](https://github.com/user-attachments/assets/6160432e-e6ad-4797-a5d5-250c3f86eeda)

# Escalada ROOT
Vemos que haciendo `sudo -l` podemos ejecutar ese script.sh como root
![image](https://github.com/user-attachments/assets/90417cbe-f9a0-4c9a-a9ea-9d276169c331)

Haciendo un `ls -la` al .sh observamos que somos los propietarios de ese archivo

![image](https://github.com/user-attachments/assets/0ac46515-3c60-49bf-a003-38b91276f28d)

 Agregamos el usuario deku al archivo sudoers ejecutando este comando `sudo ./feedback.sh` y agregamos esto:
 
![image](https://github.com/user-attachments/assets/3dc3dc47-c3f5-4029-bbac-aff745efcded)

Entonces ahora como establecimos NOPASSWD en ALL, en la terminal hacemos un `sudo /bin/bash` y listo 

![image](https://github.com/user-attachments/assets/76a56c8a-6512-42be-ad68-bf82739aab2b)




