## Lightbox图片效果

Lightbox 是基于 prototype Javascript 框架的一款图片展示方案,由于它独特的图片展示效果和简单方便的使用方法,在 Blog、相册等各种场合被广泛运用。

### 标准Lightbox效果

此版本直接使用标准的 Lightbox 脚本,用户需要下载 Lightbox 脚本,并进行如下修改: 

1、解压 lightbox 插件,将 js 目录里的文件:prototype.js、effects.js、builder.js、scriptaculous.js、 lightbox.js 复制到网站 catalog 根目录下的 js 目录里,将 images 目录里的所有图片文件复制到 catalog 下的 images 目录里,再将 css 文件夹里的 lightbox.css 文件复制到 catalog 下的 css 目录里 (如果没有 css 目录则新建此目录)

2、修改产品页面 product_info.php

在<head>标签里添加如下代码:

```html
<script type="text/javascript" src="js/prototype.js"></script>
<script type="text/javascript"
src="js/scriptaculous.js?load=effects,builder"></script>
<script type="text/javascript" src="js/lightbox.js"></script>
         
<link rel="stylesheet" href="css/lightbox.css" type="text/css" media="screen" />
```

查找(约第 108 行):

```php
<script language="javascript"><!--
document.write('<?php echo '<a href="javascript:popupWindow(\\\'' . tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' . $product_info['products_id']) . '\\\')">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], addslashes($product_info['products_name']), SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>'); //--></script>
<noscript>
<?php echo '<a href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>
</noscript>
```

替换成如下代码:

```php
<?php echo '<a rel=”lightbox” href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>
```

### jQuery Lightbox插件

在这个版本里我们使用Lightbox jQuery with Ultra Pics插件,这个插件同时包含了Lightbox for jQuery(jQuery 版本的 lightbox)和 Ultra Pics(一款扩展产品多图片插件)两个插件,在此我们只使用 其中的 Lightbox for jQuery 一个插件。

[安装]

1、将 lightbox 文件夹复制到网站 catalog 根目录下,将 images/prettyPhoto 目录复制到网站 catalog/images/下。 

2、修改文件 product_info.php 

查找:

`<link rel="stylesheet" type="text/css" href="stylesheet.css">`

在其下添加以下代码:

```html     
<!-- Light Box J Query Add on BOF -->
<link rel="stylesheet" href="lightbox/prettyPhoto.css" type="text/css"
title="prettyPhoto main stylesheet" charset="utf-8" />
<script src="lightbox/jquery-1.2.3.pack.js" type="text/javascript"
charset="utf-8"></script>
<script src="lightbox/prettyPhoto.js" type="text/javascript"
charset="utf-8"></script>
<!-- Light Box J Quesry Add on EOF -->
```

查找:

```php
<script language="javascript"><!--
document.write('<?php echo '<a href="javascript:popupWindow(\\\'' . tep_href_link(FILENAME_POPUP_IMAGE, 'pID=' . $product_info['products_id']) . '\\\')">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], addslashes($product_info['products_name']), SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>'); //--></script>
<noscript>
<?php echo '<a href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>
</noscript>
```

替换成如下代码:

```php
<?php echo '<a rel="prettyOverlay[gallery]" href="' . tep_href_link(DIR_WS_IMAGES . $product_info['products_image']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $product_info['products_image'], $product_info['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT, 'hspace="5" vspace="5"') . '<br>' . TEXT_CLICK_TO_ENLARGE . '</a>'; ?>
```
