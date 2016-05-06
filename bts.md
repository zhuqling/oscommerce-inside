## BTS

### BTS特点

BTS全称Basic Template Structure(简单模板框架),是osCommerce系统里使用最广泛的模 板插件之一,它与 STS 一样都是用于扩展 osCommerce 以支持模板管理的工具,而与 STS 不同的 是,BTS 实现了 osCommerce 逻辑代码与界面显示的“半分离”,相对于完全的逻辑与界面分离, “半分离”状态使得 osCommerce RC 2.2 编码可以更加灵活与自如,模板与程序的切换反而更流 畅了,并且冗余代码也有所减少,

所以笔者认为 BTS 插件是 osCommerce RC 2.2 系统下最理想的 模板系统,相对于 STS 更加好用


> 提示:
> 
> 笔者认为 BTS 插件是 osCommerce RC 2.2 系统下最理想的模板系统,是基于 osCommerce RC 2.2 更多的采用流程模块式编程,而非对象化编程所下的定义。因为 BTS 可以充分利用了 osCommerce 众多的现有模块,从而使得用户可以以极少的代码来实现模板的显示。

以下介绍 BTS 系统里几个重要的文件夹:

-  includes/boxes/ 这个目录用于存放所有内容框文件,因为 BTS 修改过所有的内容框所以有必
要将此目录进行说明
-  includes/javascript/ 用于存放 javascript 代码文件的目录
-  templates/TEMPLATE/content/ 用于存放每个程序文件对应主体内容的模板,“TEMPLATE”
为模板名
-  templates/TEMPLATE/boxes/ 用于存放内容框模板
-  templates/TEMPLATE/images/ 用于存放对应于此模板的图片文件
-  templates/TEMPLATE/stylesheets/ 用于存放此模板拥有的 CSS 样式表

### 安装BTS

安装前提:未经过任何修改的osCommerce RC2.2,可以通过复制BTS安装目录里的文件到网站 根目录“catalog/”下,替换原有文件即可。如果对 osCommerce 作过变动的话,就需要对所有需 要更新的文件单独进行修改

插件地址:http://www.oscommerce.com/community/contributions,1263

1、执行 BTS.sql 文件,或执行以下 SQL 语句

```sql
INSERT INTO configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) VALUES ('Default Template Directory', 'DIR_WS_TEMPLATES_DEFAULT', 'help', 'Subdirectory (in templates/) where the template files are stored which should be loaded by default.', '1', '22', now());
INSERT INTO configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) VALUES ('Template Switching Allowed', 'TEMPLATE_SWITCHING_ALLOWED', 'true', 'Allow template switching through the url (for easy new template testing).', '1', '22', 'tep_cfg_select_option(array(\'true\', \'false\'), ', now());
INSERT INTO configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) VALUES ('Show Templates Menu', 'TEMPLATE_SWITCHING_MENU', 'true', 'Show the templates menu (mainly for easy template testing, needs Template Switching Allowed to be true).', '1', '22', 'tep_cfg_select_option(array(\'true\', \'false\'), ', now());
```

2、复制 catalog 目录下的所有文件到网站的 catalog 目录里

3、配置参数

BTS 的参数位于:后台的 configuration->My Store 里,参数说明如下:

- Default Template Direcotry:激活的模块名(即模块目录)
  模块文件存放在 templates 文件夹下,每套模块放置在一个单独的文件夹里,模块名用目录名来区分
- TemplateSwitchingAllowed:是否允许切换模块
  切换模块通过使用GET方式传递tplDir的
值实现,tplDir 即模块目录名
- Show Templates Menu:是否显示模块菜单
  选择此选项后,页面顶部会显示模块切换菜单

### BTS原理

BTS 插件实现了 osCommerce 的代码与界面的“半分离”,为什么说它是“半分离”呢?

因为 BTS 在处理界面部分还是很令人满意的,而在代码部分,却综合了程序与显示的功能,所以笔者 谓之“半分离”。采用“半分离”模式进行代码与界面的分离对于 osCommerce 来说应该是一个不 错的折中解决方案,

因为 BTS 通过这种方式将显示部分全部独立到了模板里,然后通过模板调用相应的模块实现每个部分的内容,使得原来的逻辑更加合理与紧凑了; 并且使文件夹结构更加灵活了,因为 BTS 默认了一套显示方案并且采取了灵活的模板应用规则,使得模板的应用更加方便, 修改也更加灵活多样了。

下面我们从程序的角度对 BTS 源代码进行分析:

1、预处理文件:includes/application_top.php[513]

```php
// BOF BTS
require(DIR_WS_INCLUDES . 'configure_bts.php');
// EOF BTS
```

application_top.php 里仅仅新增了加载“configure_bts.php”文件的代码,“configure_bts.php”是 BTS 插件的核心文件,其中最重要的函数“bts_select”即在其中。

2、BTS 函数文件:includes/configure_bts.php 

```php
$tplDir = bts_template_switch();
```

初始化模板 includes/configure_bts.php [12]:

BTS 配置文件里,通过定义基本的目录后,便通过调用“bts_template_switch”来切换当前的模板。

```php
function bts_template_switch() {
  if ((TEMPLATE_SWITCHING_ALLOWED == 'true') && (isset($_GET['tplDir'])) && is_dir(DIR_WS_TEMPLATES_BASE . basename($_GET['tplDir'])) ) {
    $tplDir = basename($_GET['tplDir']); // 模板的变换需要通过 GET 参数“tplDir”来指定 tep_session_register('tplDir');
  } else {
    if ((tep_session_is_registered('tplDir'))&&(TEMPLATE_SWITCHING_ALLOWED == 'true') && is_dir(DIR_WS_TEMPLATES_BASE . basename($_SESSION[tplDir]))){
      $tplDir = basename($_SESSION[tplDir]); // 记住指定要显示的模板 }else{
      $tplDir = DIR_WS_TEMPLATES_DEFAULT; // 否则显示默认的模板 }
  }

  if ((ereg('^[[:alnum:]|_|-]+$', $tplDir)) && (is_dir (DIR_WS_TEMPLATES_BASE . $tplDir))){
    // 'Input Validated' only allow alfanumeric characters and underscores in template name
    define('DIR_WS_TEMPLATES', DIR_WS_TEMPLATES_BASE . $tplDir . '/' ); // 正式指定 模板目录,下面的程序将使用此目录为基准
  } else {
    if($bts_debug === TRUE) echo strip_tags($tplDir) . '<br>';
    exit('Illegal template directory!');
  }

  return $tplDir;
}
```

在上面我们讲到在 BTS 配置文件里,有一个重量级的函数“bts_select”,为什么说它具有如 此重要性,因为在 BTS 实现代码与界面的过程中,bts_select 函数承担了“唤醒”模板的功能, 不论是整个页面的模板,或者是某个特定功能的小模块,甚至精确到图片、CSS 样式表的显示都 离不开它,你说它有多重要呢?

```php
/*
$template_type指定要调用的资源类型,BTS里资源共分为10种: 
1) main:页面
2) content:模块
3) boxes:内容框
4) popup:弹出窗口
5) content_popup:弹出窗口模块
6) javascript:javascript代码
7) stylesheet/stylesheets:CSS样式表
8) column:栏目
9) images:图片 
10) common:普通
*/
function bts_select($template_type, $filename = '') {
  global $content_template, $content, $box_base_name;

  switch ($template_type) {
    case 'main':
      // default or main_page
      if(is_file(DIR_WS_TEMPLATES . TEMPLATENAME_MAIN_PAGE)) {
        $path = (DIR_WS_TEMPLATES . TEMPLATENAME_MAIN_PAGE);
        // 默认的页面模板文件名是 main_page.tpl.php
      } else {
        $path = (DIR_WS_TEMPLATES_FALLBACK . TEMPLATENAME_MAIN_PAGE);
        // 如果当前 模板里没有“main_page.tpl.php”文件,则使用默认模板“fallback”
      }
      break;
    case 'content':
      // pages or content (middle area)
      // extra security: added basename()
      if (isset($content_template)) {
        // 全局变量$content_template 指定需要显示的文件
        if(is_file(DIR_WS_CONTENT . basename($content_template))) {
          $path = (DIR_WS_CONTENT . basename($content_template)); 
        } elseif (is_file(DIR_WS_CONTENT_FALLBACK . basename($content_template))) {
          $path = (DIR_WS_CONTENT_FALLBACK . basename($content_template));
        }
      } else { 
        // 默认使用全局变量$content 来传递文件名,区别$content_template 的地方在于将 引用$content_template 的文件名全称,而使用$content 时,只使用其文件名,扩展名将被替换为“ .tpl.php”
        if(is_file(DIR_WS_CONTENT . basename($content . '.tpl.php'))) {
          $path = (DIR_WS_CONTENT . basename($content . '.tpl.php')); 
        } else {
          $path = (DIR_WS_CONTENT_FALLBACK . $content . '.tpl.php'); 
        }
      }
      break;
    case 'boxes':
      // small sideboxes
      if(is_file(DIR_WS_BOX_TEMPLATES . $box_base_name . '.tpl.php')) {
        // 首先查找当 前模板里是否存在指定名称的内容框的模板
        // if exists, load unique box template for this box from templates/boxes/
        $path = (DIR_WS_BOX_TEMPLATES . $box_base_name . '.tpl.php');
      } elseif(is_file(DIR_WS_BOX_TEMPLATES_FALLBACK . $box_base_name . '.tpl.php')){
        // 否则在默认模板“fallback”里查找是否存在该名称的内容框
        // if exists, load unique box template for this box from templates/boxes/
        $path = (DIR_WS_BOX_TEMPLATES_FALLBACK . $box_base_name . '.tpl.php');
      } elseif(is_file(DIR_WS_BOX_TEMPLATES . TEMPLATENAME_BOX)) {
        // 同样对于默认的内 容框也必须先在当前模板里查找,默认内容框的文件名是“box.tpl.php”
        // if exists, load unique box template for this box from templates/boxes/
        $path = (DIR_WS_BOX_TEMPLATES . TEMPLATENAME_BOX);
      } else {
        $path = (DIR_WS_BOX_TEMPLATES_FALLBACK . TEMPLATENAME_BOX);
        // 使用默认模板里的默认内容框
      }
      break;
    case 'popup':
      // popup main page (images/advanced search)
      if(is_file(DIR_WS_TEMPLATES . TEMPLATENAME_POPUP)) {
        // 查找当前模板里是否有弹出窗 口模板,弹出窗口模板文件名为“popup.tpl.php”
        $path = (DIR_WS_TEMPLATES . TEMPLATENAME_POPUP);
      } else {
        $path = (DIR_WS_TEMPLATES_FALLBACK . TEMPLATENAME_POPUP);
        // 否则使用默认 模板里的弹出窗口模板
      }
      break;
    case 'content_popup':
      // popup pages or content (images/advanced search)
      if(is_file(DIR_WS_CONTENT . basename($content) . '.tpl.php')) {
        //首先在当前模 块的“content”目录查找
        $path = (DIR_WS_CONTENT . basename($content) . '.tpl.php');
      } else {
        $path = (DIR_WS_CONTENT_FALLBACK . basename($content) . '.tpl.php');
        // 否则使用默认模板里的模板
      }
      break;
    case 'javascript':
      $path = ''; // 因为 javascript 文件是通过的,不需要区分模板,所以它未使用 bts_select 进行管理,而改在了直接通过模块调用,javascript 文件存放在“includes/javascript/”
      break;
    case 'stylesheet':
      // $path = DIR_WS_TEMPLATE_FILES . $filename;
      if(is_file(DIR_WS_TEMPLATES . $filename)) {
        // 当前模块里如果有指定的样式表,则返回
        $path = DIR_WS_TEMPLATES . $filename;
      } else {
        $path = DIR_WS_TEMPLATES_FALLBACK . $filename; // 否则使用默认模块里的样式表
      }
      break;
    case 'stylesheets':
      // for example to load different stylesheets per page :: new
      if(is_file(DIR_WS_TEMPLATES . 'stylesheets/' . basename($filename, '.php') . '.css')) {
        $path = DIR_WS_TEMPLATES . 'stylesheets/' . basename($filename, '.php') . '.css';
      } elseif (is_file(DIR_WS_TEMPLATES_FALLBACK . 'stylesheets/' . basename($filename, '.php') . '.css')) {
        $path = DIR_WS_TEMPLATES_FALLBACK . 'stylesheets/' . basename($filename, '.php') . '.css';
      } else {
        return (FALSE);
      }

      // stylesheet 与 stylesheets 的不同在于,stylesheet 允许调用 CSS 样式表之外的文件,因为文 件名直接由$filename 指定,而 stylesheets 则必须是 CSS 样式表,并且 CSS 文件存放于模板的“ stylesheets/”目录里
      break;
    case 'column':
      // enables different columns per template function, falls back to (to fallback/ and then to) stock osC columns (inludes/) if no column templates are found
      if(is_file(DIR_WS_TEMPLATES . $filename)) {
        $path = DIR_WS_TEMPLATES . $filename;
      } elseif (is_file(DIR_WS_TEMPLATES_FALLBACK . $filename)) {
        $path = DIR_WS_TEMPLATES_FALLBACK . $filename;
      } else {
        $path = DIR_WS_INCLUDES . $filename;
      }
      break;
    case 'images':
      // added for loading images directly from your templates directory (w.o. the tep_image() function)
      if (is_file(DIR_WS_TEMPLATES . 'images/' . $filename)) {
        // 图片文件存放于模板的 “images/”目录下,如果当前模板里有指定的图片,则返回
        $path = DIR_WS_TEMPLATES .'images/' . $filename;
      } else {
        $path = DIR_WS_TEMPLATES_FALLBACK . 'images/' . $filename; // 否则返回默认模板里图片
      }
      break;
    case 'common':
      if (is_file(DIR_WS_TEMPLATES_BASE . $filename)) {
        // 普通资源位于模板的顶级目录里, 因为它不属于任何模板,BTS 只用普通资源来作为切换模板之用,细心的读者可以查看文件“ templates/common_top.php”
        $path = (DIR_WS_TEMPLATES_BASE . $filename);
      } else {
        return (FALSE);
      }
      break;
    default:
      // exit ('Error bts_select()! No template selected for template type: ');
      echo ('Error: bts_select()! No template selected for template type: ' . $template_type);
      break;
  }

  return ($path);
}
```

3、模板运用分析

1)界面的调用(templates/fallback/main_page.tpl.php) 

我们以默认的模板“fallback”来介绍模板是如何调用 bts_select 函数实现内容输出的。

```php
<!doctype html public "-//W3C//DTD HTML 4.01 Transitional//EN">
<html <?php echo HTML_PARAMS; ?>>
<head>
<?php require(DIR_WS_INCLUDES . 'meta_tags.php');

// 对于有些模块,可以直接使用 include/require 方法进行调用,直接输出结果,也可以作为预处理过程调用,然后在调用函数过后输出相应 的变量或定义的常量
// meta_tags.php 文件是 BTS 为输出页面的 META 头信息而特别增加的一个文件 
?>
<title><?php echo META_TAG_TITLE; ?></title>
<meta http-equiv="Content-Type" content="text/html; charset=<?php echo CHARSET; ?>"> <meta name="keywords" content="<?php echo META_TAG_KEYWORDS; ?>">
<meta name="description" content="<?php echo META_TAG_DESCRIPTION; ?>">
<base href="<?php echo (($request_type == 'SSL') ? HTTPS_SERVER : HTTP_SERVER) . DIR_WS_CATALOG; ?>">
<link rel="stylesheet" type="text/css" href="<?php echo (bts_select('stylesheet','stylesheet.css')); // 通过 bts_select 函数调用 CSS 样式表,这样就可以使用模块目录里的样式表了 ?>">
<link rel="stylesheet" type="text/css" href="<?php echo (bts_select('stylesheet','stylesheet-new.css')); // BTSv1.5 ?>">
<link rel="stylesheet" type="text/css" href="<?php echo (bts_select('stylesheet','print.css')); // BTSv1.5 ?>" media="print">
<?php if (isset($javascript) && file_exists(DIR_WS_JAVASCRIPT . basename($javascript))) { require(DIR_WS_JAVASCRIPT . basename($javascript)); } ?> </head>
<body>

<!-- warnings //-->
<?php require(DIR_WS_INCLUDES . 'warnings.php'); // BTS 将提示信息框独立成了一个文件 ?> <!-- warning_eof //-->

<?php
// include i.e. template switcher in every template
if(bts_select('common', 'common_top.php')) include (bts_select('common', 'common_top.php')); // BTSv1.5

// common_top.php 文件用于显示切换模块的内容区,是否允许切换模块还需要开启后台的相关设置,这些设 置的判断都在 common_top.php 文件里进行
?>

<!-- header //-->
<table border="0" width="100%" cellspacing="0" cellpadding="0">
  <tr class="header">
    <td valign="middle"><?php echo '<a href="' . tep_href_link(FILENAME_DEFAULT) . '">' . tep_image(DIR_WS_IMAGES . 'store_logo.png', 'osCommerce') . '</a>'; ?></td> <td align="right" valign="bottom"><?php echo '<a href="' . tep_href_link(FILENAME_ACCOUNT, '', 'SSL') . '">' . tep_image(DIR_WS_IMAGES . 'header_account.gif', HEADER_TITLE_MY_ACCOUNT) . '</a>&nbsp;&nbsp;<a href="' . tep_href_link(FILENAME_SHOPPING_CART) . '">' . tep_image(DIR_WS_IMAGES . 'header_cart.gif', HEADER_TITLE_CART_CONTENTS) . '</a>&nbsp;&nbsp;<a href="' . tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL') . '">' . tep_image(DIR_WS_IMAGES . 'header_checkout.gif', HEADER_TITLE_CHECKOUT) . '</a>'; ?>&nbsp;&nbsp;</td>
  </tr>
</table>
<table border="0" width="100%" cellspacing="0" cellpadding="1">
  <tr class="headerNavigation">
    <td class="headerNavigation">&nbsp;&nbsp;<?php echo $breadcrumb->trail(' &raquo;'); ?></td>
    <td align="right" class="headerNavigation"><?php if (tep_session_is_registered('customer_id')) { ?><a href="<?php echo tep_href_link(FILENAME_LOGOFF, '', 'SSL'); ?>" class="headerNavigation"><?php echo HEADER_TITLE_LOGOFF; ?></a> &nbsp;|&nbsp; <?php } ?><a href="<?php echo tep_href_link(FILENAME_ACCOUNT, '', 'SSL'); ?>" class="headerNavigation"><?php echo HEADER_TITLE_MY_ACCOUNT; ?></a> &nbsp;|&nbsp; <a href="<?php echo tep_href_link(FILENAME_SHOPPING_CART); ?>" class="headerNavigation"><?php echo HEADER_TITLE_CART_CONTENTS; ?></a> &nbsp;|&nbsp; <a href="<?php echo tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL'); ?>" class="headerNavigation"><?php echo HEADER_TITLE_CHECKOUT; ?></a> &nbsp;&nbsp;</td>
  </tr>
</table>
<!-- header_eof //-->
<!-- body //-->
<table border="0" width="100%" cellspacing="3" cellpadding="3">
  <tr>
    <td width="<?php echo BOX_WIDTH; ?>" valign="top">
      <table border="0" width="<?php echo BOX_WIDTH; ?>" cellspacing="0" cellpadding="2">
        <!-- column_left //-->
        <?php require(bts_select('column', 'column_left.php')); // 通过 bts_select 函数调用左 边栏内容 ?>
        <!-- column_left eof //-->
      </table>
    </td>
    <td width="100%" valign="top">
      <!-- content //-->
      <?php
        require (bts_select ('content'));
        // 在这里,每个文件才显示出自己独有的内容,bts_select 函数只传递第一个参数时,我们可以通过全局变量$content_template 或者$content 来指定要显示的模板, 通常模板名称与文件名一致,如 login.php 文件的内容模板为 login.tpl.php,那么指定 $content_template 或者$content 的值就是 login
      ?>
      <!-- content_eof //-->
    </td>
    <td width="<?php echo BOX_WIDTH; ?>" valign="top">
      <table border="0" width="<?php echo BOX_WIDTH; ?>" cellspacing="0" cellpadding="2">
        <!-- column_right //-->
        <?php require(bts_select('column', 'column_right.php')); // 调用右边栏的内容 ?>
        <!-- column_right eof //-->
      </table>
    </td>
  </tr>
</table>
<!-- body_eof //-->
<!-- footer //-->
<?php require(DIR_WS_INCLUDES . 'counter.php'); ?>
<table border="0" width="100%" cellspacing="0" cellpadding="1">
  <tr class="footer">
   <td class="footer">&nbsp;&nbsp;<?php echo strftime(DATE_FORMAT_LONG);
?>&nbsp;&nbsp;</td>
   <td align="right" class="footer">&nbsp;&nbsp;<?php echo $counter_now . ' ' .
FOOTER_TEXT_REQUESTS_SINCE . ' ' . $counter_startdate_formatted; ?>&nbsp;&nbsp;</td>
  </tr>
</table>
<br>
<table border="0" width="100%" cellspacing="0" cellpadding="0">
  <tr>
    <td align="center" class="smallText"><?php echo FOOTER_TEXT_BODY ?></td>
  </tr>
</table>

<?php if ($banner = tep_banner_exists('dynamic', '468x50')) { ?>
<br>
<table border="0" width="100%" cellspacing="0" cellpadding="0">
  <tr>
   <td align="center"><?php echo tep_display_banner('static', $banner); ?></td>
  </tr>
</table>
<?php } ?>
<!-- footer_eof //-->
<br>
</body>
</html>
```

2)程序文件的改造

我们通过分析文件 login.php 来展示使用 BTS 前后,代码有哪些变化

```php
<?php
/*
  $Id: login.php,v 1.80 2003/06/05 23:28:24 hpdl Exp $
  osCommerce, Open Source E-Commerce Solutions
  http://www.oscommerce.com
        Copyright (c) 2003 osCommerce
  Released under the GNU General Public License
*/

require('includes/application_top.php');

// redirect the customer to a friendly cookie-must-be-enabled page if cookies are disabled (or the session has not started)
if ($session_started == false) {
  tep_redirect(tep_href_link(FILENAME_COOKIE_USAGE));
}

require(DIR_WS_LANGUAGES . $language . '/' . FILENAME_LOGIN);

$error = false;
if (isset($HTTP_GET_VARS['action']) && ($HTTP_GET_VARS['action'] == 'process')) {
  $email_address = tep_db_prepare_input($HTTP_POST_VARS['email_address']);
  $password = tep_db_prepare_input($HTTP_POST_VARS['password']);
  
  // Check if email exists
  $check_customer_query = tep_db_query("select customers_id, customers_firstname, customers_password, customers_email_address, customers_default_address_id from " . TABLE_CUSTOMERS . " where customers_email_address = '" . tep_db_input($email_address) . "'");
  if (!tep_db_num_rows($check_customer_query)) {
    $error = true;
  } else {
    $check_customer = tep_db_fetch_array($check_customer_query);

    // Check that password is good
    if (!tep_validate_password($password, $check_customer['customers_password'])) {
      $error = true; 
    } else {
      if (SESSION_RECREATE == 'True') {
        tep_session_recreate();
      }

      $check_country_query = tep_db_query("select entry_country_id, entry_zone_id from " . TABLE_ADDRESS_BOOK . " where customers_id = '" . (int)$check_customer['customers_id'] . "' and address_book_id = '" . (int)$check_customer['customers_default_address_id'] . "'");
      
      $check_country = tep_db_fetch_array($check_country_query);
      $customer_id = $check_customer['customers_id'];
      $customer_default_address_id = $check_customer['customers_default_address_id'];
      $customer_first_name = $check_customer['customers_firstname'];
      $customer_country_id = $check_country['entry_country_id'];
      $customer_zone_id = $check_country['entry_zone_id'];
      tep_session_register('customer_id');
      tep_session_register('customer_default_address_id');
      tep_session_register('customer_first_name');
      tep_session_register('customer_country_id');
      tep_session_register('customer_zone_id');

      tep_db_query("update " . TABLE_CUSTOMERS_INFO . " set customers_info_date_of_last_logon = now(), customers_info_number_of_logons = customers_info_number_of_logons+1 where customers_info_id = '" . (int)$customer_id . "'");

      // restore cart contents
      $cart->restore_contents();

      if (sizeof($navigation->snapshot) > 0) {
        $origin_href = tep_href_link($navigation->snapshot['page'],

        tep_array_to_string($navigation->snapshot['get'], array(tep_session_name())),
        $navigation->snapshot['mode']);
        $navigation->clear_snapshot();
        tep_redirect($origin_href);
      } else {
        tep_redirect(tep_href_link(FILENAME_DEFAULT));
      }
    }
  }
}

// 以上代码处理了用户的登录,如果用户输入的信息与注册信息相符时,则登录成功,这些代码与原本的处理 代码一样
if ($error == true) {
  $messageStack->add('login', TEXT_LOGIN_ERROR);
}

$breadcrumb->add(NAVBAR_TITLE, tep_href_link(FILENAME_LOGIN, '', 'SSL'));

// 以下部分显示了 BTS 与原始 osCommerce 程序代码的不同
$content = CONTENT_LOGIN; // 通过变量$content 指定需要显示的主体内容
$javascript = $content . '.js'; // 通过变量$javascript 指定需要嵌入的 javascript 代码文 件

include (bts_select('main', $content_template)); // 调用主模板文件,文件名为“ main_page.tpl.php”
require(DIR_WS_INCLUDES . 'application_bottom.php');
?>
```

> 提示:
> 
> 可以看到,在处理用户输入与分析结果等部分,BTS 使用了原生的 osCommerce 代码,也就是说 BTS 只改造了系统的显示部分,而逻辑部分仍然保持一致。 
> 
> 在改造后显示部分,程序代码只有区区三行(更多情况下只需要二行代码就可以),便可以完成所 有显示界面的任务,这要归功于 BTS 目录的合理分配、模块的有效转换以及模块的灵活运用。


3)新的内容框

内容框在 osCommerce 被广泛使用,所以内容框的样式要如何根据模板进行变更,这就要求
BTS 对内容框进行全面改造了。

BTS 对 osCommerce 的所有内容框文件(includes/boxes/里的所有文件)都进行了相应修改,以便使内容框的显示效果可以由模板自定义,下面是信息内容框(文件: includes/boxes/information.php),我们将说明改造后的内容框有什么不同

```php
<?php /*
  $Id: information.php,v 1.6 2003/02/10 22:31:00 hpdl Exp $
  modified by paulm_nl 2003/12/23
  osCommerce, Open Source E-Commerce Solutions
  http://www.oscommerce.com
  Copyright (c) 2003 osCommerce
  Released under the GNU General Public License
*/
$boxHeading = BOX_HEADING_INFORMATION;
$corner_left = 'square';
$corner_right = 'square';

// 上面三个参数与原来的代码一致,用于显示内容框的标题以及指定内容框的边角样式
$box_base_name = 'information'; // for easy unique box template setup (added BTSv1.2)

$box_id = $box_base_name . 'Box'; // for CSS styling paulm (editted BTSv1.2)

$boxContent = '<a href="' . tep_href_link(FILENAME_SHIPPING) . '"> ' . BOX_INFORMATION_SHIPPING . '</a><br>' . '<a href="' . tep_href_link(FILENAME_PRIVACY) . '"> ' . BOX_INFORMATION_PRIVACY . '</a><br>' . '<a href="' . tep_href_link(FILENAME_CONDITIONS) . '"> ' . BOX_INFORMATION_CONDITIONS . '</a><br>' . '<a href="' . tep_href_link(FILENAME_CONTACT_US) . '"> ' .
BOX_INFORMATION_CONTACT . '</a>';

// $boxContent变量指定要显示在内容框主体的内容,这里与原来的代码不同,原先需要使用 infoBoxHeading 类显示内容框标题,然后通过 infoBox 类显示内容框的主体内容

include (bts_select('boxes', $box_base_name)); // BTS 1.5

// 新的显示方案,只需要调用 bts_select 函数返回的模板文件即可,这里通过传入变量$box_base_name 指定需要显示的模板文件,使得用户可以在使用统一的模板样式(刚相应的模板文件不存在时自动调用统一模板 文件)外,还可以自定义每个单独的内容框具有不同的样式
?>
```

以下是还原 table 样式的内容框(文件:templates/fallback/boxes/box.tpl.php) 

```php
<?php /* infobox template */ ?>
<tr>
  <td>
    <table border="0" width="100%" cellspacing="0" cellpadding="0">
      <tr>
        <td height="14" class="infoBoxHeading"><img src="images/infobox/<?php switch ($corner_left) { case 'square': echo 'corner_right_left.gif'; break; case 'rounded': echo 'corner_left.gif'; break;} ?>" border="0" alt="" width="11" height="14"></td>
        <td width="100%" height="14" class="infoBoxHeading"><?php echo
$boxHeading; ?></td>
        <td height="14" class="infoBoxHeading" nowrap><?php echo $boxLink; ?><img src="images/<?php switch ($corner_right) { case 'square': echo 'pixel_trans.gif'; break; case 'rounded': echo 'infobox/corner_right.gif'; break;} ?>" border="0" alt="" width="11" height="14"></td>
      </tr>
    </table>
    <table border="0" width="100%" cellspacing="0" cellpadding="1"
class="infoBox">
      <tr>
        <td>
          <table border="0" width="100%" cellspacing="0" cellpadding="3" class="infoBoxContents">
            <tr>
              <td><img src="images/pixel_trans.gif" border="0" alt="" width="100%"
height="1"></td>
            </tr>
            <tr>
              <td class="boxText"<?php echo $boxContent_attributes; ?>><?php echo $boxContent; ?></td>
            </tr>
            <tr>
             <td><img src="images/pixel_trans.gif" border="0" alt="" width="100%" height="1"></td>
            </tr>
          </table>
        </td>
      </tr>
    </table>
  </td>
</tr>
```

这个则是 DIV+CSS 设计的内容框模板(文件:templates/ CSS-fluid-1/boxes/box.tpl.php)

```php
<?php /* infobox template */ ?>
<div id="<?php if (isset($box_id)) {echo $box_id;} ?>" class="infoBoxFL">
  <div class="infoBoxHeadingFL"><span class="boxLink"><?php echo $boxLink;
?></span><?php echo $boxHeading; ?></div>
  <div class="infoBoxContentsFL"><?php echo $boxContent; ?></div>
</div>
```
