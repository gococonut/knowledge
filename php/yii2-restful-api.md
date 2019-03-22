# Yii2 RESTful api

> Yii2 实现 RESTful API， 看了几天 Yii2 的源码， 总结一下Yii2实现 RESTful API 流程.

## entry for the application

入口文件也就是web server配置的root文件， `Yii2\backend\web\index.php`

```php
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

require(__DIR__ . '/../../vendor/autoload.php');
require(__DIR__ . '/../../vendor/yiisoft/yii2/Yii.php');
require(__DIR__ . '/../../common/config/bootstrap.php');
require(__DIR__ . '/../config/bootstrap.php');

$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../common/config/main.php'),
    require(__DIR__ . '/../../common/config/main-local.php'),
    require(__DIR__ . '/../config/main.php'),
    require(__DIR__ . '/../config/main-local.php')
);

(new yii\web\Application($config))->run();
```

入口文件的作用是: 加载各种config文件，run`yii\web\Application`， 想要探究Yii2实现RESTful API的原理接下来看一下`yii\web\Application->run()`的源代码。在`yii\base\Application`中可以找 到`public function run()`。

## source code

```php
<?php
public function run()
{
    try {

        $this->state = self::STATE_BEFORE_REQUEST;
        $this->trigger(self::EVENT_BEFORE_REQUEST);

        $this->state = self::STATE_HANDLING_REQUEST;
        $response = $this->handleRequest($this->getRequest());

        $this->state = self::STATE_AFTER_REQUEST;
        $this->trigger(self::EVENT_AFTER_REQUEST);

        $this->state = self::STATE_SENDING_RESPONSE;
        $response->send();

        $this->state = self::STATE_END;

        return $response->exitStatus;

    } catch (ExitException $e) {

        $this->end($e->statusCode, isset($response) ? $response : null);
        return $e->statusCode;

    }
}
```

在`fun run()`中可以很清晰的看出yii2处理`request`的过程： 1. trigger `EVENT_BEFORE_REQUEST` 2. `handleRequest` 3. trigger `EVENT_AFTER_REQUEST`

**TODO**

* [ ] 两个event的作用

先看一下`handleRequest`的源码。

```php
<?php
public function handleRequest($request)
{
    if (empty($this->catchAll)) {
        list ($route, $params) = $request->resolve();
    } else {
        $route = $this->catchAll[0];
        $params = array_splice($this->catchAll, 1);
    }
    try {
        Yii::trace("Route requested: '$route'", __METHOD__);
        $this->requestedRoute = $route;
        $result = $this->runAction($route, $params);
        if ($result instanceof Response) {
            return $result;
        } else {
            $response = $this->getResponse();
            if ($result !== null) {
                $response->data = $result;
            }

            return $response;
        }
    } catch (InvalidRouteException $e) {
        throw new NotFoundHttpException($e->getMessage(), $e->getCode(), $e);
    }
}
```

在Yii2中，在请求到达入口文件之后，接下来就是处理请求信息：

* **分析请求得到**`$route`**,** `$params`

  `$request->resolve()`：

```php
    {
        $result = Yii::$app->getUrlManager()->parseRequest($this);
        if ($result !== false) {
            list ($route, $params) = $result;
            $_GET = array_merge($_GET, $params);
            return [$route, $_GET];
        } else {
            throw new NotFoundHttpException(Yii::t('yii', 'Page not found.'));
        }
    }
```

> `$request->resolve`也就解释了为什么在`action`可以中`_GET()`获得`queryString`中的`params`,`_GET()`本质上就是一个`param`组成的数组。

* `runAction`

  Yii2中一个API的基本结构为：

```php
    class ExampleController
    {
        public function actionIndex()
        {
            # code;
            return $items;
        }
    }
```

* `return $response`

其中最为重要为`runAction`,即为Yii2中一个API的基本结构为中的`actionIndex()`

```php
<?php
public function runAction($route, $params = [])
{
    $parts = $this->createController($route);
    if (is_array($parts)) {
        /* @var $controller Controller */
        list($controller, $actionID) = $parts;
        $oldController = Yii::$app->controller;
        Yii::$app->controller = $controller;
        $result = $controller->runAction($actionID, $params);
        Yii::$app->controller = $oldController;

        return $result;
    } else {
        $id = $this->getUniqueId();
        throw new InvalidRouteException('Unable to resolve the request "' . ($id === '' ? $route : $id . '/' . $route) . '".');
    }
}
```

## file tree

```text
** init之后的Yii2 advanced 文件目录结 **

Yii2
├── backend
│   ├── assets
│   │   └── AppAsset.php
│   ├── codeception.yml
│   ├── config
│   │   ├── bootstrap.php
│   │   ├── main-local.php
│   │   ├── main.php
│   │   ├── params-local.php
│   │   ├── params.php
│   │   ├── test-local.php
│   │   └── test.php
│   ├── controllers
│   │   └── SiteController.php
│   ├── models
│   ├── runtime
│   ├── tests
│   │   ├── _bootstrap.php
│   │   ├── _data
│   │   │   └── login_data.php
│   │   ├── functional
│   │   │   ├── _bootstrap.php
│   │   │   └── LoginCest.php
│   │   ├── functional.suite.yml
│   │   ├── _output
│   │   ├── _support
│   │   │   ├── FunctionalTester.php
│   │   │   └── UnitTester.php
│   │   ├── unit
│   │   │   └── _bootstrap.php
│   │   └── unit.suite.yml
│   ├── views
│   │   ├── layouts
│   │   │   └── main.php
│   │   └── site
│   │       ├── error.php
│   │       ├── index.php
│   │       └── login.php
│   └── web
│       ├── assets
│       ├── css
│       │   └── site.css
│       ├── favicon.ico
│       ├── index.php
│       ├── index-test.php
│       └── robots.txt
├── codeception.yml
├── common
│   ├── codeception.yml
│   ├── config
│   │   ├── bootstrap.php
│   │   ├── main-local.php
│   │   ├── main.php
│   │   ├── params-local.php
│   │   ├── params.php
│   │   ├── test-local.php
│   │   └── test.php
│   ├── fixtures
│   │   └── User.php
│   ├── mail
│   │   ├── layouts
│   │   │   ├── html.php
│   │   │   └── text.php
│   │   ├── passwordResetToken-html.php
│   │   └── passwordResetToken-text.php
│   ├── models
│   │   ├── LoginForm.php
│   │   └── User.php
│   ├── tests
│   │   ├── _bootstrap.php
│   │   ├── _data
│   │   │   └── user.php
│   │   ├── _output
│   │   ├── _support
│   │   │   └── UnitTester.php
│   │   ├── unit
│   │   │   └── models
│   │   │       └── LoginFormTest.php
│   │   └── unit.suite.yml
│   └── widgets
│       └── Alert.php
├── composer.json
├── console
│   ├── config
│   │   ├── bootstrap.php
│   │   ├── main-local.php
│   │   ├── main.php
│   │   ├── params-local.php
│   │   └── params.php
│   ├── controllers
│   ├── migrations
│   │   └── m130524_201442_init.php
│   ├── models
│   └── runtime
├── environments
│   ├── dev
│   │   ├── backend
│   │   │   ├── config
│   │   │   │   ├── main-local.php
│   │   │   │   ├── params-local.php
│   │   │   │   └── test-local.php
│   │   │   └── web
│   │   │       ├── index.php
│   │   │       └── index-test.php
│   │   ├── common
│   │   │   └── config
│   │   │       ├── main-local.php
│   │   │       ├── params-local.php
│   │   │       └── test-local.php
│   │   ├── console
│   │   │   └── config
│   │   │       ├── main-local.php
│   │   │       └── params-local.php
│   │   ├── frontend
│   │   │   ├── config
│   │   │   │   ├── main-local.php
│   │   │   │   ├── params-local.php
│   │   │   │   └── test-local.php
│   │   │   └── web
│   │   │       ├── index.php
│   │   │       └── index-test.php
│   │   ├── yii
│   │   ├── yii_test
│   │   └── yii_test.bat
│   ├── index.php
│   └── prod
│       ├── backend
│       │   ├── config
│       │   │   ├── main-local.php
│       │   │   └── params-local.php
│       │   └── web
│       │       └── index.php
│       ├── common
│       │   └── config
│       │       ├── main-local.php
│       │       └── params-local.php
│       ├── console
│       │   └── config
│       │       ├── main-local.php
│       │       └── params-local.php
│       ├── frontend
│       │   ├── config
│       │   │   ├── main-local.php
│       │   │   └── params-local.php
│       │   └── web
│       │       └── index.php
│       └── yii
├── frontend
│   ├── assets
│   │   └── AppAsset.php
│   ├── codeception.yml
│   ├── config
│   │   ├── bootstrap.php
│   │   ├── main-local.php
│   │   ├── main.php
│   │   ├── params-local.php
│   │   ├── params.php
│   │   ├── test-local.php
│   │   └── test.php
│   ├── controllers
│   │   └── SiteController.php
│   ├── models
│   │   ├── ContactForm.php
│   │   ├── PasswordResetRequestForm.php
│   │   ├── ResetPasswordForm.php
│   │   └── SignupForm.php
│   ├── runtime
│   ├── tests
│   │   ├── acceptance
│   │   │   ├── _bootstrap.php
│   │   │   └── HomeCest.php
│   │   ├── acceptance.suite.yml.example
│   │   ├── _bootstrap.php
│   │   ├── _data
│   │   │   ├── login_data.php
│   │   │   └── user.php
│   │   ├── functional
│   │   │   ├── AboutCest.php
│   │   │   ├── _bootstrap.php
│   │   │   ├── ContactCest.php
│   │   │   ├── HomeCest.php
│   │   │   ├── LoginCest.php
│   │   │   └── SignupCest.php
│   │   ├── functional.suite.yml
│   │   ├── _output
│   │   ├── _support
│   │   │   ├── FunctionalTester.php
│   │   │   └── UnitTester.php
│   │   ├── unit
│   │   │   ├── _bootstrap.php
│   │   │   └── models
│   │   │       ├── ContactFormTest.php
│   │   │       ├── PasswordResetRequestFormTest.php
│   │   │       ├── ResetPasswordFormTest.php
│   │   │       └── SignupFormTest.php
│   │   └── unit.suite.yml
│   ├── views
│   │   ├── layouts
│   │   │   └── main.php
│   │   └── site
│   │       ├── about.php
│   │       ├── contact.php
│   │       ├── error.php
│   │       ├── index.php
│   │       ├── login.php
│   │       ├── requestPasswordResetToken.php
│   │       ├── resetPassword.php
│   │       └── signup.php
│   └── web
│       ├── assets
│       ├── css
│       │   └── site.css
│       ├── favicon.ico
│       ├── index.php
│       ├── index-test.php
│       └── robots.txt
├── init
├── init.bat
├── LICENSE.md
├── README.md
├── requirements.php
├── vagrant
│   ├── config
│   │   └── vagrant-local.example.yml
│   ├── nginx
│   │   ├── app.conf
│   │   └── log
│   └── provision
│       ├── always-as-root.sh
│       ├── once-as-root.sh
│       └── once-as-vagrant.sh
├── Vagrantfile
├── yii
├── yii.bat
├── yii_test
└── yii_test.bat
```

> 文件夹的不逐一介绍， [Yii2官方文档](http://www.yiichina.com/)有很详细的说明。

## the end

本文只写了Yii2实现RESTful API的流程，还不尽详实，随时更新。

