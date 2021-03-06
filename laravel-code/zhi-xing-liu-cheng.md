# 执行流程

#### 自动加载

经历过的文件 :

* 从 **public/index.php** 入口定位到 **bootstrap/autoload.php**
* 从 **bootstrap/autoload.php** 定位到 **vendor/autoload.php**
* 从 **vendor/autoload.php** 定位到`__DIR__ . '/composer' . '/autoload_real.php';`

这就是我们要找的文件**autoload\_real.php **, 文件中就是我们要找的 , 调用其方法 :

```
ComposerAutoloaderInit10488597059b401fe5386bdff1f41d9d::getLoader();
```

先从getLoader\(\)开始 .

**getLoader\(\)**

她是一个静态方法

```php
public static function getLoader()
```

第一部分内容 , 判断如果静态变量`$loader`不为空则返回`$loader`

```php
if (null !== self::$loader) {
    return self::$loader;
}
```

然后是注册一个自动加载程序 , 加载程序为本类的`loadClassLoader()`方法 , 先看一下这个方法

```php
public static function loadClassLoader($class)
{
    if ('Composer\Autoload\ClassLoader' === $class) {
        require __DIR__ . '/ClassLoader.php';
    }
}
```

静态方法 , 含有一个`$class`参数 , 判断如果`$class`等于`Composer\Autoload\ClassLoader` , 则载入当前目录下 的 ClassLoader.php 文件 , 实际上是这只是个入口 , 其目的也是为后面赋值用的 .

```php
spl_autoload_register(array('ComposerAutoloaderInit10488597059b401fe5386bdff1f41d9d', 'loadClassLoader'), true, true);
self::$loader = $loader = new \Composer\Autoload\ClassLoader();
spl_autoload_unregister(array('ComposerAutoloaderInit10488597059b401fe5386bdff1f41d9d', 'loadClassLoader'));
```

这里首先注册本类中的`loadClassLoader`函数 , 其参数$class就是后面`new \Composer\Autoload\ClassLoader();`的类名 , 然后按照`loadClassLoader`函数中的规则 , 引入了`ClassLoader.php`文件 , 最后再unregister这个函数`loadClassLoader`.简单来说 , 就是`$loader`得到`ClassLoader`类\(`\Composer\Autoload\ClassLoader`\)的一个实例 , 卸载自动加载程序 loadClassLoader .

得到了$loader , 然后就是载入设置一些路径信息 , 新版本的composer加入了autoload\_static.php文件 , 也就是在composer 1.1.0 版本开始 , 如果php的版本大于5.6等条件 , composer 会进行加载优化 .

```php
$useStaticLoader = PHP_VERSION_ID >= 50600 && !defined('HHVM_VERSION') && (!function_exists('zend_loader_file_encoded') || !zend_loader_file_encoded());
if ($useStaticLoader) {
    require_once __DIR__ . '/autoload_static.php';

    call_user_func(\Composer\Autoload\ComposerStaticInit10488597059b401fe5386bdff1f41d9d::getInitializer($loader));
} else {
    $map = require __DIR__ . '/autoload_namespaces.php';
    foreach ($map as $namespace => $path) {
        $loader->set($namespace, $path);
    }

    $map = require __DIR__ . '/autoload_psr4.php';
    foreach ($map as $namespace => $path) {
        $loader->setPsr4($namespace, $path);
    }

    $classMap = require __DIR__ . '/autoload_classmap.php';
    if ($classMap) {
        $loader->addClassMap($classMap);
    }
}
```

> 这里的getInitalizer\($loader\)方法用到了闭包类的`\Closure::bind()`方法 , 等于绑定了一个匿名函数 给ClassLoader类中的属性赋值 , autoload\_static.php中 , 其他的也都是生成好的静态属性 .
>
> `\Closure::bind()`其实就是把闭包当成对象的成员方法或者静态成员方法.
>
> ```
> Closure::bind($cl1, null, 'A'); // 就相当于在类里面加了个静态成员方法
> Closure::bind($cl2, new A(), 'A'); // 相当于在类里面加了个成员方法
> ```
>
> 成员方法中使用`$this`访问对象 , 静态成员方法直接使用`类名::成员`的方法.但是因为是匿名函数 , 没有函数名 , 所以返回一个已经绑定$this对象和类作用域的闭包给你使用 .

其他载入文件 :

* autoload\_namespaces.php
* autoload\_psr4.php
* autoload\_classmap.php
* autoload\_file.php

这些载入文件进行各自的循环 set 操作 , 如`$loader->set($namespace, $path);`等 . 具体逻辑这里不做过多描述 .

下一步要执行注册 :

```php
$loader->register(true);
// ClassLoader
public function register($prepend = false)
{
    spl_autoload_register(array($this, 'loadClass'), true, $prepend);
}
```

一个布尔值参数 , 将传给 spl\_autoload\_register 第三个参数中 . 而自动加载程序为 : `array($this, 'loadClass')`, 也就是本类的`loadClass()`方法 . \(第三个参数的意思是 , 添加函数到队列之首 , 而不是队列尾部\) .

这里再看一下注册的loadClass方法

```php
public function loadClass($class)
{
    if ($file = $this->findFile($class)) {
        includeFile($file);

        return true;
    }
}
```

这里用了`findFile()`方法判断文件是否存在 , 存在则调用函数`includeFile()`载入文件 . 其中`includeFile()`很简单 , 就是include文件 , 但是写在类外面 , 为了隔离范围 .

然后是载入autoload\_file.php文件 , 这里就是composer.json配置中自动加载的文件 , 调用下面的函数 , 放到全局变量

`$GLOBALS['__composer_autoload_files'][要加载的类生成的md5字符串]`如果包含该文件那么该全局变量的值为true .

最后 , 返回`$loader` . 也代表着 , `getLoader()`结束 , 自动加载配置完成 .

#### 载入应用

也就是入口文件中的第二行代码

```php
$app = require_once __DIR__.'/../bootstrap/app.php';
```

app.php最后返回$app这个对象 , 所以文件中第一步就是实例化laravel应用程序 . 注释中描述其是所有组件的"粘合剂" , 也是系统绑定各个部分的IoC容器 .

```php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);
```

我们先来看看这个类初始化了那些东西

```php
public function __construct($basePath = null)
{
    if ($basePath) {
        $this->setBasePath($basePath);
    }

    $this->registerBaseBindings();

    $this->registerBaseServiceProviders();

    $this->registerCoreContainerAliases();
}
```

**setBasePath\(\)设置基本路径**

这里传入并设置的是应用的根目录 , **setBasePath\(\)**方法是这样写的

```
public function setBasePath($basePath)
{
    $this->basePath = rtrim($basePath, '\/');

    $this->bindPathsInContainer();

    return $this;
}
```

将根目录赋值给了成员属性 basePath , 然后执行`$this->bindPathsInContainer()`方法 , 绑定应用的所有路径到容器 . 这里用了

```php
protected function bindPathsInContainer()
{
    $this->instance('path', $this->path());
    $this->instance('path.base', $this->basePath());
    $this->instance('path.lang', $this->langPath());
    $this->instance('path.config', $this->configPath());
    $this->instance('path.public', $this->publicPath());
    $this->instance('path.storage', $this->storagePath());
    $this->instance('path.database', $this->databasePath());
    $this->instance('path.resources', $this->resourcePath());
    $this->instance('path.bootstrap', $this->bootstrapPath());
}
```

`$this->instance`方法将路径存入容器的instances数组属性中 , 这里的路径都是写死的 .

**registerBaseBindings\(\)注册基本绑定**

简单点说 , 此方法内部进行3次赋值

```php
protected function registerBaseBindings()
{
    static::setInstance($this);

    $this->instance('app', $this);

    $this->instance(Container::class, $this);
}
```

这个过程也就是把Laravel这个应用对象分别设置到其继承的父类Container中的成员属性中 .

```
# 一个在这,当前全局可用的容器,如果有的话
protected static $instance;
# 后两个在之前设置过的instances数组中
"app" => Application {#3 ▶}
"Illuminate\Container\Container" => Application {#3 ▶}
```

总之 , 此时父类容器中已经得到了应用对象 .

**registerBaseServiceProviders\(\)注册基本服务提供者**

```php
protected function registerBaseServiceProviders()
{
    $this->register(new EventServiceProvider($this));

    $this->register(new LogServiceProvider($this));

    $this->register(new RoutingServiceProvider($this));
} 
```

注册基本的服务提供者 , 可以明显看出三个服务提供者分别是什么 , 先看一下本类中的这个register注册方法

```php
public function register($provider, $options = [], $force = false)
{
    if (($registered = $this->getProvider($provider)) && ! $force) {
        return $registered;
    }

    // If the given "provider" is a string, we will resolve it, passing in the
    // application instance automatically for the developer. This is simply
    // a more convenient way of specifying your service provider classes.
    if (is_string($provider)) {
        $provider = $this->resolveProvider($provider);
    }

    if (method_exists($provider, 'register')) {
        $provider->register();
    }

    $this->markAsRegistered($provider);

    // If the application has already booted, we will call this boot method on
    // the provider class so it has an opportunity to do its boot logic and
    // will be ready for any usage by this developer's application logic.
    if ($this->booted) {
        $this->bootProvider($provider);
    }

    return $provider;
}
```

首先判断是否已经注册过服务提供者`getProvider()`,然后判断服务提供者是不是个字符串类名 , 如果是则resolveProvider\(\)实例化这个类 . 继续 , 寻找这个服务提供者中的register方法并调用她 , 进行注册 . 注册之后进行标记 , 就是修改本类的成员属性

```php
$this->markAsRegistered($provider);

# 方法内容
protected function markAsRegistered($provider)
{
    $this->serviceProviders[] = $provider;

    $this->loadedProviders[get_class($provider)] = true;
}
```

然后是判断服务提供者是否已经启动 , 其和前面的判断服务提供者中是否有register方法一样 . 最后返回`$provider` .

**registerCoreContainerAliases\(\)注册核心容器别名**

初始化的最后一步是注册核心容器别名 , 其中的内容都是写死的 , 其目的是调用Container中的alias方法 , 然后把要注册的别名分别存储在Container的成员属性aliases和$abstractAliases中 .

```php
public function alias($abstract, $alias)
{
    $this->aliases[$alias] = $abstract;

    $this->abstractAliases[$abstract][] = $alias;
}
```



