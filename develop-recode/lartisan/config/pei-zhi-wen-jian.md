# 配置文件

#### 配置env文件

根据不同环境配置 :

* 本地开发环境
* 测试环境
* 生产环境

```
APP_ENV   # 来设定当前应用的运行环境local,testing,production
APP_DEBUG # 来设定是否在应用报错时显示调试信息
APP_KEY   # 来生成应用的密钥用于加密一些较为敏感的数据
```

对数据库的连接方式、数据库名、用户名密码等做相关配置 :

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=sample
DB_USERNAME=homestead
DB_PASSWORD=secret
```

缓存、会话、队列等驱动的相关配置信息 :

```
CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync
```

等等 .

---

#### 配置config/app.php

基本配置都在env文件中 , 这里暂时只修改UTC时区和本地语言 .

```
'timezone' => 'PRC',
'locale' => 'zh',
```


