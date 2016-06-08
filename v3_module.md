## 模块系统

### 初始化

下面将以文件“index.php”为例来说明 osCommerce V3 模块系统

文件:index.php[21]

```php
$osC_Template = osC_Template::setup('index');

require('templates/' . $osC_Template->getCode() . '.php');
```

第 21 行代码,调用 osC_Template 类的静态方法 setup 初始化模块,参数“index”指定了需要显 示的主模板。

第 23 行代码通过得到的 osC_Template 对象的方法 getCode 获得模板的 Code 文件,即模板系统的 主显示文件,并执行它。

osC_Template 类文件:includes/classes/templates.php[178]

```php
function &setup($module) {
  $group = basename($_SERVER['SCRIPT_FILENAME']);
  if (($pos = strrpos($group, '.')) !== false) {
    $group = substr($group, 0, $pos); // 当调用文件为“index.php”时,$group 的值将为 “index” 
  }

  if (empty($_GET) === false) {
    $first_array = array_slice($_GET, 0, 1); // GET 参数的第一位被设定为自动模块设置, 也就是说 GET 参数的第一位将被替代参数$module 的作用
    $_module = osc_sanitize_string(basename(key($first_array))); // 只取第一位参数的 KEY
    if (file_exists('includes/content/' . $group . '/' . $_module . '.php')) {
      $module = $_module;
    }
  }

  include('includes/content/' . $group . '/' . $module . '.php'); // 默认情况下, 将调用“includes/content/index/index.php”文件,此文件将输出首页的主体内容
  $_page_module_name = 'osC_' . ucfirst($group) . '_' . ucfirst($module); // 主 模块是类,类名的格式是:osC_GROUP_MODULE,GROUP:组名(主文件名去除后缀)、 MODULE:主模块名
  $object = new $_page_module_name(); // 初始化主模块

  // 主模块在初始化后就已经完成了页面主体内容的准备功能,初始化后,$object->_page_title 内 部参数保存了主模块的标题(通常也是页面的标题),$object->_page_contents 内部参数保存最终需要显 示的主体内容,因为主模块类都是osC_Template类的子类,所以可以通过osC_Template的方法来访问这些 参数并输出。

  if ( isset($_GET['action']) && !empty($_GET['action']) ) { // GET参数的action 指定全局动作,如加入购物车、移出购物车等
    include('includes/classes/actions.php');
    osC_Actions::parse($_GET['action']); // 动作类 osC_Action 只有一个方法 parse 用于 分析处理全局动作
  }
  return $object; // 返回主模块对象,用于最终的输出
}
```

osC_Template 的方法 setup 只有一个参数:主模块$module,该方法会返回对象,对象的类型取决 于参数$module 的值。

>  提示:
>
> 文件夹“includes/content/”存放的是每个页的主体内容(页面的中间部分),同时文件按约定目录 及名称保存,首先是在“includes/content”下以主文件名(如 index.php、account.php 等)为目录, 再在其下以主模块作为文件的文件名。

 
### 工作原理

osC_Index_Index 类文件:includes/content/index/index.php

```php
class osC_Index_Index extends osC_Template { // 所有主模块类继承自 osC_Template 模板类 /* Private variables */
  var $_module = 'index',
      $_group = 'index',
      $_page_title,
      $_page_contents = 'index.php',
      $_page_image = 'table_background_default.gif';

  /* Class constructor */
  function osC_Index_Index() {
    global $osC_Database, $osC_Services, $osC_Language, $osC_Breadcrumb, $cPath,
           $cPath_array, $current_category_id, $osC_Category;

    $this->_page_title = sprintf($osC_Language->get('index_heading'), STORE_NAME);

    // 用内部变量_page_title 保存页面的标题
    if (isset($cPath) && (empty($cPath) === false)) {
      if ($osC_Services->isStarted('breadcrumb')) {
        $Qcategories = $osC_Database->query('select categories_id, categories_name from :table_categories_description where categories_id in (:categories_id) and language_id = :language_id');
        $Qcategories->bindTable(':table_categories_description', TABLE_CATEGORIES_DESCRIPTION);
        $Qcategories->bindTable(':categories_id', implode(',', $cPath_array));
        $Qcategories->bindInt(':language_id', $osC_Language->getID());
        $Qcategories->execute();
                                                           

        $categories = array();
        while ($Qcategories->next()) {
          $categories[$Qcategories->value('categories_id')] = $Qcategories->valueProtected('categories_name'); // 查询当前产品分类及其以上所有分类的名 称与 ID,并保存到关联数组$categories
        }
        $Qcategories->freeResult();
        for ($i=0, $n=sizeof($cPath_array); $i<$n; $i++) {
          $osC_Breadcrumb->add($categories[$cPath_array[$i]],
          osc_href_link(FILENAME_DEFAULT, 'cPath=' . implode('_', array_slice($cPath_array, 0, ($i+1))))); // 创建产品分类的 Breadcrumb 导航
        }
      }

      $osC_Category = new osC_Category($current_category_id); // 类 osC_Category 用 于处理分类的查询
      $this->_page_title = $osC_Category->getTitle(); // 以当前分类的名称作为页面的标题 ,getTitle 方法返回当前分类的名称
      if ( $osC_Category->hasImage() ) {
        $this->_page_image = 'categories/' . $osC_Category->getImage();// 当前分类的图片取代默认的图片
      }

      $Qproducts = $osC_Database->query('select products_id from :table_products_to_categories where categories_id = :categories_id limit 1');
      $Qproducts->bindTable(':table_products_to_categories', TABLE_PRODUCTS_TO_CATEGORIES);
      $Qproducts->bindInt(':categories_id', $current_category_id);
                                                                 

      $Qproducts->execute();
      if ($Qproducts->numberOfRows() > 0) { // 当前分类下是否有直属产品
        $this->_page_contents = 'product_listing.php'; // 将调用“includes/modules/product_listing.php”文件显示出当前分类所有的产品
        $this->_process(); // 处理过滤以及排序 } else {
        $Qparent = $osC_Database->query('select categories_id from :table_categories where parent_id = :parent_id limit 1');
        $Qparent->bindTable(':table_categories', TABLE_CATEGORIES);
        $Qparent->bindInt(':parent_id', $current_category_id);
        $Qparent->execute();
        if ($Qparent->numberOfRows() > 0) { // 否则显示当前分类属下的所有产品分类
          $this->_page_contents = 'category_listing.php';
        } else { // 将显示没有任何内容的提示信息
          $this->_page_contents = 'product_listing.php';
          $this->_process();
        }
      }
    }
  }

  /* Private methods */
  function _process() {
    global $current_category_id, $osC_Products;
    include('includes/classes/products.php');
    $osC_Products = new osC_Products($current_category_id);

    if (isset($_GET['filter']) && is_numeric($_GET['filter']) && ($_GET['filter'] > 0)) {
      $osC_Products->setManufacturer($_GET['filter']);
    }

    if (isset($_GET['sort']) && !empty($_GET['sort'])) {
      if (strpos($_GET['sort'], '|d') !== false) {
        $osC_Products->setSortBy(substr($_GET['sort'], 0, -2), '-');
      } else {
        $osC_Products->setSortBy($_GET['sort']);
      }
    }
  }
}
```

上面是整个主模块文件“index.php”,osC_index_index 类只有两个方法,一个是初始化方法 osC_Index_Index,另一个是_process 方法,初始化方法用于输出显示的内容,_process 方法是一 个内部方法(private),是可选方法,通常用于处理更复杂的数据交互。

介绍完主模块文件,接着就要开始模板的显示任务了,我们回到 index.php 的第 23 行:

`require('templates/' . $osC_Template->getCode() . '.php');`

这行代码便是开始加载模板的关键。

osC_Templae 模板类文件:includes/classes/template.php[230]

```php     
function getCode($id = null) {
  if (isset($this->_template) === false) {
    $this->set(); // 设定初始模板 
  }
  
  if (is_numeric($id)) { // 如果指定了模板 ID,则需要变更初始模板
      foreach ($this->getTemplates() as $template) {
        if ($template['id'] == $id) {
          return $template['code']; // 得到需要的模板
        }
      }
  } else {
    return $this->_template; // 使用初始模板进行显示
  }
}
```

> 提示:
> 
> 模块 code 决定了模块的文件名,即模块文件名=模板 code.php,并且模板必须保存在“templates/” 目录

设置初始模板 includes/classes/template.php [490]

```php
function set($code = null) {
  if ( (isset($_SESSION['template']) === false) || !empty($code) || (isset($_GET['template']) && !empty($_GET['template'])) ) { // 使用会话 template 来保 存当前的模板
    if ( !empty( $code ) ) {
      $set_template = $code; // 如参数$code 指定模板
    } else {
      $set_template = (isset($_GET['template']) && !empty($_GET['template'])) ? $_GET['template'] : DEFAULT_TEMPLATE; // 或者使用$_GET 参数 template 当作当前模板(这样便 可允许临时切换模板,对于新模板的调试非常有用),否则使用默认的模板
    }
    
    $data = array();
    $data_default = array();
    foreach ($this->getTemplates() as $template) { // getTemplates 方法获得所有可用 的模板
      if ($template['code'] == DEFAULT_TEMPLATE) {
        $data_default = array('id' => $template['id'], 'code' => $template['code']);
      } elseif ($template['code'] == $set_template) {
        $data = array('id' => $template['id'], 'code' => $template['code']);
      }
    }

    if (empty($data)) {
      $data =& $data_default;
    }
    
    $_SESSION['template'] =& $data;
  }

  $this->_template_id =& $_SESSION['template']['id'];
  $this->_template =& $_SESSION['template']['code'];
  // 内部变量$_template_id(默 认模板 ID)和$_template(默认模板 code)与会话$_SESSION[‘template’][‘id’]、 $_SESSION[‘template’][‘code’]是引用的关系。所以无论是修改内部变量的值或者是修改$_SESSION 的 值,两边的值都保证是同步的
}
```

获得可用模板 includes/classes/template.php [376]

```php
function &getTemplates() {
  global $osC_Database;
  
  $templates = array();
  // 直接通过数据库查询可用模板,并且开启的缓存功能
  $Qtemplates = $osC_Database->query('select id, code, title from :table_templates');
  $Qtemplates->bindTable(':table_templates', TABLE_TEMPLATES);
  $Qtemplates->setCache('templates');//开启缓存 $Qtemplates->execute();
  
  while ($Qtemplates->next()) {
    $templates[] = $Qtemplates->toArray(); // 得到所有可用模板
  }

  $Qtemplates->freeResult();
  return $templates;
}
```

模板文件 templates/default.php[15]

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" dir="<?php echo
$osC_Language->getTextDirection(); ?>" xml:lang="<?php echo
$osC_Language->getCode(); ?>" lang="<?php echo $osC_Language->getCode(); ?>">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=<?php echo
$osC_Language->getCharacterSet(); ?>" />
<title><?php echo STORE_NAME . ($osC_Template->hasPageTitle() ? ': ' .
$osC_Template->getPageTitle() : ''); ?></title>
<base href="<?php echo osc_href_link(null, null, 'AUTO', false); ?>" />
<script language="javascript" src="ext/jquery/jquery-1.3.2.min.js"></script>
<link rel="stylesheet" type="text/css" href="templates/<?php echo $osC_Template->getCode(); ?>/stylesheet.css" />
<?php
  if ($osC_Template->hasPageTags()) {
   echo $osC_Template->getPageTags();
  }
  if ($osC_Template->hasJavascript()) {
   $osC_Template->getJavascript();
} ?>
```

模板文件里使用的最多的两个类是 osC_Language 和 osC_Template,因为模板是负责显示的,所 以需要切换语言(通过 osC_Language 类)以及得到需要的内容(通过 osC_Template 类)。

osC_Template 类的 getPageTitle 方法是取得页面标题,代码如下: 

osC_Template 类文件 includes/classes/template.php[275]

```php
function getPageTitle() {
  return osc_output_string_protected($this->_page_title); // 还记得在主模块文件 index.php 里已经设置了此内部变量的值吗?
}
```


方法hasPageTags和getPageTags负责管理Meta Tags,一个是判断是否存在Tags,另一个是得到 Tags。同样 hasJavascript 方法和 getJavascript 方法负责管理需要附加的 Javascript 文件。

> 提示:
>
> Meta Tags 由 osC_Template 类的内部变量$_page_tags 保存,它是一个关联数组,Key 指定了 Meta 的名称,Value 指定 Meta 的值
> Javascript 文 件 是 通 过 osC_Template 类 的 内 部 变 量 $_javascript_filenames 保 存 , $_javascript_filenames 是一个数组,所以可以在一个文件里引入多个 Javascript 文件

显示主模块内容 templates/default.php [80]

```php
if ($osC_Template->getCode() == DEFAULT_TEMPLATE) { //可能是出于执行性能的考虑,才有这样的判断,其实直接使用下面 else 的语言就可以实现加载主模块内容的任务
  include('templates/' . $osC_Template->getCode() . '/content/' . $osC_Template->getGroup() . '/' . $osC_Template->getPageContentsFilename());

  //getGroup 方法返回 osC_Template 类的内部变量 $_group,它的值是在主模块里已经定义的(如 osC_Index_Index 的$_group=’index’), getPageContentsFilename 方法便是返回内部变量$_page_contents,此值在主模块里设置,决定了最终 的结果,前文已经讲过。
} else {
  if (file_exists('templates/' . $osC_Template->getCode() . '/content/' . $osC_Template->getGroup() . '/' . $osC_Template->getPageContentsFilename())) {
    include('templates/' . $osC_Template->getCode() . '/content/' . $osC_Template->getGroup() . '/' . $osC_Template->getPageContentsFilename());
  } else {
    include('templates/' . DEFAULT_TEMPLATE . '/content/' . $osC_Template->getGroup() . '/' . $osC_Template->getPageContentsFilename());
  }
}
```

> 提示:
> 
> 在前面讲到的主模块 index.php 可能会返回三种结果(通过设置内部变量$_page_contents),下面 是三种情况的介绍:

|  返回值 |  调用文件路径 | 说明 |
------
|  index.php | templates/TEMPLATE_CODE/content/index/index.php | 首页的显示 |
| category_listing.php | templates/TEMPLATE_CODE/content/index/category_listing.php | 无直属产品的分类 将显示所有下属分类 |
| product_listing.php | templates/TEMPLATE_CODE/content/index/product_listing.php | 有产品的分类将显 示所有产品 |
