## Updating
_Nuevo en la versión 9.3._
Al igual que las consultas SQL SELECT , debe evitar el SQL simple y utilizar los métodos proporcionados por la plataforma del objeto DB .
## General
Actualmente, el marco proporciona automáticamente el escape de datos para todos los datos pasados desde GET o POST ; no tienes que cuidarlos (esto cambiará en una versión futura). Debe tener cuidado de que se escapen los datos cuando usa valores que provienen de otro lugar.
La parte WHERE de los métodos UPDATE y DELETE utiliza las mismas capacidades de criterio que las consultas SELECT .
## Insertar una fila
Puede insertar una fila en la base de datos utilizando el método insert() :
```sh
<?php

$DB->insert(
   'glpi_my_table', [
      'a_field'      => 'My value',
      'other_field'  => 'Other value'
   ]
);
// => INSERT INTO `glpi_my_table` (`a_field`, `other_field`) VALUES ('My value', Other value)
```
También se proporciona un método insertOrDie() .
## Actualizar una fila
Puede actualizar filas en la base de datos utilizando el método update() :
```sh
<?php
$DB->update(
   'glpi_my_table', [
      'a_field'      => 'My value',
      'other_field'  => 'Other value'
   ], [
      'id' => 42
   ]
);
// => UPDATE `glpi_my_table` SET `a_field` = 'My value', `other_field` = 'Other value' WHERE `id` = 42
```
También se proporciona un método updateOrDie() .
_Nuevo en la versión 9.3.1._
Al emitir una consulta de ACTUALIZACIÓN , puede usar una cláusula ORDER y/o LIMIT junto con el lugar (que sigue siendo obligatorio ). Para lograr eso, use una matriz indexada con las claves apropiadas:
```sh
<?php
$DB->update(
   'my_table', [
      'my_field'  => 'my value'
   ], [
      'WHERE'  => ['field' => 'value'],
      'ORDER'  => ['date DESC', 'id ASC'],
      'LIMIT'  => 1
   ]
);
```
## Eliminando una fila
Puede eliminar filas de la base de datos utilizando el método delete() :
```sh 
<?php
$DB->delete(
   'glpi_my_table', [
      'id' => 42
   ]
);
// => DELETE FROM `glpi_my_table` WHERE `id` = 42
```
## Usar declaraciones preparadas
En algunos casos, es posible que desee utilizar declaraciones preparadas para mejorar el rendimiento. Para lograrlo, deberá crear una consulta con algunos parámetros (sin nombre, ya que mysqli no admite parámetros con nombre), luego prepararla y, finalmente, vincular los parámetros y ejecutar la instrucción.
Veamos un ejemplo con una declaración de inserción:
```sh
<?php
$insert_query = $DB->buildInsert(
   'my_table', [
      'field'  => new Queryparam(),
      'other'  => new Queryparam()
   ]
);
// => INSERT INTO `glpi_my_table` (`field`, `other`) VALUES (?, ?)
$stmt = $DB->prepare($insert_query);
foreach ($data as $row) {
   $stmt->bind_params(
      'ss',
      $row['field'],
      $row['other']
   );
   $stmt->execute();
}
```
Al igual que el método buildInsert() utilizado aquí, los métodos buildUpdate y buildDelete están disponibles. Toman exactamente los mismos argumentos que los métodos "no construidos".
|Nota|
|----|
|Tenga en cuenta el uso del objeto Queryparam . Esto se usa para que el constructor sepa que no está pasando un valor, sino un parámetro (que no debe escaparse ni entrecomillarse).|
Preparar una consulta SELECT es un poco diferente:
```sh
<?php
$it = new DBmysqlIterator();
$it->buildQuery([
   'FROM'   => 'my_table',
   'WHERE'  => [
      'something' => new Queryparam(),
      'foo'       => 'bar'
]);
$query = $it->getSql();
// => SELECT FROM `my_table` WHERE `something` = ? AND `foo` = 'bar'
$stmt = $DB->prepare($query);
// [...]
```