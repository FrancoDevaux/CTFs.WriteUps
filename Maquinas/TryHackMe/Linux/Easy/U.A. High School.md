# Escaneo (22,80)
Lanzamos un nmap para descubrir puertos abiertos en la maquina
```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn 10.10.123.253
```
![image](https://github.com/user-attachments/assets/59ea6117-802f-48c6-905a-a74579ad2199)
![image](https://github.com/user-attachments/assets/771c3ab0-3d7b-4987-a3cc-8051882d1068)

# Web port80
Esta es la web
![image](https://github.com/user-attachments/assets/58385192-83ae-49da-ad5e-8c6f8d6b36ca)

En la web encontramos posibles usuraios para hacer **fuerza bruta** y conectranos por `SSH` (pero no es por ahí la via)
![image](https://github.com/user-attachments/assets/f9cbe102-7605-451a-b767-651f7c883c55)

# Gobuster
Utlizamos gobuster para enumerrar directorios y encontramos este con llamo la atencion `asssets`
```ruby
gobuster dir -u http://10.10.123.253  -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,html,php --no-error
```
![image](https://github.com/user-attachments/assets/4e10a5e0-fab2-44ef-87aa-a67fbad038ab)

Pero vemos que esta vacio por ende vamos a fuzear otra ves pero con el subdirectorio *assets*
![image](https://github.com/user-attachments/assets/282dcbf6-307a-46f5-9f9c-2759cedcf05f)

```ruby
gobuster dir -u http://10.10.123.253/assets  -t 100 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php --no-error
```
Encontramos un `index.php`
![image](https://github.com/user-attachments/assets/753d3a94-41f5-4576-aa26-6ec4defe33af)

Vemos que tenemos inyección de comandos con el parametro `cmd`
![image](https://github.com/user-attachments/assets/625391f5-4fed-4b1d-b379-11f632c460e7)

El texto de abajo esta en **base 64** por ende si lo decodificamos esto nos dice lo siguiente
![image](https://github.com/user-attachments/assets/55ba61a5-f545-4c83-b2de-32feac2fa75e)

Ahora le hacemo un `cat` al /etc/passwd, lo decodificamos y vemos este posible uusario `deku` (trate de listar su *id_rsa* pero no pude)
![image](https://github.com/user-attachments/assets/b6926774-e7c0-48b7-a698-525e12a5e69c)

### Reverse Shell
Ahora nos entablamos una conexxion con php de [Shell](https://www.revshells.com/)
![image](https://github.com/user-attachments/assets/8c00f4e9-0130-4e08-9502-c46610776468)
![image](https://github.com/user-attachments/assets/3c81d897-5985-4257-a899-86259dfa7e50)

# Escalada (*Deku*)
En el directorio `/var/www` vemos este otro directorio `Hidden_Content` que habia este archivo.txt
![image](https://github.com/user-attachments/assets/7b63816d-2a42-452c-b843-dacdd6e12296)

Lo decodificamos como hicimos anteriormente y dice esto:
![image](https://github.com/user-attachments/assets/9f6ee051-e491-400d-8ade-1dec1256ec5c)





