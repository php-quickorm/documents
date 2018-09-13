# PHP QuickORM 框架开发文档

版本：20180913

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
| :------- | :----- | :-------------------------- |
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
	$this->connect = self::initialConnect('localhost', 'orm', 'root', 'password');
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

## 请求

PHP QuickORM 框架针对每一次请求，都会生成一个 Request 类的对象，在开发过程中关于请求的信息均可从中获取。

### 请求内容

#### 全部字段

获取请求的全部字段，包括但不限定 `GET` 、`POST` 等方法。
返回为 PHP `Array` 数组，在 Controller 控制器层的操作具体如下：

```php
// 假设此时访问 /?info=test

$content = $this->request()->all(); // $content['info'] = test
```

#### 单个字段

获取请求中某个字段的内容，包括但不限定 `GET` 、`POST` 等方法。
返回为字符串或数组，且对字段的获取与请求方法无关。在 Controller 控制器层的操作具体如下：
```php
// 假设此时以 GET 方式访问 /?info=test

$content = $this->request('info'); // test

// 假设此时以 POST 方式发送 user=demo

$content = $this->request('user'); // demo

// 假设此时以 DELETE 方式发送 user=demo 到 /?info=test

$content = $this->request('user'); // demo
$content = $this->request('info'); // test

// 以下几种方法效果一致，可根据习惯使用：
$content = $this->request()->get('user'); // demo
$content = $this->request()->get('info'); // test
```

### 请求方法

获取请求的方法。具体如下：

```php
$method = $this->request()->getMethod();
```

### 请求路径

#### 目录地址

获取返回的目录地址。具体如下：

```php
// 假设访问 /hello/word/?info=test
$method = $this->request()->getPath(); // /hello/word/
```

#### 请求链接
获取返回的请求链接。具体如下：

```php
// 假设访问 /hello/word/?info=test
$method = $this->request()->getUrl(); // /hello/word/?info=test
```

## 响应

在项目开发过程中，针对每一个请求，都必须做出合理的响应。作为针对 `MVVM` 架构而生的 PHP QuickORM，响应方法如下。

### JSON 数据

直接返回 JSON 格式的数据，并返回 HTTP 状态码。具体如下：

```php
$data = ["text" => "hello word"];

// 直接返回 JSON 格式的数据，默认 HTTP 状态码为 200
return $this->response()->json($data);

// 直接返回 JSON 格式的数据，并设置 HTTP 状态码为 404
return $this->response()->json($data, '404');
```

### JSON 对象

返回 JSON 格式的对象，包含 `errcode`、`errmsg`、`data` 三个属性，常用于使前后端交互更加规范化，具体如下：

```php

$data = ["text" => "hello word"];

// 返回 JSON 对象，默认 HTTP 状态码为 200，errcode 为 0，errmsg 为空
return $this->response($data);

// 返回 JSON 对象，并设置 HTTP 状态码为 404，且保持默认的 errcode 为 0，errmsg 为空
return $this->response($data, '404');

// 返回 JSON 对象，并设置 HTTP 状态码为 404，并设置 errcode 为 500001，errmsg 为 hello
return $this->response($data, '404', '500001', );

// 以下几种方法效果一致，可根据习惯使用
return $this->response()->dataEcode($data);
return $this->response()->dataEcode($data, '404');
return $this->response()->dataEcode($data, '404', '500001', );

// JSON 对象示例
//    {
//      "errcode": "0",
//      "errmsg": "null",
//      "data": {
//          "text": "hello word"
//      }
//    }
```

若您的数据调用了 `paginate()` 方法实现分页，则应该使用 JSON 分页对象，具体表现为在返回对象增加了 `page` 属性，如下：

```php
$data = Demo::piginate(3);

// 返回 JSON 对象，默认 HTTP 状态码为 200，errcode 为 0，errmsg 为空
return $this->response()->pageEncode($data);

// JSON 分页对象示例
//	{
// 		"errcode": 0,
//		"errmsg": "",
//		"data": [{
//			"id": "1",
//			"title": "嘿！Every Body @~@",
//			"content": "测试的内容！！",
//			"author": "3"
//		}, {
//			"id": "2",
//			"title": "还是标题",
//			"content": "这是无敌的测试",
//			"author": "Rytia"
//		}, {
//			"id": "4",
//			"title": "标题啦啦",
//			"content": "这是依旧是无敌的测试",
//			"author": "Rytia"
//		}],
//		"page": {
//			"currentPage": 1,
//			"totalItems": 14,
//			"totalPages": 5,
//			"hasNext": "true",
//			"nextUrl": "\/api\/test\/1?page=2"
//		}
//	}
```

### 重定向

使用 302 重定向至其他链接，具体如下：

```php
return $this->response()->redirect("http://www.github.com/php-quickorm");
```

### 响应头部

用于响应请求时，增加额外的 `responose header` 信息，支持链式操作。具体如下：

```php
$data = ["text" => "hello word"];

// 分步操作
$this->response()->setHeader("Powered_By", "php-quickorm");
return $this->response()->json($data);

// 链式操作
return $this->response()->setHeader("Powered_By", "php-quickorm")->json($data);

```

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

检索条件。
传入可以为数组或者是 SQL 格式的条件，支持链式操作，具体如下：

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

检索条件。
传入为 SQL 格式的条件，效果与 where() 一致（只不过少了数组功能，兼顾熟悉 Laravel 框架的朋友快速上手），支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 传入原始 SQL 格式条件
$result = $database->select("*")->where('id="1" AND name="Rytia"')->fetchAll();

```

#### orWhere() 方法

或 检索条件。
传入可以为数组或者是 SQL 格式的条件，支持链式操作，具体如下：

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

#### orWhereRaw() 方法

或 检索条件。
传入为 SQL 格式的条件，效果与 orWhere() 一致（只不过少了数组功能，兼顾熟悉 Laravel 框架的朋友快速上手），支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 传入原始 SQL 格式条件
$result = $database->select("*")->whereRaw('id="1"')->orWhereRaw('name="Rytia"')->fetchAll();

```

#### join() 方法

用于根据两个或多个表中的列之间的关系查询数据。
其中 method 可选 `left`，`right`，`full`，`inner`，且默认为 `inner`，支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

$result = $database->select('DISTINCT zhong.name,wiki.coordinate')
					->join('zhong', 'full')
                    ->on('wiki.zhong=zhong.id')
                    ->fetchAll();
```

#### on() 方法

根据关系查询数据表。
传入为 SQL 格式的条件，支持链式操作，具体如下：

```php
$table = "demo";
$database = new Database($table);

$result = $database->select('*')->on('id="1"')->fetchAll();

// 此方法更常见于与 join() 配合使用
$result = $database->select('DISTINCT zhong.name,wiki.coordinate')
					->join('zhong', 'full')
                    ->on('wiki.zhong=zhong.id')
                    ->fetchAll();
```

#### orderBy() 方法

根据字段排列结果集。
传入为排列所依据的字段名以及排序方式，排序方式为 ASC（顺序，默认）与 DESC （倒序）

```php
$table = "demo";
$database = new Database($table);

$result = $database->select('*')->orderBy("id", "DESC")->fetchAll();
```

#### insert() 方法

添加一条新的数据。
传入为关联数组，返回为是否成功的布尔值，具体如下：

```php
$table = "demo";
$database = new Database($table);

$data = [
  "title" => "测试标题",
  "content" => "测试内容",
  "author" => "Rytia"
]
$result = $database->insert($data);

```

#### update() 方法

更新数据表中的数据。
传入为关联数组，返回为是否成功的布尔值，更新前应该先使用 where 等条件语句进行定位，具体如下：

```php
$table = "demo";
$database = new Database($table);

$data = [
  "title" => "测试标题",
  "content" => "测试内容",
  "author" => "Rytia"
]
$result = $database->where(["id" => 1])->update($data);
```

#### delete() 方法

删除数据表中的数据。
返回为是否成功的布尔值，更新前应该先使用 where 等条件语句进行定位，具体如下：

```php
$table = "demo";
$database = new Database($table);

$result = $database->where(["id" => 1])->delete();
```

#### paginate() 方法

数据库分页，内部通过调用 SQL LIMIT 实现。
建议在 Controller 层调用，需使用 `GET` 方式传入 `page` 字段用于展示第 n 页。 返回为 Collection 集合（具体参照文档 Collection 一章），传入的第一个参数为每页所展示的条数，第二个参数为是否开启分页的相关信息输出，默认为 true。开启后会展示当前页码、总计条数、总计页数以及是否还有下一页等信息，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 当 where() 方法接受空数组时，将返回全部数据（相当于传入条件为空
// setModel() 方法将为该数据表指定一个模型，而返回的 Collection 集合中元素便是该模型的实例 （具体参照文档 Model 一章）
$result = $database->where([])->setModel(Demo::class)->paginate(3);

// 通过 dd 函数调试结果
dd($result)

// 此时请求 /?page=1，则会返回以下信息
System\Collection Object
(
    [collectionItems:protected] => Array
        (
            [0] => Model\Demo Object
                (
                    [objectData:protected] => Array
                        (
                            [id] => 1
                            [title] => 嘿！Every Body @~@
                            [content] => 测试的内容！！
                            [author] => Rytia！
                            [created_at] => 2018-08-08 00:00:00
                            [updated_at] => 2018-08-09 00:00:00
                        )

                )

            [1] => Model\Demo Object
                (
                    [objectData:protected] => Array
                        (
                            [id] => 2
                            [title] => 还是标题
                            [content] => 这是无敌的测试
                            [author] => Rytia
                            [created_at] => 0000-00-00 00:00:00
                            [updated_at] => 0000-00-00 00:00:00
                        )

                )

            [2] => Model\Demo Object
                (
                    [objectData:protected] => Array
                        (
                            [id] => 4
                            [title] => 标题啦啦
                            [content] => 这是依旧是无敌的测试
                            [author] => Rytia
                            [created_at] => 2018-08-22 00:00:00
                            [updated_at] => 
                        )

                )

        )

    [collectionPages] => stdClass Object
        (
            [currentPage] => 1
            [totalItems] => 14
            [totalPages] => 5
            [hasNext] => true
            [nextUrl] => /api/test/2?page=2
        )

)

```

该方法由于结合了 Collection 集合以及 Model 模型的相关内容，如不理解建议阅读完全文档后实操。

#### fetch() 方法

执行 SQL 语句并取回第一条结果。

#### fetchAll() 方法

执行 SQL 语句并取回结果集。

#### get() 方法
执行以及构造好的 SQL 语句。
返回为 Collection 集合。
上面几个构造查询的方法中，均使用了 fetchAll() 方法取出数据表中的数据，而本方法与 fetchAll() 有异曲同工之处，且在 Model 模型已经指定的情况下可以相互替换。

> `get()`、`fetch()` 与 `fetchAll()` 之间有什么区别？
> 1. 三者功能均为根据已经构造好的条件从数据表中取出数据。
> 2. `get()` 在使用之前，必须已经为查询构造器设置好 Model 模型，包括但不限于以下几种形式：
>   a. `Demo::where([ ])->get()`
>   b. `(new Database("tableName"))->setModel(Demo::class)->where([ ])->get()`
>   c. `Database::table("tableName")->setModel(Demo::class)->where([ ])->get()`
>   d. `Database::model(Demo::class)->where([ ])->get()`
> 3. `get()` 返回为相应对象数组，即为 Collection 实例；`fetch()` 返回为关联数组，且是数据表中符合条件的第一条数据；`fetchAll()` 返回为关联二维数组，与 `get()` 返回结果相同但形式不同。

### 事务操作

PHP QuickORM 框架支持数据库事务操作，内部亦是通过对 `PDO` 的封装，具体如下：

```php
$table = "demo";
$database = new Database($table);

// 开始一个事务
$database->beginTransaction();

$data = [
  "title" => "测试标题",
  "content" => "测试内容",
  "author" => "Rytia"
]
$result = $database->where(["id" => 1])->update($data);

// 提交事务
$database->commit();
```

以上代码并没有使用事务回滚，在具体项目中，我们更建议您这样做：

```php
$table = "demo";
$database = new Database($table);

try{
	// 开始一个事务
	$database->beginTransaction();

	$data = [
  		"title" => "测试标题",
  		"content" => "测试内容",
  		"author" => "Rytia"
	];
	$result = $database->where(["id" => 1])->update($data);

	// 提交事务
	$database->commit();
	
} catch (Exception $e) {

	// 操作失败，事务回滚
	$database->rollBack();
  	trigger_error("Failed: " . $e->getMessage(), E_USER_ERROR);
  	
}
```
## 集合

PHP QuickORM 框架提供了一种新的数据类型 `Collection`，是对 PHP 序列数组的二次封装，以便于支持数据聚合功能，以及对对象的储存。

### 创建与判断

Collection 集合的创建支持两种方式，内部实现效果一致。具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 通过静态方法创建集合
$collection = Collection::make($array);

// 判断变量是不是 Collection 集合
if($object instanceof Collection){
  
}

// 判断集合是否为空
if($collection->isEmpty()){
  
}
```

### 栈的实现

Collection 集合可以很方便的实现栈的操作（即后进先出），具体如下：

```php
$collection = new Collection();

// 元素进栈
$collection->push("1");
$collection->push("2");

// 元素出栈
echo $collecton->pop(); // 显示 2

```

### 队列实现

Collection 集合可以很方便的实现队列的操作（即先进先出），具体如下：

```php
$collection = new Collection();

// 元素入队
$collection->push("1");
$collection->push("2");

// 元素出队
echo $collecton->shift(); // 显示 1

// 插入元素到队列头部
$collection->unShift("3");

```

### 数据聚合

Collection 自带多种数据聚合方法，分别有 `first()`，`last()`，`max()`，`min()`，`average()`，`sum()`，`count()`

#### 序列数组

当 Collection 集合中所储存的数据均为数值时，数据聚合功能具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 显示第一个元素（1）
echo $collection->first();

// 显示最后一个元素（5）
echo $collection->last();

// 显示最大值（5）
echo $collection->max();

// 显示最小值（1）
echo $collection->min();

// 显示平均值（3）
echo $collection->average();

// 显示数值总和（15）
echo $collection->sum();

// 显示集合中元素个数
echo $collection->count();

```

#### 实例集合

实例集合，即是集合中储存的内容为模型实例化之后的对象，可以跳至本文档 Model 模型部分的聚合查看。

### 集合合并

将两个集合合并为一个大集合。具体如下：

```php
$array1 = ["1", "2", "3", "4", "5"];
$array2 = ["6", "7", "8", "9", "10"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array1);

// 将集合与数组合并
$collection->merge($array2);

// 将集合与集合合并
$collectionTemp = new Collection($array2);
$collection->merge($collectionTemp);
```


### 集合包含

检测元素是否在集合中或某集合是否为该集合的子集。具体如下：

```php
$array1 = ["1", "2", "3", "4", "5"];
$array2 = ["1", "2", "3"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array1);

// 判断元素是否在集合中
$result = $collection->contains("1");

// 判断集合/数组是否为当前集合子集
$result = $collection->contains($array2);

```

### 集合排序

#### 序列数组

当 Collection 集合中所储存的数据均不为对象时，可以看作一个普通序列数组。
其默认排序方式为升序（ASC），默认排序方法为 PHP 自带的 `asrot` (底层由变种的快排实现，时间复杂度为 `o(NLogN))`，若调用其余排序方法请传入参数 `sortFunction`。具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 升序排序
$collection->sort();

// 降序排序
$collection->sort("DESC");

// 采用其他 PHP 排序方法，这里使用 ksort() 为例
$collection->sort("DESC", "ksort");

// 采用其他需要回调函数的 PHP 排序方法，这里以 usort() 为例
$collection->sort("", "usort", function($a, $b){
	// 排序函数主体
});

```

#### 实例集合

当集合中储存的内容为模型实例化之后的对象，可以跳至文档 Model 模型部分的 数据排序 查看。


### 集合分页

Collection 集合自带分页功能。
建议在 Controller 层调用，需使用 `GET` 方式传入 `page` 字段用于展示第 n 页。 返回为 Collection 集合，传入的第一个参数为每页所展示的条数，第二个参数为是否开启分页的相关信息输出，默认为 true。开启后会展示当前页码、总计条数、总计页数以及是否还有下一页等信息，具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 通过 dd 函数调试结果
$result = $collection->paginate(2);
dd($result);

// 此时请求 /?page=1，则会返回以下信息
System\Collection Object
(
    [collectionItems:protected] => Array
        (
            [0] => "1"
            [1] => "2"
            [2] => "3"

        )

    [collectionPages] => stdClass Object
        (
            [currentPage] => 1
            [totalItems] => 5
            [totalPages] => 2
            [hasNext] => true
            [nextUrl] => /api/test/2?page=2
        )

)

```

### 集合迭代

Collection 集合实现了 `ArrayAccess` 和 `IteratorAggregate` 接口，可以直接使用 foreach 进行遍历，且诸多使用方法与 PHP 数组相同。具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 使用 foreach 遍历
foreach ($collection as $item) {
    echo $item;
}

// 使用 for 遍历
for ($i = 0; $i < $collection->count(); $i++) { 
	echo $collection[$i];
}
```

### 格式转换

Collection 集合实例可以向其他形式转换，以方便开发使用。

#### JSON 

当强制以字符串形式操作 Collection 集合时，将会自动转化为 JSON 格式输出，同时亦提供 toJSON() 方法进行转换。具体如下：

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 显示输出 JSON 
echo $collection;

// 转化为 JSON 格式
$result = $collection->toJson();

```

#### 普通数组

```php
$array = ["1", "2", "3", "4", "5"];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 转化为普通数组
$newArray = $collection->toArray();
```

#### 实例集合

符合条件(字段齐全)的关联数组可以直接格式化为某个 Model 的对象实例集合。具体如下：

```php
$array = [
	[
		"id" 		=>	"1",
		"title"		=>	"测试标题",
		"content"	=>	"测试内容",
		"author"	=>	"Rytia"
	],
	[
		"id" 		=>	"2",
		"title"		=>	"测试标题",
		"content"	=>	"测试内容",
		"author"	=>	"Rytia"
	]
];

// 通过创建新对象的方式创建集合
$collection = new Collection($array);

// 转换为 Demo 模型的实例集合
$collection->format(Demo::class);
```

## 模型

PHP QuickORM 框架在 Model 模型层提供整套`ORM` 相关方法，可以方便的实现 `CURD` ，助你完成开发。

### 定义

这里以 `Demo` 为例定义一个新的模型。

我们需要将其命名为 `Demo.php` 并放置于 `App\Model` 目录下，具体内容如下：

```php
namespace Model;
use System\Model;

class Demo extends Model
{
    // 数据表名称
	public static $table = 'demo';
  
}
```

接着按照自己项目实际需要，对数据表 `demo` 进行定义。本文档的模型演示中数据表字段如下：

- id, int, AUTO_INCREMENT
- title, varchar(255)
- content, varchar(255)
- author, varchar(255)

### 新增

新添加一条数据的方法有多种，具体如下：

使用模型静态方法新增数据

```php
$data = [
	'title'		=>	'测试标题',
  	'content'	=>	'测试内容',
  	'author'	=>	'Rytia'
];
Demo::create($data);
```

通过实例化新增数据
```php
$data = [
	'title'		=>	'测试标题',
  	'content'	=>	'测试内容',
  	'author'	=>	'Rytia'
];

$instance = new Demo($data);
$instance->save();
```

实例化之后通过赋值新增数据
```php
$instance = new Demo();
$instance->title = '测试标题';
$instance->content = '测试内容';
$instance->author = 'Rytia';
$instance->save();
```

若想通过所接收的  `POST` 内容来新增数据，可以使用如下写法：
```php
// 使用 PHP QuickORM 框架的 Request 请求类
Demo::create($this->request->all());

// 使用 PHP POST 数组
Demo::create($_POST);
```

### 修改

修改一条数据的方法主要有以下多种，具体如下：

使用模型静态方法修改数据
```php
// 找到 ID 为 1 的那行数据
$instance = Demo::find(1);
  
// 将标题修改为 demo
$data = ['title' => 'demo'];
$instance->update($data);
```

实例化之后通过赋值修改数据
```php
// 找到 ID 为 1 的那行数据
$instance = Demo::find(1);
  
// 将标题修改为 demo
$instance->title = 'demo';
$instance->save();
```

使用数据库方法批量修改数据

```php
// 找到 title 为 demo 的多行数据
$instance = Demo::where(['title' => 'demo']);
  
// 将内容修改为 text
$instance->update(['content' => 'text']);
```

### 删除

删除一条数据的方法主要有以下多种，具体如下：

实例化之后执行删除方法

```php
// 找到 ID 为 1 的那行数据
$instance = Demo::find(1);
  
// 将这行数据删除
$instance->delete();
```

使用数据库方法批量删除数据

```php
// 找到 title 为 demo 的多行数据
$instance = Demo::where(['title' => 'demo'])；
  
// 将符合要求的多行数据
$instance->delete();
```

### 查询

PHP QuickORM 框架支持多种查询方法。
其中使用静态方法查询将会返回一个对应模型的实例或者实例集合，可以参考本文档中的“集合”一章进行操作。
而采用查询构造器将返回 `Database` 对象，可参考本文档中 “数据库” 一章进行操作。您亦可以在查询构造器的链式操作中加上 `get()` 方法将其转换为 `Collection` 对象（即转换为实例集合），从而参考本文档中的“集合”一章进行操作。

#### 静态查询

静态查询即通过模型提供的静态方法进行查询，具体如下：

```php
// 找到 ID 为 1 的一行数据
$instance = Demo::find(1);

// 搜索 titile 中含有 hello 的数据
$collection = Demo::search("title","%hello%");

// 显示全部数据
$collection = Demo::all();
```

#### 查询构造器

通过模型的 `where()`、`whereRaw()`、`raw()`  方法，可以直接调用数据库的查询构造器，具体如下：

```php
// 查询构造器演示，更多方法请参照本文档“数据库”一章

$database	= Demo::where(["title" => "测试标题"])->orWhere(["title" => "演示标题"]);
$collection	= Demo::where(["title" => "测试标题"])->orWhere(["title" => "演示标题"])->get();

$database	= Demo::whereRaw('title="测试标题"')->orderBy("id", "DESC");
$collection	= Demo::whereRaw('title="测试标题"')->orderBy("id", "DESC")->get();

$database	= Demo::raw("SELECT * FROM {table} WHERE id=1");
$collection = Demo::raw("SELECT * FROM {table} WHERE id=1")->get();
```

#### 查询分页

PHP QuickORM 框架为模型提供了 `paginate()` 方法，可以将当前模型全部数据通过分页输出。
该方法需使用 `GET` 方式传入 `page` 字段用于展示第 n 页。 返回为 Collection 集合，传入的第一个参数为每页所展示的条数，第二个参数为是否开启分页的相关信息输出，默认为 true。开启后会展示当前页码、总计条数、总计页数以及是否还有下一页等信息，具体如下：

```php
// 将全部数据以每 3 条为一页输出
Demo::paginate(3);

// 此时请求 /?page=1，则会返回第一页的信息
```

若返回的是 `Collection` 或者 `Database` 实例，同样可以使用 `paginate()` 方法进行分页，具体参照本文档相对于章节。 

### 聚合

Model 模型的聚合，即是储存着模型多个实例的 `Collection` 集合的聚合，具体如下：

```php
// 获取模型中 title 字段为 test 的全部数据
$collection = Demo::where([ "title" => "test" ])->get();

// 显示第一个元素
echo $collection->first();

// 显示最后一个元素
echo $collection->last();

// 显示集合中 age 字段为最大值的实例
echo $collection->max("age");

// 显示集合中 age 字段为最小值的实例
echo $collection->min("age");

// 显示集合中 age 字段的平均值
echo $collection->average("age");

// 显示集合中 age 字段的数值总和
echo $collection->sum("age");

// 显示集合中元素个数
echo $collection->count();
```

### 排序

Model 模型的排序，即是储存着模型多个实例的 `Collection` 集合的排序。
其默认排序方式为升序（ASC），排序方法为 PHP 自带的 `usrot`，具体如下：

```php
// 获取模型中 title 字段为 test 的全部数据
$collection = Demo::where([ "title" => "test" ])->get();

// 根据 age 字段升序排序
$collection->sortBy("age");

// 根据 age 字段降序排序
$collection->sortBy("age", "DESC");
```

### 转换 

Model 模型的实例对象可以转换为 `JSON` 字符串，具体如下：

```php
// 找到 ID 为 1 的一行数据
$instance = Demo::find(1);

// 显示输出 JSON 
echo $collection;

// 转化为 JSON 格式
$result = $collection->toJson();

```

若返回的是 `Collection` 或者 `Database` 实例，同样可以使用 `toJson()` 方法进行转换。

同时 `Collection` 另有 `toArray()` 方法可以将集合转换为普通 PHP 数组，具体参照本文档相对于章节。 

