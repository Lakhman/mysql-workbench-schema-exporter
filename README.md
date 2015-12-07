forked from johmue/mysql-workbench-schema-exporter.
Supported only Doctrine 2.0
README   
======

ATTENTION
---------

This software is EXPERIMENTAL and not ready for production.
It is just a proof of concept.

Prerequisites
-------------

  * PHP 5.3+
  * Composer to install the dependencies

Installing Dependencies
-----------------------

To fetch the defined dependencies into your local project, just run the
`install` command of `composer.phar`.

    $ php composer.phar install

What is MySQL Workbench schema exporter?
----------------------------------------

The application is intended to create:

  * Doctrine 2.0 [YAML Schema](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/yaml-mapping.html), [Annotation Classes](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/annotations-reference.html)

schema files from MySQL Workbench models (*.mwb).
It is inspired by [mysql-workbench-doctrine-plugin](http://code.google.com/p/mysql-workbench-doctrine-plugin/).

Foreign key name
----------------

To replace relations name by the name of the foreign key, start the foreign key name with "d:".

Command Line Interface (CLI)
----------------------------

There is a new CLI to simplify the export process named `export.php`, you can look under the `cli` folder.
The CLI has feature to customize export configuration before exporting. By default, CLI application will
use config file `export.json` located in the current directory to supply the parameter if it find it. To
disable this behaviour, see the option below.

The syntax of CLI:

    php cli/export.php [options] FILE [DEST]

Where:

  * `options`:
    * `--export=type`, choose the result of the export, currently available types:
      * `doctrine2-annotation`, Doctrine 2.0 Annotation classes
    * `--config=file`, read export parameters from file (in JSON format)
    * `--saveconfig`, save export parameters to file `export.json`, later can be used as value for `--config=file`
    * `--list-exporter`, show all available exporter
    * `--no-auto-config`, disable automatic config file lookup
    * `--zip`, compress the result
    * `--help`, show the usage (or suppress any parameters)
  * `FILE`, the mwb file to export
  * `DEST`, the destination directory (optional), if not specified current directory assumed

Sample usage:

    php cli/export.php --export=doctrine2-annotation example/data/test.mwb ./generated
    php cli/export.php --zip example/data/test.mwb

Sample export paramaters (JSON) for doctrine2-annotation:

    {
        "export": "doctrine2-annotation",
        "zip": false,
        "dir": "temp",
        "params": {
            "backupExistingFile": true,
            "skipPluralNameChecking": false,
            "enhanceManyToManyDetection": true,
            "bundleNamespace": "",
            "entityNamespace": "",
            "repositoryNamespace": "",
            "useAnnotationPrefix": "ORM\\",
            "useAutomaticRepository": true,
            "indentation": 4,
            "filename": "%entity%.%extension%",
            "quoteIdentifier": false,
            "generateEntitySerialization": false,
            "generateEntityToArray": true,
            "extendsClass": false
        }
    }

Exporter Options
----------------

### General options

General options applied to all formatter.

| Parameter `(Default)`               	| Description                                                                                                                                                       	|
|-------------------------------------	|-------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| indentation                         	| The indentation size for generated code.                                                                                                                          	|
| useTabs                             	| Use tabs for indentation instead of spaces. Setting this option will ignore the `indentation`-option                                                              	|
| filename `(%entity%.%extension%)`   	| The output filename format, use the following tag `%schema%`, `%table%`, `%entity%`, and `%extension%` to allow the filename to be replaced with contextual data. 	|
| skipPluralNameChecking `(false)`    	| Skip checking the plural name of model and leave as is, useful for non English table names.                                                                       	|
| backupExistingFile `(true)`         	| If target already exists create a backup before replacing the content.                                                                                            	|
| enhanceManyToManyDetection `(true)` 	| If enabled, many to many relations between tables will be added to generated code.                                                                                	|
| logToConsole `(false)`              	| If enabled, output the log to console.                                                                                                                            	|
| logFile `(null)`                    	| If specified, output the log to a file. If this option presence, option `logToConsole` will be ignored instead.                                                   	|
|                                     	|                                                                                                                                                                   	|

### Comment behavior for All

  * `{MwbExporter:external}true{/MwbExporter:external}` (applied to Table, View)

    Mark table/view as external to skip table/view code generation. For Doctrine use `{d:external}true{/d:external}` instead.

### Option list for Doctrine 2.0 Annotation

  * `useAnnotationPrefix`

    Doctrine annotation prefix. Default is `ORM\`.

  * `useAutomaticRepository`

    See above.

  * `bundleNamespace`

    See above.

  * `entityNamespace`

    See above.

  * `repositoryNamespace`

    See above.

  * `skipGetterAndSetter`

    Don't generate columns getter and setter. Default is `false`.

  * `generateEntitySerialization`

    Generate method `__sleep()` to include only real columns when entity is serialized. Default is `true`.

  * `quoteIdentifier`

    If this option is enabled, all table names and column names will be quoted using backtick (`` ` ``). Usefull when the table name or column name contains reserved word. Default is `false`.

  * `extendsClass`

    Extend a base class, must be full namespace path to cache. Default is `false`.

### Comment behavior for Doctrine 2.0 Annotation

  * `{d:bundleNamespace}AcmeBundle{/d:bundleNamespace}` (applied to Table)

    Override `bundleNamespace` option.

  * `{d:m2m}false{/d:m2m}` (applied to Table)

    MySQL Workbench schema exporter tries to automatically guess which tables are many-to-many mapping tables and will not generate entity classes for these tables.
    A table is considered a mapping table, if it contains exactly two foreign keys to different tables and those tables are not many-to-many mapping tables.

    Sometimes this guessing is incorrect for you. But you can add a hint in the comment of the table, to show that it is no mapping table. Just use "{d:m2m}false{/d:m2m}" anywhere in the comment of the table.

  * `{d:unidirectional}true{/d:unidirectional}` (applied to ForeignKey)

    All foreign keys will result in a bidirectional relation by default. If you only want a unidirectional relation, add a flag to the comment of the foreign key.

  * `{d:owningSide}true{/d:owningSide}` (applied to ForeignKey)

    In a bi-directional many-to-many mapping table the owning side of the relation is randomly selected. If you add this hint to one foreign key of the m2m-table, you can define the owning side for Doctrine.

  * `{d:cascade}persist, merge, remove, detach, all{/d:cascade}` (applied to ForeignKey or Field)

    You can specify Doctrine cascade options as a comment on a foreign key. The will be generated into the Annotation. ([Reference](http://doctrine-orm.readthedocs.org/en/latest/reference/working-with-associations.html#transitive-persistence-cascade-operations))

  * `{d:fetch}EAGER{/d:fetch}` (applied to ForeignKey)

    You can specify the fetch type for relations in the comment of a foreign key. (EAGER or LAZY, doctrine default is LAZY)

  * `{d:orphanRemoval}true{/d:orphanRemoval}` (applied to ForeignKey)

    Another option you can set in the comments of foreign key. ([Reference](http://doctrine-orm.readthedocs.org/en/latest/reference/working-with-associations.html#orphan-removal))

  * `{d:o2o}true{/d:o2o}` (applied to Field)

    To make one to one relation.

  * `{d:orphanRemoval}true{/d:orphanRemoval}` (applied to Field)

    Add orphanRemoval attr.

  * `{d:orderBy}priority=ASC{/d:orderBy}` (applied to Field)

     User order.

  * `{d:skip}true{/d:skip}` (applied to Field or Table)

    Skip field or table

  * `{d:generator-strategy}NONE{/d:generator-strategy}` (applied to id Field) [AUTO, SEQUENCE, TABLE, IDENTITY, NONE, UUID, CUSTOM]

    Generator strategy for id

  * `{d:field-mapped}parentCategory{/d:field-mapped}` (applied to Field)

    Override name mapped filed.

  * `{d:field-inversed}childCategories{/d:field-inversed}` (applied to Field)

    Override name inversed filed.

  * `{d:lifecycleCallbacks}postLoad:fnc,prePersist:fnc,postPersist:fnc,preRemove:fnc,postRemove:fnc,preUpdate:fnc,postUpdate:fnc{/d:lifecycleCallbacks}` (applied to Table)

    You can specify Doctrine lifecycleCallbacks options as a comment on a table. The will be generated into the Annotation.

   
  * `{d:type}dateinterval|datetime_with_millisecond{/d:type}` (applied to Field)
    Force cast datetype

   
  * `{d:timestampable}on=“create”{/d:timestampable}` (applied to Field)
  * `{d:timestampable}on=“update”{/d:timestampable}` (applied to Field)
    Add the `Gedmo` Timestampable annotation to field (datetime field), you can include other options here also.

  * `{d:sluggable}fields={"title"}{/d:sluggable}` (applied to Field)
    Add the `Gedmo` Sluggable annotation to field (e.g: varchar or text field), you can include other options here also.

  * `{d:tbbc}true,getPrice,setPrice,priceCurrency,priceAmount{/d:tbbc}` (applied to Field)
    Make this a TBBC Money Bundle Field (view solution 1 here: https://github.com/TheBigBrainsCompany/TbbcMoneyBundle)
    The values passed are 0-true, 1+2-getter and setter name, 3-currencyFieldName, 4-priceFieldName. You can omit these (except the `true` to use these defaults) 

  * `{d:constants}{"CONSTANT1" : 1,"CONSTANT2" : 2,"":"","CONSTANT3" : 3}{/d:constants}` (applied to Table)
    This must be a json object (key, value) containing our constants (placed at the start of the class). Will add e.g `const CONSTANT1 = 1;..`. Use a empty key value for a blank line (e.g: `"":"",`) to seperate constant types for readability.

#### Additional option list for Doctrine 2.0 Annotation ZF2 Inputfilter

  * `generateEntityPopulate`
  * `generateEntityGetArrayCopy`

Requirements
------------

Works with PHP 5.3 and up.

Links
-----
  * [MySQL Workbench](http://wb.mysql.com/)
  * [Doctrine Project](http://www.doctrine-project.org/)
  * [Symfony Project](http://www.symfony.com/)

Test-Database
-------------
  * [Sakila DB *.mwb](http://downloads.mysql.com/docs/sakila-db.zip)
  * [Sakila DB docs](http://dev.mysql.com/doc/sakila/en/index.html)

Example
-------

    <?php

    // enable autoloading of classes  
    $libDir = __DIR__.'/lib';
    require_once($libDir.'/MwbExporter/SplClassLoader.php');

    $classLoader = new SplClassLoader();
    $classLoader->setIncludePath($libDir);
    $classLoader->register();

    // create bootstrap
    $bootstrap = new \MwbExporter\Bootstrap();

    // define a formatter and do configuration
    $formatter = $bootstrap->getFormatter('doctrine2-annotation');
    $formatter->setup(array());

    // specify the workbench document to load, output directory, and storage type (zip or file)
    $mwbfile = 'myDatabaseModel.mwb';
    $outDir = getcwd();
    $storage = 'zip';
    // load document and export
    $document = $bootstrap->export($formatter, $mwbfile, $outDir, $storage);

    // show the output
    echo sprintf("Saved to %s.\n\n", $document->getWriter()->getStorage()->getResult());

    ?>
