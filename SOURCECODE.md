## Gestión de código fuente
La gestión del código fuente de GLPI está a cargo de GIT y está alojada en [GitHub](https://github.com/glpi-project/glpi).
Para contribuir con el código fuente, deberá saber algunas cosas sobre Git y el modelo de desarrollo que seguimos.
# Sistema de jerarquía de archivos
|Nota|
|----|
|Esto enumera los archivos y directorios actuales enumerados en el código fuente de GLPI. Algunos archivos no forman parte de los archivos distribuidos.|
Esta es una breve descripción de las carpetas y archivos principales de GLPI:


The main branch is where new features are added. This code is reputed as non stable.

The xx/bugfixes branches is where bugs are fixed. This code is reputed as stable.

Those branches are created when any major version is released. At the time I wrote these lines, latest stable version is 9.1 so the current bugfix branch is 9.1/bugfixes. As old versions are not supported, old bugfixes branches will not be changed at all; while they’re still existing.

Testing
There are more and more unit tests in GLPI; we use the atoum unit tests framework.

Every proposal must contains unit tests; for new features as well as bugfixes. For the bugfixes; this is not a strict requirement if this is part of code that is not tested at all yet. See the unit testing section at the bottom of the page.

Anyways, existing unit tests may never be broken, if you made a change that breaks something, check your code, or change the unit tests, but fix that! ;)

File Hierarchy System
Note

This lists current files and directories listed in the source code of GLPI. Some files are not part of distribued archives.

This is a brieve description of GLPI main folders and files:

 - .tx: Transifex configuration
 - ajax
 - - *.php: Ajax components
 - - files Files written by GLPI or plugins (documents, session files, log files, …)
 - front
 - - *.php: Front components (all displayed pages)
 - config (only populated once installed)
 - - config_db.php: Database configuration file
 - - local_define.php: Optional file to override some constants definitions (see `inc/define.php`)
 - css
 - - …: CSS stylesheets
 - - *.css: CSS stylesheets
 - inc
 - - *.php: Classes, functions and definitions
 - install
 - -mysql: MariaDB/MySQL schemas
 - -*.php: upgrades scripts and installer
 - js
 - - *.js: Javascript files
 - lib
 - -…: external Javascript libraries
 - locales
 - - glpi.pot: Gettext’s POT file
 - - *.po: Gettext’s translations
 - - *.mo: Gettext’s compiled translations
 - pics
 - *.*: pictures and icons
 - plugins:
 - -…: where all plugins lends
 - scripts: various scripts which can be used in crontabs for example
 - tests: unit and integration tests
 - tools: a bunch of tools
 - vendor: third party libs installed from composer (see composer.json below)
 - .gitignore: Git ignore list
 - .htaccess: Some convenient apache rules (all are commented)
 - .travis.yml: Travis-CI configuration file
 - apirest.php: REST API main entry point
 - apirest.md: REST API documentation
 - apixmlrpc.php: XMLRPC API main entry point
 - AUTHORS.txt: list of GLPI authors
 - CHANGELOG.md: Changes
 - composer.json: Definition of third party libraries (see composer website)
 - COPYING.txt: Licence
 - index.php: main application entry point
 - phpunit.xml.dist: unit testing configuration file
 - README.md: well… a README ;)
 - status.php: get GLPI status for monitoring purposes
 