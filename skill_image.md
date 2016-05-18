## 图片防盗链

如果直接将图片的地址显示出来,图片将极容易被其它人利用。如果是自己辛苦拍摄出来的图片, 实现图片防盗链则显得更加重要了。

要实现图片防盗需要实现两个方面的内容:一是隐藏图片真实地址,二是大图片自动添加水印或者限制显示图片的最大尺寸。

代码如下:

新建文件:get_image.php

```php
<?php
require_once 'includes/app_top.php';

if(isset($_GET['id']) && $_GET['id']>0){ // 参数指定需要显示图片的产品ID 
  $sqlProduct = sprintf(‘SELECT * FROM %s WHERE products_id=%d’,TABLE_PRODUCTS,(int)$_GET[‘id’]);

  $resourceProduct = tep_db_query($sqlProduct);
  $detail = tep_db_fetch_array($resourceProduct); if(!isset($detail['products_image'])) die('Image Error.');

  $index = 1;
  if(isset($_GET['img'])) $index = (int)$_GET['img']; //参数img指定需要第几张图片(
  需要安装图片扩展插件)

  $pic =((1>=$index)?$detail['products_image']:$detail['products_image_'.$index]);
} else {
  die(‘Product ID Error.’);
}

if (isset($pic) && file_exists($pic)) {
  $imageHandle = @imagecreatefromjpeg($pic);
  if(!$imageHandle) die(‘Create Image Resource Error.’);

  $width = imagesx($imageHandle);
  $height = imagesy($imageHandle);

  // 添加水印
  $fileWaterFall = 'images/waterfall.png';
  $imgWaterFall = imagecreatefrompng($fileWaterFall);
  list($w,$h) = getimagesize($fileWaterFall);

  // 将水印放置在图片中间
  $x = ($width-$w)/2;
  $y = ($height-$h)/2;
  imagecopymerge($imageHandle,$imgWaterFall,$x,$y,0,0,$w,$h,50); // 参数 50 为透明系数

  imagedestroy($imgWaterFall);
  header("Content-type: image/jpeg");
  imagejpeg($imageHandle); // 输出图片
  imagedestroy($imageHandle);
}

require_once 'includes/app_bottom.php';
?>
```

调用方法:

`<img src=”get_image.php?id=1234&img=2”/>`

参数 id 是产品 ID,参数 img 是图片序号
