# RestApi module for Zend Framework 3

## Requirements
This module has the following requirements:

* Zend Framework 3 or greater.
* PHP 7 or greater.

## Installation
You can install this module into your Zend Framework application using [composer](http://getcomposer.org).

The recommended way to install composer packages is:
```
composer require multidots/zf3-rest-api
```

Now copy this file "vender/multidots/zf3-rest-api/config/restapi.global.php" and paste to root "config/autoload/restapi.global.php"

If you do not setup at the time of package installation then add this 'RestApi' to modules.config.php file.

```php
return [
    ....
    //add this if not available
    'RestApi'
];
```

## Usage
You just need to create your API related controller and extend it to `ApiController` instead of default `AbstractActionController`.  You just need to set you results in `apiResponse` variable and your response code in `httpStatusCode` variable and return $this->createResponse(). For example,
```php
namespace Application\Controller;

use RestApi\Controller\ApiController;

/**
 * Foo Controller
 */
class FooController extends ApiController
{

    /**
     * bar method
     *
     */
    public function barAction()
    {
        // your action logic

        // Set the HTTP status code. By default, it is set to 200
        $this->httpStatusCode = 200;

        // Set the response
        $this->apiResponse['you_response'] = 'your response data';

        return $this->createResponse();
    }
}
```
You can define your logic in your action function as per your need. For above example, you will get following response in `json` format,
```json
{"status":"OK","result":{"you_response":"your response data"}}
```
The URL for above example will be `http://yourdomain.com/foo/bar`. You can customize it by setting the your module.config.php as following.

```php
'router' => [
        'routes' => [
            'home' => [
                'type' => Literal::class,
                'options' => [
                    'route'    => '/',
                    'defaults' => [
                        'controller' => Controller\FooController::class,
                        'action' => 'bar',
                        'isAuthorizationRequired' => true // set true if this api Required JWT Authorization.
                    ],
                ],
            ],
        ],
    ],
```
Simple :)

## Configurations
This module provides several configurations related to Response, Request and `JWT` authentication. The default configurations are in previously you copy and past file this restapi.global.php have configurations`.
```php
<?php

return [
    'ApiRequest' => [
        'responseFormat' => [
            'statusKey' => 'status',
            'statusOkText' => 'OK',
            'statusNokText' => 'NOK',
            'resultKey' => 'result',
            'messageKey' => 'message',
            'defaultMessageText' => 'Empty response!',
            'errorKey' => 'error',
            'defaultErrorText' => 'Unknown request!',
            'authenticationRequireText' => 'Authentication Required.',
            'pageNotFoundKey' => 'Request Not Found.',
        ],
        'jwtAuth' => [
            'cypherKey' => 'R1a#2%dY2fX@3g8r5&s4Kf6*sd(5dHs!5gD4s',
            'tokenAlgorithm' => 'HS256'
        ],
    ]
];
```
### Request authentication using JWT
You can check for a presence of auth token in API request. You need to define a flag `isAuthorizationRequired` to `true` or `false`. For example,
```php
'router' => [
        'routes' => [
            'home' => [
                'type' => Literal::class,
                'options' => [
                    'route'    => '/',
                    'defaults' => [
                        'controller' => Controller\FooController::class,
                        'action'     => 'bar',
                        'isAuthorizationRequired' => true // set true if this api Required JWT Authorization.
                    ],
                ],
            ],
        ],
    ],
```

Above API method will require an auth token in a request. You can pass the auth token in either header, in GET parameter or in POST field.

If you want to pass token in a header, use below format.
```php
Authorization: Bearer [token]

Example:
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoidmFsdWUifQ.xQgVrW5o3iNzA4iY3NDKwDp-pYoIhr-vyFgmi9RuMFo
```
In a case of GET or POST parameter, pass the token in `token` parameter.

#### Generate jwt token
This module provides methods to generate jwt token and sign with same key and algorithm. Use `$this->generate()` method wherever required. Most probably, you will need this in to user login and register API. See below example,
```php
public function login()
{
    /**
     * process your data and validate it against database table
     */

    // generate token if valid user
    $payload = ['email' => $user->email, 'name' => $user->name];

    $this->apiResponse['token'] = $this->generateJwtToken($payload);

    $this->apiResponse['message'] = 'Logged in successfully.';
    return $this->createResponse();
}
```

## Response format
The response format of API is `json` and its structure is defined as below.
```json
{
  "status": "OK",
  "result": {
    //your result data
  }
}
```

## Examples
Below one example to understand how this module works.

### Retrieve articles
Let's create an API which returns a list of articles with basic details like id and title. Our controller will look like,
```php
<?php

namespace Application\Controller;

use RestApi\Controller\ApiController;

/**
 * Articles Controller
 *
 * 
 */
class ArticlesController extends ApiController
{

    /**
     * index method
     *
     */
    public function indexAction()
    {
        // $this->token gives you to token which generated.
        // $this->tokenPayload gives you to payload details which you sets at the time of login or generate token.
        $payload = $this->tokenPayload;
        
        $articles = $this->entityManager->getRepository(Article::class)
                ->findBy([], ['id'=>'ASC']);

        $this->apiResponse['articles'] = $articles;
        return $this->createResponse();
    }
}
```
The response of above API call will look like,
```json
{
  "status": "OK",
  "result": {
    "articles": [
      {
        "id": 1,
        "title": "Lorem ipsum"
      },
      {
        "id": 2,
        "title": "Donec hendrerit"
      }
    ]
  }
}
```
## Reporting Issues
If you have a problem with this module or any bug, please open an issue on [GitHub](https://github.com/multidots/zf3-rest-api/issues).
