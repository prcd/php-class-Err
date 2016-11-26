# php-class-Err

Error handling class for PHP. Handles all errors generated by PHP, enables logging for production environments and instant feedback for development.

* [Basic set up](#basic-set-up)
* [Error types](#error-types)
* [Terminal actions](#terminal-actions)
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

If any errors occur that are not listed in *ignore* or *background* then the script will terminate. Errors will be dumped to screen or logged in the defined terminal file - see [Terminal message](#terminal-message).

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

##Terminal actions

By default, if the class encounters a *terminal* error it will dump the entire error log to screen. There are other ways to handle a terminal error.

###Terminal message

A string can be echo'd instead, and the error will be logged to the defined terminal file. 

Here is how to set the terminal message during initialisation.

```php
include '/path/to/Err.php';

Err::initialise([
  'log_directory'    => '/path/to/log/dir',
  'terminal_message' => '<h1>Sorry</h1><hr><p>Something went wrong. We have logged the error.</p>'
]);

```

###Terminal function

If the terminal message parameter is not set, a terminal function parameter can be set. The function does not have to be loaded at the time of initialisation, a second parameter can be set with the function's file path. If the terminal function, or file path does not exist, the error data will be logged and a terminal message will be displayed describing the error.

The terminal function will be passed an array as the first argument. This array will be as returned by `Err::extract(true)`, see [extracting error data before shutdown](#extracting-error-data-before-shutdown).

Here is how to set the terminal function, and optional file path, during initialisation.

```php
include '/path/to/Err.php';

Err::initialise([
  'log_directory'               => '/path/to/log/dir',
  'terminal_function'           => 'terminal_function',
  'terminal_function_file_path' => '/path/to/terminal-function.php'  
]);

```

##All valid options during initialisation

The following example shows initialisation with all valid options being set with their default values. The only required option is `log_directory`. The class will function with all other default values.


```php
Err::initialise([
	'errors_background'           => E_WARNING | E_CORE_WARNING | E_COMPILE_WARNING | E_USER_WARNING | E_DEPRECATED | E_USER_DEPRECATED,
	'errors_ignore'               => E_NOTICE | E_USER_NOTICE | E_STRICT,
	'log_directory'               => '',
	'log_file_background'         => 'background.txt',
	'log_file_terminal'           => 'terminal.txt',
	'terminal_function'           => false,
	'terminal_function_file_path' => false,
	'terminal_message'            => false,
	'timestamp'                   => time()
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
