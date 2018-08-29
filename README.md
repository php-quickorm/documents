# PHP QuickORM 框架开发文档

版本：20180829

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
## 路由 

PHP QuickORM 框架的路由系统采用的是请求地址与控制器方法一一映射的方式，在请求地址中将会以 `/Controller/Method/Parameter` 的方式运行。具体例子如下：

```php
// 请求地址与方法映射关系示例

请求地址：http://localhost/api/article
请求方式：GET
响应方法：apiController 中的 articleGet() 方法

请求地址：http://localhost/api/article
请求方式：POST
响应方法：apiController 中的 articlePost() 方法

请求地址：http://localhost/api/article/1
请求方式：GET
响应方法：apiController 中的 articleGet() 方法，其中 1 作为该方法的参数：articleGet(1) 

请求地址：http://localhost/api/article/1
请求方式：POST
响应方法：apiController 中的 articlePost() 方法，其中 1 作为该方法的参数：articlePost(1) 

请求地址：http://localhost/api/article/1
请求方式：DELETE
响应方法：apiController 中的 articleDelete() 方法，其中 1 作为该方法的参数：articleDelete(1) 

```

其中若需要对方法传入多个参数，只需要改动请求的地址即可，具体例子如下：

```php
// 多参数示例

请求地址：http://localhost/api/article/1/description/200
请求方式：GET
响应方法：apiController 中的 articleGet() 方法，结合的参数之后为：articleGet(1, description, 200)

```

理论上 PHP 所能接受的所有 HTTP 方法，在 PHP QuickORM 框架中都可以使用，只需要新建好控制器以及方法即可。

## 数据库

PHP QuickORM 框架采用 PHP 的 `PDO` 接口链接数据库，在使用之前请确保按照本文 “*安装-使用配置-填写 MySQL 信息*” 所述正确配置数据库链接。

### 原生 PDO 操作

本框架支持直接调用 `PDO` 原生方法进行数据库操作，此处假定需要操作的表格为 `demo` 具体方法如下：

```php
$table = "demo";
$pdo = (new Database($table))->PDOConnect;

// 在这之后请继续你的 PDO 操作
$pdo->prepare(***);
```

### 执行 SQL 语句

若想要执行 SQL 语句，可以使用 `prepare()` 方法以及 `query()` 方法，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 直接执行 MySQL 语句（两个方法其实）效果一致，根据个人兴趣选择其一即可
$database->query("SELECT * FROM demo WHERE id=1");
$database->prepare("SELECT * FROM demo WHERE id=1");

// 使用预处理，推荐
$database->query("SELECT * FROM demo WHERE id=?",["id" => 1]);
$database->prepare("SELECT * FROM demo WHERE id=?",["id" => 1]);

// 取回结果中的第一行
$result = $database->fetch();
// 取回结果
$result = $database->fetchAll();

```

### 查询构造器(推荐)

在实际项目中，为了开发方便，推荐使用 PHP QuickORM 框架已经封装好的方法，以快速完成数据库相关操作

#### select() 方法

对字段进行选择

#### where() 方法

检索条件，传入可以为数组或者是 SQL 格式的条件，支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

$condition = [
  "id" 		=> 1,
  "name"	=> Rytia
]

// 传入条件数组
$result = $database->select("*")->where($conditon)->fetchAll();
// 传入原始 SQL 格式条件
$result = $database->select("*")->where('id="1" AND name="Rytia"')->fetchAll();

```

#### whereRaw() 方法

检索条件，传入可以为 SQL 格式的条件，效果与 where() 一致（只不过少了数组功能，兼顾熟悉 Laravel 框架的朋友快速上手），支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 传入原始 SQL 格式条件
$result = $database->select("*")->where('id="1" AND name="Rytia"')->fetchAll();

```

#### orWhere() 方法

或 检索条件，传入可以为数组或者是 SQL 格式的条件，支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

$condition1 = [
  "id" 		=> 1
]

$condition2 = [
  "name"	=> Rytia
]

// 传入条件数组
$result = $database->select("*")->where($conditon1)->orWhere($condition2)->fetchAll();
// 传入原始 SQL 格式条件
$result = $database->select("*")->where('id="1"')->orWhere('name="Rytia"')->fetchAll();

```

#### whereRaw() 方法

检索条件，传入可以为 SQL 格式的条件，效果与 orWhere() 一致（只不过少了数组功能，兼顾熟悉 Laravel 框架的朋友快速上手），支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 传入原始 SQL 格式条件
$result = $database->select("*")->whereRaw('id="1"')->orWhereRaw('name="Rytia"')->fetchAll();

```
