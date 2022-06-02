## Acciones Masivas

# Metas
Agregar a las listas de búsqueda de tipos de elementos :

- una casilla de verificación antes de cada elemento,
- una casilla de verificación para seleccionar todas las casillas de elementos,
- un botón Acciones para aplicar modificaciones a cada elemento seleccionado.

## Actualizar los campos del artículo
La primera opción del `Actions` botón es `Update`. Permite modificar el contenido de los campos de los ítems seleccionados.

La lista de campos que se muestran en la sublista depende de las opciones de búsqueda del tipo de elemento actual. De forma predeterminada, todas las opciones de búsqueda se muestran automáticamente en esta lista. Para prohibir esta visualización para un campo, debe definir la clave `massiveaction` como falsa en la declaración de opciones de búsqueda , por ejemplo:
```sh 
<?php
$tab[] = [
   'id'            => '1',
   'table'         => self::getTable(),
   'field'         => 'name',
   'name'          => __('Name'),
   'datatype'      => 'itemlink',
   'massiveaction' => false // <- NO MASSIVE ACTION
];
```
## Acciones masivas específicas
Después de la `Update` entrada, podemos declarar acciones masivas específicas adicionales para nuestro tipo de elemento actual.

Primero, necesitamos declarar en nuestra clase un `getSpecificMassiveActions` método que contenga nuestras definiciones de acciones masivas:
```sh 
<?php
...
function getSpecificMassiveActions($checkitem=NULL) {
   $actions = parent::getSpecificMassiveActions($checkitem);
   // add a single massive action
   $class        = __CLASS__;
   $action_key   = "myaction_key";
   $action_label = "My new massive action";
   $actions[$class.MassiveAction::CLASS_ACTION_SEPARATOR.$action_key] = $action_label;
   return $actions;
}
```
Una sola declaración se define por estas partes:
- a `classname`
- un `separator`: `siempreMassiveAction::CLASS_ACTION_SEPARATOR`
- a `key`
- y un `label`

Podemos tener varias acciones para la misma clase y podemos apuntar a una clase diferente de nuestro objeto actual.
A continuación, para mostrar la forma de nuestras definiciones, necesitamos declarar un showMassiveActionsSubFormmétodo:
```sh
<?php
...
static function showMassiveActionsSubForm(MassiveAction $ma) {
   switch ($ma->getAction()) {
      case 'myaction_key':
         echo __("fill the input");
         echo Html::input('myinput');
         echo Html::submit(__('Do it'), array('name' => 'massiveaction'))."</span>";

         break;
   }

   return parent::showMassiveActionsSubForm($ma);
}
```
Finalmente, para procesar nuestra definición, necesitamos un `processMassiveActionsForOneItemtype` método:
```sh
<?php
...
static function processMassiveActionsForOneItemtype(MassiveAction $ma, CommonDBTM $item,
                                                    array $ids) {
   switch ($ma->getAction()) {
      case 'myaction_key':
         $input = $ma->getInput();
         foreach ($ids as $id) {
            if ($item->getFromDB($id)
                && $item->doIt($input)) {
               $ma->itemDone($item->getType(), $id, MassiveAction::ACTION_OK);
            } else {
               $ma->itemDone($item->getType(), $id, MassiveAction::ACTION_KO);
               $ma->addMessage(__("Something went wrong"));
            }
         }
         return;
   }
   parent::processMassiveActionsForOneItemtype($ma, $item, $ids);
}
```
Además de una instancia de la clase MassiveAction `$ma` , también tenemos una instancia de la actual .`itemtype` `$item and the list of selected id ``$ids` 
En este método, podríamos usar algunas funciones de utilidad opcionales del objeto proporcionado en el parámetro: `MassiveAction $ma`

- `itemDone`, indica el resultado de la corriente `$id`, ver constantes de la clase MassiveAction . Si perdemos esta llamada, la corriente `$id` aún se considerará correcta.
- `addMessage`, una cadena para enviar al usuario para explicar el resultado al procesar el actual `$id`