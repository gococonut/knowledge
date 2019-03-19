> cookie在发送到客户端之前如果不经任何加密，会很容易被伪造，下面我们来简单介绍下Yii2框架是怎么对cookie进行加密的。

```php
$cookie = new yii\web\Cookie([
    'name' => 'username',
    'value' => 'test',
]);
Yii::$app->response->getCookies()->add($cookie);
```

首先，我们new一个新的Cookie对象

```php
$serial = serialize([$cookie->name, $cookie->value]);
```

接着，Yii2会把$cookie对象按照上面的方式进行序列化

```php
$value = Yii::$app->getSecurity()->hashData($serial, $validationKey);
```

接着，Yii2 会把序列化好之后的 `$serial` 做一次哈希，其中，`$validateionKey` 是用来做哈希的 key，在配置文件中可以设置。`hashData()` 这个方法会把生成的哈希值 $hash 和 $serial 拼接起来，然后返回。生成哈希值的方法使用了 `hash_hmac()` 这个函数，根据不同的哈希算法，哈希值的长度可能会不一样，比如默认的 `sha256算法` 就是 64 位的，我们可以根据这个原理将 $hash 和 $serial 拆开，然后来验证 cookie 的合法性。

最后，上面的 $value 将会作为 cookie 的新值发送给客户端。

知道了 cookie 是怎么加密的，那么也很容易知道怎么去验证 cookie 的合法性了，由于 cookie 加密后的值是由 $hash 和 $serial 直接拼接而成的，那么我们只要根据 `不同算法生成的哈希值长度不一样` 这个原理，把 $hash 和 $serial 提取出来，然后使用相同的 `$validationKey` 对 $serial 做哈希，然后和 $hash 的值作比较看是否相等，如果相等的话，那么就证明这个 cookie 是合法的了。
