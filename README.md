# Monolog adapter for Bitrix CMS

[![Build Status](https://travis-ci.org/vadimpalgov/monolog-bitrix.svg)](https://travis-ci.org/vadimpalgov/monolog-adapter)
[![Latest Stable Version](https://poser.pugx.org/vadimpalgov/monolog-bitrix/v/stable)](https://packagist.org/packages/vadimpalgov/monolog-adapter) 
[![Total Downloads](https://poser.pugx.org/vadimpalgov/monolog-bitrix/downloads)](https://packagist.org/packages/vadimpalgov/monolog-adapter) 
[![License](https://poser.pugx.org/vadimpalgov/monolog-bitrix/license)](https://packagist.org/packages/vadimpalgov/monolog-adapter)

[Monolog](https://github.com/Seldaek/monolog) adapter for Bitrix CMS:

* Bitrix handler and formatter for Monolog.
* Handler for logger uncaught exceptions of the Bitrix.
* Configuration loggers with using the `.settings.php`.

[MonologAdapter](https://github.com/bitrix-expert/monolog-adapter) based package
## Installation

Download the library using Composer:

```bash
composer require vadimpalgov/monolog-bitrix
```

Write in the [`init.php`](https://dev.1c-bitrix.ru/learning/course/?COURSE_ID=43&LESSON_ID=2916) file:

```php
<?php

\Bex\Monolog\MonologAdapter::loadConfiguration();
```

## Usage

### Configuring your loggers

Configurate the logger in the `.settings.php`:

```php
return array(
    'exception_handling' => array(
        'value' => array(
            'log' => array(
                'class_name' => '\Bex\Monolog\ExceptionHandlerLog',
                'settings' => array(
                    'logger' => 'app'
                ),
            ),
        ),
        'readonly' => false
    ),
    'monolog' => array(
        'value' => array(
            'handlers' => array(
                'default' => array(
                    'class' => '\Monolog\Handler\StreamHandler',
                    'level' => 'DEBUG',
                    'stream' => '/path/to/logs/app.log'
                ),
                'feedback_event_log' => array(
                    'class' => '\Bex\Monolog\Handler\BitrixHandler',
                    'level' => 'DEBUG',
                    'event' => 'TYPE_FOR_EVENT_LOG',
                    'module' => 'vendor.module'
                ),
            ),
            'loggers' => array(
                'app' => array(
                    'handlers'=> array('default'),
                ),
                'feedback' => array(
                    'handlers'=> array('feedback_event_log'),
                )
            )
        ),
        'readonly' => false
    )
);
```

Use rules property for filter logging uncaught exceptions by instanceof logic:
```php
'exception_handling' => array(
    'value' => array(
        'log' => array(
            'class_name' => '\Bex\Monolog\ExceptionHandlerLog',
            'settings' => array(
                'logger' => 'app',
                'rules' => array(
                    'instanceof' => '\Vendor\Exception\UnloggedInterface', // or opposite: !instanceof
                )
            ),
        ),
    ),
    'readonly' => false
)
```

Use context property for change log debug data format:
```php
'exception_handling' => array(
    'value' => array(
        'log' => array(
            'class_name' => '\Bex\Monolog\ExceptionHandlerLog',
            'settings' => array(
                'logger' => 'app',
                'context' => function ($exception) {
                     return array(
                         'file' => $exception->getFile(),
                         'line' => $exception->getLine(),
                         'trace' => $exception->getTrace(),
                         'some_param' => $exception->getSomeParam(),
                     );
                 },
            ),
        ),
    ),
    'readonly' => false
)
```

### Write logs

Write logs from your application. For example, write logs when created new message from the feedback form:

```php
<?php

use Monolog\Registry;

$logger = Registry::getInstance('feedback');

// Write info message with context: invalid message from feedback
$logger->info('Failed create new message on feedback form', array(
    'item_id' => 21,
    'Invalid data' => $addResult->getErrorMessages(), // error savings
    'Form data' => $formRequest // data from feedback form
));
```

The result in the Control Panel of Bitrix:

![Event Log](event-log.png)

## Requirements

* PHP >= 5.3
* Bitrix CMS >= 16.5.6
