## 可视化编辑器

产品描述信息需要灵活的文本编辑功能,这样才可以编辑出有特色的产品。目前使用的最普遍的 两个可视化编辑器要数:TinyMCE 和 FCKEditor,下面我们分别介绍在 osCommerce 后台如何嵌 入 TinyMCE 和 FCKEditor 编辑器。

### TinyMCE

TinyMCE Anywhere 插件可以很方便地在 osCommerce 用 TinyMCE 替换掉原始的内容输入框。 插件地址:http://addons.oscommerce.com/info/4852
 

[安装]

1、复制文件:

-  复制admin/includes/javascript/tiny_mce目录
-  复制admin/includes/configure.php配置文件到以下两个目录
  -  admin\includes\javascript\tiny_mce\plugins\ibrowser\config
  -  admin\includes\javascript\tiny_mce\plugins\ibrowser2\config

2、写权限;

赋予以下文件夹写权限

-  admin\includes\javascript\tiny_mce\plugins\ibrowser\temp
-  admin\includes\javascript\tiny_mce\plugins\ibrowser2\temp
-  admin\includes\javascript\tiny_mce\plugins\ibrowser\scripts\phpThumb\cache
-  admin\includes\javascript\tiny_mce\plugins\ibrowser2\scripts\phpThumb\cache

3、编辑产品时生效 TinyMCE,修改文件:admin/categories.php 

在“<head></head>”标签内添加:

```php
<?php // START tinyMCE Anywhere
if ($action == 'new_product') { // No need to put JS on all pages.
  $languages = tep_get_languages(); // Get all languages

  // Build list of textareas to convert
  for ($i = 0, $n = sizeof($languages); $i < $n; $i++) {
    $str.="products_description[".$languages[$i]['id']."],";
  } //end for each language

  $mce_str = rtrim ($str,","); // Removed the last comma from the string.

  // You can add more textareas to convert in the $str, be careful that they are all separated by a comma.
  echo '<script language="javascript" type="text/javascript"
src="includes/javascript/tiny_mce/tiny_mce.js"></script>';
  include "includes/javascript/tiny_mce/general.php";
} // END tinyMCE Anywhere ?>
```

4、Newsletter 使用 TinyMCE,修改文件:admin/newsletters.php 

在“<head></head>”标签内添加:

```php
<!-- START tinyMCE Anywhere //--> <script language="javascript"
type="text/javascript" src="includes/javascript/tiny_mce/tiny_mce.js"></script>
<?php include "includes/javascript/tiny_mce/mail.php"; // END tinyMCE ?>
```

5、邮件支持使用 TinyMCE,修改文件:admin/mail.php 

查找(约第 42 行):

`$mimemessage->add_text($message);`

替换为:

`$mimemessage->add_html($message);`

查找(约第 129 行):

`<td class="smallText"><b><?php echo TEXT_MESSAGE; ?></b><br><?php echo nl2br(htmlspecialchars(stripslashes($HTTP_POST_VARS['message']))); ?></td>`

替换为:

`<td class="smallText"><b><?php echo TEXT_MESSAGE; ?></b><br><?php echo(stripslashes($HTTP_POST_VARS['message'])); // For html email, otherwise nl2br(htlmlspecialchars?></td>`

6、其它地址使用 TinyMCE

在其它文件里使用 TinyMCE,只需要在文件的“<head></head>”标签内添加以下代码即可:

```php
$mce_str = YOURCODEHERE // Comma separated list of textarea names

// You can add more textareas to convert in the $mce_str, be careful that they are all separated by a comma.
echo '<script language="javascript" type="text/javascript" src="includes/javascript/tiny_mce/tiny_mce.js"></script>';
include "includes/javascript/tiny_mce/general.php";
```

### 使用FCKEditor

[安装] 

1、复制以下文件夹及其所有内容到对应目录: 

-  catalog/admin/fckeditor/
-  catalog/userfiles/

2、修改以下文件夹的权限为 777(Windows 系统里只需要确定文件夹非“只读”属性即可): 

-  catalog/userfiles/
-  catalog/userfiles/file
-  catalog/userfiles/media
-  catalog/userfiles/image 
-  catalog/userfiles/flash

3、修改文件:/admin/categories.php 

查找(约第 538 行)

```php
<td class="main"><?php echo tep_draw_textarea_field('products_description[' . $languages[$i]['id'] . ']', 'soft', '70', '15', (isset($products_description[$languages[$i]['id']]) ? $products_description[$languages[$i]['id']] : tep_get_products_description($pInfo->products_id, $languages[$i]['id']))); ?></td>
```

替换为:

```php
// 600是编辑器的宽度,300是编辑器的高度
<td class="main"><?php echo tep_draw_fckeditor('products_description[' . $languages[$i]['id'] . ']','600','300',(isset($products_description[$languages[$i]['id']]) ? stripslashes($products_description[$languages[$i]['id']]) : tep_get_products_description($pInfo->products_id, $languages[$i]['id']))); ?></td>
```

4、编辑文件:/admin/includes/functions/html_output.php 

查找(约第 13-14 行)

```php
////
// The HTML href link wrapper function
```

在其上面添加以上代码:

`require("fckeditor/fckeditor.php");`

查找(约第 235-253 行):

```php
////
// Output a form textarea field
function tep_draw_textarea_field($name, $wrap, $width, $height, $text = '',
$parameters = '', $reinsert_value = true) {
  $field = '<textarea name="' . tep_output_string($name) . '" wrap="' . tep_output_string($wrap) . '" cols="' . tep_output_string($width) . '" rows="' . tep_output_string($height) . '"';

  if (tep_not_null($parameters)) $field .= ' ' . $parameters;

  $field .= '>';
  if ( (isset($GLOBALS[$name])) && ($reinsert_value == true) ) {
    $field .= tep_output_string_protected(stripslashes($GLOBALS[$name]));
  } elseif (tep_not_null($text)) {
    $field .= tep_output_string_protected($text);
  }

  $field .= '</textarea>';
  return $field;
}
```

在其下添加以下代码:

```php
////
// Output a form textarea field w/ fckeditor
function tep_draw_fckeditor($name, $width, $height, $text) {
  $oFCKeditor = new FCKeditor($name);
  $oFCKeditor -> Width = $width;
  $oFCKeditor -> Height = $height;
  $oFCKeditor -> BasePath= 'fckeditor/';
  $oFCKeditor -> Value = $text;
  
  $field = $oFCKeditor->Create($name);
  return $field;
}
```

5、修改文件:admin/fckeditor/editor/filemanager/connectors/php/config.php

编辑$Config['UserFilesAbsolutePath']的值,填入 userfiles 目录的物理地址。

`$Config['UserFilesAbsolutePath'] = '/此处填入userfiles目录的物理地址/' ;`

6、安装完成,你可以进入后台的 Administration -> Catalog -> Categories/Products 编辑或者新建产品查看效果了。
