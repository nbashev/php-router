# Greg PHP Routing

[![StyleCI](https://styleci.io/repos/70080128/shield?style=flat)](https://styleci.io/repos/70080128)
[![Build Status](https://travis-ci.org/greg-md/php-router.svg)](https://travis-ci.org/greg-md/php-router)
[![Total Downloads](https://poser.pugx.org/greg-md/php-router/d/total.svg)](https://packagist.org/packages/greg-md/php-router)
[![Latest Stable Version](https://poser.pugx.org/greg-md/php-router/v/stable.svg)](https://packagist.org/packages/greg-md/php-router)
[![Latest Unstable Version](https://poser.pugx.org/greg-md/php-router/v/unstable.svg)](https://packagist.org/packages/greg-md/php-router)
[![License](https://poser.pugx.org/greg-md/php-router/license.svg)](https://packagist.org/packages/greg-md/php-router)

A smarter router for web artisans.

# Table of contents:

* [Requirements](#requirements)
* [How It Works](#how-it-works)
* [Routing Schema](#routing-schema)
* [Router](#router)
* [Group Route](#group-route)
* [Request Route](#request-route)
* [Hidden Route](#hidden-route)

# Requirements

* PHP Version `^7.1`

# How It Works

**First of all**, you have to initialize a Router:

```php
$router = new \Greg\Routing\Router();
```

**Optionally**, you can add an action dispatcher to support custom actions.
The dispatcher should return a callable of the action.

Let say you want to support an action like `Controller@action`:

```php
$router->setDispatcher(function ($action): callable {
    [$controllerName, $actionName] = explode('@', $action);
    
    return [new $controllerName, $actionName];
});
```

**Then**, set up some routes:

```php
$router->any('/', function() {
    return 'Hello World!';
}, 'home');

$router->post('/user/{id#uint}', 'UsersController@save', 'user.save');
```

**Now**, you can dispatch actions:

```php
echo $router->dispatch('/'); // result: Hello World!

// Initialize "UsersController" and execute "save" method.
echo $router->dispatch('/user/1', 'POST');
```

**and**, get URLs for them:

```php
$router->url('home'); // result: /

$router->url('home', ['foo' => 'bar']); // result: /?foo=bar

$router->url('user.save', ['id' => 1]); // result: /user/id

$router->url('user.save', ['id' => 1, 'debug' => true]); // result: /user/id?debug=true
```

# Routing Schema

Routing schema supports **parameters** and **optional segments**.

**Parameter** format is `{<name>[:<default>][#<type>][|<regex>]}?`.

`<name>` - Parameter name;  
`<default>` - Default value;  
`<type>` - Parameter type. Supports `int`, `uint`, `boolean`(or `bool`);  
`<regex>` - Parameter regex;  
`?` - Question mark from the end determine if the parameter should be optional.

> Only `<name>` is required in the parameter.

**Optional segment** format is `[<schema>]`. Is working recursively.

`<schema>` - Any [routing schema](#routing-schema).

> It is very useful when you want to use the same action with different routing schema.

### Example

Let say we have a page with all articles of the same type, including pagination. The route for this page will be:

```php
$router->get('/articles/{type:lifestyle|[a-z0-9-]+}[/page-{page:1#uint}]', 'ArticlesController@type', 'articles.type');
```

`type` parameter is required in the route. Default value is `lifestyle` and should consist of **letters, numbers and dashes**.

`page` parameter is required in its segment, but the segment entirely is optional. Default value is `1` and should consist of **unsigned integers**.
If the parameter will not be set or will be the same as default value, the entire segment will be excluded from the URL.

```php
echo $router->url('articles.type'); // result: /articles/lifestyle

echo $router->url('articles.type', ['type' => 'travel']); // result: /articles/travel

echo $router->url('articles.type', ['type' => 'travel', 'page' => 1]); // result: /articles/travel

echo $router->url('articles.type', ['type' => 'travel', 'page' => 2]); // result: /articles/travel/page-2
```

As you can see, there are no more URLs where you can get duplicated content, which is best for SEO.
In this way, you can easily create good user friendly URLs.

# Router

* [request](#request) - Create a route for a specific request method.
    * [get](#request) - Create a GET route.
    * [head](#request) - Create a HEAD route.
    * [post](#request) - Create a POST route.
    * [put](#request) - Create a PUT route.
    * [delete](#request) - Create a DELETE route.
    * [connect](#request) - Create a CONNECT route.
    * [options](#request) - Create a OPTIONS route.
    * [trace](#request) - Create a TRACE route.
    * [patch](#request) - Create a PATCH route.
* [any](#any) - Create a route for any request method;

## request

Create a route for a specific request method.

```php
request(string $schema, $action, ?string $name = null, ?string $method = null): RequestRoute
```

You can also create a route method by calling the method name directly.
Available types are: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS`, `TRACE`, `PATCH`.

```php
[get|head|post|put|delete|connect|options|trace|patch](string $schema, $action, ?string $name = null): RequestRoute
```
