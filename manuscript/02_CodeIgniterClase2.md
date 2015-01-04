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

## Trabajando sobre el controlador

Cuando analizamos el nivel o capa del *controlador* salta a la vista que es aqu�
donde se enlazan las vistas y los modelos de nuestra aplicaci�n.    
Nos vamos a esforzar en encontrar convenciones para producir un autoload de nuestras vistas y modelos intentando obtener un mecanismo a�n m�s convencional.
As� tambi�n podemos ver la posibilidad de usar filtros con la idea de ejecutar
c�digo pre- y post- acci�n.

    Autoloading de vistas

Reflexionemos sobre esta posible convenci�n:

Las vistas deber�an ser puestas en una carpeta que sea llamada con el nombre del controlador y luego deber�a agregarse el nombre de la acci�n.    

Es decir, si usamos la URI /users/list tiene sentido cargar la vista en views/users/list.php. De hecho, puedes ya estar familiarizado con este modelo.

Con esta convenci�n en mente podemos reducir la necesidad de llamar a $this->load->view() y hacer que esta se cargue autom�ticamente despu�s que ejecuta el controlador. Esto nos llevar� a tener un controlador m�s sucinto y claro.

Vamos a necesitar una caracter�stica de CodeIgniter, se trata de la funci�n *_remap()*; es un m�todo que de existir en un controlador, ser� llamado por CodeIgniter en lugar de la acci�n del controlador. Con este mecanismo, el desarrollador puede ejecutar funcionalidades antes y despu�s del proceso de cada *acci�n*.    

{:lang="php"}
    <?
    class MY_Controller extends CI_Controller {
      public function __construct()
      {
        parent:__construc();
      }
    }
    ?>

Definimos nuestra funci�n _remap(), recibe dos par�metros, $method que es el nombre de la acci�n,
y $parameters que son los par�metros obtenidos de los distintos segmentos de la ruta URL los que son pasados al controlador de la acci�n.

{:lang="php"}
    <?
    public function _remap($method, $parameters) {
      call_user_func_array(array($this,
                  $method), $parameters);
    }
    ?>

    Autoloading de modelos

Podemos emplear una t�cnica bastante simple para seguir limpiando y poniendo m�s claro el c�digo del controlador, consistir�a en proveer un peque�a interfaz model-autoloading.

La idea es cargar modelo basado en convenciones, algunas de estas ya hemos usado.   
Recordemos: *singular_recurso_model.php* lo que ser�a para una tabla *users* user_model.php; mientras que para un modelo que manipula un ficheros deber�a ser file_model.php.

Adem�s sabemos que accedemos a nuestro modelo as�:   
$this->user->get_all();    
$this->file->upload();    

