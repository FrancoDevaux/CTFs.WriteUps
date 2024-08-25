# Escaneo (**22,80**)
Comenzamos y hacemos un nmap para ver que puertos tiene abiertos
```css
 sudo nmap -p- 172.17.0.2 -sSVC --min-rate 5000 -n -Pn --open -oN escaneo.txt
```
![image](https://github.com/user-attachments/assets/4750c8a5-8f9d-47ef-82d9-159faf47c0ed)

# WEB port80
-Observamos que en la web hay una imagen, por ende nos lo descragamos en nuestra maquina
![image](https://github.com/user-attachments/assets/166e12ef-2dc5-4ab8-bf41-4509ad8345f3)

## Exiftool, Steghide
 Haciendo uso de exiftool no encontramos nada pero con **steghide** vemos que nos da un archivo 'hint.txt'
 ```ruby
    steghide extract -sf [imagen.jpg]
```
 ![image](https://github.com/user-attachments/assets/da70eff4-b0b7-4846-8890-730e61a6373b)
 
Al darle un cat el **hint.txt**, vemos posibles rutas de directorios para poner en la web
![image](https://github.com/user-attachments/assets/4908ca15-7018-4ecc-a1df-5ecfca00a9d0)
![image](https://github.com/user-attachments/assets/421797de-a838-4606-ac16-abcd77a7d9de)

Haciendo `control+u` para ver el codigo fuente, econtramos unas credeniales por el cual podemos probar para conectranos por `SSH` (ya que el puerto 22 estaba *OPEN*)
![image](https://github.com/user-attachments/assets/60c99a9c-dc9a-48c2-9675-562ed7faa355)

## SSH
Probamos conectranos con las credenciales que econtramos y tuvimos éxito!
![image](https://github.com/user-attachments/assets/607ba1c3-8311-4555-a482-7ea7dfdef1cd)

# Escalada (*Alice*)
Vemos que haciendo `sudo -l`, tenemos este binario para accder como `rabbit` que esta dentro del **directorio Alice** en el cual tenemos **permisos de TODO**  y si analizamos ese script.py importaba la `librería random` y entonces lo que hicimos fue crear un archivo llamado **random.py y importábamos una bash**
![image](https://github.com/user-attachments/assets/e4bff7fb-39f2-4043-825f-40a2f6ec304f)
![image](https://github.com/user-attachments/assets/29af9849-2645-4ca0-81ee-3a2c5ab27fb9)

## Rabbit --> (Hatter)
Ahora como somos el user Rabbit vamos a tener que pivotar al usuario Hatter de este manera. Buscanco binarios SUID enconramos esto
```ruby
 find / -perm -4000 2>/dev/null
```
![image](https://github.com/user-attachments/assets/e7e44c4a-0bf5-4c46-85f6-5c939f472332)

Este binario nos lo `transferimos` a nuestra máquina atacante para verlo mejor 
![image](https://github.com/user-attachments/assets/d507aee0-61f7-4409-91b4-194345eba0a3)

Una ves transferido y poniendo strings `./teaParty` hemos observado que el comando **date** esta declarado haciendo algo, y lo que hicimos es ir a */tmp/* crear un archivo date y dentro poner **/bin/bash**, darle ejecución y listo
Y hacer un *Path Hijacking* para que lea primero lo que hay en tmp que seria nuestro archivo date creado anteriormente
![image](https://github.com/user-attachments/assets/f8e9d944-757e-4423-8f9d-a281be660135)

## Escalada ROOT
Para escalar root, buscando capabilities de la raiz encontramos este
```ruby
  getcap -r / 2>/dev/null
```
![image](https://github.com/user-attachments/assets/4c505e1e-b5e3-4ba6-b7a8-132ec75bafef)

Con la ayuda de [GTFsObins](https://gtfobins.github.io/) pivotamos al usuario `root`
![image](https://github.com/user-attachments/assets/9c030d5e-f0b4-4a18-a0a2-0075ffd863cd)
