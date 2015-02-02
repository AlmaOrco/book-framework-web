# JSF Clase 4

Si bien, en esta clase nos ocupamos de modificar la clase aceptando acciones de modificaci�n y
eliminaci�n, al principio nos proponemos algunas indicaciones sobre el uso de par�metros.

## Usando par�metros

La etiqueta _f:param_ permite pasar par�metros a un componente, en el siguiente ejemplo vemos
un uso de _f:param_ en un componente _h:outputFormat_

Tal como tenemos el c�digo de _index.xhtml_ al final de la clase anterior, puedes agregar 
al final del body, luego del componente h:dataTable
{:lang="xhtml"}
    <ul>
      <ui:repeat value="#{viewManager.cacheList}" var="item">
        <li>
         <h:link value="View details of #{item.key}" outcome="edit">
             <f:param name="id1" value="#{item.key}" />
             <f:param name="id2" value="#{item.value}" />
         </h:link>
        </li>
      </ui:repeat>
    </ul>

Se observa el uso del componente h:link donde existe un atributo _outcome_ y dos subelementos
facetas _f:param_, esto genera el siguiente c�digo HTML
{:lang="xhtml"}
    <a href="/.../edit.xhtml?id1=xxx&id2=zzzz">View details of xxx</a>

Seg�n valor de _outcome_ es renderizado _edit.xhtml_, que podr�a tener un contenido como sigue,
{:lang="xhtml"}
    <h2>CRUD estadio dos, editando uno</h2>
    <i>Key vale #{param['id1']} y Value vale #{param['id2']}</i>

Tambi�n se ha usado la faceta _ui:repeat_ para realizar el bucle sobre la lista, en este caso
no nos interesa generar una tabla HTML.

En el siguiente ejemplo usamos _f:viewParam_ dentro de una faceta _f:metadata_
de la vista _cursoView.xhtml_; entonces con este componente _h:link_
{:lang="xhtml"}
    <h:link outcome="cursoView.xhtml?code=#{v_curso.code}"
       value="#{v_curso.title}" />
ser� renderizado _cursoView.xhtml_, el cual tiene
{:lang="xhtml"}
    <f:metadata>
      <f:viewParam name="code" value="#{cursoHome.cargar}" />
    </f:metadata>
Donde _f:viewParam_ procesa el par�metro GET, y tal como lo hace un componente h:inputText, realiza setting,
conversi�n y validaci�n, con lo que el valor del par�metro _code_ sirve para realizar el _set_ del atributo
_cargar_ del bean _cursoHome_, representado como _#{cursoHome.cargar}_ en el atributo value.    
Si el atributo value se omite, entonces queda disponible en la vista en _#{code}_
	
�Qu� diferencia existe en usar @ManagedProperty en relaci�n a f:viewParam?
Veamos, tendr�amos los siguientes  dos bloques de c�digo:

{:lang="java"}
    @ManagedProperty(value = "#{param.id}")
    private Integer id;

{:lang="xhtml"}
    <f:metadata>
      <f:viewParam name="id" value="#{someBean.id}"/>
    </f:metadata>

*f:viewParam*

Set el valor durante la fase _update model_.

El valor asignado no est� disponible durante @PostConstruct, entonces necesitas
agegar un f:event dentro de f:metadata para inicializar los valores, o bien y m�s simple
puedes usar f:viewAction para el mismo fin.    

Permite anidar f:converter y f:validator para una m�s detallada conversi�n/validaci�n; pudi�ndose usar un h:message. Ejemplo:
{:lang="xhtml"}
    <f:metadata>
      <f:viewParam id="id" name="id" value="#{bean.id}" required="true">
        <f:validateLongRange minimum="10" maximum="20" />
      <f:viewParam>
      <f:viewAction action="#{bean.init}" />
    </f:metadata>
<h:message for="id" />

*@ManagedProperty*

Set los valores inmedi�tamente luego de la construcci�n del bean.    

El valor est� disponible durante @PostConstruct lo cual permite inicializaci�n de otras propiedades basadas sobre los valores.

Puedes [leer m�s](http://balusc.blogspot.com.es/2011/09/communication-in-jsf-20.html#ProcessingGETRequestParameters) sobre el uso y proceso de par�metros GET 

## CRUD: estadio final 

#### Una vista, ViewScoped, add, edit y delete

Partiendo del �ltimo estadio de la app conseguido en la clase anterior, tendr�amos nuestra �nica vista con dos elementos h:form,
utilizando el primero para introducir datos y el segundo destinado a permitir la _modificaci�n_ de datos de items ya existentes.
En un tercer formulario deber�amos mostrar los datos existentes a ese momento, podemos utilizar una tabla, y permitir la edici�n
de cualquiera de los items utilizando un bot�n o un enlace; sin preocuparnos de este �ltimo formulario, el index.xhtml lo encontramos
de esta manera,
{:lang="xhtml"}
    <h:panelGroup rendered="#{empty viewManager.cacheList}">
       <p>No hay datos a�n.</p>
    </h:panelGroup>
    <h:panelGroup rendered="#{!viewManager.edit}">
       <h3>Alta de Datos</h3>
       <h:form>
          <p>Key:
            <h:inputText value="#{viewManager.item.key}" />
          </p>
          <p>Value:
            <h:inputText value="#{viewManager.item.value}" />
          </p>
          <p>
            <h:commandButton value="add" action="#{viewManager.add}" />
          </p>
       </h:form>
    </h:panelGroup>
    <h:panelGroup rendered="#{viewManager.edit}">
       <h3>Editar un item #{viewManager.item.key}</h3>
       <h:form>
          <p>Key:
             <h:inputText value="#{viewManager.item.key}" />
          </p>
          <p>Value:
             <h:inputText value="#{viewManager.item.value}" />
          </p>
          <p>
             <h:commandButton value="save" action="#{viewManager.save}" />
          </p>
       </h:form>
    </h:panelGroup>
    <h:form rendered="#{not empty viewManager.cacheList}">
            !--< Mostrar los datos en una tabla -->
    </h:form>
Hasta aqu�, vemos que se renderiza "No hay datos a�n", en caso que el miembro lista est� vac�o y pretender�amos que en ese caso
como no hay nada para editar por ausencia de items, se diera la posibilidad de dar de alta items, para ello, en la clase, el valor
de una _variable boolena edit_ deber�a ser _false_.

{:lang="java"}
    private boolena edit;
    // y el m�todo que se ejecuta cuando el valor del atributo
    // rendered en un componente es "#{viewManager.edit}"
    public boolean isEdit() {
      return edit;
    }
Mientras que si _edit_ es _true_ la renderizaci�n es del segundo formulario. Ambos admiten datos mediante un elemento h:inputText,
la diferencia est� en la acci�n a ejecutar tras el env�o del formulario, en caso de edici�n es _save_, cuyo c�digo es como sigue
{:lang="java"}
    public void save() {
      item = new Property();
      edit = false;
    }
Notamos que este m�todo ser� ejecutado una vez el usuario haya tenido la posibilidad de _editar un item_, para ello se prevee como
se ha indicado anteriormente un bot�n en lo que ser�a el tercer formulario de index.xhtml, que describimos m�s abajo.    
A la variable _edit_, se le asigna _true_ al ejecutar la acci�n del bot�n de edici�n, produciendo el efecto de la renderizaci�n del
formulario de edici�n, as� pues con la acci�n _save_, la edici�n ha terminado (se edit� y se grab�), entonces asignamos _false_ a _edit_,
para que sea renderizado la parte _insertar datos_   
El formulario que se encarga de renderizar la tabla con los datos y permitir la edici�n de un item, podr�a tener esta forma
{:lang="xhtml}
    <h:form rendered="#{not empty viewManager.cacheList}">
      <h:dataTable value="#{viewManager.cacheList}" var="item"
         <h:column>
           <f:facet name="header">Key</f:facet>
           <h:outputText value="#{item.key}" />
         </h:column>
         <h:column>
           <f:facet name="header">Value</f:facet>
           <h:outputText value="#{item.value}" />
         </h:column>
         <h:column>
           <h:commandButton value="edit" action="#{viewManager.edit(item)}" />
         </h:column>
         <h:column>
      </h:dataTable>
    </h:form>
Vemos que el m�todo de la clase a ejecutar tras el env�o del cliente, la acci�n del h:commandButton, recibir� un par�metro el cual
es un objeto item dentro de la tabla.   
En los estadios de la app desarrollados la clase anterior se opt� por mostrar los datos usando ui:repeat y h:outputText.    
Este es el c�digo del _m�todo edit_ de la clase,
{:lang="java"}
    public void edit(Property item) {
      this.item = item;
      edit = true;
    }
Por �ltimo podemos completar las acciones CRUD, agregando la posibilidad de _eliminar un item_, ser� definir en la clase un m�todo
_delete_, el cu�l ser� ejecutado mediante atributo action de un h:commandButton que a�adimos al �ltimo formulario,
{:lang="java"}
    public void delete(Property item) {
        cacheList.remove(item);
    }
Y en el formulario retocamos el elemento h:dataTable
{:lang="xhtml"}
    <h:dataTable value="#{viewManager.cacheList}" var="item"
      <h:column>
        <f:facet name="header">Key</f:facet>
        <h:outputText value="#{item.key}" />
      </h:column>
      <h:column>
        <f:facet name="header">Value</f:facet>
        <h:outputText value="#{item.value}" />
      </h:column>
      <h:column>
        <h:commandButton value="edit" action="#{viewManager.edit(item)}" />
      </h:column>
      <h:column>
        <h:commandButton value="delete"
                         action="#{viewManager.delete(item)}" />
      </h:column>
    </h:dataTable>
