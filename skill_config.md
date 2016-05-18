## Configuration 静态化 

1、创建缓存文件夹

在 catalog 目录下创建一个缓存目录“cache”,确认此目录的权限为 755 或者 777,即允许可写

2、修改配置文件:includes/configure.php 

添加如下定义代码:

`define(‘DIR_MY_CACHE’, DIR_FS_CATALOG . ’cache/’);`

3、修改文件 includes/application_top.php 

查找:
      
```php
// set the application parameters
$configuration_query = tep_db_query('select configuration_key as cfgKey,
configuration_value as cfgValue from ' . TABLE_CONFIGURATION);
while ($configuration = tep_db_fetch_array($configuration_query)) {
  define($configuration['cfgKey'], $configuration['cfgValue']);
}
```

替换为如下代码:

```php
// 将配置信息写入缓存文件
function writeConfiguration(&$var, $filename='configuration.cache') {
  $filename = DIR_MY_CACHE . $filename;
  $success = false;

  if ($fp = @fopen($filename, 'w')) {
    flock($fp, 2); // LOCK_EX         
    fputs($fp, serialize($var));
    flock($fp, 3); // LOCK_UN
    fclose($fp);
    $success = true;
  }

  return $success;
}

// 读取配置信息的缓存文件
function readConfiguration(&$var, $filename='configuration.cache'){
  $filename = DIR_MY_CACHE . $filename;
  $success = false;

  if ($fp = @fopen($filename, 'r')) {
    $szdata = fread($fp, filesize($filename));
    fclose($fp);
    $var = unserialize($szdata);
    $success = true;
  }
  
  return $success;
}

if (!readConfiguration($result)) {
  // 如果没有配置缓存文件,则会先获取所有配置,然后创建缓存文件
  $result = array();

  $configuration_query = tep_db_query('select configuration_key as cfgKey, configuration_value as cfgValue from ' . TABLE_CONFIGURATION);
  while ($configuration = tep_db_fetch_array($configuration_query)) {
    $result[] = array('key'=>$configuration['cfgKey'],'value'=>$configuration['cfgValue']);
  }

  tep_db_free_result($configuration_query);
  writeConfiguration($result);
}
                                                                   
foreach ($result as $value) {
  define($value['key'],$value['value']);
}
```

4、修改配置文件:admin/includes/configure.php 添加如下定义代码:

`define(‘DIR_MY_CACHE’, DIR_FS_CATALOG . ’cache/’);`

5、修改文件:admin/configuration.php

查找:

```php
tep_db_query("update " . TABLE_CONFIGURATION . " set configuration_value = '" . tep_db_input($configuration_value) . "', last_modified = now() where configuration_id = '" . (int)$cID . "'");
```

追加以下代码:

`updateConfiguration();`

6、修改文件:admin/modules.php 查找:

```php
  tep_db_query("update " . TABLE_CONFIGURATION . " set configuration_value = '" . $value . "' where configuration_key = '" . $key . "'");
}
```

追加以下代码:

`updateConfiguration();`

7、修改文件:admin/includes/functions/general.php

在最后的“?>”之前追加 writeConfiguration 函数和 updateConfiguration 函数:

```php
function writeConfiguration(&$var, $filename='configuration.cache') {
  $filename = DIR_MY_CACHE . $filename;
  $success = false;
  if ($fp = @fopen($filename, 'w')) {       
    flock($fp, 2); // LOCK_EX
    fputs($fp, serialize($var));
    flock($fp, 3); // LOCK_UN
    fclose($fp);
    $success = true;
  }

  return $success;
}

function updateConfiguration($filename='configuration.cache') {
  $result = array();
  $configuration_query = tep_db_query('select configuration_key as cfgKey, configuration_value as cfgValue from ' . TABLE_CONFIGURATION);
  
  while ($configuration = tep_db_fetch_array($configuration_query)) {
    $result[] = array('key'=>$configuration['cfgKey'],'value'=>$configuration['cfgValue']);
  }

  tep_db_free_result($configuration_query);
  writeConfiguration($result,$filename);
}
```
