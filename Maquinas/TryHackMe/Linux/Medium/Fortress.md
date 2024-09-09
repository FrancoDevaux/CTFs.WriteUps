###Tags: `` `` ``

# Escaneo (*22,5581,5752,7331*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <ip> -oN escaneo
```
![image](https://github.com/user-attachments/assets/22e7607e-cb95-468b-a4d8-de6d863b9345)

-  En la descripción del CTF nos dice que agreguemos estos dos dominios al **/etc/hosts** `fortress y temple.fortress`

# FTP (*anonymous*)

Nos conectamos como el user **anonymous** sin proporcionar contraseña y nos traemos el ``.txt`` y un `.file`

```ruby
 ftp 10.10.11.86 -p 5581
```
![image](https://github.com/user-attachments/assets/9db41c68-2eed-4b8e-bb7b-2ab1c7d4e0a9)

Le hacemos un cat al archivo `marked.txt` y dice lo siguiente:

```
If youre reading this, then know you too have been marked by the overlords... Help memkdir /home/veekay/ftp I have been stuck inside this prison for days no light, no escape... Just darkness... Find the backdoor and retrieve the key to the map... Arghhh, theyre coming... HELLLPPPPPmkdir /home/veekay/ftp
```

Vemos una ruta `home/veekay/ftp`, un usuario `veekay`
Y el archivo `.file` es un es un archivo de bytes compilado en Python 2.7

![image](https://github.com/user-attachments/assets/0b4b41a8-189a-49d0-aad2-e26822e44e47)

### Descompilar 

Vamos a descompilar el archivo de esta manera usando `uncompyle2`, este repo lo que hace es descompilar el código de bytes de la versión 2.5, 2.6, 2.7 de Python
Pasos:

```ruby
https://github.com/BlueEffie/uncompyle2.git
cd uncompyle2
sudo python2 setup.py install
cd ..
uncompyle2 .file
```

![image](https://github.com/user-attachments/assets/f8ee0d00-43d5-4e8e-9f6e-899b7e872389)

Creé un pequeño script en Python que me permitiría decodificar las credenciales que espera el servicio. 

```python
from Crypto.Util.number import long_to_bytes 

usern = 232340432076717036154994 
passw = 10555160959732308261529999676324629831532648692669445488 

print(long_to_bytes(usern)) 
print(long_to_bytes(passw))
```

![image](https://github.com/user-attachments/assets/020333d9-94df-4190-90cb-603ade47ab34)

Ahora como vemos llama al puerto 5572 que nos podemos conectar por **telnet** poniendo estas credenciales 

```ruby
 telnet <ip> 5752
```

Observamos como una clave `t3mple_0f_y0ur_51n5`

![image](https://github.com/user-attachments/assets/ab1fe7d9-3197-42e8-a2a7-86d784d4cfdf)

