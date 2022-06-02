
## Querying
El marco GLPI proporciona un generador de solicitudes simple:
- sin tener que escribir SQL
- sin tener que citar el nombre de la tabla y el campo
- sin tener que ocuparse de liberar recursos
- iterable
- contable

## Uso básico
```sh
<?php
foreach ($DB->request(...) as $id => $row) {
   //... work on each row ...
}
$req = $DB->request(...);
if ($row = $req->next()) {
  // ... work on a single row
}
$req = $DB->request(...);
if (count($req)) {
  // ... work on result
}
```
## Argumentos
El requestmétodo toma dos argumentos:
- nombre(s) de la tabla : una cadena o una matriz de cadenas (opcional cuando se da como FROMopción)
- opción(es) : matriz de opciones

## Dando declaración SQL completa
Si la única opción es una instrucción SQL completa, se utilizará. Este uso está en desuso y debe evitarse cuando sea posible.

|Nota|
|----|
|Para realizar una consulta a la base de datos que no se pudo realizar de la forma recomendada (llamando a funciones SQL como NOW(), ADD_DATE(), ... por ejemplo), puede hacer lo siguiente:|
```sh
<?php
$DB->request('SELECT id FROM glpi_users WHERE end_date > NOW()');
```
## Sin opción
En este caso, se iteran todos los datos de la tabla seleccionada:
```sh
<?php
$DB->request(['FROM' => 'glpi_computers']);
// => SELECT * FROM `glpi_computers`
$DB->request('glpi_computers');
// => SELECT * FROM `glpi_computers`
```
## Selección de campos
Puede utilizar las opciones `SELECT` o , es posible que se especifique una opción adicional.`FIEL` `DSDISTINCT`

|Nota|
|----|
|_Cambiado en la versión 9.5.0._|
|El uso de opciones o está en desuso. `DISTINCTFIELDS` `SELECT DISTINCT`|
```sh
<?php
$DB->request(['SELECT' => 'id', 'FROM' => 'glpi_computers']);
// => SELECT `id` FROM `glpi_computers`
$DB->request('glpi_computers', ['FIELDS' => 'id']);
// => SELECT `id` FROM `glpi_computers`
$DB->request(['SELECT' => 'name', 'DISTINCT' => true, 'FROM' => 'glpi_computers']);
// => SELECT DISTINCT `name` FROM `glpi_computers`
$DB->request('glpi_computers', ['FIELDS' => 'name', 'DISTINCT' => true]);
// => SELECT DISTINCT `name` FROM `glpi_computers`
```
La matriz de campos también puede contener una submatriz por tabla:
```sh
<?php
$DB->request('glpi_computers', ['FIELDS' => ['glpi_computers' => ['id', 'name']]]);
// => SELECT `glpi_computers`.`id`, `glpi_computers`.`name` FROM `glpi_computers`"
```
## Uso de JOIN
Debe usar criterios, generalmente un `FKEY` para describir cómo unir las tablas.
|Nota|
|----|
|_Nuevo en la versión 9.3.1._|
|La `ON` palabra clave también se puede utilizar como un alias de `FKEY`.|
## Múltiples tablas, combinación nativa
Debe usar criterios, generalmente a `FKEY` (o el `ON` equivalente), para describir cómo unir las tablas:
```sh
<?php
$DB->request(['FROM' => ['glpi_computers', 'glpi_computerdisks'],
              'FKEY' => ['glpi_computers'=>'id',
                         'glpi_computerdisks'=>'computer_id']]);
$DB->request(['glpi_computers', 'glpi_computerdisks'],
             ['FKEY' => ['glpi_computers'=>'id',
                         'glpi_computerdisks'=>'computer_id']]);
// => SELECT * FROM `glpi_computers`, `glpi_computerdisks`
//       WHERE `glpi_computers`.`id` = `glpi_computerdisks`.`computer_id`
```
## Unirse a la izquierda
Utilizando la opción, con algún criterio, normalmente un (o el equivalente): `LEFT JOIN` `FKEY` `ON`
```sh
<?php
$DB->request(['FROM'      => 'glpi_computers',
              'LEFT JOIN' => ['glpi_computerdisks' => ['FKEY' => ['glpi_computers'     => 'id',
                                                                  'glpi_computerdisks' => 'computer_id']]]]);
// => SELECT * FROM `glpi_computers`
//       LEFT JOIN `glpi_computerdisks`
//         ON (`glpi_computers`.`id` = `glpi_computerdisks`.`computer_id`)
```
## Unir internamente
Utilizando la opción, con algún criterio, normalmente un (o el equivalente): `INNER  JOIN` `FKEY` `ON`
```sh
<?php
$DB->request(['FROM'       => 'glpi_computers',
              'INNER JOIN' => ['glpi_computerdisks' => ['FKEY' => ['glpi_computers'     => 'id',
                                                                   'glpi_computerdisks' => 'computer_id']]]]);
// => SELECT * FROM `glpi_computers`
//       INNER JOIN `glpi_computerdisks`
//         ON (`glpi_computers`.`id` = `glpi_computerdisks`.`computer_id`)
```
## Unirse a la derecha
Utilizando la opción, con algún criterio, normalmente un (o el equivalente):`RIGHT  JOIN` `FKEY` `ON`
```sh
<?php
$DB->request(['FROM'       => 'glpi_computers',
              'RIGHT JOIN' => ['glpi_computerdisks' => ['FKEY' => ['glpi_computers'     => 'id',
                                                                   'glpi_computerdisks' => 'computer_id']]]]);
// => SELECT * FROM `glpi_computers`
//       RIGHT JOIN `glpi_computerdisks`
//         ON (`glpi_computers`.`id` = `glpi_computerdisks`.`computer_id`)
```
## Criterio de unión
_Nuevo en la versión 9.3.1._
También es posible agregar un criterio adicional para cualquier cláusula JOIN . Debe pasar una matriz con la primera clave igual a `AND` o `OR` y cualquier criterio válido del iterador:
```sh
<?php
$DB->request([
   'FROM'       => 'glpi_computers',
   'INNER JOIN' => [
      'glpi_computerdisks' => [
         'FKEY' => [
            'glpi_computers'     => 'id',
            'glpi_computerdisks' => 'computer_id',
            ['OR' => ['glpi_computers.field' => ['>', 42]]]
         ]
      ]
   ]
]);
// => SELECT * FROM `glpi_computers`
//       INNER JOIN `glpi_computerdisks`
//         ON (`glpi_computers`.`id` = `glpi_computerdisks`.`computer_id` OR
//              `glpi_computers`.`field` > '42'
//            )
```
## Consultas de la UNIÓN
_Nuevo en la versión 9.4.0._
Una consulta de unión es un objeto que contiene una matriz de subconsultas . Solo tiene que dar una lista de subconsultas que ya ha preparado, o matrices de parámetros que se utilizarán para construirlas.
```sh 
<?php
$sub1 = new \QuerySubQuery([
   'SELECT' => 'field1 AS myfield',
   'FROM'   => 'table1'
]);
$sub2 = new \QuerySubQuery([
   'SELECT' => 'field2 AS myfield',
   'FROM'   => 'table2'
]);
$union = new \QueryUnion([$sub1, $sub2]);
$DB->request([
   'FROM'       => $union
]);
// => SELECT * FROM (
//       SELECT `field1` AS `myfield` FROM `table1`
//       UNION ALL
//       SELECT `field2` AS `myfield` FROM `table2`
//    )
```
Como puede ver en el ejemplo anterior, se crea una consulta. Si desea que sus resultados sean deduplicados, (estándar ):`UNION` `ALLUNION`
```sh 
<?php
 //...
 //passing true as second argument will activate deduplication.
 $union = new \QueryUnion([$sub1, $sub2], true);
 //...
```
|Advertencia|
|-----------|
|Tenga en cuenta que la deduplicación de una consulta UNION puede tener un costo enorme en el servidor de la base de datos.|
|La mayoría de las veces, puede emitir y desduplicar en el código. `UNION ALL`|
## Contando
Usando la `COUNT` opción:
```sh 
<?php
$DB->request(['FROM' => 'glpi_computers', 'COUNT' => 'cpt']);
// => SELECT COUNT(*) AS cpt FROM `glpi_computers`
```
## Agrupamiento
Usando la `GROUPBY` opción, que contiene un nombre de campo o una matriz de nombres de campo.
```sh
<?php
$DB->request(['FROM' => 'glpi_computers', 'GROUPBY' => 'name']);
// => SELECT * FROM `glpi_computers` GROUP BY `name`
$DB->request('glpi_computers', ['GROUPBY' => ['name', 'states_id']]);
// => SELECT * FROM `glpi_computers` GROUP BY `name`, `states_id`
```
## Ordenar
Usando la `ORDER` opción, con valor un campo o una matriz de campos. El nombre del campo también puede contener el sufijo ASC o DESC.
```sh 
<?php
$DB->request(['FROM' => 'glpi_computers', 'ORDER' => 'name']);
// => SELECT * FROM `glpi_computers` ORDER BY `name`
$DB->request('glpi_computers', ['ORDER' => ['date_mod DESC', 'name ASC']]);
// => SELECT * FROM `glpi_computers` ORDER BY `date_mod` DESC, `name` ASC
```
## Solicitar localizador
Usando las opciones `START` y : `LIMIT`
```sh
<?php
$DB->request('glpi_computers', ['START' => 5, 'LIMIT' => 10]);
// => SELECT * FROM `glpi_computers` LIMIT 10 OFFSET 5"
```
## Criterios
Otras opciones se consideran como una matriz de criterios (lógicos implícitos `AND`)
También `WHERE` se puede utilizar para la legibilidad.

## Criterios simples
Un nombre de campo y su valor deseado:
```sh
<?php
$DB->request(['FROM' => 'glpi_computers', 'WHERE' => ['is_deleted' => 0]]);
// => SELECT * FROM `glpi_computers` WHERE `is_deleted` = 0
$DB->request('glpi_computers', ['is_deleted' => 0,
                                'name'       => 'foo']);
// => SELECT * FROM `glpi_computers` WHERE `is_deleted` = 0 AND `name` = 'foo'
$DB->request('glpi_computers', ['users_id' => [1,5,7]]);
// => SELECT * FROM `glpi_computers` WHERE `users_id` IN (1, 5, 7)
```
## lógico OR, AND,NOT
Usando la opción , o con una serie de `OR` criterios `AND`:`NOT`
```sh
<?php
$DB->request('glpi_computers', ['OR' => ['is_deleted' => 0,
                                         'name'       => 'foo']]);
// => SELECT * FROM `glpi_computers` WHERE (`is_deleted` = 0 OR `name` = 'foo')"

$DB->request('glpi_computers', ['NOT' => ['id' => [1,2,7]]]);
// => SELECT * FROM `glpi_computers` WHERE NOT (`id` IN (1, 2, 7))
```
Usando una expresión más compleja con `AND` y `OR`:
```sh
<?php
$DB->request('glpi_computers', ['is_deleted' => 0,
    ['OR' => ['name' => 'foo', 'otherserial' => 'otherunique']],
    ['OR' => ['locations_id' => 1, 'serial' => 'unique']]
]);
// => SELECT * FROM `glpi_computers` WHERE `is_deleted` = '0' AND ((`name` = 'foo' OR `otherserial` = 'otherunique')) AND ((`locations_id` = '1' OR `serial` = 'unique'))
```
##Operadores
El operador predeterminado es `=`, pero se pueden usar otros operadores, proporcionando una matriz que contenga el operador y el valor.
```sh
<?php
$DB->request('glpi_computers', ['date_mod' => ['>' , '2016-10-01']]);
// => SELECT * FROM `glpi_computers` WHERE `date_mod` > '2016-10-01'
$DB->request('glpi_computers', ['name' => ['LIKE' , 'pc00%']]);
// => SELECT * FROM `glpi_computers` WHERE `name` LIKE 'pc00%'
```
Los operadores conocidos son `=`, `!=`, `<`, `<=`, `>`, `>=`, `LIKE`, `REGEXP`, , , (BITS Y) y (BITS O).`NOT LIKE` `NOT REGEX` &`|`
## Alias
Puede utilizar alias de SQL ( `AS` palabra clave de SQL). Para lograrlo, simplemente escriba el alias que desee en el nombre de la tabla o en el nombre del campo; luego úsalo en tus parámetros:
```sh
<?php
$DB->request('glpi_computers AS c');
// => SELECT * FROM `glpi_computers` AS `c`
$DB->request(['SELECT' => 'field AS f', 'FROM' => 'glpi_computers AS c']);
// => SELECT `field` AS `f` FROM `glpi_computers` AS `c`
```
## Funciones agregadas
_Nuevo en la versión 9.3.1._
Puede utilizar algunas funciones SQL de agregación en los campos: , `COUNT`, `SUM` y `AVG` son compatibles. Simplemente configure la función como la clave en su matriz de campos:`MIN` `MAX`
```sh 
<?php
$DB->request(['SELECT' => ['COUNT' => 'field', 'bar'], 'FROM' => 'glpi_computers', 'GROUPBY' => 'field']);
// => SELECT COUNT(`field`), `bar` FROM `glpi_computers` GROUP BY `field`
$DB->request(['SELECT' => ['bar', 'SUM' => 'amount AS total'], 'FROM' => 'glpi_computers', 'GROUPBY' => 'amount']);
// => SELECT `bar`, SUM(`amount`) AS `total` FROM `glpi_computers` GROUP BY `amount`
```
## Subconsultas
_Nuevo en la versión 9.3.1._
Puede usar subconsultas, usando la clase QuerySubQuery específica . Se necesitan dos argumentos: el primero es una matriz de criterios para generar la consulta y el segundo es un operador opcional para usar. Los operadores permitidos son los mismos que se documentan a continuación más IN y NOT IN . El operador predeterminado es IN .
```sh 
<?php
$sub_query = new \QuerySubQuery([
   'SELECT' => 'id',
   'FROM'   => 'subtable',
   'WHERE'  => [
      'subfield' => 'subvalue'
   ]
]);
$DB->request(['FROM' => 'glpi_computers', 'WHERE' => ['field' => $sub_query]]);
// => SELECT * FROM `glpi_computers` WHERE `field` IN (SELECT `id` FROM `subtable` WHERE `subfield` = 'subvalue')
$sub_query = new \QuerySubQuery([
   'SELECT' => 'id',
   'FROM'   => 'subtable',
   'WHERE'  => [
      'subfield' => 'subvalue'
   ]
]);
$DB->request(['FROM' => 'glpi_computers', 'WHERE' => ['NOT' => ['field' => $sub_query]]]);
// => SELECT * FROM `glpi_computers` WHERE NOT `field` IN (SELECT `id` FROM `subtable` WHERE `subfield` = 'subvalue')
$sub_query = new \QuerySubQuery([
   'SELECT' => 'id',
   'FROM'   => 'subtable',
   'WHERE'  => [
      'subfield' => 'subvalue'
   ]
], 'myalias');
$DB->request(['FROM' => 'glpi_computers', 'SELECT' => [$sub_query, 'id']]);
// => SELECT (SELECT `id` FROM `subtable` WHERE `subfield` = 'subvalue') AS `myalias`, id FROM `glpi_computers`
```
## ¿Qué sucede si el iterador no proporciona lo que estoy buscando?
Incluso si hacemos todo lo posible para implementar tantas cosas como sea posible en el iterador, hay varias cosas que faltan... Considere, por ejemplo, que quiere usar la función SQL NOW() , o quiere usar un valor basado en otro campo : no hay una forma nativa de lograr eso.
En este momento, hay una clase QueryExpression que permitiría hacer tales cosas en los valores (y no en los campos, ya que no es posible usar una instancia de clase como clave de matriz).
|Advertencia|
|-----------|
|La clase QueryExpression pasará SQL sin formato. ¡Usted está a cargo de escapar del nombre y los valores que usa en él!|
Por ejemplo, para usar la función SQL NOW() :
```sh
<?php
$DB->request([
   'FROM'   => 'my_table',
   'WHERE'  => [
      'date_end'  => ['>', new \QueryExpression('NOW()')]
   ]
]);
// SELECT * FROM `my_table` WHERE `date_end` > NOW()
```
Otro ejemplo con un valor de campo:
```sh
<?php
$DB->request([
   'FROM'   => 'my_table',
   'WHERE'  => [
      'field'  => new \QueryExpression(DBmysql::quoteName('other_field'))
   ]
]);
// SELECT * FROM `my_table` WHERE `field` = `other_field`
```
_Nuevo en la versión 9.3.1._
También puede usar alguna función o cosas no admitidas en la parte del campo usando una entrada RAW en la consulta:
```sh
<?php
$DB->request([
   'FROM'   => 'my_table',
   'WHERE'  => [
     'RAW'  => [
         'LOWER(' . DBmysql::quoteName('field') . ')' => strtolower('Value')
     ]
   ]
]);
// SELECT * FROM `my_table` WHERE LOWER(`field`) = 'value'
```
