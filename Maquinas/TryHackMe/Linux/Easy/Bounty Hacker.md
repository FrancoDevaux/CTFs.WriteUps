### Tags: `ftp` `anonymous` `hydra` `ssh`

# Escaneo (*21,22,80*)

```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.203.131 -oN escaneo
```
![image](https://github.com/user-attachments/assets/d09ba907-34cb-4c0a-8a63-b8018254253c)

## FTP 
Como podemos accder al `ftp` como **anonymous** lo vamos a hacer y con `mget *` nos tremos los 2 archivos.txt

![image](https://github.com/user-attachments/assets/4e6d0731-18a0-4bd3-a980-7f31cbf29b17)

El archivo `locks.txt` tiene muchas contraseñas 

![image](https://github.com/user-attachments/assets/6cfadc3a-7ee1-4db8-85c6-84bdcadffc26)

Y el archivo `task.txt` hay un posible usuario `lin`

![image](https://github.com/user-attachments/assets/5bbf2a9f-80b9-45a8-9564-cd922aaa3677)

## Hydra 
Como tenemos posibles contraseñas y un usuario, vamos a hacer **fuerza bruta** con hydra sobre el ``puerto 22``

![image](https://github.com/user-attachments/assets/f45fed9f-0ded-4d82-93bc-02f76f68ed0c)

### SSH
Nos conectamos por ssh con el usuario `lin:RedDr4gonSynd1cat3`

```css
  ssh lin@10.203.131
```
![image](https://github.com/user-attachments/assets/bbb68e49-c4ca-4582-8a82-b2c5a2c52f14)

# Escalada ROOT
Haciendo un ``sudo -l`` vemos que podemos ejecutar como root `/bin/tar` y lo buscamos en **GTFOBINS** ---> [Link](https://gtfobins.github.io/)

![image](https://github.com/user-attachments/assets/cd1c7906-dd3e-48e2-ae51-82d8a705139c)


