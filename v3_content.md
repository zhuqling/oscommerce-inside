## 模块与内容框

osCommerce V3 可以在后台直接设置所有模块、内容框的参数,甚至可以自定义它们的显示位置, 以及控制它们在哪些页面才生效。

下面是系统默认的所有功能模块:

以及它们的位置参数:

下面是系统默认的所有内容框:

以及它们的位置设置:


模块与内容框的位置设置参数是一样的,包含了以下几个参数:

- Pages:模块、内容框将显示在哪些页面,如果为“*”表示将出现在所有页面,如果为 “products/*”表示将出现在所有归于 products.php 主文件的页面,例如:products/info(产品介绍)、products/new(新产品页面,URL:products.php?new)等,但如果设置为“products/info”, 那么此模块将只显示在一个页面里,也就是产品介绍页面
- Page Specific:指定此模块/内容框为页面独占,如果设置了此参数,则在同一页面同一类(同一位置,如左边栏、右边栏)的其它模块或内容框将不会显示,除非那个模块/内容框也设置为页面独占
- Group:显示的位置,默认情况下内容框只有“left”和“right”两个选项可选,left即为左边栏、right 为右边栏;功能模块也只有两个可选:“before”、“after”,before 即在主内容前,after为在主内容后。如果对页面的架构有更高的要求,可以命名自定义的位置
- Sort Order:在同一类位置中的显示次序

> 提示:
>
> 如果需要自定义显示位置,不仅需要在模块/内容框里设置位置的名称,而且需要在模板文件(如 templates/default.php)里增加相应的代码,以便能够将它们显示在特定的位置上。

### 显示与位置

在主内容之前显示功能模块 templates/default.php[55]

```php
if ($osC_Template->hasPageContentModules()) { // hasPageContentModules方法总返回 True
  foreach ($osC_Services->getCallBeforePageContent() as $service) { // 如果需要在处 理功能模块之前开启的服务,则加载此服务
    $$service[0]->$service[1](); // getCallBeforePageContent方法返回的是二维数组, $service[0]是对象,$service[1]是对象的方法,即需要在处理左边栏之前运行的代码
  }

  foreach ($osC_Template->getContentModules('before') as $box) { // 获取所有类型为 “before”的功能模块类名,所有功能模块继承自 osC_Modules 类
    $osC_Box = new $box();
    $osC_Box->initialize(); // 初始化功能模板,$osC_Box 的内部变量$_content 将被赋予显示 的内容
    if ($osC_Box->hasContent()) { // hasContent是osC_Modules类的一个方法,它检查内部 变量$_content 是否有内容
      if ($osC_Template->getCode() == DEFAULT_TEMPLATE) {
        // 得到了内容后,则需要将其 显示出来,负责这一任务的是功能模块之模板,其保存路径是“ templates/TEMPLATE_NAME/modules/content/MODULE_CODE.php”,其中的 MODULE_CODE 由功能模 块类的$_code 内部变量指定,通过与功能模块名称一致,下面的$osC_Box->getCode()方法便是获得内部变 量$_code 的值
        include('templates/' . $osC_Template->getCode() . '/modules/content/' . $osC_Box->getCode() . '.php');
      } else {
        if (file_exists('templates/' . $osC_Template->getCode() . '/modules/content/' . $osC_Box->getCode() . '.php')) {
          include('templates/' . $osC_Template->getCode() . '/modules/content/' . $osC_Box->getCode() . '.php');
        } else {
          include('templates/' . DEFAULT_TEMPLATE . '/modules/content/' . $osC_Box->getCode() . '.php');
        }
      }
    }
    
    unset($osC_Box);
  }
}
```

需要提醒一点:紧跟着这段代码的后面便是我们前面讲到的显示主模块内容(行号:80),所以这段代码生成的内容将显示在主内容前面。

鉴于 osCommerce V3 的模块与内容框可自定义位置是基于其新增的若干个表,所以有必要介绍一
下模块/内容框的加载过程

osC_Modules 类 includes/classes/modules.php[127]

```php
function __construct($group) {
  // 参数有两种,“content”和“boxes”,分别对应模块和内容框, 也就是说系统使用同一个类 osC_Modules 来管理模块和内容框。同时它们都是 osC_Modules 类的子类。
  global $osC_Database, $osC_Template, $osC_Cache;

  $this->_group = $group;

  // 模块和内容框的配置也被缓存起来了,这对性能的提升相当重要
  if ($osC_Cache->read('templates_' . $this->_group . '_layout-' . $osC_Template->getCode() . '-' . $osC_Template->getGroup() . '-' . $osC_Template->getPageContentsFilename())) {
    $data = $osC_Cache->getCache();
  } else {
    $data = array();

    // 首先加载那些独占型的模块/内容框,请注意 SQL 语句中的“page_specific = 1”
    $Qspecific = $osC_Database->query('select b2p.boxes_group, b.code from :table_templates_boxes_to_pages b2p, :table_templates_boxes b, :table_templates t where b2p.templates_id = :templates_id and b2p.page_specific = 1 and b2p.content_page in (:content_page) and b2p.templates_boxes_id = b.id and b.modules_group = :modules_group and b2p.templates_id = t.id order by b2p.boxes_group, b2p.sort_order');
    $Qspecific->bindTable(':table_templates_boxes_to_pages', TABLE_TEMPLATES_BOXES_TO_PAGES);
    $Qspecific->bindTable(':table_templates_boxes', TABLE_TEMPLATES_BOXES);
    $Qspecific->bindTable(':table_templates', TABLE_TEMPLATES);
    $Qspecific->bindInt(':templates_id', $osC_Template->getID()); // 位置同样与模板相关联,这样不同的模板可以做到显示位置不同
    $Qspecific->bindRaw(':content_page', '"*", "' . $osC_Template->getGroup() . '/*", "' . $osC_Template->getGroup() . '/' . substr($osC_Template->getPageContentsFilename(), 0,strrpos($osC_Template->getPageContentsFilename(), '.')) . '"'); // 判断是否在调用页里 出现,这里指定了三种可能性

    $Qspecific->bindValue(':modules_group', $this->_group); // 是模块,还是内容框
    $Qspecific->execute();
    if ($Qspecific->numberOfRows()) {
      while ($Qspecific->next()) {
        $data[$Qspecific->value('boxes_group')][] = $Qspecific->value('code'); // “boxes_group”字段即位置类,“code”字段为识别字
      }
    } else {
      $_data = array(); // 普通加载模式
      $Qmodules = $osC_Database->query('select b2p.boxes_group, b2p.content_page, b.code from :table_templates_boxes_to_pages b2p, :table_templates_boxes b, :table_templates t where b2p.templates_id = :templates_id and b2p.content_page in (:content_page) and b2p.templates_boxes_id = b.id and b.modules_group = :modules_group and b2p.templates_id = t.id order by b2p.boxes_group, b2p.sort_order');
      $Qmodules->bindTable(':table_templates_boxes_to_pages', TABLE_TEMPLATES_BOXES_TO_PAGES);
      $Qmodules->bindTable(':table_templates_boxes', TABLE_TEMPLATES_BOXES);
      $Qmodules->bindTable(':table_templates', TABLE_TEMPLATES);
      $Qmodules->bindInt(':templates_id', $osC_Template->getID());
      $Qmodules->bindRaw(':content_page', '"*", "' . $osC_Template->getGroup() . '/*", "' . $osC_Template->getGroup() . '/' . substr($osC_Template->getPageContentsFilename(), 0, strrpos($osC_Template->getPageContentsFilename(), '.')) . '"');

      $Qmodules->bindValue(':modules_group', $this->_group);
      $Qmodules->execute();
      while ($Qmodules->next()) {
        $_data[$Qmodules->value('boxes_group')][] = array('code' => $Qmodules->value('code'), 'page' => $Qmodules->value('content_page'));// 与独占模式不同,这里多了一个“page”值,其目的用于清除重复数据,见下面的代码
      }

      // 清除重复出现的模块/内容框
      foreach ($_data as $groups => $modules) {
        $clean = array();
        foreach ($modules as $module) {
          if (isset($clean[$module['code']])) {
            if (substr_count($module['page'], '/') > substr_count($clean[$module['code']]['page'], '/')) {
              unset($clean[$module['code']]);
            }
          }
          $clean[$module['code']] = $module;
        }
        $_data[$groups] = $clean;
      }

      foreach ($_data as $groups => $modules) {
        foreach ($modules as $module) {
          $data[$groups][] = $module['code'];
        }
      }
    }

    $osC_Cache->write($data);
  }

  $this->_modules = $data;
}
```

osC_Template 类的 getContentModules 方法 includes/classes/template.php[318]

```php
function getContentModules($group) {
  if (isset($this->osC_Modules_Content) === false) {
    $this->osC_Modules_Content = new osC_Modules('content'); // 指定要加载功能模块
  }
  
  return $this->osC_Modules_Content->getGroup($group); //通过调用osC_Modules类的 getGroup 方法实现其功能,通过指定参数$group 来获得指定位置的模块或内容框
}
```

osC_Modules 类的 getGroup 方法 includes/classes/modules.php[127]

```php
function getGroup($group) {
  $modules = array();
  if (isset($this->_modules[$group])) { // 直接通过内部变量$_modules 数组便可得知是否 存在满足条件的内容
    foreach ($this->_modules[$group] as $module) {
      if (file_exists('includes/modules/' . $this->_group . '/' . $module . '.php')) {
        $class = 'osC_' . ucfirst($this->_group) . '_' . $module; // 类的命名规则

        if (class_exists($class) === false) {
          include('includes/modules/' . $this->_group . '/' . $module . '.php'); // 加载子类,即满足条件的模块/内容框
        }

        $modules[] = $class; // 返回所有类名 }
      }
    }
  }

  return $modules;
}
```

在这里,于主体内容前如何显示功能模块已经介绍完了。至于如何在主体内容后显示功能模块, 和如何显示左边栏以及右边栏,其过程与我们前面所说的大体一致,所以不再详述,它们的不同之处在于:

-  主体内容后的功能模块使用“after”做为位置名称
-  左边栏和右边栏分别使用“left”和“right”做为位置名称
-  加载内容框使用的是$osC_Template->getBoxModules方法,而非getContentModules方法
