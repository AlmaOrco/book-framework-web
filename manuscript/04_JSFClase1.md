# JSF Clase 1

## JSF en pocas palabras

No usamos el viejo estilo de sintaxis jsp; nuestras p�ginas usar�n xhtml (1), 
en el que definimos un namespace  xmlns:h="http://java.sun.com/jsf/html".
Se usan componentes JSF como h:head, h:body, h:form. Tambi�n los componentes tienen facetas
est�s se definen en otro namespace xmlns:f...
M�s adelante veremos que tambi podemos usar elementos a�n de otro namespace  xmlns:ui...
Las p�ginas que no contienen input pueden omitir h:form.
No es necesaria una entrada @taglib como en p�ginas jsp.
Recuerda que la URL no coteja el real filename, usar�s tipos de nombres como *blala.xhtml* para p�ginas a mostrar al usuario.   
Veamos un ejemplo,    

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
       "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml"
       xmlns:h="http://java.sun.com/jsf/html"
       xmlns:f="http://java.sun.com/jsf/core">
      <head>
       <title>
        Beer Selection
       </title>
      </head>
      <h:body>
       <h1 align="center">Beer Selection Page</h1>
        Select un color<p/>
        Color:
       <h:form>
        <h:selectOneMenu value="#{order.color}" >
          <f:selectItem itemValue="light" itemLabel="Es ligth" />
          <f:selectItem itemValue="amber" itemLabel="Es amber" />
          <f:selectItem itemValue="brown" itemLabel="Es brown" />
          <f:selectItem itemValue="dark" itemLabel="Es dark" />
        </h:selectOneMenu>
        <center>
         <h:commandButton value="submit" action="suc"/>
        </center>
       </h:form>
      </h:body>
    </html>

Una vez visto como vienen organizadas nuestras p�ginas usando componentes del framework, pasemos a introducir otros conceptos como *beans*.   
Los _beans administrados_, aunque mejor usaremos el t�rmino *Managed Beans*, son clases java que bien pueden instanciarse mediante definici�n en el fichero
faces-config.xml o mediante anotaciones Java como  esta *@ManagedBean*

1. Son normalmente POJOs.
2. Tienen pares de m�todos getter y setter correspondientes a cada elemento de input en el formulario
3. Tienen un m�todo *action controller* que no toma argumento y retorna un string.  Este m�todo est� ligado a la acci�n del h:commandButton en el formulario.
4. Normalmente tienen marcadores para propiedades derivadas, lo que es informaci�n a tenerse en cuenta en el proceso de los datos de entrada.

Si escogemos anotaci�n, podr�amos tener:
{:lang="java"}
    @ManagedBean 
    public class SomeName { ... }

Con esto nos referimos a un bean del backend con *#{someName.blah}*, donde el
nombre del bean es el nombre de la clase con la primer letra pasada a min�scula.
Request es el �mbito por defecto, y "blah" es un nombre de m�todo o una
propiedad la que enlaza con un m�todo getter o setter; esto �ltimo ocurre en
h:inputText, mientras que la action de h:commandButton es un exacto nombre de m�todo.

Acerca del valor de retorno del m�todo *action controller* tenemos que conocer que hay un mecanimo detr�s de la escena que consiste en que si
el m�todo retorna _foo_ o _bar_ y no hay un expl�cito mapeo en el fichero faces-config.xml, entonces
la p�gina resultante es foo.xhtml � bar.xhtml, desde la misma carpeta que est� contenido el formulario.

Por ejemplo, si en una p�gina inicial tenemos un bot�n as�:
{:lang="xhtml"}
    <h:commandButton action="#{navigator.choosePage}"/>

Y un bean como el siguiente

{:lang="java"}
    @ManagedBean 
    Class  Navigator {...}

Podr�amos plantear que el m�todo _choosePage_ retorne tres posibles strings *page1*, *page2*, � *page3*.   
Los nombres cotejados por el retorno del m�todo choosePage ser�n page1.xhtml, page2.xhtml, y page3.xhtml. Esto significa que son tres vistas, escritas en xhtml, formando parte de tu aplicaci�n, dise�adas y escritas para ser enviadas como 
respuesta al cliente.

Veamos la clase Navigator
{:lang="java"}
    @ManagedBean                                                          
    public class Navigator {
      private String[] resultPages = { "page1", "page2", "page3" };
      public String choosePage() {
        return(RandomUtils.randomElement,resultPages);
      }
      // getter y setter
    }

Esta es una declaraci�n de un *managed bean*, sin requerir una entrada en faces-config.xml, ya que se ha usado una *Java annotation*.
Debido a que no se da un nombre ser� usado el nombre de la clase con la primer letra en min�scula.
Podemos dar un nombre al bean, que crear� el framework para nosotros con la
siguiente notaci�n,
{:lang="java"}
    @ManagedBean(name="someName")   
El �mbito no se ha dado entonces, por defecto el *scope* es *request*. Podemos indicar el
�mbito del bean tambi�n con notaci�n, esto podr�a ser as�,
{:lang="java"}
    @SessionScoped 
Puesto que en nuestro ejemplos no agregamos expl�citas reglas de navegaci�n
en el fichero faces-config.xml, el cual ni estar�a presente en esta supuesta aplicaci�n ejemplo.
Los valores de retorno corresponden a page1.xhtml, page2.xhtml, y page3.xhtml.

## Fases del framework

Antes de pasar al detalle de las fases del *ciclo de vida* interesa tener presente como vas a intervenir como desarrollador en todo este proceso de escribir una aplicaci�n.
Como primera cuesti�n habr�s instalado una *implementaci�n* de la especificaci�n de referencia
de JSF, conocida como JSF RI. Mojarra es el nombre de la actual JSF RI, y es la implementaci�n que viene con servidores como Glassfish entre otros.

Cuando alguien comienza a estudiar y comprender como trabaja JSF, ya viene con alguna pr�ctica realizada
con la API servlet y el uso de t�cnicas que pretenden eliminar c�digo Java de las vistas, es decir, que quien comienza con JSF, normalmente conoce lo que es un scriptlet; la cuesti�n aqu� es hacer incapi� en el hecho
de la existencia de est�ndares Java destinados ha organizar mejor el c�digo y 
hacer m�s productivo el desarrollo de aplicaciones, nos referimos a est�ndares como Expression Language y la librer�a JSTL.    
Antes de estudiar JSF, tal ves sea preciso formularnos la siguiente pregunta:   
*Los servlets, �no tienen damasiadas responsibilidades?*       
En en su respuesta estar�a la justificaci�n del por qu� estudiamos JSF.

Tomemos la pregunta como excusa para repasar algunas de las acciones y tareas de las que se ocupa
un servlet; es quien obtiene los datos de las peticiones HTTP, los convierte 
si es necesario, los valida, los pone a disposici�n de las p�ginas JSP despach�ndolo (forward), agregando previamente para esto 
�ltimo, nuevos atributos al objeto request; instancia objetos a partir
de definiciones que se encuentran en el modelo, y pasa datos de la petici�n cliente
hacia el modelo, enlazando la l�gica de negocio con lo que es la input/output HTTP.

Partimos del hecho que un framework como JSF, nos va a ofrecer un conjunto de componentes espec�ficos para alcanzar las tareas 
de la aplicaci�n, trabajando modularmente y sobretodo nos permitir� dedicar
nuestro esfuerzo a desarrollar la l�gica de negocio, y no distraer nuestra
atenci�n en tareas repetitivas y comunes en las aplicaciones web. JSF nos obliga a trabajar en un *flujo bien concreto*, el cual se conoce como _ciclo de vida_.

JSF es un framework basado en _componentes_ y no en *acciones* como CodeIgniter u otros
similares sin distinci�n del lenguaje; cuando surgi� JSF rompi� con
la costumbre de la �poca. JSF ten�a una forma muy diferente, el "status quo" del momento, de los framworks web, estaba basado en acciones.   

## Controlador: enlace entre usuario y aplicaci�n

Con el c�digo del ejemplo xhtml visto anteriormente, podemos ver el aspecto que pueden
tener las vistas, �stas adem�s de no contar con la existencia de c�digo 
Java entremezclado marcan a su vez una distancia con la codificaci�n HTML;
esto en cuanto a las p�ginas destinada al usuario, pero �que pasa con el *modelo*?   
el modelo estar�a participando, de similar
forma a lo visto en p�ginas JSP usando EL, expression language, concretamente "#{order.color}" en el formulario
podr�a pensarse que hace referencia a un atributo de un objeto, tal vez creado por el framework JSF; con t�cnica de uso servlet/JSP, hubi�ramos dicho que tal objeto fue creado por el servlet, *agregado* en el objeto request y enviado a la vista.

La realidad es que el *controlador JSF* tiene definido en exacto flujo de trabajo. El controlador o servlet Faces
act�a como enlace entre el usuario y la l�gica de la aplicaci�n, siendo el
ciclo de vida un flujo de eventors entre c�digo de aplicaci�n y peticiones de usuario.    
La descripci�n del ciclo ser�a algo as�:   
Una vez recibida una petici�n web, digamos una primera petici�n, el servlet,
*controlador* Faces, va a manipular esta petici�n creando el contexto JSF, el cual es un *objeto Java* que *retiene* todos los datos de la aplicaci�n, luego 
el *controlador* enruta la petici�n a la p�gina requerida. La p�gina normalmente devuelve o retorna los datos
de la aplicaci�n desde el contexto JSF usando un simple Expression Language. Esto acontece en cuanto a la primera petici�n, con las las siguientes
peticiones, el *controlador* modifica cualquier dato del Modelo, suministrando cualquier nueva input y actualizando el contexto.
Los desarrolladores JSF tienen acceso program�tico al ciclo descripto; acceso en cualquier momento de la ejecuci�n del ciclo, es decir en
cualquiera de sus fases, significa que pueden ejercer un alto grado de control
sobre la conducta de la aplicaci�n, en el sitio y momento que se lo propongan.  
Dicho esto, es de esperar que alguien que tenga pensado usar el framewor conozca las *seis faces del ciclo de vida de JSF*,
reciben los siguientes nombres,

1. Restore view
2. Apply request values
3. Process validations
4. Update model values
5. Invoke application
6. Render response

I> Buena propuesta para captar un poco mejor las [fases del framewok JSF](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html), es la de codificar
una aplicaci�n completa, tan s�lo con el prop�sito de entender lo que pasa con una petici�n web de principio a fin.

## Un framework statefull

Cuando desarrollamos con un framework *basado en acci�n* tenemos 'tags' que nos
ayudan en el desarrollo de la vista, y podemos pensar que est� ligado, lo que 
significa que si quitamos las tags y lo hacemos a mano tambi�n lo
conseguir�amos.
Cuando submit un formulario basado en acci�n, nuestra aplicaci�n basada en la aci�n no se fija en el estado de la vista.
No tiene en cuenta como la vista fue escrita, tampoco si es un html est�tico
o un jsp. Se montar� una petici�n que tiene como par�metro tal o cual dato, que los aporta el usuario en el formulario.
Los frameworks basados en aci�n no conocen un formulario, conocen la *request* y la informaci�n que est� siendo pasada en ella al servidor.

Cuando trabajamos con JSF, _�l_ es quien crea el formulario, es decir cuando se
accede desde una petici�n
a la URL de una p�gina *xhtml*, ocurre que en lugar de ejecutar un servlet,
_JSF va a leer el archivo y montarlo en memoria_, lo que es lo mismo, JSF crea un �rbol de componentes; luego ese �rbol en memoria ser� pasado por un renderizador.

El �rbol de componentes representa la estructura de la p�gina y JSF lo utilizar� para escribir el c�digo HTML, en un momento del ciclo de vida.

En definitiva JSF, guarda en memoria el �rbol que fue usado para generar un formulario.    
Esto no ocure en un servlet, que genera autom�ticamente un HTML a partir de
una p�gina JSP, esto lo hace el servidor para enviar datos al cliente, es para
esto que que tiene en cuenta el formulario, para enviarlo, lo mismo es en los frameworks
basado en acci�n, mientras que JSF, al guardar en memoria la estructura de
la p�gina, en ese �rbol de componentes, _�l_ va a comparar cada atributo de
la petici�n con los campos que estaban disponibles para el usuario.
Teniendo toda esta informaci�n JSF, podr� validar los datos enviados por el usuario.
Se puede decir que JSF sabe todo acerca de un formulario, lo que hab�a antes, y lo que hay ahora que el usuario envi� datos, JSF conoce el estado de la vista y lo mantiene durante las peticiones, por ello podemos considerarlo un *framework statefull*.

Una herramienta �til tal como [demuestra Balusc](https://balusc.blogspot.com.es/2006/09/debug-jsf-lifecycle.html) (nombre de un usuario con merecido respeto
en stackoverflow), es destinar una aplicaci�n JSF para tener una visi�n de cada una de las fases, la entrada y salida de cada una de ellas. La aplicaci�n de 
Balusc pretende que seamos avisados antes y despu�s de cada fase, ofrece sus reflexiones e indica las posibles buenas pr�cticas que se derivan de esta *aplicaci�n explicativa*

En la _siguiente clase_ vamos a proponernos *jugar*, aunque sea m�nimamente con el ciclo de vida, con el fin de
ver m�s de cerca la responsabilidad de alguna de las fases, utilizaremos un _PhaseListener_, y principalmente demostraremos que estamos en condiciones de obtener un *acceso granular a un punto concreto* del ciclo de vida de una petici�n.

Ahora veamos una clase Java, representa el modelo y este es el c�digo que contiene
la l�gica de la aplicaci�n,

{:lang="java"}
    package mypackage;
    import javax.faces.bean.ManagedBean;
    import javax.faces.bean.SessionScoped;
    import javax.faces.bean.ApplicationScoped;
    
    @ManagedBean
    @ApplicationScoped
    public class Customer {
      private String firstName;
      private String lastName;
      public String getFirstName() {
        return firstName;
      }
      public void setFirstName(String firstName) {
        this.firstName = firstName;
      }
       // resto de getter y setter
      public String save() {
        return "/showCustomer.xhtml";
      }
    }

Mostramos parcialmente el contenido de _web.xml_
{:lang="xml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <!-- se omiten las definiciones -->
    <!-- Faces Servlet -->
    <servlet>
      <servlet-name>Faces Servlet</servlet-name>
      <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
      <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- Faces Servlet Mapping -->
    <servlet-mapping>
      <servlet-name>Faces Servlet</servlet-name>
      <url-pattern>*.jsf</url-pattern>
    </servlet-mapping>
    <!-- Welcome files -->
    <welcome-file-list>
      <welcome-file>index.html</welcome-file>
    </welcome-file-list>
    <context-param>
      <param-name>javax.faces.PROJECT_STAGE</param-name>
      <param-value>Development</param-value>
    </context-param>

Del fichero web.xml destacamos el siguiente punto:    
La extensi�n *.jsf* normalmente se mapeaba en web.xml, as� lo vemos en el  ejemplo.
Mientras que la extensi�n *.xhtml* es la extensi�n de los ficheros _actuales_
y que f�sicamente est�n en el webcontent de tu aplicaci�n.   
La conclusi�n es que si invocas una p�gina con extensi�n .jsf como por ejemplo
http://localhost:8080/webapp/page.jsf, entonces el 
servlet JSF debe localizar a page.xhtml y realizar el correspondiente  parse/render de los componentes JSF

Desde JSF 2.0 un mapeo a p�ginas *.xhtml es posible, lo que significa que la URL http://localhost:8080/webapp/page.xhtml puede ser accedida, tendremos en web.xml    

{:lang="xml">
    <url-pattern>*.xhtml</url-pattern>

Observa el par�metro *Project Stage* este es un par�metro de contexto que identifica
el *estadio* o etapa de la aplicaci�n en su ciclo de vida. El estadio de la aplicaci�n
puede afectar la conducta de la misma. Un ejemplo de ello puede ser mensajes de error, que se busca mostrar durante la etapa o fase de desarrollo, mientras
que ser� preciso suprimirlos en etapa de Producci�n.

Siguiendo con el ejemplo, este ser�a el fichero _index.html_
{:lang="html"}
    <html>
      <head>
       <meta http-equiv="refresh" content="0; URL=editCustomer.jsf">
      </head>
    </html>

Mostramos el *body* de _cancelled.xhtml_, una de las p�ginas de las vistas de la aplicaci�n,
{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h:outputText value="Cancelled customer editing!"/>
    </body>

Parcialmente el fichero _editCustomer.xhtml_, otra de las p�ginas,
{:lang="xhtml"}
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h2><h:outputText value="Edit Customer"/></h2>
      <h:messages showDetail="true" showSummary="false"/>
      <h:form id="form">
        <h:panelGrid id="grid" columns="2">
          <h:outputLabel value="First Name:" for="firstName"/>
          <h:inputText id="firstName" value="#{customer.firstName}"
                   required="true"/>
          <h:outputLabel value="Last Name:" for="lastName"/>
          <h:inputText id="lastName" value="#{customer.lastName}"
                   required="true"/>
        </h:panelGrid>
        <h:commandButton id="save" action="#{customer.save}" value="Save"/>
        <h:commandButton id="cancel" action="/cancelled.xhtml" value="Cancel"
                immediate="true"/>
      </h:form>
    </body>

De la anterior vista resaltamos el uso del atributo *immediate* con valor true,
este produce que desde la actual fase pasar� directamente a la �ltima, sin producir cambios de ning�n tipo en los datos del modelo, con lo que la renderizaci�n
desde ese punto de vista es inmediata. Mientras que el atributo *required* con
valor *true* para alg�n componente, en este caso los dos inputText, indica que el ciclo
de vida finalizar� en la fase 3 *proccess validations* si los componentes est�n _vac�os_,
y el servlet controller *renderiza error* en la �ltima etapa.

Parcialmente _showCustomer.xhtml_, la �ltima vista,
{:lang="xhtml"}
    <body>
      <h1><h:outputText value="MyGourmet"/></h1>
      <h2><h:outputText value="Show Customer"/></h2>
      <h:panelGrid id="grid" columns="2">
        <h:outputText value="First Name:"/>
        <h:outputText value="#{customer.firstName}"/>
        <h:outputText value="Last Name:"/>
        <h:outputText value="#{customer.lastName}"/>
      </h:panelGrid>
      <h:outputText value="Customer saved successfully!" />
    </body>

Por �ltimo ver�s que los datos se mantienen, en los sucesivos accesos a la
aplicaci�n, observa el *scope* que se ha usado para la creaci�n del bean.
Puedes modificar el �mbito del bean mediante las correspondientes *anotaciones*,
verificando la conducta de la aplicaci�n.
