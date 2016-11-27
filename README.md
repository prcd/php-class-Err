# php-class-Err

Error handling class for PHP. Handles all errors generated by PHP, enables logging for production environments and instant feedback for development.

* [Basic set up](#basic-set-up)
* [Error types](#error-types)
* [Decoding errors](#decoding-errors)
* [Custom terminal actions](#custom-terminal-actions)
* [All valid options during initialisation](#all-valid-options-during-initialisation)
* [Extracting error data before shutdown](#extracting-error-data-before-shutdown)
* [Logging extra data](#logging-extra-data)

----

##Basic set up

Create a directory for the log files and add two empty files called background.txt and terminal.txt.

Initialise Err as early as possible and tell it where the log directory is.

```php
include '/path/to/Err.php';

Err::initialise([
  'log_directory' => '/path/to/log/dir'
]);

```

If errors are recorded and need to be logged, the details are encoded as a JSON string and stored on an individual line in the relevant log file.

##Error types

The class categorises errors as three types: *ignore*, *background* and *terminal*. It is set up with default values which can be overwritten if needed. Only *ignore* and *background* values are set, anything else is considered *terminal*.

If only *ignore* errors occur during run time, nothing is logged and the script runs as intended.

If any *background* errors occur (regardless of *ignore* errors or not), then *all* errors will be logged in the defined background file and the script runs as intended.

If any errors occur that are not listed in *ignore* or *background* then the script will terminate. Errors will be dumped to screen or logged in the defined terminal file depending on mode. It is also possible to [create terminal actions](#custom-terminal-actions)

Allowed values for *ignore* and *background* are `E_WARNING`, `E_NOTICE`, `E_CORE_WARNING`, `E_COMPILE_WARNING`, `E_USER_WARNING`, `E_USER_NOTICE`, `E_STRICT`, `E_RECOVERABLE_ERROR`, `E_DEPRECATED` and `E_USER_DEPRECATED`. Any other constants submitted will result in an Exception being thrown.

Here is an example of initialisation including defining error types (this example sets the defult values).

```php
include '/path/to/Err.php';

Err::initialise([
  'log_directory'     => '/path/to/log/dir',
  'errors_background' => E_WARNING | E_CORE_WARNING | E_COMPILE_WARNING | E_USER_WARNING | E_DEPRECATED | E_USER_DEPRECATED,
  'errors_ignore'     => E_NOTICE | E_USER_NOTICE | E_STRICT
]);

```

If an error code is set for *ignore* and *background*, the error code will be categorised as *ignore*.

##Decoding errors

The class includes an error decoding method. Once initialised, it can be used like,

```php
$error_name = Err::getName($error_code);

```

For example, if the error code was `4` then `E_PARSE` would be returned.

##Custom terminal actions

The class will perform one of two actions in the event of a terminal error. If in development mode (default setting) it will run a private method called `terminalActionDevelopment()`. If in production mode (set during initilisation ([see all initilisation options](#all-valid-options-during-initialisation)) then it will run a private method called `terminalActionProduction()`.

These methods can be customised by extending the class, including updating the property `$class_name`. The following is an example,

```php
ErrExtended extends Err {
  
  /**
   * @var string The name of the extended class
   */
  protected static $class_name = 'ErrExtended';

  /**
   * Method will run in the event of a terminal action when in development mode
   */
  protected static function terminalActionDevelopment()
  {
    // Because the data is being extracted, there will be nothing to log.
    // If extract() is not used, the errors will be logged.
    $data = parent::extract(true);

    // Any extra log data is also available in this class
    $extra_log_data = parent::$extra_log_data;

    // A simple dump of the data
    echo '<hr>';
    echo '<h1>PHP error terminated script</h1>';
    echo '<hr>';
    echo '<pre>';
    print_r($data['counts']);
    echo '</pre>';
    echo '<hr>';
    echo '<pre>';
    print_r($data['errors']);
    echo '</pre>';
  }

  /**
   * Method will run in the event of a terminal action when in production mode
   */
  protected static function terminalActionProduction()
  {
    echo '<h1>Sorry, an application error occurred</h1><hr><p>Details have been logged</p>';
  }
}

```

Once the extended class has been written, it can be used just like the original class. 

```php
include '/path/to/Err.php';
include '/path/to/ErrExtended.php';

ErrExtended::initialise([
  'log_directory' => '/path/to/log/dir',
]);

```

Keep the custom methods as simple as possible, errors will not be logged or displayed. It may be helpful to begin each method with `error_reporting(E_ALL);` for debugging during development.

###Terminal method

The class can be extended with a method named `terminalAction` (this method already exists in the Err class but does nothing). If the terminal message parameter is not set, then this method will be called after a terminal error.

For example, 

```php
include '/path/to/Err.php';

class ErrExtended extends Err {

  protected static function terminalAction()
  {
    list($counts, $errors) = parent::extract(true);

    echo '<hr>';
    echo '<h1>PHP error terminated script</h1>';
    echo '<hr>';
    echo '<pre>';
    print_r($counts);
    echo '</pre>';
    echo '<hr>';
    echo '<pre>';
    print_r($errors);
    echo '</pre>';
  }
}

// now initilisation needs to happen with the extended class
ErrExtended::initialise([
  'log_directory'    => '/path/to/log/dir'
]);

```

With the above example, in the event of a terminal error, the error counts and error data will be echo'd. Nothing will be logged because `extract()` has been called which removes all error data from the class.

##All valid options during initialisation

The following example shows initialisation with all valid options being set with their default values. The only required option is `log_directory`. The class will function with all other default values.


```php
Err::initialise([
  'development'         => true,
  'errors_background'   => E_WARNING | E_CORE_WARNING | E_COMPILE_WARNING | E_USER_WARNING | E_DEPRECATED | E_USER_DEPRECATED,
  'errors_ignore'       => E_NOTICE | E_USER_NOTICE | E_STRICT,
  'log_directory'       => '',
  'log_file_background' => 'background.txt',
  'log_file_terminal'   => 'terminal.txt',
  'timestamp'           => time()
]);

```

##Extracting error data before shutdown

During development it may be useful to view errors immediately after each request runs so that it is not necessary to check the logs. It is possible to extract all error data at any point. This will return the error data as an array, all data that is returned will not be logged. For example,

```php
$error_data = Err::extract();

```

An example dump of `$error_data` is,

```
Array (
	[0] => Array (
		[error] => 8
		[message] => Undefined index: foo
		[file] => /var/www/html/example.php
		[line] => 21
		[backtrace] => Array (
			[0] => Array (
				[file] => /var/www/html/example.php
				[line] => 21
				[function] => errorHandler
				[class] => Err
 				[type] => ::
			)
		)
	)
)
```

You may also add `true` as an argument, 

```php
$error_data = Err::extract(true);

```

to get a little more detail,

```
Array (
	[counts] => Array (
		[ignore] => 2
		[background] => 1
		[terminal] => 0
	)
	[errors] => Array (
		// array of error data as in the previous example
	)
)
```

##Logging extra data

It is possible to add extra data to the error logs for help with debugging. Adding log data should be done after initialisation. Here's an example,

```php
Err::addLogData([
  'organisation_id' => $_SESSION['organisation_id'],
  'user_id'         => $_SESSION['user_id']
]);

```

The Organisation ID and User ID will now be stored in the log. All submitted array values are typecast as strings.
