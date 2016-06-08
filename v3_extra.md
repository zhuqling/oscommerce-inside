## 产品扩展

osCommerce V3里的osC_Product类负责单个产品的资料收集,如对多图片的支持、对产品选项
的多类型支持、以及新增的重量单位等,都是由它全权管理的

### 图片与图片组

osC_Product 类 includes/classes/product.php[78]

```php
$this->_data['images'] = array();

$Qimages = $osC_Database->query('select id, image, default_flag from
:table_products_images where products_id = :products_id order by sort_order');
$Qimages->bindTable(':table_products_images', TABLE_PRODUCTS_IMAGES);
$Qimages->bindInt(':products_id', $this->_data['master_id']);
$Qimages->execute();

while ($Qimages->next()) {
  $this->_data['images'][] = $Qimages->toArray();
}
```

上面的代码出自 osC_Product 类的初始化方法,在加载了产品的基本资料后,紧接下来的便是这段代码。

代码首先通过查询 products_images 表,获得指定产品的所有图片,然后将图片资料保存到内部变量$_date[‘images’]里。

如何将产品图片全部显示出来,则需要模板的支持

templates/TEMPLATE/content/products/images.php[46]

```php
foreach ($osC_Product->getImages() as $images) {
  if ( isset($_GET['image']) && ($_GET['image'] == $images['id']) ) { // 判断是  否将指定图片放大
    $large_image = $osC_Image->show($images['image'], $osC_Product->getTitle(), 'id="productImageLarge"', 'large'); // osC_Image类的show方法负责显示图片
  }

  echo '<span style="width: ' . $osC_Image->getWidth($osC_Image->getCode(DEFAULT_IMAGE_GROUP_ID)) . 'px; padding: 2px; float: left; text-align: center;">' . osc_link_object(osc_href_link(FILENAME_PRODUCTS, 'images&' . $osC_Product->getKeyword() . '&image=' . $images['id']), $osC_Image->show($images['image'], $osC_Product->getTitle(), 'height="' . $osC_Image->getHeight($osC_Image->getCode(DEFAULT_IMAGE_GROUP_ID)) . '" style="max-width: ' . $osC_Image->getWidth($osC_Image->getCode(DEFAULT_IMAGE_GROUP_ID)) . 'px;"'), 'onclick="loadImage(\'' . $osC_Image->getAddress($images['image'], 'large') . '\'); return false;"') . '</span>';
  // 这段代码其实只是显示一个小图片,并且当图片被点击时会提交新 的参数,以便将此图片放大显示
}
```

images.php 负责显示产品的图片,osC_Product 类的 getImages 方法会返回当前产品的所有图片, 返回的结果是关联数组。

图片组的实现

includes/classes/image.php 的 osC_Image 类只负责前台图片的展示,这部分代码比较简单,在此不 详加描述,我们的问题是“图片是如何被自动保存到不同的图片组的呢?”

这要归功于 osC_Image_Admin 类,它位于“admin/includes/classes/image.php”。在对产品资料新建 或修改完成时会执行后台的动作类

osC_Application_Products_Actions_save,其中的一段代码如下:

osC_Application_Products_Actions_save 类

admin/includes/applications/products/action/save.php[114]

```php
if ( osC_Products_Admin::save((isset($_GET[$this->_module]) && is_numeric($_GET[$this->_module]) ? $_GET[$this->_module] : null), $data) ) {
```

它调用了 osC_Products_Admin 类的 save 方法,而 osC_Products_Admin 类的 save 方法中将图片保 存到数据库,以及真正形成图片组的代码如下:

osC_Products_Admin 类 admin/includes/appliations/products/classes/products.php[330]

```php
foreach ($images as $image) { // 首先将图片保存到数据库
  $Qimage = $osC_Database->query('insert into :table_products_images (products_id, image, default_flag, sort_order, date_added) values (:products_id, :image, :default_flag, :sort_order, :date_added)');
                      //
  $Qimage->bindTable(':table_products_images', TABLE_PRODUCTS_IMAGES);
  $Qimage->bindInt(':products_id', $products_id);
  $Qimage->bindValue(':image', $image);
  $Qimage->bindInt(':default_flag', $default_flag);
  $Qimage->bindInt(':sort_order', 0);
  $Qimage->bindRaw(':date_added', 'now()');
  $Qimage->setLogging($_SESSION['module'], $products_id);
  $Qimage->execute();

  if ( $osC_Database->isError() ) {
    $error = true;
  } else {
    foreach ($osC_Image->getGroups() as $group) { // 得到所有图片组
      if ($group['id'] != '1') {
        $osC_Image->resize($image, $group['id']); // osC_Image_Admin 类的 resize 方法负责调整图片尺寸,并进行按组保存
      }
    }
  }

  $default_flag = 0;
}
```
               
osC_Image_Admin 类的方法 resize 实际却是调用了 resizeWithGD 方法实现图片尺寸调整

```php
function resizeWithGD($image, $group_id) {
  $img_type = false;

  // 根据实际图片格式,自动调整新图片的格式
  switch (substr($image, (strrpos($image, '.')+1))) {
    case 'jpg':
    case 'jpeg':
      if (imagetypes() & IMG_JPG) {
        $img_type = 'jpg';
      }
      break;
    case 'gif':
      if (imagetypes() & IMG_GIF) {
        $img_type = 'gif';
      }
      break;
    case 'png':
      if (imagetypes() & IMG_PNG) {
        $img_type = 'png';
      }
      break;
  }

  if ($img_type !== false) {
    if (!file_exists(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'])) { // 确保图片组所在的文件夹一定存在
      mkdir(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'], 0777);
    }

    list($orig_width, $orig_height) = getimagesize(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[1]['code'] . '/' . $image);
    $height = $this->_groups[$group_id]['size_height'];
    if ($this->_groups[$group_id]['force_size'] == '1') {
      $width = $this->_groups[$group_id]['size_width'];
    } else {
      $width = round($orig_width * $height / $orig_height); // 按比例动态调整尺寸
    }

    $im_p = imagecreatetruecolor($width, $height);
    if ( ($img_type == 'gif') || ($img_type == 'png') ) {
      imagealphablending($im_p, false);
      // 对于 GIF/PNG 格式文件保留其透明背景的特性 
      imagesavealpha($im_p, true);
      $transparent = imagecolorallocatealpha($im_p, 255, 255, 255, 127);
      imagefilledrectangle($im_p, 0, 0, $height, $width, $transparent);
    }

    $x = 0;
    if ($this->_groups[$group_id]['force_size'] == '1') {
      if ( ($img_type != 'gif') && ($img_type != 'png') ) {
        $bgcolour = imagecolorallocate($im_p, 255, 255, 255); // white
        imagefill($im_p, 0, 0, $bgcolour);
      }
      $width = round($orig_width * $height / $orig_height);
      if ($width < $this->_groups[$group_id]['size_width']) {
        $x = floor(($this->_groups[$group_id]['size_width'] - $width) / 2);
      }
    }

    // 创建对应格式的图片类型 
    switch ($img_type) {
      case 'jpg':
        $im = imagecreatefromjpeg(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[1]['code'] . '/' . $image);
        break;
      case 'gif':
        $im = imagecreatefromgif(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[1]['code'] . '/' . $image);
        break;
      case 'png':
        $im = imagecreatefrompng(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[1]['code'] . '/' . $image);
        break;
    }

    // 复制原图片到新图片
    imagecopyresampled($im_p, $im, $x, 0, 0, 0, $width, $height, $orig_width, $orig_height);

    // 将新图片保存到文件
    switch ($img_type) {
      case 'jpg':
        imagejpeg($im_p, DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'] . '/' . $image);
        break;
      case 'gif':
        imagegif($im_p, DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'] . '/' . $image);
        break;
      case 'png':
        imagepng($im_p, DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'] . '/' . $image);
        break;
    }

    imagedestroy($im_p);
    imagedestroy($im);
    chmod(DIR_FS_CATALOG . DIR_WS_IMAGES . 'products/' . $this->_groups[$group_id]['code'] . '/' . $image, 0777);
  } else {
    return false;
  }
}
```

> 提示:
> 
> imagetypes 函数返回 PHP 支持的图片格式,如下面这段代码可检查 PHP 是否支持 PNG 格式:
> 
> ```php
> if (imagetypes() & IMG_PNG) {
>    echo "PNG Support is enabled";
> }
> ```


### 多样化的产品选项

osC_Product 类初始化产品选项 includes/classes/product.php[96]

```php
if ( $this->_data['has_children'] === 1 ) { // 判断是否拥有子产品
  $this->_data['variants'] = array(); // 将使用内部变量$_data['variants']保存该产品的选项

  // 载入所有子产品的基本信息
  $Qsubproducts = $osC_Database->query('select * from :table_products where parent_id = :parent_id and products_status = :products_status');
  $Qsubproducts->bindTable(':table_products', TABLE_PRODUCTS);
  $Qsubproducts->bindInt(':parent_id', $this->_data['master_id']);
  $Qsubproducts->bindInt(':products_status', 1);
  $Qsubproducts->execute();
  while ( $Qsubproducts->next() ) {
    $this->_data['variants'][$Qsubproducts->valueInt('products_id')]['data'] = array(
      'price' => $Qsubproducts->value('products_price'),
      'tax_class_id' => $Qsubproducts->valueInt('products_tax_class_id'), 
      'model' => $Qsubproducts->value('products_model'),
      'quantity' => $Qsubproducts->value('products_quantity'),
      'weight' => $Qsubproducts->value('products_weight'),
      'weight_class_id' => $Qsubproducts->valueInt('products_weight_class'),
      'availability_shipping' => 1
    );

    // 载入该产品所拥有的所有选项组及其它可用的值
    $Qvariants = $osC_Database->query('select pv.default_combo, pvg.id as group_id, pvg.title as group_title, pvg.module, pvv.id as value_id, pvv.title as value_title, pvv.sort_order as value_sort_order from :table_products_variants pv, :table_products_variants_groups pvg, :table_products_variants_values pvv where pv.products_id = :products_id and pv.products_variants_values_id = pvv.id and pvv.languages_id = :languages_id and pvv.products_variants_groups_id = pvg.id and pvg.languages_id = :languages_id order by pvg.sort_order, pvg.title');
    $Qvariants->bindTable(':table_products_variants', TABLE_PRODUCTS_VARIANTS);
    $Qvariants->bindTable(':table_products_variants_groups', TABLE_PRODUCTS_VARIANTS_GROUPS);
    $Qvariants->bindTable(':table_products_variants_values', TABLE_PRODUCTS_VARIANTS_VALUES);
    $Qvariants->bindInt(':products_id',
    $Qsubproducts->valueInt('products_id'));
    $Qvariants->bindInt(':languages_id', $osC_Language->getID());
    $Qvariants->bindInt(':languages_id', $osC_Language->getID());
    $Qvariants->execute();
    while ( $Qvariants->next() ) {
      $this->_data['variants'][$Qsubproducts->valueInt('products_id')]['values'][$Qvari ants->valueInt('group_id')][$Qvariants->valueInt('value_id')] = array(
        'value_id' => $Qvariants->valueInt('value_id'),// 选项值 ID
        'group_title' => $Qvariants->value('group_title'),// 选项组名,即选项名称 
        'value_title' => $Qvariants->value('value_title'),// 选项值的文本值 
        'sort_order' => $Qvariants->value('value_sort_order'),// 排序号 
        'default' => (bool)$Qvariants->valueInt('default_combo'),
        'module' => $Qvariants->value('module')
      ); // module为选项类名,用于实现不同选项类型的显 示方式不同
    }
  }
}
```

> 提示:
> 
> 从上面第一行代码,便可以得知,osCommerce V3 在处理产品选项方面采用了与 RC2.2 完全不同 的方式。在 V3 里产品是分主产品与子产品的,如果某个产品要实现属性选项的话,那么就会自 动从主产品派生出一个从属于它的子产品,这个子产品可以允许设置众多的选项、不同的重量、 拥有的库存数量、不同的价格、或者是独有的产品编号。
> 
> 如此 osCommerce V3 便可以满足实际销售过程中对于产品设置的所有要求,在这一方面相比 RC2.2 简单的属性设置,V3 版本显然要成熟、高明很多。

产品选项的显示

templates/default/content/products/info.php[70]

```php
foreach ( $osC_Product->getVariants() as $group_id => $value ) {
  echo osC_Variants::parse($value['module'], $value);
}
```

osC_Product 类的方法 getVariants 会返回该产品的所有选项资料,通过遍历所有选项,便可以根据 选项所属的类,调用其类的 parse 方法解析、并以特定的格式显示出该选项值。

获取指定产品的所有选项

osC_Product 类 includes/classes/product.php[356]

```php
function getVariants($filter_duplicates = true) {
  if ( $filter_duplicates === true ) { // 默认过滤重复的选项值
    $values_array = array();

    // 将层层包裹起来的选项按新的规范填入结果数组里
    foreach ( $this->_data['variants'] as $product_id => $variants ) {
      foreach ( $variants['values'] as $group_id => $values ) {
        foreach ( $values as $value_id => $value ) {
          if ( !isset($values_array[$group_id]) ) {
            $values_array[$group_id]['group_id'] = $group_id;
            $values_array[$group_id]['title'] = $value['group_title'];
            $values_array[$group_id]['module'] = $value['module'];
          }

          $value_exists = false; // 过滤相同的选项
          if ( isset($values_array[$group_id]['data']) ) {
            foreach ( $values_array[$group_id]['data'] as $data ) {
              if ( $data['id'] == $value_id ) {
                $value_exists = true;
                break;
              }
            }
          }

          if ( $value_exists === false ) {
            $values_array[$group_id]['data'][] = array(
              'id' => $value_id,
              'text' => $value['value_title'],
              'default' => $value['default'],
              'sort_order' => $value['sort_order']
            ); // 填入选项值

          } elseif ( $value['default'] === true ) {
            foreach ( $values_array[$group_id]['data'] as &$existing_data ) {
              if ( $existing_data['id'] == $value_id ) {
                $existing_data['default'] = true;
                break;
              }
            }
          }
        }
      }
    }

    // 重新排序,osC_Product 类的_usortVariantValues 方法负责处理排序,这里使用了自然排序
    foreach ( $values_array as $group_id => &$value ) {
      usort($value['data'], array('osC_Product', '_usortVariantValues'));
    }

    return $values_array;
  }

  return $this->_data['variants']; // 如果不需要过滤重复选项,则直接返回内部变量
  $_data[‘variants’]
}
```

osC_Variants 类的 parse 方法 includes/classes/variants.php[30]

```php
static public function parse($module, $data) {
  if ( !class_exists('osC_Variants_' . $module) ) {
    if ( file_exists(DIR_FS_CATALOG . 'includes/modules/variants/' . basename($module) . '.php') ) { // 导入相应的产品选项解析类
      include(DIR_FS_CATALOG . 'includes/modules/variants/' . basename($module) . '.php');
    }
  }
  
  if ( class_exists('osC_Variants_' . $module) ) {
    return call_user_func(array('osC_Variants_' . $module, 'parse'), $data); // 再由产品选项解析类调用其 parse 方法解析选项值
  }
}
```

下面我们举两个例子说明产品选项解析类是如何显示出产品选项的

首先是下拉框的解析类:

osC_Variants_pull_down_menu 类 

includes/modules/variants/pull_down_menu.php

```php
static public function parse($data) {
  $default_value = null;
  
  foreach ( $data['data'] as $variant ) {
    if ( $variant['default'] === true ) {
      $default_value = $variant['id'];//得到默认选项的ID break;
    }
  }

  // 用 osc_draw_pull_down_menu 函数直接输出选项的下边框,并通过$default_value 指定默认的选择 项
  $string = '<table border="0" cellspacing="0" cellpadding="2">' . ' <tr>' . ' <td width="100">' . $data['title'] . ':</td>' . ' <td>' . osc_draw_pull_down_menu('variants[' . $data['group_id'] . ']', $data['data'], $default_value, 'onchange="refreshVariants();" id="variants_' . $data['group_id'] . '"') . '</td>' .
' </tr>' . '</table>';

  return $string;
}
```

> 提示:
> 
> osc_draw_pull_down_menu 函数所附带的参数:refreshVariants 是一个 javascript 函数,位于 “templates/default/javascript/products/info.js”,该函数是为了实现实时价格更新,即当用户更改了 产品选项,产品的显示价格也会自动更新(增加或者减去所选选项的金额)

接下来是文本框的解析类:

osC_Variants_text_field includes/modules/variants/text_field.php

```php
static public function parse($data) {
  $string = '<table border="0" cellspacing="0" cellpadding="2">';
  $i = 0;
  
  foreach ( $data['data'] as $field ) {
    $i++;
    // 直接调用函数 osc_draw_input_field 输出一个文本框,供用户填写
    $string .= ' <tr>' . ' <td width="100">' . $field['text'] . ':</td>' . ' <td>' . osc_draw_input_field('variants[' . $data['group_id'] . '][' . $field['id'] . ']', null, 'id="variants_' . $data['group_id'] . '_' . $i . '"') . '</td>' . ' </tr>';
  }

  $string .= '</table>';
  return $string;
}
```

### 自定义属性

加载自定义属性 includes/classes/product.php[134]

```php
$this->_data['attributes'] = array();
// 通过查询 product_attributes 表与 templates_boxes 表得到所有自定义属性 

$Qattributes = $osC_Database->query('select tb.code, pa.value from :table_product_attributes pa, :table_templates_boxes tb where pa.products_id = :products_id and pa.languages_id in (0, :languages_id) and pa.id = tb.id');
$Qattributes->bindTable(':table_product_attributes');
$Qattributes->bindTable(':table_templates_boxes');
$Qattributes->bindInt(':products_id', $this->_data['master_id']);
$Qattributes->bindInt(':languages_id', $osC_Language->getID());
$Qattributes->execute();

while ( $Qattributes->next() ) {
  $this->_data['attributes'][$Qattributes->value('code')] = $Qattributes->value('value');
}
```

自定义属性的加载,与产品选项一样都是在 osC_Product 类里初始化的。产品所拥有的自定义属 性被保存在内部变量$_data[‘attributes’]里。

> 提示:
> 
> templates_boxes 表不仅保存着功能模块、内容框的设置信息,还保存着付款模块、运输模块、统 计模块以及产品自定义属性的信息,所以上面的代码通过将 templates_boxes 表与product_attributes 表关联起来,便可以查询到产品的自定义属性

自定义属性的显示

以下是 osCommerce V3 默认安装下的三个自定义属性显示代码 

(1) 供应商名称 templates/default/content/products/info.php[92]

```php
<?php
  if ( $osC_Product->hasAttribute('manufacturers') ) {
?>
  <tr>
   <td class="productInfoKey">Manufacturer:</td>
   <td class="productInfoValue"><?php echo
$osC_Product->getAttribute('manufacturers'); ?></td>
  </tr>
<?php } ?>
```

(2) 有效日期 templates/default/content/products/info.php [110]

```php
<?php
  if ( $osC_Product->hasAttribute('date_available') ) {
?>
  <tr>
   <td class="productInfoKey">Date Available:</td>
   <td class="productInfoValue"><?php echo
osC_DateTime::getShort($osC_Product->getAttribute('date_available')); ?></td>
  </tr>
<?php } ?>
```

(3) 允许发货状态 templates/default/content/products/info.php [132]

```php
var productInfoAvailability = '<?php if ( $osC_Product->hasAttribute('shipping_availability') ) {
  echo addslashes($osC_Product->getAttribute('shipping_availability'));
} ?>';
```

可以看到要显示产品自定义属性,先要调用 osC_Product 类的 hasAttribute 方法,查询是否存在需 要显示的自定义属性值,如果有则,使用 getAttribute 方法将自定义属性值显示出来,下面是 osC_Product 类的 getAttribute 方法的代码:

osC_Product 类的 getAttribute 方法 includes/classes/product.php[447]

```php
function getAttribute($code) {
  if ( !class_exists('osC_ProductAttributes_' . $code) ) {
    if ( file_exists(DIR_FS_CATALOG . 'includes/modules/product_attributes/' . basename($code) . '.php') ) {
      include(DIR_FS_CATALOG . 'includes/modules/product_attributes/' .
      basename($code) . '.php');
    }
  }

  if ( class_exists('osC_ProductAttributes_' . $code) ) {
    return call_user_func(array('osC_ProductAttributes_' . $code, 'getValue'),   $this->_data['attributes'][$code]);
  }
}
```

产品自定义属性类保存在“includes/modules/product_attributes/”目录,它们的类名规则是: “osC_ProductAttributes_“+CODE,其中 CODE 为 template_boxes 表里保存的字段“code”。 

当加载好自定义属性类后,便将显示的任务交给它的 getValue 方法去执行,同时将内部变量 $_data[‘attributes’][CODE]的值作为参数传递进去。

下面三个便是默认安装下的产品自定义属性类的代码

(1) 供应商类 includes/modules/product_attributes/manufacturers.php

```php
class osC_ProductAttributes_manufacturers {
  static public function getValue($value) {
    global $osC_Database; // 通过供应商 ID 查询到供应商名称
    
    $Qmanufacturer = $osC_Database->query('select manufacturers_name from :table_manufacturers where manufacturers_id = :manufacturers_id');
    $Qmanufacturer->bindTable(':table_manufacturers');
    $Qmanufacturer->bindInt(':manufacturers_id', $value);
    $Qmanufacturer->execute();
    if ( $Qmanufacturer->numberOfRows() === 1 ) {
      return $Qmanufacturer->value('manufacturers_name'); // 返回供应商名称
    }
  }
}
```

(2) 有效日期类 includes/modules/product_attributes/date_available.php

```php
class osC_ProductAttributes_date_available {
  static public function getValue($value) {
    return $value; //因为保存的便是有效日期,所以可以直接返回此数据 
  }
}
```

(3) 允许发货状态类 includes/modules/product_attributes/shipping_availability.php

```php
class osC_ProductAttributes_shipping_availability {
  static public function getValue($value) {
    global $osC_Database, $osC_Language;

    $string = '';
    // 查询 shipping_availability 表,取得发货方式的名称与样式
    $Qstatus = $osC_Database->query('select title, css_key from :table_shipping_availability where id = :id and languages_id = :languages_id');
    $Qstatus->bindTable(':table_shipping_availability');
    $Qstatus->bindInt(':id', $value);
    $Qstatus->bindInt(':languages_id', $osC_Language->getID());
    $Qstatus->execute();
    
    if ( $Qstatus->numberOfRows() === 1 ) {
      $string = $Qstatus->value('title');
      if ( !osc_empty($Qstatus->value('css_key')) ) {
        $string = '<span class="' . $Qstatus->value('css_key') . '">' . string . '</span>'; // 返回发货的描述信息
      }
    }

    return $string;
  }
}
```

### 重量单位

产品的重量单位是在 osCommerce V3 才加入的新特性,在 V3 版本,除了增加重量单位的支持, 还增加了发货重量计算、发货包裹数量的计算,实现了按重量进行自动分配包裹的功能

osC_Product 类加载重量 includes/classes/product.php[23]

```php
$Qproduct = $osC_Database->query('select products_id as id, parent_id, products_quantity as quantity, products_price as price, products_model as model, products_tax_class_id as tax_class_id, products_weight as weight, products_weight_class as weight_class_id, products_date_added as date_added, manufacturers_id, has_children from :table_products where products_id = :products_id and products_status = :products_status');

$Qproduct->bindTable(':table_products', TABLE_PRODUCTS);
$Qproduct->bindInt(':products_id', $id);
$Qproduct->bindInt(':products_status', 1);
$Qproduct->execute();
if ( $Qproduct->numberOfRows() === 1 ) {
  $this->_data = $Qproduct->toArray(); // 将查询结果保存至内部变量$_data
```

上面代码显示了 osC_Product 类如果加载指定产品的基本资料,其中包括了本节的重点:重量和 重量单位。重量由字段 products_weight 保存,查询得到后会命名为“weight”;重量单位 ID 由字 段 products_weight_class 保存,得到后命名为“weight_class_id”。

产品的重量除了上面的产品自身重量以外,当产品拥有多选项时,它还将拥有子产品的重量 (即选项所附加的重量),子产品重量的初始化,代码在前面“osC_Product 类初始化产品选项” 一节里已经涉及,将读者查阅

重量的显示 includes/modules/product_listing.php[150]

```php
case 'PRODUCT_LIST_WEIGHT':
  $lc_align = 'right';
  $lc_text = '&nbsp;' . $osC_Product->getWeight() . '&nbsp;';
```

在产品列表页面,如果允许显示产品重量,那么上面的代码便会生效。

当初始化 osC_Product 产品类后,需要显示产品重量,只须调用它的 getWeight 方法就可以了

osC_Product 类的 getWeight 方法 includes/classes/product.php [269]

```php
function getWeight() {
  global $osC_Weight;

  $weight = 0;
  if ( $this->hasVariants() ) { // 如果拥有选项,则需计算选项重量
    foreach ( $this->_data['variants'] as $subproduct_id => $variants ) {
      foreach ( $variants['values'] as $group_id => $values ) {
        foreach ( $values as $value_id => $data ) {
          if ( $data['default'] === true ) {
            $weight = $osC_Weight->display($variants['data']['weight'], $variants['data']['weight_class_id']); //通过osC_Weight类的display方法显示包含默认选 项时的重量
            break 3;
          }
        }
      }
    }
  } else {
    $weight = $osC_Weight->display($this->_data['weight'], $this->_data['weight_class_id']); // 直接显示产品重量
  }
  
  return $weight;
}
```

在查看 osC_Weight 类的 display 方法了解将如果显示产品重量之前,让我们先来了解 osC_Weight 类是如果初始化的

osC_Weight 类 includes/classes/weight.php[38]

```php
function prepareRules() {
  global $osC_Database, $osC_Language; // 得到所有可用的重量单位与换算规则
  
  $Qrules = $osC_Database->query('select r.weight_class_from_id, r.weight_class_to_id, r.weight_class_rule from :table_weight_class_rules r, :table_weight_class c where c.weight_class_id = r.weight_class_from_id');
  
  $Qrules->bindTable(':table_weight_class_rules', TABLE_WEIGHT_CLASS_RULES);
  $Qrules->bindTable(':table_weight_class', TABLE_WEIGHT_CLASS);
  $Qrules->setCache('weight-rules');
  $Qrules->execute();

  while ($Qrules->next()) { // 保存换算规则
    $this->weight_classes[$Qrules->valueInt('weight_class_from_id')][$Qrules->valueInt('weight_class_to_id')] = $Qrules->value('weight_class_rule');
  }

  // 继续得到所有重量单位的名称以及简称,字段 weight_class_key 为单位简称,weight_class_title 为单位全称
  $Qclasses = $osC_Database->query('select weight_class_id, weight_class_key,   weight_class_title from :table_weight_class where language_id = :language_id');
  $Qclasses->bindTable(':table_weight_class', TABLE_WEIGHT_CLASS);
  $Qclasses->bindInt(':language_id', $osC_Language->getID());
  $Qclasses->setCache('weight-classes');
  $Qclasses->execute();

  while ($Qclasses->next()) {
   $this->weight_classes[$Qclasses->valueInt('weight_class_id')]['key'] = $Qclasses->value('weight_class_key');
   $this->weight_classes[$Qclasses->valueInt('weight_class_id')]['title'] = $Qclasses->value('weight_class_title');
  }

  $Qrules->freeResult();
  $Qclasses->freeResult();
}
```

osC_Weight 类的 display 方法 includes/classes/weight.php[76]

```php
function display($value, $class) { // 参数$value 指定重量值,$class 指定重量单位的 ID 
  global $osC_Language;

  return number_format($value, (int)$this->precision, $osC_Language->getNumericDecimalSeparator(), $osC_Language->getNumericThousandsSeparator()) . $this->weight_classes[$class]['key'];
  // 使用 number_format 函数格式化重量,并在其后追加 重量单位
}
```

> 提示:
> 
> number_format 的第二个参数为小数点精确到的位数,由内部变量$precision 指定,默认值为 2,  第三个参数为小数点符号,由 osC_Language 类的方法 getNumericDecimalSeparator 指定全局设定, 第四个参数是千位符,由 osC_Language 类的方法 getNumericThousandsSeparator 获得全局设定值

通过上面的介绍,我们已经了解了产品重量及重量单位的加载过程,以及产品重量的显示过程。
而对于如运输模块等对重量有要求的功能又是如何得到用户所购买产品的总重量的呢?

下面我们将就此方面的内容进行介绍

购物车产品的重量计算

osC_ShoppingCart 类的汇总方法_calculate 

includes/classes/shopping_cart.php[907]

```php
foreach ( $this->_contents as $data ) { // 循环所有产品,统计重量总值 
  $products_weight = $osC_Weight->convert($data['weight'], $data['weight_class_id'], SHIPPING_WEIGHT_UNIT); 
  // 通过 osC_Weight 类的 convert 方法统 计重量单位,最终的重量单位由常量 SHIPPING_WEIGHT_UNIT 指定
  $this->_weight += $products_weight * $data['quantity']; // 将重量值加总 ......
}

$this->_shipping_boxes_weight = $this->_weight;
$this->_shipping_boxes = 1;

// 计算发货包裹重量及包裹数量
if ( SHIPPING_BOX_WEIGHT >= ($this->_shipping_boxes_weight * SHIPPING_BOX_PADDING/100) ) {
  $this->_shipping_boxes_weight = $this->_shipping_boxes_weight + SHIPPING_BOX_WEIGHT;
} else {
  $this->_shipping_boxes_weight = $this->_shipping_boxes_weight + ($this->_shipping_boxes_weight * SHIPPING_BOX_PADDING/100);
}

// 拆分包裹
if ( $this->_shipping_boxes_weight > SHIPPING_MAX_WEIGHT ) { // Split into many boxes
  $this->_shipping_boxes = ceil($this->_shipping_boxes_weight / SHIPPING_MAX_WEIGHT);
  $this->_shipping_boxes_weight = $this->_shipping_boxes_weight / $this->_shipping_boxes;
}
```

重量的换算

osC_Weight 类的 convert 方法 includes/classes/weight.php[66]

```php
function convert($value, $unit_from, $unit_to) {
  global $osC_Language;

  if ($unit_from == $unit_to) { // 如果单位一致,则不需要换算,只需直接将重量格式化 
    return number_format($value, (int)$this->precision, $osC_Language->getNumericDecimalSeparator(), $osC_Language->getNumericThousandsSeparator());
  } else {
    return number_format($value * $this->weight_classes[(int)$unit_from][(int)$unit_to], (int)$this->precision, $osC_Language->getNumericDecimalSeparator(), $osC_Language->getNumericThousandsSeparator()); 
    // 通过内部变量$weight_classes 获得换 算比例,然后再计算出重量
  }
}
```

> 提示:
> 
> 想要获得购物车内所有产品的发货重量,需使用 osC_ShoppingCart 类的 getShippingBoxesWeight 方法,如果要获得发货包裹数则需要使用 numberOfShippingBoxes 方法。
