## modelo de base de datos
La base de datos GLPI actual contiene más de 250 tablas; el objetivo de la documentación actual es ayudarlo a comprender la lógica del proyecto, no detallar cada tabla y posibilidad.

Como en toda base de datos, hay tablas, relaciones entre ellas (más o menos complejas), algunas relaciones tienen descripciones almacenadas en otra tabla, algunas tablas están unidas entre sí… Bueno, es bastante común :) Empecemos con un ejemplo simple :

![Image text](https://github.com/GustavoIUDigial/documentacionGLPI/blob/main/images/db_model_computer.png)

|Nota|
|----|
|El esquema anterior es un ejemplo, ¡está lejos de estar completo!|
Lo que podemos ver aquí:

- las computadoras están directamente vinculadas a los sistemas operativos, versiones de sistemas operativos, arquitecturas de sistemas operativos, …,
- las computadoras están vinculadas a memorias, procesadores y monitores mediante una tabla de relaciones (que en ese caso permite vincular esos componentes a otros elementos que no sean una computadora),
- Los recuerdos tienen un tipo.

Como se indica en la nota anterior, esto está lejos de ser completo; pero esto es bastante representativo de todo el esquema de la base de datos.

## Conjuntos de resultados
Todos los conjuntos de resultados enviados desde la base de datos GLPI siempre deben ser matrices asociativas.

## Convenciones de nombres
Todos los nombres de tablas y campos están en minúsculas y siguen la misma lógica. Si no respetas eso; GLPI no podrá encontrar información relevante.

## Mesas
Los nombres de las tablas están vinculados con los nombres de las clases de PHP; todos tienen el prefijo `glpi_` , y el nombre de la clase se establece en plural. Las tablas de complementos deben tener el prefijo `glpi_plugin_`; seguido del nombre del complemento, otro guión y luego el nombre de la clase en plural.

Algunos ejemplos:
|nombre de la clase PHP|Nombre de la tabla            |
|----------------------|------------------------------|
|`Computer`            | `glpi_computers`             |
|`Ticket`              | `glpi_tickets`               |
|`ITILCategory`	       |`glpi_itilcategories`         |
|`PluginExampleProfile`|`glpi_plugin_example_profiles`|

## Campos
|Advertencia|
|-----------|
|Cada tabla debe tener una clave principal de incremento automático llamada id.|

La denominación de los campos depende principalmente de usted; excepto para identificadores y claves foráneas. ¡Solo sé claro y conciso!

Para agregar un campo de clave externa; simplemente use el nombre de la tabla externa sin `glpi_` prefijo y agregue el `_id` sufijo. 
 
|Advertencia|
|-----------|
|Incluso si agregar una clave externa en una tabla debería ser perfectamente correcto; esta no es la forma habitual en que se hacen las cosas en GLPI, consulte Hacer relaciones para obtener más información.|

Algunos ejemplos:
|nombre de la clase PHP         |Nombre de la tabla            |
|-------------------------------|------------------------------|
|`glpi_computers`               | `computers_id`               |
|`glpi_tickets`                 | `tickets_id`                 |
|`glpi_itilcategories`	        | `itilcategories_id`          |
|`glpi_plugin_example_profiles` | `plugin_example_profiles_id` |

## hacer relaciones
En la mayoría de los casos, es posible que desee vincular muchos elementos diferentes a otra cosa. Supongamos que desea hacer posible vincular una computadora , una impresora o un teléfono a un componente de memoria . Debe agregar claves externas en las tablas de elementos; pero en algo tan grande como GLPI, tal vez no sea una buena idea.

En su lugar, cree una tabla de relaciones que haga referencia al componente de memoria junto con una identificación de elemento y un tipo, como por ejemplo:
```sh
CREATE TABLE `glpi_items_devicememories` (
   `id` int(11) NOT NULL AUTO_INCREMENT,
   `items_id` int(11) NOT NULL DEFAULT '0',
   `itemtype` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
   `devicememories_id` int(11) NOT NULL DEFAULT '0',
   PRIMARY KEY (`id`),
   KEY `items_id` (`items_id`),
   KEY `devicememories_id` (`devicememories_id`),
   KEY `itemtype` (`itemtype`,`items_id`),
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
Nuevamente, este es un ejemplo muy simplificado de lo que ya existe en la base de datos, pero entendiste el punto;)

En este ejemplo, `itemtype` sería `Computer`, `Printer ` o `Phone`; `items_id` del `id ` artículo relacionado.

## Índices
Para obtener un rendimiento correcto consultando la base de datos, deberás encargarte de configurar algunos índices. No tiene sentido agregar índices en todos los campos de la base de datos; pero algunos de ellos deben ser definidos:

- campos de clave externa;
- campos que se utilizan con mucha frecuencia (por ejemplo, campos como is_visible, ´itemtype`, ...),
- claves primarias ;)

Solo debe usar el nombre del campo como nombre clave.
 