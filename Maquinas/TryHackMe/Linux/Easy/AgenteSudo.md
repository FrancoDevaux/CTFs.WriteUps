# Escaneo (**22,80**)
Comenzamos y hacemos un nmap para ver que puertos tiene abiertos
```css
 sudo nmap -p- 172.17.0.2 -sSVC --min-rate 5000 -n -Pn --open -oN escaneo.txt
```
![image](https://github.com/user-attachments/assets/4750c8a5-8f9d-47ef-82d9-159faf47c0ed)

# WEB ---> 80
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
Haciendo control+u para ver el codigo fuente, econtramos unas credeniale por el cual podemos probar para conectranos por SSH (ya que el puerto 22 estaba OPEN)
![image](https://github.com/user-attachments/assets/60c99a9c-dc9a-48c2-9675-562ed7faa355)
