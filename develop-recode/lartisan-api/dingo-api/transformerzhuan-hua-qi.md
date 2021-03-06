# Transformer转化器

通过转化器 , 将对象转化为数组 , 并强制转化整型和布尔类型 , 包括分页结果和嵌套关联 .

这里主要讨论转化器及其使用 , 这里的转化器包括以下两层意思 :

* 转化层（transformation layer）是一个准备和处理转化器的类
* 转化器（transformer）是一个获取原始数据并将其转化为数组格式的类 , 处理器的处理方式取决于转化层 . 

#### 使用转化器

有多种方式使用转化器 :

**注册转化器**

当你为某个类注册转化器后就可以从路由返回这个类\(假设它可以被转化为数组\)并自动经过转化器处理 :

```php
app('Dingo\Api\Transformer\Factory')->register('User', 'UserTransformer');
```

**在响应构建器中使用**

我们前面已经创建了Transformer用来转换数据 . 参考Response响应中的内容 . 

