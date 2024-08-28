### Tags: `gpg2john` `asc` `ajp13` 

# Escaneo (*22,53,8009,8080*)

```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.183.55 -oN escaneo
```
![image](https://github.com/user-attachments/assets/054cc939-ec73-4b5f-a00d-673ae2bb7be7)

# Web port8080
En el Puerto 8009 no había NADA, y en el `8080` tenemos este **Tomcat** de versión `9.0.30`, buscamos un exploit en **GitHub**
Encontramos este exploit y lo utilizamos tal cual esta ahí  **-->** Lo que si es que tuvimos ayuda de **CHATGPT** porque el exploit estaba hecho en una **versión antigua de Python** entonces daba errores
Y vemos que al reproducir este exploit nos da una ``usuario:contraseña``.

![image](https://github.com/user-attachments/assets/ae1fd7ba-31f2-4a94-b691-5441b08bb8f1)
![image](https://github.com/user-attachments/assets/4c5eb029-9991-41fd-90e8-2b6788f19830)

# SSH
Nos conectamos por **SSH** ya que ese puerto estaba **OPEN** cuando hicimos el **NMAP**

```css
ssh skyfuck@10.10.183.55
```
![image](https://github.com/user-attachments/assets/6902362c-2e3c-4ccb-9509-097a1f302715)

# Escalada (**merlin**)
Lo que hicimos acá es que con **SCP** (-r para traernos todo el directorio) nos trajimos a nuestra máquina de atacante todo la carpeta de SKYFUNK porque ahí habia 2 archivos que nos llamaba la atención y que teníamos que hacerlo con herramientas.

![image](https://github.com/user-attachments/assets/594b6502-7e9d-424d-9ab9-7f577031c066)

## GPG2JOHN
Esta herramienta lo que hace es que cuando hay ``asc`` te lo hashea así después con **John The Ripper** lo desencripta

![image](https://github.com/user-attachments/assets/084374c7-6583-40f6-85d6-da23d742a71e)
![image](https://github.com/user-attachments/assets/96bc0bfd-3238-473b-b845-87b39d8d8566)

Y ahora había que poner estos 2 parámetros que era lo que dice en esta web ----->[Decrypt](https://superuser.com/questions/46461/decrypt-pgp-file-using-asc-key) , basicamente es para desencriptar un **GPG** y **ASC**
Vemos que abajo nos pide como una **Contraseña** y es la del hash con **JOHN**

![image](https://github.com/user-attachments/assets/4fc88e5e-8ff2-4f26-bb54-319fd52c3f16)

Una ves puesto la Contraseña, nos da otra Password que es la Merlin

![image](https://github.com/user-attachments/assets/596659bf-66db-411f-94c6-3c092c1d1fd3)
![image](https://github.com/user-attachments/assets/4f22bcdc-23db-47f3-ba5a-969cbac48531)

# Ecalada ROOT
Haciendo `sudo -l` vemos este `/zip`, Te vas a [GTFObins](https://gtfobins.github.io/)y copias y pegas y LISTO

![image](https://github.com/user-attachments/assets/363bbbc4-3c37-4542-8926-8d91b46f1d1e)
