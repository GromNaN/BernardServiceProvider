RaekkeServiceProvider
=====================

Brings Raekke to Silex.

Getting Started
---------------

Registering and configuring `MultiPredisServiceProvider` is required as a
`$app['predis']['raekke']` service is used by the connection.

``` php
<?php

use Raekke\Silex\RaekkeServiceProvider;
use Predis\Silex\MultiPredisServiceProvider;

// .. create $app
$app->register(new RaekkeServiceProvider);
$app->register(new MultiPredisServiceProvider);

$app['predis.clients'] = array(
    'raekke' => array(
        'parameters' => 'tcp://localhost',
        'options' => array('prefix' => 'raekke:'),
    ),
);
```

Now you are ready to produce messages to a queue.

``` php
<?php

use Raekke\Message\DefaultMessage;

// .. create $app
$app['raekke.producer']->produce(new DefaultMessage('SendNewsletter', array(
    'id' => 12,
));
```

Or consume messages.

``` php
<?php

use Raekke\Command\ConsumeCommand;

// .. create $app
$app['raekke.service_resolver'] = $app->share($app->extend('raekke.service_resolver', function ($resolver, $app) {
    $resolver->register('SendNewsletter', $app['send_newsletter_service']);

    return $resolver;
}));

$app['console']->add(new ConsumeCommand($app['raekke.service_resolver'], $app['raekke.queue_factory']));
$app['console']->run();
```

``` bash
$ ./bin/console raekke:consume 'send-newsletter'
```

A Note on Debug
---------------

When `$app['debug']` is true it will use the in memory queuing instead of redis.
This can be circumvented by doing the following after registering this provider.

``` php
<?php
// .. create $app and register RaekkeServiceProvider
$app['raekke.queue_factory'] = $app->raw('raekke.queue_factory.real'];
```
