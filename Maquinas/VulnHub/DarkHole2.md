### Tags: `GIT` `sqli` `RPFW` `chisel`

# Reconocimiento (`Linux`)

Lo primero que hacemos es un **escaneo de equipos** que estén conectados ami red de esta forma:

```ruby
arp-scan -I eth0 --localnet
```

![image](https://github.com/user-attachments/assets/f23ce77f-173d-4f2d-87e2-673fc73fc6af)

# Escaneo(*22, 80*)

```css
sudo nmap -p- --open -sS --min-rate 4000 -vvv -n -Pn <IP>
```

![image](https://github.com/user-attachments/assets/cfd64dc4-3e99-4cb9-85c1-118f63cbe813)

```ruby
sudo nmap -p22,80 -sCV <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/4f3f8d85-2dd8-410a-99fc-a3eb63d95143)

# Port 80

Vemos que en la web hay un `panel de login` pero nosotros no tenemos credenciales validas, probamos con *Burp* **SQLinjections** pero nada.

![image](https://github.com/user-attachments/assets/ae7fb136-0827-4317-bbcf-86298c75ad94)

# .git/

Si nos acordamos cuando hicimos el escaneo teníamos un proyecto **.git**, si accedemos vemos que tenemos dictory listing

![image](https://github.com/user-attachments/assets/9d073ee5-0796-43df-8e95-cca7e23c1309)

Entonces lo que hacemos es traernos con wget de forma recursiva todo lo que haya dentro de ahí

```ruby
wget -r http://192.168.0.169/.git/
```

![image](https://github.com/user-attachments/assets/050292f4-1247-493c-8639-8129ba63b484)

Como es un proyecto de github lo primero que hacemos es mirar con `git log` commits que haya habido, movidas, etc

![image](https://github.com/user-attachments/assets/bd5ce776-8b12-4f23-98b9-c3be4cfe5d4a)

- Ahora hacemos un `git show a4d900a8d85e8938d3601f3cef113ee293028e10` para ver los cambios que hubo 
	- Vemos que tenemos **credenciales** en texto claro, la cual vamos a probar en el panel de login `lush@admin.com:321`

![image](https://github.com/user-attachments/assets/29c3b6eb-3285-404b-8962-efe1fa91b2dc)


# Dentro del Panel

Vemos que funcionaron las credenciales que encontramos 




















