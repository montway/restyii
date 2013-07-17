# Restyii

A RESTful HATEOAS compliant extension for Yii. Allows your application to serve and accept json, jsonp, xml, csv or html using the same code.


## WARNING: Pre-alpha software, use at your own risk.

# Installation.

Install using composer. Requires php 5.3 or later.

Restyii makes use of a custom `WebApplication` that replaces the Yii default. For this reason it's
necessary to replace existing calls to

    Yii::createWebApplication($config)

with

    Yii::createApplication('Restyii\\Web\\Application', $config)

in your `index.php` file.


# Configuration.

Example Restyii application config

    <?php
    Yii::setPathOfAlias('vendor', __DIR__.'/../../vendor'); // the path to the composer vendors dir
    return array(
      'name' => 'Restyii Demo',
      'description' => 'This is a description of the application!',
      'import' => array(
        'application.resources.*',
      ),
      'modules' => array(
          'gii'=>array(
              'class'=>'system.gii.GiiModule',
              'password'=>'yourSecretPassword',
              // If removed, Gii defaults to localhost only. Edit carefully to taste.
              'ipFilters'=>array('127.0.0.1','::1'),
              'generatorPaths'=>array(
                  'vendor.codemix.restyii.gii-templates',
              ),
          ),
      ),
      'components' => array(
            ...
      ),
    );


Restyii comes with a Gii code generator that makes generating resources easy.


# RAVE

Restyii follows the RAVE application architecture. RAVE is quite similar to MVC,  but there are some important differences,
RAVE stands for:

* __Resources__
* __Actions__
* __Views__
* __Events__


Importantly, Resources, Actions, Views and Events all __descrbe__ themselves.
This allows the RAVE application to be self-documenting.


## Resources

Resources are analogous to the Model in MVC, but they have some key differences.

* Individual resources are explicitly tied to particular URLs.
* Resources know how to link to themselves and to related resources.
* Resources know how to *describe* themselves and their attributes.
* Resources know how to *format* their own attributes.


In RAVE, the bulk of your business logic should be placed in resources, RAVE embraces the __fat__ model approach.


## Actions

Actions are analogous to the operations performed by a Controller in MVC.

* Actions *should* accept any kind of resource. This promotes code reuse and is made possible by the rich meta data resources provide.
* Actions know which http headers and query string parameters to expect and how to describe them.
* Actions are associated with a specific HTTP verb that is used to perform an action, e.g. DELETE for delete actions.

Actions usually operate either on individual resources or collections of resources.

## Views

RAVE Views are effectively identical to views in MVC. In Restyii they are used to decorate data (usually as HTML) when the client requests something other than XML, JSON etc.

## Events

Events are triggered by resources when they change state in some way, such as when a particular model is created or updated.
RAVE events are usually published to a message bus or a pub/sub channel to allow communication with other processes in the application,
for example, to allow 'realtime' notifications in the browser.

# Differences from standard Yii applications

In order to implement RAVE, Restyii applications require several changes to the standard Yii way of doing things.

Some important ones:

* The web application must be an instance of `Restyii\Web\Application`.

* The default controller for all modules, including the main application, should be called `DefaultController`. This means
the normal `SiteController` should be renamed. `DefaultController` should extend the `Restyii\Controller\Root` class.

* The request (`Yii::app()->request`) now has a `getResponse()` method that returns a `Restyii\Web\Response`
  instance. This response instance is responsible for formatting and sending the data to the client.

* Controllers should __always__ use class based actions, and controllers are explicitly tied to resource types. Resource controllers extend `Restyii\Controller\Model`

* Actions should __always__ extend the `Restyii\Action\Base` class

* Rather than implementing `run()`, actions should implement `present()` and `perform()` methods.

* Rather than calling `$this->render(...)`, actions should return the data for the response from within the `present()` and `perform()` methods.

* Application has a `schema` component that introspects the application and determines the available resources, actions etc.

* Resources extend `Restyii\Model\ActiveRecord`.

* Resources have a `links()` method that returns the appropriate links for the current resource.

* Resources have `classLabel()` and `instanceLabel()` methods that return the appropriate labels for the resource type and a particular resource instance.

