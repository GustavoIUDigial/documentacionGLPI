## Buscador
### Meta

La [clase de búsqueda](https://forge.glpi-project.org/apidoc/class-Search.html) tiene como objetivo proporcionar un motor de búsqueda de criterios múltiples para tipos de elementos GLPI.

Incluye algunas funciones de atajos:
- `show()`: muestra la página de búsqueda completa.
- `showGenericSearch()`: muestra solo el formulario multicriterio.
- `showList()`: muestra solo la lista resultante.
- `getDatas()`: devuelve una matriz de datos sin procesar.
- `manageParams()`: completa los `$_GET` valores con los `$_SESSION` valores.

La función show analiza los `$_GET` valores (llamando a `manageParams()`) pasados por la página para recuperar los criterios y construir la consulta SQL.
Para la función showList, los parámetros se pueden pasar en el segundo argumento.

Las clases de tipo de elemento pueden definir un conjunto de opciones de búsqueda para configurar qué columnas se pueden consultar, cómo se puede acceder y mostrar, etc.
|Que hacer|
| ------ |
| opción de campos de datos |
|diferencia entre unidad de búsqueda y unidad de retardo |
|traducciones desplegables|
|darItem|
|exportar|
|búsqueda de texto completo |

## Ejemplos
Para mostrar el motor de búsqueda con sus opciones predeterminadas (formulario de criterios, buscapersonas, lista):
```sh
<?php
$itemtype = 'Computer';
Search::show($itemtype);
```
Si desea mostrar solo el formulario multicriterio (con algunas opciones adicionales):
```sh
<?php
$itemtype = 'Computer';
$p = [
   'addhidden'   => [ // some hidden inputs added to the criteria form
      'hidden_input' => 'OK'
   ],
   'actionname'  => 'preview', //change the submit button name
   'actionvalue' => __('Preview'), //change the submit button label
];
Search::showGenericSearch($itemtype, $p);
```
Si desea mostrar solo una lista sin el formulario de criterios:
```sh
<?php
// display a list of users with entity = 'Root entity'
$itemtype = 'User';
$p = [
   'start'      => 0,      // start with first item (index 0)
   'is_deleted' => 0,      // item is not deleted
   'sort'       => 1,      // sort by name
   'order'      => 'DESC'  // sort direction
   'reset'      => 'reset',// reset search flag
   'criteria'   => [
      [
         'field'      => 80,        // field index in search options
         'searchtype' => 'equals',  // type of search
         'value'      => 0,         // value to search
      ],
   ],
];
Search::showList($itemtype, $p);
```
### OBTENER Parámetros
![Image text](https://github.com/GustavoIUDigial/documentacionGLPI/blob/main/images/search_criteria.png)
|Nota|
|----|
|GLPI guarda `$_SESSION['glpisearch'][$itemtype]` el último conjunto de parámetros para el tipo de elemento actual para cada consulta de búsqueda. Se restaura automáticamente en una nueva búsqueda si no se define `reset`, `criteria` o . `metacriteria`|

Aquí está la lista de posibles claves que podrían pasarse para controlar el motor de búsqueda. Todos son opcionales.
`criteria`
Una matriz multidimensional de criterios para filtrar la búsqueda. Cada matriz de criterios debe proporcionar:
- `link`: uno de los operadores lógicos _AND_ , _OR_ , _AND_ _NOT_ o _OR_ _NOT_ , opcional para el primer elemento,
- `field`: id de la opción de búsqueda,
- `searchtype`: tipo de búsqueda, una de:
- - `contains`
- - `equals`
- - `notequals`
- - `lessthan`
- - `morethan`
- - `under`
- - `notunder`
- `value`: el valor a buscar

|Nota|
|----|
|Para encontrar la `field` identificación que desea, puede echar un vistazo a la secuencia de comandos de la herramienta getsearchoptions.php .|

`metacriteria`
Muy similar al parámetro de criterio pero permite buscar en las opciones de búsqueda de un tipo de ítem vinculado al actual (los softwares de una computadora, por ejemplo).

No todos los tipos de elementos se pueden vincular, consulte el método getMetaItemtypeAvailable() de la clase de búsqueda para saber cuáles podrían ser.

El parámetro necesita las mismas claves que los criterios más una adicional:
- `itemtype` : segundo tipo de elemento para vincular.

`sort`
id de la opción de búsqueda por ordenar.
`order`
Ya sea `ASC` para finalizar la clasificación o `DESC` para finalizar la clasificación.
`start`
Un entero para indicar el punto de inicio de la paginación (SQL `OFFSET`).
`is_deleted`
Un valor booleano para mostrar papelera.
`reset`
Un valor booleano para restablecer los parámetros de búsqueda guardados, consulte la nota a continuación.
## Opciones de búsqueda
Cada tipo de elemento puede definir un conjunto de opciones para representar las columnas que el motor de búsqueda puede consultar/mostrar. Cada opción se identifica con un número entero único (debemos evitar conflictos).

Cambiado en la versión 9.2: la matriz de opciones de búsqueda se ha reescrito por completo; principalmente para capturar duplicados y agregar una prueba de unidad para evitar problemas futuros.

Para permitir el uso de sintaxis tanto antiguas como nuevas; se ha creado un nuevo método, `getSearchOptionsNew()`. La sintaxis antigua sigue siendo válida (pero no permite detectar duplicados).

¡El formato ha cambiado, pero no las opciones posibles y sus valores!
```sh
<?php
function getSearchOptionsNew() {
   $tab = [];
   $tab[] = [
      'id'                 => 'common',
      'name'               => __('Characteristics')
   ];
   $tab[] = [
      'id'                 => '1',
      'table'              => self::getTable(),
      'field'              => 'name',
      'name'               => __('Name'),
      'datatype'           => 'itemlink',
      'massiveaction'      => false
   ];
   ...
   return $tab;
}
```
|Nota|
|---|
|Como referencia, la forma antigua de escribir las mismas opciones de búsqueda era:|
```sh 
<?php
function getSearchOptions() {
   $tab                       = array();
   $tab['common']             = __('Characteristics');
   $tab[1]['table']           = self::getTable();
   $tab[1]['field']           = 'name';
   $tab[1]['name']            = __('Name');
   $tab[1]['datatype']        = 'itemlink';
   $tab[1]['massiveaction']   = false;
   ...
   return $tab;
}
```
Cada opción debe definir las siguientes claves:

`table`
La tabla SQL donde `field` se puede encontrar la clave.

`field`
La columna SQL a consultar.

`name`
Una etiqueta utilizada para mostrar la opción de búsqueda en las páginas de búsqueda (como el encabezado, por ejemplo).

#### Opcionalmente, se pueden definir las siguientes claves:

`linkfield`
_Clave externa utilizada para unirse a la tabla de tipo de elemento actual.
Si no está vacío, la acción masiva estándar (función de actualización) para esta opción de búsqueda será imposible_

`searchtype`
Una cadena o una matriz que contiene el tipo de búsqueda forzada:
- `equals` (puede forzar el uso de campo en lugar de id al agregar la 
- `searchequalsonfield` opción)
- `contains`

`forcegroupby`
Un valor booleano para forzar el grupo en esta opción de búsqueda

`splititems`
Usar `<hr>` en lugar de `<br>` para dividir elementos agrupados

`usehaving`
Use `HAVINGl` a cláusula SQL en lugar de `WHERE` en la consulta SQL

`massiveaction`
Establézcalo en falso para deshabilitar las acciones masivas para esta opción de búsqueda.

`nosort`
Establézcalo en verdadero para deshabilitar la clasificación con esta opción de búsqueda .

`nosearch`
Establézcalo en verdadero para deshabilitar la búsqueda en esta opción de búsqueda .

`nodisplay`
Establézcalo en verdadero para deshabilitar la visualización de esta opción de búsqueda .

`joinparams`
Define cómo se debe realizar la unión SQL. Consulte el párrafo sobre joinparams continuación.

`additionalfields`
Una matriz de campos adicionales para agregar en la `S ELECT` cláusula. Por ejemplo:`'additionalfields' => ['id', 'content', 'status']`

`datatype`
Defina cómo se mostrará la opción de búsqueda y si se necesita usar un control para la modificación (por ejemplo, selector de fecha para la fecha) y afectar el menú desplegable del tipo de búsqueda.
Los parámetros opcionales se agregan a la matriz base de la opción de búsqueda para controlar más exactamente el tipo de datos.

Consulte el párrafo sobre tipos de datos a continuación.
## Parámetros de unión
Para definir parámetros de unión, puede utilizar uno o más de los siguientes:

`beforejoin`

Defina qué tablas deben unirse para acceder al campo.

La matriz contiene `table`clave y puede contener un archivo `joinparams`.
En el caso de anidado `beforejoin`, comenzamos la unión SQL desde la última dimensión.

Ejemplo:
```sh
<?php
[
   'beforejoin' => [
      'table'        => 'mytable',
      'joinparams'   => [
         'beforejoin' => [...]
      ]
   ]
]
```
` jointype`
Defina el tipo de unión:
- `emptypara` un tipo de unión estándar::
```sh
REFTABLE.`#linkfield#` = NEWTABLE.`id`
```
- `childpara` una mesa infantil::
```sh
REFTABLE.`id` = NEWTABLE.`#linkfield#`
```
- `itemtype` itempara enlaces usando `itemtype` y `items_id` campos en nueva tabla:
```sh 
REFTABLE.`id` = NEWTABLE.`items_id`
AND NEWTABLE.`itemtype` = '#ref_table_itemtype#'
```
- `itemtype_item_revert` (desde 9.2.1) para enlaces que usan `itemtype` y `items_id` campos en la tabla de referencia:
```sh
NEWTABLE.`id` = REFTABLE.`items_id`
AND REFTABLE.`itemtype` = '#new_table_itemtype#'
```
- `mainitemtype_mainitem` Igual que `itemtype_item` pero usando los campos mainitemtype y mainitems_id::
```sh 
REFTABLE.`id` = NEWTABLE.`mainitems_id`
AND NEWTABLE.`mainitemtype` = 'new table itemtype'
```
- `itemtypeonly` igual que `itemtype_item` jointype pero sin vincular id::
```sh
NEWTABLE.`itemtype` = '#new_table_itemtype#'
```
- `item_item` para la tabla utilizada para vincular dos elementos similares: `glpi_tickets_tickets` por ejemplo: los campos de vínculo son `standardfk_1` y `standardfk_2`::
```sh 
REFTABLE.`id` = NEWTABLE.`#fk_for_new_table#_1`
OR REFTABLE.`id` = NEWTABLE.`#fk_for_new_table#_2`
```
- `item_item_revert` igual que `item_item` y child jointtypes::
```sh 
NEWTABLE.`id` = REFTABLE.`#fk_for_new_table#_1`
OR NEWTABLE.`id` = REFTABLE.`#fk_for_new_table#_2`
```
`condition`
Condición adicional para agregar al enlace estándar.
Use `NEWTABLE` o `REFTABLE` etiquete para usar los nombres de las tablas.
Cambiado en la versión 9.4.
Una matriz de parámetros utilizados para crear una cláusula _WHERE_ a partir de las funciones de consulta de GLPI . Anteriormente era sólo una cadena.

`nolink`
Establézcalo en verdadero para indicar que la unión actual no se vincula a la unión/desde anterior (anidada `joinparams`)

## Tipos de datos
Los tipos de datos disponibles para la búsqueda son:
`date`
Parámetros disponibles (todos opcionales):

- `searchunit`: una de las unidades MySQL DATE_ADD , por defecto `MONTH`
- `maybefuture`: muestra el selector de fecha con la selección de fecha futura, por defecto es `false`
- `emptylabel`: cadena para mostrar en caso de `null` valor

`datetime`
- Los parámetros disponibles (todos opcionales) son los mismos que `date`.

`date_delay`
Fecha con retraso en el mes ( `end_warrant`y, `end_date`).
Los parámetros disponibles (todos opcionales) son los mismos que `date` y:

- `datafields`: matriz de campos de datos que se utilizarían.
- - `datafields[1]`: el campo de fecha,
- - `datafields[2]`: el campo de retraso,
- - `datafields[2]`: ?
- `delay_unit`: una de las unidades MySQL DATE_ADD , por defecto `MONTH`

`timestamp`
Use `Dropdown::showTimeStamp()` para modificar
Parámetros disponibles (todos opcionales):
- `withseconds`: booleano ( falsepor defecto)

`weblink`
Cualquier URL

`email`
Cualquier dirección de correo electrónico

`color`
Usar `Html::showColorField()` para modificar

`text`
texto sencillo

`string`
Use un editor de texto enriquecido para la modificación

`ip`
Cualquier dirección IP

`mac`
Parámetros disponibles (todos opcionales):
- `htmltext`: booleano, escapar del valor ( `false` por defecto)

`number`
Use a `Dropdown::showNumber()` para modificación (en caso de `equals` `searchtype`).
Para `contains searchtype`, puede usar el prefijo < y > `value` en .
Parámetros disponibles (todos opcionales):

- `width`: atributo html pasado a Dropdown::showNumber()
- `min`: valor mínimo (predeterminado `0`)
- `max`: valor máximo (predeterminado `100`)
- `step`: paso para seleccionar (predeterminado `1`)
- `toadd`: matriz de valores para agregar al comienzo del menú desplegable

`integer`
Alias para `numbe`

`count`
Igual que `number` pero cuenta el número de artículos en la tabla

`decimal`
Igual que `number` pero formateado con decimal

`bool`
Usar `Dropdown::showYesNo()` para modificar

`itemlink`
Crear un enlace al elemento

`itemtypename`
Usar `Dropdown::showItemTypes()` para modificar
Parámetros disponibles (todos opcionales) para definir los tipos de elementos disponibles:
- ´itemtype_list`: uno de [$CFG_GLPI[“unicity_types”]](https://github.com/glpi-project/glpi/blob/9.1.2/config/define.php#L166)
- `types`: matriz que contiene los tipos disponibles

`language`
Usar `Dropdown::showLanguages()` para modificar
Parámetros disponibles (todos opcionales):

- `display_emptychoice`: muestra una opción vacía ( `-------`)

`right`
Usar `Profile::dropdownRights()` para modificar
Parámetros disponibles (todos opcionales):

- `nonone` : ocultar ninguna opción ? (predeterminado en `false`)
- `noread`: ¿ocultar opción de lectura? (predeterminado en `false`)
- `nowrite`: ¿ocultar opción de escritura? (predeterminado en `false`)

`dropdown`
Uso `Itemtype::dropdown()` para modificación.
El menú desplegable puede tener varios parámetros adicionales según el tipo de menú desplegable: `right` para el usuario uno, por ejemplo

`specific`

Si ninguna de las opciones anteriores coincide con la forma en que desea mostrar su campo, puede usar este tipo de datos.
Consulte el párrafo de opciones de búsqueda específicas para su implementación.

## Opciones de búsqueda específicas
Es posible que desee controlar cómo seleccionar y mostrar su campo en una opción de búsqueda.
Debe configurar 'tipo de datos' => 'específico' en su opción de búsqueda y declarar estos métodos en su clase:

`getSpecificValueToDisplay`
Defina cómo mostrar el campo en la lista.
Parámetros:
- `$field`: nombre de columna, coincide con la clave de 'campo' de sus opciones de búsqueda
- `$values`: todos los valores de la fila actual (para seleccionar)
- `$options`: will contiene estas claves:
- - `html`,
- - `searchopt`: la opción de búsqueda completa actual

`getSpecificValueToSelect`
Defina cómo mostrar la entrada de campo en el formulario de criterios y acción masiva.
Parámetros:
- `$field`: nombre de columna, coincide con la clave de 'campo' de sus opciones de búsqueda
- `$values`: el valor de criterio actual pasado en los parámetros $_GET
- `$name`: el nombre del atributo html para que se muestre la entrada
- `$options`: esta matriz puede variar mucho en función de la opción de búsqueda o de la visualización masiva de acciones o criterios. Revisa los archivos correspondientes:
-  - [valoropciónbúsqueda.php](https://github.com/glpi-project/glpi/blob/ee667a059eb9c9a57c6b3ae8309e51ca99a5eeaf/ajax/searchoptionvalue.php#L128l)
-  - [acción masiva.clase.php](https://github.com/glpi-project/glpi/blob/ee667a059eb9c9a57c6b3ae8309e51ca99a5eeaf/inc/massiveaction.class.php#L881)

Ejemplo simplificado extraído de [CommonItilObject Class](https://forge.glpi-project.org/apidoc/class-CommonITILObject.html) para `glpi_tickets.status` el campo:

```sh 
<?php
function getSearchOptionsMain() {
   $tab = [];
   ...
   $tab[] = [
      'id'          => '12',
      'table'       => $this->getTable(),
      'field'       => 'status',
      'name'        => __('Status'),
      'searchtype'  => 'equals',
      'datatype'    => 'specific'
   ];
   ...
   return $tab;
}
static function getSpecificValueToDisplay($field, $values, array $options=array()) {
   if (!is_array($values)) {
      $values = array($field => $values);
   }
   switch ($field) {
      case 'status':
         return self::getStatus($values[$field]);
      ...
   }
   return parent::getSpecificValueToDisplay($field, $values, $options);
}
static function getSpecificValueToSelect($field, $name='', $values='', array $options=array()) {
   if (!is_array($values)) {
      $values = array($field => $values);
   }
   $options['display'] = false;
   switch ($field) {
      case 'status' :
         $options['name']  = $name;
         $options['value'] = $values[$field];
         return self::dropdownStatus($options);
      ...
   }
   return parent::getSpecificValueToSelect($field, $name, $values, $options);
}
```
## Selección predeterminada/Dónde/Unirse
La clase de búsqueda implementa tres métodos que agregan algunas cosas a las consultas SQL antes del cálculo de las opciones de búsqueda.
Para algún tipo de elemento, necesitamos filtrar la consulta o campos adicionales.
Por ejemplo, filtrar los tickets que no puede ver si no tiene los derechos adecuados.

GLPI llamará automáticamente a métodos predefinidos en los que puede confiar desde su `hook.php` archivo de complemento.

### addDefaultSelect
Consulte [la documentación del método addDefaultSelect()](https://forge.glpi-project.org/apidoc/class-Search.html#_addDefaultSelect)
`hook.php` Y en el archivo del complemento :
```sh 
<?php
function plugin_mypluginname_addDefaultSelect($itemtype) {
   switch ($type) {
      case 'MyItemtype':
         return "`mytable`.`myfield` = 'myvalue' AS MYNAME, ";
   }
   return '';
}
```
### addDefaultDónde
Consulte [la documentación del método addDefaultwhere()](https://forge.glpi-project.org/apidoc/class-Search.html#_addDefaultWhere)
`hook.php` Y en el archivo del complemento :
```sh
<?php
function plugin_mypluginname_addDefaultJoin($itemtype, $ref_table, &$already_link_tables) {
   switch ($itemtype) {
      case 'MyItemtype':
         return Search::addLeftJoin(
            $itemtype,
            $ref_table,
            $already_link_tables,
            'newtable',
            'linkfield'
         );
   }
   return '';
}
```
### agregarPredeterminadoUnirse
Consulte [la documentación del método addDefaultJoin()](https://forge.glpi-project.org/apidoc/class-Search.html#_addDefaultJoin)
`hook.php` Y en el archivo del complemento :
```sh 
<?php
function plugin_mypluginname_addDefaultWhere($itemtype) {
   switch ($itemtype) {
      case 'MyItemtype':
         return " `mytable`.`myfield` = 'myvalue' ";
   }
   return '';
}
```
## Marcadores
La `glpi_boomarks` tabla almacena una lista de consultas de búsqueda para los usuarios y permite recuperarlas.
El `query`campo contiene una construcción de consulta de URL a partir de parámetros con la función PHP [http_build_query](https://www.php.net/manual/en/function.http-build-query.php)

## Preferencias de visualización
La `glpi_displaypreferences` tabla almacena la lista de columnas predeterminadas que deben mostrarse a un usuario para un tipo de elemento.

Un conjunto de preferencias puede ser personal o global ( ). Si un usuario no tiene preferencias personales para un tipo de elemento, el motor de búsqueda utilizará las preferencias `globales.users_id = 0`