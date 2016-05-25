## 快速产品属性编辑

如果你使用过 osCommerce 后台的属性设置(product attribute),就会发现单独页面操作产品属性是 多么困难。我们将要介绍的插件是 Quick Attributes 快速产品属性编辑插件。这款插件可以在浏览 产品时,直接定位该产品的属性。并可以快速添加、修改、删除该产品属性。

插件地址:http://addons.oscommerce.com/info/6345
 

[安装]

1、复制语言文件:/catalog/admin/includes/languages/english/products_attributes_quick.php. 

2、复制主程序文件: /catalog/admin/products_attributes_quick.php 

3、复制图标文件:/catalog/admin/images/icon_quick_attribute.png

4、修改 categories 文件

文件: /catalog/admin/categories.php

查找:

```php
<td class="dataTableContent" align="right">&nbsp;
<?php if (isset($pInfo) && is_object($pInfo) && ($products['products_id'] == $pInfo->products_id)) { echo tep_image(DIR_WS_IMAGES . 'icon_arrow_right.gif', ''); } else { echo '<a href="' . tep_href_link(FILENAME_CATEGORIES, 'cPath=' . $cPath . '&pID=' . $products['products_id']) . '">' . tep_image(DIR_WS_IMAGES . 'icon_info.gif', IMAGE_ICON_INFO) . '</a>'; } ?>&nbsp;</td>
</tr>
```

替换为下面的代码:

```php
<td class="dataTableContent" align="right">&nbsp;
<?php if (isset($pInfo) && is_object($pInfo) && ($products['products_id'] == $pInfo->products_id)) { echo tep_image(DIR_WS_IMAGES . 'icon_arrow_right.gif', ''); } else { echo '<a href="' . tep_href_link(FILENAME_CATEGORIES, 'cPath=' . $cPath . '&pID=' . $products['products_id']) . '">' . tep_image(DIR_WS_IMAGES . 'icon_info.gif', IMAGE_ICON_INFO) . '</a>'; } ?>&nbsp; <?php
// BOF Quick Attributes
echo '<a href="' . tep_href_link(FILENAME_PRODUCTS_ATTRIBUTES_QUICK, 'pID=' . $products['products_id'].'&cPath='.$cPath, 'NONSSL') . '">' . tep_image(DIR_WS_IMAGES . 'icon_quick_attribute.png', IMAGE_ICON_QUICK_ATTRIBUTE) . '</a>';
// EOF Quick Attributes
?></td>
</tr>
```
   
5、打开 catalog/admin/includes/filenames.php 文件。

在文件的最后添加以下的定义:

```php
// BOF Quick Attributes
define('FILENAME_PRODUCTS_ATTRIBUTES_QUICK', 'products_attributes_quick.php');
// EOF Quick Attributes
```

6、打开语言文件:/catalog/admin/includes/languages/english.php. 

在结束符之前添加如下代码:

```php
// BOF Quick Attributes
define('IMAGE_ICON_QUICK_ATTRIBUTE', 'Add Quick Attribute');
// EOF Quick Attributes
```
