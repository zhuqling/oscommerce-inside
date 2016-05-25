## 社交标签

社交网站与标签网站在 Web2.0 时代对于产品的宣传有非常重要的作用,Social Bookmarks 插件提供了最常用的社交标签网站的代码,所以我们选用它作为提供用户分享产品信息的工具。 

插件地址:http://addons.oscommerce.com/info/5261

[安装]

1、复制/includes/modules/social_bookmarks.php 到相应目录

2、复制/images/bookmark_icons 目录到“/images/”文件夹里

3、复制语言文件/includes/languages/LANGUAGE/social_bookmarks.php 到语言文件夹里,其中 LANGUAGE 为你当前使用的语言

4、修改文件:product_info.php 

查找:

```php
          <tr>
       <td align="center" class="smallText"><?php echo sprintf(TEXT_DATE_ADDED,
tep_date_long($product_info['products_date_added'])); ?></td>
     </tr>
<?php }
?>
```

在其下添加以下代码:

```php
<!-- social bookmarks start //-->
<?php
include(DIR_WS_MODULES . FILENAME_SOCIAL_BOOKMARKS);
?>
<!-- social bookmarks finish //-->
```

5、修改文件:catalog/includes/filenames.php 

添加以下定义:

```php
// Social Bookmark v2
define('FILENAME_SOCIAL_BOOKMARKS', 'social_bookmarks.php') ;
```
