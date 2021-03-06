# 当前状态

> 各个访问地址在百度首页我的导航中记录 .
>
> 各个环境之间访问的密钥对都以配置完 .

### Laravel初始化

```
composer create-project laravel/laravel larabbs --prefer-dist "5.5.*"
```

### 使用Git版本控制

> 这里指Homestead中的git

#### **Git配置**

进入Homestead虚拟机中 , 配置Git用户名和邮箱 :

```
git config --global user.name "WebModel"
git config --global user.email headplan@163.com
```

设置 Git 推送分支时相关配置 , 执行`git push`没有指定分支时 , 自动使用当前分支 , 而不是报错 :

```
git config --global push.default simple
```

**Git初始化**

```
cd ~/Code/Laravel
git init
```

添加所有文件到Git中\(当然是除了`.gitignore`忽略的\)

```
git add -A
```

提交

```
git status
git commit -m "Initial commit"
git log
```

推送

```
git remote add origin git@github.com:<username>/larabbs.git
git push -u origin master
```

> 如果有误删 , 可以`git checkout -f`恢复 , 更多可以参考Git相关Note

### 同步到远程\(本地宿主机\)中的Gogs统一管理 . \(宿主:192.168.66.1\)

#### Gogs配置

Homestead在初始化时 , 通过 Homestaed.yaml 文件中的 `keys`选项 , 我们已经把专门的Homestead的 SSH Key 私钥复制到虚拟机中 , 这里需要将 SSH Key 添加到 ssh-agent 中 :

    $ eval `ssh-agent -s`
    $ ssh-add ~/.ssh/homestead

然后将公钥添加到 Gogs 账号即可 .

### 通过walle拉取Gogs中的版本部署到线上服务器 .

walle配置可以查看部署上线中walle的安装和Git项目配置 . 其中会出现问题的通常是目录权限问题 .

项目配置分为 :

* 测试环境 - 绑定本地host
* 线上环境 - 绑定域名

代码仓库 :

```
~Desktop/Server/PHP/deploy
./deploy
├── package/
│   ├── env/:线上线下的配置文件
│   └── vendor/:公共vendor
├── prod/:正式仓库
└── test/:测试仓库
```

高级任务 :

```
# post_deploy代码检出之后
# 复制vendor仓库:因为宿主在本地,这里直接复制
cp -r {WORKSPACE}/../../package/vendor {WORKSPACE}/vendor
# 生成.env文件:这里的env配置分为线上线下(正式和测试),即online和offline
cp {WORKSPACE}/../../package/env/offline.env {WORKSPACE}/.env

# post_release代码同步并创建链接之后
# 复制vendor
cp /home/www/vendor.zip {WORKSPACE}/
unzip {WORKSPACE}/vendor.zip
rm -f {WORKSPACE}/vendor.zip
# 生成App Key
php artisan key:generate
```

#### 共享目录

package目录修改 , 仅存放env文件 . 配置远程服务器MySQL配置 .

