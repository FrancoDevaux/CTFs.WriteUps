### Tags: `perl` 

# Escaneo (*22,80*)

```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.16.65 -oN escaneo
```
![image](https://github.com/user-attachments/assets/619b8370-f689-4f49-8d05-0b66da8ba85d)

# Web port80
Visualizamos la web que contiene y no vemos nada, haciendo `Ctrl + u` vemos un nombre de usuario `R1ckRul3s`

![image](https://github.com/user-attachments/assets/763275f1-02de-4a5e-a513-3e011486737b)

## Gobuster 
Utlizando gobuster encontramos un directorio llamado **login.php** y **robots.txt**

![image](https://github.com/user-attachments/assets/a60a6aba-8fc1-4337-89b7-4c13b24b1edf)
![image](https://github.com/user-attachments/assets/7a513598-58f9-4055-86fd-d194e2ac0889)

Accedimos con la contraseña de `robots.txt` y el user que contramos `R1ckRul3s`
Como vemos dentro de la web tenemos un panel que si ejecutamos comandos nos lista.

![image](https://github.com/user-attachments/assets/2876ea0a-25ad-4741-9a17-0fb6a1702d41)

Como el comando *CAT* no funcionaba podes probar con ``TAC`` y funciona. (flag de user)

![image](https://github.com/user-attachments/assets/918b7a75-68ab-4cab-ac5f-4be4a6e48624)

Para que sea más cómodo y podes tener una terminal en vez de hacerlo por la web lo que podes hacer es un **reverse shell** --> Lo hicimos con **perl** porque los otros no funcionaban.

![image](https://github.com/user-attachments/assets/17921c5c-7dc5-4588-a761-700c10603c44)

Nos ponemos en escucha y ya esta.
![image](https://github.com/user-attachments/assets/14cc0c2b-c58e-49d1-8cbb-e45d5986d62e)

# Escalada ROOT
Vemos con **sudo -l** no necesitamos poner contraseña para ser root ----> Entonces lo que haces es poner **sudo bash** y listo o también **sudo root** y LISTO.

![image](https://github.com/user-attachments/assets/46a1a2b1-acdc-4b0b-b1dc-555bc739ee7d)
![image](https://github.com/user-attachments/assets/2e640f9d-4f8d-4921-a9d8-b2f4f771a821)
