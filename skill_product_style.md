## 更改产品列表显示方式

osCommerce 默认的产品列表显示样式是从上至下的列表式,这种方式可以显示较多的产品信息, 满足用户对产品的了解;

但如果只提供给用户一种显示样式的话,用户浏览时间较长就会感觉到疲劳,而且列表式的显示方式如果要显示很多产品的话,对于排列靠后的产品,浏览起来是很困难的。

所以针对这种情况,我们将对所有产品列表增加一种表格显示方式,产品排列以表格样式显示出来,使得用户一次可以了解多个产品,对于产品的浏览有较大的帮助。

1、修改文件 includes/classes/boxes.php

我们需要修改 productListingBox 类,因为 productListingBox 类是负责产品列表输出的类,所以我 们对 productListingBox 类的修改,将影响到所以产品列表的输出,从而做到一劳永逸。

新的 productListingBox 类代码如下:

```php
class productListingBox extends tableBox {
  function productListingBox($contents) {
    global $HTTP_SESSION_VARS,$PHP_SELF;

    $this->table_parameters = 'class="productListing"';

    // 当设置了显示样式为“grid”时,将使用自定义的方法显示产品列表
    if((substr(basename($PHP_SELF), 0, 13) != 'shopping_cart') && tep_session_is_registered('product_list_style') && 'grid' == $_SESSION['product_list_style'])
      $this->productListingContent($contents);
    else
      $this->tableBox($contents, true);
  }

  // 这个就是我们以表格方式显示产品列表的方法
  function productListingContent($contents){
    $result = '<ul class="pl_g">';
    $resultArray = array();

    for($i=1,$n=count($contents);$i<$n;$i++) {
      $productImage = $contents[$i][0]['text'];
      $productName = $contents[$i][1]['text'];
      $productPrice = $contents[$i][2]['text'];
      $productBuy = $contents[$i][3]['text'];
      $result .= sprintf('<li>%s<br/>%s<br/><strong>%s</strong><br/><div class="bb">%s</div></li>',$productImage,$productName,$productPrice,$productBuy);
    }
    $result .= '<div class="clear"></div>';
    $result .= '</ul>';
    echo $result;
                                                         

    return $result;
  }
}
```

> 提示:
> 
> 1. 上述代码里的 if((substr(basename($PHP_SELF), 0, 13) != 'shopping_cart')部分是为了避免购物车的显示也区分样式,因为购物车的产品列表使用了与普通产品列表一样的 productListingBox 类输出。
> 
> 2. 新的 productListingContent 方法,是我们以表格样式显示产品的关键,但笔者避免了使用 table 进行包裹的方式,改用 ul,意在提示读者我们不仅可以扩展出表格样式,而且可以扩展出更多的 个性化格局,而最终的显示样式依赖 CSS 样式表的实现,productListingContent 方法只负责数据
的输出。

2、修改文件:includes/classes/split_page_results.php

split_page_result 类是显示分页结果的封装,我们将在此文件里增加一个方法输出显示样式选择框。 增加以下方法:

```php
function display_style(){
  global $PHP_SELF;

  $style = 'list';

  if(tep_session_is_registered('product_list_style') && 'grid' ==   $_SESSION['product_list_style'])
    $style = 'grid';

  $result = '<span class="pl_s">'. TEXT_PRODUCT_LISTING_STYLE_LEGEND;
  $result .= '<a class="'.('grid' == $style?'curr':'').'" href="' . tep_href_link(basename($PHP_SELF),tep_get_all_get_params(array('style')).'style=g rid') .'"/><img src="images/grid_icon.gif"/>'.TEXT_PRODUCT_LISTING_STYLE_GRID.'</a>';
  // grid_icon.gif 图片是表格样式的图标

  $result .= '<a class="'.('list' == $style?'curr':'').'" href="' .   tep_href_link(basename($PHP_SELF),tep_get_all_get_params(array('style')).'style=list') .'"/><img src="images/list_icon.gif"/>'.TEXT_PRODUCT_LISTING_STYLE_LIST.'</a>';
  // list_icon.gif 图片是列表样式的图标
  
  $result .= '</span>';
  return $result;
}
```

3、修改文件:includes/modules/product_listing.php 查找:

```php
<td class="smallText" align="right"><?php echo TEXT_RESULT_PAGE . ' ' . $listing_split->display_links(MAX_DISPLAY_PAGE_LINKS, tep_get_all_get_params(array('page', 'info', 'x', 'y'))); ?></td>
```

替换成如下代码:

```php                     
<td class="smallText" align="right"><?php echo $listing_split->display_style(); ?> <?php echo TEXT_RESULT_PAGE . ' ' . $listing_split->display_links(MAX_DISPLAY_PAGE_LINKS, tep_get_all_get_params(array('page', 'info', 'x', 'y'))); ?></td>
```

4、修改语言文件:includes/languages/english.php 增加以下定义:

```php
define(‘TEXT_PRODUCT_LISTING_STYLE_LEGEND’,’View:’);
define(‘TEXT_PRODUCT_LISTING_STYLE_GRID’,’Grid’);
define(‘TEXT_PRODUCT_LISTING_STYLE_LIST’,’List’);
```

5、修改样式表:stylesheet.css

增加以下样式:

```css
ul.pl_g{margin:0;padding:0;font-size:12px;}
ul.pl_g li{width:200px;height:240px;display:block;float:left;list-style:none;padding:10px 0 0 10px;border-bottom:1px dotted #ccc;position:relative;}
ul.pl_g li .bb{position:absolute;bottom:10px;left:10px;}
.pl_s{margin-right:10px;}
.pl_s a{margin:0 2px;padding:2px 5px;}
.pl_s a img{vertical-align:middle;}
.pl_s a.curr{background-color:#ff9;border:1px solid #ccc;font-weight:bold;}
```
