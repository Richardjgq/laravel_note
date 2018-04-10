# 会话管理

目的 : 由于 HTTP 协议是无状态的 , 无法在两个页面之间保证用户身份的同步 , 因此需要借助会话在浏览器中临时存储用户的身份信息 , 进而保证在同一浏览器中 , 用户在不同页面具有相同的登录状态 .

#### **会话控制器**

```
php artisan make:controller SessionsController
```

然后添加路由

```
Route::get('login', 'SessionController@create')->name('login');
Route::post('login', 'SessionController@store')->name('login');
Route::get('logout', 'SessionController@destroy')->name('logout');
```

路由方法对应添加 .

**登录表单**

```php
<div class="col-md-offset-2 col-md-8">
    <div class="panel panel-default">
        <div class="panel-heading">
            <h5>登录</h5>
        </div>
        <div class="panel-body">
            @include('includes.errors')

            <form method="POST" action="{{ route('login') }}">
                {{ csrf_field() }}

                <div class="form-group">
                    <label for="email">邮箱：</label>
                    <input type="text" name="email" class="form-control" value="{{ old('email') }}">
                </div>

                <div class="form-group">
                    <label for="password">密码：</label>
                    <input type="password" name="password" class="form-control" value="{{ old('password') }}">
                </div>

                <button type="submit" class="btn btn-primary">登录</button>
            </form>

            <hr>

            <p>还没有账号?<a href="{{ route('signup') }}">马上注册</a></p>
        </div>
    </div>
</div>
```

**认证用户**

这里需要些一下store方法

对用户post提交来的数据进行认证 , 这里的验证相对简单 , 只验证了不为空和基本格式的正确与否 :

```php
$data = $this->validate($request, [
    'email' => 'required|email|max:255',
    'password' => 'required'
]);
```

用户身份认证

借助Laravel中`Auth`的`attempt`方法 , 可以很方便的认证用户数据 , 消息提示和页面重定向和前面的一样 , 其中

`Auth::user()`可以获取用户信息 .

```php
if (Auth::attempt($data)) {
    $user = Auth::user();
    session()->flash('success', "欢迎回来,{$user->name}!");
    return redirect()->route('users.show', [$user]);
} else {
    session()->flash('danger', '很抱歉,您的邮箱和密码不匹配');
    return redirect()->back();
}
```

**用户登录**

修改登录时的页面以及页面链接等 , 修改header公共模板 : 

```php
<header class="navbar navbar-fixed-top navbar-inverse">
    <div class="container">
        <div class="col-md-offset-1 col-md-10">
            <a href="{{ route('home') }}" id="logo">Lartisan Example</a>
            <nav>
                <ul class="nav navbar-nav navbar-right">
                    @if(Auth::check())
                        <li><a href="#">用户列表</a></li>
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                                {{ Auth::user()->name }} <b class="caret"></b>
                            </a>
                            <ul class="dropdown-menu">
                                <li><a href="{{ route('users.show', Auth::user()->id) }}">个人中心</a></li>
                                <li><a href="#">编辑资料</a></li>
                                <li class="divider"></li>
                                <li>
                                    <a id="logout" href="#">
                                        <form method="POST" action="{{ route('logout') }}">
                                            {{ csrf_field() }}
                                            {{ method_field('DELETE') }}
                                            <button type="submit" name="button" class="btn btn-block btn-danger">退出</button>
                                        </form>
                                    </a>
                                </li>
                            </ul>
                        </li>
                    @else
                        <li><a href="{{ route('help') }}">帮助</a></li>
                        <li><a href="{{ route('login') }}">登录</a></li>
                    @endif
                </ul>
            </nav>
        </div>
    </div>
</header>
```


