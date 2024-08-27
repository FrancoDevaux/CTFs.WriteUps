### Tags: `FuelCMS` 

# Escaneo (*80*)

```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn 10.10.192.212 -oN escaneo
```
![image](https://github.com/user-attachments/assets/e5486701-3df4-4fca-9683-f91448ebb4d5)

# Web port80
En la web como descubrimos anteriormente el directorio `fuel` y encontramos un panel de **login**

![image](https://github.com/user-attachments/assets/89fe7eb4-47be-4cae-bb82-65aba4cc9bb3)

En google buscamos **fuel cms default credentials**, obtuvimos `admin:admin` y funcionó

# Dentro (**FuelCMS**)
Una ves que estábamos dentro de la web buscamos una forma de acceder para explotarlo y ganar acceso a la máquina
Obtuvimos la version que era la `Fuel cms 1.4` la cual buscamos un exploit en github [Exploit](https://github.com/padsalatushal/CVE-2018-16763)

```ruby
  git clone https://github.com/Trushal2004/CVE-2018-16763.git
  cd CVE-2018-16763/
  python3 -m pip install -r requirements.txt
  chmod +x exploit.py
  ./exploit.py
```
Vemos que habíamos entrado de dentro de la máquina pero queríamos una **shell interactiva**. Entonces lo que hicimos fue crear un archivo **llamado ``shell.php``** y **subirlo** dentro de la web y después **buscarlo y conectarnos** ( lo hicimos de esta manera porque no pudimos conectarnos de otra forma ya que **ninguna reverse shell funcionaba**)

![image](https://github.com/user-attachments/assets/a3f0ed0b-7ba5-461f-8c6f-1f16853645f4)

El archivo `shell.php` como vimos se subio en `/var/www/html`, por ende primero nos poenmos en escucha

```css
  nc -nlvp 443
```

Ahora vamos al firefox y buscamos por el el archivo **shell.php**

![image](https://github.com/user-attachments/assets/e87f4b67-b541-4422-aa5e-73df7fad5c06)
![image](https://github.com/user-attachments/assets/3b136eaa-9306-46dd-9d4b-62b41da4f8a8)

# Escalada ROOT
Buscando por google donde se almacenaban la bases de datos de configuracion de un FuelCMS y era en este directorio
```css
  /var/www/html/fuel/application/config/
```

![image](https://github.com/user-attachments/assets/3affe8da-905b-43c1-9f2e-371335662b4d)
![image](https://github.com/user-attachments/assets/612d44eb-574c-49e1-85a8-802ac10e94be)




