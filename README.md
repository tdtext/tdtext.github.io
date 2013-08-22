# tdtext

Have you as well been dreaming about a world where it is possible for anyone to write a The DataTank extension and amaze the world of Open Data?

With tdtext, this should become possible. Just like people can easily create ckan extensions (ckanext, see what we did there?) with a minimum of documentation, you are able to create The DataTank extensions which plug into the workflow. The TDT extension systems is built for the ease of developers. Once a The DataTank is installed, it should be a breeze to install: scrapers, extra formatters, extra visualizations, strategies to read data, and so on.

## Architecture ##

A lot of classes in The DataTank become Observables. From the moment something happens in a certain object of such a class, an event will be triggered and the Observers will be updated about this event.

One observer will be the TdtextNotifier. TdtextNotifier stands for: The DataTank Extensions Notifier. It will detect the kind of event that happened, and will alert the right instances of an extension about this.

_ For the DataTank techies who are afraid of switching to this architecture: it only consists of making some classes splSubjects (http://www.php.net/manual/en/class.splsubject.php) and adding the right hooks. They will by default always have one observer: TdtextNotifier which will hold a list of all extensions installed and which interfaces they implement. _

## Repositories

http://github.com/tdt/

The organisation for the repositories needed for the basic functionality of The DataTank (e.g. LTS 2013.12)

http://github.com/tdtext

The organisation where we will add official extensions for The DataTank and a barebone repository which everyone can fork to start its own tdtextension.
It also contains docs at http://tdtext.github.io/ which can be edited at http://github.com/tdtext/tdtext.github.io.

## Installing extensions using composer

If you're not familiar with composer, check out http://getcomposer.org/doc/00-intro.md.

```bash
extensionname= # Search for extensions at https://packagist.org/search/?q=tdtext
cd /path/to/tdt/root/
composer require tdtext/${extensionname}
composer update # this will also trigger the code to enable the extension. You can disable it in your config.
```

## Writing a tdt extension

* Fork the barebone tdtextension at http://github.com/tdtext/barebone 
* (Re)Name your git repository towards tdtext-{extensionname} (this will make your extension discoverable through https://packagist.org)
* Add the description of your extension in composer.json
* run composer update
* Edit your class and start creating your extension

Once you have tested it by installing it locally, you can add it to packagist: https://packagist.org/packages/submit

### The Interfaces (the hard way)

#### tdt/core/tdtext/IRouteEditor

With the route mapper you can add a controller for a specific route. This may come in handy when you want to handle your 

```php
Interface IRouteEditor {
    abstract function editRoutes(&$routes); //add, remove or edit routes
}
```

#### tdt/core/tdtext/IDefinitionsEditor

When the documentation in tdt/core is ready, it will update the TdtextNotifier, which will in its turn notify all the extensions which implement this interface.

You can add your own documentation for a new resource if you want for instance to write a scraper (nice example in AScraper - see further)

```php
Interface IDefinitionsEditor {
    abstract function editDefinitions(&$definitions);
}
```

#### tdt/core/tdtext/IFormattersEditor

When the documentation in tdt/core is ready, it will update the TdtextNotifier, which will in its turn notify all the extensions which implement this interface.

You can add your own documentation for a new resource if you want for instance to write a scraper (nice example in AScraper - see further)

```php
Interface IFormattersEditor {
    /**
    * Add or edit formatters in this array
    */
    abstract function editFormatters(&$formatters);
}
```

#### tdt/core/tdtext/ITransformer

A transformer transforms an object after it is read into memory.

```php
Interface ITransformer {
   /**
    * Add or edit an object from the moment is read into memory
    * @param $resourceconfiguration contains the identifier of a resource and the configuration
    * @param $object is the data object
    */
    abstract function transform($resourceconfiguration, &$object);
}
```


...WIP

### Using an abstract class (recommended)

#### tdt/core/tdtext/AStrategy

If you want to implement a new strategy next to the standard ones for reading a certain source (e.g. a NoSQL cluster or a certain data format), extend this abstract class.

```php
abstract class AStrategy implements ... {

    /**
     * Returns an array according to the discovery API of parameter objects.
     * They include the parameters needed to read a resource which's source uses this strategy.
     */
    abstract function getGETParameters();

    /**
     * Returns an array according to the discovery API of parameter objects.
     * They include documentation about whether the parameter is required or not when configuring a source of this strategy type through a PUT request.
     */
    abstract function getConfigParameters();

    /**
     * when reading the a resource configured with this strategy, this is what's going to happen.
     * The resourceconfiguration contains the resourceidentifier and the config parameters (as defined by the getConfigParameters() function)
     */
    abstract function read($resourceconfiguration, $parameters);
}
```

#### tdt/core/tdtext/AFormatter

Write a new behaviour for a certain format.

```php
abstract class AFormatter implements IFormattersEditor {
    $name = "";

    /**
     * @param $name e.g. "JSON"
     */
    public function __construct($name){
        $this->name = $name;
    }

    function editFormatters(&$formatters){
        $formatters[$this->name] = get_class($this); //Question: does this work across namespaces?
    }

    /**
     * Returns an array according to the discovery API of parameter objects.
     * They include extra parameters which may be given to a formatter
     * e.g. for a chart visualization, you might want to ask which fields to use, or for CSV, you might want to ask for the delimiter to use.
     */
    abstract function getGETParameters();

    /**
     * when reading the a resource configured with this strategy, this is what's going to happen.
     * @param $parameters both contains the formatter parameters as the resource parameter
     * @param $resourceconfiguration contains the resourceidentifier and the config parameters
     * @param $object the object to print
     */
    abstract function print($resourceconfiguration, $parameters, $object);
}
```

#### tdt/core/tdtext/AScraper

Writing a scraper can be done easily by extending the AScraper class. Use this class if you want to have a certain URI in The DataTank to scrape data from a certain source.

```php
abstract class AScraper implements IDefinitionsEditor, IRouteEditor {

    /**
     * @param $resourceidentifier the path of the 
     */
    public function __construct($resourceidentifier){
        $this->resourceidentifier = $resourceidentifier;  
    }
    
    /**
     * @override
     */
    public function editRoutes(&$routes){
        $routes[$resourceidentifier] = get_class($this);
    }

    public function editDefinitions(&$definitions){

    }

    /**
     * Returns an array according to the discovery API of parameter objects.
     * They include the parameters needed to read a resource which's source uses this strategy.
     */
    abstract function getGETParameters();

    abstract function read($parameters);
}
```
