# CodeIgniter Clase 2

## Configurar par�metros para la Base de datos

Hay un fichero de configuraci�n en application/config/database.php que contiene valores como username, password, nombre de la database, los que sirven como
par�metros de conexi�n con la base de datos.   

Disponemos de un array asociativo para almacenar los valores de conexi�n

## Configurar rutas

Como puede haber una relaci�n uno a uno entre el string de una URL y el controlador correspondiente clase/m�todo. El segmento en una URI sigue el siguiente patr�n:   
 
example.com/class/function/id/    
 
En ciertos casos, podr�a interesarte renunciar a esta relaci�n, y necesitar
invocar a otra funci�n, en lugar de la correspondiente a la URL.

Por ejemplo tus URLs tendr�n este prototipo.
 
example.com/product/1/   
example.com/product/2/   

Normalmente el segundo segmento de la URI est� reservado al nombre de funci�n
pero en este caso es un id de producto. CodeIgniter permite modificar manipulando el mapeo de la URI.   

Las reglas de enrutamiento est�n definidas en application/config/routes.php.   
En un array llamado $routes se asignan nuestro propios criterios de rutas.
Pueden usarse expresiones regulares y/o metacaracteres para asignar las rutas.

Ejemplo:   

$route['product/:num'] = "catalog/product_lookup";

La key contiene la URI a ser cotejada, y el valor contiene el destino, al cual
deber�a ser re-enrutado.

(:num) coteja un segmento conteniendo s�lo n�meros.   
(:any) coteja un segmento conteniendo cualquier caracter.

{:lang="php"}
