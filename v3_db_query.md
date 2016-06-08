## 数据查询

osCommerce V3 对数据库的操作进行了类封装,使得数据库的转移及扩展能够得以实现。另一个 方面,对于数据库最常用的操作,osCommerce V3 也通过 osC_Database_Result 类进行了封装,并 且对数据类型进行了规范,本节将讲述有关数据库操作(包括数据查询与修改)的 osC_Database_Result 类是如何工作的。

### 操作示例

例一:供应商查询 includes/classes/manufacturer.php[181]

```php
function osC_Manufacturer($id) {
  global $osC_Database;

  $Qmanufacturer = $osC_Database->query('select manufacturers_id as id, manufacturers_name as name, manufacturers_image as image from :table_manufacturers where manufacturers_id = :manufacturers_id');
  $Qmanufacturer->bindTable(':table_manufacturers', TABLE_MANUFACTURERS);
  $Qmanufacturer->bindInt(':manufacturers_id', $id);
  $Qmanufacturer->execute();
  if ($Qmanufacturer->numberOfRows() === 1) {
    $this->_data = $Qmanufacturer->toArray();
  }
}
```

上面的代码是供应商 osC_Manufacturer 类的初始化方法,其中就应用了 osC_Database_Result 类的多个方法。

其中$osC_Database 类的 query 方法便是得到 osC_Database_Result 对象的地方,然后通过操 作 osC_Database_Result 类的方法 bindTable 绑定表、bindInt 进行数据绑定,最后通过 execute 方法 真正启动 SQL 查询。

最下面的 numberOfRows 方法返回查询的结果数目,如果查询到相应供应商信息,则使用 toArray 方法将查询结果转化成关联数据,保存到内部变量$_data。

例二:编辑用户资料 includes/classes/account.php[130]

```php
public static function saveEntry($data) {
  global $osC_Database, $osC_Customer;

  $Qcustomer = $osC_Database->query('update :table_customers set customers_gender = :customers_gender, customers_firstname = :customers_firstname, customers_lastname = :customers_lastname, customers_email_address = :customers_email_address, customers_dob = :customers_dob, date_account_last_modified = :date_account_last_modified where customers_id = :customers_id');

  $Qcustomer->bindTable(':table_customers', TABLE_CUSTOMERS);
  $Qcustomer->bindValue(':customers_gender', ((ACCOUNT_GENDER > -1) && isset($data['gender']) && (($data['gender'] == 'm') || ($data['gender'] == 'f'))) ? $data['gender'] : '');
  $Qcustomer->bindValue(':customers_firstname', $data['firstname']);
  $Qcustomer->bindValue(':customers_lastname', $data['lastname']);
  $Qcustomer->bindValue(':customers_email_address', $data['email_address']);
  $Qcustomer->bindValue(':customers_dob', (ACCOUNT_DATE_OF_BIRTH == '1') ?   date('Ymd', $data['dob']) : '');
  $Qcustomer->bindRaw(':date_account_last_modified', 'now()');
  $Qcustomer->bindInt(':customers_id', $osC_Customer->getID());
  $Qcustomer->execute();

  return ( $Qcustomer->affectedRows() === 1 );
}
```

上面的代码是保存用户资料的代码,saveEntry 方法是 osC_Account 类的一个方法。

与上面的例子一样,首先调用 osC_Dabase 类的 query 方法得到 osC_Database_Result 类,不
同之处在于此处使用了 UPDATE 语句,然后是 bindTable 方法绑定数据表,bindValue、bindRaw、 bindInt 方法绑定数据,再由 execute 方法真正执行数据库修改。最后通过 affectedRows 方法得知 UPDATE 语句的更新数据条数。

### 数据查询类

数据查询类 osC_Database_Result 都是通过 osC_Database 类间接生成的。这是因为 osC_Database_Result 需要依赖 osC_Database 类,只有从 osC_Database 类生成的 osC_Database_Result 类对象才可以操作数据库。

osC_Database 类的 query 方法 includes/classes/database.php[57]

```php
function &query($query) {
  $osC_Database_Result =& new osC_Database_Result($this); // 将当前 osC_Database 对象作为参数生成 osC_Database_Result 对象。

  $osC_Database_Result->setQuery($query);// 执行osC_Database_Result类的setQuery 方法设置 SQL 语句

  return $osC_Database_Result; // 返回 osC_Database_Result 对象的句柄
}
```

* osC_Database_Result 类的数据类型绑定方法 *

osC_Database_Result 类支持数据类型绑定,以下是各种数据类型所对应的方法: 

-  bindTable:绑定数据表
-  bindValue:绑定字符类型
-  bindInt:绑定整型数值
-  bindFloat:绑定浮点型数值
-  bindDate:绑定日期类型
-  bindRaw:不改变原始类型

同样在获取数据时,也支持以下几种方法: 

-  valueProtected:进行HTML编码
-  valueInt:整型转换
-  valueDecimal:浮点数转换
-  value:不进行类型转换

下面对 osC_Database_Result 类最常用的几个方法进行介绍:

-  function next() 得到一条数据,并移动到下一条数据的位置
-  function numberOfRows() 获取查询结果的条数
-  function affectedRows() 获取更新影响的结果条数
-  function execute() 正式执行 SQL 语句/读缓存
-  function executeRandom() 乱序查询,通过 SQL 语句实现
-  function executeRandomMulti() 随机返回查询结果,通过随机筛选查询结果实现
-  function freeResult() 注销查询资源/写缓存
-  function setCache($key, $expire = 0) 设置缓存
-  function toArray() 获得查询结果,并将结果转换成关联数组

