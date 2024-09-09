### Tags: `Tuenl_SSH` `Jenkins` `Wordpress`

# Escaneo (*22,80*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/fb9878c5-6de1-4953-a122-72d0ce817b93)

# Port 80

La página como vimos anteriormente es un *ubuntu default page* por lo que no había nada y por ende vamos a jugar con `gobuster` para encontrar *directorios* que haya

```css
gobuster dir -u http://IP/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html --no-error
```

Vemos que encontramos este directorio `/blog` y `/wordpress`, el cual encontramos un dominio `internal.thm` que vamos a agregar al **/etc/hosts**

![image](https://github.com/user-attachments/assets/f4db4b8f-37f7-4f24-8107-a29d052ae404)
![image](https://github.com/user-attachments/assets/143e6961-5ef0-4528-b50f-55bbced8af92)

Dentro de `/blog` tenes esto que no hay nada interesante

![image](https://github.com/user-attachments/assets/f469ca8d-a9e2-46df-bfe0-5cedcdad2fa1)

Y dentro de este directorio `/wordpress` tenemos un panel de login que te redirige a lo mismo que `/blog` si vas a login

![image](https://github.com/user-attachments/assets/3def58c4-fdfe-4799-b670-56b27b82a91b)

# Wpscan

Tenemos un posible usuario `admin` por esto:

![image](https://github.com/user-attachments/assets/3dd07698-24a5-45bb-8c4f-fa71b2bc68b6)

Por ende como tenemos *un posible usuario* vamos a ver si encontramos su *password*: `my2boys` y vemos que tuvimos **éxito**.
Comando: `wpscan --url http://internal.thm/worpress -U admin -P /usr/share/wordlist/rockyou.txt `

![image](https://github.com/user-attachments/assets/f2269bc9-28be-4f85-a110-0cce88d6201d)
![image](https://github.com/user-attachments/assets/c1a10734-792d-4713-9635-c069c700907f)

# Explotación

Para escalar privilegios en un wordpress se hace de esta forma 

![image](https://github.com/user-attachments/assets/ab787643-ae76-4dee-a651-89953421780c)
![image](https://github.com/user-attachments/assets/6be653a5-a10e-405a-86f2-9486b6a78959)
![image](https://github.com/user-attachments/assets/73432f08-ea0f-4eab-a1fa-219a2c761f01)
![image](https://github.com/user-attachments/assets/c66788c8-8ef4-42f8-a6c3-29f4350f5eb1)

# Escalada (`aubreanna`)

Dentro del directorio `opt` encontramos este archivo con credenciales `aubreanna:bubb13guM!@#123`

![image](https://github.com/user-attachments/assets/64ec5b41-61bf-4932-8245-dac15ae6b7ff)
![image](https://github.com/user-attachments/assets/597ca82c-ff6a-4e8a-8a4d-5a344f7af81e)

# Escalada `ROOT`

Viendo este `jenkins.txt` indica que un servicio interno de Jenkins se está ejecutando en una dirección IP diferente a la de nuestra máquina atacante.

![image](https://github.com/user-attachments/assets/e47db0be-76ea-4350-8d2f-29d5b85ce574)

### Túnel SSH

Vamos a intentar acceder al docker mediante un túnel ssh para reenviar la IP y el puerto de Jenkins desde la máquina de destino a la IP y el puerto de nuestra máquina atacante.
Comando: `ssh -L 8081:172.17.0.2:8080 aubreanna@internal.thm` y después pegar la  `passwd` (hacerlo en la maquina de atacante esto)
Para acceder a Jenkins, escriba localhost:[Número de puerto] en el navegador:

![image](https://github.com/user-attachments/assets/eea5778f-d224-45c5-a0a3-eb86a883f182)

![image](https://github.com/user-attachments/assets/9cba20af-9267-4a17-80ca-a30936c07c51)

#### HYDRA 
Lo hize con `hydra` porque no me funciono con Fuff
Comando: `hydra -l admin -P /usr/share/wordlists/rockyou.txt 127.0.0.1 -s 8081 -f http-post-form '/j_acegi_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password' -t 50`
**-s** es el puerto con el que estoy viendo la web que es el *8081* 

![image](https://github.com/user-attachments/assets/21bbd697-89a9-4c7e-876f-c8399678d785)

## Acceso

Tamos dentro del **Jenkins** y vamos a darnos una ReverseShell, primero vamos a ---> Manage Jenkins’ > ‘Tools and Actions,’ there is a ‘Script Console’
Nos vamos acá [Reverseshell](https://www.revshells.com/) y nos entablamos una shell en groovy

![image](https://github.com/user-attachments/assets/024d39f2-e118-4f26-9014-a47827ed2519)
![image](https://github.com/user-attachments/assets/3cdf6222-ea00-4a51-8f8b-c4ce110b5eb4)
![image](https://github.com/user-attachments/assets/fa434209-9c2a-49d6-9d71-b3a1befeb75d)

Dentro de **/opt** estaba esta nota con la contraseña de root
Nota: `root:tr0ub13guM!@#123`

![image](https://github.com/user-attachments/assets/46a58b77-cc6f-4d9d-acce-90e6fe9dfd0b)

Ahora la contraseña que encontramos la ponemos en la máquina en la que habíamos pivotado al user `aubreanna`

![image](https://github.com/user-attachments/assets/27eb02b4-9f6f-4878-ad3d-501b53042af5)
