# CodeIgniter Clase 1

Se espera que antes de comenzar la lectura de estas clases, tomes un contacto con el sitio [CodeIgniter](http://www.codeigniter.com/user_guide/toc.html)
y dediques un tiempo a seguir algunos de los tutoriales obtendr�s as� una r�apida idea acerca como trabaja el framework, y podr�s ponerte en marcha, una vez que hayas realizado el dowloading del c�digo
construyendo una aplicaci�n introductoria de las que vienen en la gu�a del usuario CodeIgniter. Apenas comiences a trabajar con CodeIgniter ver�s a primera vista, la muy comentada
organizaci�n del c�digo de tu aplicaci�n en tres partes: el *modelo*, *las vistas*, y el *controlador*; con esa previa ejercitaci�n por tu parte, podemos ahora continuar.

## Directos al patr�n MVC

En esta clase nos centramos en la M de MVC, vemos que podr�a ser lo mejor en el
modelo e intentaremos aprender el poder de este nivel y lo que deber�a ser; incluso
podemos inspirarnos en los patrones de Rails, y obtener algunas conclusiones acerca
de como las cosas pueden ser m�s eficientes.

Dos conceptos que provienen del mundo Rails:   

* Modelo grueso, controlador ligero. 
* Convenci�n sobre configuraci�n.

Nos dicen que necesitamos escribir las aplicaciones alrededor de la l�gica de
nuestros modelos, y de aqu� viene esto de... "cualquier c�digo que no puedas justificar
ponerlo en alguna parte deber�a estar en el modelo".

El patr�n de dise�o MVC, nos ense�a una serie de reglas para construir aplicaciones
m�s robustas y estructuradas.    
Lo m�s importante de �stas es que el modelo contiene todo el c�digo relacionado al proceso de datos.    
Nuestros modelos controlan los cambios y el estado de los datos.

Luego la _M_ de MVC significa cualquier c�digo que almacena datos en la BD, que valide datos,
que env�a emails, que conecta con APIs, que calcula estad�sticas, etc.

La pregunta que surge: �Hay un Modelo estandard?

Intentando una contestaci�n, bien podemos decir que detr�s de cualquier aplicaci�n hay un conjunto de convenciones.    
Los desarrolladores s�lo especifican aspectos no convencionales de la aplicaci�n.   
El resultado es que el desarrollador deba tomar menos decisiones.

Como desarrolladores podemos abstraernos de varias funcionalidades, y sabemos que
conseguiremos una aplicaci�n consistente en diferentes entornos.

En estas convenciones se asumen cuestiones bastantes razonables, atendamos a algunas de ellas:

1. Cada modelo mapea a una tabla de la base de datos. 
2. El nombre del modelo es singular, el nombre de la tabla plural.
3. Cada tabla contien una columna id.
4. Cada tabla contine dos columnas una created_at y otra updated_at.

## Sin patr�n ni framework

Antes de continuar con el desarrollo de estas convenciones, tratemos de analizar una codificaci�n particular donde no usamos framework alguno y tampoco el patr�n
MVC, para estructurar el c�digo.

{:lang="php"}
    <html>
     <head>
      <title>Test Page</title>
     </head>
     <body>
      <?php
        $serverip="localhost";
        $username="user1";
        $password="xxxxx";
        $database="rebbin";
        $con=mysql_connect($serverip,$username,$password)
	  or die("no puedo conectar con mysql server");
        @mysql_select_db($database) or die("no selecci�n de DB");
        $sql="select id from pastes order by created_on desc";
        $result=mysql_query($sql) or die("Problema con pastes");
        if(!mysql_num_rows($result)) {
          echo "<p>No hay pastes en la BD</p>";
        }
        else {
          while($row=mysql_fetch_array($result)){
            print_r($row);
          }
        }
      ?>
     </body>
    </html>

## Aproximaci�n usando CI

{:lang="php"}
    <?
     class Pastes_model Extends CI_Model {
      public function __construct()
      {
       $this->load->database();
      }
       
      function getpastes()
      {
       $this->db->select('id')->from('pastes')
                              ->order_by('id','desc');
       $query=$this->db->get();
       return $query->result_array();
      }
     }
    ?>

Observamos que el c�digo es OO y hace uso de herencia, �ste extiende una clase del
framework, el nombre que usamos es *Pastes_model*; el nombre de la tabla de la BD en
el backend es *pastes*.

Para el caso que nos ocupa basta definir un m�todo getpastes().
Varias preguntas caben aqu�.

1. �Sobre que BD est� trabajando nuestro c�digo?.
2. �Cu�l es el nombre del fichero que contiene esta clase?.
3. �Es todo lo necesario?.

La Base de datos y los par�metros usados para la conexi�n, no se ven aqu�; estos
son escritos en un fichero de configuraci�n.

El nombre de fichero que contiene la clase Pastes_model es pastes.php en la carpeta application/models

No es todo lo necesario; debe instanciarse un objeto de esta clase, en el momento de procesar la petici�n HTTP cliente, para esto se codifica un controlador, el
que llamamos pastes_controller.php y salvamos en application/controllers, veamos
su muy sencillo c�digo.

{:lang="php"}
    <?
     class Pastes Extends CI_Controller {
      function index() {
        $this->load->model('pastes_model');
        $data['results']=$this
                         ->pastes_model
                         ->getpastes();
        $this->load->view('pastes/home',$data);
      }
     }
    ?>

Observamos como el resultado de la funci�n del modelo invocada en el
controlador es asignado a una key *results* del array asociativo $data;
array �ste al que podr� acceder la vista una vez sea cargada por el controlador.

Podemos decir que el controlador de nuestra applicaci�n es una clase derivada
de una clase del framework llamada CI_Controller y tiene como objeto cargar el
los modelos y las vistas de la aplicaci�n.

Faltar�a ver el c�digo de la vista, lo tenemos en application/views/pastes/home.php:

{:lang="php"}
    <html>
    <head>
      <title>Test Page</title>
    </head>
    <body>
    <?
    if(count($results)==0) {
     echo "<p>No hay pastes en la database</p>";
    }
    else {
      foreach($results as $row){
        foreach($row as $k => $v){
          echo $k . " => " . $v . "<br>";
        }
      }
    }
    ?>
    </body>
    </html>

Ahora podemos retomar los tres puntos enumerados m�s arriba, y tratamos de ejemplificar nuevamente, comencemos por el primero de los puntos:

    �Sobre que BD est� trabajando nuestro c�digo?

Para la mayor�a de los casos cada modelo interact�a con una �nica tabla de la base de datos; y tenemos algunas acciones que podr�amos llamar
*acciones est�ndares* de interacci�n con esa tabla, son conocidos como m�todos
CRUD (Create, Read, Update y Destroy).    
Entonces podr�amos especificar la tabla de la base de datos a trav�s de la
clase, tomemos a la tabla *users*, los m�todos del modelo podr�an estar codificados as�:

{:lang="php"}
    public function get($where) {
      return $this->db->where($where)
                  ->get('users')->row();
    }
    public function get_all($where) {
      return $this->db->where($where)
                  ->get('users')->result();
    }
    public function insert($user) {
      return $this->db->insert('users',$user);
    }
    public function update($where, $user) {
      return $this->db->where($where)
                  ->update('users', $user);
    }
    public function delete($where) {
      return $this->db->where($where)
                  ->delete('users');
    }

    �Cu�l es el nombre del fichero que contiene esta clase?

Bien podr�a ser models/user.php.
Podemos llamar a la clase *User*, en realidad no estamos obligados a agregar *_model* al nombre al nombre. Mientras que la tabla de la BD sobre la cual trabaja tiene el mismo nombre de la clase en plural, *users*

    �Es todo lo necesario?.    

No, necesitaremos de c�digo PHP en controladores y en vistas.

Veamos la siguiente definici�n lo que paara CI es un modelo:

{:lang="php"}
    <?
    class Pastes_model Extends CI_Model {
      public function __construct()
      {
        $this->load->database();
      }
      public function get_pastes() 
      {
        $query = $this->db->get('pastes');
        return $query->result_array();
      }
    }
    ?>

Con este modelo y una vez realizada la petici�n a la URL:
http://example.com/index.php/pastes
obtenemos una respuesta como la que se muestra abajo. �Cu�les son las piezas de c�digo que han intervenido?

    Array
    (
        [0] => Array
            (
                [id] => 1
                [author] => paulin
                [language] => Perl
                [description] => lee de file
                [body] => #!/usr/bin/perl -w
                          while(<>) {
                           print $_;
                          }
                [created_on] => 2007-12-25 13:49:14
            )
     
        [1] => Array
            (
                [id] => 2
                [author] => paulin
                [language] => Perl
                [description] => Lee desde un fichero
                [body] => #/usr/bin/perl -w
                          while(<>){
                            $i++;
                            print $i,":",$_;
                          }
                [created_on] => 2007-12-25 13:55:12
            )
      
        [2] => Array
            (
            ...
            ...
 
Deber�amos tener un *controlador* como el siguiente,    

{:lang="php"}
    <?
    class Pastes Extends CI_Controller{
      function index(){
        $this->load->model('pastes_model');
        $data['results']=$this
                  ->pastes_model->get_pastes();
        echo "<pre>";
        print_r($data['results']);
        echo "</pre>";
      }
    }
    ?>

Como vemos hacemos que desde el controlador se env�e la respuesta, evitando c�digo en una vista, lo que en el primer ejemplo fue el uso de views/pastes/home.html

Proseguimos cambiando m�nimamente el modelo, la finalidad es demostrar que la
l�gica de negocio est� situada en el modelo, aunque en este caso nuestra propuesta es 
una selecci�n m�s restringida de los datos existentes en la BD, debemos dirigirnos al modelo para acometerla.

{:lang="php"}
    <?
    class Pastes_model Extends CI_Model {
      public function __construct()
      {
        $this->load->database();
      }
      public function get_pastes($num=4, $start=0) 
      {
        $this->db->select('description,language')
           ->from('pastes')->where('language','Ruby')
           ->order_by('created_on','desc')->limit($num,$start);
        $query=$this->db->get();
        return $query->result_array();
      }
    }
    ?>

La respuesta obtenida es:

    Array
    (
        [0] => Array
            (
                [description] => Esto funciona
                [language] => Ruby
            )
    
         [1] => Array
            (
                [description] => Esto funciona
                [language] => Ruby
            )
       
         [2] => Array
            (
                [description] => Equivale al acostumbrado modo ... 
                [language] => Ruby
            )
      
         [3] => Array
            (
                [description] => Seguro convence
                [language] => Ruby
            )
    )

## Un paso m�s, codificamos MY_Model

Nuestro siguiente trabajo ser� intentar *generalizar* las clases del modelo, de
tal manera que el desarrollador s�lo deba pensar en definir una clase sencillamente as�:

{:lang="php"}
    <?
     class Paste_model extends MY_Model
     {
      //protected $_table = 'pastes';
     }
    ?>

Para esta propuesta, escribiremos MY_Model en application/core:

{:lang="php"}
    <?
    class MY_Model extends CI_Model
    {
      public function __construct()
      {
        parent::__construct();
        $this->load->database();
        $this->load->helper('inflector');
        if ( ! isset($this->_table)) {
          $this->_table = 
          strtolower(plural(str_replace('_model',
                     '', get_class($this))));
        }
      }
      public function get($where)
      {
        return $this->db->where($where)
               ->get($this->_table)->row();
      }
      public function get_all($where)
      {
        $pp=array();
        $pp['language']=$where;
        $pp['author']='paulinohuerta';
        return $this->db->where($pp)
                        ->get($this->_table)
                        ->result();
      }
    ?>

Y en el c�digo del controlador tendr�amos:

{:lang="php"}
    <?
    class Paste Extends CI_Controller {
      function index() {
         $this->load->model('paste_model');
         $data['results']=$this
	        ->paste_model->get_all('Ruby');
	 $this->load->view('paste/home',$data);
      }
    }
    ?>

Observamos que el nombre de la clase es en singular siendo as� tanto en
el modelo como en el controlador, y ya sabemos que el modelo trabaja sobre la
tabla en plural, en este caso la *tabla pastes* de la BD.   
Hemos agregado en el modelo la funci�n *get* aunque en nuestro ejemplo no la
usamos, estar�a destinada a conseguir una �nica fila de la tabla, es decir, 
la usaremos cuando esperamos una sola fila como resultado de la consulta.   
El controlador *invoca get_all* enviando _Ruby_' como par�metro, esto es s�lo
a los fines de demostraci�n, lo oportuno ser�a que ese par�metro se corresponda
con algunos de los datos proporcionados por el usuario en la petici�n o bien
que ha sido generado en otro momento en el flujo del procesamiento.    
Tambi�n vemos como en el modelo se crea un array asociativo usando como clave
el nombre de un par de columnas de la tabla, en este caso *author* y *language*, es debido a que el par�metro de la _funci�n where_ debe ser dado como array.

    Damos otra vuelta de tuerca en MY_Model

{:lang="php"}
    <?
    class MY_Model extends CI_Model
    {
      public function __construct()
      {
        // se omite codificaci�n
      }

      public function get_all()
      {
        $args = func_get_args();
        if (count($args) > 1 || is_array($args[0]))
         {
           $this->db->where($args[0]);
         }
        else
         {
           $this->db->where('id', $args[0]);
         }
        return $this->db->get($this->_table)
                        ->result();
      }
      // Resto de funciones CRUD
    }
    ?>

En el controlador preparamos un array para ser aplicado en la cl�usula where de la selecci�n.    

{:lang="php"}
    <?
    class Paste Extends CI_Controller{
      function index(){
        $pp=array('language' => 'Ruby',
          'author' => 'paulinohuerta', 'id >' => 9);
        $this->load->model('paste_model');
        $data['results']=$this->paste_model
                         ->get_all($pp);
        $this->load->view('paste/home',$data);
      }
    }
    ?>

Y voil�, hemos conseguido tener un modelo muy sencillo, por otra parte hemos 
de alguna manera, descifrado el sentido com�n del patr�n MVC, como las ventajas del
tan conocido principio DRY (Don't Repeat Yourself).
Y acerca del patr�n MVC tenemos las idea m�s clara, como para saber *donde* acudir para modificar o conseguir una nueva funcionalidad de la aplicaci�n

Por �ltimo hay que notar que hasta ahora no hemos visto en ninguna parte del c�digo, hacer referencia a la BD 
sobre la cual trabaja la aplicaci�n, con esto y otros simples detalles de configuraci�n que afecta a las aplicaciones usando CodeIgniter comenzaremos nuestra
segunda clase.

Como �ltima cuesti�n podemos probar de hacer inserciones en la BD, para ello retomemos el m�todo *insert* mostrado anteriormente:

{:lang="php"}
    public function insert($user) {
      return $this->db->insert('users',$user);
    }

Podr�amos usar nuevamente MY_Model, esta vez con el objetivo que el m�todo insert devuelva el *id* de la fila insertada, toda vez que la acci�n se llev�
acabo con �xito, mientras que si por alguna raz�n la inserci�n no se realiza
sea devuelto FALSE.    
Esto nos aportar�a algo m�s de comodidad en el controlador, con lo que
podemos tener algo as� en el modelo:

{:lang="php"}
    public function insert($data)
    {
        $success = $this->db
            ->insert($this->_table, $data);
        if ($success)
        {
         return $this->db->insert_id();
        }
        else
        {
         return FALSE;
        }
    }

Aprovechando esta funcionalidad agregada al modelo en el *controlador* podr�amos tener:

{:lang="php"}
     if($this->paste_model->insert(array(
           'author' => 'juanaguilera',
           'description' => 'Grovy desde Perl',
           'language' => 'Perl',
	   'body' => 'Es bueno, bueno')))
      {
        $this->load->view('paste/home',$data);
      }
      else { 
        echo "Imposible<br>";
      }

Se espera que los datos que se graben en la tabla de la BD, sean generados por el
mismo proceso que se inicia con la recepci�n de la petici�n cliente, y no como
se muestra aqu�, tal como ya se coment�, esto s�lo tiene como fin mostrar la
incidencia del c�digo del *controlador* en el resto de la l�gica de negocio.

