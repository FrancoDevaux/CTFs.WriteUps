### Tags: `zip2john` `display` `binwalk` `steghide`

# Escaneo (*21,22,80*)
```css
  nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.74.7 -oN escaneo
```
![image](https://github.com/user-attachments/assets/dec7fa1a-bb07-44fc-94f7-b1fe60c49b54)

# Web port80
Acá lo que me está diciendo es que usando **BuerpSuite** y modificando el `User-Agent` poniendo Letras para saber cual era la inicial del agente que era la letra C ---> (es ir probando *A,B,C,D*,etc)

![image](https://github.com/user-attachments/assets/70c7541a-b402-4fc4-8e58-737dfb1d024e)
![image](https://github.com/user-attachments/assets/2290d37a-b27d-484d-a998-f9aaf54c64c9)

# Hydra (**FTP**)
Como tenemos un posible usuario `chris`, y como tenemos el *PUERTO 21 -OPEN* hicimos un ataque de fuerza bruta

![image](https://github.com/user-attachments/assets/776ff8a8-acf9-4c74-a879-3393bddab3ee)

Nos conectamos dentro del ftp : `ftp 10.10.74.7` y con `mget` nos traemos estos archivos 
![image](https://github.com/user-attachments/assets/ba7c1ecb-3dd8-4eb5-a7fe-d1856ff58fb5)

Le hacemos un `cat` al .txt y nos dice esto:
![image](https://github.com/user-attachments/assets/4bc69992-4f5f-4930-abff-8811eb7161ba)

Con el comando `display` podemos visualizar la foto
![image](https://github.com/user-attachments/assets/ef6692a6-bacf-4d8d-9157-d6eef12a2c9f)

## Exiftool / Binwalk
Examinamos las *imágenes* si había algo escondido y vimos que si pero con **exiftool** no pudimos verlo
Con **steghide** nos pedía una contraseña para el salvoconducto que no sabíamos por ahora 

Pero con `binwalk -e` que es otro parámetro pudimos extraer lo que había

![image](https://github.com/user-attachments/assets/e433828e-a54f-4b5e-8244-e462615fd353)

Lo descomprimimos con **7x** pero después también nos pedía una *contraseña*
Entonces probamos con **zip2john** para extraer el zip
![image](https://github.com/user-attachments/assets/bdd1f265-a140-4122-8e86-1b55bb95cb62)

Una ves que lo haya extraído vemos la contraseña para el **7x**
![image](https://github.com/user-attachments/assets/373a7d95-03d0-4a92-af5d-ae2984fe30ba)
![image](https://github.com/user-attachments/assets/34e45b97-3cf2-44c7-a54a-b80b308890a2)

Estaba en `base 64` por ende lo decodificamos y tenemos esta contraseña **Area51**

![image](https://github.com/user-attachments/assets/cd164f2f-3689-4769-a70f-4ceb04c0276f)

Pusimos Area51 en el salvaconducto que nos pedia con steghide y funcionó, obtuvimos la passwd de `james:hackerrules!` para conectranos por **SSH**
![image](https://github.com/user-attachments/assets/6eb5b915-9b4a-4916-b8bf-ed09ad59caaf)

# Escalada ROOT
Para ser root podíamos ejecutar `/bin/bash` sin proporcionar contraseña y accedemos como root

![image](https://github.com/user-attachments/assets/ba3f60e7-44cc-412c-976c-1a1d601946b4)

Comando:
```css
 sudo -u#-1 /bin/bash
```
![image](https://github.com/user-attachments/assets/129cc2ac-9a86-4ae4-8aa3-65eece580f92)

