# JSF Clase 2

Conociendo que una petici�n es procesada por JSF, a trav�s de varias fases,
intentaremos analizar alguna de �stas y adem�s aprenderemos acerca del concepto
JSF PhaseListener.

## JSF: eventos y ciclo de vida

Es posible poner un *phase listener* a cada una de las fases. Podr�amos modificar los datos enviados en la petici�n, si
as� nos lo propusi�ramos, o bien podr�amos modificar la vista que ser�
renderizada.    
En el siguiente ejemplo nos proponemos desactivar o bien esconder (disable/hide) un componente InputText presente en la vista.
En concreto pretendemos que cuando el usuario introduzca _hide me_ como texto y
el formulario sea enviado a la aplicaci�n en el servidor, nuestro PhaseListener
deber�a poner el atributo *rendered* de dicho componente a *false*.
Mientras que introducir _disable me_ como texto del componente en cuesti�n, indicar�a que deber�a ser *disabled*.

Escribimos la clase FieldDisableListener.java

{:lang="java"}
    package mypackage;
    import javax.faces.component.UIComponent;
    import javax.faces.component.html.HtmlInputText;
    import javax.faces.event.PhaseEvent;
    import javax.faces.event.PhaseId;
    import javax.faces.event.PhaseListener;
        
    public class FieldDisableListener implements Pha
    seListener {
      private static final long serialVersionUID = -7607159318721947672L;
       // fase que interesa, en esta es llamado el listener
      private PhaseId phaseId = PhaseId.RENDER_RESPONSE;
       /**
        * Recorrido recursivo del �rbol de componentes 
        */
      public void beforePhase(PhaseEvent event) {
         processViewTree(event.getFacesContext().getViewRoot());	
      }
      public void afterPhase(PhaseEvent event) {
        // No nos interesa hacer algo
      }
      public PhaseId getPhaseId() {
        return phaseId;
      }
      private void processViewTree(UIComponent component) {
        // Pasando por todos los nodos hijos
        for (UIComponent child: component.getChildren()) {
          // Display el ID y tipo de componente
          System.out.println("+ " + child.getId() +
                           " ["+child.getClass()+"]");
          // conoce si es nuestro componente cotejando ID,
          // y si es, verifica si debe esconderlo
          if ("dummy-text-id".equals(child.getId())) {
            // Obtiene el texto del componente
            HtmlInputText inputText = (HtmlInputText)child;
            String  inputTextValue = (String)inputText.getValue();
            // Si coteja el texto con "hide me", esconde el campo
            if ("hide me".equals(inputTextValue))
              inputText.setRendered(false);
               // Disable el campo si el texto coteja "disable me"
            if ("disable me".equals(inputTextValue))
              inputText.setReadonly(true);
          }
          // procesa siguiente nodo
          processViewTree(child);
        }
      }
    }

Nuestra clase implementa la interfaz PhaseListener.   
�Cuando es ejecutado el c�digo?   
Es ejecutado antes de la �ltima fase RENDER RESPONSE
En el _m�todo beforePhase_ el *view tree* es procesado hacia abajo de forma
recursiva. Cuando el InputText componente es encontrado, usa el ID para cotejar,
nuestro valor a encontrar es *dummy-text-id*, entonces es cuando verifica
si el el valor del texto es *hide me* o *disable me*, con lo que el componente
podr� ser modificado.

El uso de un Phaselistener exige un detalle de configuraci�n en el fichero
faces-config.xml Para esto usamos la *tag* phase-listener.
En caso de usar m�s de un listener, el orden en que aparecen en el fichero
de configuraci�n es tenido en cuenta como �rden de ejecuci�n.

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?>
    <faces-config xmlns="http://java.sun.com/xml/ns/javaee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
         http://java.sun.com/xml/ns/javaee/web-facesconfig_2_1.xsd"
       version="2.1">
     
       <lifecycle>
          <phase-listener>mypackage.FieldDisableListener</phase-listener>
       </lifecycle>
    </faces-config>

El siguiente es el contenido del fichero *index.xhtml*, es la vista
de nuestra aplicaci�n, donde podemos apreciar el componente
InputText *dummy-text-id*

{:lang="xhtml"}
    <?xml version="1.0" encoding="UTF-8"?> 
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 
    <html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:f="http://java.sun.com/jsf/core"      
      xmlns:h="http://java.sun.com/jsf/html"> 
    <h:head> 
      <title>Phase Listener example</title> 
    </h:head> 
    <h:body> 
      <h3>Phase Listener example</h3> 
      <h:form> 
        <h:inputText id="dummy-text-id"/> 
    	<h:commandButton id="button" value="Submit" /> 
        <h:messages /> 
    </h:body> 
    </html>

Y el fichero web.xml que no tiene nada especial para esta aplicaci�n,

{:lang="xhtml"}
    <!DOCTYPE web-app PUBLIC
     "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
     "http://java.sun.com/dtd/web-app_2_3.dtd" >
    <web-app>
        <display-name>Ejemmplo JSF 2 Phase listener</display-name>
        <!-- The welcome page -->
        <welcome-file-list>
            <welcome-file>index.xhtml</welcome-file>
        </welcome-file-list>
        <!-- Start JSF -->
        <servlet>
            <servlet-name>Faces Servlet</servlet-name>
            <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <!-- JSF URL mapping -->
        <servlet-mapping>
            <servlet-name>Faces Servlet</servlet-name>
            <url-pattern>*.xhtml</url-pattern>
        </servlet-mapping>
    </web-app>

## Diferencias entre managed bean y CDI

Un bean tiene una verdadera identidad en el �mbito del container.
Viene a cuenta un poco de historia. Antes de Java EE 6 no hab�a una clara
definici�n del t�rmino, con lo cual se ha llamado beans a clases Java y
hemos usado beans durante a�os.

�De donde se parte? 
Hab�a beans en las especificaciones EE, las que incluyen EJB beans
y _JSF managed beans_, y otros frameworks third-party como Spring tambi�n
indrodujeron su propia idea del significado de bean.

La *conclusi�n* a vistas de las diferencias es que no hay una com�n definici�n.

Java EE 6, dio por v�lida la *especificaci�n de Managed Bean*, como as� tambi�n
la siguiente:   
_objetos container-managed_, con un m�nimo de restricciones, y conocidos como POJO.    
Los beans soportan un peque�o conjunto de servicios b�sicos, tales como resource injection, interceptors y lifecycle callbacks. EJB y CDI se construyeron
sobre este modelo.
 
La _especificaci�n CDI_ aporta nuevos servicios, estos pueden ser usados por el
container para crear y destruir instancias de tus beans asociados con un contexto, injectarlos en otros beans, usarlos en expresiones El, especialializarlos
con cualificadores (anotaciones) y agregarles interceptores y decoradores, sin
modificar tu c�digo ya existente. S�lo necesitar�s agregar anotaciones.

## Creaci�n de beans usando CDI

Nos proponemos codificar una aplicaci�n que demuestre el uso del soporte CDI
en JSF, para esto debemos trabajar con una API diferente a las empleadas con el
modelo *managed bean*, concretamente usamos la notaci�n Java *@Named*, con lo que importar�amos el paquete javax.inject.Named, y para nuestra propuesta
en cuanto al �mbito o scope de nuestro bean usar�amos *Application* o *View*; una vez
desplegada y probada la aplicaci�n, podemos comentar las l�neas @Named y
@ViewScoped y volver a compilar la clase tal como la vez m�s abajo.
Con este cambio en el c�digo Java del modelo, nada cambiar�a en el resultado. Se trata que CDI, puede ser usado en nuestra aplicaci�n, sin afectar al c�digo cuando se trata de
una codificaci�n que ven�a usando *managed bean*, como hemos dicho; el �nico cambio est�
en las notaciones; el soporte CDI es m�s eficiente, a tal punto que la comunidad recomienda su uso, quiz�s con la �nica excepci�n de los containers Tomcat,
m�s que nada por no ser trivial la configuraci�n de este servidor con soporte CDI.

{:lang="java"}
    package mypackage;
    import java.io.Serializable;
    import java.util.ArrayList;
    import java.util.List;
    import javax.annotation.PostConstruct;
      //import javax.faces.view.ViewScoped;
      //import javax.inject.Named;
    import javax.faces.bean.ManagedBean;
    import javax.faces.bean.SessionScoped;
     
    @ManagedBean
    @SessionScoped
    public class TestBean implements Serializable {
     
      private static final long serialVersionUID = 1L;
     
      private List<String> items;
      private String item;
      
      @PostConstruct
      private void init() {
        items = new ArrayList<String>();
        items.add("Item 1");
        items.add("Item 2");
        items.add("Item 3");
      }
     
      public String addItem() {
        if (item != null && !item.isEmpty()) {
          items.add(item);
          item = null;
        }
	return null;
      }
      public String getItem() {
        return item;
      }
      public void setItem(String item) {
        this.item = item;
      }
      public List<String> getItems() {
         return items;
      }
    }

La vista es como sigue,
{:lang="xhtml"}
    <!DOCTYPE html>
    <html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core"
      xmlns:ui="http://java.sun.com/jsf/facelets">
     <h:head>
       <title>Managed bean  / CDI ViewScoped example</title>
     </h:head>
     <h:body>
       <h:form>
         <ui:repeat value="#{testBean.items}"
                      var="item">
          <div>
            <h:outputText value="#{item}" />
          </div>
         </ui:repeat>
         <br />
         <h:inputText value="#{testBean.item}" />
         <h:commandButton value="Add item"
                 action="#{testBean.addItem}">
           <f:ajax execute="@form" render="@form" />
         </h:commandButton>
       </h:form>
     </h:body>
    </html>

En este ejemplo se muestra, como un componente es afectado por el uso de *Ajax*, decir que JSF tiene un buen soporte para ajax, provee la tag _f:ajax_ para
manejar llamadas ajax. El atributo *execute* admite como valores una lista de IDs de componentes los cuales deber�an ser incluidos en una petici�n Ajax.
Y *render* admite una lista de IDs de componente que ser�n *modificados* despu�s
de una petici�n Ajax.

