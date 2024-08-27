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

