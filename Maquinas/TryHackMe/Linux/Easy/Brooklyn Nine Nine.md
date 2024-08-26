### Tags: `ftp` `anonymous` `hydra`

# Escaneo (*21,22,80*)
```css
  sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn -oN escaneo
```
![image](https://github.com/user-attachments/assets/a4985c39-ce0e-4cd6-81d3-f4c331d91823)
![image](https://github.com/user-attachments/assets/210435e6-ff8e-4934-a3ed-3176b5f46d1c)

# FTP
Como vimos podemos accder como ``anonymous`` y es lo que vamos a hacer, una ves dentro con `get` nos traemos el `.txt`

![image](https://github.com/user-attachments/assets/3c927834-8f23-4bc9-a705-e6f7d5007717)

En la nota hay 2 posibles usuarios `amy` y `jake` 

![image](https://github.com/user-attachments/assets/3de86668-a5a7-4c47-a3e0-d9cc59a54120)

## Hydra (**jake**)
Con hydra obtuvimos la password de `jake` para conectranos por `ssh` ya que el puerto `22/open`

```ruby
 hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.51.68
````

![image](https://github.com/user-attachments/assets/b7b97fb8-1ea4-48bf-b6e0-5190d1c86dad)
![image](https://github.com/user-attachments/assets/4e7c48d7-ba50-4759-9c9f-f323c33c64d4)

# Escalada ROOT
Ejecutando `sudo -l` podemos ejecutar como root `/usr/bin/less`, nos vamos a [GTFOBins](https://gtfobins.github.io/) y listo

![image](https://github.com/user-attachments/assets/bf3e25ca-5160-450b-b968-479eac603455)
