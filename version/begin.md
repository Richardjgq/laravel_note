# 起步

### 安装\(Laravel 5.4\)

**安装需求**

* PHP版本 &gt;= 5.6.4
* PHP扩展：OpenSSL
* PHP扩展：PDO
* PHP扩展：Mbstring
* PHP扩展：Tokenizer
* PHP扩展：XML

**通过Composer安装**

下载安装包

`composer global require "laravel/installer"`

添加环境变量,开启Laravel命令行

`export PATH="$HOME/.composer/vendor/bin:$PATH"`

在中.zshrc即可 .

现在就可以通过命令创建项目了

`laravel new headplan`

就会在当前目录下创建一个名为headplan的目录,里面是全新安装的Laravel,并且所有依赖包也是已经安装好了的.

还可以通过Composer Create-Project命令安装Laravel

`composer create-project laravel/laravel --prefer-dist`

> **注意 : Laravel5.3版本会出现以下错误:**
>
> `There is no suitable CSPRNG installed on your system`
>
> **在composer.json文件中的require中添加:**
>
> `"paragonie/random_compat": "~1.4"`
>
> 然后`composer udate`安装.
>
> 跟新会删除原来的paragonie\/random\_compat \(v2.0.2\),重新安装paragonie\/random\_compat \(v1.4.1\).具体原因查看下面的解决方案:
>
> http:\/\/stackoverflow.com\/questions\/37152618\/there-is-no-suitable-csprng-installed-on-your-system

**本地开发服务器**

```
php artisan serve
# 和php内置的一样
php -S localhost:1111 -t ./
```

### 应用配置

**Public目录**

安装完Laravel后，需要将HTTP服务器的web根目录指向`public`目录，该目录下的`index.php`文件将作为前端控制器，所有HTTP请求都会通过该文件进入应用。

**配置文件**

Laravel框架的所有配置文件都存放在`config`目录下，所有的配置项都有注释，所以你可以轻松遍览这些配置文件以便熟悉所有配置项。

**目录权限**

安装完 Laravel 后，需要配置一些目录的读写权限：**storage 和 bootstrap/cache** 目录应该是可写的，如果你使用 Homestead 虚拟机做为开发环境，这些权限已经设置好了。

**应用 Key**

接下来要做的事情就是将应用的 key（APP\_KEY）设置为一个随机字符串，如果你是通过 Composer 或者 Laravel 安装器安装的话，该 key 的值已经通过`php artisan key:generate`命令生成好了。

通常，该字符串应该是32位长，通过`.env`文件中的`APP_KEY`进行配置，如果你还没有将`.env.example`文件重命名为`.env`，现在立即这样做。如果应用 key 没有被设置，用户 Session 和其它加密数据将会有安全隐患。

**更多配置**

Laravel 几乎不再需要其它任何配置就可以正常使用了，但是，你最好再看看`config/app.php`文件，其中包含了一些基于应用可能需要进行改变的配置，比如`timezone`和`locale`（分别用于配置时区和本地化）。

### Web服务器配置 {#toc_3}

#### 美化URL {#toc_4}

**Apache**

框架中自带的`public/.htaccess`文件支持URL中隐藏`index.php`，如过你的Laravel应用使用Apache作为服务器，需要先确保Apache启用了`mod_rewrite`模块以支持`.htaccess`解析。

如果Laravel自带的`.htaccess`文件不起作用，试试将其中内容做如下替换：

```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

**Nginx**

如果你使用的是Nginx，使用如下站点配置指令就可以支持URL美化：

```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

Laravel框架所有配置文件都被存放在config目录下,每个配置项都有文档说明.

### **基本配置**

**目录权限**

web服务器需要拥有

storage目录下所有目录写权限

bootstrap\/cache目录的写权限

**应用程序的秘钥**

使用命令安装的程序会用php artisan key:generate命令自动生成.

存放在.env中,这个秘钥是用来加密会话和其他需要加密的数据的.

**额外的配置**

浏览一下config\/app.php

里面的timezone和locale可以稍作修改

Larval中的几个组件也需要简单配置一下:

* 缓存
* 数据库
* 会话

**美化链接**

Apache配置,开启mod\_rewrite模块

```
Options +FollowSymLinks
RewriteEngine On 

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

Nginx配置

```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```



