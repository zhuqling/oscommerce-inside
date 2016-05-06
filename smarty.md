## Smarty

Smarty 是 PHP 里使用最广泛的模板系统,令它脱颖而出的原因在于它强大的代码界面分离功能。

Smarty 在处理界面时使用了一套自己的语法,使得美工等界面开发人员可以轻松修改前台模板,从而平衡了程序人员的开发量,同时保证了界面的美观与代码的易维护。

### Smarty简介

首先让我们了解一下 Smarty 使用方法。下面通过一个例子开始。

```php
require_once 'includes/smarty/smarty.class.php’; // 我们假设 Smarty 库存放在 ”includes/smarty/”目录下

$smarty = new Smarty(); // 初始化 Smarty 对象
$smarty->template_dir = 'includes/template’; // 设置模块文件所在目录 
$smarty->compile_dir =’includes/template_c/’; // 设置模块编译文件所在目录 
$smarty->cache_dir = 'includes/cache_dir/’; // 设置缓存文件所在目录 
$smarty->config_dir = 'includes/config_dir/’; // 设置配置文件所在目录 
$smarty->plugins_dir = array('includes/plugins/’,'plugins'); // 设置插件文件所在目录 
//上面的五个目录设置,最上面两个为必填,且目录必须真实存在,其它三个目录为可选项可以省略 $user_name = 'Jason Chuh’;

// 赋值到模板
$smarty->assign('name',$user_name); // 文本类型
$attribute = array('age’=>20,’ gender’ =>’male’); $smarty->assign('attribute', $attribute); // 数组也可以传递
$person = new Obj_Person(); // 此处假设 Obj_Person 类已存在 $person.name = 'Jason’;
$person.age = '20’;
$smarty->assign('person', $attribute); // 传递对象都没问题

// 显示模板文件
$smarty->display('template_test.tpl’); // 文件名的后缀可以随意设置,Smarty 默认使用“tpl ”作为模板文件的扩展名
```

模块文件 includes/template/template_test.tpl

```html
<h1>显示文本</h1>
Name: {$name}
{* 这里是注释 *}
{* Smarty 默认使用{}作为变量的特征符,如果想改变特征符,可以通过设置 Smarty 的成员 left_delimiter 左特征符和 right_delimiter 右特征符进行修改。变量以$美元符开头,与 PHP 一致 *} <h1>显示数组</h1>
Gender: {$attribute.gender},Age: {$attribute.age}
{* 关联数组可以直接用“.”引用,如果为普通数组则用“[]”加索引号来引用 *}
<h1>显示对象</h1>
Name: {$person->name},Age: {$person->age} {* 对象使用“->”的方式引用 *}
```

### 如何改造成Smarty模板

#### Smarty的初始化

Smarty 的初始化需要在每个单独的文件执行前执行,所以 Smarty 的初始化理所当然地应该放在 application_top.php 里。

预处理文件 includes/application_top.php

```php
require_once 'includes/smarty/smarty.class.php’;
$smarty = new Smarty();
$smarty->template_dir = 'includes/template/’;
$smarty->compile_dir =’includes/template_c/’;
$smarty->cache_dir = 'includes/cache_dir/’;
$smarty->config_dir = 'includes/config_dir/’;
$smarty->plugins_dir = array('includes/plugins/’,'plugins');
```

#### Smarty的赋值

由于每个页面的内容都会不同,所以 Smarty 的赋值任务应该放在各自页面进行,这也就意味 着需要对所有页面进行修改。所以如果需要对 osCommerce 进行完全的 Smarty 改造,就必须对所 有页面进行重新编码,并且对所有页面(前台有 40 多个页面)编写单独的模板

我们要讨论的是基于一般页面所拥有通用显示元素,也就是通常页面都会显示的内容。我们打算将这些内容封装为函数,便于我们的调用。

#### 界面的显示

由于模板的显示是基于 Smarty 的赋值过后,所以我们将模板显示的任务安排进上面提及的封装函 数内。这样可以使流程更加顺畅

### 改造实例

我们以 product_info.php 实际页面的改造为例,来介绍改造 osCommerce 使用 Smarty 作为其模板 系统的方法。

1) Smarty 通用赋值及显示函数

通用赋值及显示函数 includes/functions/general.php

```php
function smarty_display_tpl($smarty, $filename) {
  global $cart,$breadcrumb,$messageStack, $currencies,$currency,$language, $languages_id,$PHP_SELF,$customer_first_name;
  global $HTTP_GET_VARS

  // 赋值通用变量
  $smarty->assign('htmlparams’, HTML_PARAMS);
  $smarty->assign('date', strftime(DATE_FORMAT_LONG)); 
  $smarty->assign('langid', $languages_id);
  $smarty->assign('sid'], tep_session_name() . '=' . tep_session_id());
  $smarty->assign('cataloglogo'], '<a href="' . tep_href_link(FILENAME_DEFAULT) . '">' . tep_image(STS_TEMPLATE_DIR.'images/'.$language . '/header_logo.gif', STORE_NAME) . '</a>'); // 网站 Logo
  $smarty->assign('urlcataloglogo', tep_href_link(FILENAME_DEFAULT));// 网站首页
  $smarty->assign('cartlogo', '<a href="' . tep_href_link(FILENAME_SHOPPING_CART, '', 'SSL') . '">' . tep_image(TEMPLATE_DIR.'images/'.$language. '/header_cart.gif', HEADER_TITLE_CART_CONTENTS) . '</a>'); // 购物车图标链接
  $smarty->assign('myaccountlogo', '<a href=' . tep_href_link(FILENAME_ACCOUNT, '', 'SSL') . ' class="headerNavigation">' . tep_image(TEMPLATE_DIR.'images/'.$language . '/header_account.gif', HEADER_TITLE_MY_ACCOUNT) . '</a>'); // 用户后台图标链接 
  $smarty->assign('checkoutlogo', '<a href="' . tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL') . '">' . tep_image(TEMPLATE_DIR.'images/'.$language.'/header_checkout.gif', HEADER_TITLE_CHECKOUT) . '</a>'); // 付款图标链接
  $smarty->assign('breadcrumbs',$breadcrumb->trail(BREADCRUMB_SEPARATOR)); // 面包屑 导航的显示
  $title=$breadcrumb->Currents(); 
  $smarty->assign('title', $title); // 标题

  if (tep_session_is_registered('customer_id')) {
    // 区分用户登录情况,显示不同的内容
    $smarty->assign('myaccount’, '<a href=' . tep_href_link(FILENAME_ACCOUNT, '', 'SSL') . ' class="headerNavigation">' . HEADER_TITLE_MY_ACCOUNT . '</a>');
    $smarty->assign('urlmyaccount’, tep_href_link(FILENAME_ACCOUNT, '', 'SSL')); $smarty->assign('logoff','<a href=' . tep_href_link(FILENAME_LOGOFF, '', 'SSL') . ' class="headerNavigation">' . HEADER_TITLE_LOGOFF . '</a>'); 
    $smarty->assign('urllogoff', tep_href_link(FILENAME_LOGOFF, '', 'SSL'));
    $smarty->assign('urlcartcontents’, tep_href_link(FILENAME_SHOPPING_CART, '', 'SSL'));
    $smarty->assign('myaccountlogoff', $smarty->get_template_vars('myaccount') . " | " . $smarty->get_template_vars('logoff'));
    $smarty->assign('loginofflogo', '<a href=' . tep_href_link(FILENAME_LOGOFF, '', 'SSL') . ' class="headerNavigation">' . tep_image(TEMPLATE_DIR.'images/'.$language . '/header_logoff.gif', HEADER_TITLE_LOGOFF) . '</a>'); // TEMPLATE_DIR 为模板目录
  } else {
    $smarty->assign('myaccount’, '<a href=' . tep_href_link(FILENAME_ACCOUNT, '', 'SSL') . ' class="headerNavigation">' . HEADER_TITLE_MY_ACCOUNT . '</a>'); $smarty->assign('urlmyaccount’, tep_href_link(FILENAME_ACCOUNT, '', 'SSL')); $smarty->assign('logoff','');
    $smarty->assign('urllogoff', '');
    $smarty->assign('urlcartcontents’, tep_href_link(FILENAME_SHOPPING_CART, '',
    'SSL'));
    $smarty->assign('urlcheckout’, tep_href_link(FILENAME_CHECKOUT_SHIPPING, '',
    'SSL'));
    $smarty->assign('myaccountlogoff', $smarty->get_template_vars('myaccount'));
    $smarty->assign('loginofflogo', '<a href=' . tep_href_link(FILENAME_ LOGIN, '', 'SSL') . ' class="headerNavigation">' . tep_image(TEMPLATE_DIR.'images/'.$language . '/header_ login.gif', HEADER_TITLE_ LOGIN) . '</a>');
  }

  $smarty->assign('cartcontents’, $cart->trail());// 购物车的显示
  $smarty->assign ('urlcartcontents',tep_href_link(FILENAME_SHOPPING_CART, '', 'SSL'));// 购物车链接
  $smarty->assign('checkout’, '<a href=' . tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL') . ' class="headerNavigation">' . HEADER_TITLE_CHECKOUT . '</a>');//付款 链接
  $smarty->assign('urlcheckout’, tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL'));//付款 URL
  $smarty->assign('contactlogo', '<a href=' . tep_href_link(FILENAME_CONTACT_US) . ' class="headerNavigation">' . tep_image(TEMPLATE_DIR.'images/'.$language . '/header_contact_us.gif', BOX_INFORMATION_CONTACT) . '</a>'); // 联系图标链接

  // 统计
  require(DIR_WS_INCLUDES . 'counter.php');
  $smarty->assign('numrequests', $counter_now . ' ' . FOOTER_TEXT_REQUESTS_SINCE . ' ' . $counter_startdate_formatted); // 访问统计
  $smarty->assign('footer_text', FOOTER_TEXT_BODY); // 页脚
  $smarty->assign('currency',$currency); // 当前币种 
  $smarty->assign('language',$language); // 当前语言 
  $smarty->assign('languages_id',$languages_id); // 语言 ID 
  $smarty->assign('php_self',$PHP_SELF); // 当前文件

  ob_start();
  echo "\n<!-- Start Category Menu -->\n";
  echo tep_draw_form('goto', FILENAME_DEFAULT, 'get', '');
  echo tep_draw_pull_down_menu('cPath', tep_get_category_tree(), $current_category_id, 'onChange="this.form.submit();"');
  echo "</form>\n";
  echo "<!-- End Category Menu -->\n";
  $smarty->assign('catmenu’, ob_get_clean()); // 产品分类菜单

  ob_start();
  if ((USE_CACHE == 'true') && empty($SID)) {
    echo tep_cache_categories_box();
  } else {
    include(DIR_WS_BOXES . 'categories.php');
  }
  $smarty->assign('categorybox’,ob_get_clean()); //产品目录框

  // 制造商框
  ob_start();
  if ((USE_CACHE == 'true') && empty($SID)) {
    echo tep_cache_manufacturers_box();
  } else {
    include(DIR_WS_BOXES . 'manufacturers.php');
  }
  $smarty->assign('manufacturerbox',ob_get_clean()); // 新产品框

  ob_start();
  require(DIR_WS_BOXES . 'whats_new.php'); $smarty->assign('whatsnewbox',ob_get_clean());

  // 通知好友框
  ob_start();
  require(DIR_WS_BOXES . 'tell_a_friend.php'); $smarty->assign('tellafriendbox', ob_get_clean());

  // 搜索框
  ob_start();
  require(DIR_WS_BOXES . 'search.php'); $smarty->assign('searchbox', ob_get_clean());

  // 信息框
  ob_start();
  require(DIR_WS_BOXES . 'information.php'); $smarty->assign('informationbox', ob_get_clean());

  // 购物车框
  ob_start();
  require(DIR_WS_BOXES . 'shopping_cart.php'); $smarty->assign('cartbox', ob_get_clean());

  // 畅销产品框
  ob_start();
  include(DIR_WS_BOXES . 'best_sellers.php'); $smarty->assign('bestsellersbox_only', ob_get_clean());

  if (isset($HTTP_GET_VARS['products_id'])) {
    ob_start();
    include(DIR_WS_BOXES . 'manufacturer_info.php'); $smarty->assign('maninfobox', ob_get_clean()); // 产品相关的制造商信息
    
    ob_start();
    include(DIR_WS_BOXES . 'order_history.php'); $smarty->assign('orderhistorybox', ob_get_clean()); // 订单历史记录框

    ob_start();
    include(DIR_WS_BOXES . 'product_notifications.php');
    $smarty->assign('notificationbox', ob_get_clean()); // 产品订阅框

    if (tep_session_is_registered('customer_id')) {
      $check_query = tep_db_query("select count(*) as count from " . TABLE_CUSTOMERS_INFO . " where customers_info_id = '" . (int)$customer_id . "' and global_product_notifications = '1'");
      $check = tep_db_fetch_array($check_query);
      if ($check['count'] > 0) {
        $smarty->assign('bestsellersbox', $smarty->get_template_vars('bestsellersbox_only'));
      } else {
        $smarty->assign('bestsellersbox', $smarty->get_template_vars('notificationbox')]);
      }
    } else {
      $smarty->assign('bestsellersbox', $smarty->get_template_vars('notificationbox')); //
    }
  } else {
    $smarty->assign('bestsellersbox', $smarty->get_template_vars('bestsellersbox_only'));
    $smarty->assign('notificationbox', '');
  }

  ob_start();
  include(DIR_WS_BOXES . 'specials.php'); $smarty->assign('specialbox', ob_get_clean()); // 特价产品框

  $smarty->assign('specialfriendbox']= $smarty->get_template_vars('specialbox'));

  if (isset($HTTP_GET_VARS['products_id']) && basename($PHP_SELF) != FILENAME_TELL_A_FRIEND) {
    ob_start();
    include(DIR_WS_BOXES . 'tell_a_friend.php');
    $smarty->assign('tellafriendbox', ob_get_clean()); // Get tell a friend box
    
    $smarty->assign('specialfriendbox', $smarty->get_template_vars('tellafriendbox')); // Shows specials or tell a friend
  } else
    $smarty->assign('tellafriendbox'], '');
  
  // Get languages and currencies boxes, empty if in checkout procedure
  if (substr(basename($PHP_SELF), 0, 8) != 'checkout') {
    ob_start();
    include(DIR_WS_BOXES . 'languages.php');
    $smarty->assign('languagebox', ob_get_clean()); // 语言选择框

    ob_start();
    include(DIR_WS_BOXES . 'currencies.php');
    $smarty->assign('currenciesbox', ob_get_clean()); // 币种选择框
  } else {
    $smarty->assign('languagebox'], '');
    $smarty->assign('currenciesbox'], '');
  }

  if (basename($PHP_SELF) != FILENAME_PRODUCT_REVIEWS_INFO) {
    ob_start();
    require(DIR_WS_BOXES . 'reviews.php'); $smarty->assign('reviewsbox', ob_get_clean()); // 评论框
  } else {
    $smarty->assign('reviewsbox', ='');
  }

  $smarty->assign('IMAGE_GIF_PIXEL_100_PERCENT’,tep_draw_separator('pixel_trans.gif', '100%', '10'));
  $smarty->assign('IMAGE_GIF_PIXEL_10_PIXEL’,tep_draw_separator('pixel_trans.gif','10', '1'));

  // 允许调用时省略模板的扩展名
  if(false === strpos($filename,'.')) $filename .= '.tpl'; $smarty->display($filename); // 显示模板文件
}
```

2) 产品详细页 product_info.php

```php
<?php
require_once 'includes/application_top.php';
require(DIR_WS_LANGUAGES . $language . '/' . FILENAME_PRODUCT_INFO);

// 获得产品信息
$product_info_query = tep_db_query("select p.products_id, pd.products_name, pd.products_description, p.products_model, p.products_quantity, p.products_image, pd.products_url, p.products_price, p.products_tax_class_id, p.products_date_added, p.products_date_available, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and pd.products_id = p.products_id and pd.language_id = '" . (int)$languages_id . "'");
$product_info = tep_db_fetch_array($product_info_query);

if(false == $product_info) { // 找不到产品
  $smarty->assign('product_info' , false);
  $smarty->assign('BOX_PRODUCT_NOT_FOUND', new infoBox(array(array('text' => TEXT_PRODUCT_NOT_FOUND))));
  $smarty->assign('url_continue’, '<a href="' . tep_href_link(FILENAME_DEFAULT) . '">' . tep_image_button('button_continue.gif', IMAGE_BUTTON_CONTINUE) . '</a>');
} else {
  tep_db_query("update " . TABLE_PRODUCTS_DESCRIPTION . " set products_viewed = products_viewed+1 where products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and language_id = '" . (int)$languages_id . "'"); // 刷新产品浏览次数
  if ($new_price = tep_get_products_special_price($product_info['products_id'])) {
    $products_price = '<s>' . $currencies->display_price($product_info['products_price'], tep_get_tax_rate($product_info['products_tax_class_id'])) . '</s> <span class="productSpecialPrice">' . $currencies->display_price($new_price, tep_get_tax_rate($product_info['products_tax_class_id'])) . '</span>';
  } else {
    $products_price = $currencies->display_price($product_info['products_price'], tep_get_tax_rate($product_info['products_tax_class_id']));
  }

  if (tep_not_null($product_info['products_model'])) {
    $products_name = $product_info['products_name'] . '<br><span class="smallText">[' . $product_info['products_model'] . ']</span>';
  } else {
    $products_name = $product_info['products_name'];
  }

  $smarty->assign('products_name' , $products_name);
  $smarty->assign('products_price' , $products_price);
  $smarty->assign('product_info' , $product_info); // 购买推荐

ob_start();
if ((USE_CACHE == 'true') && empty($SID)) {
  echo tep_cache_also_purchased(3600);
} else {
  include(DIR_WS_MODULES . FILENAME_ALSO_PURCHASED_PRODUCTS);
}

$smarty->assign('also_purchase’,ob_get_clean());
$smarty->assign('url_product_review’,'<a href="' . tep_href_link(FILENAME_PRODUCT_REVIEWS, tep_get_all_get_params()) . '">' . tep_image_button('button_reviews.gif', IMAGE_BUTTON_REVIEWS) . '</a>'); $smarty->assign('button_submit’, tep_draw_hidden_field('products_id', $product_info['products_id']) . tep_image_submit('button_in_cart.gif', IMAGE_BUTTON_IN_CART));

// 产品编辑时间
if ($product_info['products_date_available'] > date('Y-m-d H:i:s')) {
  $smarty->assign('product_date’, sprintf(TEXT_DATE_AVAILABLE,
  tep_date_long($product_info['products_date_available'])));
} else {
  $smarty->assign('product_date’, sprintf(TEXT_DATE_ADDED,
  tep_date_long($product_info['products_date_added'])));
}
$smarty->assign('url_product_url’, sprintf(TEXT_MORE_INFORMATION, tep_href_link(FILENAME_REDIRECT, 'action=url&goto=' . urlencode($product_info['products_url']), 'NONSSL', true, false))); // 产品留言

$reviews_query = tep_db_query("select count(*) as count from " . TABLE_REVIEWS . " where products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "'");
$reviews = tep_db_fetch_array($reviews_query);
if ($reviews['count'] > 0) {
  $smarty->assign('review_text’, TEXT_CURRENT_REVIEWS . ' ' . $reviews['count']);
} else {
  $smarty->assign('review_text’,’’);
}

// 产品图片
$smarty->assign('url_product_image’, tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' . $product_info['products_id']));
$smarty->assign('product_image’, tep_image(DIR_WS_IMAGES . $product_info['products_image'], addslashes($product_info['products_name']), SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"'));
$smarty->assign('TEXT_CLICK_TO_ENLARGE’, TEXT_CLICK_TO_ENLARGE);

// 产品属性
$products_attributes_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_ATTRIBUTES . " patrib where patrib.products_id='" . (int)$HTTP_GET_VARS['products_id'] . "' and patrib.options_id = popt.products_options_id and popt.language_id = '" . (int)$languages_id . "'");
$products_attributes = tep_db_fetch_array($products_attributes_query);
if ($products_attributes['total'] > 0) {
  $ProductAttributesArray = array();
  $products_options_name_query = tep_db_query("select distinct popt.products_options_id, popt.products_options_name from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_ATTRIBUTES . " patrib where patrib.products_id='" . (int)$HTTP_GET_VARS['products_id'] . "' and patrib.options_id = popt.products_options_id and popt.language_id = '" . (int)$languages_id . "' order by popt.products_options_name");
  
  while ($products_options_name = tep_db_fetch_array($products_options_name_query)) {
    $products_options_query = tep_db_query("select pov.products_options_values_id, pov.products_options_values_name, pa.options_values_price, pa.price_prefix from " . TABLE_PRODUCTS_ATTRIBUTES . " pa, " . TABLE_PRODUCTS_OPTIONS_VALUES . " pov where pa.products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and pa.options_id = '" . (int)$products_options_name['products_options_id'] . "' and pa.options_values_id = pov.products_options_values_id and pov.language_id = '" . (int)$languages_id . "'");
      while ($products_options = tep_db_fetch_array($products_options_query)) {
        $tmpOptionValuesArray = array();
        $displayText = $products_options['products_options_values_name'];

        if ($products_options['options_values_price'] != '0') {
          $displayText .= ' (' . $products_options['price_prefix'] . $currencies->display_price($products_options['options_values_price'], tep_get_tax_rate($product_info['products_tax_class_id'])) .') ';
        }

        // 当前选择的属性值
        $optionValueSelected = false;
        if(isset($cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']])) {
          $selected_attribute = $cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']];

          if($selected_attribute == $products_options['products_options_values_id']] ) $optionValueSelected = true;
          $tmpOptionValuesArray[] = array(
            'id' => $products_options['products_options_values_id'], 
            'text' =>$displayText, 
            'selected' => $optionValueSelected
          );
        }

        $ProductAttributesArray[] = array(
          'id' => $products_options['products_options_id], 
          'name' => $products_options_name['products_options_name’], 
          'values' => $tmpOptionValuesArray
        );
      }

      $smarty->assign('attributes', $ProductAttributesArray);
      $smarty->assign('TEXT_PRODUCT_OPTIONS', TEXT_PRODUCT_OPTIONS);
    }

    // 表单提交地址
    $smarty->assign('$form_add_product',tep_href_link(FILENAME_PRODUCT_INFO, tep_get_all_get_params(array('action')) . 'action=add_product'));
  }

  smarty_display_tpl('product_info');
  require_once 'includes/application_bottom.php';
?>
```

3) 产品页模板 includes/template/product_info.tpl

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html {$htmlparams}>
<head>
  <title>{$title}</title>
<link rel="stylesheet" type="text/css" href="stylesheet.css">
</head>
<body marginwidth="0" marginheight="0" topmargin="0" bottommargin="0" leftmargin="0" rightmargin="0">
<table border="0" width="100%" cellspacing="0" cellpadding="10">
   <tr class="header">
       <td valign="middle">
          <img border="0" src="images/store_logo.png"></td>
       <td align=right>
          <a href="{$urlmyaccount}">
          <img border="0" src="images/header_account.gif"></a>
          <a href="{$urlcartcontents}">
          <img border="0" src="images/header_cart.gif"></a>
          <a href="{$urlcheckout}">
          <img border="0" src="images/header_checkout.gif"></a>
</td> </tr>
</table>
<table border="0" width="100%" cellspacing="0" cellpadding="1">
    <tr class="headerNavigation">
       <td class="headerNavigation">
          &nbsp;&nbsp;
          {$breadcrumbs}
       </td>
       <td align="right" class="headerNavigation">
          {$myaccountlogoff} | {$cartcontents} | {$checkout}
          &nbsp;&nbsp;
</td> </tr>
</table>
<table border="0" width="100%" cellspacing="3" cellpadding="3">
  <tr>
   <td valign="top" width="125">
     <table border="0" width="125" cellspacing="0" cellpadding="2">
       <tr><td>$catmenu</td></tr>
       <tr><td>$categorybox</td></tr>
       <tr><td>$manufacturerbox</td></tr>
       <tr><td>$whatsnewbox</td></tr>
       <tr><td>$specialbox</td></tr>
       <tr><td>$searchbox</td></tr>
       <tr><td>$informationbox</td></tr>
     </table>
   </td>
   <td valign="top">
   <form name="cart_quantity" action="{$form_add_product}"><table border="0"
width="100%" cellspacing="0" cellpadding="0">
{if !$product_info}
     <tr>
       <td>{$BOX_PRODUCT_NOT_FOUND}</td>
</tr>
     <tr>
        <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
     <tr>
       <td><table border="0" width="100%" cellspacing="1" cellpadding="2"
class="infoBox">
        <tr class="infoBoxContents">
          <td><table border="0" width="100%" cellspacing="0" cellpadding="2">
            <tr>
             <td width="10">{$IMAGE_GIF_PIXEL_10_PIXEL}</td>
             <td align="right">{$url_continue}</td>
             <td width="10">{$IMAGE_GIF_PIXEL_10_PIXEL}</td>
            </tr>
          </table></td>
        </tr>
       </table></td>
</tr> {else}
     <tr>
       <td><table border="0" width="100%" cellspacing="0" cellpadding="0">
<tr>
<td class="pageHeading" valign="top">{$products_name}</td>
<td class="pageHeading" align="right" valign="top">{$products_price}</td>
        </tr>
       </table></td>
</tr> <tr>
       <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
     <tr>
       <td class="main">
        {if '' != $product_info.products_image}
        <table border="0" cellspacing="0" cellpadding="2" align="right">
<tr>
     <td align="center" class="smallText">
<script language="javascript"><!--
document.write('<a
href="javascript:popupWindow({$url_product_image})">{$product_image}<br>{$TEXT_CL
ICK_TO_ENLARGE}</a>');
//--></script>
<noscript>
<a href="{$url_product_image}"
target="_blank">{$product_image}<br>{$TEXT_CLICK_TO_ENLARGE}</a>
</noscript>
</td> </tr>
        </table>
        {/if}
        <p>{$product_info.products_description|escape}</p>
        {if !attributes && count($attributes)>0}
        <table border="0" cellspacing="0" cellpadding="2">
          <tr>
            <td class="main" colspan="2">{$TEXT_PRODUCT_OPTIONS}</td>
          </tr>
            {foreach from=$attributes item=option}
          <tr>
            <td class="main">{$option.name}</td>
            <td class="main">
                <select name="id[{$option.id}]">
                   {foreach from=$option.values item=option_value}<option
value="{$option_value.id}" {if
$option_value.selected}selected="selected"{/if}>{$option_value.text}</option>{/foreach}
                </select>
            </td>
          </tr>
            {/foreach}
              </table>
         {/if}
</td> </tr>
     <tr>
       <td><{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
     {if '' != $review_text}
     <tr>
       <td class="main">{$review_text}</td>
     </tr>
     <tr>
       <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
     {/if}
     {if '' != $product_info.products_url}
     <tr>
       <td class="main">{$url_product_url}</td>
     </tr>
     <tr>
       <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
     {/if}
     <tr>
       <td align="center" class="smallText">{$product_date}</td>
     </tr>
     <tr>
       <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
</tr> <tr>
       <td><table border="0" width="100%" cellspacing="1" cellpadding="2"
class="infoBox">
        <tr class="infoBoxContents">
                <td><table border="0" width="100%" cellspacing="0" cellpadding="2">
            <tr>
             <td width="10">{$IMAGE_GIF_PIXEL_10_PIXEL}</td>
             <td class="main">{$url_product_review}</td>
             <td class="main" align="right">{$button_submit}></td>
             <td width="10">{$IMAGE_GIF_PIXEL_10_PIXEL}</td>
            </tr>
          </table></td>
        </tr>
       </table></td>
</tr> <tr>
       <td>{$IMAGE_GIF_PIXEL_100_PERCENT}</td>
     </tr>
<tr> <td>
       {$also_purchase}
{/if}
</td> </tr>
</table></form>
</td>
   <td valign="top" width="125">
     <table border="0" width="125" cellspacing="0" cellpadding="2">
       <tr><td>$cartbox</td></tr>
       <tr><td>$maninfobox</td></tr>
       <tr><td>$orderhistorybox</td></tr>
       <tr><td>$bestsellersbox</td></tr>
       <tr><td>$reviewsbox</td></tr>
       <tr><td>$tellafriendbox</td></tr>
       <tr><td>$languagebox</td></tr>
       <tr><td>$currenciesbox</td></tr>
</table>
      </td> </tr>
</table>
<table border="0" width="100%" cellspacing="0" cellpadding="1">
  <tr class="footer">
   <td class="footer">&nbsp;&nbsp;$date&nbsp;&nbsp;</td>
   <td align="right" class="footer">&nbsp;&nbsp;$numrequests&nbsp;&nbsp;</td>
  </tr>
</table>
<br>
<table border="0" width="100%" cellspacing="0" cellpadding="0">
  <tr>
   <td align="center" class="smallText">
     $footer_text
   </td>
  </tr>
</table>
</body>
</html>
```                                           

### Smarty Plugin

体现 Smarty 强大功能的另一个特点在于它的插件 Plugin,Smarty 本身已经内置有不少的插 件,为模板的制作提供了最基础的支持,如果用户需要特殊的功能,并且希望可以直接在模板文 件里执行时,那么就需要自己编写插件了。

#### Smarty 插件介绍

Smarty 插件的种类很多种,它们是:function、modifier、block、compiler、prefilter、postfilter、
outputfilter、resource、insert。

其中function和modifier使用最多,function即为函数,内置的Smarty 插件里 mailto、math、fetch 都属于 function 一类插件,而如 date_format、truncate、escape、strip_tags 都属于 modifier 修改器一类的插件。下面的表格列出了 function 与 modifier 在使用上以及编写等 方面的不同之处。

<table>
  <thead>
    <tr>
      <th>  </th>
      <th> function </th>
      <th> function </th>
      <th> modifier </th>
      <th> modifier </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>  </td>
      <td> 示例 </td>
      <td> 说明 </td>
      <td> 示例 </td>
      <td> 说明 </td>
    </tr>
    <tr>
      <td> 调用方式 </td>
      <td> {mailto address="me@exampl e.com" text="send me some mail"} </td>
      <td> 像调用函数一样调 用,参数以变量形式 指定 </td>
      <td> {$articleTitle</td>
      <td> escape:"html" } | 在参数后以“|” 开始附加修改 器的名称,参数 以“:”分隔 </td>
    </tr>
    <tr>
      <td> 文件名规则 </td>
      <td> function.mailto.php </td>
      <td> function.函数名称.php </td>
      <td> modifier.esca pe.php </td>
      <td> modifier.修改器 名称.php </td>
    </tr>
    <tr>
      <td> 插件编写定义规则 </td>
      <td> function smarty_function_mailt o($params, &amp;$smarty) </td>
      <td> <div>名称为 smarty_function_ 函 数 名,参数有两个,第 一个为传入的参数数 组(关联数组),第二 个参数为 smarty 对 象,smarty 对象以引 用方式传入(理论上只 有以引入方式转入 smarty 对象,才可以 通过函数修改 smarty的内容,但在实现的 过程中,笔者发现以 传值的形式传入也是 没有问题的,不过还 是强烈建议使用引用 方式,以实现编码的 规范)。 </td>
      <td> function smarty_modif ier_escape($st ring, $esc_type = 'html', $char_set = 'ISO-8859-1') </td>
      <td> 名称规则为：smarty_modifier _修改器名,参 数的个数由用 户指定,但第一 个 参 数 $string 为必选值 ($string 的值 为需要修改的 内容),其它参 数按传入的顺序导入,参数名 称是用户指定 的。</div> </td>
    </tr>
    <tr>
      <td> 结果的返回 </td>
      <td> <code><pre>if (!empty($params['assign'])) {
  $smarty->assign($para ms['assign'], $_contents);
} else {
  return $_contents;
}
</pre></code></td>
      <td> 通常有两种形式返回 结果(确定没是必要 的话,可以只使用一 种形式返回结果),当指定了参数 assign 时, 会将结果填入 assign 变量,否则直接使用 return 返回 </td>
      <td> return str_replace($s earch, $replace, $string); </td>
      <td> 只需要使用 return 形式返回 结果即可 </td>
    </tr>
  </tbody>
</table>

#### 编写 smarty 插件

考虑到 modifier 的编写比较简单,所以笔者将以编写 function 作为本节的示例。 

本例的目的是实现产品的价格显示,价格能够按照当前币种自动切换显示格式,并且能够兼顾税 率、数量等因素。

Smarty Plugin 文件 includes/plugins/function.priceFormat.php

```php
function smarty_function_priceFormat($param,$smarty)
{
  global $currencies;
  if(isset($param['value'])) {
    $tax = 0;
    if(isset($param['tax'])) $tax = $param['tax']; // 税率以参数 tax 传入 $quantity = 1;
    if(isset($param['quantity'])) $quantity = $param['quantity'];// 数量以参数 quantity 传入
    $price = $currencies->display_price($param['value'],$tax,$quantity);//产 品价格以参数 value 传入
    if(isset($param['assign']))
      $smarty->assign($param['assign'],$price);
    else
      return $price;
  }
}
```

在一个模板文件里,使用以下方式便可调用上面的插件了:

```smarty
{priceFormat value=”99.99”}
{priceFormat value=”99.99” tax=”3”}
{priceFormat value=”99.99” quantity=9}
```

> 注意:
> 
> 插件文件必须放置在正确的目录才会被执行,默认的插件目录为 smarty 下的 plugins 目录,当指 定了专用目录时,则必须放入指定目录。

> 提示:
> 
> 上面的自定义插件,要求用户转入产品价格,然后对价格进行格式输出。那么如果传入的是产品 ID,要求通过产品 ID 查找其价格,最终实现价格的格式化,又将怎么实现呢?请读者自行思考, 然后通过实现来真正理解 smarty 插件的编写方法。
