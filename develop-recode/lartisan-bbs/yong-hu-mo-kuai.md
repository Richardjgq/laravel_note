# 用户模块

#### 注册登录

生成代码 :

```
php artisan make:auth
```

`make:auth`会询问我们是否要覆盖`app.blade.php`这里选择no

可以使用`git status`查看文件更改状态 .

查看新增的路由 :

```
php artisan route:list
```

删除生成的多余内容 :

```
# 多余的路由绑定
Route::get('/home', 'HomeController@index')->name('home');
# 多余的控制器和模板
$ rm app/Http/Controllers/HomeController.php
$ rm resources/views/home.blade.php
```

**修改顶部模板**

```
# 两处路由别名修改
<li><a href="{{ route('login') }}">登录</a></li>
<li><a href="{{ route('register') }}">注册</a></li>

# 然后是添加判断,这里直接判断未登录用户和登录用户展示的页面内容
@guest
    <li><a href="{{ route('login') }}">登录</a></li>
    <li><a href="{{ route('register') }}">注册</a></li>
@else
    .
    .
    .
@endguest

# 还要注意下退出登录时候的CSRF令牌
<form id="logout-form" action="{{ route('logout') }}" method="POST" style="display: none;">
    {{ csrf_field() }}
</form>
```

**测试注册功能**

```
# 执行迁移文件
php artisan migrate
```

修改注册成功后跳转路径 :

搜索"/home"修改 ,

```
protected $redirectTo = '/';
```

```
modified:   app/Http/Controllers/Auth/LoginController.php
modified:   app/Http/Controllers/Auth/RegisterController.php
modified:   app/Http/Controllers/Auth/ResetPasswordController.php
modified:   app/Http/Middleware/RedirectIfAuthenticated.php
```

测试登录 . 提交git . 

#### 注册验证码

防止机器注册 . 

**安装扩展包**

```
composer require "mews/captcha:~2.0"
```

**生成配置文件**

```
php artisan vendor:publish --provider='Mews\Captcha\CaptchaServiceProvider' 
```

查看配置文件`config/captcha.php`

`characters`选项是用来显示给用户的所有字符串 ; 

`default`,`flat`,`mini`,`inverse`分别是定义的四种验证码类型 , 可以在此修改对应选项自定义验证码的长度、背景颜色、文字颜色等属性 . 
