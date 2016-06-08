## 动作与响应

### 全局动作

在前面我们讲到每个主文件(index.php、products.php 等)都是从 osC_Template 类的 setup 方法开 始执行模板的初始任务,而全局动作的处理也同样由 osC_Template 类的 setup 方法启动。

osC_Template 类 includes/classes/template.php[199]

```php
if ( isset($_GET['action']) && !empty($_GET['action']) ) {
  include('includes/classes/actions.php');
  osC_Actions::parse($_GET['action']);
}
```

这段代码便是判断是否有执行全局动作的必要,如果设置了$_GET[‘action’]的值,那么就加载 osC_Actions 类,osC_Actions 类是一个简单的类,它只有一个静态方法 parse,作用是将动作转向 到正确的处理类

osC_Actions 类的 parse 方法 includes/classes/actions.php

```php
public static function parse($module) {
  $module = basename($module);
  if ( !empty($module) && file_exists('includes/modules/actions/' . $module . '.php') ) {
    include('includes/modules/actions/' . $module . '.php');
    
    // 所有全局动作类的文 件保存在“includes/modules/actions/”目录,动作的名称为文件名,动作类的名称为“osC_Actions_ ”+动作名称,如添加到购物车的动作名是“cart_add”,那么它的类名即为“osC_Actions_cart_add”,它 的文件名是“cart_add.php”

    call_user_func(array('osC_Actions_' . $module, 'execute')); // 调用动作类的 execute 方法
  }
}
```

我们以添加到购物车为例,看看动作执行的代码

osC_Actions_cart_add 类 includes/modules/acitons/cart_add.php

```php
class osC_Actions_cart_add {
  function execute() {
    global $osC_Session, $osC_ShoppingCart, $osC_Product;
    if ( !isset($osC_Product) ) { // 首先要创建 osC_Product 对象,需要通过传递过来的参数 进行构造
      $id = false;
      foreach ( $_GET as $key => $value ) {
        if ( (is_numeric($key) || ereg('^[a-zA-Z0-9 -_]*$', $key)) && ($key != $osC_Session->getName()) ) {
          $id = $key; // $id可以是产品ID,也可以是产品关键字,因为osCommerce V3支持了新的URL规则 
        }

        break;
      }

      if ( ($id !== false) && osC_Product::checkEntry($id) ) {
        // osC_Product::checkEntry 方法判断$id 是否有效,即是否存在 ID 值等于$id 的产品,或是关键字等于$id 的产品
        $osC_Product = new osC_Product($id); // 构造 osC_Product 对象,该对象包括了指定 产品的所有资料
      }
    }

    if ( isset($osC_Product) ) {
      if ( $osC_Product->hasVariants() ) {
        if ( isset($_POST['variants']) && is_array($_POST['variants']) && !empty($_POST['variants']) ) { // 同时添加产品的选项
          if ( $osC_Product->variantExists($_POST['variants']) ) { // 判断是否存在此产品属性
            $osC_ShoppingCart->add($osC_Product->getProductVariantID($_POST['variants'])); // 加入购物车
          } else {
            osc_redirect(osc_href_link(FILENAME_PRODUCTS, $osC_Product->getKeyword()));
            return false;
          }
        } else {
          osc_redirect(osc_href_link(FILENAME_PRODUCTS, $osC_Product->getKeyword()));
          return false;
        }
      } else {
        $osC_ShoppingCart->add($osC_Product->getID()); // 没有属性选项时的加入购物车 
      }
    }

    osc_redirect(osc_href_link(FILENAME_CHECKOUT)); // 跳转到购物车页面
  }
}
```

### 页面动作

页面动作是由某个页面触发,并且具有明确指向的动作。我们以处理用户登陆为例进行说明

页面动作的触发 templates/default/content/account/login.php[27]

`<form name="login" action="<?php echo osc_href_link(FILENAME_ACCOUNT,
'login=process', 'SSL'); ?>" method="post">`

指明了登陆的信息将发送到“account.php?login=process”进行处理
“account.php?login”URL 的请求会发送到下面的文件,当显示上面的模板内容也要先经过这个文件处理的

osC_Account_Login 类 includes/content/account/login.php[29]

```php
function osC_Account_Login() {
  global $osC_Language, $osC_Services, $osC_Breadcrumb;

  // redirect the customer to a friendly cookie-must-be-enabled page if cookies are disabled (or the session has not started)
  if (osc_empty(session_id())) {
    osc_redirect(osc_href_link(FILENAME_INFO, 'cookie', 'AUTO'));
  }

  $this->_page_title = $osC_Language->get('sign_in_heading');
  if ($osC_Services->isStarted('breadcrumb')) {
    $osC_Breadcrumb->add($osC_Language->get('breadcrumb_sign_in'), osc_href_link(FILENAME_ACCOUNT, $this->_module, 'SSL'));
  }

  if ($_GET[$this->_module] == 'process') { // 与普通显示内容不同的地方在这里,如果设 置了“prcess“那么表示要处理页面动作了
       $this->_process();
  }
}

/* Private methods */
function _process() {
  global $osC_Database, $osC_Session, $osC_Language, $osC_ShoppingCart,
    $osC_MessageStack, $osC_Customer, $osC_NavigationHistory;

  if (osC_Account::checkEntry($_POST['email_address'])) { // osC_Account::checkEntry 检查是否存在此 email 地址
    if (osC_Account::checkPassword($_POST['password'], $_POST['email_address'])) { // 验证 Email 和密码的合法性
      if (SERVICE_SESSION_REGENERATE_ID == '1') {
        $osC_Session->recreate();
      }

      $osC_Customer->setCustomerData(osC_Account::getID($_POST['email_address'])); // 加 载登陆用户的资料
      $Qupdate = $osC_Database->query('update :table_customers set date_last_logon = :date_last_logon, number_of_logons = number_of_logons+1 where customers_id = :customers_id'); // 更新最后登录时间和登录次数 
      $Qupdate->bindTable(':table_customers', TABLE_CUSTOMERS); 
      $Qupdate->bindRaw(':date_last_logon', 'now()'); 
      $Qupdate->bindInt(':customers_id', $osC_Customer->getID()); 
      $Qupdate->execute();
      $osC_ShoppingCart->synchronizeWithDatabase(); // 同步购物车内容
      $osC_NavigationHistory->removeCurrentPage();
      if ($osC_NavigationHistory->hasSnapshot()) {
        $osC_NavigationHistory->redirectToSnapshot(); // 返回登录前的页面
      } else {
        osc_redirect(osc_href_link(FILENAME_DEFAULT, null, 'AUTO')); // 如果没有登录前的页面,则转到首页
      }
    } else {
      $osC_MessageStack->add('login', $osC_Language->get('error_login_no_match'));
    }
  } else {
    $osC_MessageStack->add('login', $osC_Language->get('error_login_no_match'));
  }
}
```                                                   
