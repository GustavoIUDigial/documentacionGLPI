## Traducciones
El idioma principal de GLPI es el inglés británico (en_GB). Todas las cadenas en el código fuente deben estar en inglés y marcadas como traducibles, usando algunas funciones convenientes.

Desde 0,84; GLPI usa [gettext](https://www.gnu.org/software/gettext/) para la localización; y [Transifex](https://www.transifex.com/glpi/GLPI/dashboard/) se utiliza para las traducciones. Si desea ayudar a traducir GLPI, regístrese en transifex y únase a nuestra [lista de correo de traducción.](https://mail.gna.org/listinfo/glpi-translation)

Lo que el sistema es capaz de hacer:

- reemplazar variables (en lenguajes LTR y RTL),
- manejar formas plurales,
- añadir información de contexto,

Este es el flujo de trabajo utilizado para las traducciones:

- Los desarrolladores agregan una cadena en el código fuente,
- Las cadenas se extraen al archivo POT,
- El archivo POT se envía a Transifex,
- Los traductores traducen,
- Los desarrolladores extraen nuevas traducciones de Transifex,
- Se generan archivos MO utilizados por GLPI.

## Funciones PHP

|Nota|
| ------ |
| Todas las funciones de traducción toman a `$domain` como argumento; el valor predeterminado es `glpi` y debe cambiarse cuando esté trabajando en un complemento. |

## traducción sencilla

Cuando tiene una cadena "simple" para traducir, puede usar varias funciones, según el caso de uso particular:

- `__($str, $domain='glpi')` (lo que probablemente usará con más frecuencia): simplemente traduzca una cadena,
- `_x($ctx, $str, $domain='glpi')`: igual que __()pero proporciona un contexto adicional,
- `__s($str, $domain='glpi')`: igual que __()pero escapa de las entidades HTML,
- `_sx($ctx, $str, $domain='glpi')`: igual que __()pero proporciona un contexto adicional y entidades HTML de escape,

## Manejar formas plurales
Cuando tienes una cadena para traducir, pero depende de un conteo o algo así. También puede usar varias funciones, según el caso de uso particular:

- `_n($sing, $plural, $nb, $domain='glpi')` (lo que probablemente usará con más frecuencia): proporcione una cadena para la forma singular, otra para la forma plural y establezca el "recuento" actual
- `_sn($str, $domain='glpi')` : igual que `_n()` pero escapa de las entidades HTML,
- `_nx($ctx, $str, $domain='glpi')` : igual que `_n()`pero proporciona un contexto adicional,

## Manejar variables
Es posible que desee reemplazar algunas partes de las traducciones; por alguna razón. Digamos que le gustaría mostrar la página actual en un número total de páginas; utilizará el método [sprintf](https://www.php.net/manual/fr/function.sprintf.php) . Esto le permitirá hacer reemplazos; pero sin apoyarse en argumentos posiciones. Por ejemplo:
```sh
<?php
$pages = 20;  //total number of pages
$current = 2; //current page
$string = sprintf(
   __('Page %1$s on %2$s'),
   $pages,
   $total
);
echo $string; //will display: "Page 2 on 20"
```
En el ejemplo anterior, `%1$s` siempre será reemplazado por `2` ; incluso si se han cambiado lugares en algunas traducciones.
|Advertencia|
| ------ |
| A veces puede ver el uso de `printf()` lo que es un equivalente que emite directamente (eco) el resultado. ¡Esto debe evitarse! |

## Funciones Javascript

_Nuevo en la versión 9.5.0._

Las funciones de traducción `__()`, `_x()`, `_n()`, `_nx()` también están disponibles en javascript en el contexto del navegador. Tienen las mismas firmas que las funciones de PHP.
```sh
alert(__('Test successful'));
```
