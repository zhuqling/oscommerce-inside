## 多图片支持

osCommerce 默认一个产品只拥有一张图片,这样对于产品的展示明显不够,所以我们需要对产品进行扩展,让一个产品可以显示多张图片。

More Pics 是 osCommerce 里扩展产品多图片的插件,More Pics 插件有经典版(classic)和高 级版(advanced),因为一般情况下,经典版都可以满足需要,所以我们在此只介绍经典版。对于高级版与经典版的不同以及如何升级到高级版,请读者自行查看插件的说明。

解压后的 more_pics_classic_1.4.3 文件夹下包括“mp_autoinstaller_2.7_english”和“mp_autoinstaller_2.7_french”两个文件夹,这两个目录是 More Pics 插件的自动安装包,分别对应于不同的语言:英语和法语,如果 osCommerce 没有做过任何修改,建议使用自动安装包,这样可以很方便地安装 More Pics 插件。

我们介绍手动安装 More Pics 插件的步骤,手动安装文件在“more_pics_classic_1.4.3/_More Pics 6 1.4.3”目录中。

[安装]

1、执行以下 SQL 语句:

```sql
ALTER TABLE `products` ADD `products_subimage1` VARCHAR(64) AFTER `products_image`; 
ALTER TABLE `products` ADD `products_subimage2` VARCHAR(64) AFTER `products_subimage1`;
ALTER TABLE `products` ADD `products_subimage3` VARCHAR(64) AFTER `products_subimage2`;
ALTER TABLE `products` ADD `products_subimage4` VARCHAR(64) AFTER `products_subimage3`;
ALTER TABLE `products` ADD `products_subimage5` VARCHAR(64) AFTER `products_subimage4`;
ALTER TABLE `products` ADD `products_subimage6` VARCHAR(64) AFTER `products_subimage5`;

INSERT INTO `configuration_group` (`configuration_group_id`, `configuration_group_title`, `configuration_group_description`, `sort_order`, `visible`) VALUES (6124, 'More Pics', 'Configuration Options for More Pics', 20, 1);
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Show All MorePics on Product Info page', 'MOPICS_SHOW_ALL_ON_PRODUCT_INFO', 'true', 'Show All extra images from the MorePics contrib on Product Info page.', 6124, 2, now(), now(), NULL, 'tep_cfg_select_option(array(''true'', ''false''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Group parent image with sub-images', 'MOPICS_GROUP_WITH_PARENT', 'true', 'Set to true to group all images with parent (original) image.', 6124, 4, now(), now(), NULL, 'tep_cfg_select_option(array(''true'', ''false''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Use SMALL_IMAGE_ Restrictions', 'MOPICS_RESTRICT_IMAGE_SIZE', 'true', 'Restrict all product images to SMALL_IMAGE_WIDTH and SMALL_IMAGE_HEIGHT values.', 6124, 6, now(), now(), NULL, 'tep_cfg_select_option(array(''true'', ''false''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Restrict parent image size', 'MOPICS_RESTRICT_PARENT', 'true', 'Restrict parent image size to SMALL_IMAGE_WIDTH and SMALL_IMAGE_HEIGHT values. This setting overrides "Use SMALL_IMAGE_ Restrictions" setting.', 6124, 8, now(), now(), NULL, 'tep_cfg_select_option(array(''true'', ''false''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Table Alignment', 'MOPICS_TABLE_ALIGNMENT', 'center', 'Align Pics table to the left or to the right of the products description.', 6124, 10, now(), now(), NULL, 'tep_cfg_select_option(array(''left'', ''right'', ''center''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Table Location', 'MOPICS_TABLE_LOCATION', 'below', 'Align Pics table to the sides, above or below description.', 6124, 12, now(), now(), NULL, 'tep_cfg_select_option(array(''sides'', ''above'', ''below'', ''bottom''),');
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Number of Columns', 'MOPICS_NUMBER_OF_COLS', '3', 'Number of columns to display.', 6124, 14, now(), now(), NULL, NULL);
INSERT INTO `configuration` ( `configuration_title`, `configuration_key`, `configuration_value`, `configuration_description`, `configuration_group_id`, `sort_order`, `last_modified`, `date_added`, `use_function`, `set_function`) VALUES ( 'Number of Rows', 'MOPICS_NUMBER_OF_ROWS', '2', 'Number of rows to display.', 6124, 16, now(), now(), NULL, NULL);
```

2、修改语言文件:catalog/includes/languages/english.php 

添加以下定义:

```php
// BOF: More Pics 6
define('CLOSE_POPUP', 'close window');
define('MORE_PIC', 'Screenshots');
// EOF: More Pics 6
```

3、修改样式表文件:stylesheet.css 

增加以下 CSS 样式:


```css
/* BOF: More Pics 6 */
TABLE.popup {
  border-width: 1px;
  border-style: dotted;
  border-color
}
/* EOF: More Pics 6 */
```

4、复制以下图片文件

-  catalog/images/left.gif
-  catalog/images/right.gif
-  catalog/images/zoom.gif

5、修改文件:catalog/admin/categories.php

查找:

```php
if (isset($HTTP_POST_VARS['products_image']) &&
tep_not_null($HTTP_POST_VARS['products_image']) &&
($HTTP_POST_VARS['products_image'] != 'none')) {
          $sql_data_array['products_image'] =
tep_db_prepare_input($HTTP_POST_VARS['products_image']);
}
```

替换为:

```php
// BOF: More Pics 6
if(($HTTP_POST_VARS['unlink_image'] == 'yes') || ($HTTP_POST_VARS['delete_image'] == 'yes')){
  $sql_data_array['products_image'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_image']) && tep_not_null($HTTP_POST_VARS['products_image']) && ($HTTP_POST_VARS['products_image'] != 'none')) {
  $sql_data_array['products_image'] = tep_db_prepare_input($HTTP_POST_VARS['products_image']);
}

// add delete image function
if ($HTTP_POST_VARS['delete_image'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_image']);
}
if(($HTTP_POST_VARS['unlink_image_3'] == 'yes') || ($HTTP_POST_VARS['delete_image_3'] == 'yes')){
  $sql_data_array['products_subimage1'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_subimage1']) && tep_not_null($HTTP_POST_VARS['products_subimage1']) && ($HTTP_POST_VARS['products_subimage1'] != 'none')) {
  $sql_data_array['products_subimage1'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage1']);
}

// add delete image function
if ($HTTP_POST_VARS['delete_image_3'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_subimage1']);
}
// end delete image function

if(($HTTP_POST_VARS['unlink_image_5'] == 'yes') || ($HTTP_POST_VARS['delete_image_5'] == 'yes')){
  $sql_data_array['products_subimage2'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_subimage2']) && tep_not_null($HTTP_POST_VARS['products_subimage2']) && ($HTTP_POST_VARS['products_subimage2'] != 'none')) {
  $sql_data_array['products_subimage2'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage2']);
}

//add delete image function
if ($HTTP_POST_VARS['delete_image_5'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_subimage2']);
}
//end delete image function

if(($HTTP_POST_VARS['unlink_image_7'] == 'yes') || ($HTTP_POST_VARS['delete_image_7'] == 'yes')){
  $sql_data_array['products_subimage3'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_subimage3']) && tep_not_null($HTTP_POST_VARS['products_subimage3']) && ($HTTP_POST_VARS['products_subimage3'] != 'none')) {
  $sql_data_array['products_subimage3'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage3']);
}

//add delete image function
if ($HTTP_POST_VARS['delete_image_7'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES .$HTTP_POST_VARS['products_previous_subimage3']);
}
//end delete image function

if(($HTTP_POST_VARS['unlink_image_9'] == 'yes') || ($HTTP_POST_VARS['delete_image_9'] == 'yes')){
  $sql_data_array['products_subimage4'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_subimage4']) && tep_not_null($HTTP_POST_VARS['products_subimage4']) && ($HTTP_POST_VARS['products_subimage4'] != 'none')) {
  $sql_data_array['products_subimage4'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage4']);
}

//add delete image function
if ($HTTP_POST_VARS['delete_image_9'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_subimage4']);
}
//end delete image function

if(($HTTP_POST_VARS['unlink_image_11'] == 'yes') || ($HTTP_POST_VARS['delete_image_11'] == 'yes')){
  $sql_data_array['products_subimage5'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
}else if (isset($HTTP_POST_VARS['products_subimage5']) && tep_not_null($HTTP_POST_VARS['products_subimage5']) && ($HTTP_POST_VARS['products_subimage5'] != 'none')) {
  $sql_data_array['products_subimage5'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage5']);
}
//add delete image function
if ($HTTP_POST_VARS['delete_image_11'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_subimage5']);
}
//end delete image function

if(($HTTP_POST_VARS['unlink_image_13'] == 'yes') || ($HTTP_POST_VARS['delete_image_13'] == 'yes')){
  $sql_data_array['products_subimage6'] = tep_db_prepare_input($HTTP_POST_VARS['none']);
} else if (isset($HTTP_POST_VARS['products_subimage6']) && tep_not_null($HTTP_POST_VARS['products_subimage6']) && ($HTTP_POST_VARS['products_subimage6'] != 'none')) {
  $sql_data_array['products_subimage6'] = tep_db_prepare_input($HTTP_POST_VARS['products_subimage6']);
}

//add delete image function
if ($HTTP_POST_VARS['delete_image_13'] == 'yes') {
  unlink(DIR_FS_CATALOG_IMAGES . $HTTP_POST_VARS['products_previous_subimage6']);
}
//end delete image function

// EOF: More Pics 6
```

查找:

```php
$product_query = tep_db_query("select products_quantity, products_model, products_image, products_price, products_date_available, products_weight, products_tax_class_id, manufacturers_id from " . TABLE_PRODUCTS . " where products_id = '" . (int)$products_id . "'");
$product = tep_db_fetch_array($product_query);
tep_db_query("insert into " . TABLE_PRODUCTS . " (products_quantity, products_model,products_image, products_price, products_date_added, products_date_available, products_weight, products_status, products_tax_class_id, manufacturers_id) values ('" . tep_db_input($product['products_quantity']) . "', '" . tep_db_input($product['products_model']) . "', '" . tep_db_input($product['products_image']) . "', '" . tep_db_input($product['products_price']) . "', now(), " . (empty($product['products_date_available']) ? "null" : "'" . tep_db_input($product['products_date_available']) . "'") . ", '" . tep_db_input($product['products_weight']) . "', '0', '" . (int)$product['products_tax_class_id'] . "', '" . (int)$product['manufacturers_id'] . "')");
```

替换为:

```php
// BOF: More Pics 6 Added: , products_subimage1, products_subimage2, products_subimage3, products_subimage4, products_subimage5, products_subimage6
$product_query = tep_db_query("select products_quantity, products_model, products_image, products_subimage1, products_subimage2, products_subimage3, products_subimage4, products_subimage5, products_subimage6, products_price, products_date_available, products_weight, products_tax_class_id, manufacturers_id from " . TABLE_PRODUCTS . " where products_id = '" . (int)$products_id . "'");

// EOF: More Pics 6
$product = tep_db_fetch_array($product_query);
// BOF: More Pics 6 Added: , products_subimage1, products_subimage2, products_subimage3, products_subimage4, products_subimage5, products_subimage6 -AND- ,'".tep_db_input($product['products_subimage1'])."','". tep_db_input($product['products_subimage2']) . "', '" . tep_db_input($product['products_subimage3']) . "', '" . tep_db_input($product['products_subimage4']) . "', '" . tep_db_input($product['products_subimage5']) . "', '" . tep_db_input($product['products_subimage6']) . "'
tep_db_query("insert into " . TABLE_PRODUCTS . " (products_quantity, products_model, products_image, products_subimage1, products_subimage2, products_subimage3, products_subimage4, products_subimage5, products_subimage6, products_price, products_date_added, products_date_available, products_weight, products_status, products_tax_class_id, manufacturers_id) values ('" . tep_db_input($product['products_quantity']) . "', '" . tep_db_input($product['products_model']) . "', '" . tep_db_input($product['products_image']) . "', '" . tep_db_input($product['products_subimage1']) . "', '" . tep_db_input($product['products_subimage2']) . "', '" . tep_db_input($product['products_subimage3']) . "', '" . tep_db_input($product['products_subimage4']) . "', '" . tep_db_input($product['products_subimage5']) . "', '" . tep_db_input($product['products_subimage6']) . "', '" . tep_db_input($product['products_price']) . "', now(), '" . tep_db_input($product['products_date_available']) . "', '" . tep_db_input($product['products_weight']) . "', '0', '" . (int)$product['products_tax_class_id'] . "', '" . (int)$product['manufacturers_id'] . "')");
// EOF: More Pics 6
```

查找:

```php
} else {
  $products_image_name = (isset($HTTP_POST_VARS['products_previous_image']) ? $HTTP_POST_VARS['products_previous_image'] : '');
}
```

在其下追加以下代码:

```php
// BOF: More Pics 6
// copy subimage1 only if modified
$products_subimage1 = new upload('products_subimage1');
$products_subimage1->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage1->parse() && $products_subimage1->save()) {
  $products_subimage1_name = $products_subimage1->filename;
} else {
  $products_subimage1_name = (isset($HTTP_POST_VARS['products_previous_subimage1']) ? $HTTP_POST_VARS['products_previous_subimage1'] : '');
}

// copy subimage2 only if modified
$products_subimage2 = new upload('products_subimage2');
$products_subimage2->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage2->parse() && $products_subimage2->save()) {
  $products_subimage2_name = $products_subimage2->filename;
} else {
  $products_subimage2_name = (isset($HTTP_POST_VARS['products_previous_subimage2']) ? $HTTP_POST_VARS['products_previous_subimage2'] : '');
}

// copy subimage3 only if modified
$products_subimage3 = new upload('products_subimage3');
$products_subimage3->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage3->parse() && $products_subimage3->save()) {
  $products_subimage3_name = $products_subimage3->filename;
} else {
  $products_subimage3_name = (isset($HTTP_POST_VARS['products_previous_subimage3']) ? $HTTP_POST_VARS['products_previous_subimage3'] : '');
}

// copy subimage4 only if modified
$products_subimage4 = new upload('products_subimage4');
$products_subimage4->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage4->parse() && $products_subimage4->save()) {
  $products_subimage4_name = $products_subimage4->filename;
} else {
  $products_subimage4_name = (isset($HTTP_POST_VARS['products_previous_subimage4']) ? $HTTP_POST_VARS['products_previous_subimage4'] : '');
}

// copy subimage5 only if modified
$products_subimage5 = new upload('products_subimage5');
$products_subimage5->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage5->parse() && $products_subimage5->save()) {
  $products_subimage5_name = $products_subimage5->filename;
} else {
  $products_subimage5_name = (isset($HTTP_POST_VARS['products_previous_subimage5']) ? $HTTP_POST_VARS['products_previous_subimage5'] : '');
}

// copy subimage6 only if modified
$products_subimage6 = new upload('products_subimage6');
$products_subimage6->set_destination(DIR_FS_CATALOG_IMAGES);
if ($products_subimage6->parse() && $products_subimage6->save()) {
  $products_subimage6_name = $products_subimage6->filename;
} else {
  $products_subimage6_name = (isset($HTTP_POST_VARS['products_previous_subimage6']) ? $HTTP_POST_VARS['products_previous_subimage6'] : '');
}
// EOF: More Pics 6
```

查找:

```php
$parameters = array(
  'products_name' => '',
  'products_description' => '',
  'products_url' => '',
  'products_id' => '',
  'products_quantity' => '',
  'products_model' => '',
  'products_image' => '',
```

在其下追加:

```php
// BOF: More Pics 6
  'products_subimage1' => '',
  'products_subimage2' => '',
  'products_subimage3' => '',
  'products_subimage4' => '',
  'products_subimage5' => '',
  'products_subimage6' => '',
// EOF: More Pics 6
```

查找:

```php
$product_query = tep_db_query("select pd.products_name, pd.products_description, pd.products_url, p.products_id, p.products_quantity, p.products_model, p.products_image, p.products_price, p.products_weight, p.products_date_added, p.products_last_modified, date_format(p.products_date_available, '%Y-%m-%d') as products_date_available, p.products_status, p.products_tax_class_id, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_id = '" . (int)$HTTP_GET_VARS['pID'] . "' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "'");
```                   

替换为:

```php
// BOF: More Pics 6 Added: , p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6
$product_query = tep_db_query("select pd.products_name, pd.products_description, pd.products_url, p.products_id, p.products_quantity, p.products_model, p.products_image, p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6, p.products_price, p.products_weight, p.products_date_added, p.products_last_modified, date_format(p.products_date_available, '%Y-%m-%d') as products_date_available, p.products_status, p.products_tax_class_id, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_id = '" . (int)$HTTP_GET_VARS['pID'] . "' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "'");
// EOF: More Pics 6
```

查找:

```php
<td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' . tep_draw_file_field('products_image') . '<br>' . tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' . $pInfo->products_image . tep_draw_hidden_field('products_previous_image', $pInfo->products_image); ?></td>
```

替换为:

```php
<!-- BOF More Pics -->
<td class="main" ><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_image') . tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' . $pInfo->products_image . tep_draw_hidden_field('products_previous_image', $pInfo->products_image); ?>
</td> </tr>
        <?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_image) != '') { ?>
        <tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">

                  <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES . $pInfo->products_image, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image', 'yes', false) . TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
echo tep_draw_separator('pixel_trans.gif', '5', '15') . tep_draw_checkbox_field('delete_image', 'yes', false) . TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
        </tr>
          <td class="main" align="top"><?php echo TEXT_PRODUCTS_SUBIMAGE1; ?>
<td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' . tep_draw_file_field('products_subimage1') . tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' . $pInfo->products_subimage1 . tep_draw_hidden_field('products_previous_subimage1', $pInfo->products_subimage1); ?>
</td> </tr>
<?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage1) != '') { <tr>
                                                              ?>

                <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage1, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_3', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_3', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
</tr> <tr>
<td class="main"><?php echo TEXT_PRODUCTS_SUBIMAGE2; ?> </td>
          <td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_subimage2') .
tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' .
$pInfo->products_subimage2 . tep_draw_hidden_field('products_previous_subimage2',
$pInfo->products_subimage2); ?>
                                                            </tr>
<?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage2) != '') {
    ?>

              <tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage2, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_5', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_5', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
</tr> <tr>
          <td class="main"><?php echo TEXT_PRODUCTS_SUBIMAGE3; ?></td>
          <td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_subimage3') .
tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' .
$pInfo->products_subimage3 . tep_draw_hidden_field('products_previous_subimage3',
$pInfo->products_subimage3); ?>
</tr>
<?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage3) != '') {
?>
<tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage3, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_7', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_7', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
</tr> <tr>
          <td class="main"><?php echo TEXT_PRODUCTS_SUBIMAGE4; ?></td>
          <td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_subimage4') .
tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' .
$pInfo->products_subimage4 . tep_draw_hidden_field('products_previous_subimage4', $pInfo->products_subimage4); ?>
</tr>

      <?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage4) != '') {
?>
<tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage4, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_9', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_9', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
</tr> <tr>
<td class="main"><?php echo TEXT_PRODUCTS_SUBIMAGE5; ?></td>
<td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_subimage5') .
tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' .
$pInfo->products_subimage5 . tep_draw_hidden_field('products_previous_subimage5',$pInfo->products_subimage5); ?>

      </tr>
<?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage5) != '') {
?>
<tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage5, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_11', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_11', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
        </tr>
        <?php } ?>
        <tr>
          <td colspan="2"><?php echo tep_draw_separator('pixel_trans.gif', '1',
'10'); ?></td>
</tr> <tr>
<td class="main"><?php echo TEXT_PRODUCTS_SUBIMAGE6 ?></td>
<td class="main"><?php echo tep_draw_separator('pixel_trans.gif', '24',
'15') . '&nbsp;' . tep_draw_file_field('products_subimage6') .
tep_draw_separator('pixel_trans.gif', '24', '15') . '&nbsp;' .
$pInfo->products_subimage6 . tep_draw_hidden_field('products_previous_subimage6',

   $pInfo->products_subimage6); ?>
        </tr>
<?php if (($HTTP_GET_VARS['pID']) && ($pInfo->products_subimage6) != '') {
?>
<tr>
          <td class="main"></td>
          <td class="main"><table border="0" cellspacing="0" cellpadding="0">
            <tr>
             <td class="main"><?php echo tep_draw_separator('pixel_trans.gif',
'24', '17" align="left') . tep_image(DIR_WS_CATALOG_IMAGES .
$pInfo->products_subimage6, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="left" hspace="0" vspace="5"'); ?></td>
             <td class="main"><?php
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('unlink_image_13', 'yes', false) .
TEXT_PRODUCTS_IMAGE_REMOVE . '<br>';
               echo tep_draw_separator('pixel_trans.gif', '5', '15') .
tep_draw_checkbox_field('delete_image_13', 'yes', false) .
TEXT_PRODUCTS_IMAGE_DELETE . '<br>';
             ?></td>
            </tr>
          </table></td>
</tr>
        <?php } ?>
<!-- EOF More Pics -->
```

查找:

```php
$product_query = tep_db_query("select p.products_id, pd.language_id, pd.products_name, pd.products_description, pd.products_url, p.products_quantity, p.products_model, p.products_image, p.products_price, p.products_weight, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_id = pd.products_id and p.products_id = '" . (int)$HTTP_GET_VARS['pID'] . "'");
```

替换为:

```php
// BOF: More Pics 6 Added: , p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6
$product_query = tep_db_query("select p.products_id, pd.language_id, pd.products_name, pd.products_description, pd.products_url, p.products_quantity, p.products_model, p.products_image, p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6, p.products_price, p.products_weight, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_id = pd.products_id and p.products_id = '" . (int)$HTTP_GET_VARS['pID'] . "'");
// EOF: More Pics 6
```

查找:

```php
$pInfo = new objectInfo($product);
$products_image_name = $pInfo->products_image;
```

在其下添加以下代码:

```php
// BOF: More Pics 6
$products_subimage1_name = $pInfo->products_subimage1;
$products_subimage2_name = $pInfo->products_subimage2;
$products_subimage3_name = $pInfo->products_subimage3;
$products_subimage4_name = $pInfo->products_subimage4;
$products_subimage5_name = $pInfo->products_subimage5;
$products_subimage6_name = $pInfo->products_subimage6;
// EOF: More Pics 6
```

查找:

```php
<td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES . $products_image_name, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') . $pInfo->products_description; ?></td>
```

替换为:

```php
<?php
       if (tep_not_null($products_image_name) && $HTTP_POST_VARS['delete_image']
!='yes' && $HTTP_POST_VARS['unlink_image'] !='yes') {
?>
<td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES . $products_image_name, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') . $pInfo->products_description; ?></td>
    <?php } ?>
<?php // BOF: More Pics 6 ?>
     <tr>
       <td class="main">
<table> <tr>
<?php
       if (tep_not_null($products_subimage1_name) &&
$HTTP_POST_VARS['delete_image_3'] !='yes' && $HTTP_POST_VARS['unlink_image_3']
!='yes') {
?>
      <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .
$products_subimage1_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
    <?php } ?>
<?php
if (tep_not_null($products_subimage2_name) && $HTTP_POST_VARS['delete_image_5'] !='yes' && $HTTP_POST_VARS['unlink_image_5'] !='yes') {
?>
        <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .
$products_subimage2_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
    <?php } ?>
<?php

      if (tep_not_null($products_subimage3_name) && $HTTP_POST_VARS['delete_image_7'] !='yes' && $HTTP_POST_VARS['unlink_image_7'] !='yes') {
?>
        <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .
$products_subimage3_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
    <?php } ?>
      </tr>
<tr>
<?php
if (tep_not_null($products_subimage4_name) && $HTTP_POST_VARS['delete_image_9']
!='yes' && $HTTP_POST_VARS['unlink_image_9'] !='yes') {
?>
       <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .
$products_subimage4_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
        <?php } ?>
<?php
if (tep_not_null($products_subimage5_name) && $HTTP_POST_VARS['delete_image_11'] !='yes' && $HTTP_POST_VARS['unlink_image_11'] !='yes') {
?>
        <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .
$products_subimage5_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
    <?php } ?>
<?php
if (tep_not_null($products_subimage6_name) && $HTTP_POST_VARS['delete_image_13'] !='yes' && $HTTP_POST_VARS['unlink_image_13'] !='yes') {
?>
        <td class="main"><?php echo tep_image(DIR_WS_CATALOG_IMAGES .

   $products_subimage6_name, $pInfo->products_name, SMALL_IMAGE_WIDTH,
SMALL_IMAGE_HEIGHT, 'align="right" hspace="5" vspace="5"') ?></td>
    <?php } ?>
      </tr>
<?php
   if (tep_not_null(products_subimage1)) {
?>
</table>
<?php }
?> </td>
     </tr>
<?php // EOF: More Pics 6 ?>
```

查找:

```php
echo tep_draw_hidden_field('products_image', stripslashes($products_image_name));
```

在其下添加:

```php
// BOF: More Pics 6
echo tep_draw_hidden_field('products_subimage1', stripslashes($products_subimage1_name));
echo tep_draw_hidden_field('products_subimage2', stripslashes($products_subimage2_name));
echo tep_draw_hidden_field('products_subimage3', stripslashes($products_subimage3_name));
echo tep_draw_hidden_field('products_subimage4', stripslashes($products_subimage4_name));
echo tep_draw_hidden_field('products_subimage5', stripslashes($products_subimage5_name));
echo tep_draw_hidden_field('products_subimage6', stripslashes($products_subimage6_name));
// BOF: More Pics 6
```

查找:

```php
  $products_query = tep_db_query("select p.products_id, pd.products_name, p.products_quantity, p.products_image, p.products_price, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status, p2c.categories_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and p.products_id = p2c.products_id and pd.products_name like '%" . tep_db_input($search) . "%' order by pd.products_name");
} else {
  $products_query = tep_db_query("select p.products_id, pd.products_name, p.products_quantity, p.products_image, p.products_price, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and p.products_id = p2c.products_id and p2c.categories_id = '" . (int)$current_category_id . "' order by pd.products_name");
```

替换为:

```php
// BOF: More Pics 6 Added: , p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6
  $products_query = tep_db_query("select p.products_id, pd.products_name, p.products_quantity, p.products_image, p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6, p.products_price, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status, p2c.categories_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and p.products_id = p2c.products_id and pd.products_name like '%" . tep_db_input($search) . "%' order by pd.products_name"); // EOF: More Pics 6
} else {
// BOF: More Pics 6 Added: , p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6
  $products_query = tep_db_query("select p.products_id, pd.products_name, p.products_quantity, p.products_image, p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6, p.products_price, p.products_date_added, p.products_last_modified, p.products_date_available, p.products_status from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and p.products_id = p2c.products_id and p2c.categories_id = '" . (int)$current_category_id . "' order by pd.products_name"); 
// EOF: More Pics 6
```

查找:

```php
$contents[] = array('text' => '<br>' . TEXT_EDIT_CATEGORIES_NAME .
$category_inputs_string);
$contents[] = array('text' => '<br>' . tep_image(DIR_WS_CATALOG_IMAGES . $cInfo->categories_image, $cInfo->categories_name) . '<br>' . DIR_WS_CATALOG_IMAGES . '<br><b>' . $cInfo->categories_image . '</b>');
$contents[] = array('text' => '<br>' . TEXT_EDIT_CATEGORIES_IMAGE . '<br>' . tep_draw_file_field('categories_image'));
```

在其下添加:

```php
// BOF: More Pics 6
$contents[] = array('text' => '<br>' . tep_draw_checkbox_field('delete_category_image', 'yes', false) . TEXT_DELETE_IMAGE);
// EOF: More Pics 6
```

查找:

```php
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_image, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_image);
```

在其下添加:

```php
// BOF: More Pics 6
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage1, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage1);
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage2, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage2);
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage3, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage3);
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage4, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage4);
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage5, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage5);
$contents[] = array('text' => '<br>' . tep_info_image($pInfo->products_subimage6, $pInfo->products_name, SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '<br>' . $pInfo->products_subimage6);
// EOF: More Pics 6
```

6、修改文件:catalog/admin/includes/functions/general.php: 

查找:

```php
function tep_remove_product($product_id) {
```

在其下添加:

```php
// BOF: More Pics 6
    $product_subimage1_query = tep_db_query("select products_subimage1 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage1 = tep_db_fetch_array($product_subimage1_query);
$duplicate_subimage1_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage1 = '" .
tep_db_input($product_subimage1['products_subimage1']) . "'");
        $duplicate_subimage1 = tep_db_fetch_array($duplicate_subimage1_query);
        if ($duplicate_subimage1['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage1['products_subimage1'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage1['products_subimage1']);
         }
}
    $product_subimage2_query = tep_db_query("select products_subimage2 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage2 = tep_db_fetch_array($product_subimage2_query);
$duplicate_subimage2_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage2 = '" . tep_db_input($product_subimage2['products_subimage2']) . "'");
        $duplicate_subimage2 = tep_db_fetch_array($duplicate_subimage2_query);
        if ($duplicate_subimage2['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage2['products_subimage2'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage2['products_subimage2']);
         }
}
    $product_subimage3_query = tep_db_query("select products_subimage3 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage3 = tep_db_fetch_array($product_subimage3_query);

      $duplicate_subimage3_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage3 = '" . tep_db_input($product_subimage3['products_subimage3']) . "'");
        $duplicate_subimage3 = tep_db_fetch_array($duplicate_subimage3_query);
        if ($duplicate_subimage3['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage3['products_subimage3'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage3['products_subimage3']);
         }
}
    $product_subimage4_query = tep_db_query("select products_subimage4 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage4 = tep_db_fetch_array($product_subimage4_query);
$duplicate_subimage4_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage4 = '" . tep_db_input($product_subimage4['products_subimage4']) . "'");
        $duplicate_subimage4 = tep_db_fetch_array($duplicate_subimage4_query);
        if ($duplicate_subimage4['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage4['products_subimage4'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage4['products_subimage4']);
         }
}
    $product_subimage5_query = tep_db_query("select products_subimage5 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage5 = tep_db_fetch_array($product_subimage5_query);

        $duplicate_subimage5_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage5 = '" . tep_db_input($product_subimage5['products_subimage5']) . "'");
        $duplicate_subimage5 = tep_db_fetch_array($duplicate_subimage5_query);
        if ($duplicate_subimage5['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage5['products_subimage5'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage5['products_subimage5']);
         }
}
    $product_subimage6_query = tep_db_query("select products_subimage6 from " .
TABLE_PRODUCTS . " where products_id = '" . tep_db_input($product_id) . "'");
        $product_subimage6 = tep_db_fetch_array($product_subimage6_query);
$duplicate_subimage6_query = tep_db_query("select count(*) as total from " . TABLE_PRODUCTS . " where products_subimage6 = '" . tep_db_input($product_subimage6['products_subimage6']) . "'");
        $duplicate_subimage6 = tep_db_fetch_array($duplicate_subimage6_query);
        if ($duplicate_subimage6['total'] < 2) {
         if (file_exists(DIR_FS_CATALOG_IMAGES .
$product_subimage6['products_subimage6'])) {
            @unlink(DIR_FS_CATALOG_IMAGES .
$product_subimage6['products_subimage6']);
         }
    }
  // EOF: More Pics 6
```
                                                             
7、修改文件:catalog/admin/includes/languages/english/categories.php

查找:

```php
define('TEXT_PRODUCTS_IMAGE', 'Products Image:');
```

在其下添加:

```php
// BOF: More Pics 6
define('TEXT_PRODUCTS_SUBIMAGE1', 'Products Image1:');
define('TEXT_PRODUCTS_SUBIMAGE2', 'Products Image2:');
define('TEXT_PRODUCTS_SUBIMAGE3', 'Products Image3:');
define('TEXT_PRODUCTS_SUBIMAGE4', 'Products Image4:');
define('TEXT_PRODUCTS_SUBIMAGE5', 'Products Image5:');
define('TEXT_PRODUCTS_SUBIMAGE6', 'Products Image6:');
define('TEXT_DELETE_IMAGE', 'Remove Image');
define('TEXT_PRODUCTS_IMAGE_REMOVE', 'Remove this Image from this Product?');
define('TEXT_PRODUCTS_IMAGE_DELETE', 'Delete this Image from the Server
(Permanent!)?');
// EOF: More Pics 6
```

8、修改文件:catalog/popup_image.php 

查找:

```php
$products_query = tep_db_query("select pd.products_name, p.products_image from " . TABLE_PRODUCTS . " p left join " . TABLE_PRODUCTS_DESCRIPTION . " pd on p.products_id = pd.products_id where p.products_status = '1' and p.products_id = '" . (int)$HTTP_GET_VARS['pID'] . "' and pd.language_id = '" . (int)$languages_id . "'");
$products = tep_db_fetch_array($products_query);
```

在其下添加:

```php
// BOF: More Pics 6
$PID = $HTTP_GET_VARS['pID'];
$invis = $HTTP_GET_VARS['invis'];
$result = mysql_query("select * from " . TABLE_PRODUCTS . " where products_id = '" . (int)$HTTP_GET_VARS['pID'] . "'");
// EOF: More Pics 6
```

查找:

```php
function resize() {
  if (navigator.appName == 'Netscape') i=40;
  if (document.images[0]) window.resizeTo(document.images[0].width +30, document.images[0].height+60-i);
  self.focus();
}
```

替换为:

```php
function resize() {
<?php // BOF: More Pics 6 ?>
  if (document.layers) i=40;
  if (document.images[0]) window.resizeTo(document.images[0].width +30, document.images[0].height+160-i);
<?php // EOF: More Pics 6 ?>
  self.focus();
}
```

查找:

```php
</head>
```

在其下添加:

```php
<?php // BOF: More Pics 6 ?>
<meta http-equiv="Page-Enter" content="blendTrans(Duration=0.5)">
<meta http-equiv="Page-Exit" content="blendTrans(Duration=0.5)">
<?php // EOF: More Pics 6 ?>
```

查找:

```php
<?php echo tep_image(DIR_WS_IMAGES . $products['products_image'],
$products['products_name']); ?>
```

替换为:

```php
<?php // BOF: More Pics 6 ?>
<table border="0" cellpadding="0" cellspacing="0" align="center">
     <?php // Lets find the last available image !
$image = tep_db_fetch_array($result);
if ($image['products_subimage6'] != ''){
$last = '6';
}elseif ($image['products_subimage5'] != ''){
$last = '5';
}elseif ($image['products_subimage4'] != ''){
$last = '4';
}elseif ($image['products_subimage3'] != ''){
$last = '3';
}elseif ($image['products_subimage2'] != ''){
$last = '2';
}elseif ($image['products_subimage1'] != ''){
$last = '1';
}elseif ($image['products_image'] != ''){
$last = '0';
}
$next = $invis + '1';
$back = $invis - '1';
?>
<?php
if (($invis == '0') || ($invis == '')){
$insert = $image['products_image'];
} else {
$insert = $image['products_subimage' . $invis. ''];
}
/*
//
// for use if you want to define a maximum width and height for the large popup images. //
$max_width=0;
$max_height=0;

      $img = DIR_WS_IMAGES . $insert;
list($width, $height, $type, $attr) = getimagesize($img);
if ($max_width!=0 && $max_width<$width && $max_height!=0 && $max_height<$height) {
  if (($max_width-$width)>($max_height-$height)) {
   $width = $max_width;
   $height = 0;
  } else {
   $width = 0;
   $height = $max_height;
  }
} elseif ($max_width!=0 && $max_width<$width) {
  $width = $max_width;
  $height = 0;
} elseif ($max_height!=0 && $max_height<$height) {
$width = 0;
  $height = $max_height;
}
echo '<tr><td align="center"><img src="' . $img . '"' . (($width!=0)?' width="'.$width.'"':'') . (($height!=0)?' height="'.$height.'"':'') . '></td>'; */
//
// to use the above code, you must remove the next two lines.
//
$img = DIR_WS_IMAGES . $insert;
echo '<tr><td align="center"><img src="' . $img . '"></td>';
?> </tr>
<tr>
   <td height="0" align="center"></td></tr>
<tr>
   <td height="20" align="center">
<?php
if (($back != '-1') || ($next <= $last)) {
  echo '<hr color="#666666" size="3">';
}
if ($back != '-1'){
echo '<a href="'.tep_href_link('popup_image.php','pID='.$PID.'&invis='.$back).'">' . tep_image(DIR_WS_IMAGES.'left.gif', 'previous', '', '', 'border="0"') . '</a> '; }
if ($next <= $last){
echo '<a href="'.tep_href_link('popup_image.php','pID='.$PID.'&invis='. $next).'">' . tep_image(DIR_WS_IMAGES.'right.gif', 'next', '', '', 'border="0"') . '</a>';
}
echo '</td></tr>';
?>
</table>
<?php // EOF: More Pics 6 ?>
```
                                                                   
替换为:

```php
```

9、修改文件:catalog/product_info.php

查找:

```php
$product_info_query = tep_db_query("select p.products_id, pd.products_name, pd.products_description, p.products_model, p.products_quantity, p.products_image, pd.products_url, p.products_price, p.products_tax_class_id, p.products_date_added, p.products_date_available, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and pd.products_id = p.products_id and pd.language_id = '" . (int)$languages_id . "'");
```

替换为：

```php
// BOF: More Pics 6 Added: , p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6
   $product_info_query = tep_db_query("select p.products_id, pd.products_name,
pd.products_description, p.products_model, p.products_quantity, p.products_image,
p.products_subimage1, p.products_subimage2, p.products_subimage3, p.products_subimage4, p.products_subimage5, p.products_subimage6, pd.products_url, p.products_price, p.products_tax_class_id, p.products_date_added, p.products_date_available, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and pd.products_id = p.products_id and pd.language_id = '" . (int)$languages_id . "'");
// EOF: More Pics 6
```

查找:

```php
if (tep_not_null($product_info['products_model'])) {
  $products_name = $product_info['products_name'] . '<br><span class="smallText">[' . $product_info['products_model'] . ']</span>';
} else {
  $products_name = $product_info['products_name'];
}
```

在其下添加:

```php
// BOF: More Pics 6
   $mopics_image_width =
(MOPICS_RESTRICT_IMAGE_SIZE=='true'?SMALL_IMAGE_WIDTH:'');
   $mopics_image_height =
(MOPICS_RESTRICT_IMAGE_SIZE=='true'?SMALL_IMAGE_HEIGHT:'');
   if (MOPICS_SHOW_ALL_ON_PRODUCT_INFO=='true') {
     $mopics_output = '';
     $mo_row = 1;
     $mo_col = 1;
     $mopics_images = array();
if (tep_not_null($product_info['products_image']) && MOPICS_GROUP_WITH_PARENT == 'true') { $mopics_images[] = $product_info['products_image']; }
     for ( $mo_item=1; $mo_item<7; $mo_item++ ) {

             if (tep_not_null($product_info['products_subimage'.$mo_item])) {
$mopics_images[] = $product_info['products_subimage'.$mo_item]; }
     }
     $mopics_count = sizeof($mopics_images);
     if ($mopics_count > 0) {
       $mopics_output .= '<table border="0" cellspacing="0" cellpadding="6"
align="'.MOPICS_TABLE_ALIGNMENT.'">';
       for ( $mo_item=0; $mo_item<$mopics_count; $mo_item++ ) {
        if ($mo_row<(MOPICS_NUMBER_OF_ROWS+1)) {
       if ($mo_col==1) {$mopics_output.='<tr>'."\n";}
$mopics_output .= ' <td align="center" class="smallText"><script language="javascript"><!--
             document.write(\'<a href="javascript:popupWindow(\\\'' .
tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' .
$product_info['products_id'].'&invis='.(MOPICS_GROUP_WITH_PARENT=='true'?$mo_item
:($mo_item+1))).'\\\')">' . tep_image(DIR_WS_IMAGES . $mopics_images[$mo_item],
addslashes($product_info['products_name']),
(MOPICS_RESTRICT_PARENT=='false'&&$mo_item==0&&MOPICS_GROUP_WITH_PARENT=='true'?'
':$mopics_image_width),
(MOPICS_RESTRICT_PARENT=='false'&&$mo_item==0&&MOPICS_GROUP_WITH_PARENT=='true'?'
':$mopics_image_height), 'hspace="5" vspace="5"') . '<br><img border=0
src=images/zoom.gif></a>\');
//--></script><noscript>
<a href="' . tep_href_link(DIR_WS_IMAGES . $mopics_images[$mo_item])
. '" target="_blank">' . tep_image(DIR_WS_IMAGES . $mopics_images[$mo_item],
$product_info['products_name'], $mopics_image_width, $mopics_image_height,
'hspace="5" vspace="5"') . '<br><img border=0 src=images/zoom.gif></a>
               </noscript></td>'."\n";

               if ($mo_col==MOPICS_NUMBER_OF_COLS) { $mo_col=1; $mo_row++;
$mopics_output.='</tr>'."\n"; } else { $mo_col++; }
} }
       if ($mo_col!=1){ while (($mo_col++)<(MOPICS_NUMBER_OF_COLS+1)) {
$mopics_output.='<td>&nbsp;</td>'; } $mopics_output.='</tr>'."\n"; }
       $mopics_output .= '</table>'."\n";
      }
   }
// EOF: More Pics 6
```

查找:

```php
if (tep_not_null($product_info['products_image'])) {
?>
        <table border="0" cellspacing="0" cellpadding="2" align="right">
          <tr>
            <td align="center" class="smallText">
<script language="javascript"><!--
document.write('<?php echo '<a href="javascript:popupWindow(\\\'' . tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' . $product_info['products_id']) . '\\\')">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], addslashes($product_info['products_name']), SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>'); //--></script>
<noscript>
<?php echo '<a href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>
```

替换为:

```php
// BOF: More Pics 6 ADDED to if statement: && MOPICS_GROUP_WITH_PARENT == 'false' if (tep_not_null($product_info['products_image']) && MOPICS_GROUP_WITH_PARENT ==
'false') { ?>
        <table border="0" cellspacing="0" cellpadding="2" align="right">
          <tr>
            <td align="center" class="smallText">
<script language="javascript"><!--
document.write('<?php echo '<a href="javascript:popupWindow(\\\'' . tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' . $product_info['products_id']) . '\\\')">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], addslashes($product_info['products_name']), (MOPICS_RESTRICT_PARENT=='false'?'':SMALL_IMAGE_WIDTH), (MOPICS_RESTRICT_PARENT=='false'?'':SMALL_IMAGE_HEIGHT), 'hspace="5" vspace="5"') . '<br><img border=0 src=images/zoom.gif></a>'; ?>');
//--></script>
<noscript>
<?php echo '<a href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], (MOPICS_RESTRICT_PARENT=='false'?'':SMALL_IMAGE_WIDTH), (MOPICS_RESTRICT_PARENT=='false'?'':SMALL_IMAGE_HEIGHT), 'hspace="5" vspace="5"') . '<br><img border=0 src=images/zoom.gif></a>'; ?>
<?php // EOF: More Pics 6 ?>
```

查找:

```php
?>
  <p><?php echo stripslashes($product_info['products_description']); ?></p>
      <?php
   $products_attributes_query = tep_db_query("select count(*) as total from " .
TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_ATTRIBUTES . " patrib where patrib.products_id='" . (int)$HTTP_GET_VARS['products_id'] . "' and patrib.options_id = popt.products_options_id and popt.language_id = '" . (int)$languages_id . "'");
```

在其下添加:

```php
// BOF: More Pics 6
    if (MOPICS_TABLE_LOCATION=='above' && !empty($mopics_output)) {
echo ' <table width="100%" border="0" cellspacing="0" cellpadding="0"> <tr>
            <td align="center" class="smallText">'.$mopics_output.'</td>
          </tr>
        </table>
         &nbsp;<br>'."\n";
    } else if (MOPICS_TABLE_LOCATION=='sides' && !empty($mopics_output)) {
      echo $mopics_output;
    }
// EOF: More Pics 6
```

查找:

```php
<tr>
            <td class="main"><?php echo
$products_options_name['products_options_name'] . ':'; ?></td>
            <td class="main"><?php echo tep_draw_pull_down_menu('id[' .
$products_options_name['products_options_id'] . ']', $products_options_array,
$selected_attribute); ?></td>
</tr> <?php
} ?>
        </table>
<?php
}
```

在其下添加:

```php
// BOF: More Pics 6
    if (MOPICS_TABLE_LOCATION=='below' && !empty($mopics_output)) {
      echo '&nbsp;<br>
        <table width="100%" border="0" cellspacing="0" cellpadding="0">
          <tr>
            <td align="center" class="smallText">'.$mopics_output.'</td>
          </tr>
        </table>'."\n";
    }
// EOF: More Pics 6
```
