
## 管理后台

### MVC原理

我们先以后台登陆为例说明 osCommerce V3 的管理后台 MVC 工作原理,登陆 URL 为: admin/index.php?login

主执行文件 admin/index.php

```php
require('includes/application_top.php'); //加载必须的类与函数库,并进行权限检查 
require('includes/classes/template.php'); // 加载模板类

$_SESSION['module'] = 'index'; // 默认模块为 index
if ( !empty($_GET) ) { // 如果设置了模块名称,则使用相应的模块
  $first_array = array_slice($_GET, 0, 1); // 模块名为$_GET 参数的第一个参数,且只有 KEY,没有 VALUE
  $_module = osc_sanitize_string(basename(key($first_array))); //以登陆 URL “admin/index.php?login”为例,此处的模块名应为“login”

  if ( file_exists('includes/applications/' . $_module . '/' . $_module . '.php') ) { // 进一步确认模块文件是否有效
    $_SESSION['module'] = $_module; // 设置“login”为当前的模块名 }
  }
}

if ( !osC_Access::hasAccess($_SESSION['module']) ) { // 模测对当前模块是否有操作权限 
  $osC_MessageStack->add('header', 'No access.', 'error');
  osc_redirect_admin(osc_href_link_admin(FILENAME_DEFAULT));
}

$osC_Language->loadIniFile($_SESSION['module'] . '.php'); // 加载语言文件

require('includes/applications/' . $_SESSION['module'] . '/' . $_SESSION['module'] . '.php');

// 加载模块类,管理后台的所有模块类都继承自 osC_Template_Admin 后台模块类,而 osC_Template_Admin 类又是 osC_Template 类的子类,所以后台模板系统采用了前台模板的框架,加上后 台的标准化,也就使得后台 MVC 结构可以更灵活,实现的代码反而更少。

$osC_Template = osC_Template_Admin::setup($_SESSION['module']); // 配置模板 
$osC_Template->set('default'); // 设置当前模板

require('templates/default.php'); // 加载模板
require('includes/application_bottom.php');
```

osC_Template_Admin 类 admin/includes/classes/template.php

```php
require('../includes/classes/template.php');

class osC_Template_Admin extends osC_Template {
  function &setup($module) { // 配置模板方法
    $class = 'osC_Application_' . ucfirst($module);
    if ( isset($_GET['action']) && !empty($_GET['action']) ) { // 由$_GET 参数“action ”指定动作
      $_action = osc_sanitize_string(basename($_GET['action']));

      if ( file_exists('includes/applications/' . $module . '/actions/' . $_action . '.php') ) {
        include('includes/applications/' . $module . '/actions/' . $_action . '.php'); // 动作保存的目录是“includes/applications/MODULE/actions/ACTION.php”,其中 MODULE 为模块名,ACTION 为动作名
        $class = 'osC_Application_' . ucfirst($module) . '_Actions_' . $_action; // 动作类的名称规则是“osC_Application_MODULE_Actions_ACTION” }
      }

      $object = new $class();

      // 动作的代码就放置在其初始化方法里,一般在执行完成后会跳转到新 的页面,从而下面的代码将不会被执行,只有动作执行出错时或者该动作只是为了预处理相关信息,才会继续下面的步骤
      // 当没有设置$_GET[‘action’]时,则不会执行动作,而是会转向到显示模块内容,当我们需要登陆时,此 时的$class 等于“osC_Application_Login”
      return $object;
    }
  }
```

osC_Application_Login 登陆类 admin/includes/applications/login/login.php

```php
class osC_Application_Login extends osC_Template_Admin {
  /* Protected variables */
  // 每个后台模块类大体都较简单,因为它们继承自 osC_Template_Admin 类,从而间接地继承于 osC_Template 类,所以代码都比较精简。模块类最重要的参数便是以下三个:

  protected $_module = 'login', // $_module指定当前的模块名
  $_page_title, // $_page_title指定当前的标题
  $_page_contents = 'main.php'; // $_page_contents指定内容模块的文件名,内容模块位于“includes/applications/MODULE/pages/”

  /* Class constructor */
  public function __construct() {
    global $osC_Language;

    $this->_page_title = $osC_Language->get('heading_title'); // 从语言文件中读取标 题
  }
}
```

显示模板 admin/templates/default.php [79]

```php
<div class="pageContents">
<?php require('includes/applications/' . $osC_Template->getModule() . '/pages/' . $osC_Template->getPageContentsFilename()); ?>
</div>
```

除去加载 header 和 footer 的部分,最主要的代码便是上面一段,这段代码决定了版面中将要 以什么内容为主角。

如下面的代码所示, osC_Template 类的 getModule 方法会返回后台模块类的$_module 变量, osC_Template 类的 getPageContentsFilename 方法会返回后台模块类的$_page_contents 变量的值。 

因此当我们的 URL 为“ admin/index.php?login ”时,其显示的中心内容将是 “includes/applications/login/pages/main.php”文件的内容,而该文件的内容正是要显示一个登陆表单

osC_Template 类的 getModule 方法 includes/classes/template.php[253]

```php
function getModule() {
  return $this->_module;
}
```

osC_Template 类的 getPageContentsFilename 方法 includes/classes/template.php [344]

```php
function getPageContentsFilename() {
  return $this->_page_contents;
}
```

*处理登陆*

当我们输入了用户名和密码,进行登陆时,我们的 URL 是
“admin/index.php?login&action=process”,同时通过$_POST 将用户名和密码也传输过来。

与显示登陆界面的不同之处在于:当执行到 osC_Template_Admin 类的 setup 方法时,上面提 到会检测 $_GET[‘action’] 值,以此来判断是否需要执行动作,在我们的 URL “admin/index.php?login&action=process”里,“process”会被作为动作名,从而会加载动作类 osC_Application_Login_Action_process (位于“includes/applications/login/actions/process.php”) , 同时对其进行实例化。

osC_Application_Login_Actions_process 类 admin/includes/applications/login/actions/process.php

```php
class osC_Application_Login_Actions_process extends osC_Application_Login {
  public function __construct() {
    global $osC_Database, $osC_Language, $osC_MessageStack;
  
    parent::__construct();
    if ( !empty($_POST['user_name']) && !empty($_POST['user_password']) ) {
      $Qadmin = $osC_Database->query('select id, user_name, user_password from :table_administrators where user_name = :user_name');
      $Qadmin->bindTable(':table_administrators', TABLE_ADMINISTRATORS);
      $Qadmin->bindValue(':user_name', $_POST['user_name']);
      $Qadmin->execute();
      if ( $Qadmin->numberOfRows() ) { // 查找是否存在指定的用户名 
        if ( osc_validate_password($_POST['user_password'], $Qadmin->value('user_password')) ) { // 比对密码
          $_SESSION['admin'] = array(
            'id' => $Qadmin->valueInt('id'),
            'username' => $Qadmin->value('user_name'),
            'access' => osC_Access::getUserLevels($Qadmin->valueInt('id'))
          ); // 加载用户的权限

          $get_string = null;
          if ( isset($_SESSION['redirect_origin']) ) {
            $get_string = http_build_query($_SESSION['redirect_origin']['get']);
            unset($_SESSION['redirect_origin']);
          }

          osc_redirect_admin(osc_href_link_admin(FILENAME_DEFAULT, $get_string)); // 如果登陆成功,则会自动跳转到登陆前的页面
        }
      }
    }

    $osC_MessageStack->add('header', $osC_Language->get('ms_error_login_invalid'), 'error'); // 如果登陆失败,则会输出错误信 息
  }
}
```

总结以上的介绍,可以得出下面的类关系图:


页面的切换

页面的切换需要先由动作触发,我们以编辑币种为例。

osC_Application_Currencies_Actions_save 类 admin/includes/application/currencies/actions/save.php 

```php
class osC_Application_Currencies_Actions_save extends osC_Application_Currencies
{
  public function __construct() {
    global $osC_Language, $osC_MessageStack;

    parent::__construct();

    if ( isset($_GET['cID']) && is_numeric($_GET['cID']) ) { // 当触发 osC_Application_Currencies 类的动作 save 时,有两种可能:一是编辑现有币种,第二是新建币种
      $this->_page_contents = 'edit.php'; // 通过设置内部变量$_page_contents 的值,即 可以更改主内容
    } else {
      $this->_page_contents = 'new.php'; // 显示新建币种的页面
    }

    if ( isset($_POST['subaction']) && ($_POST['subaction'] == 'confirm') ) { // 当 提交了特定的参数,表示编辑完成,需要提交输入的内容,此处便进入了动作的调用
       $data = array(
        'title' => $_POST['title'],
        'code' => $_POST['code'],
        'symbol_left' => $_POST['symbol_left'],
        'symbol_right' => $_POST['symbol_right'],
        'decimal_places' => $_POST['decimal_places'],
        'value' => $_POST['value']
      );

      if ( osC_Currencies_Admin::save((isset($_GET['cID']) && is_numeric($_GET['cID']) ? $_GET['cID'] : null), $data, ((isset($_POST['default']) && ($_POST['default'] == 'on')) || (isset($_POST['is_default']) && ($_POST['is_default'] == 'true') && ($_POST['code'] != DEFAULT_CURRENCY)))) ) {
        $osC_MessageStack->add($this->_module,
$osC_Language->get('ms_success_action_performed'), 'success');
      } else {
        $osC_MessageStack->add($this->_module,

        $osC_Language->get('ms_error_action_not_performed'), 'error');
      }
      
      osc_redirect_admin(osc_href_link_admin(FILENAME_DEFAULT, $this->_module));
    }
  }
}
```

下面的流程图,显示了动作类区分显示内容与执行动作的过程。


### 子模块的调用

单独的模块在 osCommerce V3 里是怎么调用的呢?其实是通过传递$_GET[‘module’]参数来 指定的,当然在代码方面还必须给予相关的支持。

因为在 osCommerce V3 里统计分为“低库存”、“销售统计”、“畅销产品统计”和“产品浏览统计”,所以使用了模块来进行管理。下面便以“低库存统计”为例说明模块的调用过程

当显示统计页面时,首先调用的是统计模板类:

osC_Application_Statistics 类 admin/includes/applications/statistics/statistics.php

```php
class osC_Application_Statistics extends osC_Template_Admin {
  /* Protected variables */
  protected $_module = 'statistics',
    $_page_title,
    $_page_contents = 'main.php'; // 文件 main.php 将负责显示子模块的内容

  /* Class constructor */
  function __construct() {
    global $osC_Language;

    $this->_page_title = $osC_Language->get('heading_title'); // 从语言文件里读取标 题
    if ( !isset($_GET['module']) ) { // 如果设置了$_GET[‘module’],则表示点击了详细的统 计项,需要进一步显示详细的内容,如“低库存统计”传递的$_GET[‘module’]值等于“low_stock”
      $_GET['module'] = '';
    }

    if ( !isset($_GET['page']) || ( isset($_GET['page']) && !is_numeric($_GET['page']) ) ) { // 跟踪页码
      $_GET['page'] = 1;
    }

    if ( !empty($_GET['module']) && !file_exists('includes/modules/statistics/' . $_GET['module'] . '.php') ) {
      $_GET['module'] = '';
    }

    if ( empty($_GET['module']) ) { // 除非设置了子模块,否则将显示出所有可用子模块,供用 户选择
      $this->_page_contents = 'listing.php'; // 文件“listing.php”列示出所有可用的子模块
    }
  }
}
```

初始化子模块 admin/includes/applications/statistics/pages/main.php[15]

```php
include('includes/modules/statistics/' . $_GET['module'] . '.php'); //加载子模块, 子模块的保存位置是“admin/includes/modules/MODULE/”,其中 MODULE 是当前模板类名称,如“ statistics”
$class = 'osC_Statistics_' . str_replace(' ', '_', ucwords(str_replace('_', ' ', $_GET['module']))); // 子模块类名规则:osC_Statistics_ + 当前模板类名 + 子模块名
$osC_Statistics = new $class(); // 实例化子模块类 $osC_Statistics->activate(); // 激活子模块,执行必要的数据加载工作
```

osC_Statistics 类 admin/includes/classes/statistics.php[39]

```php
function activate() {
  $this->_setHeader();
  $this->_setData();
}
```

通过上面 osC_Statistics 类的 activate 方法,可以了解到它执行了_setData 方法,而负责“低库存 统计”的 osC_Statistics_Low_Stock 类在统计低库存数据时正是通过_setData 方法实现的

osC_Statistics_Low_Stock 类 admin/includes/modules/statistics/low_stock.php[51]

```php
function _setData() {
  global $osC_Database, $osC_Language;

  $this->_data = array();
  $this->_resultset = $osC_Database->query('select p.products_id, pd.products_name, products_quantity from :table_products p, :table_products_description pd where p.products_id = pd.products_id and pd.language_id = :language_id and p.products_quantity <= :stock_reorder_level order by p.products_quantity desc');
  $this->_resultset->bindTable(':table_products', TABLE_PRODUCTS);
  $this->_resultset->bindTable(':table_products_description', TABLE_PRODUCTS_DESCRIPTION);
  $this->_resultset->bindInt(':language_id', $osC_Language->getID());
  $this->_resultset->bindInt(':stock_reorder_level', STOCK_REORDER_LEVEL);
  $this->_resultset->setBatchLimit($_GET['page'], MAX_DISPLAY_SEARCH_RESULTS);
  
  $this->_resultset->execute();
  while ( $this->_resultset->next() ) {
    $this->_data[] = array(osc_link_object(osc_href_link_admin(FILENAME_DEFAULT, 'products&pID=' . $this->_resultset->valueInt('products_id') . '&action=preview'), $this->_icon . '&nbsp;' . $this->_resultset->value('products_name')), $this->_resultset->valueInt('products_quantity')); // 将数据保存至内部变量$_data
  }
}
```

显示子模块内容 admin/includes/applications/statistics/pages/main.php [67]

```php
<?php
foreach ( $osC_Statistics->getData() as $data ) { // osC_Statistics 类的 getData 方法会得到其内部变量$_data 的值
if ( !isset($columns) ) {
  $columns = sizeof($data);
} ?>
<tr onmouseover="rowOverEffect(this);" onmouseout="rowOutEffect(this);">
<?php
for ( $i = 0; $i < $columns; $i++ ) {
  echo '<td>' . $data[$i] . '</td>' . "\n";
}
?>
</tr>
<?php } ?>
                 
### RPC原理

RPC(全称:Remote Procedure Call)远程过程调用,也是osCommerce V3里新增的功能。 RPC 可以允许其它程序控制 osCommerce 系统的操作,例如获取产品分类、删除指定产品资料, 或者备份数据库等,可以说凡是可以通过后台操作的功能,只要实现了 RPC,便可以通过远程进行操作。

osCommerce V3 RPC 的调用是由后台的“rpc.php”文件开始的:

远程过程调用 admin/rpc.php

```php
header('Cache-Control: no-cache, must-revalidate'); 
header('Expires: Mon, 26 Jul 1997 05:00:00 GMT'); // 设置为不进行缓存
require('includes/application_top.php'); // RPC调用结果标志符
define('RPC_STATUS_SUCCESS', 1); // 执行成功 
define('RPC_STATUS_NO_SESSION', -10); // 未登陆/登陆已过期 define('RPC_STATUS_NO_MODULE', -20); // 未设置模块/模块不正确 define('RPC_STATUS_NO_ACCESS', -50); // 权限受限 define('RPC_STATUS_CLASS_NONEXISTENT', -60); //不存在指定的类 define('RPC_STATUS_NO_ACTION', -70); // 未设置动作 define('RPC_STATUS_ACTION_NONEXISTENT', -71); // 动作不存在

if ( !isset($_SESSION['admin']) ) { // 判断登陆
  echo json_encode(array('rpcStatus' => RPC_STATUS_NO_SESSION)); exit;
}

$module = null;
$class = null;
if ( empty($_GET) ) { // 判断模块
  echo json_encode(array('rpcStatus' => RPC_STATUS_NO_MODULE));
  exit;
} else {
  $first_array = array_slice($_GET, 0, 1);
  $_module = osc_sanitize_string(basename(key($first_array))); // 模块为$_GET[0]
  if ( !osC_Access::hasAccess($_module) ) { // 判断操作权限
    echo json_encode(array('rpcStatus' => RPC_STATUS_NO_ACCESS)); exit;
  }

  $class = (isset($_GET['class']) && !empty($_GET['class'])) ? osc_sanitize_string(basename($_GET['class'])) : 'rpc'; // 默认调用类“rpc”,否则使用 $_GET[‘class’]指定类名
  $action = (isset($_GET['action']) && !empty($_GET['action'])) ? osc_sanitize_string(basename($_GET['action'])) : ''; // 使用$_GET[‘action’]指定动作 (即类的方法)
 
  if ( empty($action) ) {
    echo json_encode(array('rpcStatus' => RPC_STATUS_NO_ACTION));
    exit;
  }
  
  if ( file_exists('includes/applications/' . $_module . '/classes/' . $class . '.php')) {
    include('includes/applications/' . $_module . '/classes/' . $class . '.php'); // 加载类
    if ( method_exists('osC_' . ucfirst($_module) . '_Admin_' . $class, $action) ) {
      call_user_func(array('osC_' . ucfirst($_module) . '_Admin_' . $class,
$action)); // 调用类的方法,结果一般为 JSON 代码 
      exit;
    } else {
      echo json_encode(array('rpcStatus' => RPC_STATUS_ACTION_NONEXISTENT));
      exit;
    }
  } else {
    echo json_encode(array('rpcStatus' => RPC_STATUS_CLASS_NONEXISTENT));
    exit;
  }
}
```

实例:

osC_Countries_Admin_rpc 国家 RPC 类 admin/includes/applications/countries/classes/rpc.php

```php
class osC_Countries_Admin_rpc {
  public static function getAll() { // 方法 getAll 获取所有国家列表或者搜索指定国家
    if ( !isset($_GET['search']) ) { // $_GET[‘search’]指定需要搜索的国家 $_GET
      ['search'] = '';
    }

    if ( !isset($_GET['page']) || !is_numeric($_GET['page']) ) { // 指定页码 
      $_GET['page'] = 1;
    }

    if ( !empty($_GET['search']) ) {
      $result = osC_Countries_Admin::find($_GET['search'], $_GET['page']); // 查找指定国家
    } else {
      $result = osC_Countries_Admin::getAll($_GET['page']); // 获取指定页码的所有国家
    }

    $result['rpcStatus'] = RPC_STATUS_SUCCESS; // 结果会是一个关联数组,结构为 array(‘entries‘=>包含所有数据的数组,’total’=>结果数,’ rpcStatus’=>1)
    echo json_encode($result); // 将结果进行 JSON 编码
  }

  public static function getAllZones() { // getAllZones 方法获取指定国家的所有省份/区 信息,或者搜索指定国家内的省份/区信息
    global $_module;

    if ( !isset($_GET['search']) ) {
      $_GET['search'] = '';
    }

    if ( !empty($_GET['search']) ) {
      $result = osC_Countries_Admin::findZones($_GET['search'], $_GET[$_module]);
    } else {
      $result = osC_Countries_Admin::getAllZones($_GET[$_module]);
    }

    $result['rpcStatus'] = RPC_STATUS_SUCCESS;
    echo json_encode($result);
  }
}
```

osC_Countries_Admin 类 admin/includes/applications/countries/classes/coutries.php[37]

```php
public static function getAll($pageset = 1) {// 获取某一页的所有国家
  global $osC_Database;
  if ( !is_numeric($pageset) || (floor($pageset) != $pageset) ) {
    $pageset = 1;
  }

  $result = array('entries' => array());
  $Qcountries = $osC_Database->query('select SQL_CALC_FOUND_ROWS * from  :table_countries order by countries_name');
  $Qcountries->bindTable(':table_countries', TABLE_COUNTRIES);
  if ( $pageset !== -1 ) {
    $Qcountries->setBatchLimit($pageset, MAX_DISPLAY_SEARCH_RESULTS);
  }
  
  $Qcountries->execute();
  while ( $Qcountries->next() ) {
    $Qzones = $osC_Database->query('select count(*) as total_zones from :table_zones where zone_country_id = :zone_country_id');
    $Qzones->bindTable(':table_zones', TABLE_ZONES);
    $Qzones->bindInt(':zone_country_id',
    $Qcountries->valueInt('countries_id'));
    $Qzones->execute();

    $result['entries'][] = array_merge($Qcountries->toArray(), $Qzones->toArray());// 将国家及该国所有省份/区信息存放于$result[‘entries’]
  }

  $result['total'] = $Qcountries->getBatchSize(); // $result['total']保存结果数
  if ( $Qcountries->numberOfRows() > 0 ) {
    $Qzones->freeResult();
  }

  $Qcountries->freeResult();
  return $result;
}

public static function find($search, $pageset = 1) { // 查找指定页码里的国家
  global $osC_Database;

  if ( !is_numeric($pageset) || (floor($pageset) != $pageset) ) {
    $pageset = 1;
  }

  $result = array('entries' => array());
  $Qcountries = $osC_Database->query('select SQL_CALC_FOUND_ROWS c.* from :table_countries c left join :table_zones z on (z.zone_country_id = c.countries_id) where (c.countries_name like :countries_name or c.countries_iso_code_2 like :countries_iso_code_2 or c.countries_iso_code_3 like :countries_iso_code_3 or z.zone_name like :zone_name or z.zone_code like :zone_code) group by c.countries_id order by c.countries_name');
  $Qcountries->bindTable(':table_countries', TABLE_COUNTRIES); 
  $Qcountries->bindTable(':table_zones', TABLE_ZONES); 
  $Qcountries->bindValue(':countries_name', '%' . $search . '%'); // 在国家名称里查找
  $Qcountries->bindValue(':countries_iso_code_2', '%' . $search . '%'); // 或者在ISO CODE2里查找
  $Qcountries->bindValue(':countries_iso_code_3', '%' . $search . '%'); // 又或者在ISO CODE3里查找
  $Qcountries->bindValue(':zone_name', '%' . $search . '%'); // 或者使用省份名查找 
  $Qcountries->bindValue(':zone_code', '%' . $search . '%'); // 或者使用省份编码查找
  
  if ( $pageset !== -1 ) {
    $Qcountries->setBatchLimit($pageset, MAX_DISPLAY_SEARCH_RESULTS);
  }

  $Qcountries->execute();
  while ( $Qcountries->next() ) {
    $Qzones = $osC_Database->query('select count(*) as total_zones from :table_zones where zone_country_id = :zone_country_id');
    $Qzones->bindTable(':table_zones', TABLE_ZONES);
    $Qzones->bindInt(':zone_country_id',
    $Qcountries->valueInt('countries_id'));
    $Qzones->execute();
    $result['entries'][] = array_merge($Qcountries->toArray(),
    $Qzones->toArray());
  }

  $result['total'] = $Qcountries->getBatchSize();
  if ( $Qcountries->numberOfRows() > 0 ) {
    $Qzones->freeResult();
  }

  $Qcountries->freeResult();
  return $result;
}

public static function findZones($search, $country_id) { // 查找指定的国家里的省份 /区信息
  global $osC_Database;

  $result = array('entries' => array());
  $Qzones = $osC_Database->query('select SQL_CALC_FOUND_ROWS * from :table_zones where zone_country_id = :zone_country_id and (zone_name like :zone_name or zone_code like :zone_code) order by zone_name');
  $Qzones->bindTable(':table_zones', TABLE_ZONES);
  $Qzones->bindInt(':zone_country_id', $country_id);

  $Qzones->bindValue(':zone_name', '%' . $search . '%');
  $Qzones->bindValue(':zone_code', '%' . $search . '%');
  $Qzones->execute();
  while ( $Qzones->next() ) {
    $result['entries'][] = $Qzones->toArray();
  }
  
  $result['total'] = $Qzones->numberOfRows();
  $Qzones->freeResult();
  return $result;
}

public static function getAllZones($country_id) {// 获取指定国家的所有省份/区信息 
  global $osC_Database;

  $result = array('entries' => array());
  $Qzones = $osC_Database->query('select * from :table_zones where zone_country_id = :zone_country_id');
  $Qzones->bindTable(':table_zones', TABLE_ZONES);
  $Qzones->bindInt(':zone_country_id', $country_id);
  $Qzones->execute();
  while ( $Qzones->next() ) {
    $result['entries'][] = $Qzones->toArray();
  }
  $result['total'] = $Qzones->numberOfRows();
  $Qzones->freeResult();
  
  return $result;
}
```

> 提示:
> 
> osCommerce V3 默认对 RPC 类只提供了获取参数的功能,并且功能非常有限。如果需要完全通过 RPC 控制后台的操作,则需要自行增加 RPC 方法,来处理数据的响应。
