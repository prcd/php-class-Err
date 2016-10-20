# php-class-Err
Error handling class for PHP. Handles all errors generated by PHP, enables logging, instant feedback for debugging and and fails gracefully.

##Basic set up
Create a directory for the log files and add two empty files called background.txt and terminate.txt.

Initialise Err as early as possible.

```php
<?php

include '/path/to/Err.php';

Err::initialise([
  'log_directory' => '/path/to/log/dir'
]);

```
