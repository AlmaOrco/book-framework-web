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
        $con=mysql_connect($serverip,$username,$password) or die("no puedo conectar con mysql server");
        @mysql_select_db($database) or die("no puedo seleccionar DB");
        $sql="select id from pastes order by created_on desc";
        $result=mysql_query($sql) or die("No puedo grab pastes");
        if(!mysql_num_rows($result)){
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

## �C�mo ser�a esto empleando CI?

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

Podemos decir que el controlador de nuestra applicaci�n es una clase derivada deuna del framework CI_Controller y tiene como objeto cargar el los modelos y las vistas de la aplicaci�n.

Y entonces faltar�a ver el c�digo de la vista, lo tenemos en application/views/pastes/home.php

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

Ahora podemos volver sobre los tres puntos enumerados m�s arriba, comenzando por el primero:

1. �Sobre que BD est� trabajando nuestro c�digo?.

Para la mayor�a de los casos cada modelo interact�a con una �nica tabla de la base de datos; y tenemos algunas acciones que podr�amos llamar *acciones est�ndares* de interacci�n con esa tabla, son conocidos como m�todos CRUD (Create, Read, Update y Destroy). Entonces podr�amos especificar la tabla de la base de datos a trav�s de la clase.

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

2. �Cu�l es el nombre del fichero que contiene esta clase?.    
Bien podr�a ser models/user.php.
Llamamos a la clase *User*, est� contenida en el fichero user.php y la tabla
de la BD sobre la cual trabaja tiene el mismo nombre de la clase en plural, *users*
3. �Es todo lo necesario?.    
No, necesitaremos de c�digo PHP en controladores y en vistas.

Veamos la definici�n completa de una clase en el modelo:

{:lang="php"}
    <?
    class Pastes_model Extends CI_Model {
      public function __construct()
      {
        $this->load->database();
      }

      function get_pastes($num,$start=4) 
      {
        $query = $this->db->get('pastes');
        return $query->result_array();
      }
    }
    ?>

Con este modelo podemos obtener una respuesta para el usuario como la siguiente: 

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
 
Para esto tenemos un controlador como:    

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

Como vemos hacemos que desde el controlador se env�a la respuesta, evitando c�digo en una vista, lo que anteriormente hicimos en views/pastes/home.html

Ahora cambiamos un poco el modelo:

{:lang="php"}
    <?
    class Pastes_model Extends CI_Model{
      public function __construct()
      {
         $this->load->database();
      }

      function get_pastes($num=4,$start=0) 
      {
         $this->db->select('description,language')->from('pastes')->where('language','Ruby')
                                    ->order_by('created_on','desc')->limit($num,$start);
         $query=$this->db->get();
         return $query->result_array();
      }
    }
    ?>


Obtiene:
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
                [description] => Equivale al acostumbrado modo . . .
                [language] => Ruby
            )

         [3] => Array
            (
                [description] => Seguro convence
                [language] => Ruby
            )
    )

## Damos otro paso sobre el modelo

{:lang="php"}
    <?
     class Paste_model extends MY_Model
     {
      //protected $_table = 'pastes';
     }
    ?>

Y hemos escrito en MY_Model en application/core:

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

Observamos que el nombre de la clase es en singular siendo as� tanto en el modelo como en el controlador, y ya sabemos que el modelo trabaja sobre la tabla *pastes* de la BD.
Hemos agregado en el modelo la funci�n *get* aunque en nuestro ejemplo no la usamos, estar�a destinada a conseguir una �nica fila de la tabla, es decir, la usaremos cuando esperamos una sola fila como resultado de la consulta.
El controlador invoca get_all enviando 'Ruby' como par�metro, esto es como muestra, lo oportuno es que sea un requerimiento dado por el usuario por ejemplo.
Tambi�n vemos como en el modelo se crea un array asociativo usando como clave
el nombre de un par de columnas de la tabla, en este caso author y language, es debido a que el par�metro de la funci�n *where* debe ser dado como array.

Damos una vuelta de tuerca en MY_Model

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

Por �ltimo hay que notar que hasta ahora no hemos visto en ninguna parte del c�digo, hacer referencia a la BD
sobre la cual trabaja la app. con esto y otros simples detalles de configuraci�n de la app comenzamos nuestra
segunda clase.

Como �ltima cuesti�n podemos pensar en una insersi�n, ten�amos mas arriba el m�todo.

Con esto

{:lang="php"}
    public function insert($user) {
      return $this->db->insert('users',$user);
    }

Tambi�n en MY_Model puedo operar de la siguiente manera

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

El objetivo principal es devolver el id de la fila insertada, cuando la inserci�n fue correcta, mientras que si no lo fue
el m�todo retorna FALSE, y esto nos pone algo m�s c�modo en el controlador, con lo que podemos tener algo as�:

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

