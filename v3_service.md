
## 服务

osCommerce V3 新增加了服务的的概念,那么到底什么是服务 Service 呢? 

服务就是对系统提供底层数据支持,并负责它所擅长的领域,而不输出任何内容的模块。 

osCommerce V3 默认拥有的服务有:

- core(osCommerce 核心服务)
- currencies(货币服务)
- language(语言服务)
- session(会话服务)
- breadcrumb(面包屑导航服务)
- category_path(产品 分类路径服务)
- output_compression(压缩输出服务)
- sefu(搜索引擎友好的 URL 服务)
- specials (产品特价服务)
- whos_online(在线统计服务)
- simple_counter(计数器服务)
- reviews(评价 留言服务)
- recently_visited(最近浏览记录服务)
- debug(调试服务)
- banner(广告栏服务)


上图是后台的服务管理截图,由图可以看到,大部分的服务是可卸载的(拥有Install或者Uninstall 图片),很多服务有参数可供设置(拥有 Edit 图片)

### 如何提供服务

服务的加载 includes/application_top.php[94]

```php
// include and start the services
require('includes/classes/services.php');
$osC_Services = new osC_Services();
$osC_Services->startServices();
```

服务的加载在 application_top.php 开始,首先加载 osC_Services 类,初始化$osC_Services 对象, 获得所有可用的服务,然后调用它的 startServices 方法启动所有服务

osC_Services 服务类 includes/classes/services.php[21]

```php
function osC_Services() {
  $this->services = explode(';', MODULE_SERVICES_INSTALLED); // 由数据库值 MODULE_SERVICES_INSTALLED 返回所有已安装的服务 }
}

function startServices() {
  $this->started_services = array(); // 内部变量$started_services 用于保存所有已启动服务

  foreach ($this->services as $service) {
    $this->startService($service); // 循环启动所有服务
  }
}

function startService($service) {
  include('includes/modules/services/' . $service . '.php');

  // 服务的文件保存在“includes/modules/services/”,文件名为“服务名.php”,服务的类名是“osC_Service_服务名”

  if (@call_user_func(array('osC_Services_' . $service, 'start'))) { // 调用服务 类的 start 方法
    $this->started_services[] = $service;
  }
}

function stopService($service) { // stopService方法用于停止服务
  @call_user_func(array('osC_Services_' . $service, 'stop'));
}
                  
```

下面以货币服务类为例来说明一般服务类的工作方式

osC_Services_currencies 类 includes/modules/services/currencies.php

```php
class osC_Services_currencies {
  function start() {
    global $osC_Language, $osC_Currencies;

    include('includes/classes/currencies.php');

    $osC_Currencies = new osC_Currencies(); // 新建货币类的全局对象$osC_Currencies
    if ((isset($_SESSION['currency']) == false) || isset($_GET['currency']) || ((USE_DEFAULT_LANGUAGE_CURRENCY == '1') && ($osC_Currencies->getCode($osC_Language->getCurrencyID()) != $_SESSION['currency']) ) ) {
      if (isset($_GET['currency']) && $osC_Currencies->exists($_GET['currency'])) { // 由$_GET 的参数切换当前币种
        $_SESSION['currency'] = $_GET['currency'];
      } else {
        $_SESSION['currency'] = (USE_DEFAULT_LANGUAGE_CURRENCY == '1') ? $osC_Currencies->getCode($osC_Language->getCurrencyID()) : DEFAULT_CURRENCY; 
        // 或 者使用与语言配套的币种(USE_DEFAULT_LANGUAGE_CURRENCY 是否生效可以在 currencies 服务的参数里 修改),否则使用默认币种
      }

      if ( isset($_SESSION['cartID']) ) { // 重建购物车识别码
        unset($_SESSION['cartID']);
      }
    }

   return true;
  }

  function stop() {
    return true; // 因为没有什么需要处理的,所以直接返回 true }
  }
}
```

### Core服务
因为Core服务是osCommerce V3的核心,它提供了系统所需的最基础数据,所以有必要对其进行说明。

osC_Services_core 类 includes/modules/services/core.php

```php
class osC_Services_core {
  function start() {
    global $osC_Customer, $osC_Tax, $osC_Weight, $osC_ShoppingCart, $osC_NavigationHistory, $osC_Image;

    // 首先要加载必需的基础类
    include('includes/classes/template.php');
    include('includes/classes/modules.php');
    include('includes/classes/category.php');
    include('includes/classes/variants.php');
    include('includes/classes/product.php');
    include('includes/classes/datetime.php');
    include('includes/classes/xml.php');
    include('includes/classes/mail.php');
    include('includes/classes/address.php');
    include('includes/classes/customer.php');

    $osC_Customer = new osC_Customer(); // osC_Customer类负责管理登录用户的资料
    include('includes/classes/tax.php');
    $osC_Tax = new osC_Tax(); // osC_Tax类负责管理税率换算
    include('includes/classes/weight.php');
    $osC_Weight = new osC_Weight(); // os_Weight类负责管理重量单位换算
    include('includes/classes/shopping_cart.php');
    $osC_ShoppingCart = new osC_ShoppingCart(); // osC_ShoppingCart类负责管理购物车 
    $osC_ShoppingCart->refresh();
    
    include('includes/classes/navigation_history.php');
    $osC_NavigationHistory = new osC_NavigationHistory(true); // osC_NavigationHistory 类负责管理浏览历史记录

    include('includes/classes/image.php');
    $osC_Image = new osC_Image(); // osC_Image类负责管理产品图片
    
    return true;
  }

  function stop() { // 关闭时同样不需要做什么处理
    return true;
  }
}
```

> 提示:
> 
> 通过上面的两个服务类的代码,不难看出,服务类主要是提供基础数据初始化,并不直接进行数据运算。也就是说服务类只是控制其它功能类的生成,通常是实例化功能类成全局变量,具体的功能然后再由全局变量来完成。所以服务类可以说是功能类的统筹、规划者

