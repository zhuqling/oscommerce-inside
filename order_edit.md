## 订单编辑

我们介绍一个编辑订单的插件,对于灵活变更产品价格、以及提供不同客户适当的优惠将非常有 用,我们选用 Order Editor 插件,截止笔者写稿时它的版本为 5.0.61

插件下载地址: http://addons.oscommerce.com/info/1435

[安装]

1、执行以下 SQL 语句,生成 Order Edit 参数选项

```sql
ALTER TABLE orders ADD shipping_module VARCHAR( 255 ) NULL ; 
# 添加新字段:运输方式 
INSERT INTO configuration_group (configuration_group_id, configuration_group_title, configuration_group_description, sort_order, visible) VALUES ('72', 'Order Editor', 'Configuration options for Order Editor', 72, 1);
INSERT into configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, last_modified, date_added, use_function, set_function) values ('Display the Payment Method dropdown?', 'ORDER_EDITOR_PAYMENT_DROPDOWN', 'true', 'Based on this selection Order Editor will display the payment method as a dropdown menu (true) or as an input field (false).', '72', '1', now(), now(), NULL, 'tep_cfg_select_option(array(\'true\', \'false\'),');
INSERT into configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, last_modified, date_added, use_function, set_function) values ('Use prices from Separate Pricing Per Customer?', 'ORDER_EDITOR_USE_SPPC', 'false', 'Leave this set to false unless SPPC is installed.', '72', '3', now(), now(), NULL, 'tep_cfg_select_option(array(\'true\', \'false\'),');
INSERT into configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, last_modified, date_added, use_function, set_function) values ('Allow the use of AJAX to update order information?', 'ORDER_EDITOR_USE_AJAX', 'true', 'This must be set to false if using a browser on which JavaScript is disabled or not available.', '72', '4', now(), now(), NULL, 'tep_cfg_select_option(array(\'true\', \'false\'),'); INSERT into configuration (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, last_modified, date_added, use_function, set_function) values ('Select your credit card payment method', 'ORDER_EDITOR_CREDIT_CARD', 'Credit Card', 'Order Editor will display the credit card fields when this payment method is selected.', '72', '5', now(), now(), NULL, 'tep_cfg_pull_down_payment_methods(');
```

2、复制如下文件到相对应的目录里: 

- admin/edit_orders.php
- admin/edit_orders_ajax.php
- admin/edit_orders_add_product.php
- admin/order_editor/cart.php
- admin/order_editor/css.php
- admin/order_editor/functions.php
- admin/order_editor/http_client.php
- admin/order_editor/javascript.php
- admin/order_editor/order.php
- admin/order_editor/order_total.php
- admin/order_editor/shipping.php
- admin/includes/languages/english/edit_orders.php 
- admin/includes/languages/english/images/buttons/button_add_article.gif
- admin/includes/languages/english/images/buttons/button_new_order_email.gif 

3、修改文件 catalog/checkout_process.php

查找(约第 87 行):

```php
'payment_method' => $order->info['payment_method'],
'cc_type' => $order->info['cc_type'],
```

替换成如下代码:

```php
'payment_method' => $order->info['payment_method'],
'shipping_module' => $shipping['id'],
'cc_type' => $order->info['cc_type'],
```

4、修改文件 admin/orders.php 

查找(约第 127 行):

```php
<td class="pageHeading" align="right"><?php echo '<a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('action'))) . '">' . tep_image_button('button_back.gif', IMAGE_BACK) . '</a>'; ?></td>
```

替换成如下代码:

```php
<td class="pageHeading" align="right"><?php echo '<a href="' . tep_href_link(FILENAME_ORDERS_EDIT, 'oID=' . $_GET['oID']) . '">' . tep_image_button('button_edit.gif', IMAGE_EDIT) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_INVOICE, 'oID=' . $_GET['oID']) . '" TARGET="_blank">' . tep_image_button('button_invoice.gif', IMAGE_ORDERS_INVOICE) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_PACKINGSLIP, 'oID=' . $_GET['oID']) . '"TARGET="_blank">' . tep_image_button('button_packingslip.gif', IMAGE_ORDERS_PACKINGSLIP) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('action'))) . '">' . tep_image_button('button_back.gif', IMAGE_BACK) . '</a> '; ?></td>
```

查找(约第 321 行):

```php
<td colspan="2" align="right"><?php echo '<a href="' . tep_href_link(FILENAME_ORDERS_INVOICE, 'oID=' . $HTTP_GET_VARS['oID']) . '" TARGET="_blank">' . tep_image_button('button_invoice.gif', IMAGE_ORDERS_INVOICE) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_PACKINGSLIP, 'oID=' . $HTTP_GET_VARS['oID']) . '" TARGET="_blank">' . tep_image_button('button_packingslip.gif', IMAGE_ORDERS_PACKINGSLIP) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('action'))) . '">' . tep_image_button('button_back.gif', IMAGE_BACK) . '</a>'; ?></td>
```

替换成以下代码:

```php
<td colspan="2" align="right"><?php echo '<a href="' . tep_href_link(FILENAME_ORDERS_EDIT, 'oID=' . $_GET['oID']) . '">' . tep_image_button('button_edit.gif', IMAGE_EDIT) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_INVOICE, 'oID=' . $_GET['oID']) . '" TARGET="_blank">' . tep_image_button('button_invoice.gif', IMAGE_ORDERS_INVOICE) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_PACKINGSLIP, 'oID=' . $_GET['oID']) . '" TARGET="_blank">' . tep_image_button('button_packingslip.gif', IMAGE_ORDERS_PACKINGSLIP) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('action'))) . '">' . tep_image_button('button_back.gif', IMAGE_BACK) . '</a> '; ?></td>
```

查找(大约第 410 行):

```php
$contents[] = array('align' => 'center', 'text' => '<a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('oID', 'action')) . 'oID=' . $oInfo->orders_id . '&action=edit') . '">' . tep_image_button('button_edit.gif', IMAGE_EDIT) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('oID', 'action')) . 'oID=' . $oInfo->orders_id . '&action=delete') . '">' . tep_image_button('button_delete.gif', IMAGE_DELETE) . '</a>');
$contents[] = array('align' => 'center', 'text' => '<a href="' . tep_href_link(FILENAME_ORDERS_INVOICE, 'oID=' . $oInfo->orders_id) . '" TARGET="_blank">' . tep_image_button('button_invoice.gif', IMAGE_ORDERS_INVOICE) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_PACKINGSLIP, 'oID=' . $oInfo->orders_id) . '" TARGET="_blank">' . tep_image_button('button_packingslip.gif', IMAGE_ORDERS_PACKINGSLIP) . '</a>');
```

替换成如下代码:

```php
$contents[] = array('align' => 'center', 'text' => '<a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('oID', 'action')) . 'oID=' . $oInfo->orders_id . '&action=edit') . '">' . tep_image_button('button_details.gif', IMAGE_DETAILS) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS, tep_get_all_get_params(array('oID', 'action')) . 'oID=' . $oInfo->orders_id . '&action=delete') . '">' . tep_image_button('button_delete.gif', IMAGE_DELETE) . '</a>');
$contents[] = array('align' => 'center', 'text' => '<a href="' . tep_href_link(FILENAME_ORDERS_INVOICE, 'oID=' . $oInfo->orders_id) . '" TARGET="_blank">' . tep_image_button('button_invoice.gif', IMAGE_ORDERS_INVOICE) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_PACKINGSLIP, 'oID=' . $oInfo->orders_id) . '" TARGET="_blank">' . tep_image_button('button_packingslip.gif', IMAGE_ORDERS_PACKINGSLIP) . '</a> <a href="' . tep_href_link(FILENAME_ORDERS_EDIT, 'oID=' . $oInfo->orders_id) . '">' . tep_image_button('button_edit.gif', IMAGE_EDIT) . '</a>');
```

5、修改文件 admin/includes/filenames.php 

添加以下的定义:

```php
  // order editor
  define('FILENAME_ORDERS_EDIT', 'edit_orders.php');
  define('FILENAME_ORDERS_EDIT_ADD_PRODUCT', 'edit_orders_add_product.php');
  define('FILENAME_ORDERS_EDIT_AJAX', 'edit_orders_ajax.php');
  // end order editor
```

6、修改文件 admin/includes/functions/general.php 

添加 tep_cfg_pull_down_payment_methods 函数:

```php
//////create a pull down for all payment installed payment methods for Order Editor configuration
// Get list of all payment modules available
function tep_cfg_pull_down_payment_methods() {
  global $language;

  $enabled_payment = array();
  $module_directory = DIR_FS_CATALOG_MODULES . 'payment/';
  $file_extension = '.php';
  if ($dir = @dir($module_directory)) {
    while ($file = $dir->read()) {
      if (!is_dir( $module_directory . $file)) {
        if (substr($file, strrpos($file, '.')) == $file_extension) {
          $directory_array[] = $file;
        }
      }
    }

    sort($directory_array);
    $dir->close();
  }

  // For each available payment module, check if enabled
  for ($i=0, $n=sizeof($directory_array); $i<$n; $i++) {
    $file = $directory_array[$i];
    include(DIR_FS_CATALOG_LANGUAGES . $language . '/modules/payment/' . $file);
    include($module_directory . $file);
    $class = substr($file, 0, strrpos($file, '.'));
    if (tep_class_exists($class)) {
      $module = new $class;
      if ($module->check() > 0) {
        // If module enabled create array of titles
        $enabled_payment[] = array('id' => $module->title, 'text' => $module->title);
      }
    }
  }

  $enabled_payment[] = array('id' => 'Other', 'text' => 'Other');
  //draw the dropdown menu for payment methods and default to the order value
  return tep_draw_pull_down_menu('configuration_value', $enabled_payment, '', '');
}
/////end payment method dropdown
```
