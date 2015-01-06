# JSF Clase 2

Conociendo que una petici�n es procesada por JSF, a trav�s de varias fases,
intentaremos analizar alguna de �stas y adem�s aprenderemos acerca del concepto
JSF PhaseListener.

[ejemplo fases JSF](http://www.itcuties.com/j2ee/jsf-phaselistener/)

Es posible poner un *phase listener* a cada una de las fases. Podr�amos modificar los datos enviados en la petici�n, si as� nos lo propusi�ramos, o bien podr�amos
modificar la vista que ser� renderizada. En este caso nos proponemos desactivar o bien esconder (disable/hide) un componente InputText presente en la vista.
En concreto pretendemos que cuando el usuario introduzca _hide me_ como texto y
el formulario sea enviado a la aplicaci�n en el servidor, nuestro PhaseListener
deber�a poner el atributo *rendered* de dicho componente a *false*.
Mientras que introducir _disable me_ como texto del componente en cuesti�n, este
deber�a ser *disabled*

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
       // La fase que interesa, en �sta es llamado el listener
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
        // Go to every child
        for (UIComponent child: component.getChildren()) {
          // Display el ID y tipo de componente
          System.out.println("+ " + child.getId() +
                           " ["+child.getClass()+"]");
          // Si el componente es el nuestro, coteja ID, entonces 
          // verifica si debe esconderlo
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
          // Process next node
          processViewTree(child);
        }
      }
    }

Nuestra clase implementa la interfaz PhaseListener.   
�Cuando es ejecutado el c�digo?   
Es ejecutado antes de la �ltima fase RENDER RESPONSE
En el m�todo beforePhase el *view tree* es procesado hacia abajo de forma
recursiva. Cuando el InputText componente es encontrado, usa el ID para cotejar,
nuestro valor a encontrar es *dummy-text-id*, entonces es cuando verifica
si el el valor del texto es *hide me* o *disable me*, con lo que el componente
podr� ser modificado.

El uso de un Phaselistener exige un detalle de configuraci�n en el fichero
faces-config.xml
Para esto usamos la *tag*
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

El siguiente es el contenido del fichero *index.xhtml*, este es la vista
de nuestra aplicaci�n, podemos apreciar el componente InputText *dummy-text-id*

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

Y el fichero web.xml que no tiene nada especial para esta aplicaci�n

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


## Diferencia entre managed bean y CDI

Un bean tiene una verdadera identidad en el �mbito del container.
Antes de Java EE 6 no hab�a una clara definici�n del t�rmino, hemos llamado bean a clases java y hemos usado beans durante a�os.

�De donde se parte? 
Hab�an beans en las especificaciones EE, donde se incluye EJB beans
y _JSF managed beans_, y otros third-party frameworks como Spring indrodujeron
su propia idea del significado de bean.

La *conclusi�n* a vistas de las diferencias es que no hay una com�n definici�n.

Java EE 6, dio por v�lida la especificaci�n Managed Bean, como as� tambi�n la siguiente *objetos container-managed*, con un m�nimo de restricciones, y conocidos como POJO.    
Los beans soportan un peque�o conjunto de servicios b�sicos, tales como resource injection, interceptors y lifecycle callbacks. EJB y CDI se construyeron
sobre este modelo.
 
La _especificaci�n CDI_ aporta nuevos servicios, estos pueden ser usados por el
container para crear y destruir instancias de tus beans asociados con un contexto, injectarlos en otros beans, usarlos en expresiones El, especialializarlos
con cualificadores (anotaciones) y agregarles interceptores y decoradores, sin
modificar tu c�digo ya existente. S�lo necesitar�s agregar anotaciones.

# Creamos bean que use CDI
---------------------------

{:lang="java"}
    package mypackage;
    import java.io.Serializable;
    import java.util.ArrayList;
    import java.util.List;
    import javax.annotation.PostConstruct;
      //import javax.faces.view.ViewScoped;
      //import javax.enterprise.context.ApplicationScoped;
      //import javax.inject.Named;
    import javax.faces.bean.ManagedBean;
    import javax.faces.bean.ApplicationScoped;
     
    //@Named
    @ManagedBean
    @ApplicationScoped
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
     
      public void addItem() {
        if (item != null && !item.isEmpty()) {
          items.add(item);
          item = null;
        }
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

{:lang="xhtml"}
    <!DOCTYPE html>
    <html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core"
      xmlns:ui="http://java.sun.com/jsf/facelets">
     <h:head>
       <title>CDI ViewScoped example</title>
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

