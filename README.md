# PHP QuickORM 框架开发文档

版本：20180831

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
建议在 Controller 层调用，需使用 `GET` 方式传入 `page` 字段用于展示第 n 页。 返回为 Collection 集合（具体参照文档 Collection 一章），传入的第一个参数为每页所展示的条数，第二个参数为是否开启分页的相关信息输出，默认为 true。开启后会展示当前页码、总计条数、总计页数以及是否还有下一页等信息，具体如下。

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
>   1. `Demo::where([ ])->get()`
>   2. `(new Database("tableName"))->setModel(Demo::class)->where([ ])->get()`
>   3. `Database::table("tableName")->setModel(Demo::class)->where([ ])->get()`
>   4. `Database::model(Demo::class)->where([ ])->get()`
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

以上代码并没有使用事务回滚，在具体项目中，我们该建议您这样做：

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

PHP QuickORM 框架提供了一种新的数据类型 Collection，是对 PHP 序列数组的二次封装，以便于支持数据聚合功能，以及对对象的储存。

### 创建与判断

Collection 集合的创建支持两种方式，内部实现效果一致。具体如下：

```php
$array = ["1"， "2", "3", "4", "5"];

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
$array = ["1"， "2", "3", "4", "5"];

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

实例集合，即是集合中储存的内容为模型实例化之后的对象，可以跳至文档 Model 模型部分的 数据聚合 查看。

### 集合合并

### 集合包含

### 集合排序

### 集合分页

### 集合迭代

### 数组转换

