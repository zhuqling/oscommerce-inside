## 缓存与性能

osCommerce V3 新增了缓存存取类,缓存将以文件的形式保存在 DIR_FS_WORK 目录下, osCommerce V3的缓存存取允许直接调用write方法写入缓存,以及用read方法读取缓存信息, 如果想保存数据库查询的结果,还可以使用数据库查询类所附带的缓存方法,实现快速的缓存读取和写入

### 缓存的读写

osC_Cache 缓存类的保存与读取方法 includes/classes/cache.php

```php
public function write($data, $key = null) {
  // write方法为写缓存,$data参数允许任 何类型的数据,保存缓存时会将数据进入序列化,所以保存对象同样可以轻易实现
  if ( empty($key) ) { // 可以临时指定缓存 KEY,KEY 将作为缓存文件的识别特征
    $key = $this->_key; // 或者使用原有的缓存 KEY
  }

  return ( file_put_contents(DIR_FS_WORK . $key . '.cache', serialize($data), LOCK_EX) !== false ); // 调用 file_put_contents 方法写入缓存,并将数据序列持久化
}

public function read($key, $expire = null) {
  // read 方法为读缓存,$key 指定了缓存 KEY ,$expire 表示过期时间(单位:秒),如果设为 null 表示永不过期
  $this->_key = $key;
  $filename = DIR_FS_WORK . $key . '.cache';
  if ( file_exists($filename) ) {
    $difference = floor((time() - filemtime($filename)) / 60); // 计算当前时间与文
件最后修改时间之差
    if ( empty($expire) || ( is_numeric($expire) && ($difference < $expire)) ) { //判断是否过期
      $this->_data = unserialize(file_get_contents($filename)); // 读取缓存文件的 内容,并将其反序列化
      return true;
    }
  }
  
  return false;
}
```

下面是存取缓存的一个例子

新产品内容框 includes/modules/boxes/whats_new.php[33]

```php
$data = array(); // 最终内容会保存在$data 数组里

if ( (BOX_WHATS_NEW_CACHE > 0) && $osC_Cache->read('box-whats_new-' . $osC_Language->getCode() . '-' . $osC_Currencies->getCode(), BOX_WHATS_NEW_CACHE) ) { // 先尝试着读取缓存,如果有缓存,并且未过期,则直接使用缓存内容作为结果
  $data = $osC_Cache->getCache(); // osC_Cache 类的 getCache 方法返回其内部变量$_data 
} else { // 否则通过数据库查询结果
  $Qnew = $osC_Database->query('select products_id from :table_products where products_status = :products_status order by products_date_added desc limit :max_random_select_new');
  $Qnew->bindTable(':table_products', TABLE_PRODUCTS);
  $Qnew->bindInt(':products_status', 1);
  $Qnew->bindInt(':max_random_select_new', BOX_WHATS_NEW_RANDOM_SELECT);
  $Qnew->executeRandomMulti();
  if ( $Qnew->numberOfRows() ) {
    $osC_Product = new osC_Product($Qnew->valueInt('products_id'));
    $data = $osC_Product->getData();// 得到基础数据 // 追加其它信息

    $data['display_price'] = $osC_Product->getPriceFormated(true);
    $data['display_image'] = $osC_Product->getImage();
  }

  $osC_Cache->write($data); // 写入缓存
}
```

> 提示:
> 
> 在上面示例中,写入缓存时并没有提供缓存 KEY,为什么呢? 因为在调用 read 方法尝试读取缓存时,使用了缓存 KEY,而在 read 方法中会自动将 KEY 保存, 当在写缓存时没有给出缓存 KEY,那么就会使用上一次自动保存的缓存进入写操作

### 数据库与缓存

当我们使用 osC_Database 类的方法 query 查询数据库时,query 方法实际返回的是一个 osC_Database_Result 对象,所以 osCommerce V3 便可以使用面向对象的方法来操作查询结果,在 了解与缓存操作有关的 osC_Database_Result 类方法之前,让我们先看一个数据库查询自动缓存的例子

带自动缓存的配置查询 includes/application_top.php[67]

```php
// initialize the cache class
require('includes/classes/cache.php');

$osC_Cache = new osC_Cache(); // 在加载 osC_Database 类与 osC_Database_Result 类之前加载 osC_Cache 类,这样才可以在它们之中使用缓存存取方法 // include the database class

require('includes/classes/database.php');
// make a connection to the database... now
$osC_Database = osC_Database::connect(DB_SERVER, DB_SERVER_USERNAME, DB_SERVER_PASSWORD);
$osC_Database->selectDatabase(DB_DATABASE);

// set the application parameters
$Qcfg = $osC_Database->query('select configuration_key as cfgKey,
configuration_value as cfgValue from :table_configuration'); // 开启一个查询 
$Qcfg->bindTable(':table_configuration', TABLE_CONFIGURATION);
$Qcfg->setCache('configuration'); // 设置缓存 KEY
$Qcfg->execute(); // 执行查询语句
while ($Qcfg->next()) { // 循环读取所有设置信息 
  define($Qcfg->value('cfgKey'), $Qcfg->value('cfgValue'));
}

$Qcfg->freeResult(); // 注销查询资源或者将查询结果保存到缓存
```

下面讲述 osC_Database_Result 类与缓存有关的方法

设置缓存 KEY 与过期日期 includes/classes/database.php [602]

```php
function setCache($key, $expire = 0) {
  $this->cache_key = $key;
  $this->cache_expire = $expire;
}
```

> 提示:
> 
> 通过上面的代码,可以知道 osCommerce V3 的配置信息一旦保存到了缓存,前台将永不过期,那 么,也就是说必须在后台修改系统配置时,才会重新生成缓存

正式执行查询语句 includes/classes/database.php[505]

```php
function execute() {
  global $osC_Cache;

  if (isset($this->cache_key)) { // 判断是否设置了缓存 KEY
    if ($osC_Cache->read($this->cache_key, $this->cache_expire)) { // 读取缓存是否有效

      $this->cache_data = $osC_Cache->getCache(); // 得到缓存数据 
      $this->cache_read = true; // 内部变量$cache_read 作为显示是否从缓存读取数据的标志
    }
  }

  if ($this->cache_read === false) { // 否则执行数据库查询语句
```
循环读取数据 includes/classes/database.php [443]

```php
function next() {
  if ($this->cache_read === true) { // 当是从缓存时获得数据时,需要操作缓存取得一条数据
    list(, $this->result) = each($this->cache_data);
  } else {
    if (!isset($this->query_handler)) {
      $this->execute();
    }

    $this->result = $this->db_class->next($this->query_handler);
    if (isset($this->cache_key)) {
      $this->cache_data[] = $this->result;
    }
  }
  return $this->result;
}
```

注销与保存 includes/classes/database.php [461]

```php
function freeResult() {
  global $osC_Cache;

  if ($this->cache_read === false) { // 如果不是从缓存读取数据,则需要注销查询资源
    if (eregi('^SELECT', $this->sql_query)) {
      $this->db_class->freeResult($this->query_handler);
    }

    if (isset($this->cache_key)) { // 并且将结果保存到缓存 
      $osC_Cache->write($this->cache_data, $this->cache_key);
    }
  }

  unset($this);
}
```

获取数据总数 includes/classes/database.php [477]

```php
function numberOfRows() { // 获取数据总数的方法在使用缓存与未使用缓存时有所差异
  if (!isset($this->rows)) {
    if (!isset($this->query_handler)) {
      $this->execute();
    }

    if (isset($this->cache_key) && ($this->cache_read === true)) { // 如果使用了 缓存,将使用 sizeof 函数返回数据总数
      $this->rows = sizeof($this->cache_data);
    } else {
      $this->rows = $this->db_class->numberOfRows($this->query_handler); // 否则 需要利用 osC_Database 类的 numberOfRows 方法得到数据总数
    }
  }

  return $this->rows;
}
```

