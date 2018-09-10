<div class='article-menu'>
  <ul>
    <li>
      <a href="#overview">Capa de abstracción de base de datos</a> <ul>
        <li>
          <a href="#adapters">Adaptadores de base de datos</a> <ul>
            <li>
              <a href="#adapters-custom">Implementing your own adapters</a>
            </li>
          </ul>
        </li>
        <li>
          <a href="#dialects">Dialectos de la base de datos</a> <ul>
            <li>
              <a href="#dialects-custom">Implementar sus propios dialectos</a>
            </li>
          </ul>
        </li>
        <li>
          <a href="#connection">Conexión a bases de datos</a> <ul>
            <li>
              <a href="#connection-factory">Conexión usando Factory</a>
            </li>
          </ul>
        </li>
        <li>
          <a href="#options">Configuración de opciones adicionales de PDO</a>
        </li>
        <li>
          <a href="#finding-rows">Encontrar Registros</a>
        </li>
        <li>
          <a href="#binding-parameters">Enlazando parámetros</a>
        </li>
        <li>
          <a href="#crud">Insertar/Actualizar/Borrar registros</a>
        </li>
        <li>
          <a href="#transactions">Transacciones y transacciones anidadas</a>
        </li>
        <li>
          <a href="#events">Eventos de base de datos</a>
        </li>
        <li>
          <a href="#profiling">Perfiles de sentencias SQL</a>
        </li>
        <li>
          <a href="#logging-statements">Log de sentencias SQL</a>
        </li>
        <li>
          <a href="#logger-custom">Implementar tu propio Logger</a>
        </li>
        <li>
          <a href="#describing-tables">Descripción de tablas o vistas</a>
        </li>
        <li>
          <a href="#tables">Crear/Modificar/Eliminar tablas</a> <ul>
            <li>
              <a href="#tables-create">Crear tablas</a>
            </li>
            <li>
              <a href="#tables-altering">Modificar tablas</a>
            </li>
            <li>
              <a href="#tables-dropping">Eliminar tablas</a>
            </li>
          </ul>
        </li>
      </ul>
    </li>
  </ul>
</div>

<a name='overview'></a>

# Capa de abstracción de base de datos

`Phalcon\Db` es el componente detrás de `Phalcon\Mvc\Model` que impulsa la capa del modelo del framework. Este consiste en una capa de abstracción de alto nivel independiente para bases de datos, escrito completamente en C.

Este componente permite una manipulación de base de datos a nivel inferior que el uso de los modelos tradicionales.

<a name='adapters'></a>

## Adaptadores de base de datos

Este componente hace uso de adaptadores para encapsular los detalles del cada motor de base de datos. Phalcon utiliza PDO_ para conectar a bases de datos. Son soportados los siguientes motores de base de datos:

| Clase                                   | Descripción                                                                                                                                                                                                                                                |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Phalcon\Db\Adapter\Pdo\Mysql`      | Es el sistema de gestión de bases de datos relacionales (RDBMS) más utilizado en el mundo. se ejecuta como un servidor que provee acceso a usuarios a un número de bases de datos                                                                          |
| `Phalcon\Db\Adapter\Pdo\Postgresql` | PostgreSQL es un sistema de base de datos relacional de código abierto muy potente. Tiene más de 15 años de desarrollo activo y una arquitectura probada que se ha ganado una sólida reputación por la fiabilidad, la integridad y exactitud de los datos. |
| `Phalcon\Db\Adapter\Pdo\Sqlite`     | SQLite es una biblioteca de software que implementa un motor de base de datos SQL transaccional independiente, sin servidor, sin configuración                                                                                                             |

<a name='adapters-custom'></a>

### Implementing your own adapters

The `Phalcon\Db\AdapterInterface` interface must be implemented in order to create your own database adapters or extend the existing ones.

<a name='dialects'></a>

## Dialectos de la base de datos

Phalcon encapsulates the specific details of each database engine in dialects. Those provide common functions and SQL generator to the adapters.

| Clase                              | Descripción                                           |
| ---------------------------------- | ----------------------------------------------------- |
| `Phalcon\Db\Dialect\Mysql`      | Dialecto específico SQL para base de datos MySQL      |
| `Phalcon\Db\Dialect\Postgresql` | Dialecto específico SQL para base de datos PostgreSQL |
| `Phalcon\Db\Dialect\Sqlite`     | Dialecto específico SQL para base de datos de SQLite  |

<a name='dialects-custom'></a>

### Implementar sus propios dialectos

The `Phalcon\Db\DialectInterface` interface must be implemented in order to create your own database dialects or extend the existing ones.

<a name='connection'></a>

## Conexión a bases de datos

To create a connection it's necessary instantiate the adapter class. It only requires an array with the connection parameters. The example below shows how to create a connection passing both required and optional parameters:

```php
<?php

//Requerido
$config = [
    'host'     => '127.0.0.1',
    'username' => 'mike',
    'password' => 'sigma',
    'dbname'   => 'test_db',
];

//Opcional
$config['persistent'] = false;

// Crear una conexión
$connection = new \Phalcon\Db\Adapter\Pdo\Mysql($config);
```

```php
<?php

// Requerido
$config = [
    'host'     => 'localhost',
    'username' => 'postgres',
    'password' => 'secret1',
    'dbname'   => 'template',
];

// Opcional
$config['schema'] = 'public';

// Crea una conección
$connection = new \Phalcon\Db\Adapter\Pdo\Postgresql($config);
```

```php
<?php

// Requerido
$config = [
    'dbname' => '/path/to/database.db',
];

// Crea una conección
$connection = new \Phalcon\Db\Adapter\Pdo\Sqlite($config);
```

<a name='options'></a>

## Configuración de opciones adicionales de PDO

You can set PDO options at connection time by passing the parameters `options`:

```php
<?php

// Crea una conección con opciones PDO
$connection = new \Phalcon\Db\Adapter\Pdo\Mysql(
    [
        'host'     => 'localhost',
        'username' => 'root',
        'password' => 'sigma',
        'dbname'   => 'test_db',
        'options'  => [
            PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES 'UTF8'",
            PDO::ATTR_CASE               => PDO::CASE_LOWER,
        ]
    ]
);
```

<a name='connection-factory'></a>

## Conexión usando Factory

También se puede utilizar un simple archivo `ini` para configurar o conectar el servicio `db` a la base de datos.

```ini
[database]
host = TEST_DB_MYSQL_HOST
username = TEST_DB_MYSQL_USER
password = TEST_DB_MYSQL_PASSWD
dbname = TEST_DB_MYSQL_NAME
port = TEST_DB_MYSQL_PORT
charset = TEST_DB_MYSQL_CHARSET
adapter = mysql
```

```php
<?php

use Phalcon\Config\Adapter\Ini;
use Phalcon\Di;
use Phalcon\Db\Adapter\Pdo\Factory;

$di = new Di();
$config = new Ini('config.ini');

$di->set('config', $config);

$di->set(
    'db', 
    function () {
        return Factory::load($this->config->database);
    }
);
```

The above will return the correct database instance and also has the advantage that you can change the connection credentials or even the database adapter without changing a single line of code in your application.

<a name='finding-rows'></a>

## Encontrar Registros

`Phalcon\Db` provides several methods to query rows from tables. The specific SQL syntax of the target database engine is required in this case:

```php
<?php

$sql = 'SELECT id, name FROM robots ORDER BY name';

// Envia una declaración SQL al sistema de Base de Datos
$result = $connection->query($sql);

// Imprime el nombre de cada robot
while ($robot = $result->fetch()) {
   echo $robot['name'];
}

// Obtiene todos los registros en un array
$robots = $connection->fetchAll($sql);
foreach ($robots as $robot) {
   echo $robot['name'];
}

// Obtiene solo el primer registro
$robot = $connection->fetchOne($sql);
```

By default these calls create arrays with both associative and numeric indexes. You can change this behavior by using `Phalcon\Db\Result::setFetchMode()`. This method receives a constant, defining which kind of index is required.

| Constante                  | Descripción                                           |
| -------------------------- | ----------------------------------------------------- |
| `Phalcon\Db::FETCH_NUM`   | Devuelve un array con índices numéricos               |
| `Phalcon\Db::FETCH_ASSOC` | Devuelve un array con índices asociativos             |
| `Phalcon\Db::FETCH_BOTH`  | Devuelve un array con índices asociativos y numéricos |
| `Phalcon\Db::FETCH_OBJ`   | Devolver un objeto en lugar de un array               |

```php
<?php

$sql = 'SELECT id, name FROM robots ORDER BY name';
$result = $connection->query($sql);

$result->setFetchMode(Phalcon\Db::FETCH_NUM);
while ($robot = $result->fetch()) {
   echo $robot[0];
}
```

`Phalcon\Db::query()` devuelve una instancia de `Phalcon\Db\Result\Pdo`. These objects encapsulate all the functionality related to the returned resultset i.e. traversing, seeking specific records, count etc.

```php
<?php

$sql = 'SELECT id, name FROM robots';
$result = $connection->query($sql);

// Recorre el conjunto de resultados
while ($robot = $result->fetch()) {
   echo $robot['name'];
}

// Busca en el tercer registro
$result->seek(2);
$robot = $result->fetch();

// Cuenta el conjunto de resutlados
echo $result->numRows();
```

<a name='binding-parameters'></a>

## Enlazando parámetros

Los parámetros enlazados también son compatibles con `Phalcon\Db`. Although there is a minimal performance impact by using bound parameters, you are encouraged to use this methodology so as to eliminate the possibility of your code being subject to SQL injection attacks. Both string and positional placeholders are supported. Binding parameters can simply be achieved as follows:

```php
<?php

// Enlazar con marcadores numéricos
$sql    = 'SELECT * FROM robots WHERE name = ? ORDER BY name';
$result = $connection->query(
    $sql,
    [
        'Wall-E',
    ]
);

// Enlazar marcadores por nombre
$sql     = 'INSERT INTO `robots`(name`, year) VALUES (:name, :year)';
$success = $connection->query(
    $sql,
    [
        'name' => 'Astro Boy',
        'year' => 1952,
    ]
);
```

When using numeric placeholders, you will need to define them as integers i.e. 1 or 2. In this case '1' or '2' are considered strings and not numbers, so the placeholder could not be successfully replaced. With any adapter data are automatically escaped using [PDO Quote](http://www.php.net/manual/en/pdo.quote.php).

This function takes into account the connection charset, so its recommended to define the correct charset in the connection parameters or in your database server configuration, as a wrong charset will produce undesired effects when storing or retrieving data.

Also, you can pass your parameters directly to the execute/query methods. In this case bound parameters are directly passed to PDO:

```php
<?php

// Enlazando con marcadores PDO
$sql    = 'SELECT * FROM robots WHERE name = ? ORDER BY name';
$result = $connection->query(
    $sql,
    [
        1 => 'Wall-E',
    ]
);
```

<a name='crud'></a>

## Insertar/Actualizar/Borrar registros

To insert, update or delete rows, you can use raw SQL or use the preset functions provided by the class:

```php
<?php

// Inserting data with a raw SQL statement
$sql     = 'INSERT INTO `robots`(`name`, `year`) VALUES ('Astro Boy', 1952)';
$success = $connection->execute($sql);

// With placeholders
$sql     = 'INSERT INTO `robots`(`name`, `year`) VALUES (?, ?)';
$success = $connection->execute(
    $sql,
    [
        'Astro Boy',
        1952,
    ]
);

// Generating dynamically the necessary SQL
$success = $connection->insert(
    'robots',
    [
        'Astro Boy',
        1952,
    ],
    [
        'name',
        'year',
    ],
);

// Generating dynamically the necessary SQL (another syntax)
$success = $connection->insertAsDict(
    'robots',
    [
        'name' => 'Astro Boy',
        'year' => 1952,
    ]
);

// Updating data with a raw SQL statement
$sql     = 'UPDATE `robots` SET `name` = 'Astro boy' WHERE `id` = 101';
$success = $connection->execute($sql);

// With placeholders
$sql     = 'UPDATE `robots` SET `name` = ? WHERE `id` = ?';
$success = $connection->execute(
    $sql,
    [
        'Astro Boy',
        101,
    ]
);

// Generating dynamically the necessary SQL
$success = $connection->update(
    'robots',
    [
        'name',
    ],
    [
        'New Astro Boy',
    ],
    'id = 101' // Warning! In this case values are not escaped
);

// Generating dynamically the necessary SQL (another syntax)
$success = $connection->updateAsDict(
    'robots',
    [
        'name' => 'New Astro Boy',
    ],
    'id = 101' // Warning! In this case values are not escaped
);

// With escaping conditions
$success = $connection->update(
    'robots',
    [
        'name',
    ],
    [
        'New Astro Boy',
    ],
    [
        'conditions' => 'id = ?',
        'bind'       => [101],
        'bindTypes'  => [PDO::PARAM_INT], // Optional parameter
    ]
);
$success = $connection->updateAsDict(
    'robots',
    [
        'name' => 'New Astro Boy',
    ],
    [
        'conditions' => 'id = ?',
        'bind'       => [101],
        'bindTypes'  => [PDO::PARAM_INT], // Optional parameter
    ]
);

// Deleting data with a raw SQL statement
$sql     = 'DELETE `robots` WHERE `id` = 101';
$success = $connection->execute($sql);

// With placeholders
$sql     = 'DELETE `robots` WHERE `id` = ?';
$success = $connection->execute($sql, [101]);

// Generating dynamically the necessary SQL
$success = $connection->delete(
    'robots',
    'id = ?',
    [
        101,
    ]
);
```

<a name='transactions'></a>

## Transacciones y transacciones anidadas

Working with transactions is supported as it is with PDO. Perform data manipulation inside transactions often increase the performance on most database systems:

```php
<?php

try {
    // Iniciar una transacción
    $connection->begin();

    // Ejecutar algunas sentencias SQL
    $connection->execute('DELETE `robots` WHERE `id` = 101');
    $connection->execute('DELETE `robots` WHERE `id` = 102');
    $connection->execute('DELETE `robots` WHERE `id` = 103');

    // Confirmar transacción si todo ha ido bien
    $connection->commit();
} catch (Exception $e) {
    // Una excepción ha ocurrido hacer rollback a la transaccion
    $connection->rollback();
}
```

In addition to standard transactions, `Phalcon\Db` provides built-in support for [nested transactions](http://en.wikipedia.org/wiki/Nested_transaction) (if the database system used supports them). When you call begin() for a second time a nested transaction is created:

```php
<?php

try {
    // Start a transaction
    $connection->begin();

    // Execute some SQL statements
    $connection->execute('DELETE `robots` WHERE `id` = 101');

    try {
        // Start a nested transaction
        $connection->begin();

        // Execute these SQL statements into the nested transaction
        $connection->execute('DELETE `robots` WHERE `id` = 102');
        $connection->execute('DELETE `robots` WHERE `id` = 103');

        // Create a save point
        $connection->commit();
    } catch (Exception $e) {
        // An error has occurred, release the nested transaction
        $connection->rollback();
    }

    // Continue, executing more SQL statements
    $connection->execute('DELETE `robots` WHERE `id` = 104');

    // Commit if everything goes well
    $connection->commit();
} catch (Exception $e) {
    // An exception has occurred rollback the transaction
    $connection->rollback();
}
```

<a name='events'></a>

## Eventos de base de datos

`Phalcon\Db` es capaz de enviar eventos al [EventsManager](/[[language]]/[[version]]/events) si está presente. Some events when returning boolean false could stop the active operation. Son soportados los siguientes eventos:

| Nombre de Evento      | Activador                                                          | ¿Puede detener la operación? |
| --------------------- | ------------------------------------------------------------------ |:----------------------------:|
| `afterConnect`        | Después de una conexión con éxito a un sistema de base de datos    |              No              |
| `beforeQuery`         | Antes de enviar una instrucción SQL en el sistema de base de datos |              Sí              |
| `afterQuery`          | Después de enviar una instrucción SQL a la base de datos           |              No              |
| `beforeDisconnect`    | Antes de cerrar una conexión temporal de base de datos             |              No              |
| `beginTransaction`    | Antes de comenzar una transacción                                  |              No              |
| `rollbackTransaction` | Antes de anular (roolback) una transacción                         |              No              |
| `commitTransaction`   | Antes de que una finalizar (commit) una transacción                |              No              |

Bind an EventsManager to a connection is simple, `Phalcon\Db` will trigger the events with the type `db`:

```php
<?php

use Phalcon\Events\Manager as EventsManager;
use Phalcon\Db\Adapter\Pdo\Mysql as Connection;

$eventsManager = new EventsManager();

// Listen all the database events
$eventsManager->attach('db', $dbListener);

$connection = new Connection(
    [
        'host'     => 'localhost',
        'username' => 'root',
        'password' => 'secret',
        'dbname'   => 'invo',
    ]
);

// Assign the eventsManager to the db adapter instance
$connection->setEventsManager($eventsManager);
```

Detener las operaciones SQL es muy útil si se desea, por ejemplo, implementar algún verificador de inyectores SQL de último recurso:

```php
<?php

use Phalcon\Events\Event;

$eventsManager->attach(
    'db:beforeQuery',
    function (Event $event, $connection) {
        $sql = $connection->getSQLStatement();

        // Check for malicious words in SQL statements
        if (preg_match('/DROP|ALTER/i', $sql)) {
            // DROP/ALTER operations aren't allowed in the application,
            // this must be a SQL injection!
            return false;
        }

        // It's OK
        return true;
    }
);
```

<a name='profiling'></a>

## Perfiles de sentencias SQL

`Phalcon\Db` includes a profiling component called `Phalcon\Db\Profiler`, that is used to analyze the performance of database operations so as to diagnose performance problems and discover bottlenecks.

Database profiling is really easy With `Phalcon\Db\Profiler`:

```php
<?php

use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Db\Profiler as DbProfiler;

$eventsManager = new EventsManager();

$profiler = new DbProfiler();

// Listen all the database events
$eventsManager->attach(
    'db',
    function (Event $event, $connection) use ($profiler) {
        if ($event->getType() === 'beforeQuery') {
            $sql = $connection->getSQLStatement();

            // Start a profile with the active connection
            $profiler->startProfile($sql);
        }

        if ($event->getType() === 'afterQuery') {
            // Stop the active profile
            $profiler->stopProfile();
        }
    }
);

// Assign the events manager to the connection
$connection->setEventsManager($eventsManager);

$sql = 'SELECT buyer_name, quantity, product_name '
     . 'FROM buyers '
     . 'LEFT JOIN products ON buyers.pid = products.id';

// Execute a SQL statement
$connection->query($sql);

// Get the last profile in the profiler
$profile = $profiler->getLastProfile();

echo 'SQL Statement: ', $profile->getSQLStatement(), "\n";
echo 'Start Time: ', $profile->getInitialTime(), "\n";
echo 'Final Time: ', $profile->getFinalTime(), "\n";
echo 'Total Elapsed Time: ', $profile->getTotalElapsedSeconds(), "\n";
```

You can also create your own profile class based on `Phalcon\Db\Profiler` to record real time statistics of the statements sent to the database system:

```php
<?php

use Phalcon\Events\Manager as EventsManager;
use Phalcon\Db\Profiler as Profiler;
use Phalcon\Db\Profiler\Item as Item;

class DbProfiler extends Profiler
{
    /**
     * Executed before the SQL statement will sent to the db server
     */
    public function beforeStartProfile(Item $profile)
    {
        echo $profile->getSQLStatement();
    }

    /**
     * Executed after the SQL statement was sent to the db server
     */
    public function afterEndProfile(Item $profile)
    {
        echo $profile->getTotalElapsedSeconds();
    }
}

// Create an Events Manager
$eventsManager = new EventsManager();

// Create a listener
$dbProfiler = new DbProfiler();

// Attach the listener listening for all database events
$eventsManager->attach('db', $dbProfiler);
```

<a name='logging-statements'></a>

## Log de sentencias SQL

Using high-level abstraction components such as `Phalcon\Db` to access a database, it is difficult to understand which statements are sent to the database system. `Phalcon\Logger` interacts with `Phalcon\Db`, providing logging capabilities on the database abstraction layer.

```php
<?php

use Phalcon\Logger;
use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Logger\Adapter\File as FileLogger;

$eventsManager = new EventsManager();

$logger = new FileLogger('app/logs/db.log');

$eventsManager->attach(
    'db:beforeQuery',
    function (Event $event, $connection) use ($logger) {
        $sql = $connection->getSQLStatement();

        $logger->log($sql, Logger::INFO);
    }
);

// Asignar el eventsManager a la instancia del adaptador de base de datos
$connection->setEventsManager($eventsManager);

// Ejecutar alguna sentencia SQL
$connection->insert(
    'products',
    [
        'Hot pepper',
        3.50,
    ],
    [
        'name',
        'price',
    ]
);
```

As above, the file `app/logs/db.log` will contain something like this:

```bash
[Sun, 29 Apr 12 22:35:26 -0500][DEBUG][Resource Id #77] INSERT INTO products
(name, price) VALUES ('Hot pepper', 3.50)
```

<a name='logger-custom'></a>

## Implementar tu propio Logger

You can implement your own logger class for database queries, by creating a class that implements a single method called `log`. The method needs to accept a string as the first argument. You can then pass your logging object to `Phalcon\Db::setLogger()`, and from then on any SQL statement executed will call that method to log the results.

<a name='describing-tables'></a>

## Descripción de tablas o vistas

`Phalcon\Db` también proporciona métodos para recuperar información detallada sobre tablas y vistas:

```php
<?php

// Obtiene las tablas de la base de datos test_db
$tables = $connection->listTables('test_db');

// ¿Hay una tabla "robots" en la base de datos?
$exists = $connection->tableExists('robots');

// Obtener nombre, tipos de datos y características especiales de los campos 'robots'
$fields = $connection->describeColumns('robots');
foreach ($fields as $field) {
    echo 'Column Type: ', $field['Type'];
}

// Obtener los indices de la tabla 'robots' 
$indexes = $connection->describeIndexes('robots');
foreach ($indexes as $index) {
    print_r(
        $index->getColumns()
    );
}

// Obtener las llaves foraneas en la tabla 'robots'
$references = $connection->describeReferences('robots');
foreach ($references as $reference) {
    // Imprimir columnas referenciadas
    print_r(
        $reference->getReferencedColumns()
    );
}
```

A table description is very similar to the MySQL describe command, it contains the following information:

| Campo            | Tipo            | Clave                                                      | Nulo                              |
| ---------------- | --------------- | ---------------------------------------------------------- | --------------------------------- |
| Nombre del campo | Tipo de columna | ¿Es la columna parte de la clave principal o de un índice? | ¿La columna permite valores null? |

Methods to get information about views are also implemented for every supported database system:

```php
<?php

// Obtener las vistas en la base de datos 'test_db'
$tables = $connection->listViews('test_db');

// ¿Hay una vista llamada "robots" en la base de datos?
$exists = $connection->viewExists('robots');
```

<a name='tables'></a>

## Crear/Modificar/Eliminar tablas

Diferentes sistemas de base de datos (MySQL, Postgresql etc.) ofrecen la capacidad para crear, modificar o eliminar tablas con el uso de comandos como CREATE, ALTER o DROP. La sintaxis SQL difiere según el sistema de base de datos utilizado. `Phalcon\Db` offers a unified interface to alter tables, without the need to differentiate the SQL syntax based on the target storage system.

<a name='tables-create'></a>

### Crear tablas

The following example shows how to create a table:

```php
<?php

use \Phalcon\Db\Column as Column;

$connection->createTable(
    'robots',
    null,
    [
       'columns' => [
            new Column(
                'id',
                [
                    'type'          => Column::TYPE_INTEGER,
                    'size'          => 10,
                    'notNull'       => true,
                    'autoIncrement' => true,
                    'primary'       => true,
                ]
            ),
            new Column(
                'name',
                [
                    'type'    => Column::TYPE_VARCHAR,
                    'size'    => 70,
                    'notNull' => true,
                ]
            ),
            new Column(
                'year',
                [
                    'type'    => Column::TYPE_INTEGER,
                    'size'    => 11,
                    'notNull' => true,
                ]
            ),
        ]
    ]
);
```

`Phalcon\Db::createTable()` accepts an associative array describing the table. Las columnas se definen con la clase `Phalcon\Db\Column`. The table below shows the options available to define a column:

| Opción          | Descripción                                                                                                                         | Opcional |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |:--------:|
| `type`          | Tipo de columna. Debe ser una constante de `Phalcon\Db\Column` (ver lista de abajo)                                               |    No    |
| `primary`       | `true` si la columna forma parte de la clave primaria de la tabla                                                                   |    Sí    |
| `size`          | Algunos tipos de columnas como `VARCHAR` o `INTEGER` puede tener un tamaño específico                                               |    Sí    |
| `scale`         | Columnas `DECIMAL` o `NUMBER` pueden tener una escala para especificar cuántos decimales deben almacenarse                          |    Sí    |
| `unsigned`      | Las columnas `INTEGER` pueden tener signo o no. Esta opción no se aplica a otros tipos de columnas                                  |    Sí    |
| `notNull`       | ¿La columna puede almacenar valores nulos?                                                                                          |    Sí    |
| `default`       | Valor por defecto (cuando se usa con `'notNull' => true`).                                                                       |    Sí    |
| `autoIncrement` | Con este atributo la columna se incrementará automáticamente con un entero. Solo una columna en la tabla puede tener este atributo. |    Sí    |
| `bind`          | Una de las constantes `BIND_TYPE_*` que indica como debe tratarse la columna antes de guardarse                                     |    Sí    |
| `first`         | La columna debe colocarse en primera posición en el orden de columnas                                                               |    Sí    |
| `after`         | La columna debe colocarse después de la columna indicada                                                                            |    Sí    |

`Phalcon\Db` supports the following database column types:

- `Phalcon\Db\Column::TYPE_INTEGER`
- `Phalcon\Db\Column::TYPE_DATE`
- `Phalcon\Db\Column::TYPE_VARCHAR`
- `Phalcon\Db\Column::TYPE_DECIMAL`
- `Phalcon\Db\Column::TYPE_DATETIME`
- `Phalcon\Db\Column::TYPE_CHAR`
- `Phalcon\Db\Column::TYPE_TEXT`

The associative array passed in `Phalcon\Db::createTable()` can have the possible keys:

| Índice       | Descripción                                                                                                                                                     | Opcional |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |:--------:|
| `columns`    | Un array con un conjunto de columnas de la tabla definida con `Phalcon\Db\Column`                                                                             |    No    |
| `indexes`    | Un array con un conjunto de índices de la tabla definida con `Phalcon\Db\Index`                                                                               |    Sí    |
| `references` | Un array con un conjunto de referencias de tabla (foreign keys) definidas con `Phalcon\Db\Reference`                                                          |    Sí    |
| `options`    | Un array con un conjunto de opciones de creación de la tabla. Estas opciones se refieren a menudo al sistema de base de datos en la que se generó la migración. |    Sí    |

<a name='tables-altering'></a>

### Modificar tablas

As your application grows, you might need to alter your database, as part of a refactoring or adding new features. Not all database systems allow to modify existing columns or add columns between two existing ones. `Phalcon\Db` está limitado por estas restricciones.

```php
<?php

use Phalcon\Db\Column as Column;

// Agregar una nueva columna
$connection->addColumn(
    'robots',
    null,
    new Column(
        'robot_type',
        [
            'type'    => Column::TYPE_VARCHAR,
            'size'    => 32,
            'notNull' => true,
            'after'   => 'name',
        ]
    )
);

// Modificar una columna existente
$connection->modifyColumn(
    'robots',
    null,
    new Column(
        'name',
        [
            'type'    => Column::TYPE_VARCHAR,
            'size'    => 40,
            'notNull' => true,
        ]
    )
);

// Borrar la columna 'name'
$connection->dropColumn(
    'robots',
    null,
    'name'
);
```

<a name='tables-dropping'></a>

### Eliminar tablas

Ejemplos de eliminación de tablas:

```php
<?php

// Elimina la tabla robot desde la base de datos activa
$connection->dropTable('robots');

// Elimina la tabla robot desde la base de datos "machines"
$connection->dropTable('robots', 'machines');
```