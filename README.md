# PHP QuickORM 框架开发文档

版本：20180827

## 简介

PHP QuickORM 框架（[php-quickorm/framework](https://github.com/php-quickorm/framework)) 是一个可以帮助你快速完成 RESTful API 开发的轻量级 PHP 框架，提供了较为完善的 ORM 功能以及基本的路由系统、请求、回应、错误抛出功能类，遵循`Apache2`开源许可协议发布。

本框架具有以下几个特点：

- 遵循 `PSR-4` 规范，采用 `Composer` 管理，可快速引用其他 PHP 拓展包
- 采用 `PDO` 数据库接口，最多支持 12 种数据库
- 提供 `Collection` 数据类型，提供更为方便的数组操作
- 全中文注释代码，方便初学者观摩

适用范围：

- 为小程序、安卓 / iOS 应用或其他采用 `MVVM` 架构的 Web APP 快速开发 RESTful API 
- 对 PHP 有一定初步了解，想要进一步学习如何制作框架的学生、PHP初级工程师
- 配合 `Blade` 等模板引擎，开发商业管理系统
- 一切前后端分离的项目....

如果您喜欢 PHP QuickORM 框架，请移步 Github 给我们一个 Star :)

## 安装

### 推荐环境

| 环境       | 版本     | 备注                          |
| -------- | :----- | --------------------------- |
| PHP      | >= 7.0 | 本框架尚未在 PHP 5 进行具体测试，请自行斟酌版本 |
| composer | *      | *                           |
| MySQL    | 5      | 亦可使用其它 `PDO` 所支持数据库         |

> 推荐使用更加智能的 Jetbrains Phpstorm 作为本地开发的 IDE，即可完成自动引入、智能提示等操作

### 安装方法

- 采用 composer 安装

```shell
composer create-project php-quickorm/framework
```

- 采用源码安装

您可以从官网、Github 等下载 PHP QuickORM 源码，解压后进入项目根目录，并执行以下语句

```shell
composer create-project php-quickorm/framework
```
### 使用配置

安装完成之后，需要对数据库部分进行设置，此处以 MySQL 数据库为例。

#### 填写 MySQL 信息

进入 `System/DatabaseDriver` 文件夹，编辑 `pdo_mysql.php`文件

在其 24 行按照 `$host`, `$database`, `$user`, `$password` 的顺序对 `self::initialConnect`方法参数进行编辑
此处地址为 localhost，用户名为 root，密码为 password，数据库为 orm 进行配置如下：

```php
public function __construct(){
	$this->connect = self::initialConnect('localhost', 'orm', 'root', 'password');+
}
```
#### 修改框架链接

进入 `System` 文件夹，编辑 `Database.php`文件
确保 `Database` 类已经使用 `pdo_mysql` 创建链接，如下：
```php
public function __construct($table) {
	// 使用 pdo 进行处理
	$this->PDOConnect = (new pdo_mysql())->connect;
	$this->table = $table;
}
```


配置成功之后，将 `/` 或者 `Public` 目录设置为 apache、nginx 等服务器软件的网站根目录即可。

> 将 `/` 或者 `Public` 目录设置为根目录有何不同？
>
> 答：两者均能正常运行，并且实现原理一样。但是在默认情况下，将 `/` 设置为根目录，将默认开启PHP 层面的伪静态，而 `Public` 目录则默认不开启，需要单独为框架在 apache、nginx、iis 等服务器软件配置伪静态规则，详情参照下一小节。建议在生产环境中使用 `Public` 为根目录以方便统一静态文件的管理和确保安全。
>
> 至于是否开启框架自带的PHP 层面的伪静态，只需要编辑这两个目录下的 `index.php` 文件，修改 `Route::initialize` 方法的第三个参数即可，其中 `true` 表示开启，`false` 表示关闭。

### 伪静态

PHP QuickORM 框架的伪静态规则和 Laravel 等一致，可以相互通用。

*注意：若使用服务器软件的伪静态规则，请确保框架本身PHP层伪静态已经关闭。*

#### Ngxin 规则

```nginx
location / {
	try_files $uri $uri/ /index.php?$query_string;
}
```
#### Apache 规则

```apache
<IfModule mod_rewrite.c>
<IfModule mod_negotiation.c>
    Options -MultiViews -Indexes
</IfModule>
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
</IfModule>

```