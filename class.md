## 基础类

### Box类

文件:includes/classes/boxes.php

#### tableBox 类

function tableBox($contents, $direct_output = false) 

功能:实现对输入内容的格式化。

参数:
-  $contents:需要格式化输出的内容
-  $direct_output:true/false,除了返回结果外,是否直接输出格式化后的内容

tableBox 类是其它 Box 类的基类,负责最基础的输出组织工作。由于其参数复杂且不易掌握,并 且针对几乎所有想要实现的效果都有对应的子类,所以一般不直接实例化 tableBox 类,而是创建 其特定功能的子类。

#### infoBox 类

父类:tableBox

function infoBox($contents)

功能:直接输出一个信息框主体

参数:
-  $contents:要格式化输出的内容 

例子:输出一个搜索信息框

```php
$info_box_contents = array();
$info_box_contents[] = array('text' => 'Quick Find');

// 输出信息框标题
new infoBoxHeading($info_box_contents, false, false);

$info_box_contents = array();
$info_box_contents[] = array(
  'form' => tep_draw_form('quick_find', tep_href_link(FILENAME_ADVANCED_SEARCH_RESULT, '', 'NONSSL', false), 'get'),
  'align' => 'center',
  'text' => tep_draw_input_field('keywords', '', 'size="10" maxlength="30" style="width: ' . (BOX_WIDTH-30) . 'px"') . '&nbsp;' . tep_hide_session_id() . tep_image_submit('button_quick_find.gif', BOX_HEADING_SEARCH) . '<br>' . BOX_SEARCH_TEXT . '<br><a href="' . tep_href_link(FILENAME_ADVANCED_SEARCH) . '"><b>' . BOX_SEARCH_ADVANCED_SEARCH . '</b></a>');

// 输出信息框主体
new infoBox($info_box_contents);
```

信息框主体的参数为数组,其中 form 表示要求将信息框主体用 form 表单包裹起来,align 指定了 文本对齐方式,text 要求填充要显示的内容。

function infoBoxContents($contents)

功能:格式化信息框主体

参数:
-  $contents:需要格式化输出的内容
-  infoBox方法通过调用本方法实现即时输出。

#### infoBoxHeading 类

父类:tableBox

function infoBoxHeading($contents, $left_corner = true, $right_corner = true, $right_arrow = false)

功能:输出一个信息框标题

参数:
-  $contents:需要格式化输出的内容
-  $left_corner:是否设置左圆角
-  $right_corner:是否设置右圆角
-  $right_arrow:false/URL,设置右箭头标志的URL 

例子:输出一个右圆角并有右箭头标志的信息框标题

```php
$info_box_contents = array();
$info_box_contents[] = array('text' => 'Shopping Cart');

new infoBoxHeading($info_box_contents, false, true,
tep_href_link(FILENAME_SHOPPING_CART));
```

#### contentBox 类

父类:tableBox

function contentBox($contents)

功能:输出一个内容框主体

参数:
-  $contents:要求格式化的内容

例子:输出最新产品的内容框主体

```php
$new_products_query = tep_db_query("select distinct p.products_id, p.products_image, p.products_tax_class_id, pd.products_name, if(s.status, s.specials_new_products_price, p.products_price) as products_price from " . TABLE_PRODUCTS . " p left join " . TABLE_SPECIALS . " s on p.products_id = s.products_id, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c, " . TABLE_CATEGORIES . " c where p.products_id = p2c.products_id and p2c.categories_id = c.categories_id and c.parent_id = '" . (int)$new_products_category_id . "' and p.products_status = '1' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' order by p.products_date_added desc limit " . MAX_DISPLAY_NEW_PRODUCTS);

$row = 0;
$col = 0;
$info_box_contents = array();
  while ($new_products = tep_db_fetch_array($new_products_query)) {
    $info_box_contents[$row][$col] = array(
    'align' => 'center',
    'params' => 'class="smallText" width="33%" valign="top"',
    'text' => '<a href="' . tep_href_link(FILENAME_PRODUCT_INFO, 'products_id=' . $new_products['products_id']) . '">' . tep_image(DIR_WS_IMAGES . $new_products['products_image'], $new_products['products_name'], SMALL_IMAGE_WIDTH, SMALL_IMAGE_HEIGHT) . '</a><br><a href="' . tep_href_link(FILENAME_PRODUCT_INFO, 'products_id=' . $new_products['products_id']) . '">' . $new_products['products_name'] . '</a><br>' . $currencies->display_price($new_products['products_price'], tep_get_tax_rate($new_products['products_tax_class_id'])));

    $col ++;
    if ($col > 2) {
      $col = 0;
      $row ++;
    }
  }

  new contentBox($info_box_contents);
```

需要输出的内容参数包括:align 指定文本对比方式,params 设置其它样式及显示参数,text 为要 显示的内容。

function contentBoxContents($contents)

功能:格式化内容框主体

参数:
-  $contents:需要格式化输出的内容
-  contentBox方法通过包装本方法的结果实现样式输出。

#### contentBoxHeading 类

父类:tableBox

function contentBoxHeading($contents) 

功能:输出一个内容框主体

参数:
-  $contents:需要格式化输出的内容 例子:输出一个内容框标题

```php
$info_box_contents = array();
$info_box_contents[] = array('text' => sprintf('New Products For %s',
strftime('%B')));
new contentBoxHeading($info_box_contents);
```

#### errorBox

父类:tableBox

function errorBox($contents)

功能:输出一个错误提示框

参数:
-  $contents:需要输出的内容

#### productListingBox 类

父类:tableBox

function productListingBox($contents)

功能:输出一个产品列表框

参数:
-  $contents:需要格式化输出的内容

例子:输出购物车的所有产品

```php
$info_box_contents = array();
$info_box_contents[0][] = array(
  'align' => 'center',
  'params' => 'class="productListing-heading"',
  'text' => 'Remove');
$info_box_contents[0][] = array(
  'params' => 'class="productListing-heading"',
  'text' => 'Product(s)');
$info_box_contents[0][] = array(
  'align' => 'center',
  'params' => 'class="productListing-heading"',
  'text' => 'Qty.');
$info_box_contents[0][] = array(
  'align' => 'right',
  'params' => 'class="productListing-heading"',
  'text' => 'Total');
$products = $cart->get_products();

for ($i=0, $n=sizeof($products); $i<$n; $i++) {
  $cur_row = sizeof($info_box_contents) - 1;
  $info_box_contents[$cur_row][] = array(
    'align' => 'center',
    'params' => 'class="productListing-data" valign="top"',
    'text' => tep_draw_checkbox_field('cart_delete[]', $products[$i]['id']));

  $info_box_contents[$cur_row][] = array(
    'params' => 'class="productListing-data"',
    'text' => $products[$i]['name']);

  $info_box_contents[$cur_row][] = array(
    'align' => 'center',
    'params' => 'class="productListing-data" valign="top"',
    'text' => tep_draw_input_field('cart_quantity[]', $products[$i]['quantity'], 'size="4"') . tep_draw_hidden_field('products_id[]', $products[$i]['id']));

  $info_box_contents[$cur_row][] = array(
    'align' => 'right',
    'params' => 'class="productListing-data" valign="top"',
    'text' => '<b>' . $currencies->display_price($products[$i]['final_price'],
tep_get_tax_rate($products[$i]['tax_class_id']), $products[$i]['quantity']) . '</b>'); 
}
new productListingBox($info_box_contents);
```

### Breadcrumb类

文件:includes/classes/breadcrumb.php

function breadcrumb()

功能:复位内部变量

function reset()

功能:内部复位操作

function add($title, $link = '')

功能:添加新的路径

参数:
-  $title:路径标题
-  $link:路径要链接的URL,如果未设置$link或$link的值为空,则路径只显示文本,不具备链接。

function trail($separator = ' - ')

功能:格式化输出 Breadcrumb 的内容

参数:
-  $separator:用于路径之间的分隔字符 

例子:创建一个三级 Breadcrumb 并输出

```php
$breadcrumb->add('Home','index.php');
$breadcrumb->add('Account', 'account.php');
$breadcrumb->add('Account Edit','account_edit.php');

echo $breadcrumb->trail(' &raquo; '); // 符号“»”用于分隔
```

### cc_validation信用卡验证类 

文件:includes/classes/cc_validation.php

#### class cc_validation

function validate($number, $expiry_m, $expiry_y) 

功能:验证信用卡号码是否有效,支持信用卡种类有 Visa,Master Card,American Express,Diners Club,Discover,JCB 和 Australian BankCard。

参数:
-  $number:信用卡卡号
-  $expiry_m:到期日期中的月份
-  $expiry_y:到期日期中的年份

返回值:true/false

例子:验证用户输入的信用卡是否有效

```php
include(DIR_WS_CLASSES . 'cc_validation.php');

$cc_validation = new cc_validation();
$result = $cc_validation->validate($HTTP_POST_VARS['cc_number'], $HTTP_POST_VARS['cc_expires_month'], $HTTP_POST_VARS['cc_expires_year']);

$error = '';
switch ($result) {
  case -1:
    $error = sprintf('The first four digits of the number entered are: %s. If
that number is correct, we do not accept that type of credit card. If it is wrong, please try again.', substr($cc_validation->cc_number, 0, 4));
    break;
  case -2:
  case -3:
  case -4:
    $error = 'The expiry date entered for the credit card is invalid. Please check the date and try again.';
    break;
  case false:
    $error ='The credit card number entered is invalid. Please check the number and try again.';
    break;
}

echo $error;
```

private function is_valid()

功能:验证信用卡号码的有效性

validate 实际上是通过调用 is_valid 方法来实现最后的验证步骤。

###  currencies币种类

文件:includes/classes/currencies.php

#### class currencies

function currencies()

功能:初始化币种信息

币种信息保存在成员$currencies,格式为

```php
$currencies[$code] = array(
  'title' => $title, // $code:币种代码,$title:币种名称
  'symbol_left' => $symbol_left, // $symbol_left:货币前缀 
  'symbol_right' => $symbol_right, // $symbol_right:货币后缀 
  'decimal_point' => $decimal_point, // $decimal_point:小数点符号 
  'thousands_point' => $thousands_point, // $thousands_point:千位符号 
  'decimal_places' => $decimal_places, // $decimal_places:小数点位数 
  'value' => $value // 与主货币(参照货币)的转换汇率
);
```

function format($number, $calculate_currency_value = true, $currency_type = '', $currency_value = '')
                 
功能:以正确的货币格式输出金额

参数:
-  $number:金额
-  $calculate_currency_value:true/false,是否以指定的参数$currency_type作为币种输出
-  $currency_type:要显示的币种,此值为空代表使用默认币种
-  $currency_value:要显示的币种对应的新转换汇率,如果为空代表使用$currency_type的默认汇率。

例子:输出订单的运费

```php
echo $currencies->format($order->info['shipping_cost'], true,
$order->info['currency'], $order->info['currency_value']);
```

function calculate_price($products_price, $products_tax, $quantity = 1)

功能:计算产品包含数量和税率后的价格 

参数:
-  $products_price:产品价格
-  $products_tax:产品税率
-  $quantity:数量 

例子:输出购物车的总金额

```php
$total = 0;
for ($i=0, $n=sizeof($order->products); $i<$n; $i++) {
  $qty = $order->products[$i]['qty'];
  $product_query = tep_db_query("select products_id, products_price, products_tax_class_id, products_weight from " . TABLE_PRODUCTS . " where products_id = '" . tep_get_prid($order->products[$i]['id']) . "'");
  
  if ($product = tep_db_fetch_array($product_query)){
    $products_tax = tep_get_tax_rate($product['products_tax_class_id']);
    $products_price = $product['products_price'];
    $total += $currencies->calculate_price($products_price, $products_tax,
$qty);
  }
}

echo $currencies->format($total);
```

function is_set($code)

功能:验证货币代码是否有效 

参数:
-  $code:货币代码 

返回值:true/false

function get_value($code)

功能:获取指定货币的转换汇率 

参数:
-  $code:货币代码 

返回值:Decimal

function get_decimal_places($code)

功能:获取货币的小数点位置 

参数:
-  $code:货币代码

function display_price($products_price, $products_tax, $quantity = 1)

功能:计算产品价格、税额以及数量后的总金额,并以当前货币形式显示 

参数:
-  $products_price:产品价格
-  $products_tax:税额
-  $quantity:数量

例子:查找指定 ID 的产品并以当前币种显示其价格

```php

$product_info_query = tep_db_query("select p.products_id, pd.products_name, pd.products_description, p.products_model, p.products_quantity, p.products_image, pd.products_url, p.products_price, p.products_tax_class_id, p.products_date_added, p.products_date_available, p.manufacturers_id from " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "' and pd.products_id = p.products_id and pd.language_id = '" . (int)$languages_id . "'");
$product_info = tep_db_fetch_array($product_info_query);

$products_price = $currencies->display_price($product_info['products_price'], tep_get_tax_rate($product_info['products_tax_class_id']));
echo $products_price;
```

### email邮件类 

文件:includes/classes/email.php

function email($headers = '') 

功能:初始化邮件类

参数:
-  $headers:Array,需要特别添加的邮件头

function get_file($filename)

功能:获取文件内容

参数:
-  $filename:文件完整路径及文件名 

返回值:String,文件完整内容

function find_html_images($images_dir)

功能:查找成员$html 里的所有图片(格式:gif/jpg/jpeg/jpe/bmp/png/tif/tiff/swf),将所有图片以附 件的形式附加在邮件里。

参数:
-  $images_dir:图片文件存放的路径,程序将在此路径查找所有邮件内容中出现的图片。

function add_text($text = '')

功能:添加文本信息 

参数:
-  $text:要添加的文本

function add_html($html, $text = NULL, $images_dir = NULL) 

功能:添加 HTML 内容

参数:
-  $html:HTML文本
-  $text:邮件接收方无法显示HTML时,将显示的无格式文本。
-  $images_dir:NULL/String 是否将 HTML 文本里的所有图片作为附件发送

function add_html_image($file, $name = '', $c_type='application/octet-stream')

功能:添加一张图片

参数:
-  $file:文件名
-  $name;图片名称
-  $c_type:图片插入邮件的类型

各种图片类型对应的插入类型
-  gif:image/gif
-  jpg:image/jpeg
-  jpeg:image/jpeg
-  jpe:image/jpeg
-  bmp:image/bmp
-  png:image/png
-  tif:image/tiff
-  tiff:image/tiff
-  swf:application/x-shockwave-flash

function add_attachment($file, $name = '', $c_type='application/octet-stream', $encoding = 'base64')

功能;添加一个文件附件

参数:
-  $file:文件名
-  $name:附件名称
-  $c_type:插入的邮件类型
-  $encoding:文件内容编码格式

文件内容编码格式:
-  base64:BASE64加密
-  7bit:7位加密
-  quoted-printable:可打印引号格式

function add_text_part(&$obj, $text) 

功能:将文本内容加入到 MIME 信息里 

参数:
-  $obj:NULL/MIME对象
-  $text:文本信息

返回值:被修改过的 MIME 对象,如果$obj 为空,则生成新的 MIME 对象并将$text 加入其中。

function add_html_part(&$obj) 

功能:将成员$html 的内容加入到 MIME 信息里 

参数:
-  $obj:NULL/MIME对象

返回值:MIME 对象。

function add_mixed_part()

功能:生成一个新的 multipart/mixed 类型的 MIME 对象 返回:MIME 对象

function add_alternative_part(&$obj)

功能:在 MIME 对象里添加一个新的 multipart/alternative 类型数据 

参数:
-  $obj:MIME对象

返回值:MIME 对象。

function add_related_part(&$obj)

功能:在 MIME 对象里添加一个新的 multipart/related 类型数据 

参数:

-  $obj:MIME对象 

返回值:MIME 对象。

function add_html_image_part(&$obj, $value) 

功能:添加一个图片至 MIME 对象

参数:
-  $obj:MIME对象
-  $value:图片参数数组,

参数格式如下: array('c_type'=>'类型',' name'=>'图片名称',' body'=>'图片内容');

function add_attachment_part(&$obj, $value) 

功能:添加一个附件至 MIME 对象 

参数:
-  $obj:MIME对象
-  $value:附件内容数组,

参数格式如下: array('c_type'=>'类型',' encoding'=>'编码方式', 'name'=>'附件名称',' body'=>'附件内容');

function build_message($params = '')

功能:转换所有内容至 MIME 格式,并以指定编码方式进行编码 

参数:
-  $params:邮件参数数组,包括:

```php
array(
  'html_encoding' => 'HTML 编 码 方 式 ',
  'text_encoding'=>' 纯 文 本 编 码 方 式 ',
  'html_charset'=>'HTML 字符集',
  'text_charset'=>'纯文本字符集',
  'text_wrap'=>'文本换行')
```

function send($to_name, $to_addr, $from_name, $from_addr, $subject = '', $headers = '')

功能:发送邮件

参数:
-  $to_name:收件人名称   
-  $to_addr:收件人邮箱
-  $from_name:发件人名称   
-  $from_addr:发件人邮箱
-  $subject:邮件主题
-  $headers:邮件头

function get_rfc822($to_name, $to_addr, $from_name, $from_addr, $subject = '', $headers = '') 

功能:获取邮件内容的 RFC822 格式

参数:
-  $to_name:收件人名称
-  $to_addr:收件人邮箱
-  $from_name:发件人名称   
-  $from_addr:发件人邮箱   
-  $subject:邮件主题
-  $headers:邮件头 

返回:RFC822 格式的邮件文本

例子:向所有已订阅用户发送一封邮件

```php
$mail_query = tep_db_query("select customers_firstname, customers_lastname, customers_email_address from " . TABLE_CUSTOMERS . " where customers_newsletter = '1'");
$mimemessage = new email(array('X-Mailer: osCommerce bulk mailer')); $mimemessage->add_text('这里是测试邮件的内容!'); $mimemessage->build_message();
while ($mail = tep_db_fetch_array($mail_query)) {
  $mimemessage->send($mail['customers_firstname'] . ' ' . $mail['customers_lastname'], $mail['customers_email_address'], 'administrator', 'admin@localhost', '测试邮件');
}
```

> 提示:
>
> tep_mail 函数已经将 email 类进行了封装,只需调用该函数就可以实现上面的发送邮件功能。使用 更加方便,详情请见下一节《general.php 标准函数》之 tep_mail 章节

> 上面的例子中为什么不使用 tep_mail 函数?
>
> 因为 tep_mail 函数实际是将上面例子的代码封装了一遍,如果要发送多封邮件或者需要群发邮件 时,不建议使用 tep_mail 函数,因为考虑到代码的效率,每发送一封邮件 tep_mail 函数就必须重 新生成一遍邮件内容,所以效率比较低。

> 如何将邮件内容加密?
>
> 1. 查找 email.php[59]代码$this->build_params['html_encoding'] = 'quoted-printable';,将其修改为: $this->build_params['html_encoding'] = 'base64';
>
> 2. 在调用 build_message 方法转换 email 类时,传入编码方式,代码如下: $mimemessage->build_message (array('html_encoding '=>'base64'));


### httpClient远程访问类

文件:includes/classes/http_client.php

function httpClient($host = '', $port = '')

功能:开启一个 HTTP 连接,无未指定$host 则表示暂时不连接,可在需要时使用 connect 方法连 接。

参数:
-  $host:连接的域名或IP
-  $port:端口号

function setProxy($proxyHost, $proxyPort)

功能:设置代理

参数:
-  $proxyHos:代理域名或IPt 
- $proxyPort:代理端口号

function setProtocolVersion($version) 

功能:设置协议版本,如 0.9、1.0、1,1,版本号不能大于 1.1

参数:
-  $version:版本号

function setCredentials($username, $password) 

功能:设置认证信息,当连接的网址开启了密码验证时,需要使用此功能,目前只支持 Basic 验 证方式

参数:
-  $username:用户名
-  $password:密码

function setHeaders($headers)

功能:设置请求头信息

参数:
-  $headers:关联数组,头信息

function addHeader($headerName, $headerValue)

功能:增加一条请求头信息 

参数:
-  $headerName:头信息名称   
-  $headerValue:头信息值

function removeHeader($headerName)

功能:删除一条请求头信息 

参数:
  $headerName:头信息名称

function Connect($host, $port = '') 

功能:连接域名或 IP

参数:
-  $host:域名或IP
-  $port:端口号

例子:连接至 United State Postal Service(USPS 美国邮政)网关

```php
$request = 'Test Request Message';
$http = new httpClient();
if ($http->Connect('production.shippingapis.com', 80)) {
  $http->addHeader('Host', 'production.shippingapis.com');
  $http->addHeader('User-Agent', 'osCommerce');
  $http->addHeader('Connection', 'Close');
  if ($http->Get('/shippingapi.dll?' . $request)) $body = $http->getBody();
  $http->Disconnect();
}

function Disconnect()
```

功能:断开连接

function Head($uri) 

功能:获取指定网址的头信息,只有当返回连接正常(代码:200)时,才有效 

参数:
-  $uri:网址URL

返回值:“Bad Response”/ 头信息文本

function Get($url) 

功能:获取指定网址的内容,只有当返回连接正常(代码:200)时,才有效 

参数:
-  $url:网址URL

返回值:“Bad Response”/ 网站内容文本

function Post($uri, $query_params = '') 

功能:发送 POST 请求

参数:
-  $uri:网址URL
-  $query_params:关联数组,POST参数 

返回值:“Bad Response”/ 网站内容文本

function Put($uri, $filecontent)

功能:发送 PUT 请求,PUT 请求允许发送一个文件至指定网址,但大部分服务器不支持或已禁 用此功能

参数:
-  $uri:网址URL
-  $query_params:关联数组,文件内容

返回值:“Bad Response”/ 网站内容文本

function getHeaders()

功能:获取返回的头信息

function getHeader($headername)

功能:获取指定名称的返回头信息

function getBody()

功能:获取返回内容文本

function getStatus()

功能:获取返回状态

状态包括(x 为数字):
-  20x :成功请求
-  30x :文档已移动
-  40x :请求信息错误(URL 错误、文档不存在等等)   
-  50x :服务器出错

function getStatusMessage()

功能:获取返回状态提示信息 提示信息不仅包括状态代码,还包括此代码所表示的意义,如“404 Document not found”

function sendCommand($command)

功能:底层的发送请求命令 

参数:
-  $command:请求命令

请求命令可以是 HEAD、GET、POST 或者 PUT 

function processReply()

功能:底层的返回信息处理方法

function processHeader($lastLine = "\r\n")

功能:底层的处理返回头信息方法 

参数:
-  $lastLine:换行符

function processBody()

功能:底层的返回信息处理方法

function makeUri($uri) 

功能:获取规范格式的 URL 

参数:
-  $uri:网址或IP

### language语言类

文件:includes/classes/language.php

function language($lng = '')

功能:初始化网站所拥有的语言,并设置当前语言。

参数:
-  $lng:要设置的当前语言代码,如果此值为空,则设置系统默认语言为当前语言。

function set_language($language)

功能:设置当前语言

参数:
-  $language:语言代码

function get_browser_language()

功能:设置浏览器语言为当前语言

例子:当$_GET 传入语言选项时,设置当前语言为$_GET 值,否则使用浏览器语言作为当前语 言

```php
$lng = new language();
if (isset($HTTP_GET_VARS['language']) && tep_not_null($HTTP_GET_VARS['language'])) {
  $lng->set_language($HTTP_GET_VARS['language']);
} else {
  $lng->get_browser_language();
}
```

###  messageStack消息列表类

文件:includes/classes/message_stack.php

#### messageStack 类

父类:tableBox

function messageStack()

功能:初始化消息列表

function add($class, $message, $type = 'error')

功能:增加一条提醒消息

参数:
-  $class:消息分类,用于消息的分组管理   
-  $message:消息文本
-  $type:消息类型,默认为error类型

消息类型包括:
-  error:错误
-  warning:警告   
-  success:成功

function add_session($class, $message, $type = 'error')

功能:添加一条消息至会话

参数:
-  $class:消息分类,用于消息的分组管理   
-  $message:消息文本
-  $type:消息类型,默认为error类型

function output($class)

功能:输出消息框

参数:
-  $class:要显示的消息分类

例子:创建一个 account_password 分类的消息,并将其输出

```php
$messageStack = new messageStack(); $messageStack->add_session('account_password', 'Your password has been successfully updated.', 'success');
if ($messageStack->size('account_password') > 0) {
  echo $messageStack->output('account_password');
}
```


function size($class)

功能:指定分类的消息条数 

参数:
-  $class:消息分类

### Email多媒体信息类

文件:includes/classes/mime.php

mime 用于管理邮件的多媒体信息,如附加图片、附件或者合并多个文本等功能。email 类使用了 mime 类。

function mime($body, $params = '')

功能:初始化多媒体信息

参数:
-  $body:多媒体信息主体
-  $params:多媒体信息头参数 信息头参数包括:content_type、encoding、cid、disposition、dfilename、description、charset

function encode()

功能:加密方法

function addSubPart($body, $params)

功能:添加一个子部分

参数:
-  $body:多媒体信息主体
-  $params:多媒体信息头参数

function _getEncodedData($data, $encoding)

功能:内部加密文本方法

参数:
-  $data:要加密的文本
-  $encoding:加密格式 加密格式包括:7bit、quoted-printable、base64

function _quotedPrintableEncode($input , $line_max = 76)

功能:内部的可打印引号加密方法 

参数:
-  $input:要加密的文本
-  $line_max:单行文本的长度

### navigationHistory历史记录类

文件:includes/classes/navigation_history.php

class navigationHistory function navigationHistory() 

功能:复位游览记录
function reset()

功能:内部复位方法

function add_current_page()

功能:将当前页面加入到游览历史记录里

function remove_current_page()

功能:将当前页面从历史记录中删除

function set_snapshot($page = '')

功能:当前页面保存快照 

参数:

-  $page:页面URL

function clear_snapshot()

功能:删除所有快照

function set_path_as_snapshot($history = 0)

功能:将页面路径转换成快照 

参数:

-  $history:单条历史记录的ID

function debug()

功能:输出调试信息

function filter_parameters($parameters)

功能:过滤参数

参数:
-  $parameters:参数表

function unserialize($broken)

功能:将值反串列化成对象 

参数:
-  $broken:对象串列化后的文本

例子:如果存在 navigation 消息对象则添加当前页面,否则生成新的 navigation 对象

```php
if (tep_session_is_registered('navigation')) {
  tep_session_register('navigation');
  $navigation = new navigationHistory;
}

$navigation->add_current_page();
```

### order订单类 

文件:includes/classes/order.php

function order($order_id = '')

功能:初始化一条订单

参数:
-  $order_id:已有订单ID,如果指定了订单ID,则会加载现有订单的信息;如果没有指定订单ID,则会将当前购物车内容转换成订单信息

> 注释:
>
> osCommerce 每次点击付款,并不总会生成一个新的订单,当上次付款未成功时,再次点击付款 会使用上次的订单 ID(需要考虑到会话持续的时间),修改订单的内容并结算。

从购物车生成订单与加载现有订单 ID,所生成的 order 对象会有所区别,我们来看它们各自具体 的成员结构:

<table>
  <tbody>
    <tr>
      <td colspan="3">
        <p>购物车 <strong>-&gt; order </strong></p>
      </td>
      <td colspan="3">
        <p>订单 <strong>ID -&gt; order </strong></p>
      </td>
    </tr>
    <tr>
      <td rowspan="17">
        <p>info</p>
      </td>
      <td>
        <p>currency</p>
      </td>
      <td>
        <p>币种</p>
      </td>
      <td rowspan="17">
        <p>info</p>
      </td>
      <td>
        <p>currency</p>
      </td>
      <td>
        <p>币种</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>currency_value</p>
      </td>
      <td>
        <p>货币汇率</p>
      </td>
      <td>
        <p>currency_value</p>
      </td>
      <td>
        <p>货币汇率</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>payment_method</p>
      </td>
      <td>
        <p>付款方式</p>
      </td>
      <td>
        <p>payment_method</p>
      </td>
      <td>
        <p>付款方式</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>cc_type</p>
      </td>
      <td rowspan="4">
        <p>信用卡信息</p>
      </td>
      <td>
        <p>cc_type</p>
      </td>
      <td rowspan="4">
        <p>信用卡信息</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>cc_owner</p>
      </td>
      <td>
        <p>cc_owner</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>cc_number</p>
      </td>
      <td>
        <p>cc_number</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>cc_expires</p>
      </td>
      <td>
        <p>cc_expires</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>orders_status</p>
      </td>
      <td>
        <p>订单状态</p>
      </td>
      <td>
        <p>orders_status</p>
      </td>
      <td>
        <p>订单状态</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>total</p>
      </td>
      <td>
        <p>总计</p>
      </td>
      <td>
        <p>total</p>
      </td>
      <td>
        <p>总计</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>shipping_method</p>
      </td>
      <td>
        <p>发货方式</p>
      </td>
      <td>
        <p>shipping_method</p>
      </td>
      <td>
        <p>发货方式</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>tax_groups</p>
      </td>
      <td>
        <p>税类</p>
      </td>
      <td>
        <p>tax_groups</p>
      </td>
      <td>
        <p>税类</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>tax</p>
      </td>
      <td>
        <p>税额</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>comments</p>
      </td>
      <td>
        <p>用户备注</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>subtotal</p>
      </td>
      <td>
        <p>小计(产品金额合 计)</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>shipping_cost</p>
      </td>
      <td>
        <p>运费</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>date_purchased</p>
      </td>
      <td>
        <p>购买日期</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>last_modified</p>
      </td>
      <td>
        <p>最后修改日期</p>
      </td>
    </tr>
    <tr>
      <td rowspan="15">
        <p>customer</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td rowspan="15">
        <p>customer</p>
      </td>
      <td>
        <p>id</p>
      </td>
      <td>
        <p>用户 ID</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>name</p>
      </td>
      <td>
        <p>姓名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>firstname</p>
      </td>
      <td>
        <p>名</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>lastname</p>
      </td>
      <td>
        <p>姓</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>zone_id</p>
      </td>
      <td>
        <p>省/区 ID</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>id=&gt;国家 ID title=&gt;国家名 iso_code_2=&gt; 国 家代码2位 iso_code_3= 国 家代码3位</p>
      </td>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>title=&gt; 国 家 名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式 ID</p>
      </td>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式</p>
        <p>ID</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>telephone</p>
      </td>
      <td>
        <p>电话</p>
      </td>
      <td>
        <p>telephone</p>
      </td>
      <td>
        <p>电话</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>email_address</p>
      </td>
      <td>
        <p>Email 邮箱</p>
      </td>
      <td>
        <p>email_address</p>
      </td>
      <td>
        <p>Email 邮箱</p>
      </td>
    </tr>
    <tr>
      <td rowspan="13">
        <p>delivery</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td rowspan="13">
        <p>delivery</p>
      </td>
      <td>
        <p>name</p>
      </td>
      <td>
        <p>姓名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>firstname</p>
      </td>
      <td>
        <p>名</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>lastname</p>
      </td>
      <td>
        <p>姓</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>zone_id</p>
      </td>
      <td>
        <p>省/区 ID</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>&nbsp;</td>
    </tr>
    <tr>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>id=&gt;国家 ID title=&gt;国家名 iso_code_2=&gt; 国 家代码2位 iso_code_3= 国 家代码3位</p>
      </td>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>title=&gt; 国 家 名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>country_id</p>
      </td>
      <td>
        <p>国家 ID</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式 ID</p>
      </td>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式</p>
        <p>ID</p>
      </td>
    </tr>
    <tr>
      <td rowspan="13">
        <p>billing</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td rowspan="13">
        <p>billing</p>
      </td>
      <td>
        <p>name</p>
      </td>
      <td>
        <p>姓名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>firstname</p>
      </td>
      <td>
        <p>名</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>lastname</p>
      </td>
      <td>
        <p>姓</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
      <td>
        <p>company</p>
      </td>
      <td>
        <p>公司名称</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
      <td>
        <p>street_address</p>
      </td>
      <td>
        <p>街道地址</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
      <td>
        <p>suburb</p>
      </td>
      <td>
        <p>街区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
      <td>
        <p>city</p>
      </td>
      <td>
        <p>城市</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
      <td>
        <p>postcode</p>
      </td>
      <td>
        <p>邮编</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
      <td>
        <p>state</p>
      </td>
      <td>
        <p>省/区</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>zone_id</p>
      </td>
      <td>
        <p>省/区 ID</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>id=&gt;国家 ID title=&gt;国家名 iso_code_2=&gt; 国 家代码2位 iso_code_3= 国 家代码3位</p>
      </td>
      <td>
        <p>country</p>
      </td>
      <td>
        <p>title=&gt; 国 家 名</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>country_id</p>
      </td>
      <td>
        <p>国家 ID</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式 ID</p>
      </td>
      <td>
        <p>format_id</p>
      </td>
      <td>
        <p>地址格式</p>
        <p>ID</p>
      </td>
    </tr>
    <tr>
      <td rowspan="10">
        <p>products</p>
      </td>
      <td>
        <p>qty</p>
      </td>
      <td>
        <p>数量</p>
      </td>
      <td rowspan="10">
        <p>products</p>
      </td>
      <td>
        <p>qty</p>
      </td>
      <td>
        <p>数量</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>id</p>
      </td>
      <td>
        <p>产品 ID</p>
      </td>
      <td>
        <p>id</p>
      </td>
      <td>
        <p>产品 ID</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>name</p>
      </td>
      <td>
        <p>产品名称</p>
      </td>
      <td>
        <p>name</p>
      </td>
      <td>
        <p>产品名称</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>model</p>
      </td>
      <td>
        <p>产品型号</p>
      </td>
      <td>
        <p>model</p>
      </td>
      <td>
        <p>产品型号</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>tax</p>
      </td>
      <td>
        <p>税额</p>
      </td>
      <td>
        <p>tax</p>
      </td>
      <td>
        <p>税额</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>tax_description</p>
      </td>
      <td>
        <p>税率描述</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>price</p>
      </td>
      <td>
        <p>价格</p>
      </td>
      <td>
        <p>price</p>
      </td>
      <td>
        <p>价格</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>final_price</p>
      </td>
      <td>
        <p>最终价格</p>
      </td>
      <td>
        <p>final_price</p>
      </td>
      <td>
        <p>最终价格</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>weight</p>
      </td>
      <td>
        <p>重量</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
      <td>
        <p>&nbsp;</p>
      </td>
    </tr>
    <tr>
      <td>
        <p>attributes</p>
      </td>
      <td>
        <p>option=&gt;选项名 value=&gt;选项值 option_id=&gt; 选 项 ID</p>
        <p>value_id=&gt; 选 项 值ID prefix=&gt;前缀(+/-) price=&gt;属性价格</p>
      </td>
      <td>
        <p>attributes</p>
      </td>
      <td>
        <p>option=&gt; 选 项名 value=&gt; 选 项值 prefix=&gt; 前 缀(+/-) price=&gt; 属 性价格</p>
      </td>
    </tr>
  </tbody>
</table>

function query($order_id)

功能:内部从现有订单中加载 

参数:
-  $order_id:原有订单ID

function cart()

功能:将购物车转换成订单的内部方法

### order_total统计模块类 

文件:includes/classes/order_total.php

function order_total() 

功能:加载所有已安装的统计模块并对所有模块初始化

function process()

功能:运行所有已安装统计模块的统计功能,得到统计结果

返回值:数组,

数组格式 

```php
array(array(
  'code'=>'模块代码',
  'title'=>'模块名称',
  'text'=>'包含金额的最 终显示文本',
  'value'=>'统计金额',
  'sort_order'=>'排序号'))
```

function output()

功能:得到所有已安装的统计模块计算结果后的特定格式显示效果

### payment付款模块类

文件:includes/classes/payment.php

function payment($module = '') 

功能:初始化所有可用的付款模块或只初始化指定的付款模块 

参数:
-  $module:付款模块代码

function update_status()

功能:调用选定的付款模块的更新状态方法

function javascript_validation()

功能:调用选定付款模块的 javascript 验证代码(当付款模块具备验证功能时)

function checkout_initialization_method()

功能:调用选定付款模块的 checkout_initialization_method 方法,以便在付款时作好准备

> 提示:
> 
> 只有当选择了某个付款模块进行最后的结算时,才需要指定$module 参数,其余时间都将初始化 所有可用的付款模块以供用户选择

function selection()

功能:列举所有可用的付款模块(可用的定义包括:已安装和已生效)

function pre_confirmation_check()

功能:调用选定付款模块的订单确认前检查方法

function confirmation()

功能:调用选定付款模块的订单确认方法

function process_button()

功能:如果选定付款模块为在线支付类型,则输出该模块的支付网关跳转代码

function before_process()

功能:调用选定付款模块的预处理方法

function after_process()

功能:调用选定付款模块的订单生成后处理方法

function get_error()

功能:获取付款模块的错误信息

### sessions会话类

文件:includes/classes/sessions.php 

#### php3session 类

session 类用于兼容 PHP4 以下版本的环境,php3session 类可使用两种 session 机制,一种是文件类 型,另一种是用户自定义类型,这两种类型分别由 php3session_files 和 php3session_user 类管理, 通过改变 php3session 的成员$save_handler 可以切换到不同的类型

function php3session()

功能:初始化 PHP3 会话类

#### php3session_user 类

function open($save_path, $sess_name) 

功能:打开会话的实现

参数:
-  $save_path:保存路径
-  $sess_name:会话名称

function close($save_path, $sess_name)

功能:关闭会话的实现 

参数:
-  $save_path:保存路径   
-  $sess_name:会话名称

function read($sess_id)

功能:读取会话值的实现 

参数:
-  $sess_id:会话ID

function write($sess_id, $val)

功能:写入会话值的实现 

参数:
-  $sess_id:会话ID
-  $val:写入值

function destroy($sess_id)

功能:销毁会话的实现 

参数:
-  $sess_id:会话ID

function gc($max_lifetime)

功能:垃圾收集器的实现

参数:
-  $max_lifetime:有效时间(单位:秒)

> 提示:
> 
> osCommerce 系统的 SESSION 管理主要有几个参数,这些参数以常量形式存在,决定这些参数值 的因素由配置文件(includes/configure.php)管理:
> 
> - PHP_SESSION_NAME: 会话名称,此值为“osCsid”
> - PHP_SESSION_PATH: 会话路径,当使用 SSL 连接时,此值为 HTTPS_COOKIE_PATH 否
则为 HTTP_COOKIE_PATH
> - PHP_SESSION_DOMAIN: 会话域名,当使用SSL连接时,此值为 HTTPS_COOKIE_DOMAIN,否则为 HTTP_COOKIE_DOMAIN
> - PHP_SESSION_SAVE_PATH: 会话保存路径,由后台用户自行设定此值,操作路径为
Configuration->Sessions->Session Directory , 默 认 值 “ /tmp ”, 最 终 常 量 名 是
SESSION_WRITE_DIRECTORY
> - 会话有效期:默认使用PHP配置文件的session.gc_maxlifetime选项,如无此选项,则默认为 1440 秒

#### php3session_files 类

function open($save_path, $sess_name) 

功能:打开会话的实现

参数:
-  $save_path:保存路径
-  $sess_name:会话名称

function close()

功能:关闭会话的实现

function read($sess_id)

功能:读取会话值的实现 

参数:
-  $sess_id:会话ID

function write($sess_id, $val)

功能:写入会话值的实现 

参数:
-  $sess_id:会话ID
-  $val:写入值

function destroy($sess_id)

功能:销毁会话的实现 

参数:
-  $sess_id:会话ID

function gc($max_lifetime)

功能:垃圾收集器的实现

参数:
-  $max_lifetime:有效时间(单位:秒)

#### PHP3 session 管理函数 

function _session_create_id() 

功能:创建 session ID

function _session_cache_limiter()

功能:会话缓存设置

function _php_encode()

功能:加密会话值

function _php_decode($data)

功能:解密会话值

function _wddx_encode($data) 

功能:使用 WDDX 封装方式加密

function _wddx_decode($data)

功能:解密 WDDX 封装方式

> 注释:
> 
> WDDX 封装加密后的内容可以通过在下面这个例子了解到。 

例子:WDDX 封装方式

```php
<?php
$a = 1;
$b = 5.5;
$c = array("blue", "orange", "violet");
$d = "colors";
$clvars = array("c", "d");
echo wddx_serialize_vars("a", "b", $clvars);
?>
```

最终的结果为:

```xml
<wddxPacket version='1.0'>
  <header/>
  <data>
    <struct>
      <var name='a'><number>1</number></var>
      <var name='b'><number>5.5</number></var>
      <var name='c'><array length='3'><string>blue</string><string>orange</string><string>violet</string></array></var>
      <var name='d'><string>colors</string></var>
    </struct>
  </data>
</wddxPacket>
```

function session_name($name = '')

功能:设置/获取 session 名称

参数:
-  $name:要设置的名称,无为空值则返回当前session名称

function session_set_save_handler($open, $close, $read, $write, $destroy, $gc) 

功能:设置自定义的 session 管理函数

参数:
-  $open:打开会话函数名
-  $close:关闭会话函数名   
-  $read:读取会话函数名
- $write:写入会话函数名   
- $destroy:销毁会话函数名   
- $gc:垃圾收集函数名

function session_module_name($name = '')

功能:获取/设置 session 的模块名

参数:
-  $name:要设置的名称,无为空值则返回当前session的模块名称

function session_save_path($path = '') 

功能:获取/设置 session 的保存路径

function session_id($id = '') 

功能:获取/设置 session ID

function session_register($var)

功能:注册一个或多个变量为会话 

参数:
-  $var:变量名

function session_unregister($var)

功能:从当前会话中注销一个全局变量 

参数:
-  $var:变量名

function session_is_registered($var) 

功能:获取指定的变量名是否已在 session 注册 

参数:
-  $var:变量名

function session_encode()

功能:将当前会话加密成文本

function session_decode($data)

功能:从文本中解密会话值 

参数:
-  $data:加密后的文本

function session_start() 

功能:初始化 session 数据

function session_destroy() 

功能:销毁 session 的所有数据

function session_close() 

功能:关闭 session

### shipping运输模块类

文件:includes/classes/ shipping.php

function shipping($module = '') 

功能:初始化所有可用的运输模块,或只初始化选定的一个运输模块

参数:
-  $module:如为空,则初始化所有可用的运输模块,否则只初始化$module指定的运输模块

function quote($method = '', $module = '')

功能:调用所有可用的运输模块的 quote 方法实现运输的计算,或者只计算指定运输模块的的运 输

参数:
-  $method:运输模块的子运输方式,如为空,则计算所有可用的子运输方式,否则将只计算指定的子运输方式
-  $module:运输模块代码,如为空,则计算所有可用的运输模块,否则只针对指定的运输模块进行计算

function cheapest()

功能;获取所有可用运输模块中运费最便宜的运输方式结果 

返回值: `array('id'=>'运输模块代码','title'=>'运输方式名称','cost'=>'运费')`


### shoppingCart购物车类

文件:includes/classes/shopping_cart.php

function shoppingCart() 

功能:初始化购物车

function restore_contents()

功能:恢复购物车内容

只方法只适用于登录的用户,用户每次登录都会将购物车保存至数据库,这样当下次再登录时可以继续上次的购买。

如果用户未登录前在购物车里已经有产品,则登录后的购物车将会同时显示现有产品和以前的购物车产品

function reset($reset_database = false)

功能:清空购物车

参数:
-  $reset_database:是否清除数据库里的购物车信息

function add_cart($products_id, $qty = '1', $attributes = '', $notify = true)

功能:添加产品
 
参数:
-  $products_id:产品ID
-  $qty:数量
-  $attributes:二维数组,产品参数
-  $notify:是否提醒有新产品加入购物车 

例子:加入一个产品到购物车

```php
if (isset($HTTP_POST_VARS['products_id']) && is_numeric($HTTP_POST_VARS['products_id'])) {
  $cart->add_cart($HTTP_POST_VARS['products_id'], $cart->get_quantity(tep_get_uprid($HTTP_POST_VARS['products_id'], $HTTP_POST_VARS['id']))+1, $HTTP_POST_VARS['id']);
}
```

产品信息由 FORM 使用 POST 方式传递,其中产品 ID 由$_POST['products_id']传递,产品属性由 $_POST['id']传递,产品属性是一个关联数组,数组的键和值分别为产品选项 ID 和属性 ID。 $cart->get_quantity 方法可以提供查询现在产品的数量。

> 注释:
> 
> 购物车的每个产品都有唯一的识别 ID,而即使是相同产品,如果产品属性不一样的话,它们也会 具有不同的 ID,购物车将视其为不同的产品。这样的识别 ID 是通过产品 ID 和产品属性组合而成 的。
> 
> 得到产品识别 ID 的函数是 tep_get_uprid,通过解析识别 ID 得出产品 ID 的函数是 tep_get_prid。

function update_quantity($products_id, $quantity = '', $attributes = '')

功能:修改产品数量 

参数:
-  $products_id:产品ID   
-  $quantity:数量
-  $attributes:产品参数

function cleanup()

功能:清理购物车

此方法与清空购物车方法是不同的,清空购物车会清空所有购物车内的产品,而清理购物车只清除数量信息错误(数量<1)的产品。 

function count_contents()

功能:获取购物车内产品总数

function get_quantity($products_id)

功能:获取指定产品的所购数量 

参数:
-  $products_id:产品ID

function in_cart($products_id)

功能:判断购物车内是否有指定产品存在 

参数:
-  $products_id:产品ID

function remove($products_id)

功能:删除购物车内的单个产品 

参数:
-  $products_id:产品ID

function remove_all()

功能:清空购物车

remove_all 方法为 reset 方法的别名

function get_product_id_list() 

功能:获得购物车内所有产品的 ID 

返回值:字符串,产品 ID 由逗号“,”分隔的字符串

function calculate()

功能:统计购物车信息,包括总金额和总重量

总金额由成员 total 保存,总重量由成员 weight 保存,可以分别由 show_total 和 show_weight 方法 获得

function attributes_price($products_id)

功能:获取指定产品的属性发生的费用 

参数:
-  $products_id:产品ID

function get_products()

功能:获取购物车内所有产品的资料 

返回值:二维关联数组,

数组格式如下: 

```php
array(array(
  'id' => '产品 ID',
  'name' => '产品名称',
  'model' =>'产品型号',
  'image' => '产品图片',
  'price' => '产品价格',
  'quantity' => '数量', 
  'weight' => '重量', 
  'final_price' => '最终价格', 
  'tax_class_id' => '税额', 
  'attributes'=>'产品属性'
));
```

function show_total()

功能:获取当前购物车总金额

function show_weight()

功能:获取当前购物车总重量

function generate_cart_id($length = 5) 

功能:创建购物车唯一识别 ID 

参数:
-  $length:ID长度

function get_content_type()

功能:获取购物车内产品类型

当网站支持非实物产品(虚拟物品,可以通过直接下载或其它非物流方法实现交易的物品:如电 子书、点卡、充值卡号码等)交易时,可以通过此方法得知购物车内的产品总的类型。 

产品类型包括:
-  physical:实物
-  virtual:虚拟物品
-  mixed:混合类型(实物+虚拟)

function unserialize($broken)

功能:反序列化

>  注释:
>  
> 序列化(serialize)是将对象永久保存的一种方法,序列化可以实现将对象转化为字符串;反序列 化就是将字符串解析成对象的过程。
> 
> osCommerce 中除了对购物车进行序列化操作外,同时也对会话 session 和浏览历史 navigationHistory 进行了序列化。

例子:反序列化还原购物车内容

```php
$broken_cart = $cart;
$cart = new shoppingCart; $cart->unserialize($broken_cart);
```

### splitPageResults数据分页类

文件:includes/classes/split_page_results.php

function splitPageResults($query, $max_rows, $count_key = '*', $page_holder = 'page') 

功能:创建分页类

参数:
-  $query:查询SQL语句
-  $max_rows:每页显示数量
-  $count_key:用于统计的字段,如为指定则默认为“*”,即所有字段
-  $page_holder:用于传递当前页码的参数名,默认为“page”

> 说明:
> 
> splitPageResults 类实现了结果自动分页功能,它是通过分析查询 SQL 语句实现的。当前分页的指 定通过$_GET[$page_holder]来传递。使用 splitPageResults 类的基本步骤如下:
> 1. 指定查询语句和显示条目来进行初始化
> 2. 通过 splitPageResults 类的成员 number_of_rows 查询是否存在可用数据
> 3. 通过 splitPageResults 类的成员 sql_query 来进行真正的查询,并输出结果
> 4. 通过 display_count 方法输出总页数和页码
> 5. 通过 display_links 方法输出页面切换代码(包括上一页、下一页链接、单独页的链接)  splitPageResults 类的成员变量还包括:
>   page_name:用于传递当前页码的参数名,即$page_holder   number_of_rows_per_page:每页显示数量,即$max_rows   number_of_pages:结果的总页数
>   current_page_number:当前页码

> 提示:
> 
> 使用 splitPageResults 类输出分类时,要求 SQL 查询语句的关键字必须使用小写字母,如不能写成 “FROM”、“ORDER BY”,而应该写成“from”、“order by”。如果不小心使用了大写字母的关键 字,将会使得 splitPageResults 类得出的分类查询语句出现语法错误。

function display_links($max_page_links, $parameters = '')

功能:显示页面切换代码

参数:
-  $max_page_links:单独页面切换链接的最大显示数目
-  $parameters:需要附加到链接的参数,一般用于保留重要的参数传递

> 提示:
> 
> 单独页面切换链接的最大显示数目通常是由后台管理来统一设置的,可以通过修改后台 Configuration->Maximum Vaues-> Page Links 来指定。其常量名为 MAX_DISPLAY_PAGE_LINKS。
 
标准的页面切换样式如图:

function display_count($text_output)

功能:显示页码提示框,内容包括当前页码,总页数等内容 

参数:
-  $text_output:样式文本

> 提示:
> 
> 页码提示框需要显示三点内容,分别是当前页的起始数目、当前页的终止数目、总数量,标准的 样式文本为:Displaying <b>%d</b> to <b>%d</b> (of <b>%d</b> products)

标准的页码提示框样式如图:

例:使用 splitPageResults 类输出所有最新产品,实现自动分类功能,每页显示 40 个产品

```php
<table>
<?php
define('MAX_DISPLAY_PRODUCTS_NEW',40);
define('TEXT_RESULT_PAGE', 'Result Pages:'); define('TEXT_DISPLAY_NUMBER_OF_PRODUCTS_NEW', 'Displaying <b>%d</b> to <b>%d</b> (of <b>%d</b> new products)');
define('TEXT_NO_NEW_PRODUCTS', 'There are currently no products.');
$products_new_query_raw = "select p.products_id, pd.products_name, p.products_image, p.products_price, p.products_tax_class_id, p.products_date_added, m.manufacturers_name from " . TABLE_PRODUCTS . " p left join " . TABLE_MANUFACTURERS . " m on (p.manufacturers_id = m.manufacturers_id), " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' order by p.products_date_added DESC, pd.products_name";

$products_new_split = new splitPageResults($products_new_query_raw, MAX_DISPLAY_PRODUCTS_NEW);

if ($products_new_split->number_of_rows > 0) { 
?>
     <tr>
       <td><table border="0" width="100%" cellspacing="0" cellpadding="2">
        <tr>
          <td class="smallText"><?php echo $products_new_split->display_count(TEXT_DISPLAY_NUMBER_OF_PRODUCTS_NEW); ?></td>
          <td align="right" class="smallText"><?php echo TEXT_RESULT_PAGE . ' ' . $products_new_split->display_links(MAX_DISPLAY_PAGE_LINKS, tep_get_all_get_params(array('page', 'info', 'x', 'y'))); ?></td>
        </tr>
       </table></td>
</tr> <tr>
       <td><?php echo tep_draw_separator('pixel_trans.gif', '100%', '10'); ?></td>
     </tr>
<?php } ?>
<tr>
       <td><table border="0" width="100%" cellspacing="0" cellpadding="2">
<?php
if ($products_new_split->number_of_rows > 0) {
  $products_new_query = tep_db_query($products_new_split->sql_query);
  while ($products_new = tep_db_fetch_array($products_new_query)) {
    if ($new_price = tep_get_products_special_price($products_new['products_id'])) {
      $products_price = '<s>' . $currencies->display_price($products_new['products_price'], tep_get_tax_rate($products_new['products_tax_class_id'])) . '</s> <span class="productSpecialPrice">' . $currencies->display_price($new_price, tep_get_tax_rate($products_new['products_tax_class_id'])) . '</span>';
    } else {
      $products_price = $currencies->display_price($products_new['products_price'], tep_get_tax_rate($products_new['products_tax_class_id']));
    }
?>
        <tr>
          <td valign="top" class="main">$products_new['products_name']</td> <td valign="top" class="main"><?php echo '<b><u>' . $products_new['products_name'] . '</u></b></a><br>' . TEXT_PRICE . ' ' . $products_price; ?></td>
          <td align="right" valign="middle" class="main"><?php echo '<a href="' . tep_href_link(FILENAME_PRODUCTS_NEW, tep_get_all_get_params(array('action')) . 'action=buy_now&products_id=' . $products_new['products_id']) . '">' . tep_image_button('button_in_cart.gif', IMAGE_BUTTON_IN_CART) . '</a>'; ?></td>
        </tr> <tr>
          <td colspan="3"><?php echo tep_draw_separator('pixel_trans.gif', '100%', '10'); ?></td>
        </tr>
<?php
  }
} else {
?>
        <tr>
          <td class="main"><?php echo TEXT_NO_NEW_PRODUCTS; ?></td>
        </tr>
        <tr>
          <td><?php echo tep_draw_separator('pixel_trans.gif', '100%', '10');       ?></td>
        </tr>
<?php } ?>
    </table></td>
  </tr>
<?php if ($products_new_split->number_of_rows > 0) { ?>
<tr>
       <td><table border="0" width="100%" cellspacing="0" cellpadding="2">
        <tr>
          <td class="smallText"><?php echo
$products_new_split->display_count(TEXT_DISPLAY_NUMBER_OF_PRODUCTS_NEW); ?></td>
          <td align="right" class="smallText"><?php echo TEXT_RESULT_PAGE . ' ' .
$products_new_split->display_links(MAX_DISPLAY_PAGE_LINKS,
tep_get_all_get_params(array('page', 'info', 'x', 'y'))); ?></td>
        </tr>
       </table></td>
</tr>
<?php } ?>
</table>
```

> 注释:
> 
> 此处使用了 tep_get_products_special_price 函数,该函数可以得到指定产品的特价,详细参数请参见下一节《general.php 标准函数》的 tep_get_products_special_price 部分。
