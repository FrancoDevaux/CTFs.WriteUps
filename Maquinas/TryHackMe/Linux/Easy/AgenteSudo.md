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
 ![image](https://github.com/user-attachments/assets/da70eff4-b0b7-4846-8890-730e61a6373b)
