## 管理后台新界面

osCommerce后台管理系统的导航相当规矩,使得如果想快速到达某个子菜单,将是不可以的。这样就导致了效率无法提高。

Mindsparx admin插件不仅美化了管理后台的界面(页面样式的美化、按钮的更新),其最具特色 的在于重写了后台的菜单导航。,在将原先左侧的菜单移到页面的上方后,使用了 Javascript 技术, 将菜单改为下拉式菜单,即扩展了页面的视野,也大大提高了操作的便利性。 插件地址:http://addons.oscommerce.com/info/6440

[安装]

以下采用的 Mindsparx admin 2.1 版本,针对原始的 osCommerce RC2.2a 进行介绍,如果涉及修改 或覆盖的文件,在此之前已经有所变更,则建议读者针对新版本文件的特征对文件进行逐一的修改。
  
1、复制以下文件到对应文件夹:

-  admin/index.php
-  admin/login.php
-  admin/includes/header.php
-  admin/includes/header.php
-  admin/includes/column_left.php
-  admin/includes/languages/english/index.php
-  admin/includes/boxes

2、修改语言文件 catalog/admin/includes/languages/english.php

添加以下定义:

```php
define('LOGG_OUT', 'Logoff');

define('HEADING_TITLE2', '10 best viewed products');
define('TABLE_HEADING_VIEWED2', 'Viewed');

define('BOX_REPORTS_STOCK_LEVEL', 'Low Stock Report');

define('HEADER_WARNING', 'Here you can put a warning for your clients <br>Warning! Please take a database backup before change these settings. '); 

// admin welcome text
define('TEXT5', 'You have ');
define('TEXT6', ' customers in total and ');
define('TEXT7', ' products in your store. ');
define('TEXT8', ' of your products has been reviewed.');
define('DO_USE', 'You can use the quick navigation at the top of the page to manage your orders.');
define('WELCOME_BACK', 'Welcome back ');
define('STOCK_TEXT_WARNING1','<b><font color="#990000">Warning!</font></b>you have ');
define('STOCK_TEXT_WARNING2', ' product(s) that 磗 running out of stock. Click here ');
define('STOCK_TEXT_WARNING3', ' to see your stock status.'); define('STOCK_TEXT_OK1', '<font color="#009900 ">Your stock status is good</font> and no new products need to be ordered. Click here ');
define('STOCK_TEXT_OK2', ' to see your stock status.');
// admin welcome text end

// summary info v1.1 plugin by conceptlaboratory.com define('TEXT_SUMMARY_INFO_WHOS_ONLINE', 'Users Online: %s'); define('TEXT_SUMMARY_INFO_CUSTOMERS', 'Total Customers: %s, Today: %s'); define('TEXT_SUMMARY_INFO_ORDERS', 'Your Order Status Is: <br> %s, <b>Today:</b> %s'); define('TEXT_SUMMARY_INFO_REVIEWS', 'Total Reviews: %s, Today: %s'); define('TEXT_SUMMARY_INFO_TICKETS', 'Ticket Status %s'); define('TEXT_SUMMARY_INFO_ORDERS_TOTAL', 'Your Order Total is: <br> %s,<b> Today: </b>%s');
// summary info v1.1 plugin by conceptlaboratory.com eof
```

3、修改文件 admin/includes/filenames.php

添加以下定义:

`define('FILENAME_STATS_LOW_STOCK', 'stats_low_stock.php');`

4、修改文件 admin/includes/application_top 

查找:

```php
// customization for the design layout
define('BOX_WIDTH', 125); // how wide the boxes should be in pixels (default: 125)
```

将 BOX_WIDTH 修改为 0:

`define('BOX_WIDTH', 0);`

5、修改文件 admin/includes/functions/html_output.php 

查找:

`function tep_image_button($image, $alt = '', $params = '') {`

将 tep_image_button 函数替换以下:
     
```php
function tep_image_button($image, $alt = '', $params = '') {
  global $language;

  if (ADMIN_BUTTON =="true"){
    return tep_image(DIR_WS_ADMIN . 'mindsparx_admin/template/'. ADMIN_TEMPLATE.'/images/buttons/'. $language . '/' . $image, $alt, '', '', $params);
  } else {
    return tep_image(DIR_WS_LANGUAGES . $language . '/images/buttons/' . $image, $alt, '', '', $params);
  }
}
```

6、重命名文件夹"admin/includes/"里的"stylesheet.css"为"stylesheet_old.css"(此步骤可以省略)
