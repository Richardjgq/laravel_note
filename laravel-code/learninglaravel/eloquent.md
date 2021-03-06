# Eloquent

Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库进行交互。每个数据库表都有一个对应的「模型」可用来跟数据表进行交互。你可以通过模型查询数据表内的数据，以及将记录添加到数据表中。

#### 定义模型

所有的 Eloquent 模型都继承自`Illuminate\Database\Eloquent\Model`类 .

**创建模型实例**

```
php artisan make:model Flight
php artisan make:model Flight --migration
php artisan make:model Flight -m
```

**测试表**

```php
Schema::create('flights', function (Blueprint $table) {
    $table->increments('id');
    $table->string('slug')->unique();
    $table->string('title');
    $table->text('content');
    $table->timestamp('published_at')->index();
    $table->timestamps();
});
```

**Eloquent 模型约定**

Eloquent模型默认情况下会使用类的「下划线命名法」与「复数形式名称」来作为数据表的名称生成规则 , 例如 :

* Article 数据模型类对应`articles`表；
* User 数据模型类对应`users`表；
* BlogPost 数据模型类对应`blog_posts`表；
* Flight 数据模型类对应`flights`表；

**配置约定**

* 数据表名称 : 自定义表名`protected $table = 'my_flights';`
* 主键 : 自定义主键名`$primaryKey = 'my_id';`
  * 使用非递增或者非数字的主键 : `public $incrementing = false;`
* 时间戳 : 
  * 关闭自动维护`created_at`和`updated_at`字段 , `public $timestamps = false;`
  * 定义时间戳格式以及当模型被序列化成数组或 JSON 格式 , `protected $dateFormat = 'U';`
  * 自定义用于存储时间戳的字段名 : 
    * `const CREATED_AT = 'creation_date';`
    * `const UPDATED_AT = 'last_update';`
* 数据库连接 : 自定义数据库连接`protected $connection = 'connection-name';`

#### 取回多个模型

一旦你创建并关联了一个模型到数据表上 , 那么就可以从数据库中获取数据 .

```
$flights = App\Flight::all();

foreach ($flights as $flight) {
    echo $flight->title;
}
```

**增加额外的限制**

每个 Eloquent 模型都可以当作一个查询构造器 , 可以使用其所有方法 .

```
$flights = App\Flight::where('my_id', '>',3)
    ->orderBy('title', 'desc')
    ->get();
```

**集合**

类似`all`以及`get`之类的可以取回多个结果的Eloquent方法 , 将会返回一个`Illuminate\Database\Eloquent\Collection`实例 .

`Collection`类提供多种辅助函数来处理Eloquent结果 .

```
$flights = App\Flight::all();

$flights = $flights->reject(function ($flight) {
    echo $flight->title."<br>";
});
```

**分块结果**

需要处理数以千计的 Eloquent 查找结果 , 则可以使用`chunk`命令 . `chunk`方法将会获取一个 Eloquent 模型的「分块」, 并将它们送到指定的`闭包 (Closure)`中进行处理 . 当在处理大量结果时 , 使用`chunk`方法可节省内存 :

```
App\Flight::chunk(2, function ($flights) {
    foreach ($flights as $flight) {
        echo $flight->title."<br>";
    }
});
```

传递到方法的第一个参数表示每次「分块」时你希望接收的数据数量 . 闭包则作为第二个参数传递 , 它将会在每次从数据取出分块时被调用 .

**使用游标**

`cursor`允许你使用游标来遍历数据库数据 , 一次只执行单个查询 . 在处理大数据量请求时`cursor`方法可以大幅度减少内存的使用 :

```
foreach (App\Flight::where('my_id', '>', 1)->cursor() as $flight) {
    echo $flight->title."<br>";
}
```

#### 取回单个模型或集合

可以通过`find`和`first`方法来取回单条记录 . 但这些方法返回的是单个模型的实例 , 而不是返回模型的集合 :

```
$flight = App\Flight::find(1);
    echo $flight->title."<br>";
$flight = App\Flight::where('my_id', '>', 1)->first();
    echo $flight->title."<br>";
$flights = App\Flight::find([1, 2, 3]);
    dump($flights);
```

**「未找到」异常**

`findOrFail`以及`firstOrFail`方法会取回查询的第一个结果 . 如果没有找到相应结果 , 则会抛出一个

`Illuminate\Database\Eloquent\ModelNotFoundException`

如果该异常没有被捕获 , 则会自动返回 HTTP`404`响应给用户 , 不用再编写 .

```php
try{
    $model = App\Flight::findOrFail(20);
} catch (Illuminate\Database\Eloquent\ModelNotFoundException $e) {
    echo $e->getMessage()."<br>";
} finally {
    echo "没找到数据<br>";
}

try{
    $model = App\Flight::where('my_id', '>', 100)->firstOrFail();
} catch (Illuminate\Database\Eloquent\ModelNotFoundException $e) {
    echo $e->getMessage()."<br>";
} finally {
    echo "也没找到数据<br>";
}
```

**其他聚合函数**

使用`count`、`sum`、`max`等 , 返回计算后的标量 :

```
$count = App\Flight::all()->count();
dump($count);
$max = App\Flight::all()->max('my_id');
dump($max);
$sum = App\Flight::all()->sum('my_id');
dump($sum);
```

#### 添加和更新模型

**基本添加**

要在数据库中创建一条新记录 , 只需创建一个新模型实例 , 并在模型上设置属性和调用`save`方法即可 :

```
$flight = new App\Flight;
$flight->slug = str_random(10);
$flight->title = str_random(10);
$flight->content = bcrypt('secret');
$flight->save();
```

**基本更新**

`save`方法也可以用于更新数据库中已经存在的模型 . `updated_at`时间戳将会被自动更新 .

```
$flight = App\Flight::find(1);
$flight->title = 'New Flight Name';
$flight->save()
```

**批量更新**

使用`update`方法针对符合指定查询的任意数量模型进行更新 .

> 当通过“Eloquent”批量更新时 , `saved`和`updated`模型事件将不会被更新后的模型代替 .

```
App\Flight::where('my_id', '>', 1)
    ->where('my_id', '<',4)
    ->update(['title' => 'aaaaa']);
```

**批量赋值**

使用`create`方法通过一行代码来保存一个新模型 , 被插入数据库的模型实例也会返回 . 但是 , 所有的Eloquent模型都针对批量赋值\(Mass-Assignment\)做了保护 .

> 批量赋值漏洞 , 当用户通过 HTTP 请求传入了非预期的参数，并借助这些参数更改了数据库中你并不打算要更改的字段，这时就会出现批量赋值（Mass-Assignment）漏洞 .

所以 , 在开始之前 , 你应该定义好哪些模型属性是可以被批量赋值的 . 定义白名单或黑名单 .

```
# 白名单,可以被批量复制的列
protected $fillable = ['slug','title'];
# 黑名单,不可以被批量复制的列,为[]表示都可以被批量赋值,默认为[*],表示都不可以被批量复制,也就是批量复制保护
protected $guarded = ['content'];
```

```
$flight = App\Flight::create(['title' => 'Flight 101']);
```

如果已经有了一个**model模型** , 可以使用fill\(\)方法批量复制 , 当然也要定义白名单 .

```
$flight = new App\Flight;
$flight->fill(['slug' => str_random()]);
echo "成功创建数据<br>";
```

**其它创建的方**

`firstOrCreate`和`firstOrNew`方法也可以通过属性批量赋值创建模型 . 他们都会尝试寻找数据库中的记录 , 如果在数据库中找不到模型 , 则会使用指定的属性来添加一条记录 . 不同的是`firstOrnew`返回的模型还尚未保存到数据库 , 需要通过手动调用`save`方法来保存它 :

```php
$flight = App\Flight::firstOrCreate(['title' => 'new title']);

$flight = App\Flight::firstOrCreate(
    ['title' => 'new title2'], ['slug' => str_random()]
);

$flight = App\Flight::firstOrNew(['title' => 'new title33333']);
$flight->save();

$flight = App\Flight::firstOrNew(
    ['title' => 'new title444444'], ['slug' => str_random()]
);
$flight->save();
```

还可以使用`updateOrCreate`方法 , 让模型已经存在则更新 , 否则创建新模型的情形 , 用法和`firstOrCreate`一样 , 第一个参数数组为条件 , 第二个参数数组为要修改大数据 :

```php
$flight = App\Flight::updateOrCreate(
    ['title' => 'new title444444'],
    ['content' => 999]
);
```

#### 删除模型

要删除模型，必须在模型实例上调用`delete`方法 :

```
$flight = App\Flight::find(77);
$flight->delete();
```

**通过主键来删除现有的模型**

```
App\Flight::destroy([1, 2, 3]);
App\Flight::destroy(4, 5, 6);
```

**通过查询来批量删除模型**

查询结果中被删除的模型 , 不会报错 .

```
$deletedRows = App\Flight::where('my_id', '>=', 7)
        ->where('my_id', '<=', 10)
        ->delete();
```

返回被删除数量 . where没有查到则返回0 .

因为批量删除时 , deleting和deleted模型事件不会在被删除模型实例上触发 , 删除语句执行时 , 不会检索回模型实例 .

#### 软删除

软删除在Laravel中也有约定可用 , 要在模型上启动软删除 , 则必须在模型上使用`Illuminate\Database\Eloquent\SoftDeletes`trait 并添加`deleted_at`字段到`$dates`属性上 :

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;

    /**
     * 需要被转换成日期的属性。
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}
```

当然 , 数据库中也需要添加一列 :

```
php artisan make:migration add_deleted_at_column_flights --table=flights
```

```
Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
```

现在 , 当在模型上调用`delete`方法时\(当然destroy\(\)方法也可以\) , `deleted_at`字段将会被设置成目前的日期和时间 . 而且 , 当查询有启用软删除的模型时 , 被软删除的模型将会自动从所有查询结果中排除 .

确认指定的模型实例是否已经被软删除使用

```
if ($flight->trashed()) {
    //
}
```

**软删除的模型操作**

包含被软删除的模型

```
$flights = App\Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

关联查询

```
$flight->history()->withTrashed()->get();
```

只取出软删除数据

```
$flights = App\Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

恢复被软删除的模型

可以在模型实例上使用`restore`方法 , 也可以在查询上批量恢复

```
App\Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

```
$flight->history()->restore();
```

永久删除模型

开启了软删除约定后 , 使用forceDelete\(\)方法进行硬删除 :

```
$flight->forceDelete();
```

#### 查询作用域

**全局作用域**

全局作用域允许我们为给定模型的所有查询添加条件约束 . 例如前面的软删除功能 .

**编写全局作用域**

定义一个实现`Illuminate\Database\Eloquent\Scope`接口 , 可以在app文件夹下创建Scopes文件夹存放 可以自动加载即可 .

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class MyidScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        return $builder->where('my_id', '>', 6);
    }
}
```

时间了接口之后 , 就可以在模型类中使用了 :

```php
protected static function boot()
{
    parent::boot();
    static::addGlobalScope(new MyidScope);
}
```

现在的查询 , 就会加上这个全局的条件 .

**匿名的全局作用域**

作用域可以使用使用闭包配置 :

```php
protected static function boot()
{
    parent::boot();
    //static::addGlobalScope(new MyidScope);
    static::addGlobalScope('my_id', function (Builder $builder) {
        return $builder->where('my_id', '>=', 1)
            ->where('my_id', '<=', 100);
    });
}
```

**移除全局作用域**

在操作模型时使用`withoutGlobalScope()`方法移除作用域方法

```php
# 移除所有
User::withoutGlobalScopes()->get();
# 移除单个
User::withoutGlobalScope(AgeScope::class)->get();
# 移除所有
User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();
# 移除闭包
User::withoutGlobalScope('my_id')->get();
```

事件

