## 函数

### 标准函数

文件:includes/functions/general.php 

#### function tep_exit()

功能:结束当前程序

#### function tep_redirect($url)

功能:跳转页面

参数:
-  $url:要跳转到的URL

例:跳转登陆页面并使用 SSL 连接 

```php
tep_redirect(tep_href_link(FILENAME_LOGIN, '', 'SSL'));
```

#### function tep_parse_input_field_data($data, $parse) 

功能:过滤输入内容,依照对应表$parse 过滤输入的内容 

参数:
-  $data:输入的内容
-  $parse:关联表,过滤对应表

#### function tep_output_string($string, $translate = false, $protected = false) 

功能:输出安全的 HTML

参数:
-  $string:要输出的文本
-  $translate:是否进行转义,如“””将转化为“&quot;”   $protected:是否进行完全转义

> 为什么要进行 HTML 转义输出?
> 
> 因为 HTML 对应特殊字符都有其对应的编码,使用编码进行输出是一种通用且安全的做法。
> 
> 如果未进行转义,则有可能会使得 HTML 失真,有时这种错误的结果可能会很严重。 
> 
> 我们举一个例子来说明 HTML 转义的重要性,当我们要输出一个链接时,如果不使用 HTML转义,正常情况下,我们是可以完整的链接地址,实现正确跳转,但如果链接地址里有双引号“”” (虽然这种情况具有极小的概率, 同样情况出现在产品名称、产品描述的可能性是完全可能的。),那么未进行 HTML 转义将使得我们的链接地址不完整(因为 HTML 解析地址时是按双引 号“””进行最近配对的)
> 
> 因此使得我们的输出完全错误。如果进行 HTML 转义,“””将被转化 为“&quot;”,从而避免了上述错误的出现。

#### function tep_output_string_protected($string) 

功能:输出安全的 HTML,即进行完全 HTML 转义 

参数:
-  $string:要输出的文本

此函数是对 tep_output_string 函数的再次封装

#### function tep_sanitize_string($string)

功能:过滤特殊字符,过滤规则包括:将“ +”替换成“ ”,以及将“[”、“]”、“<”、“>”替换成“_” 

参数:
-  $string:需要过滤的文本

#### function tep_random_select($query)

功能:随机返回查询结果中的一条记录 

参数:
-  $query:查询SQL语句 

返回值:Array/false,一条记录

例子:随机显示一条评论 Review 记录 

```php
define('SMALL_IMAGE_WIDTH',40); 
define('SMALL_IMAGE_HEIGHT,40); 
define('BOX_REVIEWS_TEXT_OF_5_STARS', '%s of 5 Stars!');
$random_select = "select r.reviews_id, r.reviews_rating, p.products_id, p.products_image, pd.products_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd, " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = r.products_id and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "'";

$random_select .= " order by r.reviews_id desc limit 10";
$random_product = tep_random_select($random_select);
if ($random_product) {
  // display random review box
  $rand_review_query = tep_db_query("select substring(reviews_text, 1, 60) as reviews_text from " . TABLE_REVIEWS_DESCRIPTION . " where reviews_id = '" . (int)$random_product['reviews_id'] . "' and languages_id = '" . (int)$languages_id . "'");
  $rand_review = tep_db_fetch_array($rand_review_query);
  $rand_review_text = tep_break_string(tep_output_string_protected($rand_review['reviews_text']), 15, '-<br>');

  echo '<div align="center"><a href="' . tep_href_link(FILENAME_PRODUCT_REVIEWS_INFO, 'products_id=' . $random_product['products_id'] . '&reviews_id=' . $random_product['reviews_id']) . '">' . tep_image(DIR_WS_IMAGES . $random_product['products_image'], $random_product['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '</a></div><a href="' . tep_href_link(FILENAME_PRODUCT_REVIEWS_INFO, 'products_id=' . $random_product['products_id'] . '&reviews_id=' . $random_product['reviews_id']) . '">' . $rand_review_text . ' ..</a><br><div align="center">' . tep_image(DIR_WS_IMAGES . 'stars_' . $random_product['reviews_rating'] . '.gif' , sprintf(BOX_REVIEWS_TEXT_OF_5_STARS, $random_product['reviews_rating'])) . '</div>';
}
```


#### function tep_get_products_name($product_id, $language = '')

功能:获取产品名称

参数:
-  $product_id:产品ID
-  $language:语言ID,如果$language为空则使用当前默认语言

#### function tep_get_products_special_price($product_id)

功能:获取产品的特价 

参数:
-  $product_id:产品ID

#### function tep_get_products_stock($products_id)

功能:获取产品库存

参数:
-  $products_id:产品ID,此处的产品ID可以直接使用购物车的产品ID(产品ID后附加有参数值)

#### function tep_check_stock($products_id, $products_quantity)

功能:检查产品库存

参数:
-  $products_id:产品ID
-  $products_quantity:需购买的数量

返回值: 有足够库存时返回空文本, 库存不足时返回'<span class="markProductOutOfStock">***</span>'

#### function tep_break_string($string, $len, $break_char = '-') 

功能:将文本按指定宽度分段(超出指定字符个数即增加一个特殊特殊符号$break_char) 

参数:

-  $string:要分段的文本
-  $len:分段的宽度(字符个数)
-  $break_char:分隔符

#### function tep_get_all_get_params($exclude_array = '')

功能:获取所有$_GET 参数

参数:
-  $exclude_array:无需收集的$_GET参数

#### function tep_get_countries($countries_id = '', $with_iso_codes = false)

功能:获取所有国家列表,或者获取指定国家的信息

参数:
-  $countries_id:国家ID
-  $with_iso_codes:是否同时返回指定国家的ISOCODE代码

#### function tep_get_countries_with_iso_codes($countries_id) 

功能:获取指定国家的名称和 ISO CODE

参数:
-  $countries_id:国家ID

#### function tep_get_path($current_category_id = '') 

功能:获取当前浏览的产品目录路径,如“12_234_50”,此处 12 为 234 的父目录,50 为 234 的 子目录

参数:
-  $current_category_id:产品目录ID,如指定了产品目录ID,则将返回指定的目录ID的完整产品目录路径,否则将返回当前浏览的产品目录路径

#### function tep_browser_detect($component)

功能:返回当前客户端的浏览器信息 

参数:
-  $component:要查找的浏览器信息

>  提示:
>  
> 要查看当前浏览器是否为 IE,可以使用 tep_browser_detect('MSIE')) ,当要查询浏览器是否为 FireFox 时可以使用 tep_browser_detect('Mozilla/')

#### function tep_get_country_name($country_id)

功能:获取国家的名称

参数:
-  $country_id:国家ID

#### function tep_get_zone_name($country_id, $zone_id, $default_zone) 

功能:获取 Zone 名称

参数:
-  $country_id:Zone所在的国家ID
-  $zone_id:Zone ID
-  $default_zone:默认返回值,如果查找不到Zone名称,则返回此值

#### function tep_get_zone_code($country_id, $zone_id, $default_zone) 

功能:获取 Zone 的代码

参数:
-  $country_id:Zone所在的国家ID
-  $zone_id:Zone ID
-  $default_zone:默认返回值,如果查找不到Zone名称,则返回此值

#### function tep_round($number, $precision)

功能:货币四舍五入

参数:
-  $number:金额
-  $precision:精确到小数点位数

例子:以当前币种显示 1000 元,精确到小数点后 4 位数

```php
$number = 1000;
$currency_value = $currencies->currencies[$currency]['value'];
echo tep_round($number * $currency_value,
$currencies->currencies[$currency]['decimal_places']);
```

#### function tep_get_tax_rate($class_id, $country_id = -1, $zone_id = -1)

功能:获取税率 

参数:
-  $class_id:税类
-  $country_id:国家ID,如果默认值“-1”,则使用当前网站的国家   
-  $zone_id:ZoneID,如果默认值“-1”,则使用当前网站所属的Zone

#### function tep_get_tax_description($class_id, $country_id, $zone_id)

功能:获取税的描述

参数:
-  $class_id:税类
-  $country_id:国家ID,如果默认值“-1”,则使用当前网站的国家
-  $zone_id:Zone ID 返回值:当指定税存时,返回指定税的描述,否则返回'Unknown tax rate' 提示文本

#### function tep_add_tax($price, $tax)

功能:计算金额与税率的总和 

参数:
-  $price:金额
-  $tax:税率

#### function tep_calculate_tax($price, $tax)

功能:计算税额 

参数:
-  $price:金额   
-  $tax:税率

#### function tep_count_products_in_category($category_id, $include_inactive = false)

功能:获取指定产品分类中产品的数量 

参数:
-  $category_id:产品分类ID
-  $include_inactive:是否计算未显示产品

> 提示:
> 
> 此函数不仅计算当前分类的产品数量,而且会自动计算分类属下的子分类的产品数量

#### function tep_has_category_subcategories($category_id)

功能:获取指定分类是否拥有子分类 

参数:
-  $category_id:产品分类ID 

返回值:true/false

#### function tep_get_address_format_id($country_id)

功能:获取国家的地址格式 

参数:
-  $country_id:国家ID 

返回值:地址格式 ID

#### function tep_address_format($address_format_id, $address, $html, $boln, $eoln)

功能:格式化输出地址

参数:
-  $address_format_id:地址格式ID
-  $address:关联数组,需要显示的地址内容   
-  $html:true/false,是否以HTML显示
-  $boln:换行符后的字符
-  $eoln:换行符

> 提示:
> 
> 地址内容采用与数据库字段相一致的命名,具体内容如下;

```php
array(
  'company'=>'公司名',
  'firstname'=>'名',
  'lastname'=>'姓',
  'street_address'=>'街道地址',
  'suburb'=>'街区',
  'city'=>'城市',
  'state'=>'省/区',
  'country_id'=>'国家 ID',
  'zone_id'=>'省/区 ID',
  'postcode'=>'邮编或 ZIP')
```

#### function tep_address_label($customers_id, $address_id = 1, $html = false, $boln = '', $eoln = "\n")

功能:获取用户地址的格式化文本 

参数:
-  $customers_id 用户 ID
-  $address_id:地址ID
-  $html:true/false,是否以HTML显示
-  $boln:换行符后的字符
-  $eoln:换行符,以HTML方式显示,当指定“\n”时会自动被替换为“<br>”

#### function tep_row_number_format($number)

功能:两位补齐,不足十自动在前面补零 

参数:
-  $number:数值

#### function tep_get_categories($categories_array = '', $parent_id = '0', $indent = '')

功能:获取指定产品分类的所有子分类

参数:
-  $categories_array:用于继承的分类数组
-  $parent_id:产品分类ID,如果$parent_id=0,则将获取所有分类及所有子分类
-  $indent:分类层级前缀

返回值:二维关联数组, `array(array('id'=>'分类 ID','text'=>'分类名称'))`

#### function tep_get_manufacturers($manufacturers_array = '')

功能:获取所有制造商列表

参数:
-  $manufacturers_array:用于继承的结果数组 

返回值:二维关联数组, `array(array('id'=>'制造商 ID','text'=>'制造商名称'))`

#### function tep_get_subcategories(&$subcategories_array, $parent_id = 0) 

功能:获取指定产品分类的所有子分类 ID

参数:
-  $subcategories_array:返回结果数组,结果格式为 `array(array('categories_id'=>'分类ID')) ` 
- $parent_id:产品分类ID,当$parent_id=0时将返回所有分类ID

#### function tep_date_long($raw_date)

功能:输出长日期格式
参数:
-  $raw_date:日期文本,需要格式化的日期

#### function tep_date_short($raw_date)

功能:输出短日期格式

参数:
-  $raw_date:日期文本,需要格式化的日期

#### function tep_parse_search_string($search_str = '', &$objects)

功能:分解查询条件

参数:
-  $search_str:查询文本
-  $objects:分解后的结果数组 

返回值:true/false,是否成功分解

#### function tep_checkdate($date_to_check, $format_string, &$date_array)

功能:检查日期文本是否符合指定的日期格式 

参数:
-  $date_to_check:需要检查的文本
-  $format_string:日期格式文本
-  $date_array:数组,输出查找后的日期,格式为: `array('年','月','日');`

#### function tep_is_leap_year($year)

功能:是否为润年 

参数:

-  $year:要查询的年份

#### function tep_create_sort_heading($sortby, $colnum, $heading)

功能:创建包含指定排序方式的键接

参数:
-  $sortby:原排序关键字,关键字组成形式为排序字段+“a”或“d”,“a”表示升序(ascendingly),“d”表示降序(descendingly)
-  $colnum:新排序字段
-  $heading:前缀,用于指示键接的含义
 
例子:由$_GET['sort']指定排序的关键字,创建一个以价格(假设价格字段为 1)由小到大排序 的键接

```php
echo tep_create_sort_heading($HTTP_GET_VARS['sort'],1, 'Price');
```

#### function tep_get_parent_categories(&$categories, $categories_id) 

功能:获取产品分类的所有上级分类 ID

参数:
-  $categories:一维数组,所有上级分类ID将保存在此数组里,分类ID的顺序由最靠近当前分类开始
-  $categories_id:要查询的产品分类ID

#### function tep_get_product_path($products_id)

功能:获取产品所有分类的完整分类路径

参数:
-  $products_id:要查询的产品ID

返回值:包含通向当前产品的所有层级分类 ID,排序顺序以实际层级一致,如“1_24_365”

#### function tep_get_uprid($prid, $params) 

功能:得到包含有产品属性的产品唯一识别 ID 

参数:
-  $prid:产品ID
-  $params:关联数组,产品属性

#### function tep_get_prid($uprid) 

功能;解析包含有产吕属性的产品唯一识别 ID,得到实际产品 ID 

参数:
-  $uprid:产品唯一识别ID

#### function tep_customer_greeting()

功能:针对登录用户与未登录用户返回不同的用户问候语

   
> 注释:
> 
> 此处使用到全局变量$customer_id 和$customer_first_name,通过检测这两个变量是否存在,可以 辨别用户是否已经登录。问候语的样式如下:
> 
> 1. 对于登录用户,返回“'Welcome back NAME! Would you like to see which new products are available to purchase?”
> 
> 2. 对于未登录用户,返回“Welcome Guest! Would you like to log yourself in? Or would you prefer to create an account?”

#### function tep_mail($to_name, $to_email_address, $email_subject, $email_text, $from_email_name, $from_email_address)

功能:发送邮件

参数:
-  $to_name:收件人姓名
-  $to_email_address:收件人Email
-  $email_subject:邮件主题
-  $email_text:邮件内容
-  $from_email_name:发件人姓名
-  $from_email_address:发件人Email

> 提示:
> 
> 通过 tep_mail 发送邮件,可以发送 HTML 样式邮件,也可以只发送纯文本邮件,该设置可以通过 后 台 的 Configuation->E-Mail Options->use mine HTML When Sending Emails( 常 量
 EMAIL_USE_HTML)项目来进行修改

#### function tep_has_product_attributes($products_id)

功能:获取产品是否拥有属性可选 

参数;
-  $products_id:产品ID 

返回值:true/false

#### function tep_word_count($string, $needle)

功能:查询单词个数

参数:
-  $string:需要查询的字符串
-  $needle:用于分隔单词的字符

#### function tep_count_modules($modules = '')

功能:获取模块的个数

参数:
-  $modules:包含所有已安装模块的文本,以“;”分隔

>  提示:
>  
> 已安装模块通常保存在数据库常量里,如所有已安装的付款模块保存在常量里 MODULE_PAYMENT_INSTALLED ,而所有已安装的运输模块则保存在常量 MODULE_SHIPPING_INSTALLED

#### function tep_count_payment_modules()

功能:获取所有已安装付款模块的个数

#### function tep_count_shipping_modules()

功能:获取所有已安装运输模块的个数

#### function tep_create_random_value($length, $type = 'mixed')

功能:产生随机数 

参数:
-  $length:随机数长度   
-  $type:随机数类型, 类型有“mixed”,“chars”和“digits”三种  

-  “mixed”:数字与字母的组合
-  “chars”:字母
-  “digits”:数字

#### function tep_array_to_string($array, $exclude = '', $equals = '=', $separator = '&') 

功能:通过数组值创建 URL 参数字符串

参数:
-  $array:关联数组,参数表
-  $exclude:一维数组,不需要包含在结果里的参数   
-  $equals:参数名与参数值的分隔符
-  $separator:参数间的分隔符

#### function tep_not_null($value)

功能:判断值是否为空,可以判断数组或者文本 

参数:
-  $value:数组/文本,需要判断的值


#### function tep_display_tax_value($value, $padding = TAX_DECIMAL_PLACES)

功能:显示税额

参数:
-  $value:税额
-  $padding:小数点位数

#### function tep_currency_exists($code)

功能:查询货币代码是否有效 

参数:
-  $code:货币代码

#### function tep_string_to_int($string)

功能:强制转化文本为数字类型

#### function tep_parse_category_path($cPath) 

功能:分解产品分类路径,得出所有分类 ID

参数:
-  $cPath:产品分类路径字符串,样式如“1_24_365” 

返回值:一维数组,包含所有分类 ID

#### function tep_rand($min = null, $max = null)

功能:生成随机数 

参数:

-  $min:最小值 
-  $max:最大值

#### function tep_setcookie($name, $value = '', $expire = 0, $path = '/', $domain = '', $secure = 0) 

功能:写 COOKIE

参数:
-  $name:COOKIE名称
-  $value:COOKIE值 
-  $expire:到期时间 
-  $path:路径
-  $domain:域
-  $secure:0/1,是否仅使用SSL安全连接方式

#### function tep_get_ip_address()

功能:获取访问者的 IP 地址

#### function tep_count_customer_orders($id = '', $check_session = true)

功能:获取用户的订单数量

参数:
-  $id:用户ID,如果未指定用户ID,会默认使用当前登录用户ID
-  $check_session:是否只检查当前登录用户,当此值为true时,$id值必须与当前登录用户一
致 

返回值:订单数,出错返回 0

#### function tep_count_customer_address_book_entries($id = '', $check_session = true)

功能:获取用户拥有的地址数量

参数:
-  $id:用户ID,如果未指定用户ID,会默认使用当前登录用户ID
-  $check_session:是否只检查当前登录用户,当此值为true时,$id值必须与当前登录用户一
致 

返回值:地址数,出错返回 0

#### function tep_convert_linefeeds($from, $to, $string)

功能:替换换行符 

参数:
-  $from:查找的换行符   
-  $to:替换字符
-  $string:查找文本

> 注释:
> 
> tep_convert_linefeeds 只是一个兼容函数,因为在 PHP4.2.0 以前 nl2br 函数不能在所有操作系统里 都操作成功,因为有的 OS 使用“\n”作为换行符,而另一些 OS 使用“\r\n”作为换行符,nl2br 函数并未对这两种区别给予正确的处理。所以基于 osCommerce 一贯的兼容为先的原则,也就有 了 tep_convert_linefeeds 这个函数

### ZIP压缩 

文件:includes/functions/gzip_compression.php

#### function tep_check_gzip()

功能:获取当前浏览器可以接受的压缩方式

如果已经发送了文件头将返回 false,如果浏览器可以接受压缩方式,正确的返回值应为: 
- “x-gzip”:x-gzip压缩格式
-  “gzip”:gzip压缩格式

#### function tep_gzip_output($level = 5)

功能:以压缩方式输出,此函数只限用于兼容 PHP4.0.4 以前的压缩

参数:
-  $level:压缩等级,压缩等级范围有0-9级,0为不压缩,9为最高压缩率 压缩输出会自动使用 tep_check_gzip 函数检测可以接受的压缩方式,然后以正确的压缩方式输出 内容

例子:以正确的压缩方式输出内容,压缩等级设为 7

```php
if (PHP_VERSION >= '4.0.4') {
  ob_start('ob_gzhandler');
} else {
  include('includes/functions/gzip_compression.php');
  ob_start();
  ob_implicit_flush();
}

echo '这里的测试的内容';
if ( (PHP_VERSION < '4.0.4') && (PHP_VERSION >= '4') ) {
  tep_gzip_output(7);
}
```

### 显示与输出 

文件:includes/functions/html_output.php

#### function tep_href_link($page = '', $parameters = '', $connection = 'NONSSL', $add_session_id = true, $search_engine_safe = true)

功能:输出链接地址

参数:
-  $page:文件名
-  $parameters:参数文本
-  $connection:连接方式,“NONSSL”或者“SSL”
-  $add_session_id:是否附加SESSION会话识别
-  $search_engine_safe:是否以 SEO FRIENDLY 方式输出

> 提示:
> 
> 以普通方式输出的链接参数格式为“index.php?cPath=128&style=1&sort=2a”, SEO FRIENDLY 即 搜索引擎友好方式,这种方式会将链接参数进行转换,得出的样式为 “index.php/cPath/128/style/1/sort/2a”,当然这种 URL 优化是一种简单的地址伪装方式,如果要将样式转换为像“index_128_1_2a.html”这样或者更复杂的静态 URL,需要使用更高级的 Url Rewrite 技术。

#### function tep_image($src, $alt = '', $width = '', $height = '', $parameters = '')

功能:输出图片

参数:
-  $src:图片源
-  $alt :描述
-  $width:宽度
-  $height:高度,如果未指定高度和宽度则默认使用图片原始大小显示   
-  $parameters:参数,通常用于增加CSS样式或者JS代码

#### function tep_image_submit($image, $alt = '', $parameters = '')

功能:输出图片提交按钮,图片自动适应当前语言

参数:
-  $image:图片源,图片路径为“includes/languages/LANGUAGE/images/buttons/”
-  $alt:描述
-  $parameters:附加参数

#### function tep_image_button($image, $alt = '', $parameters = '')

功能:输出按钮图片,图片自动适应当前语言

参数:
-  $image:图片源,图片路径为“includes/languages/LANGUAGE/images/buttons/” 
-  $alt:描述
-  $parameters:附加参数

#### function tep_draw_separator($image = 'pixel_black.gif', $width = '100%', $height = '1')

功能:输出分隔符

参数:
-  $image:用作分隔符的图片
-  $width:宽度
-  $height:高度

> 提示:
>  
> osCommerce 使用图片进行界面的分隔,这种方式存在弊端,建议抛弃此方式,改用 CSS 进行页 面的设计


#### function tep_draw_form($name, $action, $method = 'post', $parameters = '') 

功能:输出 FORM 表单

参数:
-  $name:表单名称
-  $action:提交地址
-  $method:提交方法,“get”或者“post”,“put”方法已经在新的HTML标准里舍弃
-  $parameters:附加参数

#### function tep_draw_input_field($name, $value = '', $parameters = '', $type = 'text', $reinsert_value = true)

功能:输出文本框 

参数:
-  $name:文本框名称
-  $value:初始值
-  $parameters:附加参数
-  $type:类型,可以接收“text”,“password”,“radio”,“checkbox”,“hidden”等类型
-  $reinsert_value:初始值是否使用 GET、POST 方式传入值,当$reinsert_value=true 时,会自动查找$_GET[$name]或者$_POST[$name]的值进行填充

> 提示:
> 
> 细心的读者应该已经观察到,其实 tep_draw_input_field 函数不单只是输出文本框,还可以输出密
 码框、复选框、隐藏内容框、甚至提交按钮等。下面的 tep_draw_password_field 函数即是对 tep_draw_input_field 再次封装

#### function tep_draw_password_field($name, $value = '', $parameters = 'maxlength="40"')

功能:输出密码框 

参数:
-  $name:密码框名称 
-  $value:初始值
-  $parameters:附加参数
-  $maxlength:最大输入字符个数

#### function tep_draw_selection_field($name, $type, $value = '', $checked = false, $parameters = '')

功能:输出选择框,选择框可以是单选框和复选框 

参数:
-  $name:选择框名称
-  $type:类型,可以是“checkbox”,“radio”
-  $value:初始值
-  $checked:默认是否选中
-  $parameters:附加参数

#### function tep_draw_checkbox_field($name, $value = '', $checked = false, $parameters = '')

功能:输出复选框

参数:
-  $name:复选框名称
-  $value:初始值
-  $checked:默认是否选中
-  $parameters:附加参数

#### function tep_draw_radio_field($name, $value = '', $checked = false, $parameters = '')

功能:输出单选框

参数:
-  $name:单选框名称
-  $value:初始值
-  $checked:默认是否选中 
-  $parameters:附加参数

#### function tep_draw_textarea_field($name, $wrap, $width, $height, $text = '', $parameters = '', $reinsert_value = true)

功能:输出多行文本框

参数:
-   $name:多行文本框名称
-   $wrap:换行,换行选项有“off”,“physical”和“virtual”,“off”表示不换行,“physical” 表示实体,“virtual”表示虚拟,换行的选项在 FireFox 中测试已经不生效,只有 IE 里有效
-  $width:宽度
-  $height:高度
-  $text:初始值
-  $parameters:附加参数
-  $reinsert_value:初始值是否使用 GET、POST 方式传入值,当$reinsert_value=true 时,会自动查找$_GET[$name]或者$_POST[$name]的值进行填充

#### function tep_draw_hidden_field($name, $value = '', $parameters = '')

功能:输出隐藏域

参数:
-  $name:隐藏域名称
-  $value:初始值
-  $parameters:附加参数

#### function tep_hide_session_id() 

功能:输出包含 SESSION 识别 ID 的隐藏域

#### function tep_draw_pull_down_menu($name, $values, $default = '', $parameters = '', $required = false) 

功能:输出 SELECT 列表框

参数:
-  $name:列表框名称
-  $value:二维关联数组,列表选项,格式为array(array('id'=>'选项值','text'=>'显示文本'))
-  $default:默认选项ID
-  $parameters:附加参数
-  $required:是否必选,如果$required=true,则将在 SELECT 列表框后追加“ * Required”的提示信息

#### function tep_get_country_list($name, $selected = '', $parameters = '')

功能:输出所有国家列表框
参数:
-   $name:列表框名称
-   $selected:默认显示的国家ID
-   $parameters:附加参数

### 密码管理 

文件:includes/functions/password_func.php

#### function tep_validate_password($plain, $encrypted)

功能:验证密码是否正确

参数:
-  $plain:需要查验的密码
-  $encrypted:密文 

返回值:true/false,如果密码正确就返回 true,否则返回 false

#### function tep_encrypt_password($plain)

功能:密码加密 

参数:
-  $plain:密码原文

### 会话 

文件:includes/functions/sessions.php

#### function tep_session_start()

功能:打开会话

#### function tep_session_register($variable) 

功能:注册 SESSION 会话值

参数:
-  $variable:会话名称

#### function tep_session_is_registered($variable)

功能:检查会话值是否存在 

参数:
-  $variable:会话名称

#### function tep_session_unregister($variable) 

功能:注销 SESSION 会话值

参数:
-  $variable:会话名称

#### function tep_session_id($sessid = '') 

功能:设置或者返回当前 SESSION 识别 ID 

参数:
-  $sessid:SESSION识别ID

#### function tep_session_name($name = '') 

功能:设置或者返回当前 SESSION 名称 

参数:
-  $name:SESSION名称

#### function tep_session_close() 

功能:关闭 SESSION 会话
#### function tep_session_destroy() 

功能:销毁 SESSION 会话
#### function tep_session_save_path($path = '') 

功能:设置或者获取 SESSION 保存路径 

参数:
-  $path:SESSION保存路径

#### function tep_session_recreate() 

功能:重建 SESSION

### 特价管理 

文件:includes/functions/specials.php

#### function tep_set_specials_status($specials_id, $status)

功能:开启或者关闭产品特价

参数:
-  $specials_id:特价ID
-  $status:0/1,开启特价应设$status=1,否则应设为0

#### function tep_expire_specials()
功能:将所有到期的产品特价设为关闭状态

### 验证函数

文件:includes/functions/validations.php

#### function tep_validate_email($email) 

功能:验证 Email 是否合法 

参数:
-  $email:Email地址 

返回值:true/false
 
### 在线管理 

文件:includes/functions/whos_online.php

#### function tep_update_whos_online()

功能:统计在线人员 此函数将更新在线人员表显示当前在线人数,如果是已登录用户,可以知道清楚的了解当前在线 的登录用户情况,如果是未登录用户,则会显示用户名称为“Guest”

### 广告栏管理 

文件:includes/functions/banner.php

#### function tep_set_banner_status($banners_id, $status) 

功能:启用/关闭广告 Banner

参数:
-  $banners_id:Banner ID
-  $status:true/false,true表示启用此Banner,false表示禁用此Banner

#### function tep_activate_banners()

功能:自动启用预设置的 Banner 

广告位 Banner 的时间设置有两个选项,一个是到期时间(Expires On),到日期到达到期时间时, Banner会自动被禁用,另一个就是启用时间(Scheduled At),当启用时间一到,osCommerce会 自动启用此 Banner,tep_activate_banners 正是实现这样的用途

#### function tep_expire_banners()

功能:自动禁用到期的 Banner

设置 Banner 的到期,可以有两种选择,一是设置具体到期时间,一是设置浏览上限,当到期时间 一到,Banner 会自动失效(不显示),当 Banner 的浏览量到达浏览上限时,此 Banner 也会被禁用, 也就是说如果同时设置了一个 Banner 的到期时间和浏览上限,那么将会同时检测这两种情况,只 要有一种情况满足条件,该 Banner 就会失效

#### function tep_display_banner($action, $identifier)

功能:显示一个 Banner

参数:
-  $action:操作类型,操作类型可以是“dynamic”或者“static”,如果是dynamic方式,则将会随机返回一个属于$identifier组的广告位,而如果说static方式,将会返回Banner ID等于 $identifier 的广告位
-  $identifier:Banner组,通常用广告位的尺寸作为组名,这样可以明确的区分Banner的区别, 如“468x50” 

返回值:用于显示 Banner 的字符串

#### function tep_banner_exists($action, $identifier)

功能:检查 Banner 是否存在

参数:
-  $action:操作类型,操作类型可以是“dynamic”或者“static”,如果是dynamic方式,则将会随机返回一个属于$identifier组的广告位,而如果说static方式,将会返回Banner ID等于$identifier 的广告位
-  $identifier:Banner组,通常用广告位的尺寸作为组名,这样可以明确的区分Banner的区别,如“468x50”

返回值:false/关联数组,如果不存在满足条件的 Banner,则会返回 false,如果存在的话就会返回 包含 Banner 信息的数组,数组格式为:array('banners_id'=>'Banner ID','banners_title'=>'Banner 名 称','banners_image'=>'显示图片','banners_html_text'=>'描述')

#### function tep_update_banner_display_count($banner_id) 

功能:更新指定 Banner 的显示统计

参数:
-  $banner_id:Banner ID

#### function tep_update_banner_click_count($banner_id) 

功能:更新指定 Banner 的点击统计

参数:
-  $banner_id:Banner ID

### 缓存管理函数 

文件:includes/functions/cache.php

#### function write_cache(&$var, $filename)

功能:将对象保存到文件 

参数:
-  $var:对象
-  $filename:文件名及路径

#### function read_cache(&$var, $filename, $auto_expire = false)

功能:读取文件内容,并将其保存到变量

参数:
-  $var:用于保存内容的变量
-  $filename:文件名及路径
-  $auto_expire:false/到期时间(单位:秒),是否自动到期,如果设置了到期时间,则会检测 文件的最后修改日期(modification time),如果超出到期时间就会直接返回错误

> 注释:
> 
> 对象保存到文件,以及将文件内容还原主对象的过程,叫做的对象序列化(serialize),PHP 序列 化使用了两种函数:serialize 和 unserialize,serialize 负责序列化对象,即将对象转化为文本;而 unserialize 负责反序列化,即将编译后的文本解析还原为对象。
>
> PHP 序列化后的文本类似于 JSON 字面量。read_cache和write_cache函数即是通过使用serialize 和unserialize函数来完成保存及读取对象的。

#### function get_db_cache($sql, &$var, $filename, $refresh = false)

功能:读取数据缓存

参数:
-  $sql:SQL查询语句
-  $var:返回数据将要存放的变量
-  $filename:缓存文件名及路径
-  $refresh:是否即时刷新,当此参数为 false 时,会自动检测缓存文件的过期时间而决定是否  需要再次执行查询,但如果使用了即时刷新,则将不论缓存文件是否到期,都将执行查询

#### function tep_cache_categories_box($auto_expire = false, $refresh = false)

功能:保存产品分类框至缓存

参数:
-  $auto_expire:false/到期时间(单位:秒),是否自动到期,如果此值为false,则缓存将永久有效
-  $refresh:是否即时刷新

>  注释:
>  
> 产品分类框是产品分类页面左边的栏目,用于显示当前分类属下的所有分类 产品分类框的缓存文件保存在“DIR_FS_CACHE /categories_box-LANGUAGE.cache”,其中 DIR_FS_CACHE 为用户设置的缓存目录,可以在后台的 Configuration->Cache->Cache directory 选 项里设置,LANGUAGE 对应于当前的语言

#### function tep_cache_manufacturers_box($auto_expire = false, $refresh = false)

功能:保存制造商缓存

参数:
-  $auto_expire:false/到期时间(单位:秒),是否自动到期,如果此值为false,则缓存将永久
有效
-  $refresh:是否即时刷新

#### function tep_cache_also_purchased($auto_expire = false, $refresh = false)

功能:保存购买推荐信息至缓存

参数:
-  $auto_expire:false/到期时间(单位:秒),是否自动到期,如果此值为false,则缓存将永久有效
-   $refresh:是否即时刷新

### 版本兼容函数

文件:includes/functions/compatibility.php

compatibility 文件包含了用于兼容 PHP 低版本的部分函数,具体函数列表如下:

-  array_splice
-  in_array
-  array_reverse
-  constant
-  is_null
-  array_merge
-  is_numeric
-  array_slice
-  array_map
-  str_repeat
-  checkdnsrr

### 数据库操作函数 

文件:includes/functions/database.php

#### function tep_db_connect($server = DB_SERVER, $username = DB_SERVER_USERNAME, $password = DB_SERVER_PASSWORD, $database = DB_DATABASE, $link = 'db_link')

功能:建立数据库连接

参数:
-  $server:服务器名
-  $username:用户名
-  $password:密码
-  $database :数据库
-  $link:建立连接后返回的resource句柄 

返回值:resource 句柄

#### function tep_db_close($link = 'db_link')

功能:关闭数据库连接

参数:
-  $link:建立连接时返回的resource句柄,用于指定关闭数据库连接的目标

#### function tep_db_error($query, $errno, $error)

功能:输出数据库错误文本 

参数:
-  $query:SQL语句
-  $errno:错误代码
-  $error:错误描述信息

#### function tep_db_query($query, $link = 'db_link') 

功能:执行 SQL 语句

参数:
-  $query:SQL语句
-  $link:建立连接后返回的resource句柄

#### function tep_db_perform($table, $data, $action = 'insert', $parameters = '', $link = 'db_link') 

功能:插入/更新数据

参数:
-  $table:表
-  $data:关联数组,插入/更新的数据
-  $action:操作类型,“insert”或“update”,分别表示插入以及更新 
-  $parameters:需要限制的更新条件,仅在更新时有效
-  $link:数据库连接句柄

例子:更新产品分类的信息

```php
$categories_id = tep_db_prepare_input($HTTP_POST_VARS['categories_id']);
$sort_order = tep_db_prepare_input($HTTP_POST_VARS['sort_order']);
$sql_data_array = array('sort_order' => (int)$sort_order);
$update_sql_data = array('last_modified' => 'now()');
$sql_data_array = array_merge($sql_data_array, $update_sql_data);
tep_db_perform(TABLE_CATEGORIES, $sql_data_array, 'update', "categories_id = '" . (int)$categories_id . "'");
$languages = tep_get_languages();// 更新所有语种的产品分类描述

for ($i=0, $n=sizeof($languages); $i<$n; $i++) {
  $categories_name_array = $HTTP_POST_VARS['categories_name'];
  $language_id = $languages[$i]['id'];
  $sql_data_array = array('categories_name' =>
tep_db_prepare_input($categories_name_array[$language_id]));
  tep_db_perform(TABLE_CATEGORIES_DESCRIPTION, $sql_data_array, 'update', "categories_id = '" . (int)$categories_id . "' and language_id = '" . (int)$languages[$i]['id'] . "'");
}
```

#### function tep_db_fetch_array($db_query)

功能:以关联数组形式返回一条查询结果

参数:

-  $db_query:查询结果句柄

例子:输出单件产品(产品 ID:1234)的所有评价

```php
$reviews_query_raw = "select r.reviews_id, left(rd.reviews_text, 100) as reviews_text, r.reviews_rating, r.date_added, r.customers_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd where r.products_id = '1234' and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' order by r.reviews_id desc";
$reviews_query = tep_db_query($reviews_query_raw);

if(tep_db_num_rows($reviews_query)>0){
  echo '<ul>';
  while ($reviews = tep_db_fetch_array($reviews_query)) {
    echo '<li>';
    echo '<a href="' . tep_href_link(FILENAME_PRODUCT_REVIEWS_INFO, 'products_id=1234&reviews_id=' . $reviews['reviews_id']) . '"><u><b>' . sprintf('by %s', tep_output_string_protected($reviews['customers_name'])) . '</b></u></a>'; echo sprintf('Date Added: %s', tep_date_long($reviews['date_added']));
    echo '<hr/>';
    echo tep_break_string(tep_output_string_protected($reviews['reviews_text']), 60, '-<br>') . ((strlen($reviews['reviews_text']) >= 100) ? '..' : '') . '<br><br><i>' . sprintf('Rating: %s [%s]', tep_image(DIR_WS_IMAGES . 'stars_' . $reviews['reviews_rating'] . '.gif', sprintf('%s of 5 Stars!', $reviews['reviews_rating'])), sprintf('%s of 5 Stars!', $reviews['reviews_rating'])) . '</i>';
    echo '</li>';
  }
  echo '</ul>';
}
```

#### function tep_db_num_rows($db_query)

功能:返回查询结果的数目 

参数:
-  $db_query:查询结果句柄

#### function tep_db_data_seek($db_query, $row_number)

功能:数据搜索定位

参数:
-  $db_query:查询结果句柄
-  $row_number:定位的行数

#### function tep_db_insert_id() 

功能:返回最后一次插入数据所生成数据的 ID

#### function tep_db_free_result($db_query)

功能:释放查询结果内存 

参数:
-  $db_query:查询结果句柄

#### function tep_db_fetch_fields($db_query)

功能:返回一条字段信息 

参数:
-  $db_query:查询结果句柄

#### function tep_db_output($string)

功能:转译输出 HTML,适用于转译 HTML 特殊字符

#### function tep_db_input($string, $link = 'db_link')

功能:插入数据前转译输入内容 

参数:
-  $string:输入内容
-  $link:数据库连接句柄

#### function tep_db_prepare_input($string)

功能:过滤特殊字符

参数:
-  $string:文本/数组,输入内容
