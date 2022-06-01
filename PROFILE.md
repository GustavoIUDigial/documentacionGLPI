# GLPI
## _Metas_
Proporcione una forma para que el administrador segmente los usos en perfiles de usuarios.

## Perfiles

La [clase Perfil](https://forge.glpi-project.org/apidoc/class-Profile.html) (correspondiente a la `glpi_profilestabla`) almacena cada conjunto de derechos.
Un perfil tiene un conjunto de campos base independientes de los subderechos y, por lo tanto, podría:

- se definirá como predeterminado para nuevos usuarios ( `is_default` campo).
- forzar el formulario de creación de tickets en el inicio de sesión ( `create_ticket_on_login` campo).
- definir la interfaz utilizada ( `interface` campo):
    - servicio de asistencia (usuarios de autoservicio)
    - central (vista técnica)

## Definición de derechos

Están definidos por la [clase ProfileRight](https://forge.glpi-project.org/apidoc/class-ProfileRight.html) (correspondiente a la `glpi_profilerights` tabla)

Cada uno consta de:
- una clave foránea de perfil ( `profiles_id` campo)
- una clave ( `name` campo)
- un valor ( `right` campo)

Las claves coinciden con la propiedad estática `$rightname` en los tipos de elementos GLPI. Ej: En la clase de Computación, tenemos un `static $rightname = 'computer';`

El valor es una suma numérica de constantes enteras.

Los valores de los derechos estándar se pueden encontrar en inc/define.php:

```sh
<?php
...
define("READ", 1);
define("UPDATE", 2);
define("CREATE", 4);
define("DELETE", 8);
define("PURGE", 16);
define("ALLSTANDARDRIGHT", 31);
define("READNOTE", 32);
define("UPDATENOTE", 64);
define("UNLOCK", 128);
```
Entonces, por ejemplo, para tener derecho a LEER y ACTUALIZAR un tipo de elemento, tendremos un `right` valor de 3.

Como se define en este bloque anterior, tenemos un cálculo de todos los estándares a la derecha = 31:
```sh
READ (1)
\+ UPDATE (2)
\+ CREATE (4)
\+ DELETE (8)
\+ PURGE (16)
= 31
```
Si necesita ampliar los posibles valores de los derechos, debe declarar esta parte en su tipo de elemento, ejemplo simplificado de la clase Ticket:

```sh
<?php
class Ticket extends CommonITILObject {
   ...
   const READALL          =   1024;
   const READGROUP        =   2048;
   ...
   function getRights($interface = 'central') {
      $values = parent::getRights();
      $values[self::READGROUP]  = array('short' => __('See group ticket'),
                                        'long'  => __('See tickets created by my groups'));
      $values[self::READASSIGN] = array('short' => __('See assigned'),
                                        'long'  => __('See assigned tickets'));
      return $values;
   }
   ...
```
Los nuevos derechos deben ser verificados por sus propias funciones, consulte derechos de verificación

## Verificar derechos

Cada clase de tipo de elemento que hereda de CommonDBTM se beneficiará de las comprobaciones correctas estándar. Consulte los siguientes métodos:

- [puedo ver](https://forge.glpi-project.org/apidoc/class-CommonDBTM.html#_canView)
- [puede Actualizar](https://forge.glpi-project.org/apidoc/class-CommonDBTM.html#_canUpdate)
- [puede crear](https://forge.glpi-project.org/apidoc/class-CommonDBTM.html#_canCreate)
- [canDelete](https://forge.glpi-project.org/apidoc/class-CommonDBTM.html#_canDelete)
- [puedePurgar](https://forge.glpi-project.org/apidoc/class-CommonDBTM.html#_canPurge)

Si necesita contrastar un derecho específico `rightname` con un posible derecho, así es como se hace:

```sh
<?php
if (Session::haveRight(self::$rightname, CREATE)) {
   // OK
}
// we can also test a set multiple rights with AND operator
if (Session::haveRightsAnd(self::$rightname, [CREATE, READ])) {
   // OK
}
// also with OR operator
if (Session::haveRightsOr(self::$rightname, [CREATE, READ])) {
   // OK
}
// check a specific right (not your class one)
if (Session::haveRight('ticket', CREATE)) {
   // OK
}
```
Ver definición de métodos:
- [tiene derecho](https://forge.glpi-project.org/apidoc/class-Session.html#_haveRight)
- [tener Derechos AND](https://forge.glpi-project.org/apidoc/class-Session.html#_haveRightsAnd)
- [tener Derechos OR](https://forge.glpi-project.org/apidoc/class-Session.html#_haveRightsOr)

Todas las funciones anteriores devuelven un valor booleano. Si queremos un troquel elegante de sus páginas, tenemos una función equivalente pero con un `check` prefijo en su lugar `have`:

- [comprobar Derecho](https://forge.glpi-project.org/apidoc/class-Session.html#_checkRight)
- [comprobar Derechos AND](https://forge.glpi-project.org/apidoc/class-Session.html#_checkRightsAnd)
- [comprobar Derechos OR](https://forge.glpi-project.org/apidoc/class-Session.html#_checkRightsOr)

Si necesita marcar un derecho directamente en una consulta SQL, utilice bit a bit `&` y `|` operadores, ex para usuarios:

```sh
<?php
$query = "SELECT `glpi_profiles_users`.`users_id`
   FROM `glpi_profiles_users`
   INNER JOIN `glpi_profiles`
      ON (`glpi_profiles_users`.`profiles_id` = `glpi_profiles`.`id`)
   INNER JOIN `glpi_profilerights`
      ON (`glpi_profilerights`.`profiles_id` = `glpi_profiles`.`id`)
   WHERE `glpi_profilerights`.`name` = 'ticket'
      AND `glpi_profilerights`.`rights` & ". (READ | CREATE);
$result = $DB->query($query);
```
En este fragmento, realiza una [operación bit](http://php.net/manual/fr/language.operators.bitwise.php) a bit para obtener la suma de estos derechos y el [operador SQL realiza](https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html) una comparación lógica con el valor actual en la base de datos. `READ | CREATE &`

## Especificidades de CommonDBRelation y CommonDBChild

Estas clases permiten administrar la relación entre elementos y, por lo tanto, tienen propiedades para propagar derechos de sus padres.

```sh
<?php
abstract class CommonDBChild extends CommonDBConnexity {
   static public $checkParentRights = self::HAVE_SAME_RIGHT_ON_ITEM;
   ...
}
abstract class CommonDBRelation extends CommonDBConnexity {
   static public $checkItem_1_Rights = self::HAVE_SAME_RIGHT_ON_ITEM;
   static public $checkItem_2_Rights = self::HAVE_SAME_RIGHT_ON_ITEM;
   ...
}
```

Los valores posibles para estas propiedades son:

- `DONT_CHECK_ITEM_RIGHTS` : no verifique a los padres, siempre tenemos todos los derechos independientemente de los derechos de los padres.
- `HAVE_VIEW_RIGHT_ON_ITEM` : tenemos todos los derechos (CREAR, ACTUALIZAR), si podemos ver el padre.
- `HAVE_SAME_RIGHT_ON_ITEM` : tenemos los mismos derechos que la clase padre.

