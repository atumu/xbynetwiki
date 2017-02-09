title: phplearn12 

#  PHP学习之MySQL初步 
  * 首选 PDO
  * 其次 mysqli_xxx
  * 而mysql_xxx已被废弃
在php5版本之前，一般是用php的mysql函数去驱动mysql数据库的，比如mysql_query()的函数，属于面向过程
在php5版本以后，增加了mysqli的函数功能，某种意义上讲，它是mysql系统函数的增强版，更稳定更高效更安全，与mysql_query()对应的有mysqli_query()，属于面向对象，用对象的方式操作驱动mysql数据库.
mysql与mysqli的区别：
1、mysql是非持继连接函数，mysql每次链接都会打开一个连接的进程。
2、mysqli是永远连接函数，mysqli多次运行mysqli将使用同一连接进程,从而减少了服务器的开销。mysqli封装了诸如事务等一些高级操作，同时封装了DB操作过程中的很多可用的方法。

**使用PDO是趋势**
PHP-MySQL 是 PHP 操作 MySQL 资料库最原始的 Extension
PHP-MySQLi 的 i 代表 Improvement ，提更了相对进阶的功能，就 Extension 而言，本身也增加了安全性.
PDO是PHP5.1之后才支持的，他为访问数据库采用了一致性的接口，有非常多的操作却是MySQL扩展库所不具备的:
1). PDO真正的以底层实现的统一接口数库操作接口
2). PDO支持更高级的DB特性操作，如：存储过程的调度等,mysql原生库是不支持的.
3). PDO是PHP官方的PECL库，兼容性稳定性必然要高于MySQL Extension,可以直接使用 pecl upgrade pdo 命令升级
推荐使用PDO进行数据库操作，MySQL Extension作为辅助.

pdo与php_mysqli的区别是：pdo提供通用的功能，这样才能支持各个不同的数据库，所以无法支持mysql特有的功能，比如多语句执行。但考虑到一般用不到特殊功能，所以影响不大。
##  mysql基本命令 
###  基础操作 
1.show databases; 显示所有的数据库
2.use 数据库名 使用为数据库名的数据库
3.show tables 显示当前数据库下的所有表
4.desc 表名 查询表结构
5.create database 数据库名 创建一个数据库
6.创建表 create table 表名(
字段名 字段属性,
字段名 字段属性
);
7.alter table 表名 rename 新表名
8.alter table 表名 add 字段名 字段属性 after 欲在哪个字段后加入 添加列
9.alter table 表名 drop column 字段名 删除列
 10 alter table 表名 change 原列名 更改后列名 字段属性 更改列
11. drop database 数据库名 删除一个数据库

###  二、CRUD 
（1）显示表记录 select * from 表名
（2）添加一条记录 insert into 表名 (字段1,字段2)values(值1，值2)
（3)修改一条记录 update 表明 set 要修改的字段名=值 where 查询条件
（4）删除表中数据 delete from 表明 where 查询条件
5汇总 select sum(字段名) from 表名 或 select sum(字段名) as ‘代名称’from 表名
(sum 总分 avg 平均分 max 最高分 min 最低分)
(5)获得表中的记录数 select count(*) from 表名
 1.精确查找 select * from 表名 where 条件 如 tid=1
2.模糊查找 select * from 表名 where 条件 如 name like '%王%'
3.区间搜索 select * from 表名 where 条件区间 如 s_fenshu between 60 and 90;
4.排序 select * from 表名 where 查询条件 order by 排序条件 升序或降序
如 select * from 表名 where s_fenshu between 60 and 90 order by s_fenshu desc;
5.分组加排序
select  按什么分组,sum(字段名称)  from 表名 group by 按什么分组 order by 排序条件， 升序或者降序
select  s_name,sum(s_fenshu) from new group by s_name order by sum(s_fenshu) desc;
6 当前正在使用的数据库 select database();
7 当前正在使用的用户信息 select user();

###  三 导入，导出sql 
在CMD下，进入你的mysql安装目录下的bin目录
导出 mysqldump -u 用户名 -p 要导出的数据库名>另存的sql文件名
mysqldump -u 用户名 -p cocokiki》d:/cocokiki_sql.sql

导出表
mysqldump -u 用户名 -p 要导出的数据库名 表名>另存的sql文件名
mysqldump -u 用户名 -p cocokiki new》d:/cocokiki_sql.sql

只导出表结构
mysqldump -u 用户名 -p  -d -add 要导出的数据库名 表名>另存的sql文件名
mysqldump -u 用户名 -p -d -add cocokiki new》d:/cocokiki_sql.sql

导入表
首先创建表，mysqladmin -u 用户名 -p  create 数据库名
mysqladmin -u 用户名 -p  create test
mysql -u 用户名 -p 导入的数据库名<源文件名
mysql -u 用户名 -p test<d:/cocokiki_sql.sql

##  pdo方式(推荐) 
PDO中包含三个预定义的类，它们分别是 PDO、PDOStatement 和 PDOException。
```

一、PDO
PDO->beginTransaction() — 标明回滚起始点
PDO->commit() — 标明回滚结束点，并执行SQL
PDO->__construct() — 建立一个PDO链接数据库的实例
PDO->errorCode() — 获取错误码
PDO->errorInfo() — 获取错误的信息
PDO->exec() — 处理一条SQL语句，并返回所影响的条目数
PDO->getAttribute() — 获取一个“数据库连接对象”的属性
PDO->getAvailableDrivers() — 获取有效的PDO驱动器名称
PDO->lastInsertId() — 获取写入的最后一条数据的主键值
PDO->prepare() — 生成一个“查询对象”
PDO->query() — 处理一条SQL语句，并返回一个“PDOStatement”
PDO->quote() — 为某个SQL中的字符串添加引号
PDO->rollBack() — 执行回滚
PDO->setAttribute() — 为一个“数据库连接对象”设定属性
二、PDOStatement
PDOStatement->bindColumn() — Bind a column to a PHP variable
PDOStatement->bindParam() — Binds a parameter to the specified variable name
PDOStatement->bindValue() — Binds a value to a parameter
PDOStatement->closeCursor() — Closes the cursor, enabling the statement to be executed again.
PDOStatement->columnCount() — Returns the number of columns in the result set
PDOStatement->errorCode() — Fetch the SQLSTATE associated with the last operation on the statement handle
PDOStatement->errorInfo() — Fetch extended error information associated with the last operation on the statement handle
PDOStatement->execute() — Executes a prepared statement
PDOStatement->fetch() — Fetches the next row from a result set
PDOStatement->fetchAll() — Returns an array containing all of the result set rows
PDOStatement->fetchColumn() — Returns a single column from the next row of a result set
PDOStatement->fetchObject() — Fetches the next row and returns it as an object.
PDOStatement->getAttribute() — Retrieve a statement attribute
PDOStatement->getColumnMeta() — Returns metadata for a column in a result set
PDOStatement->nextRowset() — Advances to the next rowset in a multi-rowset statement handle
PDOStatement->rowCount() — Returns the number of rows affected by the last SQL statement
PDOStatement->setAttribute() — Set a statement attribute
PDOStatement->setFetchMode() — Set the default fetch mode for this statement
  

```
```

<?php
$dsn = 'mysql:dbname=test;host=localhost'; //mysql
$user = 'root'; //mysql
$password = '1'; //mysql
$db = new PDO($dsn, $user, $password，array(PDO::ATTR_PERSISTENT => true)); //mysql
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$sql = 'INSERT INTO `user` (`id`, `name`, `desc`) VALUES (\'2\',\'lucy\',\'hello\');'; //mysql
?>

```
PDO数据库连接中DSN(Data Source Name,即数据源名称)格式为 
```

驱动名:host=主机名;dbname=数据库名;port=端口号;charset=字符集

```
外部保存配置:
```

<?php
require 'settings.php';

try {
    $pdo = new PDO(
        sprintf(
            'mysql:host=%s;dbname=%s;port=%s;charset=%s',
            $settings['host'],
            $settings['name'],
            $settings['port'],
            $settings['charset']
        ),
        $settings['username'],
        $settings['password']
    );
} catch (PDOException $e) {
    // Database connection failed
    echo "Database connection failed";
    exit;
}


```
可以看出，使用pdo，如果数据库从mysql迁移到pgsql，只需要改dsn，而sql语法不同可以通过SQL Maker来自动生成。
许多Web应用会因为使用了向数据库的持久连接而得到优化。持久连接不会在脚本结束时关闭，相反它会被缓存起来并在另一个脚本通过同样的标识请求一个连接时得以重新利用。持久连接的缓存可以使你避免在脚本每次需要与数据库对话时都要部署一个新的连接的资源消耗，让你的Web应用更加快速。
` 上面实例中的array(PDO::ATTR_PERSISTENT => true)就是把连接类型设置为持久连接。 `
```

   //执行sql语句获得
   $sql = 'select*fromgoods';
   $result = $pdo->query($sql);//query方法返回的是PDOStatement对象
   //如果想获得具体的数据，需要调用对象的方法：fetchAll();参数是类常量，表示返回什么样的数据
   $rows = $result->fetchAll(PDO::FETCH_BOTH);
   //var_dump($rows);

 
   //更新数据库的操作
   $sql = 'update goods set goods_name="lalala" wheregoods_id=3';
   //执行增删改的语句,exec()方法，执行查询的语句 query()
   //exec()返回受影响的行数      query()返回PDOStatement对象
   $nums = $pdo->exec($sql);
   var_dump($nums);

```
###  PDO预编译机制 
http://php.net/manual/en/class.pdostatement.php
将不带数据的部分预编译一下` prepare（） `————》在编译结果上绑定数据` bandparam() `————》执行编译结果` execute() `
  * PDOStatement::bindParam — 绑定一个参数到指定的变量名
bool PDOStatement::bindParam ( mixed $parameter , mixed &$variable [, int $data_type = PDO::PARAM_STR [, int $length [, mixed $driver_options ]]] )
  * PDOStatement::bindValue — 把一个值绑定到一个参数，作为引用被绑定
bool PDOStatement::bindValue ( mixed $parameter , mixed $value [, int $data_type = PDO::PARAM_STR ] )

bindParam与bindValue的区别:一个绑定的是变量名，一个绑定的是值。
参数说明:
parameter:参数标识符。对于使用命名占位符的预处理语句，应是类似 :name 形式的参数名。**对于使用问号占位符的预处理语句，应是以1开始索引的参数位置**。
data_type:使用 PDO::PARAM_* 常量明确地指定参数的类型。

写代码实现PDO的预编译（与处理机制）
```

<?php
    //预编译：PDO::prepare($sql);   返回PDOStatement对象
    //绑定数据PDOStamentt->bindParam();  给预编译的结果绑定数据
    //执行编译结果 PDOStament->execute();
    //使用PDO操作数据库
    //第一个参数：连接数据库的类型：主机名；数据库名
    $dsn  = 'mysql:host=localhost;dbname=mysql_text';
    $user = 'root';
    $pass = '123';
    $pdo = new PDO($dsn,$user,$pass);
   //var_dump($pdo);

    //预编译：prepare();参数是不带数据的sql语句
    //先将sql语句中的数据部分用占位符代替 :占位符名称
    $sql = 'insert intogoods values(null,:name,:price,:number)';
    $smt = $pdo->prepare($sql);  //返回一个PDOStament 对象
    //绑定数据 PDOStament对象的bindParam()来绑定参数：占位符，实际数据
    $goods_name= 'surface';
    $goods_price= '3500';
    $goods_num= '41';
    $smt->bindParam(':name',$goods_name);
    $smt->bindParam(':price',$goods_price);
    $smt->bindParam(':number',$goods_num);
//也可以使用$stmt->execute(array(':name',$goods_name, ':price',$goods_price，':number',$goods_num));
    $smt->execute()；
?>

```
绑定时指定数据类型:
```

// Prepared statement
$sql = 'SELECT email FROM users WHERE id = :id';
$userId = filter_input(INPUT_GET, 'id');

$statement = $pdo->prepare($sql);
$statement->bindValue(':id', $userId, PDO::PARAM_INT);

```
简单的总结一下上面的操作:
**查询操作主要是PDO::query()、PDO::exec()、PDO::prepare()。**
  * PDO::query()主要是用于有记录结果返回的操作，特别是SELECT操作，
  * PDO::exec()主要是针对没有结果集合返回的操作，比如INSERT、UPDATE、DELETE等操作，它返回的结果是当前操作影响的列数。
  * PDO::prepare()主要是预处理操作，需要通过$rs->execute()来执行预处理里面的SQL语句，这个方法可以绑定参数，功能比较强大。
**获取结果集操作主要是：PDOStatement::fetchColumn()、PDOStatement::fetch()、PDOStatement::fetchALL()、PDOStatement::fetchObject()。**
  * PDOStatement::fetchColumn()是获取结果指定第一条记录的某个字段，缺省是第一个字段。
  * PDOStatement::fetch()是用来获取一条记录，
  * PDOStatement::fetchAll()是获取所有记录集到一个中，获取结果可以通过PDOStatement::setFetchMode来设置需要结果集合的类型。
  * PDOStatement::fetchObject()获取一条记录到对象。

fetch()和fetchAll()还可以接受一个常量参数，指定返回数据的格式:
（默认为 PDO::FETCH_BOTH ）。
  * **PDO::FETCH_ASSOC**：返回一个索引为结果集列名的数组
  * **PDO::FETCH_BOTH（默认）**：返回一个索引为结果集列名和以0开始的列号的数组
  * PDO::FETCH_BOUND：返回 TRUE ，并分配结果集中的列值给 PDOStatement::bindColumn() 方法绑定的 PHP 变量。
  * PDO::FETCH_CLASS：返回一个请求类的新实例，映射结果集中的列名到类中对应的属性名。如果 fetch_style 包含 PDO::FETCH_CLASSTYPE（例如：PDO::FETCH_CLASS | PDO::FETCH_CLASSTYPE），则类名由第一列的值决定
  * PDO::FETCH_INTO：更新一个被请求类已存在的实例，映射结果集中的列到类中命名的属性
  * PDO::FETCH_LAZY：结合使用 PDO::FETCH_BOTH 和 PDO::FETCH_OBJ，创建供用来访问的对象变量名
  * **PDO::FETCH_NUM**：返回一个索引为以0开始的结果集列号的数组
  * **PDO::FETCH_OBJ**：返回一个属性名对应结果集列名的匿名对象
另外有两个周边的操作，一个是**PDO::lastInsertId()和PDOStatement::rowCount()**。PDO::lastInsertId()是返回上次插入操作，主键列类型是自增的最后的自增ID。
PDOStatement::rowCount()主要是用于PDO::query()和PDO::prepare()进行DELETE、INSERT、UPDATE操作影响的结果集，对PDO::exec()方法和SELECT操作无效。

```

$dbh = new PDO('mysql:host=localhost;dbname=access_control', 'root', '');
$dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$dbh->exec('set names utf8');
/*添加*/
//$sql = "INSERT INTO `user` SET `login`=:login AND `password`=:password";
$sql = "INSERT INTO `user` (`login` ,`password`)VALUES (:login, :password)";
$stmt = $dbh->prepare($sql);
$stmt->execute(array(':login'=>'kevin2',':password'=>''));
echo $dbh->lastinsertid();
/*修改*/
$sql = "UPDATE `user` SET `password`=:password WHERE `user_id`=:userId";
$stmt = $dbh->prepare($sql);
$stmt->execute(array(':userId'=>'7', ':password'=>'4607e782c4d86fd5364d7e4508bb10d9'));
echo $stmt->rowCount();
/*删除*/
$sql = "DELETE FROM `user` WHERE `login` LIKE 'kevin_'"; //kevin%
$stmt = $dbh->prepare($sql);
$stmt->execute();
echo $stmt->rowCount();
/*查询*/
$login = 'kevin%';
$sql = "SELECT * FROM `user` WHERE `login` LIKE :login";
$stmt = $dbh->prepare($sql);
$stmt->execute(array(':login'=>$login));
while($row = $stmt->fetch(PDO::FETCH_ASSOC)){
    print_r($row);
}
print_r( $stmt->fetchAll(PDO::FETCH_ASSOC) );

```
###  PDO的错误处理机制 
PDO 提供了3中不同的错误处理策略PDO::ATTR_ERRMODE。
（1）静默模式PDO::ERRMODE_SILENT
默认情况下与mysql处理方式一致，不现实错误信息（静默模式）但是我们可以通过固定的方法获得错误信息
**PDO和PDOStatement对象有errorCode() 和 errorInfo() 方法**，如果没有任何错误, errorCode() 返回的是: 00000，否则就会返回一些错误代码。errorInfo() 返回的一个数组，包括PHP定义的错误代码和MySQL的错误代码和错误信息，数组结构如下：
```

[0] => 42S22
[1] => 1054
[2] => Unknown column 'aaa' in'field list'

```
（2）警告模式PDO::ERRMODE_WARNING
```

//更改属性设置错误处理模式
$pdo ->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_WARNING);

```
（3）异常模式，当发生错误时，抛出一个异常 PDO::ERRMODE_EXCEPTION
```

$pdo ->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_EXCEPTION);
$sql = 'select*fromgood';
try {
   //尝试可能会处错误的代码
   $pdo ->query($sql);
}catch (PDOException $e){
   //现在捕获异常后，自己看着办，是让他显示出来呢，还是输出到日志文件里呢？
    //通常是将错误信息输出到日志文件里
   var_dump($e->getMessage());
   file_put_contents('D://mysql.log',$e->getMessage());
}

```
PDOException异常类的属性结构：
```

<?php
class PDOException extendsException
{
public$errorInfo = null; // 错误信息，可以调用 PDO::errorInfo() 或 PDOStatement::errorInfo()来访问
protected$message; // 异常信息，可以试用Exception::getMessage() 来访问
protected$code; // SQL状态错误代码，可以使用Exception::getCode() 来访问
}
?>

```
###  PDO中的事务 
PDO->beginTransaction()，PDO->commit()，PDO->rollBack()这三个方法是在支持回滚功能时一起使用的。
```

<?php
try {
$dbh = new PDO('mysql:host=localhost;dbname=test', 'root', '1234');
$dbh->query('set names utf8;');
$dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$dbh->beginTransaction();
$dbh->exec(”INSERT INTO `test`.`table` (`name` ,`age`)VALUES ('mick', 22);”);
$dbh->exec(”INSERT INTO `test`.`table` (`name` ,`age`)VALUES ('lily', 29);”);
$dbh->exec(”INSERT INTO `test`.`table` (`name` ,`age`)VALUES ('susan', 21);”);
$dbh->commit();
} catch (Exception $e) {
$dbh->rollBack();
echo “Failed: ” . $e->getMessage();
}
?>

```
**事务处理，它们提供了4个主要的特性：原子性，一致性，独立性和持久性（Atomicity, Consistency, Isolation and Durability，ACID）**
通俗一点讲，一个事务中所有的工作在提交时，即使它是分阶段执行的，也要保证安全地应用于数据库，不被其他的连接干扰。事务工作也可以在请求发生错误时轻松地自动取消。


###  属性选项 
PDO::ATTR_AUTOCOMMIT在设置成true的时候，PDO会自动尝试停止接受委托，开始执行
PDO::ATTR_PREFETCH设置应用程序提前获取的数据大小，并非所有的数据库哦度支持
PDO::ATTR_TIMEOUT设置连接数据库超时的值
**PDO::ATTR_ERRMODE设置Error处理的模式**
PDO::ATTR_SERVER_VERSION只读属性，表示PDO连接的服务器端数据库版本
PDO::ATTR_CLIENT_VERSION只读属性，表示PDO连接的客户端PDO驱动版本
PDO::ATTR_SERVER_INFO只读属性，表示PDO连接的服务器的meta信息
PDO::ATTR_CONNECTION_STATUS
PDO::ATTR_CASE
通过PDO::CASE_*中的内容对列的形式进行操作
PDO::ATTR_CURSOR_NAME获取或者设定指针的名称
PDO::ATTR_CURSOR设置指针的类型，PDO现在支持PDO::CURSOR_FWDONLY和PDO::CURSOR_FWDONLY
PDO::ATTR_DRIVER_NAME返回使用的PDO驱动的名称
PDO::ATTR_ORACLE_NULLS将返回的空字符串转换为SQL的NULL
PDO::ATTR_PERSISTENT获取一个存在的连接
PDO::ATTR_STATEMENT_CLASS
PDO::ATTR_FETCH_CATALOG_NAMES
在返回的结果集中，使用自定义目录名称来代替字段名。
PDO::ATTR_FETCH_TABLE_NAMES
在返回的结果集中，使用自定义表格名称来代替字段名。
PDO::ATTR_STRINGIFY_FETCHES
PDO::ATTR_MAX_COLUMN_LEN
PDO::ATTR_DEFAULT_FETCH_MODE
Available since PHP 5.2.0
PDO::ATTR_EMULATE_PREPARES

其余：
PDO::CASE_NATURAL
回复列的默认显示格式
PDO::CASE_LOWER
强制列的名字小写
PDO::CASE_UPPER
强制列的名字大写
PDO::NULL_NATURAL
PDO::NULL_EMPTY_STRING
PDO::NULL_TO_STRING
##  php_mysqli方式 
http://php.net/manual/zh/book.mysqli.php
mysqli提供了3个主要类
1.mysqli：和数据库连接有关的类（连接，选择库，发送SQL语句，设置字符集）
2.mysqli_result:表达了对数据库的查询所返回的**结果集**
方法：
  * $result->fetch_row()
  * $result->fetch_assoc()   建议用此
  * $result->fetch_array()   (MYSQLI_NUM,MYSQLI_ASSOC,MYSQLI_BOTH(default))
  * $result->fetch_object()
3.mysqli_stmt：表达了一个预备好的语句

```

<?php
$mysqli = new mysqli('localhost', 'root', '1', 'test');
//$a = "hello world\fjim\n\r";
//$sql = 'INSERT INTO `user` (`id`, `name`, `desc`) VALUES (\'2\',\'lucy\',' . $mysqli->real_escape_string($a) . '\');';
$sql = 'SELECT `id`, `name`, `desc` FROM `user` LIMIT 2';
$r = $mysqli->query($sql);
$arr = $r->fetch_all();
var_dump($arr);
exit;
?>

```
下面这个示例演示从mysql数据库获取并格式化搜索结果，以便显式结果。
```

<html>
<head>
  <title>Book-O-Rama Search Results</title>
</head>
<body>
<h1>Book-O-Rama Search Results</h1>
<?php
  // create short variable names
  $searchtype=$_POST['searchtype'];
  $searchterm=trim($_POST['searchterm']);

  if (!$searchtype || !$searchterm) {
     echo 'You have not entered search details.  Please go back and try again.';
     exit;
  }

  if (!get_magic_quotes_gpc()){
    $searchtype = addslashes($searchtype);
    $searchterm = addslashes($searchterm);
  }

  @ $db = new mysqli('localhost', 'bookorama', 'bookorama123', 'books'); //创建mysqli对象，它也代表了建立一个数据库连接。参数为主机、用户名、密码、数据库名
//@ $db前面加个@代表抑制错误。转而通过mysqli_connect_errno()进行判断。
  if ($mysqli->connect_errno) { //判定是否正确连接。值得注意的是。它是成功了返回0，而php中0代表false。所以当if通过时表示获取失败
     echo 'Error: Could not connect to database.  Please try again later.';
     exit;
  }

  $query = "select * from books where ".$searchtype." like '%".$searchterm."%'";
  $result = $db->query($query); //创建查询，返回结果集对象

  $num_results = $result->num_rows; //获取匹配行数

  echo "<p>Number of books found: ".$num_results."</p>";

  for ($i=0; $i <$num_results; $i++) {
     $row = $result->fetch_assoc();  //将当前行查询为一个关联数组。
     echo "<p><strong>".($i+1).". Title: ";
     echo htmlspecialchars(stripslashes($row['title']));//stripslashes与入库时的addslashes相对
     echo "</strong><br />Author: ";
     echo stripslashes($row['author']);
     echo "<br />ISBN: ";
     echo stripslashes($row['isbn']);
     echo "<br />Price: ";
     echo stripslashes($row['price']);
     echo "</p>";
  }

  $result->free(); //释放结果集
  $db->close(); //关闭连接

?>
</body>
</html>

```
也可以通过mysqli_select_db(dbname)或者$db->select_db(dbname)来选择切换数据库。

$row = $result->fetch_assoc();  将当前行查询为一个关联数组。$row['title']
$row=$result->fetch_row()保存到一个数组中。$row[0]
$row=$result->fetch_object()保存到一个对象中。然后使用$row->title访问

以下脚本将新的图书写入到数据库：
```

<html>
<head>
  <title>Book-O-Rama Book Entry Results</title>
</head>
<body>
<h1>Book-O-Rama Book Entry Results</h1>
<?php
  // create short variable names
  $isbn=$_POST['isbn'];
  $author=$_POST['author'];
  $title=$_POST['title'];
  $price=$_POST['price'];

  if (!$isbn || !$author || !$title || !$price) {
     echo "You have not entered all the required details.<br />"
          ."Please go back and try again.";
     exit;
  }

  if (!get_magic_quotes_gpc()) {
    $isbn = addslashes($isbn);
    $author = addslashes($author);
    $title = addslashes($title);
    $price = doubleval($price);
  }

  @ $db = new mysqli('localhost', 'bookorama', 'bookorama123', 'books');

  if (mysqli_connect_errno()) {
     echo "Error: Could not connect to database.  Please try again later.";
     exit;
  }

  $query = "insert into books values
            ('".$isbn."', '".$author."', '".$title."', '".$price."')";
  $result = $db->query($query);

  if ($result) { //如果执行成功
      echo  $db->affected_rows." book inserted into database.";  //获取影响的行数。
  } else {
  	  echo "An error has occurred.  The item was not added.";
  }

  $db->close();//关闭连接
?>
</body>
</html>

```
###  执行多条sql语句 
```

<?php
$mysqli = new mysqli("example.com", "user", "password", "database");
if ($mysqli->connect_errno) {
    echo "Failed to connect to MySQL: (" . $mysqli->connect_errno . ") " . $mysqli->connect_error;
}

if (!$mysqli->query("DROP TABLE IF EXISTS test") || !$mysqli->query("CREATE TABLE test(id INT)")) {
    echo "Table creation failed: (" . $mysqli->errno . ") " . $mysqli->error;
}

//注意这里是字符串拼接这三条语句
$sql = "SELECT COUNT(*) AS _num FROM test; ";
$sql.= "INSERT INTO test(id) VALUES (1); ";
$sql.= "SELECT COUNT(*) AS _num FROM test; ";

if (!$mysqli->multi_query($sql)) {
    echo "Multi query failed: (" . $mysqli->errno . ") " . $mysqli->error;
}

do {
    if ($res = $mysqli->store_result()) {
        var_dump($res->fetch_all(MYSQLI_ASSOC));
        $res->free();
    }
} while ($mysqli->more_results() && $mysqli->next_result());
?>

```
###  mysqli事务 
http://php.net/manual/zh/mysqli.begin-transaction.php
http://www.cnblogs.com/quinnxu/archive/2012/07/18/2597306.html

InnoDB支持事务（MyISAM不支持）

区分PHP5.5以前和以后使用
**PHP5.5以前：mysqli->autocommit(false)**
```

<?php
$mysqli = new mysqli("localhost", "my_user", "my_password", "world");

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

$mysqli->query("CREATE TABLE Language LIKE CountryLanguage");

/* set autocommit to off */
$mysqli->autocommit(FALSE);

/* Insert some values */
$mysqli->query("INSERT INTO Language VALUES ('DEU', 'Bavarian', 'F', 11.2)");
$mysqli->query("INSERT INTO Language VALUES ('DEU', 'Swabian', 'F', 9.4)");

/* commit transaction */
$mysqli->commit();

/* drop table */
$mysqli->query("DROP TABLE Language");

/* close connection */
$mysqli->close();
?>

```
**PHP5.5以后有了mysqli::begin_transaction** 
面向对象风格 (method):
public bool mysqli::begin_transaction ([ int $flags [, string $name ]] )
过程化风格:
bool mysqli_begin_transaction ( mysqli $link [, int $flags [, string $name ]] )
link仅以过程化样式：由mysqli_connect() 或 mysqli_init() 返回的链接标识。

flags有如下选项
  * MYSQLI_TRANS_START_READ_ONLY: Start the transaction as "START TRANSACTION READ ONLY".
  * MYSQLI_TRANS_START_READ_WRITE: Start the transaction as "START TRANSACTION READ WRITE".
  * MYSQLI_TRANS_START_WITH_CONSISTENT_SNAPSHOT: Start the transaction as "START TRANSACTION WITH CONSISTENT SNAPSHOT".
name：Savepoint name for the transaction.

返回值 ：成功时返回 TRUE， 或者在失败时返回 FALSE。

```

<?php
$mysqli = new mysqli("127.0.0.1", "my_user", "my_password", "sakila");

if ($mysqli->connect_errno) {
    printf("Connect failed: %s\n", $mysqli->connect_error);
    exit();
}

$mysqli->begin_transaction();

$mysqli->query("SELECT first_name, last_name FROM actor");
$mysqli->commit();

$mysqli->close();
?>

```


###  预编译语句 
防止sql注入。
```

$query ="insert into books values (?,?,?,?)";
$stmt =$db->prepare($query);
$stmt->bind_param('sssd','str1','str2','str3',100);//第一个参数代表格式化字符串，可以有s,i,d,b分别代表字符串，整数，浮点数，blob
$stmt->execute();
echo $stmt->affected_rows();
$stmt->close();

```
于绑定参数一样，也可以绑定结果。此处不作介绍


##  php_mysql方式(已废弃) 
已废弃